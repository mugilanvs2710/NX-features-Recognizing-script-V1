# 🔩 NX Geometric Feature Extractor

> **Automatically analyze any 3D CAD model in Siemens NX and extract every geometric feature — its type, exact location, and dimensions — saved to a structured report.**

[![NX Version](https://img.shields.io/badge/Siemens%20NX-v2506-0078D4?style=flat-square)](https://plm.sw.siemens.com/en-US/nx/)
[![Language](https://img.shields.io/badge/Language-Python%20%28NX%20Open%29-3776AB?style=flat-square)](https://docs.sw.siemens.com/en-US/doc/209349590/PL20201002100200012.xid1873320/xid1873333)
[![Output](https://img.shields.io/badge/Output-TXT%20%2B%20CSV-28A745?style=flat-square)]()
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)]()

---

## 🧠 What Is This?

When you design a 3D part in CAD software, it contains many **geometric features** — holes, fillets (rounded edges), chamfers (angled cuts), flat faces, and more. Normally, extracting detailed information about each of these features requires manually clicking through menus or reading a feature tree.

**This tool automates that entire process.**

You run a single script inside Siemens NX, and it:
1. Walks through every surface (face) of your 3D model
2. Identifies what type of geometric feature each face belongs to
3. Records its **exact position in 3D space** (X, Y, Z coordinates)
4. Records its **dimensions** (diameter, depth, radius, angle, etc.)
5. Saves everything to a clean `.txt` report **and** a `.csv` spreadsheet

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🔍 **B-Rep Geometry Analysis** | Reads raw surface geometry — works on native `.prt` files AND dumb STEP/IGES imports |
| 🎯 **No Feature Tree Required** | Does not rely on the model history or feature tree — finds features from pure geometry |
| 📍 **MCS Coordinates** | Reports the position of every feature in the Model Coordinate System (mm) |
| 📐 **Full Dimensions** | Diameter, depth, radius, angle — whatever applies to that feature type |
| 📄 **Dual Output** | Saves both a human-readable `.txt` report and an Excel-ready `.csv` file |
| ⚡ **One-Click Execution** | Runs as a single NX Journal (macro) — no installation, no dependencies |

---

## 🏗️ What Features Does It Detect?

```
┌─────────────────────────┬──────────────────────────────────────────────┐
│ Feature Type            │ What It Captures                             │
├─────────────────────────┼──────────────────────────────────────────────┤
│ HOLE — Simple           │ Diameter, Depth, Axis Direction              │
│ HOLE — Counterbore      │ Hole Ø, Hole Depth, CB Ø, CB Depth          │
│ HOLE — Countersink      │ Hole Ø, Hole Depth, CS Ø, CS Angle          │
│ FILLET / ROUND          │ Fillet Radius                                │
│ CHAMFER                 │ Chamfer dimensions (D1 × D2)                 │
│ PLANAR FACE             │ Edge lengths, face area                      │
│ BOSS / PIN              │ Diameter (approx), axial length              │
│ CONICAL FACE            │ Edge extents                                 │
└─────────────────────────┴──────────────────────────────────────────────┘
```

---

## 🛠️ How It Works (Simple Explanation)

Think of a 3D model as being made up of hundreds of individual **surfaces** (called faces). Each face has a mathematical type — flat, cylindrical, spherical, cone-shaped, or torus-shaped.

This script interrogates each face using the **NX Open Python API** and asks:

- Is this face cylindrical AND does it belong to a hole feature? → **HOLE**
- Is this face a blend (smoothly curved junction between two faces)? → **FILLET**
- Is this face a cone shape at a sharp edge? → **CHAMFER**
- Is this face completely flat? → **PLANAR FACE**

For each identified feature, it then reads the actual geometric data (coordinates, diameter, depth, etc.) directly from the NX kernel — not from labels or the feature tree.

---

## 📁 Project Structure

```
nx-feature-extractor/
│
├── NX_Feature_Extractor_v1.py    ← The main macro (run this in NX)
│
├── diagnostics/                   ← Development diagnostics (documented proof)
│   ├── diag1_api_discovery.py     ← Found correct API methods
│   ├── diag2_return_structures.py ← Confirmed return types
│   ├── diag3_enum_values.py       ← Mapped SolidFaceType integers
│   └── diag4_coordinates.py      ← Verified coordinate units
│
├── sample_output/
│   ├── model1_FEATURES.txt        ← Sample human-readable report
│   └── model1_FEATURES.csv        ← Sample spreadsheet output
│
├── docs/
│   ├── model_screenshot_1.png     ← Test model (isometric view)
│   ├── model_screenshot_2.png     ← Test model (front-face view)
│   └── project_report.pdf         ← Full technical project report
│
└── README.md                      ← This file
```

---

## 🚀 How to Run

### Prerequisites
- Siemens NX **v2506** (verified) — may work on nearby versions
- No additional Python packages required (uses NX's built-in Python)

### Steps

**1. Open your part in NX**
```
File → Open → select your .prt (or import your STEP/IGES file)
```

**2. Run the journal**
```
Menu → Tools → Journal → Play Journal
→ Browse to NX_Feature_Extractor_v1.py
→ Click OK
```

**3. Done!**

Two files are saved automatically in the same folder as your `.prt`:
```
YourPart_FEATURES.txt    ← Open in Notepad / any text editor
YourPart_FEATURES.csv    ← Open in Excel
```

---

## 📊 Sample Output

### TXT Report (excerpt)
```
============================================================================
  NX GEOMETRIC FEATURE EXTRACTION REPORT
============================================================================
  Part        : model1
  Total feats : 11
  Coordinates : MCS  (mm)

  SUMMARY
----------------------------------------------------------------------------
    CHAMFER                                           2  instance(s)
    FILLET / ROUND                                    2  instance(s)
    HOLE                                              1  instance(s)
    PLANAR FACE                                       6  instance(s)

----------------------------------------------------------------------------
  ID        : #0011   Body: Body-1
  Type      : HOLE — SIMPLE

  MCS Position (feature centre)
       X =      +265.0000 mm
       Y =      +160.0000 mm
       Z =      +115.0000 mm

  Dimensions
       Diameter         (mm)               = +110.0000
       Depth            (mm)               = +230.0000
       Entry Chamfer Angle (deg)           = +45.0000
```

### CSV Output
The `.csv` file contains one row per feature with all dimensions in columns — ready to filter, sort, and process in Excel or any data tool.

---

## 🔬 Verified API Reference (NX v2506)

This project was built through **live diagnostics** run directly in NX 2506, not from assumptions. Every API call below was confirmed to work:

```python
face.SolidFaceType.value     # int: 1=Planar, 2=Cylinder, 3=Cone, 5=Torus
face.GetHoleData()           # → (ResizeHoleData, bool) or None
face.GetBlendData()          # → (radius: float, is_blend: bool)
face.GetChamferData()        # → (is_chamfer: bool, type_enum, [d1, d2])
face.GetEdges()              # → list[Edge]
face.JournalIdentifier       # str: "FACE N {(X,Y,Z) FEATURE_NAME}"
edge.GetLength()             # → float (mm)
edge.GetLocations()          # → list[CurveLocation]
CurveLocation.Location       # → Point3d (mm)
ResizeHoleData.GetHoleDiameter()  # → float (mm)
ResizeHoleData.GetHoleDepth()     # → float (mm)
ResizeHoleData.GetOrigin()        # → Point3d (METRES — multiply ×1000)
ResizeHoleData.GetDirection()     # → Vector3d (unit vector)
```

> **Important discovery:** `ResizeHoleData.GetOrigin()` returns coordinates in **metres**, not mm. Multiply by 1000. This was confirmed by comparing against `JournalIdentifier` coordinate parsing.

---

## 🗺️ Coordinate System

All coordinates are reported in the **Model Coordinate System (MCS)** — the absolute origin of the NX part file. This is the same coordinate system shown in the NX viewport triad (the X/Y/Z axes in the corner of the screen).

Units: **millimetres (mm)**

---

## 🐛 Known Limitations

- **Axis direction** for planar faces and chamfers is currently reported as `[0, 0, 1]` (default). True face normals require probing UV parameters — a future enhancement.
- **Boss/Pin diameter** is approximated from the longest circular edge circumference (`L / π`), not read from a dedicated API method, since NX does not expose a `GetBossData()` equivalent.
- Does **not** currently detect threads, knurls, or surface finish annotations.

---

## 🔭 Future Roadmap

- [ ] True face normal vectors using `uf.Modl.AskFaceProps()`
- [ ] Thread detection via feature tree cross-reference
- [ ] Assembly support (multiple bodies across components)
- [ ] HTML report with 3D viewer integration
- [ ] Slot and pocket grouping (collect coplanar faces into regions)

---

## 📋 Development Log

This project was built methodically using a **diagnostic-first approach**:

| Diagnostic | Purpose | Key Finding |
|---|---|---|
| Diag 1 | Discover available API | `AskFaceData()` does NOT exist in NX 2506 |
| Diag 2 | Confirm return types | `GetHoleData()` → `ResizeHoleData` object |
| Diag 3 | Map enum integers | `SolidFaceType.value`: 1, 2, 3, 5 confirmed |
| Diag 4 | Verify coordinates | `GetOrigin()` is in metres, not mm |

No API call in the final macro is a guess — all were verified against live NX 2506 output.

---

## 📜 License

MIT License — free to use, modify, and distribute.

---

## 🙋 Author

Built with a diagnostic-first methodology for NX 2506.
Feel free to open an issue if you encounter a face type not yet handled.
