# Blade Templates

## Introduction

All Blade templates are compiled into plain PHP code and cached until the are modified, meaning Blade add essentially zero overhead to your application. Blade template files use the `.blade.php` file extension and are typically stored in the `resources/views` directory.

Blade views ma be returned from routes or controllers using the global `view` helper. As mentioned in the documentation of [views][views], data may be passed to the Blade view using the `view` helper's second argument.

```php
Route::get('/', function() {
    return view('greeting', ['name' => 'Finn']);
})
```

### Suppercharging Blade With Livewire

[Laravle Livewire][laravel-livewire] allows you to write Blade components that are augmented with dynamic functionality that would typically only be possible via frontend frameworks like React or Vue, providing a great approach to building modern, reactive frontends without the complexities, client-side rendering, or build steps of many JavaScript frameworks.

## Displaying Data

You may display data that is passed to your Blade views by wrapping the variable in curly braces. For example, given the following route:

```php
Route::get('/', function() {
    return view('welcome', ['name' => 'John']);
})
```

You may display the contents of the `name` variable like so:

```php
Hello, {{ $name }}.
```

> Blade's `{{ }}` echo statements are autometically sent throug PHP's htmlspecialchars function to prevent XSS attacks.

You are not limited to displaying the contents of the variables passed to the view. You may also echo the results of any PHP funciton. In fact, you can put any PHP code you wish inside of a Blade echo statement.

```php
The current UNIX timestamp is {{ time() }}.
```

### HTML Entity Encoding

By default, Blade (and the Laravel `e` helper) will double encode HTML entities. If you would like to disable double encoding, call the `Blade::withoutDoubleEncoding` method from the `boot` method of your `AppServiceProvider`.

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::withoutDoubleEncoding();
    }
}
```

#### Displaying Unescaped Data

By default, Blade `{{ }}` statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks. If you do not want your data to be escaped, you may use the following syntax.

```php
Hello, {!! $name !!}.
```

### Blade & JavaScript Frameworks

Since many JavaScript frameworks also use "curly" braces to indicate a given expression should be displayed in the browser, you may use the `@` symbol to inform the Blade rendering engine an expression should remain untouched. For example -

```php
<h1>Laravel</h1>

hello, @{{ name }}.
```

In this example, the `@` symbol will be removed by Blade. However, `{{ name }}` expression will remain untouched by the Blade engine, allowing it to be rendered by your JavaScript framework.

The `@` symbol may also be used to escape Blade directives -

```php
# blade template
@@if()

# HTML output
@if()
```

#### Rendering JSON

Sometimes you may pass an array to your view with the intention of rendering it as JSON in order to initialize a JavaScript variable. For example -

```php
<script>
  var app = <?php echo json_encode($array); ?>;
</script>
```

However, instead of manually calling `json_encode`, you may use the `Illuminate\Support\Js::from` method directive. The `from` method accepts the same arguments as PHP's `json_encode` function; however, it will ensure that the resulting JSON is properly escaped for inclusion within HTML quotes. The `from` method will return a string `JSON.parse` JavaScript statement that will convert the given object or array into a valid JavaScript object.

```php
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

The latest versions of the Laravel application skeleton include a `Js` facade, which provides convenient access to this functionality within your Blade templates.

```php
<script>
    var app = {{ Js::from($array) }};
</script>
```

> You should only use the `Js::from` method to render existing variables as JSON. The Blade templating is based on regular expressions and attempts to pass a complex expression to the directive may cause unexpected failures.

#### The `@verbatim` Directive

If you are displaying JavaScript variables in a large portion of your template, you may wrap the HTML in the `@varbatim` directive so that you do not have to prefix each Blade echo statement with an `@` symbol.

```php
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## Blade Directives

### If Statements

You may construct `if` statements using the `@if, @elseif, @else`, and `@endif` directives. These directives function identically to thier PHP counterparts.

```php
@if(count($records) === 1)
    "I have one record!"
@elseif(count($records) > 1)
    "I have multiple records!"
@else
    "I don't have any records!"
@endif
```

For convenience, Blade also provides an `@unless` directive.

```php
@unless(Auth::check())
    "You are not signed in."
@endunless
```

The `@isset` and `@empty` directives may be used as convenient shortcuts for their respcetive PHP functions

```php
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

#### Authentication Directives

The `@auth` and `@guest` directives may be used to quickly determine if the current user is [authenticated][authentication] or is a guest.

```php
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

If needed, you may specify the authentication guard that should be checked when using the `@auth` and `@guest` directives.

```php
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

#### Environment Directives

You may check if the application is running in the production environment using the `@production` directive.

```php
@production
    // Production specific content...
@endproduction
```

