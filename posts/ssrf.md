# SSRF

#### Introduction to Server-Side Request Forgery (SSRF)

Server-Side Request Forgery (SSRF) is a vulnerability that occurs when an application retrieves remote resources without validating user input. This flaw allows an attacker to control the resources requested by the server, often through a user-supplied URL. With control over these requests, attackers can gain access to sensitive data or potentially compromise a vulnerable host. SSRF is listed as #10 on the 2021 OWASP Top 10 and is a rising threat, especially to APIs.

#### SSRF Impact

The impact of SSRF can be significant:

* **Data Exposure**: An attacker may retrieve private data from internal services.
* **Network Scanning**: SSRF can enable attackers to scan the target’s internal network.
* **Remote Code Execution**: In some cases, SSRF can lead to remote code execution.

#### Types of SSRF

1. **In-Band SSRF**: The server responds with information from the specified URL, which can reveal data to the attacker directly.
2. **Blind SSRF**: The server makes the request but does not send the response back to the attacker. The attacker can verify the request by monitoring a server they control.

**Example of In-Band SSRF**

* **Intercepted Request**:

  ```json
  POST /api/v1/store/products
  { "inventory": "http://store.com/api/v3/inventory/item/12345" }
  ```
* **Attack**:

  ```json
  POST /api/v1/store/products
  { "inventory": "http://localhost/secrets" }
  ```
* **Response**:

  ```json
  { "secret_token": "crapi-admin" }
  ```

**Example of Blind SSRF**

For Blind SSRF, you can use a service like [webhook.site](http://webhook.site) to confirm the server’s request:

* **Attack**:

  ```json
  POST /api/v1/store/products
  { "inventory": "https://webhook.site/<random-URL>" }
  ```

Check webhook.site for request logs instead of relying on the application response.

#### Ingredients for SSRF

When hunting for SSRF vulnerabilities in an API, look for:

* Full URLs or URL paths in POST bodies or parameters.
* Headers that may contain URLs (e.g., Referer).
* Any form of user input that the server might process as a URL.                                                                                                                                       &#x20;

#### Testing SSRF in crAPI

Identify requests involving URLs, such as:

* **POST /community/api/v2/community/posts**
* **POST /workshop/api/shop/orders/return\_order?order\_id=4000**
* **POST /workshop/api/merchant/contact\_mechanic**

### Testing for SSRF

Either using Postman or the web browser, proxy the requests that you are targeting to Burp Suite.<br>

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/mQzAVhkrTO2Rfoumy9Rt_ssrf6.PNG" alt=""><figcaption></figcaption></figure>

Next, send the request over to Repeater to get an idea of a typical response.

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/IIcdakknQ7y3KjFakNwZ_ssrf5.PNG" alt=""><figcaption></figcaption></figure>

In the case of the return\_order request, we are able to return an item once. If we attempt to return the item twice then we will receive a response that the item has already been returned.<br>

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/yCTiwfUTTvCKxvKjQGmo_ssrf7.PNG" alt=""><figcaption></figcaption></figure>

To test SSRF on the `return_order` endpoint, you’ll need several valid `order_id`s. Use the `POST /workshop/api/shop/orders` endpoint to purchase multiple items, allowing you to generate distinct `order_id`s for the attack.<br>

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/Mg93ujzSVyb0MeTJFXcw_ssrf8.PNG" alt=""><figcaption></figcaption></figure>

**Use Burp Intruder with Pitchfork Attack Mode**: The Pitchfork mode in Burp Intruder pairs separate payloads, which is ideal here as it allows you to pair each valid `order_id` with different SSRF payloads (target URLs).                           <br>

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/hIDoWht0RfWW60zFfSdT_ssrf9.PNG" alt=""><figcaption></figcaption></figure>

**Payload 1**: Use valid `order_id`s generated in the previous step.

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/YGjcoz6XSxmzhQ5KK0iC_ssrf10.PNG" alt=""><figcaption></figcaption></figure>

**Payload 2**: Set the second payload to potentially interesting URLs including your webhook.site URL.  For additional SSRF payload ideas check out [PayloadAllTheThings SSRF List](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery).<br>

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/J6R516giR0G2dLNZGO9B_ssrf11.PNG" alt=""><figcaption></figcaption></figure>

**Analyze Responses**: Send the requests and carefully observe the response codes, lengths, and any indicators of unusual behavior.

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/Z79Q0RK9Q4mKQa09HsF3_ssrf12.PNG" alt=""><figcaption></figcaption></figure>

**Look for SSRF Evidence**: If you control the resources (e.g., via webhook.site), check there for incoming requests. An incoming request from the server can confirm SSRF exploitation even when responses don’t reveal server interaction directly.  Again, the URL was not requested and this request does not appear to be vulnerable. Notice the requests shows 0/500 and the message "Waiting for first request...".<br>

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/DQe3MF8aThyrpztmuWZy_ssrf14.PNG" alt=""><figcaption></figcaption></figure>

Let's try this out on the contact\_mechanic request. Set the attack position, copy and paste the payloads you previously used for URLs, and send the attack. Review the results and see if there is anything interesting.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/rCAq7jpJRwabZ2rzmXVH_ssrf15.PNG)

Sure enough, the localhost requests fail, but the other URLs provided are successful. As far as reviewing for anomalies, we can see that there are a variety of status codes and response lengths. Upon reviewing the responses from the successful requests, we can see that the remote resources we requested were sent back over the API request. <br>

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/wrsbVrSBRrmtndHVIgfd_ssrf16.PNG" alt=""><figcaption></figcaption></figure>

We can also verify that a request was made from the server by visiting our webhook.site page.<br>

<figure><img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/WtZCOd2TStefkvaIIhYK_ssrf13.PNG" alt=""><figcaption></figcaption></figure>

&#x20;Congratulations, you have successfully exploited an SSRF vulnerability!
