# Brume

A browser prototype for laying out fog from utility passes.

Load a frame's utility EXR (world position `P`, normals `N`, alpha, and a beauty or surface colour pass) and Brume:

- raymarches fog straight against the plate, with per-pixel holdouts from `P`
- rebuilds the set from `P` as a rough mesh or point cloud you can orbit with Maya navigation (Alt + LMB / MMB / RMB)
- reads the ground, hollows, cold-air drainage lines, walls, tree canopy, slope direction and sun shade from the passes, so each fog type forms where and how it does in nature:
  - **Ground fog** (radiation fog): densest at the ground under a flat top, collects in hollows and drainage lines, thin by walls and under trees; burn-off lifts it and leaves it lingering in shade
  - **Valley fog**: fills the connected basin under the inversion, denser towards its top, with waves when wind runs over it; lifts into low cloud as it burns off
  - **Low cloud** (stratus / hill fog): condensation-level base that drops on windward slopes and clears in the lee, with scud underneath
  - **Fog bank** (advection fog): streaked along the wind, rides over rising ground, thins pouring down lee slopes
  - **Rising mist**: wisps and puffs off wet forest canopy or ground
  - plus thin **mist**, free **pockets** and camera-facing **veils** for value structure
- lights fog with a measured cloud-droplet (Mie) phase function, multiple scattering and soft sun shafts from the reconstructed set
- lets you paint fog in or clear it out, check value structure (Values / Value bands), and copy a Nuke stub or Houdini VEX for hand-off

It opens on a generated sample frame. Multipart scanline EXRs with NONE, RLE, ZIPS or ZIP compression load in the browser; the camera comes from EXR metadata when present, or is solved from `P`. Files never leave your machine.

Live: https://x14halo.github.io/brume/
