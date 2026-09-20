# User Role Controlled by Request Parameter

## Overview

This lab demonstrates a **broken access control** vulnerability where the application uses a client-controlled cookie to determine whether a user has administrator privileges.

By modifying the `Admin` cookie from `false` to `true`, a normal user can gain access to the administrator panel.

## Lab Objective

The objective was to access the administrator panel and delete the user `carlos`.

## Vulnerability

The application relies on the `Admin` cookie to determine whether the current user has administrator privileges.

Because this cookie is controlled by the client, it can be modified without any server-side authorization check.

## Exploitation

### 1. Log in as a normal user

Log in using the provided credentials:

```text
Username: wiener
Password: peter
```

The account initially has normal user privileges.

### 2. Inspect the request

After logging in, the account page uses a cookie containing the user's authorization state.

The relevant request looked like:

```http
GET /my-account?id=wiener HTTP/2
Host: <LAB-HOST>
Cookie: Admin=true; session=[REDACTED]
```

The important part was the `Admin` cookie.

### 3. Modify the Admin cookie

Using the browser's developer tools, the `Admin` cookie was changed to:

```text
Admin=true
```

After refreshing the page, the application treated the user as an administrator.

### 4. Access the administrator panel

After the cookie was modified and the page was refreshed, the administrator panel became accessible.

This demonstrated that the application was trusting a client-controlled cookie to determine the user's role.

### 5. Delete Carlos

From the administrator panel, I deleted the user:

```text
carlos
```

The lab was successfully solved.

## Why It Works

The application makes an authorization decision based on a cookie controlled by the user.

The vulnerable flow is:

```text
Normal user
    ↓
Admin=false
    ↓
Modify cookie
    ↓
Admin=true
    ↓
Application grants admin access
    ↓
Administrator panel
    ↓
Delete Carlos
```

The server should determine a user's privileges using trusted server-side information rather than accepting an authorization value directly from the client.

## Key Takeaways

* Client-controlled cookies should never determine authorization privileges.
* Authentication and authorization are separate security controls.
* A user being logged in does not mean they should be able to choose their own role.
* Authorization decisions must be enforced server-side.
* Burp Suite and browser developer tools can help identify client-controlled authorization mechanisms.

## Tools

* Burp Suite
* Browser Developer Tools
* PortSwigger Web Security Academy

## Vulnerability Classification

**Category:** Access Control

**Vulnerability:** Broken Access Control / Client-Side Role Manipulation

**Impact:** Unauthorized access to administrator functionality

## Disclaimer

This write-up documents testing performed against an intentionally vulnerable PortSwigger Web Security Academy lab for educational purposes.
