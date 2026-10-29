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
- a transform gizmo in the view: move (Shift lifts), height, size, uniform scale and rotate. **Paint** adds or clears fog, **Sculpt** raises or lowers parts of an element
- **Draft / Final**: Draft is the fast interactive march; Final renders at full plate resolution with twice the steps, every detail octave and detailed self-shadowing, and keeps refining
- lights fog with a measured cloud-droplet (Mie) phase function, multiple scattering and soft sun shafts from the reconstructed set
- checks value structure (Values / Value bands) and copies a Nuke stub or Houdini VEX plus a fog-set JSON for hand-off

The shot menu is a stand-in for the sequence's shot list. It holds two generated test shots: `tst_0010`, a wide on a valley village, and `tst_0020`, a medium shot up a winding lane through a hillside village for smaller volumes. Dropping a utility EXR on the view still loads it (multipart scanline, NONE / RLE / ZIPS / ZIP); the camera comes from EXR metadata when present, or is solved from `P`. Files never leave your machine.

Live: https://x14halo.github.io/brume/
