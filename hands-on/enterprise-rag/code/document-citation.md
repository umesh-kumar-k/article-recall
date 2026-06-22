---
aliases:
  - Citations
Source 1: https://docs.cohere.com/docs/rag-citations
Source 2: https://dev.to/tensorlake/make-rag-provable-page-bbox-citations-for-all-extracted-data-4ipc
---
# Verify Structured Output with Field-Level Citations

## Introduction
* **Key Points:**
  - Missing evidence is one of the biggest blockers in production AI workflows.
  - It's not enough to say what a document claims, you need to show where in the source that claim came from. Whether you're auditing bank statements, verifying medical referral forms, or investigating fraud, traceability is a hard requirement.
  - When provide_citations=True, every extracted field includes: Page number; Bounding box (bbox) coordinates.
  - This means structured outputs are no longer just machine-readable; they're auditable, verifiable, and traceable back to the source document.
* **Technical Entities (Classes/Functions/APIs):** `StructuredExtractionOptions`, `provide_citations=True`

## Traceable Context Means Trustworthy RAG
* **Key Points:**
  - In many workflows, "close enough" isn't good enough. Teams need confidence that extracted values align with the document's ground truth.
  - Banking & Finance: Auditors need to understand exactly which account, statement, or transaction produced a reported number. If an account balance doesn't reconcile, citations let you trace back to the precise page and bounding box where the discrepancy originates. No more guesswork in backtracking totals.
  - Fraud Detection: When anomalies appear in reported values, bounding-box citations provide the evidence trail. Investigators can quickly verify whether a suspicious number came from an altered document, a duplicated entry, or a genuine filing.
  - Healthcare & Forms Processing: At UCLA, teams processing medical referral forms wanted faster verification of ground truth. With citations, a structured field (like "referral date" or "doctor's signature") can point directly to the page span and bounding box where it was found, cutting human review time dramatically.
  - In short: Citations turn structured extraction into a compliance-grade tool.

## Implement Citations with One Line of Code
* **Key Points:**
  - Let's take a simple example: extracting transaction summaries from a bank statement.
  - Each field is now annotated with a citation: the page number and bounding-box coordinates.
* **Technical Entities (Classes/Functions/APIs):** `Tensorlake DocumentAI`, `StructuredExtractionOptions`, `BankStatement`, `provide_citations=True`, `parse_and_wait`
* **Code Snippet:**
```python
from tensorlake.documentai import DocumentAI, StructuredExtractionOptions
from pydantic import BaseModel, Field
from typing import List

class Transaction(BaseModel):
    date: str = Field(description="Transaction date")
    description: str = Field(description="Transaction description")
    amount: float = Field(description="Transaction amount")

class BankStatement(BaseModel):
    transactions: List[Transaction]

doc_ai = DocumentAI()

structured_extraction_options = [
    StructuredExtractionOptions(
        schema_name="BankStatement",
        json_schema=BankStatement,
        provide_citations=True   # <-- new parameter
    )
]

result = doc_ai.parse_and_wait(
    file="https://tlake.link/documents/bank-statement",
    structured_extraction_options=structured_extraction_options
)

print(result.structured_data[0].data)
```

## From Data to Evidence
* **Key Points:**
  - "In insurance, structured outputs power our workflows, but people still verify. With field-level citations, reviewers can jump from a data row straight to the exact COI or endorsement language. That's the difference between 'parsed' and provable." — Jesse McClure, CTO and Co-Founder, Sublynk
  - Citations aren't just nice-to-have, our customers across industries know that they unlock new workflows:
    - Audit-ready outputs: Every number is backed by ground-truth evidence.
    - Automated review: Flag discrepancies automatically and point reviewers directly to the source.
    - Explainability in RAG/Agents: Don't just return answers—return the highlighted document snippets.
    - UI Enhancements: Build document viewers that highlight the exact fields extracted.
  - The benefit is twofold: engineers can build more reliable systems and stakeholders (auditors, compliance teams, regulators) get confidence and transparency.

## Traceability Built In
* **Key Points:**
  - With the new provide_citations parameter, structured extraction becomes not only machine-readable but also evidence-backed.
  - Every field can now point back to its exact source location in the document, making Tensorlake the foundation for audit-ready, compliance-grade, and fraud-resistant AI workflows.

## RAG Citations

