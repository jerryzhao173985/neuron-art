# Training-Animation Synchronization

## Problem Statement

**Disconnect Between Training and Animation:**
- Training speed controlled by `trainingSpeed` parameter (1-10 epochs per frame)
- Animation speed controlled by `flowSpeed` parameter (0.1-3.0 multiplier)
- These were **completely independent**, causing unrealistic simulation

**Example of Problem:**
```
Training Speed = 10 (fast)
Flow Speed = 0.1 (slow)
→ Training completes in 1 second
→ Particles barely start moving
→ Unrealistic: animations lag behind training
```

## Solution: Synchronized Speed Model

### Core Formula

```javascript
effectiveAnimationSpeed = trainingSpeed × (flowSpeed / 1.5)
```

**Components:**
- `trainingSpeed`: Controls epochs per frame (1-10)
- `flowSpeed`: Now acts as animation multiplier (0.1-3.0)
- `1.5`: Normalization constant (default flowSpeed value)
- `effectiveAnimationSpeed`: Unified speed for all animations

### What Gets Synchronized

All visual elements now use `effectiveAnimationSpeed`:

1. **Phase Transition**
   ```javascript
   phaseProgress += 0.012 * effectiveAnimationSpeed
   ```

2. **Particle Movement**
   ```javascript
   particle.speed = random(0.015, 0.025) * effectiveAnimationSpeed
   ```

3. **Particle Creation Rate**
   ```javascript
   adjustedInterval = baseInterval / effectiveAnimationSpeed
   ```

## Implementation Details

### Changed Variables

```javascript
// Added global variables
let effectiveAnimationSpeed = 1.5;  // Line 809
let epochAccumulator = 0;  // Line 806 (for fractional training speeds)

// Calculate in draw() loop (line 1074)
effectiveAnimationSpeed = params.trainingSpeed * (params.flowSpeed / 1.5);
```

### Fractional Training Speed Support

For training speeds < 1.0, epochs accumulate until reaching 1.0:

```javascript
// Training Speed = 0.1
Frame 1: epochAccumulator = 0.1
Frame 2: epochAccumulator = 0.2
...
Frame 10: epochAccumulator = 1.0 → Run 1 epoch, reset to 0

// Result: 1 epoch every 10 frames (3 epochs/second at 30fps)
```

For training speeds >= 1.0, run multiple epochs per frame:

```javascript
// Training Speed = 3.0
Each frame: Run 3 epochs

// Result: 90 epochs/second at 30fps
```

### Changed Components

| Component | Before | After |
|-----------|--------|-------|
| Phase animation | `params.flowSpeed` | `effectiveAnimationSpeed` |
| Particle speed | `params.flowSpeed` | `effectiveAnimationSpeed` |
| Creation interval | `params.flowSpeed` | `effectiveAnimationSpeed` |

### UI Changes

**Label updated:**
- Old: "Flow Speed"
- New: "Animation Multiplier (syncs with Training Speed)"

## Usage

### Primary Control: Training Speed (0.1-10)

Controls both training rate AND animation rate:
- **0.1**: Very slow training (1 epoch every 10 frames), very slow animations - perfect for observation
- **0.5**: Slow training (1 epoch every 2 frames), slow animations
- **1.0**: Moderate training (1 epoch per frame), moderate animations
- **3.0**: Default training (3 epochs per frame), default animations
- **10.0**: Fast training (10 epochs per frame), fast animations

### Secondary Control: Animation Multiplier (0.1-3.0)

Fine-tunes animation speed relative to training:
- **0.5**: Animations run slower than training (for detailed observation)
- **1.5**: Animations match training 1:1 (realistic simulation)
- **3.0**: Animations run faster than training (for dramatic effect)

### Calculation Examples

```
Training Speed = 0.1, Multiplier = 1.5
→ effectiveAnimationSpeed = 0.1 × (1.5/1.5) = 0.1
→ 1 epoch every 10 frames, ultra-slow animations

Training Speed = 0.5, Multiplier = 1.5
→ effectiveAnimationSpeed = 0.5 × (1.5/1.5) = 0.5
→ 1 epoch every 2 frames, slow animations

Training Speed = 3, Multiplier = 1.5
→ effectiveAnimationSpeed = 3 × (1.5/1.5) = 3.0
→ 3 epochs per frame, normal animations

Training Speed = 10, Multiplier = 1.5
→ effectiveAnimationSpeed = 10 × (1.5/1.5) = 10.0
→ 10 epochs per frame, fast animations

Training Speed = 5, Multiplier = 0.3
→ effectiveAnimationSpeed = 5 × (0.3/1.5) = 1.0
→ 5 epochs per frame, but slow animations
```

