### 30 Rules for Professional API Design
**URI & Resource Structure**
 1. **Nouns, Not Verbs:** Use /flags, not /getFlags.
 2. **Pluralize Everything:** /flags/{id} is more consistent than mix-and-matching singulars.
 3. **Hierarchical Paths:** Use /users/{id}/preferences/flags to show ownership.
 4. **Hyphens for Readability:** /feature-flags, not /feature_flags.
 5. **Lowercase URIs:** Always use lowercase to avoid server-side routing issues.
 6. **No File Extensions:** Use headers, not .json or .xml.
 7. **No Trailing Slashes:** They can break some routing logic and look messy.
**Request Methods (The Verbs)**
8.  **GET for Read-Only:** Safe and cacheable.
9.  **PUT for Idempotent Updates:** Replaces the whole resource.
10. **POST for Creating:** Adding a new flag to the collection.
11. **PATCH for Partial Updates:** Changing just the status or effectiveDate.
12. **DELETE for Removal:** Standardized cleanup.
**Data Consistency & Logic (Your Argument)**
13. **Avoid Booleans in URIs:** Don't put /isFlagEnabled in the path; it’s an attribute, not a resource.
14. **Resource Enveloping:** Return a structured object { "id": "A", "isEnabled": true, "effectiveDate": "..." }.
15. **Filtering with Query Strings:** Use /flags?active=true rather than creating a new endpoint.
16. **Statelessness:** The server shouldn't "remember" the client state.
17. **Prefer JSON:** It's the industry standard for interoperability.
18. **Pagination:** Essential for large collections of flags.
**Success & Error Handling**
19. **Use Standard Status Codes:** Don't return 200 with an "error" string in the body.
20. **Consistent Error Bodies:** Always return the same JSON structure for errors.
21. **201 for Creation:** Be specific when a new flag is generated.
22. **204 for No Content:** Perfect for successful deletes.
**Advanced & Leading Design**
23. **Versioning in the URI:** /v1/flags protects your users from breaking changes.
24. **HATEOAS:** Provide links to "related" actions (e.g., a link to deactivate the flag).
25. **Rate Limiting:** Protect your infrastructure.
26. **HTTPS Everywhere:** Security is not optional.
27. **Input Validation:** Use schema validation to reject bad flag data early.
28. **Design-First (OpenAPI):** Define the contract before writing a line of code.
29. **Time-to-First-Call:** Optimize documentation so developers can test it in minutes.
30. **Effective Caching:** Use ETags so clients don't re-download unchanged flags.
### The Status Code Cheat Sheet
When leading a design, knowing these shows you understand the **HTTP protocol**, not just your code.
| Category | Code | Name | Use Case |
|---|---|---|---|
| **Success** | **200** | OK | Standard success response. |
|  | **201** | Created | Successfully created a new flag. |
|  | **202** | Accepted | Request received but still processing (asynchronous). |
|  | **204** | No Content | Success, but nothing to send back (e.g., after DELETE). |
| **Client Error** | **400** | Bad Request | The client sent malformed JSON or invalid data. |
|  | **401** | Unauthorized | Authentication is missing or invalid. |
|  | **403** | Forbidden | Authenticated, but no permission to this flag. |
|  | **404** | Not Found | That flag ID doesn't exist. |
|  | **405** | Method Not Allowed | You tried to DELETE a read-only flag. |
|  | **429** | Too Many Requests | Rate limit hit. |
| **Server Error** | **500** | Internal Error | Your code crashed. |
|  | **503** | Service Unavailable | The database or flag-service is down. |
