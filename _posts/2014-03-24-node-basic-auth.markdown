---
layout: post
title: node-basic-auth
category: posts
---

In working with Angular, and Express with Node.js one of the problems I ran into was how to use basic authentication.  Using angular to keep a cookie of the basic auth hash and then verifying that hash against protected resources on a Node.js server.

I decided to use [passport](passport) as it already has support for things like basic auth and digest.  However one of the problems with [passport-http](passport-http) is that if a user
recieves a 401 (Unauthorized) error by failing to login successfully it will return that user a WWW-Authenticate header.  This isn't a problem until you notice that browsers respond to this header with a browser login popup.  This is a problem if you want your users to use your login page rather than the browsers.

I modified the [passport-http](passport-http) in my fork to allow the WWW-Authenticate header to be disabled by using the flag `disableBasicChallenge`. Setting this to true will prevent the header from being returned to the user and prevent the browser based popup.

The changes have not been merged in as of this writing so I set up the following node package it is available in npm via running `$ npm install gt-passport-http` or add it to `package.json` as `"gt-passport-http": "0.2.2"` and the code is available [here](gt-passport-http).

In order to setup the server side you just have to declare passport and set up a basic strategy object.

**app.js:**
```javascript
	var passport = require('passport');
	var BasicStrategy = require('gt-passport-http').BasicStrategy;
	passport.use(new BasicStrategy({ disableBasicChallenge: true },
	  function (userid, password, done) {
	    var user = users[userid];
	    if (!user) {
	      return done(null, false);
	    }
	    if (password != user.password) {
	      return done(null, false);
	    }
	    return done(null, user);
	  }
	));
```

This handles all authenticated requests. In order to protect a resource you need to add a passport authenticate call onto your route.

**app.js:**
```javascript
app.get('/api/v1/resources',
  passport.authenticate('basic', { session: false }),
  resources.list
);
```

That takes care of the server side, now there are pieces needed on the client side in order to request that resource with basic authentication.

First the angular app needs an interceptor that intercepts the 401 errors as well as checking regular requests.

**public/javascripts/app.js:**
```javascript
	app.config(function ($httpProvider) {
	  $httpProvider.interceptors.push(
	    function ($rootScope, $location, $cookieStore, $q) {
	
	      return {
	        'request': function (request) {
	          $rootScope.currentUser = $cookieStore.get('authdata');
	          if (!$rootScope.currentUser && $location.path() != '/login') {
	            $location.path('/login');
	          }
	          return request;
	        },
	        'responseError': function (rejection) {
	          if (rejection.status === 401 && $location.path() != '/login') {
	            if ($rootScope.currentUser) {
	              $cookieStore.remove('authdata');
	              $rootScope.currentUser = '';
	              $rootScope.loginError = 'Error 401 Invalid username/password';
	            }
	            $location.path('/login');
	          }
	          return $q.reject(rejection);
	        }
	      };
	    });
	});
```

After that there needs to be an authentication service that angular uses to set the basic authentication header and call login and logout from a controller.

**public/javascripts/services.js:**
```javascript
api.factory('Auth', ['Base64', '$cookieStore', '$http',
  function (Base64, $cookieStore, $http) {
    $http.defaults.headers.common['Authorization'] =
      'Basic ' + $cookieStore.get('authdata');
    return {
      login: function (user) {
        var encoded = Base64.encode(user.username + ':' + user.password);
        $http.defaults.headers.common.Authorization = 'Basic ' + encoded;
        $cookieStore.put('authdata', encoded);
      },
      logout: function () {
        document.execCommand("ClearAuthenticationCache");
        $cookieStore.remove('authdata');
        $http.defaults.headers.common.Authorization = 'Basic ';
      }
    };
  }]);
```

Then in the angular controller you can call login and logout.  Login is used from a login.html partial that sends along the user information which is used from the scope to pass to the Auth service that was setup previously.

**public/javascripts/controllers.js:**
```javascript
$scope.loginUser = function () {
    // Do basic authentication
    Auth.login($scope.user);
    $rootScope.currentUser = $scope.user.username;
    $location.path('resources');
  };
  $scope.logoutUser = function () {
    Auth.logout();
    $location.path('login');
  }
```

Thats the basics of it, the source code for the example I created is located [here](node-basic-auth).

[node-basic-auth]: https://github.com/geothird/node-basic-auth
[gt-passport-http]: https://github.com/geothird/passport-http
[passport-http]: https://github.com/jaredhanson/passport-http
[passport]: https://github.com/jaredhanson/passport