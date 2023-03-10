# Routing

## Basic Routing

```php
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function() {
  return 'Hello world';
});
```

#### The Default Route Files

All laravel routes are defined in your route files, which are located in the `routes` directory. These files are automatically loaded by your applications's `App\Providers\RouteServiceProvider`. The `routes/web.php` file defines routes that are for your web interface. These routes are assigned the `web` middleware group, which provides features like `session state` and `CSRF protection`. The routes in `routes/api.php` are stateless and are assigned the `api` middleware group.

Routes defined in the `routes/api.php` file are nested within a route gropu by the `RouteServiceProvider`. Within this group, the `/api` URI prefix is autometically applied so you do not need to manually apply it to every route in the file. You may modify the prefix and other route group options by modifying `RouteServiceProvider` class.

#### Available Router Methods

The router allows you to register routes that respond to any HTTP verb

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may do so using the `match` method. Or, you may even register a route that responds to all HTTP verbs using the `any` method.

```php
Route::match(['get', 'post'], '/', function() {
  //
});

Route::any('/', function() {
  //
});
```

> when defining multiple routes that share the same URI, routes using the `get`, `post`, `put`, `patch`, `delete` and `options` methods should be defined before routes using the `match`, `any` and `redirect` methods.

#### Dependency Injection

You may type-hint any dependencies required by your route in your route's callback signature. The declared dependencies will autometically be resolved and injected into the callback by the Laravel [service container][container].

You may type-hint the `Illuminate\Http\Request` class to have the current HTTP request autometically injected into your route callback:

```php
use Illuminate\Http\Request;

Route::get('/users', function(Request $request) {
  //
});
```

#### CSRF Protection

Any HTML forms pointing to `POST`, `PUT`, `PATCH` or `DELETE` routes that are defined in the `web` routes file should include a CSRF token field. Otherwise the request will be rejected.

```php
<form method="POST" action="/profile">
  @csrf
  ...
</form>
```

### Redirect Routes

If you are defining a route that redirects to another URI, you may use the `Route::redirect` method.

```php
Route::redirect('/here', '/there');
```

By default, `Route::redirect` returns a `302` status code. You may customise the status code using the optional third parameter.

```php
Route::redirect('/here', '/there', 301);
```

Or, you may use the `Route::PermanentRedirect` method to return a `301` status code:

```php
Route::permanentRedirect('/here', '/there');
```

> When using route parameters in redirect routes, the following parameters are reserved by Laravel and cannot be used: `destination` and `status`.

### View Routes

If your route only needs to return a [view][view], you may use the `Route::view` method. The `view` method accepts a URI as its first argument and a view name as its second argument. In addition, you may provide an array of data to pass to the view as an optional third argument:

```php
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name'=>'Taylor']);
```

> When using route parameters in view routes, the following parameters are reserved by laravel and connot be used: `view`, `data`, `status` and `headers`.

### The Route List

The `route:list` Artisan command can easily provide an overview of all of the routes that are defined

```sh
php artisan route:list
```

By default, the route middleware that are assigned to each route will not be displayed in the `route:list` output; however, you can instruct Laravel to display the route middleware by adding the `-v` option to the command.

```sh
php artisan route:list -v
```

You may also instruct Laravel to only show routes that begin with a give URI

```sh
php artisan route:list --path=api
```

In addition, you may instruct Laravel to hide any routes that are defined by third-party packages by providing the `--except-vendor` option when executing the `route:list` command

```sh
php artisan route:list --except-vendor
```

Likewise, you may aslo instruct Laravel to only show routes that are defined by third-party packages by providing the `--only-vendor` option when executing the `route:list` command

```sh
php artisan route:list --only-vendor
```

## Route Parameters

### Required Parameters

Sometimes you will need to capture segments of the URI within your route.

```php
Route::get('/user/{id}', function($id) {
  //
});
```

You may define as many route parameters as required by your route

```php
Route::get('/posts/{post}/comments/{comment}', function($postId, $commentId) {
  //
});
```

> Route parameters are injected into route callbacks / controllers based on their order - the names of the route callback / controller arguments do not matter

#### Parameters & Dependency Inejction

If your route has dependencies that you would like the Laravel service container to autometically inject into your route's callback, you should list your route parameters afeter your dependencies

