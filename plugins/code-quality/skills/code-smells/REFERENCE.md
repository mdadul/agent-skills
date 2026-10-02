# Code Smells — Reference Catalog

Detailed catalog of each smell. Every entry follows the same shape:

- **Signs and Symptoms** — how to recognize it.
- **Reasons for the Problem** — why it happens.
- **Treatment** — the standard refactoring(s) to apply.
- **Payoff** — what you gain after treating it.
- **When to Ignore** — when the smell is acceptable.

(Not every smell has every section; some have extra notes such as **Performance**.)

---

## Bloaters

### Long Method
A method contains too many lines of code.

**Signs and Symptoms**
- A method has too many lines of code. As a rule of thumb, any method longer than **ten lines** should make you start asking questions.

**Reasons for the Problem**
- Like the Hotel California, something is always being *added* to a method but nothing is ever taken *out*. Because it's easier to write code than to read it, the smell stays unnoticed until the method turns into an ugly, oversized beast.
- Mentally, it's often harder to create a new method than to add to an existing one: "But it's just two lines, there's no use creating a whole method just for that..." — so another line is added, then another, giving birth to a tangle of spaghetti code.

**Treatment**
Rule of thumb: **if you feel the need to comment on something inside a method, extract that code into a new method.** Even a single line can and should be split off if it needs explanation — with a descriptive name, nobody has to read the body to know what it does.
- To reduce the length of a method body, use **Extract Method**.
- If local variables and parameters interfere with extracting, use **Replace Temp with Query**, **Introduce Parameter Object**, or **Preserve Whole Object**.
- If none of those help, move the entire method to a separate object via **Replace Method with Method Object**.
- Conditionals and loops are good clues that code can be moved out. For conditionals, use **Decompose Conditional**; for loops in the way, use **Extract Method**.

**Payoff**
- Among all object-oriented code, classes with short methods live longest. The longer a method is, the harder it is to understand and maintain.
- Long methods are the perfect hiding place for unwanted duplicate code.

**Performance**
- Does adding methods hurt performance? In almost all cases the impact is negligible — not worth worrying about. And with clear code you're more likely to spot genuinely effective restructurings for real performance gains if the need ever arises.

### Large Class
A class contains many fields, methods, and lines of code.

**Signs and Symptoms**
- A class contains many fields / methods / lines of code.

**Reasons for the Problem**
- Classes usually start small, but get bloated over time as the program grows.
- As with long methods, programmers find it mentally less taxing to place a new feature in an existing class than to create a new class for it.

**Treatment**
When a class is wearing too many (functional) hats, split it up:
- **Extract Class** — if part of the behavior can be spun off into a separate component.
- **Extract Subclass** — if part of the behavior can be implemented in different ways or is used only in rare cases.
- **Extract Interface** — if it's necessary to have a list of the operations and behaviors the client can use.
- If a large class is responsible for the graphical interface, move some of its data and behavior to a separate domain object. This may require keeping copies of some data in two places and keeping them consistent — **Duplicate Observed Data** offers a way to do this.

**Payoff**
- Spares developers from having to remember a large number of attributes for a class.
- Splitting large classes often avoids duplication of code and functionality.

### Primitive Obsession
Using primitives instead of small objects for simple tasks.

**Signs and Symptoms**
- Use of primitives instead of small objects for simple tasks (currency, ranges, special strings such as phone numbers, etc.).
- Use of constants for coding information (e.g. a constant `USER_ADMIN_ROLE = 1` for referring to users with administrator rights).
- Use of string constants as field names for use in data arrays.

**Reasons for the Problem**
- Like most smells, primitive obsession is born in a moment of weakness: "Just a field for storing some data!" Creating a primitive field is easier than making a whole class — then another field is added the same way, and the class becomes huge and unwieldy.
- Primitives are often used to "simulate" types: instead of a separate data type, you have a set of numbers or strings forming the list of allowable values, with easy-to-understand names given via constants spread wide and far.
- Another poor use is field simulation: the class holds a large array of diverse data, and string constants are used as array indices to get at that data.

