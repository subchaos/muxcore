# #5: Global Functions #
## Thanks! ##
The header, footer, and subheader functions, as well as the capall function are based on code from St. Petersburg. Thanks to Chopin@StP, as well as any other past coders who worked on these functions. The idea behind the tags system comes from some code that I saw a very very long time ago from Firan, when they were still making certain softcode public. My tags code is my implementation of that idea, as their code was no longer available when I wrote it, but credit for the idea goes to Firan.

## Flags ##

The Global Functions object is set INHERIT and SAFE. INHERIT is necessary to give the object access to wizard commands and functions, and SAFE is to prevent accidental deletion.

## Attributes ##

### Startup ###
```
@Startup #5=
	@dolist lattr(me/FN.*)=@function after(##,FN.)=me/##;
	@dolist lattr(me/FPRIV.*)=@function/privileged after(##,FPRIV.)=me/##
```
The startup on #5 adds all the functions on the object as @functions, which allows them to be accessed globally without needing to use u(#5/FN.FOO,bar). Any attribute preceeded with FN. is added as a normal function with the name as whatever follows the period. Attributes starting with FPRIV. work similarly, but will evaluate with wizard permissions. For this reason you should be careful to add access checks to FPRIV functions. Adding new global functions is as easy as adding an attribute to the object following the naming scheme, and either using "@tr #5/startup" or @restart.

### FN.ISSTAFF ###
```
&FN.ISSTAFF #5=
	orflags(%0,WZ)
```
This function checks %0, which should be a dbref, for the WIZARD or ROYALTY flag. The most common use of this is to use "isstaff(%#)" in commands as part of a @switch to restrict commands to staff. Some games also check for the STAFF flag-- orflags(%0,WwZ)-- but I chose not to do that in MUX Core since the STAFF flag does not actually convey powers. Any character flagged WIZARD, in addition to a variety of other hardcoded benefits, can see all attributes on all objects (see\_all), and change anything in the database that is not locked to #1 (control\_all). Characters flagged ROYALTY can only see\_all, but cannot control\_all (see "help powers list" for information on this terminology). STAFF can do neither, and do not have any of the other benefits of the ROYALTY or WIZARD flags.

### FN.ISMEMBER ###
```
&FN.ISMEMBER #5=
	t(grab(%0,%1,%2))

ex: think ismember(test list,test)
--> 1
    think ismember(test|list,test)
--> 0
    think ismember(test|list,test,|)
--> 1
```
This function allows you to see if %1 is a member of %0, with %2 as an optional delimiter. Because it uses grab(), the function is case insensitive.

### FN.PARTIAL ###
```
&FN.PARTIAL #5=
	localize(
		ifelse(
			setr(0,grab(%0,%1*,%2)),
			%q0,
			0)
		)

ex: think partial(test list of STUFF,t)
--> test
    think partial(test|list|of|STUFF,t)
--> test|list|of|STUFF
    think partial(test|list|of|STUFF,s,|)
--> stuff
    think partial(test|list|of|STUFF,z,|)
--> 0
```
Partial() is a shortcut function to allow you to do rudimentary partial matching. The function returns the match if there is one, and 0 if there is not.

### FN.ISPOSINT ###
```
&FN.ISPOSINT #5=
	and(
		gt(%0,0),
		regmatch(%0,^\[0-9\]*$)
		)

ex: think isposint(42)
--> 1
    think isposint(-1)
--> 0
    think isposint(forty-two)
--> 0
    think isposint(16.5)
--> 0
```
Isposint() checks to see that a number is a positive integer. An integer is a whole number greater than zero. The regmatch is a shortcut around needing to check isnum() or deal with floating point numbers.

### FN.ROLLXDY ###
```
&FN.ROLLXDY #5=
	case(0,
		isposint(%0),
		#-1 INVALID NUMBER OF DICE,
		lte(%0,v(D.MAXDICE)),
		#-2 NUMBER OF DICE MUST BE 1-[v(D.MAXDICE)],
		and(lte(%1,v(D.MAXFACES)),isposint(%1)),
		#-3 NUMBER OF FACES MUST BE 1-[v(D.MAXFACES)],
		iter(lnum(%0),die(1,%1))
		)

ex: think rollxdy(5,100)
--> 67 81 92 7 34
```
This function rolls %0 dice with %1 faces. There are basic limits on the maximum number of dice and faces that can be rolled to prevent performance issues from players rolling obscene numbers of dice. The base limit on dice is 30, and faces is 100. This function provides a simple base to build further dice code.

