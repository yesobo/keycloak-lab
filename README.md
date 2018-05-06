# Keycloak container behind kong base url

Running a Keycloak container behind Kong at *http://[kong_proxy_host]/keycloak/*

Run keycloak container
```
sudo docker run -d --name keycloak-proxy -p 8083:8080 -e DB_VENDOR=H2 -e PROXY_ADDRESS_FORWARDING=true -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=pass yesobo/keycloak
```

Where *yesobo/keycloak* is  the image created from the *Dockerfile* in this repository
```
FROM jboss/keycloak:latest

USER jboss

RUN sed -i -e 's/<web-context>auth<\/web-context>/<web-context>keycloak\/auth<\/web-context>/' $JBOSS_HOME/standalone/configuration/standalone.xml
```

Kong API Object (Since Kong 0.13.0. It is strongly recommended to migrate your APIs to Routes and Services.)
```
{
  "created_at": 1525514033117,
  "strip_uri": true,
  "id": "462cd04f-f1c3-4ad8-ac98-e4b46ac49716",
  "hosts": [
    "yesobo-xps-13-9343"
  ],
  "name": "keycloak",
  "http_if_terminated": false,
  "https_only": false,
  "retries": 5,
  "uris": [
    "/keycloak"
  ],
  "preserve_host": false,
  "upstream_connect_timeout": 60000,
  "upstream_read_timeout": 60000,
  "upstream_send_timeout": 60000,
  "upstream_url": "http://172.18.0.1:8083/keycloak"
}
```

Keycloak url
```
http://yesobo-xps-13-9343:8000/keycloak/auth/
```

## Keycloak: Getting Started (ref: )
- Creating realm (i.e. demo)
- Creating user (i.e. testuser)
- Enter user console http://yesobo-xps-13-9343:8000/keycloak/auth/realms/demo/account

## Keycloak: Creating and Registering the client
- Create a client with keycloak admin console (i.e. Testclient)
- Set Client protocol to *openid-connect*
- Set Client Access-type to *confidential*
- Generate token in *Credentials* tab
- Set client base url (i.e. http://localhost:3000)

## Keycloak: Securing Node app

Use *passport* and *passport-openidconnect* strategy and configure passport with the Keycloak client info

```
passport.use(new Strategy({
    clientID: 'testclient',
    clientSecret: 'b9674551-58db-48c3-aa64-fcaacf74df9e',
    authorizationURL: 'http://yesobo-xps-13-9343:8000/keycloak/auth/realms/demo/protocol/openid-connect/auth',
    tokenURL: 'http://yesobo-xps-13-9343:8000/keycloak/auth/realms/demo/protocol/openid-connect/token',
    userInfoURL: 'http://yesobo-xps-13-9343:8000/keycloak/auth/realms/demo/protocol/openid-connect/userinfo',
    callbackURL: 'http://localhost:3000/callback'
  },
  function(token, tokenSecret, profile, cb) {
    // In this example, the user's Twitter profile is supplied as the user
    // record.  In a production-quality application, the Twitter profile should
    // be associated with a user record in the application's database, which
    // allows for account linking and authentication with other identity
    // providers.
    return cb(null, profile);
  }));
```

Tthe config info can be retrieved via the url

http://keycloakhost:keycloakport/auth/realms/{realm}/.well-known/openid-configuration
For example, if the realm is demo,

```
  http://yesobo-XPS-13-9343/keycloak/auth/realms/demo/.well-known/openid-configuration
```