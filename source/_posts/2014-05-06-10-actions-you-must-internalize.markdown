---
layout: post
title: "10 actions you must internalize in your Editor"
date: 2014-05-06 10:45:07 +0530
comments: true
categories: editors productivity
---

I say *10*, but, it could well be *16*.  The point is having a minimum viable list.

We love our Emacs vs Vim vs &lt;others&gt; debates.  In one argument Emacs rocks, in another Vim wins the day and in some other, TextMate takes the cake.  However, more than we love our editors, we love being productive; after all that is all what this is about.  By now, we know that the joy of writing code is well augmented by a good editor. It acts like a great force multiplier.  We go to great lengths to curate the editor and it's configuration to what suits us.

What happens when you want to use a different editor?  Or the same editor with different dotfiles?  They say that opening an editor and finding someone else's dotfiles is like going to bed and finding someone else's partener there.  From the outset, looks like an extremely awkward and uncomfortable experience. But, great if you were looking to switch.  So how do you learn an editor in order to be effective enough; enough being the keyword here?  The short answer to this is "do the grunt work and put in the time".  However, I think all is not lost and learning a few actions as the first thing can save you a lot of time.  I often keep cheat sheet ready with these actions in order to build the required muscle memory.  

Now, this list is of course not complete.  A lot of things could be added to it or removed from it.  However, this is my recipe for quickly learning an editor.  And I would love to hear yours.  One thing to keep in mind is that this is not an exhaustive list of features that our editors must have.  No.  This is a list that we as programmers can use to quickly pass the learning curve of an editor.



1. **Navigation within the file, moving around.**  This is the most fundamental and the very first thing you'll probably do.  If in Emacs you can't C-n C-p C-f or C-b your way around, you'll feel exasperated soon.  So first thing to learn would be move the caret forward and backward.  There are different flavours to moving around:
    1. One character at a time
    1. One word at a time
    1. One line at a time
    1. One paragraph at a time etc.
1. **Select, cut, copy and paste.**  This comes naturally from Point 1.  Once you are able to move around, you would want to move text around.  Selecting text is also extremely important.  You don't have to use your mouse to actually select.  Selection is actually demarkation, and it works a scope.  Select a bunch of text and then perform an action only on that text.  I'll elaborate on this further down the list. <small> &lt;rant&gt; This ability is overrated and often superceded by others actions in the list. &lt;/rant&gt; </small>

1. **Duplicate, delete, move and displace lines and blocks.**  When working with code, we often work with whole lines, rather than words or characters. Which is why this ability comes extremely handy.  It is best used while refactoring code or formating code.  I cannot stress how important this is.  Duplicating lines and them moving them around is indispensible.  Once you've learned to use this well, I'll bet you would be uncomfortable without it.

1. **Search and replace strings and regular expressions; scoped or not.**  Search and replace, combined with text selection that acts as scope is an integral part of refactoring code.  When I think of S&R, I think of renaming variables, functions, classes.  I think of upgrading from `:key => value` to `key: value`.  I think of upgrading from Bootstrap 2 to Bootstrap 3, where `span12` becomes `col-md-12`.  You get the point.  As a side note, if you are building an editor to learn something new, this can a fun feature to implement.  Go ahead and try if it suits you.

1. **Quickly generate boilerplate code.**  Once you are past the steeper part of the learning curve of a language technology, boilerplate manifests as the killer of you productivity and brings death to it by boredom. Unless, you have tuned your editor to snuff it out quickly.  Type `def<tab>` and boom, you have a method definition placeholder. `cla<tab>` and you have class declaration, `for<tab>` you have your block ready. etc. etc.  More than the terse languages like Ruby, Python, this will be more handy in verbose languages like Java, Objective C.

1. **Quickly jump to recently used files. Jump to any file within the project in few keystrokes.**  `Command+T` in TextMate `Control-P` in Vim etc.  When working on a project, the impact this feature can have is tremendous.  One of the best use cases is jumping to last file.  I often find that I typically work with 2 files at a time ( Model and the Controller or The presenter and the view, the header and the implementation, Model and its Tests etc. ).  So I need to be able switch to other file fast.  In Emacs, `C-x b <enter>`, in TextMate `Command-T <enter>`, in RubyMine `Command-e <enter>` do the job quite fanasticaly for me.  In fact, as a principle, I do not use editors where this isn't possible or isn't fast enough ( I am looking at you Eclipse and Netbeans ).  Most editors who do this well allow you to perform "partial name search" i.e. searching for bdct will return \*broadcast\* amongst others.  Folder scoping is also often allowed. i.e. searching for `a/v/a/bdht` can return `app/views/admin/_broadcast.html.erb`.

1. **See or edit different parts of the same file or vertical & horizontal split views.**  When jumping between the files is not enough, you need to be able to look at the files at the same time, or look at different parts of the same file.  E.g. while editing markdown, you'll want live preview on right.  Or while writing a public API for your class, you'll want to look at the private methods etc.  In another case, you'll want to open up a REPL on the side while you edit code and preiodically send code to the REPL for evaluation.  Use cases are plenty and hence this is a part of this list.

1. **Dumb autocomplete.**  Your editor and you should know how to quickly complete words in the same file, words typed before, keywords in the current language (programming or not).  `C-\` in Emacs, `Alt-/` in IntelliJ or `Escape` in TextMate are good examples of this.  Fancy, code aware completions are heavily IDE dependent and very hard to get right.  But dumb completions can be fast and very handy.

1. **Run a test or open a shell.**  I love my `Ctrl+Shift+R` in RubyMine where it runs the test I am curretly editing or the test file I am currently editing, depending on where my caret is.  Same goes for `Command+Shift+R` in TextMate. In Emacs `M-x shell` opens the terminal within Emacs and use it often, for running tasks, creating files, git commits, etc.

1. **Comment and uncomment code.**  I contemplated whether this should be a part of this list and decided to keep it.  When working on existing code, commenting and uncommenting can be extremely valuable.