### FN.HEADER ###
```
&FN.HEADER #5=%xh%xx
	[switch(
		%0,,
		repeat(-+-,26),
		cpad(%xh%xx%[ %xn%xh%0%xx %],78,%xh%xx-+-)
		)]
```
This function is to control the look and feel of command headers. With no arguments given, it's a simple bar. With an argument though, it will center that argument inside the bar and highlight it. Tweaking the header, subheader, and footer functions is the key way to control the look and feel of your game, and all code in MUX Core uses them. When you tweak these functions, remember that several pieces of code depend on them being able to take arguments.

### FN.FOOTER ###
```
&FN.FOOTER #5=%xh%xx
	[switch(
		%0,,
		repeat(-+-,26),
		[repeat(-+-,17)][cpad(%xh%xx%[ %xn%xh%0%xx %],27,%xh%xx-+-)]
		)]%r
```
The footer() function is similar to header, but is meant to be used at the bottom of command output. The same considerations apply regarding rewrites as with the header function. If given an argument, footer() will offset it to the right hand side of the footer. For an example, type "+bbread" with a Wizard character and note the capacity display.

### FN.SUBHEADER ###
```
&FN.SUBHEADER #5=%xh%xx
	[switch(
		%0,,
		repeat(-,78),
		cpad(%b%xh%0%xn%b,78,%xh%xx-)
		)]
```
A subheader is similar to a header, but is used to "break up" the display output for variety, and to be used as a header for subsections. Given an argument, it centers it similar to header, but does not add brackets. For an example, type +help. The top is a header(), while the subsection headings use subheader().

### FN.CAPALL ###
```
&FN.CAPALL #5=
	[edit(
		edit(
			edit(
				iter(lcstr(%0),capstr(##)),
				%bOf%b,%bof%b
				),
			%bThe%b,%bthe%b
			),
		%bAnd%b,%band%b
		)]
```
This function is a simple cheat to capitalize the initial letter of all the words of a string, minus certain conjunctions. This is used in +finger, the jnotes system, +info, the tags system, and the building system.

### FN.ALERT ###
```
&FN.ALERT #5=%xh%xx%1%xn%xw%1%xh%xw%1 %0
```
Alert() is a simple function used to alert players about something in an eye-catching manner. It was basically a toy coded for testing, but it's used in +join, +summon, and +return to let players know that someone is coming or going. %0 should be the alert string, and %1 should be a "decoration character" that will be repeated in ascending colors. This function is not terribly useful, and was mainly a toy to test a concept.

### Tags ###
```
&FPRIV.TAGS #5=
	localize(
		ifelse(
			isstaff(owner(%@)),
			case(
				setr(0,locate(%#,%0,*)),
				#-1,
				#-1 NOT FOUND,
				#-2,
				#-2 TOO MANY MATCHES,
				get(%q0/_TAGS)
				),
			Permission denied.
			)
		)
```

```
&FPRIV.HASTAG #5=
	localize(
		ifelse(
			isstaff(owner(%@)),
			case(
				setr(0,locate(%#,%0,*)),
				#-1,
				#-1 NOT FOUND,
				#-2,
				#-2 TOO MANY MATCHES,
				ismember(tags(%q0),%1)
				),
			Permission denied.
			)
		)
```

```
&FPRIV.ANDTAGS #5=
	localize(
		ifelse(
			isstaff(owner(%@)),
			case(
				setr(0,locate(%#,%0,*)),
				#-1,
				#-1 NOT FOUND,
				#-2,
				#-2 TOO MANY MATCHES,
				eq(words(setinter(tags(%q0),lcstr(%1))),words(%1))
				),
			Permission denied.
			)
		)
```

```
&FPRIV.ORTAGS #5=
	localize(
		ifelse(
			isstaff(
				owner(%@)),
				case(
					setr(0,locate(%#,%0,*)),
					#-1,
					#-1 NOT FOUND,
					#-2,
					#-2 TOO MANY MATCHES,
					gt(words(setinter(tags(%q0),lcstr(%1))),0)
					),
				Permission denied.
			)
		)
```
These functions are integral to the Tags system. Note that any player can use these functions, and they must be FPRIVs because tags are stored in a wizard-only attribute (_TAGS) to prevent snooping or players changing their tags. The tags() function lists all of a players tags. Hastag() will check if %0 has a tag named %1 (useful for tagging players with races or spheres and locking commands to those spheres), and andtags() and ortags() will do a logical and() or or() on a list. This is useful if you have, say, police and FBI PCs. You can have separate tags so that there are things only each type can do, but use ortags(player,police fbi) to lock things that both should be able to do._

### Other Stuff ###
```
&D.MAXDICE #5=30
&D.MAXFACES #5=100
```
These are the data attributes that control the limits on the rollxdy() function. Set them however you like based on your game's needs.

```
&D.MUXCORE.VERSION #5=1.0 Alpha
```
This shows the current MUX Core version. This is not currently used, but may be necessary in the future.