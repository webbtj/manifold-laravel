# manifold-laravel

Official Laravel package connecting your [Manifold](https://manifold.co) secrets into your Laravel application.

[Code of Conduct](./CODE_OF_CONDUCT.md) |
[Contribution Guidelines](./.github/CONTRIBUTING.md)

[![GitHub release](https://img.shields.io/github/tag/manifoldco/manifold-laravel.svg?label=latest)](https://github.com/manifoldco/manifold-laravel/releases)
[![Travis](https://img.shields.io/travis/manifoldco/manifold-laravel/master.svg)](https://travis-ci.org/manifoldco/manifold-laravel)
[![License](https://img.shields.io/badge/license-BSD-blue.svg)](./LICENSE)
[![Latest Stable Version](https://poser.pugx.org/manifoldco/manifold-laravel/v/stable)](https://packagist.org/packages/manifoldco/manifold-laravel)

## Introduction

The Manifold Laravel package allows you to connect to your Manifold account and
pull your credentials, keys, configurations, etc. from your Manifold account
into your Laravel application.

## Installation

1. Install the package
```
composer require manifoldco/manifold-laravel
```

2. Publish the config file and select `manifoldco\manifold-laravel` from the vendor list.
```
php artisan vendor:publish
```

3. Add, at the very least, your Manifold Bearer token to your `.env` file as
follows: `MANIFOLD_API_TOKEN=YOUR-TOKEN-HERE`

4. You may optionally specify a Project by providing the label in your `.env` file.
```
MANIFOLD_PROJECT=YOUR-PROJECT-LABEL
```

## Usage
Once installed and configured, your project/resource credentials from Manifold
will be pulled into your Laravel application as configurations. Your credentials
can be accessed using the `config` helper, using the label of the resource and
the key of credential, as they exist in Manifold, using dot-notation. For
example, if you have a Mailgun resource setup with a credential named `API_KEY`,
it can be accessed using `config('mailgun.API_KEY')` (where `mailgun` is the
resource's label).

Note that keys are case sensitive. Also note that when configurations conflict
with other Laravel configs, the Manifold configurations will take priority.

## Aliases
In some cases, you may wish to use credentials from Manifold within
configuration files (inside your `/config` directory). The most obvious use case
would be database credentials that are needed within `/config/database.php`.
Since you cannot reliably access configurations from within configuration files
using the `config()` helper, aliases may be defined in your `config/manifold.php`
file. Aliases can be defined in arrays (as with standard configurations) or
using dot-notation for array keys. Define the existing config as the key and
the Manifold credential as the value. For example, pulling a PostgreSQL password
from a custom PostgreSQL service in Manifold could look like this:
`'database.connections.pgsql.password' => 'custom-pgsql.DB_PASSWORD'`. This will
pull the `DB_PASSWORD` credential from the `custom-pgsql` resource and assign
its value to `database.connections.pgsql.password`, so there is no need to
manipulate your existing `config/database.php` file.

## Examples
1. You have a project in Manifold with a label of `my-project`.
You want your Mailgun API key available in a controller method. Your Mailgun
resource is named `mailgun` and the API key credential is `API_KEY`.

Add the following to `.env`
```
MANIFOLD_API_TOKEN=0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF0123456789AB
MANIFOLD_PROJECT=my-project
```

In your controller's php file:
```
class MyController extends Controller{
    public function process_mail(){

        $mailgun_key = config('mailgun.API_KEY');

        //mail processing logic here
    }
}
```

2. You have a project in Manifold with a label of `my-project`.
You want your PostgreSQL credentials from your custom service stored in Manifold.
Your custom service is named `custom-pgsql` and the PostgreSQL credential keys
are `DB_HOST`, `DB_DATABASE`, `DB_USERNAME`, and `DB_PASSWORD`.

Add the following to `.env`
```
MANIFOLD_API_TOKEN=0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF0123456789AB
MANIFOLD_PROJECT=my-project
```

In your `config/manifold.php`:
```
return [
    'token' => env('MANIFOLD_API_TOKEN', null),
    'project' => env('MANIFOLD_PROJECT', null),
    'aliases' => [
        /*
            note that while you can mix and match array syntax and dot-notation
            it's best to use one or the other, this merely illustrates that both
            are possible
        */
        'database.connections.pgsql.password' => 'custom-pgsql.DB_PASSWORD',
        'database' => [
            'connections' => [
                'pgsql' => [
                    'host' => 'custom-pgsql.DB_HOST',
                    'database' => 'custom-pgsql.DB_DATABASE',
                    'username' => 'custom-pgsql.DB_USERNAME',
                ]
            ]
        ]
    ],
];
```

3. You have a project in Manifold with a label of `my-project`.
You want your MySQL credentials from your JAWS stored in Manifold.
Your JAWS service is named `jaws-mysql` and the connection credentials are in
URL syntax as `JAWSDB_URL`. The URL syntax does not work with Laravel out of the
box and must be parsed. For this you can pass a closure as the value of an
alias. The closure receives no parameters, but all configs are loaded so you can
access and Manifold stored credentials and manipulate them as necessary.

Add the following to `.env`
```
MANIFOLD_API_TOKEN=0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF0123456789AB
MANIFOLD_PROJECT=my-project
```

In your `config/manifold.php`:
```
return [
    'token' => env('MANIFOLD_API_TOKEN', null),
    'project' => env('MANIFOLD_PROJECT', null),
    'aliases' => [
        'database' => [
            'connections' => [
                'mysql' => [
                    'host' => function(){
                        $url = parse_url(config('custom-service.jaws'));
                        return $url['host'];
                    },
                    'password' => function(){
                        $url = parse_url(config('custom-service.jaws'));
                        return $url['pass'];
                    },
                    'username' => function(){
                        $url = parse_url(config('custom-service.jaws'));
                        return $url['user'];
                    },
                    'database' => function(){
                        $url = parse_url(config('custom-service.jaws'));
                        return substr($url["path"], 1);
                    }
                ]
            ]
        ]
    ],
];
```
