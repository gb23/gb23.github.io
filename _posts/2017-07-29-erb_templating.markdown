---
layout: post
title:  "ERB templating!"
date:   2017-07-28 20:20:38 -0400
---

Bust open an irb session and follow along, if you want.  ERB, or embedded ruby, is a templating tool for ruby.  It provides a way to inject ruby code into text.  Templating is a great timesaver -- it allows you to create a framework for a document, yet customize the contents however you choose.  By the end of this blog entry, I will have created a ruby class template that creates a new class file based off of data provided by the user: the class name, any attr_accessors, any attr_readers, and/or any attr_writers.  

**Simple ERB play**

For now, let's examine what exactly is going on when we create an instance of an ERB class.  

If we create a string,
```
str = "Currently #{Time.now}"
=> "Currently 2017-07-28 16:03:13 -0700"
```
the value of str is static.  Let's wait 20 seconds and check the value of str again.
```
str
=> "Currently 2017-07-28 16:03:13 -0700"
```
The time should be 20 seconds later if the value of str was compiled again when we called it, but it didn't change.  How can we make a variable be a dynamic representation of the current time?  ERB sounds like a good answer to me. 

To use the ERB class goodies we need to ```require 'erb'``` in our session.  Similar to what we did before, let's create a string, str, to contain the current time; however, this time we are going to inject ruby code into the string by delimiting our ruby code with ```<%=``` and ```%>```.  Later on, when we compile this string, the ruby code that is delimited will be evaluated during the compilation.  For now though, it is still just a string!
```
str = "Currently <%= Time.now %>"
=> "Currently <%= Time.now %>"
```
To get the delimited ruby code to be evaluated, first we need to instantiate a new ERB object, passing in the string we just made.  Yes, we are just passing in a string -- but, very importantly, it has ERB delimiters that will allow it to be evaluated soon.  The returned result is an ERB object.  
```
erb_obj = ERB.new(str)
 => #<ERB:0x007fb4a910a170 @safe_level=nil, @src="#coding:UTF-8\n_erbout = ''; _erbout.concat \"Currently \"; _erbout.concat(( Time.now ).to_s); _erbout.force_encoding(__ENCODING__)", @encoding=#<Encoding:UTF-8>, @filename=nil, @lineno=0>
```

Well that doesn't look too pretty.  We are looking at attribute values of an ERB object, but these aren't important now, so we can acknowledge they exist and move on.

Finally, we take our ERB object and call the method ```result``` to finally evaluate our delimited ruby code, and inject that value into the string at it's location.
```
erb_obj.result
=>"Currently 2017-07-28 16:21:21 -0700"
```
Now lets wait about 20 seconds and repeat our code
```
erb_obj.result
=> "Currently 2017-07-28 16:21:41 -0700"
```
Our time is different by 20 seconds.  We see that each time we call erb_obj with the method result, the result is determined right then and there.  

So, in summary: we took a string and injected it with ruby code that we wanted to be evaluated each time the result method is called on the corresponding ERB object to that string.  

That's neat, but templating is more powerful than that.

**Creating Ruby Class Template**

Let's template a ruby class using ERB.  A Ruby class looks something like this:
```
class ClassName
     attr_accessors :any, :thing, :here
		 attr_readers :any, :thing
     attr_writers :any
end
```

Using ERB templating, I want to be able to fill in the class name, and any attr_accessors, attr_readers, and/or attr_writers just by telling my program that I created what the values are, if any.

Let me show you a sample run of the program and the file generated, and then I'll go into specifics.

Sample Program Run:
```
Enter class name:
>Animal
Will class have attr_accessors? Y/N
>y
How many attr_accessors do you want in your class?
>2
Name the attr_accessor variable:
>height
Name the attr_accessor variable:
>weight
Will class have attr_readers? Y/N
>y
How many attr_readers do you want in your class?
>1
Name the attr_reader variable:
>species
Will class have attr_writers? Y/N
>y
How many attr_writers do you want in your class?
>3
Name the attr_writer variable:
>hair_color
Name the attr_writer variable:
>eye_color
Name the attr_writer variable:
>head_shape
```

