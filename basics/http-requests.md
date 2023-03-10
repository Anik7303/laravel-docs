# HTTP Requests

## Introduction

Laravel's `Illuminate\Http\Request` class provides an object-oriented way to interact with the current HTTP request being handled by your application as well as retrieve the input, cookies and files that were submitted with the request.

## Interacting With The Request

### Accessing The Request

To obtain an instance of the current HTTP request via dependency injection, you should type-hint the `Illuminate\Http\Request` class on your route closure or controller method.

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     *
     * @param \Illuminate\Http\Request $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        // ...
    }
}
```

```php
use Illuminate\Http\Request;

Route::get('/', function(Request $request) {
    //
});
```

#### Dependency Injection & Route Parameters

If your controller method is also expecting input from a route-parameter you should list your route parameter after your other dependencies. For example, if your route is defined like so:

```php
Route::put('/user/{id}', [UserController::class, 'update']);
```

You may still type-hint the `Illuminate\Http\Request` and access your `id` route parameter by defining your controller method as follows:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the specified user.
     *
     * @param \Illuminate\Http\Request $request
     * @param string $id
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, $id)
    {
        //
    }
}
```

### Request Path, Host & Method

The `Illuminate\Http\Request` instance provides a variety of methods for examining the incoming HTTP request and extends the `Symfony\Component\HttpFoundation\Requst` class. We will discuss a few of the most important methods below.

#### Retrieving The Request Path

The `path` method returns the request's path information. If the incoming request is targeted at `http://example.com/foo/bar`, the `path` method will return `foo/bar`:

```php
$uri = $request->path();
```

#### Inspecting The Request Path / Route

The `is` method allows you to verify that the incoming reqeust path matches a given pattern. You may use the `*` character as a wildcard when utilizing this method.

```php
if($request->is('admin/*')) {
    //
}
```

Using the `routeIs` method, you may determine if the incoming request has matched a [named route][routing-named-routes].

```php
if($request->routeIs('admin.*')) {
    //
}
```

#### Retrieving The Request URL

To retrieve the full URL for the incoming request you may use the `url` of `fullUrl` methods. The `url` method will return the URL without the query string, while the `fullUrl` method includes the query string.

```php
$url = $request->url()

$urlWithQueryString = $request->fullUrl();
```

If you would like to append query string data to the current URL, you may call the `fullUrlWithQuery` method. This method merges the given array of query string variables with the current query string.

```php
$request->fullUrlWithQuery(['type' => 'phone']);
```

#### Retrieve The Request Host

You may retrieve the "host" of the incoming request via the `host`, `httpHost` and `schemeAndHttpHost` methods.

```php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
```

#### Retrieve The Request Method

The `method` will return the HTTP verb for the request. You may use the `isMethod` method to verify that the HTTP verb matches a given string.

```php
$method = $request->method();

if($request->isMethod('post')) {
    //
}
```

### Request Headers

You may retrieve a request header from the `Illuminate\Http\Request` instance using the `header` method. If the header is not present on the request, `null` will be returned. However, the `header` method accepts an optional second argument that will be returned if the header is not present on the request.

```php
$value = $request->header('X-Header-Name');

$value = $request->header('X-Header-Name', 'default');
```

The `hasHeader` method may be used to determine if the request contains a given header.

```php
if($request->hasHeader('X-Header-Home')) {
    //
}
```

For convenience, the `bearerToken` method may be used to retrieve a bearer token from the `Authorization` header. If no such header is present, an empty string will be returned.

```php
$token = $request->bearerToken();
```

### Request IP Address

The `ip` method may be used to retrieve the IP address of the client that made the request.

```php
$ipAddress = $request->ip();
```

### Content Negotiation

Laravel provides several methods for inspecting the incoming reqeust's requested content types via the `Accept` header. First, the `getAcceptableContentTypes` method will return an array containing all of the content types accepted by the request.

