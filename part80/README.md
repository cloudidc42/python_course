# Part 80: Deep Learning - TensorFlow & Keras

## บทนำ

Deep Learning คือ subset ของ Machine Learning ที่ใช้ Neural Networks หลายชั้น (layers) TensorFlow เป็น framework หลักสำหรับ Deep Learning พัฒนาโดย Google ส่วน Keras เป็น high-level API ที่ทำให้การสร้าง Neural Networks ง่ายขึ้น

ในบทนี้เราจะเรียน:
- Deep Learning concepts และ Neural Network fundamentals
- TensorFlow 2.x และ Keras API
- Sequential Model และ Functional API
- Layer types หลากหลาย
- Activation functions, Loss functions, Optimizers
- Callbacks สำหรับ training control
- Transfer learning
- Model saving/loading
- GPU support

---

## 1. Deep Learning Concepts

### 1.1 Neural Network Fundamentals

```python
import numpy as np
import matplotlib.pyplot as plt

# Activation Functions
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

def relu(x):
    return np.maximum(0, x)

def tanh(x):
    return np.tanh(x)

def leaky_relu(x, alpha=0.01):
    return np.where(x > 0, x, alpha * x)

def softmax(x):
    e_x = np.exp(x - np.max(x))
    return e_x / e_x.sum()

# Plot activation functions
x = np.linspace(-3, 3, 300)

fig, axes = plt.subplots(2, 3, figsize=(15, 8))

for ax, (name, func, color) in zip(axes.flatten(), [
    ('Sigmoid', sigmoid, 'blue'),
    ('ReLU', relu, 'red'),
    ('Tanh', tanh, 'green'),
    ('Leaky ReLU', leaky_relu, 'purple'),
    ('Linear', lambda x: x, 'orange'),
    ('Softmax\n(x=[1,2,3])', lambda x: [softmax(np.array([x[i], x[i]+1, x[i]+2]))[0] for i in range(len(x))], 'brown')
]):
    ax.plot(x, [func(xi) if not callable(getattr(func, '__self__', None)) else func(xi) for xi in x] 
             if name != 'Softmax\n(x=[1,2,3])' else func(x), 
             color=color, linewidth=2)
    ax.set_title(name)
    ax.axhline(y=0, color='black', linestyle='--', alpha=0.3)
    ax.axvline(x=0, color='black', linestyle='--', alpha=0.3)
    ax.grid(True, alpha=0.3)

plt.suptitle('Activation Functions')
plt.tight_layout()
plt.savefig('activation_functions.png', dpi=100)
print("Activation functions saved!")

# Manual Neural Network (without TF)
class SimpleNeuralNetwork:
    """Simple 2-layer NN from scratch"""
    
    def __init__(self, input_size, hidden_size, output_size, lr=0.01):
        # Initialize weights
        self.W1 = np.random.randn(input_size, hidden_size) * 0.01
        self.b1 = np.zeros((1, hidden_size))
        self.W2 = np.random.randn(hidden_size, output_size) * 0.01
        self.b2 = np.zeros((1, output_size))
        self.lr = lr
    
    def forward(self, X):
        self.z1 = X.dot(self.W1) + self.b1
        self.a1 = sigmoid(self.z1)
        self.z2 = self.a1.dot(self.W2) + self.b2
        self.a2 = sigmoid(self.z2)
        return self.a2
    
    def backward(self, X, y):
        m = X.shape[0]
        dz2 = self.a2 - y
        dW2 = self.a1.T.dot(dz2) / m
        db2 = dz2.mean(axis=0, keepdims=True)
        
        da1 = dz2.dot(self.W2.T)
        dz1 = da1 * self.a1 * (1 - self.a1)
        dW1 = X.T.dot(dz1) / m
        db1 = dz1.mean(axis=0, keepdims=True)
        
        self.W2 -= self.lr * dW2
        self.b2 -= self.lr * db2
        self.W1 -= self.lr * dW1
        self.b1 -= self.lr * db1
    
    def fit(self, X, y, epochs=1000):
        losses = []
        for epoch in range(epochs):
            y_pred = self.forward(X)
            loss = -np.mean(y * np.log(y_pred + 1e-8) + (1-y) * np.log(1-y_pred + 1e-8))
            losses.append(loss)
            self.backward(X, y)
            if epoch % 200 == 0:
                print(f"Epoch {epoch}: Loss = {loss:.6f}")
        return losses
    
    def predict(self, X):
        return (self.forward(X) > 0.5).astype(int)

# XOR problem
X_xor = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y_xor = np.array([[0], [1], [1], [0]])

nn = SimpleNeuralNetwork(2, 4, 1, lr=0.5)
losses = nn.fit(X_xor, y_xor, epochs=1000)
y_pred = nn.predict(X_xor)
print(f"\nXOR results: {y_pred.flatten()}")
print(f"Correct: {(y_pred.flatten() == y_xor.flatten()).all()}")
```

---

## 2. TensorFlow 2.x และ Keras API

### 2.1 Installation และ Setup

```python
# ติดตั้ง TensorFlow
# pip install tensorflow
# pip install tensorflow-gpu  # สำหรับ GPU

import tensorflow as tf
import numpy as np
print(f"TensorFlow version: {tf.__version__}")

# ตรวจสอบ GPU
gpus = tf.config.list_physical_devices('GPU')
print(f"GPUs available: {len(gpus)}")
if gpus:
    for gpu in gpus:
        print(f"  {gpu.name}")
else:
    print("Running on CPU")

# TensorFlow Basics
# Tensor
a = tf.constant([1, 2, 3, 4, 5], dtype=tf.float32)
b = tf.constant([[1, 2], [3, 4]], dtype=tf.float32)

print(f"\nTensor a: {a}")
print(f"Tensor b:\n{b}")
print(f"Shape: {b.shape}, Dtype: {b.dtype}")

# Operations
c = tf.matmul(b, b)
print(f"\nMatrix multiply:\n{c.numpy()}")

# Variable (trainable)
w = tf.Variable(tf.random.normal([3, 3]))
print(f"\nVariable w:\n{w.numpy()}")

# GradientTape
x = tf.Variable(3.0)
with tf.GradientTape() as tape:
    y = x ** 2 + 2*x + 1
dy_dx = tape.gradient(y, x)
print(f"\nd/dx(x² + 2x + 1) at x=3: {dy_dx.numpy()} (expected: {2*3 + 2})")
```

### 2.2 Keras Basics

```python
from tensorflow import keras
from tensorflow.keras import layers, models, optimizers, losses, metrics

# ตรวจสอบ Keras
print(f"Keras version: {keras.__version__}")

# Dataset
from tensorflow.keras.datasets import mnist
(X_train, y_train), (X_test, y_test) = mnist.load_data()

print(f"\nMNIST Dataset:")
print(f"  Train: X={X_train.shape}, y={y_train.shape}")
print(f"  Test: X={X_test.shape}, y={y_test.shape}")
print(f"  Classes: {np.unique(y_train)}")

# Preprocess
X_train_norm = X_train.astype('float32') / 255.0
X_test_norm = X_test.astype('float32') / 255.0

# Visualize MNIST
fig, axes = plt.subplots(2, 5, figsize=(12, 5))
for i, ax in enumerate(axes.flatten()):
    ax.imshow(X_train[i], cmap='gray')
    ax.set_title(f'Label: {y_train[i]}')
    ax.axis('off')
plt.suptitle('MNIST Samples')
plt.tight_layout()
plt.savefig('mnist_samples.png', dpi=100)
print("MNIST samples saved!")
```

---

## 3. Sequential Model

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

# โหลดข้อมูล
(X_train, y_train), (X_test, y_test) = keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test = X_test.reshape(-1, 784).astype('float32') / 255.0

# สร้าง Sequential Model
model = keras.Sequential([
    layers.Dense(128, activation='relu', input_shape=(784,), name='hidden1'),
    layers.Dropout(0.2, name='dropout1'),
    layers.Dense(64, activation='relu', name='hidden2'),
    layers.Dropout(0.2, name='dropout2'),
    layers.Dense(10, activation='softmax', name='output')
], name='mnist_classifier')

# Model Summary
model.summary()

# Compile
model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Train
history = model.fit(
    X_train, y_train,
    epochs=15,
    batch_size=128,
    validation_split=0.1,
    verbose=1
)

