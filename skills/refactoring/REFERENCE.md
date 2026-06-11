# Refactoring Techniques — Reference Catalog

Mechanics for each technique. Every entry follows the same shape:

- **Problem** — the situation that calls for it.
- **Solution** — what the transformation does.
- **Why Refactor** — the motivation / benefit.
- **How To** — the ordered, behavior-preserving steps.
- **When to Ignore / Payoff** — trade-offs, when not to apply.

(Not every technique has every section. Apply in small steps and keep tests green after each one.)

---

## Composing Methods

### Extract Method
**Problem.** You have a code fragment that can be grouped together.

**Solution.** Move this code to a separate new method (or function) and replace the old code with a call to the method.

```ts
// Before
printOwing(): void {
  printBanner();

  // Print details.
  console.log("name: " + name);
  console.log("amount: " + getOutstanding());
}

// After
printOwing(): void {
  printBanner();
  printDetails(getOutstanding());
}

printDetails(outstanding: number): void {
  console.log("name: " + name);
  console.log("amount: " + outstanding);
}
```

**Why Refactor.** The more lines in a method, the harder it is to figure out what it does — this is the main reason. Beyond smoothing rough edges, extracting methods is also a step in many other refactorings.

Benefits:
- **More readable code** — give the new method a name that describes its purpose: `createOrder()`, `renderCustomerInfo()`, etc.
- **Less duplication** — code in a method can often be reused elsewhere, replacing duplicates with calls to the new method.
- **Isolates independent parts of code**, so errors are less likely (e.g. the wrong variable being modified).

**How To.**
1. Create a new method and name it so its purpose is self-evident.
2. Copy the relevant fragment into the new method. Delete it from the old location and put a call to the new method there instead.
3. Find all variables used in the fragment. If they're declared *inside* it and not used outside, leave them — they become local variables of the new method.
4. If variables are declared *before* the extracted code, pass them as parameters to the new method to preserve their values. Sometimes it's easier to eliminate them via **Replace Temp with Query**.
5. If a local variable *changes* in the extracted code, its new value may be needed later in the main method. Double-check — and if so, return that value to the main method to keep everything functioning.

### Inline Method
**Problem.** A method body is more obvious than the method itself.

**Solution.** Replace calls to the method with the method's content and delete the method.

```ts
// Before
class PizzaDelivery {
  getRating(): number {
    return moreThanFiveLateDeliveries() ? 2 : 1;
  }
  moreThanFiveLateDeliveries(): boolean {
    return numberOfLateDeliveries > 5;
  }
}

// After
class PizzaDelivery {
  getRating(): number {
    return numberOfLateDeliveries > 5 ? 2 : 1;
  }
}
```

**Why Refactor.** A method that simply delegates to another is no problem in itself, but many such methods become a confusing tangle that's hard to sort through. Methods often aren't too short originally but become that way as the program changes — so don't be shy about removing methods that have outlived their use.

Benefits:
- Minimizing the number of unneeded methods makes the code more straightforward.

**How To.**
1. Make sure the method isn't redefined in subclasses. If it is, refrain from this technique.
2. Find all calls to the method. Replace each call with the method's content.
3. Delete the method.

### Extract Variable
**Problem.** You have an expression that's hard to understand.

**Solution.** Place the result of the expression, or its parts, in separate self-explanatory variables.

```ts
// Before
renderBanner(): void {
  if ((platform.toUpperCase().indexOf("MAC") > -1) &&
       (browser.toUpperCase().indexOf("IE") > -1) &&
        wasInitialized() && resize > 0 )
  {
    // do something
  }
}

// After
renderBanner(): void {
  const isMacOs = platform.toUpperCase().indexOf("MAC") > -1;
  const isIE = browser.toUpperCase().indexOf("IE") > -1;
  const wasResized = resize > 0;

  if (isMacOs && isIE && wasInitialized() && wasResized) {
    // do something
  }
}
```

**Why Refactor.** To make a complex expression more understandable by dividing it into intermediate parts. Good candidates:
- A condition of an `if()` operator, or part of a `?:` operator in C-based languages.
- A long arithmetic expression without intermediate results.
- Long multipart lines.

Extracting a variable may be the first step toward **Extract Method**, if the extracted expression is used in other places too.

Benefits:
- **More readable code** — give extracted variables names that announce their purpose loud and clear (`customerTaxValue`, `cityUnemploymentRate`, `clientSalutationString`). More readability, fewer long-winded comments.

Drawbacks:
- More variables present in the code — counterbalanced by the ease of reading.
- **Watch short-circuit evaluation.** Compilers optimize conditionals to minimize calculations: in `if (a() || b())`, `b()` is never called if `a()` returns `true`. But if you extract both parts into variables, *both* methods always run, which may hurt performance — especially if they do heavyweight work.

**How To.**
1. Insert a new line before the relevant expression and declare a new variable there. Assign part of the complex expression to it.
2. Replace that part of the expression with the new variable.
3. Repeat for all complex parts of the expression.

### Inline Temp
**Problem.** You have a temporary variable assigned the result of a simple expression and nothing more.

**Solution.** Replace the references to the variable with the expression itself.

```ts
// Before
hasDiscount(order: Order): boolean {
  let basePrice: number = order.basePrice();
  return basePrice > 1000;
}

// After
hasDiscount(order: Order): boolean {
  return order.basePrice() > 1000;
}
```

**Why Refactor.** Inline temps are almost always used as part of **Replace Temp with Query** or to pave the way for **Extract Method**.

Benefits:
- Almost no benefit on its own. But if the variable is assigned the result of a method, you marginally improve readability by removing the unnecessary variable.

Drawbacks:
- Sometimes a seemingly useless temp caches the result of an expensive operation reused several times. Before applying this, make sure simplicity won't come at the cost of performance.

**How To.**
1. Find all places that use the variable. Use the assigned expression instead of the variable.
2. Delete the variable's declaration and assignment line.

### Replace Temp with Query
**Problem.** You place the result of an expression in a local variable for later use.

**Solution.** Move the entire expression to a separate method and return the result. Query the method instead of using a variable. Incorporate the new method in other methods if necessary.

```ts
// Before
calculateTotal(): number {
  let basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  } else {
    return basePrice * 0.98;
  }
}

// After
calculateTotal(): number {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  } else {
    return basePrice() * 0.98;
  }
}
basePrice(): number {
  return quantity * itemPrice;
}
```

**Why Refactor.** Can lay the groundwork for applying **Extract Method** to a portion of a very long method. The same expression may also appear in other methods — a reason to create a common method.

Benefits:
- **Code readability** — it's much easier to understand `getTax()` than the line `orderPrice() * 0.2`.
- **Slimmer code via deduplication**, if the replaced line is used in multiple methods.

**Performance.** This may add a slight cost from querying a new method, but with fast CPUs and good compilers the burden is almost always minimal — and readability plus reuse are very noticeable benefits. Still, if your temp caches the result of a truly time-consuming expression, you may want to stop after extracting the expression to a new method.

**How To.**
1. Make sure a value is assigned to the variable once and only once within the method. If not, use **Split Temporary Variable** so the variable stores only the result of your expression.
2. Use **Extract Method** to place the expression in a new method. Make sure it only returns a value and doesn't change the object's state. If it affects the object's visible state, use **Separate Query from Modifier**.
3. Replace the variable with a query to your new method.

### Split Temporary Variable
**Problem.** A local variable is used to store various intermediate values inside a method (except loop variables).

**Solution.** Use different variables for different values. Each variable should be responsible for only one particular thing.

```ts
// Before
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);

// After
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

**Why Refactor.** If you skimp on the number of variables and reuse them for unrelated purposes, you'll hit problems as soon as you change the code — having to recheck each use to make sure the correct values are used.

Benefits:
- Each component should be responsible for one thing only, making code easier to maintain — you can replace any particular thing without fear of unintended effects.
- **More readable code** — a variable created in a rush often has a meaningless name (`k`, `a2`, `value`); the new variables can be named self-explanatorily (`customerTaxValue`, `cityUnemploymentRate`).
- Useful if you anticipate using **Extract Method** later.

**How To.**
1. Find the first place the variable is given a value. Rename the variable with a name corresponding to the value being assigned.
2. Use the new name wherever this value is used.
3. Repeat for places where the variable is assigned a different value.

### Remove Assignments to Parameters
**Problem.** Some value is assigned to a parameter inside a method's body.

**Solution.** Use a local variable instead of a parameter.

```ts
// Before
discount(inputVal: number, quantity: number): number {
  if (quantity > 50) {
    inputVal -= 2;
  }
  // ...
}

// After
discount(inputVal: number, quantity: number): number {
  let result = inputVal;
  if (quantity > 50) {
    result -= 2;
  }
  // ...
}
```

**Why Refactor.** Same reasons as **Split Temporary Variable**, but for a parameter rather than a local variable.
- If a parameter is passed by reference, changing it inside the method passes the changed value back to the caller's argument. This often happens accidentally and causes unfortunate effects. Even where parameters are passed by value, this quirk may confuse those unaccustomed to it.
- Multiple assignments of different values to a single parameter make it hard to know what data the parameter should hold at any point — worse if the parameter is documented but its actual value differs from what's expected inside the method.

Benefits:
- Each element should be responsible for only one thing, making maintenance easier — you can replace code without side effects.
- Helps extract repetitive code into separate methods (**Extract Method**).

**How To.**
1. Create a local variable and assign the initial value of the parameter to it.
2. In all method code that follows, replace the parameter with the new local variable.

### Replace Method with Method Object
**Problem.** You have a long method whose local variables are so intertwined that you can't apply **Extract Method**.

**Solution.** Transform the method into a separate class so the local variables become fields. Then you can split the method into several methods within the same class.

```ts
// Before
class Order {
  price(): number {
    let primaryBasePrice;
    let secondaryBasePrice;
    let tertiaryBasePrice;
    // Perform long computation.
  }
}

