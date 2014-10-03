---
title: Testing Zend Framework 2 Code Which Uses a HTTP Request object.
category: Zend Framework 2
layout: oldpost
tags:
    - zf2
    - php
---

Yesterday, I was trying to test that the `ServiceManager` was correctly creating
an instance of a class which took a `Zend\Http\Request` object as a
parameter to the constructor. The problem I ran into was that since the Zend
MVC Application had been initialised from PHPUnit, it decided it was running in
a console context and was using a `Zend\Console\Request` instead.

After a bit of hunting around I came up with the following, simple
solution: 

{% highlight php %}
<?php

class MyModuleTest extends \PHPUnit_Framework_TestCase
{
    /**
     * The test which requires a HTTP Request object.
     */
    public function test_service_manager_creates_successfully()
    {
        $serviceManager = \TestBootstrap::getApplication()
                                    ->getServiceManager();

        $this->setRequestTypeToHttp();

        $this->assertInstanceOf(
            'MyModule\ClassBeingTested',
            $serviceManager->get('MyModule\ClassBeingTested')
        );
    }

    /**
     * Sets the service manager to return an instance of \Zend\Http\Request
     * instead of its default.
     */
    private function setRequestTypeToHttp($serviceManager)
    {
        // This needs to be set so we can change the service manager config
        $serviceManager->setAllowOverride(true);

        // Replace the Request service with a callback which creates
        // a \Zend\Http\Request instance instead
        $serviceManager->setFactory('Request', function ($sm) {
            return new \Zend\Http\Request();
        });
    }
}
{% endhighlight %}

### My PHPUnit boostrap file for reference

{% highlight php %}
<?php

class TestBootstrap
{
    private static $autoloaderFiles = [
        '../vendor/autoload.php',
    ];

    private static $application;

    /**
     * Setup the testing environment.
     *
     * @param  string $config Path to the Zend application config file.
     * @return void
     */
    public static function init($config)
    {
        $loader = self::getAutoloader();

        //$loader->add('ModuleTestNamespace\\', __DIR__);

        self::$application = \Zend\Mvc\Application::init($config);
    }

    /**
     * Return the application instance.
     *
     * @return \Zend\Mvc\Application
     */
    public static function getApplication()
    {
        return self::$application;
    }

    private static function getAutoloader()
    {
        global $loader;

        foreach (self::$autoloaderFiles as $file) {
            if ($file[1] !== '/') {
                $file = __DIR__ . '/' . $file;
            }


            if (file_exists($file)) {
                $loader = include $file;
                break;
            }
        }

        if (!isset($loader) || !$loader) {
            throw new \RuntimeException('vendor/autoload.php not found. Have you run composer?');
        }

        return $loader;
    }
}

TestBootstrap::init(include __DIR__ . '/application.config.php');
{% endhighlight %}
