# Anatomy of Variables
This post intends to look into what variables stand for, starting from what they are to how they can be used.

By the end of this post, you should be able to:

 * Explain what variables are.
 * Identify the different types of variables in ruby.
 * Explain what makes one variable different from one another.


We will start from what variables are in programming generally, then sight some examples in some programming languages and finish off on **Ruby Variables**

## What are Variables?
**Variables**, as defined by [wikipedia](https://en.wikipedia.org/wiki/Variable_(computer_science)) " are storage locations paired with an associated symbolic name, ***an identifier***, which contains some known or unknown quantity of information referred to as a value." In other words, a variable can be referred to as a ***container*** that holds values or information. The variable's name represents what information the variable contains. They are called variables because the represented information can change but the operations on the variable remain the same.

Variables can be likened to Buckets where information can be kept while on the outside of the buckets are names. When referring to one of the buckets, we use the name of the bucket, not the data stored in the bucket.

Variables are the same across all computer programming languages. What differentiates them is the way they are used and how they are been assigned.

##Types of Variables
There are two general types of variables. They are:

* Local Variables
* Global Variables

**Local Variables**

These are variables that are assigned within a function or a method. They can be used only within the method or function in which they have been assigned.

```ruby
def counter
    count = 0
    while count <= 10
        puts count
        count += 1
    end
end

count #=> 0 1 2 3 4 5 6 7 8 9 10
```

This is a ruby method named `counter`. A local variable `count` was assigned in this method and it was used to print `0..10`. However if you call count outside this method, it would throw a `method or variable not defined` error.

Here is the JavaScript version of the above code:

```JavaScript
function counter(){
    var count = 0;
    while (count <= 10) {
       console.log(count);
        count ++;
    }
}

counter(); //=> 0 1 2 3 4 5 6 7 8 9 10
```

It is important to note that arguments passed into a method or function, are also treated as local variables. For example, the code snippet below works exactly as the one above.

```ruby
def counter(count)
    while count <= 10
        puts count
        count += 1
    end
end

counter(0); #=>0 1 2 3 4 5 6 7 8 9 10
```

Here is the JavaScript version of the code above.

```JavaScript
function counter(count){
    while (count <= 10) {
       console.log(count);
        count ++;
    }
}

var count = 0;
counter(count); //=>0 1 2 3 4 5 6 7 8 9 10
```

**Global Variables**

These are variables that can be used by two or more methods or function. These variables are assigned outside the methods or functions and they can be manipulated within methods or functions.

The example below shows how global variables are assigned and used in Ruby and JavaScript.

```ruby
@count = 0

def counter
    while @count <= 10
        puts @count
        @count += 1
    end
end

counter #=>0 1 2 3 4 5 6 7 8 9 10
```


```JavaScript
var count = 0;

function counter(count){
    while (count <= 10) {
       console.log(count);
        count ++;
    }
}

counter(); //=>0 1 2 3 4 5 6 7 8 9 10
```

You will notice that `count` was assigned outside the function or method, and it was incremented within the function or method. Also, we need not pass `count` as an argument when calling the method or function.


Based on these two types of variables, Ruby has other types of variables. They are:

* Instance Variables
* Class Variables
* Class Instant Variables
* Constants

**Instance Variables**

Instance Variables are global variables that are **ONLY** used in **Instance Methods** within a class. Instance variables in ruby start with a single  `@` sign. Any variable assigned starting with a single `@` sign can be manipulated within an instance method. Just in case, you don't know what an instance method is in ruby, let's take a quick look.

Ruby instance methods are defined with the keyword `def`, then followed by the name of the method and ended with the keyword `end`. Basically, the common methods in ruby are instance methods. These methods can only be called on an instance of the class they belong to.
The example below shows an instance variable been used in an instance method.

```ruby
class User
    def initialize
        @name = "Adebayo"
    end

    def greet
        puts "Hello #{@name}"
    end
end

User.new.greet #=> Hello Adebayo
```

The above example has instance variable `@name` which holds a name `Adebayo`. The instance method `greet` adds `Hello` to the value of `@name`.  This tells us that instance variables act as global variables **within** a class.  

**Class Variables**

Class variables work exactly like instance methods. These varibales are denoted with a double `@` sign, i.e `@@`. These variables are  **ONLY** used in **Class Methods** within a class. Just in case, you don't know what a class method is in ruby, let's take a quick look.

The keyword `self` makes a class method quite diferent from an instance method. A class method is also defined by the `def` keyword, then the keyword `self` is prpended to the method name. For example, if we were to rewrite the above example as a class method, the method `greet` will be written as `self.greet`. Class methods, however, can be called directly on the class they belong to. The example below shows a class variable been used in a class method.

```ruby
Class User
    @@name = Adebayo

    def self.name
         puts "Hello #{@@name}"
    end
end

User.name  #=> "Hello Adebayo"
```

**Class Instance Variables**

These variables are a combination of the two we have discussed earlier. They are used as global variables but they are assigned as local variables. This means they are assigned in a class method like instance variables but they can be used in any class methods within the same class.

The example below shows how class instance variables are used.

```ruby
class Person
    def self.username(name)
      @name = name
    end

    def self.greet
      "Hello #{@name}"
    end
end

Person.username("Adebayo") #=> "Adebayo"
Person.greet #=> "Hello Adebayo"

```

In the snippet above, `name` method was used to get the person's name and it was assigned to `@name` variable. This variable, however, can be used in any other class method within that class.

One important thing to know is that this kind of variables will not be available in classes that inherit from the class in which they are assigned.

```ruby
class Property < Person
    def self.owner
        "I belong to #{@name}"
    end
end


Property.owner #=> "I belong to "
@name = nil
```
In other to use this vaiable in this new class, we have to redeclare the variable. A correct way to do this will be;

```ruby
class Property < Person
    def self.owner
        "I belong to #{@name}"
    end
end

Property.username("Adebayo") #=> "Adebayo"
Property.owner #=> "I belong to Adebayo"
```

## What makes these varables different from each other?

The major differences between these variables wrap around the scope at which they are to be used and the syntax for assigning them.

* The diference between an instance variable and a class instance variable is the scope at which they are been used.

    > Instance Variables are scoped to Instance Methods while Class Instance Variable are scoped to Class Methods.

* The diference between a Class Instance Variable and a Class Variable is the syntax of assinging them.

    > Class instance Variables are assigned with a single `@` symbol while Class Variables are assigned with double `@` symbol, i.e `@@`
