---
layout: post
title:  That's Sooo Meta: Making a dynamic attr_accessor
date:   2017-06-02 13:31:17 +0000
---

The concept of metaprogramming is really really neat.  It seems magical to me that you can write some code that then writes additional code itself.  As an exercise, I decided to create a class that had zero hard-coded instance variables and, additionally, zero hard-coded getter and setter methods for those instance variables.  
Shout out to Avi for demonstrating the code behind a more static attr_accessor and inspiring me to create this more dyanmic version!

> At runtime, the user tells the program, "I want the class to be able to get and set instance variables of my choosing." This is essentially a dynamic attr_accessor.

This is just an exercise and not feasible for real world use because, for one thing, it is not good design for a user to easily alter a class model.  That is, for, say, a class Rocket, if a user added an attribute @favorite_food, that wouldn't make much sense and shouldn't be allowed (*although I guess "liquid hydrogen" could be acceptable*).  Regardless, it was a fun way to explore metaprogramming concepts.

Before going through the code, I'll show sample use by user and subsequent output.

```
rover = Dog.new "name"
=> #<Dog:0x007fa2859e0848>
```
This code creates an instance of the Dog class named rover and, additionally, creates two methods: name and name= that handle the instance variable @name.  Now the user can set and get this instance variable.
```
rover.name = "Rover"
=> "Rover"

rover.name
=> "Rover"
```

Now lets add another attribute @breed as well as methods breed and breed=.

```
Dog.add_attribute("breed")
=> [:name, :breed]

rover.breed = "Pug"
=> "Pug"

rover.breed
=> "Pug"

rover
=> #<Dog:0x007fa2859e0848 @breed="Pug", @name="Rover">
```

Let me create another instance to show that all instances of the class have the same instance variables.
```
fido = Dog.new
=> #<Dog:0x007fa28612abd0>

fido.name = "Fido"
=> "Fido"

fido.breed = "Mutt"
=> "Mutt"

fido
=> #<Dog:0x007fa28612abd0 @breed="Mutt", @name="Fido">

rover
=> #<Dog:0x007fa2859e0848 @breed="Pug", @name="Rover">
```

Lastly let's update these instances with one more attribute, @owner.

```
Dog.add_attribute("owner")
=> [:name, :breed, :owner]

fido.owner = "Tim"
=> "Tim"

rover.owner = "Tom"
=> "Tom"

fido
=> #<Dog:0x007fa28612abd0 @breed="Mutt", @name="Fido", @owner="Tim">

rover
=> #<Dog:0x007fa2859e0848 @breed="Pug", @name="Rover", @owner="Tom">
```

I think that's really neat!

Here's the code.

```
class Dog
    @@attributes = []
  
    def initialize(attribute = nil)
        if attribute
            self.class.add_attribute(attribute)
        end
    end
    def self.add_attribute(symbol)
        @@attributes << :"#{symbol}"
        self.init_getter_and_setter
    end    

    def self.define_properties(props)
        props.each do |prop|
            define_method(prop) do
                instance_variable_get(:"@#{prop}")
            end

            define_method("#{prop}=") do |arg|
                instance_variable_set(:"@#{prop}", arg)
            end
        end
    end
    
    def self.init_getter_and_setter
       self.define_properties(@@attributes) 
    end
end
```

* The class variable array ```@@attributes``` will be holding the dynamically growing list of class attributes.

* The initialize method can take an attribute name to be created as a parameter, though this isn't a requirement.

* The class method ```add_attribute``` takes the string of the attribute name and pushes the symbol version of that name into the ```@@attributes``` array.

* The class method ```init_getter_and_setter``` will then call the method ```define_properties``` to create the getter and setter methods. ```define_properties``` takes ```@@attributes``` as a parameter and, for each array index value, will define the getter method based on that value and the setter method based on that value concatenated with "=". This is done using the ```define_method``` method.  Now that the method names have been defined, to actually enable them get and set, you use ```instance_variable_get``` and ```instance_variable_set``` methods.

* Here's an example of what define_method is doing:
```
define_method("shamwow") do
      puts "Wow!"
end
```
     is the same as
```
def shamwow
      puts "Wow!"
end
```
					

More information on these meta methods below:

* [define_properties method](https://ruby-doc.org/core-2.2.0/Module.html#method-i-define_method)
* [instance_variable_set](https://ruby-doc.org/core-2.4.1/Object.html#method-i-instance_variable_set)
* [instance_variable_get](https://ruby-doc.org/core-2.4.1/Object.html#method-i-instance_variable_get)
