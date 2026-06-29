# Secure Code Review Checklists

## General — All Languages

Before diving into language-specific checks, apply these universally:

- [ ] **Authentication**: Is every sensitive endpoint protected? Are tokens validated server-side?
- [ ] **Authorization**: Is authorization checked per request, not just at login?
- [ ] **Input validation**: Is all user input validated, sanitized, and type-checked?
- [ ] **Output encoding**: Is output encoded for the correct context (HTML, SQL, shell)?
- [ ] **Error handling**: Are stack traces / internal details hidden from users?
- [ ] **Secrets**: No hardcoded API keys, passwords, or tokens in source code?
- [ ] **Dependencies**: Are third-party libraries up to date and from trusted sources?
- [ ] **Logging**: Are security events logged without logging sensitive data?
- [ ] **Cryptography**: Are standard libraries used? No custom crypto?
- [ ] **Race conditions**: Are shared resources protected with locks where needed?

---

## Python

### Injection
- [ ] SQL: Use parameterized queries (`cursor.execute("SELECT * FROM t WHERE id=%s", (id,))`) — never f-strings in queries
- [ ] Command injection: Avoid `os.system()`, `subprocess.call(shell=True)` with user input; use `subprocess.run([...])` with list args
- [ ] Template injection: Avoid `render_template_string()` with user data in Jinja2

### Dangerous Functions — Flag These
```python
# DANGEROUS — review carefully
eval(user_input)          # Arbitrary code execution
exec(user_input)          # Arbitrary code execution
__import__(user_input)    # Dynamic import abuse
pickle.loads(user_data)   # Arbitrary deserialization
yaml.load(data)           # Use yaml.safe_load() instead
subprocess.call(cmd, shell=True)  # Shell injection risk
```

### Authentication & Sessions (Django/Flask)
- [ ] Passwords hashed with `bcrypt`, `argon2`, or Django's `make_password` (never MD5/SHA1)
- [ ] Session cookies: `SESSION_COOKIE_SECURE=True`, `SESSION_COOKIE_HTTPONLY=True`, `SESSION_COOKIE_SAMESITE='Lax'`
- [ ] CSRF protection enabled (`django.middleware.csrf`, Flask-WTF)
- [ ] Rate limiting on login endpoints

### File Handling
- [ ] File uploads: Validate MIME type server-side (not just extension); store outside webroot
- [ ] Path traversal: Sanitize filenames with `os.path.basename()`; never concatenate user input to paths

### Dependency Checks
```bash
pip-audit                          # Check for known CVEs
bandit -r ./myapp                  # Static analysis
safety check -r requirements.txt   # Known vulnerabilities
```

---

## JavaScript / TypeScript (Node.js)

### Injection
- [ ] SQL: Use parameterized queries (`db.query('SELECT * FROM t WHERE id = ?', [id])`)
- [ ] NoSQL (MongoDB): Avoid `$where` with user input; validate object shapes
- [ ] Command injection: Avoid `child_process.exec()` with user input; use `execFile()` with array args
- [ ] XSS: Never use `innerHTML`, `document.write()`, or `dangerouslySetInnerHTML` with user data

### Dangerous Patterns — Flag These
```javascript
// DANGEROUS — review carefully
eval(userInput)                        // Arbitrary execution
new Function(userInput)()              // Same as eval
innerHTML = userInput                  // XSS
child_process.exec(`cmd ${userInput}`) // Command injection
require(userInput)                     // Arbitrary module load
JSON.parse(unvalidatedInput)           // Prototype pollution risk
```

### Authentication (Express)
- [ ] JWT: Validate `alg` header; reject `alg: none`; use short expiry
- [ ] Passwords: Use `bcrypt` (`cost ≥ 12`) — never `crypto.createHash('md5')`
- [ ] Sessions: `httpOnly: true`, `secure: true`, `sameSite: 'strict'`
- [ ] Helmet.js configured for security headers

### Prototype Pollution
```javascript
// Vulnerable
function merge(obj, source) {
    for (let key in source) obj[key] = source[key]; // pollutes __proto__
}
// Safe: use structuredClone(), Object.assign({}, ...) with schema validation
```

### Dependency Checks
```bash
npm audit
npx snyk test
npx retire        # Checks for known vulnerable JS libs
```

