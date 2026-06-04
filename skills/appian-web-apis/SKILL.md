---
name: "appian-web-apis"
description: "Create and modify Appian Web APIs — HTTP endpoints that expose Appian data and services to external systems. Covers Web API creation with HTTP methods (GET, POST, PUT, DELETE, PATCH), URL alias conventions, expression bodies using a!httpResponse(), and request body type configuration. Use when creating Web APIs, configuring endpoints, or exposing Appian data to outside systems."
---

## Relevant Tools

Web API tools cover the full lifecycle:

- **Create a Web API** — provide a name, URL alias, and HTTP method; optionally include a description, expression body, and request body type; associate with an application via `appUuid`
- **List and get Web APIs** — retrieve Web APIs by UUID or search with optional filtering; scope to an application with `appUuid`
- **Update a Web API** — modify name, description, expression body, URL alias, HTTP method, or request body type on an existing Web API
- **Delete a Web API** — permanently remove a Web API by UUID

## Creating Web APIs

### Required Parameters

Every Web API requires three fields at creation:

- `name` — internal object name visible to Appian developers
- `urlAlias` — the endpoint slug that forms part of the Web API's URL (called "Endpoint" in Appian Designer)
- `httpMethod` — one of: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`

### Optional Parameters

- `description` — short description of what the Web API does
- `expression` — the Appian expression body that executes when the endpoint is called; must return an `a!httpResponse()` object
- `requestBodyType` — how the Web API handles incoming request body content; one of: `NONE` (default), `BINARY`, `MULTIPART`
- `appUuid` — application UUID; when provided, the Web API is automatically added to the application and inherits its default security

### Naming Conventions

- **name**: Use the application prefix followed by a descriptive name. Examples: `ACME Query Employees`, `CM Create Case`, `HR Update Employee`
- **name** is only visible to developers in Appian Designer — it does not appear in the URL

### URL Alias Conventions

The URL alias (endpoint) becomes part of the Web API's URL: `/suite/webapi/<urlAlias>`. Follow these rules:

- Use lowercase with hyphens for multi-word endpoints: `employees`, `case-status`, `create-application`
- Keep it short and descriptive — it appears in URLs and log files
- The combination of HTTP method + URL alias must be unique across all Web APIs in the environment. You can have a `GET` and a `POST` on the same endpoint, but not two `GET` Web APIs with the same endpoint.
- Avoid special characters, spaces, or uppercase letters
- The URL alias does not auto-update if you rename the Web API later — update it explicitly if the endpoint needs to change

### HTTP Methods

Choose the HTTP method based on what the Web API does:

| Method | Use When |
|---|---|
| `GET` | Reading/querying data. Cannot execute smart services. |
| `POST` | Creating data, starting processes, or performing actions that change state. Can execute smart services. |
| `PUT` | Replacing or fully updating an existing resource. Can execute smart services. |
| `DELETE` | Removing a resource. Can execute smart services. |
| `PATCH` | Partially updating an existing resource. Can execute smart services. |

Key constraint: `GET` Web APIs cannot execute smart services (write to data store, start process, send email, etc.). Attempting to do so produces an error. Use `POST`, `PUT`, `DELETE`, or `PATCH` for any Web API that needs to perform write operations.

The system automatically handles `OPTIONS` and `HEAD` requests based on which Web APIs exist for a given endpoint — you do not need to create separate Web APIs for these methods.

### Request Body Type

The `requestBodyType` field controls how the Web API handles incoming request body content:

| Value | Use When |
|---|---|
| `NONE` | The Web API does not expect a request body (typical for `GET` and `DELETE`). This is the default. |
| `BINARY` | The Web API accepts binary content (e.g., file uploads). Use with `POST`, `PUT`, or `PATCH`. Document size limit is 75 MB. |
| `MULTIPART` | The Web API accepts multipart form data (e.g., file uploads with metadata). Use with `POST`, `PUT`, or `PATCH`. |

### Expression Body

The expression body is the logic that executes when the endpoint is called. It must return an `a!httpResponse()` object.

Key patterns:

- Access the incoming HTTP request via the `http!request` domain — `http!request.queryParameters`, `http!request.headers`, `http!request.body`
- Return responses using `a!httpResponse(statusCode, headers, body)`
- Set the `Content-Type` header explicitly (typically `application/json`)
- Use `a!toJson()` to serialize Appian values to JSON for the response body
- Use `a!fromJson()` to parse JSON from the request body

A typical GET Web API expression:

```
=a!localVariables(
  local!records: a!queryRecordType(
    recordType: recordType!EntityName,
    fields: { ... },
    pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 50)
  ).data,
  a!httpResponse(
    headers: {
      a!httpHeader(name: "Content-Type", value: "application/json")
    },
    body: a!toJson(value: local!records)
  )
)
```

A typical POST Web API expression using a smart service:

```
=a!writeToDataStoreEntity(
  dataStoreEntity: cons!ENTITY_CONSTANT,
  valueToStore: a!fromJson(http!request.body),
  onSuccess: a!httpResponse(
    statusCode: 200,
    headers: { a!httpHeader(name: "Content-Type", value: "application/json") },
    body: a!toJson(fv!storedValues)
  ),
  onError: a!httpResponse(
    statusCode: 500,
    headers: { a!httpHeader(name: "Content-Type", value: "application/json") },
    body: a!toJson({ error: "Write failed" })
  )
)
```

## Updating Web APIs

The update operation supports partial updates — only provided fields are changed. You can update any combination of:

- `name` — rename the Web API
- `description` — update the description
- `expression` — change the expression body
- `urlAlias` — change the endpoint URL
- `httpMethod` — change the HTTP method
- `requestBodyType` — change how the request body is handled

When changing the HTTP method or URL alias, verify that the new combination does not conflict with an existing Web API.

## Common Pitfalls

- **Missing required fields on create** — all three of `name`, `urlAlias`, and `httpMethod` are required; omitting any one causes a 400 error
- **Duplicate method + endpoint combination** — the combination of HTTP method and URL alias must be unique across the environment; creating a second `GET` on the same endpoint fails
- **Using smart services in a GET Web API** — `GET` Web APIs cannot execute smart services; use `POST`, `PUT`, `DELETE`, or `PATCH` instead
- **Forgetting to return a!httpResponse()** — the expression body must return an `a!httpResponse()` object; returning raw data produces unexpected results
- **Not setting Content-Type header** — clients may not parse the response correctly without an explicit `Content-Type` header; always set it (typically `application/json`)
- **URL alias with spaces or uppercase** — use lowercase kebab-case; spaces and special characters in the URL alias cause issues
- **Not associating with the application** — pass `appUuid` when creating the Web API so it inherits the application's default security and appears in the application's object list
- **Changing the name and expecting the URL to update** — the URL alias is independent of the name; update it explicitly if the endpoint needs to change

## Dependency Position

Web APIs come late in the Appian object dependency order — after process models and before sites. A Web API's expression body may reference record types, expression rules, constants, and other objects, so those must exist first. Web APIs themselves are rarely referenced by other Appian objects (sites don't point to Web APIs; external systems call them directly via URL).

## When You Need More

For expression syntax used in Web API expression bodies, activate the expressions knowledge skill. For security patterns including role-based access to Web API endpoints, activate the security knowledge skill.

For questions about Web API authentication, CORS configuration, advanced request/response handling, or platform capabilities beyond this reference, call `mcp__appian-docs__search_appian_knowledge_sources` to search Appian's official documentation. Prefer it over WebFetch — results are scoped to the current Appian release and far smaller in context.
