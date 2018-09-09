---
layout: post
title: "Trivia Kata: A Refactoring Story"
date: 2017-04-10 07:52:15 +0100
comments: true
categories: ["refactoring", "java"]
authors: ["renaud"]
---

Refactoring a legacy code can be a tricky business. More often than not, the code is not tested, unclear and contains some bugs. If not planned, the time required to refactor will eat all the time allocated to the feature. The feature will then be implemented very quickly and it will result in more buggy unmaintainable code.

In this blog post, I will try to show some tools and methods to refactor safely a piece of code. I will use the [trivia kata](https://github.com/jbrains/trivia) as a support for this. The resulting code can be found on [GitHub](https://github.com/rnowif/trivia-refactoring/commits/master).

<!-- more -->

## Characterization Test: the Golden Master

The prerequisite to any refactor is to build a test harness to be sure that the refactoring will not break anything. In this example, it could seem like a waste of time to test all possible paths with unit tests. Fortunately, the code contains a lot of traces that print what the code does. It is then possible to run the code with a set of inputs, capture the output and save it. This output is called the *Golden Master*. Every time the code is modified, the same inputs are used to run the code and the output is compared to the golden master. As you may have noticed, this will not ensure that the code is bug-free, it will just make sure that the code always **behaves** the same.

In Java, the golden master can be implemented using a library called _approval_:

```java
<dependency>
    <groupId>com.github.nikolavp</groupId>
    <artifactId>approval-core</artifactId>
    <version>0.3</version>
</dependency>
```

The associated test is shown below (see [commit](https://github.com/rnowif/trivia-refactoring/commit/0c4255e69cc5395a70de417913de20c82606b0c2)):

```java
@Test
public void should_record_and_verify_golden_master() {
    String result = playGame(1L);

    Approvals.verify(result, Paths.get("src", "main", "resources", "approval", "result.txt")); // 2
}

private String playGame(long seed) {
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    System.setOut(new PrintStream(outputStream)); // 3

    boolean notAWinner;
    Game aGame = new Game();

    aGame.add("Chet");
    aGame.add("Pat");
    aGame.add("Sue");

    Random rand = new Random(seed); // 1

    do {

        aGame.roll(rand.nextInt(5) + 1);

        if (rand.nextInt(9) == 7) {
            notAWinner = aGame.wrongAnswer();
        } else {
            notAWinner = aGame.wasCorrectlyAnswered();
        }

    } while (notAWinner);

    return new String(outputStream.toByteArray());
}
```

Since the code uses a `Random` to roll a dice, we have to fix the seed of the `Random` so it always returns the same random rolls in the same order (`1`). The code has to be run once with this seed and the output has to be pasted into `src/main/resources/approval/result.txt` to create the golden master (`2`). Afterwards, the test will capture the output (`3`) and compare it to the golden master. If a difference appears, a diff will prompt.

Sometimes during my refactoring sessions, I like to make mistakes on purpose just to check the robustness of my test harness. I find it very useful to increase the confidence you have in your tests and fix them if you have to.

## First Step: Clean Up

Personnaly, I can't really think when I am in front of a bloated and unclear code. Thus, the first thing I did to refactor this piece of code is to rearrange it, delete unused imports and dead code, remove magic strings and magic numbers, use generics and reduce the visibility of fields and methods that can be reduced to be sure that I can touch them without breaking the public API. The major part of these refactorings can be made for you by your IDE so don't hesitate to do it as it reduces the risk of error and it is much quicker. You can also use a plugin like [SonarLint](http://www.sonarlint.org/index.html) for instance to detect issues directly into your IDE.

Once the code easier to think about, I noticed that strings where used to identify categories. This seemed like a good place to create an enum (see [commit](https://github.com/rnowif/trivia-refactoring/commit/1578c0ce9d7da73641e5f4202b1a72f00ba13723)). To start, I just replaced the strings with the enum:

```java
private void askQuestion() {
    if (currentCategory() == "Pop")
        print(popQuestions.removeFirst());
    if (currentCategory() == "Science")
        print(scienceQuestions.removeFirst());
    if (currentCategory() == "Sports")
        print(sportsQuestions.removeFirst());
    if (currentCategory() == "Rock")
        print(rockQuestions.removeFirst());
}

private String currentCategory() {
    if (places[currentPlayer] == 0) return "Pop";
    if (places[currentPlayer] == 4) return "Pop";
    if (places[currentPlayer] == 8) return "Pop";
    if (places[currentPlayer] == 1) return "Science";
    if (places[currentPlayer] == 5) return "Science";
    if (places[currentPlayer] == 9) return "Science";
    if (places[currentPlayer] == 2) return "Sports";
    if (places[currentPlayer] == 6) return "Sports";
    if (places[currentPlayer] == 10) return "Sports";
    return "Rock";
}
```

became

```java
private void askQuestion() {
    if (currentCategory().equals(Category.POP))
        print(popQuestions.removeFirst());
    if (currentCategory().equals(Category.SCIENCE))
        print(scienceQuestions.removeFirst());
    if (currentCategory().equals(Category.SPORTS))
        print(sportsQuestions.removeFirst());
    if (currentCategory().equals(Category.ROCK))
        print(rockQuestions.removeFirst());
}

private Category currentCategory() {
    if (places[currentPlayer] == 0) return Category.POP;
    if (places[currentPlayer] == 4) return Category.POP;
    if (places[currentPlayer] == 8) return Category.POP;
    if (places[currentPlayer] == 1) return Category.SCIENCE;
    if (places[currentPlayer] == 5) return Category.SCIENCE;
    if (places[currentPlayer] == 9) return Category.SCIENCE;
    if (places[currentPlayer] == 2) return Category.SPORTS;
    if (places[currentPlayer] == 6) return Category.SPORTS;
    if (places[currentPlayer] == 10) return Category.SPORTS;
    return Category.ROCK;
}
```

This kind of `if` statement that looks a lot like a `switch` can almost always be replaced by a `Map` so that is what I did (see [commit](https://github.com/rnowif/trivia-refactoring/commit/b8780743929eb10ab5f8a90c9fe68c9dd3c310a9)). I created two maps to contain questions of each category and the category for each position:

```java
private void askQuestion() {
    print(questionsByCategory.get(currentCategory()).removeFirst());
}

private Category currentCategory() {
   return categoriesByPosition.get(currentPosition());
}
```

## Second Step: Segregate

The second step I chose to follow is to put specific logic into private methods. Once again, your IDE can help you here, use it!

For instance, that piece of code:

```java
print(players.get(currentPlayer) + " is getting out of the penalty box");

places[currentPlayer] = places[currentPlayer] + roll;
if (places[currentPlayer] >= NB_CELLS) {
    places[currentPlayer] = places[currentPlayer] - NB_CELLS;
}

print(players.get(currentPlayer)  + "'s new location is " + places[currentPlayer]);
```

became

```java
print(players.get(currentPlayer) + " is getting out of the penalty box");
move(roll);
print(players.get(currentPlayer) + "'s new location is " + currentPosition());
```

By doing that, you can remove duplication by factorizing algorithms (like move a player) into specific methods. Moreover, you prepare the field for extracting objects from this code (see the next section). However, in order to do that properly, the private methods have to be in the purest form possible: they should use or modify the least amount of private fields possible (see [commit](https://github.com/rnowif/trivia-refactoring/commit/160c52d4a393fef3a58ad062fa9b552f0a9631a0)). With that in mind, the following code:

```java
private Category currentCategory() {
    return categoriesByPosition.get(currentPosition());
}
```

has been replaced by

```java
private Category currentCategory(int position) {
    return categoriesByPosition.get(position);
}
```

## Third Step: Divide and Conquer

Now that logic has been segregated into private methods, it is pretty straightforward to extract objects from these methods.

In this kata, we saw that several concepts emerge: `Player`, `Board` and `QuestionDeck`. A `PlayerList` class has also been added to handle the players rotation. All these classes can be tested in isolation in order not to rely solely on the golden master anymore.

For instance, the following code prints the next question to ask:

```java
public void roll(int roll) {

  // ...

  Category currentCategory = board.categoryOf(newPosition);
  print("The category is " + currentCategory);
  print(nextQuestionAbout(currentCategory));
}

private String nextQuestionAbout(Category category) {
  return questionsByCategory.get(category).removeFirst();
}
```

By creating a `QuestionDeck` object, we can have a code like this (see [commit](https://github.com/rnowif/trivia-refactoring/commit/6ae3f22923b425d51f7fb657d7236926d2149309)):

```java
public void roll(int roll) {

  // QuestionDeck deck = new QuestionDeck(NB_QUESTIONS, CATEGORIES);

  Category currentCategory = board.categoryOf(newPosition);
  print("The category is " + currentCategory);
  print(deck.nextQuestionAbout(currentCategory));
}
```

The private method `nextQuestionAbout` and the field `questionsByCategory` are no longer in the `Game` class.

When you extract classes from the main code, it is very important not to break the public API. It is possible to annotate some methods with `@Deprecated` though (see [commit](https://github.com/rnowif/trivia-refactoring/commit/a7c4d85c50395a0a66908eb15e3926b6d8d822de)). A good example of this can be seen in this [video](https://www.crafties.fr/videos/2017-02-27-episode-9-refactoring-dependances-statiques.html).

## Fourth Step: Break up

The last step is when we remove all deprecated methods that we introduced during the refactoring. It is not always possible to do it right after the refactoring session as we don't always have access to the client code that consumes our public API. In this case we do, so I removed these methods (see [commit](https://github.com/rnowif/trivia-refactoring/commit/eb7ab019f62808b589de0f6a4fcf904163fd667d)).

The client code looked like this:

```java
Game aGame = new Game();

aGame.add("Chet");
aGame.add("Pat");
aGame.add("Sue");

// aGame.roll()
```

Now the `Game` class is not in charge of questions, board, categories and players anymore so we have to inject them into the constructor.

```java
private static final int NB_CELLS = 12;
private static final int NB_QUESTIONS = 50;
private static final List<Category> CATEGORIES = asList(Category.POP, Category.SCIENCE, Category.SPORTS, Category.ROCK);
private static final List<String> PLAYERS = asList("Chet", "Pat", "Sue");

// ...
Game aGame = new Game(
        System.out,
        new Board(NB_CELLS, CATEGORIES),
        new QuestionDeck(NB_QUESTIONS, CATEGORIES),
        new PlayerList(PLAYERS)
);

// aGame.roll()
```

## Wrapping Up

Of course, as it is always the case with refactoring, I could have done more. The main risk is that _more_ does not always mean _better_. A refactoring session must have a purpose and it is very important to know when to stop. It could be when you can add your new feature or fix a bug seamlessly or simply because you don't have any more time allocated to this task.

I hope that this post gave you some tools and methods to refactor your code without the fear of breaking something.
