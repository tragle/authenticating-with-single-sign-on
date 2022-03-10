# Authenticating with single sign-on

This repo contains a reference app that integrates with an Ouath 2.0 identity provider to provide a single sign-on option for the user. 

To use this repo, read the background information in the How this works section, then follow the instructions in the Using the app section.

## How this works

[Oauth 2.0](https://oauth.net/2/) is the industry standard protocol for authorization and single sign-on. If you want your application to allow users to "Connect with Google" (for example) in order to log in to your app or to access an API, then you need to understand how Oauth 2.0 (and the OpenID Connect extension) works. 

The good news is that once you understand the basic Oauth 2.0 and OIDC flow, you can use any identity provider to implement authorization and authentication for your application. All identity providers follow the same Oauth 2.0 specification, which means that if you can integrate with one identity provider, you know everything you need to integrate with the others.

### What OAuth provides

OAuth provides a standard way to implement **delegated authorization**. In a delegated authorization workflow, the user grants permission for one party to access information from another party. The user achieves this by proving their identity and granting permission to specific resources, which kicks off the secure transmission of data from the resource.

You have likely seen this workflow many times. If you have ever signed up for a new service by clicking a "Connect with Google" or "Connect with Facebook" button, for instance, you have experienced delegated authorization.

The advantage of such a workflow is that users can centralize their online identity and use a single service (or a few) to log on to many applications. Developers of these applications likewise need not create a complex user account system, since they can delegate this task to the identity provider. They can instead focus on creating better user experiences by leveraging the services of the identity provider.

In order for this to work, there needs to be a standard way to grant permission to a resource and subsequently access that resource. That's where OAuth 2.0 comes in. 

OAuth 2.0 is a protocol for delegated authorization, and it provides a framework for multiple parties to negotiate the exchange of permissions. Understanding the basics of OAuth 2.0 will help you understand how to integrate with identity providers, allowing you to create solid and professional authorization experiences.

#### What OAuth doesn't provide

1. Strictly speaking, OAuth 2.0 was designed for *authorization*, not *authentication*.  That is, it is concerned with *what* someone can access, not who they are. 

  To handle authentication, the OAuth 2.0 group created an extension to the specification, called **Open ID Connect** or OIDC. OIDC is a small layer on top of OAuth 2.0, and provides a way to grant permission to user information.

2. OAuth is not concerned with how to structure permissions within an application. Most identity providers will include features like user roles and attributes, however, if you need fine-grained access control.

3. OAuth is not concerned with user sessions or cookies. 

### Terms

| Term | Definition |
| ---- | ---------- |
| Identity Provider | Cloud-based service to store and manage user identities. Also called an "Idp". |
| Grant | An OAuth flow.  |
| Claim | A piece of information asserted about a subject. The body of an **Access Token** contains claims about the token, like its scope and expiration. | 
| Resource Owner | The user; the person who owns the identity and data connected to their accounts. |
| Client | The application that wants to access data on behalf of the **Resource Owner**. |
| Authorization Server | The application where the **Resource Owner** already has an account. |
| Resource Server | The API that the **Client** wants to use on behalf of the **Resource Owner**. |
| Redirect URI | The URL that the **Authorization Server** sends the **Resource Owner** back to after they have granted permission to the **Client**. Also called the "Callback URL." |
| Response Type | The type of information the **Client** expects back from the **Authorization Server**. |
| Scopes | The permissions the **Client** wants. |
| Consent | The message displayed to the **Resource Owner** by the **Authorization Server** asking if the **Client** should be given permission to the **Scopes** listed. |
| Client ID | An ID issued to the **Client** by the **Authorization Server**. |
| Client Secret | A secret password that only the **Client** and the **Authorization Server** know, allowing the **Access Token** to be retrieved securely. |
| Authorization Code | A short-lived code that the **Client** sends to the **Authorization Server** in exchange for an **Access Token**. |
| Access Token | The key used by the **Client** to communicate with the **Resource Server**. Also called a **Bearer Token**.|

### OAuth Flows

The OAuth specification defines several flows (or grants), depending on your use case.

#### Authorization Code Flow ####

In this flow, the Client receives a short-lived Authorization Code, which it exchanges for an Access Token on a secure back-channel.

Here are the steps of the Authorization Code Flow:

1. The Client opens the Authorization Server's login page on the Resource Owner's browser, embedding a Redirect URI, Response Type (which must include `code`), and Scopes in the URI.
2. The login page includes a Consent message based on the Scopes (permissions) requested by the Client.
3. Once the Resource Owner grants permission, the Authorization Server redirects their browser to the address in the Redirect URI, with the Authorization Code embedded in the URI.
4. The browser sends the Authorization Code to the Client backend. 
5. The Client backend sends the Authorization Code to the Authorization Server's `/token` endpoint, along with a Client ID and Client Secret.
6. If everything is correct, the Authorization Server returns an Access Token to the Client backend. (The Access Token is normally a [JWT](https://auth0.com/docs/secure/tokens/json-web-tokens).)
7. The Client backend then makes a call to the Resource Server, passing along the Access Token. 
8. The Resource Server examines the claims in the token and sends back the requested resource.

The Authorization Code flow is secure but complex. 

Secure because it is practically impossible for an attacker to intercept the Access Token unless they also obtain the Client Secret, which is not exposed to the frontend. 

Complex because the Authorization Server requires the extra step of exchanging the Authorization Code for the Access Token, instead of providing the Access Token directly. This exchange happens server-to-server, meaning that it would not work for single-page applications.

#### Implicit Flow ####

In the Implicit Flow, the Authorization Server sends the Access Token to the Client via the final frontend redirect. There is no Access Code -- the Client simply gets the Access Token directly. 

The steps are similar to the Authorization Code Flow, with these differences:

- The Response Type must include `token` instead of `code`.
- The Authorization Server returns the token as a parameter in the Redirect URI during the final redirect.
- There is no need to call the Authorization Server on the backend.

The Implicit Flow is simpler but less secure than the Authorization Code Flow.

It allows single-page applications to take advantage of OAuth, however it is more vulnerable to attack because the token is transmitted on the front-channel.

The OAuth 2.0 authors and all major identity providers now consider the Implicit Flow to be **deprecated**. Instead, use the **Authorization Code Flow with PKCE** flow.

#### Authorization Code Flow with PKCE ####

PKCE stands for "proof key for code exchange" and was created to allow untrusted, public clients to use the Authorization Code Flow.

PKCE works by generating a dynamic secret that the Authorization Server can verify, and that can be used in a public client like a SPA (unlike a fixed secret).

This flow is similar to the Authorization Code Flow, with these differences:

- The Client includes a hashed random value called a Code Challenge in the initial request to the Authorization Server.
- Instead of passing the Authorization Code to a backend, the frontend calls the `/token` endpoint directly, passing along the Code Verifier (the unhashed value of the Code Challenge) as the secret.
- There is no need to call the Authorization Server on the backend.

PKCE combines the ease of the Implicit Flow with the security of the Authorization Code Flow, and is now commonly used in browser and mobile-based applications.

#### Other Flows ####

The OAuth 2.0 specification defines [several other grant types](https://oauth.net/2/grant-types/).

- **Client Credentials** for server-to-server communication, where no user is involved. 
- **Device Code** for input-constrained devices (for example, a gaming console).
- **Refresh Token** which allows the Client to get a new Access Token without user input.

### OpenID Connect ###

OpenID Connect adds authentication to OAuth 2.0. OIDC is not a grant type, but an extension to the OAuth 2.0 specification. 

OIDC is different from the existing flows in these ways:

- The Client passes an `openid` scope in the initial request to the Authorization Server.
- The Authorization Server returns an **ID Token** along with the Access Token.
- The Client can use the claims in the ID token to get information about the user, or it can request user information from a `/userinfo` endpoint.

Once the Client has the ID token, it can consider the user authenticated. 

## Using the app


