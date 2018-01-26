# Unified logistics #

Unified logistics is a script to run the logistics in a Mayhem empire
the way I want logistics to be run.

It is written for Litcubes Universe and will not work on AP. It has
not been tested with plain LU, since logistics there is relatively
trivial, it has been extensively tested on Mayhem.

Unilog is intended to replace the functionality of dock agents,
station agents, IDN and some couriers. Not because I want to compete
with them but because I want things done my way.

## High level goals ##

 - Reduce the number of clicks for setting things up

 - Reduce micromanagement, especially as the empire grows

 - Try to be efficient

## Usage example ##

Let's start with a simple example. You have one sanctuary and 2
factories and want the factories to never stop running. Press the
hotkey for UL, click on "select logistics dock", pick your sanctuary
as a logistics dock. Toggle the "Claim all factories" button. Within a
couple of seconds a list of all your factories pops up. Now your
logistics dock will know to manage those factories.  Change the view
on the top of the menu to "Trucks", click on the ships you want to add
as trucks. You're done. Your factories should now chug along as long
as a few basic requirements are met.

### A few things that just happened ###

 - A logistics dock is the dock where the script runs, this is the
   central hub of your logistics. All wares produced by the factories
   will be stored there and all wares in it are up for grabs to make
   the factories run. Yes, that also means food. We prefer that the
   sanctuary with the logistics dock starves rather than letting the
   wheels of the industry stop.

 - The claim all factories toggle when enabled tells the script on the
   logistics dock to manage all factories. If a new factory is built
   it will get automagically added. Currently there is no way add
   factories manually or to remove them. Once the factory is in, it's
   in forever. If someone uses this script and wants that
   functionality I could add it, but I don't need it at this moment,
   so it's not there.
   
 - A truck is any ship that will be doing work for us. Yes, if you
   want M5s to run logistics for you, they'll try (it's not a good
   idea). I picked the word "truck" because other words like
   "freighter", "agent" were already used by other scripts. You don't
   pick what the truck will do like dock agent or IDN (or CAG or
   CLS). UL will do that for you. One disadvantage here is that it can
   be hard to get the balance right if you don't have sufficient
   amount of trucks. UL will try its best to make things work, but if
   you don't have enough ships working for you it can not perform
   miracles (actually it can, it's very easy to script it in, but
   that's cheating).
   
 - If you had a problem with the simple example it was most likely
   that you didn't have any trucks to add. The reason for that is that
   trucks need to be homebased and docked at the logistics dock. It
   was because in initial development it made the most sense in the
   save I was starting with and I haven't bothered changing that. If
   there's interest that can definitely be changed.

## Details ##

### Trucks ###

Our trucks are dumb as bricks. They have no logic at all. In fact, the
only command they execute is: "go to X", nothing else. All the logic
in UL is centralized in the logistics dock. The logistics dock decides
when, which and how many wares to load and unload.

UL tries to be smart about using resources when they are available.
Let's say our cahoona bakery needs 500 argnu beef, we have lots of
argnu beef in the logistics dock and a truck is docked. We load 500
argnu beef into the truck but before sending it to the cahoona bakery
we check if the cahoona bakery also needs something else, we find out
that it also needs 2000 energy cells. So we add those to the truck and
send it on its way to the cahoona bakery. It docks, UL unloads the
wares and the cahoona bakery is happy once again. The truck is then
requested to go somewhere else for a job. Before going there we check
if that station needs cahoonas and if it does, we load them and take
them there.

### What is managed ###

We can add docks (other sanctuaries or MLCC) to be managed as well. In
that case UL will look at the dockware configuration capacities to see
what the dock needs and either send trucks from factories there (as in
the previous paragraph) or it will top them up from the logistics
dock. A thing worth noting is that if you have a TL as a truck and we
need to move a large amount of stuff we'll prefer to use the TL for
dock to dock trucking (this is not yet explicitly implemented, but it
kind of happens naturally). If we have a well stocked logistics dock
and build a new sanctuary it can get up and running in no time.
Standard freighters can take hours to stock a sanctuary with basic
wares and if you set dockware limits to reasonable amounts from the
beginning they might spend dozens of runs on "nice to have" things
like half a million energy cells while the sanctuary starves. Dockware
configuration is obeyed to +-10%. We won't send a truck to top up 24
scruffin fruits every time the few inhabitants at a sanctuary eat. If
the limit is 10000 we won't send a truck until they've eaten
1000. Same goes for if a dock happens to produce something with a
limit of 1000, we won't empty it until there's 1100 (beware when
producing things, if 1000 is the maximum possible limit on something
in a dock and that's what the dockware limit is set to, we'll never
empty it. Always leave some slack.

Dockware limits on the logistics dock work a bit differently. They are
+-50%.  When a ware reaches 50% of the limit and none of our factories
have it available we send a truck to buy it up to 100%. If a ware
reaches 150% of the limit we send a truck to sell it down to
100%. But, those limits are not respected when sending stuff to other
sanctuaries or factories. We will only buy things if the source of a
ware on the logistics dock dockware manager sources is set to 'Buy'.

### Priorities ###

There are currently three priority levels for things that need to be
done. High, normal and meh. Meh priority is only handled if a truck
happens to go to that destination and has spare cargo space. Normal
priority is "this needs to be done". This is stuff like topping up a
factory with resources that aren't very low yet or wares that are
getting low on docks. High priority is "do this or else". High
priority is only used for resources for factories that are about to
stall.

### More about trucks ###

Every available truck will be sent as soon as possible to fill the
needs of a client. We keep track of exactly how many wares a client
needs and how many are on the way, so if we need 100k energy cells,
we'll send 5 trucks with 20k each, we won't just send one truck with
one ware at a time to avoid conflicts.  We really do keep track of how
much is on the way and if a truck is destroyed en route we'll send a
new one immediately. This has the slight disadvantage that hostiles
detection logic is not implemented yet so if you have an armada of
pirates at the jump in point for a high priority client we'll send all
our trucks to die there are soon as they are available (yes, this is
hilarious when it happens and will be fixed).

## Bugs ##

 - Not critical. If a truck gets destroyed en route to a station all
   statistics about incoming wares to that station get reset. For the
   next few minutes we can end up with too many trucks delivering to
   that station and some wares will remain in those trucks until they
   happen to get to the logistics dock or some other consumer (by
   accident). This can be solved by keeping track of exact wares in
   jobs, but it would be a massive pain just to solve an annoyance
   that will eventually balance itself out.

 - Wares that have no demand won't be emptied from factories. If a
   factory produces a ware and nothing in our empire actually needs it
   we just won't send a truck to empty it (if a truck happens to go to
   that factory we'll still incidentally empty things, but this is not
   reliable). This could be considered a feature since we won't waste
   time and resources on unnecessary production but we could at least
   sell the overproduction. Which we won't.

 - Not critical. Wares in the logistics docks show very large amounts
   in the Needs menu. This is on purpose since we set those needs
   internally so that other clients can always offload wares into the
   logi dock. We won't actually run and buy those wares unless someone
   actually needs them.

 - Critical. Sometimes things get confused and we keep sending a truck
   from the logi dock to the logi dock. It picks up everything, then a
   few seconds later it drops everything. This would be hilarious
   except it spams the log and we often end up removing the wares the
   logi dock needs at that time.

## TODO ##

This is a list of things that need to be fixed before this is
releasable.

 - factory death (not yet implemented and will make the script die).

 - mining carrier clients (add mining carrier as client, the rest is
   automatic)

 - OWP clients (resupply missiles and maybe fighters?)

 - hostile sector detection and handling

 - dynamic priorities? Right now the priorities are crude and not very
   efficient. Stations about to stop would still have prio, but then that
   prio should be able to influence the stations that those stations depend
   on (etc.).

 - don't add trucks that don't have some basic level of software installed

 - high prio for sanctuary food

 - Only allow selecting a sanctuary as the logistics dock (if you
   screw that one up, there is no undo).

 - migrating logistics dock to a different sanctuary
