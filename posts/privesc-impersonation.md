# Privilege Escalation via Impersonation Features



hey there,

I’m going to share with you an interesting bug I found in one of Bugcrowd’s programs. It was in a program that had an “impersonating user” feature.

## What is Impersonation?
Impersonation allows administrators to “login as” another user without knowing their credentials. This feature is common in platforms where admins need to debug issues, review user permissions, or resolve complaints. Admins use impersonation to:
- Troubleshoot issues reported by users.
- Validate user-specific configurations.
- Debug platform behavior from the user’s perspective.

For example, if a user reports that he cannot access his files, an admin might impersonate that user to test the upload feature and investigate the problem. Once resolved, the admin ends the impersonation session and resumes their admin account.

![Impersonation Concept](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*v-NSkyIYKgXUahxa.png)

## How Impersonation Works on our target
Now let’s talk about the target system — example.com. The platform offers a feature to impersonate users to help them troubleshoot their issues. Here’s how it works:

**1. Admin Impersonates a User:**
- The admin navigates to the “Users” section and selects the user they want to impersonate.
- Upon clicking Impersonate, the application creates a new session for the admin to act as the user.

**2. Session Management:**
- While impersonating, the admin’s original session is paused.
- The impersonation session is created and tied to the user being impersonated.

**3. Stopping Impersonation:**
- When the admin clicks Stop Impersonating, they are returned to their original session.

While i was investigating the “Active Sessions” feature (which is part of a different endpoint), I found that the page displays all active sessions tied to the logged-in user, including those created during impersonation. It shows something like this:

![Active Sessions UI](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*CMEm8C5udbwZLSRqvZfpnw.png)

I decided to revoke all sessions except for the current one (important for cleaning up unnecessary sessions). Now, the only active session is mine.

![Clean Sessions UI](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*FPqlwsUbSx2l_0Wjkm3BCg.png)

now when the admin impersonates my account, we will find something interesting:

![Admin Impersonation UI](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*FBIZQNLHtF9uX-Wg16elaQ.png)

Yes, when the admin impersonates my account, it shows up in the Active Sessions section with its session ID visible in two ways:

**Option 1:** By inspecting the “Revoke” button in the HTML source of the page (using F12 in your browser’s developer tools). The session ID is exposed in the request payload. It looks like this:

```html
<button id="revoke-session-1234" class="revoke-button" data-session-id="abcd1234efgh5678ijklmnop">Revoke</button>
```

**Option 2:** By intercepting the HTTP request after clicking the “Revoke” button, which looks like this:

```http
POST /users/sessions/revoke HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer <access_token>

{
  "session_id": "abcd1234efgh5678ijklmnop"
}
```

i copied the session ID from the intercepted request and dropped it. Now, I had the temporary session ID assigned to the admin who was impersonating my account.

## To take control, I:
Opened F12, went to the Cookies section, and replaced the session ID with the admin’s session ID:

![Replacing Cookie](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Z4lun-Z_qw7uTiBLZCkm1w.png)

Alternatively, I could manually set it in the console:

```javascript
document.cookie = "_session_id=SESSION_ID";
```

then refreshed the page.

At this point, I was logged in as the admin, impersonating my account.

## What Happened?
The reason the exploit worked was that the temporary session was still active, and the admin was still impersonating it. Essentially, I jumped into the temporary session used by the admin, rather than the admin’s original session.

When the admin requests to impersonate a user, the app first checks if the user is an admin. If they are, a temporary session ID is created for the impersonation. When the admin stops impersonating, the application checks the session ID and restores the admin’s original session.

### That is the Key part
After getting into the admin’s account, I clicked “Stop Impersonating,” which ended the impersonation session and restored the admin’s original session. However, since the system only checked the temporary session ID value...

I was able to remain logged in as the admin, effectively escalating my privileges.

## Steps to Exploit the Vulnerability:
1. Log into the platform as a regular user (attacker).
2. Revoke all the sessions in the “Active Sessions” tab, except for the attacker’s session.
3. The admin logs in and impersonates the attacker’s account.
4. After impersonating, the admin receives a temporary session ID associated with the impersonated user (attacker).
5. The attacker inspects the “Revoke” button in the HTML source of the page or intercepts the request and copies the session ID from the request.
6. The attacker opens the Developer Console (F12), goes to the Cookies section, clears their existing session cookies, and replaces it with the admin’s session ID. Then, they refresh the page.
7. Now, the attacker is logged in as the admin. The attacker can now click the “Stop Impersonating” button, which the admin would normally click.
8. Since the attacker’s session is now using the impersonated session ID, clicking “Stop Impersonating” will return them to the admin’s original session.

In the end, the company acknowledged the issue as a high severity and got paid $$$$

---
Thanks for reading!
