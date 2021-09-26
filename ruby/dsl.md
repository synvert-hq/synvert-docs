---
layout: ruby
title: DSL
---

Synvert provides a simple dsl to define a snippet.

```ruby
Synvert::Rewriter.new "group_name", "snippet_name" do
  description "description"

  if_gem gem_name, gem_version

  within_file file_pattern do
    within_node rules do
      with_node rules do
        remove
      end
    end
  end

  within_files files_pattern do
    with_node rules do
      unless_exist_node rule do
        append code
      end
    end
  end
end
```

### description

Describe what the snippet does.

```ruby
description 'description of snippet'
```

### if\_ruby

Checks if current ruby version is greater than or equal to the
specified version.

```ruby
if_ruby '2.0.0'
```

### if\_gem

Checks the gem in `Gemfile.lock`, if gem version in `Gemfile.lock`
is less than, greater than or equal to the version in `if_gem`,
the snippet will be executed, otherwise, the snippet will be ignored.

```ruby
if_gem 'factory_girl', '= 2.0.0'
if_gem 'factory_girl', '~> 2.0.0'
if_gem 'factory_girl', '> 2.0.0'
if_gem 'factory_girl', '< 2.0.0'
if_gem 'factory_girl', '>= 2.0.0'
if_gem 'factory_girl', '<= 2.0.0'
```

### add\_file

Add a new file and write content.

```ruby
content = <<~EOS
  ActiveSupport.on_load(:action_controller) do
    wrap_parameters format: [:json]
  end
EOS
add_file 'config/initializers/wrap_parameters.rb', content
```

### remove\_file

Remove a file.

```ruby
remove_file 'config/initiliazers/secret_token.rb'
```

### within\_file / within\_files

Find files according to file pattern, the block will be executed
only for the matching files.

```ruby
within_file 'spec/spec_helper.rb' do
  # find nodes
  # check nodes
  # add / replace / remove code
end
```

```ruby
within_files 'spec/**/*_spec.rb' do
  # find nodes
  # check nodes
  # add / replace / remove code
end
```

### with\_node / within\_node

Find ast nodes according to the [rules][1], the block will be executed
for the matching nodes.

```ruby
with_node type: 'send', receiver: 'FactoryGirl', message: 'create' do
  # check nodes
  # add / replace / remove code
end
```

```ruby
with_node type: 'block', 'caller.receiver': 'FactoryGirl', message: 'create' do
  # check nodes
  # add / replace / remove code
end
```

### goto\_node

Go to the specified child code.

```ruby
with_node type: 'block' do
  goto_node :caller do
    # change code in block caller
  end
end
```

```ruby
with_node type: 'block' do
  goto_node 'caller.receiver' do
    # change code in block caller's receiver
  end
end
```

### if\_exist\_node

Check if the node matches [rules][1] exists, if matches, then executes
the block.

```ruby
if_exist_node type: 'send', receiver: 'params', message: '[]' do
  # add / replace / remove code
end
```

### unless\_exist\_node

Check if the node matches [rules][1] does not exist, if does not match,
then executes the block.

```ruby
unless_exist_node type: 'send', message: 'include', arguments: ['FactoryGirl::Syntax::Methods'] do
  # add / replace / remove code
end
```

### if\_only\_exist\_node

Check if the current node contains only one child node and the child
node matches [rules][1], if matches, then executes the node.

```ruby
if_only_exist_node type: 'send', receiver: 'self', message: 'include_root_in_json=', arguments: [false] do
  # add / replace / remove code
end
```

### append

Add the code at the bottom of the current node body.

```ruby
append 'config.eager_load = false'
```

### prepend

Add the code at the top of the current node body.

```ruby
prepend "include FactoryGirl::Syntax::Methods"
```

### insert

Insert the code at the beginning or end of the current node.

```ruby
insert '.first', at: 'end'
insert 'URI.', at: 'beginning'
```

### insert\_after

Add the code next to the current node.

```ruby
{% raw %}
secret = SecureRandom.hex(64)
insert_after "{{receiver}}.secret_key_base = '#{secret}'"
{% endraw %}
```

### replace\_with

Replace the current node with the code.

```ruby
{% raw %}
replace_with "create({{arguments}})"
{% endraw %}
```

### replace

Reaplce child node with the code.

```ruby
{% raw %}
replace :message, with: 'tr'
{% endraw %}
```

### remove

Remove the current node.

```ruby
with_node type: 'send', message: 'rename_index' do
  remove
end
```

### delete

Delete the child nodes

```ruby
with_node type: 'send', message: 'flatten' do
  delete :dot, :mesage
end
```

### wrap

Wrap the current node with a new code.

```ruby
with_node type: 'class', name: 'TestCase' do
  wrap with: 'module ActiveSupport'
end
```

### replace\_erb\_stmt\_with\_expr

Replace erb statemet code with expression code.

```ruby
with_node type: 'block', caller: { type: 'send', receiver: nil, message: 'form_for' } do
  replace_erb_stmt_with_expr
end
```

### warn

Don't change any code, but will give a warning message.

```ruby
warn 'Using a return statement in an inline callback block causes a LocalJumpError to be raised when the callback is executed.'
```

### add\_snippet

Add other snippet, it's easy to reuse other snippets.

```ruby
add_snippet 'rails', 'convert_dynamic_finders'
```

### redo\_until\_no\_change

Rerun the snippet until no change affected.

```ruby
redo_until_no_change
```

### helper\_method

Add a method which is available in the current snippet.

```ruby
helper_method :method1 do |arg1, arg2|
  # do anything you want
end

method1(arg1, arg2)
```

### todo

List somethings the snippet should do, but not do yet.

```ruby
todo <<~EOS
  Rails 4.0 no longer supports loading plugins from vendor/plugins. You
  must replace any plugins by extracting them to gems and adding them to
  your Gemfile. If you choose not to make them gems, you can move them
  into, say, lib/my_plugin/* and add an appropriate initializer in
  config/initializers/my_plugin.rb.
EOS
```

Check out the source code of DSLs here: [this][2] and [that][3]

[1]: /ruby/rules/
[2]: https://github.com/xinminlabs/synvert-core-ruby/blob/master/lib/synvert/core/rewriter.rb
[3]: https://github.com/xinminlabs/synvert-core-ruby/blob/master/lib/synvert/core/rewriter/instance.rb
