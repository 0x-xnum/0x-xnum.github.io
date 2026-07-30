# Improper Model Validation

Improper Model Validation occurs when an ASP.NET application fails to properly validate user input against the model validation framework, or doesn't use the validation framework at all. This allows attackers to submit malicious or unexpected data that bypasses validation rules, leading to data corruption, privilege escalation, business logic bypass, and data injection attacks.

The vulnerability is specific to ASP.NET's model binding and validation system. Developers often bypass or misconfigure the framework, trusting client-side validation or assuming certain data will always be valid.

***

### Real-World Attack Scenarios

#### Scenario 1: Bypassing Required Field Validation

An application has a registration form with client-side validation for required fields:

```csharp
public class UserRegistration {
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email")]
    public string Email { get; set; }
    
    [Required(ErrorMessage = "Password is required")]
    [MinLength(8)]
    public string Password { get; set; }
}
```

**The vulnerability:**

A developer implements validation only on the client-side (HTML5 required attributes), trusting the browser to enforce it:

```html
<form>
    <input type="email" name="Email" required>
    <input type="password" name="Password" required minlength="8">
    <button type="submit">Register</button>
</form>
```

**The attack:**

An attacker bypasses client-side validation by:

1. Disabling JavaScript in the browser
2. Modifying the HTML form
3. Using a tool like Burp Suite to intercept and modify requests
4. Sending a POST request with empty fields

```
POST /register HTTP/1.1
Content-Type: application/x-www-form-urlencoded

Email=&Password=
```

Since server-side validation is missing, the application accepts empty values and creates an invalid account.

**Result:**

* Invalid user accounts in the database
* NULL values in email field
* Broken authentication logic
* Data corruption

**Finding it:** Test registration/form endpoints with empty fields. Disable JavaScript and submit forms. Use Burp Suite to intercept requests and remove field values. Check if the server accepts invalid data.

**Exploit:**

```bash
# Send request with empty required fields
curl -X POST http://example.com/register \
  -d "Email=&Password=&FirstName=&LastName="

# If successful, invalid account is created
```

***

#### Scenario 2: Type Mismatch and Data Type Bypass

An application expects an integer for a product quantity:

```csharp
public class OrderItem {
    [Range(1, 1000, ErrorMessage = "Quantity must be between 1 and 1000")]
    public int Quantity { get; set; }
}
```

**The vulnerability:**

The application doesn't properly validate the type or range:

```csharp
public ActionResult AddToCart(OrderItem item) {
    // No validation check
    // Directly add to cart
    cart.Add(item);
    return RedirectToAction("ViewCart");
}
```

**The attack:**

An attacker submits a non-integer or out-of-range value:

```
POST /cart/add HTTP/1.1
Content-Type: application/x-www-form-urlencoded

Quantity=99999999
```

Or uses a negative number:

```
Quantity=-100
```

**Result:**

* Database accepts invalid quantity
* Integer overflow/underflow occurs
* Customer gets negative quantity (refund instead of purchase)
* Order system breaks

**Finding it:** Test numeric fields with non-numeric values. Submit negative numbers. Submit extremely large numbers. Submit decimal values where integers expected. Check if validation occurs server-side.

**Exploit:**

```bash
# Submit non-numeric value
curl -X POST http://example.com/order \
  -d "Quantity=abc"

# Submit out-of-range value
curl -X POST http://example.com/order \
  -d "Quantity=99999999"

# Submit negative value
curl -X POST http://example.com/order \
  -d "Quantity=-100"
```

***

#### Scenario 3: Privilege Escalation via Role Assignment

An admin dashboard allows creating user accounts and assigning roles:

```csharp
public class CreateUser {
    [Required]
    public string Email { get; set; }
    
    [Required]
    public string Password { get; set; }
    
    public string Role { get; set; }  // No validation!
}
```

**The vulnerability:**

The Role field has no validation. An attacker can assign themselves admin role during registration:

```
POST /admin/create-user HTTP/1.1
Content-Type: application/x-www-form-urlencoded

Email=attacker@example.com&Password=Password123&Role=Admin
```

Or modify an existing user's role:

