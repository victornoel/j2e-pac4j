## What is the j2e-pac4j library ?

The **j2e-pac4j** library is a J2E multi-protocols client.

It supports these 4 protocols on client side :

1. OAuth (1.0 & 2.0)
2. CAS (1.0, 2.0, SAML, logout & proxy)
3. HTTP (form & basic auth authentications)
4. OpenID.

It's available under the Apache 2 license and based on my [pac4j](https://github.com/leleuj/pac4j) library.


## Providers supported

<table>
<tr><th>Provider</th><th>Protocol</th><th>Maven dependency</th><th>Client class</th><th>Profile class</th></tr>
<tr><td>CAS server</td><td>CAS</td><td>pac4j-cas</td><td>CasClient & CasProxyReceptor</td><td>CasProfile</td></tr>
<tr><td>CAS server using OAuth Wrapper</td><td>OAuth 2.0</td><td>pac4j-oauth</td><td>CasOAuthWrapperClient</td><td>CasOAuthWrapperProfile</td></tr>
<tr><td>DropBox</td><td>OAuth 1.0</td><td>pac4j-oauth</td><td>DropBoxClient</td><td>DropBoxProfile</td></tr>
<tr><td>Facebook</td><td>OAuth 2.0</td><td>pac4j-oauth</td><td>FacebookClient</td><td>FacebookProfile</td></tr>
<tr><td>GitHub</td><td>OAuth 2.0</td><td>pac4j-oauth</td><td>GitHubClient</td><td>GitHubProfile</td></tr>
<tr><td>Google</td><td>OAuth 2.0</td><td>pac4j-oauth</td><td>Google2Client</td><td>Google2Profile</td></tr>
<tr><td>LinkedIn</td><td>OAuth 1.0</td><td>pac4j-oauth</td><td>LinkedInClient</td><td>LinkedInProfile</td></tr>
<tr><td>Twitter</td><td>OAuth 1.0</td><td>pac4j-oauth</td><td>TwitterClient</td><td>TwitterProfile</td></tr>
<tr><td>Windows Live</td><td>OAuth 2.0</td><td>pac4j-oauth</td><td>WindowsLiveClient</td><td>WindowsLiveProfile</td></tr>
<tr><td>WordPress</td><td>OAuth 2.0</td><td>pac4j-oauth</td><td>WordPressClient</td><td>WordPressProfile</td></tr>
<tr><td>Yahoo</td><td>OAuth 1.0</td><td>pac4j-oauth</td><td>YahooClient</td><td>YahooProfile</td></tr>
<tr><td>Web sites with basic auth authentication</td><td>HTTP</td><td>pac4j-http</td><td>BasicAuthClient</td><td>HttpProfile</td></tr>
<tr><td>Web sites with form authentication</td><td>HTTP</td><td>pac4j-http</td><td>FormClient</td><td>HttpProfile</td></tr>
<tr><td>MyOpenId</td><td>OpenID</td><td>pac4j-openid</td><td>MyOpenIdClient</td><td>MyOpenIdProfile</td></tr>
</table>


## Technical description

This library has **only 6 classes** :

1. the **ClientConfiguration** class gathers all the clients configuration
2. the **ClientFactory** is the interface to implement to define the clients
3. the **ClientsConfigFilter** is an abstract J2E filter in charge of loading clients configuration
4. the **RequiresAuthenticationFilter** is a J2E filter to protect urls and requires authentication for them (redirection to the appropriate provider)
5. the **CallbackFilter** is a J2E filter to handle the callback of the provider after authentication to finish the authentication process
6. the **UserUtils** is an helper class to know if the user is authenticated, his profile and log out him.

and is based on the <i>pac4j-*</i> libraries.

