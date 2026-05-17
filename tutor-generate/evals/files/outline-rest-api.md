# Learning Outline: REST API Fundamentals

## Learner Profile
- **Current level**: Frontend developer with 1 year of experience. Knows JavaScript/HTML/CSS. Has used `fetch()` to call existing APIs but doesn't understand how APIs are designed. No backend experience, no HTTP beyond GET/POST URLs.
- **Depth-breadth strategy**: Breadth-first. Build the mental model of how the web works at the protocol level, then layer on REST design principles.

---

## Chapter 1: How the Web Actually Works — HTTP as a Conversation
- **Prerequisites**: Basic understanding of URLs (already known from frontend work)
- **Teaching approach**:
  1. **Concrete example**: Show what actually happens when you type `https://api.github.com/users/octocat` in the browser — the request message the browser sends, and the response it gets back. Show raw HTTP messages.
  2. **Pattern recognition**: Loading a webpage, submitting a form, fetching data with `fetch()` — all are the same "request-response" pattern with different methods and URLs.
  3. **Abstract definition**: HTTP = a request-response protocol. Client sends a request (method + URL + headers + optional body), server sends back a response (status code + headers + optional body). Stateless — each request is independent.
- **Core takeaway**: Every web interaction is an HTTP request-response pair. The URL says "where", the method says "what action".
- **Omitted for now**: HTTPS/TLS details, HTTP/2 multiplexing, caching headers, CORS. [(More details to come later)]
- **Transition to next**: Knowing the request-response format, let's look at how URLs are structured to identify resources.

## Chapter 2: URLs as Nouns — Resources and Endpoints
- **Prerequisites**: Chapter 1 (HTTP basics)
- **Teaching approach**:
  1. **Concrete example**: Compare a messy API (`/getUser`, `/updateUser`, `/deleteUserById`) with a clean one (`GET /users/42`, `PATCH /users/42`, `DELETE /users/42`). Show why the clean one is easier to understand and predict.
  2. **Pattern recognition**: File system paths (`/home/documents/report.pdf`) — URLs can follow a similar hierarchical structure. Library catalog system — each book has a unique identifier in a logical hierarchy.
  3. **Abstract definition**: In REST, URLs identify resources (nouns), not actions (verbs). Resources are organized hierarchically: `/users` (collection), `/users/42` (specific item), `/users/42/orders` (sub-collection).
- **Core takeaway**: URLs should be nouns that identify resources. The HTTP method (GET, POST, PUT, DELETE) tells the server what to do with that resource.
- **Omitted for now**: HATEOAS, content negotiation, API versioning strategies, pagination patterns. [(More details to come later)]
- **Transition to next**: Resources are identified by URLs, but what does a resource actually look like? That's where status codes and response formats come in.

## Chapter 3: Status Codes and JSON — The Server's Response Language
- **Prerequisites**: Chapter 1 (HTTP basics), Chapter 2 (resources and URLs)
- **Teaching approach**:
  1. **Concrete example**: Call a REST API and get back `200 OK` with JSON data (success), `404 Not Found` (bad URL), `500 Internal Server Error` (server bug). Walk through each response — what the browser sees, what the status code means, what the JSON body contains.
  2. **Pattern recognition**: Status codes are like traffic lights — 2xx (green, go), 3xx (yellow, redirect), 4xx (red, your fault), 5xx (flashing red, server's fault). JSON is like a universal data format — strings, numbers, arrays, objects.
  3. **Abstract definition**: Status code = the server's summary of what happened. JSON = the standard data format for REST APIs (lightweight, human-readable, language-agnostic).
- **Core takeaway**: Status codes tell you WHAT happened (success, not found, error), and the JSON body gives you the DETAILS.
- **Omitted for now**: XML/other formats, custom error response patterns (RFC 7807), GraphQL comparison. [(More details to come later)]
- **Transition to next**: Now you understand the full request-response cycle. Next, you'd learn how to actually build a REST API server.
