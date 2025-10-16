# USDMultiMeshWriter

Small helper to write **multiple deforming triangle meshes** to a single **OpenUSD** file with time samples, optional per-vertex velocities, and Y↔Z up-axis remapping.

- **Multi-object**: one mesh per object under `/World/<Name>`
- **Time-varying**: `points` (and optionally `velocities`) are time-sampled
- **Up-axis aware**: remap from sim **Y/Z-up** to stage **Y/Z-up**
- **USD-native**: uses `pxr` (`Usd/UsdGeom/Sdf/Gf/Vt`) + NumPy only

---

## Requirements

- Python 3.11 =< 
- `numpy`
- usd-core 

---

## Install (dev)

```
git clone https://github.com/<you>/usdmultimeshwriter.git
cd usdmultimeshwriter
pip install -e .
```

## Example Usage  
```
import numpy as np
from usdmultimeshwriter import USDMultiMeshWriter

w = USDMultiMeshWriter("out.usdc", fps=24, stage_up="Z", mesh_up="Y", write_velocities=True)
w.open()

# topology once (triangles)
F = np.array([[0,1,2],[2,1,3]], dtype=np.int32)
counts = np.array([3,3]); indices = F.flatten()
w.add_mesh("Left",  counts, indices, num_points=4)
w.add_mesh("Right", counts, indices, num_points=4)

baseL = np.array([[-1,0,0],[0,0,0],[-1,1,0],[0,1,0]], dtype=np.float32)
baseR = np.array([[ 0,0,0],[1,0,0],[ 0,1,0],[1,1,0]], dtype=np.float32)

for k in range(60):
    fall = -0.02 * k  # Y-down in sim
    VL = baseL.copy(); VL[:,1] += fall
    VR = baseR.copy(); VR[:,1] -= fall
    w.write_points("Left", VL,  timecode=k)
    w.write_points("Right", VR, timecode=k)

w.close()
```
