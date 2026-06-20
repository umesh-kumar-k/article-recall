---
aliases:
  - Prompt Injection
Source 1: https://www.paloaltonetworks.in/cyberpedia/what-is-a-prompt-injection-attack
Source 2: https://genai.owasp.org/llmrisk/llm01-prompt-injection/
---

# LLM01:2025 Prompt Injection


* **Key Points:**
  - "A Prompt Injection Vulnerability occurs when user prompts alter the LLM's behavior or output in unintended ways."
  - "These inputs can affect the model even if they are imperceptible to humans, therefore prompt injections do not need to be human-visible/readable, as long as the content is parsed by the model."
  - "Prompt Injection vulnerabilities exist in how models process prompts, and how input may force the model to incorrectly pass prompt data to other parts of the model, potentially causing them to violate guidelines, generate harmful content, enable unauthorized access, or influence critical decisions."
  - "While techniques like Retrieval Augmented Generation (RAG) and fine-tuning aim to make LLM outputs more relevant and accurate, research shows that they do not fully mitigate prompt injection vulnerabilities."
  - "While prompt injection and jailbreaking are related concepts in LLM security, they are often used interchangeably. Prompt injection involves manipulating model responses through specific inputs to alter its behavior, which can include bypassing safety measures. Jailbreaking is a form of prompt injection where the attacker provides inputs that cause the model to disregard its safety protocols entirely."
  - "Developers can build safeguards into system prompts and input handling to help mitigate prompt injection attacks, but effective prevention of jailbreaking requires ongoing updates to the model's training and safety mechanisms."
* **Technical Entities (Classes/Functions/APIs):** `RAG` (Retrieval Augmented Generation)

## Types of Prompt Injection Vulnerabilities
### Direct Prompt Injections
* **Key Points:**
  - "Direct prompt injections occur when a user's prompt input directly alters the behavior of the model in unintended or unexpected ways. The input can be either intentional (i.e., a malicious actor deliberately crafting a prompt to exploit the model) or unintentional (i.e., a user inadvertently providing input that triggers unexpected behavior)."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Indirect Prompt Injections
* **Key Points:**
  - "Indirect prompt injections occur when an LLM accepts input from external sources, such as websites or files. The content may have in the external content data that when interpreted by the model, alters the behavior of the model in unintended or unexpected ways. Like direct injections, indirect injections can be either intentional or unintentional."
  - "The severity and nature of the impact of a successful prompt injection attack can vary greatly and are largely dependent on both the business context the model operates in, and the agency with which the model is architected. Generally, however, prompt injection can lead to unintended outcomes, including but not limited to: Disclosure of sensitive information; Revealing sensitive information about AI system infrastructure or system prompts; Content manipulation leading to incorrect or biased outputs; Providing unauthorized access to functions available to the LLM; Executing arbitrary commands in connected systems; Manipulating critical decision-making processes"
  - "The rise of multimodal AI, which processes multiple data types simultaneously, introduces unique prompt injection risks. Malicious actors could exploit interactions between modalities, such as hiding instructions in images that accompany benign text. The complexity of these systems expands the attack surface. Multimodal models may also be susceptible to novel cross-modal attacks that are difficult to detect and mitigate with current techniques. Robust multimodal-specific defenses are an important area for further research and development."
* **Technical Entities (Classes/Functions/APIs):** None specified

## Prevention and Mitigation Strategies
* **Key Points:**
  - "Prompt injection vulnerabilities are possible due to the nature of generative AI. Given the stochastic influence at the heart of the way models work, it is unclear if there are fool-proof methods of prevention for prompt injection. However, the following measures can mitigate the impact of prompt injections:"
* **Technical Entities (Classes/Functions/APIs):** None specified

### 1. Constrain model behavior
* **Key Points:**
  - "Provide specific instructions about the model's role, capabilities, and limitations within the system prompt. Enforce strict context adherence, limit responses to specific tasks or topics, and instruct the model to ignore attempts to modify core instructions."
* **Technical Entities (Classes/Functions/APIs):** None specified

