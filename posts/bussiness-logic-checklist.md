# bussiness logic checklist

* [ ] Change the price with another price: 100 -> 50

* [ ] Change the price with a negative price: 100 -> -100

* [ ] Change the price with another price by adding a negative value: 100 -> (+-120)

* [ ] Change the price with another price by multiplying by 0.5: 100 -> (0.5 \* 100)

* [ ] Retrieving a Profile: For example, Jack’s profile can be fetched with id=1001. Change the value from 1001 to 1089 to see another user’s information.

* [ ] Shopping Cart: Test for authentication bypass to log into the shopping cart without paying for items.

**Review Functionality:**

* [ ] Try to post a review as a Verified Reviewer without purchasing that product.
* [ ] Attempt to provide a rating beyond the scale (e.g., 0 or 6).
* [ ] Check if the same user can post multiple ratings for a product (race conditions).
* [ ] Test if file uploads allow any extensions.
* [ ] Try posting reviews as other users.
* [ ] Perform CSRF on this functionality.

**Coupon Code Functionality:**

* [ ] Apply the same code more than once to see if the coupon code is reusable.
* [ ] Test for Race Condition by using the same code for two accounts simultaneously.
* [ ] Attempt Mass Assignment or HTTP Parameter Pollution to add multiple coupon codes.
* [ ] Check for missing input sanitization vulnerabilities (e.g., XSS, SQLi).
* [ ] Tamper with requests to add discount codes to non-discounted items.

#### \[ ] Delivery Charges Abuse

* [ ] Tamper with delivery charge rates to negative values to reduce the final amount.
* [ ] Check for free delivery by modifying parameters.

#### \[ ] Currency Arbitrage

* [ ] Pay in one currency (e.g., USD) and request a refund in another currency (e.g., EUR).

#### \[ ] Premium Feature Abuse

* [ ] Forcefully browse premium account areas or endpoints.
* [ ] Pay for a premium feature, cancel the subscription, and check if the feature remains usable.
* [ ] Use Burp's Match & Replace to manipulate access to premium features.
* [ ] Check cookies or local storage for variables related to premium access.

#### \[ ] Refund Feature Abuse

* [ ] Purchase a subscription and request a refund to see if access remains.
* [ ] Attempt currency arbitrage with refunds.
* [ ] Test for race conditions by making multiple cancellation requests.

#### \[ ] Cart/Wishlist Abuse

* [ ] Add a product with negative quantity alongside positive quantities.
* [ ] Add a product exceeding the available quantity.
* [ ] Check if moving a product from the wishlist to the cart can affect another user's cart.

#### \[ ] Thread Comment Functionality

* [ ] Attempt to post unlimited comments on a thread.
* [ ] Test for race conditions if a user can comment only once.
* [ ] Tamper with parameters to post comments as a verified user.
* [ ] Attempt to impersonate other users when posting comments.

#### \[ ] Parameter Tampering

* [ ] Tamper with payment or critical fields.
* [ ] Use HTTP Parameter Pollution and Mass Assignment to add unexpected fields.
* [ ] Manipulate responses to bypass restrictions (e.g., 2FA).
* [ ] Attempt parameter tampering to manipulate product prices.

#### \[ ] Exam Result Manipulation

* [ ] Complete exams and retake them, then manipulate requests to obtain certificates with correct answers.

#### \[ ] Authentication Flags and Privilege Escalation

* [ ] Observe HTTP traffic for suspicious parameters related to ACL/Permission.
* [ ] Analyze parameter values for potential tampering (e.g., encoding changes).

#### \[ ] Critical Parameter Manipulation

* [ ] Observe HTTP traffic for parameters that can be tampered with for unauthorized access.

#### \[ ] Developer Cookie Tampering

* [ ] Capture cookie responses and manipulate them to access other users’ information.

**Functions where authorization might be important:**

* [ ] Profile Update Functions
* [ ] Order History Functions
* [ ] Cart Functions
* [ ] Payment Functions
* [ ] Any Other CRUD Functions
