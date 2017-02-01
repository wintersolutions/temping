# Temping

[![Code Climate](https://codeclimate.com/github/jpignata/temping.png)](https://codeclimate.com/github/jpignata/temping)
[![Build Status](https://travis-ci.org/jpignata/temping.png?branch=master)](https://travis-ci.org/jpignata/temping)
[![Gem Version](https://badge.fury.io/rb/temping.png)](http://badge.fury.io/rb/temping)


## Description

Temping allows you to create arbitrary ActiveRecord models backed by a temporary
SQL table for use in tests. You may need to do something like this if you're
testing a module that is meant to be mixed into ActiveRecord models without
relying on a concrete class.

Temping will use your existing database connection. As we're using temporary
tables all data will be dropped when the database connection is terminated.

## Examples

The basic setup of a model involves calling _create_ with a symbol that
represents the class name of the model you wish to create. By default,
this will create a temporary table with an _id_ column.

```ruby
Temping.create :dog

Dog.create => #<Dog id: 1>
Dog.table_name => "dogs"
Dog => Dog(id: integer)
```

Keep in mind, the table name will always be pluralized, while the class name will be singular.

```ruby
Temping.create :dogs

Dog.table_name => "dogs"
Dogs => NameError: uninitialized constant Dogs
```

Additional database columns can be specified via the _with_columns_ method
which uses Rails migration syntax:

```ruby
Temping.create :dog do
  with_columns do |t|
    t.string :name
    t.integer :age, :weight
  end
end

Dog.create

# => #<Dog id: 1, name: nil, age: nil, weight: nil>
```

When a block is passed to _create_, it is evaluated in the context of the class.
This means anything you do in an ActiveRecord model class body can be
accomplished in the block including method definitions, validations, module
includes, etc.

```ruby
Temping.create :dog do
  validates :name, presence: true

  with_columns do |t|
    t.string :name
    t.integer :age, :weight
  end

  def quack
    "arf!"
  end
end

Dog.create!

# => ActiveRecord::RecordInvalid: Validation failed: Name can't be blank

codey = Dog.create! name: "Codey"
codey.quack

# => "arf!"
```

All attributes you can pass to `create_table` can be evaulated too. For example you can create a dog with a primary key of the type uuid:

```ruby
Temping.create :dog, id: :uuid, default: -> { 'uuid_generate_v4()' }

Dog.create

# => #<Dog id: d937951b-765c-4bc9-804e-3171d22117b0>
```

## Installation

In your Gemfile:

```ruby
gem "temping"
```

In `test_helper.rb` add the following block to `ActiveSupport::TestCase`:

```ruby
class ActiveSupport::TestCase
  # ...
  teardown do
    Temping.teardown
  end
  # ...
end
```

Or, if you're using `rspec`, in `spec_helper.rb` add the following block to `RSpec.configure`:

```ruby
RSpec.configure do |config|
  # ...
  config.after do
    Temping.teardown
  end
  # ...
end
```

Alternatively you may want to just cleanup tables, but keep defined models:

```ruby
Temping.cleanup
```

## Bugs, Features, Feedback

All contributions are welcome! Please take a look at `CONTRIBUTING.md` for some
tips.
