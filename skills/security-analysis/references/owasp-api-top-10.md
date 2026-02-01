# OWASP API Security Top 10 (2023)

Reference document for classifying vulnerabilities against the OWASP API Security Top 10.

## OWASP API Top 10 Quick Reference

| ID | Vulnerability | Check For |
|----|---------------|-----------|
| API1 | Broken Object Level Authorization | Missing ownership checks |
| API2 | Broken Authentication | Weak auth, exposed credentials |
| API3 | Broken Property Level Authorization | Mass assignment, sensitive fields |
| API4 | Unrestricted Resource Consumption | Missing rate limits, pagination |
| API5 | Broken Function Level Authorization | Missing RBAC, admin exposure |
| API6 | Unrestricted Business Flows | Abuse potential, no flow controls |
| API7 | SSRF | User input in URLs |
| API8 | Security Misconfiguration | CORS, verbose errors |
| API9 | Improper Inventory Management | Deprecated endpoints |
| API10 | Unsafe API Consumption | Unvalidated external responses |

## API1:2023 - Broken Object Level Authorization (BOLA)

**Description:** API endpoints that handle object IDs without proper authorization checks, allowing attackers to access other users' data.

**What to Look For:**
- Direct object references in URLs/parameters without ownership validation
- Missing `userId` checks when accessing resources
- Endpoints that accept resource IDs and return data without verifying the requester owns it
- Sequential or predictable IDs that can be enumerated

**Code Patterns:**
```typescript
// VULNERABLE: No ownership check
app.get('/api/orders/:orderId', async (req, res) => {
  const order = await Order.findById(req.params.orderId);
  res.json(order); // Anyone can access any order!
});

// SECURE: Ownership verification
app.get('/api/orders/:orderId', async (req, res) => {
  const order = await Order.findOne({
    _id: req.params.orderId,
    userId: req.user.id  // Ensures user owns the order
  });
});
```

**Questions to Ask:**
- Does the endpoint verify the authenticated user can access this specific resource?
- Are there authorization checks beyond just authentication?
- Can a user access another user's data by changing an ID?

---

## API2:2023 - Broken Authentication

**Description:** Flawed authentication mechanisms that allow attackers to compromise authentication tokens or exploit implementation flaws.

**What to Look For:**
- Weak password policies
- Credential stuffing vulnerabilities (no rate limiting on login)
- JWT vulnerabilities (weak secrets, algorithm confusion, no expiration)
- Session fixation or management issues
- Exposed credentials in logs, URLs, or responses
- Missing MFA on sensitive operations

**Code Patterns:**
```typescript
// VULNERABLE: No rate limiting, weak error handling
app.post('/login', async (req, res) => {
  const user = await User.findOne({ email: req.body.email });
  if (user && user.password === req.body.password) { // Plain text comparison!
    return res.json({ token: jwt.sign({ id: user.id }, 'weak-secret') });
  }
  res.status(401).json({ error: 'Invalid credentials' });
});
```

**Questions to Ask:**
- Is there rate limiting on authentication endpoints?
- Are passwords properly hashed (bcrypt, argon2)?
- Are JWT secrets strong and properly managed?
- Do tokens expire appropriately?

---

## API3:2023 - Broken Object Property Level Authorization

**Description:** APIs that expose sensitive object properties or allow modification of properties users shouldn't access.

**What to Look For:**
- Mass assignment vulnerabilities (accepting entire objects from user input)
- Sensitive fields in API responses (password hashes, internal IDs, admin flags)
- Users able to modify properties like `isAdmin`, `balance`, `role`

**Code Patterns:**
```typescript
// VULNERABLE: Mass assignment
app.put('/api/users/:id', async (req, res) => {
  await User.findByIdAndUpdate(req.params.id, req.body); // User can set isAdmin: true!
});

// SECURE: Explicit field selection
app.put('/api/users/:id', async (req, res) => {
  const { name, email } = req.body; // Only allow specific fields
  await User.findByIdAndUpdate(req.params.id, { name, email });
});
```

**Questions to Ask:**
- Are API responses filtered to exclude sensitive fields?
- Does the update operation allow modifying unauthorized properties?
- Is user input validated against a schema before database operations?

---

## API4:2023 - Unrestricted Resource Consumption

**Description:** APIs that don't properly limit the size or number of resources that can be requested.

**What to Look For:**
- Missing pagination on list endpoints
- No limits on file upload sizes
- Missing rate limiting
- Expensive operations without throttling
- Unbounded queries (e.g., `find({})` without limits)

**Code Patterns:**
```typescript
// VULNERABLE: No pagination
app.get('/api/transactions', async (req, res) => {
  const transactions = await Transaction.find({ userId: req.user.id }); // Could return millions
  res.json(transactions);
});

// SECURE: Pagination with limits
app.get('/api/transactions', async (req, res) => {
  const limit = Math.min(req.query.limit || 20, 100); // Max 100
  const skip = req.query.skip || 0;
  const transactions = await Transaction.find({ userId: req.user.id })
    .limit(limit)
    .skip(skip);
  res.json(transactions);
});
```

