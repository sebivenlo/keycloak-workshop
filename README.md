# keycloak-workshop

Keycloak is an open source software product to allow single sign-on with Identity and Access Management aimed at modern applications and services.

## Installation
The steps included here requires Docker.

### Create a user defined network

To make it easy to connect Keycloak to LDAP and the mail server create a user defined network:

    docker network create demo-network

### Start Keycloak

We're going to use an extended Keycloak image that includes a custom theme and some custom providers.

First, build the custom providers and themes with:

    mvn clean install

Then build the image with:
    
    docker build -t demo-keycloak -f keycloak/Dockerfile .

Finally run it with:

    docker run --name demo-keycloak -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin \
        -p 8080:8080 --net demo-network demo-keycloak

### Start LDAP server

For the LDAP part of the demo we need an LDAP server running.

First build the image with:

    docker build -t demo-ldap ldap
    
Then run it with:

    docker run --name demo-ldap --net demo-network demo-ldap
    
### Start mail server

In order to allow Keycloak to send emails we need to configure an SMTP server. MailHog provides an excellent email
server that can used for testing.

Start the mail server with:

    docker run -d -p 8025:8025 --name demo-mail --net demo-network mailhog/mailhog
    
### Start JS Console application

The JS Console application provides a playground to play with tokens issued by Keycloak.

First build the image with:

    docker build -t demo-js-console js-console
    
Then run it with:

    docker run --name demo-js-console -p 8000:80 demo-js-console

### Done
You should have all the applications required for the demo running.

Running *docker ps* should output something like this:

```
bogdan@Bogdans-MacBook-Pro ~ % docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
a2a58e7633b0        mailhog/mailhog     "MailHog"                24 minutes ago      Up 24 minutes       1025/tcp, 0.0.0.0:8025->8025/tcp   demo-mail
f968f9485b3e        demo-keycloak       "/opt/jboss/tools/do…"   29 minutes ago      Up 29 minutes       0.0.0.0:8080->8080/tcp, 8443/tcp   demo-keycloak
7f37e5f3aa72        demo-js-console     "httpd-foreground"       39 minutes ago      Up 28 minutes       0.0.0.0:8000->80/tcp               js-console
b80f20f793d9        demo-ldap           "/container/tool/run"    42 minutes ago      Up 28 minutes       389/tcp, 636/tcp                   demo-ldap