## Benefits

### Before Synchronization
- ❌ Training completes while animations still running
- ❌ Training slow, animations fast (or vice versa)
- ❌ Disconnected, unrealistic simulation

### After Synchronization
- ✅ Training and animations complete together
- ✅ Training speed = animation speed (by default)
- ✅ Realistic neural network simulation
- ✅ Fine-tuning available via multiplier

## Technical Notes

### Particle Density Preservation

Creation interval adjusts inversely to speed:
```javascript
adjustedInterval = baseInterval / effectiveAnimationSpeed
```

This maintains **constant visual density** regardless of speed:
- Fast speed → particles created more often → same density
- Slow speed → particles created less often → same density

### Pause/Resume Behavior

Synchronization respects pause state:
- `effectiveAnimationSpeed` calculated only during training
- Frozen state preserves last calculated value
- Resume continues with current training speed

## File Locations

**Modified lines in `neural_illumination.html`:**
- Line 806: Added `epochAccumulator` variable
- Line 809: Added `effectiveAnimationSpeed` variable
- Line 314: Training Speed slider range (0.1-10, step 0.1)
- Line 2116: Training Speed display formatting (decimal)
- Line 1074: Calculate `effectiveAnimationSpeed` in draw loop
- Line 1084-1105: Fractional training speed implementation
- Line 1107: Phase animation uses `effectiveAnimationSpeed`
- Line 1373: Particle speed uses `effectiveAnimationSpeed`
- Line 1540: Creation interval uses `effectiveAnimationSpeed`
- Line 312: UI label updated with "(epochs per frame)"
- Line 380: Animation Multiplier label updated

## Related Documentation

- [Pause/Resume Functionality](pause-resume-mechanism.md) (if exists)
- [Particle System](particle-system.md) (if exists)
- [Training Speed Control](training-controls.md) (if exists)

---

# Fractional Training Speed (0.1-10)

## Problem

**Training was too fast even at minimum speed:**
- Training Speed minimum was 1.0 (1 epoch per frame)
- At 30fps: 30 epochs per second
- 500 epochs completed in ~16 seconds
- Animations (particles, gradients, phase transitions) barely visible
- **Could not observe individual training cycles**

## Solution

Extended Training Speed range to **0.1 - 10.0** with fractional epoch support.

### New Range

| Training Speed | Epochs Per Frame | Epochs Per Second (30fps) | Time for 500 Epochs |
|----------------|------------------|---------------------------|---------------------|
| **0.1** | 0.1 (1 every 10 frames) | 3 | ~166 seconds (~2.7 min) |
| **0.2** | 0.2 (1 every 5 frames) | 6 | ~83 seconds (~1.4 min) |
| **0.5** | 0.5 (1 every 2 frames) | 15 | ~33 seconds |
| **1.0** | 1 | 30 | ~16 seconds |
| **3.0** | 3 | 90 | ~5.5 seconds |
| **10.0** | 10 | 300 | ~1.6 seconds |

## Implementation

### Epoch Accumulator System

```javascript
let epochAccumulator = 0;  // Accumulates fractional epochs

if (params.trainingSpeed >= 1.0) {
    // Fast mode: run multiple epochs per frame
    const epochsToRun = Math.floor(params.trainingSpeed);
    for (let i = 0; i < epochsToRun; i++) {
        trainEpoch();
    }
}
else {
    // Slow mode: accumulate until we have 1 full epoch
    epochAccumulator += params.trainingSpeed;
    if (epochAccumulator >= 1.0) {
        epochAccumulator -= 1.0;
        trainEpoch();
    }
}
```

### How It Works

**Training Speed = 0.1:**
```
Frame 1: accumulator = 0.1
Frame 2: accumulator = 0.2
Frame 3: accumulator = 0.3
...
Frame 10: accumulator = 1.0 → Train 1 epoch, reset to 0
Frame 11: accumulator = 0.1
...
```

