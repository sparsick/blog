---
title: 'Strategy Pattern Revisited With Spring'
date: 2019-09-18
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
  - Spring Framework
tags:
  - design pattern
  - spring


# comment: false # Disable comment if false.
---


This blog post wants to show another approach how to implement the Strategy Pattern with dependency injection. As DI framework, I choose Spring framework

![](https://upload.wikimedia.org/wikipedia/commons/4/45/W3sDesign_Strategy_Design_Pattern_UML.jpg)

From [Wikipedia](https://en.wikipedia.org/wiki/Strategy_pattern)

Firstly, let's have a look how the Strategy Pattern is implemented in the classic way.  
As starting point, we have a `HeroController` that should add a hero in `HeroRepository` depends on which repository was chosen by the user.

```java
package com.github.sparsick.springbootexample.hero.universum;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
public class HeroControllerClassicWay {

    @PostMapping("/hero/new")
    public String addNewHero(@ModelAttribute("newHero") NewHeroModel newHeroModel) {
        HeroRepository heroRepository = findHeroRepository(newHeroModel.getRepository());
        heroRepository.addHero(newHeroModel.getHero());
        return "redirect:/hero";
    }

    private HeroRepository findHeroRepository(String repositoryName) {
        if (repositoryName.equals("Unique")) {
            return new UniqueHeroRepository();
        }

        if(repositoryName.equals(("Duplicate")){
            return new DuplicateHeroRepository();
        }

        throw new IllegalArgumentException(String.format("Find no repository for given repository name \[%s\]", repositoryName));
    }
}
```

```java
package com.github.sparsick.springbootexample.hero.universum;

import java.util.Collection;
import java.util.HashSet;
import java.util.Set;

import org.springframework.stereotype.Repository;

@Repository
public class UniqueHeroRepository implements HeroRepository {

    private Set<Hero> heroes = new HashSet<>();

    @Override
    public String getName() {
        return "Unique";
    }

    @Override
    public void addHero(Hero hero) {
        heroes.add(hero);
    }

    @Override
    public Collection<Hero> allHeros() {
        return new HashSet<>(heroes);
    }

}
```

```java
package com.github.sparsick.springbootexample.hero.universum;

import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

@Repository
public class DuplicateHeroRepository implements HeroRepository {

    private List<Hero> heroes = new ArrayList<>();

    @Override
    public void addHero(Hero hero) {
        heroes.add(hero);
    }

    @Override
    public Collection<Hero> allHeros() {
        return List.copyOf(heroes);
    }

    @Override
    public String getName() {
        return "Duplicate";
    }
}
```

This implementation has some pitfalls. The creation of the repository implementations aren't managed by the Spring Context (it breaks the dependency injection / inverse of control). This will be painful as soon as you want to expand the repository implementation with further feature that need to inject other classes (for example, counting the usage of this class with `MeterRegistry`).

```java
package com.github.sparsick.springbootexample.hero.universum;

import java.util.Collection;
import java.util.HashSet;
import java.util.Set;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Repository;

@Repository
public class UniqueHeroRepository implements HeroRepository {

    private Set<Hero> heroes = new HashSet<>();
    private Counter addCounter;

    public UniqueHeroRepository(MeterRegistry meterRegistry) {
        addCounter = meterRegistry.counter("hero.repository.unique");
    }

    @Override
    public String getName() {
        return "Unique";
    }

    @Override
    public void addHero(Hero hero) {
        addCounter.increment();
        heroes.add(hero);
    }

    @Override
    public Collection<Hero> allHeros() {
        return new HashSet<>(heroes);
    }

}
```

It breaks also the separation of concern. When I want to test the controller class, I have no possibility to mock the repository interface easily. So the first idea is to put the creation of repository implementation to the Spring context. The repository implementation are annotated with `@Repository` annotation. So Spring's component scan find them.  
The next question how to inject them into the controller class. Here, a Spring feature can help. I define a list of `HeroRepository` in the controller. This list has to be filled during the creation of the controller instance.

```java
package com.github.sparsick.springbootexample.hero.universum;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

import java.util.List;

@Controller
public class HeroControllerRefactoringStep1 {

    private List<HeroRepository> heroRepositories;

    public HeroControllerRefactoringStep1(List<HeroRepository> heroRepositories) {
        this.heroRepositories = heroRepositories;
    }

    @PostMapping("/hero/new")
    public String addNewHero(@ModelAttribute("newHero") NewHeroModel newHeroModel) {
        HeroRepository heroRepository = findHeroRepository(newHeroModel.getRepository());
        heroRepository.addHero(newHeroModel.getHero());
        return "redirect:/hero";
    }

    private HeroRepository findHeroRepository(String repositoryName) {
        return heroRepositories.stream()
                .filter(heroRepository -> heroRepository.getName().equals(repositoryName))
                .findFirst()
                .orElseThrow(()-> new IllegalArgumentException(String.format("Find no repository for given repository name \[%s\]", repositoryName)));

    }
}
```

Spring searches in its context for all implementation of the interface `HeroRepostiory` and put them all to the list. One disadvantage has this solution, every adding a hero browses the list of `HeroRepository` to find the right implementation. This can be optimized by creating a map in the controller constructor that has the repository name as key and the corresponded implementation as value.

```java
package com.github.sparsick.springbootexample.hero.universum;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
public class HeroControllerRefactoringStep2 {

    private Map<String, HeroRepository> heroRepositories;

    public HeroControllerRefactoringStep2(List<HeroRepository> heroRepositories) {
        this.heroRepositories = heroRepositoryStrategies(heroRepositories);
    }

    private Map<String, HeroRepository> heroRepositoryStrategies(List<HeroRepository> heroRepositories){
        Map<String, HeroRepository> heroRepositoryStrategies = new HashMap<>();
        heroRepositories.forEach(heroRepository -> heroRepositoryStrategies.put(heroRepository.getName(), heroRepository));
        return heroRepositoryStrategies;
    }

    @PostMapping("/hero/new")
    public String addNewHero(@ModelAttribute("newHero") NewHeroModel newHeroModel) {
        HeroRepository heroRepository = findHeroRepository(newHeroModel.getRepository());
        heroRepository.addHero(newHeroModel.getHero());
        return "redirect:/hero";
    }

    private HeroRepository findHeroRepository(String repositoryName) {
        HeroRepository heroRepository = heroRepositories.get(repositoryName);
        if(heroRepository != null) {
            return  heroRepository;
        }
        throw new IllegalArgumentException(String.format("Find no repository for given repository name \[%s\]", repositoryName));
    }
}
```

The final question is what if other classes in the application need the possibility to choose a repository implementation during the runtime. I could copy and paste the private method in each class that have this need or I move the creation of the map to the Spring Context and inject the Map to each class.

```java
package com.github.sparsick.springbootexample.hero;

import com.github.sparsick.springbootexample.hero.universum.HeroRepository;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@SpringBootApplication
public class HeroApplicationRefactoringStep3 {

	public static void main(String\[\] args) {
		SpringApplication.run(HeroApplication.class, args);
	}

	@Bean
    Map<String, HeroRepository> heroRepositoryStrategy(List<HeroRepository> heroRepositories){
        Map<String, HeroRepository> heroRepositoryStrategy = new HashMap<>();
        heroRepositories.forEach(heroRepository -> heroRepositoryStrategy.put(heroRepository.getName(), heroRepository));
        return heroRepositoryStrategy;
    }
}

```

```java
package com.github.sparsick.springbootexample.hero.universum;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

import java.util.Map;

@Controller
public class HeroControllerRefactoringStep3 {

    private Map<String, HeroRepository> heroRepositoryStrategy;

    public HeroControllerRefactoringStep3(Map<String, HeroRepository> heroRepositoryStrategy) {
        this.heroRepositoryStrategy = heroRepositoryStrategy;
    }

    @PostMapping("/hero/new")
    public String addNewHero(@ModelAttribute("newHero") NewHeroModel newHeroModel) {
        HeroRepository heroRepository = findHeroRepository(newHeroModel.getRepository());
        heroRepository.addHero(newHeroModel.getHero());
        return "redirect:/hero";
    }

    private HeroRepository findHeroRepository(String repositoryName) {
        return heroRepositoryStrategy.get(repositoryName);
    }

}

```

This solution is a little bit ugly, because it isn't obvious that the Strategy Pattern is used. So the next refactoring step is moving the map of hero repositories to an own component class. Therefore, the bean definition `heroRepositoryStrategy` in the application configuration can be removed.

```java
package com.github.sparsick.springbootexample.hero.universum;

import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

@Component
public class HeroRepositoryStrategy {

    private Map<String, HeroRepository> heroRepositoryStrategies;

    public HeroRepositoryStrategy(Set<HeroRepository> heroRepositories) {
        heroRepositoryStrategies = createStrategies(heroRepositories);
    }

    HeroRepository findHeroRepository(String repositoryName) {
        return heroRepositoryStrategies.get(repositoryName);
    }

    Set<String> findAllHeroRepositoryStrategyNames () {
        return heroRepositoryStrategies.keySet();
    }

    Collection<HeroRepository> findAllHeroRepositories(){
        return heroRepositoryStrategies.values();
    }


    private Map<String, HeroRepository> createStrategies(Set<HeroRepository> heroRepositories){
        Map<String, HeroRepository> heroRepositoryStrategies = new HashMap<>();
        heroRepositories.forEach(heroRepository -> heroRepositoryStrategies.put(heroRepository.getName(), heroRepository));
        return heroRepositoryStrategies;
    }

}

```

```java
package com.github.sparsick.springbootexample.hero.universum;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

import java.net.Inet4Address;
import java.net.UnknownHostException;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

@Controller
public class HeroController {

    private HeroRepositoryStrategy heroRepositoryStrategy;

    public HeroController(HeroRepositoryStrategy heroRepositoryStrategy) {
        this.heroRepositoryStrategy = heroRepositoryStrategy;
    }

    @PostMapping("/hero/new")
    public String addNewHero(@ModelAttribute("newHero") NewHeroModel newHeroModel) {
        HeroRepository heroRepository = heroRepositoryStrategy.findHeroRepository(newHeroModel.getRepository());
        heroRepository.addHero(newHeroModel.getHero());
        return "redirect:/hero";
    }
}
```

The whole sample is hosted on [GitHub](https://github.com/sparsick/strategy-pattern-revisited-spring).
