# luce-spz

Compact Gaussian splat files in Luce Base: Niantic's SPZ (versions 1 to 4
read, version 4 written, 3 and 2 on request) and antimatter15's headerless
`.splat`, as luce-geocore point clouds.

```luce
from luce_spz.spz import Spz

let splats = Spz.load("capture.spz")       # or "capture.splat"
Spz.save(splats, "copy.spz")               # version 4, zstd streams
Spz.save(splats, "copy.splat")
print(Spz.warnings())
```

## The cloud

Loading gives a point cloud in a 3DGS PLY's raw names, exactly what
luce-ply reads a PLY into: positions, `f_dc_0..2`, `opacity` (a logit),
`scale_0..2` (natural logs), `rot_0..3` (w, x, y, z) as f32 point attributes,
and the SH bands 1 to 3 as one array attribute `f_rest` of RGB items,
coefficient-major. The splats are in a PLY's axes (RDF: x right, y down, z
front): SPZ stores RUB (OpenGL's) unless Adobe's coordinate system extension
names another of its sixteen systems, and the stored axes are converted as
Niantic's loader converts them for a PLY, SH included. luce-geocore's Bake
GSplats then turns the names into its splat conventions (`orient`, `scale`,
`opacity`, linear `Cd`, `sh`) exactly as it does a PLY's, so a capture loads
the same from either format. An SPZ's antialiased flag becomes the detail
attribute `gsplat_antialiased`. SH band 4 is dropped with a warning; other
extension records are skipped with one.

Saving takes such a cloud, or a baked splat cloud, which is unbaked first: a
cloud baked from a Y-down file (`gsplat_up_axis`) goes back to RDF and is
converted to RUB; one without it is Y-up already and written as it is.

## Exactness

Decoding follows the reference loader's f32 arithmetic: on Niantic's
`hornedlizard.spz` every value matches the reference library's bit for bit
except alpha bytes 0 and 255 (here a finite logit a quarter step inside, which
encodes back to the same byte; the reference gives infinities) and the
first-three rotations' w within 6e-6. Encoding quantizes as the reference
packer does: the tests' fixtures, written by Niantic's library, come out of
the encoder byte for byte, and decoding then encoding a file gives its streams
back. On a 1.16M-splat capture a few hundred values (0.03%) land one step
apart from the reference's, where its compiler fused a multiply-add. `.splat`
round trips its bytes.

## Speed

Splats unpack and pack in parallel blocks on luce-std's pool, and version 4's
six zstd streams decompress and compress in parallel (luce-compress's zstd).

## Tests

`luc test` runs `tests/spz`: encoding against the reference fixtures (versions
2, 3 and 4, SH degrees 3 and 0), decoding within quantization, malformed,
truncated and damaged files, version 1's f16 positions, all sixteen coordinate
systems (conversions invert; SH colors follow the axes), the coordinate
extension, `.splat`, the editor's decode, Bake GSplats and encode round trip,
allocation failures, and a million-splat timing. `tests/fixtures/README.md`
says how the fixtures were made.
