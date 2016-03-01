## Ruby on Steroids: The Magic of MetaProgramming - Method Spells

In part 1 of this journey, you were introduced to anatomy of magic(metaprogramming) and you saw some of the spells you can cast with the magic of metaprogramming. In this post, I will be showing you spells you can cast when dealing with methods
Method Spells

The spells we will be discussing on this journey are spells we will need when working with methods.

### 1. alias_method

One of the most startling features of Ruby is: **open classes**. Ruby’s open classes means that you can change the behavior of any class at any time. You can add new methods. You can also replace the code behind an existing method.

So what if you want to modify an existing method, but you still want to use the original method in the future? Well, that's where `alias_method` comes in. It is used for renaming methods in ruby. Let's see how it works.

```ruby
class Person
  def name
    "Name is Aboki"
  end

  alias_method :aboki_name, :name

  def name
    "Ikem Okonkwo"
  end
end

person = Person.new
person.name #=> "Ikem Okonkwo"
person.aboki_name #=> "Name is Aboki"
```

### 2. remove\_method & undef_method

To remove existing methods, you can use the remove_method within the scope of a given class. If a method with the same name is defined for an ancestor of that class, the ancestor class method is not removed. The undef_method, by contrast, prevents the specified class from responding to a method call even if a method with the same name is defined in one of its ancestors.

```ruby
class Person
  def method_missing(m, *args, &block)
    puts "Method Missing: Called #{m} with #{args.inspect} and #{block}"
  end

  def hello
    puts "Hello from class Person"
  end
end

class Fellow < Person
  def hello
    puts "Hello from class Fellow"
  end
end

obj = Fellow.new
obj.hello # => Hello from class Fellow

class Fellow
  remove_method :hello  # removed from Fellow, but still in Person
end
obj.hello # => Hello from class Person

class Fellow
  undef_method :hello   # prevent any calls to 'hello'
end
obj.hello # => Method Missing: Called hello with [] and
```

### 3. method_missing



### 4. respond\_to_missing

### 5. send
`send` is used to invoke a method dynamically at runtime. `send` takes, as its first argument, the name of the method that you want to call. This name can either be a symbol or a string.

Why should you care about `send` method? Well, sometimes you do not know exactly what methods are going to be called. `send` method provides a generic way of calling any method. For example, in the method fees below, we have a lot of if/else statement. Notice that there is a direct correlation between the school parameter and the method that returns the fees for that school.

```ruby
class Student
  def fees(school)
    if school == "engineering"
      engineering_fees
    elsif school == "physical_science"
      physical_science_fees
    elsif school == "law"
      law_fees
    elseif school == "medicine"
      medicine_fees
    else
      20000
    end
  end
end
```
Also notice that as we add new schools, the method `fees` grows larger and larger. We can easily refactor fees method with send method. Let's find out how.

```ruby
class Student
  def fees(school)
    send("#{school}_fees")
  end
  def method_missing(method_name)
    return super unless /^.+_fees$/ =~ method_name
    20000
  end
end
```




