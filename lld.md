# System Design — Absolute Beginner Guide
### You know nothing about system design. That's fine. Start here.

---

## WHAT IS SYSTEM DESIGN?

System design is just answering the question: **"How would you build this?"**

Not writing actual code. Just explaining:
- What are the **pieces** of the system?
- How do the pieces **talk to each other**?
- What happens when something **breaks**?

That's literally it. No math. No memorization. Just logical thinking about how things connect.

---

## ANALOGY: A PIZZA SHOP

Before we touch any tech stuff, let's design a pizza shop. Seriously.

**The question:** "Design a system to serve pizza to customers."

**The pieces:**
```
Customer → Cashier → Kitchen → Oven → Delivery/Pickup Counter → Customer
```

**Decisions you'd make:**
- How many ovens do we need? (depends on how many pizzas per hour)
- What if 100 people order at once? (we need a queue/waiting system)
- What if an oven breaks? (we need a backup or a plan to handle it)
- Do we write orders on paper or use a screen? (paper is simple, screen is faster)
- Do we make every pizza from scratch or prep dough in advance? (speed vs freshness tradeoff)

**That's system design.** You just designed a system. Every tech system design question is the same thing — identify the pieces, connect them, handle the "what ifs."

---

## THE BUILDING BLOCKS

Every system is made of a few basic building blocks. Learn these and you can talk about any system.

### 1. Client

The thing that asks for something.

Pizza shop: the customer.
Web: your browser or phone app.
Robot: the fleet manager saying "go clean aisle 5."

### 2. Server

The thing that does the work and sends back a response.

Pizza shop: the kitchen.
Web: a computer somewhere that processes your request and sends back data.
Robot: the robot's onboard computer that processes sensor data.

### 3. Database

Where you store stuff so you don't lose it.

Pizza shop: the order book, the recipe binder, the inventory list.
Web: where user accounts, posts, messages live.
Robot: the map of the store, the log of where it's been, the cleaning schedule.

**Two main types:**
- **SQL (Relational):** Data in tables with rows and columns. Like an Excel spreadsheet. Good when data is structured and you need to query it precisely. (PostgreSQL, MySQL)
- **NoSQL:** More flexible. Can store documents, key-value pairs, etc. Good when data doesn't fit neatly into tables or you need speed over structure. (MongoDB, Redis)

**When to pick which:**
- User accounts, orders, structured records → SQL
- Sensor logs coming in fast, flexible data, caching → NoSQL
- Not sure → SQL is the safe default

### 4. API

How the pieces talk to each other. (You just learned this in the API doc!)

Pizza shop: the cashier is the API between customer and kitchen.
Web: the URLs and endpoints that let client and server communicate.
Robot: the interface between the perception module and the planning module.

### 5. Queue (Message Queue)

A waiting line for tasks. When work comes in faster than you can process it, it waits in a queue.

Pizza shop: the order tickets lined up by the oven.
Tech: RabbitMQ, Kafka, Redis Queue.
Robot: sensor data frames waiting to be processed.

**Why queues matter:** Without a queue, if 50 requests come in at once, your system crashes. With a queue, they wait their turn.

### 6. Cache

A shortcut. Store frequently-used results so you don't recompute them.

Pizza shop: pre-rolled dough balls. You COULD make dough from scratch for every pizza, but why?
Tech: Redis, Memcached — store results in fast memory instead of hitting the slow database every time.
Robot: store the map in RAM instead of reading it from disk every loop.

**Rule of thumb:** If you're reading the same data over and over, cache it.

### 7. Load Balancer

Distributes incoming work across multiple workers so no single one gets overwhelmed.

Pizza shop: if you have 3 ovens, you don't shove all pizzas into oven 1.
Tech: distributes web requests across multiple servers.
Robot fleet: distributes cleaning tasks across multiple robots.

---

## HOW THESE PIECES CONNECT

Here's how a typical web system looks:

```
User's Phone
    |
    ↓
Load Balancer  (picks which server handles this request)
    |
    ↓
Server  (runs the logic)
    |
    ├──→ Cache  (check here first — fast)
    |
    ├──→ Database  (if not in cache — slower)
    |
    └──→ Queue  (for heavy tasks: "process this later")
              |
              ↓
          Worker  (does the heavy task in background)
```

That's like 80% of system design right there. Most web systems are just different arrangements of these same pieces.

