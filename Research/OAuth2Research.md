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
To implement the Oauth2.0 process you have to complete a few steps.

### Front-end
The first step is logging in. Since we use auth0 this can be quite simple. You can put this step in any component or view you want.
First you want to import the useAuth0 function form the @auth0/auth0-vue library like this:
```js
import { useAuth0 } from '@auth0/auth0-vue';
```
This function enables authentication and user management functionality in the component or view.

Then inside the setup the useAuth0 function is invoked to initialize the auth0 object, which provides access to authentication-related functions and properties.
```js
const auth0 = useAuth0();
```

In the return function you have the isAuthenticated variable which is a reactive property that tells you if someone is authenticated, the isLoading variable which is a reactive property that that tells you if auth0 is loading and then the user variable which represents the user obtained from auth0. There are also functions in the return, thich are login and logout. 
```js
export default {
  name: "NavBar",
  setup() {
    const auth0 = useAuth0();

    return {
      isAuthenticated: auth0.isAuthenticated,
      isLoading: auth0.isLoading,
      user: auth0.user,
      login() {
        auth0.loginWithRedirect();
      },
      logout() {
        auth0.logout({
          logoutParams: {
            returnTo: window.location.origin
          }
        });
      }
    }
  }
}
```
Now you need to trigger the login and logout functions at the right times, you can do that like this:
```js
    <li v-if="!isAuthenticated && !isLoading" style="float:right"><a @click.prevent="login">Log in</a></li>
    <li v-if="isAuthenticated" style="float:right"><a @click.prevent="logout">Log Out</a></li>
```

On the website of auth0 you need to configure a few details like callback url's and thing like that. You can also find some details you need to define in your code. You can use those details to configure your auth0 Login like this:
```js
    createAuth0({
        domain: "dev-oempr7he.eu.auth0.com",
        clientId: "noc9WVhpChShUB5upXEeEEXKxvJNmL8j",
        authorizationParams: {
          redirect_uri: window.location.origin,
          audience: "http://localhost:8080/~/api/",
        }
    })
```

You add this configuration when you create the app. Like this:
```js
import { createApp } from 'vue'
import App from './App.vue'

import 'bootstrap/dist/css/bootstrap.css'
import bootstrap from 'bootstrap/dist/js/bootstrap'
import { createAuth0 } from '@auth0/auth0-vue';

createApp(App).use(bootstrap).use(
    createAuth0({
        domain: "dev-oempr7he.eu.auth0.com",
        clientId: "noc9WVhpChShUB5upXEeEEXKxvJNmL8j",
        authorizationParams: {
          redirect_uri: window.location.origin,
          audience: "http://localhost:8080/~/api/",
        }
    })
).mount('#app')
```
Now that the login should be working, lets go continue with the next steps:
Getting and sending the acces token.

Because we have our API's protected you want to send an acces token with you call to the API to ensure that its authenticated. 
First you need to get the token. You do that by calling the Auth0 function ```js getAccessTokenSilently() ``` like this:
```js
      const token = await this.$auth0.getAccessTokenSilently();
```
Now that you have reveived the token, you want to add it to your API call. You do this by adding it in your header like this.
```js
axios.post('~/api/search/GetProductsBySearch', this.productName, {
            Headers: {
              Authorization: 'Bearer ' + token
            }
          })
```

The total method should look something like this:
```js
 async GetAllResults(){
      const token = await this.$auth0.getAccessTokenSilently();
          axios.post('~/api/search/GetProductsBySearch', this.productName, {
            Headers: {
              Authorization: 'Bearer ' + token
            }
          })
          .then(response =>{
              this.products = response.data;
          })
          .catch(error => {
            console.error(error);
          })
    }
```

### Back-end
In the backend we want to make sure we have the right dependenys installed in the pom.xml file. The one that you want to look for is the dependency for OAuth2 which looks like this:
```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
		</dependency>
```
Next we go to the application.properties or the application.yml file. In this file we want to set the recourse server to auth0 by doing this.
application.properties:
```
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://dev-oempr7he.eu.auth0.com/
```
application.yml:
```yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issur-uri: https://dev-oempr7he.eu.auth0.com/  
```
Next is creating the security config file. In our case its called the SecurityConfig

```java
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{
        http
                .authorizeHttpRequests ()
                .requestMatchers ("/GetAllProducts").authenticated()
                .requestMatchers ("/GetProductsBySearch").authenticated()
                .requestMatchers("/hello").permitAll()
                .and().cors()
                .and().oauth2ResourceServer().jwt();
        return http.build();
    }
}
```

First you want to enable web security for this application. You do that with by putting ``` @EnableWebSecurity ``` before you make your class.
Then before making your Security filter you want to put ``` @Bean ``` This means that the method will create a bean that will be managed by the Spring container.

Next you want to define the method ``` SecurityFilterChain filterChain(HttpSecurity http) throws Exception ```. This method defines the security filter chain that is responsible for processing and enforcing the security rules.

Now lets get to the security rules. First you want to state ``` http.authorizeHttpRequests ```. This will sttart the configuration of authorizing HTTP requests.
After that come the ``` .requestMatchers()``` In these you want to specify which endpoint needs what level of authorization. ``` /GetAllProducts ``` Needs to be authenticated while ``` .hello ``` Doesnt.
Next you want to enable cors and after that you want to enable OAuth 2.0 resource server support with JWT as the authentication mechanism. Meaning you want to receive a valid token.

And the final step is to return the configured HttpSecurity Object.
