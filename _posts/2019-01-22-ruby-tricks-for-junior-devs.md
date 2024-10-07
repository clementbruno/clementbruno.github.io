---
layout: post
title: Ruby tricks for junior developers
subtitle: Some small tricks and examples to improve your code
date: "2019-01-22"
---

This article was originally posted on [Getaround Tech Blog](https://getaround.tech/ruby-tricks-for-junior-devs/) which is a company I was employed at. It was then featured in the articles' section of [Ruby Weekly](https://rubyweekly.com/issues/434) 😎. The stack at Getaround Europe was a big Ruby on Rails monolith and most of the logic was handled by the back-end there. Hereafter are a few tips and tricks for developers that I gathered while on the job.

---

## The dig method

The **`dig`** method can be used on hashes (and Arrays) to, as its name suggests, dig through the potential multiple layers in the object and retrieve the value corresponding to the argument provided to the method.<br />
By using **`dig`** you find yourself able to nicely shorten your code and improve overall readability:

```ruby
# Let's consider the following hash structure
user_info = {
  name: 'Bob',
  email: 'bob@drivy.com',
  family_members: {
    brother: {
      name: 'Bobby',
      age: 16
    },
    mother: {
      name: 'Constance',
      age: 55
    },
    father: {
      name: 'John',
      age: 60
    }
  }
}
```

Formerly, if I wanted to access our user's brother name, I would have written something like that:

```ruby
user_info[:family_members][:brother][:name]
```

But that is considering the fact that I was **SURE** that each key in fact existed in the hash structure.<br />
For instance, if I executed the following statement, my program would crash because of a non existing key:

```ruby
user_info[:family_members][:grand_father][:name]
# => `NoMethodError: undefined method `[]' for nil:NilClass`
```

Therefore, if I wanted to be safe while navigating in my hash structure, I should have written something like that:

```ruby
user_info[:family_members] && user_info[:family_members][:grand_father] && user_info[:family_members][:grand_father][:name]
# => nil
```

This is way too long and annoying to write... With the **`dig`** method I can simplify this statement a lot:

```ruby
users_info.dig(:first, :family_members, :brother, :name)
#=> "Bobby"
users_info.dig(:first, :family_members, :grand_father, :name)
#=> nil
```

## Protected methods

Everyone knows about `public` and `private` methods but I've found that not many people use `protected` ones.

Neither `private` methods nor `protected` ones can be called **directly** by instances of the class in which the method is defined.<br />
For an instance to access these methods they must be called within a public method defined in the class.

```ruby
# Let's consider the following classes
class Animal
  def current_activity
    puts "I am #{define_activity}"
  end

  private

  def define_activity
    @activity ||= ["eating", "hunting", "sleeping"].sample
  end
end
```

Hereabove, the `current_activity` method could be called by any instance of the Animal class (_or any other class inheriting from the Animal class_).

```ruby
# For instance
class Cat < Animal
  def preferred_activity
    "My favorite activity is #{define_activity}"
  end
end

felix = Cat.new
felix.preferred_activity
# => "My favorite activity is sleeping"
```

However, when a method is `private` it cannot be called on `self` within a class...
For instance, this would raise an error:

```ruby
class Animal
  def current_activity
    puts "I am #{self.define_activity}"
  end

  private

  def define_activity
    @activity ||= ["eating", "hunting", "sleeping"].sample
  end
end

kong = Animal.new.current_activity
# => NoMethodError: private method `define_activity' called for #<Animal:0x007ff6881d08b8>
```

Being able to do so could be especially handy if you wanted to call it on other instances of your class passed as method arguments. For instance it would be useful to be able to do that:

```ruby
class Animal
  ...
  def same_activity_as?(other_animal)
    define_activity == other_animal.define_activity
  end

  private

  def define_activity
    @activity ||= ["eating", "hunting", "sleeping"].sample
  end
end

fido = Animal.new
bobby = Animal.new

fido.same_activity_as?(bobby)
# => NoMethodError: private method `define_activity' called for #<Animal:0x007ff688142f90>
```

In order for the above code not to break we can make the `define_activity` method `protected` instead of `private` and everything will work just fine:

```ruby
class Animal
  ...
  def same_activity_as?(other_animal)
    define_activity == other_animal.define_activity
  end

  protected

  def define_activity
    @activity ||= ["eating", "hunting", "sleeping"].sample
  end
end

fido = Animal.new
bobby = Animal.new

fido.same_activity_as?(bobby)
# => true
```

\_NB: Please note that the protected method can here be called on instances of the class but only within the class definition body. These methods cannot be directly called on an instance of the class such as:

```ruby
fido.define_activity
```

which would return an error:

```ruby
NoMethodError: protected method 'define_activity' called for #<Animal:0x007ff688846120>
```

## ||= assignment

When I got started I didn't know about ruby's `double pipe equals:` `||=`. The concept is fairly simple and can be very useful in a variety of situations.
Basically using `||=` allows you to perform a variable assignment **if and only if** the variable is not yet defined or if its value is currently falsey (`nil` or `false`).

```ruby
number = 10

nil_variable = nil
nil_variable ||= number
# => 10 --> nil_variable is now set to 10

false_variable = false
false_variable ||= number
# => 10 --> false_variable is now set to 10

not_defined_variable ||= number
# => 10 --> not_defined_variable is now set to 10

content = "I already have some content"
content ||= number
# => "I already have some content" --> The content variable is not reassigned and keeps its initial value
```

## Safe navigation operator: _&_

This one allows to navigate safely through the layers of objects relations.
Basically, let's say that we have a company with only one employee and that this employee has a name and an email address:

```ruby
class Company
  attr_reader :employee

  def initialize(employee)
    @employee = employee
  end
end

class Person
  attr_reader :name, :email

  def initialize(name, email)
    @name = name
    @email = email
  end
end

bobby = Person.new('Bobby', 'bobby@drivy.com')
drivy = Company.new(bobby)
```

In this context if I wanted to access Drivy's employee name I woud probably do the following:

```ruby
puts drivy.employee.name
# => Bobby
```

However, this only works in an environment where none of the elements in the chain (except possibly for the last one) can be `nil`.
Now, let's imagine a case where the company does not really have any employee. The `drivy` object would be instantiated as follows and the above code would raise an error:

```ruby
drivy = Company.new(nil)

puts drivy.employee.name
# => NoMethodError: undefined method `name' for nil:NilClass
```

In order to prevent this behaviour, ruby has the `&` operator (since version 2.3) which behaves a bit like the `try` method in rails.
It tries to fetch the object attribute and returns `nil` if any element in the chain is `nil`.
For instance:

```ruby
drivy = Company.new(nil)

puts drivy&.employee&.name
# => nil

google = Company.new(bobby)
puts google&.employee&.name
# => Bobby
```

## Struct

`Struct` is kind of a shortcut class which acts in a way more "liberal" manner. It allows for faster development when you are not willing to create a new class with an `initialize` method and other behavioural methods.

`Struct` is created as follows:

```ruby
animal = Struct.new(:name, :type, :number_of_legs)
fido = animal.new("fido", "dog", 4)
```

And the parameters passed to the `new` method are instance variables directly accessible from the instance:

```ruby
puts fido.name
# => "fido"
fido.type = 'cat'
# => "cat"

# I said that Struct are more liberal because they do not enforce the presence of the arguments that you have to provide to create new instances:
bob = animal.new("bob")
```

In addition to `Struct`, ruby provides the ability to use `OpenStruct`. The main difference is that `Struct` creates a new class whereas `OpenStruct` directly creates a new object.

```ruby
dinosaur = OpenStruct.new(name: "t-rex", number_of_legs: 2)

puts dinosaur.number_of_legs
# => 2
```

These tools can be very useful to stub simple object or classes while testing since they allow to replicate basic behaviours very quickly without much hassle.

## Symbol#to_proc

Finally I'd like to share a ruby idiom which allows to nicely shorten some statements and improve readability :)

You may have already seen things such as:

```ruby
[1,2,3].reduce(&:+)
# => 6

[1,2,3].map(&:to_s)
# => ["1", "2", "3"]
```

When ruby sees the `&` followed by a symbol, it calls the `to_proc` method on the symbol and passes the proc as a block to the method.

The above examples are equivalent to writing:

```ruby
[1,2,3].reduce(0) do |sum, num|
  sum + num
end
# => 6

[1,2,3].map do |n|
  n.to_s
end
# => ["1", "2", "3"]
```
