## Ruby on Steroids(DSLs): The Powerful Spell Called DSL

Software engineering is all about trade-offs. There rarely is a “best” or “correct” solution to programming problems. The typical general-purpose programming language is good at solving a huge range of problems. Unfortunately, there is a price to be paid for being general purpose. A language that tries to do everything can’t afford to be great at any one thing. Domain specific languages, or DSLs for short, strike a different sort of bargain.

### What are DSLs

According to Martin Fowler:

> The basic idea of a domain specific language (DSL) is a computer language that's targeted to a particular kind of problem, rather than a general purpose language that's aimed at any kind of software problem. Domain specific languages have been talked about, and used for almost as long as computing has been done.

Basically, DSLs are mini-programming language of limited expressiveness focused on solving a particular problem. Since DSLs are very specific, it solves any problem in it's domain very efficiently. 

### Types of DSLs

 There are two major types of DSLs: Internal and External DSLs. DSLs come in two forms – external and internal. External DSL’s exist independently from any other language, SQL, SASS are good examples of external DSLs. Internal DSL’s on the other hand live inside another programming language – for example RSpec is an internal DSL which is hosted within the Ruby programming language.
                                                                
Typically internal DSLs are easier to create but aren’t as flexible as external DSL’s. Internal DSL’s need not worry about parsing or grammars but must conform to valid syntax within the host language (eg. all RSpec code is valid Ruby syntax), conversely an external DSL can have any syntax its creator wishes at the cost more work to build a parser and grammar. Ruby, with its “the programmer is always right” feature set and very flexible syntax, makes a great platform for building internal DSLs.

### DSLs In the wild

What makes rails very flexible and magical is the use of internal DSLs in a lot of places. Let's take a look at the different internal DSLs used in rails.
#### 1. Rails Routing

```ruby
Rails.application.routes.draw do
  root 'home#index'
  resources :events, only: [:index, :show]
end
```
#### 2. Rake Tasks

```ruby
namespace :fake do
  desc "Generate Fake Data
  task :data => :environment do
    generate_fake
  end
end
```

#### 3. DB Migrations

```ruby
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :first_name
      t.string :last_name
      t.string :email
      t.timestamps null: false
    end
  end
end
```

#### 4. ActiveRecord Validations

```ruby
class Person < ActiveRecord::Base
  validates :email_confirmation, presence: true
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 1000,
    too_long: "%{count} characters is the maximum allowed" }
end
```
#### 5. RSpec

Not really part of rails but used a lot of times to test rails applications.
 
```ruby
describe Array do
  describe "includes_subset?" do
    it "finds subsets" do
      a = [1,2,3,4,5]
      b = [1,2]
      expect(a.includes_subset?(b)).to eq(true)
    end
  end
end
```

As you can see, Rails uses internal DSLs to solve very specific problems with the framework. 

### Creating our own DSL: mark_ruby

`mark_ruby` is a Ruby DSL that generates html. Below is the syntax.

```ruby
mark_ruby do
  body id: :body_id do
    div id: :container do
      ul class: :content do
        li "Item 1", class: :active
        li "Item 2"
      end
    end
  end 
end
```
Which generates something like this.

```ruby
<html>
  <body id="body_id"_>
    <div id="container">
      <ul class="content">
        <li class="active">Item 1</li>
        <li>Item 2</li>
      </ul>
    </div>
  </body>
</html>
```

Below is the actual DSL code. There are a lot metaprogramming magic happening here. Check out the `Ruby on Steroids` series if you need a refresher on metaprogramming. Most Ruby Internal DSLs are powered by metaprogramming and blocks. If you cna master both, then you have all the tools you need to build internal DSL

```ruby
class MarkRuby
  def initialize(&block)
    @curent_indentation = 0
    @html = ""
    html(&block)
  end
  
  def method_missing(method_name, *args, &block)
    text = args.shift if args[0].is_a? String
    html_tag(method_name, text, *args, &block)
  end
  
  def to_s
    @html
  end

  private
  
  def html_tag(value, text = "", options = {}, &block)
    @html << "\n<#{value}#{attributes options}>#{text}"
    if block_given?
      @curent_indentation += 1
      instance_eval(&block)
      @curent_indentation -= 1
      @html << "\n#{indent}"
    end
    @html << "</#{value}>"
  end

  def attributes(options)
    if options.size > 0
      " " + options.map { |key, value|  "#{key}=\"#{value}\"" }.join(" ")
    end
  end

  def indent
    "  " * @curent_indentation
  end
end

module Kernel
  private def mark_ruby(&block)
    MarkRuby.new(&block).to_s
  end
end
```

### Final Thoughts

DSLs are very powerful spells. They are are best-suited:
                   
- When tasks require a lot of repeated, abstractable functionality, such as handling migrations or creating a rake task
- In situations where source should be readable by people besides the developers (Chef, Cucumber, RSpec to a degree)
- When we need a language that makes it easier to get stuff done as a programmer, and not more complex (all of the DSLs we have seen so far)
- In situations where we want to create a highly modular API with focus on DRYness and flexibility.
- When we want to tackle specific problems with an expressive language.

Ruby's flexibility encourages the creation of DSLs. Ruby has a lot of inbuilt DSL and so do rails. You should take advantage of ruby's flexibility and build your own DSL when the situation presents itself.