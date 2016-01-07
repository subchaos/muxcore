# Player Globals (#6) #

## Flags ##
The Player Globals object is set INHERIT and SAFE. INHERIT is necessary to give the object access to wizard commands and functions, and SAFE is to prevent accidental deletion.

## Attributes ##

### +Finger ###
```
&CMD.FINGER #6=$+finger *:@switch 
	[isdbref(setr(0,ifelse(strmatch(%0,me),%#,pmatch(%0))))]=
	0,
	{@pemit %#=No such player.},
	{
	@pemit %#=
	[header([name(%q0)] (%q0)[if(isstaff(%#),%b[flags(%q0)])])]%r
	[ljust(%xhAlias:%xn [get(%q0/ALIAS)][case(1,hasflag(%q0,WIZARD),%b(Wizard),hasflag(%q0,ROYALTY),%b(Staff))],39)]
	%xh%xx|%xn
	
	[ljust(
		switch(
			[hasflag(%q0,CONNECTED)]
			[hasflag(%q0,DARK)]
			[isstaff(%q0)]
			[isstaff(%#)]
			[hasattr(%q0,DARK-TIME)],
			0*,
			%xhLast On: %xn[first(get(%q0/LAST),.)],
			11101,
			%xhLast On: %xn[first(get(%q0/DARK-TIME),.)],
			11100,
			%xhLast On: %xn[first(get(%q0/last),.)],
			%xhOn For: %b%xn[singletime(conn(%q0))]%b%b%xhIdle: %xn[singletime(idle(%q0))]
			)
		,39)]%r
	
	[ljust(%xhSex:%xn %b%b[get(%q0/SEX)],39)]
	%xh%xx|%xn
	[ljust(%xhMail:%xn[space(4)][extract(setr(1,mail(%q0)),2,1)] Unread / [add(extract(%q1,1,1),extract(%q1,3,1))] Total,39)]%r
	[ljust(%xhEmail:%xn [get(%q0/EMAIL)],39)]%xh%xx|%xn%r
	[subheader()]%r
	[iter(
		cat(
			setinter(iter(v(D.FINGER-LIST),first(%i0,:),|),lcstr(lattr(%q0))),
			lcstr(lattr(%q0/FINGER-*))
			),
		%xh[rjust(
			[ifelse(
				ismember(v(D.FINGER-LIST),%i0:*,|),
				rest(grab(v(D.FINGER-LIST),%i0:*,|),:),
				capall(edit(rest(%i0,finger-),_,%b))
				)]:,
			12)]%xn%b[wrap(get(%q0/%i0),65,,,,13)],,%r)]%r
	
	[if(isstaff(%#),[subheader()]%r
	%xhLast IP: %b%b%xn[get(%q0/LASTIP)]%r
	%xhLast Site: %xn[get(%q0/LASTSITE)]%r)]
	[footer()]}
```
This gigantic wall of text, broken down for readability, is the +finger command. The top section of the +finger eats up the majority of the code space. The only real oddball thing up here is the switch() function that decides how to display Last On based on whether the target is staff, is dark, and has a DARK-TIME attribute (see +dark). The real "magic" is in an iter that starts near the bottom. It reads in all the attributes on the player that match items in D.FINGER-LIST on #6 as well as any attributes that start with FINGER- and then goes about displaying them. The wrap() function controls the autowrapping of the contents of the attributes. A +finger command is a good first project for new coders, because it contains elements of a lot of the core things you'll need to do in other parts of your coding-- laying out text, grabbing attributes, doing logic, etc.

```
&D.FINGER-LIST #6=fullname:Full Name|age:Age|app-age:Apparent Age|position:Position|fame:Fame|rp-prefs:RP-Prefs|alts:Alts|themesong:Theme Song|quote:Quote|hours:Hours|temperament:Temperament|vacation:Vacation|url:URL|wiki:Wiki Page
```
This list controls the built-in finger attributes. The first part of each x:y pair is the attribute name, and the second is how that section should be titled in the output.

### +Staff ###
```
&CMD.STAFF #6=$+staff:
	@pemit %#=[header(Online Staff)]%r%xh
	[ljust(Name,14)][ljust(Alias,6)][ljust(Position,35)][ljust(Status,10)][ljust(Idle,6)]%r
	[subheader()]%r
	[iter(
		setinter(objeval(%#,lwho()),get(#8/D.STAFFLIST)),
		[ljust(name(%i0),13)]%b[ljust(get(%i0/ALIAS),5)]%b[ljust(get(%i0/POSITION),33)]%b%b
		[ljust(if(and(isstaff(%#),hasflag(%i0,DARK)),DARK,if(hasflag(%i0,transparent),Off Duty,On Duty)),9)]%b
		[singletime(idle(%i0))],|,%r
		)]%r
	[footer()]

&CMD.STAFF/ALL #6=$+staff/all:
	@pemit %#=
		[setq(0,cat(setinter(objeval(%#,lwho()),get(#8/D.STAFFLIST)),setdiff(get(#8/D.STAFFLIST),objeval(%#,lwho()))))]
		[header(Staff List)]%r
		%xh[ljust(Name,14)][ljust(Alias,7)][ljust(Position,35)][ljust(Last,10)]%r
		[subheader()]%r
		[iter(%q0,
			[ljust(name(%i0),13)]%b[ljust(get(%i0/ALIAS),6)]%b[ljust(get(%i0/POSITION),33)]%b%b
			[switch(
				[hasflag(%i0,CONNECTED)]
				[hasflag(%i0,DARK)]
				[isstaff(%i0)]
				[isstaff(%#)]
				[hasattr(%i0,DARK-TIME)],
				0*,
				first(get(%i0/last),.),
				11101,
				first(get(%i0/dark-time),.),
				11100,
				first(get(%i0/last),.),
				Connected
				)]
			,,%r)]%r
		[footer()]
```
These control the +staff commands. There is some magic in the /all version to determine whether to show last on time based on the same isstaff/DARK/DARK-TIME calculation used in +finger. The actual list of staffers is built by the @startup of #8 to grab everyone flagged WIZARD or STAFF, and there is a staff command to force that list to refresh. The big setq at the beginning of /all is just to sort the list in such a way so that online staffers get listed first.

### +Uptime ###
```
&CMD.UPTIME #6=$+uptime:
	@pemit %#=[mudname()] runtime stats:%r%t
	MUSH boot time.......: [starttime()]%r%t
	Current time.........: [time()]%r%t
	In operation for.....: [div(sub(convtime(time()),convtime(starttime())),86400)] days, [div(mod(sub(convtime(time()),convtime(starttime())),86400),3600)] hours, [div(mod(sub(convtime(time()),convtime(starttime())),3600),60)] minutes, [mod(sub(convtime(time()),convtime(starttime())),60)] seconds.
```
This code is retained unchanged from SGP.


---IN PROGRESS