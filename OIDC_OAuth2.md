Title: OpenID Connect or OAuth2 specificiations using Keycloak as an IDP
OpenID Connect (OIDC) and OAuth 2.0 are both protocols used for authentication and authorization, respectively. While they are related and often used together, they serve different purposes and have distinct flows. Here's a comparison of the two:

OAuth 2.0
Purpose:
	OAuth 2.0 is primarily used for authorization. It allows a user to grant a third-party application limited access to their resources without exposing their credentials.

Flows:
	Authorization Code Grant:
	Used by web and mobile applications.
	Involves exchanging an authorization code for an access token.
	More secure as the client secret is not exposed.
	
	Implicit Grant:
	Used by single-page applications (SPAs) and mobile apps.
	Access token is returned directly in the URL fragment.
	Less secure as the token is exposed in the URL.
	
	Resource Owner Password Credentials Grant:
	Used when the user trusts the client application.
	User provides their username and password directly to the client.
	Less secure and generally not recommended.
	
	Client Credentials Grant:
	Used for machine-to-machine communication.
	No user involvement; the client authenticates itself to get an access token.

Tokens:
	Access Token: Used to access protected resources.
	Refresh Token: Used to obtain a new access token without re-authenticating the user.

OpenID Connect (OIDC)
Purpose:
	OIDC is an identity layer built on top of OAuth 2.0. It is used for authentication, allowing clients to verify the identity of the user and obtain basic profile information.

Flows:
	Authorization Code Flow:
	Similar to OAuth 2.0's Authorization Code Grant.
	Involves exchanging an authorization code for an ID token and access token.
	More secure as the client secret is not exposed.

	Implicit Flow:
	Similar to OAuth 2.0's Implicit Grant.
	ID token and access token are returned directly in the URL fragment.
	Less secure as the tokens are exposed in the URL.

	Hybrid Flow:
	Combines elements of both Authorization Code and Implicit flows.
	Allows the client to receive some tokens directly from the authorization endpoint and others from the token endpoint.

Tokens:
	ID Token: A JSON Web Token (JWT) that contains user identity information.
	Access Token: Used to access protected resources.
	Refresh Token: Used to obtain new tokens.

Key Differences
	Purpose:
	OAuth 2.0: Authorization.
	OIDC: Authentication and identity verification.

	Tokens:
	OAuth 2.0: Access Token, Refresh Token.
	OIDC: ID Token, Access Token, Refresh Token.
	
	Flows:
	OAuth 2.0: Authorization Code, Implicit, Resource Owner Password Credentials, Client Credentials.
	OIDC: Authorization Code, Implicit, Hybrid.
	
	Use Cases:
	OAuth 2.0: Granting third-party applications access to user resources.
	OIDC: Logging in users and obtaining user profile information.

Conclusion
While OAuth 2.0 and OIDC are related, they serve different purposes. OAuth 2.0 is used for authorization, allowing applications to access resources on behalf of a user. OIDC, on the other hand, is used for authentication, allowing applications to verify the identity of a user. They can be used together to provide both authentication and authorization in a secure and standardized manner.

############################################# MY NOTES #############################################
Authentication flows: 
	Authorization Code Flow -
		Keycloak (Standard Flow): This enables standard OpenID Connect redirect based authentication with authorization code.
		My understanding: When user, with their username/pwd, and BE service, with client_secret client_id scope grant_type scope, both have to get verified to get the token

	Resource Owner Password Credentials Grant -
		Keycloak(Direct access grants): This enables support for Direct Access Grants, which means that client has access to username/pwd of user and exchange it directly with Keycloak server for access token.
		My understanding: When user, with their username/pwd, have to get verified to get the token
	
	Client Credentials Grant -
		Keycloak(Service accounts roles): Allows you to authenticate this client to Keycloak and retrieve access token dedicated to this client.
		My understanding: When BE service, with client_secret client_id scope grant_type scope, have to get verified to get the token
	
	Implicit Flow -
		Keycloak(Implicit Flow): This enables support for OpenID Connect redirect based authentication without authorization code.
		My understanding: The is designed for applications where the client secret cannot be securely stored, such as single-page applications (SPAs). In this flow, tokens are returned directly via the redirect URI after a successful authentication, without an intermediate code exchange step. However, it's important to note that the Implicit flow is considered less secure than the Authorization Code flow with PKCE and has been deprecated in OAuth 2.1 in favor of more secure alternatives. If possible, you should use the Authorization Code flow with PKCE for public clients like SPAs.
	
	Device Authorization Grant
		Keycloak(OAuth 2.0 Device Authorization): This means that client is an application on device that has limited input capabilities or lack a suitable browser.
		My understanding: In simple terms, when the device is command line interface and it is an external device and not a BE service. Also, if the external device is not trusted to have username/pwd then device can only request for a code and verification_uri and then the user takes the code and adds the code to the verification_uri with the username/pwd :D while the device keeps requesting for the token and upon success (within the code expiry time) device get the token :D
			The external device requests to keyclaok for a device_code and you don't need any client_secret or username/pwd. Now the device will get the device_code and the user will take it on a secondary device and navigate to the verification_uri (or verification_uri_complete) to enter the user_code. The user will be prompted to log in to Keycloak (if not already logged in) and authorize the device. And while this is being done the external device keeps polling he token endpoint until it receives an access token or an error response indicating that the user did not authorize the device within the allowed time frame. Once the user completes the authorization process, the token endpoint will respond with an access token that the device can use to make authenticated requests on behalf of the user.
	
	OIDC CIBA Grant
		Keycloak(OIDC CIBA Grant): This means that the user is authenticated via some external authentication device instead of the user's browser.
		My understanding: 
	

