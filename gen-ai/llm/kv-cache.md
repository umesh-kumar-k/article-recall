---
aliases:
  - KV Cache
Source 1: https://ngrok.com/blog/prompt-caching
---
## Prompt caching: 10x cheaper LLM tokens, but how?

## LLM Architecture
* **Key Points:**
  - At their core, LLMs are giant mathematical functions. They take a sequence of numbers as input, and produce a number as output. Inside the LLM there is an enormous graph of billions of carefully arranged operations that transform the input numbers into an output number.
* **Code Snippet:**
```python
prompt = "What is the meaning of life?";
 
tokens = tokenizer(prompt);
while (true) {
	embeddings = embed(tokens);
	for ([attention, feedforward] of transformers) {
		embeddings = attention(embeddings);
		embeddings = feedforward(embeddings);
	}
	output_token = output(embeddings);
	if (output_token === END_TOKEN) {
		break;
	}
	tokens.push(output_token);
}
 
print(decode(tokens));
```

## LLMs are surprisingly small
* **Key Points:**
  - While the above is greatly simplified, it surprised me just how few lines of code modern LLMs have.
  - One of the current leading open models, Olmo 3, comes in at only a couple hundred lines of code for example.
  - The place where prompt caching happens is in the "attention" mechanism of the transformer.
* **Technical Entities (Classes/Functions/APIs):** `Olmo 3`, `PyTorch`

## Tokenizer
* **Key Points:**
  - Before the LLM can do anything with your prompt, it needs to convert it into a representation it can work with. This is a two-step process shared between the tokenizer and the embedding stages.
  - The tokenizer takes your prompt, chops it up into small chunks, and assigns each unique chunk an integer ID called a "token."
  - The same prompt always results in the same tokens. Tokens are also case-sensitive, and this is because capitalisation tells you something about the word.
  - Tokens are the fundamental unit of input and output for LLMs. When you ask ChatGPT a question, the response is streamed back to you one token at a time as each iteration of the LLM is completed.
  - LLMs need all of the context to produce good answers. If we only fed the prompt in, it would continually try to produce the first token of the answer. If we only fed the answer in, it would immediately forget the question. The whole prompt + the answer need to be fed into the LLM, every single iteration.
  - Inference has to stop at some point. LLMs have a variety of "special" tokens they can output, one of which signals the end of a response. In the GPT-5 tokenizer this is token 199999.
  - There are also special tokens for denoting the start and end of conversational messages, and these are how chat models like ChatGPT and Claude know when one message ends and another begins.
  - There are lots of tokenizers! The tokenizer that ChatGPT uses is different to the one Claude uses. Even different models made by OpenAI use different tokenizers. Each tokenizer has its own rules for splitting text into tokens.
* **Technical Entities (Classes/Functions/APIs):** `GPT-5 tokenizer`, `tokenizer`, `tiktokenizer`

## Embedding
* **Key Points:**
  - An embedding is an array of length n representing a position in n-dimensional space. If n were 3, an embedding might be [10, 4, 2] representing the location x=10, y=4, z=2 in 3 dimensional space. When LLMs are being trained, each token gets assigned a random starting location in this space, and the training process nudges all of the tokens around until it finds an arrangement that produces the best outputs.
  - The embedding stage starts out by looking up each token's embedding.
  - So we take tokens, an array of integers, and convert it into an array of embeddings. An array of arrays, or a "matrix."
  - The more dimensions we give the embeddings, the more dimensions it has to compare sentences with. We've been talking about embeddings with 3 dimensions, but current models have embeddings with thousands of dimensions. The biggest ones have more than 10,000.
  - After fetching a token's embeddings, it encodes the token's position within the prompt into the embeddings. I didn't dig deeply into how this works other than enough to know it doesn't make much difference to how prompt caching works, but without it the LLM would not be able to tell the order of the tokens in the prompt.
* **Code Snippet:**
```javascript
// Created during training, never changes during inference.
const EMBEDDINGS = [...];
 
function embed(tokens) {
	return tokens.map(token => {
		return EMBEDDINGS[token];
	});
}
```

