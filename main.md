# Brain Corp Final Interview — Master Prep Guide
### Ayush Luhar | Thursday May 21, 2026 | 2:00–4:30 PM (Virtual)

---

## YOUR INTERVIEW SCHEDULE

| Time | Interviewer(s) | Role | Likely Focus |
|------|---------------|------|-------------|
| 2:00–3:00 | Sahil Dhayalkar + Shrenik Muralidhar | Staff Autonomy Engineer (SLAM/CV/DL) + Research SWE (Motion Planning) | Technical deep dive: SLAM concepts, path planning, perception, likely coding |
| 3:30–4:30 | Charles Deledalle | Research Scientist/Engineer (ML) | ML fundamentals, your ML projects, possibly research-flavored coding |

---

## PART 1: THE RESUME PROBLEM — WHAT THEY MIGHT BE READING

You have 5 resumes and don't know which one they have. Here's what's constant and what varies.

### SAFE — On ALL 5 Resumes (They WILL ask about these)

| Claim | Numbers to Know Cold |
|-------|---------------------|
| CSULB Robotics Lab (ISA) | 4 platforms, 50+ field tests, 95% reliability, 30% nav accuracy improvement, 25+ cross-layer failures debugged |
| CSULB Research Assistant | YOLO 92% mAP@0.5, 170ms → 65ms (2.6×), PySpark 100K+ records, MLflow 200+ experiments, 28-30 FPS |
| CHARUSAT Research Intern | T5, 2.5M data points, +42% BLEU, DDP + NCCL, 72h → 18h, Springer publication |
| Multi-Agent RL | OpenAI baseline within 15%, cooperative PPO, emergent behaviors |
| Springer Publication | "From Pixels to Text: CNN vs RNN in Image Processing" |

### VARIES — Only On Some Resumes (Be Ready But Don't Volunteer)

| Item | Which Resumes | Risk Level |
|------|--------------|------------|
| **Drone Landing Project** | Agility, Lila, Terabase (3/5) | MEDIUM — If they ask, you know it. If they don't mention it, don't bring it up unprompted as proof it's on yours. |
| **PID Control / MAVLink** | Agility, Lila, Terabase (3/5) | MEDIUM — Same as above |
| **ROS/ROS2 in skills** | DeepReach, Boston, Lila, Terabase (4/5) | LOW risk — 4 out of 5 mention it |
| **SLAM in skills** | Lila, Terabase only (2/5) | HIGH — If they ask "I see you have SLAM experience," you know it's one of these two |
| **HyperVect Internship** | DeepReach, Boston only (2/5) | MEDIUM — If they ask about it, you applied with one of those two |
| **ROS2 + MoveIt + Nav2 project** | Lila only (1/5) | HIGH signal — If they ask about your mobile manipulation platform, it's the Lila resume |
| **Visuomotor / Diffusion Policy** | DeepReach only (1/5) | HIGH signal — If they mention imitation learning or visuomotor, it's DeepReach |
| **Warehouse Box Detection (ROS)** | Boston only (1/5) | HIGH signal — If they mention warehouse picking, it's Boston Dynamics resume |
| **Contact-Rich Manipulation (force/torque)** | Agility only (1/5) | HIGH signal — If they mention force sensing or 500+ demos, it's Agility |
| **Tensor/Autograd Engine (C++)** | DeepReach only (1/5) | HIGH signal |
| **LLM Code Migration Agent** | Boston only (1/5) | HIGH signal |
| **GPS in drone project** | Terabase only (1/5) | If they mention GPS, it's Terabase |

### STRATEGY: How to Handle "Tell Me About X" When You're Not Sure

If they reference something specific from your resume and you can't tell which version they have:
- **Just answer confidently.** You did all these projects. The resume versions just frame them differently.
- **Listen to their exact wording** in the first few minutes — it'll tell you which resume they're reading.
- **Never say** "Oh, I'm not sure which resume I submitted." That's a red flag.
- **If they ask about something on only 1 resume** (like MoveIt or Diffusion Policy), now you know exactly which one they have. Adjust accordingly.

### YOUR CSULB ISA TITLE VARIES — Here's What They Might See

| Resume | Title They See |
|--------|---------------|
| DeepReach | Robotics Research Engineer (ISA) |
| Boston Dynamics | Robotics Lab Assistant |
| Agility Robotics | Instructional Student Assistant (Robotics) |
| Lila Sciences | Robotics Systems Engineer (ISA) |
| Terabase | Robotics Engineer (ISA) |