```php
use Illuminate\Http\Request;

Route::get('/user/{id}', function(Request $request, $id) {
  return 'User '.$id;
});
```

### Optional Parameters

Occasionally you may need to specify a route parameter that may not always be present in the URI. You may do so by placing a `?` mark after the parameter name. Make sure to give the route's corresponding variable a default value

```php
Route::get('/user/{name?}', function($name = null) {
  return $name;
});

Route::get('/user/{name?}', function($name= 'John') {
  return $name;
});
```

### Regular Expression Constraints

You may constrain the format of your route parameters using the `where` method on a route instance. The `where` method accepts the name of the parameter and a regular expression defining how the parameter sould be contrained.

```php
Route::get('/user/{name}', function($name) {
  //
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function($id) {
  //
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function($id, $name) {
  //
})->where(['id'=>'[0-9]+', 'name'=>'[a-z]+']);
```

For convenience, some commonly used regular expression patterns have helper methods that allow you to quickly add pattern constraints to your routes

```php
Route::get('/user/{id}/{name}', function($id, $name) {
  //
})->whereNumber('id')->whereAlpha('name');

Route::get('/user/{name}', function($name) {
  //
})->whereAlphaNumeric('name');

Route::get('/user/{id}', function($id) {
  //
})->whereUuid('id');

Route::get('/user/{id}', function($id) {
  //
})->whereUlid('id');

Route::get('/category/{category}', function($category) {
  //
})->whereIn('category', ['movie', 'song', 'painting']);
```

If the incoming request does not match the route pattern constraints, a 404 HTTP response will be returned.

#### Global Constraints

If you would like a route parameter to always be constrained by a given regular expression, you may use the `pattern` method. You should define these patterns in the `boot` method of your `RouteServiceProvider` class

```php
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
  Route::pattern('id', '[0-9]+');
}
```

Once the pattern has been defined, it is autometically applied to all routes using that parameter name

```php
Route::get('/user/{id}', function($id) {
  //
});
```

#### Encoded Forward Slashes

The Laravel routing component allows all characters except `/` to be present within route parameter values. You must explicitly allow `/` to be part of your placeholder using a `where` condition regular expression

```php
Route::get('/search/{search}', function($search) {
  return $search;
})->where('search', '.*');
```

> Encoded forward slashes are only supported within the last route segment

## Named Routes

Named routes allow the convenient generation of URLs or redirects for specific routes.

```php
Route::get('/user/profile', function() {
  //
})->name('profile');
```

You may also specify route names for controller actions

```php
Route::get('/user/profile', [UserProfileController::class, 'show'])->name('profile');
```

> Route names should always be unique.

### Generating URLs to Named Routes

Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via Laravel's `route` and `redirect` helper functions

```php
// Generating URLs...
$url = route('profile');

// Generating Redirects
return redirect()->route('profile');

return to_route('profile');
```

If the named route defines parameters, you may pass the parameters as the second argument to the `route` funcdtion. The given parameters will autometically be inserted into the generated URL in their correct positions

```php
Route::get('/user/{id}/profile', function($id) {
  //
})->name('profile');

$url = route('profile', ['id' => 1]);
```

If you pass additional parameters in the array, those key / value pairs will autometically be added to the generated URL's query string.

```php
Route::get('/user/{id}/profile', function($id) {
  //
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']); // /user/1/profile?photos=yes
```

> Sometimes, you may wish to specify request-wide default values for URL parameters, such as the current locale. To accomplish this, you may use the [URL::defaults method][url-default-values].

### Inspecting The Current Route

If you would like to determine if the current request was routed to a given named route, you may use the `named` method on a Route instance.

```php
/**
 * Handle an incoming request.
 *
 * @param \Illuminate\Http\Request $request
 * @param \Closure $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
  if($request->route()->named('profile')) {
    //
  }
  return $next($request);
}
```

## Route Groups

Route groups allow you to share route attributes, such as middleware, across a large number of routes without needing to define those attributes on each individual route.
Nested groups attempt to intelligently "merge" attributes with their parent group. Middleware and `where` conditions are merged while names and prefixes are appended. Namespace delimiters and slashes in URI prefixes are autometically added when appropriate.

### Middleware

