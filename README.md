# Neural Illumination

A real-time visualization of neural network training that transforms abstract mathematics into flowing light, pulsing neurons, and emergent understanding.

## What You're Seeing

This is **not a mock animation**—it's a genuine neural network training in real-time, visualized as algorithmic art. Every glow, pulse, and connection represents actual computation.

### The Network
- **Task**: Binary classification (circles vs squares)
- **Architecture**: 64 inputs (8×8 grid) → 16 hidden neurons → 2 outputs
- **Training**: Real backpropagation with stochastic gradient descent
- **Dataset**: 240 training samples, 40 test samples (more data = better learning observation)
- **Training**: 200 epochs (longer observation of convergence)

### What Forward & Backward Mean

**⬆ Forward Pass** (White particles flowing up):
- Processing a batch of samples through the network
- Computing predictions based on current weights
- Represents: "What the network currently thinks"

**⬇ Backward Pass** (Red particles flowing down):
- Computing gradients via backpropagation
- Updating all 1,074 weights using gradient descent
- Represents: "How to improve the weights"

**Read `TRAINING_PROCESS.md`** for detailed explanation of what each phase represents!

### Visual Elements

**Neurons (Glowing Bulbs)**
- Brightness = activation strength (how strongly the neuron fires)
- **Forward Pass**: White/blue energy waves emanating outward (3 concentric waves)
- **Backward Pass**: Red/orange pulsing halos with rotating gradient indicators (6 spokes)
- **Phase Transition**: Sparkles when forward and backward meet (shows interaction)
- Position = layer in network (input left, hidden center, output right)

**Synapses (Connections)**
- Thickness = weight magnitude (strength of connection)
- Color = weight sign during forward (blue = positive, orange = negative)
- Gradient overlay showing flow direction (bright → dim)
- Pulsing effect synchronized with computation phase
- Opacity = adjustable visibility

**Flow Particles (Animated Information)**
- **Forward**: Bright white/blue particles with arrows flowing upward
  - Represent: Data and activations moving through network
  - Size: Proportional to weight importance
  - Speed: Adjustable (0.5-3x)

- **Backward**: Red/orange particles with arrows flowing downward
  - Represent: Gradients (error signals) propagating back
  - Appear on: Important connections (|weight| > 0.2)
  - Glow: Changes from orange to red as they travel

**Training Phases**
- **Forward Pass**: White particles flowing upward → The network makes predictions
- **Backward Pass**: Red particles flowing downward → Gradients flow back to update weights
- **Interaction**: Sparkles and color blending when phases transition

**Prediction Grid** (top-left)
- Shows 20 test samples with current predictions
- Blue border = correct prediction
- Red border = incorrect prediction
- Watch it shift from "mostly red" → "mostly blue" as learning progresses

**Metrics Panel** (right)
- Loss curve: descends as network learns (red)
- Accuracy curve: rises as predictions improve (blue)
- Layer gradients: shows where learning is happening
- Output probabilities: current network confidence

## How to Use

### Running the Visualization

1. Open `neural_illumination.html` in any modern browser
2. Watch the network train automatically
3. Explore different random initializations using the seed controls

### Interactive Controls

**Training Parameters**
- **Learning Rate**: How aggressively the network updates (0.001-0.5)
- **Training Speed**: Epochs per second (1-10x)
- **Batch Size**: Samples per update (4-32)

**Visualization Parameters**
- **Neuron Glow Intensity**: Brightness of activations (0.5-3x)
- **Flow Speed**: Animation speed of forward/backward flows (0.5-3x)
- **Synapse Visibility**: Opacity of connection lines (0.1-1.0)

**Seed Controls**
- Each seed produces a different random initialization
- Previous/Next: Browse sequential seeds
- Random: Jump to random initialization
- Enter seed directly for reproducible results

**Actions**
- **Pause/Resume**: Stop and restart training
- **Reset**: Restart training from scratch with current parameters
- **Print Full Diagnostics**: Show comprehensive training details in console
- **Show/Hide Math**: Toggle live mathematical calculations display

### Live Mathematics View

Click "🔢 Show Math" to see the **actual calculations** happening in real-time:

**Forward Pass (White particles flowing up)**:
1. **Input Layer**: Shows sample data being fed in
2. **Hidden Neuron Computation**: `z = Σ(input × weight) + bias`, then `activation = ReLU(z)`
3. **Hidden Activations**: Display values for all 16 neurons
4. **Output Computation**: `y = Σ(hidden × W2) + bias`
5. **Softmax Probabilities**: Normalized predictions for Circle vs Square
6. **Loss Calculation**: Cross-entropy loss with actual logarithms

**Backward Pass (Red particles flowing down)**:
1. **Output Gradient**: `∂L/∂output = ŷ - y` (prediction error)
2. **Hidden Gradient**: Backpropagation through weights using chain rule
3. **Weight Gradient**: `∂L/∂W = gradient × activation`
4. **Update Rule**: `W_new = W_old - learning_rate × gradient`
5. **Gradient Norms**: Average gradient magnitudes (shows convergence)
6. **Complete**: All 1,074 parameters updated

The math display cycles through these steps automatically, showing **real numbers** from actual computation—not animations!

## The Algorithm

