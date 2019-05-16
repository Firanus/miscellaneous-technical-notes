# Authentication and Authorization 101

The first is the process of identifying and validating a user. The second is the process of granting them access.

## Man in the middle attacks (MITM)

Man in the middle attacks depend on someone intercepting a request and using the items contained in it to pose as you. They are a thing because HTTP is stateless, so your identity needs to be validated on each request. To control these, use HTTPS, which provides transport level security between client and server.

# State Management

There are two main ways to manage state:

* Sessions - These are standard. User is given a session key. This needs to be passed and validated with every request, which requires a lookup, and can be time consuming. Session keys are regarded as *opaque*: they contain no useful data of their own.

* JWTs - industry standard for representing secure claims between two parties. Tokens have 3 parts, all encoded with some security algorithm. The tokens themselves are signed, stateless, they can verify the integrity of the claims contained in the tokens themselves, but they are *not opaque*; don't store sensitive information here!

# Web Authentication Methods

There are a variety of methods, but most require the use of HMAC:

> **HMAC** - Hash Based Message Authentication Codes. These are a group of algorithms used to sign something using a secret. They require a cryptographic hash function. In general they work by applying the algorithm to a plain text and secrets.

There are 3 main types of web authentication method:
* SSO (Single Sign On) - Normally, you use it when you want to log onto a service using credentials provided by some 3rd party (e.g. login to Reddit using gmail credentials). There are a number of ways to do this. The most common is SAML, Security Assertion Markup Language
* API keys - Secure service to service interactions. Can be generated independent of user credentials, and can easily be reissued and rotated. Think of the keys service providers give you when you're trying out their services.
* HTTP Authentication - Securing client access to an API. In this case, you put a token or secret in the Authorization header of an HTTP call (though different variants might use, e.g. a different header). Variants include (but are not limited to):
  * **Basic** - Just a username and password in a header. Needs HTTPS
  * **Digest** - Communicates enrypted credentials. Client asks to authenticate. The server returns a single use nonce and realm. The realm is encrypted with username, password, method and URI, and the result is returned to the server with the nonce. Also prone to MITM, needs HTTTPS. Slower, but prevents attacker learning usernames and passwords
  * **Bearer** - Think of this as "give access to the bearer of this token". Usually, tokens themselves are opaque. The token needs to be stored on the server after authentication to manage the user's state. In addition, token lookup is performed on each request.
  * **OAuth 1.0a** - Built around the core concept of delegated authorization, and can use a 2-legged or 3-legged flow. OAuth doesn't use usernames and passwords. Instead, credentials are split into **client credentials** and **user tokens**. There are 3 roles, user, consumer and service provider. This is a complex, form of web session authentication, and is currently viewed as deprecated. It has largely been superseded by OAuth 2, which is too big a topic to cover in this note alone.