To assign [middleware][middleware] to all routes within a group, you may use the `middleware` method before defining the group. Middlewares are executed in the order they are listed in the array

```php
Route::middleware(['first', 'second'])->group(function() {
  Route::get('/', function() {
    // Uses first & second middleware....
  });

  Route::get('/user/profile', function() {
    // uses first & second middleware....
  });
});
```

### Controllers

If a group of routes all utilize the same [controller][controllers], you may use the `controller` method to define the common controller for all of the routes within the group. Then, when defining the routes, you only need to provide the controller method that they invoke

```php
use App\Http\Controllers\OrderController;

Route::controller(OrderController::class)->group(function() {
  Route::get('/orders/{id}', 'show');
  Route::post('/orders', 'store');
});
```

### Subdomain Routing

Route groups may also be used to handle subdomain routing. Subdomains may be assigned route parameters just like route URIs, allowing you to capture a portion of the subdomain for usage in your route or controller. The subdomain may be specified by calling the `domain` method before defining the group

```php
Route::domain('{account}.example.com')->group(function() {
  Route::get('user/{id}', function($account, $id) {
    //
  });
});
```

> In order to ensure your subdomain routes are reachable, you should register subdomain routes before registering root domain route. This will prevent root domain routes from overwriting subdomain routes which have the same URI path.

### Route Prefixes

The `prefix` method may be used to prefix each route in the group with a given URI.

```php
Route::prefix('adming')->group(function() {
  Route::get('/users', function() {
    // Matches The "/admin/users" URL
  });
});
```

### Route Name Prefixes

The `name` method may be used to prefix each route name in the gropu with a given string.

```php
Route::name('admin.')->group(function() {
  Route::get('/users', function() {
    // Route assigned name "admin.users"....
  })->name('users');
});
```

> The given string is prefixed to the route name exactly as it is specified, so be sure to provide the trailing `.` character in the prefix

## Route Model Binding

### Implicit Binding

Laravel autometically resolves Eloquent models defined in routes or controller actions whose type-hinted variable names match a route segment name.

```php
use App\Models\User;

Route::get('/users/{user}', function(User $user) {
  return $user->email;
});
```

Since `$user` variable is type-hinted as the `App\Models\User` Eloquent model and the variable name matches the `{user}` URI segment, Laravel will automatically inject the model instance that has an ID matching the corresponding value form the request URI. If a matching model instance is not found in the databse, a 404 HTTP response will automatically be generated.
Implicit binding is also possible when using controller methods.

```php
use App\Http\Controllers\UserController;
use App\Models\User;

// Route defination
Route::get('/users/{user}', [UserController::class, 'show']);

// Controller method defination
public function show(User $user)
{
  return view('user.profile', ['user' => $user]);
}
```

#### Soft Deleted Models

Typically, implicit model binding will not retrieve models that have been [soft deleted][eloquent-soft-deleting]. However, you may instruct the implicit binding to retrieve these models by chaining the `withTrashed` method onto your route's defination.

```php
use App\Models\User;

Route::get('/users/{user}', function(User $user) {
  return $user->email;
})->withTrashed();
```

#### Customizing The Key

Sometimes you may wish to resolve Eloquent models using a column other than `id`.

```php
use App\Models\Post;

Route::get('/posts/{post:slug}', function(Post $post) {
  return $post;
});
```

If you would like your model binding to always use a database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model

```php
/**
 * Get the route key for the model.
 *
 * @return string
 */
public function getRouteKeyName()
{
  return 'slug';
}
```

#### Custom Keys & Scoping

When implicitly binding multiple Eloquent models in a single route definition, you may wish to scope the second Eloquent model such taht it must be a child of the previous Eloquent model. For example, consider this route definition that retrieves a blog post by slug for a specific user.

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post:slug}', function(User $user, Post $post) {
  return $post;
});
```

When using a custom keyed implicit binding as a nested route parameter, Laravel will automatically scope the query to retrieve the nested model by its parent using conventions to guesss the relationship name on the parent. In this case, it will be assumed that the `User` model has a relationship named `posts` (the plural form of the route parameter name) which can be used to retrieve the `Post` model.

If you wish, you may instruct Laravel to scope "child" bindings even when a custom key is not provided. To do so, you may invoke the `scopeBindings` method when defining the route:

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

Or, you may instruct an entire gropu of route definitions to use scope bindings

```php
use App\Models\Post;
use App\Models\User;

