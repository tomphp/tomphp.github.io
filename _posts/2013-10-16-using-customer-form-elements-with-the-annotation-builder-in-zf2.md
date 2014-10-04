---
title: Using Customer Form Elements With the Annotation Builder in Zend Framework 2
category: Zend Framework 2
layout: oldpost
tags:
    - zf2
    - forms
    - php
---

Just a quick little tip:

Since ZF 2.2, we now have the `FormElementManager` service available. This
makes it nice as easy to register new, custom form elements and fieldsets in
your ZF2 application (see <a
href="http://framework.zend.com/manual/2.1/en/modules/zend.form.advanced-use-of-forms.html"
target="_blank">Creating Custom Elements</a>). However, I quickly discovered
that it wasn't possible to use custom elements easily in my forms if the form
was created using the annotation builder.

According to the ZF2 manual the annotation builder is used like so:

{% highlight php %}
<?php

use Zend\Form\Annotation\AnnotationBuilder;

$builder = new AnnotationBuilder();
$form = $builder->createForm('User');
{% endhighlight %}

The problem here is that the `AnnotationBuilder` instance has no knowledge of
the `FormElementManager` to find the custom elements. It has no `FormFactory`
set so it creates it's own internal instance, which isn't aware of the
`FormElementManager`.

The solution I have found to this, is to set the `FormFactory` of the
`AnnotationBuilder` like so:

{% highlight php %}
<?php

use Zend\Form\Annotation\AnnotationBuilder;
use Zend\Form\Factory;

$serviceManager = /* Get application service manager */;

$builder = new AnnotationBuilder();

$annotationBuilder->setFormFactory(
    new Factory($serviceManager->get('FormElementManager'))
);     

$form = $builder->createForm('User');
{% endhighlight %}

Now if the `User` class is annotated with custom form elements, it will all
work nicely.

That's my solution, let me know if you have a better one.
