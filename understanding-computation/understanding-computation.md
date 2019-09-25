# Understanding Computation

[Understanding Computation](https://www.amazon.com/Understanding-Computation-Machines-Impossible-Programs-ebook/dp/B00CT3C4IM) by Tom Stuart.
  > _...learn computation theory and programming language design in an engaging, practical way. Understanding Computation explains theoretical computer science in a context you’ll recognise, helping you appreciate why these ideas matter and how they can inform your day-to-day programming._

## Chapter 1: Just Enough Ruby

_Our conversation moved through this pretty quickly as most of the group are Rubyists to some degree._

## Chapter 2: The Meaning of Programs

__Compared to Knuth’s stuff, this is more accessible, but we still struggled. It's the first technical book we've done and 
we need to find a format that works. We've agreed to spend another week re-reading Chapter 2 and especially working through
the code to see if we can find a clearer understanding and generate more to talk about.__

_There are a bunch of different ways of doing semantics: he’ll work through three._

We use programming languages to clarify complex ideas to ourselves, communicate those ideas to each other, and, most important, implement those ideas inside our computers.

But computer programming isn't really about programs, it's about ideas. A program is a frozen representation of an idea, a snapshot of a structure that once existed in a programmer's imagination. => meaning

In linguistics, semantics is the study of the connection between words and their meanings... In computer science, the field of formal semantics is concerned with finding ways to define meanings of programs and how to prove abstract concepts. (for example math equations)

Programming languages needs: syntax and semantics.

### Syntax

Every programming language comes with a collection of rules that describe what kind of character strings may be considered valid programs in that language; these rules specify the language's syntax.

### Operational semantics

The most practical way to think about the meaning of a program is what it does—when we run the program, what do we expect to happen?

### Big step semantics

For example, unlike Ruby, SIMPLE is a language that makes a distinction between expressions, which return a value, and statements, which don't.

If we added more elaborate features to the language—data structures, procedure calls, exceptions, an object system—we'd need to make many more design decisions and express them unambiguously in the semantic definition.

Small-step semantics has a mostly iterative flavour, requiring the abstract machine to repeatedly perform reduction steps.

The idea of big-step semantics is to specify how to get from an expression or statement straight to its result. This necessarily involves thinking about program execution as a recursive rather than an iterative process.

Big-step semantics is often written in a looser style that just says which subcomputations to perform without necessarily specifying what order to perform them in. Small-step semantics also gives us an easy way to observe the intermediate stages of a computation, whereas big-step semantics just returns a result and doesn't produce any direct evidence of how it was computed.

```rb
class Number
  def evaluate(environment)
    self
  end
end

class Add
  def evaluate(environment)
    Number.new(left.evaluate(environment).value + right.evaluate(environment).value)
  end
end

class Assign
  def evaluate(environment)
    environment.merge({ name => expression.evaluate(environment) })
  end
end

class DoNothing
  def evaluate(environment)
    environment
  end
end
```

By contrast, this big-step implementation makes much greater use of the stack, relying entirely on it to remember where we are in the overall computation, to perform smaller computations as part of performing larger ones, and to keep track of how much evaluation is left to do. What looks like a single call to #evaluate actually turns into a series of recursive calls, each one evaluating a subprogram deeper within the syntax tree.

It probably hasn't escaped your attention that, by writing down SIMPLE's small- and big- step semantics in Ruby instead of mathematics, we have implemented two different Ruby interpreters for it.

### Denotational semantics

So far, we've looked at the meaning of programming languages from an operational perspective, explaining what a program means by showing what will happen when it's executed. Another approach, denotational semantics, is concerned instead with translating programs from their native language into some other representation.

We've already seen operationally that an expression takes an environment and turns it into a value; one way to express this in Ruby is with a proc that takes some argument representing an environment argument and returns some Ruby object representing a value. For simple constant expressions like «5» and «false», we won't be using the environment at all, so we only need to worry about how their eventual result can be represented as a Ruby object. 

```rb
class Number
  def to_ruby
    "-> e { #{value.inspect} }"
  end
end
```

Each of these methods produces a string that happens to contain Ruby code, and because Ruby is a language whose meaning we already understand, we can see that both of these strings are programs that build `proc`s. Each proc takes an environment argument called `e`, completely ignores it, and returns a Ruby value.

At this stage, it's tempting to avoid procs entirely and use simpler implementations of #to_ruby that just turn `Number.new(5)` into the string '5' instead of `-> e { 5 }` and so on, but part of the point of building a denotational semantics is to capture the essence of constructs from the source language, and in this case, we're capturing the idea that expressions in general require an environment, even though these specific expressions don't make use of it.

```rb
class Variable
  def to_ruby
    "-> e { e[#{name.inspect}] }"
  end
end
```
This translates a variable expression into the source code of a Ruby proc that looks up the appropriate value in the environment hash.

*Denotation: the literal or primary meaning of a word, in contrast to the feelings or ideas that the word suggests.*

An important aspect of denotational semantics is that it's compositional: the denotation of a program is constructed from the denotations of its parts. We can see this compositionality in practice when we move onto denoting larger expressions like Add, Multi ply, and LessThan.

```rb
class Add
  def to_ruby
    "-> e { (#{left.to_ruby}).call(e) + (#{right.to_ruby}).call(e) }"
  end
end
```

Since statements return a modified environment, `Assign#to_ruby` needs to produce code for a `proc` whose result is an updated environment hash:

```rb
class Assign
  def to_ruby
    "-> e { e.merge({ #{name.inspect} => (#{expression.to_ruby}).call(e) }) }"
  end
end
```

To give this translation some explanatory power, it's helpful to bring parts of the language's meaning to the surface instead of allowing them to remain implicit. For example, this semantics makes the environment explicit by representing it as a tangible Ruby object—a hash that's passed in and out of procs—instead of denoting variables as real Ruby variables and relying on Ruby's own subtle scoping rules to specify how variable access works.

### Formal semantics in practice

For example, since an operational semantics corresponds quite closely to the implementation of an interpreter, computer scientists can treat a suitable interpreter as an operational semantics for a language, and then prove its correctness with respect to a denotational semantics for that language—this means proving that there is a sensible connection between the meanings given by the interpreter and those given by the denotational semantics.

Denotational semantics has the advantage of being more abstract than operational semantics, by ignoring the detail of how a program executes and concentrating instead on how to convert it into a different representation.

### Alternatives

Other styles of formal semantics are available. One alternative is axiomatic semantics, which describes the meaning of a statement by making assertions about the state of the abstract machine before and after that statement executes: if one assertion (the pre- condition) is initially true before the statement is executed, then the other assertion (the postcondition) will be true afterward. Axiomatic semantics is useful for verifying the correctness of programs: as statements are plugged together to make larger programs, their corresponding assertions can be plugged together to make larger assertions, with the goal of showing that an overall assertion about a program matches up with its intended specification. (RSpec)