Learn more by browsing the [j2e-pac4j Javadoc](http://www.pac4j.org/apidocs/j2e-pac4j/index.html) and the [pac4j Javadoc](http://www.pac4j.org/apidocs/pac4j/index.html).


## How to use it ?

### Add the required dependencies

If you want to use a specific client support, you need to add the appropriate Maven dependency in the *pom.xml* file :

* for OAuth support, the *pac4j-oauth* dependency is required
* for CAS support, the *pac4j-cas* dependency is required
* for HTTP support, the *pac4j-http* dependency is required
* for OpenID support, the *pac4j-openid* dependency is required.

For example, to add OAuth support, add the following XML snippet :

    <dependency>
      <groupId>org.pac4j</groupId>
      <artifactId>pac4j-oauth</artifactId>
      <version>1.4.1-SNAPSHOT</version>
    </dependency>

As these snapshot dependencies are only available in the [Sonatype snapshots repository](https://oss.sonatype.org/content/repositories/snapshots/org/pac4j/), the appropriate repository must be added in the *pom.xml* file also :

    <repositories>
      <repository>
        <id>sonatype-nexus-snapshots</id>
        <name>Sonatype Nexus Snapshots</name>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        <releases>
          <enabled>false</enabled>
        </releases>
        <snapshots>
          <enabled>true</enabled>
        </snapshots>
      </repository>
    </repositories>

### Define the clients

All the clients used to communicate with various providers (Facebook, Twitter, a CAS server...) must be defined in a specific class implementing the *org.pac4j.j2e.configuration.ClientsFactory* interface. For example :

    public class MyClientsFactory implements ClientsFactory {
      
      @Override
      public Clients build() {
        final FacebookClient facebookClient = new FacebookClient("fbkey", "fbsecret");
        final TwitterClient twitterClient = new TwitterClient("twkey", "twsecret");
        // HTTP
        final FormClient formClient = new FormClient("http://localhost:8080/theForm.jsp", new SimpleTestUsernamePasswordAuthenticator());
        final BasicAuthClient basicAuthClient = new BasicAuthClient(new SimpleTestUsernamePasswordAuthenticator());        
        // CAS
        final CasClient casClient = new CasClient();
        casClient.setCasLoginUrl("http://localhost:8888/cas/login");        
        // OpenID
        final MyOpenIdClient myOpenIdClient = new MyOpenIdClient();
        final Clients clients = new Clients("http://localhost:8080/callback", facebookClient, twitterClient, formClient, basicAuthClient, casClient, myOpenIdClient);
        return clients;
      }
    }
    
### Define the "callback filter"

To handle callback from providers, you need to define the appropriate J2E filter and its mapping :

    <filter>
      <filter-name>CallbackFilter</filter-name>
      <filter-class>org.pac4j.j2e.filter.CallbackFilter</filter-class>
      <init-param>
      	<param-name>clientsFactory</param-name>
      	<param-value>org.leleuj.config.MyClientsFactory</param-value>
      </init-param>
      <init-param>
      	<param-name>defaultUrl</param-name>
      	<param-value>/</param-value>
      </init-param>
    </filter>
    <filter-mapping>
      <filter-name>CallbackFilter</filter-name>
      <url-pattern>/callback</url-pattern>
      <dispatcher>REQUEST</dispatcher>
    </filter-mapping>

### Protect the urls

You can protect your urls and force the user to be authenticated by a client by using the appropriate filter and mapping. Key parameters are all the clients and the specific client (*clientName*) used by this filter.  
For example, for Facebook :

    <filter>
      <filter-name>FacebookFilter</filter-name>
      <filter-class>org.pac4j.j2e.filter.RequiresAuthenticationFilter</filter-class>
      <init-param>
       	<param-name>clientsFactory</param-name>
       	<param-value>org.leleuj.config.MyClientsFactory</param-value>
      </init-param>
      <init-param>
       	<param-name>clientName</param-name>
       	<param-value>FacebookClient</param-value>
      </init-param>
    </filter>
    <filter-mapping>
      <filter-name>FacebookFilter</filter-name>
      <url-pattern>/facebook/*</url-pattern>
      <dispatcher>REQUEST</dispatcher>
    </filter-mapping>

### Get redirection urls

You can also explicitely compute a redirection url to a provider for authentication by using the *getRedirectionUrl* method and the *ClientsConfiguration* class. For example with Facebook :

    <%
	  WebContext context = new J2EContext(request, response);
	  Clients client = ClientsConfiguration.getClients();
	  FacebookClient fbClient = (FacebookClient) client.findClient("FacebookClient");
	  String redirectionUrl = Client.getRedirectionUrl(context);
	%>

### Get the user profile

After successful authentication, you can test if the user is authenticated using ```UserUtils.isAuthenticated()``` or get the user profile using ```UserUtils.getUserProfile()```.

The profile returned is a *CommonProfile*, from which you can retrieve the most common properties that all profiles share. 
But you can also cast the user profile to the appropriate profile according to the provider used for authentication.
For example, after a Facebook authentication :
 
    // facebook profile
    FacebookProfile facebookProfile = (FacebookProfile) commonProfile;

Or for all the OAuth 1.0/2.0 profiles, to get the access token :
    
    OAuth10Profile oauthProfile = (OAuth10Profile) commonProfile
    String accessToken = oauthProfile.getAccessToken();
    // or
    String accessToken = facebookProfile.getAccessToken();

### Demo

A demo with Facebook, Twitter, CAS, form authentication, basic auth authentication and myopenid.com providers is available with [j2e-pac4j-demo](https://github.com/leleuj/j2e-pac4j-demo).


## Versions

The current version **1.0.0-SNAPSHOT** is under development...


## Contact

Find me on [LinkedIn](http://www.linkedin.com/in/jleleu) or by email : leleuj@gmail.com