---
title: Refactoring — Untangling Conditionals
description: As programmers, we love to minimise duplication and maximise reuse. We love the acronym DRY — Don’t Repeat Yourself, we hate deeply nested blocks of code, and we like small short methods and functions. However, sometimes duplicating code and embedding blocks inside other blocks can be the perfect enabler for refactoring. This is what I will demonstrate in this article.
featured_image: '/images/refactoring---untangling-conditionals/featured-image.jpg'
---

As programmers, we love to minimise duplication and maximise reuse. We love the
acronym DRY — Don’t Repeat Yourself, we hate deeply nested blocks of code, and
we like small short methods and functions. However, sometimes duplicating code
and embedding blocks inside other blocks can be the perfect enabler for
refactoring. This is what I will demonstrate in this article.

*Just to be clear, what I’m talking about are steps along the way in your
refactoring journey. I’m not suggesting that you should leave long methods full
of duplication and nested if statements in your code, rather, these are small
steps you can take along the way. I previously recorded some videos about
taking small steps when refactoring
[here](https://cloudnative.ly/parallel-change-refactoring-f5f3c9188a90) and
[here](https://cloudnative.ly/composing-refactorings-989f93f4a7a2), you may
want to watch them first.*

## The Starting Code

I’ve created a small refactoring kata for this article (the repository is
available [here](https://github.com/tomphp/untangled-conditionals-kata) if you
want to try it yourself). It starts with the following code. The goal is to
untangle it into something easier to understand.

```java
public void run(Project project) {
    boolean testsPassed;
    boolean buildSuccessful;

    if (project.hasTests()) {
        if ("success".equals(project.runTests())) {
            log.info("Tests passed");
            testsPassed = true;
        } else {
            log.severe("Tests failed");
            testsPassed = false;
        }
    } else {
        log.info("No tests");
        testsPassed = true;
    }

    if (testsPassed) {
        if ("success".equals(project.build())) {
            log.info("Deployment successful");
            buildSuccessful = true;
        } else {
            log.severe("Deployment failed");
            buildSuccessful = false;
        }
    } else {
        buildSuccessful = false;
    }

    if (config.sendEmailSummary()) {
        log.info("Sending email");
        if (testsPassed) {
            if (buildSuccessful) {
                emailer.send("Deployment completed successfully");
            } else {
                emailer.send("Deployment failed");
            }
        } else {
            emailer.send("Tests failed");
        }
    } else {
        log.info("Email disabled");
    }
}
```

The sign that the refactoring approach that I’m about to demonstrate is
appropriate in this code is that there are values that are being set, checked,
updated, and rechecked through multiple if statements. That is, there are
multiple top-level if blocks that a coupled by local, mutable state.

![](https://cdn-images-1.medium.com/max/2140/1*yU4gwSuRompdIWm50TmBFQ.png)

All these embedded if statements make this code hard to follow. Maybe we can
find a way to simplify this code by bringing the sets and checks closer
together, let’s try it.

## Adding Duplication

My initial goal here is to bring the checks of testsPassed and buildSuccessful
closer to where they have their values set — maybe then we can even remove the
mutable state altogether.

I’m going to start by moving the second top-level if block into **both** of the
branches of the first top-level if block.

```java
public void run(Project project) {
    boolean testsPassed;
    boolean deploySuccessful;

    if (project.hasTests()) {
        if ("success".equals(project.runTests())) {
            log.info("Tests passed");
            testsPassed = true;
        } else {
            log.severe("Tests failed");
            testsPassed = false;
        }

        if (testsPassed) {
            if ("success".equals(project.deploy())) {
                log.info("Deployment successful");
                deploySuccessful = true;
            } else {
                log.severe("Deployment failed");
                deploySuccessful = false;
            }
        } else {
            deploySuccessful = false;
        }
    } else {
        log.info("No tests");
        testsPassed = true;
        
        if (testsPassed) {
            if ("success".equals(project.deploy())) {
                log.info("Deployment successful");
                deploySuccessful = true;
            } else {
                log.severe("Deployment failed");
                deploySuccessful = false;
            }
        } else {
            deploySuccessful = false;
        }
    }

    if (config.sendEmailSummary()) {
        log.info("Sending email");
        if (testsPassed) {
            if (deploySuccessful) {
                emailer.send("Deployment completed successfully");
            } else {
                emailer.send("Deployment failed");
            }
        } else {
            emailer.send("Tests failed");
        }
    } else {
        log.info("Email disabled");
    }
}
```

In fact, I’m going to do the same thing again with the inner if statements in
the first branch.

```java
public void run(Project project) {
    boolean testsPassed;
    boolean deploySuccessful;

    if (project.hasTests()) {
        if ("success".equals(project.runTests())) {
            log.info("Tests passed");
            testsPassed = true;
            if (testsPassed) {
                if ("success".equals(project.deploy())) {
                    log.info("Deployment successful");
                    deploySuccessful = true;
                } else {
                    log.severe("Deployment failed");
                    deploySuccessful = false;
                }
            } else {
                deploySuccessful = false;
            }
        } else {
            log.severe("Tests failed");
            testsPassed = false;
            if (testsPassed) {
                if ("success".equals(project.deploy())) {
                    log.info("Deployment successful");
                    deploySuccessful = true;
                } else {
                    log.severe("Deployment failed");
                    deploySuccessful = false;
                }
            } else {
                deploySuccessful = false;
            }
        }
    } else {
        log.info("No tests");
        testsPassed = true;

        if (testsPassed) {
            if ("success".equals(project.deploy())) {
                log.info("Deployment successful");
                deploySuccessful = true;
            } else {
                log.severe("Deployment failed");
                deploySuccessful = false;
            }
        } else {
            deploySuccessful = false;
        }
    }

    if (config.sendEmailSummary()) {
        log.info("Sending email");
        if (testsPassed) {
            if (deploySuccessful) {
                emailer.send("Deployment completed successfully");
            } else {
                emailer.send("Deployment failed");
            }
        } else {
            emailer.send("Tests failed");
        }
    } else {
        log.info("Email disabled");
    }
}
```

The tests still pass but this definitely doesn’t feel like an improvement
right?

However, now we have some of the if statements right after where testsPassed is
being set. This makes some of these inner branches impossible to be reached, so
let’s remove those.

*Note, IntelliJ will help you locate and remove unreachable branches.*

```java
public void run(Project project) {
    boolean testsPassed;
    boolean deploySuccessful;

    if (project.hasTests()) {
        if ("success".equals(project.runTests())) {
            log.info("Tests passed");
            testsPassed = true;
            if ("success".equals(project.deploy())) {
                log.info("Deployment successful");
                deploySuccessful = true;
            } else {
                log.severe("Deployment failed");
                deploySuccessful = false;
            }
        } else {
            log.severe("Tests failed");
            testsPassed = false;
            deploySuccessful = false;
        }
    } else {
        log.info("No tests");
        testsPassed = true;

        if ("success".equals(project.deploy())) {
            log.info("Deployment successful");
            deploySuccessful = true;
        } else {
            log.severe("Deployment failed");
            deploySuccessful = false;
        }
    }

    if (config.sendEmailSummary()) {
        log.info("Sending email");
        if (testsPassed) {
            if (deploySuccessful) {
                emailer.send("Deployment completed successfully");
            } else {
                emailer.send("Deployment failed");
            }
        } else {
            emailer.send("Tests failed");
        }
    } else {
        log.info("Email disabled");
    }
}
```

That’s improved things a bit, but we are still left with some duplication.
Before we address that, let’s repeat the process we just went through, this
time for the bottom top-level if statement.

First duplicate:

```java
public void run(Project project) {
    boolean testsPassed;
    boolean deploySuccessful;

    if (project.hasTests()) {
        if ("success".equals(project.runTests())) {
            log.info("Tests passed");
            testsPassed = true;
            if ("success".equals(project.deploy())) {
                log.info("Deployment successful");
                deploySuccessful = true;
                if (config.sendEmailSummary()) {
                    log.info("Sending email");
                    if (testsPassed) {
                        if (deploySuccessful) {
                            emailer.send("Deployment completed successfully");
                        } else {
                            emailer.send("Deployment failed");
                        }
                    } else {
                        emailer.send("Tests failed");
                    }
                } else {
                    log.info("Email disabled");
                }
            } else {
                log.severe("Deployment failed");
                deploySuccessful = false;
                if (config.sendEmailSummary()) {
                    log.info("Sending email");
                    if (testsPassed) {
                        if (deploySuccessful) {
                            emailer.send("Deployment completed successfully");
                        } else {
                            emailer.send("Deployment failed");
                        }
                    } else {
                        emailer.send("Tests failed");
                    }
                } else {
                    log.info("Email disabled");
                }
            }
        } else {
            log.severe("Tests failed");
            testsPassed = false;
            deploySuccessful = false;
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                if (testsPassed) {
                    if (deploySuccessful) {
                        emailer.send("Deployment completed successfully");
                    } else {
                        emailer.send("Deployment failed");
                    }
                } else {
                    emailer.send("Tests failed");
                }
            } else {
                log.info("Email disabled");
            }
        }
    } else {
        log.info("No tests");
        testsPassed = true;

        if ("success".equals(project.deploy())) {
            log.info("Deployment successful");
            deploySuccessful = true;
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                if (testsPassed) {
                    if (deploySuccessful) {
                        emailer.send("Deployment completed successfully");
                    } else {
                        emailer.send("Deployment failed");
                    }
                } else {
                    emailer.send("Tests failed");
                }
            } else {
                log.info("Email disabled");
            }
        } else {
            log.severe("Deployment failed");
            deploySuccessful = false;
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                if (testsPassed) {
                    if (deploySuccessful) {
                        emailer.send("Deployment completed successfully");
                    } else {
                        emailer.send("Deployment failed");
                    }
                } else {
                    emailer.send("Tests failed");
                }
            } else {
                log.info("Email disabled");
            }
        }
    }
}
```

Then remove all unreachable branches:

```java
public void run(Project project) {
    boolean testsPassed;
    boolean deploySuccessful;

    if (project.hasTests()) {
        if ("success".equals(project.runTests())) {
            log.info("Tests passed");
            testsPassed = true;
            if ("success".equals(project.deploy())) {
                log.info("Deployment successful");
                deploySuccessful = true;
                if (config.sendEmailSummary()) {
                    log.info("Sending email");
                    emailer.send("Deployment completed successfully");
                } else {
                    log.info("Email disabled");
                }
            } else {
                log.severe("Deployment failed");
                deploySuccessful = false;
                if (config.sendEmailSummary()) {
                    log.info("Sending email");
                    emailer.send("Deployment failed");
                } else {
                    log.info("Email disabled");
                }
            }
        } else {
            log.severe("Tests failed");
            testsPassed = false;
            deploySuccessful = false;
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                emailer.send("Tests failed");
            } else {
                log.info("Email disabled");
            }
        }
    } else {
        log.info("No tests");
        testsPassed = true;

        if ("success".equals(project.deploy())) {
            log.info("Deployment successful");
            deploySuccessful = true;
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                emailer.send("Deployment completed successfully");
            } else {
                log.info("Email disabled");
            }
        } else {
            log.severe("Deployment failed");
            deploySuccessful = false;
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                emailer.send("Deployment failed");
            } else {
                log.info("Email disabled");
            }
        }
    }
}
```

Now we see that the intermediate variables are no longer used, let’s remove
them too:

```java
public void run(Project project) {
    if (project.hasTests()) {
        if ("success".equals(project.runTests())) {
            log.info("Tests passed");
            if ("success".equals(project.deploy())) {
                log.info("Deployment successful");
                if (config.sendEmailSummary()) {
                    log.info("Sending email");
                    emailer.send("Deployment completed successfully");
                } else {
                    log.info("Email disabled");
                }
            } else {
                log.severe("Deployment failed");
                if (config.sendEmailSummary()) {
                    log.info("Sending email");
                    emailer.send("Deployment failed");
                } else {
                    log.info("Email disabled");
                }
            }
        } else {
            log.severe("Tests failed");
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                emailer.send("Tests failed");
            } else {
                log.info("Email disabled");
            }
        }
    } else {
        log.info("No tests");

        if ("success".equals(project.deploy())) {
            log.info("Deployment successful");
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                emailer.send("Deployment completed successfully");
            } else {
                log.info("Email disabled");
            }
        } else {
            log.severe("Deployment failed");
            if (config.sendEmailSummary()) {
                log.info("Sending email");
                emailer.send("Deployment failed");
            } else {
                log.info("Email disabled");
            }
        }
    }
}
```

We are left with a single, large, multi-level conditional block. This block
contains quite a lot of duplication, so now let’s change our focus to
addressing that.

## Removing Duplication

The most obviously repeated blocks are the if (config.sendEmailSummary()) ones.
Let’s use extract method to remove this duplication.

```java
public void run(Project project) {
    if (project.hasTests()) {
        if ("success".equals(project.runTests())) {
            log.info("Tests passed");
            if ("success".equals(project.deploy())) {
                log.info("Deployment successful");
                sendEmail("Deployment completed successfully");
            } else {
                log.severe("Deployment failed");
                sendEmail("Deployment failed");

            }
        } else {
            log.severe("Tests failed");
            sendEmail("Tests failed");
        }
    } else {
        log.info("No tests");

        if ("success".equals(project.deploy())) {
            log.info("Deployment successful");
            sendEmail("Deployment completed successfully");
        } else {
            log.severe("Deployment failed");
            sendEmail("Deployment failed");
        }
    }
}

private void sendEmail(String message) {
    if (config.sendEmailSummary()) {
        log.info("Sending email");
        emailer.send(message);
    } else {
        log.info("Email disabled");
    }
}
```

Now things are starting to look better!

Next, we can use the Invert 'if' condition and *convert conditional to guard
clause* refactorings to make it possible to remove the other duplicated block.

```java
public void run(Project project) {
    if (project.hasTests()) {
        if (!"success".equals(project.runTests())) {
            log.severe("Tests failed");
            sendEmail("Tests failed");
            return;
        }
        log.info("Tests passed");
    } else {
        log.info("No tests");
    }

    if ("success".equals(project.deploy())) {
        log.info("Deployment successful");
        sendEmail("Deployment completed successfully");
    } else {
        log.severe("Deployment failed");
        sendEmail("Deployment failed");
    }
}

private void sendEmail(String message) {
    if (config.sendEmailSummary()) {
        log.info("Sending email");
        emailer.send(message);
    } else {
        log.info("Email disabled");
    }
}
```

Finally, we can use all the basic refactoring techniques to *extract methods*
and *remove else statements. *With a bit of work we can refactor the code to
something much more understandable and manageable:

```java
public void run(Project project) {
    if (testsFailed(project)) {
        sendEmail("Tests failed");
        return;
    }

    if (deployFailed(project)) {
        sendEmail("Deployment failed");
        return;
    }

    sendEmail("Deployment completed successfully");
}

private boolean testsFailed(Project project) {
    if (!project.hasTests()) {
        log.info("No tests");
        return false;
    }

    if ("success".equals(project.runTests())) {
        log.info("Tests passed");
        return false;
    }

    log.severe("Tests failed");
    return true;
}

private boolean deployFailed(Project project) {
    boolean deployFailed = !"success".equals(project.deploy());
    if (deployFailed) {
        log.severe("Deployment failed");
        return true;
    }

    log.info("Deployment successful");
    return false;
}

private void sendEmail(String s) {
    if (!config.sendEmailSummary()) {
        log.info("Email disabled");
        return;
    }

    log.info("Sending email");
    emailer.send(s);
}
```

## Final Thoughts

Often when refactoring, it’s tempting to reach straight for techniques which
break our code down into small, digestible pieces (e.g. *extract method*). If
the bigger pieces (i.e. the three top-level if blocks in the example in the
article) that we are breaking down are tightly coupled to each other, then
breaking each of them down independently leave things even more tangled. A way
to avoid this is to bring the coupled pieces of code together first — even if
this introduces duplication — and then start to address the duplications once
the coupling is localised.

I hope you found this useful and maybe have a go at the kata yourself ;-)

## Resources

* [The Untangled Conditionals Kata](https://github.com/tomphp/untangled-conditionals-kata)

---

Featured photo by
[_Alicja_](https://pixabay.com/users/_alicja_-5975425/) on
[Pixabay](https://pixabay.com/photos/wool-yarn-color-tangled-handicraft-3820129/).