### 2. Define and validate expected output formats
* **Key Points:**
  - "Specify clear output formats, request detailed reasoning and source citations, and use deterministic code to validate adherence to these formats."
* **Technical Entities (Classes/Functions/APIs):** None specified

### 3. Implement input and output filtering
* **Key Points:**
  - "Define sensitive categories and construct rules for identifying and handling such content. Apply semantic filters and use string-checking to scan for non-allowed content. Evaluate responses using the RAG Triad: Assess context relevance, groundedness, and question/answer relevance to identify potentially malicious outputs."
* **Technical Entities (Classes/Functions/APIs):** `RAG Triad` (context relevance, groundedness, question/answer relevance)

### 4. Enforce privilege control and least privilege access
* **Key Points:**
  - "Provide the application with its own API tokens for extensible functionality, and handle these functions in code rather than providing them to the model. Restrict the model's access privileges to the minimum necessary for its intended operations."
* **Technical Entities (Classes/Functions/APIs):** `API tokens`

### 5. Require human approval for high-risk actions
* **Key Points:**
  - "Implement human-in-the-loop controls for privileged operations to prevent unauthorized actions."
* **Technical Entities (Classes/Functions/APIs):** None specified

### 6. Segregate and identify external content
* **Key Points:**
  - "Separate and clearly denote untrusted content to limit its influence on user prompts."
* **Technical Entities (Classes/Functions/APIs):** None specified

### 7. Conduct adversarial testing and attack simulations
* **Key Points:**
  - "Perform regular penetration testing and breach simulations, treating the model as an untrusted user to test the effectiveness of trust boundaries and access controls."
* **Technical Entities (Classes/Functions/APIs):** None specified

## Example Attack Scenarios
* **Key Points:**
  - "Scenario #1: Direct Injection: An attacker injects a prompt into a customer support chatbot, instructing it to ignore previous guidelines, query private data stores, and send emails, leading to unauthorized access and privilege escalation."
  - "Scenario #2: Indirect Injection: A user employs an LLM to summarize a webpage containing hidden instructions that cause the LLM to insert an image linking to a URL, leading to exfiltration of the the private conversation."
  - "Scenario #3: Unintentional Injection: A company includes an instruction in a job description to identify AI-generated applications. An applicant, unaware of this instruction, uses an LLM to optimize their resume, inadvertently triggering the AI detection."
  - "Scenario #4: Intentional Model Influence: An attacker modifies a document in a repository used by a Retrieval-Augmented Generation (RAG) application. When a user's query returns the modified content, the malicious instructions alter the LLM's output, generating misleading results."
  - "Scenario #5: Code Injection: An attacker exploits a vulnerability (CVE-2024-5184) in an LLM-powered email assistant to inject malicious prompts, allowing access to sensitive information and manipulation of email content."
  - "Scenario #6: Payload Splitting: An attacker uploads a resume with split malicious prompts. When an LLM is used to evaluate the candidate, the combined prompts manipulate the model's response, resulting in a positive recommendation despite the actual resume contents."
  - "Scenario #7: Multimodal Injection: An attacker embeds a malicious prompt within an image that accompanies benign text. When a multimodal AI processes the image and text concurrently, the hidden prompt alters the model's behavior, potentially leading to unauthorized actions or disclosure of sensitive information."
  - "Scenario #8: Adversarial Suffix: An attacker appends a seemingly meaningless string of characters to a prompt, which influences the LLM's output in a malicious way, bypassing safety measures."
  - "Scenario #9: Multilingual/Obfuscated Attack: An attacker uses multiple languages or encodes malicious instructions (e.g., using Base64 or emojis) to evade filters and manipulate the LLM's behavior."
* **Technical Entities (Classes/Functions/APIs):** `RAG` (Retrieval-Augmented Generation), `CVE-2024-5184`

# What Is a Prompt Injection Attack? [Examples & Prevention]


* **Key Points:**
  - "A prompt injection attack is a GenAI security threat where an attacker deliberately crafts and inputs deceptive text into a large language model (LLM) to manipulate its outputs."
  - "This type of attack exploits the model's response generation process to achieve unauthorized actions, such as extracting confidential information, injecting false content, or disrupting the model's intended function."
