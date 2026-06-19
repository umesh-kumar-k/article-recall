---
aliases:
  - Progress Tracking
Source 1: https://modelcontextprotocol.io/specification/2025-03-26/basic/utilities/progress
---
# Progress

* **Key Points:**
  - The Model Context Protocol (MCP) supports optional progress tracking for long-running operations through notification messages. Either side can send progress notifications to provide updates about operation status.
  - When a party wants to receive progress updates for a request, it includes a progressToken in the request metadata.
  - Progress tokens MUST be a string or integer value
  - Progress tokens can be chosen by the sender using any means, but MUST be unique across all active requests.
  - The receiver MAY then send progress notifications containing: The original progress token; The current progress value so far; An optional "total" value; An optional "message" value
  - The progress value MUST increase with each notification, even if the total is unknown.
  - The progress and the total values MAY be floating point.
  - The message field SHOULD provide relevant human readable progress information.
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `notifications/progress`, `progressToken`
* **Code Snippet:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "some_method",
  "params": {
    "_meta": {
      "progressToken": "abc123"
    }
  }
}
```
```json
{
  "jsonrpc": "2.0",
  "method": "notifications/progress",
  "params": {
    "progressToken": "abc123",
    "progress": 50,
    "total": 100,
    "message": "Reticulating splines..."
  }
}
```

## Behavior Requirements
* **Key Points:**
  - Progress notifications MUST only reference tokens that: Were provided in an active request; Are associated with an in-progress operation
  - Receivers of progress requests MAY: Choose not to send any progress notifications; Send notifications at whatever frequency they deem appropriate; Omit the total value if unknown
  - For task-augmented requests, the progressToken provided in the original request MUST continue to be used for progress notifications throughout the task's lifetime, even after the CreateTaskResult has been returned. The progress token remains valid and associated with the task until the task reaches a terminal status.
  - Progress notifications for tasks MUST use the same progressToken that was provided in the initial task-augmented request
  - Progress notifications for tasks MUST stop after the task reaches a terminal status (completed, failed, or cancelled)
* **Technical Entities (Classes/Functions/APIs):** `progressToken`, `CreateTaskResult`
* **Code Snippet:** None

## Implementation Notes
* **Key Points:**
  - Senders and receivers SHOULD track active progress tokens
  - Both parties SHOULD implement rate limiting to prevent flooding
  - Progress notifications MUST stop after completion
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None