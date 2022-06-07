---
theme: seriph
background: https://source.unsplash.com/collection/94734567/1920x1080
class: 'text-center'
highlighter: shiki
lineNumbers: false
info: |
  ## Iterate & Mediator Design Patterns
  Presentation slides By Amin Mokhtari.

drawings:
  persist: false
---

---
layout: cover
background: https://refactoring.guru/images/patterns/content/iterator/iterator-en-3x.png
---

# **ITERATOR** 
## Design Pattern

---

# What is iterator?
<br>
Iterator is a behavioral design pattern that allows sequential traversal through a complex data structure without exposing its internal details.

Most collections store their elements in simple lists. However, some of them are based on stacks, trees, graphs and other complex data structures.

But no matter how a collection is structured, it must provide some way of accessing its elements so that other code can use these elements. There should be a way to go through each element of the collection without accessing the same elements over and over.

This may sound like an easy job if you have a collection based on a list. You just loop over all of the elements. But how do you sequentially traverse elements of a complex data structure, such as a tree? For example, one day you might be just fine with depth-first traversal of a tree. Yet the next day you might require breadth-first traversal. And the next week, you might need something else, like random access to the tree elements.

---

# <mdi-emoticon-dead-outline /> Problem

Collections are one of the most used data types in programming. Nonetheless, a collection is just a container for a group of objects.

<div class="flex justify-center opacity-75 my-10">
<img src="https://refactoring.guru/images/patterns/diagrams/iterator/problem1.png" />
</div>

Most collections store their elements in simple lists. However, some of them are based on stacks, trees, graphs and other complex data structures.

This may sound like an easy job if you have a collection based on a list. You just loop over all of the elements. But how do you sequentially traverse elements of a complex data structure, such as a tree


---

# <mdi-emoticon-dead-outline /> Problem 

Adding more and more traversal algorithms to the collection gradually blurs its primary responsibility, which is efficient data storage. Additionally, some algorithms might be tailored for a specific application, so including them into a generic collection class would be weird.

<div class="flex justify-center opacity-75 my-10">
<img src="https://refactoring.guru/images/patterns/diagrams/iterator/problem2.png" />
</div>

---
layout: two-cols
---

::default::

# <mdi-emoticon-excited-outline /> Solution 
The main idea of the Iterator pattern is to extract the traversal behavior of a collection into a separate object called an iterator.
<br>

In addition to implementing the algorithm itself, an iterator object encapsulates all of the traversal details, such as the current position and how many elements are left till the end. Because of this, several iterators can go through the same collection at the same time, independently of each other.


::right::

<div class="flex justify-center opacity-75 my-10 h-[30vh]">
<img src="https://refactoring.guru/images/patterns/diagrams/iterator/solution1.png" />
</div>


---

# Real-World Analogy

<div class="flex justify-center opacity-75 my-10 ">
<img src="https://refactoring.guru/images/patterns/content/iterator/iterator-comic-1-en.png" />
</div>


---

# Structure

<div class="flex justify-center opacity-75 my-10 h-[340px]">
<img src="https://refactoring.guru/images/patterns/diagrams/iterator/structure.png" />
</div>


---
layout: image-right
image: https://source.unsplash.com/collection/94734566/1920x1080
---

# Example (Typescript)
Intent: Lets you traverse elements of a collection without exposing its underlying representation (list, stack, tree, etc.)

```ts {1-14|14-18|all}
interface Iterator<T> {
    // Return the current element.
    current(): T;
    // Return the current element
    // and move forward to next element.
    next(): T;
    // Return the key of the current element.
    key(): number;
    // Checks if current position is valid.
    valid(): boolean;
    // Rewind the Iterator to the first element.
    rewind(): void;
}

interface Aggregator {
    // Retrieve an external iterator.
    getIterator(): Iterator<string>;
}

```

---

# Example (Typescript)
Concrete Iterators implement various traversal algorithms. These classes store the current traversal position at all times.


```ts
class AlphabeticalOrderIterator implements Iterator<string> {
    private collection: WordsCollection;

    /**
     * Stores the current traversal position. An iterator may have a lot of
     * other fields for storing iteration state, especially when it is supposed
     * to work with a particular kind of collection.
     */
    private position: number = 0;

    /**
     * This variable indicates the traversal direction.
     */
    private reverse: boolean = false;

    constructor(collection: WordsCollection, reverse: boolean = false) {
        this.collection = collection;
        this.reverse = reverse;

        if (reverse) {
            this.position = collection.getCount() - 1;
        }
    }

    public rewind() {
        this.position = this.reverse ?
            this.collection.getCount() - 1 :
            0;
    }

    public current(): string {
        return this.collection.getItems()[this.position];
    }

    public key(): number {
        return this.position;
    }

    public next(): string {
        const item = this.collection.getItems()[this.position];
        this.position += this.reverse ? -1 : 1;
        return item;
    }

    public valid(): boolean {
        if (this.reverse) {
            return this.position >= 0;
        }

        return this.position < this.collection.getCount();
    }
}

```