Or, you may determine if the application is running in a specific enviroment using the `@env` directive.

```php
@env('staging')
    // The application is running in "staging"...
@endenv

@env(['staging', 'production'])
    // The application is running in "staging" or "production"...
@endenv
```

#### Section Directives

You may determine if a template inheritance section has content using the `@hasSection` directive.

```php
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

You may use the `@sectionMission` directive to determine if a section does not have content.

```php
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

### Switch Statements

Switch statements can be constructed using the `@switch, @case, @break, @default` and `@endswitch` directives.

```php
@switch($i)
    @case(1)
        First case...
        @break
    @case(2)
        Second case...
        @break
    @default
        Default case...
@endswitch
```

### Loops

Blade provides simple directives for working with PHP loop structures.

```php
@for($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while(true)
    <p>I am looping forever.</p>
@endwhile
```

> While iterating through a `foreach` loop, you may use the [loop variable][blade-loop-variable] to gain valuable information about the loop, such as whether you are in the first or last iteration through the loop.

When using loops you may also skip the current iteration or end the loop using the `@continue` or `@break` directives.

```php
@foreach($users as $user)
    @if($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if($user->number == 5)
        @break
    @endif
@endforeach
```

You may also include the continuation or break condition within the directive declaration.

```php
@foreach($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number ==5)
@endforeach
```

### The Loop Variable

While iterating through a `foreach` loop, a `$loop` variable will be available inside of your loop. This variable provides access to some useful bits of information such as the current loop index and whether this is the first or last iteration through the loop.

