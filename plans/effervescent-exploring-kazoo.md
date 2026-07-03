# Plan v2: 6-Axis Robot Arm — Adjustable Link Lengths + Presets

## Context

v1 delivered a working FK solver with fixed DH parameters (ABB GoFa). The user now wants:
- **Link lengths freely adjustable** via sliders (d1, a2, a3, d4, d6)
- **Preset buttons** for common robot models (ABB, FANUC, KUKA, UR5e, etc.)
- Everything else from v1 stays the same

## What Changes

### 1. Dynamic DH Parameters

Replace the module-level `DH_PARAMS` constant with a **builder function**:

```python
def build_dh_params(d1, a2, a3, d4, d6) -> np.ndarray:
    # alpha values and theta_offsets are structural constants
    # only a/d values vary
```

The alpha (α) angles and theta offsets are **fixed** — they define the joint *type* (revolute, parallel/perpendicular to previous). Only the lengths (a, d) change.

### 2. New Sliders (5 link-length controls)

Placed in a **second column** beside the existing joint-angle sliders:

| Slider  | Parameter     | Range        | Default |
|---------|---------------|--------------|---------|
| d1      | Base height   | 100 – 700 mm | 350     |
| a2      | Upper arm     | 100 – 600 mm | 325     |
| a3      | Forearm       |  80 – 500 mm | 275     |
| d4      | Wrist link    |  80 – 500 mm | 237     |
| d6      | Flange        |  30 – 250 mm |  85     |

### 3. Preset Buttons (5 models)

| Preset           | d1  | a2  | a3  | d4  | d6  | Reach ~ |
|------------------|-----|-----|-----|-----|-----|---------|
| ABB GoFa CRB15000| 350 | 325 | 275 | 237 |  85 | ~950 mm |
| FANUC LR Mate    | 330 | 260 | 220 | 190 |  70 | ~700 mm |
| KUKA KR6 R900    | 400 | 400 | 350 | 300 |  80 | ~900 mm |
| UR5e (approx)    | 162 | 425 | 392 | 127 |  92 | ~850 mm |
| Custom            | —   | —   | —   | —   | —   | last-set |

Clicking a preset updates all 5 link-length sliders + invalidates workspace.

### 4. DH Table Display

Add a small panel below the TCP info showing the current DH table rows (a, α, d, θ_offset) in a compact monospace format. Updates whenever link lengths change.

### 5. Workspace Invalidation

- Changing any link length **invalidates** the cached workspace point cloud.
- The workspace button shows "Workspace (outdated)" state.
- User must click to recompute (avoids lag on every slider drag).

### 6. Layout Redesign

Figure size: **19×11** (wider to fit two slider columns)

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  3D Viewport                       TCP Info Panel            │
│  [0.03, 0.28, 0.52, 0.71]         [0.58, 0.48, 0.40]        │
│                                                              │
│                                    DH Table                  │
│                                    [0.58, 0.28, 0.40]        │
│                                                              │
│                                    Rotation Matrix           │
│                                    [0.58, 0.10, 0.40]        │
│                                                              │
├─────────────────┬──────────────────┬─────────────────────────┤
│ Joint Angles    │ Link Lengths     │ Presets + Buttons       │
│ (6 sliders)     │ (5 sliders)      │ [ABB] [FANUC] [KUKA]   │
│ J1 ─────●──    │ d1 ─────●──     │ [UR5e] [Default]       │
│ J2 ─────●──    │ a2 ─────●──     │                         │
│ J3 ─────●──    │ a3 ─────●──     │ [Reset] [Workspace]    │
│ J4 ─────●──    │ d4 ─────●──     │                         │
│ J5 ─────●──    │ d6 ─────●──     │                         │
│ J6 ─────●──    │                  │                         │
└─────────────────┴──────────────────┴─────────────────────────┘
```

## Files Modified

- `C:\Users\Blackmai\.local\bin\robot_kinematics.py` — substantial rewrite of RobotApp class, build_dh_params, presets

## Verification

```powershell
python C:\Users\Blackmai\.local\bin\robot_kinematics.py
```

Manual checks:
1. 6 joint sliders + 5 link-length sliders visible and functional
2. Drag link-length slider → robot arm scales in real-time, workspace invalidated
3. Click ABB/FANUC/KUKA/UR5e preset → all 5 link sliders snap to preset values
4. DH table updates on any link change
5. Workspace button shows "outdated" after link change, recomputes on click
6. TCP pose display still works correctly