# Evaluate
test_loss, test_acc = model.evaluate(X_test, y_test, verbose=0)
print(f"\nTest Accuracy: {test_acc:.4f}")
print(f"Test Loss: {test_loss:.4f}")

# Plot training history
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

axes[0].plot(history.history['accuracy'], label='Train Accuracy', color='blue')
axes[0].plot(history.history['val_accuracy'], label='Val Accuracy', color='red')
axes[0].set_xlabel('Epoch')
axes[0].set_ylabel('Accuracy')
axes[0].set_title('Model Accuracy')
axes[0].legend()
axes[0].grid(True)

axes[1].plot(history.history['loss'], label='Train Loss', color='blue')
axes[1].plot(history.history['val_loss'], label='Val Loss', color='red')
axes[1].set_xlabel('Epoch')
axes[1].set_ylabel('Loss')
axes[1].set_title('Model Loss')
axes[1].legend()
axes[1].grid(True)

plt.tight_layout()
plt.savefig('training_history.png', dpi=100)
print("Training history saved!")
```

---

## 4. Functional API

Functional API มีความยืดหยุ่นมากกว่า Sequential เหมาะสำหรับ models ที่ซับซ้อน (multi-input, multi-output, shared layers)

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers, Model
import numpy as np

# Simple Functional API
inputs = keras.Input(shape=(784,), name='input')
x = layers.Dense(256, activation='relu', name='fc1')(inputs)
x = layers.BatchNormalization(name='bn1')(x)
x = layers.Dropout(0.3, name='drop1')(x)
x = layers.Dense(128, activation='relu', name='fc2')(x)
x = layers.BatchNormalization(name='bn2')(x)
x = layers.Dropout(0.3, name='drop2')(x)
outputs = layers.Dense(10, activation='softmax', name='output')(x)

model_func = Model(inputs=inputs, outputs=outputs, name='functional_mnist')
model_func.summary()

# Multi-Input Model
input_a = keras.Input(shape=(64,), name='input_a')
input_b = keras.Input(shape=(32,), name='input_b')

x_a = layers.Dense(32, activation='relu')(input_a)
x_b = layers.Dense(16, activation='relu')(input_b)

merged = layers.Concatenate()([x_a, x_b])
hidden = layers.Dense(32, activation='relu')(merged)
output = layers.Dense(1, activation='sigmoid')(hidden)

multi_input_model = Model(inputs=[input_a, input_b], outputs=output, 
                            name='multi_input_model')
multi_input_model.summary()

# Multi-Output Model
input_img = keras.Input(shape=(128,), name='image_input')
shared = layers.Dense(64, activation='relu')(input_img)

# Task 1: Classification
class_hidden = layers.Dense(32, activation='relu')(shared)
class_output = layers.Dense(5, activation='softmax', name='class_output')(class_hidden)

# Task 2: Regression
reg_hidden = layers.Dense(32, activation='relu')(shared)
reg_output = layers.Dense(1, name='reg_output')(reg_hidden)

multi_output_model = Model(inputs=input_img, outputs=[class_output, reg_output],
                             name='multi_output')
multi_output_model.compile(
    optimizer='adam',
    loss={'class_output': 'sparse_categorical_crossentropy', 
           'reg_output': 'mse'},
    loss_weights={'class_output': 1.0, 'reg_output': 0.1},
    metrics={'class_output': 'accuracy'}
)
print("\nMulti-Output Model compiled!")
```

---

## 5. Layer Types

### 5.1 Dense Layers

```python
import tensorflow as tf
from tensorflow.keras import layers

# Dense Layer
dense = layers.Dense(
    units=128,
    activation='relu',
    use_bias=True,
    kernel_initializer='glorot_uniform',
    bias_initializer='zeros',
    kernel_regularizer=tf.keras.regularizers.l2(0.01),
    name='dense_layer'
)

# Test with random input
x = tf.random.normal((32, 64))
output = dense(x)
print(f"Dense output shape: {output.shape}")
print(f"Weights shape: {dense.kernel.shape}")
print(f"Bias shape: {dense.bias.shape}")
```

### 5.2 Convolutional Layers (Conv2D)

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

# CNN Architecture for Image Classification
def build_cnn(input_shape=(32, 32, 3), n_classes=10):
    model = keras.Sequential([
        # Block 1
        layers.Conv2D(32, (3, 3), activation='relu', padding='same', input_shape=input_shape),
        layers.Conv2D(32, (3, 3), activation='relu', padding='same'),
        layers.MaxPooling2D((2, 2)),
        layers.BatchNormalization(),
        layers.Dropout(0.25),
        
        # Block 2
        layers.Conv2D(64, (3, 3), activation='relu', padding='same'),
        layers.Conv2D(64, (3, 3), activation='relu', padding='same'),
        layers.MaxPooling2D((2, 2)),
        layers.BatchNormalization(),
        layers.Dropout(0.25),
        
        # Block 3
        layers.Conv2D(128, (3, 3), activation='relu', padding='same'),
        layers.MaxPooling2D((2, 2)),
        layers.BatchNormalization(),
        layers.Dropout(0.4),
        
        # Classifier
        layers.Flatten(),
        layers.Dense(256, activation='relu'),
        layers.BatchNormalization(),
        layers.Dropout(0.5),
        layers.Dense(n_classes, activation='softmax')
    ], name='cnn_classifier')
    return model

cnn_model = build_cnn()
cnn_model.summary()

# CIFAR-10 training
(X_train, y_train), (X_test, y_test) = keras.datasets.cifar10.load_data()
X_train = X_train.astype('float32') / 255.0
X_test = X_test.astype('float32') / 255.0

class_names = ['airplane', 'automobile', 'bird', 'cat', 'deer',
               'dog', 'frog', 'horse', 'ship', 'truck']

print(f"\nCIFAR-10:")
print(f"  Train: {X_train.shape}")
print(f"  Test: {X_test.shape}")
print(f"  Classes: {class_names}")
```

### 5.3 LSTM & GRU Layers

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

# LSTM for sequence classification
def build_lstm(vocab_size=10000, max_len=200, embed_dim=128, n_classes=2):
    model = keras.Sequential([
        layers.Embedding(vocab_size, embed_dim, input_length=max_len, name='embedding'),
        layers.LSTM(128, return_sequences=True, name='lstm1'),
        layers.Dropout(0.3),
        layers.LSTM(64, name='lstm2'),
        layers.Dropout(0.3),
        layers.Dense(64, activation='relu'),
        layers.Dense(n_classes, activation='softmax' if n_classes > 2 else 'sigmoid')
    ], name='lstm_classifier')
    return model

lstm_model = build_lstm()
lstm_model.summary()

# GRU (faster than LSTM)
def build_gru(vocab_size=10000, max_len=200, embed_dim=128, n_classes=2):
    model = keras.Sequential([
        layers.Embedding(vocab_size, embed_dim, input_length=max_len),
        layers.Bidirectional(layers.GRU(64, return_sequences=True)),
        layers.Dropout(0.3),
        layers.GlobalMaxPooling1D(),
        layers.Dense(64, activation='relu'),
        layers.Dropout(0.3),
        layers.Dense(1, activation='sigmoid')
    ], name='bigru_classifier')
    return model

gru_model = build_gru()
gru_model.summary()
```

### 5.4 Embedding Layer

```python
import tensorflow as tf
from tensorflow.keras import layers
import numpy as np

# Embedding ทำ one-hot เป็น dense vector
vocab_size = 10000
embed_dim = 128

embedding = layers.Embedding(vocab_size, embed_dim, input_length=50)

# ทดสอบ
sample_seq = np.array([[1, 5, 23, 50, 100, 0, 0, 0, 0, 0]])  # padded sequence
embedded = embedding(sample_seq)
print(f"Input shape: {sample_seq.shape}")
print(f"Embedded shape: {embedded.shape}")  # (1, 10, 128)

# Pre-trained embeddings (Word2Vec, GloVe)
# สร้าง random word vectors สำหรับตัวอย่าง
embedding_matrix = np.random.normal(0, 1, (vocab_size, embed_dim)).astype('float32')

pretrained_embedding = layers.Embedding(
    vocab_size, embed_dim,
    weights=[embedding_matrix],
    trainable=False,  # Freeze pre-trained weights
    name='pretrained_embedding'
)
print("Pre-trained embedding created (non-trainable)")
```

---

## 6. Activation Functions ใน Keras