```php
@foreach($users as $user)
    @if($loop->first)
        This is the first iteration.
    @endif

    @if($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

If you are in a nested loop, you may access the parent loop's `$loop` variable via the `parent` property.

```php
@foreach($users as $user)
    @foreach($user->posts as $post)
        @if($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

The `$loop` variable also contains a variety of other useful properties -

| Property           | Description                                            |
| ------------------ | ------------------------------------------------------ |
| `$loop->index`     | The index of the current loop iteration (starts at 0). |
| `$loop->iteration` | The current loop iteration (starts at 1).              |
| `$loop->remaining` | The iterations remaining in the loop.                  |
| `$loop->count`     | The total number of items in the array being iterated. |
| `$loop->first`     | Whether this is the first iteration through the loop.  |
| `$loop->last`      | Whether this is the last iteration through the loop.   |
| `$loop->even`      | Whether this is an even iteration through the loop.    |
| `$loop->odd`       | Whether this is an odd iteration through the loop.     |
| `$loop->depth`     | The nesting level of the current loop.                 |
| `$loop->parent`    | When in a nested loop, the parent's loop variable.     |

### Conditional Classes

The `@class` directive conditionally complies a CSS class string. The directive accepts an array of classes where the array key contains the class or classes you wish to add, while the value is a boolean expression. If the array element has a numric key, it will always be included in the rendered class list.

```php
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-grey-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

# output
<span class="p-4 text-grey-500 bg-red"></span>
```

### Additional Attributes

You may use the `@checked` directive to easily indicate if a given HTML checkbox input is "checked". This directive will echo `checked` if the provided condition evaluates to `true`.

```php
<input type="checkbox"
    name="active"
    value="active"
    @checked(old('active', $user->active)) />
```

The `@selected` directive may be used to indicate if a given select options should be "selected".

```php
<select name="version">
    @foreach($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

The `@disabled` directive may be used to indicate if a given element should be "disabled".

```php
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

The `@readonly` directive may be used to indicate if a given element should be "readonly".

```php
<input type="email"
    name="email"
    value="email@laravel.com"
    @readonly($user->isNotAdmin()) />
```

The `@required` directive may be used to indicate if a given element should be "required".

```php
<input type="text"
    name="title"
    value="title"
    @required($user->isAdmin()) />
```

### Including Subviews

Blade's `@include` directive allows you to include a Blade view from within another view. All variables that are available to the parent view will be made available to the included view.

```php
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Even though the included view will inherit all data available in the parent view, you may also pass an array of additional data that should be made available to the included view.

```php
@include('view.name', ['status' => 'complete'])
```

If you attempt to `@include` a view which does not exist, Laravel will throw an error. If you would like to inclde a view that may not be present, you should use the `@includeIf` directive.

```php
@includeIf('view.name', ['status' => 'complete'])
```

If you would like to `@include` a view if a given boolean expression evaluates to `true` or `false`, you may use the `@includeWhen` and `@includeUnless` directives.

```php
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

To include the first view that exists from a given array of views, you may use the `@includeFirst` directive.

```php
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

> You should avoid using the `__DIR__` and `__FILE__` constants in your Blade views, since they will refer to the location of the cached, compiled view.

#### Rendering Views For Collections

You may combine loops and includes into one line with Blade's `@each` directive.

```php
@each('view.name', $jobs, 'job')
```

The `@each` directive's first argument is the view to render for each element in the array or collection. The second argument is the array or collection you wish to iterate over, while the third argument is the variable name that will be assigned to the current iteration within the view. So, for example, if you are iterating over an array of `jobs`, typically you will want to access each job as a `job` variable within the view. The array key for the current iteration will be available as the `key` variable within the view.

You may also pass a fourth argument to the `@each` directive. This argument determines the view that will be rendered if the given array is empty.

```php
@each('view.name', $jobs, 'job', 'view.empty')
```

> Views rendered via `@each` do not inherit the variables from the parent view. If the child view required these variables, you should use the `@foreach` and `@include` directives instead.

### The `@once` Directive

The `@once` directive allows you to define a portion of the template that will only be evaluated once per rendering cycle. This may be useful for pushing a given piece of JavaScript into the page's header using [stacks][blade-stacks]. For example, if you are rendering a given [component][blade-components] within a loop, you may wish to only push the JavaScript to the header the first time the component is rendered.

```php
@once
    @push('scripts')
        <script>
            // You custom javascript...
        </script>
    @endpush
@endonce
```

Since the `@once` directive is often used in conjuction with the `@push` or `@prepend` directives, the `@pushOnce` and `@prependOnce` directives are available for your convenience.

```php
@pushOnce('scripts')
    <script>
        // Your custom javascript...
    </script>
@endPushOnce
```

### Raw PHP

In some situations, it's useful to embed PHP code into your views. You can use the Blade `@php` directive to execute a block of plain PHP within your template.

```php
@php
    $counter = 1;
@endphp
```

If you only need to write a single PHP statement, you can include the statement within the `@php` directive.

```php
@php($counter = 1;)
```

### Comments

Blade also allows you to define comments in your views. However, unlike HTML comments, Blade comments are not included in the HTML returnedd by your application.

```php
{{-- This comment will not be present in the rendered HTML --}}
```

## Components

Components and slots provide similar benefits to sections, layouts and includes. However, some may find the mental model of components and slots easier to understand. There are two approaches to writing components: class based components and anonymous components.

To create a class based component, you may use the `make:component` Artisan command. To illustrate how to use components, we will create a simple `Alert` component. The `make:component` command will place the component in the `app/View/Compoennts` directory.

```sh
php artisan make:component Alert
```

The `make:component` command will also create a view template for the component. The view will be placed in the `resources/views/components` directory. When writing components for your own application, components are autometically discovered within the `app/View/Components` directory and `resources/views/components` directory, so no further component registration is typically required.

You may also create components within subdirectories.

```sh
php artisan make:component Forms/Input
```

The command above will create an `Input` component in the `app/View/Components/Forms` directory and the view will be placed in the `resources/views/components` directory.

If you would like to create an anonymous component (a component with only a Blade template and no class), you may use the `--view` flag when invoking the `make:component` command.

```sh
php artisan make:component forms.input --view
```

The command above will create a Blade file at `resources/views/components/forms/input.blade.php` which can be rendered as a component via `<x-forms.input />`.

### Manually Registering Package Components

If you are building a package that utilizes Blade components, you will need to manually register your component class and its HTML tag alias. You should typically register your components in the `boot` method of your package's service provider.

```php
use Illumiate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::component('package-alert', Alert::class);
}
```

Once your component has been registered, it may be rendered using its tag alias.

```php
<x-package-alert />
```

Alternatively, you may use teh `componentNamespace` method to autoload component classes by convention. For example, a `Nightshade` package might have `Calendar` and `ColorPicker` components that reside within the `Package\Views\Components` namespace.

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

This will allow the usage of package components by their vendor namespace using the `package-name::` syntax.

```php
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade will autometically detect the class that's linked to this component by pascal-casing the component name. Subdirectories are also supported using "dot" notation.

### Rendering Components

To display a component, you may use a Blade component tag within one of your Blade templates. Blade component tags start with the string `x-` followed by the _kebab case_ name of the component class.

```php
<x-alert />

<x-user-profile />
```

If the component class is nested within the `app/View/Components` directory, you may use the `.` character to indicate directory nesting. For example, if we assume a component is located at `app/View/Components/Inputs/Button.php`, we may render it like so -

```php
<x-input.button />
```

### Passing Data To Components

You may pass data to Blade components using HTML attributes. Hard-coded, primitive values may be passed to the component using simple HTML attribute strings. PHP expressions and variables should be passed to the component via attributes that use the `:` character as a prefix.

