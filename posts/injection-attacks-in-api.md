# Injection Attacks in API

Before injecting a payload into an API, it's essential to identify the most effective requests to target. The best way to find these injection points is by fuzzing and then analyzing the responses you receive. Fuzzing is the process of sending various inputs to an endpoint to provoke an unintended response, helping you discover vulnerabilities. Inputs could include symbols, numbers, system commands, SQL and NoSQL queries, emojis, hexadecimal, and boolean statements. The objective is to observe whether the API fails to handle certain types of input, resulting in a verbose response or unintended behavior.

### Key Areas for Fuzzing

Focus your fuzzing efforts on:

* **Headers**
* **Query String Parameters**
* **POST/PUT Parameters**

Targeting these areas will increase the chance of finding vulnerabilities, especially if you understand the target's technology stack (e.g., database type, OS, programming language). Knowledge of the stack allows for more focused payloads.

### Recognizing Injection Vulnerabilities

Look for verbose error messages, delays in processing time, internal server errors, or database errors. A message like “SQL Syntax Error” or a delayed response could indicate that your payload bypassed security controls and was interpreted at the system or database level. Additionally, if a weakness is discovered in one endpoint, it may also be present in others.

### Discovering Injection Vulnerabilities

To exploit an injection vulnerability, it's crucial to identify the best places to fuzz and the types of payloads to use. The right payload can often be inferred from reconnaissance. Start by testing widely across the API, then narrow down to specific requests. In this guide, we use Postman to fuzz broadly across the API collection and then leverage Burp Suite and Wfuzz for targeted fuzzing.

### Input Types to Test for Injection

During fuzzing, consider testing the following input types:

* A very large number
* A very large string
* A negative number
* Strings (instead of expected number or boolean values)
* Random characters
* Boolean values
* Meta characters

If an input type triggers a verbose error or delay, this could signal an injection vulnerability.

### SQL Injection and Metacharacters

SQL metacharacters are symbols that SQL interprets as commands rather than data, such as `--`, which starts a comment in SQL. If an endpoint does not filter out SQL syntax, it may allow an attacker to manipulate the backend database.

For instance, if the API does not validate user input, an SQL injection could enable unauthorized access or manipulation of sensitive data. Here are some common SQL metacharacters to try:

* `'`
* `--`
* `;`
* `' OR '1`
* `' OR 1=1 --`

For example, suppose an API query checks for username and password using SQL syntax. By using an input like `' OR 1=1 --`, you can bypass authentication by turning the query into one that always evaluates to true.

&#123;% content-ref url="/pages/XLqxACXrE5yH6Ovo4J3E" %}
[SQL Injection](/sql-injection)
&#123;% endcontent-ref %}

### NoSQL Injection

APIs often use NoSQL databases, which handle large-scale architectures efficiently. Due to lesser-known injection techniques, NoSQL databases can sometimes be more susceptible to injection attacks.

Unlike SQL, NoSQL databases have varied structures, so payloads differ across NoSQL types. Here are some common NoSQL payloads:

* `$gt`
* `{"$gt": ""}`
* `$ne`
* `{"$ne": ""}`
* `$nin`
* `{"$nin": [1]}`
* `{"$where": "sleep(1000)"}`

In MongoDB, for example, `$gt` selects documents greater than a specified value, while `$nin` is a "not in" operator. These payloads can sometimes bypass authentication or create delays, revealing injection vulnerabilities.

&#123;% content-ref url="/pages/EWnqaN19WyrBoyRpfvEq" %}
[NoSQL Injection](/nosql-injection)
&#123;% endcontent-ref %}

### OS Command Injection

In OS command injection, you inject commands to the operating system instead of SQL queries. Understanding the target OS helps determine which commands to use. Begin by finding an injection point, such as a URL query, parameter, or header.

Command separators like `|`, `&`, `;`, and `&&` allow chaining commands within a single line. Here are some example commands:

**Windows**

* `ipconfig` – shows network configuration
* `dir` – lists directory contents
* `ver` – shows OS version

