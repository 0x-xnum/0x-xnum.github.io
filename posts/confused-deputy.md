# Confused Deputy

A Confused Deputy vulnerability occurs when a privileged service performs actions on behalf of a user without properly verifying who initiated the request. The deputy (the privileged service) gets confused about which party it should trust, allowing attackers to trick it into performing unauthorized actions.

The attacker doesn't need the deputy's credentials—they exploit the trust relationship between the deputy and other systems. If you can make the deputy believe *you* are making the request, it will use its own high privileges to execute your commands.

### Why It Matters

This vulnerability sits in the gray area between authentication and authorization. The deputy *is* authenticated, it's just confused about whose behalf it's acting on. This makes it a reliable finding in real-world applications, especially those with:

* Complex service-to-service interactions
* Cross-domain or cross-account operations
* Implicit trust between components
* Privilege elevation without proper context validation

***

### Real-World Attack Scenarios

#### Scenario 1: File Processing Service

A web application has a file conversion service running as root that processes user-uploaded documents.

**The vulnerability:**

```
User A → Web App → File Converter (runs as root)
```

The web app passes a file path to the converter: `convert_file("/uploads/user_a.doc")`

An attacker manipulates the application to pass a different path: `convert_file("/etc/shadow")`. The converter, running as root, reads the restricted file because it trusts the web app without verifying the original requester.

**Finding it:** Check where file paths originate. Can you inject different paths through parameters, headers, or indirect references? Does the processing service validate file ownership?

***

#### Scenario 2: Cross-Account AWS S3 Access

A Lambda function assumes an IAM role with broad S3 permissions and accepts a bucket name as input:

```python
def list_bucket_contents(bucket_name):
    s3_client = boto3.client('s3')
    response = s3_client.list_objects_v2(Bucket=bucket_name)
    return response
```

**The vulnerability:** You're an authenticated user, so Lambda accepts your request. You ask it to list someone else's bucket: `list_bucket_contents("competitor-secrets-bucket")`. Lambda uses its own IAM role (with cross-account permissions) to perform the action.

**Finding it:** Map service permissions. Identify functions accepting resource identifiers. Can you request actions on resources you shouldn't access? Check CloudTrail logs—actions show Lambda's credentials, not yours, obscuring the source.

***

#### Scenario 3: Cross-Account Third-Party Access (AWS)

A third-party monitoring service (Example Corp) is granted access to your AWS account through an IAM role. You set up a trust relationship allowing their AWS account to assume role `ExampleRole`.

**The vulnerability:**

1. You give Example Corp the role ARN: `arn:aws:iam::111122223333:role/ExampleRole`
2. Example Corp assumes the role using this ARN
3. Another customer discovers or guesses your role ARN
4. That customer asks Example Corp to assume the same role
5. Example Corp (unable to distinguish requests) assumes the role on behalf of the attacker
6. The attacker now accesses *your* resources

