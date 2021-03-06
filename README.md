# Heimdall

![Heimdall logo](logo.jpg)

Heimdall is an [OAuth 2.0](https://tools.ietf.org/html/rfc6749) client specifically designed for easy usage and high flexibility. It supports all grants as described in [Section 4](https://tools.ietf.org/html/rfc6749#section-4) as well as refreshing an access token as described in [Section 6](https://tools.ietf.org/html/rfc6749#section-6).

This library makes use of [RxAndroid](https://github.com/ReactiveX/RxAndroid). Therefore you should be familar with [Observables](https://github.com/ReactiveX/RxJava/wiki/Observable).

## Example

Heimdall's main class is the `OAuth2AccessTokenManager`. It is responsible for retrieving a new access token and keeping it valid by refreshing it.

In order to initialize an `OAuth2AccessTokenManager` instance, you need to pass an object implementing the `OAuth2AccessTokenStorage` interface. You can use the predefined `SharedPreferencesOAuth2AccessTokenStorage` if it suits your needs. Make sure that your `OAuth2AccessTokenStorage` is as secure as possible!

```java 

SharedPreferencesOAuth2AccessTokenStorage<OAuth2AccessToken> storage = new SharedPreferencesOAuth2AccessTokenStorage<>(mySharedPreferences, OAuth2AccessToken.class);
OAuth2AccessTokenManager<> manager = new OAuth2AccessTokenManager<OAuth2AccessToken>(storage);

```

On your manager instance you can now call `grantNewAccessToken(grant)` to receive a new access token. The grant instance you pass must implement the `OAuth2Grant` interface and your actual server call. 

Here is an example of an `OAuth2ResourceOwnerPasswordCredentialsGrant`.

```java 
public class MyOAuth2Grant extends OAuth2ResourceOwnerPasswordCredentialsGrant<OAuth2AccessToken> {

    // Constructor

    @Override
    public Observable<OAuth2AccessToken> grantNewAccessToken() {
        // Create the network request based on the username, the password and the grant type.
        // You can use Retrofit to make things easier.
    }
}
```

Your manager instance also has a method called `getValidAccessToken(refreshGrant)`. This is probably the main reason we build this library. It firstly checks if the stored access token is expired and then either emits the unexpired one or refreshs it if it is expired using the passed refresh grant. 

Here is an example of an `OAuth2RefreshAccessTokenGrant`.

```java
public class MyOAuth2Grant extends OAuth2RefreshAccessTokenGrant<OAuth2AccessToken> {

    // Constructor

    @Override
    public Observable<OAuth2AccessToken> grantNewAccessToken() {
        // Create the network request based on the grant type and the refresh token.
        // You can use Retrofit to make things easier.
    }
}
```

Mostly you will use the `OAuth2AuthorizationCodeGrant` to authorize the user via a third party service such as Trakt.tv.

The implemention of a grant authorizing with Trakt.tv might look as following:

```java
public final class TraktTVAuthorizationCodeGrant extends OAuth2AuthorizationCodeGrant<OAuth2AccessToken> {

    public String clientSecret;

    @Override
    public Uri buildAuthorizationUri() {
        return Uri.parse("https://trakt.tv/oauth/authorize")
                .buildUpon()
                .appendQueryParameter("client_id", clientId)
                .appendQueryParameter("redirect_uri", redirectUri)
                .appendQueryParameter("response_type", RESPONSE_TYPE).build();
    }

    @Override
    public Observable<OAuth2AccessToken> exchangeTokenForCode(String code) {
        // Create the network request based on the grant type, clientSecret and the retrieved code.
        // You can use Retrofit to make things easier.
    }
}
```

Using that grant with an Android WebView might look like this (please note that we use [Retrolambda](https://github.com/evant/gradle-retrolambda) here):

```java
// Create the grant
TraktTVAuthorizationCodeGrant grant = new TraktTVAuthorizationCodeGrant();
grant.clientSecret = "secret"
grant.clientId = "id"
grant.redirectUri = "uri"

// Set up web view loading
webView.setWebViewClient(new WebViewClient() {
 	
 	@Override
    public void onPageFinished(WebView view, String url) {
    	super.onPageFinished(view, url);

		// Tell the grant we loaded an url
        grant.onUrlLoadedCommand.onNext(Uri.parse(url));
    }
});

// Load the authorization url once build
grant.authorizationUri()
    .map(Uri::parse)
	.observeOn(AndroidSchedulers.mainThread())
	.subscribe(myWebView::load)

// Start the authorization process
grant.grantNewAccessToken()
	.subscrive(token -> Log.d("Heimdall", "New token: " + token))

```

## Installation

// TBD

## About

Heimdall was built by [Rheinfabrik](http://www.rheinfabrik.de) :factory:

## License

Heimdall is licensed under Apache Version 2.0.