**Linux/Unix**

* `ifconfig` – shows network configuration
* `ls` – lists directory contents
* `pwd` – shows the current directory

<https://medium.com/@aka.0x4C3DD/os-command-injection-vulnerabilities-in-web-applications-49420429b53>

&#123;% embed url="<https://0xn3va.gitbook.io/cheat-sheets/web-application/command-injection>" %}

## Fuzzing Wide with Postman

Postman really shines when it comes to testing an entire API collection thanks to the Collection Runner. Whereas, Burp Suite CE and WFuzz are much better at digging into individual requests. Since there are so many places for an injection vulnerability to hide it helps to cast a wide net across a collection for weaknesses with Postman and then transition to other tools. We will be testing so many requests that I recommend duplicating the entire collection so that we can add variables throughout the collection. This will continue to maintain the integrity of the original collection and let us develop a baseline of expected responses.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/aA6AoJ0QzWfh0DK1k32g_Injection2.PNG)

I have renamed the duplicate collection to crAPI\_Swagger Fuzz. We can create a fuzzing environment that can be reused from one collection to another.&#x20;

## Injection Targets

For injection targets, we will begin by casting a wide net and seeing which requests respond in interesting ways. Let's target many of the requests that include user input. With this in mind, I have selected the following requests.

* PUT videos by id
* GET videos by id
* POST change-email
* POST verify-email-token
* POST login
* GET location
* POST check-otp
* POST posts
* POST validate-coupon
* POST orders

Now, let's use the crAPI\_Swagger collection and the Collection Runner to establish our baseline. Select the ten requests listed earlier and ensure they are well-formed to avoid failures due to authorization issues or missing resources. After updating your token, run the entire collection to assess the baseline, noting the number of successful and failed requests.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/sDzM7qtWSvKa6BNRL8K1_Injection8.PNG)

&#x20;

In this baseline, we can see that there were:

* **Three** 200 Success responses
* **Three** requests received 500 Internal Server Error
* **Three** 404 Not Found
* **One** 403 Forbidden

You can explore the variety of reasons that each response was sent, but if you have well-formed requests then proceed. Now that we have a baseline, let's update our environment with some fuzzing variables.&#x20;

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/Wpg8VfVJS5S3LRwUW2j6_Injection4.PNG)

Now depending on information from reconnaissance, you may want to start with a specific fuzzing variable. However, it is easy enough to update the values of the variables, so I will stick with &#123;&#123;fuzz}}. Now go through the requests that you are targeting and add fuzzing variables where user input is found.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/DJAB2Nl4Rz6i5UCMKth0_Injection3.PNG)

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/9oUm8Qz7SZmlPcrByvHC_Injection7.PNG)

Now run the collection with the fuzz variable set throughout the targeted requests and investigate the results for anomalies.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/scIfFU2ROa1TEWl8uU4L_Injection9.PNG)

In this test the total count was:

* **One** 200 Success
* **Four** 500 Internal Server Error
* **Three** 404 Not Found
* **One** 400 Bad Request

One request passed, making it worthwhile to investigate the response. The community request was successful, and the fuzzing variables were posted in a community post. Additionally, review the "Failed" results for any anomalies or interesting findings, such as verbose error messages. After this review, we will repeat the process with updated fuzzing variables.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/wz5nfYTTy2CgMDpggVJs_Injection10.PNG)

Simply update the current value of fuzz with a new test, then use the collection runner, and review the results for anomalies.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/AKQonp3dRciEz9BsvvFP_Injection11.PNG)

Sure enough, we see very similar results. In this test the total count was:

* **One** 200 Success
* **Four** 500 Internal Server Error
* **Two** 404 Not Found
* **Three** 400 Bad Request

The community post was successful, while the others failed similarly. Although there were some variations in the number of 400 Bad Requests, the responses were as expected upon investigation. This indicates a new baseline is forming: the application behaves predictably when fuzzed with certain inputs. By updating our fuzzing variables correctly, any changes will be more apparent. So far, we've conducted SQL and OS injection tests. Now, let's try a NoSQL injection test.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/5GA2lmARRM6hok5xwtDy_Injection13.PNG)

