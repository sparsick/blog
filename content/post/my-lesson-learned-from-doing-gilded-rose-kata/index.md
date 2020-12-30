---
title: 'My Lesson Learned From Doing Gilded Rose Kata'
date: 2016-06-28
#description: "Article description." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
#codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Java
  - JVM
tags:
  - coding dojo
  - kata
# comment: false # Disable comment if false.
---
I'd like to share some of my thoughts about my approach to solve the Gilded Rose Refactoring Kata by Emily Bache. If you don't know this kata, read the [description](https://github.com/emilybache/GildedRose-Refactoring-Kata) for a better understanding. I have published my whole solution on [GitHub](https://github.com/sparsick/coding-katas/tree/master/GildedRose-Refactoring-Kata-Java) . I tried to make a commit after every step, so you can keep track of my steps in the log of git. The chosen programming language is Java.

Solving Gilded Rose Step-By-Step
================================

Let's have a look at what I have done step-by-step. Before adding the new feature, I wanted to refactor the given code base. Therefore, I started writing tests till I had a 100% line and branch coverage. During writing the tests, I was having the idea,, that the calculation of the quality is depended by the name of the item. Hence, the idea arose to use something similar like the [Strategy Pattern](https://en.wikipedia.org/wiki/Strategy_pattern). When I reached for 100% coverage, I tried to start with the implementation for the first strategy ("Aged Brie"). But I was unsure, what was my limit values for this first strategy. My problem was that I hadn't tests for the limit values. So my first lessons learned was that 100% line or branch coverage doesn't mean all test cases are covered. So I added tests for the limit values and finished implementing the "Aged Brie" strategy, added it to the original `updateQualtity` method (see below code snippet) and ran the tests. All tests were green.

```java
ItemStrategy itemStrategy = new ItemStrategy();
...
for (int i = 0; i < items.length; i++) {
   if("Aged Brie".equals(items[i].name)) {
      items[i] = itemStrategy.updateQualityForAgedBrieItem(items[i]);
      continue;
   }

// original code follows
}
```
These cycle I repeated four times: Find missing test cases (mostly for limit values); add new tests for these cases; implement a further strategy; add this new strategy to the original `updateQualtiy` method and ran the tests. If the tests are green, the next cycle with a new strategy begins. At the end the extended `updatedQuality` method looked like the following code snippet.

```java
ItemStrategy itemStrategy = new ItemStrategy();

...
for (int i = 0; i < items.length; i++) {
   if("Aged Brie".equals(items[i].name)) {
      items[i] = itemStrategy.updateQualityForAgedBrieItem(items[i]);
      continue;
   } else if ("Sulfuras, Hand of Ragnaros".equals(items[i].name)) {
      items[i] = itemStrategy.updateQualityForSulfurasItem(items[i]);
      continue;
   } else if("Backstage passes to a TAFKAL80ETC concert".equals(items[i].name)) {
      items[i] = itemStrategy.updateQualityForBackstagePassItem(items[i]);
      continue;
   } else {
      items[i] = itemStrategy.updateQualityForNormalItem(items[i]);
      continue;
   }

// commented out original code
}
```
My second Lessons Learned was "Refactoring needs time" and the refactoring wasn't finished. The next steps were cleaning up unnecessary code and refactoring the strategy implementations like replacing if-else construct by ternary operator and extracting if-condition to private methods. After that I implemented the new feature "conjured item" following the above describe work flow. After this step I could say "Ready", but I was unhappy with the if-else if-else chain. Therefore, I decided to extract each strategy implementation to an own class (following the "classic" strategy pattern). That helps to replace the if-else if-else chain by an `itemStrategyMap`. So the next Lesson Learned was "The status 'Ready' depends by the definition". The last step was doing clean up and choosing better names for the interface and its method.

```java
static Map<String, ItemStrategy> itemStrategyMap = new HashMap<>();

static {
   itemStrategyMap.put("Aged Brie", new AgedBrieItemStrategy());
   itemStrategyMap.put("Sulfuras, Hand of Ragnaros", new SulfurasItemStrategy());
   itemStrategyMap.put("Backstage passes to a TAFKAL80ETC concert", new BackstagePassItemStrategy());
   itemStrategyMap.put("Conjured", new ConjuredItemStrategy());
}

public void updateQuality() {
   for (int i = 0; i < items.length; i++) {
      ItemStrategy itemStrategy = itemStrategyMap.getOrDefault(items[i].name, new NormalItemStrategy());
      items[i] = itemStrategy.updateItem(items[i]);
   }
}
```
Let's summarize the Lesson Learned:

1. 100% line or branch coverage doesn't mean all test cases are covered.
2. Refactoring needs time.
3. The status 'Ready' depends by the definition. These insights aren't really new for me. I can often observe these insights in my daily work. Nevertheless, it was good to have these insights again, following the rule "learning through repetition" ☺

What I forgot
=============

I stopped after that step. Thinking about it some days later, I have realized that there exists more improvements. For example, the tests from `GildedRoseTest` class could be extracted to separate test classes regarding to the specific strategy classes.
