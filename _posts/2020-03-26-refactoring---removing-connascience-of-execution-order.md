---
title: Refactoring — Removing Connascience of Execution Order
description: Execution Order is the lowest strength dynamic [connascience](https://cloudnative.ly/connascense-9b3972c57415). The characterisation of dynamic connasciences is that they cannot be understood without considering how the code runs. When we encounter any dynamic connasciences in legacy codebases, they can present quite a challenge to understand. In this post, I present an example of how removing this type of coupling from the code can quickly help efforts to improve the understanding of the system.
featured_image: '/images/refactoring---removing-connascience-of-execution-order/featured-image.jpg'
---

# Refactoring — Removing Connascience of Execution Order

Image Source: GooKingSword via Pixabay

Execution Order is the lowest strength dynamic [connascience](https://cloudnative.ly/connascense-9b3972c57415). The characterisation of dynamic connasciences is that they cannot be understood without considering how the code runs. When we encounter any dynamic connasciences in legacy codebases, they can present quite a challenge to understand. In this post, I present an example of how removing this type of coupling from the code can quickly help efforts to improve the understanding of the system.

## How I Got Here

For this article, I will be using the [ugly trivia coding kata](https://github.com/jbrains/trivia). This is a legacy code kata where the exercise is to refactor the code to a more understandable and maintainable state.

I was first introduced to this kata about two years ago during a remote pairing session with [Adrian Bolboaca](https://twitter.com/adibolb) (with [part 1](https://www.youtube.com/watch?v=ewlVFipzoLg) and [part 2](https://www.youtube.com/watch?v=6YZRhN7ytVc) posted on his [YouTube channel](https://www.youtube.com/watch?v=6YZRhN7ytVc)). The approach we took was to create a [Golden Master test ](https://codurance.com/2012/11/11/testing-legacy-code-with-golden-master/)and then start extracting methods from the tangled code to try and gain understanding. I was never particularly pleased with the result of the pairing session as we never got to the point of truly understanding the code.

I’ve since tried the same approach that I used with Adrian several times and it’s never gone particularly well. It gives the illusion that the code is becoming understandable, but it never escapes the fact that there’s a strange coupling between methods which is very hard to reason about.

At DDD Europe 2020 I took part in [Johan Martinsson](undefined)’s Bug Zero Kata workshop. Johan uses the same kata as the basis of this workshop. This time I took a very different approach with my pair. Rather than starting by trying to make the code more descriptive, we removed the source of the confusion early on — this source of confusion was the *connascience of execution order*.

At first, this step seemed to make the code messier (creating one big method), but the result was that all the tangled code was brought together into one place and the connascience of execution order was removed. From this point on, refactoring was easy.

Now I’ll show this in action.

## The Initial Code

You can get the original, legacy [Ugly Trivia code from GitHub](https://github.com/jbrains/trivia). It comes in many different languages; I have chosen Java for this article. The code comprises of two classes, the Game class and the GameRunner class.

Below you can see some of the Game class, there’s more in it than what I have shown below, but this is the part we’re interested in. Please don’t try and understand the detail of this code, rather, notice that the methods roll(), wasAnsweredCorrectly() and wrongAnswer() are three different public methods that depend on the states of inPenaltyBox and isGettingOutOfPenaltyBox. Without knowing which order these are called in (or even when you do know) it’s very hard to understand what this code is doing.

    public class Game {
    *    /* More fields are defined here */*
    
        boolean[] inPenaltyBox  = new boolean[6];
    
        int currentPlayer = 0;
        boolean isGettingOutOfPenaltyBox;
    
        public void roll(int roll) {
            System.*out*.println(players.get(currentPlayer) +
                " is the current player");
            System.*out*.println("They have rolled a " + roll);
    
            if (**inPenaltyBox**[currentPlayer]) {
                if (roll % 2 != 0) {
                    **isGettingOutOfPenaltyBox** = true;
    
                    System.*out*.println(players.get(currentPlayer) +
                        " is getting out of the penalty box");
                    places[currentPlayer] = places[currentPlayer] +
                        roll;
                    if (places[currentPlayer] > 11)
                        places[currentPlayer] =
                            places[currentPlayer] - 12;
    
                    System.*out*.println(players.get(currentPlayer)
                            + "'s new location is "
                            + places[currentPlayer]);
                    System.*out*.println("The category is " +
                        currentCategory());
                    askQuestion();
                } else {
                    System.*out*.println(players.get(currentPlayer) +
                        " is not getting out of the penalty box");
                    **isGettingOutOfPenaltyBox** = false;
                }
            } else {
                places[currentPlayer] = places[currentPlayer] + roll;
                if (places[currentPlayer] > 11)
                    places[currentPlayer] = places[currentPlayer] - 12;
    
                System.*out*.println(players.get(currentPlayer)
                        + "'s new location is "
                        + places[currentPlayer]);
                System.*out*.println("The category is " +
                    currentCategory());
                askQuestion();
            }
        }
    
        public boolean wasCorrectlyAnswered() {
            if (**inPenaltyBox**[currentPlayer]){
                if (**isGettingOutOfPenaltyBox**) {
                    System.*out*.println("Answer was correct!!!!");
                    purses[currentPlayer]++;
                    System.*out*.println(players.get(currentPlayer)
                            + " now has "
                            + purses[currentPlayer]
                            + " Gold Coins.");
    
                    boolean winner = didPlayerWin();
                    currentPlayer++;
                    if (currentPlayer == players.size())
                        currentPlayer = 0;
    
                    return winner;
                } else {
                    currentPlayer++;
                    if (currentPlayer == players.size())
                        currentPlayer = 0;
                    return true;
                }
            } else {
                System.*out*.println("Answer was corrent!!!!");
                purses[currentPlayer]++;
                System.*out*.println(players.get(currentPlayer)
                        + " now has "
                        + purses[currentPlayer]
                        + " Gold Coins.");
    
                boolean winner = didPlayerWin();
                currentPlayer++;
                if (currentPlayer == players.size()) currentPlayer = 0;
    
                return winner;
            }
        }
    
        public boolean wrongAnswer(){
            System.*out*.println("Question was incorrectly answered");
            System.*out*.println(players.get(currentPlayer) +
                " was sent to the penalty box");
            **inPenaltyBox**[currentPlayer] = true;
    
            currentPlayer++;
            if (currentPlayer == players.size()) currentPlayer = 0;
            return true;
        }
    
        */* More methods are defined here */*
    }

In the GameRunner class, we get to see how these methods are called. The two classes are coupled, such that the order in which the methods are called in GameRunner is critical to the behaviour inside the methods in Game. This is the connascience of execution order.

    public class GameRunner {
    
        private static boolean *notAWinner*;
    
        public static void main(String[] args) {
            Game aGame = new Game();
    
            aGame.add("Chet");
            aGame.add("Pat");
            aGame.add("Sue");
    
            Random rand = new Random();
    
            do {
                **aGame.roll(rand.nextInt(5) + 1);**
    
                if (rand.nextInt(9) == 7) {
                    *notAWinner *= **aGame.wrongAnswer();**
                } else {
                    *notAWinner *= **aGame.wasCorrectlyAnswered();**
                }
            } while (*notAWinner*);
        }
    }

This connascience of execution order is what makes the code in Game hard to understand.

## Adding Tests

Before we make any changes, we need to add tests to give us the confidence that we don’t break anything. For legacy code like this, creating a [golden master](https://codurance.com/2012/11/11/testing-legacy-code-with-golden-master/) is a good approach. I’m not going to explain how to do this here, but you can find details in one of the following ways:

* Read [Sandro’s post](https://codurance.com/2012/11/11/testing-legacy-code-with-golden-master/) on golden master testing with the Approvals testing framework.

* Review the test from the [Bug Zero kata codebase](https://github.com/martinsson/BugsZero-Kata/blob/master/java/src/test/java/com/adaptionsoft/games/trivia/GameTest.java) which uses the [Approvals testing framework](https://approvaltests.com/).

* Review the [first video](https://www.youtube.com/watch?v=ewlVFipzoLg) of the session I recorded with Adrian.

Since the GameRunner uses Random to generate its behaviour, we need to seed it to have a consistent test. To make this possible, I used IntelliJ’s Extract Method refactoring to separate the creation of the Random object from the rest of the code like so:

    public class GameRunner {
        private static boolean *notAWinner*;
    
        public static void main(String[] args) {
            *run*(new Random());
        }
    
        public static void run(Random rand) {
            Game aGame = new Game();
    
            aGame.add("Chet");
            aGame.add("Pat");
            aGame.add("Sue");
    
            do {
                aGame.roll(rand.nextInt(5) + 1);
    
                if (rand.nextInt(9) == 7) {
                    *notAWinner *= aGame.wrongAnswer();
                } else {
                    *notAWinner *= aGame.wasCorrectlyAnswered();
                }
            } while (*notAWinner*);
        }
    }

This allows the golden master test to create its own instance of Random and call the run() method with it, while still keeping the main() function behaving as it always has. *Note, this is a temporary step and not something I’d leave in the codebase permanently.*

For completeness, my test looks like this.

    public class GameTest {
        @Test
        public void golden_master() {
            Approvals.*verify*(runWithStdout());
        }
    
        private String runWithStdout() {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            PrintStream ps = new PrintStream(baos);
            PrintStream old = System.*out*;
            System.*setOut*(ps);
    
            for (int seed = 1; seed < 20; seed++) {
                Random random = new Random(seed);
                GameRunner.*runMain*(random);
            }
    
            System.*out*.flush();
            System.*setOut*(old);
            return baos.toString();
        }
    }

Here you should use *code coverage reports* and *mutation testing* (see [Billie Thompson](undefined)’s video [here](https://cloudnative.ly/mutation-testing-f9e415046bb7) if you don’t know what mutation testing is) to gain some confidence about how much you can rely on this test — finding ways to increase the confidence if it’s not good enough. I’m not going to go into any more detail on that now but it may be something I write more about in the future.

## Removing Execution Order Connascience

Now we have a golden master test, we can confidently start refactoring. The first thing I want to try and do is remove the connascience of execution order. For this, I’m going to focus on the three methods being called inside the loop. Technically the calls to add() also form connascience of execution order, but they aren’t causing too much confusion right now, so I’ll leave that until later.

To remove connascience of execution order from the loop, I want GameRunner to make just one call to Game in each iteration of the loop. The first step I take is to separate the inputs from the logic inside of the loop in the run() method. Here the random numbers are the *inputs*, so I use the Extract Variable refactoring each time a random number is generated and move the declarations to the top of the loop (the important thing is to ensure the order of generating the numbers doesn’t change).

    public static void run(Random rand) {
       Game aGame = new Game();
    
       aGame.add("Chet");
       aGame.add("Pat");
       aGame.add("Sue");
    
       do {
    **      int roll = rand.nextInt(5) + 1;
          boolean incorrectAnswer = rand.nextInt(9) == 7;**
          
          aGame.roll(**roll**);
    
          if (**incorrectAnswer**) {
             *notAWinner *= aGame.wrongAnswer();
          } else {
             *notAWinner *= aGame.wasCorrectlyAnswered();
          }
       } while (*notAWinner*);
    }

I also notice that notAWinner is defined as a static field on the class, but it is only ever used in local scope, so I make it a local variable.

    public static void run(Random rand) {
       Game aGame = new Game();
    
       aGame.add("Chet");
       aGame.add("Pat");
       aGame.add("Sue");
    
       **boolean notAWinner;**
       
       do {
          int roll = rand.nextInt(5) + 1;
          boolean incorrectAnswer = rand.nextInt(9) == 7;
    
          aGame.roll(roll);
    
          if (incorrectAnswer) {
             notAWinner = aGame.wrongAnswer();
          } else {
             notAWinner = aGame.wasCorrectlyAnswered();
          }
       } while (notAWinner);
    }

I now use Extract Method to move the logic to a new playTurn() method.

    private static void run(Random rand) {
       Game aGame = new Game();
    
       aGame.add("Chet");
       aGame.add("Pat");
       aGame.add("Sue");
    
       boolean notAWinner;
    
       do {
          int roll = rand.nextInt(5) + 1;
          boolean incorrectAnswer = rand.nextInt(9) == 7;
    
          notAWinner = ***playTurn*(aGame, roll, incorrectAnswer)**;
       } while (notAWinner);
    }
    
    **private static boolean playTurn(
            Game aGame, int roll, boolean incorrectAnswer) {
       boolean notAWinner;
       aGame.roll(roll);
    
       if (incorrectAnswer) {
          notAWinner = aGame.wrongAnswer();
       } else {
          notAWinner = aGame.wasCorrectlyAnswered();
       }
       return notAWinner;
    }**

Finally, I can use IntelliJ’s Convert To Instance Method refactoring on playTurn() to move it into theGame class.

**GameRunner:**

    public class GameRunner {
       public static void main(String[] args) {
          *run*(new Random());
       }
    
       private static void run(Random rand) {
          Game aGame = new Game();
    
          aGame.add("Chet");
          aGame.add("Pat");
          aGame.add("Sue");
    
          boolean notAWinner;
    
          do {
             int roll = rand.nextInt(5) + 1;
             boolean incorrectAnswer = rand.nextInt(9) == 7;
    
             notAWinner = **aGame.playTurn(roll, incorrectAnswer);**
          } while (notAWinner);
       }
    }

**Game:**

    public class Game {
        /* Other class content */

    **    public boolean playTurn(int roll, boolean incorrectAnswer) {
            boolean notAWinner;
            roll(roll);
    
            if (incorrectAnswer) {
                notAWinner = wrongAnswer();
            } else {
                notAWinner = wasCorrectlyAnswered();
            }
            return notAWinner;
        }**
    }

At this point, I have removed the connascience of execution order from the loop. But what has this achieved?

We still have all this complicated logic spread throughout the methods in Game, but now all the calls to these methods is from the same class (in the language of connascience, the coupling is more local). One benefit here is that the Game and GameRunner classes are less coupled, but we’ve only made a minor impact on the clarity of the code. Let’s see what can be done now.

## Inlining the Methods

Because all these methods are now in the same class, I can inline them all into playTurn() and see if it presents any new ways to structure the code. Specifically, in this case, we can use the approach I described in [Refactoring — Untangling Conditionals](https://medium.com/p/cc5693b8ec3c/edit?source=your_stories_page---------------------------) to localise the complexity and quickly refactor the logic to a more desirable state.

The inlining looks like this:

    public boolean playTurn(int roll, boolean incorrectAnswer) {
        boolean notAWinner;
        System.*out*.println(players.get(currentPlayer) +
            " is the current player");
        System.*out*.println("They have rolled a " + roll);
    
        if (inPenaltyBox[currentPlayer]) {
            if (roll % 2 != 0) {
                isGettingOutOfPenaltyBox = true;
    
                System.*out*.println(players.get(currentPlayer) +
                    " is getting out of the penalty box");
                places[currentPlayer] =
                    places[currentPlayer] + roll;
                if (places[currentPlayer] > 11)
                    places[currentPlayer] = places[currentPlayer] - 12;
    
                System.*out*.println(players.get(currentPlayer)
                        + "'s new location is "
                        + places[currentPlayer]);
                System.*out*.println("The category is " + 
                    currentCategory());
                askQuestion();
            } else {
                System.*out*.println(players.get(currentPlayer) +
                    " is not getting out of the penalty box");
                isGettingOutOfPenaltyBox = false;
            }
        } else {
            places[currentPlayer] = places[currentPlayer] + roll;
            if (places[currentPlayer] > 11)
                places[currentPlayer] = places[currentPlayer] - 12;
    
            System.*out*.println(players.get(currentPlayer)
                    + "'s new location is "
                    + places[currentPlayer]);
            System.*out*.println("The category is " + currentCategory());
            askQuestion();
        }
    
        if (incorrectAnswer) {
            System.*out*.println("Question was incorrectly answered");
            System.*out*.println(players.get(currentPlayer) +
                " was sent to the penalty box");
            inPenaltyBox[currentPlayer] = true;
    
            currentPlayer++;
            if (currentPlayer == players.size()) currentPlayer = 0;
            notAWinner = true;
        } else {
            boolean result;
            if (inPenaltyBox[currentPlayer]) {
                if (isGettingOutOfPenaltyBox) {
                    System.*out*.println("Answer was correct!!!!");
                    purses[currentPlayer]++;
                    System.*out*.println(players.get(currentPlayer)
                            + " now has "
                            + purses[currentPlayer]
                            + " Gold Coins.");
    
                    boolean winner = didPlayerWin();
                    currentPlayer++;
                    if (currentPlayer == players.size())
                        currentPlayer = 0;
    
                    result = winner;
                } else {
                    currentPlayer++;
                    if (currentPlayer == players.size())
                        currentPlayer = 0;
                    result = true;
                }
            } else {
                System.*out*.println("Answer was corrent!!!!");
                purses[currentPlayer]++;
                System.*out*.println(players.get(currentPlayer)
                        + " now has "
                        + purses[currentPlayer]
                        + " Gold Coins.");
    
                boolean winner = didPlayerWin();
                currentPlayer++;
                if (currentPlayer == players.size()) currentPlayer = 0;
    
                result = winner;
            }
            notAWinner = result;
        }
        return notAWinner;
    }

This is a pretty horrible piece of code, but by applying the techniques from [Refactoring — Untangling Conditionals](https://medium.com/p/cc5693b8ec3c/edit?source=your_stories_page---------------------------), I’m able to start simplifying the code. This brings me to something like this:

    public boolean playTurn(int roll, boolean incorrectAnswer) {
        System.*out*.println(players.get(currentPlayer) +
            " is the current player");
        System.*out*.println("They have rolled a " + roll);

        boolean notAWinner;
        notAWinner = moveAndAnswerQuestion(roll, incorrectAnswer);
        nextPlayer();
        return notAWinner;
    }
    
    private boolean moveAndAnswerQuestion(int roll,
                                          boolean incorrectAnswer) {
        if (inPenaltyBox[currentPlayer]) {
            if (failedToLeavePenaltyBox(roll, incorrectAnswer)) {
                return true;
            }
        }
    
        move(roll);
        askQuestion();
    
        if (incorrectAnswer) {
            answerIncorrectly();
            return true;
        }
    
        answerCorrectly();
        return didPlayerWin();
    }
    
    private boolean failedToLeavePenaltyBox(int roll,
                                            boolean incorrectAnswer) {
        boolean rollIsOdd = roll % 2 != 0;
    
        if (rollIsOdd) {
            System.*out*.println(players.get(currentPlayer) +
                " is getting out of the penalty box");
            return false;
        }
    
        System.*out*.println(players.get(currentPlayer) +
            " is not getting out of the penalty box");
    
        if (incorrectAnswer) {
            answerIncorrectly();
            return true;
        }
    
        return true;
    }
    
    private void answerCorrectly() {
        System.*out*.println("Answer was correct!!!!");
        purses[currentPlayer]++;
        System.*out*.println(players.get(currentPlayer)
            + " now has "
            + purses[currentPlayer]
            + " Gold Coins.");
    }
    
    private void answerIncorrectly() {
        System.*out*.println("Question was incorrectly answered");
        System.*out*.println(players.get(currentPlayer) +
            " was sent to the penalty box");
        inPenaltyBox[currentPlayer] = true;
    }
    
    private void move(int roll) {
        places[currentPlayer] = places[currentPlayer] + roll;
        if (places[currentPlayer] > 11)
            places[currentPlayer] = places[currentPlayer] - 12;
    
        System.*out*.println(players.get(currentPlayer)
            + "'s new location is "
            + places[currentPlayer]);
        System.*out*.println("The category is " + currentCategory());
    }
    
    private void nextPlayer() {
        currentPlayer++;
        if (currentPlayer == players.size()) currentPlayer = 0;
    }

This last step took about 30 minutes of moving the structure of the code about and then extracting methods. The process was not mentally taxing and it didn’t require me to understand the code. It was just a process of repeatedly adjusting (improving) the structure and checking that the tests still passed.

During the process, I identified one typo bug and fixed that, and I completely removed the isGettingOutOfPenaltyBox field from the class. The result is not beautiful code, but it is rationalised enough that I can now start to understand the logic enough to make some more interesting architectural decisions (perhaps extracting out Roll and Player classes for example). I even started to understand enough to start questioning the logic and whether there might be a bug.

## Conclusion

To refactor legacy code, we don’t necessarily need to understand the logic we are refactoring first, rather, we can use the process of refactoring to gain understanding. However, **to do this we need tests that we trust! **We can use techniques like *golden master tests* to when we don’t already have tests, and we can use code coverage and mutation testing to start to gain confidence in these tests. *Note that the purpose of such a test is not to test the system behaviour is correct, but rather to test that we do not change the behaviour while we refactor.*

Once we are ready to start refactoring, we might be tempted to start applying techniques which will make the code “more readable”. However, some times there is some complexity which gets in the way of this. Identifying and removing that first can be a more productive approach. In this case, the complexity came from the connascience of execution order. If you identify any of the dynamic types of connascience in your code then it’s a good indicator that it might be a candidate for something to try and remove first.