```python
import tensorflow as tf
from tensorflow.keras import layers, activations
import numpy as np

# Activation functions
x = tf.constant([-2.0, -1.0, 0.0, 1.0, 2.0])

print("Activation Function Values:")
print(f"  sigmoid:     {activations.sigmoid(x).numpy().round(4)}")
print(f"  relu:        {activations.relu(x).numpy().round(4)}")
print(f"  tanh:        {activations.tanh(x).numpy().round(4)}")
print(f"  elu:         {activations.elu(x).numpy().round(4)}")
print(f"  selu:        {activations.selu(x).numpy().round(4)}")
print(f"  gelu:        {activations.gelu(x).numpy().round(4)}")
print(f"  swish:       {activations.swish(x).numpy().round(4)}")
print(f"  softplus:    {activations.softplus(x).numpy().round(4)}")

# ใช้ใน layers
model_example = tf.keras.Sequential([
    layers.Dense(128, activation='relu'),        # ReLU: default for hidden layers
    layers.Dense(64, activation='elu'),          # ELU: avoids dying ReLU
    layers.Dense(32, activation='selu'),         # SELU: self-normalizing
    layers.Dense(1, activation='sigmoid')        # Sigmoid: binary output
])

# Custom activation
def custom_swish(x):
    return x * tf.sigmoid(x)

model_custom = tf.keras.Sequential([
    layers.Dense(64, activation=custom_swish),
    layers.Dense(1)
])

print("\nCustom activation function works!")
```

---

## 7. Loss Functions

```python
import tensorflow as tf
from tensorflow.keras import losses
import numpy as np

# Binary Classification Loss
y_true_bin = tf.constant([1, 0, 1, 1, 0], dtype=tf.float32)
y_pred_bin = tf.constant([0.9, 0.1, 0.8, 0.7, 0.3], dtype=tf.float32)

bce = losses.BinaryCrossentropy()
print(f"Binary Crossentropy: {bce(y_true_bin, y_pred_bin).numpy():.6f}")

# Multi-class Classification Loss
y_true_multi = tf.constant([0, 1, 2, 1])  # integer labels
y_pred_multi = tf.constant([[0.9, 0.05, 0.05], [0.1, 0.8, 0.1],
                               [0.05, 0.05, 0.9], [0.2, 0.7, 0.1]])

cce = losses.SparseCategoricalCrossentropy()
print(f"Sparse Categorical Crossentropy: {cce(y_true_multi, y_pred_multi).numpy():.6f}")

# Regression Loss
y_true_reg = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0])
y_pred_reg = tf.constant([1.1, 1.9, 3.2, 3.8, 5.1])

mse_loss = losses.MeanSquaredError()
mae_loss = losses.MeanAbsoluteError()
huber_loss = losses.Huber(delta=1.0)

print(f"\nMSE: {mse_loss(y_true_reg, y_pred_reg).numpy():.6f}")
print(f"MAE: {mae_loss(y_true_reg, y_pred_reg).numpy():.6f}")
print(f"Huber: {huber_loss(y_true_reg, y_pred_reg).numpy():.6f}")

# Custom Loss
def focal_loss(gamma=2.0, alpha=0.25):
    """Focal Loss สำหรับ imbalanced data"""
    def loss(y_true, y_pred):
        y_pred = tf.clip_by_value(y_pred, 1e-7, 1 - 1e-7)
        pt = tf.where(tf.equal(y_true, 1), y_pred, 1 - y_pred)
        focal_weight = alpha * (1 - pt) ** gamma
        loss_val = -focal_weight * tf.math.log(pt)
        return tf.reduce_mean(loss_val)
    return loss

custom_loss = focal_loss(gamma=2.0)
print(f"\nFocal Loss: {custom_loss(y_true_bin, y_pred_bin).numpy():.6f}")
```

---

## 8. Optimizers

```python
import tensorflow as tf
from tensorflow.keras import optimizers

# Optimizers comparison
lr = 0.001

sgd = optimizers.SGD(learning_rate=lr, momentum=0.9, nesterov=True)
adam = optimizers.Adam(learning_rate=lr, beta_1=0.9, beta_2=0.999, epsilon=1e-7)
rmsprop = optimizers.RMSprop(learning_rate=lr, rho=0.9)
adagrad = optimizers.Adagrad(learning_rate=lr)
adamw = optimizers.AdamW(learning_rate=lr, weight_decay=0.01)

print("Optimizers:")
for name, opt in [('SGD', sgd), ('Adam', adam), ('RMSprop', rmsprop), 
                   ('Adagrad', adagrad), ('AdamW', adamw)]:
    print(f"  {name}: {opt.get_config()}")

# Learning Rate Schedules
# 1. Exponential Decay
lr_exponential = tf.keras.optimizers.schedules.ExponentialDecay(
    initial_learning_rate=0.01,
    decay_steps=1000,
    decay_rate=0.9
)

# 2. Cosine Decay
lr_cosine = tf.keras.optimizers.schedules.CosineDecay(
    initial_learning_rate=0.01,
    decay_steps=10000
)

# 3. Warmup + Cosine
class WarmupCosineDecay(tf.keras.optimizers.schedules.LearningRateSchedule):
    def __init__(self, initial_lr, warmup_steps, total_steps):
        self.initial_lr = initial_lr
        self.warmup_steps = warmup_steps
        self.total_steps = total_steps
    
    def __call__(self, step):
        warmup = tf.cast(step / self.warmup_steps, tf.float32) * self.initial_lr
        cosine = self.initial_lr * 0.5 * (1 + tf.cos(
            np.pi * (step - self.warmup_steps) / (self.total_steps - self.warmup_steps)
        ))
        return tf.where(step < self.warmup_steps, warmup, cosine)

lr_schedule = WarmupCosineDecay(initial_lr=0.001, warmup_steps=100, total_steps=1000)
opt_with_schedule = optimizers.Adam(learning_rate=lr_schedule)

# Visualize LR schedule
steps = np.arange(0, 1000)
lrs = [lr_schedule(s).numpy() for s in steps]

import matplotlib.pyplot as plt
plt.figure(figsize=(10, 4))
plt.plot(steps, lrs, color='blue')
plt.xlabel('Step')
plt.ylabel('Learning Rate')
plt.title('Warmup + Cosine LR Schedule')
plt.grid(True)
plt.savefig('lr_schedule.png', dpi=100)
print("LR schedule saved!")
```

---

## 9. Callbacks

```python
import tensorflow as tf
from tensorflow.keras import callbacks
import os

# 1. ModelCheckpoint - บันทึก model ที่ดีที่สุด
checkpoint = callbacks.ModelCheckpoint(
    filepath='best_model.keras',
    monitor='val_accuracy',
    mode='max',
    save_best_only=True,
    verbose=1
)

# 2. EarlyStopping - หยุดเมื่อ model ไม่ improve
early_stopping = callbacks.EarlyStopping(
    monitor='val_loss',
    patience=10,
    min_delta=0.001,
    restore_best_weights=True,
    verbose=1
)

# 3. ReduceLROnPlateau - ลด LR เมื่อ stuck
reduce_lr = callbacks.ReduceLROnPlateau(
    monitor='val_loss',
    factor=0.5,
    patience=5,
    min_lr=1e-6,
    verbose=1
)

# 4. TensorBoard
tensorboard = callbacks.TensorBoard(
    log_dir='./logs',
    histogram_freq=1,
    write_graph=True,
    update_freq='epoch'
)

# 5. CSVLogger - บันทึก training log
csv_logger = callbacks.CSVLogger('training_log.csv', append=True)

# 6. Custom Callback
class TrainingProgress(callbacks.Callback):
    def __init__(self, print_every=5):
        super().__init__()
        self.print_every = print_every
        self.best_val_acc = 0
    
    def on_epoch_end(self, epoch, logs=None):
        logs = logs or {}
        val_acc = logs.get('val_accuracy', 0)
        
        if val_acc > self.best_val_acc:
            self.best_val_acc = val_acc
            print(f"\n  New best val_accuracy: {val_acc:.4f} at epoch {epoch+1}")
        
        if (epoch + 1) % self.print_every == 0:
            print(f"\n  Epoch {epoch+1}: "
                  f"loss={logs.get('loss', 0):.4f}, "
                  f"acc={logs.get('accuracy', 0):.4f}, "
                  f"val_acc={val_acc:.4f}")
    
    def on_train_end(self, logs=None):
        print(f"\nTraining completed! Best val_accuracy: {self.best_val_acc:.4f}")

# ใช้ callbacks ในการ training
(X_train, y_train), (X_test, y_test) = tf.keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test = X_test.reshape(-1, 784).astype('float32') / 255.0

model = tf.keras.Sequential([
    tf.keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    tf.keras.layers.Dropout(0.2),
    tf.keras.layers.Dense(64, activation='relu'),
    tf.keras.layers.Dropout(0.2),
    tf.keras.layers.Dense(10, activation='softmax')
])

model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])

history = model.fit(
    X_train, y_train,
    epochs=50,
    batch_size=128,
    validation_split=0.1,
    callbacks=[
        early_stopping,
        reduce_lr,
        checkpoint,
        csv_logger,
        TrainingProgress(print_every=5)
    ],
    verbose=0
)

print(f"\nTraining stopped at epoch: {len(history.history['loss'])}")
print(f"Final test accuracy: {model.evaluate(X_test, y_test, verbose=0)[1]:.4f}")
```