```php
<x-alert type="error" :message="$message" />
```

You should define all of the component's data attributes in its class constructor. All public properties on a component will autometically be made available to the component's view. It is not necessary to pass the data to the view from the component's `render` method.

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The alert type.
     *
     * @var string
     */
    public $type;

    /**
     * The alert message.
     *
     * @var string
     */
    public $message;

    /**
     * Create the component instance.
     *
     * @param string $type
     * @param string $message
     * @return void
     */
    public function __construct($type, $message)
    {
        $this->type = $type;
        $this->message = $message;
    }

    /**
     * Get the view / contents that represent the component.
     *
     * @return \Illuminate\View\View|\Closure|string
     */
    public function render()
    {
        return view('components.alert');
    }
}
```

When your component is rendered, you may display the conents of your component's public variables by echoing the variables by name.

```php
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

#### Casing

Component constructor arguments should be specified using `camelCase`, while `kebab-case` should be used when referencing the argument names in your HTML attributes. For example, given the following component constructor -

```php
/**
 * Create the component instance.
 *
 * @param string $alertType
 * @return void
 */
public function __construct($alertType)
{
    $this->alertType = $alertType;
}
```

The `$alertType` argument may be provided to the component like so -

```php
<x-alert alert-type="danger" />
```

#### Short Attribute Syntax

When passing attributes to components, you may also use a **short attribute** syntax. This often convenient since attribute names frequently match the variable names they correspond to.

```php
# Short attribute syntax...
<x-profile :$userId :$name />

# Is equivalent to...
<x-profile :user-id="$userId" :name="$name" />
```

#### Escaping Attribute Rendering

Since some JavaScript frameworks such as Alpine.js also use colon-prefixed attributes, you may use a double colon (`::`) prefix to inform Blade that the attribute is not a PHP expression. For example, given the following component -

```php
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

The following HTML will be rendered by Blade -

```php
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

#### Component Methods

In addition to public variables being available to your component template, any public methods on the component may be invoked. For example, imagine a component that has an `isSelected` method.

```php
/**
 * Determine if the given option is the currently selection option.
 *
 * @param string $option
 * @return bool
 */
public function isSelected($option)
{
    return $option === $this->selected;
}
```

You may execute this method from your component template by invoking the variable matching the name of the method.

```php
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

#### Accessing Attributes & Slots Within Component Class

Blade components also allow you to access the component name, attributes and slot inside the class's render method. However, in order to access this data, you should return a closure from your component's `render` method. The closure will receive a `$data` array as its only argument. This array will contain several elements that provide information about the component.

```php
/**
 * Get the view / contents that represent the component.
 *
 * @return \Illuminate\View\View|\Closure|string
 */
public function render()
{
    return function (array $data) {
        // $data['componentName'];
        // $data['attributes'];
        // $data['slot'];

        return "<div>Components content</div>";
    };
}
```

The `componentName` is equal to the name used in the HTML tag after `x-` prefix. So `<x-alert />`'s `componentName` will be `alert`. The `attributes` element will contain all of the attributes that were presetn on the HTML tag. The `slot` element is an `Illuminate\Support\HtmlString` instance with the contents of the component's slot.

The closure should return a string. If the returned string corresponds to an existing view, that view will be rendered; otherwise the returned string will be evaluated as an inline Blade view.

```php
/**
 * Get the view / contents that represent the component.
 *
 * @return \Illuminate\View\View|\Closure|string
 */
public function render()
{
    return function (array $data) {
        // $data['componentName'];
        // $data['attributes'];
        // $data['slot'];

        return "forms.input"; // view('forms.input')
    };
}
```

#### Additional Dependencies

If your component requires dependencies from Laravel's [service container][container], you may list them before any of the component's data attributes and they will autometically be injected by the container.

```php
use App\Service\AlertCreator;

/**
 * Create the component instance.
 *
 * @param \App\Services\AlertCreator $creator
 * @param string $type
 * @param string $message
 * @return void
 */
public function __construct(AlertCreator $creator, $type, $message)
{
    $this->creator = $creator;
    $this->type = $type;
    $this->message = $message;
}
```

#### Hiding Attributes / Methods

If you would like to prevent some public methods or properties from being exposed as variables to your component template, you may add them to an `$except` array property on your component.

```php
<?php
namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The alert type
     *
     * @var string
     */
    public $type;

    /**
     * The properties / methods that should not be exposed to the component template.
     *
     * @var string
     */
    protected $except = ['type'];
}
```

### Component Attributes

You may need to specify additional HTML attributes, such as `class`, that are not part of the data required for a component to function. Typically, you want to pass these additional attributes down to the root element of the component template. For example, imagine we want to render an `alert` component like so -

```php
<x-alert type="error" :message="$message" class="mt-4" />
```

All of the attributes that are not part of the component's constructor will autometically be added to the component's **attribute bag**. This attribute bag is automatically made available to the component via the `$attributes` variable. All of the attributes may be rendered within the component by echoing this variable.

```php
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