**Training Speed = 0.5:**
```
Frame 1: accumulator = 0.5
Frame 2: accumulator = 1.0 → Train 1 epoch, reset to 0
Frame 3: accumulator = 0.5
Frame 4: accumulator = 1.0 → Train 1 epoch, reset to 0
...
```

**Training Speed = 3.0:**
```
Each frame: Train 3 epochs immediately
(No accumulation needed)
```

## Benefits

### Before (Minimum = 1.0)
- ❌ Too fast to observe animations
- ❌ Particles complete only 1-2 cycles
- ❌ Gradient progress bar barely visible
- ❌ Cannot see individual forward/backward passes
- ❌ Training completes in 5-16 seconds

### After (Minimum = 0.1)
- ✅ Ultra-slow motion available (0.1 = 2.7 minutes for 500 epochs)
- ✅ Particles complete many visible cycles
- ✅ Gradient progress bar clearly visible
- ✅ Individual forward/backward passes observable
- ✅ Full control from ultra-slow to ultra-fast

## Recommended Settings

### For Detailed Observation
```
Training Speed: 0.1 - 0.3
Animation Multiplier: 1.5
→ Watch individual particles flow
→ See gradient propagation clearly
→ Observe each forward/backward cycle
```

### For Understanding Training
```
Training Speed: 0.5 - 1.0
Animation Multiplier: 1.5
→ See training progress clearly
→ Animations sync with learning
→ Good balance of speed and observation
```

### For Quick Experiments
```
Training Speed: 3.0 - 10.0
Animation Multiplier: 1.5
→ Fast prototyping
→ Quick parameter testing
→ Rapid iteration
```

## Synchronization Impact

Fractional training speeds fully integrate with animation synchronization:

```javascript
effectiveAnimationSpeed = trainingSpeed × (flowSpeed / 1.5)

// Training Speed = 0.1, Multiplier = 1.5
effectiveAnimationSpeed = 0.1 × (1.5/1.5) = 0.1

// This affects:
// - Phase animation speed
// - Particle movement speed
// - Particle creation rate
```

**Result:** All animations slow down proportionally with training.

## UI Changes

### Slider
```html
<!-- Before -->
<input type="range" id="trainingSpeed"
       min="1" max="10" step="1" value="3">

<!-- After -->
<input type="range" id="trainingSpeed"
       min="0.1" max="10" step="0.1" value="3">
```

### Display
```javascript
// Before
displayElement.textContent = parseInt(value);  // "1", "2", "3"

// After
displayElement.textContent = parseFloat(value).toFixed(1);  // "0.1", "0.5", "1.0"
```

### Label
```html
<!-- After -->
<label>Training Speed <small>(epochs per frame)</small></label>
```

## Technical Details

### Precision
- Step size: 0.1
- Range: 0.1 to 10.0 (100 discrete values)
- Display: 1 decimal place (e.g., "0.1", "1.5", "10.0")

### Reset Behavior
```javascript
function initializeTraining() {
    epochAccumulator = 0;  // Always reset
    // ...
}
```

### Pause Behavior
```javascript
// When paused, accumulator holds its value
// When resumed, continues from held value
// Example:
// Pause at accumulator = 0.7
// Resume continues: 0.7 → 0.8 → 0.9 → 1.0 → epoch!
```

## Testing Checklist

### Ultra-Slow Mode (0.1)
- [ ] Set Training Speed to 0.1
- [ ] Training should run 1 epoch every 10 frames
- [ ] Animations should be ultra-slow
- [ ] Particles should flow slowly and visibly
- [ ] Should take ~2.7 minutes for 500 epochs

### Fractional Values (0.5)
- [ ] Set Training Speed to 0.5
- [ ] Training should run 1 epoch every 2 frames
- [ ] Should take ~33 seconds for 500 epochs
- [ ] Animations should be slow but smooth

### Integer Values (3.0)
- [ ] Set Training Speed to 3.0
- [ ] Training should run 3 epochs per frame
- [ ] Should take ~5.5 seconds for 500 epochs
- [ ] Animations should match previous behavior

## File Locations

**Modified code in `neural_illumination.html`:**
- Line 806: `epochAccumulator` variable
- Line 314: Slider min/max/step
- Line 2116: Display formatting (decimal)
- Line 1084-1105: Fractional epoch logic
- Line 854: Reset accumulator in `initializeTraining()`

## Related Documentation

- [Particle Density Control](particle-density-control.md)

