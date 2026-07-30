# Excessive Data Exposure

**Understanding API Responses**

* API providers may return complete data objects, relying on clients to filter necessary information.
* The main concern is the sensitivity of the data sent, not just the quantity.

**API Documentation**\
Understanding API documentation is crucial for effective testing. Key sections typically include:

* **Overview**:
  * Introduces the API and details authentication and rate limits.
  * Example: Financial APIs may impose strict rate limits to prevent abuse.
* **Functionality**:
  * Lists actions using HTTP methods (GET, PUT, POST, DELETE) and endpoints.
  * Example: An endpoint should only return data for the authenticated user, avoiding sensitive information about others.
* **Request Requirements**:
  * Specifies authentication, parameters, path variables, headers, and request body.
  * Example: Omission of necessary authentication headers can lead to unauthorized access to private user information.

**API Documentation Conventions**

Familiarity with common API documentation conventions helps in forming well-structured requests:

* **Path Variables:** Indicated by a colon (:) or curly brackets ({}).\
  *Example: `/user/:id` or `/user/{id}`*
* **Optional Input:** Square brackets (\[]) indicate optional inputs.\
  *Example: `/api/v1/user?find=[name]`*
* **Multiple Values:** Double bars (|) show alternative values.\
  *Example: `"blue" | "green" | "red"`*

**Using Postman for API Testing**

* Add valid tokens for authorized requests (e.g., Bearer Tokens) in collection settings.
* Use the variables tab in Postman to set placeholders for values like the `baseUrl`, tokens, and other reusable elements.
* Switch HTTP methods easily within Postman.

#### Testing for Excessive Data Exposure

**1. Authenticate:**

* Send a POST request to the API’s authentication endpoint with valid credentials (e.g., username and password) to get your token.
* **Example Response:**

  ```json
  {"token": "Bearer <token>"}
  ```

**2. Configure Authorization:**

* Add the Bearer Token to the **Authorization** section in Postman to ensure your requests are authorized.
* Save this configuration for consistency when making multiple requests.

**3. Check Response Size:**

* Monitor response sizes when requesting resources, such as user profile or dashboard information.
* **Example Response:**

  ```json
  jsonCopy code{
    "id": "123",
    "name": "User",
    "email": "user@example.com",
    "additionalInfo": {
      "emails": ["user1@example.com", "user2@example.com"],
      "privilege": "admin",
      "MFA": false
    }
  }
  ```

**4. Identify Excessive Data:**

* **Ingredients:**\
  Responses may include more data than necessary, such as personal details or sensitive information about other users.
* **Example:** **Request:**

  ```http
  httpCopy codeGET /api/v1/user?=CloudStrife
  ```

  **Response:**

  ```json
  jsonCopy code{
    "id": "123",
    "name": "Cloud",
    "email": "cloud@example.com",
    "privilege": "user",
    "representative": {
      "name": "Don Corneo",
      "id": "2203",
      "email": "dcorn@example.com",
      "privilege": "admin",
      "MFA": false
    }
  }
  ```

  In this case, the response reveals not only the requested user's information but also sensitive details about an admin, which should not be exposed.

**Practical Tips for Testing Excessive Data Exposure**

* **Review Response Data**: Look for unnecessary or sensitive information in responses.
* **Test Edge Cases**: Use invalid or unexpected parameters to see if sensitive data leaks.
* **Monitor API Behavior**: Log and analyze API behavior over time for exposure patterns.
* **Role-Based Testing**: Validate data returned for different user roles to ensure proper access control.
