# OpenID Connect or OAuth2 Specifications Using Keycloak as an IDP

OpenID Connect (OIDC) and OAuth 2.0 are both protocols used for authentication and authorization, respectively. While they are related and often used together, they serve different purposes and have distinct flows. Here's a comparison of the two:

## OAuth 2.0

**Purpose:** OAuth 2.0 is primarily used for authorization. It allows a user to grant a third-party application limited access to their resources without exposing their credentials.

**Flows:**

- **Authorization Code Grant:**
  - Used by web and mobile applications.
  - Involves exchanging an authorization code for an access token.
  - More secure as the client secret is not exposed.

- **Implicit Grant:**
  - Used by single-page applications (SPAs) and mobile apps.
  - Access token is returned directly in the URL fragment.
  - Less secure as the token is exposed in the URL.

- **Resource Owner Password Credentials Grant:**
  - Used when the user trusts the client application.
  - User provides their username and password directly to the client.
  - Less secure and generally not recommended.

- **Client Credentials Grant:**
  - Used for machine-to-machine communication.
  - No user involvement; the client authenticates itself to get an access token.

**Tokens:**

- **Access Token:** Used to access protected resources.
- **Refresh Token:** Used to obtain a new access token without re-authenticating the user.

## OpenID Connect (OIDC)

**Purpose:** OIDC is an identity layer built on top of OAuth 2.0. It is used for authentication, allowing clients to verify the identity of the user and obtain basic profile information.

**Flows:**

- **Authorization Code Flow:**
  - Similar to OAuth 2.0's Authorization Code Grant.
  - Involves exchanging an authorization code for an ID token and access token.
  - More secure as the client secret is not exposed.

- **Implicit Flow:**
  - Similar to OAuth 2.0's Implicit Grant.
  - ID token and access token are returned directly in the URL fragment.
  - Less secure as the tokens are exposed in the URL.

- **Hybrid Flow:**
  - Combines elements of both Authorization Code and Implicit flows.
  - Allows the client to receive some tokens directly from the authorization endpoint and others from the token endpoint.

**Tokens:**

- **ID Token:** A JSON Web Token (JWT) that contains user identity information.
- **Access Token:** Used to access protected resources.
- **Refresh Token:** Used to obtain new tokens.

## Key Differences

**Purpose:**

- **OAuth 2.0:** Authorization.
- **OIDC:** Authentication and identity verification.

**Tokens:**

- **OAuth 2.0:** Access Token, Refresh Token.
- **OIDC:** ID Token, Access Token, Refresh Token.

**Flows:**

- **OAuth 2.0:** Authorization Code, Implicit, Resource Owner Password Credentials, Client Credentials.
- **OIDC:** Authorization Code, Implicit, Hybrid.

**Use Cases:**

- **OAuth 2.0:** Granting third-party applications access to user resources.
- **OIDC:** Logging in users and obtaining user profile information.

## Conclusion

While OAuth 2.0 and OIDC are related, they serve different purposes. OAuth 2.0 is used for authorization, allowing applications to access resources on behalf of a user. OIDC, on the other hand, is used for authentication, allowing applications to verify the identity of a user. They can be used together to provide both authentication and authorization in a secure and standardized manner.

---

# My Notes

## Authentication Flows:

### Authorization Code Flow - Keycloak (Standard Flow)

This enables standard OpenID Connect redirect-based authentication with authorization code.

**My understanding:** When the user, with their username/password, and the backend service, with client_secret, client_id, scope, and grant_type, both have to get verified to get the token.

### Resource Owner Password Credentials Grant - Keycloak (Direct Access Grants)

This enables support for Direct Access Grants, which means that the client has access to the username/password of the user and exchanges it directly with the Keycloak server for an access token.

**My understanding:** When the user, with their username/password, has to get verified to get the token.

### Client Credentials Grant - Keycloak (Service Accounts Roles)

Allows you to authenticate this client to Keycloak and retrieve an access token dedicated to this client.

**My understanding:** When the backend service, with client_secret, client_id, scope, and grant_type, has to get verified to get the token.