```php
$contentTypes = $request->getAcceptableContentTypes();
```

The `accepts` method accepts an arraay of content types and returns `true` if any of the content types are accpeted by the request. Otherwise, false will be returned.

```php
if($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

You may use the `prefers` method to determine which content type out of a given array of content types is most preferred by the request. If none of the provided content types are accepted by the request, `null` will be returned.

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

Since many applications only serves HTML or JSON, you may use the `expectsJson` method to quickly determine if the incoming reqeust expects a JSON response.

```php
if($request->expectsJson()) {
    // ...
}
```

### PSR-7 Requests

The [PSR-7 standard][psr-7] specifies interfaces for HTTP messages, including requests and responses. If you would like to obtain an instance of a PSR-7 request instead of a Laravel request, you will first need to install a few libraries. Laravel uses Symfony HTTP Message Bridge component to convert typical Laravel reqeusts and responses into PSR-7 compatible implementations.

```sh
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

Once you have installed these libraries, you may obtain an instance of a PSR-7 request by type-hinting the request interface on your route closure or controller method.

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request)) {
    //
}
```

## Input

### Retrieve Input

#### Retrieving All Input Data

You may retrieve all of the incoming request's input data as an `array` using the `all` method. This method may be used regardless of whether the incoming request is from an HTML form or is an XHR request.

```php
$input = $request->all();
```

Using the `collect` method, you may retrieve all of the incoming request's input data as a [collection][collections].

```php
$input = $request->collect();
```

The `collect` method also allows you to retrieve a subset of the incoming request input as a collection.

```php
$request->collect('users')->each(function ($user) {
    // ...
});
```

#### Retrieving An Input Value

Using a few simple methods, you may access all of the user input from your `Illuminate\Http\Request` instance without worrying about which HTTP verb was used for the request. Regardless of the HTTP verb, the `input` method may be used to retrieve user input.

```php
$name = $request->input('name');
```

You may pass a default value as the second argument to the `input` method. This value will be returned if the requested input value is not present on the request.

```php
$name = $request->input('name', 'John');
```

When working with form that contain array inputs, use "dot" notation to access the arrays.

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

You may call the input method without any arguments in order to retrieve all of the input values as an associative array.

```php
$input = $request->input();
```

#### Retrieve Input From The Query String

While the `input` method retrieves values from the entire request payload (including the query string), the `query` method will only retrieve values from the query string.

```php
$name = $request->query('name');
```

If the requested query string value data is not present, the second argument to this method will be returned.

```php
$name = $request->query('name', 'Jane');
```

You may call the `query` method without any arguments in order to retrieve all of the query string values as an associative array.

```php
$input = $request->query();
```

#### Retrieving JSON input values

When sending JSON requests to your application, you may access the JSON data via the `input` method as long as the `Content-Type` header of the request is properly set to `application/json`. You may even use "dot" syntax to retrieve values that are nested within JSON arrays / objects.

```php
$name = $request->input('user.name');
```

#### Retrieving Stringable Input Values

Instead of retrieving the request's input data as a primitive `string`, you may use the `string` method to retrieve the request data as an instance of `Illuminate\Support\Stringable`.

```php
$name = $request->string('name')->trim();
```

#### Retrieving Boolean Input Values

When dealing with HTML elements like checkboxes, your application may receive "truthy" values that are actually strings. For example, "true" or "on". For convenience, you may use the `boolean` method to retrieve these values as booleans. The `boolean` method returns `true` for **1**, **"1"**, **true**, **"true"**, **"on"** and **"yes"**. All other values will return `false`.

```php
$archived = $request->boolean('archived');
```

#### Retrieve Date Input Values

For convenience, input values containing dates / times may be retrieved as Carbon instances using the `date` method. If the request does not contain an input value with the given name, `null` will be returned.

```php
$birthday = $request->date('birthday');
```

The second and thrid arguments accepted by the `date` method may be used to specify the date's _format_ and _timezone_, respectively.

```php
$elapsed = $request->date('elapsed', '!H:i', 'Eurpose/Madrid');
```

If the input value is present but has an invalid format, an `InvalidArgumentException` will be thrown. Therefore, it is recommended that you validate the input before invoking the `date` method.

#### Retrieving Enum Input Values

Input values that corresponds to [PHP enums][php-enumerations] may also be retrieved from the request. If the request does not contain an input value with the given name or the enum does not have a backing value that matches the input value, `null` will be returned. The `enum` method accepts the name of the input value and the enum class as its first and second arguments.

```php
use App\Enums\Status;

