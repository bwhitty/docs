# Migrations

<p class="uk-article-lead">Use migrations to run code when your extension is installed or updated.</p>

An extension (or theme) can be either enabled, disabled or not installed.
When changing the state, you might need to modify your database schema or run
other custom code. To do so, there are three actions you can hook into.

| Action | Description |
|--------|-------------|
| `enable`     | Triggered when the extension is activated after installation. |
| `disable`    | Triggered when you disable the extension from the admin area and when the extension gets temporarily disabled for a version update. |
| `uninstall`  | Triggered when you remove it from the admin area. |

## Enable hook

Make sure your extension extends the framework's base class
`Pagekit\Extension\Extension` which comes with an `enable` method that does
nothing by default. Overwrite `enable` to run your own code.

```php
<?php

namespace Pagekit\Hello;

use Pagekit\Extension\Extension;


class HelloExtension extends Extension
{
    public function enable()
    {
        // your migration code here
    }
}
```

You can use the `migrator` service to
automatically run any migration script needed, as long as you stick to certain
conventions.

```php
public function enable()
{
  if ($version = $this['migrator']->run('extension://hello/migrations', $this['option']->get('hello:version'))) {
    $this['option']->set('hello:version', $version);
  }
}
```

The migrator will look in the `/migrations` folder of your extension, get all
migrations newer than the database value of the `hello:version` option,
run all those scripts and then set the new value to be `$version`, which is the
resulting version after the migrator has finished.

### Migration files

Migration files are typically located in the `/migrations` folder of your extension and need to stick to certain conventions. A migration file is named using the date and a name of your chosing: `YYYY-MM-DD_some_name.php` (for example `0000-00-00_init.php`).

Inside the migration file, you create an array which has the keys `up` and `down` with a closure function for your code. By using `use ($app)`, you have access to all application services, for example the `db` service for database access.

```php
<?php

return [

    'up' => function() use ($app) {
        // your code here ...
    },

    'down' => function() use ($app) {
        // your code here ...
    }

];

```

In a lot of cases you will not need to specifically implement `down`, unless you want to implement downgrades to an older version.

## Disable hook

Pagekit will not modify the tables you've created, even when your extension
gets *disabled* or *removed* in the admin interface. You will have to take
care of needed database changes yourself.

When your extension gets disabled (either by the user or when installing an
update), you can overwrite the `disable` method in
your `Extension` subclass, just the way you did in the `enable` case.

```php
<?php
namespace Pagekit\Hello;

use Pagekit\Extension\Extension;

class HelloExtension extends Extension
{
    public function disable()
    {
        // your code here
    }
}
```

## Uninstall hook

When your extension gets uninstalled, you should probably drop all tables the
extension has created. In some cases, you will want to keep the possibility
of reinstalling the extension without losing data, but again, it's your
decision.

As you might have guessed, there is an `uninstall` method you can overwrite to
hook into the uninstall action.

```php

class HelloExtension extends Extension
{
    public function uninstall()
    {
        $util = $this['db']->getUtility();
        $util->dropTable('@hello_greetings');
    }
}
```