---

## Java

### Injection
- [ ] SQL: Use `PreparedStatement` — never `Statement` with string concatenation
- [ ] XML: Disable external entity processing (XXE): `factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true)`
- [ ] JNDI: Validate and restrict lookup URLs (Log4Shell vector)
- [ ] Expression Language (EL): Sanitize input used in EL expressions

### Dangerous Patterns — Flag These
```java
// DANGEROUS — review carefully
Runtime.getRuntime().exec(userInput);     // Command injection
new ProcessBuilder(userInput).start();    // Command injection
Class.forName(userInput);                 // Unsafe reflection
ObjectInputStream.readObject();           // Insecure deserialization
(T) ois.readObject();                     // Insecure deserialization
```

### Deserialization
- [ ] Never deserialize untrusted data with native Java serialization
- [ ] Use allowlists with `ValidatingObjectInputStream` (Apache Commons IO)
- [ ] Prefer JSON/XML over Java serialization for data exchange

### Authentication (Spring Security)
- [ ] Passwords: `BCryptPasswordEncoder` with strength ≥ 12
- [ ] CSRF protection enabled (default in Spring Security — don't disable)
- [ ] Method-level security: `@PreAuthorize` on sensitive methods
- [ ] JWT: Validate signature and claims; use `jjwt` or `nimbus-jose-jwt`

### Dependency Checks
```bash
mvn dependency-check:check   # OWASP Dependency Check
./gradlew dependencyCheckAnalyze
```

---

## C# / .NET

### Injection
- [ ] SQL: Use `SqlCommand` with `Parameters.AddWithValue()` — never string interpolation in queries
- [ ] Command injection: Avoid `Process.Start()` with user input; validate and allowlist arguments
- [ ] LDAP injection: Encode special chars with `System.DirectoryServices.Protocols`
- [ ] XSS (Razor): Never use `Html.Raw(userInput)`; Razor auto-encodes by default

### Dangerous Patterns — Flag These
```csharp
// DANGEROUS — review carefully
Process.Start(userInput);                    // Command injection
Assembly.Load(userInput);                    // Arbitrary assembly load
BinaryFormatter.Deserialize(stream);        // Insecure deserialization (deprecated)
new XmlDocument() { XmlResolver = ... };    // XXE if resolver not null
Convert.FromBase64String(userInput) + exec  // Shellcode loading pattern
```

### Deserialization
- [ ] `BinaryFormatter` is deprecated and disabled by default in .NET 5+ — do not re-enable
- [ ] Use `System.Text.Json` or `Newtonsoft.Json` with type handling restrictions
- [ ] Never use `TypeNameHandling.All` in Newtonsoft — prototype pollution / RCE risk

### Authentication (ASP.NET Core)
- [ ] Identity: Use `PasswordHasher<T>` (PBKDF2 by default)
- [ ] Anti-forgery tokens: `[ValidateAntiForgeryToken]` on all POST endpoints
- [ ] Authorization: `[Authorize]` on all sensitive controllers; use policy-based auth
- [ ] Cookie: `HttpOnly`, `Secure`, `SameSite=Strict` in `CookieOptions`
- [ ] JWT: Validate `issuer`, `audience`, `expiry`; use `Microsoft.AspNetCore.Authentication.JwtBearer`

### Dependency Checks
```bash
dotnet list package --vulnerable
dotnet tool install -g dotnet-retire
```

---

## Code Review Severity Guide

| Issue | Severity | Examples |
|-------|----------|----------|
| Direct code execution from user input | Critical | `eval()`, `exec()`, deserialization |
| SQL / Command injection | Critical | Unsanitized input in queries/shells |
| Hardcoded secrets | High | API keys, passwords in source |
| Broken authentication | High | Bypassable auth checks |
| XSS (stored) | High | User data rendered without encoding |
| Insecure deserialization | High | Untrusted object graphs |
| Missing authorization check | High | IDOR, missing `@PreAuthorize` |
| Sensitive data in logs | Medium | Passwords, tokens logged |
| Missing CSRF protection | Medium | State-changing GET requests |
| Weak cryptography | Medium | MD5/SHA1 for passwords |
| Verbose error messages | Low | Stack traces shown to users |
| Missing security headers | Low | No CSP, HSTS, X-Frame-Options |