$status = $request->enum('status', Status::class);
```

#### Retrieving Input Via Dynamic Properties

You may also access user input using dynamic properties on the `Illuminate\Http\Request` instance. For example, if one of your application's forms contains a `name` field, you may access the value of the field like so:

```php
$name = $request->name;
```

When using dynamic properties, Laravel will first look for the parameter's value in the request payload. If it is not present, Laravel will search for the field in the matched route's parameters.

#### Retrieving A Portion Of The Input Data

If you need to retrieve a subset of the input data, you may use the `only` and `except` methods. Both of these methods accept a single `array` or a dynamic list of arguments.

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

> The `only` method returns all of teh key / value pairs that you request; however, it will not return key / value pairs that are not present on the request.

### Determining If The Input Is Present

You may use the `has` method to determine if a value is present on the request. The `has` method returns `true` if the value is present on the request.

```php
if($request->has('name')) {
    //
}
```

When given an array, the `has` method will determine if all of the specified values are present.

```php
if($request->has(['name', 'email'])) {
    //
}
```

The `whenHas` method will execute the given closure if a value is present on the request.

```php
$request->whenHas('name', function($input) {
    //
});
```

A second closure may be passed to the `whenHas` method that will be executed if the specified value is not present on the request.

```php
$request->whenHas('name', function($input) {
    // The "name" value is present...
}, function() {
    // The "name" value is not present...
});
```

The `hasAny` method returns `true` if any of teh specified values are present.

```php
if($request->hasAny(['name', 'email'])) {
    //
}
```

If you would like to determine if a value is present on the request and is not an empty string, you may use the `filled` method.

```php
if($request->filled('name')) {
    //
}
```

The `whenFilled` method will execute the given closure if a value is present on the request and is not an empty string.

```php
$request->whenFilled('name', function($input) {
    //
});
```

A second closure may be passed to the `whenFilled` method that will be executed if the specified value is not "filled".

```php
$request->whenFilled('name', function($input) {
    // The 'name' value is present...
}, function() {
    // The 'name' value is not present...
});
```

To determine if a given key is absent from the request, you may use the `missing` and `whenMissing` methods.

```php
if($request->missing('name')) {
    //
}

$request->whenMissing('name', function($input) {
    // The 'name' value is missing...
}, function() {
    // The 'name' value is present...
});
```

### Merging Additional Input

Sometimes you may need to manually merge additional input into the request's existing input data. To accomplish this, you may use the `merge` method. If a given input key already exists on the request, it will be overwritten by the data provided to the `merge` method.

```php
$request->merge(['votes' => 0]);
```

The `mergeIfMissing` method may be used to merge input into the request if the corresponding keys do not already exist within the request's input data.

```php
$request->mergeIfMissing(['votes' => 0]);
```

### Old Input

Laravel allows you to keep input from one request during the next request. This feature is particularly useful for re-populating forms after detecting validation errors. However, if you are using Laravel's included [validation features][validation], it is possible that you will not need to manually use these session input flashing methods directly, as some of the Laravel's built-in validation facilities will call them automatically.

#### Flashing Input To The Session

The `flash` method on the `Illuminate\Http\Request` class will flash the current input to the [session][session] so that it is available during the user's next request to the application.

```php
$request->flash();
```

You may also use the `flashOnly` and `flashExcept` methods to flash a subset of the request data to the session. These methods are useful for keeping sensitive information such as passwords out of the session.

```php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

