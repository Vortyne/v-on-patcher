# Playable bosses

Jaguarandi and Z-Gradt as player machines in a one-player game, earned by
beating them and chosen on the machine select like the eight. The patch is
**Playable bosses** in the patcher's extras, unticked by default, with
**Pre-unlock the bosses** beside it for anyone who would rather skip the
earning. The source is [`asm/bosses.asm`](../asm/bosses.asm); this page says
what it does, then how.

## For players

### Unlocking

- **Jaguarandi** - beat Jaguarandi on **Very Hard** without losing a match
  anywhere in the run up to that point.
- **Z-Gradt** - with Jaguarandi already unlocked, finish the game on Very
  Hard with any machine, without a single lost match.

A continue is a lost match, and changing the difficulty spoils the run;
only a new game from the title starts a clean one.

Each unlock is announced on a screen of its own: the boss turning on the
left of a black screen, YOU UNLOCKED and its name on the right in the
select's lettering, and the title jingle. PRESS BUTTON TO CONTINUE flashes
at the bottom after a few seconds, and only a fresh press moves on - a
button held from the fight does not skip it. The press is answered with
the select's confirm sound, and the boss leaves the screen with it.

Jaguarandi's screen comes straight after it falls, before the Player Data
Report; Z-Gradt's after the credits, before the initials. The game then
carries on as it would have.

Unlocks are kept in `bosses.bin` beside the game and survive restarts and
re-patching. **Pre-unlock the bosses** writes it with both unlocked. It
never takes progress away, and **Restore original** leaves it alone - the
unpatched game never reads it.

### The machine select

- An unlocked boss stands in the row after Raiden: Jaguarandi first, then
  Z-Gradt raised on the hangar's lip. Locked ones are not there at all.
- Each has a portrait of its own in the row along the bottom, custom-made for
  the patch in the style of the eight's - framed and backed the same way,
  at the same 48 by 64 - after Raiden's. A locked boss's place stays
  blank.
- The portraits, the 1P/2P marks and the red frame move left to make room
  - by four columns with Jaguarandi, seven with both - so the row stays on
  a 4:3 screen. With nothing unlocked the select is exactly the original.
- The countdown runs 20 seconds longer with a boss unlocked, for the walk
  to the end of the row and the extra colours.
- The bosses are lit in their own colours like the eight, and the
  widescreen hangar fades them at the right edge as it does the others.
- **Machine Color Select**: up and down on a boss cycle it through the
  same eight colours the others have, any of which it can wear.
- Each boss launches from the hangar: Jaguarandi on Raiden's launch animation,
  Z-Gradt lifting off the lip and out through the tunnel. The water, the
  splash and every machine's thrust keep their own colours throughout.

### In the game

- Both bosses play as player machines, every stage, including Z-Gradt
  against Z-Gradt on the last one.
- The KO replay and the win and lose shots pull back to frame a boss's
  size; on a small arena the win shots that need floor under the camera
  fall back to the usual distance rather than search for ever.
- Z-Gradt's chase camera is pulled back out of its body, and turns with
  it as it turns on the spot.
- Z-Gradt's fly-in is shortened where the arena is in the way: indoors
  (Deathtrap, the Spaceport, the Secret Base) it only drops into place, and
  over the Flooded City, the Ruins and the Green Hills it starts part of the
  way in. GET READY's count waits for it to land before the round starts;
  on the last stage, where the CPU's Z-Gradt flies in instead, the round
  starts as it always did.
- Z-Gradt has no jump. The jump - both levers out, or the jump key - turns
  it to face the opponent instead, a half turn in about a second, the
  camera with it; not while its super laser is out.
- Z-Gradt's laser turns the right Z-Gradt gold when both are on the field,
  and a Z-Gradt that falls darkens alone: the player's keeps the colours it
  was given.
- The Player Data Report after stage 5 turns the boss itself: Jaguarandi
  in its select pose, as on its unlock screen, and Z-Gradt standing.
- The ending plays for a boss: Jaguarandi fires its own weapon at the moon
  gate and Z-Gradt its own charge and laser, then the text and the button
  wait. The staff roll is skipped: its battle-damaged model is one the
  bosses do not have.

### Limits

- One player only, by design: the bosses are far stronger than the eight,
  so two-player and internet play keep the select to the eight for a fair
  fight, and nothing here runs there.