* **Technical Entities (Classes/Functions/APIs):** None specified

## How does a prompt injection attack work?
* **Key Points:**
  - "A prompt injection attack is a type of GenAI security threat that happens when someone manipulates user input to trick an AI model into ignoring its intended instructions."
  - "Normally, large language models (LLMs) respond to inputs based on built-in prompts provided by developers. The model treats these built-in prompts and user-entered inputs as a single combined instruction."
  - "Because the model can't distinguish developer instructions from user input. Which means an attacker can take advantage of this confusion to insert harmful instructions."
  - "We recently assessed mainstream large language models (LLMs) against prompt-based attacks, which revealed significant vulnerabilities. Three attack vectors—guardrail bypass, information leakage, and goal hijacking—demonstrated consistently high success rates across various models. In particular, some attack techniques achieved success rates exceeding 50% across models of different scales, from several-billion parameter models to trillion-parameter models, with certain cases reaching up to 88%."
* **Technical Entities (Classes/Functions/APIs):** None specified

## What are the different types of prompt injection attacks?
* **Key Points:**
  - "At a high level, prompt injection attacks generally fall into two main categories: Direct prompt injection" and "Indirect prompt injection"
* **Technical Entities (Classes/Functions/APIs):** None specified

### Direct prompt injection
* **Key Points:**
  - "Direct prompt injection happens when an attacker explicitly enters a malicious prompt into the user input field of an AI-powered application."
  - "Basically, the attacker provides instructions directly that override developer-set system instructions."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Indirect prompt injection
* **Key Points:**
  - "Indirect prompt injection involves placing malicious commands in external data sources that the AI model consumes, such as webpages or documents."
  - "Because the model can unknowingly pick up hidden commands when it reads or summarizes external content."
  - "There are also stored prompt injection attacks—a type of indirect prompt injection. Stored prompt injection attacks embed malicious prompts directly into an AI model's memory or training dataset, like so."
  - "Important: This can affect the model's responses long after the initial insertion of malicious data."
* **Technical Entities (Classes/Functions/APIs):** None specified

## Examples of prompt injection attacks
* **Key Points:**
  - "Prompt injection isn't limited to a single tactic. Attackers use a wide range of techniques to manipulate how large language models interpret and respond to input."
  - "Prompt injection techniques are evolving constantly. The below list is foundational, but not exhaustive."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Prompt injection techniques
* **Key Points:**
  - Table of attack types: Code injection, Payload splitting, Multimodal injection, Multilingual/obfuscated attack, Model data extraction, Template manipulation, Fake completion (guiding the LLM to disobedience), Reformatting, Exploiting LLM friendliness and trust
  - "Code injection: An attacker injects executable code into an LLM's prompt to manipulate its responses or execute unauthorized actions."
  - "Payload splitting: A malicious prompt is split into multiple inputs that, when processed together, produce an attack."
  - "Multimodal injection: An attacker embeds a prompt in an image, audio, or other non-textual input, tricking the LLM into executing unintended actions."
  - "Multilingual/obfuscated attack: Malicious inputs are encoded in different languages or obfuscation techniques (e.g., Base64, emojis) to evade detection."
  - "Model data extraction: Attackers extract system prompts, conversation history, or other hidden instructions to refine future attacks."
  - "Template manipulation: Manipulating the LLM's predefined system prompts to override intended behaviors or introduce malicious directives."
  - "Fake completion (guiding the LLM to disobedience): An attacker inserts pre-completed responses that mislead the model, causing it to ignore original instructions."
  - "Reformatting: Changing the input or output format of an attack to bypass security filters while maintaining malicious intent."
  - "Exploiting LLM friendliness and trust: Leveraging persuasive language or social engineering techniques to convince the LLM to execute unauthorized actions."
* **Technical Entities (Classes/Functions/APIs):** None specified