---

## 10. Data Augmentation

```python
import tensorflow as tf
from tensorflow.keras import layers
import numpy as np

# Image Augmentation
data_augmentation = tf.keras.Sequential([
    layers.RandomFlip("horizontal"),
    layers.RandomRotation(0.1),
    layers.RandomZoom(0.1),
    layers.RandomContrast(0.1),
    layers.RandomBrightness(0.1),
    layers.RandomTranslation(0.1, 0.1)
], name='data_augmentation')

# ใช้ใน model
def build_augmented_cnn(input_shape=(32, 32, 3), n_classes=10):
    inputs = tf.keras.Input(shape=input_shape)
    
    # Augmentation เฉพาะ training
    x = data_augmentation(inputs)
    x = layers.Rescaling(1./255)(x)
    
    # CNN
    x = layers.Conv2D(32, 3, activation='relu', padding='same')(x)
    x = layers.MaxPooling2D()(x)
    x = layers.Conv2D(64, 3, activation='relu', padding='same')(x)
    x = layers.MaxPooling2D()(x)
    x = layers.Conv2D(128, 3, activation='relu', padding='same')(x)
    x = layers.GlobalAveragePooling2D()(x)
    x = layers.Dense(128, activation='relu')(x)
    x = layers.Dropout(0.5)(x)
    outputs = layers.Dense(n_classes, activation='softmax')(x)
    
    return tf.keras.Model(inputs, outputs)

aug_model = build_augmented_cnn()
aug_model.summary()

# Visualize augmented images
(X_train, _), _ = tf.keras.datasets.cifar10.load_data()
sample_images = X_train[:5].astype('float32')

augmented = data_augmentation(sample_images, training=True)

import matplotlib.pyplot as plt
fig, axes = plt.subplots(2, 5, figsize=(15, 6))
for i in range(5):
    axes[0, i].imshow(sample_images[i].astype('uint8'))
    axes[0, i].set_title('Original')
    axes[0, i].axis('off')
    
    axes[1, i].imshow(np.clip(augmented[i].numpy(), 0, 255).astype('uint8'))
    axes[1, i].set_title('Augmented')
    axes[1, i].axis('off')

plt.suptitle('Data Augmentation')
plt.tight_layout()
plt.savefig('data_augmentation.png', dpi=100)
print("Augmentation comparison saved!")
```

---

## 11. Transfer Learning

```python
import tensorflow as tf
from tensorflow.keras import layers, Model
from tensorflow.keras.applications import MobileNetV2, VGG16, ResNet50

# Transfer Learning: ใช้ pre-trained model

def build_transfer_model(base_model_name='MobileNetV2', n_classes=10, 
                           input_shape=(224, 224, 3)):
    """สร้าง Transfer Learning model"""
    
    # โหลด base model (without top layers)
    if base_model_name == 'MobileNetV2':
        base_model = MobileNetV2(
            weights='imagenet',
            include_top=False,
            input_shape=input_shape
        )
    elif base_model_name == 'VGG16':
        base_model = VGG16(
            weights='imagenet',
            include_top=False,
            input_shape=input_shape
        )
    
    # Freeze base model
    base_model.trainable = False
    
    # Add custom head
    inputs = tf.keras.Input(shape=input_shape)
    x = base_model(inputs, training=False)
    x = layers.GlobalAveragePooling2D()(x)
    x = layers.Dense(256, activation='relu')(x)
    x = layers.Dropout(0.5)(x)
    outputs = layers.Dense(n_classes, activation='softmax')(x)
    
    model = Model(inputs, outputs, name=f'transfer_{base_model_name}')
    return model, base_model

# สร้าง Transfer Learning model
transfer_model, base = build_transfer_model('MobileNetV2', n_classes=10)
print("Transfer Learning Model:")
transfer_model.summary()

print(f"\nBase model trainable: {base.trainable}")
print(f"Trainable parameters: {sum([tf.size(v).numpy() for v in transfer_model.trainable_variables]):,}")
print(f"Non-trainable parameters: {sum([tf.size(v).numpy() for v in transfer_model.non_trainable_variables]):,}")

# Fine-tuning: Unfreeze top layers
def fine_tune_model(model, base_model, unfreeze_from=100):
    """Unfreeze top layers of base model for fine-tuning"""
    base_model.trainable = True
    
    # Freeze all layers except top ones
    for layer in base_model.layers[:unfreeze_from]:
        layer.trainable = False
    
    print(f"Fine-tuning: Training last {len(base_model.layers) - unfreeze_from} layers")
    
    # Recompile with lower lr
    model.compile(
        optimizer=tf.keras.optimizers.Adam(learning_rate=1e-5),
        loss='sparse_categorical_crossentropy',
        metrics=['accuracy']
    )
    return model

fine_tuned = fine_tune_model(transfer_model, base, unfreeze_from=100)
trainable_count = sum([tf.size(v).numpy() for v in fine_tuned.trainable_variables])
print(f"After fine-tuning trainable params: {trainable_count:,}")
```

### 11.1 Feature Extraction vs Fine-tuning

```python
"""
Transfer Learning Strategies:
1. Feature Extraction: Freeze base, train only head
2. Fine-tuning: Unfreeze some base layers, train with low lr
"""

import tensorflow as tf
from tensorflow.keras import layers
import numpy as np

# สำหรับ MNIST (ต้องปรับ image size)
(X_train, y_train), (X_test, y_test) = tf.keras.datasets.mnist.load_data()

# Resize และแปลงเป็น RGB สำหรับ pre-trained models
def preprocess_for_transfer(X, target_size=224):
    # Normalize
    X = X.astype('float32') / 255.0
    # Expand dims: (n, 28, 28) -> (n, 28, 28, 1)
    X = np.expand_dims(X, -1)
    # Repeat to 3 channels
    X = np.repeat(X, 3, axis=-1)
    return X

# สำหรับ demo ใช้ Simple CNN แทน
def build_simple_cnn_mnist():
    model = tf.keras.Sequential([
        # ใช้ convolutional base จาก VGG-style
        layers.Conv2D(32, 3, activation='relu', input_shape=(28, 28, 1), padding='same'),
        layers.MaxPooling2D(),
        layers.Conv2D(64, 3, activation='relu', padding='same'),
        layers.MaxPooling2D(),
        layers.Conv2D(64, 3, activation='relu', padding='same'),
        layers.Flatten(),
        layers.Dense(128, activation='relu'),
        layers.Dropout(0.5),
        layers.Dense(10, activation='softmax')
    ])
    return model

# Preprocess
X_train_cnn = X_train.reshape(-1, 28, 28, 1).astype('float32') / 255.0
X_test_cnn = X_test.reshape(-1, 28, 28, 1).astype('float32') / 255.0

model_cnn = build_simple_cnn_mnist()
model_cnn.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
model_cnn.summary()

# Train
history = model_cnn.fit(
    X_train_cnn, y_train,
    epochs=5,
    batch_size=128,
    validation_split=0.1,
    callbacks=[tf.keras.callbacks.EarlyStopping(patience=3, restore_best_weights=True)],
    verbose=1
)

test_acc = model_cnn.evaluate(X_test_cnn, y_test, verbose=0)[1]
print(f"\nCNN Test Accuracy: {test_acc:.4f}")
```

---

## 12. Model Saving and Loading

