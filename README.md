# CueScript
This is the scripting language we use to control our in-game events.

It's very simple so far, it just is about doing a sequence of things at a specified time.  Maybe future versions will want to add parameterization or something, but for now, it's just as simple as we could make it.

## Overall Structure

A script is text file containing a sequence of events, each of which is either a message to various subsystems, a directive to control timing, or a comment.  Each whitespace-delineated line of the script starts with the type of event, and then everything afterwards has semantics determined by that subsystem.  If required, single arguments can contain spaces and special characters by using quotes and C-style escaping.

There are three special types of cues, which are fundamental to the scripting language itself.  Everything else is just application specific subsystems.

## Timing and Processing

When a script is started, the processor stores the start time, and a nominal current offset (which starts at 0).  All script timings are relative to that start time. The processor will then dispatch events as fast as it can until it reaches a timing event (Time or Delay).  Note that all delays are based on the difference between the nominal time and hte current time, so even if slow processing delays an individual event a little, we catch up by the next timestamp, and in the pathological case where we run past that, we will catch up as soon as we have a chance, and then things immediately return to normal.  

Event | Syntax | Semantics
------ | ------ | ------
`Time` | `Time millis` | Change the nominal time to `millis` (or if prefixed with a +, add `millis` to the nominal time). Then wait until elapsed time (now - start time) catches up to nominal time.  E.g, `Time 1000`, `Time +100`
`Delay` | `Delay millis` | Change the start time to be `millis` earlier, and wait until elapsed time catches up to the nominal time.  This effectively shifts all subsequent `Time` events (absolute and relative) to happen `millis` later.  E.g., `Delay 1000` 
`Trace` | `Trace text` | Print `text` to whatever console is available, for debugging and UX reasons.  Does nothing.  E.g., `Trace Starting Phase 1 Light Show

## Other Events

These are more application-specifc.  These are some of the ones Ghost Patrol uses:

Event | Syntax | Semantics
------ | ------ | ------
`OSC` | `OSC address args` | Send an OSC message to the `address` with the specified arguments. E.g., `OSC /Maglocks/Safe 0`
`DMX` | `DMX address val1 [val2 ...]` | Set one or more DMX controlled lights with consecutive addresses, starting at `address`.  (Addresses can be named for ease of use) E.g, `DMX Sconces 0 0 0.5 0.5 1`
`Audio` | `Audio file.wav gain speaker_mapping` | Plays a sound effect or music file named `file.wav`, with `gain`, routing audio according to the named speaker configuration `speaker_mapping`.  
`GameState` | `GameState component args` | Send a message to update the state of part of the game.  E.g., `GameState SafeKeypad Disable` or `GameState CurrentPhase 2`
`14Seg` | `14Seg address effect text` | Show `text` on 14 segment display specified by osc `address`, with specified `effect` (`Static`, `Scroll`, `Dissolve`). E.g.,  `14Seg /PM/Status Scroll "ASSEMBLING POWERLING"`
