# idor checklist

#### Base Steps:

1. **Account Setup**: Create two accounts or enumerate users first.
2. **Endpoint Check**: Determine if the endpoint is private or public and if it contains any ID parameter.
3. **Parameter Manipulation**: Change the parameter value to another user's ID and observe any changes to their account.
4. **Done!**

#### Additional Tests:

* [ ] **Profile Actions**: Check actions like image profile, account deletion, account information, API key management, comment reading, price changes, and currency conversions (e.g., dollar to euro).
* [ ] **ID Decoding**: If the ID is encoded (e.g., MD5, base64), try decoding it.
  * Example: `GET /GetUser/dmljdGltQG1haWwuY29t`
* [ ] **HTTP Method Change**: Test different HTTP methods.
  * Example:
    * `GET /users/delete/victim_id` → 403
    * `POST /users/delete/victim_id` → 200
* [ ] **Parameter Replacement**: Swap parameter names.
  * Example:
    * Instead of `GET /api/albums?album_id=<album id>`, try `GET /api/albums?account_id=<account id>`.
* [ ] **Burp Extension**: Use Paramalyzer to remember all parameters passed to a host.
* [ ] **Path Traversal**:
  * Example:
    * `POST /users/delete/victim_id` → 403
    * `POST /users/delete/my_id/..victim_id` → 200
* [ ] **Change Request Content-Type**:
  * Example:
    * Change from `Content-Type: application/xml` to `Content-Type: application/json`.
* [ ] **ID Swap**: Swap non-numeric IDs with numeric ones.
  * Example:
    * `GET /file?id=90djbkdbkdbd29dd`
    * `GET /file?id=302`.
* [ ] **Function Level Access Control**:
  * Example:
    * `GET /admin/profile` → 401
    * `GET /Admin/profile` → 200 (and variations).
* [ ] **Wildcard Parameter**:
  * Example:
    * `GET /api/users/user_id` →
    * `GET /api/users/*`.
* [ ] **Encoded/Hashed ID**: Never ignore encoded/hashed IDs. Create multiple accounts to understand patterns.
* [ ] **Google Dorking**: Search for indexed endpoints containing IDs.
* [ ] **Brute Force Hidden Parameters**: Use tools like Arjun or ParamMiner.
* [ ] **Bypass Object Level Authorization**: Add parameters to endpoints if not present by default.
  * Example:
    * `GET /api_v1/messages` → 200
    * `GET /api_v1/messages?user_id=victim_uuid` → 200.
* [ ] **HTTP Parameter Pollution**: Send multiple values for the same parameter.
  * Example:
    * `GET /api_v1/messages?user_id=attacker_id&user_id=victim_id`.
* [ ] **Change File Type**:
  * Example:
    * `GET /user_data/2341` → 401
    * `GET /user_data/2341.json` → 200 (and others).
* [ ] **JSON Parameter Pollution**:
  * Example: `{"userid":1234,"userid":2542}`.
* [ ] **Wrap ID in Array**:
  * Example:
    * `{"userid":123}` → 401
    * `{"userid":[123]}` → 200.
* [ ] **Wrap ID in JSON Object**:
  * Example:
    * `{"userid":123}` → 401
    * `{"userid":{"userid":123}}` → 200.
* [ ] **Outdated API Version**:
  * Example:
    * `GET /v3/users_data/1234` → 401
    * `GET /v1/users_data/1234` → 200.
* [ ] **GraphQL IDOR Testing**: If using GraphQL, test for IDOR.
  * Example:
    * `GET /graphql`
    * `GET /graphql.php?query=`
