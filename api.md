# API Basics — From Zero to Interview Ready
### No bullshit, no jargon soup. Just what you need to know by Thursday.

---

## WHAT EVEN IS AN API?

Forget the textbook definition. Think of it like this:

You go to a restaurant. You don't walk into the kitchen and cook your own food. Instead:
1. You (the **client**) look at the **menu** (the API documentation)
2. You tell the **waiter** (the API) what you want
3. The waiter goes to the **kitchen** (the server/database) and gets your food
4. The waiter brings it back to you in a **plate** (the response format — usually JSON)

That's it. An API is a waiter between whoever is asking for data and wherever that data lives.

---

## WHAT IS REST?

REST is just a set of rules for how the waiter should behave. The main rules:

1. **Use URLs to identify things** — each "thing" (resource) gets its own address
2. **Use HTTP methods to say what you want to do** — don't put the action in the URL

That's basically it. Here's what it looks like:

```
I want to see all robots     →  GET    /robots
I want to see robot #5       →  GET    /robots/5
I want to add a new robot    →  POST   /robots        (send new robot data in the body)
I want to update robot #5    →  PUT    /robots/5      (send updated data in the body)
I want to delete robot #5    →  DELETE /robots/5
```

The URL says WHAT thing. The HTTP method says WHAT ACTION. That's REST.

**Bad (not RESTful):**
```
GET /getRobot?id=5
GET /deleteRobot?id=5
GET /createNewRobot
```
This is bad because the action is in the URL. REST says: let the HTTP method handle the action.

---

## WHAT IS JSON?

JSON is just how data is formatted when it travels between client and server. Think of it as a Python dictionary that got turned into text:

```json
{
  "name": "Ayush",
  "age": 24,
  "skills": ["Python", "OpenCV", "PyTorch"],
  "employed": false
}
```

That's it. Curly braces, key-value pairs, strings in quotes. Python dicts and JSON are almost identical — Python even has a built-in `json` module to convert between them.

---

## HTTP STATUS CODES — Just Remember These

Think of them as the waiter's response:

| Code | Plain English | When |
|------|--------------|------|
| **200** | "Here you go" | Everything worked |
| **201** | "Done, I made it for you" | You created something new (POST) |
| **400** | "I don't understand your order" | You sent bad/missing data |
| **404** | "We don't have that" | The thing you asked for doesn't exist |
| **500** | "Kitchen is on fire" | Server crashed |

That's it. You don't need to memorize 50 codes. These 5 cover 90% of interviews.

---

## YOUR FIRST API IN PYTHON — Flask

Flask is the simplest way to build an API in Python. Here's the absolute minimum:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/hello")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

Run this → go to `http://localhost:5000/hello` in your browser → you see "Hello World!"

That's a working API. One file. 8 lines. Let's build on this.

---

## BUILDING UP STEP BY STEP

### Step 1: Return JSON Instead of Plain Text

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/robots")
def get_robots():
    robots = [
        {"id": 1, "name": "CleanBot-A", "status": "active"},
        {"id": 2, "name": "CleanBot-B", "status": "charging"},
    ]
    return jsonify(robots)
```

`jsonify()` converts your Python list/dict into a proper JSON response. That's all it does.

### Step 2: Get a Specific Item by ID

```python
@app.route("/robots/<int:robot_id>")
def get_robot(robot_id):
    robots = [
        {"id": 1, "name": "CleanBot-A", "status": "active"},
        {"id": 2, "name": "CleanBot-B", "status": "charging"},
    ]
    
    # Find the robot with matching id
    for robot in robots:
        if robot["id"] == robot_id:
            return jsonify(robot)
    
    # If we get here, robot wasn't found
    return jsonify({"error": "Robot not found"}), 404
```

The `<int:robot_id>` part means: grab whatever number is in the URL and pass it to the function.

So `GET /robots/1` → `robot_id = 1` → returns CleanBot-A
And `GET /robots/99` → not found → returns 404

### Step 3: Filter with Query Parameters

```python
from flask import Flask, jsonify, request

@app.route("/robots")
def get_robots():
    robots = [
        {"id": 1, "name": "CleanBot-A", "status": "active"},
        {"id": 2, "name": "CleanBot-B", "status": "charging"},
        {"id": 3, "name": "CleanBot-C", "status": "active"},
    ]
    
    # Check if user passed ?status=something
    status_filter = request.args.get("status")
    
    if status_filter:
        robots = [r for r in robots if r["status"] == status_filter]
    
    return jsonify(robots)
```

Now:
- `GET /robots` → returns all 3 robots
- `GET /robots?status=active` → returns only CleanBot-A and CleanBot-C
- `GET /robots?status=charging` → returns only CleanBot-B

`request.args.get("status")` grabs the value after `?status=` in the URL. If there's no `?status=`, it returns `None`.

### Step 4: Create Something New (POST)

```python
@app.route("/robots", methods=["POST"])
def create_robot():
    # Get the JSON data the client sent
    data = request.get_json()
    
    # Basic validation — did they send the required fields?
    if not data or "name" not in data:
        return jsonify({"error": "name is required"}), 400
    
    new_robot = {
        "id": 4,  # In real life this would auto-increment
        "name": data["name"],
        "status": data.get("status", "inactive"),  # Default to inactive
    }
    
    # In real life you'd save this to a database
    # robots.append(new_robot)
    
    return jsonify(new_robot), 201  # 201 = Created