**Treatment**
- If you have a large variety of primitive fields, logically group some into their own class — and move the behavior associated with that data into the class too. Use **Replace Data Value with Object**.
- If the values of primitive fields are used in method parameters, use **Introduce Parameter Object** or **Preserve Whole Object**.
- When complicated data is coded in variables, use **Replace Type Code with Class**, **Replace Type Code with Subclasses**, or **Replace Type Code with State/Strategy**.
- If there are arrays among the variables, use **Replace Array with Object**.

**Payoff**
- Code becomes more flexible thanks to objects instead of primitives.
- Better understandability and organization: operations on particular data live in one place instead of being scattered. No more guessing about strange constants and why they're in an array.
- Easier finding of duplicate code.

### Long Parameter List
More than three or four parameters for a method.

**Signs and Symptoms**
- More than three or four parameters for a method.

**Reasons for the Problem**
- A long list might appear after several algorithms are merged into a single method, with parameters controlling which algorithm runs and how.
- It can also be a byproduct of making classes more independent: code for creating objects needed in a method was moved out to the caller, and the created objects are passed in as parameters. Dependency decreases, but each created object now needs its own parameter — a longer list.
- Long lists are hard to understand and become contradictory and hard to use as they grow. Instead of a long list, a method can use the data of its own object; if that object lacks the data, another object (holding the needed data) can be passed as a parameter.

**Treatment**
- Check what values are passed. If some arguments are just results of method calls on another object, use **Replace Parameter with Method Call** — that object can live in a field of the class or be passed as a parameter.
- Instead of passing a group of data received from another object, pass the object itself via **Preserve Whole Object**.
- If the parameters come from different sources, pass them as a single parameter object via **Introduce Parameter Object**.

**Payoff**
- More readable, shorter code.
- Refactoring may reveal previously unnoticed duplicate code.

**When to Ignore**
- Don't get rid of parameters if doing so would cause unwanted dependency between classes.

### Data Clumps
Identical groups of variables recurring across the code.

**Signs and Symptoms**
- Different parts of the code contain identical groups of variables (such as parameters for connecting to a database). These clumps should be turned into their own classes.

**Reasons for the Problem**
- Often these data groups are due to poor program structure or "copypasta programming".
- A test: delete one of the data values and see whether the others still make sense. If they don't, that's a good sign the group should be combined into an object.

**Treatment**
- If repeating data comprises the fields of a class, use **Extract Class** to move the fields to their own class.
- If the same data clumps are passed in method parameters, use **Introduce Parameter Object** to set them off as a class.
- If some of the data is passed to other methods, consider passing the entire data object instead of individual fields — **Preserve Whole Object** helps here.
- Look at the code that uses these fields; it may be a good idea to move that code into the data class.

**Payoff**
- Improves understanding and organization: operations on particular data are gathered in one place instead of scattered.
- Reduces code size.

**When to Ignore**
- Passing an entire object instead of just its primitive values may create an undesirable dependency between the two classes.

---

## Object-Orientation Abusers

### Alternative Classes with Different Interfaces
Two classes do the same thing but expose different method names.

**Signs and Symptoms**
- Two classes perform identical functions but have different method names.

**Reasons for the Problem**
- The programmer who created one of the classes probably didn't know that a functionally equivalent class already existed.

**Treatment**
Try to put the interface of the classes in terms of a common denominator:
- **Rename Methods** to make them identical in all alternative classes.
- **Move Method**, **Add Parameter**, and **Parameterize Method** to make the signatures and implementations the same.
- If only part of the functionality is duplicated, try **Extract Superclass** — the existing classes become subclasses.
- After applying the chosen treatment, you may be able to delete one of the classes.

**Payoff**
- Gets rid of unnecessary duplicated code, making the result less bulky.
- More readable and understandable code — you no longer have to guess why a second class exists doing the exact same thing as the first.

**When to Ignore**
- Sometimes merging classes is impossible or so difficult as to be pointless — e.g. when the alternative classes are in different libraries that each have their own version of the class.