### Accessing citations
* **Key Points:**
  - The Chat endpoint generates fine-grained citations for its RAG response. This capability is included out-of-the-box with the Command family of models.

#### Non-streaming
* **Key Points:**
  - First, define the documents to be passed as the context of the model's response.
  - In the non-streaming mode (using chat to generate the model response), the citations are provided in the message.citations field of the response object.
  - Each citation object contains: start and end: the start and end indices of the text that cites a source(s); text: its corresponding span of text; sources: the source(s) that it references
* **Technical Entities (Classes/Functions/APIs):** `cohere`, `ClientV2`, `chat`, `message.citations`
* **Code Snippet:**
```python
import cohere
import json
co = cohere.ClientV2("COHERE_API_KEY")

documents = [
    {
        "data": {
            "title": "Tall penguins",
            "snippet": "Emperor penguins are the tallest.",
        }
    },
    {
        "data": {
            "title": "Penguin habitats",
            "snippet": "Emperor penguins only live in Antarctica.",
        }
    },
]

messages = [
    {"role": "user", "content": "Where do the tallest penguins live?"}
]
response = co.chat(
    model="command-r-08-2024",
    messages=messages,
    documents=documents,
)
print(response.message.content[0].text)
for citation in response.message.citations:
    print(citation, "\n")
```

#### Streaming
* **Key Points:**
  - In a streaming scenario (using chat_stream to generate the model response), the citations are provided in the citation-start events.
  - Each citation object contains the same fields as the non-streaming scenario.
* **Technical Entities (Classes/Functions/APIs):** `chat_stream`, `citation-start`, `content-delta`
* **Code Snippet:**
```python
messages = [
    {"role": "user", "content": "Where do the tallest penguins live?"}
]
response = co.chat_stream(
    model="command-a-plus-05-2026",
    messages=messages,
    documents=documents,
)
response_text = ""
citations = []
for chunk in response:
    if chunk:
        if chunk.type == "content-delta":
            response_text += chunk.delta.message.content.text
            print(chunk.delta.message.content.text, end="")
        if chunk.type == "citation-start":
            citations.append(chunk.delta.message.citations)
for citation in citations:
    print(citation, "\n")
```

#### Document ID
* **Key Points:**
  - When passing the documents as context, you can optionally add custom IDs to the id field in the document object. These IDs will be used by the endpoint as the citation reference.
  - If you don't provide the id field, the ID will be auto-generated in the format of doc:<auto_generated_id>. Example: doc:0.
* **Code Snippet:**
```python
documents = [
    {
        "data": {
            "title": "Tall penguins",
            "snippet": "Emperor penguins are the tallest.",
        },
        "id": "100",
    },
    {
        "data": {
            "title": "Penguin habitats",
            "snippet": "Emperor penguins only live in Antarctica.",
        },
        "id": "101",
    },
]
```

### Citation modes
* **Key Points:**
  - When running RAG in streaming mode, it's possible to configure how citations are generated and presented. You can choose between fast citations or accurate citations, depending on your latency and precision needs.

#### Accurate citations
* **Key Points:**
  - The model produces its answer first, and then, after the entire response is generated, it provides citations that map to specific segments of the response text. This approach may incur slightly higher latency, but it ensures the citation indices are more precisely aligned with the final text segments of the model's answer.
  - This is the default option, or you can explicitly specify it by adding the citation_options={"mode": "accurate"} argument in the API call.
* **Technical Entities (Classes/Functions/APIs):** `citation_options={"mode": "accurate"}`

#### Fast citations
* **Key Points:**
  - The model generates citations inline, as the response is being produced. In streaming mode, you will see citations injected at the exact moment the model uses a particular piece of external context. This approach provides immediate traceability at the expense of slightly less precision in citation relevance.
  - You can specify it by adding the citation_options={"mode": "fast"} argument in the API call.
* **Technical Entities (Classes/Functions/APIs):** `citation_options={"mode": "fast"}`
* **Code Snippet:**
```python
response = co.chat_stream(
    model="command-a-plus-05-2026",
    messages=messages,
    documents=documents,
    citation_options={"mode": "fast"},
)
response_text = ""
for chunk in response:
    if chunk:
        if chunk.type == "content-delta":
            response_text += chunk.delta.message.content.text
            print(chunk.delta.message.content.text, end="")
        if chunk.type == "citation-start":
            print(
                f" [{chunk.delta.message.citations.sources[0].id}]",
                end="",
            )
```