## Transformer
* **Key Points:**
  - The transformer stage is all about taking embeddings as input and moving them around their n-dimensional space. It does this in two ways, and we're only going to focus on the first: attention.
  - The job of the attention mechanism is to help the LLM understand the relationships between each token in the prompt, by allowing tokens to influence each others' positions in n-dimensional space. It does this by combining the embeddings of the prompt's tokens in a weighted fashion. The input is an entire prompt's embeddings, and the output is a single new embedding that is a weighted combination of all of the input embeddings.
  - Most of the calculations in attention are matrix multiplications. The only thing you need to know about matrix multiplication for this post is that the shape of the output matrix is determined by the shapes of the input matrices. The output always has the same number of rows as the first input matrix and the same number of columns as the second input matrix.
  - The first problem with this matrix of scores is that it allows future tokens to influence the past. To fix this, we apply a triangular mask to the matrix to zero out future tokens.
  - The second problem is that these scores are arbitrary numbers. They would be much more useful to us if they were a distribution that sums to 1 across each row. This is exactly what the softmax function does.
* **Technical Entities (Classes/Functions/APIs):** `WQ`, `WK`, `WV`, `attentionWeights()`, `softmax`, `transpose`
* **Code Snippet:**
```javascript
// Similar to EMBEDDINGS from the pseudocode
// earlier, WQ and WK are learned during 
// training and do not change during inference.
// 
// These are both n*n matrices, where n is the
// number of embedding dimensions. In our example
// above, n = 3.
const WQ = [[...], [...], [...]];
const WK = [[...], [...], [...]];
 
// The input embeddings look like this:
// [
//   [-0.1, 0.1, -0.3], // Mary
//   [1.0, -0.5, -0.6], // had
//   [0.0, 0.8, 0.6],   // a
//   [0.5, -0.7, 1.0]   // little
// ]
function attentionWeights(embeddings) {
	const Q = embeddings * WQ;
	const K = embeddings * WK;
	const scores = Q * transpose(K);
	const masked = mask(scores);
	return softmax(masked);
}
```

## Prompt caching
* **Key Points:**
  - Every new token is appended to the input and reprocessed in full. But look closely, play the animation back a few times: none of previous weights change. We're redoing lots of calculations we don't need to.
  - You can avoid these duplicate calculations by making two changes to the inference loop:
    - Cache the K and V matrices every iteration.
    - Only feed the newest token into the model, not the entire prompt.
  - The data that gets cached is the result of embeddings * WK and embeddings * WV, so K and V. As a result, prompt caching tends to be called "KV caching."
  - That's it, those K and V matrices above, they are the 1s and 0s that the providers save in their giant datacenters to offer us 10x cheaper tokens, and much faster responses.
  - Providers hold on to these matrices for each prompt for 5-10 minutes after the request is made, and if you send a new request that starts with the same prompt, they reuse the cached K and V rather than recalculating them. What's really cool is that you can partially match a cache entry and still use the bit that matched, not the whole thing.
  - OpenAI and Anthropic do caching very differently from each other. OpenAI do it all automatically for you, attempting to route requests to cached entries when possible.
  - Anthropic give you more control, letting you decide when to cache and for how long.
* **Technical Entities (Classes/Functions/APIs):** `KV caching`, `K`, `V`, `WK`, `WV`, `embeddings`

## Wait, what about temperature?
* **Key Points:**
  - There are a variety of parameters that LLM providers offer you to control the randomness of what a model will produce. Common ones are temperature, top_p, and top_k. These parameters all affect the final step of the inference loop, where the model picks a token based on the probabilities it has assigned to each token in its vocabulary. This happens after the attention mechanism has produced the final embeddings, so prompt caching is unaffected by these parameters. You can change them freely without worrying about invalidating your cached prompts.
* **Technical Entities (Classes/Functions/APIs):** `temperature`, `top_p`, `top_k`

## Conclusion
* **Key Points:**
  - I had a blast learning everything I've presented in this post. LLMs are fascinating technology and I think, as an industry, we've only scratched the surface of what they can do.