### Refused Bequest
A subclass uses only some of what it inherits.

**Signs and Symptoms**
- A subclass uses only some of the methods and properties inherited from its parents — the hierarchy is off-kilter. The unneeded methods may simply go unused or be redefined and throw exceptions.

**Reasons for the Problem**
- Someone created inheritance between classes only out of a desire to reuse the code in a superclass — but the superclass and subclass are completely different.

**Treatment**
- If inheritance makes no sense and the subclass really has nothing in common with the superclass, eliminate inheritance via **Replace Inheritance with Delegation**.
- If inheritance is appropriate, get rid of unneeded fields and methods in the subclass. Extract the fields and methods actually needed by the subclass from the parent, put them in a new superclass, and have both classes inherit from it (**Extract Superclass**).

**Payoff**
- Improves code clarity and organization. You'll no longer wonder why the `Dog` class inherits from the `Chair` class (even though they both have 4 legs).

### Switch Statements
A complex `switch` operator or sequence of `if` statements.

**Signs and Symptoms**
- You have a complex `switch` operator or sequence of `if` statements.

**Reasons for the Problem**
- Relatively rare use of `switch`/`case` is one of the hallmarks of object-oriented code. Code for a single `switch` is often scattered across the program — when a new condition is added, you have to find all the `switch` code and modify it.
- Rule of thumb: **when you see `switch`, think of polymorphism.**

