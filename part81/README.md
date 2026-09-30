# Part 81: Deep Learning - PyTorch

## สารบัญ
1. [PyTorch vs TensorFlow](#pytorch-vs-tensorflow)
2. [Tensors](#tensors)
3. [Autograd - Automatic Differentiation](#autograd)
4. [Neural Network with nn.Module](#neural-network)
5. [Training Loop](#training-loop)
6. [DataLoader และ Dataset](#dataloader-dataset)
7. [Transfer Learning with torchvision](#transfer-learning)
8. [GPU Training](#gpu-training)
9. [Model Saving/Loading](#model-saving-loading)
10. [TorchScript](#torchscript)
11. [PyTorch Lightning](#pytorch-lightning)
12. [ตัวอย่างโปรแกรมจริง](#real-examples)
13. [แบบฝึกหัด](#exercises)

---

## 1. PyTorch vs TensorFlow {#pytorch-vs-tensorflow}

### ความเป็นมา

**PyTorch** พัฒนาโดย Facebook AI Research (FAIR) เปิดตัวในปี 2016 เน้น dynamic computation graph ที่ทำให้การ debug และ development ง่ายขึ้น

**TensorFlow** พัฒนาโดย Google Brain เปิดตัวในปี 2015 ใช้ static computation graph ใน version แรก แต่ TensorFlow 2.0 ปรับเป็น eager execution

### การเปรียบเทียบหลัก

| Feature | PyTorch | TensorFlow |
|---------|---------|------------|
| Computation Graph | Dynamic (Eager) | Static (+ Eager) |
| Debugging | ง่าย (Python native) | ยากกว่า |
| API Design | Pythonic | Keras (high-level) |
| Research Use | นิยมมาก | ใช้กันมาก |
| Production | TorchServe | TF Serving |
| Mobile | PyTorch Mobile | TensorFlow Lite |
| Community | ใหญ่มาก | ใหญ่มาก |

### เมื่อไหร่ควรใช้ PyTorch

- งานวิจัยและ prototype ใหม่ๆ
- ต้องการ flexibility สูง
- Custom layer/loss functions ซับซ้อน
- Dynamic architecture (RNN ที่มี variable length)

### เมื่อไหร่ควรใช้ TensorFlow

- Production deployment บน mobile/edge
- มี infrastructure TF อยู่แล้ว
- ต้องการ TensorBoard integration
- Keras API ที่เรียนง่าย

```python
# การติดตั้ง PyTorch
# pip install torch torchvision torchaudio

# ตรวจสอบ version
import torch
print(f"PyTorch version: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"CUDA version: {torch.version.cuda}")

# เปรียบเทียบ syntax เบื้องต้น
# PyTorch
import torch
import torch.nn as nn

# TensorFlow
# import tensorflow as tf
# from tensorflow import keras
```

---

## 2. Tensors {#tensors}

### Tensor คืออะไร

Tensor คือ generalization ของ matrix หลายมิติ เป็น building block หลักของ PyTorch

- 0D Tensor = scalar (ตัวเลขเดี่ยว)
- 1D Tensor = vector
- 2D Tensor = matrix
- 3D+ Tensor = higher-dimensional array

### การสร้าง Tensor

```python
import torch
import numpy as np

# ตัวอย่างที่ 1: การสร้าง Tensor แบบต่างๆ
# สร้างจาก list
t1 = torch.tensor([1, 2, 3, 4, 5])
print("1D tensor:", t1)
print("Shape:", t1.shape)
print("Dtype:", t1.dtype)

# สร้าง 2D tensor
t2 = torch.tensor([[1.0, 2.0, 3.0],
                    [4.0, 5.0, 6.0]])
print("\n2D tensor:\n", t2)
print("Shape:", t2.shape)

# สร้างจาก NumPy
np_array = np.array([[1, 2], [3, 4]])
t3 = torch.from_numpy(np_array)
print("\nFrom NumPy:", t3)

# แปลงกลับเป็น NumPy
np_back = t3.numpy()
print("Back to NumPy:", np_back)
```

```python
# ตัวอย่างที่ 2: Tensor initialization functions
# Zeros และ Ones
zeros = torch.zeros(3, 4)
ones = torch.ones(3, 4)
print("Zeros:\n", zeros)
print("Ones:\n", ones)

# Random tensors
rand_uniform = torch.rand(3, 4)      # Uniform [0, 1)
rand_normal = torch.randn(3, 4)      # Normal distribution
rand_int = torch.randint(0, 10, (3, 4))  # Random integers

print("Random uniform:\n", rand_uniform)
print("Random normal:\n", rand_normal)
print("Random integers:\n", rand_int)

# Identity matrix
eye = torch.eye(4)
print("Identity:\n", eye)

# Arange และ linspace
arange = torch.arange(0, 10, 2)     # start, stop, step
linspace = torch.linspace(0, 1, 5)  # start, stop, num_points
print("Arange:", arange)
print("Linspace:", linspace)
```

```python
# ตัวอย่างที่ 3: Tensor data types
# Float types
f32 = torch.tensor([1.0], dtype=torch.float32)
f64 = torch.tensor([1.0], dtype=torch.float64)
f16 = torch.tensor([1.0], dtype=torch.float16)

# Integer types
i32 = torch.tensor([1], dtype=torch.int32)
i64 = torch.tensor([1], dtype=torch.int64)

# Boolean
b = torch.tensor([True, False, True])

print(f"float32: {f32.dtype}")
print(f"float64: {f64.dtype}")
print(f"int64: {i64.dtype}")
print(f"bool: {b.dtype}")

# Type conversion
x = torch.tensor([1, 2, 3])
x_float = x.float()         # to float32
x_double = x.double()       # to float64
x_half = x.half()           # to float16
print("Int to float:", x_float)
```

### Tensor Operations

```python
# ตัวอย่างที่ 4: Basic arithmetic operations
a = torch.tensor([1.0, 2.0, 3.0])
b = torch.tensor([4.0, 5.0, 6.0])

# Element-wise operations
print("a + b =", a + b)          # or torch.add(a, b)
print("a - b =", a - b)          # or torch.sub(a, b)
print("a * b =", a * b)          # or torch.mul(a, b)
print("a / b =", a / b)          # or torch.div(a, b)
print("a ** 2 =", a ** 2)        # Power
print("sqrt(a) =", torch.sqrt(a))

# In-place operations (modifies tensor directly)
c = torch.ones(3)
c.add_(2)    # c = c + 2
print("In-place add:", c)
```

```python
# ตัวอย่างที่ 5: Matrix operations
A = torch.tensor([[1.0, 2.0], [3.0, 4.0]])
B = torch.tensor([[5.0, 6.0], [7.0, 8.0]])

# Matrix multiplication
C = torch.mm(A, B)          # 2D matrix multiply
D = A @ B                   # @ operator (same as mm for 2D)
E = torch.matmul(A, B)      # more general (handles batches)

print("A @ B =\n", C)

# Transpose
print("A.T =\n", A.T)
print("A.t() =\n", A.t())

# Determinant และ inverse
det = torch.det(A)
inv = torch.inverse(A)
print(f"det(A) = {det}")
print("inv(A) =\n", inv)

# Batch matrix multiply
batch_A = torch.randn(10, 3, 4)  # batch of 10 matrices
batch_B = torch.randn(10, 4, 5)
result = torch.bmm(batch_A, batch_B)  # (10, 3, 5)
print(f"Batch matmul shape: {result.shape}")
```

```python
# ตัวอย่างที่ 6: Reshaping operations
x = torch.arange(24)
print("Original:", x.shape)

# Reshape
a = x.reshape(4, 6)
b = x.reshape(2, 3, 4)
c = x.view(6, 4)         # view shares memory with original

print("Reshaped (4,6):", a.shape)
print("Reshaped (2,3,4):", b.shape)

# Squeeze และ Unsqueeze
d = torch.randn(3, 1, 4, 1)
squeezed = d.squeeze()      # remove all dim=1
print("After squeeze:", squeezed.shape)

e = torch.randn(3, 4)
unsqueezed = e.unsqueeze(0)  # add dim at position 0
print("After unsqueeze(0):", unsqueezed.shape)

# Permute (transpose multiple dims)
f = torch.randn(2, 3, 4)
permuted = f.permute(2, 0, 1)  # (4, 2, 3)
print("After permute:", permuted.shape)

# Flatten
g = torch.randn(2, 3, 4)
flat = g.flatten()
flat_from1 = g.flatten(start_dim=1)
print("Flatten all:", flat.shape)
print("Flatten from dim 1:", flat_from1.shape)
```

```python
# ตัวอย่างที่ 7: Indexing and Slicing
x = torch.arange(16).reshape(4, 4)
print("Original:\n", x)

# Basic indexing
print("Row 0:", x[0])
print("Element [1,2]:", x[1, 2])

# Slicing
print("Rows 1-2:\n", x[1:3])
print("Cols 1-3:\n", x[:, 1:3])
print("Submatrix:\n", x[1:3, 1:3])

# Boolean indexing
mask = x > 8
print("Mask:\n", mask)
print("Elements > 8:", x[mask])

# Fancy indexing
rows = torch.tensor([0, 2])
cols = torch.tensor([1, 3])
print("Fancy index:", x[rows, cols])

# Where (conditional selection)
result = torch.where(x > 8, x, torch.zeros_like(x))
print("Where x>8:\n", result)
```

```python
# ตัวอย่างที่ 8: Reduction operations
x = torch.tensor([[1.0, 2.0, 3.0],
                   [4.0, 5.0, 6.0]])

print("Sum all:", x.sum())
print("Sum dim=0:", x.sum(dim=0))    # column-wise
print("Sum dim=1:", x.sum(dim=1))    # row-wise

print("Mean:", x.mean())
print("Max:", x.max())
print("Min:", x.min())

# Max with index
max_val, max_idx = x.max(dim=1)
print(f"Max values: {max_val}, indices: {max_idx}")

# Cumulative sum
print("Cumsum:", x.cumsum(dim=1))

# Norm
print("L2 norm:", torch.norm(x))
print("L1 norm:", torch.norm(x, p=1))
```

---

## 3. Autograd - Automatic Differentiation {#autograd}

### หลักการ Autograd

Autograd เป็นระบบ automatic differentiation ของ PyTorch ที่คำนวณ gradient โดยอัตโนมัติ ทำงานโดยสร้าง computation graph ขณะ forward pass แล้วใช้ backpropagation ในขั้น backward pass

```python
# ตัวอย่างที่ 9: Basic autograd
import torch

# สร้าง tensor ที่ต้องการ gradient
x = torch.tensor(3.0, requires_grad=True)
print(f"x = {x}")
print(f"requires_grad: {x.requires_grad}")

# Forward pass: y = x^2 + 2x + 1
y = x**2 + 2*x + 1
print(f"y = x^2 + 2x + 1 = {y}")

# Backward pass: compute dy/dx
y.backward()

# dy/dx = 2x + 2 = 2(3) + 2 = 8
print(f"dy/dx = {x.grad}")  # Should be 8.0
```

```python
# ตัวอย่างที่ 10: Gradient computation for vectors
x = torch.tensor([1.0, 2.0, 3.0], requires_grad=True)

# y = sum(x^2)
y = (x**2).sum()
y.backward()

# dy/dx_i = 2*x_i
print(f"x = {x}")
print(f"dy/dx = {x.grad}")  # [2, 4, 6]

# Gradient accumulation (must zero grad before next backward)
x.grad.zero_()  # Reset gradients

z = (x**3).sum()
z.backward()
print(f"dz/dx = {x.grad}")  # [3, 12, 27]
```

```python
# ตัวอย่างที่ 11: Chain rule with complex functions
import torch

x = torch.tensor(2.0, requires_grad=True)

# f(x) = (3x^2 + 1)^2
# f'(x) = 2(3x^2 + 1) * 6x = 12x(3x^2 + 1)
# f'(2) = 12*2*(12+1) = 24*13 = 312
inner = 3 * x**2 + 1
outer = inner**2
outer.backward()
print(f"df/dx at x=2: {x.grad}")  # 312

# Disable gradient tracking
with torch.no_grad():
    y = x**2 + 1
    print(f"y (no grad): {y}, requires_grad: {y.requires_grad}")

# Detach from computation graph
z = x.detach()
print(f"z.requires_grad: {z.requires_grad}")
```

```python
# ตัวอย่างที่ 12: Gradient of matrix operations
W = torch.randn(3, 4, requires_grad=True)
x = torch.randn(4, 2)

# Forward
y = W @ x           # (3, 2)
loss = y.sum()

# Backward
loss.backward()
print(f"W.grad shape: {W.grad.shape}")  # (3, 4)
print(f"dL/dW:\n{W.grad}")

# Jacobian computation
from torch.autograd.functional import jacobian

def f(x):
    return x**2 + x

x = torch.tensor([1.0, 2.0, 3.0])
J = jacobian(f, x)
print(f"Jacobian:\n{J}")
```

---

## 4. Neural Network with nn.Module {#neural-network}

### nn.Module

`nn.Module` เป็น base class สำหรับสร้าง neural network layer และ model ทั้งหมดใน PyTorch

```python
# ตัวอย่างที่ 13: Simple neural network
import torch
import torch.nn as nn
import torch.nn.functional as F

class SimpleNet(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super(SimpleNet, self).__init__()
        # Define layers
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.fc2 = nn.Linear(hidden_size, hidden_size)
        self.fc3 = nn.Linear(hidden_size, output_size)
        self.dropout = nn.Dropout(p=0.5)
        self.batch_norm = nn.BatchNorm1d(hidden_size)
        
    def forward(self, x):
        # Define forward pass
        x = F.relu(self.fc1(x))
        x = self.batch_norm(x)
        x = self.dropout(x)
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

# สร้าง model
model = SimpleNet(input_size=784, hidden_size=256, output_size=10)
print(model)

# ดู parameters
total_params = sum(p.numel() for p in model.parameters())
trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"Total parameters: {total_params:,}")
print(f"Trainable parameters: {trainable_params:,}")

# Forward pass test
x = torch.randn(32, 784)  # batch of 32
output = model(x)
print(f"Output shape: {output.shape}")  # (32, 10)
```

```python
# ตัวอย่างที่ 14: Convolutional Neural Network
class CNN(nn.Module):
    def __init__(self, num_classes=10):
        super(CNN, self).__init__()
        
        # Convolutional layers
        self.conv1 = nn.Conv2d(1, 32, kernel_size=3, padding=1)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.conv3 = nn.Conv2d(64, 128, kernel_size=3, padding=1)
        
        # Pooling
        self.pool = nn.MaxPool2d(2, 2)
        
        # Batch normalization
        self.bn1 = nn.BatchNorm2d(32)
        self.bn2 = nn.BatchNorm2d(64)
        self.bn3 = nn.BatchNorm2d(128)
        
        # Dropout
        self.dropout = nn.Dropout(0.25)
        
        # Fully connected layers
        self.fc1 = nn.Linear(128 * 3 * 3, 512)
        self.fc2 = nn.Linear(512, num_classes)
        
    def forward(self, x):
        # Block 1: 28x28 -> 14x14
        x = self.pool(F.relu(self.bn1(self.conv1(x))))
        # Block 2: 14x14 -> 7x7
        x = self.pool(F.relu(self.bn2(self.conv2(x))))
        # Block 3: 7x7 -> 3x3
        x = self.pool(F.relu(self.bn3(self.conv3(x))))
        
        # Flatten
        x = x.view(x.size(0), -1)
        
        # Fully connected
        x = F.relu(self.fc1(x))
        x = self.dropout(x)
        x = self.fc2(x)
        
        return x

cnn = CNN(num_classes=10)
x = torch.randn(8, 1, 28, 28)  # MNIST-like input
output = cnn(x)
print(f"CNN output shape: {output.shape}")
```

```python
# ตัวอย่างที่ 15: Custom activation functions and layers
class SwiGLU(nn.Module):
    """SwiGLU activation - used in modern LLMs"""
    def forward(self, x):
        x, gate = x.chunk(2, dim=-1)
        return F.silu(gate) * x

class ResidualBlock(nn.Module):
    """Residual block with skip connection"""
    def __init__(self, dim):
        super().__init__()
        self.block = nn.Sequential(
            nn.Linear(dim, dim * 4),
            nn.GELU(),
            nn.Linear(dim * 4, dim)
        )
        self.norm = nn.LayerNorm(dim)
        
    def forward(self, x):
        return self.norm(x + self.block(x))  # Skip connection

class AttentionLayer(nn.Module):
    """Simple self-attention"""
    def __init__(self, dim, num_heads=8):
        super().__init__()
        self.attention = nn.MultiheadAttention(dim, num_heads, batch_first=True)
        self.norm = nn.LayerNorm(dim)
        
    def forward(self, x):
        attended, _ = self.attention(x, x, x)
        return self.norm(x + attended)

# Test
residual = ResidualBlock(256)
x = torch.randn(4, 256)
print("ResidualBlock output:", residual(x).shape)

attention = AttentionLayer(64, num_heads=4)
x = torch.randn(4, 10, 64)  # (batch, seq, dim)
print("Attention output:", attention(x).shape)
```

### Common Loss Functions

```python
# ตัวอย่างที่ 16: Loss functions
# Classification losses
ce_loss = nn.CrossEntropyLoss()
bce_loss = nn.BCEWithLogitsLoss()
nll_loss = nn.NLLLoss()

# Regression losses
mse_loss = nn.MSELoss()
mae_loss = nn.L1Loss()
huber_loss = nn.HuberLoss(delta=1.0)

# Example usage
# Multi-class classification
logits = torch.randn(8, 10)
labels = torch.randint(0, 10, (8,))
loss_ce = ce_loss(logits, labels)
print(f"CrossEntropy loss: {loss_ce.item():.4f}")

# Regression
predictions = torch.randn(8)
targets = torch.randn(8)
loss_mse = mse_loss(predictions, targets)
loss_mae = mae_loss(predictions, targets)
print(f"MSE loss: {loss_mse.item():.4f}")
print(f"MAE loss: {loss_mae.item():.4f}")

# Binary classification
binary_logits = torch.randn(8, 1)
binary_labels = torch.randint(0, 2, (8, 1)).float()
loss_bce = bce_loss(binary_logits, binary_labels)
print(f"BCE loss: {loss_bce.item():.4f}")
```

---

## 5. Training Loop {#training-loop}

### Complete Training Pipeline

```python
# ตัวอย่างที่ 17: Complete training loop
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# Create synthetic data
X_train = torch.randn(1000, 20)
y_train = (X_train[:, :5].sum(dim=1) > 0).long()
X_val = torch.randn(200, 20)
y_val = (X_val[:, :5].sum(dim=1) > 0).long()

# Create DataLoader
train_dataset = TensorDataset(X_train, y_train)
val_dataset = TensorDataset(X_val, y_val)
train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)
val_loader = DataLoader(val_dataset, batch_size=32)

# Model
class BinaryClassifier(nn.Module):
    def __init__(self, input_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, 64),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(64, 32),
            nn.ReLU(),
            nn.Linear(32, 2)
        )
    
    def forward(self, x):
        return self.net(x)

model = BinaryClassifier(20)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=5, gamma=0.5)

# Training function
def train_epoch(model, loader, optimizer, criterion):
    model.train()
    total_loss, correct, total = 0, 0, 0
    
    for X_batch, y_batch in loader:
        # Forward
        output = model(X_batch)
        loss = criterion(output, y_batch)
        
        # Backward
        optimizer.zero_grad()
        loss.backward()
        
        # Gradient clipping
        torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        
        # Update weights
        optimizer.step()
        
        # Metrics
        total_loss += loss.item()
        pred = output.argmax(dim=1)
        correct += (pred == y_batch).sum().item()
        total += len(y_batch)
    
    return total_loss / len(loader), correct / total

# Validation function
def validate(model, loader, criterion):
    model.eval()
    total_loss, correct, total = 0, 0, 0
    
    with torch.no_grad():
        for X_batch, y_batch in loader:
            output = model(X_batch)
            loss = criterion(output, y_batch)
            
            total_loss += loss.item()
            pred = output.argmax(dim=1)
            correct += (pred == y_batch).sum().item()
            total += len(y_batch)
    
    return total_loss / len(loader), correct / total

# Training loop
best_val_loss = float('inf')
history = {'train_loss': [], 'val_loss': [], 'train_acc': [], 'val_acc': []}

for epoch in range(20):
    train_loss, train_acc = train_epoch(model, train_loader, optimizer, criterion)
    val_loss, val_acc = validate(model, val_loader, criterion)
    scheduler.step()
    
    history['train_loss'].append(train_loss)
    history['val_loss'].append(val_loss)
    history['train_acc'].append(train_acc)
    history['val_acc'].append(val_acc)
    
    # Save best model
    if val_loss < best_val_loss:
        best_val_loss = val_loss
        torch.save(model.state_dict(), 'best_model.pth')
    
    if (epoch + 1) % 5 == 0:
        print(f"Epoch {epoch+1:3d}: "
              f"Train Loss={train_loss:.4f} Acc={train_acc:.4f} | "
              f"Val Loss={val_loss:.4f} Acc={val_acc:.4f}")
```

### Optimizers

```python
# ตัวอย่างที่ 18: Different optimizers
model_sgd = SimpleNet(784, 256, 10)
model_adam = SimpleNet(784, 256, 10)
model_adamw = SimpleNet(784, 256, 10)

# SGD with momentum
optimizer_sgd = optim.SGD(
    model_sgd.parameters(),
    lr=0.01,
    momentum=0.9,
    weight_decay=1e-4,
    nesterov=True
)

# Adam
optimizer_adam = optim.Adam(
    model_adam.parameters(),
    lr=0.001,
    betas=(0.9, 0.999),
    eps=1e-8,
    weight_decay=1e-5
)

# AdamW (Adam with decoupled weight decay) - best for transformers
optimizer_adamw = optim.AdamW(
    model_adamw.parameters(),
    lr=0.001,
    weight_decay=0.01
)

# RMSprop
optimizer_rmsprop = optim.RMSprop(
    model_sgd.parameters(),
    lr=0.001,
    alpha=0.99
)

print("Optimizers created successfully")
```

```python
# ตัวอย่างที่ 19: Learning rate schedulers
model = SimpleNet(784, 256, 10)
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Step scheduler: lr = lr * gamma every step_size epochs
step_sched = optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.5)

# Cosine annealing
cosine_sched = optim.lr_scheduler.CosineAnnealingLR(
    optimizer, T_max=50, eta_min=1e-6
)

# One cycle (best for fast training)
one_cycle = optim.lr_scheduler.OneCycleLR(
    optimizer,
    max_lr=0.01,
    steps_per_epoch=100,
    epochs=20
)

# ReduceLROnPlateau (adjust based on metrics)
plateau_sched = optim.lr_scheduler.ReduceLROnPlateau(
    optimizer,
    mode='min',
    factor=0.5,
    patience=5,
    verbose=True
)

# Warmup with cosine decay
from torch.optim.lr_scheduler import LinearLR, CosineAnnealingLR, SequentialLR

warmup = LinearLR(optimizer, start_factor=0.1, total_iters=5)
cosine = CosineAnnealingLR(optimizer, T_max=45)
scheduler = SequentialLR(optimizer, schedulers=[warmup, cosine], milestones=[5])

# Show LR changes
lrs = []
for epoch in range(50):
    lrs.append(optimizer.param_groups[0]['lr'])
    scheduler.step()

print("Learning rates over 50 epochs (first 10):", lrs[:10])
```

---

## 6. DataLoader และ Dataset {#dataloader-dataset}

### Custom Dataset

```python
# ตัวอย่างที่ 20: Custom Dataset class
import torch
from torch.utils.data import Dataset, DataLoader
import numpy as np
from pathlib import Path

class CustomImageDataset(Dataset):
    """Dataset สำหรับข้อมูล image ที่อยู่ในโฟลเดอร์"""
    
    def __init__(self, data, labels, transform=None):
        """
        Args:
            data: numpy array of images (N, H, W, C)
            labels: list or array of labels
            transform: torchvision transforms
        """
        self.data = torch.FloatTensor(data)
        self.labels = torch.LongTensor(labels)
        self.transform = transform
        
    def __len__(self):
        return len(self.labels)
    
    def __getitem__(self, idx):
        image = self.data[idx]
        label = self.labels[idx]
        
        if self.transform:
            image = self.transform(image)
        
        return image, label

# สร้าง dataset จาก synthetic data
N = 1000
images = np.random.randn(N, 28, 28, 1).astype(np.float32)
labels = np.random.randint(0, 10, N)

dataset = CustomImageDataset(images, labels)
print(f"Dataset size: {len(dataset)}")
print(f"Sample shape: {dataset[0][0].shape}, label: {dataset[0][1]}")

# DataLoader
train_loader = DataLoader(
    dataset,
    batch_size=32,
    shuffle=True,
    num_workers=0,    # ใช้ 4 ใน production
    pin_memory=True,  # เร็วขึ้นเมื่อใช้ GPU
    drop_last=False
)

# Iterate
for batch_idx, (images, labels) in enumerate(train_loader):
    print(f"Batch {batch_idx}: images={images.shape}, labels={labels.shape}")
    if batch_idx >= 2:
        break
```

```python
# ตัวอย่างที่ 21: Dataset with transforms
from torchvision import transforms

# Define transforms
train_transform = transforms.Compose([
    transforms.ToPILImage(),
    transforms.Resize((32, 32)),
    transforms.RandomHorizontalFlip(p=0.5),
    transforms.RandomRotation(degrees=15),
    transforms.ColorJitter(brightness=0.2, contrast=0.2),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                        std=[0.229, 0.224, 0.225])
])

val_transform = transforms.Compose([
    transforms.ToPILImage(),
    transforms.Resize((32, 32)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                        std=[0.229, 0.224, 0.225])
])

# CIFAR-10 with torchvision
from torchvision import datasets

# Download and load CIFAR-10
train_dataset = datasets.CIFAR10(
    root='./data',
    train=True,
    transform=train_transform,
    download=True
)

test_dataset = datasets.CIFAR10(
    root='./data',
    train=False,
    transform=val_transform
)

print(f"Train size: {len(train_dataset)}")
print(f"Test size: {len(test_dataset)}")
```

```python
# ตัวอย่างที่ 22: Weighted sampler for imbalanced datasets
from torch.utils.data import WeightedRandomSampler

# สมมติว่ามี class imbalance
# Class 0: 900 samples, Class 1: 100 samples
class_counts = [900, 100]
total = sum(class_counts)
class_weights = [total / count for count in class_counts]

# สร้าง sample weights
sample_labels = [0] * 900 + [1] * 100
sample_weights = [class_weights[label] for label in sample_labels]
sample_weights = torch.DoubleTensor(sample_weights)

sampler = WeightedRandomSampler(
    weights=sample_weights,
    num_samples=len(sample_weights),
    replacement=True
)

# DataLoader with sampler
balanced_loader = DataLoader(
    dataset,
    batch_size=32,
    sampler=sampler,  # Cannot use shuffle=True with sampler
    num_workers=0
)

print("Balanced DataLoader created")
```

---

## 7. Transfer Learning with torchvision {#transfer-learning}

```python
# ตัวอย่างที่ 23: Transfer learning with ResNet
import torch
import torch.nn as nn
import torchvision.models as models

# โหลด pretrained ResNet50
resnet = models.resnet50(pretrained=True)

# Freeze all layers ยกเว้น final layer
for param in resnet.parameters():
    param.requires_grad = False

# Replace final layer
num_classes = 5  # ปรับตามงานของเรา
resnet.fc = nn.Linear(resnet.fc.in_features, num_classes)

# ตอนนี้ training จะ update เฉพาะ fc layer
optimizer = torch.optim.Adam(resnet.fc.parameters(), lr=0.001)

# Fine-tuning: unfreeze some layers
# Unfreeze last block
for param in resnet.layer4.parameters():
    param.requires_grad = True

# ใช้ learning rate ต่างกันสำหรับ layer ต่างๆ
optimizer_ft = torch.optim.Adam([
    {'params': resnet.layer4.parameters(), 'lr': 0.0001},
    {'params': resnet.fc.parameters(), 'lr': 0.001}
])

print("ResNet50 for transfer learning:")
print(f"Total params: {sum(p.numel() for p in resnet.parameters()):,}")
trainable = sum(p.numel() for p in resnet.parameters() if p.requires_grad)
print(f"Trainable params: {trainable:,}")
```

```python
# ตัวอย่างที่ 24: Multiple pretrained models
# VGG16
vgg16 = models.vgg16(pretrained=True)
for param in vgg16.features.parameters():
    param.requires_grad = False
vgg16.classifier[6] = nn.Linear(4096, num_classes)

# EfficientNet
efficientnet = models.efficientnet_b0(pretrained=True)
efficientnet.classifier[1] = nn.Linear(
    efficientnet.classifier[1].in_features, num_classes
)

# MobileNetV3 (lightweight)
mobilenet = models.mobilenet_v3_small(pretrained=True)
mobilenet.classifier[3] = nn.Linear(
    mobilenet.classifier[3].in_features, num_classes
)

# ViT (Vision Transformer)
vit = models.vit_b_16(pretrained=True)
vit.heads.head = nn.Linear(
    vit.heads.head.in_features, num_classes
)

print("Models created!")
print(f"VGG16 params: {sum(p.numel() for p in vgg16.parameters()):,}")
print(f"EfficientNet-B0 params: {sum(p.numel() for p in efficientnet.parameters()):,}")
print(f"MobileNetV3-Small params: {sum(p.numel() for p in mobilenet.parameters()):,}")
```

---

## 8. GPU Training {#gpu-training}

```python
# ตัวอย่างที่ 25: GPU training setup
import torch
import torch.nn as nn

# Check GPU availability
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")

if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    print(f"Memory: {torch.cuda.get_device_properties(0).total_memory / 1e9:.1f} GB")

# Move model to GPU
model = SimpleNet(784, 256, 10).to(device)

# Move data to GPU
X = torch.randn(32, 784).to(device)
y = model(X)
print(f"Output on {y.device}")

# Multiple GPU training
if torch.cuda.device_count() > 1:
    model = nn.DataParallel(model)
    print(f"Using {torch.cuda.device_count()} GPUs!")

# Training with GPU
criterion = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters())

for epoch in range(5):
    for batch_X, batch_y in train_loader:
        # Move to GPU
        batch_X = batch_X.to(device)
        batch_y = batch_y.to(device)
        
        optimizer.zero_grad()
        output = model(batch_X)
        loss = criterion(output, batch_y)
        loss.backward()
        optimizer.step()
    
    print(f"Epoch {epoch+1} done")
```

```python
# ตัวอย่างที่ 26: Mixed precision training (faster on modern GPUs)
import torch
from torch.cuda.amp import autocast, GradScaler

model = SimpleNet(784, 256, 10).to(device)
optimizer = torch.optim.Adam(model.parameters())
scaler = GradScaler()  # For gradient scaling

for epoch in range(5):
    for batch_X, batch_y in train_loader:
        batch_X = batch_X.to(device)
        batch_y = batch_y.to(device)
        
        optimizer.zero_grad()
        
        # Mixed precision forward pass
        with autocast():
            output = model(batch_X)
            loss = criterion(output, batch_y)
        
        # Scaled backward pass
        scaler.scale(loss).backward()
        scaler.step(optimizer)
        scaler.update()
    
    print(f"Epoch {epoch+1}: loss = {loss.item():.4f}")
```

---

## 9. Model Saving/Loading {#model-saving-loading}

```python
# ตัวอย่างที่ 27: Saving and loading models
import torch

model = SimpleNet(784, 256, 10)
optimizer = torch.optim.Adam(model.parameters())

# Method 1: Save only weights (recommended)
torch.save(model.state_dict(), 'model_weights.pth')

# Load weights
model_loaded = SimpleNet(784, 256, 10)
model_loaded.load_state_dict(torch.load('model_weights.pth'))
model_loaded.eval()  # Set to evaluation mode

# Method 2: Save entire model (less portable)
torch.save(model, 'entire_model.pth')
model2 = torch.load('entire_model.pth')

# Method 3: Save checkpoint (best practice)
checkpoint = {
    'epoch': 50,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': 0.023,
    'accuracy': 0.97,
    'config': {
        'input_size': 784,
        'hidden_size': 256,
        'output_size': 10
    }
}
torch.save(checkpoint, 'checkpoint.pth')

# Load checkpoint
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
start_epoch = checkpoint['epoch']
print(f"Resumed from epoch {start_epoch}")
print(f"Previous loss: {checkpoint['loss']}")
```

---

## 10. TorchScript {#torchscript}

```python
# ตัวอย่างที่ 28: TorchScript for production deployment
import torch
import torch.nn as nn

class ScriptableModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.linear = nn.Linear(10, 5)
        
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return torch.relu(self.linear(x))

model = ScriptableModel()

# Method 1: Scripting (works for most models)
scripted_model = torch.jit.script(model)
print("Scripted model:")
print(scripted_model.code)

# Method 2: Tracing (for models without control flow)
example_input = torch.randn(1, 10)
traced_model = torch.jit.trace(model, example_input)

# Save and load scripted model
scripted_model.save('scripted_model.pt')
loaded = torch.jit.load('scripted_model.pt')

# Use the scripted model
x = torch.randn(4, 10)
output = loaded(x)
print(f"Output: {output.shape}")
```

---

## 11. PyTorch Lightning {#pytorch-lightning}

```python
# ตัวอย่างที่ 29: PyTorch Lightning
# pip install lightning

import torch
import torch.nn as nn
import lightning as L
from torch.utils.data import DataLoader, TensorDataset

class LightningModel(L.LightningModule):
    def __init__(self, input_size, hidden_size, num_classes, lr=0.001):
        super().__init__()
        self.save_hyperparameters()  # Saves all __init__ args
        
        self.model = nn.Sequential(
            nn.Linear(input_size, hidden_size),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(hidden_size, num_classes)
        )
        self.criterion = nn.CrossEntropyLoss()
        
    def forward(self, x):
        return self.model(x)
    
    def training_step(self, batch, batch_idx):
        x, y = batch
        y_hat = self(x)
        loss = self.criterion(y_hat, y)
        acc = (y_hat.argmax(1) == y).float().mean()
        
        self.log('train_loss', loss, on_epoch=True, prog_bar=True)
        self.log('train_acc', acc, on_epoch=True, prog_bar=True)
        return loss
    
    def validation_step(self, batch, batch_idx):
        x, y = batch
        y_hat = self(x)
        loss = self.criterion(y_hat, y)
        acc = (y_hat.argmax(1) == y).float().mean()
        
        self.log('val_loss', loss, prog_bar=True)
        self.log('val_acc', acc, prog_bar=True)
    
    def configure_optimizers(self):
        optimizer = torch.optim.Adam(
            self.parameters(), 
            lr=self.hparams.lr
        )
        scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(
            optimizer, T_max=50
        )
        return {
            'optimizer': optimizer,
            'lr_scheduler': scheduler,
            'monitor': 'val_loss'
        }

# Create data
X = torch.randn(1000, 20)
y = torch.randint(0, 5, (1000,))
train_ds = TensorDataset(X[:800], y[:800])
val_ds = TensorDataset(X[800:], y[800:])

train_dl = DataLoader(train_ds, batch_size=32, shuffle=True)
val_dl = DataLoader(val_ds, batch_size=32)

# Train with Trainer
model = LightningModel(20, 64, 5, lr=0.001)

trainer = L.Trainer(
    max_epochs=10,
    accelerator='auto',  # auto-detect GPU/CPU
    devices=1,
    enable_progress_bar=True,
    log_every_n_steps=10,
)

trainer.fit(model, train_dl, val_dl)
print("Training complete!")
```

---

## 12. ตัวอย่างโปรแกรมจริง {#real-examples}

### Example 1: Image Classifier (CIFAR-10)

```python
# ตัวอย่างที่ 30: Complete Image Classifier
import torch
import torch.nn as nn
import torch.optim as optim
import torchvision
import torchvision.transforms as transforms
from torch.utils.data import DataLoader

# Data transforms
transform_train = transforms.Compose([
    transforms.RandomCrop(32, padding=4),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465),
                        (0.2023, 0.1994, 0.2010))
])

transform_test = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465),
                        (0.2023, 0.1994, 0.2010))
])

# Load CIFAR-10
# trainset = torchvision.datasets.CIFAR10('./data', train=True, 
#                                          transform=transform_train, download=True)
# testset = torchvision.datasets.CIFAR10('./data', train=False,
#                                         transform=transform_test)

# Define ResNet-style block
class ResBlock(nn.Module):
    def __init__(self, in_channels, out_channels, stride=1):
        super().__init__()
        self.conv1 = nn.Conv2d(in_channels, out_channels, 3, stride, 1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)
        self.conv2 = nn.Conv2d(out_channels, out_channels, 3, 1, 1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        
        self.shortcut = nn.Sequential()
        if stride != 1 or in_channels != out_channels:
            self.shortcut = nn.Sequential(
                nn.Conv2d(in_channels, out_channels, 1, stride, bias=False),
                nn.BatchNorm2d(out_channels)
            )
    
    def forward(self, x):
        out = torch.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        out += self.shortcut(x)
        out = torch.relu(out)
        return out

class CIFAR10ResNet(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.conv1 = nn.Conv2d(3, 64, 3, 1, 1, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        
        self.layer1 = self._make_layer(64, 64, 2, stride=1)
        self.layer2 = self._make_layer(64, 128, 2, stride=2)
        self.layer3 = self._make_layer(128, 256, 2, stride=2)
        
        self.pool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(256, num_classes)
    
    def _make_layer(self, in_ch, out_ch, num_blocks, stride):
        layers = [ResBlock(in_ch, out_ch, stride)]
        for _ in range(1, num_blocks):
            layers.append(ResBlock(out_ch, out_ch, 1))
        return nn.Sequential(*layers)
    
    def forward(self, x):
        x = torch.relu(self.bn1(self.conv1(x)))
        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.pool(x)
        x = x.view(x.size(0), -1)
        return self.fc(x)

# Create model
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = CIFAR10ResNet(num_classes=10).to(device)
print(f"Model params: {sum(p.numel() for p in model.parameters()):,}")

# Training would be done here with train_loader
# (shortened for demonstration)
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=0.1, momentum=0.9, weight_decay=5e-4)
scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=200)

print("CIFAR-10 ResNet ready for training!")
```

### Example 2: Regression with Neural Network

```python
# ตัวอย่างที่ 31: Neural Network Regression
import torch
import torch.nn as nn
import numpy as np
import matplotlib.pyplot as plt

# สร้าง synthetic regression data
torch.manual_seed(42)
X = torch.linspace(-3, 3, 500).unsqueeze(1)
y = torch.sin(X) + 0.2 * torch.randn(500, 1)

# Train/test split
split = int(0.8 * len(X))
X_train, X_test = X[:split], X[split:]
y_train, y_test = y[:split], y[split:]

class RegressionNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(1, 64),
            nn.Tanh(),
            nn.Linear(64, 128),
            nn.Tanh(),
            nn.Linear(128, 64),
            nn.Tanh(),
            nn.Linear(64, 1)
        )
    
    def forward(self, x):
        return self.net(x)

model = RegressionNet()
optimizer = torch.optim.Adam(model.parameters(), lr=0.01)
criterion = nn.MSELoss()

# Training
train_losses = []
for epoch in range(500):
    model.train()
    pred = model(X_train)
    loss = criterion(pred, y_train)
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    
    train_losses.append(loss.item())

# Evaluation
model.eval()
with torch.no_grad():
    test_pred = model(X_test)
    test_mse = criterion(test_pred, y_test)
    print(f"Final Test MSE: {test_mse.item():.4f}")
    print(f"Final Test RMSE: {test_mse.item()**0.5:.4f}")

print("Regression model trained!")
```

### Example 3: Time Series with LSTM

```python
# ตัวอย่างที่ 32: Time Series Prediction with LSTM
import torch
import torch.nn as nn
import numpy as np

# สร้าง time series data (sine wave + noise)
def create_dataset(data, seq_length):
    X, y = [], []
    for i in range(len(data) - seq_length):
        X.append(data[i:i+seq_length])
        y.append(data[i+seq_length])
    return torch.FloatTensor(X).unsqueeze(-1), torch.FloatTensor(y)

# Generate data
t = np.linspace(0, 20*np.pi, 2000)
data = np.sin(t) + 0.1 * np.random.randn(len(t))

SEQ_LENGTH = 50
X, y = create_dataset(data, SEQ_LENGTH)

# Split
split = int(0.8 * len(X))
X_train, X_test = X[:split], X[split:]
y_train, y_test = y[:split], y[split:]

class LSTMModel(nn.Module):
    def __init__(self, input_size=1, hidden_size=64, num_layers=2, output_size=1):
        super().__init__()
        self.lstm = nn.LSTM(
            input_size, hidden_size, num_layers,
            batch_first=True,
            dropout=0.2
        )
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        # x: (batch, seq, features)
        lstm_out, _ = self.lstm(x)
        # Use last hidden state
        out = self.fc(lstm_out[:, -1, :])
        return out

model = LSTMModel()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
criterion = nn.MSELoss()

# Training
batch_size = 64
for epoch in range(50):
    model.train()
    epoch_loss = 0
    
    for i in range(0, len(X_train), batch_size):
        X_batch = X_train[i:i+batch_size]
        y_batch = y_train[i:i+batch_size]
        
        optimizer.zero_grad()
        pred = model(X_batch).squeeze()
        loss = criterion(pred, y_batch)
        loss.backward()
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        optimizer.step()
        epoch_loss += loss.item()
    
    if (epoch + 1) % 10 == 0:
        model.eval()
        with torch.no_grad():
            test_pred = model(X_test).squeeze()
            test_loss = criterion(test_pred, y_test)
        print(f"Epoch {epoch+1}: Train Loss = {epoch_loss:.4f}, Test MSE = {test_loss:.4f}")

print("LSTM Time Series model trained!")
```

```python
# ตัวอย่างที่ 33: Transformer for sequence classification
import torch
import torch.nn as nn
import math

class PositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000):
        super().__init__()
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len).float().unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * 
                           (-math.log(10000.0) / d_model))
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)
        self.register_buffer('pe', pe)
    
    def forward(self, x):
        return x + self.pe[:, :x.size(1)]

class TransformerClassifier(nn.Module):
    def __init__(self, vocab_size, d_model=128, nhead=8, 
                 num_layers=3, num_classes=5):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, d_model)
        self.pos_enc = PositionalEncoding(d_model)
        
        encoder_layer = nn.TransformerEncoderLayer(
            d_model, nhead, dim_feedforward=512,
            dropout=0.1, batch_first=True
        )
        self.transformer = nn.TransformerEncoder(encoder_layer, num_layers)
        self.classifier = nn.Linear(d_model, num_classes)
        
    def forward(self, x, mask=None):
        x = self.embedding(x)
        x = self.pos_enc(x)
        x = self.transformer(x, src_key_padding_mask=mask)
        # Classify using CLS token (first token)
        return self.classifier(x[:, 0, :])

# Test
model = TransformerClassifier(vocab_size=10000, num_classes=5)
x = torch.randint(0, 10000, (8, 50))  # batch=8, seq_len=50
output = model(x)
print(f"Transformer output: {output.shape}")  # (8, 5)
```

```python
# ตัวอย่างที่ 34: GAN (Generative Adversarial Network)
import torch
import torch.nn as nn

class Generator(nn.Module):
    def __init__(self, latent_dim=100, img_size=28):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(latent_dim, 256),
            nn.LeakyReLU(0.2),
            nn.BatchNorm1d(256),
            nn.Linear(256, 512),
            nn.LeakyReLU(0.2),
            nn.BatchNorm1d(512),
            nn.Linear(512, img_size * img_size),
            nn.Tanh()
        )
        self.img_size = img_size
    
    def forward(self, z):
        return self.net(z).view(-1, 1, self.img_size, self.img_size)

class Discriminator(nn.Module):
    def __init__(self, img_size=28):
        super().__init__()
        self.net = nn.Sequential(
            nn.Flatten(),
            nn.Linear(img_size * img_size, 512),
            nn.LeakyReLU(0.2),
            nn.Dropout(0.3),
            nn.Linear(512, 256),
            nn.LeakyReLU(0.2),
            nn.Dropout(0.3),
            nn.Linear(256, 1),
            nn.Sigmoid()
        )
    
    def forward(self, x):
        return self.net(x)

# Initialize
G = Generator()
D = Discriminator()
g_optimizer = torch.optim.Adam(G.parameters(), lr=0.0002, betas=(0.5, 0.999))
d_optimizer = torch.optim.Adam(D.parameters(), lr=0.0002, betas=(0.5, 0.999))
criterion = nn.BCELoss()

# Training step
def train_gan_step(real_imgs):
    batch_size = real_imgs.size(0)
    real_labels = torch.ones(batch_size, 1)
    fake_labels = torch.zeros(batch_size, 1)
    
    # Train Discriminator
    d_optimizer.zero_grad()
    real_pred = D(real_imgs)
    d_loss_real = criterion(real_pred, real_labels)
    
    z = torch.randn(batch_size, 100)
    fake_imgs = G(z).detach()
    fake_pred = D(fake_imgs)
    d_loss_fake = criterion(fake_pred, fake_labels)
    
    d_loss = d_loss_real + d_loss_fake
    d_loss.backward()
    d_optimizer.step()
    
    # Train Generator
    g_optimizer.zero_grad()
    z = torch.randn(batch_size, 100)
    fake_imgs = G(z)
    fake_pred = D(fake_imgs)
    g_loss = criterion(fake_pred, real_labels)  # Want D to think fake is real
    g_loss.backward()
    g_optimizer.step()
    
    return d_loss.item(), g_loss.item()

print("GAN created successfully!")

# Test one step
real_imgs = torch.randn(32, 1, 28, 28)
d_loss, g_loss = train_gan_step(real_imgs)
print(f"D loss: {d_loss:.4f}, G loss: {g_loss:.4f}")
```

```python
# ตัวอย่างที่ 35: Complete training pipeline with callbacks
import torch
import torch.nn as nn
from dataclasses import dataclass, field
from typing import List, Dict, Callable, Optional
import time

@dataclass
class TrainingConfig:
    max_epochs: int = 100
    batch_size: int = 32
    learning_rate: float = 0.001
    weight_decay: float = 1e-4
    patience: int = 10
    save_dir: str = 'checkpoints'

class EarlyStopping:
    def __init__(self, patience=10, min_delta=1e-4):
        self.patience = patience
        self.min_delta = min_delta
        self.counter = 0
        self.best_loss = float('inf')
        self.should_stop = False
    
    def __call__(self, val_loss):
        if val_loss < self.best_loss - self.min_delta:
            self.best_loss = val_loss
            self.counter = 0
        else:
            self.counter += 1
            if self.counter >= self.patience:
                self.should_stop = True
        return self.should_stop

class Trainer:
    def __init__(self, model, config: TrainingConfig):
        self.model = model
        self.config = config
        self.device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
        self.model.to(self.device)
        
        self.optimizer = torch.optim.AdamW(
            model.parameters(), 
            lr=config.learning_rate,
            weight_decay=config.weight_decay
        )
        self.criterion = nn.CrossEntropyLoss()
        self.early_stopping = EarlyStopping(patience=config.patience)
        self.history = {
            'train_loss': [], 'val_loss': [], 
            'train_acc': [], 'val_acc': []
        }
    
    def train_epoch(self, loader):
        self.model.train()
        losses, correct, total = [], 0, 0
        
        for X, y in loader:
            X, y = X.to(self.device), y.to(self.device)
            self.optimizer.zero_grad()
            pred = self.model(X)
            loss = self.criterion(pred, y)
            loss.backward()
            nn.utils.clip_grad_norm_(self.model.parameters(), 1.0)
            self.optimizer.step()
            
            losses.append(loss.item())
            correct += (pred.argmax(1) == y).sum().item()
            total += len(y)
        
        return sum(losses)/len(losses), correct/total
    
    @torch.no_grad()
    def evaluate(self, loader):
        self.model.eval()
        losses, correct, total = [], 0, 0
        
        for X, y in loader:
            X, y = X.to(self.device), y.to(self.device)
            pred = self.model(X)
            loss = self.criterion(pred, y)
            losses.append(loss.item())
            correct += (pred.argmax(1) == y).sum().item()
            total += len(y)
        
        return sum(losses)/len(losses), correct/total
    
    def fit(self, train_loader, val_loader):
        print(f"Training on {self.device}")
        start_time = time.time()
        
        for epoch in range(self.config.max_epochs):
            train_loss, train_acc = self.train_epoch(train_loader)
            val_loss, val_acc = self.evaluate(val_loader)
            
            self.history['train_loss'].append(train_loss)
            self.history['val_loss'].append(val_loss)
            self.history['train_acc'].append(train_acc)
            self.history['val_acc'].append(val_acc)
            
            if (epoch + 1) % 10 == 0:
                elapsed = time.time() - start_time
                print(f"Epoch {epoch+1}/{self.config.max_epochs} "
                      f"[{elapsed:.0f}s] | "
                      f"Train: loss={train_loss:.4f} acc={train_acc:.4f} | "
                      f"Val: loss={val_loss:.4f} acc={val_acc:.4f}")
            
            if self.early_stopping(val_loss):
                print(f"Early stopping at epoch {epoch+1}")
                break
        
        print(f"Training complete! Total time: {time.time()-start_time:.0f}s")
        return self.history

# Test the trainer
config = TrainingConfig(max_epochs=20, batch_size=32)
model = SimpleNet(20, 64, 5)
trainer = Trainer(model, config)
print("Trainer created!")
```

---

## 13. แบบฝึกหัด {#exercises}

### แบบฝึกหัดที่ 1: Tensor Manipulation

**โจทย์:** สร้าง tensor ขนาด (5, 4) จาก random normal distribution จากนั้น:
1. Normalize ให้ mean=0, std=1 ตาม column
2. หา top-3 values และ indices ในแต่ละ row
3. สร้าง boolean mask สำหรับ values ที่ > 0.5

```python
# เฉลย
import torch

torch.manual_seed(42)
x = torch.randn(5, 4)
print("Original:\n", x)

# Normalize ตาม column
mean = x.mean(dim=0, keepdim=True)
std = x.std(dim=0, keepdim=True)
x_norm = (x - mean) / (std + 1e-8)
print("\nNormalized:\n", x_norm)

# Top-3 values per row
values, indices = x.topk(3, dim=1)
print("\nTop-3 values:\n", values)
print("Indices:\n", indices)

# Boolean mask
mask = x > 0.5
print("\nMask (x > 0.5):\n", mask)
print("Values > 0.5:", x[mask])
```

### แบบฝึกหัดที่ 2: Custom Loss Function

**โจทย์:** Implement Focal Loss สำหรับ imbalanced classification

```python
# เฉลย
import torch
import torch.nn as nn
import torch.nn.functional as F

class FocalLoss(nn.Module):
    """
    Focal Loss for addressing class imbalance.
    FL(p_t) = -alpha_t * (1 - p_t)^gamma * log(p_t)
    """
    def __init__(self, gamma=2.0, alpha=None, reduction='mean'):
        super().__init__()
        self.gamma = gamma
        self.alpha = alpha
        self.reduction = reduction
    
    def forward(self, inputs, targets):
        # inputs: (N, C), targets: (N,)
        ce_loss = F.cross_entropy(inputs, targets, reduction='none')
        
        # Compute p_t
        p = torch.softmax(inputs, dim=1)
        p_t = p[torch.arange(len(targets)), targets]
        
        # Focal weight
        focal_weight = (1 - p_t) ** self.gamma
        
        # Apply alpha if provided
        if self.alpha is not None:
            alpha_t = self.alpha[targets]
            focal_weight = alpha_t * focal_weight
        
        focal_loss = focal_weight * ce_loss
        
        if self.reduction == 'mean':
            return focal_loss.mean()
        elif self.reduction == 'sum':
            return focal_loss.sum()
        return focal_loss

# Test
criterion = FocalLoss(gamma=2.0)
logits = torch.randn(8, 5)
targets = torch.randint(0, 5, (8,))
loss = criterion(logits, targets)
print(f"Focal Loss: {loss.item():.4f}")
```

### แบบฝึกหัดที่ 3: Custom Dataset

**โจทย์:** สร้าง Dataset class สำหรับ text classification ที่รองรับ caching

```python
# เฉลย
import torch
from torch.utils.data import Dataset
import hashlib
import pickle
from pathlib import Path

class TextDataset(Dataset):
    def __init__(self, texts, labels, tokenizer, max_len=128, cache_dir=None):
        self.texts = texts
        self.labels = labels
        self.tokenizer = tokenizer
        self.max_len = max_len
        self.cache_dir = Path(cache_dir) if cache_dir else None
        self._cache = {}
        
        if self.cache_dir:
            self.cache_dir.mkdir(exist_ok=True)
    
    def __len__(self):
        return len(self.texts)
    
    def _get_cache_key(self, text):
        return hashlib.md5(text.encode()).hexdigest()
    
    def __getitem__(self, idx):
        text = self.texts[idx]
        label = self.labels[idx]
        
        # Check in-memory cache
        if text in self._cache:
            return self._cache[text], label
        
        # Check disk cache
        if self.cache_dir:
            cache_key = self._get_cache_key(text)
            cache_file = self.cache_dir / f"{cache_key}.pkl"
            if cache_file.exists():
                with open(cache_file, 'rb') as f:
                    tokens = pickle.load(f)
                self._cache[text] = tokens
                return tokens, label
        
        # Tokenize
        tokens = self.tokenizer(text, max_len=self.max_len)
        
        # Save to cache
        self._cache[text] = tokens
        if self.cache_dir:
            cache_key = self._get_cache_key(text)
            cache_file = self.cache_dir / f"{cache_key}.pkl"
            with open(cache_file, 'wb') as f:
                pickle.dump(tokens, f)
        
        return tokens, label

# Simple tokenizer for testing
def simple_tokenizer(text, max_len=128):
    # ใน practice จะใช้ BPE tokenizer
    chars = [ord(c) % 100 for c in text[:max_len]]
    # Pad to max_len
    chars += [0] * (max_len - len(chars))
    return torch.tensor(chars, dtype=torch.long)

# Test
texts = ["Hello world!", "Python is great", "Deep learning rocks"]
labels = [0, 1, 2]
dataset = TextDataset(texts, labels, simple_tokenizer, max_len=32)

for i, (tokens, label) in enumerate(dataset):
    print(f"Sample {i}: tokens={tokens[:5]}... label={label}")
```

### แบบฝึกหัดที่ 4: Learning Rate Finder

**โจทย์:** Implement learning rate finder (Leslie Smith's LR range test)

```python
# เฉลย
import torch
import torch.nn as nn
import numpy as np
from copy import deepcopy

def lr_finder(model, train_loader, criterion, device,
              start_lr=1e-7, end_lr=10, num_iter=100):
    """Find optimal learning rate using LR range test"""
    
    # Save initial state
    initial_state = deepcopy(model.state_dict())
    
    optimizer = torch.optim.SGD(model.parameters(), lr=start_lr)
    
    # Calculate lr multiplier
    lr_mult = (end_lr / start_lr) ** (1 / (num_iter - 1))
    
    lrs, losses = [], []
    best_loss = float('inf')
    avg_loss = 0
    beta = 0.98  # Smoothing factor
    
    model.train()
    data_iter = iter(train_loader)
    
    for i in range(num_iter):
        try:
            X, y = next(data_iter)
        except StopIteration:
            data_iter = iter(train_loader)
            X, y = next(data_iter)
        
        X, y = X.to(device), y.to(device)
        
        optimizer.zero_grad()
        pred = model(X)
        loss = criterion(pred, y)
        
        # Smooth the loss
        avg_loss = beta * avg_loss + (1 - beta) * loss.item()
        smooth_loss = avg_loss / (1 - beta ** (i + 1))
        
        # Stop if loss explodes
        if i > 1 and smooth_loss > 4 * best_loss:
            break
        
        if smooth_loss < best_loss:
            best_loss = smooth_loss
        
        lrs.append(optimizer.param_groups[0]['lr'])
        losses.append(smooth_loss)
        
        loss.backward()
        optimizer.step()
        
        # Update lr
        optimizer.param_groups[0]['lr'] *= lr_mult
    
    # Restore initial state
    model.load_state_dict(initial_state)
    
    # Find optimal lr (at steepest gradient)
    gradients = np.gradient(losses)
    optimal_idx = np.argmin(gradients)
    optimal_lr = lrs[optimal_idx]
    
    print(f"Optimal LR: {optimal_lr:.2e}")
    return lrs, losses, optimal_lr

print("LR Finder implemented!")
```

### แบบฝึกหัดที่ 5: Grad-CAM Visualization

**โจทย์:** Implement Gradient-weighted Class Activation Mapping

```python
# เฉลย
import torch
import torch.nn as nn
import numpy as np

class GradCAM:
    """Gradient-weighted Class Activation Mapping"""
    
    def __init__(self, model, target_layer):
        self.model = model
        self.target_layer = target_layer
        self.gradients = None
        self.activations = None
        
        # Register hooks
        self.forward_hook = target_layer.register_forward_hook(
            self._save_activation
        )
        self.backward_hook = target_layer.register_backward_hook(
            self._save_gradient
        )
    
    def _save_activation(self, module, input, output):
        self.activations = output.detach()
    
    def _save_gradient(self, module, grad_input, grad_output):
        self.gradients = grad_output[0].detach()
    
    def generate(self, input_tensor, class_idx=None):
        # Forward pass
        self.model.eval()
        output = self.model(input_tensor)
        
        if class_idx is None:
            class_idx = output.argmax(dim=1).item()
        
        # Backward pass
        self.model.zero_grad()
        score = output[0, class_idx]
        score.backward()
        
        # Compute Grad-CAM
        weights = self.gradients.mean(dim=(2, 3), keepdim=True)  # GAP
        cam = (weights * self.activations).sum(dim=1, keepdim=True)
        cam = torch.relu(cam)  # ReLU
        
        # Normalize
        cam = cam - cam.min()
        cam = cam / (cam.max() + 1e-8)
        
        return cam.squeeze().numpy()
    
    def remove_hooks(self):
        self.forward_hook.remove()
        self.backward_hook.remove()

# Test with simple CNN
model = CNN(num_classes=10)
grad_cam = GradCAM(model, model.conv3)

x = torch.randn(1, 1, 28, 28)
cam = grad_cam.generate(x)
print(f"Grad-CAM output shape: {cam.shape}")
grad_cam.remove_hooks()
```

### แบบฝึกหัดที่ 6: Model Distillation

**โจทย์:** Knowledge Distillation จาก teacher model ไปยัง student model

```python
# เฉลย
import torch
import torch.nn as nn
import torch.nn.functional as F

def distillation_loss(student_logits, teacher_logits, true_labels,
                      T=3.0, alpha=0.7):
    """
    Knowledge Distillation Loss
    
    Args:
        T: Temperature (higher = softer probabilities)
        alpha: Weight for distillation loss (1-alpha for CE loss)
    """
    # Soft targets from teacher
    soft_targets = F.softmax(teacher_logits / T, dim=1)
    soft_predictions = F.log_softmax(student_logits / T, dim=1)
    
    # KL divergence (distillation loss)
    distill_loss = F.kl_div(
        soft_predictions, soft_targets,
        reduction='batchmean'
    ) * (T ** 2)  # Scale by T^2
    
    # Hard targets (regular CE loss)
    hard_loss = F.cross_entropy(student_logits, true_labels)
    
    # Combined loss
    return alpha * distill_loss + (1 - alpha) * hard_loss

# Create teacher and student models
teacher = CNN(num_classes=10)   # Larger
student = SimpleNet(784, 128, 10)  # Smaller

# Freeze teacher
for param in teacher.parameters():
    param.requires_grad = False

# Training with distillation
optimizer = torch.optim.Adam(student.parameters(), lr=0.001)

# Example batch
x_cnn = torch.randn(8, 1, 28, 28)
x_mlp = x_cnn.view(8, -1)
labels = torch.randint(0, 10, (8,))

with torch.no_grad():
    teacher_logits = teacher(x_cnn)

student_logits = student(x_mlp)
loss = distillation_loss(student_logits, teacher_logits, labels)
print(f"Distillation loss: {loss.item():.4f}")
```

### แบบฝึกหัดที่ 7: Custom Optimizer

**โจทย์:** Implement Lion optimizer (ดีกว่า Adam ในบางงาน)

```python
# เฉลย
import torch
from torch.optim import Optimizer

class Lion(Optimizer):
    """
    Lion (EvoLved Sign Momentum) Optimizer
    Paper: https://arxiv.org/abs/2302.06675
    """
    
    def __init__(self, params, lr=1e-4, betas=(0.9, 0.99), weight_decay=0.0):
        defaults = dict(lr=lr, betas=betas, weight_decay=weight_decay)
        super().__init__(params, defaults)
    
    @torch.no_grad()
    def step(self, closure=None):
        loss = None
        if closure is not None:
            with torch.enable_grad():
                loss = closure()
        
        for group in self.param_groups:
            for p in group['params']:
                if p.grad is None:
                    continue
                
                grad = p.grad
                beta1, beta2 = group['betas']
                lr = group['lr']
                weight_decay = group['weight_decay']
                
                # Weight decay
                if weight_decay != 0:
                    grad = grad.add(p, alpha=weight_decay)
                
                # Get or initialize momentum
                state = self.state[p]
                if 'momentum' not in state:
                    state['momentum'] = torch.zeros_like(p)
                
                m = state['momentum']
                
                # Update using sign of interpolation
                update = m.mul(beta1).add_(grad, alpha=1 - beta1).sign_()
                p.add_(update, alpha=-lr)
                
                # Update momentum
                m.mul_(beta2).add_(grad, alpha=1 - beta2)
        
        return loss

# Test
model = SimpleNet(784, 256, 10)
optimizer = Lion(model.parameters(), lr=1e-4, weight_decay=0.01)

x = torch.randn(8, 784)
y = torch.randint(0, 10, (8,))
pred = model(x)
loss = nn.CrossEntropyLoss()(pred, y)
loss.backward()
optimizer.step()
print(f"Lion optimizer step completed, loss: {loss.item():.4f}")
```

### แบบฝึกหัดที่ 8: Neural Architecture Search (NAS) - Simple Version

**โจทย์:** Implement simple architecture search สำหรับ MLP

```python
# เฉลย
import torch
import torch.nn as nn
import itertools
from torch.utils.data import DataLoader, TensorDataset

class FlexibleMLP(nn.Module):
    def __init__(self, input_dim, hidden_dims, output_dim, 
                 activation='relu', dropout=0.0):
        super().__init__()
        
        activations = {
            'relu': nn.ReLU(),
            'gelu': nn.GELU(),
            'tanh': nn.Tanh(),
            'silu': nn.SiLU()
        }
        
        layers = []
        prev_dim = input_dim
        
        for dim in hidden_dims:
            layers.extend([
                nn.Linear(prev_dim, dim),
                activations[activation],
                nn.Dropout(dropout)
            ])
            prev_dim = dim
        
        layers.append(nn.Linear(prev_dim, output_dim))
        self.net = nn.Sequential(*layers)
    
    def forward(self, x):
        return self.net(x)
    
    @property
    def num_params(self):
        return sum(p.numel() for p in self.parameters())

def simple_nas(input_dim, output_dim, train_loader, val_loader,
               search_epochs=5):
    """Simple grid search NAS"""
    
    # Search space
    hidden_configs = [
        [64], [128], [256],
        [64, 64], [128, 64], [256, 128],
        [128, 64, 32]
    ]
    activations = ['relu', 'gelu', 'silu']
    dropouts = [0.0, 0.2]
    
    results = []
    device = torch.device('cpu')
    criterion = nn.CrossEntropyLoss()
    
    for hidden_dims, activation, dropout in itertools.product(
        hidden_configs, activations, dropouts
    ):
        model = FlexibleMLP(input_dim, hidden_dims, output_dim,
                           activation, dropout).to(device)
        optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
        
        # Quick training
        for _ in range(search_epochs):
            model.train()
            for X, y in train_loader:
                optimizer.zero_grad()
                loss = criterion(model(X), y)
                loss.backward()
                optimizer.step()
        
        # Evaluate
        model.eval()
        correct, total = 0, 0
        with torch.no_grad():
            for X, y in val_loader:
                pred = model(X).argmax(1)
                correct += (pred == y).sum().item()
                total += len(y)
        val_acc = correct / total
        
        results.append({
            'hidden_dims': hidden_dims,
            'activation': activation,
            'dropout': dropout,
            'val_acc': val_acc,
            'num_params': model.num_params
        })
        
        print(f"Config: {hidden_dims}, {activation}, dropout={dropout} "
              f"-> Val Acc: {val_acc:.4f}, Params: {model.num_params:,}")
    
    # Find best
    best = max(results, key=lambda x: x['val_acc'])
    print(f"\nBest config: {best}")
    return best

# Test NAS
X = torch.randn(500, 20)
y = torch.randint(0, 5, (500,))
train_ds = TensorDataset(X[:400], y[:400])
val_ds = TensorDataset(X[400:], y[400:])
train_dl = DataLoader(train_ds, batch_size=32, shuffle=True)
val_dl = DataLoader(val_ds, batch_size=32)

best_config = simple_nas(20, 5, train_dl, val_dl, search_epochs=3)
```

---

## สรุป

ใน Part 81 นี้ เราได้เรียนรู้:

1. **PyTorch vs TensorFlow** - การเลือกใช้ framework ที่เหมาะสม
2. **Tensors** - การสร้าง จัดการ และ operations
3. **Autograd** - Automatic differentiation สำหรับ training
4. **nn.Module** - การสร้าง neural network layers
5. **Training Loop** - Complete pipeline ด้วย best practices
6. **DataLoader** - การโหลดและ preprocess data อย่างมีประสิทธิภาพ
7. **Transfer Learning** - ใช้ pretrained models
8. **GPU Training** - เร่งความเร็วด้วย CUDA
9. **Model I/O** - Saving/loading checkpoints
10. **TorchScript** - Production deployment
11. **PyTorch Lightning** - Clean training code

### ขั้นต่อไป
- Part 82: NLP - Natural Language Processing
- ศึกษาเพิ่มเติม: https://pytorch.org/tutorials/

---
*Part 81 - Deep Learning: PyTorch | Python Course*
