---
title: "LookupWord - From Mac"
categories:
  - Tutorial
tags:
  - vapor
  - owlbot
  - applescript
  - heroku
  - lookup word
  - dictionary
  - vocabulary
---

## Save the word when using Mac to look up

### preparation

> 1. [Server side set up]()
2. [Applescript](http://apple.stackexchange.com/questions/121790/is-there-a-way-to-track-my-look-ups-in-the-osx-dictionary/121802#121802)

This one is extremely easy, only one step, which is use Automator to create a service, and set up a shortcut.

here is the script you need to use, replace `[your app's url]` to yours, and copy it.

```applescript
on run {input, parameters}
	
	set lookUpWord to quoted form of (input as string)
	--This sets the word to the selected text, from input.
	
	do shell script "open dict://" & lookUpWord
	--This opens the word in Dictionary.
	
	do shell script "curl https://[your app's url]/word/" & lookUpWord
	
	return input
end run
```


Then open Automator, and create a new service

![create applescript](http://g.recordit.co/sTn7yyMpSt.gif)

After you've set up service, you just need to access it with shortcut, so go to System preferences/keyboard/shortcut to set it up.

![add shortcut](https://media.giphy.com/media/l0MYND3ULhJot27Pa/source.gif)

Then, done!!

You can now select a word a use the short to search.

and you can check it from the server you've made.

![](http://g.recordit.co/2adBWatLlF.gif)

Looking forward to build the iOS client to use the data in the next tutorial.

see ya.
