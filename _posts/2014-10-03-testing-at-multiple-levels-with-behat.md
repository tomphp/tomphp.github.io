---
title: Testing at Multiple Levels with Behat
category: bdd
layout: post
tags:
    - behat
    - bdd
    - php
---

Today has been a rather exciting day for me. Konstantin Kudryashov, aka
[everzet](https://twitter.com/everzet), posted [this fantastic
article](http://everzet.com/post/99045129766/introducing-modelling-by-example).
I was probably somewhat more excited than many other people about this because,
in it he talks about using Behat in the way that I've currently been trying
myself. It's also the way I'm presenting working with Behat in the [upcoming
book I'm working
on](https://leanpub.com/modern-application-development-with-php). This made me
feel much more confident about the way I've been working with Behat. It also
prompted me to write this article which contains some further thoughts about
the approach of testing at multiple levels with Behat.

I've primarily been testing at 2 levels with the same features: once at
the domain level and once at the user interface level. I've also been
experimenting with the idea of using the same features to test a RESTful API,
and even pondered on whether you could use CucumberJS to run the same features
on a Javascript, client side interface. At one level all this works really
well. On another level there are still slightly different sets of information
and languages which are relevant at each layer. These difference are what I
want to discuss here.

## What is an Acceptance Test?

We use acceptance tests to prove to ourselves and to the stakeholder, that the
requested features have been implemented. If you are a full stack developer
then passing feature can be a great thing to work towards. However, if
you have 2 separate teams working on the domain and the user interface, or even if
you simply approach the development of each at separate times, then only the
user interface developer has a nice feature to work towards. Using the same features
to test each of these levels provides a means for the developer at each level
to work their own acceptance criteria (even though the stakeholder is only
really interested in seeing the final, full stack, user interface tests).

It also means that when a new or replacement user interface is developed, there
are still fantastic tests present on the entry points to the domain layer.

## A Case Study

Say we have a simple system which lists widgets and when you click on one you
can see it's details on a new page. From chatting with the stakeholders you
come up with something like this:

{% highlight cucumber %}
Feature: List Widgets

    Scenario: List widgets
        Given there is a widget "Widget A"
        And there is a widget "Widget B"
        When I list widgets
        Then I should see the following list of widgets:
            | name     |
            | Widget A |
            | Widget B |
{% endhighlight %}

{% highlight cucumber %}
Feature: View a widget

    Scenario: View a widget
        Given there is a widget "Widget A" with details "a great widget"
        When I view widget "Widget A"
        Then I should see widget name as "Widget A"
        And I should see widget details as "a great widget"
{% endhighlight %}

### Building the Application

First up we decide to put on our domain modelling hat. We're not going to
concern ourselves with how to build the front end for now.

We decide to build 2 use cases: one called `ListWidgets` and another
called `ViewWidget`.  The `ListWidgets` use case is nice and simple, it just
fetches all widgets and returns a list of their names. The second is a bit more
tricky, it fetches a single widget and returns it's details but, how does it
identify which widget to fetch?

Of course, as programmers we have the answer - we love unique IDs! We know that
what we need to do is return the ID a long with the name of each widget from
the `ListWidgets` use case. Then we can display the list along with a link to
the view page, with an ID in the URL. The `ViewWidget` use case can fetch the
widget from the repository using the ID from the URL. However, the scenarios
have no mention of this ID, and rightly so! The stakeholder has no interest in
such a detail. From these scenarios we come up with a `DomainContext` which
contains something like this:

{% highlight php %}
/**
 * @var WidgetRepository
 */
private $repository;

/**
 * @var mixed
 */
private $result;

/**
 * @var array
 */
private $widgetIds = [];

/**
 * @Given there is a widget :name
 */
function thereIsAWidget($name)
{
    $this->repository->store(new Widget($name, 'test details'));
}

/**
 * @When I list widgets
 */
function iListWidgets()
{
    $usecase = new ListWidgets();

    $this->result = $usecase->execute();
}

/**
 * @Then I should see the following list of widgets:
 */
function iShouldSeeTheFollowingListOfWidgets(Table $table)
{
    $callback = function ($widget) {
        return [(string) $widget['name']];
    };

    Assert::assertEquals(
        array_map($callback, $this->result),
        array_map($callback, $table->getHash())
    );
}

/**
 * Given there is a widget :name with details :details
 */
function thereIsAWidgetWithDetails($name, $details)
{
    $this->repository->store(new Widget($name, $details));

    $this->widgetIds[$name] = $this->repository->getLastInsertId();
}

/**
 * @When I view widget :name
 */
function iViewWidget($name)
{
    $usecase = new ViewWidget();

    $this->result = $usecase->execute($this->widgetIds[$name]);
}

/**
 * @Then I should see widget :name as :value
 */
function iShouldSeeWidgetNameAs($name, $value)
{
    Asset::assertEquals($this->result[$name], $value);
}
{% endhighlight %}

Once all this passes we can feel rather good about ourselves, take off our
domain modelling hat, and put on our front end developer hat.

### Building the Front End

For the user interface testing we can create a new `UserInterfaceContext` which
makes use of `Mink` to inspect the pages. Rather than using the provided `Mink`
snippets, we want to use the same features running with the new context.

In the `UserInterfaceContext` we use Mink to nagivate the site:

* To test the *List Widgets* feature we simply navigate to
`http://devsite/widgets` and check the expected names of the widgets appear
on the page.
* To test viewing a widget, we navigate to the the list widgets page again, then
*click* the widget we want to view. This is better than creating a URL with an
ID in it because we probably don't actually want IDs in the URL - a nice
readable URL is much better!

Once we've built the user interface and the features pass then we can celebrate
our success and demonstrate the feature to the stakeholder.

The stakeholder is very happy. They then announce that they've decided to get
someone else to build a mobile application. This mobile application wants to be
able to view and list widgets as well. We now have to build a RESTful API for
the mobile app to connect to. No problem, we can create a new `ApiContext` and
make the same features pass again - or can we?

### The API Features

We've decided with the mobile app team that the API will provide a list of
widgets from a `GET` request to `http://devsite/api/widgets`. Also, an
individual widget's details will be available via
`http://devsite/api/widgets/some-id`.

Here's the issue, for the mobile app to work the ID is very important. We could
say that *in the context of the API the ID is part of the Ubiquitous Language*.
At this point we have 2 possible routes we can take, the first is to use a
new set of features with a RESTful API related Ubiquitous Language like so:

{% highlight cucumber %}
Feature: List Widgets

    Scenario: List Widgets
        Given there is a widget "Widget A"
        And there is a widget "Widget B"
        When I make a GET request to "/widgets"
        Then I should get the following collection:
            | id | name     |
            | 1  | Widget A |
            | 2  | Widget B |
{% endhighlight %}

{% highlight cucumber %}
Feature: View a widget

    Scenario: View a widget
        Given there is a widget "Widget A" which details "widget info" and id 5
        When I make a GET request to "/widgets/5"
        Then I should get a field called id with value 5
        And I should get a field called name with value "Widget A"
        And I should get a field called details value "widget info"
{% endhighlight %}

But there's something interesting to think about here - the ID is also of
interest to the `DomainContext` we created earlier. Since the ID was of no
interest to the stakeholder we thought it had no place in the feature files.
However, thinking about it now, in the domain context who are the features
really for? The stakeholder isn't really interested in the fact that the domain
model works, they're only really interested that the full stack and mobile
applications work. You could say that the features run against the
`DomainContext` are actually *integration tests*. Or, you could say that they
are *acceptance tests for developers*. If they are *acceptance tests for
developers* then can the "Ubiquitous Langauge" in this context contain terms of
interest to the developers? Are the developers the stakeholders?

### Tagging Scenarios

Behat provides a great tagging system. You can tag a scenario then, configure
Behat to include or exclude scenarios with certain tags from each suite. With
this in mind, we can take the original scenarios and add some extra tagged
scenarios which do know about the IDs. These can then be run against only the
`ApiContext` and `DomainContext` where IDs are appropriate language:

{% highlight cucumber %}
Feature: List Widgets

    @ui
    Scenario: List widgets
        Given there is a widget "Widget A"
        And there is a widget "Widget B"
        When I list widgets
        Then I should the following list of widgets:
            | name     |
            | Widget A |
            | Widget B |

    @api
    @domain
    Scenario: List widgets with ID
        Given there is a widget "Widget A"
        And there is a widget "Widget B"
        When I list widgets
        Then I should the following list of widgets:
            | id | name     |
            | 1  | Widget A |
            | 2  | Widget B |
{% endhighlight %}

{% highlight cucumber %}
Feature: View a widget

    @ui
    Scenario: View a widget
        Given there is a widget "Widget A" which details "a great widget"
        When I view widget "Widget A"
        Then I should see widget name "Widget A"
        And I should see widget details "a great widget"

    @api
    @domain
    Scenario: View a widget
        Given there is a widget "Widget A" which details "widget info" with id 5
        When I view widget "Widget A"
        Then I should see widget id 5"
        And I should see widget name "Widget A"
        And I should see widget details "widget info"
{% endhighlight %}

The Behat configuration would look something like this:

{% highlight yaml %}
default:
  suites:
    domain:
      contexts: [ DomainContext ]
      filters:  { tags: '@domain' }
    ui:
      contexts: [ UserInterfaceContext ]
      filters:  { tags: '@ui' }
    api:
      contexts: [ ApiContext ]
      filters:  { tags: '@api' }
{% endhighlight %}

## Conclusion

Which approach is best? Should the features have no knowledge of IDs and 
have separate features for the API? Or, should we try to use the features at
all levels an use tags to exclude context specific scenarios?

Personally, I think it depends on who the features are for and what language
they speak. This means either approach could be valid depending on the
parameters of the project and team.

I'd really love to hear everyone else thoughts on this!
