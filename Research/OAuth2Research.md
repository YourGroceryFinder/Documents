# OAuth2

## What is OAuth2
Simply said, OAuth2 is the “Continue with Facebook” or “Log in with Google” option you see when trying to log in to some sites. Lets take tiktok as example. When you try to log into tiktok.com you get a list of options of other websites to use to log into. This is all made possible with OAuth2. You can see it as tiktok.com asking Facebook.com for your account information. If you have accepted the request then Facebook.com shares your information with an access token. This way your password keeps protected in one place but you can still use it to log in on tiktok.com.
Not only does OAuth2 do logging in but it also works for other kinds of other kinds of Authentication. The one that I want to use is the API Authentication where when a call is made to the API it expects an access token that has been authenticated.
 
## What is the process
In general there is a total of 8 steps involved in the process of authenticating an API call:
1.	Client registration
The client application(The application you have made so in this case YourGroceryFinder) needs to register with the authorization server to obtain certain credentials(ID and Secret). Some examples of these authorization servers are: Auth0 Authorization Server, Google Authorization Server and Facebook Authorization Server. These are just some examples. There are many more to be found, each with its own features and capabilities.
2.	User Authorization
The client(YourGroceryFinder) redirects the user of the application to the authentication server where the user will be asked to log in and grant the permission to the client.
3.	Grant authorization 
When the user has granted the authorization the server provides an authorization grant to the client. An authorization grant serves as proof that the user has been authorized.
4.	Access Token Request
The client then uses the authorization grant and the client credentials(ID and Secret) to request an access token from the authorization server.
5.	Issuing Of Access Token
Now the server verifies the provided authorization grant and client credentials and return an access token to the client. 
6.	Make API Request
The client then enters the access token in the parameters of its API request to the backend. The access token serves as proof of authentication and authorization.
7.	Validate Token
The backend the validated the access token to ensure it is valid in authorized by performing multiple checks. If any of the checks fail, the backend considers the access token invalid and may reject the API request.
8.	API Responds
If the access token is valid, the backend processes the API request and returns the appropriate response to the client.
 
## How to implement
### Front-end
To implement the Oauth2.0 process you have to complete a few steps.
The first step is getting a login screen when you try to call an API. Since I am using Spring Boot there is a dependency I can add that takes care of this for me. 
 
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```
 
Make sure to put this at the top of all dependencies because it didn’t work when I put it at the bottom. 
This dependency makes it so that when you try to make a call to any endpoint it goes to a login screen when the user is not authenticated yet. It is not connected yet to any Authentication server so that will be then next step. You can still login without an authentication server, by using user as username and use the given password from the console as password.

### Back-end