---

## API5:2023 - Broken Function Level Authorization

**Description:** APIs that don't properly restrict access to administrative or privileged functions.

**What to Look For:**
- Missing role checks on admin endpoints
- Regular users accessing admin functions by discovering endpoint URLs
- Inconsistent authorization between different functions
- Authorization only checked on some endpoints in a flow

**Code Patterns:**
```typescript
// VULNERABLE: No role check
app.delete('/api/admin/users/:id', async (req, res) => {
  await User.findByIdAndDelete(req.params.id); // Any authenticated user can delete!
});

// SECURE: Role-based access
app.delete('/api/admin/users/:id', requireRole('admin'), async (req, res) => {
  await User.findByIdAndDelete(req.params.id);
});
```

---

## API6:2023 - Unrestricted Access to Sensitive Business Flows

**Description:** APIs that expose sensitive business flows to abuse without proper safeguards.

**What to Look For:**
- Purchase flows without velocity limits
- Refund endpoints without abuse prevention
- Coupon/referral systems without fraud checks
- Automated abuse of limited-time offers
- Race conditions in financial operations

**Code Patterns:**
```typescript
// VULNERABLE: No abuse prevention
app.post('/api/redeem-coupon', async (req, res) => {
  const coupon = await Coupon.findOne({ code: req.body.code });
  if (coupon && !coupon.used) {
    await applyDiscount(req.user.id, coupon.amount);
    coupon.used = true;
    await coupon.save();
  }
});

// SECURE: Rate limiting + idempotency
app.post('/api/redeem-coupon',
  rateLimit({ windowMs: 60000, max: 5 }),
  async (req, res) => {
    // Use transactions for atomic operations
    const session = await mongoose.startSession();
    session.startTransaction();
    // ... implementation with proper locking
  }
);
```

---

## API7:2023 - Server Side Request Forgery (SSRF)

**Description:** APIs that fetch remote resources based on user-supplied URLs without proper validation.

**What to Look For:**
- User input used to construct URLs for HTTP requests
- Webhook implementations
- Image/file fetching from user-provided URLs
- URL redirect parameters

**Code Patterns:**
```typescript
// VULNERABLE: No URL validation
app.post('/api/fetch-image', async (req, res) => {
  const response = await fetch(req.body.url); // Can access internal services!
  res.send(await response.buffer());
});

// SECURE: URL allowlist validation
app.post('/api/fetch-image', async (req, res) => {
  const url = new URL(req.body.url);
  if (!ALLOWED_HOSTS.includes(url.hostname)) {
    return res.status(400).json({ error: 'Invalid URL' });
  }
  // Also block internal IPs, localhost, etc.
});
```

---

## API8:2023 - Security Misconfiguration

**Description:** Insecure default configurations, missing security headers, verbose error messages, or improper CORS settings.

**What to Look For:**
- CORS allowing all origins (`*`) on sensitive endpoints
- Verbose error messages exposing stack traces or internal details
- Credentials logged in plaintext
- Debug mode enabled in production
- Missing security headers

**Code Patterns:**
```typescript
// VULNERABLE: Verbose errors
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack, // Exposes internal details!
    query: req.query  // Might contain sensitive data
  });
});

// SECURE: Generic errors
app.use((err, req, res, next) => {
  log.error('Request failed', { error: err, requestId: req.id });
  res.status(500).json({ error: 'Internal server error', requestId: req.id });
});
```

---

## API9:2023 - Improper Inventory Management

**Description:** Lack of proper API documentation, versioning, or management of deprecated endpoints.

**What to Look For:**
- Deprecated endpoints still accessible
- Old API versions with known vulnerabilities
- Undocumented endpoints
- Development/testing endpoints in production

**Questions to Ask:**
- Are deprecated endpoints properly removed or restricted?
- Is the new code introducing endpoints that bypass existing security controls?
- Are there test/debug endpoints being added?

---

## API10:2023 - Unsafe Consumption of APIs

**Description:** Trusting data from external APIs without proper validation.

**What to Look For:**
- External API responses used without validation
- Blind trust in third-party data
- Missing error handling for external API failures
- Sensitive data sent to external APIs without necessity

**Code Patterns:**
```typescript
// VULNERABLE: Trusting external data
const providerResponse = await externalPaymentApi.verify(paymentId);
await Order.findByIdAndUpdate(orderId, {
  status: providerResponse.status, // Could be manipulated!
  amount: providerResponse.amount
});

// SECURE: Validate external data
const providerResponse = await externalPaymentApi.verify(paymentId);
if (!isValidPaymentResponse(providerResponse)) {
  throw new Error('Invalid payment response');
}
// Cross-check amount matches our records
const order = await Order.findById(orderId);
if (order.amount !== providerResponse.amount) {
  log.warn('Amount mismatch', { orderId, expected: order.amount, received: providerResponse.amount });
  throw new Error('Payment amount mismatch');
}
```
