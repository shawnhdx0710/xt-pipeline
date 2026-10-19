# Expected Threat (xT) Pipeline

End-to-end pipeline that trains a **32 × 24 Expected Threat grid** from
[StatsBomb Open Data](https://github.com/statsbomb/open-data), designed to run on a
consumer laptop (M1 MacBook Air, 8 GB RAM).

- 3,927 matches
- ~14M raw events → 13.7M SPADL actions
- 32 × 24 grid (768 cells)
- Resumable per-match Parquet cache
- ~17s training once the cache is built

## Layout

```
scripts/
  01_cache_spadl.py          # fetch + convert + cache matches
  02_train_xt.py             # train the grid
  03_visualize_xt.py         # 2D heatmap
  04_voxel_terrain.py        # 3D voxel PNG
  05_voxel_terrain_plotly.py # interactive HTML
  xt_utils.py                # custom StatsBomb → SPADL converter
requirements-xt.txt
sb_cache/                    # generated, gitignored (~210 MB)
output/                      # generated, gitignored
```

## Setup

Python 3.12:

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements-xt.txt
```

The pins matter: `socceraction 1.5.3` pulls `pandera 0.17.x`, which needs NumPy 1.x;
PyArrow 17+ requires NumPy 2, so the environment stays on `numpy<2` + `pyarrow<17`.

## Run

```bash
.venv/bin/python scripts/01_cache_spadl.py
.venv/bin/python scripts/02_train_xt.py
.venv/bin/python scripts/03_visualize_xt.py
.venv/bin/python scripts/04_voxel_terrain.py
.venv/bin/python scripts/05_voxel_terrain_plotly.py
```

`01` is resumable: cached matches are skipped. `02` reads the Parquet cache, trains the
grid, and writes `output/xt_grid_32x24.{npy,csv,json}`. The later scripts render the
heatmap and 3D terrain from that grid.

## Why the custom converter?

Modern `statsbombpy` returns flattened columns (`pass_end_location`, `shot_outcome`, …),
while `socceraction 1.5.3` expects the legacy nested `extra` dict. `scripts/xt_utils.py`
maps the modern format directly to SPADL, so the pipeline does not depend on an old,
unmaintained `statsbombpy`.

## Data

StatsBomb Open Data is free for research/non-commercial use with attribution. Raw event
data and the generated Parquet cache are **not** committed here.

Project page: <https://shawnhdx0710.github.io/projects/xtgrid/xtgrid/>
