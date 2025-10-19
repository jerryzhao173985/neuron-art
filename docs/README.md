# Neural Network Visualization - Technical Documentation

## Overview

This directory contains technical documentation for critical timing and synchronization systems in the neural network visualization.

## Core Systems

### 1. Training-Animation Synchronization
**File:** [training-animation-synchronization.md](training-animation-synchronization.md)

**What it solves:** Disconnection between training speed and animation speed

**Key concept:**
```javascript
effectiveAnimationSpeed = trainingSpeed × (flowSpeed / 1.5)
```

All visual elements (particles, phase transitions) now synchronize with training speed, creating realistic simulation.

---

### 2. Particle Density Control
**File:** [particle-density-control.md](particle-density-control.md)

**What it solves:** Particle accumulation at slow speeds (167 particles vs 6 particles)

**Key concept:**
```javascript
adjustedInterval = baseInterval / effectiveAnimationSpeed
```

Particle creation rate adjusts inversely to speed, maintaining constant visual density (~17 particles).

---

### 3. Pause/Resume Timing
**File:** [pause-resume-timing.md](pause-resume-timing.md)

**What it solves:** Timing issues from p5.js global variables (`frameCount`, `random()`)

**Key concept:**
```javascript
let trainingFrameCount = 0;  // Only increments when training
```

Custom counter that pauses with training, preventing particle bursts and random state desync.

---

## System Dependencies

These three systems work together:

```
┌─────────────────────────────────────┐
│   Training Speed Control (1-10)    │
│   Animation Multiplier (0.1-3.0)   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  effectiveAnimationSpeed            │
│  = trainingSpeed × (flowSpeed/1.5)  │
└──────────────┬──────────────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
┌─────────────┐  ┌─────────────────┐
│  Particle   │  │  Particle       │
│  Speed      │  │  Creation Rate  │
│             │  │  = base/speed   │
└─────────────┘  └─────────────────┘
       │                │
       └────────┬───────┘
                ▼
       ┌─────────────────┐
       │ Constant Density│
       │ ~17 particles   │
       └─────────────────┘

       All using trainingFrameCount
       (pauses with training)
```

## Quick Reference

### Problem-Solution Map

| Problem | Solution | Document |
|---------|----------|----------|
| Training finishes but animations continue | Synchronized speed | [training-animation-synchronization.md](training-animation-synchronization.md) |
| 167 particles at slow speed vs 6 at fast | Density control | [particle-density-control.md](particle-density-control.md) |
| Particle bursts after resume | trainingFrameCount | [pause-resume-timing.md](pause-resume-timing.md) |
| Synapse flashing when paused | Deterministic hash | [pause-resume-timing.md](pause-resume-timing.md) |
| Different particles after pause | Deterministic hash | [pause-resume-timing.md](pause-resume-timing.md) |

### Code Locations

**Core variables:**
- Line 803: `trainingFrameCount`
- Line 806: `effectiveAnimationSpeed`
- Line 799-800: `frozenPredictions`, `frozenNetworkState`

**Synchronization:**
- Line 1074: Calculate `effectiveAnimationSpeed`
- Line 1094: Phase animation
- Line 1367: Particle speed
- Line 1529: Particle creation interval

**Timing:**
- Line 1077: Increment `trainingFrameCount`
- Line 847: Reset `trainingFrameCount`
- Line 1447: Deterministic synapse hash
- Line 1532: Deterministic particle hash

## Usage Examples

### Ultra Slow Motion (NEW!)
```
Training Speed: 0.1
Animation Multiplier: 1.5
→ effectiveSpeed = 0.1
→ 1 epoch every 10 frames
→ Perfect for detailed observation of gradients and particles
```

### Slow Motion Analysis
```
Training Speed: 0.5
Animation Multiplier: 1.5
→ effectiveSpeed = 0.5
→ 1 epoch every 2 frames
→ Slower but still observable
```

### Realistic Simulation
```
Training Speed: 3
Animation Multiplier: 1.5
→ effectiveSpeed = 3.0
→ Balanced, ~17 particles
```

### Fast Experiments
```
Training Speed: 10
Animation Multiplier: 1.5
→ effectiveSpeed = 10.0
→ Rapid, ~17 particles
```

## Testing Checklist

### Synchronization Test
- [ ] Set Training Speed = 10
- [ ] Training and animations finish together
- [ ] Set Training Speed = 1
- [ ] Training and animations both slow

### Density Test
- [ ] Set Animation Multiplier to 0.1 (slow)
- [ ] Count particles: should be ~15-20
- [ ] Set Animation Multiplier to 3.0 (fast)
- [ ] Count particles: should be ~15-20

### Pause Test
- [ ] Pause during training
- [ ] No synapse flashing
- [ ] Particles frozen in place
- [ ] Resume smoothly (no bursts)

## Mathematical Foundations

### Constant Density Proof

```
Density = ParticleLifetime / CreationInterval

ParticleLifetime = 1 / (baseSpeed × effectiveSpeed)
CreationInterval = baseInterval / effectiveSpeed

Density = [1/(baseSpeed × effectiveSpeed)] / [baseInterval/effectiveSpeed]
        = 1 / (baseSpeed × baseInterval)
        = constant (independent of effectiveSpeed)
```

### Synchronization Formula

```
effectiveSpeed = trainingSpeed × (flowSpeed / 1.5)

Where:
- trainingSpeed ∈ [1, 10]: epochs per frame
- flowSpeed ∈ [0.1, 3.0]: animation multiplier
- 1.5: normalization constant (default flowSpeed)
```

## Version History

### Current Implementation
- Synchronized training and animation speed
- Constant particle density
- Deterministic pause/resume timing
- Zero CPU usage when paused

### Previous Issues (Fixed)
- Independent training/animation speeds
- Particle accumulation (167 particles at slow speed)
- Particle bursts after resume
- Random state desync
- Synapse flashing during pause

## File Structure

```
docs/
├── README.md (this file)
├── training-animation-synchronization.md
├── particle-density-control.md
└── pause-resume-timing.md
```

## Contributing

When modifying timing or synchronization systems:

1. Update relevant documentation
2. Test all three scenarios: slow, normal, fast
3. Test pause/resume behavior
4. Verify particle density stays constant
5. Check that training and animations sync

## References

**Main implementation file:**
- `neural_illumination.html` (lines 798-1550 contain all critical systems)

**Key parameters:**
- `params.trainingSpeed`: 1-10
- `params.flowSpeed`: 0.1-3.0 (now animation multiplier)
- `trainingFrameCount`: Custom frame counter
- `effectiveAnimationSpeed`: Calculated synchronized speed