All describe the same role. If they call you "Robotics Engineer" vs "Lab Assistant," you'll know which version.

---

## PART 2: TOPIC PREP BY INTERVIEWER

### Interview 1 — Sahil (SLAM/CV/DL) + Shrenik (Motion Planning)

#### SLAM Concepts (for Sahil) — Know These Cold

**"What is SLAM?"**
> Simultaneous Localization and Mapping — the robot builds a map of its environment while simultaneously figuring out where it is within that map. The chicken-and-egg problem: you need a map to localize, but you need to know your location to build the map.

**"Filtering vs Graph-based SLAM?"**
> - Filtering (EKF-SLAM, Particle Filter): Maintains a probability distribution over the robot's state. Online, real-time, but doesn't revisit past estimates. EKF-SLAM uses Gaussian assumption. Particle filters (like FastSLAM) can handle non-linear/non-Gaussian.
> - Graph-based (Pose Graph): Stores all poses as nodes, constraints as edges. Optimizes the whole graph when loop closure is detected. More accurate, can correct past errors, but more computationally expensive. This is what most modern systems use (Cartographer, RTAB-Map).

**"What is loop closure?"**
> When the robot recognizes it has returned to a previously visited location. This lets it correct accumulated drift in its trajectory estimate. Without loop closure, small errors compound and the map becomes inconsistent.

**"What is an occupancy grid?"**
> A 2D grid where each cell stores the probability of being occupied (obstacle), free, or unknown. Built from sensor data (LiDAR, depth cameras). Used as the foundation for path planning — the robot plans paths through free cells.

**"How does your sensor fusion work?"**
> On the rover: fused camera (visual features, object detection), LiDAR (range/distance, point cloud), and IMU (orientation, acceleration) data. Each sensor compensates for others' weaknesses — camera fails in low light, LiDAR has no color/texture, IMU drifts over time. Used complementary filtering to combine IMU and visual odometry for localization.

**Your YOLO Edge Optimization (Sahil will love this — Brain Corp runs on edge devices)**
> Reduced inference from 170ms to 65ms on Jetson Nano (4GB RAM):
> - Pruning: removed low-magnitude weights/channels that contributed little to accuracy
> - Mixed-precision (FP16): halved memory bandwidth, Jetson's GPU has FP16 tensor cores
> - Result: 2.6× speedup with minimal accuracy loss, enabling 28-30 FPS real-time perception

#### Motion Planning (for Shrenik) — Know These Cold

**"BFS vs Dijkstra vs A* — when do you use each?"**
> - BFS: Unweighted graphs/grids. Guarantees shortest path in hop count. O(V+E). Use when all moves cost the same.
> - Dijkstra: Weighted graphs. Guarantees shortest weighted path. Use when terrain has varying costs (e.g., rough vs smooth floor).
> - A*: Dijkstra + heuristic. Faster than Dijkstra because the heuristic guides search toward the goal. Optimal if heuristic is admissible (never overestimates). This is what real robots use.

**"What is a costmap?"**
> A grid layered on top of the occupancy grid that assigns movement costs. Static layer (from the map), obstacle layer (from sensors), inflation layer (adds cost near obstacles for safety margin). The planner uses this to find paths that are not just shortest but also safest.

**"Global vs Local planner?"**
> - Global: Plans the overall path from start to goal using the full map (A*, Dijkstra, RRT). Runs less frequently.
> - Local: Handles real-time obstacle avoidance and trajectory tracking. Reacts to dynamic obstacles (people walking). Runs at high frequency. Examples: DWA (Dynamic Window Approach), TEB (Timed Elastic Band).

**"What is RRT?"**
> Rapidly-exploring Random Tree. Samples random points in the configuration space, extends the nearest tree node toward the sample. Good for high-dimensional spaces where grid-based search is too expensive. RRT* is the optimal variant that rewires the tree.

**"How does the robot avoid a person who walks in front of it?"**
> Local planner detects the person via perception (camera/LiDAR), updates the costmap with the new obstacle, and replans a trajectory around them. If no safe path exists, the robot stops and waits. Brain Corp robots operate in stores with people — this is their core challenge.

**Coordinate Transforms**
> Every sensor reading is in the sensor's local frame. To combine LiDAR and camera data, or to plan in the world frame, you apply rotation + translation transforms. For 2D: x' = x·cos(θ) - y·sin(θ) + tx, y' = x·sin(θ) + y·cos(θ) + ty. In ROS this is handled by tf2.

