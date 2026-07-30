# How I Escalated Simple HTML Injection to SSRF via PDF Rendering



hello everyone,
it’s been a while since I wrote something here, but yeah, I guess I’m back.

![Intro GIF](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*cfUF9xZ9N77q3SmmTk7h-g.jpeg)

Grab your coffee, and let’s get started! 😉

## First things First …
while testing a subdomain `learn.target.com`, which a .net website that hosts online courses, I initially reported a couple of low-severity bugs. nothing exciting.
I thought I was done.

but then I realized that the only part I hadn’t looked into yet was the course system itself. I noticed there were some paid courses on the platform. one of them was a business course for $5. I figured that was a small price to pay to explore the feature properly.

so I paid and started testing from inside. I tried the usual stuff like bypassing payments, accessing locked content, checking for IDORs, and looking for exposed assets. nothing worked.

eventually I gave up and moved on to other targets. the course was not bad so I kept going through it while focusing on a different program.

a couple of weeks later, after completing the course, I got this message:

![Course Message](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OQyJDxjXcG0lVSTaiB1RFw.png)

well, well, well… things started to get a bit interesting
the site offered me a certificate generator to download my certificate.

that’s where everything started.

I left my current target to take a closer look at the certificate generator feature.

## The certificate generator feature
the page let you enter:
- Your name
- Your title
- (Optionally) a quote or message

then you click “generate”, and the server gives you back a PDF or PNG file with your info on a nice certificate template.

the request body looked something like this:

```json
{
  "name": "name",
  "title": "title",
  "quote": "quote "
}
```

seemed innocent at first. I got back a normal Picture.

![Certificate Example](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*-QgC2cmw_E5j_QoS_6iv3Q.png)

sounds innocent, right?

As always, I started by testing all input parameters for injection. The `title` parameter was the paramter that have no input sanitization.

I replaced it with a simple HTML tag:

```json
"title": "<i>ahmed</i>"
```

Since there was no input sanitization, the HTML payload got rendered in both pdf and image in italic font. the cert was somthing like that :

![HTML Injection Result](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*cybgM1fvIzC7Xo84HqNWSg.png)

After a bit of investigation and trying out a variety of HTML tags and attributes, I noticed the server wasn’t just taking my input and throwing it into the PDF as plain text. It was actually trying to render it. Normal HTML tags were being processed and displayed, while anything related to JavaScript like `<script>` or event handlers was being filtered out or ignored.

This meant I had HTML injection, but not XSS.

still, something was rendering HTML on the backend. definitely worth digging deeper.

## Identifying SSRF
next, i wanted to test if this rendering process would actually fetch resources from the iframe.

so i modified the payload:

```json
"title": "<iframe src='http://sub.tarek.dev/probe'></iframe>"
```

and set up a listener on my server.

when i opened the PDF…

![DNS Callback](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*TNB2I2obt6qBeFs4PHXssQ.png)

I received a DNS and HTTP hit to listener from the IP address of the web application server.

Now i confirmed that the server was rendering HTML and loading iframe content from our input.
that’s SSRF via HTML Injection in the PDF rendering flow.

![Architecture Diagram](https://miro.medium.com/v2/resize:fit:600/format:webp/0*h6WrNoIqQ6EIr6IQ.png)

## Going for AWS metadata
Having SSRF is one thing, but proving its impact is another. Since the server was making outbound HTTP requests, I suspected it might be hosted on AWS — and if so, I wanted to reach the Instance Metadata Service (IMDS).

To begin the attack, I injected a new payload designed to access the metadata endpoint:

```json
"title": "<iframe src='http://169.254.169.254/latest/meta-data/iam/security-credentials/'></iframe>"
```

The generated PDF included the IAM role name, `my-app-instance-role`, confirming that the server was running on an AWS EC2 instance with IMDSv1 enabled. IMDSv1 is particularly vulnerable because it does not require authentication tokens, unlike IMDSv2.

I then crafted another payload to target the specific IAM role and leak temporary credentials:

```json
"title": "<iframe src='http://169.254.169.254/latest/meta-data/iam/security-credentials/my-app-instance-role'></iframe>"
```

The resulting PDF contained sensitive data, including:
- AccessKeyId
- SecretAccessKey
- Token

these were temporary AWS credentials, depending on the permissions tied to that IAM role, this could’ve granted access to other AWS services like S3, DynamoDB, etc.

![Metadata Export](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*KzU4JMvYI6lhaYbc4BUMMQ.png)

Besides AWS access keys, see if there is any sensitive data in the `user-data` IMDS endpoint

```url
http://169.254.169.254/latest/user-data
```

the user-data section often includes bootstrapping scripts, environment variables, or even hardcoded secrets in plaintext.

## what really happened?
this might seem weird at first. how does putting an iframe in a PDF result in the server sending HTTP requests?

here’s the answer.

when i submit my name, title, and quote, the server builds an HTML version of the certificate and passes it to a rendering engine like `wkhtmltopdf`, `puppeteer`, or another headless browser tool.
those tools process the full HTML just like a normal browser would. that means they fetch images, iframes, stylesheets, and more — automatically — to fully render the page before converting it to a PDF or PNG.

so when you include this:

```html
<iframe src="http://169.254.169.254/latest/meta-data/"></iframe>
```

the rendering tool on the server sees that iframe and tries to fetch the content from that IP address.
and since `169.254.169.254` is the AWS metadata IP, this turns into an SSRF from inside the cloud environment.

your input is not just sitting in the PDF — it is being processed and loaded before the final file is created.

this is the core of the vulnerability.
you’re abusing HTML injection, not to trigger JavaScript or do XSS, but to trick the server into making internal HTTP requests by rendering your iframe or other HTML tags.

In the end, the company lowered the severity because of their policy, but I still got a $1,000.

![Bounty Proof](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*IiJIdKhJ2-hYcIUIJCZxjg.png)

---
Thanks for reading!
