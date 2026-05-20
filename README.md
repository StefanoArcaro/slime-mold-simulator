# Slime Mold Simulator — WebGL

## What this is
A real-time slime mold (Physarum polycephalum) simulation running entirely on the GPU via WebGL fragment shaders. Each pixel participates in the simulation — agents deposit a chemical trail, sense nearby trail concentrations, and steer toward them. The result is emergent, organic-looking networks that form and dissolve over time.

This is a fun side project with no production goals. Priorities are: **it looks alive**, **the code is understandable**, and **there are interesting things to tweak**.

---

## How it works (the model)
The simulation runs on two textures that ping-pong each frame:

1. **Agent texture** — each pixel stores an agent's position and heading angle
2. **Trail texture** — a scalar field of deposited chemical trail, which diffuses and decays each frame

Each frame, two passes run:
- **Agent pass**: read agent state → sense trail at three points ahead → steer toward strongest → move forward → deposit trail
- **Trail pass**: diffuse (blur) the trail map → decay it by a constant factor

Agents are initialized at random positions with random headings, or optionally seeded from a pattern (circle, point cluster, etc).

---

## Scope & constraints

### Must have
- Single HTML file (no build tools, no dependencies, just open in browser)
- Two-texture ping-pong simulation loop in WebGL fragment shaders
- At minimum: 512×512 simulation grid at 60fps; ideally 1024×1024
- A small, visible UI panel (plain HTML overlaid on canvas) with sliders for the key parameters (see below)
- Reset button that re-randomizes agents

### Key parameters to expose
| Parameter | What it does |
|---|---|
| `sensorAngle` | Angle between the three forward sensors |
| `sensorDistance` | How far ahead agents sample the trail |
| `turnSpeed` | How sharply agents steer |
| `depositAmount` | How much trail each agent lays per frame |
| `decayRate` | How fast the trail fades |
| `diffuseStrength` | How much the trail spreads/blurs |

### Nice to have (do these if they fall naturally, skip if they complicate things)
- A dropdown to switch agent spawn patterns (random, circle, center point)
- Trail color theme picker (e.g. classic yellow-on-black, bioluminescent blue, etc)
- Ability to click the canvas to add a burst of new agents at that point

### Out of scope
- No audio
- No saving/exporting
- No mobile/touch support
- No React, no bundler, no npm — single file only

---

## File structure
```
index.html   ← everything lives here
```

---

## Aesthetic goal
It should look like something biological. The default parameter values should produce clearly visible, actively reorganizing filament networks within a few seconds of load. Avoid defaults that produce static noise or a uniform grey blob — the whole point is watching it *move*.

---

## Notes for the agent
- The WebGL boilerplate (context setup, texture creation, framebuffer ping-pong) is the tedious part — get that scaffolded first and verify a basic render loop works before implementing simulation logic
- GLSL fragment shaders can't "move" data between pixels directly; agent movement is encoded as: each pixel recomputes where its agent *would* be after moving, then looks up what agent would have landed on it. This is the key conceptual trick.
- Use `NEAREST` texture filtering for the agent texture, `LINEAR` for the trail texture
- If debugging is needed, a simple pass-through shader that renders the trail texture directly is useful early on
