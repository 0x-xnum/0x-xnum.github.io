# Improper Assets Management

#### Discovery <a href="#discovery" id="discovery"></a>

* **Explore API Documentation**: Review API documentation to pinpoint parameters tied to account properties, crucial functions, and admin actions. This can reveal areas of potential risk in asset management.
* **Intercept Requests and Responses**: Use tools like Burp Suite to intercept API traffic. Inspect parameters and responses to spot any that may need further testing.
* **Parameter Guessing and Fuzzing**: Experiment with parameters that accept user input, focusing on those that could affect account variables or allow manipulation during account creation and editing.

#### Testing Procedure <a href="#testing-procedure" id="testing-procedure"></a>

Follow these steps to test for Improper Asset Management vulnerabilities:

**1. Baseline Versioning Information**

* Understand the API’s versioning by examining paths, headers, and parameters. Track supported production versions (e.g., `v1`, `v2`, `v3`).
* In **Postman**, create a test to detect status code 200, verifying successful responses for baseline comparison.

![](https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FS72WcsuQbeN8lqXzc46C_IAM1.PNG\&width=768\&dpr=4\&quality=100\&sign=5af8dba9\&sv=1)

**2. Run an Unauthenticated Baseline Scan**

* Conduct an unauthenticated scan on the API collection using Postman’s Collection Runner.
* Save responses to establish a baseline and for further analysis.

![](https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FjUdhPEQdQymPkG21pimA_IAM6.PNG\&width=768\&dpr=4\&quality=100\&sign=f3f9156f\&sv=1)

**3. Review and Analyze**

* Examine the unauthenticated scan results to understand how the API responds across different versioning.
* Look for anomalies in behavior or access control.

![](https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2Fz5SzbIF0RkGh5tcV9b4m_IAM5.PNG\&width=768\&dpr=4\&quality=100\&sign=376c27f9\&sv=1)

**4. Collection Version Replacement**

* Use the "Find and Replace" function to convert version-specific details into variables within Postman for flexibility in testing multiple versions (e.g., `v1`, `v2`, `v3`).

![](https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FbGq8FL27ST25xkG7YL3n_IAM3.PNG\&width=768\&dpr=4\&quality=100\&sign=effcdeae\&sv=1)

**5. Set Environment Variables**

* Add a Postman variable called `"ver"` and set its initial value to `v1`.
* Update this variable to test other versions, such as `mobile`, `internal`, `test`, `uat`, and observe responses.

![](https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FRVVSH2FOTtWszQ1LyPze_IAM4.PNG\&width=768\&dpr=4\&quality=100\&sign=fa0bfa63\&sv=1)

**6. Collection Runner with Version Variables**

* Run the collection with different version values, checking for inconsistencies in response.
* Note any unexpected responses for non-existent paths, particularly if they result in 200 OK statuses, which may signal asset management issues.

![](https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FDzWdP3b7QK6dDmS6R6nZ_IAM9.PNG\&width=768\&dpr=4\&quality=100\&sign=d2c365c1\&sv=1)

**7. Identify and Investigate Anomalies**

* Review differences in responses, especially in critical actions like password resets, across versions.
* Investigate specific endpoints for potential weaknesses, such as unlimited password reset attempts or open OTP validation paths

Impact Analysis and Brute Force Testing

1. **Impact Assessment**: Gauge the potential impact, such as whether an API version allows excessive attempts for password resets without restriction.
2. **Brute Force with WFuzz**: Use WFuzz to test parameters like OTP validation.

   ```bash
   wfuzz -d '{"email":"hapihacker@email.com", "otp":"FUZZ","password":"NewPassword1"}' \
   -H 'Content-Type: application/json' \
   -z file,/usr/share/wordlists/SecLists-master/Fuzzing/4-digits-0000-9999.txt \
   -u http://crapi.apisec.ai/identity/api/auth/v2/check-otp --hc 500
   ```

   * Look for unauthorized access indicators in successful brute force responses.

**Brute Force Review**

* Analyze brute force outcomes for any unauthorized access capabilities. Confirm if the vulnerability could allow unapproved actions.

![](https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FRWgS2x0SRSdu2GfFgouz_IAM14.PNG\&width=768\&dpr=4\&quality=100\&sign=d17143b4\&sv=1)

Authenticated User Testing

* Run the tests again as an authenticated user to ensure consistency and rule out unexpected behavior in protected API versions.
