---
aliases:
  - Cancellation
Source 1: https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/cancellation
---
# Cancellation

* **Key Points:**
  - The Model Context Protocol (MCP) supports optional cancellation of in-progress requests through notification messages. Either side can send a cancellation notification to indicate that a previously-issued request should be terminated.
  - When a party wants to cancel an in-progress request, it sends a notifications/cancelled notification containing: The ID of the request to cancel; An optional reason string that can be logged or displayed
* **Technical Entities (Classes/Functions/APIs):** `MCP`, `notifications/cancelled`
* **Code Snippet:**
```json
{
  "jsonrpc": "2.0",
  "method": "notifications/cancelled",
  "params": {
    "requestId": "123",
    "reason": "User requested cancellation"
  }
}
```

## Behavior Requirements
* **Key Points:**
  - Cancellation notifications MUST only reference requests that: Were previously issued in the same direction; Are believed to still be in-progress
  - The initialize request MUST NOT be cancelled by clients
  - For task-augmented requests, the tasks/cancel request MUST be used instead of the notifications/cancelled notification. Tasks have their own dedicated cancellation mechanism that returns the final task state.
  - Receivers of cancellation notifications SHOULD: Stop processing the cancelled request; Free associated resources; Not send a response for the cancelled request
  - Receivers MAY ignore cancellation notifications if: The referenced request is unknown; Processing has already completed; The request cannot be cancelled
  - The sender of the cancellation notification SHOULD ignore any response to the request that arrives afterward
* **Technical Entities (Classes/Functions/APIs):** `notifications/cancelled`, `tasks/cancel`
* **Code Snippet:** None

## Timing Considerations
* **Key Points:**
  - Due to network latency, cancellation notifications may arrive after request processing has completed, and potentially after a response has already been sent.
  - Both parties MUST handle these race conditions gracefully.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Implementation Notes
* **Key Points:**
  - Both parties SHOULD log cancellation reasons for debugging
  - Application UIs SHOULD indicate when cancellation is requested
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None

## Error Handling
* **Key Points:**
  - Invalid cancellation notifications SHOULD be ignored: Unknown request IDs; Already completed requests; Malformed notifications
  - This maintains the "fire and forget" nature of notifications while allowing for race conditions in asynchronous communication.
* **Technical Entities (Classes/Functions/APIs):** None specified
* **Code Snippet:** None