// After
class Order {
  price(): number {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  private _primaryBasePrice: number;
  private _secondaryBasePrice: number;
  private _tertiaryBasePrice: number;

  constructor(order: Order) {
    // Copy relevant information from the order object.
  }

  compute(): number {
    // Perform long computation.
  }
}
```

**Why Refactor.** A method is too long and you can't separate it due to tangled local variables hard to isolate. Isolating the whole method into a class and turning its locals into fields (1) isolates the problem at the class level, and (2) paves the way for splitting a large, unwieldy method into smaller ones that wouldn't have fit the original class anyway.

Benefits:
- Stops a method from ballooning; allows splitting into submethods within the new class without polluting the original class with utility methods.

Drawbacks:
- Another class is added, increasing overall complexity.

**How To.**
1. Create a new class. Name it based on the purpose of the method you're refactoring.
2. In the new class, create a private field storing a reference to an instance of the original class (to fetch required data if needed).
3. Create a separate private field for each local variable of the method.
4. Create a constructor that accepts the values of all local variables and initializes the corresponding fields.
5. Declare the main method and copy the original method's code into it, replacing local variables with the private fields.
6. Replace the body of the original method by creating a method object and calling its main method.

### Substitute Algorithm
**Problem.** You want to replace an existing algorithm with a new one.

**Solution.** Replace the body of the method that implements the algorithm with a new algorithm.

```ts
// Before
foundPerson(people: string[]): string {
  for (let person of people) {
    if (person.equals("Don")) { return "Don"; }
    if (person.equals("John")) { return "John"; }
    if (person.equals("Kent")) { return "Kent"; }
  }
  return "";
}

// After
foundPerson(people: string[]): string {
  let candidates = ["Don", "John", "Kent"];
  for (let person of people) {
    if (candidates.includes(person)) {
      return person;
    }
  }
  return "";
}
```

**Why Refactor.**
1. Gradual refactoring isn't the only way to improve a program. Sometimes a method is so cluttered that it's easier to tear it down and start fresh — especially if you've found a much simpler, more efficient algorithm.
2. Your algorithm may now exist in a well-known library or framework, and you want to drop your independent implementation to simplify maintenance.
3. Requirements may have changed so heavily that the existing algorithm can't be salvaged.

**How To.**
1. Make sure you've simplified the existing algorithm as much as possible. Move unimportant code to other methods with **Extract Method** — fewer moving parts make it easier to replace.
2. Create your new algorithm in a new method. Replace the old algorithm with the new one and start testing.
3. If the results don't match, return to the old implementation and compare. Identify the cause of the discrepancy — often an error in the old algorithm, but more likely something not working in the new one.
4. When all tests pass, delete the old algorithm for good.

---

## Moving Features between Objects

### Move Method
**Problem.** A method is used more in another class than in its own class.

**Solution.** Create a new method in the class that uses the method the most, then move code from the old method to it. Turn the original method into a reference to the new method in the other class, or remove it entirely.

**Why Refactor.**
- Move a method to the class that holds most of the data it uses — this makes classes more internally coherent.
- Move a method to reduce or eliminate the calling class's dependency on the class where the method lives — useful when the calling class is already dependent on the target class. This reduces inter-class dependency.

**How To.**
1. Verify all features used by the old method in its class. Consider moving them too: if a feature is used *only* by this method, move it as well; if used by other methods too, move those methods as well. Sometimes it's much easier to move many methods than to set up relationships between them across classes.
2. Make sure the method isn't declared in superclasses or subclasses. If it is, either refrain from moving, or implement polymorphism in the recipient class to provide the varying functionality.
3. Declare the new method in the recipient class. You may want to give it a more appropriate name for its new home.
4. Decide how to refer to the recipient class. You may already have a field or method returning an appropriate object; if not, write a new method or field to store the recipient object.
5. With a way to refer to the recipient object and a new method in its class, turn the old method into a reference to the new method.
6. Check whether you can delete the old method entirely. If so, place a reference to the new method everywhere the old one was used.

### Move Field
**Problem.** A field is used more in another class than in its own class.

**Solution.** Create a field in a new class and redirect all users of the old field to it.

**Why Refactor.** Fields are often moved as part of **Extract Class**. Deciding which class to leave a field in can be tough — rule of thumb: **put a field in the same place as the methods that use it** (or where most of them are). This also helps when a field is simply located in the wrong place.

**How To.**
1. If the field is public, refactoring is much easier if you make it private and provide public access methods first (use **Encapsulate Field**).
2. Create the same field with access methods in the recipient class.
3. Decide how to refer to the recipient class. You may already have a field or method returning the appropriate object; if not, write a new method or field to store the recipient object.
4. Replace all references to the old field with appropriate calls to methods in the recipient class. If the field isn't private, take care of this in the superclass and subclasses.
5. Delete the field in the original class.

### Extract Class
**Problem.** When one class does the work of two, awkwardness results.

**Solution.** Create a new class and place the fields and methods responsible for the relevant functionality in it.

**Why Refactor.** Classes start out clear and focused, but as the program expands a method is added, then a field, and eventually some classes carry far more responsibility than ever envisioned.

Benefits:
- Helps maintain the **Single Responsibility Principle** — your classes become more obvious and understandable.
- Single-responsibility classes are more reliable and tolerant of changes. If a class is responsible for ten things, changing it for one risks breaking the other nine.

Drawbacks:
- If you overdo this technique, you'll have to resort to **Inline Class**.

**How To.**
1. Before starting, decide exactly how to split up the class's responsibilities.
2. Create a new class to contain the relevant functionality.
3. Create a relationship between the old and new class. Optimally unidirectional — this allows reusing the second class without issues. A two-way relationship can be set up if you think it's necessary.
4. Use **Move Field** and **Move Method** for each field and method you're moving. Start with private methods to reduce the risk of errors. Relocate a little at a time and test after each move to avoid a pileup of error-fixing at the end.
5. When done, review the resulting classes. The old class with changed responsibilities may be renamed for clarity. Check again whether you can get rid of any two-way relationships.
6. Consider accessibility to the new class from outside: hide it from the client entirely (make it private, managed via the old class's fields), or make it public (letting the client change values directly). The choice depends on how safe the old class's behavior is when unexpected direct changes are made to the new class.

### Inline Class
**Problem.** A class does almost nothing, isn't responsible for anything, and no additional responsibilities are planned for it.

**Solution.** Move all features from the class to another one.

**Why Refactor.** Often needed after the features of one class are "transplanted" to other classes, leaving that class with little to do.

Benefits:
- Eliminating needless classes frees up operating memory on the computer — and bandwidth in your head.

**How To.**
1. In the recipient class, create the public fields and methods present in the donor class. Methods should refer to the equivalent methods of the donor class.
2. Replace all references to the donor class with references to the fields and methods of the recipient class.
3. Test the program to make sure no errors were added. If everything works, use **Move Method** and **Move Field** to completely transplant all functionality from the original class to the recipient. Continue until the original class is completely empty.
4. Delete the original class.

### Hide Delegate
**Problem.** The client gets object B from a field or method of object A, then calls a method of object B.

**Solution.** Create a new method in class A that delegates the call to object B. Now the client doesn't know about, or depend on, class B.

**Terminology.** *Server* is the object the client has direct access to. *Delegate* is the end object containing the functionality the client needs.

**Why Refactor.** A call chain appears when a client requests an object from another object, which requests another, and so on — involving the client in navigation along the class structure. Any change in these interrelationships then requires changes on the client side.

Benefits:
- Hides delegation from the client. The less the client needs to know about relationships between objects, the easier it is to change the program.

Drawbacks:
- If you create an excessive number of delegating methods, the server class risks becoming an unneeded go-between — an excess of **Middle Man**.

**How To.**
1. For each method of the delegate class called by the client, create a method in the server class that delegates the call to the delegate class.
2. Change the client code to call the methods of the server class.
3. If your changes free the client from needing the delegate class, remove the access method to the delegate class from the server class (the method originally used to get the delegate).

### Remove Middle Man
**Problem.** A class has too many methods that simply delegate to other objects.

**Solution.** Delete these methods and force the client to call the end methods directly.

**Terminology** (from **Hide Delegate**). *Server* is the object the client has direct access to. *Delegate* is the end object containing the functionality the client needs.

**Why Refactor.** Two types of problem:
- The server class doesn't do anything itself and simply creates needless complexity — consider whether the class is needed at all.
- Every time a new feature is added to the delegate, you must create a delegating method for it in the server class. With many changes, this is tiresome.

**How To.**
1. Create a getter for accessing the delegate-class object from the server-class object.
2. Replace calls to delegating methods in the server class with direct calls to methods in the delegate class.

### Introduce Foreign Method
**Problem.** A utility class doesn't contain the method you need, and you can't add the method to the class.

**Solution.** Add the method to a client class and pass an object of the utility class to it as an argument.

```ts
// Before
class Report {
  sendReport(): void {
    let nextDay: Date = new Date(previousEnd.getYear(),
      previousEnd.getMonth(), previousEnd.getDate() + 1);
    // ...
  }
}

// After
class Report {
  sendReport() {
    let newStart: Date = nextDay(previousEnd);
    // ...
  }
  private static nextDay(arg: Date): Date {
    return new Date(arg.getFullYear(), arg.getMonth(), arg.getDate() + 1);
  }
}
```

**Why Refactor.** You have code that uses the data and methods of a class and realize it would work much better inside a new method *in that class* — but you can't add the method (e.g. the class lives in a third-party library). Big payoff when the code you want to move is repeated in several places. Since you pass an object of the utility class as a parameter, you have access to all its fields and can do practically anything inside the method, as if it were part of the utility class.

Benefits:
- **Removes code duplication** — replace repeated fragments with a method call, better than duplication even though the foreign method is in a suboptimal place.

Drawbacks:
- The reason for a utility-class method living in a client class won't always be clear to future maintainers. If the method could be used in other classes, consider a wrapper for the utility class instead — **Introduce Local Extension** helps, especially with several such utility methods.

**How To.**
1. Create a new method in the client class.
2. Create a parameter to which the utility-class object will be passed. If the object can be obtained from the client class, you don't need such a parameter.
3. Extract the relevant code fragments into this method and replace them with method calls.
4. Leave a *Foreign method* tag in the comments, with advice to move this method into the utility class if that ever becomes possible. This helps future maintainers understand why the method lives where it does.

### Introduce Local Extension
_(pending)_

---

## Organizing Data

### Change Value to Reference
**Problem.** You have many identical instances of a single class that you need to replace with a single object.

**Solution.** Convert the identical objects to a single reference object.

**Why Refactor.** Objects can be classified as values or references:
- **References:** one real-world object corresponds to only one object in the program (user/order/product objects).
- **Values:** one real-world object corresponds to multiple objects in the program (dates, phone numbers, addresses, colors).

The choice isn't always clear-cut. Sometimes a simple value with a small amount of unchanging data later needs changeable data passed on every access — at which point it must become a reference.

Benefits:
- An object holds all the most current information about an entity. Change it in one part of the program and the change is visible from every other part that uses it.

Drawbacks:
- References are much harder to implement.

**How To.**
1. Use **Replace Constructor with Factory Method** on the class from which references are generated.
2. Determine which object is responsible for providing access to references. Instead of creating a new object, fetch it from a storage object or static dictionary field.
3. Decide whether references are created in advance or dynamically. If in advance, make sure to load them before use.
4. Change the factory method to return a reference. If objects are created in advance, decide how to handle requests for non-existent objects. You may use **Rename Method** to signal that the method returns only existing objects.

### Change Reference to Value
**Problem.** You have a reference object that's too small and infrequently changed to justify managing its life cycle.

**Solution.** Turn it into a value object.

**Why Refactor.** The inconvenience of working with references can prompt the switch. References require management:
- They always require requesting the object from storage.
- References in memory may be inconvenient to work with.
- They're particularly difficult, compared to values, on distributed and parallel systems.

Values are especially useful if you prefer unchangeable objects over objects whose state may change during their lifetime.

Benefits:
- An important property of objects is that they should be **unchangeable** — each query returning an object value should give the same result. If so, no problems arise from many objects representing the same thing.
- Values are much easier to implement.

Drawbacks:
- If a value is changeable, you must ensure that when any object changes, the values in all other objects representing the same entity are updated. This is so burdensome it's easier to create a reference instead.

**How To.**
1. Make the object unchangeable — no setters or other state-changing methods (**Remove Setting Method** may help). The only place data is assigned to a value object's fields is the constructor.
2. Create a comparison method so two values can be compared.

### Duplicate Observed Data
**Problem.** Domain data is stored in classes responsible for the GUI.

**Solution.** Separate the data into separate classes, ensuring connection and synchronization between the domain class and the GUI.

**Why Refactor.** You want multiple interface views for the same data (e.g. both a desktop app and a mobile app). If you fail to separate the GUI from the domain, you'll have a very hard time avoiding code duplication and a large number of mistakes.

Benefits:
- Splits responsibility between business-logic and presentation classes (**Single Responsibility Principle**), making the program more readable.
- To add a new interface view, create new presentation classes without touching the business logic (**Open/Closed Principle**).
- Different people can work on the business logic and the user interfaces.

**When Not to Use.** In its classic form (using the **Observer** pattern), this isn't applicable to web apps where all classes are recreated between queries to the server. The general principle of extracting business logic into separate classes still applies to web apps, but via different techniques depending on the system's design.

**How To.**
1. Hide direct access to domain data in the GUI class — best done with **Self Encapsulate Field** (create getters/setters).
2. In GUI event handlers, use setters to set new field values, so they can be passed to the associated domain object.
3. Create a domain class and copy the necessary fields from the GUI class into it. Create getters and setters for all of them.
4. Create an Observer pattern for these two classes:
   - In the domain class, create an array for storing observer objects (GUI objects), plus methods for registering, deleting, and notifying them.
   - In the GUI class, create a field for the domain-class reference and an `update()` method reacting to changes and updating the GUI fields. Set value updates directly in the method to avoid recursion.
5. In the GUI constructor, create a domain-class instance, save it in the field, and register the GUI object as an observer.
6. In the domain-class setters, call the observer-notification method (the GUI's update) to pass new values to the GUI.
7. Change the GUI-class setters to set new values in the domain object directly. Make sure values aren't set through a domain-class setter — otherwise infinite recursion results.

### Self Encapsulate Field
> Distinct from ordinary **Encapsulate Field**: this technique is performed on a *private* field.

**Problem.** You use direct access to private fields inside a class.

**Solution.** Create a getter and setter for the field, and use only them for accessing the field.

```ts
// Before
class Range {
  private low: number;
  private high: number;
  includes(arg: number): boolean {
    return arg >= low && arg <= high;
  }
}

// After
class Range {
  private low: number;
  private high: number;
  includes(arg: number): boolean {
    return arg >= getLow() && arg <= getHigh();
  }
  getLow(): number { return low; }
  getHigh(): number { return high; }
}
```

**Why Refactor.** Sometimes directly accessing a private field isn't flexible enough. You may want to initialize a field on first query, perform operations on new values when assigned, or do all this differently in subclasses.

Benefits:
- **Indirect access via getters/setters is much more flexible than direct access.**
  - You can perform complex operations when data is set or received — lazy initialization and validation are easily implemented inside getters/setters.
  - More crucially, you can redefine getters and setters in subclasses.
- You can choose *not* to implement a setter at all — the value is set only in the constructor, making the field unchangeable throughout the object's lifespan.

Drawbacks:
- With direct field access, code looks simpler and more presentable, though flexibility is diminished.

**How To.**
1. Create a getter (and optional setter) for the field. They should be `protected` or `public`.
2. Find all direct invocations of the field and replace them with getter and setter calls.

### Replace Data Value with Object
**Problem.** A class (or group of classes) contains a data field. The field has its own behavior and associated data.

**Solution.** Create a new class, place the old field and its behavior in it, and store an object of the class in the original class.

**Why Refactor.** Basically a special case of **Extract Class** — the difference is the cause. In Extract Class, one class is responsible for different things and we split its responsibilities. Here, a primitive field (number, string, etc.) is no longer simple due to program growth and now has associated data and behaviors. Nothing scary in itself, but this fields-and-behaviors family can appear in several classes at once, creating duplicate code. So we create a new class and move the field plus its related data and behaviors to it.

Benefits:
- Improves relatedness inside classes — data and the relevant behaviors live in a single class.

**How To.**
1. Before starting, check for direct references to the field from within the class. If so, use **Self Encapsulate Field** to hide it in the original class.
2. Create a new class and copy the field and its relevant getter to it. Add a constructor that accepts the simple value. This class won't have a setter — each new field value sent to the original class creates a new value object.
3. In the original class, change the field type to the new class.
4. In the original class's getter, invoke the getter of the associated object.
5. In the setter, create a new value object. You may also need to create a new object in the constructor if initial values were set there previously.

**Next Steps.** Afterward it's wise to apply **Change Value to Reference** on the field holding the object — this stores a reference to a single object per value instead of dozens of objects for the same value. Most useful when one object should represent one real-world object (users, orders, documents); not useful for dates, money, ranges, etc.

### Replace Array with Object
> A special case of **Replace Data Value with Object**.

**Problem.** You have an array that contains various types of data.

**Solution.** Replace the array with an object that has separate fields for each element.

```ts
// Before
let row = new Array(2);
row[0] = "Liverpool";
row[1] = "15";

// After
let row = new Performance();
row.setName("Liverpool");
row.setWins("15");
```

**Why Refactor.** Arrays are excellent for storing data and collections of a *single* type. But using an array like post office boxes — username in box 1, address in box 14 — leads to catastrophic failures when someone puts something in the wrong box, and wastes your time figuring out which data lives where.

Benefits:
- In the resulting class you can place all associated behaviors previously stored in the main class or elsewhere.
- A class's fields are much easier to document than array elements.

**How To.**
1. Create the new class to contain the array data. Place the array itself in the class as a public field.
2. Create a field in the original class to store the object of this class. Also create the object itself where you initialized the data array.
3. In the new class, create access methods one by one for each array element, with self-explanatory names. Simultaneously replace each use of an array element in the main code with the corresponding access method.
4. When access methods exist for all elements, make the array private.
5. For each array element, create a private field in the class and change the access methods to use this field instead of the array.
6. When all data has been moved, delete the array.

### Change Unidirectional Association to Bidirectional
**Problem.** You have two classes that each need to use the features of the other, but the association between them is only unidirectional.

**Solution.** Add the missing association to the class that needs it.

**Why Refactor.** Originally the classes had a unidirectional association, but over time client code needed access to both sides.

Benefits:
- If a class needs a reverse association, you can simply calculate it — but if those calculations are complex, it's better to keep the reverse association.

Drawbacks:
- Bidirectional associations are much harder to implement and maintain than unidirectional ones.
- They make classes interdependent. With a unidirectional association, one class can be used independently of the other.

**How To.**
1. Add a field for holding the reverse association.
2. Decide which class is "dominant". This class contains the methods that create or update the association as elements are added or changed — establishing the association in its own class and calling utility methods to establish it in the associated object.
3. Create a utility method for establishing the association in the "non-dominant" class. It should use its parameters to complete the field. Give it an obvious name so it isn't reused for other purposes.
4. If old methods for controlling the unidirectional association were in the "dominant" class, complement them with calls to the associated object's utility methods.
5. If the old control methods were in the "non-dominant" class, create the methods in the "dominant" class, call them, and delegate execution to them.

### Change Bidirectional Association to Unidirectional
**Problem.** You have a bidirectional association between classes, but one of them doesn't use the other's features.

**Solution.** Remove the unused association.

**Why Refactor.**
- A bidirectional association is harder to maintain than a unidirectional one, requiring extra code for properly creating and deleting the relevant objects — making the program more complicated.
- An improperly implemented bidirectional association can cause garbage-collection problems (memory bloat from unused objects). Example: a `User`-`Order` pair is created, used, then abandoned, but the objects aren't cleared because they still refer to each other. (This matters less now thanks to languages that automatically identify and remove unused references.)
- There's also interdependency: in a bidirectional association both classes must know about each other and can't be used separately. With many such associations, parts of the program become too dependent and any change may ripple outward.

Benefits:
- Simplifies the class that doesn't need the relationship — less code to maintain.
- Reduces dependency between classes. Independent classes are easier to maintain since changes affect only that class.

**How To.**
1. Make sure one of the following holds: no association is used; there's another way to get the associated object (e.g. a database query); or the associated object can be passed as an argument to the methods that use it.
2. Depending on your situation, replace use of the field that holds the association with a parameter or a method call that gets the object another way.
3. Delete the code that assigns the associated object to the field.
4. Delete the now-unused field.

### Encapsulate Field
**Problem.** You have a public field.

**Solution.** Make the field private and create access methods for it.

```ts
// Before
class Person {
  name: string;
}

// After
class Person {
  private _name: string;