Route::scopeBindings()->group(function() {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});
```

Similarly, you may explicitly instruct Laravel to not scope bindings by invoking the `withoutScopeBindings` method.

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->withoutScopeBindings();
```

#### Customizing Missing Model Behavior

Typically, a 404 HTTP response will be generated if an implicitly bound model is not found. However, you may customize this behavior by calling the `missing` method when defining your route. The `missing` method accepts a closure that will be invoked if an implicitly bound model can not be found

```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        return Redirect::route('locations.index');
    });
```

### Implicit Enum Binding

PHP 8.1 Introduced support for [Enums][backed-enumaritions]. To compliment this feature, Laravel allows you to type-hint a [string-backend-Enum][backed-enumaritions] on your route definition and Laravel will only invoke the route if that route segment corresponds to a valid Enum value. Otherwise a 404 HTTP response will be returned automatically.

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}

```

You may define a route that will only be invoked if the `{category}` route segment is `fruits` or `people`. Otherwise, Laravel will return a 404 HTTP response.

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function(Category $category) {
    return $category->value;
})
```

### Explicit Binding

To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings at the beginning of the `boot` method of your `RouteServiceProvider` class

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::model('user', User::class);

    // ...
}
```

Next, define a route that contains a `{user}` parameter

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    //
});
```

#### Customizing The Resolution Logic

If you wish to define your own model binding resolution logic, you may use the `Route::bind` method. The closure you pass to the bind method will receive the value of the URI segment and should return the instance of the class that should be injected into the route. This customizing should take place in the `boot` method of `RouteServiceProvider` class

```php
user App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::bind('user', function($value) {
        return User::where('name', $value)->firstOrFail();
    });

    // ...
}
```

Alternatively, you may override the `resolveRouteBinding` method on your Eloquent model. This method will receive the value of the URI segment and should return the intance of the class that should be injected into the route

```php
/**
 * Retrieve the model for a bound value.
 *
 * @param mixed $value
 * @param string|null $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail():
}
```