> Using directives such as `@env` within component tags is not supported at this time. For example, `<x-alert :live="@env('production')" />`

#### Default / Merged Attributes

Sometimes you may need to specify default values for attributes or merge additional values into some of the component's attributes. To accomplish this, you may use the attribute bag's `merge` method. This method is particularly useful for defining a set of default CSS classes that should always be applied to a component.

```php
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

If we assume this component is utilized like so -

```php
<x-alert type="error" :message="$message" class="mb-4" />
```

The final, rendered HTML of the component will appear like the following -

```php
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

#### Conditionally Merge Classes

Sometimes you may wish to merge classes if a given condition is `true`. You can accomplish this via the `class` method, which accepts an array of classes where the array key contains the class or classes you wish to add, while the value is a boolean expression. If the array element has a numeric key, it will always be included in the rendered class list.

```php
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

If you need to merge other attributes onto your component, you can chain the `merge` method onto the `class` method.

```php
<div {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</div>
```

> If you need to conditionally compile classes on other HTML elements that shouldn't receive merge attributes, you can use the [`@class` directive][blade-class-directive]

#### Non-Class Attribute Merging

When merging attributes that are not `class` attributes, the values provided to the `merge` method will be considered the **default** values of the attribute. However, unlike the `class` attribute, these attributes will not be merged with injected attribute values, instead, they will be overwritten. For example, a `button` component's implementation may look like the following -

```php
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

To render the button component with a custom `type`, it may be specified when consuming the component. If no type is specified, the `button` type will be used.

```php
<x-button type="submit">Submit</x-button>
```

The rendered HTML of the `button` component in this example would e -

```php
<button type="submit">Submit</button>
```

If you would like an attribute other than `class` to have its default value and injected values joined together, you may use the `prepends` method. In this example, the `data-controller` attribute will always begin with `profile-controller` and any additional injected `data-controller` values will be placed after this default value -

```php
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

#### Retrieving & Filtering Attributes

You may filter attributes using the `filter` method. This method accepts a closure which should return `true` if you wish to retain the attribute in the attribute bag.

```php
{{ $attributes->filter(fn ($value, $key) => $key == 'foo') }}
```

For convenience, you may use the `whereStartsWith` method to retrieve all attributes whose keys begin with a given string.

```php
{{ $attributes->whereStartsWith('wire:model') }}
```

Conversely, the `whereDoesntStartWith` method may be used to exclude all attributes whose keys begin with a given string.

```php
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

Using the `first` method, you may render the first attribute in a given attribute bag -

```php
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

If you would like to check if an attribute is present on the component, you may use the `has` method. This method accepts the attribute name as its only argument and returns a boolean indicating whether or not the attribute is present.

```php
@if($attributes->has('class'))
    <div>"Class attribute is present"</div>
@endif
```

You may retrieve a specific attribute's value using the `get` method.

```php
{{ $attributes->get('class') }}
```

### Reserved Keywords

By default, some keywords are reserved for Blade's internal use in order to render components. The following keywords cannot be defined as public properties or method names within your components.

- data
- render
- resolveView
- shouldRender
- view
- withAttributes
- withName

### Slots

You will often need to pass additional content to your component via **slots**. Component slots are rendered by echoing the `$slot` variable. To explore this concept, let's imagine that an `alert` component has the following markup -

```php
<!-- resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

We may pass content to the `slot` by injecting content into the component.

```php
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

Sometimes a component may need to render multiple different slots in different locations within the component. Let's modify our alert component to allow for the injection of a **title** slot.

```php
<span class="alert-title">{{ $title }}</span>
<div class="alert alert-danger">
    {{ $slot }}
</div>
```

You may define the content of the named slot using the `x-slot` tag. Any content not within an explicit `x-slot` tag will be passed to the component in the `$slot` variable.

```php
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

#### Scoped Slots

If you have used a JavaScript framework such as _Vue_, you may be familiar with **scoped slots**, which allow you to access data or methods from the component within your slot.. You may achieve similar behavior in Laravel by defining public methods or properties on your component and accessing the component within your slot via the `$component` variable. In this example, we will assume that the `x-alert` component has a public `formatAlert` method defined on its component class -

```php
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

#### Slot attributes

Like Blade components, you may assign additional [attributes][blade-component-attributes] to slots such as CSS class names -

```php
<x-alert class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>

    Content

    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-alert>
```

To interact with slot attributes, you may access the `attributes` property of the slot's variable.

```php
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

### Inline Component Views

For very small components, it may feel cumbersome to manage both the component class and the component's view template. For this reason, you may return the component's markup directly from the `render` method.

```php
/**
 * Get the view / contents that represent the component.
 *
 * @return \Illuminate\View\View|\Closure|string
 */
public function render()
{
    return <<<'blade'
        <div class="alert alert-danger">
            {{ $slot }}
        </div>
    blade;
}
```

#### Generating Inline View Components

To create a component that renders an inline view, you may use the `inline` option when executing the `make:component` command.

```php
php artisan make:component Alert --inline
```

### Dynamic Components

Sometimes you may need to render a component but not know which component should be rendered until runtime. In this situation, you may use Laravel's built-in `dynamic-component` component to render the component based on a runtime value or variable.

```php
<x-dynamic-component :component="$componentName" class="mt-4" />
```

### Manually Registering Components

When writing components for your own application, components are automatically discovered within the `app/View/Components` directory and `resources/views/components` directory.

However, if you are building a package that utilizes Blade components or placing components in non-conventional directories, you will need to manually register your component class and its HTML tag alias so that Laravel knows where to find the component. You should typically register your components in the `boot` method of your package's service provider.

```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::component('package-alert', AlertComponent::class);
}
```

Once your component has been registered, it may be rendered using its tag alias.

```php
<x-package-alert />
```

#### Autoloading Package Components

Alernatively, you may use the `componentNamespace` method to autoload component classes by convention. For example, a `Nightshade` package might have `Calendar` and `ColorPicker` components that reside within the `Package\Views\Components` namespace -

```php
use Illuminatte\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 *
 * @return void
 */
public function boot()
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

This will allow the usage of package components by their vendor namespace using the `package-name::` syntax -

```php
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade will automatically detect the class that's linked to this component by pascal-casing the component name. Subdirectories are also supported using **dot** notation.

## Anonymous Components

Similar to inline components, anonymous components provide a mechanism for managing a component via a single file. However, anonymous components utilize a single view file and have no associated class. To define an anonymous component, you only need to place a Blade template within your `resources/views/components` directory. For example, assuming you have defined a component at `resources/views/components/alert.blade.php`, you may simply render it like so -

```php
<x-alert />
```

You may use the `.` character to indicate if a component is nested deeper inside the `components` directory. For example, assuming the component is defined at `resources/views/components/inputs/button.blade.php`, you may render it like so -

```php
<x-input.button />
```

### Anonymous Index Components

Sometimes, when a component is made up of many Blade templates, you may wish to group the given component's templates within a single directory. For example, image an _accordion_ component with the following directory structure -

```
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

This directory structure allows you to render the accrodion component and its item like so -

```php
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

However, in order to render the accordion component via `x-accordion`, we were forced to place the _index_ accordion component template in the `resources/views/components` directory instead of nesting it within the `accordion` directory with the other accordion related templates.

Blade allows you to place an `index.blade.php` file within a component's template directory. When an `index.blade.php` template exists for the component, it will be rendered as the **root** node of the component. So, we can use the same Blade syntax given in the example above, however, we will adjust our directory structure like so -

```
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
```

### Data Properties / Attributes

You may specify which attributes should be considered data variables using the `@props` directive at the top of your component's Blade template. All other attributes on the component will be available via the component's attribute bag. If you wish to give a data variable a default value, you may specify the variable's name as the array key and the default value as the array value.

```php
<!-- resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => "alert alert-$type"]) }}>
    {{ $message }}
</div>
```

Given the component definition above, we may render the component like so -

```php
<x-alert type="error" :$message class="mb-4" />
```

### Accessing Parent Data

Sometimes you may want to access data from a parent component inside a child component. In these cases, you may use the `@aware` directive. For example, imagine we are building a complex menu component consisting of a parent `<x-menu>` and child `<x-menu.item>` -

```php
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

The `<x-menu>` component may have an implementation like so -

```php
<!-- resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => "bg-${color}-200"]) }}>
    {{ $slot }}