## What is the difference between prompt injections and jailbreaking?
* **Key Points:**
  - "Prompt injection and jailbreaking are both techniques that manipulate AI behavior. However: They work in different ways and have different goals."
  - "Prompt injection happens when an attacker embeds malicious instructions into an AI's input field to override its original programming. So the goal is to make the AI ignore prior instructions and follow the attacker's command instead."
  - "Jailbreaking is a technique used to remove or bypass an AI system's built-in safeguards. The goal is to make the model ignore its ethical constraints or safety filters."
  - "Prompt injections target how the AI processes input. Jailbreaking targets what the AI is allowed to generate. While the two techniques can be used together, they're distinct in purpose and execution."
  - "Deceptive Delight works by blending restricted or unsafe topics within seemingly harmless content, framing them in a positive and innocuous way. This approach causes LLMs to overlook the problematic elements and produce responses that include unsafe content."
  - "Deceptive Delight is a multi-turn attack method that involves iterative interactions with the target model to trick it into generating unsafe content. The technique requires at least two turns. However: Adding a third turn often increases the severity, relevance and detail of the harmful output."
* **Technical Entities (Classes/Functions/APIs):** None specified

## What are the potential consequences of prompt injection attacks?
* **Key Points:**
  - "Prompt injection attacks pose significant risks to AI-driven systems, including exposing sensitive data, altering outputs, and even enabling unauthorized access."
  - "Even minor prompt manipulations can have outsized impacts. For example, imagine a healthcare system providing incorrect dosage guidance, a financial model making flawed investment recommendations, or a manufacturing predictive system misjudging supply chain risks."
  - "The potential consequences of prompt injection attacks include: Data exfiltration, Data poisoning, Data theft, Response corruption, Remote code execution, Misinformation propagation, Malware transmission"
* **Technical Entities (Classes/Functions/APIs):** None specified

### Data exfiltration
* **Key Points:**
  - "Attackers can manipulate AI systems into disclosing confidential data, including business strategies, customer records, or security credentials. If the AI is vulnerable, it may reveal information that should remain protected."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Data poisoning
* **Key Points:**
  - "Malicious actors can inject false or biased data into an AI model, gradually distorting its outputs. Over time, this manipulation can degrade the model's reliability, leading to inaccurate predictions or flawed decision-making."
  - "If stakeholders cannot rely on the outputs of GenAI systems, organizations risk reputational damage, regulatory noncompliance, and the erosion of user confidence."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Data theft
* **Key Points:**
  - "A compromised AI system could expose intellectual property, proprietary algorithms, or other valuable information. Attackers may use prompt injection to extract strategic insights, financial projections, or internal documentation that could lead to financial or competitive losses."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Response corruption
* **Key Points:**
  - "Prompt injections can cause AI models to generate false or misleading responses. Which can impact decision-making in applications that rely on AI-generated insights."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Remote code execution
* **Key Points:**
  - "If an AI system is integrated with external tools that execute commands, an attacker may manipulate it into running unauthorized code. And that means threat actors could potentially deploy malicious software, extract sensitive information, or take control of connected systems."
  - "Remote code execution is only possible in specific conditions where an AI system is connected to executable environments. If an AI-powered system is linked to external plugins, automation scripts, or command execution tools, an attacker could theoretically inject prompts that trick the AI into running unintended commands."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Misinformation propagation
* **Key Points:**
  - "Malicious prompts can manipulate AI systems into spreading false or misleading content. If an AI-generated source is considered reliable, this could impact public perception, decision-making, or reputational trust."
  - "Misinformation propagation through prompt injection can have far-reaching consequences, particularly when AI-generated content is perceived as authoritative. Attackers can manipulate AI responses to spread false narratives, which may influence public opinion, financial markets, or even political events."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Malware transmission
* **Key Points:**
  - "AI-driven assistants and chatbots can be manipulated into distributing malicious content. A crafted prompt could direct an AI system to generate or forward harmful links, tricking users into interacting with malware or phishing scams."
* **Technical Entities (Classes/Functions/APIs):** None specified

