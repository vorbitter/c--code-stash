### 1xx: Informational

ASP.NET Core does not have specific classes for informational responses since they are not commonly returned manually in APIs.

- **100 Continue**
- **101 Switching Protocols**

### 2xx: Successful

- **200 OK**: `Ok()`
    - Used when the request has succeeded.
- **201 Created**: `Created()` / `CreatedAtAction()` / `CreatedAtRoute()`
    - Indicates that a resource has been successfully created.
- **202 Accepted**: `Accepted()` / `AcceptedAtAction()` / `AcceptedAtRoute()`
    - Used when the request has been accepted for processing, but the processing has not been completed.
- **204 No Content**: `NoContent()`
    - Used when the server successfully processes the request, but there is no content to send in the response.

### 3xx: Redirection

- **301 Moved Permanently**: Can be set using response properties manually.
- **302 Found**: Can be set using `Redirect()` for URL redirection.
- **304 Not Modified**: Can be set manually in the response headers.

### 4xx: Client Error

- **400 Bad Request**: `BadRequest()`
    - Indicates that the server cannot process the request due to client-side issues.
- **401 Unauthorized**: `Unauthorized()`
    - Used when the user is not authenticated.
- **403 Forbidden**: `Forbid()`
    - Used when the user is authenticated but does not have permission to access the resource.
- **404 Not Found**: `NotFound()`
    - Used when the requested resource could not be found.
- **405 Method Not Allowed**: Automatically handled by ASP.NET Core when an unsupported HTTP method is used for a route.
- **409 Conflict**: `Conflict()`
    - Indicates that the request could not be completed due to a conflict with the current state of the resource.
- **415 Unsupported Media Type**: Set manually using response properties.
- **422 Unprocessable Entity**: `UnprocessableEntity()`
    - Used when the server understands the content type of the request but was unable to process the contained instructions.

### 5xx: Server Error

- **500 Internal Server Error**: Set manually using response properties or `StatusCode(500)`.
- **502 Bad Gateway**: Set manually using response properties.
- **503 Service Unavailable**: Set manually using response properties.
- **504 Gateway Timeout**: Set manually using response properties.

### Other Common Status Code Helpers:

- **StatusCode()**: Allows setting any arbitrary status code, e.g., `StatusCode(418)` (for fun with status codes like "I'm a teapot").

These methods and helpers are typically used in controller actions or middleware to send appropriate responses to the client. You can also customize status code pages if needed using middleware.

```c#
public IActionResult GetItem(int id)
{
    var item = _repository.GetItem(id);
    if (item == null)
    {
        return NotFound();  // 404 Not Found
    }
    return Ok(item);  // 200 OK
}

```