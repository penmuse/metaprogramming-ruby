## Ruby on Steroids(External DSL): Template Engine in 50 lines.

In my previous post on DSL, I introduced you to DSL and the different types of DSL(internal & external). Today we are going to build a simple external DSL(a template engine) in 45 lines of code. This template engine will be used to render html views. It is similar to `erb`. The name of this template engine is `shot`. 

`shot` will enable us conveniently generate any kind of text, in any quantity, from templates. The templates themselves combine plain text with Ruby code for variable substitution and flow control, which makes them easy to write and maintain. Apart from generating html pages, it can also be used to produce XML documents, RSS feeds, source code, and other forms of structured text file.


### Template Structure

Our template will have 2 main markup elements

HTML and text use no markers and appear plainly on the page
`{{` and `}}` wrap Ruby code whose return value will be output in place of the marker
`@` wrap Ruby code whose return value will `not` be output

Basically, we can create files like this:

```ruby
<html>
  <body>
    <h1>{{ title }}</h1>
    @ posts.each do |post|
      <article> {{ post }} </article>
    @ end
  </body>
</html>
```

And it will generate:

```ruby
<html>
  <body>
    <h1>Shot Template Engine</h1>
      <article> First Post </article>
      <article> Second Post </article>
      <article> Third Post </article>
  </body>
</html>
```
If title is `"Shot Template Engine"` and posts is `["First Post", "Second Post", "Third Post"]`
Open lib/zucy/shot/template.rb and paste this code.

### Building Out the Engine

Create a new gem with `bundle gem shot` command. This will setup the boilerplate code for the template engine gem. Open `lib/shot.rb` and paste this code.

```ruby
module Shot
  class Template
    def initialize(template, scope: Object.new)
      template = File.read(template) if template.end_with?(".shot")
      @template_lines = template.split("\n")
      @scope = scope
    end

    def render(locals = {})
      block = build_lambda(locals)
      parsed_result = @scope.instance_eval(block)
      return parsed_result.call unless block_given?
      parsed_result.call { yield }
    end

    private

    def build_lambda(locals)
      <<-block
        lambda do
          parsed_lines = []
          #{local_variables(locals)}
          #{parse_lines}
          parsed_lines.join("\n")
        end
      block
    end

    def local_variables(locals)
      locals.map do |key, value|
        value = "\"#{value}\"" if value.is_a?(String)
        "#{key} = #{value}"
      end.join("\n")
    end

    def parse_lines
      @template_lines.map do |line|
        if line =~ /^\s*@(.*?)\s*$/
          line.gsub(/^\s*@(.*?)\s*$/, '\1')
        else
          "parsed_lines << \"#{line.gsub(/{{([^\r\n]+)}}/, '#{\1}')}\""
        end
      end.join("\n")
    end
  end
end
```
This library accepts any string or any file with `shot` extension as a template, and imposes no limitations on the source of the template. You may define a template entirely within your code, or store it in an external location and load it as required. This means that you can keep templates in files, SQL databases, or any other kind of storage that you want to use.

The template rendering is handled by the `render` method. There is a lot going on in the render method. Let's break it down.

`build_lambda` method builds a lambda string from the template array.
`local_variables` method uses the hash passed as locals(when render method is called) and uses it to create local variables in the lambda string.
`parse_lines` loops through the template string and checks if any line contains `@` or `{{ }}`. If it does, it performs the necessary substitution.
`scope.instance_eval(block)` evaluates the lambda string in the context of scope object. This method will return an actual lambda.

### Test the Engine.

Open spec/shot_spec.rb and paste this code.

```ruby
require 'spec_helper'

describe "Template Engine" do
  context "plain template string" do
    subject do
      Shot::Template.new("This is shot template engine").render
    end
    it { is_expected.to eq "This is shot template engine" }
  end

  context "string with conditional" do
    subject do
      template = (<<-template).gsub(/  /, "")
        @if true
          Zucy
          This is shot
        @else
          Another template engine
        @end
      template
      Shot::Template.new(template).render
    end
    it { is_expected.to eq "Zucy\nThis is shot" }
  end

  context "nil variable" do
    subject do
      Shot::Template.new("{{ name }}").
        render name: nil
    end
    it { is_expected.to eq "" }
  end

  context "comment" do
    subject do
      template = (<<-EOT).gsub(/  /, "")
      Awesome
      @ # Comment is here
      Shot Template Engine
      EOT
      Shot::Template.new(template).render
    end
    it { is_expected.to eq "Awesome\nShot Template Engine" }
  end

  context "html tags" do
    subject do
      template = (<<-EOT)
        <html>
          <head>
            <title>{{ title }}</title>
          </head>
        </html>
        EOT
      Shot::Template.new(template).
        render(title: "Zucy Framework")
    end

    let(:compiled_template) do
      (<<-EOT)
        <html>
          <head>
            <title>Zucy Framework</title>
          </head>
        </html>
        EOT
    end
    it { is_expected.to eq compiled_template.chomp }
  end

  context "html file" do
    subject do
      file = File.join(__dir__, "todo.html.shot")
      Shot::Template.new(file).render(title: "Shot Template Engine")
    end

    let(:compiled_template) do
      "<html>\n  <head>\n    <title>Shot Template Engine</title>\n  </head>\n</html>"
    end
    it { is_expected.to eq compiled_template }
  end
end
```
Try running the test using:

$ rspec

### Final Thoughts

There are a lot of DSLs out there. By rebuilding some of them, we can demystify some of the mysteries around them. We can also learn to appreciate them more since we now understand how they are built. In subsequent series, we will be rebuilding some popular DSLs like `RSpec`, `Cucumber`, `ActiveRecord Migrations` etc. Find the full source code of the gem [here](https://github.com/andela-iokonkwo/shot) on github.