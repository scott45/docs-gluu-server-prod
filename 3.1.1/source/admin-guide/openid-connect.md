# OpenID Connect Provider (OP)
The Gluu Server's oxAuth software passes all [OpenID Provider conformance profiles](http://openid.net/certification/) and supports the following OpenID Connect specifications: Core, Dynamic Client Registration, Discovery, Form Post Response Mode, Session Management, and the draft for Front Channel Logout.

!!! Note
    oxAuth is a required component in all Gluu Server deployments.   

## Protocol Overview
OpenID Connect is an identity layer that profiles and extends OAuth 2.0. 
It defines a sign-in flow that enables an application (client) to 
authenticate a person, and to obtain authorization to obtain 
information (or "claims") about that person. For more information, 
see [http://openid.net/connect](http://openid.net/connect)

It's handy to know some OpenID Connect terminology:

- The *end user* or *subject* is the person being authenticated.

- The *OpenID Provider* or *OP* is the equivalent of the SAML IDP. It 
holds the credentials (like a username/ password) and information about 
the subject. The Gluu Server is an OP.

- The *Relying Party* or  *RP*  or *client* is software, like a mobile application 
or website, which needs to authenticate the subject. The RP is an OAuth 
client. 

!!! Note
    To learn more about the differences between OAuth, SAML and OpenID Connect, read [this blog](http://gluu.co/oauth-saml-openid).

## OpenID Connect APIs

Review the Gluu Server's OpenID Connect API endpoints in the [API Guide](../api-guide/openid-connect-api.md). 

## OpenID Connect Flows

The Gluu Server supports all flows defined in the [OpenID Connect Core spec](http://openid.net/specs/openid-connect-core-1_0.html), including implicit, authorization code, and hybrid flows. 

### Implicit Flow
The implicit flow, where the token and id_token are returned from the authorization endpoint, should only 
be used for applications that run in the browser, like a Javascript 
client. 

### Code / Hybrid Flow
The code flow or hybrid flow should be used for server side
applications, where code on the web server can more securely call
the token endpoint to obtain a token. 

The most useful response type for the hybrid flow is `code id_token`. Using this flow, you can verify
the integrity of the code by inspecting the `c_hash` claim in the `id_token`.

If you are using the code flow, the response type should only be code.
There is no point in using response type `code token id_token`--the extra
tokens returned by the authorization endpoint will only create additional
calls to the LDAP server and slow you down. 

If you are going to trade the code at the token endpoint for a new token and id_token, you don't
need them from the authorization endpoint too.

### Flow Comparison Chart
|Step   |  Authorization code flow 	| Implicit flow	 | Hybrid flow |
|----------|----------|-------------------------------|----------------|
|1|User accesses an application.|User accesses an application.|User accesses an application.|
|2|The application/relaying party (RP) prepares an authentication request containing the desired request parameters and sends it to the OpenID Provider (Gluu Server). The `response_type requested` is `code`.|The application/replaying party (RP) prepares an authentication request containing the desired request parameters and sends it to the OpenID Provider (Gluu Server). The `response_type requested` is `id_token` or `id_token token`.|The application/replaying party (RP) prepares an authentication request containing the desired request parameters and sends it to the OpenID Provider (Gluu Server). The `response_type` requested is `code id_token`, `code token`, or `code id_token token`.|
|3|The OpenID Provider (Gluu Server) verifies the user’s identity and authenticates the user.|The OpenID Provider (Gluu Server) verifies the user’s identity and authenticates the user.|The OpenID Provider (Gluu Server) verifies the user’s identity and authenticates the user.|
|4|The OpenID Provider (Gluu Server) sends the user back to the application with an authorization code.|The OpenID Provider (Gluu Server) sends the user back to the application with an ID Token (`id_token` or `id_token token`) and an Access Token (`token`).|The OpenID Provider (Gluu Server) sends the user back to the application with an authorizatio code (`code id_token`, `code token`, or `code id_token token`) and an Access Token (`token`).|
|5|The application sends the code to the Token Endpoint to receive an Access Token and ID Token in the response.|The application uses the ID Token to authorize the user. At this point the application/RP can access the `UserInfo endpoint` for claims.|The application sends the code to the Token Endpoint to receive an Access Token and ID Token in the response.|
|6|The application uses the ID Token to authorize the user. At this point the application/RP can access the `UserInfo endpoint` for claims.||The application uses the ID Token to authorize the user. At this point the application/RP can access the `UserInfo endpoint` for claims.|
 

## Configuration / Discovery 

A good place to start when you're learning about OpenID Connect is the configuration endpoint, which is located in the Gluu Server
at the following URL: `https://{hostname}/.well-known/openid-configuration`.

The Gluu Server also supports [WebFinger](http://en.wikipedia.org/wiki/WebFinger), as specified in the [OpenID Connect discovery specification](http://openid.net/specs/openid-connect-discovery-1_0-21.html). 

## Client Registration / Configuration

OAuth clients need a client_id, and need to supply a login redirect uri--
where the Authorization Server should redirect the end user to, post
authorization. The Gluu Server enables an administrator to manually create
a client via the oxTrust web interface. However, OpenID Connect also
defines a standard API where clients can register themselves--
[Dynamic Client Registration](http://openid.net/specs/openid-connect-registration-1_0.html). You can
find the registration URL by calling the configuration endpoint 
(`/.well-known/openid-configuration`).        

You may not want clients to dynamically register themselves! To disable this endpoint, in the oxAuth JSON properties, set the 
`dynamicRegistrationEnabled` value to False.                 

If you want to add a client through oxTrust, you can use the manual form:
by click the `Add Client` button.            

![add-client](../img/openid/add-client311.png)

![add-client1](../img/openid/add-client1-311.png)

![add-client2](../img/openid/add-client2-311.png)

![add-client3](../img/openid/add-client3-311.png)

![add-client4](../img/openid/add-client4-311.png)

There are many client configuration parameters. Most of these are specified in the OpenID Connect [Dynamic Client Registration](http://openid.net/specs/openid-connect-registration-1_0.html) specification.

There are two configurations params which can only be configured via oxTrust by an administrator. These include:

 - Pre-Authorization -- Use this if you want to suppress the end user
 authorization prompt. This is handy for SSO scenarios where the clients
 are your own (not third party), and there is no need to prompt the 
 person to approve the release of information.      
 
 - Persist Client Authorizations -- Use this option if you only want 
 to prompt the end user once to authorize the release of user 
 information. It will cause the data to be persisted under the person's
 entry in the Gluu LDAP server.                

## Custom Client Registration

Using the Client Registration custom interception scripts,
you can implement post-registration business logic. You have access to 
the data that the client used to register. You could validate data, 
populate extra client claim, or modify the scope registrations. You
could even call API's to determine if you want to allow the 
registration at all. To access the interface for custom scripts in 
oxTrust, navigate to Configuration --> Custom Scripts --> Client Registration.

![custom-client](../img/openid/custom-client.png)           

The script is [available here](./sample-client-registration-script.py)     

## Scopes

In OAuth, scopes are used to specify extents of access. For a sign-in 
flow like OpenID Connect, scopes end up corresponding to the release of
user claims. The Gluu Server supports the 
[standard scopes](http://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims) defined 
in the OpenID Connect specification. You can also define your own scopes,
and map them to any user attributes which you have registered. 

To add Scope and Claims in OpenID Connect

1. Click on `Configuration` > `OpenID Connect`           
     
    ![menu](../img/openid/clientmenu.png)

2. Click on Add scope on the screen to the right            
    ![scopeadd](../img/openid/admin_oauth2_scope.png)
3. You will presented the screen below to the enter the Scope Details                      
    
    ![scopedetails](../img/openid/add-scope1.png)

4. To add more claims, simply click "Add Claim" and you will be presented
with the following screen:                     

    ![Add Claims](../img/openid/add-scope-claim.png)

| Field | Description |
|---| ---|
| Display Name | Name of the scope which will be displayed when searched |
| Description | Text that will be displayed to the end user during approval of the scope |
| Scope Type | OpenID, Dynamic or OAuth  |
| Default Scope | If True, the scope may be added to clients' registrations created via Dynamic Client Registration protocol |

Scope Type "OpenID" specifies to the Gluu Server that this scope will
be used to map user claims; "Dynamic" specifies to the Gluu Server that
the scope values will be generated from the result of the Dynamic Scopes 
custom interception script; "OAuth" specifies that the scope will have
no claims, it will be meaningful to an external resource server. 

Specifying a scope as "Default" means that any OIDC client using Dynamic 
Client Registration protocol is allowed to enlist it amongst scopes 
that will be requested by RP(s) the client represents. As this may result in  
sensitive users' data being leaked to unauthorized parties, thorough assessment 
of all claims which belong to scopes about to be marked as "Default" is advised.
Right after the installation, the only default scope is `openid`, 
which is required by the OpenID Connect specification. Gluu server's 
administrator can always explicitly add additional scopes some client is allowed 
to request by editing its registration metadata manually in web UI later on.

## Authentication

The OpenID Connect `acr_values` parameter is used to specify a workflow for authentication. The value of this parameter, or the `default_acr_values` client metadata value, corresponds to the "Name" of a custom authentication script in the Gluu Server.

The default distribution of the Gluu Server includes custom authentication scripts with the following `acr` values: 

|  ACR Value  	| Description			|
|---------------|-------------------------------|
|  u2f		| [FIDO U2F Device](../authn-guide/U2F.md)|
|  super_gluu	| [Multi-factor authentication](../authn-guide/supergluu.md)|
|  duo		| [Duo soft-token authentication](../authn-guide/duo.md)|
|  cert	| [Smart card or web browser X509 personal certificates](../authn-guide/cert-auth/)|
|  cas	| External CAS server|
|  gplus	| [Google+ authentication](../authn-guide/google.md)|
|  OTP	| [OATH one time password](../authn-guide/otp.md) |
|  asimba	| Use of the Asimba proxy for inbound SAML |
|  twilio_sms	| Use of the Twilio Saas to send SMS one time passwords |
|  passport	| Use of the [Passport component for social login](../ce/authn-guide/passport.md) |
|  yubicloud	| Yubico cloud OTP verification service |
|  uaf	| experimental support for the FIDO UAF protocol |
|  basic_lock	| [Enables lockout after a certain number of failures](../authn-guide/intro.md/#configuring-account-lockout) |
|  basic	| [Sample script using local LDAP authentication](../ce/authn-guide/basic.md) |

Your client can request any authentication mechanism that is enabled in your Gluu Server. To enable an authentication script, login to your Gluu Server admin interface, navigate to Configuration > Manage Custom Scripts, find the desired script, check the `Enabled` box, scroll to the bottom of the page and click `Update`. 

Learn more in the [authentication guide](../authn-guide/intro.md).  

## Logout

The OpenID Connect [Session Management](http://openid.net/specs/openid-connect-session-1_0.html) specification is still marked as draft, and new mechanisms for logout are in the works. The current specification requires JavaScript to detect that the session has been ended in the browser. It works... unless the tab with the JavaScript happens to be closed when the logout event happens on another tab. Also, inserting JavaScript into every page is not feasible for some applications. 

The Gluu Server also support the draft for [Front Channel Logout](http://openid.net/specs/openid-connect-frontchannel-1_0.html). This
is our recommended logout strategy. Using this mechanism, an html page is rendered which contains one iFrame for each application that 
needs to be notified of a logout. The Gluu Server keeps track of which clients are associated with a session (i.e. your browser). This 
mechanism is not perfect. If the end user's web browser is blocking third party cookies, it may break front channel logout. Also, the Gluu Server has no record if the logout is successful--only the browser knows. This means that if the logout fails, it will not be logged or retried. The good thing about front channel logout is that the application can clear application cookies in the end user's browser. To use front channel logout, the client should register logout_uri's, or `frontchannel_logout_uri` for clients using the Dynamic Client Registration API. 

## Disable OpenID Connect Client registration entry
Gluu Server 3.1.1 provides you an option to disable specific OpenID Connect client's registration entry instead of deleting it completely.

To achieve this,

1. Navigate to `OpenID Connect` > `Clients`

2. Find registration entry of the client in question and click it

3. Scroll to the end of its settings' list and check the `Disabled` checkbox

4. Click the "Update" button

![disable-openid](../img/openid/openidconnect-disable.png)

## OpenID Connect Relying Party (RP)

In order to leverage your Gluu Server OpenID Provider (OP) for central authentication, web and mobile apps will need to support OpenID Connect. In OpenID Connect jargon, your app will act as an OpenID Connect Relying Party (RP) or "client". 

There are many ways to go about supporting OpenID Connect in your apps. When possible, it is best to use existing client software implementations that have been verified to implement OpenID Connect properly (and securely!). A good OpenID Connect client will do much of the heavy lifting for you. 

Review the [SSO integration guide](../integration/index.md) to determine which software and strategy will be best for your target application(s). 

### oxAuth RP

The Gluu Server ships with an optional OpenID Connect RP web application called oxauth-rp which is handy for testing. During Gluu Server setup you'll be asked if you want to install the oxauth-rp (it should be installed on a development environment). If you decide to install it, oxAuth RP will be deployed on `https://<hostname>/oxauth-rp`. 

Using the oxAuth RP you can exercise all of the OpenID Connect API's, including discovery, client registration, authorization, token, 
userinfo, and end_session. 
