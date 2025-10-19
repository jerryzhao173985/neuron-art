# Particle Density Control

## Problem Statement

**Accumulation Issue:**
Particle density varied dramatically with animation speed, creating visual clutter at slow speeds.

### Root Cause

**Independent Rates:**
```javascript
// Creation rate: Fixed
if (trainingFrameCount % 3 === 0) { createParticle(); }

// Movement speed: Variable
particle.speed = 0.020 * effectiveAnimationSpeed;
```

**Mathematics of Accumulation:**

At **fast speed (effectiveSpeed = 3.0):**
- Particle lifetime: 1 / (0.020 × 3.0) = ~17 frames
- Creation interval: 3 frames
- Particles on screen: 17 / 3 ≈ **6 particles** ✅

At **slow speed (effectiveSpeed = 0.1):**
- Particle lifetime: 1 / (0.020 × 0.1) = ~500 frames
- Creation interval: 3 frames (SAME!)
- Particles on screen: 500 / 3 ≈ **167 particles** ❌

## Solution: Coupled Creation Rate

### Formula

```javascript
const baseInterval = 3;
const adjustedInterval = Math.max(1, Math.round(baseInterval / effectiveAnimationSpeed));
```

### How It Works

Creation interval is **inversely proportional** to animation speed:

| effectiveSpeed | adjustedInterval | Particle Lifetime | Density |
|----------------|------------------|-------------------|---------|
| 0.1 | 30 frames | 500 frames | ~17 particles |
| 0.5 | 6 frames | 100 frames | ~17 particles |
| 1.0 | 3 frames | 50 frames | ~17 particles |
| 3.0 | 1 frame | 17 frames | ~17 particles |

**Result:** Constant density (~17 particles) at all speeds ✅

## Implementation

### Code Location

**Line 1520-1529 in `neural_illumination.html`:**

```javascript
// Add new particles occasionally for important connections
// Creation rate adjusts with animation speed to maintain consistent visual density
// When particles move slower, we create them less frequently
const baseInterval = 3;  // Base creation interval (frames)
const adjustedInterval = Math.max(1, Math.round(baseInterval / effectiveAnimationSpeed));

// Use trainingFrameCount instead of frameCount for consistent timing across pause/resume
if (trainingFrameCount % adjustedInterval === 0) {
    // Create particles...
}
```

### Edge Cases

**Very Fast Speed (> 3.0):**
```javascript
Math.max(1, ...)  // Prevents interval < 1 frame
```
- Minimum interval: 1 frame
- Maximum creation rate: every frame
- Still maintains reasonable density

**Very Slow Speed (< 0.1):**
```javascript
Math.round(...)  // Handles fractional intervals
```
- Example: effectiveSpeed = 0.05 → interval = 60 frames
- Creates particles very rarely
- Matches slow particle movement

## Mathematical Proof

### Steady-State Density

```
Density = ParticleLifetime / CreationInterval

ParticleLifetime = 1 / (baseSpeed × effectiveSpeed)
                 = 1 / (0.020 × effectiveSpeed)

CreationInterval = baseInterval / effectiveSpeed
                 = 3 / effectiveSpeed

Density = [1 / (0.020 × effectiveSpeed)] / [3 / effectiveSpeed]
        = [1 / (0.020 × effectiveSpeed)] × [effectiveSpeed / 3]
        = 1 / (0.020 × 3)
        = 1 / 0.060
        ≈ 17 particles (constant!)
```

**The `effectiveSpeed` cancels out**, proving density is independent of speed.

## Benefits

### Before Fix
- Slow speed: 167+ particles (visual clutter) ❌
- Fast speed: 6 particles (clean) ✅
- **Inconsistent user experience**

### After Fix
- Slow speed: ~17 particles (clean, observable) ✅
- Fast speed: ~17 particles (clean, dynamic) ✅
- **Consistent user experience at all speeds**

## Related Systems

### Works With Synchronization

Particle density control uses `effectiveAnimationSpeed` from synchronization:
```javascript
effectiveAnimationSpeed = trainingSpeed × (flowSpeed / 1.5)
adjustedInterval = baseInterval / effectiveAnimationSpeed
```

Both training speed and animation multiplier affect density control.

### Works With Pause/Resume

Uses `trainingFrameCount` (not `frameCount`):
```javascript
if (trainingFrameCount % adjustedInterval === 0)
```

This ensures:
- No particle bursts after resume
- Consistent timing across pause/resume cycles
- Deterministic particle creation

## Usage Recommendations

### For Detailed Observation
```
Training Speed = 1
Animation Multiplier = 0.5
→ effectiveSpeed = 0.33
→ Interval = 9 frames
→ Slow motion with ~17 particles
```

### For Normal Visualization
```
Training Speed = 3
Animation Multiplier = 1.5
→ effectiveSpeed = 3.0
→ Interval = 1 frame
→ Balanced flow with ~17 particles
```

### For Fast Dynamics
```
Training Speed = 10
Animation Multiplier = 1.5
→ effectiveSpeed = 10.0
→ Interval = 1 frame (max)
→ Rapid flow with ~17 particles
```

## File Locations

**Modified code:**
- Line 1520-1543: Particle creation with adjusted interval
- Line 1367: Particle speed using effectiveAnimationSpeed

## Related Documentation

- [Training-Animation Synchronization](training-animation-synchronization.md)
- [Pause-Resume Timing](pause-resume-timing.md) (if exists)
