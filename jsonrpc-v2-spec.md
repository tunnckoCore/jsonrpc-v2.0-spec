## JSON-RPC 2.0 Specification

### 1. Overview

JSON-RPC 2.0 is a stateless, lightweight remote procedure call (RPC) protocol. It defines several data structures and the rules for their processing. It's designed to be simple and transport-agnostic, meaning it can be used over various communication channels like sockets, HTTP, etc. It uses JSON (RFC 4627) as its data format.

### 2. Conventions

*   Keywords like "MUST", "MUST NOT", "REQUIRED", etc., follow RFC 2119 definitions.
*   It uses JSON's type system: Primitive types (String, Number, Boolean, Null) and Structured types (Object, Array). Type names are capitalized (e.g., `String`).
*   Member names are case-sensitive.
*   `Client`: Origin of Request objects, handler of Response objects.
*   `Server`: Origin of Response objects, handler of Request objects.

### 3. Request Object

A remote procedure call is made by sending a `Request` object to a server.

**Structure:**

*   `jsonrpc` (String): **REQUIRED**. Specifies the JSON-RPC protocol version. MUST be exactly `"2.0"`.
*   `method` (String): **REQUIRED**. The name of the method to be invoked.
    *   Method names starting with `rpc.` are reserved for internal methods and extensions.
*   `params` (Structured): **OPTIONAL**. An Object or Array containing parameter values for the method. If omitted, the method is assumed to take no parameters.
*   `id` (String | Number | Null): **REQUIRED** (unless it's a Notification). An identifier established by the Client. It MUST be a String, Number, or Null. If not included, it's treated as a Notification. The Server MUST reply with the same `id` in the Response object. Using `Null` is discouraged.

**Example (Positional Params):**

```json
{
  "jsonrpc": "2.0",
  "method": "subtract",
  "params": [42, 23],
  "id": 1
}
```

**Example (Named Params):**

```json
{
  "jsonrpc": "2.0",
  "method": "subtract",
  "params": { "subtrahend": 23, "minuend": 42 },
  "id": 3
}
```

### 4. Notification

A `Request` object without an `id` member is a Notification. The Server MUST NOT reply to a Notification. Notifications are for cases where no response is required.

**Example:**

```json
{
  "jsonrpc": "2.0",
  "method": "update",
  "params": [1, 2, 3, 4, 5]
}
```

### 5. Response Object

When a client sends a Request object (that is not a notification), the Server MUST reply with a `Response` object.

**Structure:**

*   `jsonrpc` (String): **REQUIRED**. Specifies the JSON-RPC protocol version. MUST be exactly `"2.0"`.
*   `result` (Any): **REQUIRED on success**. The value returned by the invoked method. Its type is defined by the method. This member MUST NOT exist if there was an error invoking the method.
*   `error` (Object): **REQUIRED on error**. An Error object (see below). This member MUST NOT exist if there was no error triggered during invocation.
*   `id` (String | Number | Null): **REQUIRED**. MUST be the same value as the `id` in the Request object it's responding to. If detecting the `id` in the Request was impossible (e.g., Parse error), it MUST be `Null`.

**Exactly one of `result` or `error` MUST be included.**

**Example (Success):**

```json
{
  "jsonrpc": "2.0",
  "result": 19,
  "id": 1
}
```

**Example (Error):**

```json
{
  "jsonrpc": "2.0",
  "error": { "code": -32601, "message": "Method not found" },
  "id": "1"
}
```

### 6. Error Object

When an RPC call encounters an error, the `Response` object includes an `error` member with the following structure:

*   `code` (Number): **REQUIRED**. An integer indicating the error type. Predefined codes are in the range -32768 to -32000.
*   `message` (String): **REQUIRED**. A short description of the error.
*   `data` (Any): **OPTIONAL**. Primitive or Structured value containing additional information about the error. May be omitted.

**Predefined Error Codes:**

| Code    | Message          | Meaning                                                                 |
| :------ | :--------------- | :---------------------------------------------------------------------- |
| -32700  | Parse error      | Invalid JSON received by the server. Error parsing JSON text.           |
| -32600  | Invalid Request  | The JSON sent is not a valid Request object.                            |
| -32601  | Method not found | The method does not exist or is not available.                          |
| -32602  | Invalid params   | Invalid method parameter(s).                                            |
| -32603  | Internal error   | Internal JSON-RPC error.                                                |
| -32000 to -32099 | Server error | Reserved for implementation-defined server-errors.                  |

### 7. Batch Requests/Responses

To send multiple Request objects at once, the Client MAY send an Array containing Request objects.

*   The Server SHOULD process batch requests in parallel. There are no guarantees about the order of execution.
*   The Server MUST return a Response for each Request that requires one.
*   The Server MUST return an Array containing the corresponding Response objects. The order of Responses SHOULD match the order of Requests if possible, but this is not guaranteed.
*   If the batch request itself is invalid JSON, the server returns a single Response object with a Parse error.
*   If the batch is an empty Array, the server returns an empty Array.
*   If the batch contains invalid Request objects, the server returns a Response object for each invalid request, plus responses for valid requests.

**Example Batch Request:**

```json
[
  { "jsonrpc": "2.0", "method": "sum", "params": [1, 2, 4], "id": "1" },
  { "jsonrpc": "2.0", "method": "notify_hello", "params": [7] },
  { "jsonrpc": "2.0", "method": "subtract", "params": [42, 23], "id": "2" },
  { "foo": "boo" },
  { "jsonrpc": "2.0", "method": "get_data", "id": "9" }
]
```

**Example Batch Response:**

```json
[
  { "jsonrpc": "2.0", "result": 7, "id": "1" },
  { "jsonrpc": "2.0", "result": 19, "id": "2" },
  {
    "jsonrpc": "2.0",
    "error": { "code": -32600, "message": "Invalid Request" },
    "id": null
  },
  {
    "jsonrpc": "2.0",
    "result": ["hello", 5],
    "id": "9"
  }
]
```

This summary covers the core aspects of the JSON-RPC 2.0 specification. For complete details, refer to the official specification document.
