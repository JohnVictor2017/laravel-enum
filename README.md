<p align="center"><img src="https://github.com/BenSampo/laravel-enum/raw/master/branding/logo.svg?sanitize=true" alt="Laravel Enum" width="250" style="margin-bottom: 20px"></p>
<p align="center">
<a href="https://travis-ci.org/BenSampo/laravel-enum"><img src="https://travis-ci.org/BenSampo/laravel-enum.svg?branch=master" alt="Build Status"></a>
<a href="https://packagist.org/packages/bensampo/laravel-enum"><img src="https://img.shields.io/packagist/v/bensampo/laravel-enum.svg?style=flat-square&label=stable" alt="Packagist Stable Version"></a>
<a href="https://packagist.org/packages/bensampo/laravel-enum"><img src="https://img.shields.io/packagist/dt/bensampo/laravel-enum.svg?style=flat-square" alt="Packagist downloads"></a>
<a href="LICENSE.md"><img src="https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square" alt="MIT Software License"></a>
</p>

## About Laravel Enum

Simple, extensible and powerful enumeration implementation for Laravel.

* Enum key value pairs as class constants
* Full featured suite of methods
* Enum Instantiation
* Type hinting
* Enum artisan generator
* Validation rules for passing enum key or values as input parameters
* Localization support
* Extendible

