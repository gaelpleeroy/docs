# How to Build a Module

## Preparation

Before you begin working on a module, you need to decide on a couple things:

Namespace
: The root namespace that your module’s classes will live in. (See the [PSR-4](https://www.php-fig.org/psr/psr-4/) autoloading specification for details.) Note that this should *not* begin with `craft\`; use something that identifies you (the developer), or the project.

Module ID
: Something that uniquely identifies the module within your project. IDs must begin with a letter and contain only lowercase letters, numbers, and dashes, and should be `kebab-cased`.
: The module ID will become the first segment in [controller action](./controllers.md) paths.

::: warning
When choosing a module ID, avoid names that will conflict with Craft’s core [controllers](https://github.com/craftcms/cms/tree/develop/src/controllers) (e.g. `app` would conflict with `AppController.php`), as well as any installed plugin handles.
:::

## Set up the basic file structure

To create a module, create a new directory for it somewhere within your Craft project, such as `modules/<ModuleID>/`. For example, if your module ID is `foo`, you might set it up like this:

```treeview
my-project/
├── modules/
│   └── foo/
│       └── Module.php
├── templates/
└── ...
```

## Set up class autoloading

Next up, you need to tell Composer how to find your module’s classes by setting the [`autoload`](https://getcomposer.org/doc/04-schema.md#autoload) field in your project’s `composer.json` file. For example, if your module’s namespace is `foo`, and it’s located at `modules/foo/`, this is what you should add:

```json
{
  // ...
  "autoload": {
    "psr-4": {
      "foo\\": "modules/foo/"
    }
  }
}
```

With that in place, go to your project’s directory in your terminal, and run the following command:

```bash
composer dump-autoload -a
```

That will tell Composer to update its class autoloader script based on your new `autoload` mapping.

## The Module class

The `Module.php` file is your module’s entry point for the system. Its `init()` method is the best place to register event listeners, and any other steps it needs to take to initialize itself.

Use this template as a starting point for your `Module.php` file:

```php
<?php
namespace foo;

use Craft;

class Module extends \yii\base\Module
{
    public function init()
    {
        // Define a custom alias named after the namespace
        Craft::setAlias('@foo', __DIR__);

        // Set the controllerNamespace based on whether this is a console or web request
        if (Craft::$app->getRequest()->getIsConsoleRequest()) {
            $this->controllerNamespace = 'foo\\console\\controllers';
        } else {
            $this->controllerNamespace = 'foo\\controllers';
        }

        parent::init();

        // Custom initialization code goes here...
    }
}
```

Replace `foo` with your module’s actual namespace, and `'@foo'` with an [alias](guide:concept-aliases) name based on your actual namespace (with any `\`s converted to `/`s).

### Initialization

Most initialization logic belongs in your module’s `init()` method.

However, there are some situations where this doesn’t guarantee a certain part of the application is ready (another plugin, for instance). Conversely, a module that isn’t bootstrapped at the beginning of a request may have `init()` called too late to listen to <craft4:craft\web\Application::EVENT_INIT>, and would never be notified that the app is indeed ready.

In those cases, it’s best to register a callback via <craft4:craft\base\ApplicationTrait::onInit()>:

```php
<?php
namespace foo;

use Craft;

class Module extends \yii\base\Module
{
    public function init()
    {
        // ...

        // Defer most setup tasks until Craft is fully initialized:
        Craft::$app->onInit(function() {
            // ...
        });
    }
}
```

If Craft has already fully initialized, the callback will be invoked immediately.

### Update the application config

You can add your module to your project’s [application configuration](../config/#application-configuration) by listing it in the [modules](yii2:yii\base\Module::modules) and [bootstrap](yii2:yii\base\Application::bootstrap) arrays. For example, if your module ID is `foo` and its Module class name is `foo\Module`, this is what you should add to `config/app.php`:

```php{4,7}
return [
    // ...
    'modules' => [
        'foo' => foo\Module::class,
    ],
    'bootstrap' => [
        'foo',
    ],
];
```

::: tip
If your module doesn’t need to get loaded on every request, you can remove its ID from the `bootstrap` array. If you end up using any event listeners in your module’s `init()` method, it should be loaded on every request.
:::


## Further Reading

To learn more about modules, see the [Yii documentation](guide:structure-modules).
