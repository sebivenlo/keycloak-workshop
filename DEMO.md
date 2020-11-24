# Demo

Keycloak is intended to be very simple to use, it is ready out of the box and allowing you to authenticate users and securely invoke REST APIs and microservices.

The way it works: When a user wants to login into an application the authentication is delegated to keycloak and then keycloak returns the access token and identity token.
Using the access token the application can invoke other REST APIs or microservices without the need of another login.

Keycloak can pull users from User Storage spaces such as LDAP or custom relational database, social media (github, facebook, google).

## Start demo

Administration console: can manage the users and realms.

### Realm
A realm manages a set of users, credentials, roles, and groups. A user belongs to and logs into a realm. Realms are isolated from one another and can only manage and authenticate the users that they control.

1. Add new realm 
   * Click add realm top left and add a name)
2. Add application to the realm 
   * Click on clients -> create -> input client ID ( js-console) -> change access type to public -> valid redirect URIs (http://localhost:8000/*) -> add web origin because we want to let the application make ajax requests so we can refresh the app during the demo ( http://localhost:8000)
3. Configure email setting
   * Go to master realm
   * Set an email address for the current user
   * Go back to demo realm->realm settings->email and use the MailHog credentials for demo purposes. host: demo-mail ; port:1025; email: keycloak@localhost
4. Instead of the admin having to create all the users we can let the users register for themselves 
   * Enable registration in the realm ( user registration and verify email)
5. Open example application
   * Register a user
   * Verify email in mailhog
You can always change the look and feel of the forms coming from keycloak. 
6. Add more information for the user in the system
   * Go to users and select the registered user
   * Go to attribute and add a new attribute ( avatar_url, value: https://cdn.pixabay.com/photo/2020/11/02/13/25/north-sea-oats-5706656_960_720.jpg)
   * Create a new client scope named avatar_url
   * Create a mapper with name: avatar_url; user attribute: avatar_url; token claim name: avatar_url; claim json type: string;
   * Change client scopes inside the client js-console

The id token is used by the application to establish the authentication of the user. The access token is used to invoke microservices ( in this way the user doesn`t have to login again).
### Client Scope 
When a client is registered, you must define protocol mappers and role scope mappings for that client. It is often useful to store a client scope, to make creating new clients easier by sharing some common settings. This is also useful for requesting some claims or roles to be conditionally based on the value of scope parameter. Keycloak provides the concept of a client scope for this.

7. Lets say that the application is a third party application, we can ask for consent during the login phase 
   * Enable consent in client settings
   * Add display name for attribute avatar_url (in client scopes click on attribute and change the consent screen text)

### Roles
Roles identify a type or category of user. Admin, user, manager, and employee are all typical roles that may exist in an organization. Applications often assign access and permissions to specific roles rather than individual users as dealing with users can be too fine grained and hard to manage.

8. Concept of Roles
    * Go to roles
    * Create 2 new roles ( user and admin)
    * Add roles to the user
    * Go to client and change the roles inside the scope
    * If we assign a role any microservice that require the admin role cannot be invoked from this application regardless if a user with those roles logs in or not.
    * Refresh token in application and check access_token.
    * Assign admin role and refresh
    * Remove admin role in user and refresh
    * It will only display the user role

### Groups
Groups manage groups of users. Attributes can be defined for a group. You can map roles to a group as well. Users that become members of a group inherit the attributes and role mappings that group defines.
Each group can have childgroups, attributes or roles.

9.Concept of Groups
     * Create a new group
     * Add an attribute user_type; value: regular_user
     * Go to role mappings and assign the user role to the group
     * Go to the user and unassign the user role and add the user to the group
     * Create another client scope name: myscope
     * Add a mapper ( group membership) with names to the previously created scope
     * Add another mapper type with user attribute and name user_type to the previously created scope
     * Go to the client and give access to the client scope.
### Ldap server
10. Import users from ldap server
     * Go to user federation
     * Add ldap provider
     * Make it writable(if the user wants to change some attributes eg. last name; keycloak will write back to the ladp server with this change)
     * Choose Other vendor
     * Connection URL : ldap://demo-ldap:389
     * Users DN: ou=People,dc=example,dc=org
     * Bind DN: cd=admin,dc=example,dc=org
     * Bind credential: admin
### Identity Providers
11. Arrange login with github
     * Go to Identity Providers and select GitHub
     * Navigate to https://github.com/settings/applications/new
     * Fill in the name and the homepage url(htttp://localhost:8080)
     * For the authorization callback URL we need the redirect URI from the admin console.
     * Click register applicatio; take the client id and client secret and fill in the form from keycloack admin console
     * Once this is done you can logout of the application and try to login using github

12. Change of themes, you can change the look and feel of the keycloak application to match your application. 
 
### Events

Keycloack supports events, such as a bad password input or new token generated there is an event generated in the system. You can build a custom event listener for certain events or you can store them for a defined time so it doesn`t fill the database.

    * Go to events->config
    * Save events on and save 
    * enter the wrong password upon login into the application and check if events appeared.

### Authentication
In Keycloak you can change the flow of an authentication.
    * Go to authentication
    * Click on copy on top right
    * Delete Username Password form
    * Delete Conditional OTP
    * Add new execution - Magic Link
    * Make Magic Link required
    * Go to Bindings and Change to Copy of Browser
    * Logout of application and fill in email ( XXXX@localhost will redirect the emails to MailHog which listens to port 8025)
 

# Questions and Answers
* Where does Keycloak store users?
  * By default, Keycloak will import users from LDAP into the local Keycloak user database. This copy of the user is either synchronized on demand, or through a periodic background task. The one exception to this is passwords. Passwords are not imported and password validation is delegated to the LDAP server. The benefits to this approach is that all Keycloak features will work as any extra per-user data that is needed can be stored locally. This approach also reduces load on the LDAP server as uncached users are loaded from the Keycloak database the 2nd time they are accessed. The only load your LDAP server will have is password validation. The downside to this approach is that when a user is first queried, this will require a Keycloak database insert. The import will also have to be synchronized with your LDAP server as needed.
* What is an LDAP server?
  * [LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol), Lightweight Directory Access Protocol, is an Internet protocol that email and other programs use to look up information from a server.