<img src="https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/sjMMlvTxTHKeqmORhwMZ_Injection14.PNG" alt="" height="655" width="806">

At first glance, this test is slightly different. The community post was not successful and upon reviewing the failed results we see the count has changed:

* **One** 500 Internal Server Error
* **Eight** 400 Bad Request
* **One** 422 Unprocessable Entity

The variation in the results here is worthy of investigation, especially with the new response. First, the request to the forum that was successful is now a 400 with the response body,

{"error": "invalid character '$' after object key:value pair"}

The POST validate-coupon request has the 422 Unprocessable Entity response and also contains the same error in the response body.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/EojSxlYpQaW4xVdUgQrw_Injection15.PNG)

These two requests are worth exploring further in Burp Suite. Proxy these two requests to Burp Suite and send the captured requests to Intruder.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/1347bf6-6115-120-45b2-bbb4ffab3352_injection18.PNG)

Using Intruder, update the attack positions for the two requests that you are targeting.&#x20;

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/7fca450-53ab-b1e-af13-6fa13fc6f2_injection17.PNG)

Since the NoSQL payload was the one that triggered an anomaly, update Intruder with a NoSQL payload list. Try this attack with Payload Encoding turned on/off to see if you notice a difference in the responses. Send the attack.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/fb4e86a-2de2-1b6d-44ea-ef52402f3a_injection19.PNG)

Now we are receiving several "200 Success" responses and we have obtained valid coupon codes for sending true statements to the database. We have successfully exploited a NoSQL vulnerability! Next, let's check out how this would be performed with WFuzz.

&#x20;

## Fuzzing Deep with WFuzz

To perform this attack with Wfuzz, you will need to build out the request. I suggest using the Burp Suite save to file so that you can easily copy and paste to the terminal. Using Repeater, you can right-click on the request that you would like to target and select "copy to file".

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/7ac1c3-40ac-61d-7ea0-3c56a5601df5_injection20.PNG)

Next, you can open a second window and use cat on the file that you saved from Burp Suite.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/530baff-fcb2-efe2-066a-01642d68c2d5_injection21.PNG)

&#x20;

You can start building out the WFuzz attack. For additional information about WFuzz, use:\
$wfuzz --help

Start by specifying the payload that you will use with **-z**. Use **-H** to add necessary headers like "Content-Type:application/json" and the authorization token. Then use **-d** to specify the post body. When you are using quotes in a post body, you will need to use backslashes (\\) for those to show up in the request. Finally, add the URL that you are targeting.

`$ wfuzz -z file,usr/share/wordlists/nosqli  -H "Authorization: Bearer TOKEN" -H "Content-Type: application/json" -d "{\"coupon_code\":FUZZ} http://crapi.apisec.ai/community/api/v2/coupon/validate-coupon`

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/d5f1c6-8c48-8755-1a3b-f1aee8bd443_injection22.PNG)

&#x20;

Once you have a successful attack, you can add filtering options to your requests. This will help make the response very clear when there is a successful attack. In the case of this request, we know that we are looking for responses that come back with a 200 status code. Use the show code option **--sc 200** to filter out the results.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/dfa0bbb-0d34-addd-d7c-4001e162fd0_injection23.PNG)

Success! Congratulations on performing an injection attack!

Before you go, check out the following advice if you need to troubleshoot you WFuzz attacks. Since WFuzz attacks can get large and complicated, I recommend getting comfortable with proxying traffic to Burp Suite. Use the -p localhost:8080 option with Burp Suite set to Intercept Requests.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/36a887-6ddf-2c-a507-742417c37c0_injection24.PNG)

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/file-uploads/site/2147573912/products/e817b5-ced6-ab-8e70-63815edfca1_injection25.PNG)

With the request in Burp Suite, you can see exactly what is being sent by WFuzz and troubleshoot from there. For example, see what happens if you do not add backslashes to the quotes around coupon\_code. &#x20;

&#x20;