  get name() {
    return this._name;
  }
  setName(arg: string): void {
    this._name = arg;
  }
}
```

**Why Refactor.** A pillar of OOP is **encapsulation** — concealing object data. Otherwise all objects would be public and other objects could get and modify your data without checks and balances; data is separated from its associated behaviors, modularity is compromised, and maintenance becomes complicated.

Benefits:
- If data and behavior are closely interrelated and in the same place, the component is much easier to maintain and develop.
- You can perform complicated operations related to field access.

**When Not to Use.**
- In rare cases, encapsulation is ill-advised for performance reasons. Example: a graphical editor with objects possessing x/y coordinates unlikely to change, present in a great many objects — accessing the coordinate fields directly saves significant CPU cycles otherwise spent calling access methods. (Java's `Point` class is such a case: all fields are public.)

**How To.**
1. Create a getter and setter for the field.
2. Find all invocations of the field. Replace reads with the getter and writes with the setter.
3. After all field invocations are replaced, make the field private.

**Next Steps.** This is only the first step in bringing data and its behaviors together. After creating simple access methods, recheck where they're called — the code there might look more appropriate inside the access methods themselves.

### Encapsulate Collection
**Problem.** A class contains a collection field and a simple getter and setter for working with the collection.

**Solution.** Make the getter-returned value read-only and create methods for adding/deleting elements of the collection.

**Why Refactor.** Collections should use a slightly different protocol than other data types. The getter shouldn't return the collection object itself — that would let clients change its contents without the owner class's knowledge, and expose too much of the object's internal structure. The getter should return a value that allows neither changing the collection nor disclosing excessive structural data.

There also shouldn't be a method that assigns a value to the whole collection. Instead, provide operations for adding and deleting elements, so the owner object controls these operations. This properly encapsulates the collection, reducing the association between the owner class and client code.

Benefits:
- The collection field is encapsulated inside the class — the getter returns a *copy*, preventing accidental changing or overwriting without the owner class's knowledge.
- If elements are stored in a primitive type (e.g. array), you create more convenient methods for working with the collection.
- If elements are in a non-primitive container (standard collection class), encapsulation lets you restrict access to unwanted standard methods (e.g. restricting addition of new elements).

**How To.**
1. Create methods for adding and deleting collection elements. They must accept collection elements in their parameters.
2. Assign an empty collection to the field as the initial value, if not already done in the constructor.
3. Find calls of the collection-field setter. Change the setter to use the add/delete operations, or make those operations call client code. Since a setter can only replace *all* elements, consider renaming it (**Rename Method**) to `replace`.
4. Find all calls of the collection getter after which the collection is changed. Change the code to use your new add/delete methods.
5. Change the getter so it returns a read-only representation of the collection.
6. Inspect the client code using the collection for code that would look better inside the collection class itself.

### Replace Magic Number with Symbolic Constant
**Problem.** Your code uses a number that has a certain meaning to it.

**Solution.** Replace the number with a constant that has a human-readable name explaining the meaning.

```ts
// Before
potentialEnergy(mass: number, height: number): number {
  return mass * height * 9.81;
}

// After
static const GRAVITATIONAL_CONSTANT = 9.81;

potentialEnergy(mass: number, height: number): number {
  return mass * height * GRAVITATIONAL_CONSTANT;
}
```

**Why Refactor.** A magic number is a numeric value in the source with no obvious meaning. This anti-pattern makes the program harder to understand and refactor. Changing it is also hard: find-and-replace won't work, because the same number may be used for different purposes in different places — so you'd have to verify every line that uses it.

Benefits:
- The symbolic constant serves as live documentation of its value's meaning.
- Much easier to change a constant's value than to search for the number throughout the codebase, with no risk of accidentally changing the same number used elsewhere for a different purpose.
- Reduces duplicate use of a number or string, especially when the value is complicated and long (`3.14159`, `0xCAFEBABE`).

**Good to Know — not all numbers are magical.** If the purpose is obvious, there's no need to replace it. Classic example: `for (i = 0; i < count; i++) { ... }`.

**Alternatives.**
1. Sometimes a magic number can be replaced with a method call. If a magic number signifies the number of elements in a collection, use the standard length method instead of hardcoding it.
2. Magic numbers are sometimes used as type code (e.g. administrators are `1`, ordinary users are `2`). In this case use one of: **Replace Type Code with Class**, **Replace Type Code with Subclasses**, or **Replace Type Code with State/Strategy**.

**How To.**
1. Declare a constant and assign the value of the magic number to it.
2. Find all mentions of the magic number.
3. For each, double-check that the number in that case corresponds to the constant's purpose. If yes, replace it. This is important — the same number can mean entirely different things (and may need different constants).

### Replace Type Code with Class
> **What's type code?** Type code occurs when, instead of a separate data type, you have a set of numbers or strings forming a list of allowable values for some entity. These are often given understandable names via constants — which is why type code is so common.

**Problem.** A class has a field that contains type code. The values of this type aren't used in operator conditions and don't affect the program's behavior.

**Solution.** Create a new class and use its objects instead of the type code values.

**Why Refactor.** A common reason for type code is working with databases, where a field codes a complex concept with a number or string. E.g. class `User` has field `user_role` coding access privileges as `A` (administrator), `E` (editor), `U` (ordinary user). Shortcomings:
- Field setters often don't check which value is sent, causing big problems when someone sends unintended/wrong values.
- Type verification is impossible — any number or string can be sent, not type-checked by your IDE, allowing the program to run and crash later.

Benefits:
- Turns sets of primitive values (coded types) into full-fledged classes with all the benefits of OOP.
- Enables type hinting at the language level — the compiler now warns inside your IDE when data not fitting the type class is passed, instead of treating your numeric constant like any arbitrary number.
- Makes it possible to move code to the type classes. Complex manipulations with type values across the program can now "live" inside one or more type classes.

**When Not to Use.** If the values of a coded type are used inside control-flow structures (`if`, `switch`, etc.) and control class behavior, use **Replace Type Code with Subclasses** or **Replace Type Code with State/Strategy** instead.

**How To.**
1. Create a new class with a name corresponding to the purpose of the coded type (call it the *type class*).
2. Copy the field containing type code into the type class and make it private. Create a getter; the field's value is set only from the constructor.
3. For each value of the coded type, create a static method in the type class that creates a new type-class object corresponding to that value.
4. In the original class, replace the type of the coded field with the type class. Create a new object of this type in the constructor and in the field setter. Change the field getter to call the type class's getter.
5. Replace any mentions of coded-type values with calls to the relevant type-class static methods.
6. Remove the coded-type constants from the original class.

### Replace Type Code with Subclasses
> **What's type code?** A set of numbers or strings (often named via constants) forming a list of allowable values for some entity, used in place of a separate data type.

**Problem.** You have a coded type that directly affects program behavior (values of this field trigger various code in conditionals).

**Solution.** Create subclasses for each value of the coded type. Extract the relevant behaviors from the original class into these subclasses. Replace the control-flow code with polymorphism.

**Why Refactor.** A more complicated twist on **Replace Type Code with Class**. As there, you have simple values constituting all allowed values for a field. Even named as constants, their use is error-prone since they're still primitives — e.g. a method expecting `USER_TYPE_ADMIN` (`"ADMIN"`) receives `"admin"` instead, executing something unintended. Here we're dealing with control-flow code (`if`, `switch`, `?:`) whose conditions use coded fields (`$user->type === self::USER_TYPE_ADMIN`). Using Replace Type Code with Class here would move all this control flow into the type class, recreating a class very like the original with the same problems.

Benefits:
- **Deletes the control-flow code** — instead of a bulky `switch`, move code to appropriate subclasses. Improves the **Single Responsibility Principle** and overall readability.
- To add a new value, just add a new subclass without touching existing code (**Open/Closed Principle**).
- Paves the way for type hinting at the language level, impossible with simple numeric/string values.

**When Not to Use.**
- If you already have a class hierarchy — you can't create a dual hierarchy via inheritance. Use **Replace Type Code with State/Strategy** (composition) instead.
- If type-code values can change after an object is created — you'd have to replace the object's class on the fly, which isn't possible. Again use **Replace Type Code with State/Strategy**.

**How To.**
1. Use **Self Encapsulate Field** to create a getter for the type-code field.
2. Make the superclass constructor private. Create a static factory method with the same parameters, including the one taking the starting coded-type value. Depending on this parameter, the factory creates objects of various subclasses via a large conditional — at least it'll be the only one truly necessary; subclasses and polymorphism handle the rest.
3. Create a unique subclass for each value of the coded type. In it, redefine the getter to return the corresponding value.
4. Delete the type-code field from the superclass. Make its getter abstract.
5. With subclasses in place, move fields and methods from the superclass to the corresponding subclasses (**Push Down Field**, **Push Down Method**).
6. When everything possible has been moved, use **Replace Conditional with Polymorphism** to eliminate the conditionals using type code once and for all.

### Replace Type Code with State/Strategy
> **What's type code?** A set of numbers or strings (often named via constants) forming a list of allowable values for some entity, used in place of a separate data type.

**Problem.** You have a coded type that affects behavior but you can't use subclasses to get rid of it.

**Solution.** Replace type code with a state object. If a field value with type code must be replaced, another state object is "plugged in".

**Why Refactor.** Type code affects the behavior of a class, so we can't use **Replace Type Code with Class**. But we also can't create subclasses for the coded type — due to an existing class hierarchy or other reasons — so **Replace Type Code with Subclasses** doesn't apply either.

Benefits:
- A way out when a coded-type field **changes its value during the object's lifetime** — replacement is made by swapping the state object the original class refers to.
- To add a new value, just add a new state subclass without altering existing code (**Open/Closed Principle**).

Drawbacks:
- For a simple case of type code, using this technique leaves you with many extra, unneeded classes.

**Good to Know — State or Strategy?** Implementation is the same either way.
- Use **Strategy** if you're splitting a conditional that selects an algorithm.
- Use **State** if each coded-type value is responsible not just for algorithm selection but for the whole condition of the class — class state, field values, and many other actions.

**How To.**
1. Use **Self Encapsulate Field** to create a getter for the type-code field.
2. Create a new class with an understandable name fitting the type code's purpose — it plays the role of state (or strategy). In it, create an abstract coded-field getter.
3. Create subclasses of the state class for each coded value. In each, redefine the getter to return the corresponding value.
4. In the abstract state class, create a static factory method accepting the coded-type value. Depending on it, the factory creates the various state objects via a large conditional — the only one when refactoring is complete.
5. In the original class, change the coded field's type to the state class. In its setter, call the factory method to get new state objects.
6. Move fields and methods from the superclass to the corresponding state subclasses (**Push Down Field**, **Push Down Method**).
7. When everything moveable has been moved, use **Replace Conditional with Polymorphism** to eliminate the type-code conditionals once and for all.

### Replace Subclass with Fields
**Problem.** You have subclasses differing only in their (constant-returning) methods.

**Solution.** Replace the methods with fields in the parent class and delete the subclasses.

**Why Refactor.** Sometimes this is just the ticket for avoiding type code. A hierarchy of subclasses may differ only in the values returned by particular methods — methods that aren't even computations, but strictly set out in the methods themselves or in fields they return. To simplify the architecture, the hierarchy can be compressed into a single class containing one or several fields with the necessary values. This often becomes necessary after moving a large amount of functionality out of a hierarchy elsewhere — the hierarchy is no longer valuable and its subclasses are now dead weight.

Benefits:
- Simplifies system architecture. Creating subclasses is overkill if all you want is to return different values in different methods.

**How To.**
1. Apply **Replace Constructor with Factory Method** to the subclasses.
2. Replace subclass constructor calls with superclass factory method calls.
3. In the superclass, declare fields for storing the values of each subclass method that returns a constant.
4. Create a protected superclass constructor for initializing the new fields.
5. Create or modify the subclass constructors to call the new parent constructor, passing the relevant values.
6. Implement each constant method in the parent class to return the value of the corresponding field. Then remove the method from the subclass.
7. If the subclass constructor has additional functionality, use **Inline Method** to incorporate it into the superclass factory method.
8. Delete the subclass.

---

## Simplifying Conditional Expressions

### Consolidate Conditional Expression
**Problem.** You have multiple conditionals that lead to the same result or action.

**Solution.** Consolidate all these conditionals into a single expression.

```ts
// Before
disabilityAmount(): number {
  if (seniority < 2) { return 0; }
  if (monthsDisabled > 12) { return 0; }
  if (isPartTime) { return 0; }
  // Compute the disability amount.
  // ...
}

// After
disabilityAmount(): number {
  if (isNotEligibleForDisability()) {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```

**Why Refactor.** Your code contains many alternating operators performing identical actions, and it isn't clear why they're split up. The main purpose of consolidation is to extract the conditional to a separate method for greater clarity.

Benefits:
- Eliminates duplicate control-flow code. Combining conditionals with the same "destination" shows you're doing only one complicated check leading to one action.
- Lets you isolate the complex expression in a new method named for the conditional's purpose.

**How To.**
Before starting, make sure the conditionals have no side effects (e.g. modifying a variable based on the result) rather than simply returning values.
1. Consolidate the conditionals into a single expression using `and`/`or`. General rule: nested conditionals join with `and`; consecutive conditionals join with `or`.
2. Perform **Extract Method** on the conditions, giving the method a name reflecting the expression's purpose.

### Consolidate Duplicate Conditional Fragments
**Problem.** Identical code can be found in all branches of a conditional.

**Solution.** Move the code outside of the conditional.

```ts
// Before
if (isSpecialDeal()) {
  total = price * 0.95;
  send();
} else {
  total = price * 0.98;
  send();
}

// After
if (isSpecialDeal()) {
  total = price * 0.95;
} else {
  total = price * 0.98;
}
send();
```

**Why Refactor.** Duplicate code in all branches of a conditional often results from the gradual evolution of the code within those branches. Team development can be a contributing factor.

Benefits:
- Code deduplication.

**How To.**
1. If the duplicated code is at the *beginning* of the branches, move it to before the conditional.
2. If it executes at the *end* of the branches, place it after the conditional.
3. If the duplicate code is randomly situated inside the branches, first try moving it to the beginning or end of the branch, depending on whether it changes the result of the subsequent code.
4. If appropriate and the duplicate code is longer than one line, try **Extract Method**.

### Decompose Conditional
**Problem.** You have a complex conditional (`if-then`/`else` or `switch`).

**Solution.** Decompose the complicated parts into separate methods: the condition, `then`, and `else`.

```ts
// Before
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
} else {
  charge = quantity * summerRate;
}

// After
if (isSummer(date)) {
  charge = summerCharge(quantity);
} else {
  charge = winterCharge(quantity);
}
```

**Why Refactor.** The longer code is, the harder to understand — worse when filled with conditions:
- While figuring out what the `then` block does, you forget the condition.
- While parsing `else`, you forget what `then` does.

Benefits:
- Extracting conditional code to clearly named methods makes life easier for whoever maintains it later (such as you, two months from now!).
- Applies even to short expressions in conditions — `isSalaryDay()` is much prettier and more descriptive than code comparing dates.

**How To.**
1. Extract the condition to a separate method via **Extract Method**.
2. Repeat for the `then` and `else` blocks.

### Replace Conditional with Polymorphism
**Problem.** You have a conditional that performs various actions depending on object type or properties.

**Solution.** Create subclasses matching the branches of the conditional. In them, create a shared method and move code from the corresponding branch into it. Then replace the conditional with the relevant method call. The proper implementation is attained via polymorphism depending on the object class.

```ts
// Before
class Bird {
  getSpeed(): number {
    switch (type) {
      case EUROPEAN:
        return getBaseSpeed();
      case AFRICAN:
        return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return (isNailed) ? 0 : getBaseSpeed(voltage);
    }
    throw new Error("Should be unreachable");
  }
}

// After
abstract class Bird {
  abstract getSpeed(): number;
}
class European extends Bird {
  getSpeed(): number { return getBaseSpeed(); }
}
class African extends Bird {
  getSpeed(): number {
    return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
  }
}
class NorwegianBlue extends Bird {
  getSpeed(): number { return (isNailed) ? 0 : getBaseSpeed(voltage); }
}

// Somewhere in client code
let speed = bird.getSpeed();
```

**Why Refactor.** Helps when code contains operators performing various tasks that vary based on: the class of the object (or interface it implements), the value of an object's field, or the result of calling one of its methods. If a new property or type appears, you must find and add code in all similar conditionals — so the benefit multiplies when conditionals are scattered across an object's methods.

Benefits:
- Adheres to the **Tell-Don't-Ask** principle: instead of asking an object about its state and then acting, just tell the object what it needs to do and let it decide how.
- Removes duplicate code — you get rid of many almost-identical conditionals.
- To add a new execution variant, just add a new subclass without touching existing code (**Open/Closed Principle**).

**How To.**
*Preparing:* you need a ready hierarchy of classes containing the alternative behaviors. If you don't have one, create it — **Replace Type Code with Subclasses** (simple but less flexible: can't subclass for other properties) or **Replace Type Code with State/Strategy** (a dedicated class per property, with subclasses per value, delegated to). The steps below assume the hierarchy exists.

1. If the conditional is in a method that performs other actions too, perform **Extract Method**.
2. For each hierarchy subclass, redefine the method containing the conditional and copy the corresponding branch's code there.
3. Delete that branch from the conditional.
4. Repeat until the conditional is empty. Then delete the conditional and declare the method abstract.

### Remove Control Flag
**Problem.** You have a boolean variable that acts as a control flag for multiple boolean expressions.

**Solution.** Instead of the variable, use `break`, `continue`, and `return`.

**Why Refactor.** Control flags date back to when "proper" programmers always had one entry point (the function declaration) and one exit point (the very end). In modern languages this is obsolete — we have operators for modifying control flow:
- `break` — stops the loop.
- `continue` — stops the current loop iteration and goes to check the loop conditions in the next iteration.
- `return` — stops execution of the entire function and returns its result if given.

Benefits:
- Control-flag code is often much more ponderous than code written with control-flow operators.

**How To.**
1. Find the value assignment to the control flag that causes the exit from the loop or current iteration.
2. Replace it with `break` (exit from a loop), `continue` (exit from an iteration), or `return` (return the value from the function).
3. Remove the remaining code and checks associated with the control flag.

### Replace Nested Conditional with Guard Clauses
**Problem.** You have a group of nested conditionals and it's hard to determine the normal flow of code execution.

**Solution.** Isolate all special checks and edge cases into separate clauses placed before the main checks. Ideally, you have a "flat" list of conditionals, one after the other.

```ts
// Before
getPayAmount(): number {
  let result: number;
  if (isDead) {
    result = deadAmount();
  } else {
    if (isSeparated) {
      result = separatedAmount();
    } else {
      if (isRetired) {
        result = retiredAmount();
      } else {
        result = normalPayAmount();
      }
    }
  }
  return result;
}

// After
getPayAmount(): number {
  if (isDead) { return deadAmount(); }
  if (isSeparated) { return separatedAmount(); }
  if (isRetired) { return retiredAmount(); }
  return normalPayAmount();
}
```

**Why Refactor.** The "conditional from hell" is easy to spot: the indentations form an arrow pointing right, in the direction of pain and woe. It's hard to figure out what each conditional does because the normal flow isn't obvious. Such conditionals indicate helter-skelter evolution, each condition added as a stopgap without thought to the overall structure. Isolate the special cases into separate conditions that immediately end execution — make the structure flat.

**How To.**
First, try to rid the code of side effects — **Separate Query from Modifier** may help, and it's necessary for the reshuffling below.
1. Isolate all guard clauses that lead to calling an exception or immediately returning a value. Place these conditions at the beginning of the method.
2. After rearrangement and passing all tests, see whether you can use **Consolidate Conditional Expression** for guard clauses leading to the same exceptions or returned values.

### Introduce Null Object
**Problem.** Since some methods return `null` instead of real objects, you have many checks for `null` in your code.

**Solution.** Instead of `null`, return a null object that exhibits the default behavior.

```ts
// Before
if (customer == null) {
  plan = BillingPlan.basic();
} else {
  plan = customer.getPlan();
}

// After
class NullCustomer extends Customer {
  isNull(): boolean { return true; }
  getPlan(): Plan { return new NullPlan(); }
  // Some other NULL functionality.
}

// Replace null values with Null-object.
let customer = (order.customer != null) ? order.customer : new NullCustomer();

// Use Null-object as if it's a normal subclass.
plan = customer.getPlan();
```

**Why Refactor.** Dozens of checks for `null` make your code longer and uglier.

Drawbacks:
- The price of getting rid of conditionals is creating yet another new class.

**How To.**
1. From the class in question, create a subclass that performs the role of null object.
2. In both classes, create the method `isNull()` — returning `true` for the null object and `false` for the real class.
3. Find all places where the code may return `null` instead of a real object. Change them to return a null object.
4. Find all places where variables of the real class are compared with `null`. Replace these checks with a call to `isNull()`.
5. Handle the conditional behavior:
   - If methods of the original class run in these conditionals when the value isn't `null`, redefine those methods in the null class and insert the code from the `else` part there. Then delete the entire conditional — differing behavior is implemented via polymorphism.
   - If the methods can't be redefined, extract the operators that were meant for the `null` case into new methods of the null object, and call them instead of the old `else` code as the default operations.

### Introduce Assertion
**Problem.** For a portion of code to work correctly, certain conditions or values must be true.

**Solution.** Replace these assumptions with specific assertion checks.

```ts
// Before
getExpenseLimit(): number {
  // Should have either expense limit or a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit :
    primaryProject.getMemberExpenseLimit();
}

// After
getExpenseLimit(): number {
  // TS/JS has no built-in assertions, so we use console.error().
  // You can always extract this into a designated assertion function.
  if (!(expenseLimit != NULL_EXPENSE ||
       (typeof primaryProject !== 'undefined' && primaryProject))) {
      console.error("Assertion failed: getExpenseLimit()");
  }

  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit :
    primaryProject.getMemberExpenseLimit();
}
```

**Why Refactor.** A portion of code may assume something about, e.g., the current condition of an object or the value of a parameter/local variable. Usually this assumption always holds true except in the event of an error. Make assumptions obvious by adding assertions — like type hinting on method parameters, they act as live documentation. A good guide to where assertions are needed: comments that describe the conditions under which a method works.

Benefits:
- If an assumption isn't true and the code gives the wrong result, it's better to stop execution before this causes fatal consequences and data corruption. It also flags that you neglected to write a necessary test.

Drawbacks:
- Sometimes an exception is more appropriate — you can pick the exception class and let the remaining code handle it correctly. An exception is better when it can be caused by user/system actions and you can handle it. Ordinary unnamed, unhandled exceptions are basically equivalent to simple assertions — caused only by a program bug that never should have occurred.

**How To.**
- When you see a condition is assumed, add an assertion for it. Adding the assertion shouldn't change the program's behavior.
- Don't overdo it — check only the conditions necessary for correct functioning. If the code works normally even when a particular assertion is false, you can safely remove it.

---

## Simplifying Method Calls

### Add Parameter
**Problem.** A method doesn't have enough data to perform certain actions.

**Solution.** Create a new parameter to pass the necessary data.

**Why Refactor.** You need to make changes to a method that require information previously unavailable to it.

Benefits:
- The choice is between adding a new parameter and adding a new private field holding the data. A parameter is preferable for occasional or frequently changing data not worth holding in the object all the time. Otherwise, add a private field and fill it before calling the method.

Drawbacks:
- Adding a parameter is always easier than removing one, which is why parameter lists balloon to grotesque sizes — the **Long Parameter List** smell.
- Needing a new parameter sometimes means the class doesn't contain the necessary data, or existing parameters lack the related data. In both cases, consider moving data to the main class or to other classes whose objects are already accessible inside the method.

**How To.**
1. See whether the method is defined in a superclass or subclass. If so, repeat all steps in those classes too.
2. (Critical for keeping the program working.) Create a new method by copying the old one and add the necessary parameter. Replace the old method's code with a call to the new method, plugging in any value for the new parameter (e.g. `null` for objects, `0` for numbers).
3. Find all references to the old method and replace them with the new method.
4. Delete the old method — unless it's part of the public interface, in which case mark it deprecated.

### Remove Parameter
**Problem.** A parameter isn't used in the body of a method.

**Solution.** Remove the unused parameter.

**Why Refactor.** Every parameter forces the reader to figure out what information it holds — and if a parameter is entirely unused, that effort is for naught. Additional parameters are also extra code to run. Sometimes we add parameters anticipating future changes, but experience shows it's better to add a parameter only when genuinely needed; anticipated changes often remain just that.

Benefits:
- A method contains only the parameters it truly requires.

**When Not to Use.**
- If the method is implemented differently in subclasses or a superclass, and your parameter is used in those implementations, leave it as-is.

**How To.**
1. See whether the method is defined in a superclass or subclass, and whether the parameter is used there. If used in one of those implementations, hold off.
2. (Important for keeping the program working.) Create a new method by copying the old one and delete the relevant parameter. Replace the old method's code with a call to the new one.
3. Find all references to the old method and replace them with the new method.
4. Delete the old method — unless it's part of a public interface, in which case mark it deprecated.

### Rename Method
**Problem.** The name of a method doesn't explain what the method does.

**Solution.** Rename the method.

**Why Refactor.** Perhaps the method was poorly named from the start — someone created it in a rush. Or it was well named at first, but as its functionality grew the name stopped being a good descriptor.

Benefits:
- **Code readability** — give the method a name that reflects what it does: `createOrder()`, `renderCustomerInfo()`, etc.

**How To.**
1. See whether the method is defined in a superclass or subclass. If so, repeat all steps in those classes too.
2. (Important for keeping the program working during refactoring.) Create a new method with the new name. Copy the old method's code into it. Delete all the code in the old method and instead insert a call to the new method.
3. Find all references to the old method and replace them with references to the new one.
4. Delete the old method. If it's part of a public interface, don't — instead mark it deprecated.

### Separate Query from Modifier
**Problem.** You have a method that returns a value but also changes something inside an object.

**Solution.** Split the method into two: one returns the value, the other modifies the object.

**Why Refactor.** This implements **Command and Query Responsibility Segregation** — separating code that gets data (a *query*) from code that changes the visible state of an object (a *modifier*). When the two are combined, you can't get data without changing its condition — you ask a question and can change the answer even as it's received. The problem worsens when the caller doesn't know about the method's side effects, often causing runtime errors.

Note: side effects are dangerous only for modifiers that change the *visible* state of an object (fields in the public interface, database entries, files, etc.). A modifier that only caches a complex operation in a private field can hardly cause side effects.

Benefits:
- A query that doesn't change program state can be called as many times as you like, without worrying about unintended changes caused merely by calling it.

Drawbacks:
- Sometimes it's convenient to get data after performing a command — e.g. deleting from a database and wanting to know how many rows were deleted.

**How To.**
1. Create a new query method that returns what the original method did.
2. Change the original method to return only the result of calling the new query method.
3. Replace all references to the original method with a call to the query method. Immediately before that line, place a call to the modifier method — this saves you from side effects if the original was used in a condition of a conditional or loop.
4. Get rid of the value-returning code in the original method, which has now become a proper modifier method.

### Parameterize Method
**Problem.** Multiple methods perform similar actions that differ only in their internal values, numbers, or operations.

**Solution.** Combine these methods using a parameter that passes the necessary special value.

**Why Refactor.** Similar methods probably mean duplicate code, with all its consequences. And if you need to add yet another version of the functionality, you'd have to create yet another method — instead you could simply run the existing method with a different parameter.

Drawbacks:
- This can be taken too far, resulting in one long and complicated common method instead of multiple simpler ones.
- Be careful moving activation/deactivation of functionality to a parameter — it can eventually lead to a large conditional operator needing **Replace Parameter with Explicit Methods**.

**How To.**
1. Create a new method with a parameter and move the code that's the same across all into it via **Extract Method**. Sometimes only a certain part of the methods is the same — in that case, extract only that part.
2. In the new method, replace the special/differing value with the parameter.
3. For each old method, find where it's called and replace those calls with calls to the new parameterized method. Then delete the old method.

### Introduce Parameter Object
**Problem.** Your methods contain a repeating group of parameters.

**Solution.** Replace these parameters with an object.

**Why Refactor.** Identical groups of parameters are often found in multiple methods, causing code duplication of both the parameters themselves and related operations. By consolidating parameters in a single class, you can also move the methods for handling that data there, freeing the other methods from this code.

Benefits:
- More readable code — instead of a hodgepodge of parameters, you see a single object with a comprehensible name.
- Removes a subtle form of duplication: even when identical code isn't being called, identical groups of parameters and arguments keep appearing.

Drawbacks:
- If you move only data to the new class and don't plan to move any behaviors or related operations, it begins to smell of a **Data Class**.

**How To.**
1. Create a new class to represent your group of parameters. Make it immutable.
2. In the method you want to refactor, use **Add Parameter** for the parameter object. In all calls, pass the object created from the old parameters.
3. Delete the old parameters one by one, replacing them in the code with fields of the parameter object. Test after each replacement.
4. When done, see whether it's worth moving part (or sometimes all) of the method to the parameter-object class. If so, use **Move Method** or **Extract Method**.

### Preserve Whole Object
**Problem.** You get several values from an object and then pass them as parameters to a method.

**Solution.** Instead, try passing the whole object.

```ts
// Before
let low = daysTempRange.getLow();
let high = daysTempRange.getHigh();
let withinPlan = plan.withinRange(low, high);

// After
let withinPlan = plan.withinRange(daysTempRange);
```

**Why Refactor.** Each time before the method is called, the methods of the future parameter object must be called. If those methods or the quantity of data obtained change, you must carefully find a dozen such places and update each. After this refactoring, the code for getting all necessary data lives in one place — the method itself.

Benefits:
- Instead of a hodgepodge of parameters, you see a single object with a comprehensible name.
- If the method needs more data from the object, you won't need to rewrite all the call sites — only the method itself.

Drawbacks:
- Sometimes this makes a method less flexible: previously it could get data from many sources; now we limit its use to objects with a particular interface.

**How To.**
1. Create a parameter in the method for the object from which you can get the necessary values.
2. Remove the old parameters one by one, replacing them with calls to the relevant methods of the parameter object. Test after each replacement.
3. Delete the getter code that had preceded the method call.

### Remove Setting Method
**Problem.** The value of a field should be set only when it's created, and never change after that.

**Solution.** Remove methods that set the field's value.

**Why Refactor.** You want to prevent any changes to the value of a field.

**How To.**
1. The value should be changeable only in the constructor. If the constructor doesn't contain a parameter for setting the value, add one.
2. Find all setter calls.
3. If a setter call is located right after a call to the constructor of the current class, move its argument to the constructor call and remove the setter.
4. Replace setter calls in the constructor with direct access to the field.
5. Delete the setter.

### Replace Parameter with Explicit Methods
**Problem.** A method is split into parts, each run depending on the value of a parameter.

**Solution.** Extract the individual parts into their own methods and call them instead of the original method.

```ts
// Before
setValue(name: string, value: number): void {
  if (name.equals("height")) {
    height = value;
    return;
  }
  if (name.equals("width")) {
    width = value;
    return;
  }
}

// After
setHeight(arg: number): void {
  height = arg;
}
setWidth(arg: number): number {
  width = arg;
}
```

**Why Refactor.** A method with parameter-dependent variants has grown massive, runs non-trivial code in each branch, and new variants are added very rarely.

Benefits:
- Improves readability — it's much easier to understand `startEngine()` than `setValue("engineEnabled", true)`.

**When Not to Use.**
- Don't apply this if a method is rarely changed and new variants aren't added inside it.

**How To.**
1. For each variant of the method, create a separate method. Run these methods based on the parameter value in the main method.
2. Find all places where the original method is called. Replace each with a call to one of the new variants.
3. When no calls to the original method remain, delete it.

### Replace Parameter with Method Call
**Problem.** You call a query method and pass its result as a parameter of another method, while that method could call the query directly.

**Solution.** Instead of passing the value through a parameter, place a query call inside the method body.

```ts
// Before
let basePrice = quantity * itemPrice;
const seasonDiscount = this.getSeasonalDiscount();
const fees = this.getFees();
const finalPrice = discountedPrice(basePrice, seasonDiscount, fees);

// After
let basePrice = quantity * itemPrice;
let finalPrice = discountedPrice(basePrice);
```

**Why Refactor.** A long list of parameters is hard to understand. Calls to such methods often resemble cascades of winding value calculations that are hard to navigate yet must be passed in. So if a parameter value can be calculated via a method, do it inside the method and get rid of the parameter.

Benefits:
- Removes unneeded parameters and simplifies method calls. Such parameters are often created not for the project as it is now, but with an eye for future needs that may never come.

Drawbacks:
- You may need the parameter tomorrow for other needs, making you rewrite the method.

**How To.**
1. Make sure the value-getting code doesn't use parameters from the current method — they'll be unavailable from inside another method. If it does, moving the code isn't possible.
2. If the relevant code is more complicated than a single method/function call, use **Extract Method** to isolate it in a new method and make the call simple.
3. In the main method, replace all references to the parameter being replaced with calls to the method that gets the value.
4. Use **Remove Parameter** to eliminate the now-unused parameter.

### Hide Method
**Problem.** A method isn't used by other classes, or is used only inside its own class hierarchy.

**Solution.** Make the method private or protected.

**Why Refactor.** The need to hide getter/setter methods often arises from developing a richer interface with additional behavior, especially if you started with a class that added little beyond data encapsulation. As new behavior is built in, public getters/setters may no longer be necessary and can be hidden. If you make them private and use direct variable access, you can delete the method.

Benefits:
- Makes code easier to evolve — when you change a private method, you only need to worry about not breaking the current class, since it can't be used anywhere else.
- Underscores the importance of the public interface and the methods that remain public.

**How To.**
1. Regularly try to find methods that can be made private. Static code analysis and good unit-test coverage help a lot.
2. Make each method as private as possible.

### Replace Constructor with Factory Method
**Problem.** You have a complex constructor that does more than just set parameter values in object fields.

**Solution.** Create a factory method and use it to replace constructor calls.

```ts
// Before
class Employee {
  constructor(type: number) {
    this.type = type;
  }
}

// After
class Employee {
  static create(type: number): Employee {
    let employee = new Employee(type);
    // Do some heavy lifting.
    return employee;
  }
}
```

**Why Refactor.** The most obvious reason relates to **Replace Type Code with Subclasses**: code previously created an object and passed it the coded-type value, but after refactoring several subclasses exist and you need to create objects depending on the coded value. You can't change the original constructor to return subclass objects, so you create a static factory method that returns objects of the necessary classes, then replace all calls to the original constructor. Factory methods are also useful when constructors aren't up to the task — important for **Change Value to Reference**, and for setting various creation modes beyond the number and types of parameters.

Benefits:
- A factory method doesn't necessarily return an object of the class it was called in — often its subclasses, selected based on the arguments.
- A factory method can have a better, descriptive name, e.g. `Troops::GetCrew(myTank)`.
- A factory method can return an already-created object, unlike a constructor, which always creates a new instance.

**How To.**
1. Create a factory method. Place a call to the current constructor in it.
2. Replace all constructor calls with calls to the factory method.
3. Declare the constructor private.
4. Investigate the constructor code and isolate the code not directly related to constructing an object of the current class, moving it to the factory method.

### Replace Error Code with Exception
**Problem.** A method returns a special value that indicates an error.

**Solution.** Throw an exception instead.

```ts
// Before
withdraw(amount: number): number {
  if (amount > _balance) {
    return -1;
  } else {
    balance -= amount;
    return 0;
  }
}

// After
withdraw(amount: number): void {
  if (amount > _balance) {
    throw new Error();
  }
  balance -= amount;
}
```

**Why Refactor.** Returning error codes is an obsolete holdover from procedural programming. In modern programming, error handling is done by special classes — exceptions. If a problem occurs you "throw" an error, which is "caught" by one of the exception handlers; special error-handling code, ignored in normal conditions, activates to respond.

Benefits:
- Frees code from a large number of conditionals checking various error codes. Exception handlers are a much more succinct way to separate normal execution paths from abnormal ones.
- Exception classes can implement their own methods, containing part of the error-handling functionality (e.g. sending error messages).
- Unlike exceptions, error codes can't be used in a constructor, which must return only a new object.

Drawbacks:
- An exception handler can turn into a goto-like crutch — avoid this. Don't use exceptions to manage code execution; throw them only to inform of an error or critical situation.

**How To.**
Do these steps for only one error code at a time — it's easier to keep the important information in your head and avoid errors.
1. Find all calls to a method that returns error codes and, instead of checking for an error code, wrap it in `try`/`catch` blocks.
2. Inside the method, instead of returning an error code, throw an exception.
3. Change the method signature to contain information about the exception being thrown (`@throws` section).

### Replace Exception with Test
**Problem.** You throw an exception in a place where a simple test would do the job.

**Solution.** Replace the exception with a condition test.

```ts
// Before
getValueForPeriod(periodNumber: number): number {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}

// After
getValueForPeriod(periodNumber: number): number {
  if (periodNumber >= values.length) {
    return 0;
  }
  return values[periodNumber];
}
```

**Why Refactor.** Exceptions should handle irregular behavior related to an unexpected error — not serve as a replacement for testing. If an exception can be avoided by simply verifying a condition before running, do so. Reserve exceptions for real errors. (Analogy: you entered a minefield, triggered a mine, and the exception was handled by flinging you to safety — but you could have avoided it all by reading the warning sign in front of the minefield.)

Benefits:
- A simple conditional can sometimes be more obvious than exception-handling code.

**How To.**
1. Create a conditional for the edge case and move it before the `try`/`catch` block.
2. Move code from the `catch` section inside this conditional.
3. In the `catch` section, place code for throwing a usual unnamed exception and run all the tests.
4. If no exceptions were thrown during the tests, get rid of the `try`/`catch` operator.

---

## Dealing with Generalization

### Pull Up Field
**Problem.** Two classes have the same field.

**Solution.** Remove the field from subclasses and move it to the superclass.

**Why Refactor.** Subclasses grew and developed separately, causing identical (or nearly identical) fields and methods to appear.

Benefits:
- Eliminates duplication of fields in subclasses.
- Eases subsequent relocation of duplicate methods, if they exist, from subclasses to a superclass.

**How To.**
1. Make sure the fields are used for the same needs in subclasses.
2. If the fields have different names, give them the same name and replace all references in existing code.
3. Create a field with the same name in the superclass. If the fields were private, the superclass field should be `protected`.
4. Remove the fields from the subclasses.
5. Consider **Self Encapsulate Field** for the new field, to hide it behind access methods.

### Pull Up Method
**Problem.** Your subclasses have methods that perform similar work.

**Solution.** Make the methods identical and then move them to the relevant superclass.

**Why Refactor.** Subclasses grew and developed independently, causing identical (or nearly identical) fields and methods.

Benefits:
- Gets rid of duplicate code. If you need to change a method, it's better to do so in a single place than to search for all duplicates in subclasses.
- Also useful if a subclass redefines a superclass method but performs essentially the same work.

**How To.**
1. Investigate similar methods. If they aren't identical, format them to match each other.
2. If methods use a different set of parameters, put the parameters in the form you want in the superclass.
3. Copy the method to the superclass. If the method code uses fields and methods that exist only in subclasses (and aren't available in the superclass):
   - For fields: use **Pull Up Field** or **Self Encapsulate Field** to create getters/setters in subclasses, then declare those getters abstractly in the superclass.
   - For methods: use **Pull Up Method** or declare abstract methods for them in the superclass (the class becomes abstract if it wasn't already).
4. Remove the methods from the subclasses.
5. Check where the method is called. In some places you may be able to replace use of a subclass with the superclass.

### Pull Up Constructor Body
**Problem.** Your subclasses have constructors with code that's mostly identical.

**Solution.** Create a superclass constructor and move the code that's the same in the subclasses to it. Call the superclass constructor in the subclass constructors.

```ts
// Before
class Manager extends Employee {
  constructor(name: string, id: string, grade: number) {
    this.name = name;
    this.id = id;
    this.grade = grade;
  }
}

// After
class Manager extends Employee {
  constructor(name: string, id: string, grade: number) {
    super(name, id);
    this.grade = grade;
  }
}
```

**Why Refactor.** How this differs from **Pull Up Method**:
1. In many languages (e.g. Java), subclasses can't inherit a constructor, so you can't simply pull up the constructor and delete it. Besides creating a superclass constructor, you must keep constructors in the subclasses with simple delegation to the superclass constructor.
2. In C++ and Java (if you didn't explicitly call the superclass constructor), the superclass constructor is automatically called before the subclass constructor — so you can only move common code from the *beginning* of the subclass constructors (you can't call the superclass constructor from an arbitrary place).
3. In most languages, a subclass constructor can have its own parameter list different from the superclass. Create a superclass constructor only with the parameters it truly needs.

**How To.**
1. Create a constructor in the superclass.
2. Extract the common code from the beginning of each subclass constructor into the superclass constructor. First try to move as much common code as possible to the beginning of the constructor.
3. Place the call to the superclass constructor in the first line of the subclass constructors.

### Push Down Field
**Problem.** A field is used only in a few subclasses.

**Solution.** Move the field to these subclasses.

**Why Refactor.** A field planned for universal use is in reality used only in some subclasses — this can happen when planned features fail to pan out, or after extraction/removal of part of a hierarchy's functionality.

Benefits:
- Improves internal class coherency — a field is located where it's actually used.
- When moving to several subclasses simultaneously, you can develop the fields independently of each other. This does create code duplication, so push down fields only when you really intend to use them in different ways.

**How To.**
1. Declare the field in all the necessary subclasses.
2. Remove the field from the superclass.

### Push Down Method
**Problem.** Behavior implemented in a superclass is used by only one (or a few) subclasses.

**Solution.** Move this behavior to the subclasses.

**Why Refactor.** A method was meant to be universal for all classes but in reality is used in only one subclass — this can happen when planned features fail to materialize, or after partial extraction/removal of functionality from a hierarchy. If a method is needed by more than one subclass but not all, consider creating an intermediate subclass and moving the method there, to avoid the duplication of pushing it down to all subclasses.

Benefits:
- Improves class coherence — a method is located where you expect to see it.

**How To.**
1. Declare the method in a subclass and copy its code from the superclass.
2. Remove the method from the superclass.
3. Find all places where the method is used and verify it's called from the necessary subclass.

### Extract Subclass
**Problem.** A class has features that are used only in certain cases.

**Solution.** Create a subclass and use it in these cases.

**Why Refactor.** Your main class has methods and fields implementing a rare use case. The class is responsible for it, so moving everything to an entirely separate class would be wrong — but they can be moved to a subclass.

Benefits:
- Creates a subclass quickly and easily.
- You can create several separate subclasses if the main class implements more than one special case.

Drawbacks:
- Inheritance can lead to a dead end if you must separate several different class hierarchies. E.g. a `Dog` class with behavior depending on size and fur could tease out two hierarchies — by size (`Large`/`Medium`/`Small`) and by fur (`Smooth`/`Shaggy`) — but problems arise when you need a dog that's both `Large` and `Smooth`, since an object can come from only one class. Avoid this with **composition instead of inheritance** (see the **Strategy** pattern): `Dog` has `size` and `fur` component fields into which you plug component objects, so you can create a `Dog` with `LargeSize` and `ShaggyFur`.

**How To.**
1. Create a new subclass from the class of interest.
2. If you need additional data to create subclass objects, create a constructor and add the necessary parameters, calling the parent implementation.
3. Find all calls to the parent constructor. Where the subclass's functionality is needed, replace the parent constructor with the subclass constructor.
4. Move the necessary methods and fields from the parent to the subclass via **Push Down Method** and **Push Down Field**. It's simpler to move methods first — fields stay accessible throughout: from the parent before the move, and from the subclass after.
5. After the subclass is ready, find the old fields that controlled the choice of functionality. Delete them using polymorphism to replace the operators that used them. (E.g. `Car` had `isElectricCar`, and `refuel()` either fuels with gas or charges with electricity; post-refactoring the field is removed and `Car` and `ElectricCar` have their own `refuel()` implementations.)

### Extract Superclass
**Problem.** You have two classes with common fields and methods.

**Solution.** Create a shared superclass for them and move all the identical fields and methods to it.

**Why Refactor.** Code duplication occurs when two classes perform similar tasks in the same way, or similar tasks in different ways. Objects offer inheritance to simplify such situations, but often the similarity goes unnoticed until the classes are created — necessitating an inheritance structure later.

Benefits:
- Code deduplication — common fields and methods now live in one place only.

**When Not to Use.**
- You can't apply this to classes that already have a superclass.

**How To.**
1. Create an abstract superclass.
2. Use **Pull Up Field**, **Pull Up Method**, and **Pull Up Constructor Body** to move common functionality up. Start with the fields — besides the common fields, you'll need to move fields used in the common methods.
3. Look for places in client code where use of subclasses can be replaced with the new class (e.g. in type declarations).

### Extract Interface
**Problem.** Multiple clients use the same part of a class interface. Another case: part of the interface in two classes is the same.

**Solution.** Move this identical portion to its own interface.

**Why Refactor.** Interfaces are apropos when classes play special roles in different situations — use Extract Interface to explicitly indicate which role. Another convenient case: describing the operations a class performs on its server. If you plan to eventually allow servers of multiple types, all servers must implement the interface.

**Good to Know.** There's a resemblance between **Extract Superclass** and Extract Interface. Extracting an interface isolates only common *interfaces*, not common *code* — so if classes contain Duplicate Code, extracting the interface won't deduplicate it. You can mitigate this by applying **Extract Class** to move the duplicated behavior into a separate component and delegating to it. If the common behavior is large, you can use **Extract Superclass** — easier, but remember you get only one parent class.

**How To.**
1. Create an empty interface.
2. Declare the common operations in the interface.
3. Declare the necessary classes as implementing the interface.
4. Change type declarations in the client code to use the new interface.

### Collapse Hierarchy
**Problem.** You have a class hierarchy in which a subclass is practically the same as its superclass.

**Solution.** Merge the subclass and superclass.

**Why Refactor.** Your program grew over time and a subclass and superclass became practically the same — a feature was removed from a subclass, a method moved to the superclass, and now you have two look-alike classes.

Benefits:
- Reduces program complexity — fewer classes mean fewer things to keep straight and fewer breakable moving parts during future changes.
- Navigating code is easier when methods are defined in one class; you don't need to comb the entire hierarchy to find a method.

**When Not to Use.**
- If the hierarchy has more than one subclass, after refactoring the remaining subclasses should become inheritors of the merged class. But keep in mind this can violate the **Liskov Substitution Principle** — e.g. if your program models city transport and you accidentally collapse `Transport` into `Car`, then `Plane` may become an inheritor of `Car`. Oops!

**How To.**
1. Select which class is easier to remove: the superclass or its subclass.
2. Use **Pull Up Field** and **Pull Up Method** if getting rid of the subclass; use **Push Down Field** and **Push Down Method** if eliminating the superclass.
3. Replace all uses of the class being deleted with the class the fields and methods migrate to — often code for creating classes, variable/parameter typing, and documentation in comments.
4. Delete the empty class.

### Form Template Method
**Problem.** Your subclasses implement algorithms that contain similar steps in the same order.

**Solution.** Move the algorithm structure and identical steps to a superclass, and leave implementation of the different steps in the subclasses.

**Why Refactor.** Subclasses are developed in parallel, sometimes by different people, leading to code duplication, errors, and maintenance difficulties since each change must be made in all subclasses.

Benefits:
- Duplication isn't always simple copy/paste — it often occurs at a higher level, e.g. a method for sorting numbers and a method for sorting object collections differing only in the comparison of elements. A template method eliminates this by merging the shared algorithm steps in a superclass and leaving just the differences in the subclasses.
- An example of the **Open/Closed Principle** in action — when a new algorithm version appears, you only create a new subclass; no changes to existing code are required.

**How To.**
1. Split the subclass algorithms into constituent parts described in separate methods (**Extract Method** helps).
2. Move the resulting methods that are identical across subclasses to the superclass via **Pull Up Method**.
3. Give the non-similar methods consistent names via **Rename Method**.
4. Move the signatures of the non-similar methods to the superclass as abstract ones via **Pull Up Method**, leaving their implementations in the subclasses.
5. Finally, pull up the main method of the algorithm to the superclass. It should now work with the method steps described in the superclass, both real and abstract.

### Replace Inheritance with Delegation
**Problem.** You have a subclass that uses only a portion of the methods of its superclass (or it's not possible to inherit superclass data).

**Solution.** Create a field and put a superclass object in it, delegate methods to the superclass object, and get rid of inheritance.

**Why Refactor.** Replacing inheritance with composition can substantially improve class design if:
- Your subclass violates the **Liskov Substitution Principle** — i.e. inheritance was implemented only to combine common code, not because the subclass is an extension of the superclass.
- The subclass uses only a portion of the superclass's methods — it's only a matter of time before someone calls a superclass method they weren't supposed to.

In essence, this splits both classes and makes the superclass the *helper* of the subclass, not its parent. Instead of inheriting all superclass methods, the subclass has only the necessary methods that delegate to the superclass object.

Benefits:
- A class doesn't contain unneeded methods inherited from the superclass.
- Various objects with various implementations can be put in the delegate field — in effect you get the **Strategy** design pattern.

Drawbacks:
- You have to write many simple delegating methods.

**How To.**
1. Create a field in the subclass for holding the superclass. Initially, place the current object in it.
2. Change the subclass methods to use the superclass object instead of `this`.
3. For methods inherited from the superclass and called in client code, create simple delegating methods in the subclass.
4. Remove the inheritance declaration from the subclass.
5. Change the initialization code of the field holding the former superclass by creating a new object.

### Replace Delegation with Inheritance
**Problem.** A class contains many simple methods that delegate to all methods of another class.

**Solution.** Make the class a delegate inheritor, which makes the delegating methods unnecessary.

**Why Refactor.** Delegation is more flexible than inheritance — it allows changing how delegation is implemented and placing other classes there too. But delegation stops being beneficial if you delegate actions to only one class and all of its public methods. In that case, replacing delegation with inheritance cleanses the class of a large number of delegating methods and spares you from creating them for each new delegate-class method.

Benefits:
- Reduces code length — all those delegating methods are no longer necessary.

**When Not to Use.**
- Don't use this if the class delegates to only a *portion* of the delegate class's public methods — doing so would violate the **Liskov Substitution Principle**.
- Can be used only if the class doesn't already have a parent.

**How To.**
1. Make the class a subclass of the delegate class.
2. Place the current object in a field containing a reference to the delegate object.
3. Delete the simple delegating methods one by one. If their names differed, use **Rename Method** to give them a single name.
4. Replace all references to the delegate field with references to the current object.
5. Remove the delegate field.
