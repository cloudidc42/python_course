# Part 83: Computer Vision with OpenCV & PIL

## สารบัญ
1. [Computer Vision Basics](#cv-basics)
2. [PIL/Pillow Library](#pillow)
3. [OpenCV Installation and Basics](#opencv-basics)
4. [Image Reading, Displaying, Saving](#image-io)
5. [Color Spaces](#color-spaces)
6. [Image Transformations](#transformations)
7. [Drawing](#drawing)
8. [Filters and Blurring](#filters)
9. [Edge Detection](#edge-detection)
10. [Object Detection - Template Matching](#template-matching)
11. [Face Detection - Haar Cascades](#face-detection)
12. [Video Processing](#video)
13. [ตัวอย่างโปรแกรมจริง](#real-examples)
14. [แบบฝึกหัด](#exercises)

---

## 1. Computer Vision Basics {#cv-basics}

### Computer Vision คืออะไร

Computer Vision คือสาขาหนึ่งของ AI ที่ให้คอมพิวเตอร์ "เห็น" และเข้าใจภาพ (images) และวิดีโอ (videos) ได้

### พื้นฐานภาพดิจิตอล

- **Pixel** = หน่วยย่อยของภาพ แต่ละ pixel มีค่า intensity
- **Resolution** = จำนวน pixels (width × height)
- **Channel** = มิติสีหนึ่ง (R, G, B)
- **Color Depth** = bits ต่อ pixel (8-bit = 0-255)

### Coordinate System

```
(0,0) → x →
  ↓
  y

In OpenCV: (x, y) = (col, row)
In NumPy: array[row, col] = array[y, x]
```

```python
# ตัวอย่างที่ 1: Understanding image as arrays
import numpy as np

# สร้าง image แบบ manual
# Grayscale: (height, width)
gray_image = np.zeros((100, 100), dtype=np.uint8)
gray_image[25:75, 25:75] = 128  # Gray rectangle

# Color: (height, width, channels)
color_image = np.zeros((100, 100, 3), dtype=np.uint8)
color_image[25:75, 25:75, 0] = 255  # Red rectangle (BGR in OpenCV)

print(f"Grayscale shape: {gray_image.shape}")
print(f"Color shape: {color_image.shape}")
print(f"Data type: {color_image.dtype}")
print(f"Value range: {color_image.min()} - {color_image.max()}")

# Access pixel values
pixel = color_image[50, 50]  # [row, col]
print(f"Pixel at (50,50): {pixel}")

# Channel separation
blue = color_image[:, :, 0]
green = color_image[:, :, 1]
red = color_image[:, :, 2]
print(f"Blue channel shape: {blue.shape}")
```

```python
# ตัวอย่างที่ 2: Basic image statistics
import numpy as np

# Create sample image
image = np.random.randint(0, 256, (480, 640, 3), dtype=np.uint8)

# Statistics
print("Image Statistics:")
print(f"  Shape: {image.shape} (H×W×C)")
print(f"  Dtype: {image.dtype}")
print(f"  Min: {image.min()}")
print(f"  Max: {image.max()}")
print(f"  Mean: {image.mean():.2f}")
print(f"  Std: {image.std():.2f}")

# Per-channel statistics
for i, channel in enumerate(['Blue', 'Green', 'Red']):
    ch = image[:, :, i]
    print(f"\n{channel} channel:")
    print(f"  Min: {ch.min()}, Max: {ch.max()}, Mean: {ch.mean():.2f}")

# Memory usage
print(f"\nMemory: {image.nbytes / 1024:.1f} KB")
print(f"Float32 version: {(image.astype(np.float32).nbytes / 1024):.1f} KB")
```

---

## 2. PIL/Pillow Library {#pillow}

```python
# ตัวอย่างที่ 3: PIL basics
# pip install Pillow

from PIL import Image, ImageDraw, ImageFont, ImageFilter, ImageEnhance
import numpy as np

# Create image from scratch
img = Image.new('RGB', (400, 300), color=(255, 255, 255))  # White background
print(f"Image mode: {img.mode}")
print(f"Image size: {img.size}")  # (width, height)

# Open image from file
# img = Image.open('photo.jpg')

# Convert between modes
img_gray = img.convert('L')      # Grayscale
img_rgba = img.convert('RGBA')   # Add alpha channel
img_hsv = img.convert('HSV')     # HSV (hue, sat, value)

print(f"RGB mode: {img.mode}")
print(f"Grayscale mode: {img_gray.mode}")
print(f"RGBA mode: {img_rgba.mode}")

# Convert to/from numpy
np_array = np.array(img)
print(f"\nNumPy array shape: {np_array.shape}")
print(f"NumPy array dtype: {np_array.dtype}")

# Back to PIL
img_from_np = Image.fromarray(np_array)
print(f"Back to PIL: {img_from_np.size}")
```

```python
# ตัวอย่างที่ 4: PIL image operations
from PIL import Image, ImageOps, ImageEnhance, ImageFilter
import numpy as np

# Create test image
width, height = 400, 300
img = Image.fromarray(
    np.random.randint(100, 200, (height, width, 3), dtype=np.uint8)
)

# Resize
resized = img.resize((200, 150), Image.LANCZOS)
print(f"Original: {img.size}, Resized: {resized.size}")

# Rotate
rotated = img.rotate(45, expand=True, fillcolor=(255, 255, 255))
print(f"Rotated: {rotated.size}")

# Flip
flipped_h = ImageOps.mirror(img)   # Horizontal flip
flipped_v = ImageOps.flip(img)     # Vertical flip

# Crop
box = (50, 50, 350, 250)  # (left, upper, right, lower)
cropped = img.crop(box)
print(f"Cropped: {cropped.size}")

# Thumbnail (maintain aspect ratio)
thumb = img.copy()
thumb.thumbnail((100, 100))
print(f"Thumbnail: {thumb.size}")

# Adjust enhancement
enhancer = ImageEnhance.Brightness(img)
bright = enhancer.enhance(1.5)  # 1.0 = original
contrast = ImageEnhance.Contrast(img).enhance(1.3)
sharpness = ImageEnhance.Sharpness(img).enhance(2.0)
color_enhanced = ImageEnhance.Color(img).enhance(1.5)

# Apply filters
blurred = img.filter(ImageFilter.BLUR)
sharpened = img.filter(ImageFilter.SHARPEN)
edges = img.filter(ImageFilter.FIND_EDGES)
emboss = img.filter(ImageFilter.EMBOSS)

# Save in different formats
# img.save('output.jpg', quality=95)
# img.save('output.png', optimize=True)
# img.save('output.webp', quality=85)

print("PIL operations completed!")
```

```python
# ตัวอย่างที่ 5: PIL Drawing
from PIL import Image, ImageDraw, ImageFont
import numpy as np

# Create blank canvas
img = Image.new('RGB', (500, 400), color=(245, 245, 245))
draw = ImageDraw.Draw(img)

# Draw rectangle
draw.rectangle([50, 50, 150, 150], 
               fill=(255, 0, 0), 
               outline=(0, 0, 0), 
               width=2)

# Draw ellipse/circle
draw.ellipse([200, 50, 350, 200], 
             fill=(0, 255, 0), 
             outline=(0, 0, 0))

# Draw polygon
points = [(400, 50), (450, 150), (350, 150)]
draw.polygon(points, fill=(0, 0, 255), outline=(0, 0, 0))

# Draw lines
draw.line([(50, 250), (450, 250)], fill=(0, 0, 0), width=3)
draw.line([(50, 300), (250, 350), (450, 300)], fill=(128, 0, 128), width=2)

# Draw text
text = "OpenCV & PIL Tutorial"
# For custom fonts: font = ImageFont.truetype("arial.ttf", 20)
draw.text((50, 370), text, fill=(0, 0, 0))

# Draw arc
draw.arc([50, 200, 200, 350], start=0, end=270, fill=(255, 165, 0), width=3)

# Draw chord
draw.chord([250, 200, 400, 350], start=0, end=180, fill=(0, 255, 255))

print("Drawing completed!")
# img.save('drawing.png')
```

```python
# ตัวอย่างที่ 6: PIL image composition and effects
from PIL import Image, ImageDraw, ImageFilter, ImageChops
import numpy as np

def create_gradient(width, height, start_color, end_color, direction='horizontal'):
    """Create gradient image"""
    img = Image.new('RGB', (width, height))
    draw = ImageDraw.Draw(img)
    
    r1, g1, b1 = start_color
    r2, g2, b2 = end_color
    
    if direction == 'horizontal':
        for x in range(width):
            t = x / width
            r = int(r1 + t * (r2 - r1))
            g = int(g1 + t * (g2 - g1))
            b = int(b1 + t * (b2 - b1))
            draw.line([(x, 0), (x, height)], fill=(r, g, b))
    else:
        for y in range(height):
            t = y / height
            r = int(r1 + t * (r2 - r1))
            g = int(g1 + t * (g2 - g1))
            b = int(b1 + t * (b2 - b1))
            draw.line([(0, y), (width, y)], fill=(r, g, b))
    
    return img

# Blend two images
def blend_images(img1, img2, alpha=0.5):
    """Blend two images"""
    if img1.size != img2.size:
        img2 = img2.resize(img1.size, Image.LANCZOS)
    return Image.blend(img1, img2, alpha)

# Add watermark
def add_watermark(image, text, position=(10, 10), opacity=0.5):
    """Add text watermark to image"""
    overlay = Image.new('RGBA', image.size, (0, 0, 0, 0))
    draw = ImageDraw.Draw(overlay)
    
    # Get text size (approximate)
    text_width = len(text) * 8
    text_height = 15
    
    # Draw semi-transparent text
    alpha = int(opacity * 255)
    draw.text(position, text, fill=(255, 255, 255, alpha))
    
    if image.mode != 'RGBA':
        image = image.convert('RGBA')
    
    return Image.alpha_composite(image, overlay)

# Test
grad = create_gradient(400, 300, (255, 0, 0), (0, 0, 255))
print(f"Gradient created: {grad.size}")

# Vignette effect
def vignette(image, intensity=0.7):
    """Add vignette effect"""
    width, height = image.size
    mask = Image.new('L', (width, height))
    draw = ImageDraw.Draw(mask)
    
    # Draw radial gradient
    cx, cy = width // 2, height // 2
    max_dist = ((cx**2 + cy**2) ** 0.5)
    
    for y in range(height):
        for x in range(width):
            dist = ((x - cx)**2 + (y - cy)**2) ** 0.5
            normalized = dist / max_dist
            value = int(255 * (1 - intensity * normalized**2))
            draw.point((x, y), fill=max(0, value))
    
    result = image.copy()
    if result.mode == 'RGB':
        r, g, b = result.split()
        r = r.point(lambda v: v)
        result = Image.merge('RGB', (r, g, b))
    
    return result

print("PIL effects functions defined!")
```

---

## 3. OpenCV Installation and Basics {#opencv-basics}

```python
# ตัวอย่างที่ 7: OpenCV installation and setup
# pip install opencv-python
# pip install opencv-contrib-python  # Extra modules

import cv2
import numpy as np

print(f"OpenCV version: {cv2.__version__}")

# Key differences: OpenCV uses BGR (not RGB)!
# Blue, Green, Red order - not Red, Green, Blue

# Create image
img = np.zeros((400, 600, 3), dtype=np.uint8)  # Black image

# OpenCV color constants
RED_BGR = (0, 0, 255)     # Note: BGR not RGB!
GREEN_BGR = (0, 255, 0)
BLUE_BGR = (255, 0, 0)
WHITE_BGR = (255, 255, 255)
BLACK_BGR = (0, 0, 0)
YELLOW_BGR = (0, 255, 255)
CYAN_BGR = (255, 255, 0)

# Draw rectangle: img, top-left, bottom-right, color, thickness
cv2.rectangle(img, (50, 50), (200, 200), RED_BGR, 3)

# Draw circle: img, center, radius, color, thickness (-1 = filled)
cv2.circle(img, (350, 150), 80, GREEN_BGR, -1)

# Draw line: img, pt1, pt2, color, thickness
cv2.line(img, (0, 300), (600, 300), BLUE_BGR, 2)

# Add text: img, text, origin, font, scale, color, thickness
cv2.putText(img, "OpenCV Tutorial", (50, 350),
            cv2.FONT_HERSHEY_SIMPLEX, 1.0, WHITE_BGR, 2)

print("OpenCV drawing completed!")
print(f"Image shape: {img.shape}")

# Display (requires GUI environment)
# cv2.imshow('Result', img)
# cv2.waitKey(0)
# cv2.destroyAllWindows()

# Save image
# cv2.imwrite('result.jpg', img)
```

---

## 4. Image Reading, Displaying, Saving {#image-io}

```python
# ตัวอย่างที่ 8: Image I/O operations
import cv2
import numpy as np
from pathlib import Path

class ImageIO:
    """Utilities for image reading/writing"""
    
    @staticmethod
    def read(filepath, mode='color'):
        """Read image file"""
        modes = {
            'color': cv2.IMREAD_COLOR,
            'grayscale': cv2.IMREAD_GRAYSCALE,
            'unchanged': cv2.IMREAD_UNCHANGED  # Includes alpha
        }
        flag = modes.get(mode, cv2.IMREAD_COLOR)
        img = cv2.imread(str(filepath), flag)
        
        if img is None:
            raise FileNotFoundError(f"Cannot read: {filepath}")
        
        return img
    
    @staticmethod
    def write(filepath, img, quality=95):
        """Write image to file"""
        ext = Path(filepath).suffix.lower()
        
        params = []
        if ext == '.jpg':
            params = [cv2.IMWRITE_JPEG_QUALITY, quality]
        elif ext == '.png':
            params = [cv2.IMWRITE_PNG_COMPRESSION, 9 - int(quality/10)]
        
        success = cv2.imwrite(str(filepath), img, params)
        return success
    
    @staticmethod
    def from_numpy(array, bgr=True):
        """Convert numpy array to OpenCV image"""
        if not bgr and len(array.shape) == 3:
            array = cv2.cvtColor(array, cv2.COLOR_RGB2BGR)
        return array.copy()
    
    @staticmethod
    def to_pil(img_bgr):
        """Convert OpenCV BGR to PIL Image"""
        from PIL import Image
        if len(img_bgr.shape) == 3:
            img_rgb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB)
        else:
            img_rgb = img_bgr
        return Image.fromarray(img_rgb)
    
    @staticmethod
    def from_pil(pil_img):
        """Convert PIL Image to OpenCV BGR"""
        np_array = np.array(pil_img)
        if len(np_array.shape) == 3:
            return cv2.cvtColor(np_array, cv2.COLOR_RGB2BGR)
        return np_array
    
    @staticmethod
    def display_info(img, name="Image"):
        """Display image information"""
        print(f"{name}:")
        print(f"  Shape: {img.shape}")
        print(f"  Dtype: {img.dtype}")
        print(f"  Size: {img.shape[1]}×{img.shape[0]} pixels")
        if len(img.shape) == 3:
            print(f"  Channels: {img.shape[2]}")
        print(f"  Min/Max: {img.min()}/{img.max()}")
        print(f"  Memory: {img.nbytes / 1024:.1f} KB")

# Create test image
test_img = np.random.randint(0, 256, (480, 640, 3), dtype=np.uint8)
io = ImageIO()
io.display_info(test_img, "Test Image")

# Test conversion
from PIL import Image
pil_img = io.to_pil(test_img)
back_to_cv = io.from_pil(pil_img)
print(f"\nRoundtrip conversion shape: {back_to_cv.shape}")
```

---

## 5. Color Spaces {#color-spaces}

```python
# ตัวอย่างที่ 9: Color space conversions
import cv2
import numpy as np

# Create test image
img_bgr = np.zeros((300, 400, 3), dtype=np.uint8)
# Draw colorful shapes
cv2.rectangle(img_bgr, (0, 0), (133, 300), (0, 0, 255), -1)    # Red area
cv2.rectangle(img_bgr, (133, 0), (267, 300), (0, 255, 0), -1)  # Green area
cv2.rectangle(img_bgr, (267, 0), (400, 300), (255, 0, 0), -1)  # Blue area

# BGR → RGB
img_rgb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB)

# BGR → Grayscale
img_gray = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2GRAY)

# BGR → HSV (Hue, Saturation, Value)
img_hsv = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2HSV)

# BGR → LAB
img_lab = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2LAB)

# BGR → YCrCb
img_ycrcb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2YCrCb)

print("Color space conversions:")
print(f"BGR shape: {img_bgr.shape}")
print(f"Grayscale shape: {img_gray.shape}")
print(f"HSV shape: {img_hsv.shape}")
print(f"LAB shape: {img_lab.shape}")

# HSV color range for common colors
print("\nHSV ranges for common colors:")
hsv_ranges = {
    'Red (lower)': ([0, 120, 70], [10, 255, 255]),
    'Red (upper)': ([170, 120, 70], [180, 255, 255]),
    'Green': ([40, 40, 40], [80, 255, 255]),
    'Blue': ([100, 150, 0], [140, 255, 255]),
    'Yellow': ([20, 100, 100], [30, 255, 255]),
    'Orange': ([10, 100, 100], [25, 255, 255]),
}

for color, (lower, upper) in hsv_ranges.items():
    print(f"  {color}: H{lower[0]}-{upper[0]}, S{lower[1]}-{upper[1]}, V{lower[2]}-{upper[2]}")
```

```python
# ตัวอย่างที่ 10: Color detection using HSV
import cv2
import numpy as np

def detect_color(img_bgr, color_name):
    """
    Detect specific color in image using HSV ranges
    Returns: mask and color percentage
    """
    img_hsv = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2HSV)
    
    color_ranges = {
        'red': [
            ([0, 120, 70], [10, 255, 255]),    # Lower red
            ([170, 120, 70], [180, 255, 255])   # Upper red
        ],
        'green': [([40, 40, 40], [80, 255, 255])],
        'blue': [([100, 150, 0], [140, 255, 255])],
        'yellow': [([20, 100, 100], [30, 255, 255])],
        'orange': [([10, 100, 100], [25, 255, 255])],
        'purple': [([130, 100, 100], [160, 255, 255])],
        'white': [([0, 0, 200], [180, 30, 255])],
        'black': [([0, 0, 0], [180, 255, 50])],
    }
    
    if color_name not in color_ranges:
        return None, 0
    
    # Combine masks for multi-range colors
    masks = []
    for lower, upper in color_ranges[color_name]:
        lower = np.array(lower, dtype=np.uint8)
        upper = np.array(upper, dtype=np.uint8)
        mask = cv2.inRange(img_hsv, lower, upper)
        masks.append(mask)
    
    final_mask = masks[0]
    for mask in masks[1:]:
        final_mask = cv2.bitwise_or(final_mask, mask)
    
    # Apply morphological operations to clean mask
    kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
    final_mask = cv2.morphologyEx(final_mask, cv2.MORPH_OPEN, kernel)
    final_mask = cv2.morphologyEx(final_mask, cv2.MORPH_DILATE, kernel)
    
    # Calculate color percentage
    percentage = (final_mask > 0).sum() / final_mask.size * 100
    
    # Apply mask to get colored region
    result = cv2.bitwise_and(img_bgr, img_bgr, mask=final_mask)
    
    return result, percentage, final_mask

# Test
test_img = np.zeros((300, 400, 3), dtype=np.uint8)
cv2.circle(test_img, (200, 150), 100, (0, 0, 255), -1)  # Red circle in BGR

result, percentage, mask = detect_color(test_img, 'red')
print(f"Red color coverage: {percentage:.1f}%")
```

```python
# ตัวอย่างที่ 11: Color histogram
import cv2
import numpy as np

def compute_color_histogram(img_bgr, bins=32):
    """Compute and analyze color histogram"""
    
    # BGR histogram
    histograms = {}
    channels = ['Blue', 'Green', 'Red']
    
    for i, ch_name in enumerate(channels):
        hist = cv2.calcHist(
            [img_bgr],      # images
            [i],            # channels
            None,           # mask
            [bins],         # histSize
            [0, 256]        # ranges
        )
        histograms[ch_name] = hist.flatten()
    
    # Overall stats
    stats = {}
    for ch_name, hist in histograms.items():
        total = hist.sum()
        # Find peak (mode)
        peak_bin = hist.argmax()
        peak_value = peak_bin * (256 / bins)
        stats[ch_name] = {
            'peak': peak_value,
            'distribution': 'uniform' if hist.std() < 100 else 'non-uniform'
        }
    
    # HSV histogram for hue distribution
    img_hsv = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2HSV)
    hue_hist = cv2.calcHist([img_hsv], [0], None, [180], [0, 180]).flatten()
    sat_hist = cv2.calcHist([img_hsv], [1], None, [256], [0, 256]).flatten()
    
    return histograms, hue_hist, stats

# Test image (blue sky simulation)
img = np.zeros((300, 400, 3), dtype=np.uint8)
img[:150] = [200, 100, 50]   # Sky (BGR): high blue
img[150:] = [50, 100, 50]    # Ground: mostly green

histograms, hue_hist, stats = compute_color_histogram(img)
print("Color Histogram Analysis:")
for ch_name, stat in stats.items():
    print(f"  {ch_name}: peak at {stat['peak']:.0f}, {stat['distribution']}")
```

---

## 6. Image Transformations {#transformations}

```python
# ตัวอย่างที่ 12: Geometric transformations
import cv2
import numpy as np

def resize_with_aspect(img, width=None, height=None, interpolation=cv2.INTER_LINEAR):
    """Resize maintaining aspect ratio"""
    h, w = img.shape[:2]
    
    if width is None and height is None:
        return img
    
    if width is None:
        scale = height / h
        new_w = int(w * scale)
        new_h = height
    elif height is None:
        scale = width / w
        new_w = width
        new_h = int(h * scale)
    else:
        # Fit within both dimensions
        scale_w = width / w
        scale_h = height / h
        scale = min(scale_w, scale_h)
        new_w = int(w * scale)
        new_h = int(h * scale)
    
    return cv2.resize(img, (new_w, new_h), interpolation=interpolation)

def rotate_image(img, angle, center=None, scale=1.0, keep_size=True):
    """Rotate image around center"""
    h, w = img.shape[:2]
    if center is None:
        center = (w // 2, h // 2)
    
    M = cv2.getRotationMatrix2D(center, angle, scale)
    
    if keep_size:
        return cv2.warpAffine(img, M, (w, h))
    else:
        # Calculate new size
        cos = abs(M[0, 0])
        sin = abs(M[0, 1])
        new_w = int(h * sin + w * cos)
        new_h = int(h * cos + w * sin)
        
        # Adjust translation
        M[0, 2] += (new_w - w) / 2
        M[1, 2] += (new_h - h) / 2
        
        return cv2.warpAffine(img, M, (new_w, new_h), borderMode=cv2.BORDER_REPLICATE)

def perspective_transform(img, src_points, dst_points):
    """Apply perspective transformation"""
    src = np.float32(src_points)
    dst = np.float32(dst_points)
    M = cv2.getPerspectiveTransform(src, dst)
    h, w = img.shape[:2]
    warped = cv2.warpPerspective(img, M, (w, h))
    return warped, M

def affine_transform(img, src_points, dst_points):
    """Apply affine transformation"""
    src = np.float32(src_points)
    dst = np.float32(dst_points)
    M = cv2.getAffineTransform(src, dst)
    h, w = img.shape[:2]
    return cv2.warpAffine(img, M, (w, h))

# Create test image
img = np.zeros((300, 400, 3), dtype=np.uint8)
cv2.rectangle(img, (100, 75), (300, 225), (0, 255, 0), 2)
cv2.circle(img, (200, 150), 50, (255, 0, 0), -1)

# Test transformations
resized = resize_with_aspect(img, width=200)
print(f"Original: {img.shape[1]}×{img.shape[0]}")
print(f"Resized: {resized.shape[1]}×{resized.shape[0]}")

rotated_45 = rotate_image(img, 45)
rotated_keep = rotate_image(img, 45, keep_size=False)
print(f"Rotated (keep size): {rotated_45.shape[1]}×{rotated_45.shape[0]}")
print(f"Rotated (expand): {rotated_keep.shape[1]}×{rotated_keep.shape[0]}")

# Flip operations
flipped_h = cv2.flip(img, 1)    # Horizontal flip
flipped_v = cv2.flip(img, 0)    # Vertical flip
flipped_b = cv2.flip(img, -1)   # Both

print("\nTransformations created!")
```

```python
# ตัวอย่างที่ 13: Advanced transformations
import cv2
import numpy as np
import math

def apply_barrel_distortion(img, k1=-0.5, k2=0.3):
    """Apply barrel/pincushion distortion"""
    h, w = img.shape[:2]
    cx, cy = w / 2, h / 2
    
    # Create map
    map_x = np.zeros((h, w), dtype=np.float32)
    map_y = np.zeros((h, w), dtype=np.float32)
    
    for y in range(h):
        for x in range(w):
            # Normalize coordinates
            xn = (x - cx) / cx
            yn = (y - cy) / cy
            
            r2 = xn**2 + yn**2
            radial = 1 + k1 * r2 + k2 * r2**2
            
            # Distorted coordinates
            map_x[y, x] = cx + cx * xn * radial
            map_y[y, x] = cy + cy * yn * radial
    
    return cv2.remap(img, map_x, map_y, cv2.INTER_LINEAR)

def create_transformation_pipeline(img, operations):
    """Apply multiple transformations"""
    result = img.copy()
    
    for op, params in operations:
        if op == 'resize':
            result = cv2.resize(result, params)
        elif op == 'rotate':
            h, w = result.shape[:2]
            M = cv2.getRotationMatrix2D((w//2, h//2), params, 1.0)
            result = cv2.warpAffine(result, M, (w, h))
        elif op == 'flip':
            result = cv2.flip(result, params)
        elif op == 'crop':
            x, y, w, h = params
            result = result[y:y+h, x:x+w]
    
    return result

# Test
img = np.zeros((400, 600, 3), dtype=np.uint8)
cv2.circle(img, (300, 200), 100, (0, 255, 0), -1)
cv2.rectangle(img, (50, 50), (200, 350), (0, 0, 255), 2)

operations = [
    ('rotate', 15),
    ('flip', 1),
    ('resize', (300, 200)),
]

result = create_transformation_pipeline(img, operations)
print(f"After pipeline: {result.shape}")
```

---

## 7. Drawing {#drawing}

```python
# ตัวอย่างที่ 14: Comprehensive drawing with OpenCV
import cv2
import numpy as np
import math

class Drawer:
    """OpenCV drawing utilities"""
    
    def __init__(self, width=600, height=400, bg_color=(240, 240, 240)):
        self.canvas = np.ones((height, width, 3), dtype=np.uint8) * bg_color[0]
        self.canvas[:, :, 1] = bg_color[1]
        self.canvas[:, :, 2] = bg_color[2]
    
    def rectangle(self, pt1, pt2, color, thickness=1, filled=False):
        t = -1 if filled else thickness
        cv2.rectangle(self.canvas, pt1, pt2, color, t)
        return self
    
    def circle(self, center, radius, color, thickness=1, filled=False):
        t = -1 if filled else thickness
        cv2.circle(self.canvas, center, radius, color, t)
        return self
    
    def ellipse(self, center, axes, angle, color, thickness=1):
        cv2.ellipse(self.canvas, center, axes, angle, 0, 360, color, thickness)
        return self
    
    def line(self, pt1, pt2, color, thickness=1, arrow=False):
        if arrow:
            cv2.arrowedLine(self.canvas, pt1, pt2, color, thickness)
        else:
            cv2.line(self.canvas, pt1, pt2, color, thickness)
        return self
    
    def polygon(self, points, color, thickness=1, filled=False):
        pts = np.array(points, dtype=np.int32).reshape((-1, 1, 2))
        if filled:
            cv2.fillPoly(self.canvas, [pts], color)
        else:
            cv2.polylines(self.canvas, [pts], True, color, thickness)
        return self
    
    def text(self, text, position, color, font_scale=0.8, thickness=1):
        font = cv2.FONT_HERSHEY_SIMPLEX
        cv2.putText(self.canvas, text, position, font, font_scale, color, thickness)
        return self
    
    def cross(self, center, size=10, color=(0, 0, 0), thickness=2):
        x, y = center
        cv2.line(self.canvas, (x-size, y), (x+size, y), color, thickness)
        cv2.line(self.canvas, (x, y-size), (x, y+size), color, thickness)
        return self
    
    def dashed_line(self, pt1, pt2, color, thickness=1, dash_length=10):
        """Draw dashed line"""
        x1, y1 = pt1
        x2, y2 = pt2
        dx, dy = x2 - x1, y2 - y1
        dist = math.sqrt(dx**2 + dy**2)
        
        if dist == 0:
            return self
        
        num_dashes = int(dist / (2 * dash_length))
        for i in range(num_dashes):
            t1 = 2 * i * dash_length / dist
            t2 = (2 * i + 1) * dash_length / dist
            t2 = min(t2, 1)
            
            p1 = (int(x1 + t1 * dx), int(y1 + t1 * dy))
            p2 = (int(x1 + t2 * dx), int(y1 + t2 * dy))
            cv2.line(self.canvas, p1, p2, color, thickness)
        
        return self
    
    def annotation(self, text, point, color=(0, 0, 0)):
        """Add annotation with line"""
        x, y = point
        end_x, end_y = x + 50, y - 30
        
        cv2.arrowedLine(self.canvas, point, (end_x, end_y), color, 1)
        cv2.putText(self.canvas, text, (end_x + 5, end_y),
                   cv2.FONT_HERSHEY_SIMPLEX, 0.5, color, 1)
        return self
    
    def show(self):
        # cv2.imshow('Canvas', self.canvas)
        # cv2.waitKey(0)
        return self.canvas

# Test drawing
drawer = Drawer(600, 400)

# Draw various shapes
drawer.rectangle((50, 50), (200, 150), (0, 0, 255), thickness=2)
drawer.circle((350, 100), 60, (0, 255, 0), filled=True)
drawer.ellipse((500, 100), (70, 40), 30, (255, 0, 0), thickness=2)
drawer.polygon([(100, 300), (150, 200), (200, 300)], (255, 165, 0), filled=True)
drawer.text("OpenCV Drawing", (50, 380), (0, 0, 0))
drawer.dashed_line((0, 200), (600, 200), (128, 128, 128), dash_length=15)

canvas = drawer.show()
print(f"Canvas created: {canvas.shape}")
```

---

## 8. Filters and Blurring {#filters}

```python
# ตัวอย่างที่ 15: Image filters
import cv2
import numpy as np

def apply_filters_comparison(img):
    """Apply and compare different blur filters"""
    results = {}
    
    # Average blur
    results['Average (3x3)'] = cv2.blur(img, (3, 3))
    results['Average (15x15)'] = cv2.blur(img, (15, 15))
    
    # Gaussian blur (best for most cases)
    results['Gaussian (3x3)'] = cv2.GaussianBlur(img, (3, 3), 0)
    results['Gaussian (15x15)'] = cv2.GaussianBlur(img, (15, 15), 0)
    
    # Median blur (best for salt & pepper noise)
    results['Median (3)'] = cv2.medianBlur(img, 3)
    results['Median (15)'] = cv2.medianBlur(img, 15)
    
    # Bilateral filter (preserves edges)
    results['Bilateral'] = cv2.bilateralFilter(img, 9, 75, 75)
    
    return results

def add_noise(img, noise_type='gaussian', intensity=25):
    """Add noise to image"""
    noisy = img.copy().astype(np.float64)
    
    if noise_type == 'gaussian':
        noise = np.random.normal(0, intensity, img.shape)
        noisy = noisy + noise
    elif noise_type == 'salt_pepper':
        salt_mask = np.random.random(img.shape[:2]) < intensity/1000
        pepper_mask = np.random.random(img.shape[:2]) < intensity/1000
        noisy[salt_mask] = 255
        noisy[pepper_mask] = 0
    elif noise_type == 'poisson':
        noisy = np.random.poisson(noisy).astype(np.float64)
    
    return np.clip(noisy, 0, 255).astype(np.uint8)

# PSNR (Peak Signal-to-Noise Ratio)
def calculate_psnr(original, processed):
    """Calculate PSNR between images"""
    mse = np.mean((original.astype(np.float64) - 
                   processed.astype(np.float64)) ** 2)
    if mse == 0:
        return float('inf')
    return 10 * np.log10(255**2 / mse)

# Test
img = np.zeros((300, 400, 3), dtype=np.uint8)
cv2.circle(img, (200, 150), 100, (255, 200, 100), -1)
cv2.rectangle(img, (50, 50), (150, 250), (100, 150, 255), -1)

# Add noise
noisy = add_noise(img, 'gaussian', 30)

# Apply filters
filters = apply_filters_comparison(noisy)

print("Filter Comparison (PSNR vs original):")
for name, filtered in filters.items():
    psnr = calculate_psnr(img, filtered)
    print(f"  {name:25} PSNR: {psnr:.2f} dB")
```

```python
# ตัวอย่างที่ 16: Morphological operations
import cv2
import numpy as np

def demonstrate_morphology(img_binary):
    """Demonstrate morphological operations"""
    results = {}
    
    # Kernels (structural elements)
    rect_kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))
    ellipse_kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
    cross_kernel = cv2.getStructuringElement(cv2.MORPH_CROSS, (5, 5))
    
    # Erosion - shrinks white regions
    results['Erosion'] = cv2.erode(img_binary, rect_kernel, iterations=2)
    
    # Dilation - expands white regions
    results['Dilation'] = cv2.dilate(img_binary, rect_kernel, iterations=2)
    
    # Opening = Erosion + Dilation (removes small white noise)
    results['Opening'] = cv2.morphologyEx(img_binary, cv2.MORPH_OPEN, rect_kernel)
    
    # Closing = Dilation + Erosion (fills small holes)
    results['Closing'] = cv2.morphologyEx(img_binary, cv2.MORPH_CLOSE, rect_kernel)
    
    # Gradient = Dilation - Erosion (finds edges)
    results['Gradient'] = cv2.morphologyEx(img_binary, cv2.MORPH_GRADIENT, rect_kernel)
    
    # Top Hat = Original - Opening (highlights bright details)
    results['TopHat'] = cv2.morphologyEx(img_binary, cv2.MORPH_TOPHAT, rect_kernel)
    
    # Black Hat = Closing - Original (highlights dark details)
    results['BlackHat'] = cv2.morphologyEx(img_binary, cv2.MORPH_BLACKHAT, rect_kernel)
    
    return results

# Create test binary image
binary_img = np.zeros((300, 400), dtype=np.uint8)
cv2.circle(binary_img, (200, 150), 80, 255, -1)
# Add noise
noise_mask = np.random.random(binary_img.shape) < 0.02
binary_img[noise_mask] = 255  # Salt noise
binary_img[~noise_mask & (np.random.random(binary_img.shape) < 0.01)] = 0  # Pepper

results = demonstrate_morphology(binary_img)
print("Morphological operations:")
for name, result in results.items():
    white_pixels = (result > 0).sum()
    print(f"  {name:15}: {white_pixels:6} white pixels")
```

```python
# ตัวอย่างที่ 17: Custom kernels and convolution
import cv2
import numpy as np

def apply_custom_kernels(img):
    """Apply various custom convolution kernels"""
    results = {}
    
    # Sharpen
    kernel_sharpen = np.array([
        [0, -1, 0],
        [-1, 5, -1],
        [0, -1, 0]
    ], dtype=np.float32)
    results['Sharpen'] = cv2.filter2D(img, -1, kernel_sharpen)
    
    # Emboss
    kernel_emboss = np.array([
        [-2, -1, 0],
        [-1, 1, 1],
        [0, 1, 2]
    ], dtype=np.float32)
    results['Emboss'] = cv2.filter2D(img, -1, kernel_emboss)
    
    # Box blur (averaging)
    kernel_blur = np.ones((5, 5), dtype=np.float32) / 25
    results['Box Blur'] = cv2.filter2D(img, -1, kernel_blur)
    
    # Horizontal Sobel (detects vertical edges)
    kernel_sobel_x = np.array([
        [-1, 0, 1],
        [-2, 0, 2],
        [-1, 0, 1]
    ], dtype=np.float32)
    results['Sobel X'] = cv2.filter2D(img, -1, kernel_sobel_x)
    
    # Vertical Sobel (detects horizontal edges)
    kernel_sobel_y = np.array([
        [-1, -2, -1],
        [0, 0, 0],
        [1, 2, 1]
    ], dtype=np.float32)
    results['Sobel Y'] = cv2.filter2D(img, -1, kernel_sobel_y)
    
    # Laplacian
    kernel_laplacian = np.array([
        [0, 1, 0],
        [1, -4, 1],
        [0, 1, 0]
    ], dtype=np.float32)
    results['Laplacian'] = cv2.filter2D(img, -1, kernel_laplacian)
    
    return results

img = np.zeros((200, 300, 3), dtype=np.uint8)
cv2.circle(img, (150, 100), 60, (200, 150, 100), -1)
cv2.rectangle(img, (20, 20), (100, 180), (100, 200, 150), -1)

results = apply_custom_kernels(img)
print("Custom kernel results:")
for name, result in results.items():
    print(f"  {name:15}: mean={result.mean():.2f}, std={result.std():.2f}")
```

---

## 9. Edge Detection {#edge-detection}

```python
# ตัวอย่างที่ 18: Edge detection methods
import cv2
import numpy as np

def compare_edge_detectors(img):
    """Compare different edge detection methods"""
    # Convert to grayscale
    if len(img.shape) == 3:
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    else:
        gray = img
    
    results = {}
    
    # Sobel
    sobel_x = cv2.Sobel(gray, cv2.CV_64F, 1, 0, ksize=3)
    sobel_y = cv2.Sobel(gray, cv2.CV_64F, 0, 1, ksize=3)
    sobel_combined = np.sqrt(sobel_x**2 + sobel_y**2)
    results['Sobel'] = np.uint8(np.clip(sobel_combined, 0, 255))
    
    # Scharr (more sensitive than Sobel)
    scharr_x = cv2.Scharr(gray, cv2.CV_64F, 1, 0)
    scharr_y = cv2.Scharr(gray, cv2.CV_64F, 0, 1)
    scharr_combined = np.sqrt(scharr_x**2 + scharr_y**2)
    results['Scharr'] = np.uint8(np.clip(scharr_combined, 0, 255))
    
    # Laplacian
    laplacian = cv2.Laplacian(gray, cv2.CV_64F)
    results['Laplacian'] = np.uint8(np.abs(laplacian))
    
    # Canny (best edge detector in most cases)
    results['Canny (low)'] = cv2.Canny(gray, 50, 100)
    results['Canny (high)'] = cv2.Canny(gray, 100, 200)
    
    # Canny with auto-thresholding
    median = np.median(gray)
    lower = max(0, 0.67 * median)
    upper = min(255, 1.33 * median)
    results['Canny (auto)'] = cv2.Canny(gray, lower, upper)
    
    # Prewitt (simpler than Sobel)
    kernel_x = np.array([[-1, 0, 1], [-1, 0, 1], [-1, 0, 1]])
    kernel_y = np.array([[-1, -1, -1], [0, 0, 0], [1, 1, 1]])
    prewitt_x = cv2.filter2D(gray, cv2.CV_64F, kernel_x)
    prewitt_y = cv2.filter2D(gray, cv2.CV_64F, kernel_y)
    prewitt = np.sqrt(prewitt_x**2 + prewitt_y**2)
    results['Prewitt'] = np.uint8(np.clip(prewitt, 0, 255))
    
    return results

# Create test image
img = np.zeros((300, 400, 3), dtype=np.uint8)
cv2.rectangle(img, (50, 50), (200, 200), (200, 200, 200), -1)
cv2.circle(img, (300, 150), 80, (150, 150, 150), -1)
img = cv2.GaussianBlur(img, (3, 3), 0)

results = compare_edge_detectors(img)
print("Edge Detection Comparison:")
for name, edges in results.items():
    edge_pixels = (edges > 0).sum()
    print(f"  {name:20}: {edge_pixels:6} edge pixels")
```

```python
# ตัวอย่างที่ 19: Canny edge detection deep dive
import cv2
import numpy as np

def canny_with_analysis(img, low_threshold=50, high_threshold=150):
    """
    Detailed Canny edge detection with analysis
    Steps:
    1. Noise reduction (Gaussian blur)
    2. Gradient computation (Sobel)
    3. Non-maximum suppression
    4. Double threshold
    5. Edge tracking by hysteresis
    """
    if len(img.shape) == 3:
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    else:
        gray = img
    
    # Step 1: Noise reduction
    blurred = cv2.GaussianBlur(gray, (5, 5), 1.4)
    
    # Step 2: Gradient
    grad_x = cv2.Sobel(blurred, cv2.CV_64F, 1, 0, ksize=3)
    grad_y = cv2.Sobel(blurred, cv2.CV_64F, 0, 1, ksize=3)
    magnitude = np.sqrt(grad_x**2 + grad_y**2)
    direction = np.arctan2(grad_y, grad_x) * 180 / np.pi
    
    # Step 3-5: Canny algorithm (OpenCV handles this)
    edges = cv2.Canny(blurred, low_threshold, high_threshold)
    
    # Find contours from edges
    contours, hierarchy = cv2.findContours(
        edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE
    )
    
    # Draw contours
    result = cv2.cvtColor(gray, cv2.COLOR_GRAY2BGR)
    cv2.drawContours(result, contours, -1, (0, 255, 0), 2)
    
    stats = {
        'edge_pixels': edges.sum() // 255,
        'num_contours': len(contours),
        'max_gradient': magnitude.max(),
        'mean_gradient': magnitude.mean(),
        'low_threshold': low_threshold,
        'high_threshold': high_threshold
    }
    
    return edges, contours, result, stats

# Test
img = np.zeros((300, 400, 3), dtype=np.uint8)
cv2.rectangle(img, (50, 50), (200, 250), (150, 150, 150), -1)
cv2.circle(img, (300, 150), 80, (100, 200, 100), -1)
cv2.triangle = [(320, 50), (380, 150), (260, 150)]
pts = np.array([[320, 50], [380, 150], [260, 150]])
cv2.fillPoly(img, [pts], (200, 100, 150))

edges, contours, result, stats = canny_with_analysis(img)
print("Canny Analysis:")
for key, value in stats.items():
    print(f"  {key}: {value}")
```

---

## 10. Object Detection - Template Matching {#template-matching}

```python
# ตัวอย่างที่ 20: Template matching
import cv2
import numpy as np

def template_matching(image, template, method=cv2.TM_CCOEFF_NORMED, 
                      threshold=0.8, multi=False):
    """
    Template matching with multiple methods
    Methods:
    - TM_CCOEFF_NORMED: Normalized correlation coefficient (best for most cases)
    - TM_CCORR_NORMED: Normalized cross-correlation
    - TM_SQDIFF_NORMED: Normalized squared difference (0=best match)
    """
    # Convert to grayscale if needed
    if len(image.shape) == 3:
        image_gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    else:
        image_gray = image
    
    if len(template.shape) == 3:
        template_gray = cv2.cvtColor(template, cv2.COLOR_BGR2GRAY)
    else:
        template_gray = template
    
    h, w = template_gray.shape
    
    # Apply template matching
    result = cv2.matchTemplate(image_gray, template_gray, method)
    
    if multi:
        # Find all matches above threshold
        if method in [cv2.TM_SQDIFF, cv2.TM_SQDIFF_NORMED]:
            matches_mask = result <= 1 - threshold
        else:
            matches_mask = result >= threshold
        
        locations = np.where(matches_mask)
        matches = []
        
        for pt in zip(*locations[::-1]):
            match = {
                'x': pt[0], 'y': pt[1],
                'w': w, 'h': h,
                'score': result[pt[1], pt[0]]
            }
            matches.append(match)
        
        # Apply Non-Maximum Suppression
        matches = non_max_suppression(matches, overlap_threshold=0.3)
        return matches
    else:
        # Best single match
        min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(result)
        
        if method in [cv2.TM_SQDIFF, cv2.TM_SQDIFF_NORMED]:
            best_loc = min_loc
            best_score = 1 - min_val
        else:
            best_loc = max_loc
            best_score = max_val
        
        if best_score >= threshold:
            return [{
                'x': best_loc[0], 'y': best_loc[1],
                'w': w, 'h': h,
                'score': best_score
            }]
        return []

def non_max_suppression(matches, overlap_threshold=0.3):
    """Remove overlapping matches"""
    if not matches:
        return []
    
    # Sort by score
    matches = sorted(matches, key=lambda x: -x['score'])
    
    kept = []
    for match in matches:
        # Check if overlaps with any kept match
        overlap = False
        for kept_match in kept:
            # Compute IoU
            x1 = max(match['x'], kept_match['x'])
            y1 = max(match['y'], kept_match['y'])
            x2 = min(match['x'] + match['w'], kept_match['x'] + kept_match['w'])
            y2 = min(match['y'] + match['h'], kept_match['y'] + kept_match['h'])
            
            if x2 > x1 and y2 > y1:
                intersection = (x2 - x1) * (y2 - y1)
                union = (match['w'] * match['h'] + 
                        kept_match['w'] * kept_match['h'] - intersection)
                iou = intersection / union if union > 0 else 0
                
                if iou > overlap_threshold:
                    overlap = True
                    break
        
        if not overlap:
            kept.append(match)
    
    return kept

# Demo
image = np.zeros((400, 600, 3), dtype=np.uint8)
template_pattern = np.zeros((50, 50, 3), dtype=np.uint8)
cv2.circle(template_pattern, (25, 25), 20, (0, 255, 0), -1)

# Place template multiple times in image
locations = [(100, 100), (250, 200), (450, 300)]
for x, y in locations:
    roi = image[y:y+50, x:x+50]
    image[y:y+50, x:x+50] = template_pattern

# Add noise
image = cv2.add(image, np.random.randint(0, 20, image.shape, dtype=np.uint8))

matches = template_matching(image, template_pattern, multi=True, threshold=0.7)
print(f"Found {len(matches)} matches:")
for i, match in enumerate(matches):
    print(f"  Match {i+1}: position=({match['x']}, {match['y']}), "
          f"score={match['score']:.4f}")
```

---

## 11. Face Detection - Haar Cascades {#face-detection}

```python
# ตัวอย่างที่ 21: Face Detection with Haar Cascades
import cv2
import numpy as np

class FaceDetector:
    """Face detector using Haar Cascades"""
    
    def __init__(self):
        # Load pre-trained classifiers
        # These come with OpenCV installation
        self.face_cascade = cv2.CascadeClassifier(
            cv2.data.haarcascades + 'haarcascade_frontalface_default.xml'
        )
        self.eye_cascade = cv2.CascadeClassifier(
            cv2.data.haarcascades + 'haarcascade_eye.xml'
        )
        self.smile_cascade = cv2.CascadeClassifier(
            cv2.data.haarcascades + 'haarcascade_smile.xml'
        )
        
        print("Haar Cascades loaded successfully!")
    
    def detect_faces(self, img, scale_factor=1.1, min_neighbors=5, 
                    min_size=(30, 30)):
        """Detect faces in image"""
        if len(img.shape) == 3:
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        else:
            gray = img
        
        # Equalize histogram for better detection
        gray_eq = cv2.equalizeHist(gray)
        
        faces = self.face_cascade.detectMultiScale(
            gray_eq,
            scaleFactor=scale_factor,
            minNeighbors=min_neighbors,
            minSize=min_size,
            flags=cv2.CASCADE_SCALE_IMAGE
        )
        
        return faces if len(faces) > 0 else []
    
    def detect_features(self, img, faces):
        """Detect eyes and smiles within faces"""
        if len(img.shape) == 3:
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        else:
            gray = img
        
        features = []
        for (x, y, w, h) in faces:
            face_roi = gray[y:y+h, x:x+w]
            
            # Detect eyes in upper half of face
            eyes = self.eye_cascade.detectMultiScale(
                face_roi[:h//2, :],
                scaleFactor=1.05,
                minNeighbors=4
            )
            
            # Detect smile in lower half
            smiles = self.smile_cascade.detectMultiScale(
                face_roi[h//2:, :],
                scaleFactor=1.5,
                minNeighbors=15
            )
            
            features.append({
                'face': (x, y, w, h),
                'eyes': [(ex + x, ey + y, ew, eh) for (ex, ey, ew, eh) in eyes],
                'smiles': [(sx + x, sy + y + h//2, sw, sh) 
                          for (sx, sy, sw, sh) in smiles]
            })
        
        return features
    
    def draw_detections(self, img, features):
        """Draw face/feature boxes on image"""
        result = img.copy()
        
        for item in features:
            x, y, w, h = item['face']
            
            # Draw face rectangle
            cv2.rectangle(result, (x, y), (x+w, y+h), (0, 255, 0), 2)
            
            # Add label
            cv2.putText(result, 'Face', (x, y-10),
                       cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
            
            # Draw eyes
            for (ex, ey, ew, eh) in item['eyes']:
                cv2.circle(result, 
                          (ex + ew//2, ey + eh//2), 
                          ew//2, (255, 0, 0), 2)
            
            # Draw smile
            for (sx, sy, sw, sh) in item['smiles']:
                cv2.rectangle(result, (sx, sy), (sx+sw, sy+sh), 
                            (0, 0, 255), 2)
        
        # Add count
        count_text = f"Faces: {len(features)}"
        cv2.putText(result, count_text, (10, 30),
                   cv2.FONT_HERSHEY_SIMPLEX, 1.0, (0, 0, 255), 2)
        
        return result
    
    def get_face_embeddings(self, img, faces):
        """Extract face regions for further processing"""
        embeddings = []
        for (x, y, w, h) in faces:
            face = img[y:y+h, x:x+w]
            resized = cv2.resize(face, (128, 128))
            normalized = resized.astype(np.float32) / 255.0
            embeddings.append(normalized.flatten())
        return embeddings

# Test (without real face image)
detector = FaceDetector()

# Simulate face detection result
fake_img = np.zeros((480, 640, 3), dtype=np.uint8)
fake_img[:] = (200, 180, 160)  # Skin-tone background

print("FaceDetector initialized!")
print(f"  Face cascade: {detector.face_cascade.empty() == False}")
print(f"  Eye cascade: {detector.eye_cascade.empty() == False}")
print(f"  Smile cascade: {detector.smile_cascade.empty() == False}")
```

```python
# ตัวอย่างที่ 22: Advanced object detection with contours
import cv2
import numpy as np

class ObjectCounter:
    """Count and analyze objects in image using contours"""
    
    def __init__(self, min_area=100, max_area=None):
        self.min_area = min_area
        self.max_area = max_area
    
    def preprocess(self, img):
        """Prepare image for contour detection"""
        if len(img.shape) == 3:
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        else:
            gray = img
        
        # Blur to reduce noise
        blurred = cv2.GaussianBlur(gray, (5, 5), 0)
        
        # Threshold
        _, binary = cv2.threshold(blurred, 0, 255, 
                                  cv2.THRESH_BINARY + cv2.THRESH_OTSU)
        
        # Morphological cleanup
        kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (3, 3))
        binary = cv2.morphologyEx(binary, cv2.MORPH_OPEN, kernel)
        
        return binary
    
    def find_objects(self, img):
        """Find and analyze objects"""
        binary = self.preprocess(img)
        
        # Find contours
        contours, hierarchy = cv2.findContours(
            binary, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE
        )
        
        objects = []
        for i, contour in enumerate(contours):
            area = cv2.contourArea(contour)
            
            # Filter by area
            if area < self.min_area:
                continue
            if self.max_area and area > self.max_area:
                continue
            
            # Get bounding box
            x, y, w, h = cv2.boundingRect(contour)
            
            # Get center
            M = cv2.moments(contour)
            if M['m00'] != 0:
                cx = int(M['m10'] / M['m00'])
                cy = int(M['m01'] / M['m00'])
            else:
                cx, cy = x + w//2, y + h//2
            
            # Compute properties
            perimeter = cv2.arcLength(contour, True)
            circularity = 4 * np.pi * area / (perimeter**2 + 1e-10)
            
            # Get convex hull
            hull = cv2.convexHull(contour)
            hull_area = cv2.contourArea(hull)
            convexity = area / (hull_area + 1e-10)
            
            # Aspect ratio
            aspect_ratio = w / h if h > 0 else 0
            
            # Classify shape
            if circularity > 0.85:
                shape = 'Circle'
            elif circularity > 0.7:
                shape = 'Oval'
            elif len(cv2.approxPolyDP(contour, 0.02 * perimeter, True)) == 4:
                shape = 'Rectangle' if 0.8 < aspect_ratio < 1.2 else 'Rectangle'
            elif len(cv2.approxPolyDP(contour, 0.02 * perimeter, True)) == 3:
                shape = 'Triangle'
            else:
                shape = 'Irregular'
            
            objects.append({
                'id': i,
                'area': area,
                'bbox': (x, y, w, h),
                'center': (cx, cy),
                'perimeter': perimeter,
                'circularity': circularity,
                'convexity': convexity,
                'aspect_ratio': aspect_ratio,
                'shape': shape
            })
        
        return objects
    
    def draw_analysis(self, img, objects):
        """Draw analysis overlay"""
        result = img.copy()
        
        for obj in objects:
            x, y, w, h = obj['bbox']
            cx, cy = obj['center']
            
            # Bounding box
            cv2.rectangle(result, (x, y), (x+w, y+h), (0, 255, 0), 2)
            
            # Center point
            cv2.circle(result, (cx, cy), 5, (0, 0, 255), -1)
            
            # Label
            label = f"{obj['shape']} #{obj['id']}"
            cv2.putText(result, label, (x, y-5),
                       cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 1)
        
        # Summary
        count_text = f"Objects: {len(objects)}"
        cv2.putText(result, count_text, (10, 30),
                   cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)
        
        return result

# Test
img = np.zeros((400, 600, 3), dtype=np.uint8)
# Draw various shapes
cv2.circle(img, (100, 200), 60, (200, 200, 200), -1)
cv2.rectangle(img, (200, 100), (350, 300), (180, 180, 180), -1)
cv2.rectangle(img, (400, 150), (550, 250), (220, 220, 220), -1)
pts = np.array([[150, 350], [250, 370], [200, 320]])
cv2.fillPoly(img, [pts], (160, 160, 160))

counter = ObjectCounter(min_area=500)
objects = counter.find_objects(img)
result = counter.draw_analysis(img, objects)

print(f"Found {len(objects)} objects:")
for obj in objects:
    print(f"  Object {obj['id']}: {obj['shape']}, "
          f"area={obj['area']:.0f}, "
          f"center=({obj['center'][0]}, {obj['center'][1]})")
```

---

## 12. Video Processing {#video}

```python
# ตัวอย่างที่ 23: Video processing
import cv2
import numpy as np
from pathlib import Path

class VideoProcessor:
    """Video processing utilities"""
    
    def __init__(self):
        self.fps = 30
        self.width = 640
        self.height = 480
    
    def read_video_info(self, filepath):
        """Read video metadata"""
        cap = cv2.VideoCapture(str(filepath))
        if not cap.isOpened():
            return None
        
        info = {
            'fps': cap.get(cv2.CAP_PROP_FPS),
            'frames': int(cap.get(cv2.CAP_PROP_FRAME_COUNT)),
            'width': int(cap.get(cv2.CAP_PROP_FRAME_WIDTH)),
            'height': int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT)),
            'duration': cap.get(cv2.CAP_PROP_FRAME_COUNT) / max(cap.get(cv2.CAP_PROP_FPS), 1),
            'codec': int(cap.get(cv2.CAP_PROP_FOURCC))
        }
        cap.release()
        return info
    
    def extract_frames(self, filepath, output_dir=None, every_n=1):
        """Extract frames from video"""
        cap = cv2.VideoCapture(str(filepath))
        frames = []
        frame_idx = 0
        
        while cap.isOpened():
            ret, frame = cap.read()
            if not ret:
                break
            
            if frame_idx % every_n == 0:
                if output_dir:
                    path = Path(output_dir) / f"frame_{frame_idx:06d}.jpg"
                    cv2.imwrite(str(path), frame)
                frames.append(frame)
            
            frame_idx += 1
        
        cap.release()
        return frames
    
    def create_video(self, frames, output_path, fps=30):
        """Create video from frames"""
        if not frames:
            return
        
        h, w = frames[0].shape[:2]
        fourcc = cv2.VideoWriter_fourcc(*'mp4v')
        writer = cv2.VideoWriter(str(output_path), fourcc, fps, (w, h))
        
        for frame in frames:
            writer.write(frame)
        
        writer.release()
    
    def process_stream(self, source=0, process_fn=None, max_frames=100):
        """Process video stream with function"""
        cap = cv2.VideoCapture(source)
        if not cap.isOpened():
            print(f"Cannot open source: {source}")
            return
        
        frames_processed = 0
        while frames_processed < max_frames:
            ret, frame = cap.read()
            if not ret:
                break
            
            # Apply processing
            if process_fn:
                processed = process_fn(frame)
            else:
                processed = frame
            
            # Display (requires GUI)
            # cv2.imshow('Stream', processed)
            # if cv2.waitKey(1) & 0xFF == ord('q'):
            #     break
            
            frames_processed += 1
        
        cap.release()
        # cv2.destroyAllWindows()
        return frames_processed

class BackgroundSubtractor:
    """Background subtraction for motion detection"""
    
    def __init__(self, method='MOG2'):
        methods = {
            'MOG2': cv2.createBackgroundSubtractorMOG2(
                detectShadows=True, varThreshold=16
            ),
            'KNN': cv2.createBackgroundSubtractorKNN(
                detectShadows=True
            )
        }
        self.subtractor = methods.get(method, methods['MOG2'])
        self.min_area = 500
    
    def process_frame(self, frame):
        """Process single frame"""
        # Apply background subtraction
        fg_mask = self.subtractor.apply(frame)
        
        # Remove shadows (shadows are gray in MOG2)
        _, fg_mask = cv2.threshold(fg_mask, 200, 255, cv2.THRESH_BINARY)
        
        # Morphological cleanup
        kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
        fg_mask = cv2.morphologyEx(fg_mask, cv2.MORPH_OPEN, kernel)
        fg_mask = cv2.morphologyEx(fg_mask, cv2.MORPH_DILATE, kernel)
        
        # Find moving objects
        contours, _ = cv2.findContours(
            fg_mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE
        )
        
        moving_objects = []
        for contour in contours:
            area = cv2.contourArea(contour)
            if area > self.min_area:
                x, y, w, h = cv2.boundingRect(contour)
                moving_objects.append({
                    'bbox': (x, y, w, h),
                    'area': area
                })
        
        # Draw on frame
        result = frame.copy()
        for obj in moving_objects:
            x, y, w, h = obj['bbox']
            cv2.rectangle(result, (x, y), (x+w, y+h), (0, 255, 0), 2)
            cv2.putText(result, f"Moving", (x, y-5),
                       cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 1)
        
        return result, fg_mask, moving_objects

# Simulate video processing
print("VideoProcessor initialized")
processor = VideoProcessor()

bg_subtractor = BackgroundSubtractor('MOG2')

# Simulate frames with moving object
frames = []
for i in range(30):
    frame = np.zeros((480, 640, 3), dtype=np.uint8)
    frame[:] = (100, 100, 100)  # Gray background
    
    # Moving object (circle)
    x = 100 + i * 10
    cv2.circle(frame, (x, 240), 40, (200, 100, 50), -1)
    frames.append(frame)

# Process
all_detections = []
for frame in frames:
    _, _, detections = bg_subtractor.process_frame(frame)
    all_detections.append(len(detections))

print(f"Processed {len(frames)} frames")
print(f"Detection counts: {all_detections[:10]}")
```

---

## 13. ตัวอย่างโปรแกรมจริง {#real-examples}

### Example 1: Face Detector App

```python
# ตัวอย่างที่ 24: Complete face detection app
import cv2
import numpy as np
from datetime import datetime

class FaceDetectionApp:
    """
    Complete face detection application with:
    - Multiple detection backends
    - Statistics tracking
    - Blur detection
    - Image quality assessment
    """
    
    def __init__(self):
        # Haar cascades
        self.face_cascade = cv2.CascadeClassifier(
            cv2.data.haarcascades + 'haarcascade_frontalface_default.xml'
        )
        
        # Stats
        self.stats = {
            'total_images': 0,
            'total_faces': 0,
            'detection_times': [],
        }
    
    def detect_blur(self, image, threshold=100):
        """Detect if image is blurry using Laplacian variance"""
        if len(image.shape) == 3:
            gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
        else:
            gray = image
        variance = cv2.Laplacian(gray, cv2.CV_64F).var()
        return variance < threshold, variance
    
    def enhance_for_detection(self, img):
        """Enhance image for better detection"""
        # CLAHE for contrast enhancement
        if len(img.shape) == 3:
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        else:
            gray = img
        
        clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
        enhanced = clahe.apply(gray)
        return enhanced
    
    def detect(self, img, enhance=True, draw=True):
        """Main detection function"""
        start_time = datetime.now()
        
        gray = self.enhance_for_detection(img) if enhance else cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        
        faces = self.face_cascade.detectMultiScale(
            gray,
            scaleFactor=1.1,
            minNeighbors=5,
            minSize=(50, 50),
            maxSize=(500, 500)
        )
        
        elapsed = (datetime.now() - start_time).total_seconds() * 1000
        
        # Update stats
        self.stats['total_images'] += 1
        self.stats['total_faces'] += len(faces)
        self.stats['detection_times'].append(elapsed)
        
        result = {
            'faces': faces.tolist() if len(faces) > 0 else [],
            'count': len(faces),
            'time_ms': elapsed,
        }
        
        # Draw
        if draw:
            drawn = img.copy()
            for (x, y, w, h) in faces:
                cv2.rectangle(drawn, (x, y), (x+w, y+h), (0, 255, 0), 2)
                cv2.putText(drawn, 'Face', (x, y-5),
                           cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
            
            info = f"Faces: {len(faces)} | {elapsed:.1f}ms"
            cv2.putText(drawn, info, (10, 30),
                       cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 0, 255), 2)
            result['drawn'] = drawn
        
        return result
    
    def get_stats(self):
        times = self.stats['detection_times']
        return {
            'total_images': self.stats['total_images'],
            'total_faces': self.stats['total_faces'],
            'avg_faces': self.stats['total_faces'] / max(self.stats['total_images'], 1),
            'avg_time_ms': sum(times) / len(times) if times else 0,
            'min_time_ms': min(times) if times else 0,
            'max_time_ms': max(times) if times else 0,
        }

# Test app
app = FaceDetectionApp()

# Simulate processing images
test_images = [
    np.random.randint(100, 200, (480, 640, 3), dtype=np.uint8)
    for _ in range(5)
]

print("Processing test images...")
for i, img in enumerate(test_images):
    result = app.detect(img)
    print(f"  Image {i+1}: {result['count']} faces detected "
          f"({result['time_ms']:.1f}ms)")

stats = app.get_stats()
print(f"\nStatistics:")
for k, v in stats.items():
    print(f"  {k}: {v:.2f}" if isinstance(v, float) else f"  {k}: {v}")
```

### Example 2: Image Filter App

```python
# ตัวอย่างที่ 25: Interactive image filter application
import cv2
import numpy as np
from enum import Enum

class FilterType(Enum):
    ORIGINAL = 'original'
    GRAYSCALE = 'grayscale'
    BLUR = 'blur'
    SHARPEN = 'sharpen'
    EDGES = 'edges'
    CARTOON = 'cartoon'
    SKETCH = 'sketch'
    HDR = 'hdr'
    WARM = 'warm'
    COOL = 'cool'
    VINTAGE = 'vintage'

class ImageFilterApp:
    """Image filter application"""
    
    def __init__(self):
        self.filters = {
            FilterType.ORIGINAL: self._filter_original,
            FilterType.GRAYSCALE: self._filter_grayscale,
            FilterType.BLUR: self._filter_blur,
            FilterType.SHARPEN: self._filter_sharpen,
            FilterType.EDGES: self._filter_edges,
            FilterType.CARTOON: self._filter_cartoon,
            FilterType.SKETCH: self._filter_sketch,
            FilterType.HDR: self._filter_hdr,
            FilterType.WARM: self._filter_warm,
            FilterType.COOL: self._filter_cool,
            FilterType.VINTAGE: self._filter_vintage,
        }
    
    def apply(self, img, filter_type: FilterType, **kwargs):
        """Apply filter to image"""
        fn = self.filters.get(filter_type, self._filter_original)
        return fn(img, **kwargs)
    
    def apply_all(self, img):
        """Apply all filters and return dict"""
        results = {}
        for filter_type in FilterType:
            results[filter_type.value] = self.apply(img, filter_type)
        return results
    
    def _filter_original(self, img, **kwargs):
        return img.copy()
    
    def _filter_grayscale(self, img, **kwargs):
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        return cv2.cvtColor(gray, cv2.COLOR_GRAY2BGR)
    
    def _filter_blur(self, img, strength=15, **kwargs):
        k = strength | 1  # Ensure odd
        return cv2.GaussianBlur(img, (k, k), 0)
    
    def _filter_sharpen(self, img, strength=1.5, **kwargs):
        kernel = np.array([[0, -1, 0], [-1, 5, -1], [0, -1, 0]])
        sharpened = cv2.filter2D(img, -1, kernel)
        return cv2.addWeighted(img, 1 - strength + 1, sharpened, strength - 1, 0)
    
    def _filter_edges(self, img, **kwargs):
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        edges = cv2.Canny(gray, 50, 150)
        return cv2.cvtColor(edges, cv2.COLOR_GRAY2BGR)
    
    def _filter_cartoon(self, img, **kwargs):
        """Cartoon-like effect"""
        # Bilateral filter for smooth colors
        color = img.copy()
        for _ in range(2):
            color = cv2.bilateralFilter(color, 9, 75, 75)
        
        # Edge detection
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        gray_blur = cv2.medianBlur(gray, 7)
        edges = cv2.adaptiveThreshold(
            gray_blur, 255,
            cv2.ADAPTIVE_THRESH_MEAN_C,
            cv2.THRESH_BINARY,
            9, 2
        )
        edges = cv2.cvtColor(edges, cv2.COLOR_GRAY2BGR)
        
        # Combine
        cartoon = cv2.bitwise_and(color, edges)
        return cartoon
    
    def _filter_sketch(self, img, **kwargs):
        """Pencil sketch effect"""
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        
        # Invert
        inv = cv2.bitwise_not(gray)
        
        # Blur the inverted
        blurred = cv2.GaussianBlur(inv, (21, 21), 0)
        
        # Dodge blend
        sketch = cv2.divide(gray, 255 - blurred, scale=256)
        
        return cv2.cvtColor(sketch, cv2.COLOR_GRAY2BGR)
    
    def _filter_hdr(self, img, **kwargs):
        """HDR-like effect"""
        # Tonemap for HDR look
        result = img.copy().astype(np.float32)
        
        # Local contrast enhancement
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY).astype(np.float32)
        blurred = cv2.GaussianBlur(gray, (0, 0), 3)
        high_pass = gray - blurred
        
        # Add detail enhancement
        for i in range(3):
            result[:, :, i] = result[:, :, i] + high_pass * 0.5
        
        result = np.clip(result, 0, 255).astype(np.uint8)
        
        # Boost saturation
        hsv = cv2.cvtColor(result, cv2.COLOR_BGR2HSV).astype(np.float32)
        hsv[:, :, 1] = np.clip(hsv[:, :, 1] * 1.4, 0, 255)
        hsv[:, :, 2] = np.clip(hsv[:, :, 2] * 1.1, 0, 255)
        
        return cv2.cvtColor(hsv.astype(np.uint8), cv2.COLOR_HSV2BGR)
    
    def _filter_warm(self, img, **kwargs):
        """Warm color tone"""
        result = img.copy().astype(np.float32)
        result[:, :, 2] = np.clip(result[:, :, 2] * 1.2, 0, 255)  # R
        result[:, :, 1] = np.clip(result[:, :, 1] * 1.1, 0, 255)  # G
        result[:, :, 0] = np.clip(result[:, :, 0] * 0.8, 0, 255)  # B
        return result.astype(np.uint8)
    
    def _filter_cool(self, img, **kwargs):
        """Cool color tone"""
        result = img.copy().astype(np.float32)
        result[:, :, 2] = np.clip(result[:, :, 2] * 0.8, 0, 255)  # R
        result[:, :, 1] = np.clip(result[:, :, 1] * 0.9, 0, 255)  # G
        result[:, :, 0] = np.clip(result[:, :, 0] * 1.3, 0, 255)  # B
        return result.astype(np.uint8)
    
    def _filter_vintage(self, img, **kwargs):
        """Vintage photo effect"""
        # Sepia tone
        result = img.copy().astype(np.float64)
        
        r = result[:, :, 2]
        g = result[:, :, 1]
        b = result[:, :, 0]
        
        new_r = r * 0.393 + g * 0.769 + b * 0.189
        new_g = r * 0.349 + g * 0.686 + b * 0.168
        new_b = r * 0.272 + g * 0.534 + b * 0.131
        
        result[:, :, 2] = np.clip(new_r, 0, 255)
        result[:, :, 1] = np.clip(new_g, 0, 255)
        result[:, :, 0] = np.clip(new_b, 0, 255)
        
        return result.astype(np.uint8)

# Test the filter app
app = ImageFilterApp()

# Create test image
img = np.zeros((300, 400, 3), dtype=np.uint8)
img[:] = (100, 150, 200)
cv2.circle(img, (200, 150), 80, (200, 100, 50), -1)
cv2.rectangle(img, (50, 50), (150, 250), (50, 200, 100), -1)

print("Testing all filters:")
for filter_type in FilterType:
    result = app.apply(img, filter_type)
    print(f"  {filter_type.value:15}: shape={result.shape}, "
          f"mean={result.mean():.1f}")
```

### Example 3: Object Counter

```python
# ตัวอย่างที่ 26: Real-time object counting
import cv2
import numpy as np
from collections import defaultdict
import time

class RealTimeObjectCounter:
    """
    Count objects crossing a line in video stream
    Uses centroid tracking to avoid double counting
    """
    
    def __init__(self, count_line=None):
        self.count_line = count_line  # (y, direction)
        self.trackers = {}
        self.next_id = 0
        self.counts = defaultdict(int)
        self.history = []
    
    def update_trackers(self, detections):
        """Update object trackers with new detections"""
        updated = {}
        
        for detection in detections:
            x, y, w, h = detection
            cx, cy = x + w // 2, y + h // 2
            
            # Match with existing tracker
            best_match = None
            best_dist = float('inf')
            
            for tid, (tx, ty, _) in self.trackers.items():
                dist = ((cx - tx)**2 + (cy - ty)**2)**0.5
                if dist < best_dist and dist < 100:
                    best_dist = dist
                    best_match = tid
            
            if best_match is not None:
                old_cx, old_cy, _ = self.trackers[best_match]
                updated[best_match] = (cx, cy, (old_cx, old_cy))
            else:
                updated[self.next_id] = (cx, cy, None)
                self.next_id += 1
        
        self.trackers = updated
        return updated
    
    def count_crossings(self, trackers, line_y):
        """Count objects crossing the counting line"""
        for tid, (cx, cy, prev) in trackers.items():
            if prev is None:
                continue
            
            prev_cx, prev_cy = prev
            
            # Check if crossed the line
            if prev_cy < line_y <= cy:
                self.counts['down'] += 1
                self.history.append({
                    'time': time.time(),
                    'direction': 'down',
                    'position': (cx, cy)
                })
            elif prev_cy >= line_y > cy:
                self.counts['up'] += 1
                self.history.append({
                    'time': time.time(),
                    'direction': 'up',
                    'position': (cx, cy)
                })
    
    def process_frame(self, frame, detections):
        """Process a single frame"""
        h, w = frame.shape[:2]
        line_y = h // 2  # Middle of frame
        
        # Update trackers
        updated = self.update_trackers(detections)
        
        # Count crossings
        self.count_crossings(updated, line_y)
        
        # Draw
        result = frame.copy()
        
        # Draw counting line
        cv2.line(result, (0, line_y), (w, line_y), (0, 255, 255), 2)
        
        # Draw trackers
        for tid, (cx, cy, _) in updated.items():
            cv2.circle(result, (cx, cy), 5, (0, 0, 255), -1)
            cv2.putText(result, f"ID:{tid}", (cx+5, cy-5),
                       cv2.FONT_HERSHEY_SIMPLEX, 0.4, (255, 255, 0), 1)
        
        # Draw counts
        up_text = f"UP: {self.counts['up']}"
        down_text = f"DOWN: {self.counts['down']}"
        total_text = f"TOTAL: {self.counts['up'] + self.counts['down']}"
        
        cv2.putText(result, up_text, (10, 30),
                   cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2)
        cv2.putText(result, down_text, (10, 60),
                   cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 0, 255), 2)
        cv2.putText(result, total_text, (10, 90),
                   cv2.FONT_HERSHEY_SIMPLEX, 0.8, (255, 255, 0), 2)
        
        return result
    
    def get_summary(self):
        return {
            'total': self.counts['up'] + self.counts['down'],
            'up': self.counts['up'],
            'down': self.counts['down'],
            'net_flow': self.counts['down'] - self.counts['up'],
            'events': len(self.history)
        }

# Test with simulated video
counter = RealTimeObjectCounter()

# Simulate 30 frames with moving objects
for frame_idx in range(30):
    frame = np.zeros((480, 640, 3), dtype=np.uint8)
    frame[:] = (50, 50, 50)
    
    # Simulate objects (moving down)
    detections = []
    for obj_id in range(3):
        x = 100 + obj_id * 150
        y = (frame_idx * 15 + obj_id * 50) % 480
        
        cv2.rectangle(frame, (x, y), (x+40, y+40), (200, 200, 200), -1)
        detections.append((x, y, 40, 40))
    
    result = counter.process_frame(frame, detections)

summary = counter.get_summary()
print("Object Counter Summary:")
for key, value in summary.items():
    print(f"  {key}: {value}")
```

---

## 14. แบบฝึกหัด {#exercises}

### แบบฝึกหัดที่ 1: Image Histogram Equalization

```python
# เฉลย
import cv2
import numpy as np

def histogram_equalization_comparison(img):
    """Compare different histogram equalization methods"""
    if len(img.shape) == 3:
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    else:
        gray = img
    
    results = {}
    
    # Standard equalization
    equalized = cv2.equalizeHist(gray)
    results['Global Equalization'] = equalized
    
    # CLAHE (Contrast Limited Adaptive Histogram Equalization)
    clahe_mild = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
    clahe_strong = cv2.createCLAHE(clipLimit=4.0, tileGridSize=(4, 4))
    
    results['CLAHE (mild)'] = clahe_mild.apply(gray)
    results['CLAHE (strong)'] = clahe_strong.apply(gray)
    
    # Manual gamma correction
    gamma = 1.5
    lut = np.array([min(255, int(((i/255.0)**gamma) * 255)) for i in range(256)], dtype=np.uint8)
    results['Gamma (1.5)'] = cv2.LUT(gray, lut)
    
    return results, gray

# Test
img = np.zeros((200, 300, 3), dtype=np.uint8)
img[:100] = (30, 30, 30)    # Dark upper half
img[100:] = (200, 200, 200)  # Light lower half
cv2.circle(img, (150, 100), 40, (100, 100, 100), -1)  # Gray circle

results, gray = histogram_equalization_comparison(img)

print("Histogram Equalization Comparison:")
print(f"Original: min={gray.min()}, max={gray.max()}, mean={gray.mean():.1f}")
print()
for name, result in results.items():
    print(f"{name}:")
    print(f"  min={result.min()}, max={result.max()}, mean={result.mean():.1f}")
    
    # Contrast measure
    hist = cv2.calcHist([result], [0], None, [256], [0, 256])
    used_bins = (hist > 0).sum()
    print(f"  Used bins: {used_bins}/256")
```

### แบบฝึกหัดที่ 2: Panorama Stitching

```python
# เฉลย
import cv2
import numpy as np

def stitch_images_simple(img1, img2):
    """Simple image stitching using ORB and RANSAC"""
    
    # Convert to grayscale
    gray1 = cv2.cvtColor(img1, cv2.COLOR_BGR2GRAY)
    gray2 = cv2.cvtColor(img2, cv2.COLOR_BGR2GRAY)
    
    # Detect features (ORB is free, SIFT/SURF need OpenCV contrib)
    orb = cv2.ORB_create(nfeatures=1000)
    kp1, desc1 = orb.detectAndCompute(gray1, None)
    kp2, desc2 = orb.detectAndCompute(gray2, None)
    
    print(f"Keypoints: {len(kp1)} and {len(kp2)}")
    
    if desc1 is None or desc2 is None:
        print("No descriptors found")
        return None
    
    # Match features
    bf = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
    matches = bf.match(desc1, desc2)
    matches = sorted(matches, key=lambda x: x.distance)[:50]
    
    print(f"Good matches: {len(matches)}")
    
    if len(matches) < 10:
        return None
    
    # Get matched points
    src_pts = np.float32([kp1[m.queryIdx].pt for m in matches]).reshape(-1, 1, 2)
    dst_pts = np.float32([kp2[m.trainIdx].pt for m in matches]).reshape(-1, 1, 2)
    
    # Find homography with RANSAC
    H, mask = cv2.findHomography(src_pts, dst_pts, cv2.RANSAC, 5.0)
    
    if H is None:
        return None
    
    inliers = mask.sum()
    print(f"Inliers: {inliers}/{len(matches)}")
    
    # Warp img1 to img2's perspective
    h1, w1 = img1.shape[:2]
    h2, w2 = img2.shape[:2]
    
    # Output size
    out_w = w1 + w2
    out_h = max(h1, h2)
    
    # Stitch
    result = np.zeros((out_h, out_w, 3), dtype=np.uint8)
    
    # Place img2
    result[:h2, :w2] = img2
    
    # Warp img1
    warped = cv2.warpPerspective(img1, H, (out_w, out_h))
    
    # Blend (simple alpha blending in overlap region)
    mask_warped = warped > 0
    result[mask_warped] = warped[mask_warped]
    
    return result

# Test with synthetic overlapping images
img1 = np.zeros((300, 400, 3), dtype=np.uint8)
img1[:] = (100, 120, 150)
cv2.rectangle(img1, (50, 50), (350, 250), (200, 100, 50), 2)
cv2.circle(img1, (200, 150), 60, (50, 200, 100), -1)

# Simulate img2 as offset of img1
img2 = np.zeros((300, 400, 3), dtype=np.uint8)
img2[:] = (110, 130, 160)
cv2.rectangle(img2, (10, 50), (310, 250), (200, 100, 50), 2)
cv2.circle(img2, (160, 150), 60, (50, 200, 100), -1)

result = stitch_images_simple(img1, img2)
if result is not None:
    print(f"\nStitched image: {result.shape}")
else:
    print("\nStitching failed (expected without real overlapping images)")
```

### แบบฝึกหัดที่ 3-8: Additional Exercises

```python
# แบบฝึกหัดที่ 3: Document Scanner
import cv2
import numpy as np

def document_scanner(img):
    """
    Auto document scanning:
    1. Edge detection
    2. Find document corners
    3. Perspective correction
    """
    
    def order_points(pts):
        """Order corners: top-left, top-right, bottom-right, bottom-left"""
        rect = np.zeros((4, 2), dtype=np.float32)
        s = pts.sum(axis=1)
        rect[0] = pts[np.argmin(s)]    # Top-left (min sum)
        rect[2] = pts[np.argmax(s)]    # Bottom-right (max sum)
        diff = np.diff(pts, axis=1)
        rect[1] = pts[np.argmin(diff)]  # Top-right (min diff)
        rect[3] = pts[np.argmax(diff)]  # Bottom-left (max diff)
        return rect
    
    # 1. Preprocess
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    edges = cv2.Canny(blurred, 50, 150)
    
    # 2. Find contours
    contours, _ = cv2.findContours(edges, cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)
    contours = sorted(contours, key=cv2.contourArea, reverse=True)[:5]
    
    # Find document contour (4-sided with largest area)
    doc_contour = None
    for cnt in contours:
        peri = cv2.arcLength(cnt, True)
        approx = cv2.approxPolyDP(cnt, 0.02 * peri, True)
        
        if len(approx) == 4:
            doc_contour = approx
            break
    
    if doc_contour is None:
        print("No document found")
        return img
    
    # 3. Order corners and apply perspective transform
    pts = order_points(doc_contour.reshape(4, 2))
    
    (tl, tr, br, bl) = pts
    
    # Compute output dimensions
    width_a = np.sqrt(((br[0] - bl[0]) ** 2) + ((br[1] - bl[1]) ** 2))
    width_b = np.sqrt(((tr[0] - tl[0]) ** 2) + ((tr[1] - tl[1]) ** 2))
    max_width = max(int(width_a), int(width_b))
    
    height_a = np.sqrt(((tr[0] - br[0]) ** 2) + ((tr[1] - br[1]) ** 2))
    height_b = np.sqrt(((tl[0] - bl[0]) ** 2) + ((tl[1] - bl[1]) ** 2))
    max_height = max(int(height_a), int(height_b))
    
    # Destination points
    dst = np.array([
        [0, 0], [max_width-1, 0],
        [max_width-1, max_height-1], [0, max_height-1]
    ], dtype=np.float32)
    
    # Perspective transform
    M = cv2.getPerspectiveTransform(pts, dst)
    warped = cv2.warpPerspective(img, M, (max_width, max_height))
    
    # 4. Enhance (adaptive threshold for paper look)
    gray_warped = cv2.cvtColor(warped, cv2.COLOR_BGR2GRAY)
    enhanced = cv2.adaptiveThreshold(
        gray_warped, 255,
        cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
        cv2.THRESH_BINARY, 11, 2
    )
    
    print(f"Document scanned: {max_width}×{max_height}")
    return warped

# Test
doc_img = np.ones((400, 600, 3), dtype=np.uint8) * 240
# Simulate document edges
cv2.rectangle(doc_img, (100, 80), (500, 320), (255, 255, 255), -1)
cv2.rectangle(doc_img, (100, 80), (500, 320), (100, 100, 100), 2)
# Add text
cv2.putText(doc_img, "Document Title", (150, 130),
           cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 0, 0), 2)
cv2.putText(doc_img, "This is document content.", (150, 200),
           cv2.FONT_HERSHEY_SIMPLEX, 0.5, (50, 50, 50), 1)

result = document_scanner(doc_img)
print(f"Result shape: {result.shape}")
```

```python
# แบบฝึกหัดที่ 4: Image Segmentation
import cv2
import numpy as np

def segment_by_color(img, num_clusters=5):
    """K-means based color segmentation"""
    
    # Reshape for k-means
    pixels = img.reshape(-1, 3).astype(np.float32)
    
    # K-means
    criteria = (cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 20, 0.5)
    _, labels, centers = cv2.kmeans(
        pixels, num_clusters, None,
        criteria, 3, cv2.KMEANS_RANDOM_CENTERS
    )
    
    centers = np.uint8(centers)
    
    # Reconstruct segmented image
    segmented = centers[labels.flatten()]
    segmented = segmented.reshape(img.shape)
    
    # Create label map
    label_map = labels.reshape(img.shape[:2])
    
    # Analyze segments
    segments = []
    total_pixels = img.shape[0] * img.shape[1]
    
    for k in range(num_clusters):
        mask = (label_map == k)
        area = mask.sum()
        percentage = area / total_pixels * 100
        color = centers[k]
        
        segments.append({
            'id': k,
            'color_bgr': tuple(color.tolist()),
            'area': area,
            'percentage': percentage,
            'mask': mask
        })
    
    # Sort by area
    segments.sort(key=lambda x: -x['area'])
    
    return segmented, segments

# Test
img = np.zeros((300, 400, 3), dtype=np.uint8)
# Sky
img[:150] = (200, 150, 80)     # Blue sky
# Ground
img[150:250] = (50, 150, 50)   # Green grass
# Object
img[250:] = (100, 100, 150)    # Gray road
cv2.circle(img, (300, 100), 60, (50, 80, 180), -1)  # Yellow sun

segmented, segments = segment_by_color(img, num_clusters=4)

print("Color Segmentation:")
for seg in segments:
    print(f"  Segment {seg['id']}: "
          f"color=BGR{seg['color_bgr']}, "
          f"area={seg['area']:6} ({seg['percentage']:.1f}%)")
```

```python
# แบบฝึกหัดที่ 5-8: Motion Detection, Optical Flow, Stereo Vision, Augmented Reality

# 5. Motion detection
def motion_detection_demo():
    """Frame differencing motion detection"""
    prev_frame = None
    
    frames = [
        np.random.randint(50, 100, (300, 400, 3), dtype=np.uint8)
        for _ in range(10)
    ]
    
    # Add moving object
    for i, frame in enumerate(frames):
        x = 50 + i * 30
        cv2.circle(frame, (x, 150), 30, (200, 200, 200), -1)
    
    detections = []
    for frame in frames:
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        gray_blur = cv2.GaussianBlur(gray, (21, 21), 0)
        
        if prev_frame is None:
            prev_frame = gray_blur
            detections.append(0)
            continue
        
        # Frame difference
        diff = cv2.absdiff(prev_frame, gray_blur)
        _, thresh = cv2.threshold(diff, 25, 255, cv2.THRESH_BINARY)
        
        # Count changed pixels
        motion_pixels = (thresh > 0).sum()
        detections.append(motion_pixels)
        
        prev_frame = gray_blur
    
    print("Motion Detection:")
    print(f"Changed pixels per frame: {detections}")
    print(f"Max motion: {max(detections):.0f} pixels")

motion_detection_demo()
```

---

## สรุป

ใน Part 83 นี้ เราได้เรียนรู้:

1. **Computer Vision Basics** - fundamentals และ image representation
2. **PIL/Pillow** - Python image processing library
3. **OpenCV** - powerful computer vision library
4. **Image I/O** - reading, displaying, saving images
5. **Color Spaces** - BGR, RGB, HSV, LAB และ conversion
6. **Transformations** - resize, rotate, flip, perspective
7. **Drawing** - rectangles, circles, lines, text
8. **Filters** - blur, sharpen, morphological operations
9. **Edge Detection** - Canny, Sobel, Laplacian
10. **Template Matching** - object detection by template
11. **Face Detection** - Haar Cascades
12. **Video Processing** - stream processing, background subtraction

### ขั้นต่อไป
- Part 84: LLM Integration - Anthropic API & OpenAI
- ศึกษาเพิ่มเติม: https://docs.opencv.org/

---
*Part 83 - Computer Vision: OpenCV & PIL | Python Course*
