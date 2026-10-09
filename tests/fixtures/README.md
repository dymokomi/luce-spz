# Fixtures

`synthetic3.ply` (257 splats, SH degree 3) and `synthetic0.ply` (100 splats,
degree 0) are original synthetic 3DGS PLYs: seeded uniform positions in
±3, f_dc in ±1.5, f_rest in ±0.6, opacity logits in ±6, log scales in
-7..-1 and normal-distributed quaternions.

The `reference*_v*.spz` files were written from them by Niantic's reference
library (github.com/nianticlabs/spz, MIT), `saveSpz` with `from = RDF`
(the PLY's axes converted to SPZ's RUB) and the default packing options, at
the version in the name. The library itself is not part of this repository.
