#SpringBoot OAuth Using KeyCloak


Demonstrates how to integrate a spring boot application with KeyCloak for basic Authentication

## KeyCloak Set up

1. Download ZIP from official KeyCloak site, unzip to a local folder
2. Start the server at port 8180: standalone.bat -Djboss.socket.binding.port-offset=100
3. Set up an admin username and password
4. Create the Realm, Client, Role, User and Map the Role to the user
```
Realm: SpringBootKeyCloak
Client: jj-demo-login
Role: user
User: jjacob/xxxxxxxx
```

## Spring Boot Application Set up

1. Configure SpringBoot Security to protect `/admin` URL so that it is accessible only to users having role "admin"
```
.antMatchers("/admin*")
.hasRole("admin")
.anyRequest()
.permitAll();
```

2. Set the following properties
```
keycloak.auth-server-url=http://localhost:8180/auth
keycloak.realm=SpringBootKeycloak
keycloak.resource=login-app
keycloak.public-client=true
#Populates the user name instead of UUID in the Principal
keycloak.principal-attribute=preferred_username
```
3. Accessing `http://localhost:8080/admin`, the application will be redirected to `http://localhost:8180/auth` for KeyCloak authentication.
After successful authentication, the request will be re-directed back to display the logged in admin user's name on the screen.
   
## Generating Access Tokens With Keycloak's API
Keycloak provides a REST API for generating and refreshing access tokens. We can easily use this API to create our own login page.
First, we need to acquire an access token from Keycloak by sending a POST request to this URL:
```
http://localhost:8180/auth/realms/SpringBootKeycloak/protocol/openid-connect/token
```
The request should have this body in a x-www-form-urlencoded format:

```
client_id:<your_client_id>
username:<your_username>
password:<your_password>
grant_type:password
```

In response, we'll get an access_token and a refresh_token.
The access token should be used in every request to a Keycloak-protected resource by simply placing it in the Authorization header:
```
headers: {
    'Authorization': 'Bearer' + access_token
}
```

Once the access token has expired, we can refresh it by sending a POST request to the same URL as above, 
but containing the refresh token instead of username and password:

```
{
    'client_id': 'your_client_id',
    'refresh_token': refresh_token_from_previous_request,
    'grant_type': 'refresh_token'
}
```
Keycloak will respond to this with a new access_token and refresh_token.

## Further Reading

1. https://spin.atomicobject.com/2016/05/30/openid-oauth-saml/
2. https://developer.okta.com/books/api-security/authn/federated/
3. https://medium.com/@robert.broeckelmann/authentication-vs-federation-vs-sso-9586b06b1380
4. https://github.com/dsyer/spring-security-oauth2-hydra