#### Potential Coding Questions (Interview 1)

1. **BFS shortest path on a grid** — Most likely. Practice writing it clean in 10 minutes.
2. **A* on a grid** — Medium-likely. Know the heap-based implementation.
3. **Number of islands / connected components** — Flood fill, DFS/BFS.
4. **2D coordinate transform** — Given a robot pose and a sensor reading in local frame, return world frame coordinates.
5. **Line segment intersection** — Does the planned path collide with an obstacle edge?
6. **LRU Cache** — Data caching on memory-limited hardware.

---

### Interview 2 — Charles Deledalle (ML Research Scientist)

#### ML Fundamentals

**"Supervised vs Unsupervised vs Reinforcement Learning?"**
> - Supervised: Learn from labeled data (input → output pairs). Classification, regression. Your YOLO work.
> - Unsupervised: Find structure in unlabeled data. Clustering, dimensionality reduction.
> - RL: Learn by interacting with environment, maximize reward. Your multi-agent PPO work.

**"What is overfitting and how do you prevent it?"**
> Model memorizes training data, fails on new data. Prevention: more data, data augmentation, dropout, regularization (L1/L2), early stopping, cross-validation. In your YOLO work: data augmentation (flips, rotations, color jitter) was key.

**"CNNs — why do they work for images?"**
> - Translation invariance: same features detected regardless of position
> - Parameter sharing: same filter applied across the whole image (vs fully connected where every pixel has its own weight)
> - Hierarchical features: early layers detect edges, middle layers detect textures/parts, deep layers detect objects
> - Local connectivity: each neuron only sees a small receptive field, matching how visual features are spatially local

**"Explain your T5 fine-tuning work"**
> Fine-tuned T5 (text-to-text transformer) on 2.5M data points for text generation. Key decisions: learning rate scheduling (warmup + cosine decay), batch size tuning, mixed-precision training (FP16) to fit in GPU memory. Distributed across GPUs using DDP (each GPU gets a data shard, gradients are all-reduced). NCCL backend for fast GPU-to-GPU communication. Cut training from 72h to 18h (4× speedup across multiple GPUs).

**"Explain PPO in your multi-agent RL project"**
> PPO (Proximal Policy Optimization): policy gradient method that clips the objective to prevent too-large updates. Advantages: stable training, doesn't collapse like vanilla policy gradient. In the multi-agent setting: each agent has its own policy but shares the environment. Emergent behaviors arose because agents learned to cooperate through reward shaping — e.g., coordinating to achieve goals neither could alone.

**"How would you approach: 'The robot needs to classify floor types from camera images'?"**
> 1. Data collection: capture images from the robot's camera across different floor types in stores
> 2. Labeling: annotate floor type per image (tile, carpet, concrete, wood, wet)
> 3. Model: start with pretrained CNN (ResNet/EfficientNet), fine-tune on floor data. Transfer learning because floor textures are visual features similar to ImageNet categories
> 4. Edge deployment: quantize/prune for Jetson, benchmark latency
> 5. Evaluation: accuracy, confusion matrix (which floors get confused?), test on new stores
> 6. Integration: feed classification to the planner — different floor types may need different cleaning speeds/patterns

#### Potential Coding Questions (Interview 2)

1. **Implement K-Nearest Neighbors from scratch**
2. **Implement a simple gradient descent step**
3. **Matrix operations / linear algebra** (dot product, matrix multiply)
4. **Data processing** — parse sensor data, compute statistics
5. **Binary search / sorting** — general algorithmic problem
6. **Simple ML pipeline** — load data, train a model, evaluate

---

## PART 3: YOUR PROJECT STORIES — MASTER VERSIONS

### Story 1: CSULB Robotics Lab (ISA) — Your #1 Story

**Situation:** Joined the CSULB Mechanical & Aerospace Engineering lab working on autonomous robotic systems across 4 platforms — ground rover, aerial drone, underwater vehicle, and a manipulator.

**What you did:**
- Built perception pipelines integrating cameras, LiDAR, IMU, and depth sensors on Jetson Nano
- Implemented sensor fusion for localization and navigation
- Designed and ran 50+ field tests, iterating on perception and control
- Debugged 25+ cross-layer failures spanning hardware and software
- Collaborated across mechanical, electrical, and software teams