```
POST /admin/edit-user/42 HTTP/1.1
Content-Type: application/x-www-form-urlencoded

Email=user@example.com&Role=Administrator
```

**Result:**

* Attacker becomes admin
* Full access to all user accounts
* Can delete users, modify data, view sensitive information
* Complete system compromise

**Finding it:** Look for role/permission fields. Test if you can assign yourself higher privileges. Intercept requests and add role parameters. Check if authorization checks exist server-side.

**Exploit:**

```bash
# Register and assign admin role to yourself
curl -X POST http://example.com/register \
  -d "Email=attacker@example.com&Password=Pass123&Role=Admin"

# If successful, you're now admin
curl http://example.com/admin -H "Cookie: sessionid=..."
# Admin panel loads
```

***

#### Scenario 4: Price Manipulation in E-Commerce

An e-commerce site allows customers to submit orders with prices:

```csharp
public class OrderItem {
    public int ProductId { get; set; }
    public decimal Price { get; set; }  // Price sent from client!
    public int Quantity { get; set; }
}
```

**The vulnerability:**

The application accepts the price from the client without validating it against the database:

```csharp
public ActionResult ProcessOrder(OrderItem[] items) {
    decimal total = 0;
    
    // NO VALIDATION - trusts client prices
    foreach (var item in items) {
        total += item.Price * item.Quantity;  // Uses client price!
    }
    
    ProcessPayment(total);
    return RedirectToAction("OrderComplete");
}
```

**The attack:**

An attacker modifies prices before submitting the order:

1. Add item to cart (price: $99.99)
2. Open DevTools and modify the price in the cart data
3. Change price to $0.01
4. Submit order
5. Charged only $0.01 instead of $99.99

Or send modified order:

```
POST /checkout HTTP/1.1
Content-Type: application/json

{
    "items": [
        {
            "productId": 1,
            "price": 0.01,
            "quantity": 1
        }
    ],
    "total": 0.01
}
```

**Result:**

* Massive revenue loss
* Attacker receives products for free or pennies
* Financial fraud
* Company loses money on every transaction

**Finding it:** Intercept checkout requests. Modify prices before submission. Check if prices validated against database. See if client-side prices are trusted.

**Exploit:**

```bash
# Intercept order and modify prices
curl -X POST http://example.com/checkout \
  -H "Content-Type: application/json" \
  -d '{"items":[{"id":1,"price":0.01,"qty":100}]}'

# Charged minimal amount instead of full price
```

***

#### Scenario 5: SQL Injection via Inadequate Validation

An application searches products by category without proper validation:

```csharp
public ActionResult SearchProducts(string category) {
    // No validation, directly used in query
    var products = context.Products
        .Where(p => p.Category == category)
        .ToList();
    
    return View(products);
}
```

While ASP.NET Linq protects against typical SQL injection, inadequate validation can allow other attacks:

```
GET /search?category=Electronics' OR '1'='1
```

Or LDAP injection if category is used in LDAP queries:

```
category=*)(|(
```

**Finding it:** Test input fields with special characters. Try SQL injection payloads. Test LDAP injection if applicable. Check if validation framework is used.

***

#### Scenario 6: Email Validation Bypass

An application requires unique email addresses:

```csharp
public class User {
    [EmailAddress]
    [Required]
    public string Email { get; set; }
}
```

**The vulnerability:**

The application has no server-side validation of email uniqueness. It only checks the \[EmailAddress] attribute format:

```csharp
public ActionResult Register(User user) {
    // Only checks format, not uniqueness
    if (!ModelState.IsValid) {
        return View("Register");
    }
    
    context.Users.Add(user);
    context.SaveChanges();
}
```

**The attack:**

An attacker registers with the same email multiple times:

```
POST /register HTTP/1.1

Email=victim@example.com&Password=Pass123
```

Request twice with same email → creates duplicate accounts

**Result:**

* Duplicate user records
* Email-based lookups break
* Password reset tokens work for multiple accounts
* Authentication vulnerabilities

**Finding it:** Register with the same email twice. Check if duplicates are allowed. Test validation messages. See if database constraints exist.

***