- The bosses' weapons have no ammo: Z-Gradt's shots skip the game's
  charge counting, so its three gauges stay full, and Jaguarandi's refill
  faster than it can fire. Its super laser has no gauge, as no machine has
  a fourth.
- `bosses.bin` is written beside the game when a boss is unlocked. Where
  the folder cannot be written, the unlock screen still shows but the
  unlock is lost at exit.
- Jaguarandi's model file, `RB_jag.bin`, is read when the select first
  draws it. If the read fails it is drawn from whatever the model pool
  holds there, until a fight loads it.

## How it works

The game already knows the bosses as fight objects - the initials demo
fights 8 against 9 - so the objects work as a player's. What does not is
everything the game only ever did with the eight: tables eight rows long,
loaders that send ids above 7 elsewhere, screens framed for their size.
`asm/bosses.asm` is one hook per such place. The game keeps a copy of its
fight machine per player - A, at `0x1ae0xxxx`, player 1's, and B, at
`0x1ef8xxxx`, player 2's, the letters `docs/HIRES.md` gives their
renderers - but B's tick (`0x40f528`) is only called from the frame loop's
two-player branch, so a one-player game never runs it and every hook is
A's. Whether the player is a boss is one flag, `boss`, which confirm sets
in one player only. The sections below follow the file.

The blob has a section of its own, `.vobs`, that the patcher appends only
when the box is ticked, so unticked it leaves the executable as the other
patches alone would. Its buffers - the bosses' motions, the model and AI
copies, the saved palettes and the rest, about 230 KB - are gathered at
its end, and the file carries the blob only as far as its last byte that
is not zero, 15 KB; the rest is the section's virtual size, which Windows
zeroes at load (`OWN_SECTIONS`).

### The select's row

The select is a scene script: the camera, the eight 20 apart, the hangar.
The patch builds a longer copy with two records after Raiden's and points
the game at it, extends the tables that turn a cursor into a machine and an
object into a machine to ten, and raises the cursor's limit by the unlock
level (`selmax_a`). The camera's step is 30 to Jaguarandi and 98 on to
Z-Gradt, which also rises onto the lip (`selstep_*`).

The bosses' models are their fight models posed by the select's motions:
Jaguarandi through Raiden's, with its head put back on, and Z-Gradt in its
stance's first frame. Each file is read once into a block of its own
(`sel_loadrb`), and the game's model slot points there only while the boss
is drawn (`sel_slotsin`, `sel_slotsout`). They are not appended to the
game's 8 MB model pool: after a full game that runs into the C runtime's
heap.

The portraits are `assets/portrait_jaguarandi.png` and
`assets/portrait_z-gradt.png`, 48 by 64 and framed like the eight's,
baked into the patcher as the select's 16-bit colour by
`tools/portraits.py` (pure black, the tiles' transparent colour, becomes
the eight's near-black backing). The patcher writes them into
`escrgame.bin`, backed up first, in the 80 empty tiles after two of the
eight's portraits (`BOSS_ICON_TILES`); `selrow_make` builds the row with
them on entering the select, blank for a boss still locked.

The row's portraits, the marks and the frame are shifted per unlock level
(`selshift`); the frame sprite reads its x from two doubles in the game's
read-only data (`BS_FRX`), whose section the patcher makes writable so the
select can set them. The countdown's start is `seltime`.

### Colours on the select

How the game picks a palette:

- Palette RAM (`0x1cb5500`) is three planes of 32 rows of 256 entries. A
  polygon names its row per plane through its colour word in the mesh
  (`+6`), whose top ten bits index a table of colours (`0x66c2c8`); each
  colour byte's top five bits are that plane's row. The raster reads the
  word from the mesh later in the frame, not when the polygon is queued.
- The bosses' meshes name the greys of rows 1 and 3 (5 and 7 as the CPU's)
  - the rows the select fills with the machine under the cursor and its
  neighbours (1 to 11, as the cursor moves), so a boss drawn through them
  takes the colours of whatever the cursor is on, or none.
- The odd rows from 13 are set once at boot: 13, 17 and 19 are the launch
  thrust; the select rewrites 21 for a frame on each move and 23 as it
  opens; the launch water reads 21 and 25; the fight re-animates 21 to 29.

