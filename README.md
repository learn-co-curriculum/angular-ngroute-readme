# What is ngRoute?

## Overview

Let's take a look at routing - this is the part of the application which spreads out your app into different pages.

## Objectives

- Describe ngRoute and ng-view
- Write a route using ngRoute
- Write a dynamic route using ngRoute
- Integrate a Controller with ngRoute
- Integrate a template with ngRoute

## ngRoute

`ngRoute` is a module that is provided to us by the Angular team. This is a basic router that allows us to specify different templates and controllers for our views.

You will need to make sure that you include `angular-route.js` in your code and add `ngRoute` to your main module's dependencies!

Now, where do we actually configure our routes?

One neat thing that Angular allows us to do is to "configure" our module. We can call the `.config` function provided to us by our modules, and specify any configuration for that module in there.

```js
angular
	.module('app', ['ngRoute'])
	.config(function () {
		// this is ran as soon as we initiate the module (ng-app)
	});
```

This function, much like our controllers, allows us to dependency inject our services into it. This means that we can inject `$routeProvider` - a service given to us by `ngRoute`, that allows us to configure our views.

To define a route, we call `$routeProvider.when` with two arguments - the URL that we want to configure, and a configuration object. This configuration object can take quite a few different options.

```js
angular
	.module('app', ['ngRoute'])
	.config(function ($routeProvider) {

		$routeProvider
			.when('/user', {
				// configuration object
			});
	});
```

In our configuration object, we can use the following items -

- `template` or `templateUrl` - specifies the template that we use for that route (this works exactly like it would in a directive)
- `controller` - specifies the controller that we use for that route (works the same as when we provide a string as the controller value in a directive)
- `resolve` - an object of resolves that provides the data we need for that route (more on this later)

Let's configure our `/user` route to use `user.html` as a template, and the `UserController` as its controller.

```js
angular
	.module('app', ['ngRoute'])
	.config(function ($routeProvider) {
		$routeProvider
			.when('/user', {
				templateUrl: 'views/user.html',
				controller: 'UserController'
			});
	});
```

Awesome! Now, when we go to `/#/user` in our application, we will see the user view, using the user controller to manage its data. Wait a minute - no we won't.

In order to actually display our routes on the page, we need to tell `ngRoute` where to actually *put* the user view. We can do this using the directive `ng-view`.

In our HTML, we would do something like this:

```html
<div ng-app="app" class="app">
	<div ng-view></div>
</div>
```

The `ng-view` directive will then put whatever route's template as it's contents when we go to that route - awesome!

### Dynamic Views

Being able to go to `/user` is great, but what if we wanted to a view a different user's profile? Well, we would need to specify the ID of that user. But our route isn't setup to receive an ID, and we can't hard code *every* ID into our router. This is where dynamic views come in.

Much like when we used `$resource`, we can put variables into our URLs by using a colon. For instance, if we wanted to be able to go to `/user/1231` and `/user/94343` etc, we'd put `:id` into our URL.

```js
angular
	.module('app', ['ngRoute'])
	.config(function ($routeProvider) {
		$routeProvider
			.when('/user/:id', {
				templateUrl: 'views/user.html',
				controller: 'UserController'
			});
	});
```

This now allows us to go to the URLs we mentioned above. Awesome! But how do we actually know what the user's ID is? We'd need to go and fetch the user data so it's important that we can access what the variable is actually equal to.

This is where another service provided by `ngRoute` comes in - `$routeParams`. We can inject this into our controllers and see what the ID variable is equal to.

```js
function UserController($routeParams) {
	// $routeParams = { id: 1231 };
}

angular
	.module('app')
	.controller('UserController', UserController);
```

Now that we can access all of our route params, we can now go off and fetch the data for that user - simple!

## Resolving Data

Now, we could go and fetch user data in our controller, but that means that the view will load and then we will have a flicker between the view having no user data and the view being populated with data.

Instead, we can use the `resolve` property we spoke about earlier - this allows us to specify a bunch of promises that we want to be resolved *before* our view is rendered.

Say that we have a `UserService` that returns a `$http.get` call to get the user's profile:

```js
function UserService($http) {
	this.getUser = function (id) {
		return $http.get('/user/' + id);
	};
}
```

And we're currently injecting that into our `UserController`, and using `$routeParams` to specify the user's ID.

```js
function UserController($routeParams, UserService) {
	var ctrl = this;

	UserService
		.getUser($routeParams.id)
		.then(function (res) {
			ctrl.user = res.data;
		});
}

angular
	.module('app')
	.controller('UserController', UserController);
```

This will cause the flicker that we mentioned earlier. Instead, we can define our `UserService.getUser` call in the `resolve` property on our route's configuration.

```js
angular
	.module('app', ['ngRoute'])
	.config(function ($routeProvider) {
		$routeProvider
			.when('/user/:id', {
				templateUrl: 'views/user.html',
				controller: 'UserController',
				resolve: {
					user: function ($routeParams, UserService) {
						return UserService.getUser($routeParams.id);
					}
				}
			});
	});
```

Here we're doing very similar things to what we're doing in our controller - injecting `$routeParams` and our `UserService`, but instead we're returning the promise given to use by our `$http.get` call. Once this promise resolves, `ngRoute` will then render our view.

That's great, but now how do we get access to that data? Simple - you notice that we're using the key `user` in our resolve object? We can now inject `user` into our controller, accessing all the data that the resolve gives us.

```js
function UserController(user) {
	var ctrl = this;

	ctrl.user = user.data;
}

angular
	.module('app')
	.controller('UserController', UserController);
```

Simple! No flicker, and we've got dynamic routes working whilst also fetching the relevant data!