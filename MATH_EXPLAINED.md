# Understanding the Mathematics: Forward and Backward Passes

This document explains **exactly** what calculations happen during neural network training, with real examples from the visualization.

## 🔢 The Math is NOW Visible

Click the **"🔢 Show Math"** button in the visualization to see:
- Live calculations in the sidebar (cycles through 6 steps)
- Activation values displayed on neurons
- Gradient values shown during backward pass
- Formula overlays on the network diagram
- Real numbers from actual computation

## ⬆️ Forward Pass: Making Predictions

The forward pass computes predictions from inputs through the network.

### Step 1: Input Layer → Hidden Layer

For **each hidden neuron** (16 total), we compute:

```
z_j = b_j + Σ(x_i × W1[i][j])  for all input i
```

Where:
- `x_i` = input pixel value (0 or 1)
- `W1[i][j]` = weight connecting input i to hidden neuron j
- `b_j` = bias term for hidden neuron j

**Example from visualization:**
```
Hidden Neuron 0:
z = -0.123 + (1.0 × 0.234) + (0.0 × -0.456) + ... (64 inputs total)
z = 0.847
```

Then apply **ReLU activation**:
```
a_j = ReLU(z_j) = max(0, z_j)
```

**Example:**
```
a_0 = ReLU(0.847) = 0.847  (positive, so unchanged)
a_1 = ReLU(-0.234) = 0.000  (negative, so clipped to 0)
```

This creates **16 hidden activations**, one per neuron.

### Step 2: Hidden Layer → Output Layer

For **each output neuron** (2 total: Circle, Square):

```
y_k = b_k + Σ(h_j × W2[j][k])  for all hidden j
```

Where:
- `h_j` = hidden neuron activation from step 1
- `W2[j][k]` = weight connecting hidden j to output k
- `b_k` = bias for output k

**Example:**
```
Circle output (k=0):
y_0 = 0.045 + (0.847 × 0.523) + (0.000 × -0.234) + ... (16 hidden)
y_0 = 1.234

Square output (k=1):
y_1 = -0.023 + (0.847 × -0.312) + ...
y_1 = -0.456
```

### Step 3: Softmax (Convert to Probabilities)

Convert raw scores to probabilities that sum to 1:

```
P(class_k) = exp(y_k) / Σ(exp(y_i))  for all outputs i
```

**Example:**
```
exp(1.234) = 3.435
exp(-0.456) = 0.634
sum = 3.435 + 0.634 = 4.069

P(Circle) = 3.435 / 4.069 = 0.844 = 84.4%
P(Square) = 0.634 / 4.069 = 0.156 = 15.6%
```

**Prediction: Circle** (higher probability)

### Step 4: Loss (How Wrong Are We?)

Calculate **cross-entropy loss** comparing prediction to truth:

```
L = -Σ(y_true × log(y_pred))
```

If the true label is Circle [1, 0]:
```
L = -(1 × log(0.844) + 0 × log(0.156))
L = -log(0.844)
L = 0.170
```

**Lower loss = better prediction**

---

## ⬇️ Backward Pass: Learning from Mistakes

The backward pass computes gradients (how to adjust weights) using the **chain rule**.

### Step 1: Output Layer Gradient

Calculate how much the output needs to change:

```
∂L/∂y_k = ŷ_k - y_k
```

Where `ŷ` is prediction, `y` is true label.

**Example (true label is Circle [1, 0]):**
```
∂L/∂y_Circle = 0.844 - 1.000 = -0.156  (too low, needs to increase)
∂L/∂y_Square = 0.156 - 0.000 = +0.156  (too high, needs to decrease)
```

### Step 2: Hidden Layer Gradient (Backpropagate)

Use **chain rule** to flow gradients backward through weights:

```
∂L/∂h_j = Σ(∂L/∂y_k × W2[j][k])  for all outputs k
```

**Example for hidden neuron 0:**
```
∂L/∂h_0 = (-0.156 × 0.523) + (+0.156 × -0.312)
∂L/∂h_0 = -0.0816 - 0.0487
∂L/∂h_0 = -0.1303
```

Then multiply by **ReLU derivative**:
```
∂L/∂z_j = ∂L/∂h_j × ReLU'(z_j)

where ReLU'(z) = 1 if z > 0, else 0
```

**Example:**
```
If z_0 = 0.847 > 0:
  ∂L/∂z_0 = -0.1303 × 1 = -0.1303

If z_1 = -0.234 < 0:
  ∂L/∂z_1 = anything × 0 = 0.0000  (gradient killed by ReLU)
```

### Step 3: Weight Gradients

Calculate how much each weight contributed to the error:

**For W2 (hidden → output):**
```
∂L/∂W2[j][k] = ∂L/∂y_k × h_j
```

**Example:**
```
∂L/∂W2[0][0] = (-0.156) × 0.847 = -0.132
∂L/∂W2[0][1] = (+0.156) × 0.847 = +0.132
```

**For W1 (input → hidden):**
```
∂L/∂W1[i][j] = ∂L/∂z_j × x_i
```

**Example:**
```
∂L/∂W1[0][0] = (-0.1303) × 1.0 = -0.1303
∂L/∂W1[1][0] = (-0.1303) × 0.0 = 0.0000
```

### Step 4: Update Weights (Gradient Descent)

Move weights in the **opposite direction** of the gradient:

```
W_new = W_old - learning_rate × ∂L/∂W
```

