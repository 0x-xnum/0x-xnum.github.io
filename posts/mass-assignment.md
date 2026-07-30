# Mass Assignment

## What is Mass Assignment? <a href="#id-2083" id="id-2083"></a>

Applications these days often rely an objects (For example user, product, …) and these objects have properties (for example product.stock). As a user, we have the authorization to edit and view specific properties of the objects but we might also be limited and not able to edit or view some specific properties (For example user can view product.stock but user should not be able to edit product.stock). These properties are then matched to parameters on the front-end and if these conversion happen automatically, they might convert parameters to properties the attacker should not be able to access (For example, the user should never be able to edit product.title but the front-end might convert a parameter “title” to product.title if the user sends a PUT request).

Here are some more examples of properties the user should not be able to edit:

* Account.AccountType or Account.discountsEnable. These are properties that relate to permissions.
* Account.wallet This property should never be editable be the attacker
* product.title These are internal properties the user should never be able to edit

A common example is when an attacker adds parameters to a user registration request, such as setting `isadmin: true`, to escalate their account privileges.

#### How to Find Mass Assignment Vulnerabilities

1. **Explore API Documentation**: Check the documentation for parameters related to user roles, permissions, and critical functionalities. Adding these parameters to requests might reveal vulnerabilities.
2. **Observe Naming Conventions**: Use the API as designed to understand parameter naming conventions, as this insight can guide potential parameters for attacks on other endpoints.
3. **Fuzz for Blind Attacks**: If there is no documentation, fuzz parameters by capturing requests and brute-forcing values. Starting with account registration is a good approach, as it often contains exploitable user input.

## Testing Account Registration for Mass Assignment

Let's intercept the account registration request for crAPI.

1. **Create a New Account**: Use the application to register a new account while intercepting the request.<br>

   <figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/bBPZiv9OQXe8IPvetw56_MA1.PNG" alt=""><figcaption></figcaption></figure>
2. Submit the form to create an account and make sure the request was intercepted with Burp Suite. <br>

   <figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/y4cAWxOLTCmWQglFjmSe_MA2.PNG" alt=""><figcaption></figcaption></figure>
3. **Send the Intercepted Request to Repeater**: Once intercepted, send the request to Burp Suite's Repeater to modify and test different payloads.<br>

   <figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/5Dx0FR19RG2qKhtyquOg_MA3.PNG" alt=""><figcaption></figcaption></figure>
4. **Modify JSON Payload**: Alter the JSON payload by adding parameters like `"isadmin": true`, `"admin": 1`, or similar variations. Send each modified request and observe the API's responses for unique indicators of privilege escalation or success.<br>

   <figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/uyN67TJTNe36mLlcH6Tw_MA6.PNG" alt=""><figcaption></figcaption></figure>
5. **Analyze API Responses**: If the API responds with no changes or indications of status alteration, the target may not be vulnerable. However, if variations in response occur, it suggests potential vulnerability.
6. **Use Intruder for Variants**: If you want to explore combinations of parameters systematically, send the request to Burp Suite's Intruder, set payload positions around the admin-related parameters, and run a “Cluster Bomb” attack.

   \ <br>

   <figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/vO8TmsEPRwuXEImM3MNP_MA4.PNG" alt=""><figcaption></figcaption></figure>

### Fuzzing For Mass Assignment With Param Miner

* **Install Param Miner**: Ensure you have the Param Miner extension in Burp Suite.

<figure><img src="https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2F1JzY2pCMQbmln51Omg3X_MA5.PNG&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=92195eff&#x26;sv=1" alt=""><figcaption></figcaption></figure>

* Right-click on a request to mine parameters using Param Miner.
* Configure Param Miner options and click OK.

<figure><img src="https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FO5NDlXkFScK12mcI3fYP_MA7.PNG&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=6e7c95c9&#x26;sv=1" alt=""><figcaption></figcaption></figure>

* **Review Detected Parameters**: After running, check the Output tab for any new parameters that can be tested.

<figure><img src="https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2F5PtTnKVNQpez4cOTgdup_MA9.PNG&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=9ba837&#x26;sv=1" alt=""><figcaption></figcaption></figure>

* **Reinsert Parameters for Fuzzing**: Insert these parameters back into the original request to see if they trigger any responses that could indicate a vulnerability.

### **Other Mass Assignment Vectors**

* **Unauthorized Access to Organizations**: If the application allows users to be part of organizational groups, you can test for unauthorized access to those groups. This can often be done by modifying request parameters to include organizational identifiers.
  * **Example**: By adding an `"org"` parameter to a request and experimenting with different values, you might be able to access groups that a user is not supposed to see or interact with. This is particularly relevant in applications that manage organizational hierarchies or user groups.
* **Access Control Over Object Properties**: Beyond organizational access, look for other sensitive properties within user objects that should be protected. This can include user roles, permissions, or configuration flags.
  * **Testing Method**: If you find an endpoint that modifies user settings, try adding parameters related to roles or permissions (like `role`, `permissions`, or `access_level`).
* **Modification of Related Objects**: Identify relationships between objects in the API. For instance, if a user can modify their profile, check if that request can also include changes to associated objects, such as linked accounts, payment methods, or preferences.

**Hunting for Mass Assignment**

1. **Analyze the Target API Collection**:

   * Review the API documentation or captured traffic to identify endpoints that:
     * Accept user input.
     * Allow modification of objects.

   <img src="https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FjNSLM1n3RonguO5mcPtP_MA10.PNG&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=3e8fa638&#x26;sv=1" alt="" data-size="line">
2. **Create a New Collection for Testing**:
   * Set up a separate collection specifically for mass assignment testing to prevent unintentional modifications to the original API collection.
   * **Duplicate Requests**: Take interesting or relevant requests and duplicate them in your new collection.

<figure><img src="https://sallam.gitbook.io/~gitbook/image?url=https%3A%2F%2Fkajabi-storefronts-production.kajabi-cdn.com%2Fkajabi-storefronts-production%2Fsite%2F2147573912%2Fproducts%2FxcmJPdWT2WVNq0P4FOQ2_MA11.PNG&#x26;width=768&#x26;dpr=4&#x26;quality=100&#x26;sign=bcdeb721&#x26;sv=1" alt=""><figcaption></figcaption></figure>

3. **Understand Each Request's Purpose:**

* Understand the functionality of each request in the API collection.

4. **Test Various Endpoints:**

* Explore endpoints used for updating accounts, group information, user profiles, company profiles, etc.
* Modify requests to test for potential mass assignment vulnerabilities by adding unexpected parameters.

5. **Analyze API Responses:**

* Observe the API’s responses to modified requests to determine if any indicate privilege escalation or unauthorized access.