If a route is utilizing [implicit binding scoping](#custom-keys--scoping), the `resolveChildRouteBinding` method will be used to resolve the child binding of the parent model

```php
/**
 * Retrieve the child model for a bound value.
 *
 * @param string $childType
 * @param mixed $value
 * @param string|null $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($childType, $value, $field = null)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```

## Fallback Routes

Using the `Route::fallback` method, you may define a route that will be executed when no other route matches the incoming request. Typically, unhandled requests will automatically render a "404" page via your applications's exception handler. However, since you would typically define the `fallback` route within your `routes/web.php` file, all middleware in the `web` middleware group will apply to the route.

```php
Route::fallback(function() {
    //
});
```

> The fallback route should always be the last route registered by your application.

## Rate Limiting

### Defining Rate Limiters

Laravel includes powerfull and customizable rate limiting services that you may utilize to restrict the amount of traffic for a given route or group of routes. To get started, you should define reate limiter configurations that meet your application's needs. Typically, this should be done within the `configureRateLimiting` method of your application's `App\Providers\RouteServiceProvider` class.

Rate Limiters are defined using the `RateLimiter` facade's `for` method. The `for` method accepts a rate limiter name and a closure that returns the limit configuration that should apply to routes that are assigned to the rate limiter. Limit configuration are instances of the `Illuminate\Cache\RateLimiting\Limit` class. This class contains helpful "builder" methods so that you can quickly define your limit.

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Configure the rate limiters for the application.
 *
 * @return void
 */
protected function configureRateLimiting()
{
    RateLimiter::for('global', function(Request $request) {
        return Limit::perMinutes(1000);
    });
}
```

If the imcoming request exceeds the specified rate limit, a response with a 429 HTTP status code will automatically be returned by Laravel. If you would like to define your own response that should be returned by a rate limit, you may use the `response` method.

```php
RateLimiter::for('global', function(Request $request) {
    return Limit::perMinute(1000)->response(function(Request $request, array $headers) {
        return response('Custom reponse....', 429, $headers);
    });
});
```

Since rate limiter callbacks receive the incoming HTTP request instance, you may build the appropriate rate limit dynamically based on the incoming request or authenticated user.

```php
RateLimiter::for('uploads', function(Request $request) {
    return $request->user()->vipCustomer()
        ? Limit::none()
        : Limit::perMinute(100);
});
```

#### Segmenting Rate Limits

Sometimes you may wish to segment rate limits by some arbitrary value. For example, you may wish to allow users to access a given route 100 times per minute per IP address. To accomplish this, you may use the `by` method when building your rate limit

```php
RateLimiter::for('uploads', function(Request $request) {
    return $request->user()->vipCustomer()
        ? Limit::none()
        : Limit::perMinute(100)->by($request->ip());
});
```

To illustrate this feature using another example, we can limit access to the route to 100 times per minute per authenticated user Id or 10 times per minute per IP address for guests.

```php
RateLimiter::for('uploads', function(Request $request) {
    return $request->user()
        ? Limit::perMinute(100)->by($request->user()->id())
        : Limit::perMinute(10)->by($request->ip());
});
```

#### Multiple Rate Limits

If needed, you may return an array of rate limits for a given rate limiter configuration. Each rate limit will be evaluated for the route based on the order they are placed within the array.

```php
RateLimiter::for('login', function(Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});
```

### Attaching Rate Limiters To Routes

Rate limiters may be attached to routes or route groups using the `throttle` [middleware][middleware]. The throttle middleware accepts the name of the rate limiter you wish to assign to the route.

```php
Route::middleware(['throttle:uploads'])->group(function() {
    Route::post('/audio', function() {
        //
    });

    Route::post('/video', function() {
        //
    });
});
```

#### Throttling With Redis

Typically, the `throttle` middleware is mapped to the `Illuminate\Routing\Middleware\ThrottleRequests` class. This mapping is defined in your application's HTTP kernet (`App\Http\Kernet`). However, if you are using Redis as your application's cache driver, you may wish to change this mapping to use the `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` class. This class is more efficient at mapping rate limiting using Redis.

```php
'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
```

## Form Method Spoofing

HTML forms do not support `PUT`, `PATCH` or `DELETE` actions. So, when defining `PUT`, `PATCH` or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method.

```php
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

For convenience, you may use the `@method` [Blade directive][blade] to generate the `_method` input field.

```php
<form action="/example" method="POST">
    @method("PUT")
    @csrf
</form>
```

## Accessing The Current Route

You may use the `current`, `currentRouteName` and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request.

```php
use Illuminate\Support\Facades\Route;

$route = Route::current(); // Illuminate\Routing\Route
$name = Route::currentRouteName(); // string
$action = Route::currentRouteAction(); // string
```

You may refer to the API documentation for both the [underlying class of the Route facade][api-router] and [Route instance][api-router] to review all of the methods that are availble on the router and route classes.

## Cross-Origin Resource Sharing (CORS)

Laravel can automaticlly respond to CORS `OPTIONS` HTTP requests with values that you configure. All CORS settings may be configured in your applciation's `config/cors.php` configuration file. The `OPTIONS` requests will automatically be handled by the `HnadleCors` [middleware][middleware] that is included by default in your global middleware stack. Your global middleware stack is located in your application's HTTP Kernel (`App\Http\Kernel`).

## Route Caching

When deploying your application to production, you should take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. To generate a route cache, execute the `route:cache` Artisan command.

```sh
php artisan route:cache
```

After running this command, your cached routes file will be loaded on every request. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you should only run the `route:cache` command during your project's deployment.

You may use the `route:clear` command to clear the route cache.

```sh
php artisan route:clear
```

[container]: https://laravel.com/docs/9.x/container
[view]: /basics/views.md
[url-default-values]: https://laravel.com/docs/9.x/urls#default-values
[middleware]: /basics/middleware.md
[controllers]: /basics/controllers.md
[eloquent-soft-deleting]: https://laravel.com/docs/9.x/eloquent#soft-deleting
[backed-enumaritions]: https://www.php.net/manual/en/language.enumerations.backed.php
[blade]: https://laravel.com/docs/9.x/blade
[api-router]: https://laravel.com/api/9.x/Illuminate/Routing/Router.html
