---
title: Using the @ComposedObject Zend Framework 2 Form Annotation
category: Zend Framework 2
layout: oldpost
tags:
    - zf2
    - forms
    - annotations
    - php
---

Recently I started playing with Zend Framework 2 Form Annotations. These
certainly make building forms much simpler. If you have not heard about this
yet see [Matthew Weier O'Phinney's
post](http://mwop.net/blog/2012-07-02-zf2-beta5-forms.html) and the [ZF2
Documentation](http://packages.zendframework.com/docs/latest/manual/en/modules/zend.form.quick-start.html)</a>.

After getting it up and running, I found out about about the `@ComposedObject`
annotation. It seemed very useful, however, it took a bit of fiddling to get
it up and running. Here's how I did it.

The `@ComposedObject` annotation lets you create a fieldset inside a form
which was created from one annotated class, by using the form annotations from
another class.

To set this up I created 2 classes `User` and `Address`:

{% highlight php %}
<?php

namespace Album\Model;

use Zend\Form\Annotation as Form;
use Album\Model\Address;

/**
 * @Form\Hydrator("Zend\Stdlib\Hydrator\ArraySerializable")
 * @Form\Name("address")
 */
class User
{
    /**
     * @Form\Attributes({"type":"text" })
     * @Form\Options({"label":"Name:"})
     */
    public $name;
    
    /**
     * @Form\Type("Zend\Form\Element\Email")
     * @Form\Options({"label":"Email:"})
     */
    public $email;
    
    /**
     * @Form\ComposedObject("Album\Model\Address")
     */
    public $address;
}
{% endhighlight %}

{% highlight php %}
<?php

namespace Album\Model;

use Zend\Form\Annotation as Form;

/**
 * @Annotation\Hydrator("Zend\Stdlib\Hydrator\ArraySerializable")
 * @Annotation\Name("address")
 */
class Address
{
    /**
     * @Form\Attributes({"type":"text" })
     * @Form\Options({"label":"Line 1:"})
     */
    public $line1;
    
    /**
     * @Form\Attributes({"type":"text" })
     * @Form\Options({"label":"Line 2:"})
     */
    public $line2;

    /**
     * @Form\Attributes({"type":"text" })
     * @Form\Options({"label":"City:"})
     */
    public $city;
    
    /**
     * @Form\Attributes({"type":"text" })
     * @Form\Options({"label":"Post Code/Zip:"})
     */
    public $postcode;
}
{% endhighlight %}

The `Address` class is pretty straight forward, however, if you look at the
annotations for the `User` class, you will see that the `address` property is annotated as being a
`ComposedObject` of type `Album\Model\Address`.

In the controller you can now add the code to create the form:

{% highlight php %}
<?php

// ...

public function formtestAction()
{
    // Create the User object and fill it with some test data
    $user = new User;
    $user->name = "Tom";
    $user->email = "tom@abc.xyz";
    
    $address = new Address;
    $address->line1 = "My House";
    $address->line2 = "Something Street";
    $address->city = "Great Town";
    $address->postcode = "AB23 4CD";
        
    $user->address = $address;
    
    // Generate the form from the User class Annotations
    $builder = new AnnotationBuilder();
    $form = $builder->createForm($user);
        
    // Add a submit button to the form
    $form->add(array(
        'name'      => 'submit',
        'attributes'=> array(
                'type'  => 'submit',
                'value' => 'Go',
                'id'    => 'submitbutton',
        ),
    ));
        
    // Bind the User object to the form
    $form->bind($user);
    
    // Handle form submittions
    $request = $this->getRequest();
    if ($request->isPost()) {
        $form->setData($request->getPost());
        if ($form->isValid()) {
            print_r($user);
        }
    }
        
    return array('form' => $form);
}

// ...
    
{% endhighlight %}

And a view script to display it:

{% highlight php %}
<?php
$title = 'User form';
$this->headTitle($title);
?>
<h1><?php echo $this->escapeHtml($title); ?></h1>

<?php
$form = $this->form;
$form->setAttribute('action', $this->url('album', array('action' => 'formtest')));
echo $this->form()->openTag($form);
?>
    <?php echo $this->formRow($form->get('name')); ?>
    <?php echo $this->formRow($form->get('email')); ?>
    <fieldset>
        <legend>Address:</legend>
        <?php echo $this->formCollection($form->get('address')); ?>
    </fieldset>
    <?php echo $this->formInput($form->get('submit')); ?>
<?php echo $this->form()->closeTag($form); ?>
{% endhighlight %}

Note that a whole fieldset can be displayed with the formCollection() view
helper.

At this point, when you view this action in the browser the form fields should
all be present. However, all the fields will be empty rather than containing the
data from the object. The reason for this is the `User` and `Address` classes
have been annotated to use the `ArraySerializable` hydrator which requires
`getArrayCopy` and `exchangeArray` methods to be defined.

The reason for choosing `ArraySerializable` instead of another hydrator, is
that, in order for fieldsets to be hydrated they need the extracted data from
the composed object to be presented as a sub-array. The other hydrators will
just return the object as an element in the array, but `ArraySerializable` lets
us define the array that is returned.

For `Address` these 2 methods are simple since it just contains simple fields:

{% highlight php %}
<?php

class Address
{

    // ... member definitions & annotations

    public function getArrayCopy()
    {
        return get_object_vars($this);
    }
    
    function exchangeArray($data)
    {
        $this->line1    = (isset($data['line1'])) ? $data['line2'] : null;
        $this->line2    = (isset($data['line2'])) ? $data['line1'] : null;
        $this->city     = (isset($data['city'])) ? $data['city'] : null;
        $this->postcode = (isset($data['postcode'])) ? $data['postcode'] : null;
    }
}
{% endhighlight %}

However, for `User` we need to call the methods of `Address` to get back and
write a serialized sub array:

{% highlight php %}
<?php

class User
{

    // ... member definitions & annotations

    public function getArrayCopy()
    {
        $data = get_object_vars($this);
        
        if (is_object($this->address)) {
            $data["address"] = $this->address->getArrayCopy();
        }
        
        return $data;
    }
    
    function exchangeArray($data)
    {
        $this->name     = (isset($data['name'])) ? $data['name'] : null;
        $this->email    = (isset($data['email'])) ? $data['email'] : null;
        
        if (isset($data["address"]))
        {
            if (!is_object($this->address)) $this->address = new Address;
            $this->address->exchangeArray($data["address"]);
        }
    }
}
{% endhighlight %}

And now it should all be working!
