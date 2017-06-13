---
layout: post
title:  "Fix Your CLI Gem Project Print Output "
date:   2017-06-13 01:07:51 -0400
---

**Stop Text From Getting Split At The End Of A Line!**

You go through the effort to make a really good CLI gem project and then what happens?   You print info, but ```unfortun```   

```ately ``` the text gets split at the end of the line sort of like that.

I wrote a piece of code that fixes this, and, man, what a better experience it is to run my gem.  It's super easy to use.  Instead of writing *puts "Display this text"* in your code, you simply write *"Display this text".print_fit* and that's it!

Here's how I did it.

**Code**

In your environment file:  require 'io/console'

In another file put this code:
```
class String

    def print_fit
        dimensions = IO.console.winsize
        window_length = dimensions[1]
        word_array = self.split(" ")
        word_index = 0

        remaining_room = window_length
        while remaining_room >= 0 && word_index < word_array.size
            remaining_room -= word_array[word_index].size
            if remaining_room >=0
                print word_array[word_index]
                if remaining_room > 0
                    print " "
                    remaining_room -= 1
                end
            else
                puts ""
                print word_array[word_index] + " "
                remaining_room = window_length - word_array[word_index].size - 1
            end
            word_index += 1
        end
        puts ""
    end
end
```

**Explanation** 

In order to get the text not to split at the end of the line, the program needs to know how big your terminal window is. The require 'io/console' that is placed in your environment file allows you to use the **winsize -> [rows, columns]** method of this standard library.  IO.console.winsize returns an array, where index 0 is the size of the window vetically and index 1 is the size of the window horizontally.  Since we are concerned with text getting split at the end of a line of text, we want to store the horizontal window size. That is done in the code here: 
```
dimensions = IO.console.winsize
window_length = dimensions[1]
 ```
 
The rest of the code is simply printing one word at a time on a line until you are out of room or just about out of room on a line of text.  When that happens, you force a new line with *puts ""* and continue printing.  The easiest way I found to do this was to split the string into an array of words, and by checking the remaining_space (which is originally set to the window_length) against the size of the word to be printed, you can tell if it can be printed on the current line or if it belongs on the next line.  If a word is printed, the size of the word is subtracted from the remaining_space variable. When you start on a brand new line, remaining_space is reset to window_length. 

It was worth the effort because the reading experience is much improved.