The resulting file, Animal_163614.rb, is generated.  Here is its content:
```
  class Animal
    
    
               attr_accessor  :height ,  :weight                      
    
               attr_reader    :species                      
    
               attr_writer    :hair_color ,  :eye_color ,   :head_shape                      
    
  end
```


Now let's look through how I was able to create this class template.  I have two files.  Let's look at the file with the injected ruby in it first.

klass_template.rb
```
  class <%= klass %>
    <% j = 0 %>
    <% while j < macros.size %>
      <% if !arrays[j].empty? -%>
         <%= macros[j] -%>
         <% for attribute in arrays[j] -%>
          :<%= attribute -%> <%= "," if attribute != arrays[j].last -%>
         <% end -%>
      <% end -%>
      <% j += 1 %>
    <% end %>
  end
```

Any time I want to inject the value of Ruby code, I use the delimiters ```<%=    %>``` and any time I want to perform logic but not inject a value into the file, I use ```<%  %>```.  All the variables that don't make sense right now, such as klass, macros[], arrays[], are defined in the other file -- so ignore them for now.  The important thing to see is that this whole file is just a huge string, and anytime we want to inject some text into the string, we use ERB delimiters.  'class' isn't in delimiters, so it is just going to be represented in the huge string as 'class'.  Similary, the last 'end' has no delimiter and will be in the string too.  You may be wondering, why are some of the delimiters using ```-%>``` to end?  This is inorder to not force a new line when the value is injected.  By default, an injection will force a new line.  

The other file creates the ERB instance and collects user input

main.rb
```
require 'erb'
macros = ["attr_accessor", "attr_reader", "attr_writer"]
arrays = [attr_accessors = [], attr_readers = [], attr_writers = []]

puts "Enter class name: "
klass = gets.strip

i = 0
while i < macros.size
  puts "Will class have #{macros[i]}s? Y/N"
  input = gets.strip
  if input == "Y" || input == "y"
    puts "How many #{macros[i]}s do you want in your class?"
    input = gets.strip.to_i
    for y in 0..input-1 do
      puts "Name the #{macros[i]} variable:"
      var = gets.strip
        arrays[i] << var
    end
  end
  i += 1
end

template_string = File.read("./klass_template.rb")
template = ERB.new(template_string, nil, '-')
result = template.result

File.write("./#{klass}_#{Time.now.strftime("%H%M%S")}.rb", result)
```

Everything prior to the File.read is just collecting user input:  the class name, and any values of attr_accessors, attr_readers, and/or attr_writers.  The ERB fun occurs on the line after File.read, where we read in the klass_template.rb and store it in the variable template_string.  Remember, the file is just a string.  It still contains the ERB delimiters.  Nothing fancy has happened.

The next line instantiates an ERB instance, template.  We pass in the template_string made from the prior line. The second parameter passed in is the safe_level and is not important here.  The third parameter is called trim_mode and is important.  I set ```trim_mode = '-'``` Remember how in the prior file, I said that values injected with delimiters ending in ```-%>``` would not force a new line?  Well, that is because of ```trim_mode = '-'```.  The '-' on the end delimiter will prevent a new line.

Now we have an ERB instance, ```template``` and we call the method ```result``` on it.  When this method is called, all of our embedded ruby gets compiled.  That is,
```
class <%= klass %>
```
is converted to 
```
class Animal
```
This string is stored in the ```result``` variable.  
Lastly, we create a filename based on the klass name and the time it was created.  It's content is the ```result``` string.



Wow, templating is neat.  I imagine how much time it can save you when tasks are very structured!


[ERB Ruby Doc](https://ruby-doc.org/stdlib-2.4.1/libdoc/erb/rdoc/ERB.html)
