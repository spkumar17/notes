# HTTP Status Codes: A Comprehensive Guide with Real-World Examples

## Quick Reference Table

| Status Code | Name | Description | Common Use Case |
|------------|------|-------------|-----------------|
| 200 | OK | Successful request with response body | GET requests returning data |
| 201 | Created | Resource successfully created | POST requests creating new records |
| 204 | No Content | Successful request with no response body | DELETE operations |
| 301 | Moved Permanently | Resource permanently relocated | Domain changes |
| 302 | Found | Resource temporarily relocated | Maintenance redirects |
| 400 | Bad Request | Invalid client request | Form validation failures |
| 401 | Unauthorized | Authentication required | Protected API access |
| 403 | Forbidden | Access denied despite authentication | Non-admin accessing admin area |
| 404 | Not Found | Resource doesn't exist | Broken links |
| 500 | Internal Server Error | Server-side failure | Database connection issues |
| 502 | Bad Gateway | Invalid upstream server response | Microservice failures |

## Success Status Codes (2xx)

### 200 OK - Successful Request
**Definition**: The request was successful, and the server has returned the requested data. This is commonly used for GET requests.

**Scenario**: Fetching Product Details
A user visits an e-commerce website and wants to check the details of a product.

### 201 Created - Resource Creation Success
**Definition**: The request was successful, and a new resource was created. This is typically used for POST requests when creating a new record.

**Scenario**: User Registration
A new user signs up on a website.

✅ Why 201 Created?
A new user account was successfully created, and the Location header indicates where to access the new resource.

### 204 No Content - Successful Request with No Response
**Definition**: The request was successful, but there is no content to return. This is often used for DELETE or PUT requests when no response body is necessary.

**Scenario**: Deleting a Comment
A user deletes their comment from a blog post.

✅ Why 204 No Content?
The comment was successfully deleted, but the response body is empty since no further information is needed.

## Redirection Status Codes (3xx)

### 301 Moved Permanently - Permanent URL Change
**Definition**: The requested resource has been permanently moved to a new URL. The client (browser or API consumer) should update bookmarks and always use the new URL in future requests.

**Scenario**: Website Domain Change
A company changes its website domain from oldsite.com to newsite.com.

✅ Why 301 Moved Permanently?
The website has permanently moved to newsite.com, and all future requests should go to the new URL.

🔹 Effect in Browsers: The browser will automatically redirect the user to the new URL and remember the change.

### 302 Found - Temporary Redirect
**Definition**: The requested resource has temporarily moved to a different URL, but future requests should still use the original URL.

**Scenario**: Maintenance Redirect
A website is under maintenance, and users are temporarily redirected to a status page.

✅ Why 302 Found?
The dashboard page is temporarily unavailable, and users are redirected to the maintenance page. However, once the maintenance is over, the dashboard will be available at its original URL.

🔹 Effect in Browsers: The browser redirects the user to the new URL but does not remember the change for future visits.

## Client Error Status Codes (4xx)

### 400 Bad Request - Invalid Client Request
**Definition**: The server cannot process the request due to client-side errors, such as invalid syntax, missing parameters, or incorrect formatting.

**Scenario**: Invalid Form Submission
A user tries to sign up but provides an invalid email format.

✅ Why 400 Bad Request?
The email format is incorrect, and the server rejects the request due to bad input.

### 401 Unauthorized - Authentication Required
**Definition**: The request requires authentication, but the user is either not authenticated or provided invalid credentials.

**Scenario**: Accessing a Protected API Without Logging In
A user tries to access their profile page without logging in.

✅ Why 401 Unauthorized?
The user is not logged in or provided an invalid token, so access is denied.

### 403 Forbidden - Access Denied
**Definition**: The user is authenticated but does not have permission to access the requested resource.

**Scenario**: Non-Admin User Trying to Access Admin Panel
A regular user tries to access an admin-only page.

✅ Why 403 Forbidden?
The user is logged in, but they don't have the necessary admin role to access this page.

### 404 Not Found - Resource Not Available
**Definition**: The requested resource does not exist on the server.

**Scenario**: Trying to Access a Non-Existent Page
A user visits a broken link.

✅ Why 404 Not Found?
The requested page does not exist on the server.

## Server Error Status Codes (5xx)

### 500 Internal Server Error - Server-Side Failure
**Definition**: The server encountered an unexpected condition that prevented it from fulfilling the request. This usually indicates a bug or misconfiguration on the server.

**Scenario**: Database Connection Failure
A user tries to log in, but the server cannot connect to the database.

✅ Why 500 Internal Server Error?
The server encountered an issue (e.g., database failure), making it unable to process the request.

### 502 Bad Gateway - Invalid Upstream Response
**Definition**: The server (acting as a gateway or proxy) received an invalid response from an upstream server.

**Scenario**: API Gateway Failure
A website uses an API gateway that connects to multiple microservices. If one of the microservices fails, the gateway cannot process requests.

✅ Why 502 Bad Gateway?
The API gateway tried to forward the request but got an invalid response from the backend microservice.