**Results:** 95% mission reliability, 30% improvement in navigation accuracy, reduced localization drift.

**Why it matters for Brain Corp:** Their robots run on edge hardware in dynamic public spaces. Your experience with real hardware, real sensors, real field testing, and real failure debugging is directly transferable.

### Story 2: YOLO Edge Optimization — Your #2 Story

**Situation:** Needed real-time object detection on Jetson Nano (4GB RAM, limited compute).

**What you did:**
- Trained YOLO model achieving 92% mAP@0.5
- Pruned low-contribution channels, applied mixed-precision (FP16)
- Built async video pipeline for real-time processing

**Results:** 170ms → 65ms inference (2.6× speedup), 28-30 FPS, runs within 4GB constraint.

**Why it matters for Brain Corp:** BrainOS runs on embedded hardware in robots. Edge optimization is their daily reality.

### Story 3: Drone Landing (if on your resume)

**Situation:** Built an autonomous system to land a drone on a moving platform.

**What you did:**
- Full autonomy stack: perception → localization → path planning → closed-loop control
- Sensor fusion (camera + GPS + IMU) on Raspberry Pi
- PID control for real-time trajectory adjustments
- MAVLink communication for position tracking

**Results:** 90% landing success, 15cm accuracy.

**Why it matters for Brain Corp:** End-to-end autonomy, real hardware, real-time control — this is what their autonomy engineers do at scale.

### Story 4: T5 / CHARUSAT Research

**Situation:** Research project fine-tuning transformer models on large dataset.

**What you did:**
- Fine-tuned T5 on 2.5M data points
- Built distributed training pipeline (DDP + NCCL) across GPUs
- Mixed-precision optimization

**Results:** 42% BLEU improvement, 72h → 18h training time, Springer publication.

**Why it matters for Brain Corp:** Shows you can train models at scale, do rigorous experiments, and publish.

---

## PART 4: QUESTIONS TO ASK THEM

### For Sahil + Shrenik (2:00 interview):
- "What does the autonomy stack look like at Brain Corp — how do SLAM, perception, and motion planning connect?"
- "What's the hardest navigation challenge the robots face in real stores?"
- "What would an intern on the autonomy team be working on day-to-day?"

### For Charles (3:30 interview):
- "What ML research problems is the team currently tackling?"
- "How do you bridge the gap between research and production at Brain Corp?"
- "What does the intern project scope typically look like — is it more research-focused or production-focused?"

---

## PART 5: BRAIN CORP KNOWLEDGE TO DROP NATURALLY

- **BrainOS** is their autonomy software platform — powers 35,000+ robots globally
- Robots operate in **stores, schools, warehouses, hospitals, airports** — all public spaces with people
- Main products: autonomous **floor scrubbers** (their bread and butter), **inventory scanners** (Sam's Club partnership — 600 locations)
- Key challenge: navigating **dynamic environments** with unpredictable humans, carts, spills
- Recently named a **Top Workplace** by San Diego Union-Tribune
- They work with OEM partners (Tennant Company) — Brain Corp provides the brain, partners build the body
- CTO is John Black — 3 decades of product development experience

---

## PART 6: THREE-DAY STUDY PLAN

### Tuesday (May 19)
- **Morning:** Review this document end-to-end. Memorize all numbers from your projects.
- **Afternoon:** Code BFS grid search, A*, and Number of Islands from scratch 3 times each until you can do them eyes-closed.
- **Evening:** Review SLAM concepts (filtering vs graph-based, loop closure, occupancy grids).

### Wednesday (May 20)
- **Morning:** Code coordinate transforms, LRU Cache, K-nearest neighbors from scratch.
- **Afternoon:** Practice explaining each project story out loud (2 minutes each, timed). Record yourself.
- **Evening:** Review ML fundamentals (CNN architecture, overfitting, PPO, supervised/unsupervised/RL). Review motion planning (A*, RRT, costmaps, global vs local planner).

### Thursday (May 21) — Interview Day
- **Morning:** Light review only. Re-read this doc once. Do NOT cram new material.
- **1:30 PM:** Set up your environment: quiet room, good internet, camera on, IDE open (VSCode or whatever you code in), water bottle.
- **1:50 PM:** Join Google Meet link early, test audio/video.
- **2:00 PM:** Interview 1 (Sahil + Shrenik)
- **3:00–3:30:** Break. Breathe. Don't review. Eat something.
- **3:30 PM:** Interview 2 (Charles)
