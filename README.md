# Brume

A browser prototype for laying out fog from utility passes.

Each shot's utility EXR (world position `P`, normals `N`, alpha, and a beauty or surface colour pass) is all Brume needs. It:

- raymarches fog straight against the plate, with per-pixel holdouts from `P`
- rebuilds the set from `P` as a rough mesh or point cloud you can orbit with Maya navigation (Alt + LMB / MMB / RMB); **Reset to shot cam** returns to the shot camera
- reads the ground, hollows, cold-air drainage lines, walls, tree canopy, slope direction and sun shade from the passes, so each fog type forms where and how it does in nature:
  - **Ground fog** (radiation fog): densest at the ground, collects in hollows and drainage lines, thin by walls and under trees; burn-off lifts it and leaves it lingering in shade
  - **Valley fog**: fills the connected basin under a soft top, with waves when wind runs over it; lifts off the ground as it burns off
  - **Low cloud** (stratus / hill fog): a broken deck with a ragged base that dips into gullies and drops on windward slopes, clears in the lee, with scud underneath
  - **Fog bank** (advection fog): streaked along the wind, rides over rising ground, thins pouring down lee slopes
  - **Rising mist**: wisps and puffs off wet forest canopy or ground
  - thin **mist**
- builds every element as a signed density field with detail octaves pushed in and out of its surface (Worley billows or fBm wisps, each octave warped by the last), faded by pixel footprint so distance never turns to speckle
- gives every element its own look: style presets per type, plus wisps ↔ billows, detail, breakup, windswept, turbulence, edge softness and structure size. Pick a tool and set the look before you place; select an element afterwards to change it
- **Settle** (S, or Settle all) lets elements come to rest the way real fog would: valley fog refills its basin from the ground up, ground fog and mist drain downhill into hollows, banks sit back on the ground, low cloud drops until the hills reach into it
- a Maya-style gizmo on the selected element: drag the fog to put it where you point, axis arrows, a height arrow, a rotate ring and resize handles on the outline. Resizing a terrain-derived element re-evaluates the ground instead of stretching it
- one **Fog brush**: pile up, thin out, add, erase or smooth fog in places (Ctrl does the opposite)
- **Draft / Final**: Draft is the fast interactive march; Final renders at full plate resolution with twice the steps, every detail octave and detailed self-shadowing, and keeps refining
- lights fog with a measured cloud-droplet (Mie) phase function, multiple scattering and soft sun shafts from the reconstructed set
- checks value structure (Values / Value bands) and copies a Nuke stub or Houdini VEX plus a fog-set JSON for hand-off

The shot menu is a stand-in for the sequence's shot list. It holds two generated test shots: `tst_0010`, a wide on a valley village, and `tst_0020`, a medium shot up a winding lane through a hillside village for smaller volumes. Dropping a utility EXR on the view still loads it (multipart scanline, NONE / RLE / ZIPS / ZIP); the camera comes from EXR metadata when present, or is solved from `P`. Files never leave your machine.

Live: https://x14halo.github.io/brume/

## Designer

`designer/` is the rebuild: a black void for designing one volume at a time, then exporting it to Houdini.

- **Library** of 30 volume types, each with what it is, when to use it and what it needs: fog (radiation, valley, advection, upslope, freezing, ice, evaporation), clouds (cap, banner, deck, scud, rain curtain, cloud mass), atmosphere (depth haze, light-shaft haze, interior haze, inversion layer), lingering (residual smoke, settled dust, spray mist), emitters (sulfur smoke, steam vent, smoke source), stylized (glowing mist, magic swirl) and sculptable shapes (fog bank, hanging mist, ground fog patch, mist veil, rising wisps, tendrils, smoke puff).
- **Stand-in ground** to design against (flat, slope, peak, valley, bowl, street, pool, lake). Fog rests on it, drains down it, flows over and along it, and it casts shadows. Fog types that form by a rule (a layer that pools in hollows, fill to a level, upslope on the windward side, stacked sheets, a deck, a lid, even haze) form relative to it.
- **Emitters**: draw the source on the ground (disc, rectangle, line, ring) with the Source tool, then set soft edge, patchiness, pulse, rate, push, heat, spread, lifetime, tail fade and source fade. Plumes rise, level off as they cool and bend over in the wind.
- **Wind** in metres per second (0 to 25), with gusts and lulls that roll through with the wind, veer slowly and slow near the ground. **Wind paths**: draw a path with the Wind path tool (P), for example to the side, then curling up and back higher, and a local stream of wind follows it through the fog; each path has its own speed and width.
- **Moves in real time**: Play loops In to Out, and every type keeps moving the way it would (ground fog creeps and churns, banks roll in on the wind, plumes keep rising).
- **Physical daylight**, defaulting to a 10:30 am sun and sky. A low sun shines through more air, so it is dimmer, and the sky darkens with it. Manual keeps sun and sky as set.
- **Looks** in one click (soft mist, wispy curling smoke, puffy dense smoke, billowy, torn, streaky), plus Curl for how curly the detail is and **Fill gaps** to close holes so the fog reads as one volume.
- **Sculpt the fog you see**: brushes work on the simulated fog at the current frame, so what you drag is what you get and nothing re-simulates when you let go. Strokes are kept with the moment they were made and replayed when the simulation runs again. Add, erase, smooth, Pull (moves the fog as one piece, like a smudge), Comb (drags uneven strands through it, so it breaks into streaks and wisps), tear and Fill (fills holes and patches with the surrounding density).
- **Paint a second look** onto parts of the fog with the Paint look brush (L): torn fog that fades into soft mist, for example. The painted look travels with the fog as it moves.
- **Shading in pipeline terms**: extinction distance, extinction colour, albedo and albedo colour, eccentricity (+0.85 for mist, negative for thick black smoke), scatter bounces (Auto = -1) and emission (off by default). The values go out with the export.
- **Real sizes** in three scale classes. On Auto the simulation box is measured, not guessed: a quick low-resolution run of the whole timeline (to Out) finds where the fog actually goes and the box is cut to that, plus a few voxels. A fixed voxel budget is then spent on that box, so a tighter box means finer voxels. Locked keeps the box as it is.
- **Detail level** per asset (Soft, Medium, Detailed, Hero, or Auto) sits at the top of the panel, because voxels are what set a volume's detail: it sets the sim grid, detail octaves and export resolution, and shows the live sim voxel count, metres per voxel, export size and render weight.
- **Viewport resolution** 1K, 2K or 4K (long edge, top right of the view; G cycles). At rest the view renders every detail octave with detailed self-shadowing, refined over many light frames so the UI stays responsive; 2K and 4K are supersampled down to the screen.
- **One simulation per change**: the box is measured first with a quick low-resolution run, then the fog simulates once in it. Tightening or growing the box afterwards resamples the fog instead of simulating it again.
- **Timeline**: scrub the simulation, set In and Out on the stretch you like, set frames and fps, retime (slow motion re-simulates in finer steps rather than blending frames), extend a range for a longer shot, or Hold so the fog churns in place.
- **Export** this frame or the whole In–Out sequence as half-float density (normalised, so extinction distance sets the look) with a Houdini Python SOP that loads the right frame and a meta.json carrying the shader values; recipes save as JSON.

Live: https://x14halo.github.io/brume/designer/