```

The client sends a POST request with JSON in the body:
```json
{"name": "CleanBot-D", "status": "active"}
```
And gets back the created robot with a 201 status code.

---

## NOW THE BRAIN CORP QUESTION: Parsing /etc/passwd

Here's the actual question they've asked, broken down for someone who's never done this before.

### What is /etc/passwd?

It's just a text file on every Linux machine that lists all users. Each line looks like:

```
root:x:0:0:root:/root:/bin/bash
ayush:x:1000:1000:Ayush Luhar:/home/ayush:/bin/bash
```

Fields separated by colons:
```
username : password(always x) : user_id : group_id : comment : home_directory : shell
```

### What is /etc/group?

Same idea but for groups:

```
sudo:x:27:ayush
docker:x:999:ayush,deploy
```

Fields:
```
group_name : password(always x) : group_id : comma_separated_members
```

### The Full Solution — With Comments Explaining Everything

```python
from flask import Flask, jsonify, request

app = Flask(__name__)


# ============================================
# STEP 1: PARSING FUNCTIONS
# These read the files and turn them into
# Python lists of dictionaries
# ============================================

def parse_passwd(filepath="/etc/passwd"):
    """
    Read /etc/passwd and return a list of user dicts.
    
    Each line looks like:
        root:x:0:0:root:/root:/bin/bash
    
    We split by ":" and grab the fields we care about.
    """
    users = []
    
    with open(filepath, "r") as f:
        for line in f:
            line = line.strip()           # Remove whitespace/newlines
            
            if not line:                  # Skip empty lines
                continue
            if line.startswith("#"):      # Skip comments
                continue
            
            parts = line.split(":")       # Split by colon
            
            if len(parts) < 7:            # Need at least 7 fields
                continue                  # Skip malformed lines
            
            user = {
                "name":    parts[0],
                "uid":     int(parts[2]),  # Convert string to int
                "gid":     int(parts[3]),
                "comment": parts[4],
                "home":    parts[5],
                "shell":   parts[6],
            }
            users.append(user)
    
    return users


def parse_group(filepath="/etc/group"):
    """
    Read /etc/group and return a list of group dicts.
    
    Each line looks like:
        docker:x:999:ayush,deploy
    """
    groups = []
    
    with open(filepath, "r") as f:
        for line in f:
            line = line.strip()
            
            if not line or line.startswith("#"):
                continue
            
            parts = line.split(":")
            
            if len(parts) < 4:
                continue
            
            # Members are comma-separated, could be empty string
            member_string = parts[3]
            if member_string:
                members = member_string.split(",")
            else:
                members = []
            
            group = {
                "name":    parts[0],
                "gid":     int(parts[2]),
                "members": members,
            }
            groups.append(group)
    
    return groups


# ============================================
# STEP 2: LOAD THE DATA
# Parse once at startup, serve from memory
# ============================================

USERS = parse_passwd()
GROUPS = parse_group()


# ============================================
# STEP 3: API ROUTES
# ============================================

# --- Users ---

@app.route("/users", methods=["GET"])
def get_users():
    """Get all users. Optionally filter by name with ?name=ayush"""
    name = request.args.get("name")
    if name:
        result = [u for u in USERS if u["name"] == name]
        return jsonify(result)
    return jsonify(USERS)


@app.route("/users/<int:uid>", methods=["GET"])
def get_user(uid):
    """Get a single user by their UID."""
    for user in USERS:
        if user["uid"] == uid:
            return jsonify(user)
    return jsonify({"error": f"User with UID {uid} not found"}), 404


@app.route("/users/<int:uid>/groups", methods=["GET"])
def get_user_groups(uid):
    """Get all groups a user belongs to."""
    # First find the user
    user = None
    for u in USERS:
        if u["uid"] == uid:
            user = u
            break
    
    if not user:
        return jsonify({"error": f"User with UID {uid} not found"}), 404
    
    # Find groups where this user is listed as a member
    # OR where the group's GID matches the user's primary GID
    result = []
    for group in GROUPS:
        if user["name"] in group["members"]:
            result.append(group)
        elif group["gid"] == user["gid"]:
            result.append(group)
    
    return jsonify(result)


# --- Groups ---

@app.route("/groups", methods=["GET"])
def get_groups():
    """Get all groups. Optionally filter by name with ?name=sudo"""
    name = request.args.get("name")
    if name:
        result = [g for g in GROUPS if g["name"] == name]
        return jsonify(result)
    return jsonify(GROUPS)


@app.route("/groups/<int:gid>", methods=["GET"])
def get_group(gid):
    """Get a single group by GID."""
    for group in GROUPS:
        if group["gid"] == gid:
            return jsonify(group)
    return jsonify({"error": f"Group with GID {gid} not found"}), 404


# ============================================
# STEP 4: RUN IT
# ============================================