Example:
1) Authorization Code Flow: 
	To obtain a token from Keycloak using the Authorization Code flow with curl, you need to follow these steps:

			Register a client in Keycloak: Make sure your client is set up in Keycloak with the Authorization Code flow enabled.

			Get the Authorization Code:
					Direct the user to the Keycloak login page with the appropriate query parameters. The user will log in and Keycloak will redirect to the redirect URI you provided with an authorization code.
					Example URL to get the authorization code (you should open this in a browser, not with curl):

					http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/auth?client_id={client-id}&redirect_uri={redirect-uri}&response_type=code&scope=openid

					Replace {realm-name}, {client-id}, and {redirect-uri} with your actual realm name, client ID, and redirect URI.

			Exchange the Authorization Code for a Token:
					Once you have the authorization code, you can exchange it for an access token using curl. Here's an example of how to do this:

	curl -X POST \
		-d "client_id={client-id}" \
		-d "client_secret={client-secret}" \
		-d "grant_type=authorization_code" \
		-d "code={authorization-code}" \
		-d "redirect_uri={redirect-uri}" \
		http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/token

			Replace {client-id}, {client-secret}, {authorization-code}, {redirect-uri}, and {realm-name} with your actual client ID, client secret, the authorization code you received, the redirect URI, and the realm name.

	Here's a breakdown of the parameters:

			client_id: The client ID for your application.
			client_secret: The client secret for your application (if it is a confidential client).
			grant_type: This must be authorization_code for the Authorization Code flow.
			code: The authorization code you received from Keycloak after the user logged in.
			redirect_uri: The same redirect URI that was used to obtain the authorization code.

	The response from Keycloak will include an access token, which you can use to authenticate API requests on behalf of the user.

	Please note that the Authorization Code flow is intended for server-side applications because it involves client secrets. If you're working with a public client (like a single-page application), you should use the Implicit flow or PKCE (Proof Key for Code Exchange) instead, as they do not expose client secrets.

2) Resource Owner Password Credentials Grant
	The Resource Owner Password Credentials Grant (also known as the "password" grant) allows a client to obtain an access token directly by providing the resource owner's username and password. This grant type is suitable for trusted clients where the resource owner has a trust relationship with the client, such as the device operating system or a highly privileged application.

	Here's how you can use curl to obtain a token from Keycloak using the Resource Owner Password Credentials Grant:

	curl -X POST \
		-d "client_id={client-id}" \
		-d "client_secret={client-secret}" \
		-d "username={username}" \
		-d "password={password}" \
		-d "grant_type=password" \
		http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/token

	Replace {client-id}, {client-secret}, {username}, {password}, and {realm-name} with your actual client ID, client secret, the resource owner's username, their password, and the realm name.

	Here's a breakdown of the parameters:

			client_id: The client ID for your application.
			client_secret: The client secret for your application (if it is a confidential client).
			username: The resource owner's username.
			password: The resource owner's password.
			grant_type: This must be password for the Resource Owner Password Credentials Grant.

	The response from Keycloak will include an access token, which you can use to authenticate API requests on behalf of the user.

	Please note that the Resource Owner Password Credentials Grant is not recommended for use in new applications. It is considered a less secure approach because it requires the application to handle the user's credentials directly. Instead, it is recommended to use more secure flows like the Authorization Code flow with PKCE, especially for applications that are available to third parties.

	Additionally, the use of this grant type must be enabled in Keycloak for the client. You can enable it by editing the client settings in the Keycloak admin console and setting "Direct Access Grants Enabled" to "ON".

