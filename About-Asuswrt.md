###About Asuswrt:

Asuswrt is a unified firmware developed by Asus for use in their recent routers.  The firmware was originally based on Tomato-RT/Tomato-USB, but has since seen many changes.  Asus started using this new firmware with their recent routers like the RT-N66U, but they also started moving other routers to this new firmware, like the DSL-N55 or the RT-N56.

Being mostly based on GPL code, almost all the source code and all necessary build tools is available from their website.  There are a few proprietary components that are closed source (like the wireless drivers from Broadcom/Ralink).  In these cases, Asus includes binary-only versions of these files.  In the end, their GPL release includes everything needed to completely recompile a working firmware, with the exact same features as found in their firmware releases.

One big advantage of Asus going with a unified firmware is that they have one single code base for all their supported routers.  That way, bugfixes for one device can automatically be applied to all other supported devices.  Same when they start adding new features (such as AiCloud), it's easier for them to support multiple routers at the same time with these news features.  And finally, it means that support for a router does not die the minute a newer router replaces it on the market.  The RT-N16 still gets updated as frequently as the RT-N66U, for instance (as they share very similar hardware).

### About Asuswrt-merlin

Asuswrt-merlin takes advantage of the fact that complete source code is available from Asus by building on top of it.  Asuswrt-merlin is my personal project, originally done for the RT-N66U, but eventually extended to the RT-AC66U (and on an experimental level, to the RT-N16).

The goal with this project is to provide an alternative to Asuswrt which will benefit from the same rapid development as Asus have for this firmware.  This means there are some very strict design goals with this project:

1. **Stay as close as possible to the original firmware.**  By limiting the amount of sweeping changes made to the code, it means that whenever Asus releases a new version of Asuswrt, it only takes a few hours to re-base Asuswrt-merlin on this new code, and test it.

2. **The goal is to improve, not to replace the original firmware's functionality**  Projects like DD-WRT and Tomato have existed for years, and benefit from those years of development to offer a lot of new features.  There is no point in reinventing the wheel - people looking for a completely different firmware with tons of advanced features should look at those excellent and well-established projects.

3. **Priorities: Stability > performance > features.**  Fewer changes to the code means fewer chances that new bugs might be introduced.  A router firmware is the core of your home network.  It must be rock-stable above all.  And performance optimizations can have unexpected side-effect when dealing with things that aren't fully understood.

4. **Targeting the novice and average user.** Asuswrt is designed to target both the novice and the average users.  This project will aim at the same target userbase.  There is enough doors left open with Optware and user-scripts so advanced users can fulfil their own requirements themselves.  Not overcrowding the webui with esoteric features will ensure that novice users won't be scared away.


Thanks to Optware and user-scripts, many features not integrated in the firmware can be manually added.  If you are advanced enough to need a feature, you will quite often be skilled enough to manually add it.

That does not mean you should't ask for feature requests.  Please, still do.  Just don't be disappointed if your request gets rejected.  With enough people asking for the same feature, it might make it popular enough to eventually be implemented.  Or you might be pointed at an alternative way to achieve your goal.