**The fix:** Add an external ID—a shared secret only you and Example Corp know. Update your role's trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::987654321098:root"
    },
    "Action": "sts:AssumeRole",
    "Condition": {
      "StringEquals": {
        "sts:ExternalId": "550e8400-e29b-41d4-a716-446655440000"
      }
    }
  }
}
```

Now Example Corp must include your external ID in every AssumeRole request. Other customers can't forge it, so the attack fails.

***

#### Scenario 4: Cross-Service CloudTrail to S3

AWS CloudTrail writes logs to an S3 bucket in a separate account. The bucket policy allows the CloudTrail service principal (`cloudtrail.amazonaws.com`) to write logs—without additional conditions.

**The vulnerability:**

1. Account A creates an S3 bucket and allows `cloudtrail.amazonaws.com` to write
2. Account B (attacker's account) configures CloudTrail to write to Account A's bucket
3. CloudTrail writes logs with the service principal's permissions
4. Account B's logs mix with Account A's, potentially exfiltrating data

**The fix:** Use source context conditions. Add `aws:SourceAccount` to ensure CloudTrail writes logs only on behalf of the bucket owner:

```json
{
  "Sid": "AWSCloudTrailWrite",
  "Effect": "Allow",
  "Principal": {"Service": "cloudtrail.amazonaws.com"},
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::my-logging-bucket/logs/*",
  "Condition": {
    "StringEquals": {
      "aws:SourceAccount": "111122223333"
    }
  }
}
```

AWS provides condition keys for service principals:

* **`aws:SourceArn`**: Specific resource (e.g., a CloudTrail trail)
* **`aws:SourceAccount`**: Specific AWS account
* **`aws:SourceOrgID`**: AWS Organization
* **`aws:SourceOrgPaths`**: Organizational path

***

#### Scenario 5: Email Service Impersonation

An internal email service sends notifications and authenticates users but doesn't validate the "from" field:

```json
POST /send-email
{
  "to": "admin@company.com",
  "from": "ceo@company.com",
  "subject": "Urgent: Password Reset",
  "body": "Click here to reset..."
}
```

**The vulnerability:** The email service (running with elevated privileges) trusts user input and sends emails appearing from the CEO. Attackers use social engineering through apparent authority.

**Finding it:** Look for services that relay requests on behalf of users. Test if the service validates claimed identity against authenticated user. Can you modify "from," "on-behalf-of," or similar fields?

***

#### Scenario 6: Privilege Escalation via Batch Jobs

A scheduled batch job runs privileged operations and reads parameters from a shared database. An unprivileged user can insert rows:

```
User (unprivileged) → Database → Batch Job (runs as admin) → System
```

An attacker inserts: `{"action": "delete_user", "target": "admin_account"}`. The batch job executes the command because it trusts the database without validating who inserted the row.

**Finding it:** Hunt for indirect request mechanisms—message queues, shared databases, scheduled tasks. Can you inject data processed by a more privileged component?

***

### Exploitation Examples

#### Testing AWS IAM Roles

```bash
curl -X POST https://app.internal/api/export \
  -H "Authorization: Bearer $YOUR_TOKEN" \
  -d '{"bucket": "restricted-company-data"}'
```

#### Testing File Service

```http
POST /convert
{"file": "../../../../etc/passwd"}
```

#### Testing Message Queues

```sql
INSERT INTO job_queue VALUES ('delete_user', 'admin_account');
```

Wait for the scheduled job to process it. Does it execute your command with elevated privileges?

#### Testing Cross-Account Access

```bash
aws iam get-role --role-name ThirdPartyRole
```

Look for trust policies without `sts:ExternalId` conditions.

#### Testing Service Principal Permissions

```bash
aws s3api get-bucket-policy --bucket my-bucket
```

Look for statements allowing service principals without `aws:SourceAccount`, `aws:SourceArn`, or `aws:SourceOrgID` conditions.

***

### Mitigation Strategies

**Use external IDs for cross-account access** Always require an external ID when allowing other AWS accounts to assume roles. Make it a shared secret controlled by the deputy, not delegating parties.

**Implement source context conditions** Use `aws:SourceArn`, `aws:SourceAccount`, `aws:SourceOrgID`, or `aws:SourceOrgPaths` whenever granting permissions to AWS service principals.

**Validate request origin at every step** Don't assume downstream services will re-verify. Each service should independently confirm the requester's identity and authority.

**Implement explicit delegation tokens** Include the original requester's identity alongside requests, cryptographically signed.

**Apply least privilege** Restrict what each service can do, not just who can access it.

**Add context validation** Cross-check resource ownership against the authenticated user.

**Centralize trust relationship monitoring** Use resource control policies (RCPs) in AWS Organizations to enforce confused deputy controls across all accounts.

***

### Reports

&#123;% embed url="<https://vavkamil.cz/blog/2021-11-25-wordpress-plugin-confusion-update-can-get-you-pwned/>" %}

&#123;% embed url="<https://hackerone.com/reports/1364851>" %}
