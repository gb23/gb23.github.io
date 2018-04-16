---
layout: post
title:      "URL-Parameterizing nested objects"
date:       2018-04-16 01:28:52 +0000
permalink:  url-parameterizing_nested_objects
---


You may know well that URLs are appended with any necessary parameters.  The form is: 
```
www.example.com/api?param1=hello&param2=goodbye
```
and to the left of the '`?`' is the URL and to the right are any parameters being sent with the request. Here we have param1 = "hello" and param2 = "goodbye".  Note that the params are separated from one another with the '`&`'.  

Using `fetch` we can achieve the same request by putting the parameters in the body section of the object passed to the `fetch` function. So, say we are creating the same request as above (we'll say it is a POST request), it would look like this:

```
fetch("www.example.com/api", {
    method: "POST",
    body: {
        "param1" : "hello",
        "param2" " "goodbye" 
    }
}).then(response => response.json())
```

But what happens if we have a nested object as our body in `fetch`?  How would we then URL-parameterize that?  This is an important question because, while Node.js does provide a `querysting` module to translate objects to the form needed as parameters in a URL (a query string), it will not work for nested objects.  

How would we URL-parameterize this `fetch` body?
```
fetch("www.example.com/api", {
    method: "POST",
    body: {
        "name" : "Joe",
        "education": {
            "undergraduate" : "UCLA",
            "graduate" : "ASU"
        } 
    }
}).then(response => response.json())
```

The key thing to remember when URL-parameterizing an object is that each parameter stands on its own; That is, for a given value, we must list all the keys necessary to get that value. No shortcuts.  So, our value of "UCLA" is located with the "education" key, followed by the "undergraduate" key.
```
education.undergraduate=UCLA
```
Similarly, our graduate value would be:
```
education.graduate=ASU
```
Note: placing key values in brackets is considered unsafe in a URL.  So this is a no-no:
```
education[graduate]=ASU
```
(If you wanted to use brackets, you should escape them.)

So our final URL for this body object would be:
```
www.example.com/api?education.undergraduate=UCLA&education.graduate=ASU
```
You can further imaging taking an even more deeply nested object and converting it to something like
```
education.undergraduate.major=Chemistry
```