#### Flashing Input Then Redirecting

Since you often wil want to flash input to the session and then redirect to the previous page, you may easily chain input flashing onto a redirect using the `withInput` method.

```php
return redirect('form')->withInput();

return redirect()->route('user.create')->withInput();

return redirect('form')->withInput(
    $request->except('password')
);
```

#### Retrieving Old Input

To retrieve flashed input from the previous request, invoke the `old` method on an instance of `Illuminate\Http\Request`. The `old` method will pull the previously flashed input data from the [session][session].

```php
$username = $request->old('username');
```

Laravel also provides a global `old` helper. If you are displaying old input within a [Blade template][blade], it is more convenient to use the `old` helper to repopulate the form. If no old input exists for the given field, `null` will be returned.

```php
<input type="text" name="username" value="{{ old('username') }}">
```

### Cookies

#### Retrieving Cookies From Requests

All cookies created by the Laravel framework are encrypted and signed with an authentication code, meaning they will be considered invalid if they have been changed by the client. To retrieve a cookie value from the request, use the `cookie` method on an `Illuminate\Http\Request` instance.

```php
$value = $request->cookie('name');
```

### Input Trimming & Normalization

By default, Laravel includes the `App\Http\Middleware\TrimStrings` and `Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull` middleware in your application's global middleware stack. These middleware are listed in the global middleware stack by the `App\Http\Kernel` class. These middleware will automatically trim all incoming string fields on the request, as well as convert any empty string fields to `null`. This allows you to not have to worry about these normalization concerns in your routes and controllers.

#### Disabling Input Normalization

If you would like to disable this behavior for all requests, yo may remove the two middleware from your application's middleware stack by removing them from the `$middleware` property of your `App\Http\Kernel` class.

If you would like to disable string trimming and empty tring conversion for a subset of requests to your application, you may use the `skipWhen` method offered by both middleware. This method accepts a closure which should return `true` or `false` to indicate if input normalization should be skipped. Typically, the `skipWhen` method should be invoked in the `boot` method of your application's `AppServiceProvider`.

```php
use App\Http\Middleware\TrimStrings;
use Illuminate\Foundation\Middleware\ConverEmptyStringsToNull;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    TrimStrings::skipWhen(function ($request) {
        return $request->is('admin/*');
    });

    ConvertEmptyStringsToNull::skipWhen(function ($request) {
        // ...
    });
}
```

## Files

### Retrieving Uploaded Files

You may retrieve uploaded files from an `Illuminate\Http\Request` instance using the `file` method or using dynamic properties. The `file` method returns an instance of the `Illuminate\Http\UploadedFile` class, which extends the PHP `SplFileInfo` class and provides a variety of methods for interacting with the file.

```php
$file = $request->file('photo');

$file = $request->photo;
```

You may determine if a file is present on the request using the `hasFile` method.

```php
if($request->hasFile('photo')) {
    //
}
```

#### Validating Successful Uploads

In addition to checking if the file is present, you may verify that there were no problems uploading the file via the `isValid` method.

```php
if($request->file('photo')->isValid()) {
    //
}
```

#### File Paths & Extensions

The `UploadedFile` class also contains methods for accessing the file's fully-qualified path and its extension. The `extension` method will attempt to guess the file's extension based on its conent. This extension may be different from the extension that was supplied by the client.

```php
$path = $request->photo->path();

$extension = $request->photo->extension();
```

#### Other File Methods

There are a variety of other methods available on `UploadedFile` instances. Check out the [API documentation for the class][symfony-uploaded-file] for more information regarding these methods.

### Storing Uploaded Files

To store an uploaded file, you will typically use one of your configured [filesystems][filesystem]. The `UploadedFile` class has a `store` method that will move an uploaded file to one of your disks, which may be a location on your local filesystem or a cloud storage location like Amazon S3.