```python
import tensorflow as tf
import numpy as np
import os

# Train a simple model
(X_train, y_train), (X_test, y_test) = tf.keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test = X_test.reshape(-1, 784).astype('float32') / 255.0

model = tf.keras.Sequential([
    tf.keras.layers.Dense(64, activation='relu', input_shape=(784,)),
    tf.keras.layers.Dense(10, activation='softmax')
])
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
model.fit(X_train, y_train, epochs=3, batch_size=128, verbose=0)
print(f"Model accuracy: {model.evaluate(X_test, y_test, verbose=0)[1]:.4f}")

# Method 1: Save complete model (Keras format)
model.save('saved_model.keras')
model_loaded = tf.keras.models.load_model('saved_model.keras')
print(f"\nLoaded model (Keras format): {model_loaded.evaluate(X_test, y_test, verbose=0)[1]:.4f}")

# Method 2: SavedModel format (TensorFlow)
model.save('saved_model_tf', save_format='tf')
model_tf = tf.keras.models.load_model('saved_model_tf')
print(f"Loaded model (TF format): {model_tf.evaluate(X_test, y_test, verbose=0)[1]:.4f}")

# Method 3: Save weights only
model.save_weights('model_weights.weights.h5')
model_new = tf.keras.Sequential([
    tf.keras.layers.Dense(64, activation='relu', input_shape=(784,)),
    tf.keras.layers.Dense(10, activation='softmax')
])
model_new.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
model_new.load_weights('model_weights.weights.h5')
print(f"Loaded weights: {model_new.evaluate(X_test, y_test, verbose=0)[1]:.4f}")

# Method 4: TFLite (for mobile/edge)
converter = tf.lite.TFLiteConverter.from_keras_model(model)
tflite_model = converter.convert()
with open('model.tflite', 'wb') as f:
    f.write(tflite_model)
print(f"\nTFLite model size: {len(tflite_model)/1024:.1f} KB")

# Load TFLite
interpreter = tf.lite.Interpreter(model_path='model.tflite')
interpreter.allocate_tensors()
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

# Inference
sample = X_test[:1].astype('float32')
interpreter.set_tensor(input_details[0]['index'], sample)
interpreter.invoke()
tflite_pred = interpreter.get_tensor(output_details[0]['index'])
print(f"TFLite prediction: {np.argmax(tflite_pred)}, True: {y_test[0]}")
```

---

## 13. GPU Support

```python
import tensorflow as tf

# ตรวจสอบ GPU
print("TensorFlow version:", tf.__version__)
print("GPU available:", tf.test.is_gpu_available() if hasattr(tf.test, 'is_gpu_available') else bool(tf.config.list_physical_devices('GPU')))

gpus = tf.config.list_physical_devices('GPU')
print(f"Number of GPUs: {len(gpus)}")

if gpus:
    # Memory growth
    for gpu in gpus:
        tf.config.experimental.set_memory_growth(gpu, True)
        print(f"Memory growth enabled for: {gpu.name}")
    
    # Use specific GPU
    tf.config.set_visible_devices(gpus[0], 'GPU')
    
    # Multi-GPU strategy
    strategy = tf.distribute.MirroredStrategy()
    print(f"Number of devices: {strategy.num_replicas_in_sync}")
    
    # Build model with strategy
    with strategy.scope():
        model_multigpu = tf.keras.Sequential([
            tf.keras.layers.Dense(128, activation='relu', input_shape=(784,)),
            tf.keras.layers.Dense(10, activation='softmax')
        ])
        model_multigpu.compile(optimizer='adam',
                                loss='sparse_categorical_crossentropy',
                                metrics=['accuracy'])
else:
    print("No GPU found. Running on CPU.")
    print("Tips for GPU:")
    print("  1. Install tensorflow-gpu")
    print("  2. Install CUDA and cuDNN")
    print("  3. Use Google Colab for free GPU")

# Mixed Precision Training (faster on GPU with Tensor Cores)
tf.keras.mixed_precision.set_global_policy('mixed_float16')
print(f"\nGlobal policy: {tf.keras.mixed_precision.global_policy()}")

# Reset to float32
tf.keras.mixed_precision.set_global_policy('float32')
```

---

## 14. Image Classification CNN Project

```python
"""
โปรแกรมจริง: Image Classification ด้วย CNN
ใช้ CIFAR-10 Dataset
"""
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers, callbacks
import numpy as np
import matplotlib.pyplot as plt
import os

# 1. โหลดข้อมูล
(X_train, y_train), (X_test, y_test) = keras.datasets.cifar10.load_data()
class_names = ['airplane', 'automobile', 'bird', 'cat', 'deer',
               'dog', 'frog', 'horse', 'ship', 'truck']

y_train = y_train.flatten()
y_test = y_test.flatten()

print(f"CIFAR-10 Dataset:")
print(f"  Train: {X_train.shape}, {y_train.shape}")
print(f"  Test: {X_test.shape}, {y_test.shape}")
print(f"  Classes: {class_names}")

# 2. Preprocessing
X_train = X_train.astype('float32') / 255.0
X_test = X_test.astype('float32') / 255.0

# Data Statistics
mean = np.mean(X_train, axis=(0, 1, 2))
std = np.std(X_train, axis=(0, 1, 2))
print(f"\nData mean: {mean.round(4)}")
print(f"Data std: {std.round(4)}")

# 3. สร้าง CNN Model
def build_cifar_cnn():
    model = keras.Sequential([
        # Data Augmentation
        layers.RandomFlip("horizontal"),
        layers.RandomRotation(0.1),
        layers.RandomZoom(0.1),
        
        # Block 1
        layers.Conv2D(32, (3, 3), padding='same', input_shape=(32, 32, 3)),
        layers.BatchNormalization(),
        layers.Activation('relu'),
        layers.Conv2D(32, (3, 3), padding='same'),
        layers.BatchNormalization(),
        layers.Activation('relu'),
        layers.MaxPooling2D((2, 2)),
        layers.Dropout(0.2),
        
        # Block 2
        layers.Conv2D(64, (3, 3), padding='same'),
        layers.BatchNormalization(),
        layers.Activation('relu'),
        layers.Conv2D(64, (3, 3), padding='same'),
        layers.BatchNormalization(),
        layers.Activation('relu'),
        layers.MaxPooling2D((2, 2)),
        layers.Dropout(0.3),
        
        # Block 3
        layers.Conv2D(128, (3, 3), padding='same'),
        layers.BatchNormalization(),
        layers.Activation('relu'),
        layers.MaxPooling2D((2, 2)),
        layers.Dropout(0.4),
        
        # Classifier
        layers.Flatten(),
        layers.Dense(256, activation='relu'),
        layers.BatchNormalization(),
        layers.Dropout(0.5),
        layers.Dense(10, activation='softmax')
    ])
    return model

model = build_cifar_cnn()
model.summary()

# 4. Compile
model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=0.001),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# 5. Callbacks
model_callbacks = [
    callbacks.EarlyStopping(
        monitor='val_accuracy', patience=15,
        restore_best_weights=True, verbose=1
    ),
    callbacks.ReduceLROnPlateau(
        monitor='val_loss', factor=0.5, patience=7,
        min_lr=1e-6, verbose=1
    ),
    callbacks.ModelCheckpoint(
        'best_cifar10.keras', monitor='val_accuracy',
        save_best_only=True, verbose=0
    )
]

# 6. Train
print("\nTraining CNN...")
history = model.fit(
    X_train, y_train,
    epochs=50,
    batch_size=128,
    validation_split=0.1,
    callbacks=model_callbacks,
    verbose=1
)

# 7. Evaluate
test_loss, test_acc = model.evaluate(X_test, y_test, verbose=0)
print(f"\nFinal Test Results:")
print(f"  Accuracy: {test_acc:.4f}")
print(f"  Loss: {test_loss:.4f}")

# 8. Predictions
y_pred = np.argmax(model.predict(X_test[:10], verbose=0), axis=1)
print("\nSample Predictions:")
for i in range(10):
    print(f"  True: {class_names[y_test[i]]:12s}, Predicted: {class_names[y_pred[i]]}")

# 9. Training Curves
fig, axes = plt.subplots(1, 2, figsize=(12, 5))
axes[0].plot(history.history['accuracy'], label='Train')
axes[0].plot(history.history['val_accuracy'], label='Validation')
axes[0].set_title('CIFAR-10 Accuracy')
axes[0].legend()
axes[0].grid(True)

axes[1].plot(history.history['loss'], label='Train')
axes[1].plot(history.history['val_loss'], label='Validation')
axes[1].set_title('CIFAR-10 Loss')
axes[1].legend()
axes[1].grid(True)

plt.tight_layout()
plt.savefig('cifar10_training.png', dpi=100)
print("Training curves saved!")
```

