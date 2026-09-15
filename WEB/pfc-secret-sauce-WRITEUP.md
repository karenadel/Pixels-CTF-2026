# PFC — The Secret Sauce — Walkthrough

**Category:** Web

## TL;DR

There was a typical SQL injection-based login bypass exploit that got us logged into the site as an employee. Then, there was the IDOR in the profile pages – we simply changed the `id` parameter in the URL until we found the admin profile, and it was protected by a password-protected ZIP file. We cracked the password using a dictionary-based attack on the ZIP file and the password turned out to be a common word from rockyou.txt.

## Recon

We tried accessing `http://pixelsctf.local:3700/pfc/login` a simple "Employee ID + Password" login form for "PFC" (Pixels Fried Chicken). There were no credentials provided, nor any account to try.
 
## Step 1 — Login bypass
 
We performed a usual SQL injection login bypass exploit:
 
```
Employee ID: ' OR '1'='1' --
Password:    anything
```

That gave us access immediately as an Employee.

## Step 2 — Locating the IDOR

In the dashboard, we spotted the "My Profile" link that led to `/pfc/profile?id=<some number>`. As the `id` was exposed in the URL, we tried modifying the parameter and refreshed the page to check how the application reacts. Nothing in the page prevented us from viewing profiles of other employees as it just showed any ID we provided.

By lowering the number manually one by one, we reached `id=1` which turned out to be the profile of "Colonel," who is Head of Operations or an admin. The profile included the following note about the Secret Sauce archive stored in `/kitchen/secret-sauce`: "This archive contains the secret sauce recipe. Do not share!".

## Step 3 — Stealing the archive

We accessed `/kitchen/secret-sauce` and found there a zip file that can be downloaded without logging in.

## Step 4 – Cracking the zip

Attempted using fcrackzip to crack the zip file, rather than guessing the password myself. Provided it the rockyou.txt list  the large list of passwords that has been leaked, and is included in most Linux security tools and it was able to crack it instantly, revealing it to be a common word.

## Step 5 – Reading the flag

Extracted the zip file using the password and was then able to read the file containing the flag.
 
## Root cause

- The login page constructed the SQL query using our input directly in the query itself, and so `' OR '1'='1' -- ` turned the query into one which returned every row, logging us in.
- The profile page ensured that the user had logged in, but did not ensure that the profile they requested belonged to the logged-in user and hence could simply change the ID parameter in the URL and view any account's details, including the administrator's.
- The “protected” zip file was only protected to the extent of its password, which was common enough to appear in a publicly available wordlist.