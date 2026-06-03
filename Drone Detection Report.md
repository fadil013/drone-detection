# Drone Detection System — Research Report
**Prepared by:** Intern Research  
**Purpose:** Military Defense Project — Drone Detection using Raspberry Pi 4  
**Date:** April 2026

---

## 1. What We're Building

A ground-based drone detection system that uses a camera and AI to spot hostile drones in real-time, tell them apart from birds and planes, and trigger a response — all running on a **Raspberry Pi 4 (4GB, v1.2)**.

The goal: **90%+ detection accuracy** with stable FPS (frames per second).

---

## 2. How Real Military Systems Do It (Reference Systems)

### AeroEye (by Drone Defence UK)
- Uses HD cameras + AI to detect drones **over 1 km away**
- Tracks multiple drones at once, full 360° coverage
- Automatically zooms in and follows the drone's path

### Anduril Roadrunner-M (USA)
- A small jet-powered drone that **hunts and destroys other drones**
- Launches in seconds from a box called the "Nest"
- Flies at high speed, locks onto the target, destroys it with a warhead
- If it turns out to be a false alarm — it flies back and lands safely
- Cost: ~$500,000 per unit

### Anduril Lattice AI (The Brain)
- Software that connects all sensors (cameras, radar, satellites) into one screen
- AI automatically detects, classifies, and tracks threats
- One operator can manage the whole system from a laptop

**Key takeaway for our project:** Real systems use multiple sensors together. Our Raspberry Pi version focuses on the **camera + AI** layer of this.

---

## 3. The Camera — Raspberry Pi Camera Module 3 NoIR

| Feature | Spec |
|---|---|
| Sensor | Sony IMX708, 12MP |
| Resolution | Up to 4608×2592 |
| Video | 1080p @ 50 FPS, 720p @ 100 FPS |
| Field of View | 75° (standard) or 120° (wide) |
| Night Vision | Yes (NoIR = no IR filter, works with IR LEDs) |
| HDR | Yes |
| Price | ~$25 |

**Detection Range:** Realistically detects a drone clearly up to **100–150 metres** with standard lens. For longer range, add a USB telephoto camera.

For night use: pair the NoIR camera with **850nm IR LED floodlights** — invisible to humans, visible to the camera.

---

## 4. The AI Model — YOLOv8 (You Only Look Once)

**Why YOLO?** It's fast, accurate, and runs on small hardware like the Raspberry Pi.

**Which version for RPi 4?** → **YOLOv8n (nano)** — smallest and fastest version, optimized for limited hardware.

**What it does:** Looks at each camera frame, draws a box around the drone, and says "drone detected — 94% confidence."

**Performance target:**
- mAP50 (accuracy score): **above 0.90** (90%+)
- FPS on RPi 4: **5–15 FPS** (stable, usable for real-time detection)
- To boost FPS: convert model to **INT8 format** using TFLite or ONNX

---

## 5. The Dataset — What to Train On

We need images of drones AND non-drones so the model learns the difference.

### Classes to include:
| Class | Why |
|---|---|
| Drone / UAV | Main target to detect |
| Bird | Looks similar on camera, must not trigger false alarm |
| Airplane | Different flight pattern, must be ignored |
| Helicopter | Rotating blades — similar to drone but bigger |
| Balloon | Slow-moving, non-threat |

### Where to get the data (free):
- **Roboflow Universe** → search "drone detection" → 2,600–22,000+ labelled images available
- **AOD-4 Dataset** → 22,516 images, 4 classes: drones, birds, helicopters, airplanes
- **Anti-UAV Dataset** → military-focused, infrared + visible light footage

### Data augmentation (making dataset bigger/stronger):
Flip, rotate, blur, crop, and change brightness on existing images → this multiplies your dataset size and makes the model more robust.

---

## 6. Training the Model

| Step | What to Do |
|---|---|
| 1. Collect dataset | Download from Roboflow (5,000–10,000 images minimum) |
| 2. Annotate | Label boxes around drones in every image (Roboflow auto-annotates) |
| 3. Train | Use **Google Colab** (free GPU) — train for **150 epochs** |
| 4. Export | Export to **TFLite or ONNX** format for Raspberry Pi |
| 5. Deploy | Copy model to RPi, run with Python + OpenCV |

**Target accuracy:** mAP50 > 0.90 achieved consistently with 150 epochs + augmentation.

---

## 7. How False Detections Are Avoided

The biggest challenge: **drones and birds look almost identical on radar and camera.**

| Object | How We Tell It Apart From a Drone |
|---|---|
| Bird | Irregular flapping movement, no rigid frame shape, different flight pattern |
| Airplane | Much faster, fixed wings, flies in straight lines at high altitude |
| Balloon | Drifts with wind, no motor, no directional flight |
| Helicopter | Larger, slower rotation, very different size profile |

**Techniques used:**
- Multi-class training (model sees all these objects during training)
- Confidence threshold set to **>0.75** (ignores weak detections)
- Behavioral tracking: if object moves like a drone (directional, fast, hovering) → alert

---

## 8. Full System Flow (How It All Connects)

```
Camera (Pi Cam v3 NoIR)
        ↓
Raspberry Pi 4 (4GB)
        ↓
YOLOv8n model processes each frame
        ↓
Drone detected? → Trigger alert / log / send coordinates
        ↓
Human operator reviews → Authorizes response
        ↓
Counter-drone action (e.g., Roadrunner-M intercept or jammer)
```

---

## 9. Budget (Basic Setup)

| Item | Estimated Cost |
|---|---|
| Raspberry Pi 4 (4GB) | ~$55 |
| Camera Module 3 NoIR | ~$25 |
| IR LED floodlight (850nm) | ~$15 |
| MicroSD card (64GB) | ~$10 |
| Power supply + case | ~$15 |
| Google Coral USB TPU (optional, boosts FPS) | ~$60 |
| **Total** | **~$120–$180** |

---

## 10. Key Takeaways

- Real military systems (AeroEye, Anduril) use **multiple sensors** — radar, RF, cameras, acoustics — fused together. Our system covers the **visual/AI layer**.
- **YOLOv8n** is the right model for Raspberry Pi 4 — fast, accurate, lightweight.
- Train on **5,000–10,000 images** with drone, bird, airplane, balloon, helicopter classes.
- **150 epochs + data augmentation** consistently hits 90%+ accuracy.
- The **Camera Module 3 NoIR** covers day and night detection up to ~150m.
- Export to **TFLite INT8** to keep FPS stable on the Pi.
- Always run a **confidence threshold of 0.75+** to kill false positives.

---

*Report prepared for internal military internship research purposes.*