---

## 15. Text Classification LSTM Project

```python
"""
โปรแกรมจริง: Sentiment Analysis ด้วย LSTM
ใช้ IMDB Movie Reviews
"""
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers, callbacks
import numpy as np

# 1. โหลด IMDB Dataset
max_features = 20000  # vocabulary size
max_len = 200  # max sequence length

(X_train, y_train), (X_test, y_test) = keras.datasets.imdb.load_data(
    num_words=max_features
)

print(f"IMDB Dataset:")
print(f"  Train: {len(X_train)}, Test: {len(X_test)}")
print(f"  Vocabulary: {max_features}")
print(f"  Sample review length: {len(X_train[0])} tokens")
print(f"  Positive reviews: {y_train.mean()*100:.0f}%")

# 2. Pad sequences
X_train_pad = keras.preprocessing.sequence.pad_sequences(X_train, maxlen=max_len)
X_test_pad = keras.preprocessing.sequence.pad_sequences(X_test, maxlen=max_len)

print(f"\nAfter padding: {X_train_pad.shape}")

# 3. Decode sample review
word_index = keras.datasets.imdb.get_word_index()
reverse_word_index = {v: k for k, v in word_index.items()}

def decode_review(encoded_review):
    return ' '.join([reverse_word_index.get(i - 3, '?') for i in encoded_review])

print(f"\nSample review: {decode_review(X_train[0])[:200]}...")
print(f"Sentiment: {'Positive' if y_train[0] == 1 else 'Negative'}")

# 4. Build LSTM Model
def build_lstm_model():
    model = keras.Sequential([
        layers.Embedding(max_features, 128, input_length=max_len),
        layers.Bidirectional(layers.LSTM(64, return_sequences=True)),
        layers.Dropout(0.3),
        layers.Bidirectional(layers.LSTM(32)),
        layers.Dense(64, activation='relu'),
        layers.Dropout(0.3),
        layers.Dense(1, activation='sigmoid')
    ], name='bilstm_sentiment')
    return model

# Alternative: 1D CNN (faster than LSTM)
def build_cnn1d_model():
    model = keras.Sequential([
        layers.Embedding(max_features, 128, input_length=max_len),
        layers.Conv1D(128, 5, activation='relu'),
        layers.GlobalMaxPooling1D(),
        layers.Dense(64, activation='relu'),
        layers.Dropout(0.3),
        layers.Dense(1, activation='sigmoid')
    ], name='cnn1d_sentiment')
    return model

# Build and compile
lstm_model = build_lstm_model()
lstm_model.summary()
lstm_model.compile(
    optimizer='adam',
    loss='binary_crossentropy',
    metrics=['accuracy', keras.metrics.AUC(name='auc')]
)

# 5. Train
print("\nTraining BiLSTM...")
history = lstm_model.fit(
    X_train_pad, y_train,
    epochs=10,
    batch_size=128,
    validation_split=0.1,
    callbacks=[
        callbacks.EarlyStopping(monitor='val_auc', patience=3, 
                                restore_best_weights=True, mode='max'),
    ],
    verbose=1
)

# 6. Evaluate
test_loss, test_acc, test_auc = lstm_model.evaluate(X_test_pad, y_test, verbose=0)
print(f"\nTest Results:")
print(f"  Accuracy: {test_acc:.4f}")
print(f"  AUC: {test_auc:.4f}")

# 7. Predictions
def predict_sentiment(model, texts, word_index, max_len=200):
    """ทำนาย sentiment ของข้อความใหม่"""
    # Simple tokenization
    sequences = []
    for text in texts:
        words = text.lower().split()
        seq = [word_index.get(w, 2) + 3 for w in words]
        sequences.append(seq)
    
    padded = keras.preprocessing.sequence.pad_sequences(sequences, maxlen=max_len)
    predictions = model.predict(padded, verbose=0)
    return predictions

sample_texts = [
    "This movie was absolutely fantastic! The acting was superb.",
    "Terrible film, complete waste of time. Very disappointed.",
    "Average movie, nothing special but not bad either."
]

preds = predict_sentiment(lstm_model, sample_texts, word_index)
print("\nSentiment Predictions:")
for text, pred in zip(sample_texts, preds):
    sentiment = "Positive" if pred[0] > 0.5 else "Negative"
    print(f"  Text: {text[:60]}...")
    print(f"  Sentiment: {sentiment} ({pred[0]:.4f})")
```

---

## 16. แบบฝึกหัด

### แบบฝึกหัดที่ 1: Build Custom Dense Neural Network

```python
# เฉลย
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

# Fashion MNIST
(X_train, y_train), (X_test, y_test) = keras.datasets.fashion_mnist.load_data()
class_names = ['T-shirt', 'Trouser', 'Pullover', 'Dress', 'Coat',
               'Sandal', 'Shirt', 'Sneaker', 'Bag', 'Ankle boot']

X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test = X_test.reshape(-1, 784).astype('float32') / 255.0

# Build model with different architectures
def create_model(hidden_layers, dropout_rate=0.3):
    model = keras.Sequential([keras.Input(shape=(784,))])
    for units in hidden_layers:
        model.add(layers.Dense(units, activation='relu'))
        model.add(layers.BatchNormalization())
        model.add(layers.Dropout(dropout_rate))
    model.add(layers.Dense(10, activation='softmax'))
    return model

architectures = {
    'Small': [64, 32],
    'Medium': [256, 128, 64],
    'Large': [512, 256, 128, 64]
}

print("Architecture Comparison - Fashion MNIST:")
for name, arch in architectures.items():
    model = create_model(arch)
    model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
    
    history = model.fit(
        X_train, y_train, epochs=20, batch_size=256,
        validation_split=0.1, verbose=0,
        callbacks=[keras.callbacks.EarlyStopping(patience=5, restore_best_weights=True)]
    )
    
    _, acc = model.evaluate(X_test, y_test, verbose=0)
    params = model.count_params()
    epochs = len(history.history['loss'])
    print(f"  {name:7s} {str(arch):25s}: Acc={acc:.4f}, Params={params:,}, Epochs={epochs}")
```

### แบบฝึกหัดที่ 2: CNN for MNIST

```python
# เฉลย
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers, callbacks
import numpy as np

(X_train, y_train), (X_test, y_test) = keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 28, 28, 1).astype('float32') / 255.0
X_test = X_test.reshape(-1, 28, 28, 1).astype('float32') / 255.0

# Build CNN
model = keras.Sequential([
    layers.Conv2D(32, 3, activation='relu', input_shape=(28, 28, 1), padding='same'),
    layers.BatchNormalization(),
    layers.MaxPooling2D(2),
    layers.Conv2D(64, 3, activation='relu', padding='same'),
    layers.BatchNormalization(),
    layers.MaxPooling2D(2),
    layers.Conv2D(64, 3, activation='relu', padding='same'),
    layers.Flatten(),
    layers.Dense(128, activation='relu'),
    layers.Dropout(0.4),
    layers.Dense(10, activation='softmax')
])

model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=0.001),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

model.summary()

history = model.fit(
    X_train, y_train, epochs=20, batch_size=128,
    validation_split=0.1, verbose=1,
    callbacks=[
        callbacks.EarlyStopping(patience=5, restore_best_weights=True),
        callbacks.ReduceLROnPlateau(patience=3, factor=0.5)
    ]
)

_, acc = model.evaluate(X_test, y_test, verbose=0)
print(f"\nCNN Test Accuracy: {acc:.4f}")
```

### แบบฝึกหัดที่ 3: Regularization Comparison

