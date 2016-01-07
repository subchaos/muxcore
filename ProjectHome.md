## Installation ##
Download TinyMUX, and configure and make the code. Remove the default "netmux.db" from the /game/data/ folder. Download the latest MUXCore zip file and extract it to your shiny, new mux2.7 folder (drag and drop in Windows, or with something like the command line 'unzip' tool in Unix-based systems), allowing it to overwrite files. Use the db\_load script to load netmux.db.FLAT to netmux.db. Fire up the game and log in as #1! Source is included in the distribution ZIP, but source-based install is only supported in the UNIX version of TinyMUX. A readme and changelog are also included in the distribution zip.

## Purpose ##

MUX Core is a global softcode system for the TinyMUX text-based gaming platform. SGP, the included softcode base in current releases of TinyMUX, has served the community well since its introduction, but over the last decade the demands of the community have changed, and certain systems have established themselves as standards due to their ubiquity.

## Features ##

  * Most of the core features of SGP are retained with a variety of rewrites.
  * Anomaly Jobs 6.1 is included, and several systems are integrated with it.
  * Myrddin's Bulletin Board system (with tweaks and updates).
  * A new Cron system written from scratch.
  * Softcoded +help and +shelp as an alternative to text-based help files.
  * A custom Building Project Tracker to simplify the task of building staff.
    * Fully-integrated with Anomaly Jobs for pitching and approving projects.
    * Commands to check new builds.
    * Included code to manage build projects using the Zones system.
    * A couple of basic project parents to simplify ownership, tenants, and locks.
    * A minimal emphasis on actually digging and moving around projects allows Building Staff to be writers and organizers, not coders.
  * A stock room parent to easily control look and feel across the entire game.
  * A basic netmux.conf to easily set up a new server.
  * A basic channel set up with channel objects configured.

## Philosophy ##
Build on SGP and include softcode systems that can be useful for starting any type of game, without regard to genre. Focus on keeping the base system lean, and avoid succumbing to bloat or feature creep. While add-on packages may be made available for specific RPG systems to supplement the base system, those packages should remain separate from the base system of MUX Core. Bugs should be tracked openly and fixed quickly, and discussion about how the project proceeds should be open to the community that uses it.

## Licensing ##
The Code license for MUX Core is set to Artistic License/GPL as a default. However, since MUX Core only releases softcode and not binaries, the actual content of the project is released under the CC-BY-SA license. Attribute work to its original authors, and make your changes available to others!