### Implicit Flow - Keycloak (Implicit Flow)

This enables support for OpenID Connect redirect-based authentication without authorization code.
**My understanding:** This flow is designed for applications where the client secret cannot be securely stored, such as single-page applications (SPAs). In this flow, tokens are returned directly via the redirect URI after a successful authentication, without an intermediate code exchange step. However, it's important to note that the Implicit flow is considered less secure than the Authorization Code flow with PKCE and has been deprecated in OAuth 2.1 in favor of more secure alternatives. If possible, you should use the Authorization Code flow with PKCE for public clients like SPAs.

### Device Authorization Grant - Keycloak (OAuth 2.0 Device Authorization)

This means that the client is an application on a device that has limited input capabilities or lacks a suitable browser.

**My understanding:** In simple terms, when the device is a command-line interface and it is an external device and not a backend service. Also, if the external device is not trusted to have username/password, then the device can only request a code and verification URI. The user takes the code and adds it to the verification URI with their username/password while the device keeps requesting the token. Upon success (within the code expiry time), the device gets the token.

- The external device requests Keycloak for a device_code without needing any client_secret or username/password.
- The device will get the device_code, and the user will take it on a secondary device and navigate to the verification_uri (or verification_uri_complete) to enter the user_code.
- The user will be prompted to log in to Keycloak (if not already logged in) and authorize the device.
- While this is being done, the external device keeps polling the token endpoint until it receives an access token or an error response indicating that the user did not authorize the device within the allowed time frame.
- Once the user completes the authorization process, the token endpoint will respond with an access token that the device can use to make authenticated requests on behalf of the user.

### OIDC CIBA Grant - Keycloak (OIDC CIBA Grant)

This means that the user is authenticated via some external authentication device instead of the user's browser.

**My understanding:** 

Example:

### Authorization Code Flow

To obtain a token from Keycloak using the Authorization Code flow with curl, you need to follow these steps:

1. **Register a client in Keycloak:** Make sure your client is set up in Keycloak with the Authorization Code flow enabled.

2. **Get the Authorization Code:**
   - Direct the user to the Keycloak login page with the appropriate query parameters. The user will log in and Keycloak will redirect to the redirect URI you provided with an authorization code.
   - Example URL to get the authorization code (you should open this in a browser, not with curl):

     ```
     http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/auth?client_id={client-id}&redirect_uri={redirect-uri}&response_type=code&scope=openid
     ```

     Replace `{realm-name}`, `{client-id}`, and `{redirect-uri}` with your actual realm name, client ID, and redirect URI.

3. **Exchange the Authorization Code for a Token:**
   - Once you have the authorization code, you can exchange it for an access token using curl. Here's an example of how to do this:

     ```sh
     curl -X POST \
     -d "client_id={client-id}" \
     -d "client_secret={client-secret}" \
     -d "grant_type=authorization_code" \
     -d "code={authorization-code}" \
     -d "redirect_uri={redirect-uri}" \
     http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/token
     ```

     Replace `{client-id}`, `{client-secret}`, `{authorization-code}`, `{redirect-uri}`, and `{realm-name}` with your actual client ID, client secret, the authorization code you received, the redirect URI, and the realm name.

Here's a breakdown of the parameters:

- `client_id`: The client ID for your application.
- `client_secret`: The client secret for your application (if it is a confidential client).
- `grant_type`: This must be `authorization_code` for the Authorization Code flow.
- `code`: The authorization code you received from Keycloak after the user logged in.
- `redirect_uri`: The same redirect URI that was used to obtain the authorization code.

The response from Keycloak will include an access token, which you can use to authenticate API requests on behalf of the user.

Please note that the CIBA flow is an advanced feature and may require additional configuration in Keycloak, such as setting up authentication policies and configuring the client for CIBA support. Additionally, the exact endpoints and parameters may vary depending on the Keycloak version and any customizations that have been applied. Always refer to the Keycloak documentation and your specific Keycloak configuration for the most accurate information.