### Data Generation
- Procedurally generates circles and squares at random positions
- Adds noise to make classification non-trivial
- 8×8 binary images (64 input features)

### Network Architecture
```
Input Layer (64 neurons)
    ↓ (64 × 16 weight matrix)
Hidden Layer (16 neurons, ReLU activation)
    ↓ (16 × 2 weight matrix)
Output Layer (2 neurons, Softmax)
```

### Training Loop
1. Forward propagation: Input → Hidden (ReLU) → Output (Softmax)
2. Loss calculation: Cross-entropy loss
3. Backward propagation: Compute gradients via chain rule
4. Weight update: Gradient descent (W -= learning_rate × ∇W)

### Visualization Mapping
- Activations → Neuron luminosity (logarithmic scaling)
- Gradients → Red halo intensity (proportional to |∇|)
- Weights → Synapse thickness and color
- Training phase → Flow particle direction and color

## Technical Details

**Implementation**: Pure JavaScript with p5.js for rendering
- No ML libraries—neural network implemented from scratch
- Real backpropagation, not simulated
- ~60 FPS real-time visualization

**Optimization Decisions**
- Synapse sampling: Only renders subset to prevent visual clutter
- Activation scaling: Logarithmic mapping preserves both subtle and strong signals
- Gradient normalization: Per-layer normalization for visual consistency
- Flow animation: Sinusoidal easing for organic movement

**Craftsmanship Notes**
- Weight initialization: Xavier/He initialization for stable training
- Loss visualization: Clamped to [0, 2] for better visual range
- Color palette: Anthropic brand colors (orange, blue, green)
- Layout: Three-column design (predictions, network, metrics)

## Exploring the Space

### Interesting Seeds to Try
- **42**: Classic random seed, balanced learning curve
- **137**: Fast convergence with dramatic accuracy jump
- **999**: Slower learning, interesting gradient patterns
- **2024**: Unusual weight patterns in final configuration

### Parameter Experiments
- **High Learning Rate (0.3+)**: Chaotic updates, oscillating loss
- **Low Learning Rate (0.01)**: Smooth but slow convergence
- **Large Batch Size (32)**: Stable gradients, slower exploration
- **Small Batch Size (4)**: Noisy gradients, faster adaptation

### What to Watch For
- **Early epochs**: Neurons fire randomly, synapses reorganize rapidly
- **Mid training**: Specialization emerges (some neurons detect edges, others shapes)
- **Late training**: Fine-tuning, subtle weight adjustments
- **Convergence**: Most synapses stabilize, only specific connections still updating

## Philosophy

This visualization embodies the algorithmic philosophy of **Neural Illumination**—the idea that intelligence is not a static structure but an emergent phenomenon arising from simple mathematical operations repeated countless times.

Each neuron is initially uncertain, firing randomly. Through gradient descent, these neurons gradually learn to respond to specific patterns. The visualization makes this emergence visible: watch as chaos crystallizes into structure, as random connections prune into meaningful pathways, as a collection of equations becomes a system that understands the difference between circles and squares.

This is not educational software pretending to be art, nor art pretending to understand neural networks. It is the mathematical truth of learning, expressed through light.

## Understanding the Visual Flow

The visualization uses **computational aesthetics** to make backpropagation visible and beautiful. For a deep dive into the artistic meaning:

**Read: `VISUAL_FLOW_GUIDE.md`** - Comprehensive guide explaining:
- Color language (what blue vs red means)
- Particle system (how flow particles work)
- Phase interaction (sparkles and transitions)
- Artistic interpretations (light, echoes, breathing)
- How to read the visual language

**Quick Visual Guide:**

| What You See | What It Means |
|--------------|---------------|
| White/blue particles flowing up | Forward pass: Data → Predictions |
| Red/orange particles flowing down | Backward pass: Errors → Gradients |
| Sparkles around neurons | Forward & backward interacting (chain rule) |
| Pulsing connections | Active computation on that weight |
| Rotating spokes on neurons | Large gradient (active learning) |
| Energy waves from neurons | Strong activation (important computation) |
| Thick glowing lines | Important weights (strong connections) |
| Thin dim lines | Weak weights (less important) |

## Files

- `neural_illumination.html` - Complete interactive visualization
- `neural_illumination_philosophy.md` - Algorithmic philosophy document
- `TRAINING_PROCESS.md` - **NEW!** What forward & backward passes really mean
- `VISUAL_FLOW_GUIDE.md` - Artistic interpretation of forward/backward flow
- `MATH_EXPLAINED.md` - Mathematical explanations with examples
- `PROOF_OF_TRAINING.md` - Evidence this is real training
- `SUMMARY.md` - Complete overview of the project
- `README.md` - This file

### Recommended Reading Order

1. **Start here**: `README.md` - Overview and quick start
2. **Understand the process**: `TRAINING_PROCESS.md` - What's really happening
3. **Visual understanding**: `VISUAL_FLOW_GUIDE.md` - Artistic interpretation
4. **Mathematical depth**: `MATH_EXPLAINED.md` - Step-by-step calculations
5. **Verification**: `PROOF_OF_TRAINING.md` - Evidence it's real

## Credits

Created using the algorithmic-art skill framework. All computation is real—no smoke, no mirrors, just mathematics made beautiful.