```python
# เฉลย
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers, regularizers
import numpy as np
import matplotlib.pyplot as plt

(X_train, y_train), (X_test, y_test) = keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test = X_test.reshape(-1, 784).astype('float32') / 255.0
X_train_small = X_train[:2000]
y_train_small = y_train[:2000]

def build_model(regularization=None, dropout=False):
    inputs = keras.Input(shape=(784,))
    x = inputs
    for units in [512, 256, 128]:
        if regularization == 'l2':
            x = layers.Dense(units, activation='relu', 
                             kernel_regularizer=regularizers.l2(0.001))(x)
        elif regularization == 'l1':
            x = layers.Dense(units, activation='relu',
                             kernel_regularizer=regularizers.l1(0.001))(x)
        else:
            x = layers.Dense(units, activation='relu')(x)
        if dropout:
            x = layers.Dropout(0.3)(x)
    outputs = layers.Dense(10, activation='softmax')(x)
    return keras.Model(inputs, outputs)

configs = {
    'No Regularization': (None, False),
    'L2 Regularization': ('l2', False),
    'Dropout': (None, True),
    'L2 + Dropout': ('l2', True)
}

results = {}
for name, (reg, drop) in configs.items():
    model = build_model(reg, drop)
    model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
    
    history = model.fit(X_train_small, y_train_small, epochs=50,
                         validation_data=(X_test, y_test), batch_size=64, verbose=0)
    
    train_acc = max(history.history['accuracy'])
    val_acc = max(history.history['val_accuracy'])
    overfit = train_acc - val_acc
    
    results[name] = history.history
    print(f"{name:25s}: Train={train_acc:.4f}, Val={val_acc:.4f}, Gap={overfit:.4f}")
```

### แบบฝึกหัดที่ 4: Learning Rate Finder

```python
# เฉลย
import tensorflow as tf
from tensorflow import keras
import numpy as np
import matplotlib.pyplot as plt

(X_train, y_train), _ = keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0

# LR Finder
class LRFinder(keras.callbacks.Callback):
    def __init__(self, min_lr=1e-7, max_lr=1, n_steps=100):
        super().__init__()
        self.min_lr = min_lr
        self.max_lr = max_lr
        self.n_steps = n_steps
        self.lrs = []
        self.losses = []
        self.step = 0
    
    def on_train_batch_begin(self, batch, logs=None):
        lr = self.min_lr * (self.max_lr / self.min_lr) ** (self.step / self.n_steps)
        tf.keras.backend.set_value(self.model.optimizer.lr, lr)
        self.lrs.append(lr)
        self.step += 1
    
    def on_train_batch_end(self, batch, logs=None):
        self.losses.append(logs['loss'])

model = keras.Sequential([
    keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    keras.layers.Dense(64, activation='relu'),
    keras.layers.Dense(10, activation='softmax')
])
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])

lr_finder = LRFinder(min_lr=1e-7, max_lr=10)
model.fit(X_train, y_train, epochs=1, batch_size=64, callbacks=[lr_finder], verbose=0)

# Smooth losses
losses = np.array(lr_finder.losses)
window = 10
smoothed = np.convolve(losses, np.ones(window)/window, mode='valid')
lrs_trimmed = lr_finder.lrs[window//2:len(losses)-window//2+1]

plt.figure(figsize=(10, 6))
plt.semilogx(lrs_trimmed[:len(smoothed)], smoothed)
plt.xlabel('Learning Rate (log scale)')
plt.ylabel('Loss')
plt.title('LR Finder')
plt.grid(True)
plt.savefig('lr_finder.png', dpi=100)
print("LR finder saved!")

# Find optimal lr (steepest negative gradient)
grad = np.gradient(smoothed)
best_lr_idx = np.argmin(grad)
best_lr = lrs_trimmed[best_lr_idx]
print(f"Suggested LR: {best_lr:.6f}")
```

### แบบฝึกหัดที่ 5: Custom Training Loop

```python
# เฉลย
import tensorflow as tf
from tensorflow import keras
import numpy as np

(X_train, y_train), (X_test, y_test) = keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test = X_test.reshape(-1, 784).astype('float32') / 255.0

# Custom training loop (ระดับ advanced)
model = keras.Sequential([
    keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    keras.layers.Dense(10)
])

optimizer = keras.optimizers.Adam()
loss_fn = keras.losses.SparseCategoricalCrossentropy(from_logits=True)
train_acc_metric = keras.metrics.SparseCategoricalAccuracy()

# Create datasets
train_dataset = tf.data.Dataset.from_tensor_slices((X_train, y_train))
train_dataset = train_dataset.shuffle(1000).batch(128).prefetch(tf.data.AUTOTUNE)

@tf.function  # Compile to graph for speed
def train_step(x, y):
    with tf.GradientTape() as tape:
        logits = model(x, training=True)
        loss = loss_fn(y, logits)
    grads = tape.gradient(loss, model.trainable_variables)
    optimizer.apply_gradients(zip(grads, model.trainable_variables))
    train_acc_metric.update_state(y, logits)
    return loss

# Training loop
for epoch in range(5):
    total_loss = 0
    n_batches = 0
    train_acc_metric.reset_states()
    
    for x_batch, y_batch in train_dataset:
        loss = train_step(x_batch, y_batch)
        total_loss += loss
        n_batches += 1
    
    train_acc = train_acc_metric.result()
    print(f"Epoch {epoch+1}: Loss={total_loss/n_batches:.4f}, Acc={train_acc:.4f}")

# Evaluate
test_logits = model(X_test, training=False)
test_preds = tf.argmax(test_logits, axis=1)
test_acc = tf.reduce_mean(tf.cast(tf.equal(test_preds, y_test), tf.float32))
print(f"\nTest Accuracy: {test_acc:.4f}")
```

### แบบฝึกหัดที่ 6: Autoencoder

```python
# เฉลย
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np
import matplotlib.pyplot as plt

(X_train, _), (X_test, _) = keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test = X_test.reshape(-1, 784).astype('float32') / 255.0

# Autoencoder: Encoder -> Bottleneck -> Decoder
encoding_dim = 32

# Encoder
inputs = keras.Input(shape=(784,))
x = layers.Dense(256, activation='relu')(inputs)
x = layers.Dense(128, activation='relu')(x)
encoded = layers.Dense(encoding_dim, activation='relu', name='bottleneck')(x)

# Decoder  
x = layers.Dense(128, activation='relu')(encoded)
x = layers.Dense(256, activation='relu')(x)
decoded = layers.Dense(784, activation='sigmoid')(x)

autoencoder = keras.Model(inputs, decoded, name='autoencoder')
encoder = keras.Model(inputs, encoded, name='encoder')

autoencoder.compile(optimizer='adam', loss='binary_crossentropy')
autoencoder.summary()

# Train
history = autoencoder.fit(
    X_train, X_train,  # input = target
    epochs=30, batch_size=256,
    validation_split=0.1,
    callbacks=[keras.callbacks.EarlyStopping(patience=5, restore_best_weights=True)],
    verbose=1
)

# Reconstruct
X_reconstructed = autoencoder.predict(X_test[:10], verbose=0)

fig, axes = plt.subplots(2, 10, figsize=(20, 4))
for i in range(10):
    axes[0, i].imshow(X_test[i].reshape(28, 28), cmap='gray')
    axes[0, i].axis('off')
    axes[1, i].imshow(X_reconstructed[i].reshape(28, 28), cmap='gray')
    axes[1, i].axis('off')

axes[0, 0].set_ylabel('Original', size=12)
axes[1, 0].set_ylabel('Reconstructed', size=12)
plt.suptitle(f'Autoencoder (bottleneck={encoding_dim})')
plt.savefig('autoencoder.png', dpi=100)

# Encoding space
X_encoded = encoder.predict(X_test, verbose=0)
print(f"\nOriginal dim: 784, Encoded dim: {encoding_dim}")
print(f"Compression ratio: {784/encoding_dim:.1f}x")
```

### แบบฝึกหัดที่ 7: Hyperparameter Tuning with Keras Tuner