## How to prevent prompt injection: best practices, tips, and tricks
* **Key Points:**
  - "Prompt injection attacks exploit the way large language models (LLMs) process user input. Because these models interpret both system instructions and user prompts in natural language, they're inherently vulnerable to manipulation."
  - "Caring about prompt attacks isn't just a technical consideration; it's a strategic imperative. Without a keen focus on mitigation, the promise of GenAI could be overshadowed by the risks of its misuse. Addressing these vulnerabilities now is vital to safeguarding innovation, protecting sensitive information, maintaining regulatory compliance, and upholding public trust in a world increasingly shaped by intelligent automation."
  - "While there's no foolproof method to eliminate prompt injection entirely, there are several strategies that significantly reduce the risk."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Constrain model behavior
* **Key Points:**
  - "LLMs should have strict operational boundaries. The system prompt should clearly define the model's role, capabilities, and limitations."
  - "Define clear operational boundaries in system prompts. Prevent persona switching by restricting the AI's ability to alter its identity or task. Instruct the AI to reject modification attempts to its system-level behavior. Use session-based controls to reset interactions and prevent gradual manipulation. Regularly audit system prompts to ensure they remain secure and effective."
  - "Use dynamic prompt injection detection alongside static rules. While setting strict operational boundaries is essential, integrating a real-time classifier that flags suspicious user inputs can further reduce risks."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Define and enforce output formats
* **Key Points:**
  - "Restricting the format of AI-generated responses helps prevent prompt injection from influencing the model's behavior. Outputs should follow predefined templates, ensuring that the model can't return unexpected or manipulated information."
  - "Implement strict response templates to limit AI-generated outputs. Enforce consistent formatting rules to prevent response manipulation. Validate responses against predefined safe patterns before displaying them. Limit open-ended generative outputs in high-risk applications. Integrate post-processing checks to detect unexpected output behavior."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Implement input validation and filtering
* **Key Points:**
  - "User input should be validated before being processed by the AI. This includes detecting suspicious characters, encoded messages, and obfuscated instructions."
  - "Use regular expressions and pattern matching to detect malicious input. Apply semantic filtering to flag ambiguous or deceptive prompts. Escape special characters to prevent unintended instruction execution. Implement rate limiting to block repeated manipulation attempts. Deploy AI-driven anomaly detection to identify unusual user behavior. Reject or flag encoded or obfuscated text, such as Base64 or Unicode variations."
  - "Use a multi-layered filtering approach. Simple regex-based filtering may not catch sophisticated attacks. Combine keyword-based detection with NLP-based anomaly detection for a more robust defense."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Enforce least privilege access
* **Key Points:**
  - "LLMs should operate with the minimum level of access required to perform their intended tasks. If an AI system integrates with external tools, it shouldn't have unrestricted access to databases, APIs, or privileged operations."
  - "Restrict API permissions to only essential functions. Store authentication tokens securely, ensuring they aren't exposed to the model. Limit LLM interactions to non-sensitive environments whenever possible. Implement role-based access control (RBAC) to limit user permissions. Use sandboxed environments to test and isolate model interactions."
  - "Regularly audit access logs to detect unusual patterns. Even with strict privilege controls, periodic reviews help identify whether an AI system is being probed or exploited through prompt injection attempts."
* **Technical Entities (Classes/Functions/APIs):** `RBAC` (role-based access control)

### Require human oversight for high-risk actions
* **Key Points:**
  - "AI-generated actions that could result in security risks should require human approval. This is especially important for tasks that involve modifying system settings, retrieving sensitive data, or executing external commands."
  - "Implement human-in-the-loop (HITL) controls for privileged operations. Require manual review for AI-generated outputs in security-critical functions. Assign risk scores to determine which actions require human validation. Use multi-step verification before AI can perform sensitive operations. Establish audit logs that track approvals and AI-generated actions."
* **Technical Entities (Classes/Functions/APIs):** `HITL` (human-in-the-loop)

