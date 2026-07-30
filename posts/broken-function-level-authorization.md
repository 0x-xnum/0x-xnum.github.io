# Broken Function Level Authorization

#### **Understanding BFLA**

* While BOLA focuses on accessing resources not belonging to you, BFLA concerns performing unauthorized actions.
* **Types of Actions:**
  * **Lateral Actions:** Performing actions of users with the same role/privilege level.
  * **Escalated Actions:** Performing actions with escalated privileges (e.g., administrator actions).

### Hunting for Broken Object Level Authorization (BOLA) Vulnerabilities

#### **Resource Identifier (Resource ID)**

* The unique identifier for a resource in the API, such as `user_id=123`. Resource IDs can be simple (e.g., numeric) or complex (e.g., UUIDs or hashes).

#### **2. Requests that Access Resources**

* &#x20;Identify the necessary requests for obtaining resources (create, read, update, delete) that your account shouldn’t have access to. This helps in testing if you can access another user's resources.

#### **3. Access Control Issues**

* Txo exploit a BOLA weakness, the API must lack proper access controls. While predictable resource IDs can be a red flag, this alone doesn’t confirm an authorization vulnerability; genuine absence of access controls is necessary for exploitation.

#### **Testing Strategy**

* Similar to BOLA, but with a focus on functional requests (POST, PUT, DELETE, potentially GET with the right parameters).
* **Example Application: crAPI**
  * **Identified Requests for Testing:**
    * `POST /workshop/api/shop/orders/return_order?order_id=5893280.0688146055`
    * `POST /community/api/v2/community/posts/w4ErxCddX4TcKXbJoBbRMf/comment`
    * `PUT /identity/api/v2/user/videos/:id`

#### **A-B-A Testing Approach**

1. **UserA Valid Requests:**
   * Make valid requests as UserA and document operations involving resource IDs.
2. **Switch to UserB:**
   * Attempt to make requests altering UserA's resources using UserB's token.
3. **Verify Changes:**
   * Return to UserA’s account to confirm if alterations were successful.

#### **Example Attack**

* **Request Analysis:**
  * Initial attempts to update UserA's video might yield unexpected responses indicating resource changes.
  * Exploring the DELETE method may reveal admin functions (e.g., `DELETE /identity/api/v2/user/videos/758`).
* **Discovering Admin Paths:**
  * Modify requests to exploit BFLA weaknesses by trying paths like `DELETE /identity/api/v2/admin/videos/758`.

#### **Caution in BFLA Testing**

* Successful BFLA attacks can alter user data.
* **Important Note:** Do not brute-force BFLA attacks. Use a secondary account for testing to avoid violations of rules of engagement.