---

## THE FRAMEWORK: Answering Any System Design Question

When they ask "design X", follow these 4 steps in order:

### Step 1: ASK QUESTIONS (2 minutes)

Before you design anything, ask:

> "How many users/robots/requests are we talking about?"
> "What's most important — speed, accuracy, or cost?"
> "Does this need to work in real-time?"
> "What happens if it goes down for a minute?"

**Why:** A system for 100 users is completely different from one for 10 million. You NEED to know the scale before you start. Asking questions also shows you think before you build.

### Step 2: DRAW THE BIG PICTURE (5 minutes)

Draw boxes and arrows. Label them. Keep it simple.

Example — "Design a URL shortener (like bit.ly)":

```
User types long URL
    |
    ↓
Server (generates short code, stores mapping)
    |
    ├──→ Database (long_url ↔ short_code)
    |
    ↓
Returns short URL to user

---

User clicks short URL
    |
    ↓
Server (looks up short_code → finds long_url)
    |
    ├──→ Cache (check here first — most popular links are cached)
    ├──→ Database (if not in cache)
    |
    ↓
Redirects user to long URL
```

That's it. That's a system design answer for a URL shortener. Boxes and arrows.

### Step 3: GO DEEPER ON ONE PART (10 minutes)

The interviewer will pick a piece and say "tell me more." This is where you show depth.

For the URL shortener they might ask:
- "How do you generate the short code?" → hash the URL, or use an auto-incrementing counter converted to base62
- "What if two people shorten the same URL?" → check if it already exists in the database first
- "What if the database goes down?" → replicas (copies of the database on other machines)
- "How do you handle 1 million clicks per second?" → cache the popular links, add more servers behind the load balancer

You don't need to know all these answers perfectly. The point is showing you can **think through problems logically.**

### Step 4: TALK ABOUT TRADE-OFFS (2 minutes)

Every design choice has a downside. Mentioning trade-offs shows maturity.

> "I used a cache to speed things up, but the trade-off is the cache might serve stale data if the database was updated recently."

> "I used SQL for consistency, but if we need to handle millions of writes per second, we might switch to NoSQL."

> "I put everything on one server for simplicity, but if it goes down, the whole system goes down. For production we'd want multiple servers."

---

## KEY CONCEPTS (Just Know What These Words Mean)

### Scalability

Can the system handle more load?

- **Vertical scaling (scale up):** Get a bigger, faster machine. Simple but has a ceiling.
- **Horizontal scaling (scale out):** Add more machines. Harder but no ceiling.

Pizza shop: Vertical = buy a bigger oven. Horizontal = buy more ovens.

### Latency vs Throughput

- **Latency:** How long ONE request takes. (How long until MY pizza is ready?)
- **Throughput:** How many requests per second. (How many pizzas per hour can the shop make?)

You can have low latency but low throughput (one super-fast oven, one pizza at a time) or high throughput but high latency (100 ovens, but each takes an hour).

### Availability

What percentage of the time is the system working?

- 99% uptime = down 3.65 days/year
- 99.9% uptime = down 8.7 hours/year
- 99.99% uptime = down 52 minutes/year

**How to improve availability:** Have backups (replicas). If server 1 dies, server 2 takes over. This is called **redundancy.**

### Single Point of Failure

If ONE thing breaking kills the whole system, that thing is a single point of failure.

Pizza shop: if you have ONE oven and it breaks, you're closed. Fix: get a second oven.
Tech: if you have ONE database server and it crashes, all data is gone. Fix: database replicas.

**Always ask yourself: "What if this piece dies?"**

### Consistency vs Availability (The CAP Trade-off)

You can't always have both:
- **Consistency:** Everyone always sees the latest data
- **Availability:** The system always responds, even if the data might be slightly old

Example: You update your profile picture on Instagram. Your friend in Japan might see the old picture for a few seconds. Instagram chose **availability** (always loads) over **consistency** (always latest data). That's fine for social media. NOT fine for a bank balance.

For a robot: you want the map to be **consistent** (latest version). You'd rather the robot stops and waits than navigates with an old map.

---

## ROBOTICS SYSTEM DESIGN (The Brain Corp Flavor)

For Brain Corp specifically, "system design" probably means: **"How would you architect the software for an autonomous robot?"**

### The Universal Robot Architecture

