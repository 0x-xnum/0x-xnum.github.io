# Broken Object Level Authorization (BOLA)

### Hunting for Broken Object Level Authorization (BOLA) Vulnerabilities

When searching for BOLA vulnerabilities, focus on three critical components to maximize your chances of success:

#### **1. Resource Identifier (Resource ID)**

* The unique identifier for a resource in the API, such as `user_id=123`. Resource IDs can be simple (e.g., numeric) or complex (e.g., UUIDs or hashes).

#### **2. Requests that Access Resources**

* &#x20;Identify the necessary requests for obtaining resources (create, read, update, delete) that your account shouldn’t have access to. This helps in testing if you can access another user's resources.

#### **3. Access Control Issues**

* Txo exploit a BOLA weakness, the API must lack proper access controls. While predictable resource IDs can be a red flag, this alone doesn’t confirm an authorization vulnerability; genuine absence of access controls is necessary for exploitation.

***

#### Finding Resource IDs and Requests

To effectively identify resource IDs and potential authorization flaws:

* **Explore the Target:** Review the API documentation and make requests to discover resource IDs.
* **Look for Patterns:** Analyze API paths and parameters to predict unauthorized resources.

<figure><img src="/files/bQ2qm2TBaDqDg5dZOUk5" alt=""><figcaption></figcaption></figure>

### Searching for BOLA

Begin by understanding the target application’s purpose and reviewing its documentation. This foundational knowledge will help you focus your search. Ask yourself questions such as:

* What functionalities does the app provide?
* Do users have personal profiles?
* Can users upload files?
* Is there any data specific to user accounts?

**Example Application: crAPI (Vulnerable App)**

* **Purpose:** crAPI is designed for new vehicle purchases, allowing users to buy items, message others in a public forum, and update their profiles.
* **Profiles:** Users have profiles with pictures and basic information (name, email, phone number).
* **Uploads:** Users can upload profile pictures and personal videos.
* **User-Specific Data:** Includes a user dashboard, profile details, past orders, and community forum posts.

**Identified Relevant Requests:**

* `GET /identity/api/v2/videos/:id?video_id=589320.0688146055`
* `GET /community/api/v2/community/posts/w4ErxCddX4TcKXbJoBbRMf` (public information, no authorization needed)
* `GET /identity/api/v2/vehicle/{resourceID}/location`

***

#### Authorization Testing Strategy

#### **A-B-A Testing Approach**

1. **UserA Valid Requests:**
   * Make valid requests as UserA and document operations involving resource IDs.
2. **Switch to UserB:**
   * Attempt to make requests altering UserA's resources using UserB's token.
3. **Verify Changes:**
   * Return to UserA’s account to confirm if alterations were successful.

**Example Request for Vehicle Location:**

* As **UserB**, register a vehicle and trigger the request using the "Refresh Location" button. Capture this request with **Burp Suite**.

Next, replace UserB’s token with UserA’s token in the captured request to check for unauthorized access to UserA's resources.

***

#### Excessive Data Exposure and BOLA

In the request `GET /community/api/v2/community/posts/recent`, excessive data exposure may occur, revealing sensitive information like `vehicleID`. Developers might mistakenly believe that such complex IDs don’t require strict authorization, relying on security through obscurity. However, this complexity only serves to delay brute-force attacks.

By combining this excessive data exposure with the BOLA vulnerability, you can construct a strong proof of concept (PoC) that emphasizes the severity of these vulnerabilities.

For Some Senraios  :

&#123;% content-ref url="/pages/Q7vL4RNf5auFr3y0Iihn" %}
[Broken mention](broken://pages/Q7vL4RNf5auFr3y0Iihn)
&#123;% endcontent-ref %}