The `store` method accepts the path where the file should be stored relative to the filesystem's configured root directory. This path should not contain a filename, since a unique ID will automatically be generated to serve as the filename.

The `store` method also accepts an optional second argument for the name of the disk that should be used to store the file. The method will return the path of the file relative to the disk's root.

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

If you do not want a filename to be automatically generated, you may use the `storeAs` method, which accepts the path, filename and disk name as its arguments.

```php
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

> For more information about the file storage in Laravel, check out the complete [file storage documentation][filesystem].

## Configuring Trusted Proxies

When running your applications behind a load balancer that terminates TLS / SSL certificates, you may notice your application sometimes does not generate HTTPS links when using the `url` helper. Typically this is because your application is being forwarded traffic from your load balancer on port 80 and does not know it should generate secure links.

To solve this, you may use the `App\Http\Middleware\TrustProxies` middleware that is included in your Laravel application, which allows you to quickly customize the load balancers or proxies that should be trusted by your application. Your trusted proxies should be listed as an array on the `$proxies` property of this middleware. In addition to configuring the trusted proxies, you may configure the proxy `$headers` that should be trusted.

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Http\Middleware\TrustProxies as Middleware;
use Illuminate\Http\Request;

class TrustProxies extends Middleware
{
    /**
     * The trusted proxies for this application.
     *
     * @var string|array
     */
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];

    /**
     * The headers that should be used to detect proxies.
     *
     * @var int
     */
    protected $headers = Request::HEADER_X_FORWARDED_FOR | Request::HEADER_X_FORWARDED_HOST | Request::HEADER_X_FORWARDED_PORT | Request::HEADER_X_FORWARDED_PROTO;
}
```

> If you are using AWS Elasitc Load Balancing, your `$headers` value should be `Request::HEADER_X_FORWARDED_AWS_ELB`. For more information on the constants that may be used in the `$headers` property, check out the Symfony's documentation on [trusting proxies][symfony-proxies].

## Configuring Trusted Hosts

By default, Laravel will respond to all requests it receives regardless of the content of the HTTP request's `Host` header. In addition, the `Host` header's value will be ued when generating absolute URLs to your application during a web request.

Typically, you should configure your web server, such as Nginx o Apache, to only send requests to your application that match a given host name. However, if you do not have the ability to customize your web server directly and need to instruct Laravel to only respond to certain host names, you may do so by enabling the `App\Http\Middleware\TrustHosts` middleware for your application.

The `TrustHosts` middleware is already included in the `$middleware` stack of your application; however, you should uncomment it so that it becomes active. Within this middleware's `hosts` method, you may specify the host names that your application should respond to. Incoming requests with other `Host` value headers will be rejected.

```php
/**
 * Get the host patterns that should be trusted.
 *
 * @return array
 */
public function hosts()
{
    return [
        'laravel.test',
        $this->allSubdomainsOfApplicationUrl(),
    ];
}
```

The `allSubdomainsOfApplicatoinUrl` helper method will return a regular expression matching all subdomains of your application's `app.url` configuration value. This helper method provies a convenient way to allow all of your application's subdomains when building an application that utilizes wildcard subdomains.

[routing-named-routes]: /basics/routing.md#named-routes
[psr-7]: https://www.php-fig.org/psr/psr-7
[collections]: https://laravel.com/docs/9.x/collections
[php-enumerations]: https://www.php.net/manual/en/language.types.enumerations.php
[validation]: https://laravel.com/docs/9.x/validation
[session]: https://laravel.com/docs/9.x/session
[blade]: https://laravel.com/docs/9.x/blade
[symfony-uploaded-file]: https://github.com/symfony/symfony/blob/6.0/src/Symfony/Component/HttpFoundation/File/UploadedFile.php
[filesystem]: https://laravel.com/docs/9.x/filesystem
[symfony-proxies]: https://symfony.com/doc/current/deployment/proxies.html
