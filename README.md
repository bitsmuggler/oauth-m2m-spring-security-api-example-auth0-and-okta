# OAuth 2.0 M2M API Authentication Example with Spring-Security

This is an example project how to map the [OAuth client credentials flow](https://tools.ietf.org/html/rfc6749#section-4.4) (machine-to-machine authentication) with spring-security and [Auth0](https://auth0.com/de/) the client credentials flow.

This introduction supports two possible Identity-as-a-Service (IDaas) solutions.

* [Auth0](https://auth0.com)
* [Okta](https://developer.okta.com)


## OAuth 2.0 Client Credentials Flow

👉  See the [OAuth 2.0 RFC 6749](https://tools.ietf.org/html/rfc6749#section-4.4)


     +---------+                                   +---------------+
     |         |                                   |               |
     |         |>--(A)- Client Authentication ---> | Authorization |
     | Client  |                                   |     Server    |
     |         |<--(B)---- Access Token ---------< |               |
     |         |                                   +---------------+
     |         |                                   |               |
     |         |                                   |   Your App    |   
     |         |>--(C)- Call private (scoped) API->|               |               
     +---------+                                   +---------------+

            Client Credentials Flow from the OAuth 2.0 RFC 6749
            
       (A)  The client authenticates with the authorization server and
            requests an access token from the token endpoint.
    
       (B)  The authorization server authenticates the client, and if valid,
            issues an access token.        
            
       (C)  The client calls your API with the issued token.     


## [Auth0](https://auth0.com)

### Configuration

* Register and setup your tenant
* Create an API ``APIs > Create API``
* Create a permission in your created API ``APIs > your created API > Permission > create Permission read:messages``
* Create an application which the api should consume ``Applications > Create Application > Machine to Machine Applications``
* Grant your app to the permission ``Applications > your application > APIs > select your API > Add the permission read:messages``

### Configuration spring-boot app

The configuration of the spring-boot app is located in ``/src/main/resources/application.yml``

1. Set your auth0 ``audience`` id
1. Set your ``issuer`` uri

### Usage of the endpoints

* Public Endpoint: ``http://localhost:8080/api/public``
* Private Endpoint: ``http://localhost:8080/api/public``
    1. Get a token: ``https://<<your-auth0-issuer-uri>>/oauth/token``
        * cURL: ``curl --location --request POST 'https://dev-1234.eu.auth0.com/oauth/token' \
                  --header 'Content-Type: application/json' \
                  --data-raw '{
                      "client_id":"your client id",
                      "client_secret":"your client secret",
                      "audience":"your audience id",
                      "grant_type":"client_credentials"
                  }'``
    1. Call the private endpoint with the ``authorization`` header and the received bearer token
        * cURL: ``curl --location --request GET 'http://localhost:8080/api/private' \
            --header 'Authorization: Bearer <<your bearer token>>'``
* Private Scoped Endpoint
    1. Get a token with the desired scope
        * cURL: ``curl --location --request POST 'https://dev-1234.eu.auth0.com/oauth/token' \
                  --header 'Content-Type: application/json' \
                  --data-raw '{
                      "client_id":"your client id",
                      "client_secret":"your client secret",
                      "audience":"your audience id",
                     "grant_type":"client_credentials",
                      "scope": "read:messages"
                  }'``
    2. Call the api with the token
        * cURL: ```curl --location --request GET 'localhost:8080/api/private' \
                  --header 'Authorization: Bearer <<your token>>'```               
    
### Troubleshooting

* "Error: Client has not been granted scopes"
    *  Grant your app to the permission ``read:messages``
    * ``Applications > APIs > Machine to Machine Applications > Your API > Select read:permissions``
    
    
## [Okta](https://developer.okta.com)

### Configuration    
       
* Creating an Authorization Server ``API > Add Suthorization Server`` 
    * Choose the name and audience of the new authorization server
* Add some scopes, e.g. 
    * ``read:messages`` for our private-scoped api
    * ``read:private`` for our private (default-scoped) api
        * Set this as default scope
* Add an new Access Policy
    * Add a new Rule which is acting on ```Client Credentials``` and save the rule 
* Add a new Application ``Applications > Add Application``
    * Select ```Service (Machine-to-Machine)```
    * Name your app
    * That's it
* Check if you get an access token with this configuration
                      
### Usage of the endpoints

* Public Endpoint: ``http://localhost:8080/api/public``
* Private Endpoint: ``http://localhost:8080/api/public``
    1. Get a token: ``https://<<your-auth0-issuer-uri>>/oauth/token``
        * cURL: ``curl --location --request POST 'https://dev-301706.okta.com/oauth2/ausr5mf82ovfbtxo74x6/v1/token' \
                  --header 'Authorization: Basic <<basic auth string>>' \
                  --header 'Content-Type: application/x-www-form-urlencoded' \
                  --data-urlencode 'grant_type=client_credentials' \
                  --data-urlencode 'scope='``
    1. Call the private endpoint with the ``authorization`` header and the received bearer token
        * cURL: ``curl --location --request GET 'http://localhost:8080/api/private' \
            --header 'Authorization: Bearer <<your bearer token>>'``
* Private Scoped Endpoint
    1. Get a token with the desired scope
        * cURL: ````curl --location --request POST 'https://dev-301706.okta.com/oauth2/ausr5mf82ovfbtxo74x6/v1/token' \
                                    --header 'Authorization: Basic <<basic auth string>>' \
                                    --header 'Content-Type: application/x-www-form-urlencoded' \
                                    --data-urlencode 'grant_type=client_credentials' \
                                    --data-urlencode 'scope=read:messages'````
    2. Call the api with the token
        * cURL: ```curl --location --request GET 'localhost:8080/api/private' \
                  --header 'Authorization: Bearer <<your token>>'```               
                          
                      
## Start the example app

``mvn spring-boot:run``       

## Resources

* [OAuth 2.0 RFC 6749](https://tools.ietf.org/html/rfc6749#section-4.4)
* [Client Credentials Flow explained by Auth0](https://auth0.com/docs/flows/client-credentials-flow)
* [Using machine to Machine (M2M) Authorization](https://auth0.com/blog/using-m2m-authorization/)
