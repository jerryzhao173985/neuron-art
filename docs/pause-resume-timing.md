# Pause/Resume Timing Mechanism

## Problem Statement

**p5.js Global Variables Never Pause:**
- `frameCount`: Increments every frame (even when paused)
- `random()`: Continues generating values (even when paused)

This caused timing issues when resuming from pause.

## Root Causes

### Issue 1: Particle Burst on Resume

**Before Fix:**
```javascript
if (frameCount % 3 === 0) { createParticles(); }
```

**Problem:**
```
Pause at frameCount = 99 (not divisible by 3)
User pauses for 10 seconds (300 frames)
frameCount continues: 100, 101, 102... 399
Resume at frameCount = 399 (divisible by 3)
→ Immediate particle creation! ❌
```

### Issue 2: Random State Desync

**Before Fix:**
```javascript
if (random() < 0.05) { createParticle(); }
```

**Problem:**
```
Pause: random() at state position 1000
User pauses for 10 seconds
p5.js calls random() internally ~300 times
Resume: random() at state position 1300+
→ Different random values = different particles ❌
```

### Issue 3: Synapse Flashing

**Before Fix:**
```javascript
if (random() > sampleRate) continue;  // Different synapses every frame!
```

**Problem:**
- Each frame calls `random()` to decide which synapses to show
- Different synapses selected each frame
- Result: Visual "flashing" even when paused ❌

## Solution: Training Frame Counter

### Implementation

```javascript
// Global variable (line 803)
let trainingFrameCount = 0;

// Increment only when training (line 1077)
if (isTraining && epoch < totalEpochs) {
    trainingFrameCount++;
}

// Reset on training initialization (line 847)
trainingFrameCount = 0;
```

### Key Properties

**Pauses with training:**
```
Training → trainingFrameCount increments
Paused → trainingFrameCount frozen
Resume → trainingFrameCount continues from last value
```

**Independent of p5.js:**
- Not affected by `frameCount`
- Not affected by internal p5.js state
- Fully under our control

## Fixed Components

### 1. Particle Creation Timing

**Before:**
```javascript
if (frameCount % 3 === 0) { ... }  // ❌ Never pauses
```

**After:**
```javascript
if (trainingFrameCount % adjustedInterval === 0) { ... }  // ✅ Pauses correctly
```

**Benefit:** No particle bursts after resume

### 2. Particle Randomness → Deterministic Hash

**Before:**
```javascript
if (random() < 0.05) { ... }  // ❌ random() never pauses
```

**After:**
```javascript
const particleHash = (i * 97 + j * 53 + trainingFrameCount * 31) % 1000;
if (particleHash < 50) { ... }  // ✅ Deterministic, pauses correctly
```

**Benefits:**
- Same probability (50/1000 = 5%)
- Deterministic for same inputs
- Pauses correctly (trainingFrameCount frozen)
- Reproducible behavior

### 3. Synapse Selection → Deterministic Hash

**Before:**
```javascript
if (random() > sampleRate) continue;  // ❌ Flashing every frame
```

**After:**
```javascript
const hash = (i * 73 + j * 37 + epoch * 7) % 100;
if (hash / 100 > sampleRate) continue;  // ✅ Stable when paused
```

**Benefits:**
- When paused: `epoch` frozen → hash frozen → no flashing
- When training: `epoch` increments → hash varies → natural variety
- No random() calls

## Complete Freeze Implementation

### Frozen Predictions Cache

**Line 799-800:**
```javascript
let frozenPredictions = null;  // Cache predictions when paused
let frozenNetworkState = null;  // Cache network activations when paused
```

**On First Pause Frame:**
```javascript
// Save network state
frozenNetworkState = {
    inputActivations: [...network.inputActivations],
    hidden1Activations: [...network.hidden1Activations],
    // ... all activations and gradients
};

// Cache predictions
for (let i = 0; i < testData.length; i++) {
    network.forward(sample.data);
    frozenPredictions.push({ pred: [...network.outputActivations], ... });
}

// Restore network state (predictions changed it)
network.inputActivations = [...frozenNetworkState.inputActivations];
// ... restore all
```

**While Paused:**
```javascript
// Use cached predictions (no forward passes!)
pred = frozenPredictions[i].pred;
```

**On Resume:**
```javascript
frozenPredictions = null;  // Clear cache
frozenNetworkState = null;
```

### Particle Freeze

**Line 1536-1539:**
```javascript
if (!isTraining) {
    particle.display();  // Display only, no update = frozen
} else {
    particle.update();   // Update + display = moving
    particle.display();
}
```

### Phase Animation Freeze

**Line 1093:**
```javascript
if (isTraining && epoch < totalEpochs) {
    phaseProgress += 0.012 * effectiveAnimationSpeed;
    // Only increments when training
}
```

## Edge Cases Handled

### Long Pause
```
Pause for 10 minutes (18,000 frames @ 30fps)
frameCount: 0 → 18,000 (ignored)
trainingFrameCount: 100 → 100 (frozen)
Resume: Continues from frame 100 ✅
```

### Multiple Pause/Resume Cycles
```
Train to frame 500 → Pause
Resume → Train to frame 1000 → Pause
Resume → Train to frame 1500
trainingFrameCount: 0 → 500 → 500 → 1000 → 1000 → 1500 ✅
```

### Reset After Pause
```
Train to epoch 50, trainingFrameCount = 1500
Pause
Reset → initializeTraining()
trainingFrameCount = 0 ✅
```

### Training Completion
```
Train to epoch 200/200 (completion)
isTraining && epoch < totalEpochs = FALSE
trainingFrameCount stops incrementing ✅
Particles fade away naturally ✅
```

## Benefits

### Before Fix
- ❌ Particle bursts after resume
- ❌ Different particle patterns after pause
- ❌ Synapse flashing during pause
- ❌ Phase animation continues when paused
- ❌ Predictions recalculate when paused

### After Fix
- ✅ Smooth resume, no bursts
- ✅ Consistent particle patterns
- ✅ Static synapses during pause
- ✅ Frozen phase animation
- ✅ Cached predictions (zero computation)

## Performance Impact

**When Paused:**
- Before: 20 forward passes/frame + particle updates
- After: 0 forward passes, 0 particle updates
- **Result: Near-zero CPU usage** ✅

## File Locations

**Modified code in `neural_illumination.html`:**
- Line 803: `trainingFrameCount` variable
- Line 1077: Increment only when training
- Line 847: Reset in `initializeTraining()`
- Line 1524: Particle creation uses `trainingFrameCount`
- Line 1532: Particle hash uses `trainingFrameCount`
- Line 1447: Synapse hash (deterministic)
- Line 799-800: Frozen cache variables
- Line 1116-1153: Prediction caching logic

## Related Documentation

- [Training-Animation Synchronization](training-animation-synchronization.md)
- [Particle Density Control](particle-density-control.md)
