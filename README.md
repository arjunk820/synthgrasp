# SynthGrasp

**Generate robot grasp training data from a single screenshot — no robot farm required.**

SynthGrasp replaces thousands of expensive teleoperation
demonstrations with synthetic data generated in under a minute.
Given any image of a scene, it detects objects, reconstructs the
environment in a physics simulation, and produces labeled grasp
trajectories ready for policy training.

![Industrial workshop scene](data/industrial_workshop_with_tools.png)
![Beijing living room scene](data/lavish_beijing_living_room.png)

## Why This Exists

Training robot manipulation policies today requires:

- **Physical robots + human operators** guiding every grasp by hand
- **Petabytes of demonstration data**
  (multi-camera video + sensor streams)
- **Poor generalization** — data from one environment doesn't
  transfer well

SynthGrasp eliminates all three bottlenecks.

## How It Works

```text
Screenshot → Object Detection → Physics Sim → Grasp Dataset
```

1. **Detect** — Claude Vision identifies every object in the
   scene with dimensions, mass, and material
2. **Simulate** — Objects are loaded into a Rapier 3D physics
   engine running in WebAssembly
3. **Evaluate** — Five grasp types are scored per object using
   a hybrid model (70% geometric + 30% physics)
4. **Export** — Download results as JSON or CSV, ready for training

### Grasp Types Evaluated

| Grasp | Description | Best For |
| --- | --- | --- |
| **Pinch** | Fingertip contact | Thin objects (0-5mm) |
| **Wrap** | Palm pad contact | Cylindrical objects (3-15cm) |
| **Hook** | Finger curl under handle | Objects with handles/straps |
| **BiSym** | Two-handed symmetric | Large or heavy objects |
| **BiAsym** | Two-handed asymmetric | Container + lid combos |

## Quick Start

### Prerequisites

- Python 3.x
- A modern browser
  (Chrome, Firefox, or Safari with WebGL + WebAssembly)
- [Anthropic API key](https://console.anthropic.com/) for object detection

### Optional
- [Marble/World Labs API Key](https://platform.worldlabs.ai/api-keys) for world generation

### Run It

```bash
# Clone the repo
git clone https://github.com/arjunkantamsetty/synthgrasp.git
cd synthgrasp

# Start the local server
python3 scripts/serve.py

# Open in your browser
# http://localhost:8080/pipeline/simulate.html
```

### Use It

1. Paste your Anthropic API key
2. Pick a workflow:
   - **From 3D World** — Select a pre-built environment
     and click "Detect from 3D World"
   - **From Screenshot** — Upload any image of a scene
     with graspable objects
3. Click **Run Simulation** to generate grasp trajectories
4. **Download** results as JSON or CSV

No Python packages to install. No GPU required. Everything runs in the browser.

## Project Structure

```text
synthgrasp/
├── pipeline/
│   └── simulate.html       # Main app (Three.js + Rapier + Claude)
├── data/
│   ├── *.png                # Pre-rendered scene screenshots
│   ├── *.glb                # 3D physics collider meshes
│   └── *.html               # Embedded 3D viewers
└── scripts/
    └── serve.py             # Local dev server with CORS support
```

## Tech Stack

- **[Three.js](https://threejs.org/)** — 3D rendering
- **[Rapier](https://rapier.rs/)** — Physics simulation (WebAssembly)
- **[Claude Vision](https://docs.anthropic.com/en/docs/build-with-claude/vision)**
  — Object detection and scene understanding
- **[World Labs | Marble API](https://www.worldlabs.ai/)**
  — Optional 3D world generation from images

## Output Format

Each object in the scene gets a complete grasp analysis:

```json
{
  "object": "ceramic_mug",
  "dimensions": { "width": 0.08, "height": 0.10, "depth": 0.08 },
  "mass_kg": 0.35,
  "material": "ceramic",
  "grasps": [
    { "type": "wrap", "score": 0.92, "liftable": true, "confidence": "high" },
    { "type": "hook", "score": 0.85, "liftable": true, "confidence": "high" },
    { "type": "pinch", "score": 0.31, "liftable": false,
      "confidence": "medium" }
  ]
}
```

## Contributing

Issues and PRs welcome. If you find SynthGrasp useful,
give it a star!

## License

MIT
