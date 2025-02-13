# What is MethodInjector?
`MethodInjector` is an open source software project that provides strong support for generating test doubles for methods, fields, and constants in the target class.
For example, the `MethodInjector` can do the following.

- Replace a function in a method with a mock function or an unnamed function and execute it.
- Rewrites the default value of the specified field
- Test by rewriting the value of a constant
- Mock the return value of a specific method itself
- Process can be inserted at the start and end of method execution

`MethodInjector` parses the original class file and reconstructs the class.
Therefore, you can easily create test doubles even if the original class is not inheritable (i.e., `final` is defined).
However, it is also possible to inherit the original class and create a test double by inheriting the class that the declaration expects.

# Documentation
- [日本語](./readme-ja.md)
- English

# Quick start
It can be installed from the following.

```
composer require --dev m3m0r7/method-injector
```


# How to use?
## Easy example
To create a test double with `MethodInjector`, do the following

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->methodGroup(
                    '*',
                    function (Condition $condition) {
                        return $condition
                            ->replaceFunction(
                                'date',
                                function (...$args) {
                                    return '2012-01-01';
                                }
                            );
                    }
                );
        }
    )
    ->patch();

$mock = $test->createMock(Test::class);
```
Calling a `factory` method returns an instance of the `MethodInjector`. `inspect` takes the first argument as the name of the class to make the test double, the name of the class
The second argument can specify the condition to create the verification and test double, and the third argument can specify whether to inherit the original class.
You can also call several `inspect` methods to create a test double of several classes at once.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$inspector = function (Inspector $inspector) {
    return $inspector
        ->methodGroup(
            '*',
            function (Condition $condition) {
                return $condition
                ->replaceFunction(
                    'date',
                    function (...$args) {
                        return '2012-01-01';
                    }
                );
        }
    );
};
$test
    ->inspect(
        Foo::class,
        $inspector
    )
    ->inspect(
        Bar::class,
        $inspector
    )
    ->patch();

$fooMock = $test->createMock(Foo::class);
$barMock = $test->createMock(Bar::class);
```

Because the test double is generated in the namespace dedicated to `MethodInjector`, it basically does not pollute the global namespace.

## Restrict the method for replacing.
You can also specify and restrict the method to be replaced by the `MethodInjector` of the class `Inspector` by specifying the method name.
Method names are case-insensitive, according to the PHP specification.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->methodGroup(
                    'test',
                    function (Condition $condition) {
                        return $condition
                            ->replaceFunction(
                                'date',
                                function (...$args) {
                                    return '2012-01-01';
                                }
                            );
                }
            );
        }
    )
    ->patch();

$mock = $test->createMock(Test::class);
```


## Replace fields in the class
You may want to rewrite the default values of the fields in your class. You can also use `replaceField` to change the default value of a field.
Of course, it is possible to change the field even if it is `private` or `protected`.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->replaceField('testField', 'changed default value');
        }
    )
    ->patch();

$mock = $test->createMock(Test::class);

echo $mock->testField;
```

## Replace constants in the class
You can use `replaceConstant` to rewrite a constant value in the same way as you can rewrite a field. Of course, this is also possible with `private`, even `protected`.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->replaceConstant('TEST', 'changed default value');
        }
    )
    ->patch();

$mock = $test->createMock(Test::class);

echo $mock::TEST;
```

## Output something when start to process the method
You may want to interrupt some processing at the start of the method execution. With `MethodInjector`, you can specify a `before` of the `Condition` class.
It is possible to interrupt some processing at the start of execution. For example, it is useful when you want to measure the execution time of a single method.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->methodGroup(
                    '*',
                    function (Condition $condition) {
                        return $condition
                            ->before(function () {
                                echo "Hello HEAD!\n";
                            });
                }
            );
        }
    )
    ->patch();

$mock = $test->createMock(Test::class);
```

## Output something when finish to process the method
You can also specify the end of the method execution as well as the start. When it is finished, `after` is called.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->methodGroup(
                    '*',
                    function (Condition $condition) {
                        return $condition
                            ->after(function () {
                                echo "Finish to run.\n";
                            });
                }
            );
        }
    )
    ->patch();

$mock = $test->createMock(Test::class);
```

## Mocking the method
You may want to create a test double when the method itself is in development, or when you want to test a test that returns a certain value, or when it is outside the scope of your testing interests.
With `MethodInjector` it is also possible to create a test double for the method itself using `replaceMethod`.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->replaceMethod(
                    'testMethod',
                    function () {
                        return 'Fixed value.';
                    }
                );
        }
    )
    ->patch();

$mock = $test->createMock(Test::class);
```

## Add a replacer
The `MethodInjector` provides several replacers beforehand, but you may want to add more replacers depending on the situation.
In that case, you can also add a replacers. The replacers of `MethodInjector` is set as the default, but if the original replacers is added, the
Note that the replayer provided by `MethodInjector` is not used by default, so it should be re-specified as an argument.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory([
    'replacer' => [
        [Inspector::FUNCTION, MyFunctionReplacer::class],
        [Inspector::CONSTANT, MyConstantReplacer::class],
    ],
]);
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->replaceMethod(
                    'testMethod',
                    function () {
                        return 'Fixed value.';
                    }
                );
        }
    )
    ->patch();

$mock = $test->createMock(Test::class);
```

If you use `addReplacer`, you can use the replacers provided by `MethodInjector` as is.
The replacers acts like a reduce, applying to the AST parsed nodes in the specified order.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory();
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->addReplacer(Inspector::FUNCTION, MyFunctionReplacer::class)
                ->addReplacer(Inspector::CONSTANT, MyConstantReplacer::class);
        }
    )
    ->patch();
```

## Change the inspector
The inspector of the `MethodInjector` only provides the bare minimum required functionality and may have obstacles such as not being able to use it on projects with a history.
In that case, you may want to replace the inspector itself in order to test it. Of course, it is also possible to implement the original Inspector. In that case, you should extend `Inspector`, which is provided by `MethodInjector` by default.

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use MethodInjector\Condition;
use MethodInjector\Inspector;
use MethodInjector\MethodInjector;

$test = \MethodInjector\MethodInjector::factory([
    'inspectorClass' => MyInspector::class,
]);
$test
    ->inspect(
        Test::class,
        function (Inspector $inspector) {
            return $inspector
                ->addReplacer(Inspector::FUNCTION, MyFunctionReplacer::class)
                ->addReplacer(Inspector::CONSTANT, MyConstantReplacer::class);
        }
    )
    ->patch();
```

## License
MIT