### Segregate and identify external content
* **Key Points:**
  - "AI models that retrieve data from external sources should treat untrusted content differently from system-generated responses. This ensures that injected prompts from external documents, web pages, or user-generated content don't influence the model's primary instructions."
  - "Clearly tag and isolate external content from system-generated data. Prevent user-generated content from modifying model instructions. Use separate processing pipelines for internal and external sources. Enforce content validation before incorporating external data into AI responses. Label unverified data sources to prevent implicit trust by the AI."
  - "Use data provenance tracking. Keeping a record of where external content originates helps determine whether AI-generated outputs are based on untrusted or potentially manipulated sources."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Conduct adversarial testing and attack simulations
* **Key Points:**
  - "Regular testing helps identify vulnerabilities before attackers exploit them. Security teams should simulate prompt injection attempts by feeding the model a variety of adversarial AI prompts."
  - "Perform penetration testing using real-world adversarial input. Conduct red teaming exercises to simulate advanced attack methods. Utilize AI-driven attack simulations to assess model resilience. Update security policies and model behavior based on test results. Analyze past attack patterns to improve future model defenses."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Monitor and log AI interactions
* **Key Points:**
  - "Continuously monitoring AI-generated interactions helps detect unusual patterns that may indicate a prompt injection attempt. Logging user queries and model responses provides a record that can be analyzed for security incidents."
  - "Deploy real-time monitoring tools for all AI interactions. Use anomaly detection algorithms to flag suspicious activity. Maintain detailed logs, including timestamps, input history, and output tracking. Automate alerts for unusual or unauthorized AI behavior. Conduct regular log reviews to identify potential security threats."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Regularly update security protocols
* **Key Points:**
  - "AI security is an evolving field. New attack techniques emerge regularly, so it's absolutely essential to update security measures."
  - "Apply frequent security patches and updates to AI frameworks. Adjust prompt engineering strategies to counter evolving attack techniques. Conduct routine security audits to identify and remediate weaknesses. Stay informed on emerging AI threats and best practices. Establish a proactive incident response plan for AI security breaches."
  - "Test security updates in a sandboxed AI environment before deployment. This ensures that new patches don't inadvertently introduce vulnerabilities while attempting to fix existing ones."
* **Technical Entities (Classes/Functions/APIs):** None specified

### Train models to recognize malicious input
* **Key Points:**
  - "AI models can be fine-tuned to recognize and reject suspicious prompts. By training models on adversarial examples, they become more resistant to common attack patterns."
  - "Implement adversarial training using real-world attack examples. Use real-time input classifiers to detect manipulation attempts. Continuously update training data to adapt to evolving threats. Conduct ongoing testing to ensure the model rejects harmful instructions."
  - "Leverage reinforcement learning from human feedback (RLHF) to refine AI security. Continuous feedback from security experts can help train AI models to reject increasingly sophisticated prompt injection attempts."
* **Technical Entities (Classes/Functions/APIs):** `RLHF` (reinforcement learning from human feedback)

### User education and awareness
* **Key Points:**
  - "Attackers often rely on social engineering to make prompt injection more effective. If users are unaware of these risks, they may unintentionally aid an attack by interacting with an AI system in ways that make it easier to exploit."
  - "Train users to recognize suspicious AI interactions. Educate teams on safe AI usage. Set clear guidelines for AI interactions. Promote skepticism in AI-generated outputs. Encourage security teams to monitor AI adoption."
* **Technical Entities (Classes/Functions/APIs):** None specified

## A brief history of prompt injection
* **Key Points:**
  - "Prompt injection was first identified in early 2022, when researchers at Preamble discovered that large language models (LLMs) were susceptible to malicious instructions hidden within user prompts."
  - "In September 2022, data scientist Riley Goodside independently rediscovered the flaw and shared his findings online, drawing widespread attention to the issue."
  - "Shortly after, Simon Willison formally coined the term 'prompt injection' to describe the attack."
  - "Researchers continued studying variations of the technique, and in early 2023, Kai Greshake and colleagues introduced the concept of indirect prompt injection, which demonstrated that AI models could be manipulated through external data sources, not just direct user input."
  - "Since then, prompt injection has remained a major concern in AI security, prompting ongoing research into mitigation strategies."
* **Technical Entities (Classes/Functions/APIs):** None specified