<style>
.slidev-code {
  @apply max-h-[370px];
}
</style>

---

# Example (Typescript)
Concrete Collections provide one or several methods for retrieving fresh iterator instances, compatible with the collection class.


```ts
class WordsCollection implements Aggregator {
    private items: string[] = [];

    public getItems(): string[] {
        return this.items;
    }

    public getCount(): number {
        return this.items.length;
    }

    public addItem(item: string): void {
        this.items.push(item);
    }

    public getIterator(): Iterator<string> {
        return new AlphabeticalOrderIterator(this);
    }

    public getReverseIterator(): Iterator<string> {
        return new AlphabeticalOrderIterator(this, true);
    }
}
```

<style>
.slidev-code {
  @apply max-h-[370px];
}
</style>

---
layout: two-cols
---


::default::

# Example (Typescript)
The client code may or may not know about the Concrete Iterator or Collection classes, depending on the level of indirection you want to keep in your program.


```ts
const collection = new WordsCollection();
collection.addItem('First');
collection.addItem('Second');
collection.addItem('Third');

const iterator = collection.getIterator();

console.log('Straight traversal:');
while (iterator.valid()) {
    console.log(iterator.next());
}

console.log('');
console.log('Reverse traversal:');
const reverseIterator = collection.getReverseIterator();
while (reverseIterator.valid()) {
    console.log(reverseIterator.next());
}
```

::right::

### Execution result

```
Straight traversal:
First
Second
Third

Reverse traversal:
Third
Second
First
```

---

# Applicability

- Use the Iterator pattern when your collection has a complex data structure under the hood, but you want to hide its complexity from clients (either for convenience or security reasons).

- Use the pattern to reduce duplication of the traversal code across your app.

- Use the Iterator when you want your code to be able to traverse different data structures or when types of these structures are unknown beforehand.


---
class: flex flex-col gap-2
---

# Pros

> <mdi-check /> **Single Responsibility Principle**. You can clean up the client code and the collections by extracting bulky traversal algorithms into separate classes.


> <mdi-check /> **Open/Closed Principle**. You can implement new types of collections and iterators and pass them to existing code without breaking anything.


> <mdi-check /> You can iterate over the same collection in parallel because each iterator object contains its own iteration state.

> <mdi-check /> For the same reason, you can delay an iteration and continue it when needed.

# Cons

> <mdi-close /> Applying the pattern can be an overkill if your app only works with simple collections.

> <mdi-close /> Using an iterator may be less efficient than going through elements of some specialized collections directly.

---
layout: cover
background: https://refactoring.guru/images/patterns/content/mediator/mediator.png
---

# **MEDIATOR** 
## Design Pattern

---
layout: image-right
image: https://source.unsplash.com/collection/94734568/1920x1080
---
# What is mediator?
<br>
Mediator is a behavioral design pattern that lets you reduce chaotic dependencies between objects. The pattern restricts direct communications between the objects and forces them to collaborate only via a mediator object.

---

# <mdi-emoticon-dead-outline /> Problem

Say you have a dialog for creating and editing customer profiles. It consists of various form controls such as text fields, checkboxes, buttons, etc.

<div class="flex justify-center opacity-75 my-10 h-40">
<img src="https://refactoring.guru/images/patterns/diagrams/mediator/problem1-en.png" />
</div>

Some of the form elements may interact with others. For instance, selecting the “I have a dog” checkbox may reveal a hidden text field for entering the dog’s name. Another example is the submit button that has to validate values of all fields before saving the data.

---

# <mdi-emoticon-dead-outline /> Problem

Adding more and more traversal algorithms to the collection gradually blurs its primary responsibility, which is efficient data storage. Additionally, some algorithms might be tailored for a specific application, so including them into a generic collection class would be weird.

<div class="flex justify-center opacity-75 my-10">
<img src="https://refactoring.guru/images/patterns/diagrams/iterator/problem2.png" />
</div>

---
layout: two-cols
---

::default::

# <mdi-emoticon-excited-outline /> Solution
The Mediator pattern suggests that you should cease all direct communication between the components which you want to make independent of each other. Instead, these components must collaborate indirectly, by calling a special mediator object that redirects the calls to appropriate components. As a result, the components depend only on a single mediator class instead of being coupled to dozens of their colleagues.

