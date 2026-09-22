# User ID Controlled by Request Parameter with Password Disclosure

## Overview

This lab demonstrates a **broken access control** vulnerability where a user-controlled ID in the URL determines which account page is displayed.

The account page also contains the user's existing password in the HTML, even though the password is visually masked in the browser.

By changing the user ID to `administrator`, the administrator's account page can be accessed and the exposed password can be recovered.

## Lab Objective

The objective was to retrieve the administrator's password and use it to log in as the administrator, then delete the user `carlos`.

The provided credentials were:

```text
Username: wiener
Password: peter
```

## Vulnerability

The application uses a user-controlled request parameter to determine which account page to display.

The normal account URL was:

```text
/my-account?id=wiener
```

The application did not properly verify whether the logged-in user was authorized to access the requested account.

By changing the `id` parameter to another username, it was possible to access that user's account page.

This is an example of **broken access control**, commonly associated with **IDOR (Insecure Direct Object Reference)**.

## Exploitation

### 1. Log in as Wiener

Log in to the application using the provided credentials:

```text
Username: wiener
Password: peter
```

After logging in, the account page was accessible through a URL similar to:

```text
/my-account?id=wiener
```

### 2. Modify the User ID

The `id` parameter is controlled by the user.

The original URL:

```text
/my-account?id=wiener
```

was changed to:

```text
/my-account?id=administrator
```

The application then displayed the administrator's account page.

This demonstrated that the application was not properly checking whether the current user was authorized to access the requested account.

### 3. Inspect the Administrator's HTML

The administrator's password appeared to be masked in the browser.

However, inspecting the page's HTML revealed that the password was still present in the underlying page content.

For example, a password field may look visually like:

```html
<input type="password" value="administrator-password">
```

Although the browser displays the value as masked characters, the actual value is still present in the HTML.

The administrator's password was therefore disclosed through the account page.

> **Note:** The actual password is intentionally not included in this write-up.

### 4. Log in as Administrator

The disclosed administrator password was then used to log in to the administrator account.

This provided access to the administrator functionality.

### 5. Delete Carlos

From the administrator panel, the user `carlos` was deleted.

The lab was successfully solved.

## Attack Flow

```text
Login as Wiener
       ↓
/my-account?id=wiener
       ↓
Change ID to administrator
       ↓
Access administrator account page
       ↓
Inspect HTML
       ↓
Administrator password disclosed
       ↓
Login as administrator
       ↓
Delete Carlos
```

## Why It Works

The application makes an authorization decision based on a user-controlled `id` parameter.

Instead of verifying:

```text
"Is Wiener authorized to access the administrator account?"
```

the application effectively trusts the requested identifier.

The account page also exposes the existing password in the HTML. Password masking only changes how the browser displays the value. It does not protect the value if the password is already included in the HTML sent to the browser.

This turns the access-control issue into a more serious sensitive information disclosure.

## Key Takeaways

* User-controlled IDs should not automatically grant access to another user's resources.
* Authentication and authorization are separate security controls.
* IDOR vulnerabilities can expose sensitive account information.
* Password fields being visually masked does not mean the password is secure if it is present in the HTML.
* Sensitive information should never be unnecessarily included in client-side HTML.
* Authorization checks must be enforced server-side.

## Tools

* Web Browser
* Browser Developer Tools
* PortSwigger Web Security Academy

## Vulnerability Classification

**Category:** Access Control

**Primary Vulnerability:** Broken Access Control / IDOR

**Additional Issue:** Password Disclosure

**Impact:** Unauthorized access to another user's account information and exposure of administrator credentials.

## Screenshots

The following screenshots document the exploitation process:

```text
screenshots/
├── 01-modified-user-id.png
├── 02-admin-password-in-html.png
├── 03-admin-panel.png
└── 04-delete-carlos.png
```

Sensitive credentials and session information have been redacted from screenshots.

## Disclaimer

This write-up documents testing performed against an intentionally vulnerable PortSwigger Web Security Academy lab for educational purposes.
