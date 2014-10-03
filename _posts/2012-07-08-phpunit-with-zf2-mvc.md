---
title: Getting PHPUnit working with a Zend Framework 2 MVC application
category: Zend Framework 2
layout: oldpost
tags:
    - phpunit
    - zf2
    - php
---

Now that I've got a little MVC application up and running in Zend Framework 2
(see my previous post) I decided it was time to try and get PHPUnit to play
with it. While I don't think I'm alone in wanting to use ZF2 & PHPUnit together
it's certainly something not many people are talking about.

Here I will show how I bootstrapped PHPUnit with the Zend Framework.

First thing I did was create a tests folder in the root of my project folder:

{% highlight console %}
composer.json
composer.phar
data
module
README.md
vendor
composer.lock
config
init_autoloader.php
public
tests
{% endhighlight %}

Next up I created a phpunit.xml configuration inside the tests directory
containing the following XML:

{% highlight xml %}
<phpunit
    bootstrap="./Bootstrap.php"
    colors="true"
    backupGlobals="false"
>
    <testsuites>
        <testsuite name="Test Suite">
            <directory>;./</directory>
        </testsuite>
    </testsuites>
</phpunit>
{% endhighlight %}

Nothing really special in there, the important part is that I've told phpunit
to use the Bootstrap.php file to get things set up.

So next up I needed to create the Boostrap.php file. I came to the conclusion
that I could probably use the same `init_autoloader.php` file that
`default/index.php` in the ZF2 Skeleton Application uses to get the autoloader
up and running.

So I created the following content in the `Bootstrap.php`:

{% highlight php %}
<?php
 
chdir(dirname(__DIR__));
 
include __DIR__ . '/../init_autoloader.php';
{% endhighlight %}

This seemed to do the trick and I could autoload classes from the Zend
Framework, however I still couldn't autoload my classes from my MVC application
module.

I knew I some how needed to pull in the MVC configuration from the the
application and after stepping through the code a bit I came to the conclusion
I needed to add the following code:

{% highlight php %}
$configuration = include 'config/application.config.php';

$serviceManager = new ServiceManager(new ServiceManagerConfiguration($configuration));
$serviceManager->setService('ApplicationConfiguration', $configuration);
$moduleManager = $serviceManager->get('ModuleManager');
$moduleManager->loadModules();
{% endhighlight %}

This gets the application config loaded from config/application.config.php and
then loads the config for the modules listed in there. So now my whole
Bootloader.php looks like this:

{% highlight php %}
<?php

use Zend\ServiceManager\ServiceManager,
    Zend\Mvc\Service\ServiceManagerConfiguration;

chdir(dirname(__DIR__));

include __DIR__ . '/../init_autoloader.php';

$configuration = include 'config/application.config.php';

$serviceManager = new ServiceManager(new ServiceManagerConfiguration($configuration));
$serviceManager->setService('ApplicationConfiguration', $configuration);
$moduleManager = $serviceManager->get('ModuleManager');
$moduleManager->loadModules();
{% endhighlight %}

Now my classes in my modules would easily autoload, marvelous!

## Update

I have since realised that this setup is down in the `init()` method of
`Zend\Mvc\Application` therefore the `Bootstrap.php` file can be reduced down
to:

{% highlight php %}
<?php
chdir(dirname(__DIR__));

include __DIR__ . '/../init_autoloader.php';

Zend\Mvc\Application::init(include 'config/application.config.php');
{% endhighlight %}

Next up was to write a test. I decided the directory structure in my tests
folder would mirror my code tree so I created `module/Album/src/Album/Model/`
and put a new file in there called `AlbumTest.php`

I wrote the following test code:

{% highlight php %}
<?php

use Album\Model\Album,
    Zend\InputFilter\InputFilterInterface;

class AlbumTest extends \PHPUnit_Framework_TestCase
{
    protected $a;

    public function setUp()
    {
        $this->a = new Album;
    }

    /**
     * @expectedException Album\Model\AlbumException
     * @expectedExceptionMessage Not used
     */
    public function testSetInputFilter()
    {
        $if = $this->getMock('Zend\InputFilter\InputFilterInterface');
        $this->a->setInputFilter($if);
    }
 
    public function testGetInputFilter()
    {
        $if = $this->a->getInputFilter();
 
        $this->assertInstanceOf("Zend\InputFilter\InputFilter", $if);
        return $if;
    }
 
    /**
     * @depends testGetInputFilter
     */
    public function testInputFilterValid($if)
    {
        $this->assertEquals(3, $if->count());
 
        $this->assertTrue($if->has('title'));
        $this->assertTrue($if->has('artist'));
        $this->assertTrue($if->has('id'));
    }
}
{% endhighlight %}

I won't go into detail about what that test does, that can all be looked up in
the PHPUnit documentation. One thing I did do through was make
`Album::setInputFilter` throw `AlbumException` instead of `Exception`, the
reason for this is PHPUnit 3.6 doesn't like checking for Exception classes,
apparently this is fixed in 3.7.

So anyway, not when I cd into my tests directory and type `phpunit` I get the
following output:

{% highlight console %}
PHPUnit 3.6.11 by Sebastian Bergmann.
Configuration read from /home/tom/workspace/ZF2Tutorial/tests/phpunit.xml
...
Time: 0 seconds, Memory: 5.25Mb
OK (3 tests, 7 assertions)
{% endhighlight %}

Success!

Next up, time to work out how to use PHPUnit with classes that access the
database and controller classes.
