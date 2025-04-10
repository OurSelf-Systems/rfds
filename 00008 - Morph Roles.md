```
Status: Test implemented in Psyche, need to move to branch in main Self repo
```

# Roles in Morphic

## The Problem
Morphs are nested graphical objects with their own behavior. Morphs are combined with each other in a tree structure to create more complex morphs. For example, a text editor morph might have submorphs which are lines, and those lines might have sub-morphs which are characters.

At some point, this hierarchy is cut off for reasons of practicality. In theory, you could have a pixel morph and build everything up from there, but that would be impractical. So at some point, the morph knows how to draw itself on a canvas in a reasonably complex way. Nevertheless, it's common to have quite heavily nested morphs. For example, the outliner code has quite deep nests of morphs.

Layout increases the nesting. Layout is done using horizontal and vertical spacing morphs. A more complex layout will need to lead to two or three levels of nesting, and then you have the next laid-out morph. A number of these morphs will be transparent, especially if they primarily exist to hand the layout for lower-level morphs.

We don't necessarily want to collapse this hierarchy! It provides a level of concreteness that might not be otherwise found. You can drag stuff out, you can drop stuff in. 

The issue is where, for example, your text editor morph at the top of your morph tree needs to be able to refer directly to morphs below it in the hierarchy. You need to be able to say "find this word and bold it" or "find this word and delete it and reflow everything." How does it find and interact with these submorphs?
## Current Solutions
There seem to be two main ways that the system currently allows you to do this:
1. **Searching Through the Morph Tree**
You can search through the submorph tree, looking for a morph that matches a certain test. A common test is whether it is a particular type of morph. Morphs keep track of their types. Each morph only has one type.
It's common that when you create or modify a morph and wish to keep that modified morph around, you give it a place in the hierarchy of morphs coming down from lobby. You give it a slot name (`myNewMorph`), you give it a type name (eg a string `"myNewMorph"`), and you give it a prototype pointing to itself, so that copies of it can keep track of what they are cloned from. 
This is a reasonably large amount of mechanism that we should do something about at some stage.
The issues with this approach are that it is detailed and somewhat clumsy, and that it often devolves into looking for a morph based on its position within the tree—"go down two levels and get me the third submorph." This is fragile. If you move anything around manually, your code breaks, and it expands the blast area of refactors of the submorph structure. And it is not intuitive for someone coming along and reading the code; it's hard to find out exactly why you're going down three stages and taking the third one. But nevertheless, that's widely used.
2. **Direct References**
The second major way that the system has for morphs talking to their submorphs is to provide a direct reference to that su-morph in a slot at the root morph.
So a text editor morph may have a whole bunch of submorphs: one of which is the text pane, one of which is a pull-down menu, one of which is a row of toolbar, one of which is a status bar showing the number of words. All of these could be wrapped in a number of row or column morphs to get the right layouts and behavior when growing and shrinking the morph.
That's fine, but you need to talk directly to your text pane morph. You don't want the fragility and general ugliness of doing a search for a morph that is of type "textEditorPaneMorph". So you create a slot in your root text editor morph called "myTextEditorPaneMorph", and when you create your hierarchy of submorphs, you keep track of which one is the text editor pane and you provide a direct link to it.
The problems with this are that the link has to be kept manually up-to-date. It does not allow late binding; it is early bound. It is fixed in there. If we yank out that text pane morph and we put in a new one, the reference will still be to the old one unless we have a manual process for keeping track of that yanking. The result of this is that you can also have behavior where you pull the pane out - but it still behaves as if it was part of the old parent morph.
## Introducing Roles
To make building morphs easier and cleaner, I have proposed and built a small amount of extra functionality which I'm calling "roles". The idea is that a morph can be tagged as fulfilling one or more roles. A role is simply some sort of object—it can be a string, it can be something else. A morph can find and interact with submorphs in its submorph tree that fulfill a role, or with morphs which are in the current hierarchy and also fulfill a role.

So to take our example of a text editor, the text pane morph will be told that it has to fulfill the "text pane" role. And the text editor can easily get access to that pane. Sending messages through, interacting with, or finding submorphs with roles only looks in the submorph tree. So if you pull out a morph, it doesn't receive those messages. As well, if you put in a morph, it will start receiving those messages.

By default, the morph will send messages to all submorphs fulfilling a role. It is a broadcast message.

I've used this system to build some small morphic tools as the test bed in Psyche. For example, I have created a window morph that wraps any morph at the top level on the desktop with some chrome—a title bar, a minimize button, a dismiss button. You can do that to any morph, and the first thing the morph does when it is dropped from a hand is that it sends a message to any owners that might be fulfilling the role of a window, saying what its title is. It's a loose, message-based mechanism.

It has worked well. I think it is simpler and more flexible than the previous approaches. I tried it without this mechanism and I'm going to keep on using it and seeing how I go. It's a very minor change and doesn't provide any real performance impact that I can see.

I think it helps a little bit to ameliorate the problem, which is that morphic sometimes tends to evolve into a tree-building system where you're constantly using code to build morphs. I think that has been the tendency which was exacerbated when Morphic moved to a class-based system, and is one of the reasons why people struggle with Morphic in Pharo/Squeak and similar systems.
## Roles vs. Morph Types
As I said before, morphs contain a slot referring to their prototype and also a string that is a name reference to that prototype, which is used for filing out and all sorts of things. But there's only one of those, and I think it is confusing the type of the morph with the role the morph is playing.

To give an example: A button makes sense to be a type. We have a "buttonMorph". But there are certain customizable things on that. The key ones are the label and the behavior when the button is pressed. Usually, those are not changed by creating a new type of button morph; they're just customized. But we want to be able to keep track of what roles which buttons are playing in a deeply nested structure.

We might want to be able to, at some point, grey out all buttons that do a destructive change (eg ‘cut’). Maybe we're writing the file out to disk or syncing with the cloud. We want to pause the possibility of making destructive changes until we've finished. So we want to be able to say "grey out all of the morphs that are buttons that are fulfilling the role of a destructive action."

I think that would be quite hard to do—it would require quite a lot of manual work—unless we have something like roles. We would have to either create a new type of morph, a "destructiveActionButtonMorph", which I think would be unnecessary (and if you go down that path, you end up with 1000 micro-types of morphs that are hard to keep track of and don't really meaningfully differ from each other). Or we would have to manually keep track of all of the button morphs that are destructive button morphs by having something like a collection held at the top level, and then we've got to manually remove and add things.

I think roles really help in this situation. They reduce the need to create new types of morph. We can have more generic morphs that are reused in a variety of roles. And by reducing the need to keep track of everything, we can make it easier to handle and deal with all this stuff just by yanking it around the GUI.

Roles also make morphs a little bit more robust. They will handle bits being missing, and the code will handle restructuring and refactoring of hierarchies without requiring a lot of refactoring of the root-level code.
## Current Interface

The prototype `morph` has a new slot added called `roles` which by default is an empty set.

The object `traits morph` has a new category `roles` with the following methods:

```
addRole: r
removeRole: r
hasRole: r = (roles includes: r)
morphsWithRole: r Do: blk IfAbsent: blk2
ownersWithRole: r Do: blk IfAbsent: blk2
```
