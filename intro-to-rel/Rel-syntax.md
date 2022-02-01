# Rel Onboarding: Rel Syntax

Welcome to the Rel workshop!

In this class we'll be learning the basics of the Rel language.
The focus here is on the Rel syntax and not so much on the business impact.

Outline:
- Data types
- Tuples
- Relations
- Persisting Relations
- Relation Abstraction
- Relation Application

To get started:
- [Rel Tutorials](https://docs.relational.ai/getting-started/rel/overview/): Introductory tutorials to Rel
- [Rel Primer](https://docs.relational.ai/rel/primer/overview/): Comprehensive Guide to all key Rel modelling and syntax concepts.
- [Docs Walkthrough](https://docs.relational.ai/getting-started/intro/walkthrough/): Guide to help you work through our documentation.


## Data Types

We have all the basic data types:
- strings: `"abc"`
- numbers: `1`, `-0.2`, ....
- dates: `2022-02-01`, ...
- ...

You can see all of the data types (including the ones we are not covering in this workshop) in our [Data type reference guide](https://docs.relational.ai/rel/ref/data-types/).

```rel  @show output @query 
def output = 1
```

The statement `def output = ...` defines what result are shown.

```rel  @show output @query 
// string values
def output = "Alice"
```

The math operators `+-*/` work as you'd expect.

```rel  @show output @query 
def output = 15 + 17
```

## Tuples

Rel has tuples.
**Tuples are ordered collections of data values.**
A tuple may hold none, one, or many data values. 

Hint: You may think of tuples as rows in a table.

```rel  @show output @query 
// Here's a tuple!
def output = ("Dylan", "Bob")
```



```rel  @show output @query 
// you can have different data types in the same tuple
def output = ("Alex", 5)
```



```rel  @show output @query 
// you can have multiple values in the same tutple, since it's ordered
def output = (1, 1, 1, 1, 1, 1, 1, 1, 1)
```

## Relations (The Heart of Rel)

Technically, a relation is a set of tuples.

Reminder: A set is an unordered collection of unique entries.

Multiple entries in a relation are delimited with **semicolons**. You can also wrap the whole set in braces (`{}`).

A relation can be defined with `def R = ...` where `R` is the name of the relation.

As you noticed, we already used relations quite heavily with `def output = `.
The relation `output` has a special purpose as it informs the RKGMS which data to return.

```rel  @show output @query 
def age = {("Adam", 24); ("Adam", 24); ("Eve", 30)}
def output = age
```

In Rel, we don't need to state the data type in a relation.
Relations can have multiple data types in the same position.

```rel  @show output @query 
def age = {("Adam", 25); ("Eve",  Year[30]);}
def output = age
```

Relations can even consists of tuples that have length of tuples (aka arities).

```rel  @show output @query 
def today = {
    (2022-02-03);
    (2022, 02, 03)
}

def output = today
```


Multiple definition statements get unioned *not overwritten*.

```rel  @show output @query 
// sets are unique. Adding the value twice doesn't add it twice.
def name = {"Alice"; "Bob"}

// If you define the same value twice, the definitions are added together.
def name = {"Alice"; "Carol"}

// notice how "Alice" and "Bob" each show in the output ONCE.
def output = name
```

### Checking a tuple is in a relation.

```rel  @show output @query 
def student_class = {
  ("Alice", "Bio");
  ("Bob", "Bio");
  ("Carol", "Math")
}

// Does Alice have a Bio class?
def output = student_class("Alice", "Bio")
```



```rel  @show output @query 
def student_class = {
  ("Alice", "Bio");
  ("Bob", "Bio");
  ("Carol", "Math")
}

//If the answer is FALSE, then there's no output.
def output = student_class("Bob", "Math")
```

### Any value (existential quantification) 


We can use an underscore `_` for specifying "any value".
In DB-speak that is known as existential quantification.

This is useful when asking, for instance, did anyone took Bio?

```rel  @show output @query 
def student_class = {
  ("Alice", "Bio");
  ("Bob", "Bio");
  ("Carol", "Math")
}

def output = student_class(_, "Bio")
```

The underscore can be also used for any element in the relation.

For instance, let's assume we have a arity 3 (tuples with 3 elements) relation `grades` that connects students with it a class and their grade in that class.

```rel  @show output @query 
// underscore is useful if you have a larger relation.
def score = {
  ("Alice", "Bio", 70);
  ("Alice", "Math", 80)
}

// Does Alice have a 70 score in any class?
def output = score("Alice", _, 70)
```

## Installing Relations

Hmm, why do we need to define `student_class` every time?

Answer: So far we only used *query* cell, which can be used to define queries and complex logic  to extract knowledge from the database but these definitions are forgotten afterwards.

To make relation definitions persistent, we need to _install_ them!
We call this installing a model.
In the notebook we have dedicated _install cells_ to install models.

Side note: There are exists also EDB relations, whose purpose is to store the actual data.
However, introducing them is beyond the scope of this workshop.

IDB relations -- what we are using here -- are known as _views_ that is logic build on top of EDB data.
Out of convenience, we use logic to define our toy data and "store" them in IDB relations.

So, let's install some logic.

```rel  @name takes_class @install 
def takes_class = {
  ("Alice", "Bio");
  ("Bob", "Bio");
  ("Carol", "Math")
}
```

Now `takes_class` is installed, and we can refer to it in any other cell (even in a cell above).

```rel  @show output @query 
def output = takes_class("Alice", "Bio")
```

## Relational Abstraction

Once we have relations, we want to be able to do things with it!

```rel  @show output @query 
def bio_classroom = x: takes_class(x, "Bio")

def output = bio_classroom
```

This is equivalent to 

```rel  @show output @query 
def bio_classroom(x) = takes_class(x, "Bio")

def output = bio_classroom
```


### Formula

We can also rewrite the above by moving the abstraction to the left-hand side. Almost like it's a programming function!

If you do this, the right-hand side **must** evaluate to `true` or `false`.
It can't evaluate to a relation, that's not allowed.

```rel  @show output @query 
def classrooms(class, person) = school_class(person, class)

def output = classrooms
```


```rel  @show output @query 
// TODO maybe we shouldn't discuss this?
// You can't do this, though:
def classrooms("bio") = "alice"

def output = add(2, 3)
```

## Unbound Relations
## Application

This is where we introduce `[]`

```rel  @show output @query 
/*

def bio_classroom = person: classes(person, "bio")
def output[:a] = classrooms

def mutuals = x, y: exists(z: classes(x, z) and classes(y, z))

def output[:b] = mutuals

def scores = {
  ("alice", "bio", 10);
  ("bob", "bio", 9);
  ("carol", "bio", 12);
  ("alice", "econ", 11);
}

def incomplete(person, class) = classes(person, class) and not scores(person, class, _)
// Do you have a score in a class you didn't take?
//def output[:c] = class: top[2, (scores[user, class], user from user)]
//def output[:c] = user: top[2, (scores[user, class], class from class)]
//def output[:c] = user: top[2, (s, c: scores(user, c, s))]
def output[:d] = c, top[2, scores[u, c]] from u, c

//def output[:c] = top[3, (scores[user, class], user, class from user, class)]

/*
This 
def classes = {
	(1, :person, "alice");
    (1, :class, "bio");
    (1, :score, 11);
}

def classes[1] = {
	(:person, "alice");
    (:class, "bio");
    (:score, 11);
}

def incomplete(person, class) = classes(class, :person, person) and not classes(classes, :score, _)
 */
```


## The Nitty-Gritty

### Tuples

The length of a tuple is called _arity_.

The arity of `(2, 2, 2)` is 3.

```rel  @show output @query 
def output = arity[(2,2,2)]
```

Tuples can *not* be collections of tuples.
Tuples of tuples will be flattened into a tuple of values.

```rel  @show output @query 
def output = ((1,2), (3, 4))
```

A tuple of one value behaves like ... a value!

```rel  @show output @query 
def output = (1, ) + 2
```

The empty tuple is special and represents `true`.

```rel  @show output @query
def output = ()
```



### Relation

A tuple is a relation of cardinality 1 (relation has only one tuple in the collection).

```rel  @show output @query 
def R = ("a", "b")
def output = R
```

A data value is a relation of cardinality 1 and arity 1.

```rel  @show output @query 
def R = {(1,)}
def output = R + 2
```

As we can see: **everything is a relation**.
Relations within tuples get expanded into a relation of tuples.


#### Relations within Relations get expanded

Relations within tuples get expanded into a relation of tuples.

```rel  @show output @query 
def output = (1, {2; 3})
```

We can verify that when explicitly writing it as a set of tuples.

```rel  @show output @query 
def output = {(1, 2) ; (1, 3)}
```


You also cannot put a relation in another relation.
If you try, Rel will "expand" the relation in a set of tuples.

```rel  @show output @query 
def R = {1; 2; 3}
def S = {"a"; "b"; "c"}
def T = {R; S}

def output = T
```
