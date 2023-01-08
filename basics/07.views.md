# Views

## Introduction

Views separate your controller / application logic from your presentation logic and are stored in the `resources/views` directory. When using Laravel, view templates are usually written using the [Blade templating language][blade]. A simple view might look something like this:

```php
<!-- view stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Since this view is stored at `resources/views/greeting.blade.php`, we may return it using the global `view` helper like so:

```php
Route::get('/', function() {
    return view('greeting', ['name' => 'John']);
});
```

### Writing Views In React / Vue

Laravel makes writing frontend templates using React or Vue painless thanks to [Insertia][inertia], a library that makes it a cinch to tie your React / Vue frontend to your Laravel backend without the typical complexities of building an SPA.

Breeze and Jetstream [starter kits][starter-kits] give you a great starting point for your next Laravel application powered by Inertia. In addition, [Laravel Bootcamp][laravel-bootcamp] provides a full demonstration of building a Laravel application powered by Inertia, including examples in Vue and React.

## Creating & Rendering Views

You may create a view by placing a file with the `.blade.php` extension in your application's `resource/views` directory. The `.blade.php` extension informs the framework that the file contains a [Blade template][blade]. Blade templates contain HTML as well as Blade directives that allow you to easily echo values, create "if" statements, iterate over data and more.

Once you have created a view, you may return it from one of your application's routes or controllers using the global `view` helper.

```php
Route::get('/', function() {
    return view('greeting', ['name' => 'John']);
});
```

Views may also be returned using the `View` facade.

```php
use App\Support\Facades\View;

return View::make('greeting', ['name' => 'Jane']);
```

### Nested View Directives

Views may also be nested within subdirectories of the `resources/views` directory. "Dot" notaion may be used to reference nested views. For example, if your view is stored at `resources/views/admin/profile.blade.php`, you may return it from one of your applications routes / controller like so:

```php
return view('admin.profie', $data);
```

### Creating The First Available View

Using the `View` facade's `first` method, you may create the first view that exists in a given array of views. This may be useful if your application or package allows views to be customized or overwritten.

```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

### Determine If A View Exists

If you need to determine if a view exists, you may use the `View` facade. The `exists` method will return `true` if the view exists.

```php
use Illuminate\Support\Facades\View;

if(View::exists('emails.customer')) {
    //
}
```

## Passing Data To Views

As you saw in the previous examples, you masy pas an array of data to views to make that data available to the view.

```php
return view('greetings', ['name' => 'Jane']);
```

When passing information in this manner, the data should be an array with key / value pairs. After providing data to a view, you can then access each value within your view using the data's keys, such as `<?php echo $name; ?>`.

As an alternative to passing a complete array of data to the `view` helper function, you may use the `with` method to add individual pieces of data to the view. The `with` method returns an instance of the view object so that you can continue chaining methods before returing the view.

```php
return view('greeting')
        ->with('name', 'John')
        ->with('occupation', 'Software Developer');
```

#### Sharing Data With All Views

You may need to share data with all views that are rendered by your application. You may do so using the `View` facade's `share` method. Typically, you should place calls to the `share` method within a service provider's `boot` method. You are free to add them to the `App\Providers\AppServiceProvider` class or generate a separate service provider to house them.

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

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
        View::share('key', 'value');
    }
}
```

## View Composers

View composers are callbacks or class methods that are called when a view is rendered. If you have data that you want to be bound to a view each time that view is rendered, a view composer can help you organize that logic into a single location. View composers may prove particularly useful if the same view is returned by multiple routes or controllers within your application and always needs a particular piece of data.

Typically, view composers are registered within one of your application's [service providers][providers]. In this example, we'll assume that we have created a new `App\Providers\ViewServiceProvider` to house this logic.

We'll use the `View` facades `composer` method to register the view composer. Laravel does not include a default directory for class based view composers, so you are free to organize them howerver you wish. For example, you could create an `app/View/Composers` directory to house all of your applications's view composers.

```php
<?php

namespace App\Providers;

use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ViewServiceProvider extends ServiceProvider
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
        // Using class based composers...
        View::composer('profile', ProfileComposer::class);

        // Using function based composers...
        View::composer('dashboard', function($view) {
            //
        });
    }
}
```

> If you create a new service provider to contain your view composer registration, you will need to add the service provider to the `providers` array in the `config/app.php` configuration file.

Now that we have registered the composer, the `composer` method of the `App\View\Composers\ProfileComposer` class will be executed each time the `profile` view is being renewed. Let's take a look at an example of the composer class:

```php
<?php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * The user repository implementation.
     *
     * @var \App\Repositories\UserRepository
     */
    protected $users;

    /**
     * Create a new profile composer.
     *
     * @param \App\Repositories\UserRepository $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * Bind data to the view
     *
     * @param \Illuminate\View\View $view
     * @return void
     */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

As you can see, all view composers are resolved via the [service container][containers], so you may type-hint any dependencies you need within a composer's constructor.

#### Attaching A Composer To Multiple Views

You may attach a view composer to multiple views at once by passing an array of views as the first argument to the `composer` method.

```php
use App\Views\Composers\MultiComposer;

View::composer(['profile', 'dashboard'], MultiComposer::class);
```

The `composer` method also accepts the `*` character as a wildcard, allowing you to attach a composer to all views.

```php
View::composer('*', function($view) {
    //
});
```

### View Creators

View "creators" are very similar to view compsoers; however they are executed immediately after the view is instantiated instead of waiting until the view is about to render. To register a view creator, use the `creator` method.

```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

## Optimizing Views

By default, Blade template views are compiled on deman. When a request is executed that renders a view, Laravel will determine if a compiled viersion of the view exists. If the file exists, Laravel will then determine if the uncompiled view has been modified more recently that the compiled view. If the compiled view either does not exist, or the uncompiled view has been modified, Laravel will recompile the view.

Compiling views during the request may have a small negative impact on performance, so Laravel provides the `view:cache` Artisan command to precompile all of the views utilized by your application. For increased performance, you may wish to run this command as part of your deployment process:

```sh
php artisan view:cache
```

You may use the `view:clear` command to clear the view cache

```sh
php artisan view:clear
```

[blade]: https://laravel.com/docs/9.x/blade
[inertia]: https://inertiajs.com
[starter-kits]: https://laravel.com/docs/9.x/starter-kits
[larave-bootcamp]: https://bootcamp.laravel.com
[providers]: https://laravel.com/docs/9.x/providers
[containers]: https://laravel.com/docs/9.x/containers
