---
title: Neat Exception Classes Using PHP Traits
category: PHP
layout: post
tags:
    - traits
    - php
---

We're all told that our exception messages should be nice and informative.
Often `sprintf` is used to format the exception messages.  This is all well and
good, it's very helpful when an exception gets thrown and logged.  However, it
leads to code that looks like this:

{% highlight php %}
<?php

public function someUsefulMethod($id)
{
    $entity = $this->repository()->findById($id);

    if (!$entity) {
        throw new MyExceptionClass(
            sprintf(
                'The nice descriptions that methods id "%d" and the type of entity "%s".',
                $id,
                is_object($entity) ? get_class($entity) : gettype($entity)
            )
        );
    }

    // Do stuff with $entity ...
}

{% endhighlight %}

This looks like a bit of a mess to me. It takes up 9 lines and has 3 extra
levels of indentation.

Some time ago I submitted a pull request on a project (I forget which) and
[Ocramius](https://twitter.com/Ocramius) suggested that I use a static factory
method inside my exception to make it neater. I had a look at some exception
classes in the project and definitely found it was neater! (I assume others are
out there using this pattern too?).

I decided to adopt this pattern, and instead of the code above I now had this:

### Exception Class
{% highlight php %}
<?php

class MyExceptionClass
{
    public static function usefulErrorDescription($id, $entity)
    {
        return new self(
            sprintf(
                'The nice descriptions that methods id "%d" and the type of entity "%s".',
                $id,
                is_object($entity) ? get_class($entity) : gettype($entity)
            )
        );
    }
}
{% endhighlight %}

### Code
{% highlight php %}
<?php

//...
public function someUsefulMethod($id)
{
    $entity = $this->repository()->findById($id);

    if (!$entity) {
        throw MyExceptionClass::usefulErrorDescription($id, $entity);
    }

    // Do stuff with $entity ...
}
//...
{% endhighlight %}

So far this has certainly tidied up the code throwing the exception, and made
it much easier to see what's going on. Also, all the exception messages are now
in 1 place, inside the exception class, so they're easier to manage.

However, at this point I still think that the exception class looks a bit messy
and could be improved further. Using the wonders of PHP 5.4's nice new feature,
I created the following trait:

{% highlight php %}
<?php

/**
 * This trait provides a factory method for exceptions which have static
 * methods to create themselves.
 *
 * @author Tom Oram <tom@scl.co.uk>
 */
trait ExceptionFactoryTrait
{
    /**
     * Create an instance of the exception with a formatted message.
     *
     * @param  string     $message  The exception message in sprintf format.
     * @param  array      $params   The sprintf parameters for the message.
     * @param  int        $code     Numeric exception code.
     * @param  \Exception $previous The previous exception.
     *
     * @return self
     */
    protected static function create(
        $message,
        array $params = [],
        $code = 0,
        $previous = null
    ) {
        return new self(vsprintf($message, $params), $code, $previous);
    }

    /**
     * Returns a string representation of the type of a variable.
     *
     * @param  mixed $variable
     *
     * @return string
     */
    protected static function typeToString($variable)
    {
        return is_object($variable)
            ? get_class($variable)
            : gettype($variable);
    }
}
{% endhighlight %}

Now, by telling my exception class to use this trait, I can have nice neat
exception classes like so:

{% highlight php %}
<?php

class MyExceptionClass
{
    use ExceptionFactoryTrait;

    public static function usefulErrorDescription($id, $entity)
    {
        return new self::create(
            'The nice descriptions that methods id "%d" and the type of entity "%s".',
            [$id, self::typeToString($entity)]
        );

        // more factory methods ...
    }
}
{% endhighlight %}

I think is a much more elegant solution!

Please let me know of any thoughts or suggestions!