**Example (learning_rate = 0.05):**
```
W2[0][0] before: 0.523
∂L/∂W2[0][0] = -0.132

W2[0][0] after: 0.523 - (0.05 × -0.132)
              = 0.523 + 0.0066
              = 0.5296  ← weight INCREASED (gradient was negative)

This makes the network more likely to predict Circle when it sees similar input!
```

**Example 2:**
```
W2[0][1] before: -0.312
∂L/∂W2[0][1] = +0.132

W2[0][1] after: -0.312 - (0.05 × 0.132)
              = -0.312 - 0.0066
              = -0.3186  ← weight DECREASED (gradient was positive)

This makes the network less likely to predict Square for this input!
```

### Step 5: Repeat for All Parameters

This process updates **all 1,074 parameters**:
- W1: 64 × 16 = 1,024 weights
- b1: 16 biases
- W2: 16 × 2 = 32 weights
- b2: 2 biases

**Each parameter moves slightly in the direction that reduces loss.**

---

## 🎯 Why This Works: The Intuition

### Forward Pass (Prediction)
1. Input pixels → weighted sum → activation
2. Creates abstract features in hidden layer
3. Combines features → output scores
4. Softmax → probabilities
5. Compare to truth → loss

### Backward Pass (Learning)
1. "How wrong was the output?" → output gradient
2. "Which hidden neurons caused that?" → hidden gradient
3. "Which weights caused those activations?" → weight gradient
4. Update weights to be less wrong next time
5. Repeat thousands of times → network learns

### The Magic of Gradients

The gradient `∂L/∂W` tells us:
- **Sign**: Should this weight increase (+) or decrease (−)?
- **Magnitude**: How much impact does this weight have?

By following gradients, the network **automatically discovers** which weights help minimize loss.

---

## 📊 What You See in the Visualization

### When Math Display is ON:

**In the Sidebar (Live Mathematics panel):**
- Cycles through 6 calculation steps
- Shows actual formulas being computed
- Displays real numbers from network state
- Updates every frame during training

**On the Network Diagram:**
- Key neurons show their activation values
- Formula boxes appear showing computation
- Gradient values displayed during backward pass
- Weight labels on important connections

**What the Colors Mean:**
- **Blue boxes**: Forward pass calculations (prediction)
- **Orange/Red boxes**: Backward pass calculations (learning)
- **White text**: Current values
- **Monospace font**: Actual computed numbers

### Numbers That Prove It's Real:

1. **Activations change** based on input (not random)
2. **Gradients decrease** as network converges
3. **Weights update** according to gradient descent formula
4. **Loss decreases** because math is working correctly
5. **Predictions improve** as evidence of learning

---

## 🧪 Experiments to Understand the Math

### Experiment 1: Watch a Single Weight Change
1. Turn on Math Display
2. Note W2[0][0] initial value (shown in "Training Proof" panel)
3. Watch it change each epoch
4. The change = learning_rate × gradient (exactly as formula predicts)

### Experiment 2: See ReLU in Action
1. Turn on Math Display
2. During forward pass, look at hidden neuron 0 calculation
3. Notice: if `z < 0`, then `activation = 0` (ReLU clipping)
4. During backward pass, notice: gradient = 0 for neurons with z < 0

### Experiment 3: Gradient Flow Visualization
1. Turn on Math Display
2. Pause during backward pass (red flow)
3. See gradients on neurons (∇ = value)
4. Larger gradients = neurons that need more adjustment

### Experiment 4: Softmax Probabilities
1. Watch output layer during forward pass
2. Notice probabilities always sum to 1.0
3. As training progresses, correct class probability → 1.0
4. This is softmax working correctly

---

## 💡 Common Questions

**Q: How does the network "know" which weights to change?**
A: Gradients! The chain rule mathematically computes exactly how each weight contributed to the error.

**Q: Why do some neurons have gradient = 0?**
A: ReLU kills gradients for negative activations. This is intentional—those neurons weren't active, so they don't need updates.

**Q: Why does loss sometimes increase slightly?**
A: Stochastic gradient descent uses mini-batches. Some batches are harder than others, causing temporary loss increases. This is normal!

**Q: How can I verify these calculations?**
A:
1. Turn on Math Display
2. Open browser console (F12)
3. Write down values shown
4. Calculate manually using formulas above
5. Compare to what visualization shows
6. They'll match exactly!

**Q: What's the difference between activation and gradient?**
A:
- **Activation**: Output of a neuron during forward pass (what it computed)
- **Gradient**: How much to change weights during backward pass (how to improve)

---

## 🎓 Deep Dive: The Chain Rule

The entire backward pass is based on **one mathematical principle**: the chain rule.

```
If y = f(g(x)), then dy/dx = (df/dg) × (dg/dx)
```

Applied to neural networks:
```
∂Loss/∂W1 = (∂Loss/∂output) × (∂output/∂hidden) × (∂hidden/∂W1)
```

The visualization shows each link in this chain:
1. Output gradient: ∂Loss/∂output
2. Hidden gradient: (∂Loss/∂output) × (∂output/∂hidden)
3. Weight gradient: hidden_gradient × (∂hidden/∂W1)

**This is why it's called "backpropagation"** — we propagate gradients backward through the chain of computations!

---

## 🔬 The Math is Real

Every number you see in the Math Display comes from actual computation:
- **Not hardcoded**
- **Not animated timers**
- **Not random fluctuations**

They are the **real mathematical values** computed during genuine backpropagation.

**To verify**: Open console, print diagnostics, calculate by hand, and compare. The math works. ✓