#### Scenario 7: Date Validation Bypass

An application accepts date input:

```csharp
public class Event {
    [DataType(DataType.Date)]
    public DateTime EventDate { get; set; }
}
```

**The vulnerability:**

No validation of date logic (e.g., start date must be before end date, event must be in future):

```csharp
public ActionResult CreateEvent(Event @event) {
    // No validation logic
    context.Events.Add(@event);
    context.SaveChanges();
    return RedirectToAction("ViewEvent");
}
```

**The attack:**

Submit invalid date combinations:

```
EventStartDate=2025-12-31&EventEndDate=2025-01-01
```

(End date before start date)

Or create events in the past:

```
EventDate=2020-01-01
```

**Result:**

* Invalid events in the system
* Broken reporting and analytics
* Business logic failures

***

### Mitigation Strategies

**Always validate server-side** Never trust client-side validation alone:

```csharp
public ActionResult Register(User user) {
    // Check ModelState
    if (!ModelState.IsValid) {
        return View("Register", user);
    }
    
    // Additional business logic validation
    if (context.Users.Any(u => u.Email == user.Email)) {
        ModelState.AddModelError("Email", "Email already exists");
        return View("Register", user);
    }
    
    context.Users.Add(user);
    context.SaveChanges();
    return RedirectToAction("RegistrationComplete");
}
```

**Use data annotations properly**

```csharp
public class User {
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; }
    
    [Required]
    [StringLength(50, MinimumLength = 3)]
    public string Username { get; set; }
    
    [Range(1, 120)]
    public int Age { get; set; }
}
```

**Implement custom validation**

```csharp
public class Event : IValidatableObject {
    public DateTime StartDate { get; set; }
    public DateTime EndDate { get; set; }
    
    public IEnumerable<ValidationResult> Validate(ValidationContext context) {
        if (EndDate <= StartDate) {
            yield return new ValidationResult(
                "End date must be after start date",
                new[] { nameof(EndDate) }
            );
        }
        
        if (StartDate < DateTime.Now) {
            yield return new ValidationResult(
                "Start date must be in the future",
                new[] { nameof(StartDate) }
            );
        }
    }
}
```

**Validate against database**

```csharp
public ActionResult CreateOrder(OrderItem[] items) {
    foreach (var item in items) {
        // Validate price against database
        var product = context.Products.Find(item.ProductId);
        if (product == null) {
            return BadRequest("Product not found");
        }
        
        // Use database price, not client price
        item.Price = product.Price;
        
        if (item.Quantity < 1 || item.Quantity > product.MaxQuantity) {
            return BadRequest("Invalid quantity");
        }
    }
    
    // Process order
    return ProcessOrder(items);
}
```

**Whitelist valid values**

```csharp
public ActionResult AssignRole(int userId, string role) {
    // Whitelist allowed roles
    var allowedRoles = new[] { "User", "Moderator" };
    
    if (!allowedRoles.Contains(role)) {
        return BadRequest("Invalid role");
    }
    
    var user = context.Users.Find(userId);
    user.Role = role;
    context.SaveChanges();
    return Ok();
}
```

**Check authorization on server**

```csharp
public ActionResult EditUser(int userId, User userData) {
    var user = context.Users.Find(userId);
    
    // Authorization check
    if (user.Id != CurrentUser.Id && !CurrentUser.IsAdmin) {
        return Unauthorized();
    }
    
    // Validation
    if (!ModelState.IsValid) {
        return BadRequest(ModelState);
    }
    
    // Update
    user.Email = userData.Email;
    context.SaveChanges();
    return Ok();
}
```

**Use validation frameworks**

* Fluent Validation
* Data Annotations
* Custom validation attributes
* Third-party validators

**Validate on every layer**

* Client-side: UX improvement
* API/Server-side: Security boundary
* Database: Data integrity (constraints)

***

&#123;% embed url="<https://cwe.mitre.org/data/definitions/1173.html>" %}

&#123;% embed url="<https://cwe.mitre.org/data/definitions/1174.html>" %}

&#123;% embed url="<https://fluentvalidation.net/>" %}

&#123;% embed url="<https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation>" %}
