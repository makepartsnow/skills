# Exporting a part for manufacturing

Send one solid part per file. Prefer **STEP** (`.step`/`.stp`): it keeps exact geometry, and CNC machining requires it. **STL**, **3MF** and **OBJ** work for 3D printing only. Model in millimeters and pass `"units": "mm"` in the quote config (or `"in"` if the file is in inches); a part that looks 25 times too small or too large was quoted in the wrong units.

Before exporting, check that the part is one closed solid (watertight), within the build volumes in `GET /capabilities` (SLS 165 x 165 x 300 mm), and has walls at least the process's `min_wall_mm` (0.8 mm for SLS PA12). The analysis reports problems in `pricing.warnings`.

## Code CAD

CadQuery:

```python
import cadquery as cq

part = cq.Workplane("XY").box(40, 20, 10).faces(">Z").hole(5)
cq.exporters.export(part, "out/bracket.step")  # or "out/bracket.stl"
```

build123d:

```python
from build123d import Box, export_step, export_stl

part = Box(40, 20, 10)
export_step(part, "out/bracket.step")
export_stl(part, "out/bracket.stl")
```

OpenSCAD (STL or 3MF only; no STEP, so not for CNC):

```bash
openscad -o out/bracket.stl bracket.scad
openscad -o out/bracket.3mf bracket.scad
```

FreeCAD (Python console or `freecadcmd`):

```python
import FreeCAD, Import

doc = FreeCAD.openDocument("bracket.FCStd")
Import.export([doc.getObject("Body")], "out/bracket.step")
```

trimesh (an existing mesh to STL):

```python
import trimesh

trimesh.load("model.obj").export("out/model.stl")
```

## Desktop CAD

- **Fusion:** File > Export > STEP (`.step`), or right-click the body > Save as Mesh for STL/3MF.
- **SolidWorks:** File > Save As > STEP AP214 (`.step`).
- **Onshape:** right-click the Part Studio tab > Export > STEP.
- **Inventor:** File > Export > CAD Format > STEP.
- **Rhino:** File > Export Selected > STEP (`.stp`).

Write exports to a build or output folder, not over the user's source files, and don't commit large exports unless the user wants them in the repo.