So on the select each boss borrows two rows the select itself leaves
alone - Jaguarandi 21 and 25, Z-Gradt 29 and 15 (`sel_bslots`). Their own
contents are kept when the select opens and put back the moment a launch
starts and when the select ends (`selpalguard`, `selpalback`). The boss's
palettes go in each tick and again as it is drawn, after the game's
one-frame write to 21 (`seljagpal`, `seldrawpal`), with the entries a
palette load leaves untouched copied from row 1, which the widescreen fade
reads as the boss goes dark (`selrest`). While a boss is drawn, each of its
polygons queued that names rows 1, 3, 5 or 7 - or another boss row - is
turned, in its copy of the mesh, to the table's grey of its own row
(`selremap`, `selcol`).

A launch is seen on the scene objects: every machine's goes to 1 as the
row moves aside and the launching one to 2, while the cursor field turns
into the object's number. A launching boss is drawn through rows 1 and 3,
which the game leaves alone on a boss, so all four borrowed rows are the
game's again for the water.

The widescreen hangar shades a machine by how far right of the camera it
is, black past 28.43 - a machine and a half at the eight's spacing.
`selfade` recomputes that shade for a boss, counting each gap between it
and the camera as one of the eight's 20.

### Unlocking

`unl_tick` follows the run: clean at the start of a game, spoiled by a
difficulty other than Very Hard, two players or a lost match (`unl_lost`,
at the game's own count). Beating Jaguarandi clean replaces the report's
`mov [state], 0x1c` with the unlock (`unl_jag`); a clean final win, the
initials' `mov [state], 0x16` (`unl_z`). The unlock screen is the report's
own state, 0x1e, driven by `unl_logic` in place of its handler: the turning
model is the report's (`model_ra` with the CPU's standing copy), the text
the report's big font, the name the select's logo tiles carried over
(`unl_grab`, `unl_place`), the palettes saved and given back whole.

Z-Gradt's screen comes after the credits, which load the dock's scene and
texture bank; the name entry's arena, glowing orbs included, is a
snapshot taken before and put back after (`unl_scnsave`, `unl_scnback`),
with the texture bank the fights had loaded reloaded.

### Elsewhere in the game

- **Palettes** - the loader (`0x4f358b`; `0x4c2026` is the other copy's)
  sends ids above 7 to a fixed table; for a player's boss it gives it the
  colour pair it was given on the select instead, from the side its
  player's machine uses.
- **PLAYER DATA** - the report's turntable reads per-machine tables only
  the eight have rows in. A boss is drawn by its fight object, or its
  select pose for Jaguarandi, from a standing copy taken at the start of
  its last round, and everything the draw touches is put back.
- **Round animations, the ending, Z-Gradt's camera, Z-Gradt against
  Z-Gradt, Z-Gradt's gold, the replay and the win shots** - each rerouted
  for ids 8 and 9, as its section in the source explains.
- **The player's Z-Gradt** - its fly-in starts by the arena loaded
  (`zflyin_a`, `zf_start`), and GET READY's count is held at its end till
  it lands (`zready_a`). Its facing goes to the chase camera each frame,
  and the jump turns the facing to the CPU at the CPU's own aim
  (`zlock`); Z-Gradt's own code turns its model with it.
- **Z-Gradt's death** - it darkens through palette events 0x401 to 0x43f,
  which the handler takes to be the CPU's; the player's are passed over
  (`zdark_a`, `zfade_a`).
- **Z-Gradt's launch flames** - Raiden's booster meshes, from Raiden's
  fight model, which the select loads only when the cursor passes Raiden;
  after a continue it starts on Z-Gradt, so the launch loads it as the
  select would (`BS_LOADFILE`).

## Testing it

- `bosses.bin` beside the game: delete it to start locked, or write the
  dword 1 or 2 to start part or fully unlocked.
- `python3 tools/selftest.py v_on.exe` checks every site and 437
  combinations of patches, including the two boss boxes with and without
  each other; `python3 tools/portraittest.py` on the game folder reads the
  portraits back out of `escrgame.bin`. `tools/check.py` runs both.
- The per-frame tick runs from the loop's idle call, which the loop skips
  on a frame with F10, F3 or Alt in the message queue; the unlock screen's
  state is cleared only by its own button press.
