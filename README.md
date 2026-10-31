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
- **Two modes.** **Shape**: the type forms once, then holds still for you to sculpt; nothing re-simulates while you work. **Animate**: your shape is frame 1 and the fog moves on from it with its own settings (wind in m/s, direction, gusts, churn, rise or sink, Hold). Play loops back to the shape instantly, and changing animation settings never changes the shape.
- **Influencers** to direct the motion in Animate, drawn in the view: **Wind path** (P; press where the air starts, on the fog or the ground, and draw: the path lies on a sheet through that point facing you, so a loop drawn from the side curls up and over; Shift draws level; Width is how far around the path the fog is carried; only a few well-placed points are kept; each path can also Twist the fog around itself), **Lift** (U; Ctrl sinks), **Calm** (C), **Thin** (T; Ctrl thickens), **Vortex** (S; Ctrl clockwise; drawn as a 3D spiral with dots to make it taller or wider). Every influencer drops a dashed line to the ground so its height reads at a glance. **Edit** (E): click to select and tune it in the panel, drag its dots to move them (Shift up and down, Ctrl the whole thing), Delete removes it.
- **Sculpt guidance**: the way you pulled, combed and blew the shape carries on gently as it plays (Follow my sculpt), so the motion continues from your sculpt rather than fighting it. Rise and sink are in metres per second.
- **Any value**: click the number on any slider to type a value, including past the slider's range.
- **Sculpt this frame**: take any animation frame (pre-roll included) as the new shape, sculpt it, and the animation plays on from there. It is kept as a step in the shape's history, so it rebuilds exactly and undoes.
- **Physical daylight**, defaulting to a 10:30 am sun and sky. A low sun shines through more air, so it is dimmer, and the sky darkens with it. Manual keeps sun and sky as set.
- **Looks** in one click (soft mist, wispy curling smoke, puffy dense smoke, billowy, torn, streaky), plus Curl for how curly the detail is and **Fill gaps** to close holes so the fog reads as one volume.
- **Sculpt the fog you see**: Add (takes on the density and breakup of the fog it is painted onto), erase, smooth, Pull (grabs the fog under the brush, detail and all, and moves it with the cursor; Strength sets how closely), Comb (drags uneven strands through it), tear, Fill (fills holes and patches) and **Blow** (B: a gust along your stroke that carries the fog and curls it, using the real flow). Strokes are kept in order and replayed if the shape forms again. Undo and Redo sit on the viewport.
- **Painted looks**: up to three entries, each a full look (soft mist, torn fog, puffy smoke…). Paint an entry onto parts of the fog with the Paint look brush (L); change the entry's look later and everything painted with it follows. The painted looks travel with the fog as it moves, and show in their colours while the brush is active (Show colours turns that off).
- **Hide handles** (eye button, or H): only the volume, nothing drawn over it.
- **Shading in pipeline terms**: extinction distance, extinction colour, albedo and albedo colour, eccentricity (+0.85 for mist, negative for thick black smoke), scatter bounces (Auto = -1) and emission (off by default). The values go out with the export.
- **Real sizes** in three scale classes. On Auto the simulation box is measured, not guessed: a quick low-resolution run of the whole timeline (to Out) finds where the fog actually goes and the box is cut to that, plus a few voxels. A fixed voxel budget is then spent on that box, so a tighter box means finer voxels. Locked keeps the box as it is.
- **Detail level** per asset (Soft, Medium, Detailed, Hero, or Auto) sits at the top of the panel, because voxels are what set a volume's detail: it sets the sim grid, detail octaves and export resolution, and shows the live sim voxel count, metres per voxel, export size and render weight.
- **Viewport resolution** 1K, 2K or 4K (long edge, top right of the view; G cycles). At rest the view renders every detail octave with detailed self-shadowing, refined over many light frames so the UI stays responsive; 2K and 4K are supersampled down to the screen.
- **One simulation per change**: the box is measured first with a quick low-resolution run, then the fog simulates once in it. Tightening or growing the box afterwards resamples the fog instead of simulating it again.
- **Timeline**: scrub the simulation, set In and Out on the stretch you like, set frames and fps, retime (slow motion re-simulates in finer steps rather than blending frames), extend a range for a longer shot, or Hold so the fog churns in place.
- **Export** this frame or the whole In–Out sequence as half-float density (normalised, so extinction distance sets the look) with a Houdini Python SOP that loads the right frame and a meta.json carrying the shader values; recipes save as JSON.

### Shot (test scene)

The **Shot** tab opens a made-up test shot (a village on a valley floor, rolling hills, a mountain range) through a shot camera. Fog is painted into the set rather than dropped in and scaled:

- **Layers** (up to four), each a fog type that forms by its own rule relative to the ground: **Ground fog** (lies on the ground, set depth), **Valley fog** (fills the low ground to a level), **Hanging mist** (floats at a height above the ground), **Low cloud** (a layer at an altitude, wrapping the slopes it meets), **Mountain cap** (sits on the peaks you paint over), **Haze** (even, up to a height).
- **Place brush** (Q): pick a layer and paint on the ground where it goes; the ground is tinted with each layer's colour while you paint, and the fog forms there when you let go. Ctrl erases.
- **Each layer is its own volume**, with its own box, voxels and detail level, look and animation, so valley fog in the village keeps its detail next to cloud on the peaks. The layers are drawn together over the set, nearest last. The layer you pick is the one every Shape and Animate tool works on. **Shot camera** returns to the shot's framing.

Live: https://x14halo.github.io/brume/designer/
