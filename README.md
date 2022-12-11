node-deezer
===========

Unofficial Node.js wrapper for the Deezer API

## Installation
First, install the npm module:

`npm install node-deezer`


## Create your app on developers.deezer.com

First, make sure you have a Deezer account.  Then go to http://developers.deezer.com/myapps and create a new application.

Grab your "Application ID" and "Secret Key"-- you'll need them below.


## Hello Deezer

Here's a quick example to get you started:

> For more, clone this repo and try out the [Express example](https://github.com/mikermcneil/node-deezer/tree/master/examples/with_express).

```javascript
// Instantiate library (can be global or instantiated in separate places-- either way is fine)
var DZ = require('node-deezer');
var deezer = new DZ();

// Now use node-deezer to generate the the link where you can redirect
// your users to allow your app to access her/his Deezer account
var appId = '28325'; // from developers.deezer.com
var appSecret = 'd92aeb22424492c2828b'; // from developers.deezer.com
var redirectUrl = 'http://localhost:3000/deezerCallback'; // somewhere in your app, see below
var loginUrl = deezer.getLoginUrl(appId, redirectUrl);

// ...
// ...
// ...
// When the user has approved your app, they're sent to the redirectUrl above
// which you might handle like so:

// `code` should have been handed back by Deezer as a parameter
// if it was not, an error occurred, and we must handle it here
var code = req.param('code');

if (!code) {
  // Handle an error if one happened (see express example for more on this)
  // we'll gloss over it here
  var err = req.param('error_reason');

  // Since we have this code, we can trust that the user 
  // actually meant to grant us access to their account.
	
  // Now we need to combine this code with our app credentials 
  // to prove to Deezer that we're a valid app-- if everything works,
  // we'll get an `access_token` we can use to represent the user
  // for a period of time (or "forever" if we have the offline_access permission)
  deezer.createSession(appId, appSecret, code, function (err, result) {
		
    // If an error occurs, we should handle it
    // (again, see express example for more)
  
    // Otherwise, everything is cool and we have the access token and lifespan (`expires`)
    // in `result.accessToken` and `result.expires`
    console.log(result);
    
    // Now we can do API requests!
    
    // e.g. search for artists with names containing the phrase 'empire'
    deezer.request(result.accessToken,
    {
      resource: 'search/artist',
      method: 'get',
      fields: { q: 'empire' }
    },
    function done (err, results) {
      if (err) throw err;
      console.log(results);
    });
    
  });
}

```



## Methods

#### getLoginUrl

```javascript
	/**
	 * Get the authentication url where your user should be redirected
	 *
	 * @param {String} appId			(your Deezer application id from the Deezer developer portal)
	 * @param {String} redirectUrl		(URL which will handle the user's code from Deezer)
	 * @param {Array|undefined} perms	(requested permissions [optional])
	 *
	 * NOTE: `redirectUrl` must be within the 'Application domain' specified for this app 
	 * in your Deezer developer portal at: http://developers.deezer.com/myapps
	 */
	deezer.getLoginUrl(appId, redirectUrl, [permissions]);
```


#### createSession

```javascript
	/**
	 * Generate an access token to access a user's Deezer account
	 *
	 * > NOTE: You must first have a valid `code` from the user proving that they're OK with this!!
	 * > You can get a code by redirecting the user to the url generated by calling `DZ.getLoginUrl(appId, callbackUrl)`
	 * > You'll probably want to call `DZ.createSession()` from the handler for the `callbackUrl` you specified
	 * > in `getLoginUrl(appId, callbackUrl)`, since that's where you'll have access to the `code`
	 *
	 * @param {String} appId		(your Deezer application id from the Deezer developer portal)
	 * @param {String} code			(the OAuth `code` generated by Deezer and sent to the `callbackUrl`)
	 * @param {String} secret		(your Deezer app's secret from the developer portal)
	 * @param {Function} cb
	 *		@param {Error|null} err
	 */
	deezer.createSession(appId, appSecret, code, cb);
```

#### request

```javascript
	/**
	 * Send an arbitrary API request to Deezer
	 *
	 * @param {String} accessToken			(the OAuth token representing a user's session)
	 * @param {Object} options
	 *		resource {String}				(the string name of resource, e.g. 'album')
	 *		method {HTTPMethod|undefined}	(the REST method [defaults to 'get'])
	 * @param {Function} cb
	 *		@param {Error|null} err
	 */
	deezer.request(accessToken, options, cb);
```

For a list of available API methods, check out http://developers.deezer.com/api



## To run the Express example

(1) Create your app on developers.deezer.com
  + Set application domain to `localhost` for now, and make sure and copy down your `Application ID` and `Secret Key`

(2) Copy `deezerCredentials.ex.js` to `deezerCredentials.js`

(3) Then change the app secret and app id in `deezerCredentials.js` to your app's credentials from developers.deezer.com

(4) `npm install`

(5) `node app.js`




## To run tests
`npm test`



## License

The MIT License (MIT)

Copyright (c) 2013 Mike McNeil

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

https://github.com/mikermcneil/node-deezer/blob/master/LICENSE




<!--
## How To Build a Deezer App

+ Create your app on developers.deezer.com
	+ Set application domain to `localhost` for now
	+ Grab the `Application ID` and `Secret Key`

+ 2) Build your Deezer login flow
	+ You must pop up an OAuth window (or redirect, or use an iframe) to acquire an access token for the user whose account your app will access
	+ The `callback url` you specify on developers.deezer.com will be accessed from Deezer's end when the login is complete.

```
// OAuth endpoint:
https://connect.deezer.com/oauth/auth.php?app_id=YOUR_APP_ID&redirect_uri=YOUR_REDIRECT_URI&perms=basic_access,email
```
-->
