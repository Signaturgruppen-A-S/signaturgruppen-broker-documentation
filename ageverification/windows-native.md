---
title: Age Verification for Windows applications 
layout: home
parent: Age Verification
nav_order: 1
---

# Windows applications 
This section describes how to implement MitID Age Verification from a native Windows application.

# Implementation overview

1. **Use the System Browser**: Use the system’s default browser for authentication instead of embedded web views. This leverages the browser’s security features, reduces the risk of phishing attacks, and ensures users interact with a trusted interface.
Leverage Loopback Redirects: For native applications, use a loopback address (e.g., http://127.0.0.1) as the redirect URI. This minimizes potential interception of tokens and aligns with the recommendations in [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252).

2. **Use the Authorization Code Flow with PKCE**
Authorization Code Flow: This is the recommended flow for native applications since it separates the authentication process from the token issuance.
Proof Key for Code Exchange (PKCE): Implement PKCE to add an extra layer of security by mitigating interception risks during the authorization code exchange process.
> Note, that this is a recommended security best practise for OAuth implicit public client flows, but for Age Verification flows it is an optional feature as the risk associated with interception of the ID token of a malign party is minimal. But you should as a service provider always do your own risk assesment of this.

4. **ID token validation (Required)**
Validate the received ID token: Always verify the ID token’s signature, check the token’s issued time, issuer, and audience to ensure it is valid and hasn’t been tampered with.
Implement Nonce and State Parameters: Use a nonce parameter in the authentication request to protect against replay attacks and a state parameter to mitigate CSRF risks.
The ID token received from the Age Verification flow is data minimized and does only contain the age verification information, no personal information is relayed in this token.
Expiration of the token is not important for age verification flows, it is more important to utilize the **nonce** parameter in order to ensure that the Age Verification result matches the correct request.

6. **Use Up-to-date and Well-supported Libraries**
Library Choices: Consider using libraries like MSAL (Microsoft Authentication Library) or IdentityModel.OidcClient which are designed for native applications and handle much of the complexity around OIDC flows.

7. **Follow OAuth 2.0 for Native Apps Guidance**
Security Recommendations: Align your implementation with the OAuth 2.0 for Native Apps best practices, which emphasize using system browsers, loopback interfaces, and secure token exchange methods.
HTTPS Everywhere: Ensure all endpoints, including redirect URIs and token endpoints, are secured via HTTPS to protect data in transit.

# Loopback example

## Windows C# example of how to setup a loopback listener. 

```c#
using System;
using System.Net;
using System.Text;
using System.Threading.Tasks;

namespace LoopbackListenerExample
{
    class Program
    {
        static async Task Main(string[] args)
        {
            // Define the loopback URL and port
            string url = "http://127.0.0.1:5000/";
            
            // Initialize HttpListener and add the URL prefix
            using (HttpListener listener = new HttpListener())
            {
                listener.Prefixes.Add(url);
                
                try
                {
                    // Start listening for incoming HTTP requests
                    listener.Start();
                    Console.WriteLine($"Listening for requests on {url}");
                    
                    // Wait for an incoming request (this can be run in a loop or handled per request)
                    HttpListenerContext context = await listener.GetContextAsync();
                    
                    // Log the incoming request details
                    Console.WriteLine("Received request: " + context.Request.RawUrl);
                    
                    // You can extract query parameters here, e.g., the authorization code from OpenID Connect
                    // string authCode = context.Request.QueryString["code"];
                    
                    // Respond to the browser (for example, a simple HTML page)
                    string responseHtml = "<html><body><h1>You may now close this window.</h1></body></html>";
                    byte[] buffer = Encoding.UTF8.GetBytes(responseHtml);
                    
                    context.Response.ContentLength64 = buffer.Length;
                    context.Response.ContentType = "text/html";
                    await context.Response.OutputStream.WriteAsync(buffer, 0, buffer.Length);
                    context.Response.OutputStream.Close();
                    
                    // Optionally, break out of the loop after a successful request or continue listening
                }
                catch (HttpListenerException hlex)
                {
                    Console.WriteLine("HttpListener Exception: " + hlex.Message);
                }
                catch (Exception ex)
                {
                    Console.WriteLine("Exception: " + ex.Message);
                }
                finally
                {
                    listener.Close();
                }
            }
            
            Console.WriteLine("Listener stopped. Press any key to exit.");
            Console.ReadKey();
        }
    }
}

```

**Key Points**
* Loopback URL: The URL http://127.0.0.1:5000/ is used as the redirect URI. In a real OpenID Connect integration, you would register this URL with your identity provider. 
* HttpListener Setup: The HttpListener is configured to listen on the specified URL prefix. Ensure your application has the necessary permissions; you may need to run the application as an administrator or register the URL namespace using the netsh command.
* Asynchronous Handling: The code uses asynchronous methods (GetContextAsync) to avoid blocking the main thread while waiting for the authentication response.
* Response to Client: Once a request is received, the application sends back a simple HTML response that can inform the user that the authentication process is complete.

This example sets up a minimal loopback listener which can be extended to handle additional authentication parameters (like the authorization code) required for completing the OpenID Connect flow.

# Setting up OIDC requests
OpenID Connect Authority: 

```
PP: https://pp.netseidbroker.dk/op
```

```
Production: https://netseidbroker.dk/op
```

## Steps overview 

1. Setup a client running OIDC Authorization Code Flow with a public client (no secret involved) for backend-less setups, and with a secure client for backend setups.
2. Use your OIDC library to start the flow in the system default browser like:

```
https://pp.netseidbroker.dk/op/connect/authorize?client_id=YourClientId&redirect_uri=http://127.0.0.1:5000&response_type=code&scope=openid%20age_verify:16&prompt=login&nonce=yourNonce
```
3. Handle the redirect_uri loopback callback, which will look something like:

```
http://127.0.0.1:5000/?code=83FEAECA845AD6DEA6319F4EFD1EC8D94DAA7D9422B12FFA63D2C08D81987DB4-1&scope=openid%20age_verify%3A16&session_state=ggqpYVItsO2VEn6HYdzNqsKaHzKSlAfP2LX3KENCwLA.58E7B5C4D83B44F8AE9A5820DB3D9E53&iss=https%3A%2F%2Fpp.netseidbroker.dk%2Fop
```
Use your OIDC library to exchange the **code** to token output, which will contain the ID token. Validate the ID token using your library. Verify the age validation and verify that the nonce parameter in the ID token matches the expected value.