if __name__ == "__main__":
    app.run(debug=True, port=5000)
```

### How to Run It

```bash
# Install Flask (one command)
pip install flask

# Run the app
python app.py

# Test it (in another terminal)
curl http://localhost:5000/users
curl http://localhost:5000/users/0
curl http://localhost:5000/users?name=root
curl http://localhost:5000/groups
curl http://localhost:5000/users/0/groups
```

### Simple Tests

```python
# test_app.py
import pytest
from app import parse_passwd, parse_group

def test_parse_passwd_reads_users(tmp_path):
    """Create a fake passwd file and check we parse it right."""
    
    # Create a temporary file with known content
    fake_file = tmp_path / "passwd"
    fake_file.write_text(
        "root:x:0:0:root:/root:/bin/bash\n"
        "ayush:x:1000:1000:Ayush:/home/ayush:/bin/bash\n"
    )
    
    users = parse_passwd(str(fake_file))
    
    assert len(users) == 2
    assert users[0]["name"] == "root"
    assert users[0]["uid"] == 0
    assert users[1]["name"] == "ayush"
    assert users[1]["uid"] == 1000


def test_parse_passwd_skips_empty_lines(tmp_path):
    fake_file = tmp_path / "passwd"
    fake_file.write_text(
        "root:x:0:0:root:/root:/bin/bash\n"
        "\n"
        "ayush:x:1000:1000:Ayush:/home/ayush:/bin/bash\n"
    )
    
    users = parse_passwd(str(fake_file))
    assert len(users) == 2  # Empty line was skipped


def test_parse_passwd_skips_comments(tmp_path):
    fake_file = tmp_path / "passwd"
    fake_file.write_text(
        "# This is a comment\n"
        "root:x:0:0:root:/root:/bin/bash\n"
    )
    
    users = parse_passwd(str(fake_file))
    assert len(users) == 1  # Comment was skipped


def test_parse_group_reads_members(tmp_path):
    fake_file = tmp_path / "group"
    fake_file.write_text(
        "docker:x:999:ayush,deploy\n"
        "root:x:0:\n"
    )
    
    groups = parse_group(str(fake_file))
    
    assert len(groups) == 2
    assert groups[0]["members"] == ["ayush", "deploy"]
    assert groups[1]["members"] == []  # root has no extra members


def test_parse_empty_file(tmp_path):
    fake_file = tmp_path / "empty"
    fake_file.write_text("")
    
    users = parse_passwd(str(fake_file))
    assert users == []


# Run with: pytest test_app.py -v
```

---

## THINGS TO SAY DURING THE INTERVIEW

When you're building this, **talk out loud**. Here's what to say at each step:

**When starting:**
> "Let me first understand the data format. Each line in /etc/passwd has fields separated by colons — username, password placeholder, UID, GID, comment, home dir, and shell."

**When writing the parser:**
> "I'm separating the parsing logic into its own function so I can test it independently from the API routes."

**When handling edge cases:**
> "I should handle empty lines and comments — real files often have these. And if a line is malformed with fewer fields than expected, I'll skip it rather than crashing."

**When designing endpoints:**
> "I'll make users and groups the two main resources, with nested routes for relationships like 'get all groups for a user.'"

**When returning errors:**
> "If someone asks for a user that doesn't exist, I'll return a 404 with a message explaining what wasn't found."

**When they ask "how would you improve this?":**
> "For a production system, I'd add pagination so we're not returning thousands of users at once, caching so we're not re-reading the file on every request, and input validation to protect against bad data."

---

## CHEAT SHEET — Fit on One Screen

```
REST = URLs identify THINGS, HTTP methods identify ACTIONS

GET    = Read       → 200 OK
POST   = Create     → 201 Created
PUT    = Replace    → 200 OK
DELETE = Remove     → 204 No Content

Errors:
  400 = Bad input
  404 = Not found
  500 = Server broke

Flask basics:
  from flask import Flask, jsonify, request
  app = Flask(__name__)
  
  @app.route("/things")              → GET all
  @app.route("/things/<int:id>")     → GET one
  @app.route("/things", methods=["POST"])  → Create
  
  request.args.get("key")  → query params (?key=value)
  request.get_json()       → POST body data
  jsonify(data)            → convert dict/list to JSON response
  return jsonify(error), 404  → return with status code

Testing:
  pytest + tmp_path fixture for fake files
  Test the parser separately from the routes
  Test: normal case, empty file, missing data, not found
```

---

## TONIGHT'S PRACTICE PLAN

1. **Install Flask:** `pip install flask` (one command, done)
2. **Type out the robot example** (the Step 1-4 section above) by hand — don't copy paste. Run it. Hit the endpoints with `curl` or your browser.
3. **Type out the /etc/passwd solution** by hand. Run it. It will actually work on your Linux machine (or WSL).
4. **Write 3 tests** without looking at the examples above.
5. **Do it again from scratch** tomorrow. Aim to write the whole thing in 25 minutes.

You don't need to memorize Flask docs. You need to be comfortable with the pattern: **parse data → store in memory → expose through routes → handle errors → test it.** That pattern is the same whether it's /etc/passwd, robot data, or sensor logs.
