Sails-OAuth2-API
----------------

Purposes
--------

Several purposes of this application:

* integrate OAuth2 to protect a REST API developped with sailsjs
* make this integration simple and easy to understand
* handle trusted and untrusted application
  - trusted application will use resource owner's password credential (no login form displayed to the user)
  - untrusted application will use authorization code grant (login form displayed and presenting the Allow / Deny options)


Status
------

- available flows (not fully tested yet)
  * Resource owner password
  * Authorization code
  * Implicit flow (not developed yet)

- models implemented
  * User: resource owner
  * Client: application willing to use the user's resource
  * AuthCode: authorization code that will be exchanged against an Access Token
  * AccessToken: token used by the client each time the API is called with the user's identity
  * RefreshToken: token used to get a new AccessToken

- controllers
  * InfoController: a basic controller that will be called to test the OAuth mecanism

- authentication
  * config/passport.js: defining passport strategy

- OAuth service
  * config/oauth2.js defining the OAuth server and the exchange strategy

Details
-------

When lifting the sails application, a default user and 2 defaults clients are created (among which one is trusted and the other is not).
In the console, the client_id and client_secret of each client are displayed and the default user credential as well.

**Resource owner password flow** (this flow is only available if the client is among the trusted clients)

Issue the following curl request to get an access token

```
curl -XPOST "http://localhost:1337/oauth/token" -d "grant_type=password&client_id=CLIENT_ID&client_secret=CLIENT_SECRET&username=USERNAME&password=PASSWORD"
```

This returns an access token and a refresh token in a the following json format

```json
{"access_token":"UeCnysUiLLzxS6tCKkEeMnRxvTsyq5bri9AFUeQV4OEOqoketVZd7HVQpjOeWOLwBhwaWokFXdBsQ34oU0Kcafq8cHgS3lu2Si6I2xvKifo46F8HiU18aicWTzizNocfHVKYYFEhcYftEVEmyvrkcPt1loaAHcKAhY8IzobgkTiMh6ZTfAdQKWn7pM0iS1sojW8H0v6pL9xLNRj0lwbTNHcMDWwdfCCGEq9NuZAiFuKspOg5LeLYKSXxm0vQAHFr","refresh_token":"zu1dbMCuP46NS2hqjmq1ZFPzNrVsSpM9BvFCOizo3GmrE9jRwrY26m1b6JK3Jbud4ejb2xw3MZZc56snT15Y9hWXsmvGSOyKufS0cu8ZKGfVwUjwBcyu7SkcZCcCLUDgq5BJzFJ9ZBv6TKwltdUb8LQAEcDSLLRAXbIHsorStKW0CXqNuL9iSVdKgTXMVkiT2ik8Z4PUMf3daLQSMvwPK69srvYttFNpM3mUMOC2Y2U0AmiRDLYIr3Nsid0hwGsi","expires_in":3600,"token_type":"Bearer"}
```

Note: if the curl command above is issued with the client_id of the untrusted client (third party client applications that require access to the resource), a 401 error is raised.

**Authorization code grant**

Within the application, redirect the user toward the authentication service (the user will need to identify and allow the client to access his resources)

```
http://localhost:1337/oauth/authorize?client_id=CLIENT_ID&response_type=code&redirect_uri=REDIRECT_URIhttp://localhost:1338&scope=http://localhost:1337
```

Once the authorization code is received, exchange it against an access token with the following request

```
curl -XPOST -d 'client_id=CLIENT_ID&client_secret=CLIENT_SECRET&grant_type=authorization_code&redirect_uri=REDIRECT_URI&code=CODE' http://localhost:1337/oauth/token
```

This returns an access token and a refresh token in a the following json format

```
{"access_token":"72HglTG5tJshVbpnTTCBFw1FNtB4FolLlV5Hntzc4prcAwVpTBXreyFzk9rCBUsaevdsJBY9v4YarEFfvhVLqL5HZmznUI3ajXmNQvlo7k5MD8E0SlVqMdtJeyYBtPa21bPOiFGpkDhoT6dOVecDWhuaT191cwsQT6jv663gRi63t4AXU443GuZKGuQU6Upt9S3BSiLmSMrvL6whvyORl66jFdL7EckRNSYNX3eHUdjcHdxluGWUNuLwhBIMOr3y","refresh_token":"1t3hKouYXAgoeNTtkxyCOBjDyEyr4U99i21B4lkLq5SAl2NQ94UaLXMVEMT93J3D0q8GMhzPIlIQD2mSOcooPSM3txZ2nBdEOa1MX8GYBQcOsN55DLhJo7PxbbTKKqQqGS04ZsVBEQYPd9Xv80aj6tO5w0lP2qfcVq9YdcvLqQ43hk2h4F7RIHUgXhM9lfqLH0K0gsmdtyR2YWzJphbbori8JhAtvBqRzuyiiwFVOzjlK21f9qULKlq7T7ftqQ8S","expires_in":3600,"token_type":"Bearer"}
```

**Implicit flow**

Not implemented yet

Credits
-------

Several projects were used to start this one:

* http://aleksandrov.ws/2013/09/12/restful-api-with-nodejs-plus-mongodb/
* https://github.com/aaron524/sails-oauth2-provider-example