</ul>
```

Because the `color` prop was only passed into the parent (`<x-menu>`), it won't be available inside `<x-menu.item>`. However, if we use the `@aware` directive, we can make it available inside `<x-menu.item>` as well.

```php
<!-- resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => "text-${color}-800"]) }}>
    {{ $slot }}
</li>
```

> The `@aware` directive can not access parent data that is not explicitly passed to the parent component via HTML attributes. Default `@props` values that are not explicitly passed to the parent component can not be accessed by the `@aware` directive.

### Anonymous Component Paths

As previously discussed, anonymous components are typically defined by placing a Blade template within your `resources/views/components` directory. However, you may occasionally want to register other anonymous component paths with Laravel in addition to the default path.

The `anonymousComponentPath` method accepts the **path** to the anonymous component location as its first argument and an optional **namespace** that components should be placed under as its second argument. Typically, this method should be called from the `boot` method of your application's [service providers][providers].

```php
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blade::anonymousComponentPath(__DIR__.'/../components');
}
```

When component paths are registered without a specific prefix as in the example above, they may be rendered in your Blade components without a corresponding prefix as well. For example, if a `panel.blade.php` component exists in the path registered above, it may be rendered like so -

```php
<x-panel />
```

Prefix **namespaces** may be provided as the second argument to the `anonymousComponentPath` method.

```php
Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');
```

When a prefix is provided, components within that **namespace** may be rendered by prefixing to the component's namespace to the component name when the component is rendered.

```php
<x-dashboard:panel />
```

## Building Layouts

### Layouts Using Components

#### Defining The Layout Component

For example, imagine we are building a **todo** list application. We might define a `layout` component that looks like the following -

```php
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr />
        {{ $slot }}
    </body>