The most significant change happens to the actual form elements. Let’s consider the submit button. Previously, each time a user clicked the button, it had to validate the values of all individual form elements. Now its single job is to notify the dialog about the click. Upon receiving this notification, the dialog itself performs the validations or passes the task to the individual elements. Thus, instead of being tied to a dozen form elements, the button is only dependent on the dialog class.


::right::

<div class="flex justify-center opacity-75 my-10 ">
<img src="https://refactoring.guru/images/patterns/diagrams/mediator/solution1-en.png" />
</div>

This way, the Mediator pattern lets you encapsulate a complex web of relations between various objects inside a single mediator object. The fewer dependencies a class has, the easier it becomes to modify, extend or reuse that class.
---

# Real-World Analogy

<div class="flex justify-center opacity-75 my-10 h-[70%]">
<img src="https://refactoring.guru/images/patterns/diagrams/mediator/live-example.png" />
</div>


---

# Structure

<div class="flex justify-center opacity-75 my-10 h-[340px] ">
<img src="https://refactoring.guru/images/patterns/diagrams/mediator/structure.png" />
</div>


---
layout: image-right
image: https://source.unsplash.com/collection/94734570/1920x1080
---

# Example (Typescript)
The Mediator interface declares a method used by components to notify the mediator about various events. The Mediator may react to these events and pass the execution to other components.


```ts
interface Mediator {
    notify(sender: object, event: string): void;
}

```

---

# Example (Typescript)
Concrete Mediators implement cooperative behavior by coordinating several components.


```ts
class ConcreteMediator implements Mediator {
    private component1: Component1;

    private component2: Component2;

    constructor(c1: Component1, c2: Component2) {
        this.component1 = c1;
        this.component1.setMediator(this);
        this.component2 = c2;
        this.component2.setMediator(this);
    }

    public notify(sender: object, event: string): void {
        if (event === 'A') {
            console.log('Mediator reacts on A and triggers following operations:');
            this.component2.doC();
        }

        if (event === 'D') {
            console.log('Mediator reacts on D and triggers following operations:');
            this.component1.doB();
            this.component2.doC();
        }
    }
}
```

<style>
.slidev-code {
  @apply max-h-[370px];
}
</style>

---

# Example (Typescript)

The Base Component provides the basic functionality of storing a mediator's instance inside component objects.

```ts
class BaseComponent {
    protected mediator: Mediator;

    constructor(mediator?: Mediator) {
        this.mediator = mediator!;
    }

    public setMediator(mediator: Mediator): void {
        this.mediator = mediator;
    }
}
```

---

# Example (Typescript)

Concrete Components implement various functionality. They don't depend on other components. They also don't depend on any concrete mediator classes.

```ts
class Component1 extends BaseComponent {
    public doA(): void {
        console.log('Component 1 does A.');
        this.mediator.notify(this, 'A');
    }

    public doB(): void {
        console.log('Component 1 does B.');
        this.mediator.notify(this, 'B');
    }
}

class Component2 extends BaseComponent {
    public doC(): void {
        console.log('Component 2 does C.');
        this.mediator.notify(this, 'C');
    }

    public doD(): void {
        console.log('Component 2 does D.');
        this.mediator.notify(this, 'D');
    }
}
```

<style>
.slidev-code {
  @apply max-h-[370px];
}
</style>

---
layout: two-cols
---


::default::

# Example (Typescript)

The client code.

```ts
const c1 = new Component1();
const c2 = new Component2();
const mediator = new ConcreteMediator(c1, c2);

console.log('Client triggers operation A.');
c1.doA();

console.log('');
console.log('Client triggers operation D.');
c2.doD();
```

::right::

 Execution result

```
Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.
```

---

# Applicability

- Use the Mediator pattern when it’s hard to change some of the classes because they are tightly coupled to a bunch of other classes.

- Use the pattern when you can’t reuse a component in a different program because it’s too dependent on other components.

- Use the Mediator when you find yourself creating tons of component subclasses just to reuse some basic behavior in various contexts.



---
class: flex flex-col gap-2
---

# Pros
 
 > <mdi-check /> **Single Responsibility Principle**. You can extract the communications between various components into a single place, making it easier to comprehend and maintain.
 
 > <mdi-check /> **Open/Closed Principle**. You can introduce new mediators without having to change the actual components.
 
 > <mdi-check /> You can reduce coupling between various components of a program.
 
 > <mdi-check /> You can reuse individual components more easily.
 
# Cons
 
 
 > <mdi-close /> Over time a mediator can evolve into a **God Object**.
