# CAS (Central Authentication Service)

`cas.unilim.fr` is a single sign-on protocol for web applications.
Internally, it's a simple [LemonLDAP::NG](https://lemonldap-ng.org/) server.

## Authenticate

Make a single request to `https://cas.unilim.fr/`, dump the HTML.

You have to read the HTML to query an hidden input containing
an CSRF token, `input#token`.


Once you have the CSRF token, you can use it to authenticate.

```
POST https://cas.unilim.fr/
Content-Type: application/x-www-form-urlencoded
```

| Key        | Value | Description |
| ---------- | ----- | ----------- |
<!-- | `url`      | (empty) | The URL to redirect to after authentication, useless here. | -->
<!-- | `skin`     | `unilim` | Skin used for the login page, useless but probably required. | -->
| `token`    | `1743557998_38739` | The CSRF token you found in the HTML earlier. |
| `user`     | (your unilim username) | Your username. |
| `password` | (your unilim password) | Your password. |

> Remember to URL encode the values since we're sending `application/x-www-form-urlencoded`.

If anything goes wrong, make sure to [check the error codes](#handle-errors).

If everything goes well, you should receive a **200 OK** response and you should check if you have to [handle 2FA](#handle-2fa) or [handle login response](#handle-login-response) if not required.

## Handle 2FA

> **Since February 2025, 2FA is mandatory for all users.** You will need to use a TOTP application (like Google Authenticator) to generate the 2FA code or use the email method.

When you authenticate, the server will return a **200 OK** response with an HTML page containing the 2FA form.

You can verify if we're in the 2FA page by looking for the `.message[trspan=choose2f]` tag.

You have to know which method you want to use to authenticate, either by email or by TOTP.
It all depends on the user settings.

### TOTP

Check if you can find the `button[name=sf][value=totp]` tag in HTML page.

If you can find it, you can use the TOTP method.
Otherwise it's not available.

### Email

Check if you can find the `button[name=sf][value=mail]` tag in HTML page.

If you can find it, you can use the email method.
Otherwise it's not available.

### Send the 2FA code to method of choice

You have to read the HTML to query an hidden input containing
an CSRF token, `input#token`.

```
POST https://cas.unilim.fr/2fchoice
Content-Type: application/x-www-form-urlencoded
```

| Key             | Value | Description |
| --------------- | ----- | ----------- |
| `token`         | `1743558786_11984` | The CSRF token you found in the HTML earlier. |
| `sf`            | `totp` or `mail` | The method you want to use. |
<!-- | `skin`          | `unilim` | Skin used for the login page, useless but probably required. |
| `checkLogins`   | (empty) | Pretty useless to us. |
| `stayconnected` | (empty) | Not sure how this should be implemented. | -->

If anything goes wrong, make sure to [check the error codes](#handle-errors).

### Verify the 2FA code

Before anything, you have to read the HTML to query an hidden input containing
an CSRF token, `input#token`.

### TOTP


### Email

```
POST https://cas.unilim.fr/mail2fcheck?skin=unilim
Content-Type: application/x-www-form-urlencoded
```

| Key             | Value | Description |
| --------------- | ----- | ----------- |
| `token`         | `1743559821_49787` | The CSRF token you found in the HTML earlier. |
| `code`          | (your 2FA code) | The 2FA code you received by email. |

If it goes well, you should receive a **302 Found** response and you can
move on directly to the [handle login response](#handle-login-response) section.
**Do not follow the redirections**, you won't be able to save the `lemonldap` cookie
and will be redirected to the CAS home page.

If anything goes wrong, make sure to [check the error codes](#handle-errors).

## Handle login response

When you're successfully logged in, either with credentials or with 2FA,
the server will return an `lemonldap` cookie inside the `Set-Cookie` headers.

This cookie is your session cookie and you have to use it for all your requests.
Once you have it, you can request any page on the `cas.unilim.fr` domain.

## Handle errors

When an error occurs, the server will return an HTML page containing the error message.

You can find the error code in the HTML page by querying the `span[trmsg]` tag,
the error code is the number contained in the `trmsg` attribute.

| Error code | Description |
| ---------- | ----------- |
| `2`        | User and password fields must be filled. |
| `5`        | Wrong credentials. |
| `81`       | Invalid authentication attempt. |
| `82`       | Exceeded authentication timeout. |
| `86`       | Your account is locked, you have to wait. |
| `96`       | Invalid verification code. |
