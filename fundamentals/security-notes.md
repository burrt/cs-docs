# Security Notes

* [Common FAQs](security-notes.md#common-faqs)
* [One time passwords](security-notes.md#one-time-passwords)

## Resources

* [OWASP - Top 10](https://owasp.org/www-project-top-ten/)
* [sans - Agile development and building secure software](https://www.sans.org/blog/agile-development-teams-can-build-secure-software/)
* [Attack Trees - security modelling](https://www.infoq.com/presentations/security-modeling-agile-teams/)

## Common FAQs

### Hashing vs Encryption

Taken from [OWASP - Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html):

> Hashing and encryption both provide ways to keep sensitive data safe. However, in almost all circumstances, passwords should be hashed, **NOT** encrypted.
>
> **Hashing is a one-way function** (i.e., it is impossible to "decrypt" a hash and obtain the original plaintext value). Hashing is appropriate for password validation. Even if an attacker obtains the hashed password, they cannot enter it into an application's password field and log in as the victim.
>
> **Encryption is a two-way function**, meaning that the original plaintext can be retrieved. Encryption is appropriate for storing data such as a user's address since this data is displayed in plaintext on the user's profile. Hashing their address would result in a garbled mess.

### What hashing algorithm should I use for passwords?

See [OWASP - Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)

### How to test for weak cryptography?

See [OWASP - Testing for weak cryptography](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/README)

### Very quick summary of OAuth 2.0

1. Initiate Sign-Up: User clicks "Sign Up" on the client application.
2. Redirect to **Authorization Server**: Client sends user to authorization server for authentication and authorization.
3. User Authenticates and Authorizes: User logs in and grants permissions.
4. Receive **Authorization Code**: User is redirected back to client with an authorization code.
5. Exchange Authorization Code for Token: Client requests an **access token** from the authorization server.
6. Receive and Use Access Token: Client receives the access token and uses it to access protected resources.

The above would be the an example of the Authorisation Code grant. The benefits of OAuth over say API keys is that access tokens can have variable (and generally shorter) lifetimes, fine-grained permissions and the jwt can contain other meta data!

Very useful blogs:

* [Stack Overflow blog 1](https://stackoverflow.blog/2022/12/22/the-complete-guide-to-protecting-your-apis-with-oauth2/)
* [Stack Overflow blog 2](https://stackoverflow.blog/2022/04/14/the-authorization-code-grant-in-excruciating-detail/)

### Recommended security sources/news

* OWASP mailing lists and blogs - preferred, my main point of reference
* Microsoft Security Blog
* Reddit subreddits like `/netsec`, `/cybersecurity`
* Tend to prefer static code analysis/automatic detection like SonarQube, CodeQL, Dependabot, Snyk

## One time passwords

### TOTP

> Time-based One-time Password Algorithm is an algorithm that computes a one-time password from a shared secret key and the current time. [Wikipedia](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm)

Also see [RFC 6238](https://tools.ietf.org/html/rfc6238)

#### TOTP Definition

TOTP is based on HOTP with a timestamp replacing the incrementing counter.

* current timestamp is turned into an integer time-counter (TC)
* start of an epoch (T0)
* counting in units of a time interval (TI).

For example:

```
TC = floor((unixtime(now) − unixtime(T0)) / TI)
TOTP = HOTP(SecretKey, TC)
TOTP-Value = TOTP mod 10d  // where d is the desired number of digits of the one-time password.
```

#### Implementation

* generate a key, `K`, which is an arbitrary byte string, and share it securely with the client.
* agree upon a `T0`, the Unix time to start counting time steps from (default Unix epoch)
* agree on an interval, `TI`, which will be used to calculate the value of the counter `C` (default 30 seconds)
* agree upon a cryptographic hash method (default is `SHA-1`)
  * TOTP implementations may use `HMAC-SHA-256` or `HMAC-SHA-512` functions based on `SHA-256` or `SHA-512` `SHA2` hash functions - [RFC 6238](https://tools.ietf.org/html/rfc6238#section-1.2))
* agree upon a token length, `N` (default is 6)

Although `RFC 6238` allows different parameters to be used, the Google implementation of the authenticator app does **not** support `T0`, `TI` values, hash methods and token lengths different from the default. It also expects the K secret key to be entered (or supplied in a QR code) in base-32 encoding according to `RFC 3548`.

Once the parameters are agreed upon, **token generation** is as follows:

* calculate `C` as the number of times `TI` has elapsed after `T0`.
* compute the HMAC hash `H` with `C` as the _message_ and `K` as the key.
* `K` should be passed as it is, `C` should be passed as a _raw 64-bit unsigned integer_.
* take the least 4 significant bits of `H` and use it as an offset, `O`.
* take 4 bytes from `H` starting at O bytes MSB, **discard** the most significant bit and store the rest as an (unsigned) 32-bit integer, `I`.
* the token is the lowest `N` digits of `I` in base 10.
* if the result has fewer digits than `N`, pad it with zeroes from the left.

Now:

* both the server and the client compute the token
* then the server checks if the token supplied by the client matches the locally generated token
* some servers allow codes that should have been generated **before or after the current time** in order to account for slight _clock skews, network latency and user delays_

### HTOP

HOTP is an HMAC-based one-time password (OTP) algorithm. See [Wikipedia](https://en.wikipedia.org/wiki/HMAC-based_One-time_Password_Algorithm#Definition).

#### HTOP Definition

* `K` be a secret key
* `C` be a counter
* `HMAC(K, m) = SHA1(K ⊕ 0x5c5c… ∥ SHA1(K ⊕ 0x3636… ∥ m))`
* `m` is a "message"
* `⊕` means `XOR`
* `∥` means concatenation
* `Truncate` be a function that selects 4 bytes from the result of the HMAC in a defined manner

Then `HOTP(K, C)` is mathematically defined by:

```
HOTP(K, C) = Truncate(HMAC(K, C)) & 0x7FFFFFFF
```

The mask `0x7FFFFFFF` sets the result's MSB to zero. This avoids problems if the result is interpreted as a signed number as some processors do.

_For HOTP to be useful for an individual to input to a system, the result must be converted into a `HOTP` value, a 6–8 digits number that is implementation dependent._

```
HOTP-Value = HOTP(K, C) mod 10d  // where d is the desired number of digits
```
