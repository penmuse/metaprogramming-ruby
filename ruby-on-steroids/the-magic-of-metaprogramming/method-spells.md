## Ruby on Steroids: The Magic of MetaProgramming - Method Spells

In part 1 of this journey, you were introduced to anatomy of magic(metaprogramming) and you saw some of the spells you can cast with the magic of metaprogramming. In this post, I will be showing you spells you can cast when dealing with methods.

### Method Spells

The spells we will be discussing on this journey are spells we will need when working with methods.

### 1. alias_method

One of the most startling features of Ruby is: **open classes**. Ruby’s open classes means that you can change the behavior of any class at any time. You can add new methods, you can also replace the code behind an existing method.

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

### 2. method_missing
```ruby
  class Person
    def name
      @name || "person"
    end
    
    def name=(val)
     return if val == "person"  
     @name = val
    end
  end
 
 person = Person.new
 person.name #=> "person"
 person.name = "Ikem"
 person.name #=> "Ikem"
 
 person.email #=> NoMethodError: undefined method `email' for #<Person:0x007fabb343b410>
```
In the example above, we created the class Person. All is fine when we called method `name` and `name=`. So what exactly happens when we called method email? Well Initially, Ruby will look for the email method in the Person class and failing to find it there, it will look for the email method in the superclass of Person class, and on up the inheritance tree. If Ruby finds the method anywhere in the inheritance tree, then that’s the method that gets called. When Ruby fails to find a method, it turns around and calls a second method. This second call, to a method with the somewhat odd name of method\_missing, is what eventually generates the exception: It’s the default implementation of method_missing, found in the Object class that raises the NoMethodError exception.

However, you are free to override method_missing in any of your classes and handle the case of the missing method yourself:

```ruby
  class Person
    attr_accessor :name
    
    def method_missing(method_name, *args, &block)
      if method_name == :email
        "person.email@gmail.com"
      else
          "Hey, you just called the #{method_name} method /
         With these arguments: #{args.join(' ')} /
         But there is no such method"
      end
    end
  end
 
 person = Person.new
 person.name = "Ikem"
 person.name #=> "Ikem"
 person.email #=> "person.email@gmail.com"
 
 person.address("lagos", "abuja") #=> "Hey, you just called the address method \n With these arguments: lagos abuja \n But there is no such method"
```



### 3. respond\_to_missing?
`respond_to?` is used to determine if an object responds to a method. It is often used to check that an object knows about a method before actually calling it, in order to avoid an error at runtime about a method existence. Since our object doesn't know about these methods but are handled by `method_missing`, we have to override a cousin method `respond_to_missing?`

To have a consistent API when using method\_missing, it’s important to implement a corresponding respond\_to_missing?.

```ruby
class Tree
  # Pretend this is a real implementation
  def find_node(conditions = {})
    "find the node by #{conditions.inspect}"
  end
  
  def method_missing(method_name, *arguments, &block)
    return super unless method_name.to_s =~ /^find_node_by_(.*)$/
    find_node($1.to_sym => arguments.first)
  end
end
  
tree = Tree.new
node = tree.find_node_by_name("root") #=> "find the node by {:name=>\"root\"}" 
tree.respond_to? :find_node_by_name #=> false

class Tree
  # ommitted
  
  # It's important to know Object defines respond_to to take two parameters: the method to check, and whether to include private methods
  def respond_to_missing?(method_name, include_private = false)
    return super unless method_name.to_s =~ /^find_node_by_(.*)$/
    true
  end
end

Tree.new.respond_to? :find_node_by_name #=> true

```

### 4. remove\_method & undef_method

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



### 5. send
`send` is used to invoke a method dynamically at runtime. `send` takes, as its first argument, the name of the method that you want to call. This name can either be a symbol or a string.

Why should you care about `send` method? Well, sometimes you do not know exactly what methods are going to be called. `send` method provides a generic way of calling any method. For example, in the method fees below, we have a lot of if/else statement. Notice that there is a direct correlation between the school parameter and the method that returns the fees for that school.

```ruby
class Student

  def law_fees
    21000
  end
  
  def physical_science_fees
    22000
  end
  
  def medicine_fees
    23000
  end
  
  def engineering_fees
    24c000
  end

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
  # ommitted

  def fees(school)
    send("#{school}_fees")
  end
  def method_missing(method_name)
    return super unless /^.+_fees$/ =~ method_name
    20000
  end  
end
```

### Conclusion
You have seen some of the magic you can perform on methods. From aliasing methods to catching missing methods to responding to missing methods to removing/undefining methods and finally to calling methods dynamically via `send`. These are very powerful tools, but with power comes great responsibilities. You shouldn't use them recklessly. Basically only use them if there are no `better` options. The same applies to other metaprogramming tools.



