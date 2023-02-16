# Validation

## Introduction

Laravel provides several different approaches to validate your application's incoming data. It is most common to use the `validate` method available on all incoming HTTP reqeusts. Laravel includes a wide variety of convenient validation rules that you may apply to data, even providing the ability to validate if values are unique in a given database table.

## Validation Quickstart

### Defining The Routes

First, let's assume we have the following routes defined in our `routes/web.php` file -

```php
use App\Http\Controllers\PostController;

Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

The `GET` route will display a form for the user to create a new blog post, while the `POST` route will store the new blog post in the database.

### Creating The Controller

Next, let's take a look at a simple controller that handles incoming requests to these routes. We'll leave the `store` method empty for now.

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class PostController extends Controller
{
    /**
     * Show the form to create a new blog post.
     *
     * @return \Illuminate\View\View
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * Store a new blog post.
     *
     * @param \Illuminate\Http\Request $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // Validate and store the blog post
    }
}
```

### Writing The Validation Logic

Now we will fill our `store` method with the logic to validate the new blog post. To do this, we will use the `validate` method provided by the `Illuminate\Http\Request` object. If the validation rules pass, you code will keep executing normally; however, if validation fails, an `Illuminate\Validation\ValidationException` exception will be thrown and the proper error response will automatically be sent to the user.

If validation fails during a traditional HTTP request, a redirect response to the previous URL will be generated. If the incoming request is an XHR request, a [JSON response containing the validation error messeages][validation-error-response-format] will be returned.

To get a better understanding of the `validate` method, let's jump back into the `store` method.

```php
/**
 * Store a new blog post.
 *
 * @param \Illuminate\Http\Request $request
 * @return \Illuminate\Http\Response
 */
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // The blog post is valid...
}
```

The validation rules are passed into the `validate` method. If the validation fails, the proper response will automatically be generated. If the validation passes, the controller will continue executing normally.

Alternatively, validation rules may be specified as array of rules instead of a single `|` delimited string.

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

In addition, you may use the `validateWithBag` method to validate a request and store any error messages within a [named error bag][validation-named-error-bag].

```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

#### Stopping On First Validation Failure

Sometimes you may wish to stop running validation rules on an attribute after the first validation failure. To do so, assign the `bail` rule to the attribute.

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

In this example, if the `unique` rule on the `title` attribute fails, the `max` rule will not be checked. Rules will be validated in the order they are assigned.

#### A Note On Nested Attributes

If the incoming HTTP request contains **nested** field data, you may specify these fields in your validation rules using **dot** syntax.

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

ON the other hand, if your field name contains a literal period, you can explicitly prevent this from being interpreted as _dot_ syntax by escaping the period with a backslash.

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

### Displaying The Validation Errors

All of the validation errors and [request input][requests-retrieving-old-input] will automatically be [flashed to the session][session-flash-data] upon validation failure.

An `$errors` variable is shared with all of your application's views by the `Illuminate\View\Middleware\ShareErrorsFromSession` middleware, which is provided by the `web` middleware group. When this middleware is applied an `$errors` variable will always be available in your views, allowing you to conveniently assume the `$errors` variable is always defined and can be safely used. The `$errors` variable will be an instance of `Illuminate\Support\MessageBag`. For more information on working with this object, [checkout its documentation][validation-working-with-error-messages].

So, in our example, the user will be redirected to our controller's `create` method when validation fails, allowing us to display the error messages in the view:

```php
<!-- /resources/views/post/create.blade.php -->
<h1>Create Post</h1>

