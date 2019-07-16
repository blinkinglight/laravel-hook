# Hongyukeji [![Build Status](https://travis-ci.org/hongyukeji/laravel-hook.svg?branch=master)](https://travis-ci.org/hongyukeji/laravel-hook)

Actions and filters in Laravel. WordPress-style. 

Hongyukeji (for lack of a better name) is a simple action and filter (or hooks if you like) system.

## About

Actions are pieces of code you want to execute at certain points in your code. Actions never return anything but merely serve as the option to hook in to your existing code without having to mess things up.

Filters are made to modify entities. They always return some kind of value. By default they return their first parameter and you should too. 

[Read more about filters](http://www.wpbeginner.com/glossary/filter/)


[Read more about actions](http://www.wpbeginner.com/glossary/action/)

## When would I use Hongyukeji?

Hongyukeji is best used as a way to allow extensibility to your code. Whether you're creating a package or an application, Hongyukeji can bring the extensibility you need. 

For example, Hongyukeji can lay down the foundation for a plugin/module based system. You offer an "action" that allows plugins to register themselves. You might offer a "filter" so plugins can change the contents of an array in the core. You could even offer an "action" so plugins can modify the menu of your application.

Hongyukeji is in no way unique in its approach. Laravel provides the Macroable trait that allows you to "hack" in to a class and hooks so you can act on specific points in your code right out of the box. 

## Installation

1. Install using Composer

```
composer require hongyukeji/laravel-hook
```

If you're using Laravel 5.5 or later you can start using the package at this point. Hongyukeji is auto-discovered by the Laravel framework.

2. Add the service provider to the providers array in your `config/app.php`.

```php
    'TorMorten\Hongyukeji\HookServiceProvider',
    'TorMorten\Hongyukeji\HookBladeServiceProvider', 
```

3. Add the facade in `config/app.php`

```php
    'Hongyukeji' => TorMorten\Hongyukeji\Facades\Hooks::class,
```


## Usage

### Actions

Anywhere in your code you can create a new action like so:

```php
Hongyukeji::action('my.hook', 'awesome');
```

The first parameter is the name of the hook; you will use this at a later point when you'll be listening to your hook. All subsequent parameters are sent to the action as parameters. These can be anything you'd like. For example you might want to tell the listeners that this is attached to a certain model. Then you would pass this as one of the arguments.

To listen to your hooks, you attach listeners. These are best added to your `AppServiceProvider` `boot()` method. 

For example if you wanted to hook in to the above hook, you could do:

```
Hongyukeji::addAction('my.hook', function($what) {
    echo 'You are '. $what;
}, 20, 1);
```

Again the first argument must be the name of the hook. The second would be a callback. This could be a Closure, a string referring to a class in the application container (`MyNamespace\Http\Listener@myHookListener`), an array callback (`[$object, 'method']`) or a globally registered function `function_name`. The third argument is the priority of the hook. The lower the number, the earlier the execution. The fourth parameter specifies the number of arguments your listener accepts.

### Filters

Filters work in much the same way as actions and have the exact same build-up as actions. The most significant difference is that filters always return their value. 

To add a filter:

```php 
$value = Hongyukeji::filter('my.hook', 'awesome');
```

If no listeners are attached to this hook, the filter would simply return `'awesome'`. 

This is how you add a listener to this filter (still in the `AppServiceProvider`):

```php
Hongyukeji::addFilter('my.hook', function($what) {
    $what = 'not '. $what;
    return $what;
}, 20, 1);
```

The filter would now return `'not awesome'`. Neat!

You could use this in conjunction with the previous hook:

```php
Hongyukeji::addAction('my.hook', function($what) {
    $what = Hongyukeji::filter('my.hook', 'awesome');
    echo 'You are '. $what;
});
```

### Using in Blade

Given you have added the `HookBladeServiceProvider` to your config, there are two directives available so you can use this in your Blade templates.

Adding the same action as the one in the action example above:

```
@action('my.hook', 'awesome')
```

Adding the same filter as the one in the filter example above:

```
You are @filter('my.hook', 'awesome')

```