Every autonomous robot follows this pattern:

```
SENSE → THINK → ACT
```

In more detail:

```
┌─────────────────────────────────────────────────────────┐
│                      THE ROBOT                          │
│                                                         │
│  SENSORS          BRAIN              BODY               │
│  ────────         ─────              ────               │
│  Camera     →     Perception    →    Motor Controller   │
│  LiDAR      →     Planning      →    Wheels/Arms       │
│  IMU        →     Decision      →    Actuators          │
│  GPS        →                                           │
│                                                         │
│  All connected by a COMMUNICATION layer (like ROS)      │
└─────────────────────────────────────────────────────────┘
         |
         | (WiFi/Cloud)
         ↓
┌─────────────────────────────────────────────────────────┐
│                    THE CLOUD                            │
│                                                         │
│  Fleet Manager (which robot does what task)              │
│  Map Storage (store and update maps)                    │
│  Analytics (how well are robots performing)              │
│  Software Updates (push new code to robots)              │
└─────────────────────────────────────────────────────────┘
```

### The Pieces Explained Simply

**Perception** = "What do I see?"
- Camera sees an image → run object detection (YOLO) → "there's a person 3 meters ahead"
- LiDAR scans the room → creates a point cloud → "here's a map of obstacles"
- IMU reads acceleration/rotation → "I'm currently facing north and tilted 2 degrees"
- Combine all of these = **sensor fusion** → "I am HERE, and the world looks like THIS"

**Localization** = "Where am I?"
- The robot matches what it currently sees to its stored map
- Like you walking through a mall — you recognize the food court, so you know you're on the 2nd floor east side

**Planning** = "Where should I go?"
- **Global planner:** Finds the overall path from A to B (like Google Maps giving you a route)
- **Local planner:** Handles real-time stuff — "a person just walked in front of me, swerve left"

**Control** = "How do I move my wheels to follow the plan?"
- Takes the planned path and converts it into actual motor commands
- PID controller: "I want to go straight but I'm drifting left → turn wheels slightly right"

**Communication** = How all these pieces talk to each other
- In ROS: each piece is a separate program (called a "node")
- Nodes send messages to each other through "topics" (like a group chat)
- Camera node publishes images → perception node subscribes and processes them → planning node subscribes to perception results

### If They Ask: "Design a Floor Cleaning Robot System"

This is THE Brain Corp question. Here's how to answer it step by step:

**Step 1: Ask questions**
> "Is this for one robot or a fleet? What sensors does it have? Does it need to handle people walking around? Does it report back to a central system?"

**Step 2: Big picture**

```
Store Manager's Tablet
    |
    ↓ (assign task: "clean zone A")
Cloud Server / Fleet Manager
    |
    ↓ (send task to specific robot)
Robot
    ├── Camera + LiDAR → Perception (detect obstacles, people, spills)
    ├── SLAM → Localization (where am I on the store map?)
    ├── Planner → Path Planning (route through zone A avoiding obstacles)
    ├── Controller → Motor Commands (drive the wheels)
    └── Cleaning System → Actuators (brushes, water, squeegee)
    |
    ↓ (report status, battery, area cleaned)
Cloud Server → Dashboard for Store Manager
```

**Step 3: Go deeper (pick one area)**

Let's say they ask about the planning part:

> "The global planner uses A* on the store's occupancy grid to find a path through zone A that covers all the floor space. It's like a lawn mowing pattern — back and forth in rows to cover the whole area.
>
> The local planner runs at high frequency — maybe 10-20 Hz — and handles dynamic obstacles. If a customer walks in front of the robot, the local planner detects them through the perception system, updates the local costmap, and either swerves around them or stops and waits.
>
> For the coverage pattern, I'd use a boustrophedon (back-and-forth) decomposition — divide the zone into cells, plan a path that covers each cell, then stitch them together."

**Step 4: Trade-offs**

> "I'm running perception and planning on the robot's edge hardware, which means we're limited by compute power. The trade-off is between how sophisticated the models can be and how fast they need to run. A bigger neural network gives better detection but might not hit 30 FPS on a Jetson. So we'd use a smaller model and optimize with pruning and mixed-precision — which is actually what I did in my research with YOLO on Jetson Nano."

See how that naturally connects back to your experience? That's the goal.

---

## COMMON DESIGN QUESTIONS AT INTERN LEVEL