```python
# เฉลย
try:
    import keras_tuner as kt
    from tensorflow import keras
    from tensorflow.keras import layers
    import numpy as np
    
    (X_train, y_train), (X_test, y_test) = keras.datasets.mnist.load_data()
    X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
    X_test = X_test.reshape(-1, 784).astype('float32') / 255.0
    
    def build_hypermodel(hp):
        model = keras.Sequential()
        model.add(keras.Input(shape=(784,)))
        
        for i in range(hp.Int('n_layers', 1, 4)):
            units = hp.Choice(f'units_{i}', [32, 64, 128, 256])
            model.add(layers.Dense(units, activation='relu'))
            model.add(layers.BatchNormalization())
            dropout = hp.Float(f'dropout_{i}', 0.1, 0.5, step=0.1)
            model.add(layers.Dropout(dropout))
        
        model.add(layers.Dense(10, activation='softmax'))
        
        lr = hp.Choice('lr', [1e-4, 5e-4, 1e-3, 5e-3])
        model.compile(
            optimizer=keras.optimizers.Adam(learning_rate=lr),
            loss='sparse_categorical_crossentropy',
            metrics=['accuracy']
        )
        return model
    
    # Tuner
    tuner = kt.RandomSearch(
        build_hypermodel,
        objective='val_accuracy',
        max_trials=10,
        executions_per_trial=1,
        directory='keras_tuner',
        project_name='mnist_tuning'
    )
    
    tuner.search(
        X_train, y_train,
        epochs=10, batch_size=128,
        validation_split=0.1,
        callbacks=[keras.callbacks.EarlyStopping(patience=3)],
        verbose=1
    )
    
    best_hps = tuner.get_best_hyperparameters(num_trials=1)[0]
    print(f"\nBest hyperparameters:")
    for key, val in best_hps.values.items():
        print(f"  {key}: {val}")
    
    best_model = tuner.hypermodel.build(best_hps)
    best_model.fit(X_train, y_train, epochs=20, validation_split=0.1, 
                    callbacks=[keras.callbacks.EarlyStopping(patience=5)], verbose=0)
    print(f"Best model test accuracy: {best_model.evaluate(X_test, y_test, verbose=0)[1]:.4f}")
    
except ImportError:
    print("keras-tuner not installed. Run: pip install keras-tuner")
    print("Using manual tuning instead...")
    
    # Manual comparison
    configs = [
        {'layers': [128, 64], 'dropout': 0.2, 'lr': 0.001},
        {'layers': [256, 128, 64], 'dropout': 0.3, 'lr': 0.001},
        {'layers': [512, 256], 'dropout': 0.4, 'lr': 0.0005}
    ]
    
    (X_train, y_train), (X_test, y_test) = keras.datasets.mnist.load_data()
    X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
    X_test = X_test.reshape(-1, 784).astype('float32') / 255.0
    
    for i, cfg in enumerate(configs):
        model = keras.Sequential([keras.Input(shape=(784,))])
        for units in cfg['layers']:
            model.add(layers.Dense(units, activation='relu'))
            model.add(layers.Dropout(cfg['dropout']))
        model.add(layers.Dense(10, activation='softmax'))
        
        model.compile(optimizer=keras.optimizers.Adam(cfg['lr']),
                      loss='sparse_categorical_crossentropy', metrics=['accuracy'])
        model.fit(X_train, y_train, epochs=15, batch_size=128, 
                   validation_split=0.1, verbose=0,
                   callbacks=[keras.callbacks.EarlyStopping(patience=3)])
        
        acc = model.evaluate(X_test, y_test, verbose=0)[1]
        print(f"Config {i+1} {cfg['layers']}: {acc:.4f}")
```

### แบบฝึกหัดที่ 8: Complete Deep Learning Project

```python
# เฉลย: Image Classification + Transfer Learning
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers, callbacks
import numpy as np

# CIFAR-10
(X_train, y_train), (X_test, y_test) = keras.datasets.cifar10.load_data()
y_train = y_train.flatten()
y_test = y_test.flatten()
X_train = X_train.astype('float32') / 255.0
X_test = X_test.astype('float32') / 255.0

class_names = ['airplane', 'automobile', 'bird', 'cat', 'deer',
               'dog', 'frog', 'horse', 'ship', 'truck']

# Augmentation
augmentation = keras.Sequential([
    layers.RandomFlip('horizontal'),
    layers.RandomRotation(0.1),
    layers.RandomZoom(0.1),
    layers.RandomContrast(0.1)
])

# Model with Residual connections
def build_resnet_small():
    inputs = keras.Input(shape=(32, 32, 3))
    
    # Augmentation + Preprocessing
    x = augmentation(inputs)
    x = layers.Rescaling(1.0)(x)  # already normalized
    
    # First Conv
    x = layers.Conv2D(64, 3, padding='same')(x)
    x = layers.BatchNormalization()(x)
    x = layers.Activation('relu')(x)
    
    # Residual Block
    def residual_block(x, filters):
        shortcut = x
        x = layers.Conv2D(filters, 3, padding='same')(x)
        x = layers.BatchNormalization()(x)
        x = layers.Activation('relu')(x)
        x = layers.Conv2D(filters, 3, padding='same')(x)
        x = layers.BatchNormalization()(x)
        if shortcut.shape[-1] != filters:
            shortcut = layers.Conv2D(filters, 1)(shortcut)
        x = layers.Add()([x, shortcut])
        x = layers.Activation('relu')(x)
        return x
    
    x = residual_block(x, 64)
    x = layers.MaxPooling2D(2)(x)
    x = layers.Dropout(0.2)(x)
    
    x = residual_block(x, 128)
    x = layers.MaxPooling2D(2)(x)
    x = layers.Dropout(0.3)(x)
    
    x = residual_block(x, 256)
    x = layers.GlobalAveragePooling2D()(x)
    x = layers.Dropout(0.4)(x)
    
    outputs = layers.Dense(10, activation='softmax')(x)
    
    return keras.Model(inputs, outputs, name='resnet_small')

model = build_resnet_small()
model.summary()

model.compile(
    optimizer=keras.optimizers.AdamW(learning_rate=1e-3, weight_decay=1e-4),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

history = model.fit(
    X_train, y_train,
    epochs=50, batch_size=128,
    validation_split=0.1,
    callbacks=[
        callbacks.EarlyStopping(monitor='val_accuracy', patience=15, restore_best_weights=True),
        callbacks.ReduceLROnPlateau(patience=7, factor=0.5, min_lr=1e-6)
    ],
    verbose=1
)

_, acc = model.evaluate(X_test, y_test, verbose=0)
print(f"\nFinal Test Accuracy: {acc:.4f}")
print(f"Trained for: {len(history.history['loss'])} epochs")

# Per-class accuracy
y_pred = np.argmax(model.predict(X_test, verbose=0), axis=1)
print("\nPer-class Accuracy:")
for i, cls in enumerate(class_names):
    mask = y_test == i
    cls_acc = (y_pred[mask] == y_test[mask]).mean()
    print(f"  {cls:12s}: {cls_acc:.4f}")
```

---

## สรุปบทที่ 80

| Topic | สิ่งที่เรียน |
|-------|------------|
| Neural Networks | Forward/Backward propagation, Activation functions |
| TensorFlow | Tensor operations, GradientTape |
| Sequential API | Linear model building |
| Functional API | Complex architectures, Multi-input/output |
| Layers | Dense, Conv2D, LSTM, GRU, Embedding, BatchNorm, Dropout |
| Loss Functions | BCE, CCE, MSE, Huber, Custom |
| Optimizers | SGD, Adam, AdamW, RMSprop + LR Schedules |
| Callbacks | EarlyStopping, ModelCheckpoint, ReduceLROnPlateau, TensorBoard |
| Transfer Learning | Feature extraction, Fine-tuning |
| Model Persistence | .keras, SavedModel, TFLite |
| Projects | Image CNN (CIFAR-10), Sentiment LSTM (IMDB) |

**Key Takeaways:**
1. ใช้ **EarlyStopping** เสมอเพื่อป้องกัน overfitting
2. **BatchNormalization** ช่วย stabilize training
3. **Dropout** = regularization สำหรับ dense layers
4. **Transfer Learning** ช่วยประหยัดเวลามากเมื่อมีข้อมูลน้อย
5. **Adam** optimizer ทำงานได้ดีในส่วนใหญ่
6. **Data Augmentation** ช่วยป้องกัน overfitting สำหรับ image data
7. ใช้ **ReduceLROnPlateau** เมื่อ validation loss ไม่ลดลง
8. **CNN** สำหรับ images, **LSTM/GRU** สำหรับ sequences
9. **Functional API** ยืดหยุ่นกว่า Sequential สำหรับ complex models
10. **TFLite** สำหรับ deploy บน mobile/edge devices

**แนวทางการศึกษาต่อ:**
- PyTorch (alternative framework)
- Hugging Face Transformers (NLP)
- Computer Vision with YOLO
- Reinforcement Learning with TF-Agents
- MLOps and Model Deployment