3) Client Credentials Grant
	To obtain a token from Keycloak using the "Client Credentials Grant" flow, you need to have a client set up in Keycloak with "Access Type" configured as "confidential" and "Service Accounts Enabled". You also need to ensure that the client has the "client_credentials" grant type allowed.

	Here's an example of a curl command to get a token from Keycloak using the "Client Credentials Grant":

	curl -X POST \
		'http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/token' \
		-H 'Content-Type: application/x-www-form-urlencoded' \
		-d 'client_id={client-id}' \
		-d 'client_secret={client-secret}' \
		-d 'grant_type=client_credentials'
	Replace the following placeholders with your actual data:

	{keycloak-server}: The base URL of your Keycloak server.
	{realm-name}: The name of the realm where your client is configured.
	{client-id}: The client ID of your confidential client.
	{client-secret}: The client secret associated with your client.
	Here's a more concrete example with placeholder values replaced:

	curl -X POST \
		'http://localhost:8080/auth/realms/myrealm/protocol/openid-connect/token' \
		-H 'Content-Type: application/x-www-form-urlencoded' \
		-d 'client_id=myclient' \
		-d 'client_secret=9f8e72e8-4356-4bab-8f8c-9c6d1e5c26e2' \
		-d 'grant_type=client_credentials'
	When you run this command, Keycloak should respond with a JSON object that includes the access token. It will look something like this:

	{
		"access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAi...",
		"expires_in": 60,
		"refresh_expires_in": 0,
		"token_type": "Bearer",
		"not-before-policy": 0,
		"session_state": "12345678-1234-1234-1234-123456789abc"
	}
	You can then use the access_token value as a Bearer token in the Authorization header for subsequent API requests that require authentication.

	Please ensure that you keep your client secret confidential and secure, as it allows anyone to obtain tokens with the permissions granted to the client.


4) Implicit Flow
	The Implicit flow is designed for applications where the client secret cannot be securely stored, such as single-page applications (SPAs). In the Implicit flow, tokens are returned directly via the redirect URI after a successful authentication, without an intermediate code exchange step.

	However, it's important to note that the Implicit flow is considered less secure than the Authorization Code flow with PKCE and has been deprecated in OAuth 2.1 in favor of more secure alternatives. If possible, you should use the Authorization Code flow with PKCE for public clients like SPAs.

	That said, if you still need to use the Implicit flow with Keycloak, here's how you would initiate the flow:

			Redirect the user to the Keycloak authentication endpoint: You need to construct a URL that points to the Keycloak authentication endpoint with the appropriate query parameters and then redirect the user to this URL. The user will authenticate and Keycloak will redirect back to the specified redirect URI with the access token in the URL fragment.

			Here's an example of the URL for the Implicit flow:

			http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/auth?client_id={client-id}&redirect_uri={redirect-uri}&response_type=token&scope=openid

			Replace {realm-name}, {client-id}, and {redirect-uri} with your actual realm name, client ID, and redirect URI.

			User Authentication: The user will be prompted to log in (if not already logged in) and to grant consent if required.

			Receive the Token: After successful authentication, Keycloak will redirect the user to the redirect URI you provided, with the access token appended to the URI fragment (not the query string). This means the token will be in the part of the URL after the #.

			Example of redirect URI with token:

			http://your-app/callback#access_token={access-token}&token_type=bearer&expires_in={expiration-time}&...

			Extract the Token: Because the token is returned in the URI fragment, it is not accessible via curl. Instead, you would typically use client-side JavaScript to extract the token from the URL.

	Here's an example of how you might extract the token in JavaScript:

	if (window.location.hash) {
		const urlFragment = new URLSearchParams(window.location.hash.substring(1)); // Remove the '#' character
		const accessToken = urlFragment.get('access_token');
		// Use the access token for subsequent API calls
	}

	Remember that the Implicit flow should not be used for new applications. Instead, consider using the Authorization Code flow with PKCE, which provides better security and is recommended for applications that cannot securely store a client secret.