These are simpler than senior-level questions. They test basic thinking, not expertise.

### "Design a URL Shortener"

```
User gives long URL → Server generates short code → Stores mapping in database
User clicks short URL → Server looks up code → Redirects to long URL

Key pieces: Server, Database (URL mappings), Cache (popular links)
Key decisions: How to generate unique codes, handle duplicates, what if DB is down
```

### "Design a Chat App"

```
User A sends message → Server receives it → Stores in database → Pushes to User B

Key pieces: Server, Database (messages), WebSocket (real-time push to client)
Key decisions: Store messages how long? Group chats? Read receipts? Offline messages?
```

### "Design a Parking Lot System"

```
Car arrives → System checks for open spots → Assigns spot → Opens gate
Car leaves → System frees the spot → Calculates fee → Opens gate

Key pieces: Sensors (detect cars), Database (spot status), Payment system
Key decisions: How to assign spots? What if sensor fails? Multiple floors?
```

### "Design a Robot Fleet Manager" (Most likely for Brain Corp)

```
Store manager creates task → Fleet manager picks best robot → Sends task to robot
Robot does the work → Reports progress → Fleet manager updates dashboard

Key pieces: 
  - Database: robot status, tasks, maps, schedules
  - Task scheduler: which robot gets which task
  - Communication: server ↔ robot over WiFi
  - Dashboard: shows manager what's happening

Key decisions:
  - How to pick which robot? (closest? most battery? least busy?)
  - What if a robot gets stuck or dies mid-task?
  - What if WiFi drops?
  - How often does the robot report back? (every second? every minute?)
```

---

## WORDS TO USE IN YOUR ANSWER

These make you sound like you know what you're talking about, even if you just learned them:

| Instead of saying... | Say this... |
|---------------------|-------------|
| "Make it faster" | "We could add a cache layer to reduce latency" |
| "Use a bigger computer" | "Scale vertically, though horizontal scaling would be more robust" |
| "What if it breaks" | "We should eliminate that single point of failure with redundancy" |
| "Save it somewhere" | "Persist it in a database — SQL for structured data, NoSQL if we need flexible schema" |
| "Things talk to each other" | "Modules communicate through a publish-subscribe pattern" (like ROS topics) |
| "Handle lots of requests" | "We'd put a load balancer in front to distribute traffic" |
| "It's slow because too much data" | "We could add pagination so we're not loading everything at once" |
| "Keep a copy just in case" | "Add replicas for redundancy and failover" |

---

## THE ONE-PAGE CHEAT SHEET

```
HOW TO ANSWER:
1. Ask questions (scope, scale, constraints)
2. Draw big picture (boxes and arrows)
3. Deep dive (pick one area, show depth)
4. Trade-offs (every choice has a downside, mention it)

BUILDING BLOCKS:
  Client    → asks for stuff
  Server    → does the work
  Database  → stores stuff (SQL = structured, NoSQL = flexible)
  Cache     → shortcut for frequent reads
  Queue     → waiting line for tasks
  Load Balancer → spreads work across multiple servers

KEY CONCEPTS:
  Scalability  → can it handle more load? (vertical = bigger machine, horizontal = more machines)
  Latency      → how fast is one request
  Throughput   → how many requests per second
  Availability → what % of time is it working
  Redundancy   → backups so one failure doesn't kill everything
  Trade-off    → every choice has a cost, always mention it

FOR ROBOT QUESTIONS:
  SENSE → THINK → ACT
  Sensors → Perception → Planning → Control → Actuators
  All connected by communication layer (ROS topics)
  Cloud layer on top for fleet management, maps, analytics
```

---

## WHAT TO DO TONIGHT

1. **Read this doc once.** Don't memorize. Just let the concepts sink in.
2. **Practice the pizza shop exercise** — pick any business (library, parking lot, coffee shop) and design it on paper. What are the pieces? How do they connect? What breaks?
3. **Draw the robot architecture** from memory on a blank piece of paper. Sensors → Perception → Planning → Control → Actuators. Add the cloud layer. Label everything.
4. **Practice saying trade-offs out loud.** Every time you make a design choice, say "the trade-off is..." This is the #1 thing that impresses interviewers.

You don't need to be a system design expert. You need to show you can **think logically about how pieces fit together.** You already do this in your robotics lab every day — this doc just gives you the words to describe it.
