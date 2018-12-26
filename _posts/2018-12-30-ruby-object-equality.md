---
layout: post
title: Different ways Ruby understands equality
description: In Ruby we have different ways to check object equality. Learn the differences here.
author: asalom
tags:   [ruby]
---

[Ruby](https://www.ruby-lang.org/en/) is a very good choice to learn as a secondary language for an iOS Engineer. Many of the tools we use daily are written in Ruby: [fastlane](https://fastlane.tools/), [CocoaPods](https://cocoapods.org/), [Danger](https://danger.systems/) and many others. Not only their code is written in Ruby, also their configuration files are.

Take a look at this [Podfile](https://guides.cocoapods.org/syntax/podfile.html). For a while I thought it was just a configuration file but if you pay close attention to it, it is actually plain Ruby code that [CocoaPods](https://cocoapods.org/) will execute when generating a project. As we can see in this example we define dependencies to be added to the project and towards the end we also write a workaround to solve an old issue that by the way it is already fixed for version `1.6.0.beta.2`. All this is written in Ruby.

```ruby
platform :ios, '11.0'
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/LivePersonInc/iOSPodSpecs.git'

use_frameworks!

target 'AppTarget' do
  pod 'Fabric'
  pod 'Crashlytics'
  pod 'PromiseKit'
  pod 'SwiftLint'

  target 'AppTargetTests' do
    inherit! :search_paths

    pod 'Nimble'
  end
end

swift42 = ['PromiseKit']

# Workaround for Cocoapods issue #7606
post_install do |installer|
  installer.pods_project.build_configurations.each do |config|
    config.build_settings['PROVISIONING_PROFILE_SPECIFIER'] = ''
    config.build_settings.delete('CODE_SIGNING_ALLOWED')
    config.build_settings.delete('CODE_SIGNING_REQUIRED')
  end
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      if swift42.include?(target.name)
        config.build_settings['SWIFT_VERSION'] = '4.2'
      end
    end
  end
end
```

The [fastfile](https://docs.fastlane.tools/advanced/Fastfile/)? Also written in Ruby. [Dangerfile](https://github.com/danger/danger/blob/master/Dangerfile)? It is as well. Learning ~~some~~ Ruby never hurts, this is why understanding how to check object equality might come in handy.

### What is equal in Ruby

In Ruby we have 4 different ways to check object equality and each of them is meant for a different purpose: `equal?`, `eql?`, `==` and `===`

#### • `equal?`
This method checks for identity equality between two objects and this means that two objects are `equal?` if they point to the same reference. It comes for free with every Ruby object since it is defined in [Object](https://ruby-doc.org/core-2.6/Object.html) and every other Ruby object inherits from it. [Well, almost every object but this is out of the scope of this post](https://stackoverflow.com/questions/8894817/whats-the-difference-between-object-and-basicobject-in-ruby).

```ruby
foo = "foo"
bar = foo
another_foo = "foo"

foo.equal?(bar) # true
foo.equal?(another_foo) # false
```

---

#### • `eql?`
The second way to check for equality is `eql?`. This is used to check for equality in [Hash](https://ruby-doc.org/core-2.5.3/Hash.html) keys and you might never call it directly. You will only need to overwrite it yourself if you want to use a custom object as a Hash key.

> Two objects refer to the same hash key when their hash value is identical and the two objects are `eql?` to each other.
A user-defined class may be used as a hash key if the hash and `eql?` methods are overridden to provide meaningful behavior. By default, separate instances refer to separate hash keys.

The default implementation for this method checks object identity by calling `equal?`.

```ruby
class Person
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def eql?(other)
    return true
  end

  def hash
    1
  end
end

person1 = Person.new("Alex")
person2 = Person.new("Salom")
hash_table = { person1 => person1.name, person2 => person2.name }

puts hash_table.count # 1
puts hash_table[person1] # Salom
puts hash_table[person2] # Salom
```

---

#### • `==`
This is what most of us understand as object equality. The double equal will execute the `==` method of the first object against the second.

```ruby
class Person
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def ==(other)
    @name == other.name
  end
end

person1 = Person.new("Alex")
person2 = Person.new("Alex")

puts person1 == person2 # person1.==(person2) is true
```

The default implementation of this method checks object identity by calling `equal?`. Knowing this, if we were to remove the implementation of `==` from the previous example, the objects wouldn't be equal.

```ruby
class Person
  attr_reader :name

  def initialize(name)
    @name = name
  end
end

person1 = Person.new("Alex")
person2 = Person.new("Alex")

puts person1 == person2 # person1.==(person2) is false
```

---

#### • `===`
Last we have the triple equal which is used inside case statements. The `===` method will be executed in each `when` object against the `case` object. The main reason for the triple equal to exist is so we can match regular expressions in a cleaner way. It's probably a good idea to leave the `===` alone unless doing so results in really ugly case statements.

By default `===` calls the double equals method.


```ruby
class Person
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def ===(other)
    return @name == other
  end
end

person1 = Person.new('Alex')
person2 = Person.new('Salom')

case 'Alex'
when person1 then puts 'Alex' # person1.===('Alex') is true, 'Alex' is printed
when person2 then puts 'Salom' # person2.===('Alex') is false
end
```

With regular expressions it makes much more sense. [Regexp](https://ruby-doc.org/core-2.5.0/Regexp.html) has a custom implementation of `===` which receives a string so it can match the regex against it.

```ruby
branch = `git rev-parse --abbrev-ref HEAD`

case branch
# This is equivalent to Regexp.new('^master$').match('master')
when /^master$/ then puts 'master' 
when /^develop$/ then puts 'develop'
end
```