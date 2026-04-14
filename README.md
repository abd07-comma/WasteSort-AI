# ♻️ WasteSort-AI

> **Note:** This is a proprietary project. Source code is owned by the company.  
> This repository documents my personal contributions, challenges faced, and results.

Kazakhstan-based AI system for automating waste sorting in a recycling container.  
I worked on three modules as part of a larger team.

---

## My Modules

---

### 📐 Module 1 — Bin Level Monitoring

**The problem:**  
The waste platform tilts over time as trash accumulates unevenly.  
The platform needed to be leveled automatically by raising the heavier side.

**What I ran into:**  
- At the lowest motor position, the camera captures the container walls — which messes up depth readings
- Shadows inside the container caused false depth values
- Tried several depth models before finding a stable one

**What I did:**  
- Divided the camera view into 6 zones
- Received motor position values from another developer via API
- Instead of averaging motor values, I take the **minimum** and round down (e.g. 43 → 40) — this avoids accidentally including wall zones in the analysis
- Added a shadow filter using Gaussian blur to reduce noise from lighting
- Tested multiple depth models; **MiDaS** turned out to be the most stable
- When one side is consistently higher, I send a command via API to raise the opposite side by a set amount

**Still to do:**  
Action detection — if 10 frames in a row look the same, the platform is idle.  
If motion is detected, pause the leveling (someone's hand might be inside).

---

### 📦 Module 2 — Automated Dataset Collection

**The problem:**  
We needed labeled data for training the waste classification model —  
without someone manually annotating every image.

**What I ran into:**  
- The collection setup changed ~5 times during development:  
  first inside the bin, then mid-fall, then a conveyor with a gate,  
  then a lab table, now a single overhead camera on a table
- The operator always wears a **glove**, so MediaPipe hand tracking doesn't work
- The operator wears bulky clothing (a kind of apron) that partially blocks the camera view — still looking for a solution to this

**What I did:**  
- Used **SAM3** to segment the object the operator picks up, then auto-generate a bounding box annotation
- Divided the frame into **5 bin zones** — when the operator's hand enters a zone, the system records which bin the object went into and assigns the correct waste category label
- To handle the glove problem: attached an **ArUco marker** to the glove, detect it in the frame, and ignore the area around it so the system doesn't mistake the hand for an object
- The ArUco approach works in controlled conditions — still refining it for the real container environment

---

### 🦾 Module 3 — Data Collection for Robotic Arm (Future)

**The idea:**  
Eventually a robotic arm should replace the human operator.  
To train it, we need examples of how a human sorts objects.

**What I did:**  
- Recorded hand orientation, index finger and thumb coordinates frame-by-frame while the operator was sorting
- Stored the data for future use in imitation learning

**Current blocker:**  
The operator's glove and clothing make hand tracking unreliable with standard tools.  
Evaluating **OAK-D Pro** (depth + RGB camera) as a possible solution.

---

## Tech Stack

| Area | Tools |
|------|-------|
| Depth estimation | MiDaS |
| Object detection | YOLOv8, YOLOv12 (n, x variants) |
| Segmentation | SAM3 (Segment Anything) |
| Hand / marker tracking | ArUco markers, MediaPipe |
| Camera input | RTSP stream, OpenCV |
| Integration | REST API (between modules) |
| Language | Python |

---

## Visual Results

### Depth Map — 6-Zone Analysis
*MiDaS depth output used to compare bin levels across zones*

depthmapv1.png


### Zone Sectors with Depth Values
*Each sector (S1–S9) shows its average depth value in real time*

depth_map.jpg

### Live Detection — Grab Detector
*Object segmentation + bin zone classification during operator sorting session*
*ArUco marker visible on operator glove (Marker: YES)*

seg_det.jpeg
seg_det2.jpeg

---

## Context

This is part of a larger effort to increase recycling rates in Kazakhstan  
by automating the sorting process in a compact container unit.  
The container includes a hydraulic press and multiple collection bins.

The project is still in active development.

---

*Contributions shown here are my own. Full system involves a larger team.*
