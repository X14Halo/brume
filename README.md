# Brume

A browser prototype for laying out fog from utility passes.

Load a frame's utility EXR (world position `P`, normals `N`, alpha, and a beauty or surface colour pass) and Brume:

- raymarches fog straight against the plate, with per-pixel holdouts from `P`
- rebuilds the set from `P` as a rough mesh or point cloud you can orbit with Maya navigation (Alt + LMB / MMB / RMB)
- reads the ground, hollows, walls and canopy from the passes, so ground fog, pooled fog, mist, fog banks, pockets and veils settle where real fog would
- lets you paint fog in or clear it out, check value structure (Values / Value bands), and copy a Nuke stub or Houdini VEX for hand-off

It opens on a generated sample frame. Multipart scanline EXRs with NONE, RLE, ZIPS or ZIP compression load in the browser; the camera comes from EXR metadata when present, or is solved from `P`. Files never leave your machine.

Live: https://x14halo.github.io/brume/