@if($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

#### Customizing The Error Messages

Laravel's built-in validation rules each have an error message that is located in your application's `lang/en/validation.php` file. Within this file, you will find a translation entry for each validation rule. You are free to change or modify these messages based on the needs of your application.

#### XHR Requests & Validation

In this example, we used a traditional form to send data to the application. However, many applications receive XHR requests from a JavaScript powered frontend. When using the `validate` method during an XHR request, Laravel will not generate a redirect response. Instead, Laravel generates a [JSON response containing all of the validation errors][validation-error-response-format]. This JSON response will be sent with a _422 HTTP status code_.

#### The `@error` Directive

You may use the `@error` [Blade][blade] directive to quickly determine if validation error messages exist for a given attribute. Within an `@error` directive, you may echo the `$message` variable to display the error message.

```php
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title" type="text" name="title" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

If you are using [named error bags][validation-named-error-bag], you may pass the name of the error bag as the second argument to the `@error` directive.

```php
<input ... class="@error('title', 'post') is-invalid @enderror">
```

### Repopulating Forms

When Laravel generates a redirect response due to a validation error, the framework will automatically [flash all of the request's nput to the session][session-flash-data]. This is done so that you may conveniently access the input during the next request and repopulate the form that the user attempted to submit.

To retrieve flashed input from the previous request, invoke the `old` method on an instance of `Illuminate\Http\Request`. The `old` method will pull the previously flashed input data from the [session][session].

```php
$title = $request->old('title');
```

Laravel also provides a global `old` helper. If you are displaying old input within a [Blade template][blade], it is more convenient to use the `old` helper to repopulate the form. If no old input exists for the given field, `null` will be returned.

```php
<input type="text" name="title" value="{{ old('title') }}">
```

### A Note On Optional Fields

By default, Laravel includes the `TrimStrings` and `ConvertEmptyStringsToNull` middleware in your application's global middleware stack. These middleware are listed in the stack by the `App\Http\Kernel` class. Because of this, you will often need to mark your **optional** request fields as `nullable` if you do not want the validator to consider `null` value as invalid. For example -

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'published_at' => 'nullable|date',
]);
```

In this example, we are specifying that the `published_at` field may be either `null` or a valid date representation. If the `null` modifier is not added to the rule definition, the validator would consider `null` an invalid date.`

### Validation Error Response Format

When your application throws a `Illuminate\Validation\ValidationException` exception and the incoming HTTP request is expecting a JSON response, Laravel will automatically format the error messages for you and return a `422 Unprocessable Entity` HTTP response.

Below, you can review an example of the JSON response format for validation errors. Note that nested error keys are flattened into **dot** noation format.

```json
{
  "message": "The team name must be a string, (and 4 more errors)",
  "errors": {
    "team_name": [
      "The team name must be a string.",
      "The team name must be at least 1 characters."
    ],
    "authorization.role": ["The selected authorization.role is invalid."],
    "users.0.email": ["The users.0.email field is required."],
    "users.2.email": ["The users.2.email must be a valid email address."]
  }
}
```

## Form Request Validation

### Creating Form Requests

For more complex validation scenarios, you may wish to create a **form request**. Form requests are custom request classes that encapsulate their own validation and authorization logic. To create a request class, you may use the `make:request` Artisan CLI command.

```php
php artisan make:request StorePostRequest
```

The generated form request class will be placed in the `app\Http\Requests` directory. If this directory does not exist, it will be created when you run the `make:request` command. Each form request generated by Laravel has two methods - `authorize` and `rules`.

- `authorize` method is responsible for determining if the currently authenticated user can perform the action represented by the request.
- `rules` method returns the validation rules that should apply to the request's data.

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

> You may type-hint any dependencies you require within the `rules` method's signature. They will automatically be resolved via the Laravel [service container][service-container]

All you need to do type-hint the request on your controller method. The incoming form request is validated before the controller method is called, meaning you do not need to clutter your controller with any validation logic.

```php
/**
 * Store a new blog post.
 *
 * @param \App\Http\Requests\StorePostRequest $request
 * @return \Illuminate\Http\Response
 */
public function store(StorePostRequest $request)
{
    // The incoming request is valid...

    // Retrieve the validated input data...
    $validated = $request->validated();

    // Retrieve a portion of the validated input data...
    $validated = $request->safe()->only(['name', 'email']);
    $validated = $request->safe()->except(['name', 'email']);
}
```

#### Adding After Hooks To Form Requests

If you would like to add an _after_ validation hook to a form request, you may use the `withValidator` method. This method receives the fully constructed validator, allowing you to call any of its methods before the validation rules are actually evaluated.

```php
/**
 * Configure the validator instance.
 *
 * @param \Illuminate\Validation\Validator $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field');
        }
    });
}
```

#### Stopping On First Validation Failure Attribute

By adding a `stopOnFirstFailure` property to your request class, you may inform the validator that it should stop validating all attributes once a single validation failure has occured.

```php
/**
 * Indicates if the validator should stop on the first rule failure.
 *
 * @var bool
 */
protected $stopOnFirstFailure = true;
```

#### Customizing The Redirect Location

As previously discussed, a redirect response will be generated to send the user back to their previous location when form request validation fails. However you are free to customize this behavior. To do so, define a `redirect` property on your form request.

```php
/**
 * The URI that users should be redirected to if validation fails.
 *
 * @var string
 */
protected $redirect = '/dashboard';
```

### Authorizing Form Requests

The form request class also contains an `authorize` method. Within this method, you may determine if the authenticated user actually has the authority to update a given resource. For example, you may determine if a user actually owns a blog comment they are attempting to update. Most likely, you will interact with your [authorization gates and policies][authorization] within this method.

```php
use App\Models\Comment;

/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

since all form requests extend the base Laravel request class, we may use the `user` method to access the currently authenticated user. Also, note the call to the `route` method in the example above. This method grants you access to the URI parameters defined on the route being called, such as the `{comment}` parameter in the example below.

```php
Route::post('/comment/{comment}');
```

Therefore, if your application is taking advantage of [route model binding][routing-route-model-binding], your code may be made even more succinct by accessing the resloved model as a property of the request.

```php
return $this->user()->can('update', $this->comment);
```

If the `authorize` method returns `false`, an HTTP response with a 403 status code will automatically be returned and your controller method will not execute.

If you plan to handle authorization logic for the request in another part of your application, you may simply return `true` from the `authorize` method.

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public functiona authorize()
{
    return true;
}
```

> You may type-hint dependencies you need within the `authorize` method's signature. They will automatically be resolved via the Laravel [service container][service-container]

### Customizing The Error Messages

You may customize the error messages used by the form request by overriding the `messages` method. This method should return an array of attribute / rule pairs and their corresponding error messages.

```php
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array
 */
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```

#### Customizing The Validation Attributes

Many of Laravel's built-in validation rule error messages contain an `:attribute` placeholder. If you would like the `:attribute` placeholder of your application message to be replaced with a custom attribute name, you may specify the custom names by overriding the `attributes` method. This method should return an array of attribute / name pairs.

```php
/**
 * Get custom attributes for validator errors.
 *
 * @return array
 */
public function attributes()
{
    return [
        'email' => 'email address',
    ];
}
```

### Preparing Input For Validation

If you need to prepare or sanitize any data from the request before you apply your validation rules, you may use the `prepareForValidation` method.

```php
use Illuminate\Support\Str;

/**
 * Prepare the data for validation.
 *
 * @return void
 */
public function prepareForValidation()
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

Likewise, if you need to normalize any request data after validation is complete, you may use the `passedValidation` method.

```php
use Illuminate\Support\Str;

/**
 * Handle a passed validation attempt.
 *
 * @return void
 */
public function passedValidation()
{
    $this->replace(['name' => 'Taylor']);
}
```

## Manually Creating Validators

If you do not want to use the `validate` method on the request, you may create a validator instance manually using the `Validator` [facade][facades]. The `make` method on the facade generates a new validator instance.

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Illuminate\Support\Facades\Validator;

class PostController extends Controller
{
    /**
     * Store a new blog post.
     *
     * @param Request $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if($validator->fails()) {
            return redirect('post/create')->withErrors($validator)->withInput();
        }

        // Retrieve the validated  input...
        $validated = $validator->validated();

        // Retrieve a portion of the validated input...
        $validated = $validator->safe()->only(['name', 'email']);
        $validated = $validator->safe()->except(['name', 'email']);

        // Store the blog post...
    }
}
```

The first argument passed to the `make` method is the data under validation. The second argument is an array of the validation rules that should be applied to the data.

After determining whether the request validation failed, you may use the `withErrors` method to flash the error messages to the session. When using this method, the `$errors` variable will automatically be shared with your views after redirection, allowing you to easily display them back to the user. The `withErrors` method accepts a validator, a `MessageBag`, or a PHP `array`.

#### Stopping On First Validation Failure

The `stopOnFirstFailure` method will inform the validator that it should stop validating all attributes once a single validation failure has occurred.

```php
if($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

### Automatic Redirection

If you would like to create a validator instance manually but still take advantage of the automatic redirection offered by the HTTP request's `validate` method, you may call the `validate` method on an existing validator instance. If validation fails, the user will automatically be redirected or, in the case of an XHR request, a [JSON response will be returned][validation-error-response-format].

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

You may use the `validateWithBag` method to store the error messages in a [names error bag][validation-named-error-bag] if validation fails.

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

### Names Error Bags

If you have multiple forms on a single page, you may wish to name the `MessageBag` containing the validation erros, allowing you to retrieve the error messages for a specific form. To achieve this, pass a name as the second argument to `withErrors`.

```php
return redirect('register')->withErrors($validator, 'login');
```

You may then access the named `MessageBag` instance from the `$errors` variable.

```php
{{ $errors->login->first('email') }}
```

### Customizing The Error Messages

If needed, you may provide custom error messages that a validator instance should use instead of the default error messages provided by Laravel. There are several ways to specify custom messages. First, you may pass the custom messages as the third argument to the `Validator::make` method.

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => 'The :attribute field is required.',
]);
```

In this example, the `:attribute` placeholder will be replaced by the actual name of the field under validation. You may also utilize other placeholders in validation messages. For example -

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

#### Specifying A Custom Message For A Given Attribute

Sometimes you may wish to specify a custom error message only for a specific attribute. You may do so using _dot_ notation. Specify the attribute's name first, followed by the rule -

```php
$messages = [
    'email.required' => 'We need to know your email address!',
];
```

#### Specifying Custom Attribute Values

Many of Laravel's built-in error messages include an `:attribute` placeholder that is replaced with the name of the field or attribute under validation. To customize the values used to replace these placeholders for specific fields, you may pass an array of custom attributes as the fourth argument to the `Validator::make` method.

```php
$validator = Validator::make($input, $rules, $messages, [
    'email' => 'email address',
]);
```

### After Validation Hook

You may also attach callbacks to be run after validation is completed. This allows you to easily perform further validation and even add more error messages to the message collection. To get started, call the `after` method on a validator instance.

```php
$validator = Validator::make(/* ... */);

$validator->after(function ($validator) {
    if($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if($validator->fails()) {
    //
}
```

## Working With Validated Input

After validating incoming request data using a form request or a manually created validator instance, you may wish to retrieve the incoming request data that actually underwent validation. This can be accomplished in several ways. First, you may call the `validated` method on a form request or validator instance. This method returns an array of the data that was validated.

```php
$validated = $request->validated();

# or

$validated = $validator->validated();
```

Alternatively, You may call the `safe` method on a form request or a validator instance. This method returns an instance of `Illuminate\Support\ValidatedInput`. This object exposes `only`, `except` and `all` methods to retrieve a subset of the validated data or the entire array of validated data.

```php
$validated = $request->safe()->only(['name', 'email']);

# or

$validated = $request->safe()->except(['name', 'email']);

# or

$validated = $request->safe()->all();
```

In addition, the `Illuminate\Support\ValidatedInput` instance may be iterated over and accessed like an array.

```php
// Validated data may be iterated...
foreach($request->safe() as $key => $value) {
    //
}

// Validated data may be accessed as an array...
$validated = $request->safe();

$email = $validated['email'];
```

If you would like to add additional fields to the validated data, you may call the `merge` method.

```php
$validated = $request->safe()->merge(['name' => 'Taylor Otwell']);
```

If you would like to retrieve the validated data as a [collection][collections] instance, you may call the `collect` method.

```php
$collection = $request->safe()->collect();
```

## Working With Error Messages

After calling the `errors` method on a `Validated` instance, you will receive an `Illuminate\Support\MessageBag` instance, which has a variety of convenient methods for working with error messages. The `$errors` variable that is automatically made available to all views is also an instance of the `MessageBag` class.

#### Retrieving The First Error Message For A Field

To retrieve the first error message for a given field, use the `first` method.

```php
$errors = $validator->errors();

echo $errors->first('email');
```

#### Retrieving All Error Messages For A Field

If you need to retrieve an array of all the messages for a given field, use the `get` method.

```php
foreach($errors->get('email') as $message) {
    //
}
```

If you are validating an array form field, you may retrieve all of the messages for each of the array elements using the `*` character.

```php
foreach($erros->get('attachments.*') as $message) {
    //
}
```

#### Retrieving All Error Messages For All Fields

To retrieve an array of all messages for all fields, use the `all` method.

```php
foreach($errors->all() as $message) {
    //
}
```

#### Determining If Messages Exist For A Field

The `has` method may be used to determine if any error messages exist for a given field.

```php
if($errors->has('email')) {
    //
}
```

### Specifying Custom Messages In Language Files

Laravel's built-in validation rules each have an error message that is located in your application's `lang/en/validation.php` file. Within this file, you will find a translation entry for each validation rule. You are free to change or modify these messages based on the needs of your application.

In addition, you may copy this file to another translation language directory to translate the messages for your application's language. To learn more about Laravel localization, check out the complete [localization documentation][localization].

#### Custom Messages For Specific Attributes

You may customize the error messages used for specific attribute and rule combinations within your application's validation language files. To do so, add your message customizations to the `custom` array of your application's `lang/xx/validation.php` language file.

```php
'custom' => [
    'email' => [
        'required' => 'We need to know your email address!',
        'max' => 'You email address is too long!',
    ],
],
```

### Specifying Attributes In Language Files

Many of Laravel's built-in error messages include an `:attribute` placeholder that is replaced with the name of the field or attribute under validation. If you would like the `:attribute` portion of your validation message to be replaced with a custom value, you may specify the custom attribute name in the `attributes` array of your `lang/xx/validation.php` language file.

```php
'attributes' = [
    'email' => 'email address',
].
```

### Specifying Values In Language Files

Some of Laravel's built-in validation rule error messages contain a `:value` placeholder that is replaced with the current value of the request attribute. However, you may occasionally need the `:value` portion of your validation message to be replaced with a custom representation of the value. For example, consider the following rule that specifies that a credit card number is required if the `payment_type` has a value of `cc` -

```php
Validator::make($request->all(), [
    'credit_card_number' => 'required_if:payment_type.cc',
]);
```

If this validation rule fails, it will produce the following error message -

```
The credit card number field is required when payment type is cc.
```

Instead of displaying `cc` as the payment type value, you may specify a more user-friendly value representation in your `lang/xx/validation.php` language file by defining a `values` array.

```php
'values' => [
    'payment_type' => [
        'cc' => 'credit card',
    ],
],
```

After defining this value, the valiation rule will produce the following error message -

```
The credit card number field is required when payment type is credit card.
```

## Available Validation Rules

Below is a list of all available validation rules and their function -

- [Accepted](#)
- [Accepted If](#)
- [Active URL](#)
- [After (Date)](#)
- [After Or Equal (Date)](#)
- [Alpha](#)
- [Alpha Dash](#)
- [Alpha Numeric](#)
- [Array](#)
- [Ascii](#)
- [Bail](#)
- [Before (Date)](#)
- [Before Or Equal (Date)](#)
- [Between](#)
- [Boolean](#)
- [Confirmed](#)
- [Current Password](#)
- [Date](#)
- [Date Equals](#)
- [Date Format](#)
- [Decimal](#)
- [Declined](#)
- [Declined If](#)
- [Different](#)
- [Digits](#)
- [Digits Between](#)
- [Dimensioons (Image Files)](#)
- [Distinct](#)
- [Doesnt Start With](#)
- [Doesnt End With](#)
- [Email](#)
- [Ends With](#)
- [Enum](#)
- [Exclude](#)
- [Exclude If](#)
- [Exclude Unless](#)
- [Exclude With](#)
- [Exclude Without](#)
- [Exists (Database)](#)
- [File](#)
- [Filled](#)
- [Greater Than](#)
- [Greater Than Or Equal](#)
- [Image (File)](#)
- [In](#)
- [In Array](#)
- [Integer](#)
- [IP Address](#)
- [JSON](#)
- [Less Than](#)
- [Less Than Or Equal](#)
- [Lowercase](#)
- [MAC Address](#)
- [Max](#)
- [Max Digits](#)
- [MIME Types](#)
- [MIME Type By File Extension](#)
- [Min](#)
- [Min Digits](#)
- [Missing](#)
- [Missing If](#)
- [Missing Unless](#)
- [Missing With](#)
- [Missing With All](#)
- [Multiple Of](#)
- [Not In](#)
- [Not Regex](#)
- [Nullable](#)
- [Numeric](#)
- [Password](#)
- [Present](#)
- [Prohibited](#)
- [Prohibited If](#)
- [Prohibited Unless](#)
- [Prohibits](#)
- [Regular Expression](#)
- [Required](#)
- [Required If](#)
- [Required Unless](#)
- [Required With](#)
- [Required With All](#)
- [Required Without](#)
- [Required Without All](#)
- [Required Array Keys](#)
- [Same](#)
- [Size](#)
- [Sometimes](#)
- [Starts With](#)
- [String](#)
- [Timezone](#)
- [Unique (Database)](#)
- [Uppercase](#)
- [URL](#)
- [ULID](#)
- [UUID](#)

## Conditionally Adding Rules

#### Skipping Validation When Fields Have Certain Values

You may occasionally wish to not validate a given field if another field has a given value. You may accomplish this using the `exclude_if` validation rule. In this example, the `appointment_date` and `doctor_name` fields will not be validated if the `has_appointment` field has a value of `false` -

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_if:has_appointment,false|required|date',
    'doctor_name' => 'exclude_if:has_appointment,false|required|string',
]);
```

Alternatively, you may use `exclude_unless` rule to not validate a given field unless another field has a given value.

```php
$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
    'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
]);
```

#### Validating When Present

In some situations, you may wish to run validation checks against a field **only** if that field is present in the data being validated. To quickly accomplish this, add the `sometimes` rule to your rule list.

```php
$validator = Validator::make($data, [
    'email' => 'sometimes|required|email',
])
```

In the example above, the `email` field will only be validated if it is present in the `$data` array.

> If you are attempting to validate a field that should always be present but may be empty, check out [this note on optional fields](#a-note-on-optional-fields).

#### Complex Conditional Validation

Sometimes you may wish to add validation rules based on more complex conditional logic. For example, you may wish to require a given field only if another field has a greater value than 100. Or, you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your static rules that never change -

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'email' => 'require|email',
    'games' => 'required|numeric',
]);
```

Let's assume our web application is for game collectors. If a game collector registers with our application and they own more that 100 games, we want them to explain why they own so many games. For example, perhaps they run a game resale shop, or maybe they just enjoy collecting games. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

```php
$validator->sometimes('reason', 'required|max:500', function($input) {
    return $input->games >= 100;
});
```

The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is a list of the rules we want to add. If the closure passed as the thrid argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once.

```php
$validator->sometimes(['reason', 'cost'], 'required', function ($input) {
    return $input->games >= 100;
});
```

> The `$input` parameter passed to your closure will be an instance of `Illuminate\Support\Fluent` and may be used to access your input and files under validation.

#### Complex Conditional Array Validation

Sometimes you may want to validate a field based on another field in the same nested array whose index you do not know. In these situations, you may alow your closure to receive a second argument which will be the current individual item in the array being validated.

```php
$input = [
    'channels' => [
        [
            'type' => 'email',
            'address' => 'abigail@example.com',
        ],
        [
            'type' => 'url',
            'address' => 'https://example.com',
        ],
    ],
];

$validator->sometimes('channels.*.address', 'email', function($input, $item) {
    return $item->type === 'email';
});

$validator->sometimes('channels.*.address', 'url', function($input, $item) {
    return $item->type !== 'email';
})
```

Like the `$input` parameter passed to the closure, the `$item` parameter is an instance of `Illuminate\Support\Fluent` when the attribute data is an array; otherwise, it is a string.

## Validating Arrays

As discussed in the [array validation rule documentation][validation-rule-array], the `array` rule accepts a list of allowed array keys. If any additional keys are present within the array, validation will fail.

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:username,locale',
]);
```

In general, you should always specify the array keys that are allowed to be present within your array. Otherwise, the validator's `validate` and `validated` methods will return all of the validated data, including the array and all of its keys, even if those keys were not validated by other nested array validation rules.

### Validating Nested Array Input

You may use _dot notation_ to validate attributes within an array. For example, if the incoming HTTP request contains a `photos[profile]` field, you may validate it like so -

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

You may also validate each element of an array. For example, to validate that each email in a given input field is unique, you may do the following -

```php
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

Likewise, you may use the \* character when specifying [custom validation messages in your languagae files][validation-custom-messages-for-specific-attributes], making it a breeze to use a single validation message for array based fields.

```php
'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must have a unique email address',
    ],
],
```

#### Accessing Nested Array Data

Sometimes you may need to access the value for a given nested array element when assigning validation rules to the attribute. You may accomplish this using the `Rule::forEach` method. The `forEach` method accepts a closure that will be invoked for each iteration of the array attribute under validation and will receive the attribute's value and explicit, fully-expanded attribute name. The closure should return an array of rules to assign to the array element.

```php
use App\Rules\HasPermission;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$validator = Validator::make($request->all(), [
    'companies.*.id' => Rule::forEach(function($value, $attribute) {
        return [
            Rule::exists(Company::class, 'id'),
            new HasPermission('manage-company', $value),
        ];
    }),
]);
```

[validation-error-response-format]: #validation-error-response-format
[validation-named-error-bag]: https://laravel.com/docs/9.x/validation#named-error-bags
[requests-retrieving-old-input]: /basics/http-requests.md#retrieving-old-input
[session-flash-data]: /basics/session#flash-data
[validation-working-with-error-messages]: https://laravel.com/docs/9.x/validation#working-with-error-messages
[blade]: /basics/blade-templates.md
[session]: /basics/session.md
[service-container]: https://laravel.com/docs/9.x/container
[authorization]: https://laravel.com/docs/9.x/authorization
[routing-route-model-binding]: /basics/routing.md#route-model-binding
[facades]: https://laravel.com/docs/9.x/facades
[collections]: https://laravel.com/docs/9.x/collections
[localization]: https://laravel.com/docs/9.x/localization
[validation-rule-array]: #rule-array
[validation-custom-messages-for-specific-attributes]: #custom-messages-for-specific-attributes