Created by [Ben Sampson](https://sampo.co.uk)

## Jump to

* [Guide](#guide)
* [Install](#install)
* [Enum library](enum-library.md)
* [Generating enums](#generating-enums)
* [Usage](#usage)
* [Static Methods](#static-methods)
* [Instantiation](#instantiation)
* [Validation](#validation)
* [Localization](#localization)
* [Extending the Enum base class](#extending-the-enum-base-class)

## Guide
I wrote a blog post about using laravel-enum:
https://sampo.co.uk/blog/using-enums-in-laravel

## Requirements

Laravel 5.4 or newer  
PHP 7.1 or newer

## Install

Via Composer

``` bash
$ composer require bensampo/laravel-enum
```

If you're using Laravel < 5.5 you'll need to add the service provider to `config/app.php`
``` php
'BenSampo\Enum\EnumServiceProvider'
```

## Enum library
Browse and download from a list of commonly used, community contributed enums.

[Enum library →](enum-library.md)

## Generating enums

```php
php artisan make:enum UserType
```

## Usage

Given the following enum:
``` php
<?php

namespace App\Enums;

use BenSampo\Enum\Enum;

final class UserType extends Enum
{
    const Administrator = 0;
    const Moderator = 1;
    const Subscriber = 2;
    const SuperAdministrator = 3;
}
```

Values can now be accessed like so:
``` php
UserType::Moderator // Returns 1
```

## Static Methods

### getKeys(): array

Returns an array of the keys for an enum.

``` php
UserType::getKeys(); // Returns ['Administrator', 'Moderator', 'Subscriber', 'SuperAdministrator']
```

### getValues(): array

Returns an array of the values for an enum.

``` php
UserType::getValues(); // Returns [0, 1, 2, 3]
```

### getKey(string|int $value): string

Returns the key for the given enum value.

``` php
UserType::getKey(1); // Returns 'Moderator'
UserType::getKey(UserType::Moderator); // Returns 'Moderator'
```

### getValue(string $key): string|int

Returns the value for the given enum key.

``` php
UserType::getValue('Moderator'); // Returns 1
```

### hasKey(string $key): bool

Check if the enum contains a given key.

``` php
UserType::hasKey('Moderator'); // Returns 'True'
```

### hasValue(string|int $value, bool $strict = true): int

Check if the enum contains a given value.

``` php
UserType::hasValue(1); // Returns 'True'

// It's possible to disable the strict type checking:
UserType::hasValue('1'); // Returns 'False'
UserType::hasValue('1', false); // Returns 'True'
```

### getDescription(string|int $value): string

Returns the key in sentence case for the enum value. It's possible to [override the getDescription](#overriding-the-getDescription-method) method to return custom descriptions.

``` php
UserType::getDescription(3); // Returns 'Super administrator'
UserType::getDescription(UserType::SuperAdministrator); // Returns 'Super administrator'
```

### getRandomKey(): string

Returns a random key from the enum. Useful for factories.

``` php
UserType::getRandomKey(); // Returns 'Administrator', 'Moderator', 'Subscriber' or 'SuperAdministrator'
```

### getRandomValue(): string|int

Returns a random value from the enum. Useful for factories.

``` php
UserType::getRandomValue(); // Returns 0, 1, 2 or 3
```

### toArray(): array

Returns the enum key value pairs as an associative array.

``` php
UserType::toArray(); // Returns ['Administrator' => 0, 'Moderator' => 1, 'Subscriber' => 2, 'SuperAdministrator' => 3]
```

### toSelectArray(): array

Returns the enum for use in a select as value => description.

``` php
UserType::toSelectArray(); // Returns [0 => 'Administrator', 1 => 'Moderator', 2 => 'Subscriber', 3 => 'Super administrator']
```

### getInstance(string|int $enumValue): Enum

Returns an instance of the called enum. Read more about [enum instantiation](#instantiation).

``` php
UserType::getInstance(UserType::Administrator);
```

## Instantiation
It can be useful to instantiate enums in order to pass them between functions with the benefit of type hinting. Additionally, it's impossible to instantiate an enum with an invalid value, therefore you can be certain that the passed value is always valid.

For convenience, enums can be instantiated in multiple ways:

``` php
// Standard new PHP class, passing the desired enum value as a parameter
$enumInstance = new UserType(UserType::Administrator);

// Static getInstance method, again passing the desired enum value as a parameter
$enumInstance = UserType::getInstance(UserType::Administrator);

// Statically calling the key name as a method
$enumInstance = UserType::Administrator();
```

### Instance Properties

Once you have an enum instance, you can access the `key`, `value` and `description` as properties. This is particularly useful if you're passing an enum instance to a blade view.

``` php
$userType = UserType::getInstance(UserType::SuperAdministrator);

$userType->key; // SuperAdministrator
$userType->value; // 0
$userType->description; // Super Administrator
```

### Instance Equality

You can check the equality of an instance against a valid enum value by passing it to the `is` method.

``` php
$userType = UserType::getInstance(UserType::SuperAdministrator);

$userType->is(UserType::SuperAdministrator); // Returns true
$userType->is(UserType::Moderator); // Returns false
$userType->is(UserType::InvalidKey); // Throws InvalidEnumMemberException exception
```

### Type Hinting

One of the benefits of enum instances is that it enables you to use type hinting, as shown below.

``` php
function canPerformAction(UserType $userType)
{
    if ($userType->is(UserType::SuperAdministrator)) {
        return true;
    }

    return false;
}

$userType1 = UserType::getInstance(UserType::SuperAdministrator);
$userType2 = UserType::getInstance(UserType::Moderator);

canPerformAction($userType1); // Returns true
canPerformAction($userType2); // Returns false
```

## Validation

### Array Validation
You may validate that an enum value passed to a controller is a valid value for a given enum by using the `EnumValue` rule.

``` php
public function store(Request $request)
{
    $this->validate($request, [
        'user_type' => ['required', new EnumValue(UserType::class)],
    ]);
}
```

By default, type checking is set to strict, but you can bypass this by passing `false` to the optional second parameter of the EnumValue class.

```php
new EnumValue(UserType::class, false) // Turn off strict type checking.
```

You can also validate on keys using the `EnumKey` rule. This is useful if you're taking the enum key as a URL parameter for sorting or filtering for example.

``` php
public function store(Request $request)
{
    $this->validate($request, [
        'user_type' => ['required', new EnumKey(UserType::class)],
    ]);
}
```

Of course, both of these work on form request classes too.

Make sure to include `BenSampo\Enum\Rules\EnumValue` and/or `BenSampo\Enum\Rules\EnumKey` and your enum class in the usings.

### Pipe Validation
You can also use the 'pipe' syntax for both the EnumKey and EnumValue rules by using `enum_value` and/or `enum_key` respectively.

**enum_value**_:enum_class,[strict]_  
**enum_key**_:enum_class_

```php
'user_type' => 'required|enum_value:' . UserType::class,
'user_type' => 'required|enum_key:' . UserType::class,
```

## Localization

You can translate the strings returned by the `getDescription` method using Laravel's built in [localization](https://laravel.com/docs/5.6/localization) features.

Add a new `enums.php` keys file for each of your supported languages. In this example there is one for English and one for Spanish.

```php
// resources/lang/en/enums.php
<?php

use App\Enums\UserType;

return [

    UserType::class => [
        UserType::Administrator => 'Administrator',
        UserType::SuperAdministrator => 'Super administrator',
    ],

];
```

```php
// resources/lang/es/enums.php
<?php

use App\Enums\UserType;

return [

    UserType::class => [
        UserType::Administrator => 'Administrador',
        UserType::SuperAdministrator => 'Súper administrador',
    ],

];
```

Now, you just need to make sure that your enum implements the `LocalizedEnum` interface as demonstrated below:
 
```php
use BenSampo\Enum\Enum;
use BenSampo\Enum\Contracts\LocalizedEnum;

final class UserType extends Enum implements LocalizedEnum
{
    // ...
}
```

The `getDescription` method will now look for the value in your localization files. If a value doesn't exist for a given key, the default description is returned instead.

## Overriding the getDescription method

If you'd like to return a custom value from the getDescription method, you may do so by overriding the method on your enum:

```php
public static function getDescription($value): string
{
    if ($value === self::SuperAdministrator) {
        return 'Super admin';
    }

    return parent::getDescription($value);
}
```

Calling `UserType::getDescription(3);` now returns `Super admin` instead of `Super administator`.

## Extending the enum base class

The `Enum` base class implements the [Laravel `Macroable`](https://laravel.com/api/5.6/Illuminate/Support/Traits/Macroable.html) trait, meaning it's easy to extend it with your own functions. If you have a function that you often add to each of your enums, you can use a macro.

Let's say we want to be able to get a flipped version of the enum `toArray` method, we can do this using:

```php
Enum::macro('toFlippedArray', function() {
    return array_flip(self::toArray());
});
```

Now, on each of my enums, I can call it using `UserType::toFlippedArray()`.

It's best to register the macro inside of a service providers' boot method.