</html>
```

#### Applying The Layout Component

Once the `layout` component has been defined, we may create a Blade view that utilizes the component. In this example, we will define a simple view that displays our task list -

```php
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

Content that is injected into a component will be supplied to the default `$slot` variable within our `layout` component. Our `layout` also respects a `$title` slot if one is provided, otherwise, a default title is shown. We may inject a custom title from our task list view using the standard slot syntax discussed in the [component documentation][blade-components].

```php
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        Custom title
    </x-slot>

    @foreach($tasks as $task)
        {{ $task }}
    @endforeach
</x-layout>
```

Now that we have defined our layout and taks list views, we just need to return the `task` view from a route.

```php
use App\Models\Task;

Route::get('/tasks', function() {
    return view('tasks', ['tasks' => Task::all()]);
});
```

### Layouts Using Template Inheritance

#### Defining A Layout

Layouts may also be created via **template inheritance**. This was the primary way of building applications prior to the introduction of [components][blade-components].

Since most web applications maintain the same general layout across various pages, it's convenient to define layout as a single Blade view.

```php
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

This file contains typical HTML mark-up. However, take note of the `@section` and `@yield` directives. The `@section` directive, as the name implies, defines a section of content, while the `@yield` directive is used to display the contents of a given section.

#### Extending A Layout

When defining a child view, use the `@extends` directive to specify which layout the child view should **inherit**. Views which extend a Blade layout may inject content into the layout's sections using `@section` directives. The contents of these sections will be displayed in the layout using `@yield`.

```php
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

In this example, the `sidebar` section is utilizing the `@parent` directive to append (rather that overwriting) content to the layout's sidebar. The `@parent` directive will be replaced by the content of the layout when the view is rendered.

> Contrary to the previous example, this `sidebar` section ends with `@endsection` instead of `@show`. The `@endsection` directive will only define a section while `@show` directive will define and **immediately yield** the section.

The `@yield` directive also accepts a default value as its second parameter. This value will be rendered if the section being yielded is undefined.

```php
@yield('content', 'Default content')
```

## Forms

### CSRF Field

Anytime you define an HTML form in your application, you should include a hidden CSRF token field in the form so that [the CSRF protection][csrf] middleware can validate the request. You may use the `@csrf` directive to generate the token field.

```php
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

### Method Field

Since HTML forms can't make `PUT`, `PATCH` or `DELETE` requests, you will need to add a hidden `_method` field to spoof these HTTP verbs. The `@method` directive can create this field for you.

```php
<form method="POST" action="/profile">
    @method('PUT')

    ...
</form>
```

### Validation Errors

The `@error` directive may be used to quickly check if [validation error messages][validation-errors] exist for a given attribute. Within an `@error` directive, you may echo the `$message` variable to display the error message.

```php
<!-- resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input
    id="title"
    type="text"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Since the `@error` directive complies to an **if** statement, you may use the `@else` directive to render content when there is not an error for an attribute.

```php
<!-- resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input
    id="email"
    type="text"
    class="@error('email') is-invalid @else is-valid @enderror">

@error('email')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

You may pass [the name of a specific error bag][validation-named-error-bags] as the second parameter to the `@error` directive to retrieve validation error messages on pages containing multiple forms.

```php
<!-- resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input
    id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror">

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

## Stacks

Blade allows you to push to named stacks which can be rendered somewhere else in another view or layout. This can be particularly useful for specifying any JavaScript libraries required by your child views.

```php
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

If you would like to `@push` content if a given boolean expression evaluates to `true`, you may use the `@pushIf` directive.

```php
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