5) Device Authorization Grant
	The Device Authorization Grant is an OAuth 2.0 extension that enables devices with no browser or limited input capability to obtain an access token. This flow involves multiple steps:

			Device requests a user code: The device makes a request to the Keycloak's device authorization endpoint to obtain a verification URI and user code.

			User enters the code on a secondary device: The user navigates to the verification URI on a device with a browser (like a smartphone or computer) and enters the user code.

			Device polls the token endpoint: While the user is authorizing the device, the device repeatedly polls the token endpoint until the user completes the authorization process or the code expires.

	Here's how you can use curl to perform each step:
	Step 1: Device Requests a User Code

	curl -X POST \
		http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/auth/device \
		-d 'client_id={client-id}' \
		-d 'scope=openid'

	Replace {realm-name} and {client-id} with your actual realm name and client ID. This request will return a response containing:

			device_code: The device verification code.
			user_code: The code the user needs to enter.
			verification_uri: The URL the user must visit to enter the code.
			verification_uri_complete: The URL including the user code, which simplifies the user experience.
			expires_in: The lifetime in seconds of the device_code and user_code.
			interval: The minimum interval in seconds that the client should wait between polling requests.

	Step 2: User Authorizes the Device

	Direct the user to navigate to the verification_uri (or verification_uri_complete) on a secondary device and enter the user_code. The user will be prompted to log in to Keycloak (if not already logged in) and authorize the device.
	Step 3: Device Polls the Token Endpoint

	While the user is completing the authorization process, the device should start polling the token endpoint at the interval specified in the initial response. Here's how you can poll using curl:

	curl -X POST \
		http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/token \
		-d 'client_id={client-id}' \
		-d 'grant_type=urn:ietf:params:oauth:grant-type:device_code' \
		-d 'device_code={device-code}'

	Replace {realm-name}, {client-id}, and {device-code} with your actual realm name, client ID, and the device_code you received in the first step.

	The device should continue polling the token endpoint until it receives an access token or an error response indicating that the user did not authorize the device within the allowed time frame.

	Once the user completes the authorization process, the token endpoint will respond with an access token that the device can use to make authenticated requests on behalf of the user.

	Please note that the Device Authorization Grant must be enabled for the client in Keycloak for this flow to work. You can enable it by editing the client settings in the Keycloak admin console and setting "OAuth 2.0 Device Authorization Grant Enabled" to "ON".

6) OIDC CIBA Grant
	The OpenID Connect Client Initiated Backchannel Authentication (OIDC CIBA) Grant is a flow that allows a client application to initiate the authentication of an end-user and receive notification of the authentication event through a backchannel. This flow is particularly useful for scenarios where the client is not directly interacting with the end-user at the time of authentication, such as "decoupled" or "headless" authentication.

	To use the CIBA flow with Keycloak, you need to follow these steps:

			Configure Keycloak for CIBA: Ensure that Keycloak is configured to support the CIBA flow. This typically involves enabling CIBA in the realm settings, configuring a client to use CIBA, and setting up the necessary policies and authentication mechanisms.

			Initiate the Authentication Request: The client initiates an authentication request to Keycloak's backchannel authentication endpoint, providing the necessary parameters.

			User Authentication: The user is authenticated out-of-band by Keycloak. This could involve the user receiving a push notification on their mobile device, for example.

			Client Polls for the Token: The client polls Keycloak's token endpoint to check if the authentication has been completed.

	Here's how you can use curl to perform the authentication request and token polling:
	Step 1: Initiate the Authentication Request

	curl -X POST \
		-H "Content-Type: application/x-www-form-urlencoded" \
		-d "client_id={client-id}" \
		-d "scope=openid" \
		-d "login_hint={user-identifier}" \
		-d "binding_message={binding-message}" \
		http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/ext/ciba/auth

	Replace {realm-name}, {client-id}, {user-identifier}, and {binding-message} with your actual realm name, client ID, the identifier for the user you want to authenticate, and a binding message that may be shown to the user during authentication.

	The response from Keycloak will include an auth_req_id that you will use to poll the token endpoint.
	Step 2: Client Polls for the Token

	After initiating the authentication request, the client needs to poll the token endpoint to check if the user has completed the authentication process.

	curl -X POST \
		-H "Content-Type: application/x-www-form-urlencoded" \
		-d "grant_type=urn:openid:params:grant-type:ciba" \
		-d "client_id={client-id}" \
		-d "auth_req_id={auth-req-id}" \
		http://keycloak-server/auth/realms/{realm-name}/protocol/openid-connect/token

	Replace {realm-name}, {client-id}, and {auth-req-id} with your actual realm name, client ID, and the auth_req_id you received from the authentication request.

	The client should continue polling the token endpoint until it receives an access token, a refresh token, or an error response indicating that the authentication was not successful or was canceled.

	Please note that the CIBA flow is an advanced feature and may require additional configuration in Keycloak, such as setting up authentication policies and configuring the client for CIBA support. Additionally, the exact endpoints and parameters may vary depending on the Keycloak version and any customizations that have been applied. Always refer to the Keycloak documentation and your specific Keycloak configuration for the most accurate information.