**Treatment**
- To isolate a `switch` and put it in the right class, you may need **Extract Method** and then **Move Method**.
- If the `switch` is based on a type code (e.g. switching the program's runtime mode), use **Replace Type Code with Subclasses** or **Replace Type Code with State/Strategy**.
- After specifying the inheritance structure, use **Replace Conditional with Polymorphism**.
- If there aren't too many conditions and they all call the same method with different parameters, polymorphism is superfluous — break that method into smaller methods with **Replace Parameter with Explicit Methods** and change the `switch` accordingly.
- If one of the conditional options is `null`, use **Introduce Null Object**.

**Payoff**
- Improved code organization.

**When to Ignore**
- When a `switch` performs simple actions, there's no reason to change the code.
- `switch` operators are often used legitimately by factory patterns (**Factory Method** or **Abstract Factory**) to select a created class.

### Temporary Field
Fields that hold values only under certain circumstances, and are empty otherwise.

**Signs and Symptoms**
- Temporary fields get their values (and thus are needed by objects) only under certain circumstances. Outside of these circumstances, they're empty.

**Reasons for the Problem**
- Often temporary fields are created for an algorithm that requires a large number of inputs. Instead of creating many parameters, the programmer creates fields for this data. These fields are used only in the algorithm and go unused the rest of the time.
- Such code is tough to understand: you expect to see data in object fields, but for some reason they're almost always empty.

**Treatment**
- Temporary fields and all code operating on them can be put in a separate class via **Extract Class** — i.e. you're creating a method object, the same result as **Replace Method with Method Object**.
- Use **Introduce Null Object** in place of the conditional code that checked the temporary field values for existence.

**Payoff**
- Better code clarity and organization.

---

## Change Preventers

### Divergent Change
Many changes are made to a single class for unrelated reasons.

> Divergent Change resembles **Shotgun Surgery** but is the opposite smell. Divergent Change = many changes made to a *single* class. Shotgun Surgery = a *single* change made to *multiple* classes simultaneously.

**Signs and Symptoms**
- You find yourself having to change many unrelated methods when you make changes to a class. For example, when adding a new product type you have to change the methods for finding, displaying, and ordering products.

**Reasons for the Problem**
- Often these divergent modifications are due to poor program structure or "copypasta programming".

**Treatment**
- Split up the behavior of the class via **Extract Class**.
- If different classes have the same behavior, combine them through inheritance (**Extract Superclass** and **Extract Subclass**).

**Payoff**
- Improves code organization.
- Reduces code duplication.
- Simplifies support.

### Parallel Inheritance Hierarchies
Creating a subclass in one hierarchy forces a subclass in another.

**Signs and Symptoms**
- Whenever you create a subclass for a class, you find yourself needing to create a subclass for another class.

**Reasons for the Problem**
- All was well while the hierarchy stayed small. But with new classes being added, making changes has become harder and harder.

**Treatment**
De-duplicate parallel hierarchies in two steps:
1. Make instances of one hierarchy refer to instances of the other.
2. Remove the hierarchy in the referred class, using **Move Method** and **Move Field**.

**Payoff**
- Reduces code duplication.
- Can improve organization of code.

**When to Ignore**
- Sometimes parallel hierarchies are just a way to avoid an even bigger mess in program architecture. If your attempts to de-duplicate produce even uglier code, step out, revert all changes, and get used to the code as is.

### Shotgun Surgery
A single change forces many small edits across many classes.

> Shotgun Surgery resembles **Divergent Change** but is the opposite smell. Divergent Change = many changes made to a *single* class. Shotgun Surgery = a *single* change made to *multiple* classes simultaneously.

**Signs and Symptoms**
- Making any modification requires that you make many small changes to many different classes.

**Reasons for the Problem**
- A single responsibility has been split up among a large number of classes. This can happen after overzealous application of treating Divergent Change.

**Treatment**
- Use **Move Method** and **Move Field** to move existing class behaviors into a single class. If there's no appropriate class, create a new one.
- If moving code leaves the original classes almost empty, get rid of these now-redundant classes via **Inline Class**.

**Payoff**
- Better organization.
- Less code duplication.
- Easier maintenance.

---

## Dispensables

### Comments
A method is filled with explanatory comments.

**Signs and Symptoms**
- A method is filled with explanatory comments.

**Reasons for the Problem**
- Comments are usually created with the best intentions, when the author realizes the code isn't intuitive or obvious. In such cases comments are like a deodorant masking the smell of fishy code that could be improved.
- The best comment is a good name for a method or class. If a fragment can't be understood without comments, change the code structure so the comments become unnecessary.

**Treatment**
- If a comment explains a complex expression, split the expression into understandable subexpressions using **Extract Variable**.
- If a comment explains a section of code, turn that section into a separate method via **Extract Method** — the new method's name can often come from the comment text itself.
- If a method has already been extracted but still needs comments to explain what it does, give it a self-explanatory name via **Rename Method**.
- If you need to assert rules about a state necessary for the system to work, use **Introduce Assertion**.

**Payoff**
- Code becomes more intuitive and obvious.

**When to Ignore**
Comments can sometimes be useful:
- When explaining *why* something is implemented in a particular way.
- When explaining complex algorithms (when all other methods for simplifying the algorithm have been tried and come up short).

### Duplicate Code
Two code fragments look almost identical.

**Signs and Symptoms**
- Two code fragments look almost identical.

**Reasons for the Problem**
- Duplication usually occurs when multiple programmers work on different parts of the same program at the same time — unaware a colleague already wrote similar code that could be repurposed.
- There's also subtler duplication, where parts of code look different but actually do the same job. This is hard to find and fix.
- Sometimes duplication is purposeful: rushing to meet deadlines when existing code is "almost right", novices copy-paste; in some cases the programmer is simply too lazy to de-clutter.

**Treatment**
- Same code in two or more methods of the **same class**: use **Extract Method** and call it from both places.
- Same code in two **subclasses of the same level**:
  - Use **Extract Method** for both classes, followed by **Pull Up Field** for the fields used.
  - If the duplicate is inside a constructor, use **Pull Up Constructor Body**.
  - If the code is similar but not completely identical, use **Form Template Method**.
  - If two methods do the same thing with different algorithms, pick the best and apply **Substitute Algorithm**.
- Duplicate code in two **different classes**:
  - If the classes aren't part of a hierarchy, use **Extract Superclass** to create a single superclass preserving all functionality.
  - If a superclass is difficult or impossible, use **Extract Class** in one class and use the new component in the other.
- If many conditional expressions perform the same code (differing only in their conditions), merge them with **Consolidate Conditional Expression** and use **Extract Method** to give the condition an easy-to-understand name.
- If the same code runs in all branches of a conditional, move it outside the condition tree with **Consolidate Duplicate Conditional Fragments**.

**Payoff**
- Merging duplicate code simplifies the structure and makes it shorter.
- Simplification + shortness = code that's easier to improve and cheaper to support.

**When to Ignore**
- In very rare cases, merging two identical fragments can make the code less intuitive and obvious.

### Data Class
A class that holds only fields and crude accessors, with no behavior.

**Signs and Symptoms**
- A class contains only fields and crude methods for accessing them (getters and setters). It's simply a container for data used by other classes — it has no additional functionality and can't independently operate on the data it owns.

**Reasons for the Problem**
- It's normal for a newly created class to contain only a few public fields (and maybe a handful of getters/setters). But the true power of objects is that they can contain behavior — operations on their data.

**Treatment**
- If a class contains public fields, use **Encapsulate Field** to hide them and require access via getters and setters only.
- Use **Encapsulate Collection** for data stored in collections (such as arrays).
- Review the client code that uses the class — you may find functionality that belongs in the data class itself. If so, use **Move Method** and **Extract Method** to migrate it into the data class.
- After filling the class with well-thought-out methods, get rid of old data-access methods that give overly broad access. **Remove Setting Method** and **Hide Method** may help.

**Payoff**
- Improves understanding and organization: operations on particular data are gathered in one place instead of scattered.
- Helps you spot duplication of client code.

### Dead Code
A variable, parameter, field, method, or class is no longer used.

**Signs and Symptoms**
- A variable, parameter, field, method, or class is no longer used (usually because it's obsolete).

**Reasons for the Problem**
- When requirements changed or corrections were made, nobody had time to clean up the old code.
- Such code can also lurk in complex conditionals, when one branch becomes unreachable (due to error or other circumstances).

**Treatment**
- The quickest way to find dead code is to use a good IDE.
- Delete unused code and unneeded files.
- For an unnecessary class, apply **Inline Class** or **Collapse Hierarchy** if a subclass or superclass is used.
- To remove unneeded parameters, use **Remove Parameter**.

**Payoff**
- Reduced code size.
- Simpler support.

### Lazy Class
A class doesn't do enough to justify its existence.

**Signs and Symptoms**
- Understanding and maintaining classes always costs time and money. So if a class doesn't do enough to earn your attention, it should be deleted.

**Reasons for the Problem**
- Perhaps a class was designed to be fully functional but after refactoring has become ridiculously small.
- Or it was designed to support future development work that never got done.

**Treatment**
- Components that are near-useless should be given the **Inline Class** treatment.
- For subclasses with few functions, try **Collapse Hierarchy**.

**Payoff**
- Reduced code size.
- Easier maintenance.

**When to Ignore**
- Sometimes a Lazy Class is created to delineate intentions for future development. In this case, try to maintain a balance between clarity and simplicity in your code.

### Speculative Generality
Unused generality created "just in case".

**Signs and Symptoms**
- There's an unused class, method, field, or parameter.

**Reasons for the Problem**
- Sometimes code is created "just in case" to support anticipated future features that never get implemented. As a result, code becomes hard to understand and support.

**Treatment**
- For unused abstract classes, try **Collapse Hierarchy**.
- Eliminate unnecessary delegation of functionality to another class via **Inline Class**.
- For unused methods, use **Inline Method**.
- For methods with unused parameters, use **Remove Parameter**.
- Unused fields can simply be deleted.

**Payoff**
- Slimmer code.
- Easier support.

**When to Ignore**
- If you're working on a framework, it's reasonable to create functionality not used in the framework itself, as long as it's needed by the framework's users.
- Before deleting elements, make sure they aren't used in unit tests. This happens when tests need a way to get internal information from a class or perform special testing-related actions.

---

## Couplers

### Feature Envy
A method accesses another object's data more than its own.

**Signs and Symptoms**
- A method accesses the data of another object more than its own data.

**Reasons for the Problem**
- This smell may occur after fields are moved to a data class. If so, you may want to move the operations on that data to the class as well.

**Treatment**
Basic rule: **if things change at the same time, keep them in the same place.** Data and the functions that use it are usually changed together (though exceptions exist).
- If a method clearly should be moved, use **Move Method**.
- If only part of a method accesses another object's data, use **Extract Method** to move the part in question.
- If a method uses functions from several other classes, determine which class holds most of the data used and place the method there. Alternatively, use **Extract Method** to split the method into parts placed in different classes.

**Payoff**
- Less code duplication (if data-handling code is put in a central place).
- Better code organization (methods for handling data sit next to the actual data).

**When to Ignore**
- Sometimes behavior is purposefully kept separate from the class that holds the data. The usual advantage is the ability to dynamically change the behavior (see **Strategy**, **Visitor**, and other patterns).

### Inappropriate Intimacy
One class delves into the internal fields and methods of another.

**Signs and Symptoms**
- One class uses the internal fields and methods of another class.

**Reasons for the Problem**
- Keep a close eye on classes that spend too much time together. Good classes should know as little about each other as possible — such classes are easier to maintain and reuse.

**Treatment**
- The simplest solution is **Move Method** and **Move Field** to move parts of one class into the class where they're used — but only if the first class truly doesn't need those parts.
- Another solution is **Extract Class** and **Hide Delegate** to make the code relations "official".
- If the classes are mutually interdependent, use **Change Bidirectional Association to Unidirectional**.
- If the "intimacy" is between a subclass and superclass, consider **Replace Delegation with Inheritance**.

**Payoff**
- Improves code organization.
- Simplifies support and code reuse.

### Incomplete Library Class
A library doesn't provide what you need, and you can't change it.

**Signs and Symptoms**
- Sooner or later, libraries stop meeting user needs. The only solution — changing the library — is often impossible since the library is read-only.

**Reasons for the Problem**
- The author of the library hasn't provided the features you need, or has refused to implement them.

**Treatment**
- To introduce a few methods to a library class, use **Introduce Foreign Method**.
- For big changes to a class library, use **Introduce Local Extension**.

**Payoff**
- Reduces code duplication — instead of creating your own library from scratch, you can piggyback off an existing one.

**When to Ignore**
- Extending a library can generate additional work if the changes to the library involve changes in code.

### Message Chains
A series of calls like `a.b().c().d()`.

**Signs and Symptoms**
- In code you see a series of calls resembling `$a->b()->c()->d()`.

**Reasons for the Problem**
- A message chain occurs when a client requests another object, that object requests yet another, and so on. These chains mean the client is dependent on navigation along the class structure — any change in these relationships requires modifying the client.

**Treatment**
- To delete a message chain, use **Hide Delegate**.
- Sometimes it's better to think about why the end object is being used. Perhaps it makes sense to **Extract Method** for that functionality and **Move Method** to move it to the beginning of the chain.

**Payoff**
- Reduces dependencies between classes of a chain.
- Reduces the amount of bloated code.

**When to Ignore**
- Overly aggressive delegate hiding can make it hard to see where functionality actually occurs — another way of saying, avoid the **Middle Man** smell as well.

### Middle Man
A class does nothing but delegate to another.

**Signs and Symptoms**
- If a class performs only one action — delegating work to another class — why does it exist at all?

**Reasons for the Problem**
- This smell can result from overzealous elimination of **Message Chains**.
- It can also result from the useful work of a class being gradually moved to other classes, leaving the class as an empty shell that does nothing but delegate.

**Treatment**
- If most of a class's methods delegate to another class, **Remove Middle Man** is in order.

**Payoff**
- Less bulky code.

**When to Ignore**
Don't delete a middle man that was created for a reason:
- A middle man may have been added to avoid interclass dependencies.
- Some design patterns create a middle man on purpose (such as **Proxy** or **Decorator**).