You may push to a stack as many times as needed. To render the complete stack contents, pass the name of the stack to the `@stack` directive.

```php
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

If you would like to prepend content onto the beginning of a stack, you should use the `@prepend` directive.

```php
@push('scripts')
    This will be second
@endpush

// Later...

@prepend('scripts')
    This will be first
@endprepend
```

## Service Injection

The `@inject` directive may be used to retrieve a service from the Laravel [service container][container]. The first argument passed to `@inject` is the name of the variable the service will be placed into, while the second argument is the class or interface name of the service you wish to resolve.

```php
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

## Rendering Inline Blade Templates

Sometimes you may need to transform a raw Blade template string into valid HTML. You may accomplish this using the `render` method provided by the `Blade` facade. The `render` method accepts the Blade template string and an optional array of data to provide to the template.

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Jane Smith']);
```

Laravel renders inline Blade templates by writing them to the `storage/framework/views` directory. If you would like Laravel to remove these temporary files after rendering the Blade template, you may provide the `deleteCachedView` argument to the method.

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Jane Smith'],
    deleteCachedView: true,
);
```

## Rendering Blade Fragments

When using frontend frameworks such as [Turbo][turbo] and [htmx][htmx], you may occasionally need to only return a portion of a Blade template within your HTTP response. Blade _fragments_ allow you to do just that. To get started, place a portion of your Blade template within `@fragment` and `@endfragment` directives.

```php
@fragment('user-list')
    <ul>
        @foreach($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

Then, when rendering the view that utilizes this template, you may invoke the `fragment` method to specify that only the specified fragment should be included in the outgoing HTTP response.

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

## Extending Blade

Blade allows you to define your own custom directives use the `directive` method. When the Blade compiler encounters the custom directive, it will call the provided callback with the expression that the directive contains.

The following example creates a `@datetime($var)` directive which formats a given `$var`, which should be an instance of `DateTime` -

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::directive('datetime', function($expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}

```

As you can see, we will chain the `format` method onto whatever expression is passed into the directive. So, in this example, the final PHP generated by this directive will be -

```php
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

> After updating the logic of Blade directive, you will need to delete all of the cached Blade views. The cached Blade views may be removed using the `view:clear` Artisan command.

### Custom Echo Handlers

If you attempt to _echo_ an object using Blade, the object's `__toString` method will be invoked. The `__toString` method is one of the PHP's built-in _magic methods_. However, sometimes you may not have control over the `__toString` method of a given class, such as when the class that you are interacting with belongs to a third-party library.

In these cases, Blade allows you to register a custom echo handler for that particular type of object. To accomplish this, you should invode Blade's `stringable` method. The `stringable` method accepts a closure. This closure should type-hint the type of object that it is responsible for rendering. Typically, the `stringable` method should be invoked within the `boot` method of you application's `AppServiceProvider` class.

```php
use Illuminate\Support\Facades\Blade;
use Money\Money;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blade::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

Once your custom echo handler has been defined, you may simply echo the object in your Blade template.

```php
Cost: {{ $money }}
```

### Custom If Statements

Programming a custom directive is sometimes more complex that necessary when defining simple, custom conditional statements. For that reason, Blade provides a `Blade::if` method which allows you to quickly define custom conditional directives using closures. For example, let's define a custom conditional that checks the configured default _disk_ for the application. We may do this in the `boot` method of our `AppServiceProvider`.

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Blade::if('disk', function ($value) {
        return config('filesystems.default') === $value;
    });
}
```

Once the custom conditional has been defined, you can use it within your templates.

```php
@disk('local')
    <!-- The applicaiton is using the local disk... -->
@elsedisk('s3')
    <!-- The applicaiton is using the s3 disk... -->
@else
    <!-- The applicaiton is using some other disk... -->
@endisk

@unlessdisk('local')
    <!-- The applicaiton is not using the local disk... -->
@enddisk
```

[views]: /basics/views.md
[laravel-livewire]: https://laravel-livewire.com
[authentication]: https://laravel.com/docs/9.x/authentication
[blade-loop-variable]: #the-loop-variable
[blade-stacks]: #stacks
[blade-components]: #components
[container]: https://laravel.com/docs/9.x/container
[blade-class-directive]: #conditional-classes
[blade-component-attributes]: #component-attributes
[providers]: https://laravel.com/docs/9.x/providers
[csrf]: /basics/csrf-protection.md
[validation-errors]: https://laravel.com/docs/9.x/validation#quick-displaying-the-validation-errors
[validation-named-error-bags]: https://laravel.com/docs/9.x/validation#named-error-bags
[turbo]: https://turbo.hotwired.dev
[htmx]: https://htmx.org
