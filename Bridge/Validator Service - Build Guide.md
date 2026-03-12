# Bridge Validator Service: Complete Build Guide

**Project:** Security sandbox for validating npm packages before installation  

---

# 📋 Table of Contents

1. Introduction: What You're Building & Why It Matters
2. Understanding Package Registries (The Basics)
3. The Problem: Supply Chain Attacks
4. The Solution: Bridge Validator Service
5. Prerequisites & Learning Resources
6. System Architecture
7. Development Roadmap
8. Phase 1: Basic Sandbox (Weeks 1-4)
9. Phase 2: Monitoring & Analysis (Weeks 5-8)
10. Phase 3: API & Integration (Weeks 9-12)
11. Phase 4: Polish & Launch (Weeks 13-16)
12. Testing & Debugging
13. Deployment Guide
14. Troubleshooting Common Issues
15. Next Steps After MVP

---

# 🎯 Introduction: What You're Building & Why It Matters

Hey! You're about to build something really cool that solves a genuine problem that costs companies millions of dollars. Let me explain what this is and why Connor needs it for Bridge.

## What is Bridge?

Bridge is a technical debt management platform with two main parts:

1. **Bridge-Desktop**: An Electron app engineers use to automate dependency updates, remove dead code, etc.
2. **Bridge-Console**: A web dashboard that shows leadership metrics about tech debt

## What You're Building

You're building the **third piece**: A security validation service that checks if npm packages are safe BEFORE they get installed on developers' machines.

**Think of it like this:**

- TSA security checkpoint at the airport (your service) vs. letting everyone board the plane without screening (current state)
- Virus scanner for your computer (your service) vs. just hoping downloaded files aren't malicious (npm audit)

## Why This Matters

> ⚠️ **The scary reality**: When you run `npm install some-package`, that package's code **executes immediately** on your computer. If it's malicious, it can:
> 
> - Steal your environment variables (AWS keys, API tokens, passwords)
> - Install backdoors
> - Mine cryptocurrency
> - Exfiltrate source code
> - And you won't know until it's too late

**This actually happens:**

- 2018: `event-stream` package stole Bitcoin wallet credentials from thousands of developers
- 2021: `ua-parser-js` package installed crypto miners
- 2022: `node-ipc` package sabotaged systems with Russian/Belarusian IPs

**What you're building prevents this** by running package installations in a secure sandbox BEFORE they touch real developer machines.

---

# 📚 Understanding Package Registries (The Basics)

Since CS programs don't really teach this, let's start from scratch.

## What is a Package Manager?

When you write code, you don't write everything from scratch. You use libraries/packages that other people wrote.

**Example**: Building a website that needs to handle dates

- ❌ Bad: Write your own date library (reinventing the wheel)
- ✅ Good: Use `date-fns` or `moment.js` libraries

**Package managers** help you download and manage these libraries:

- **npm** (Node Package Manager) - for JavaScript
- **pip** - for Python
- **maven** - for Java
- **cargo** - for Rust

## What is a Registry?

A **registry** is basically a giant library/warehouse of packages.

**The main npm registry** is at `registry.npmjs.org`

- Contains ~2.5 million packages
- Anyone can publish packages
- Anyone can download packages

**When you run `npm install react`:**

```
Your computer
  ↓ asks "where is react?"
npm registry (registry.npmjs.org)
  ↓ responds "here's react version 18.2.0"
Your computer
  ↓ downloads react
  ↓ installs it in node_modules folder
```

## What is a PRIVATE Registry?

Companies often want their own private registry for:

1. **Internal packages** (code they don't want public)
2. **Security** (control what packages developers can use)
3. **Speed** (cache packages locally)

**Examples of private registries:**

- Artifactory (by JFrog)
- Verdaccio (open source)
- GitHub Packages
- AWS CodeArtifact

## What You're Building: A Validation Registry

You're building a **security-focused validation service** that sits BETWEEN the developer and the public npm registry:

```
Developer runs: npm install some-package
  ↓
Bridge-Desktop intercepts
  ↓ asks "is this safe?"
Bridge Validator Service (YOUR PROJECT)
  ↓ installs package in isolated sandbox
  ↓ monitors for suspicious behavior
  ↓ returns: ✅ Safe or ⚠️ Suspicious
Bridge-Desktop
  ↓ shows result to developer
Developer decides: install anyway or investigate
```

**Key difference from a traditional registry:**

- Traditional registry: Just stores/serves packages
- Your service: **Tests packages for security issues** before giving the green light

---

# 🚨 The Problem: Supply Chain Attacks

## What's a Supply Chain Attack?

**Analogy**: You order food from DoorDash. The restaurant is legit, but the delivery driver poisons it. You trusted the chain, but one link was compromised.

**In software:**

```
Your app depends on package A
  ↓ which depends on package B
    ↓ which depends on package C (MALICIOUS)
      ↓ which steals your secrets
```

You only installed A, but C snuck in.

## Why It's Getting Worse

1. **Dependency depth**: Average app has 100+ dependencies, many you've never heard of
2. **Maintainer burnout**: Open source maintainers quit, attackers take over their packages
3. **Typosquatting**: Attackers publish `reactt` (note the extra 't') hoping for typos
4. **Automated attacks**: Bots scan GitHub for leaked API keys in your commits

## Real-World Examples

### event-stream (2018)

- Popular npm package (2M downloads/week)
- Maintainer gave access to "helpful contributor"
- Contributor added malicious code in version 3.3.6
- Code stole Bitcoin wallet credentials
- Went undetected for 3 months

### ua-parser-js (2021)

- 8M downloads/week
- Attacker compromised maintainer's npm account
- Published versions 0.7.29, 0.8.0, 1.0.0 with crypto miners
- Infected thousands of CI/CD pipelines

### codecov bash uploader (2021)

- Attackers modified Codecov's bash script
- Script exfiltrated environment variables from CI/CD
- Affected: Hashicorp, Twilio, Monday.com
- Cost: Millions in incident response

## What Doesn't Work

### npm audit

- ❌ Only catches KNOWN vulnerabilities in CVE database
- ❌ Zero-day attacks slip through
- ❌ Doesn't monitor install-time behavior
- ❌ Too many false positives (developers ignore it)

### Manual code review

- ❌ Can't review every dependency
- ❌ Can't review transitive dependencies (dependencies of dependencies)
- ❌ Doesn't scale

### "Just don't use untrusted packages"

- ❌ Impossible to know what's trusted
- ❌ Trusted packages get compromised
- ❌ Transitive dependencies are out of your control

## The Gap You're Filling

**What's needed**: Automated, behavioral analysis during package installation

- ✅ Runs in sandbox (no risk to real machines)
- ✅ Monitors actual behavior (network, file system, processes)
- ✅ Fast (cached results)
- ✅ Integrated into developer workflow

**That's what you're building.**

---

# 💡 The Solution: Bridge Validator Service

## High-Level Overview

You're building a **SaaS API service** that:

1. Receives validation requests from Bridge-Desktop
2. Spins up isolated Docker containers
3. Installs the requested package in the container
4. Monitors what the package does during installation
5. Analyzes behavior for red flags
6. Returns validation result
7. Caches results for future requests

## User Flow

**From a developer's perspective:**

```
Developer: "I want to update React from 18 to 19"
  ↓
Bridge-Desktop: "Let me check if React 19 is safe..."
  ↓ [sends request to your service]
Your Service: 
  - Spins up fresh Ubuntu container
  - Installs React 19 while monitoring
  - Checks: Did it make network calls? Access weird files? Spawn unexpected processes?
  - Analysis: ✅ No red flags detected
  ↓ [returns validation result]
Bridge-Desktop: "✅ React 19 passed security validation. Safe to install."
  ↓
Developer: Proceeds with update confidently
```

**If something suspicious happens:**

```
Developer: "I want to install some-sketchy-package"
  ↓
Your Service:
  - Installs package in sandbox
  - Detects: Package made HTTP POST to suspicious-domain.ru during install
  - Detects: Package tried to access ~/.ssh/id_rsa (SSH keys)
  - Analysis: ⚠️ SUSPICIOUS BEHAVIOR DETECTED
  ↓
Bridge-Desktop: "⚠️ Warning: some-sketchy-package attempted to:
                  - Contact suspicious-domain.ru
                  - Access SSH keys
                  Recommendation: Do NOT install"
  ↓
Developer: Investigates or chooses different package
```

## What Makes This Valuable

1. **Proactive, not reactive**: Catches attacks BEFORE installation
2. **Behavioral, not signature-based**: Catches zero-days
3. **Automated**: No manual review needed
4. **Fast**: Cached results = instant validation for popular packages
5. **Developer-friendly**: Integrated into existing workflow (Bridge-Desktop)

## Why Companies Will Pay

**The math:**

- Average security breach: $4.45 million
- Average time to detect: 287 days
- Bridge Validator: $100/month per engineer
- ROI: Astronomical if it prevents even one breach

**Compliance:**

- SOC 2 audits ask: "How do you vet dependencies?"
- Answer: "We use Bridge Validator" ✅
- vs. "We trust npm" ❌

---

# 📖 Prerequisites & Learning Resources

## What You Need to Know (Before Starting)

**Required:**

- ✅ JavaScript/Node.js basics
- ✅ Command line / terminal comfort
- ✅ Git basics
- ✅ Basic understanding of APIs (REST)

**Helpful but not required:**

- Docker basics (you'll learn as you go)
- Linux commands
- Networking fundamentals
- Security concepts

## What You'll Learn (During This Project)

- Docker containerization
- Process monitoring (strace, tcpdump)
- API design and development
- Caching strategies (Redis)
- Security analysis
- Production deployment
- And a ton more

## Learning Resources

### Docker Fundamentals

**Start here if you've never used Docker:**

1. **Docker Official Tutorial** (2 hours)
    
    - https://www.docker.com/101-tutorial
    - Hands-on, interactive
    - Free
2. **freeCodeCamp Docker Course** (2 hours)
    
    - https://www.youtube.com/watch?v=fqMOX6JJhGo
    - Video format
    - Covers everything you need

**Key concepts to understand:**

- What is a container? (lightweight VM)
- Dockerfile (recipe for building container)
- Images vs. Containers
- Mounting volumes
- Networking between containers

### Node.js API Development

**If you need a refresher:**

1. **Building REST APIs with Express** (3 hours)
    
    - https://www.youtube.com/watch?v=pKd0Rpw7O48
    - Practical tutorial
2. **Express.js Documentation**
    
    - https://expressjs.com/
    - Official docs are actually pretty good

### Linux Process Monitoring

**You'll need this for behavioral analysis:**

1. **strace tutorial** (1 hour)
    
    - https://jvns.ca/blog/2021/04/03/what-problems-do-people-solve-with-strace/
    - Julia Evans is amazing at explaining things
2. **tcpdump basics** (30 min)
    
    - https://www.tcpdump.org/manpages/tcpdump.1.html
    - For network monitoring

### Redis Caching

**For making your service fast:**

1. **Redis Crash Course** (30 min)
    
    - https://www.youtube.com/watch?v=jgpVdJB2sKQ
2. **Redis in Node.js** (1 hour)
    
    - https://redis.io/docs/clients/nodejs/

## Recommended Timeline for Learning

**Week 0 (Before starting to build):**

- 3-4 hours: Docker fundamentals
- 2 hours: Express.js refresher
- 1 hour: strace/tcpdump basics

**Total: ~6 hours of prep work**

> 💡 Don't try to learn everything upfront. Learn as you build.

---

# 🏗️ System Architecture

## Components Overview

```
┌─────────────────────────────────────────────────────┐
│                 Bridge-Desktop                      │
│            (Connor's working on this)               │
└────────────────────┬────────────────────────────────┘
                     │
                     │ HTTP POST /validate
                     │
┌────────────────────▼────────────────────────────────┐
│           Bridge Validator API                      │
│                                                     │
│  ┌──────────────┐      ┌──────────────┐           │
│  │   Express    │◄────►│    Redis     │           │
│  │     API      │      │    Cache     │           │
│  └──────┬───────┘      └──────────────┘           │
│         │                                          │
│         │ spawn container                          │
│         ▼                                          │
│  ┌──────────────┐                                 │
│  │   Docker     │                                 │
│  │   Worker     │                                 │
│  └──────────────┘                                 │
└─────────────────────────────────────────────────────┘
         │
         │ analyze behavior
         ▼
┌─────────────────────────────────────────────────────┐
│           Behavior Analyzer                         │
│  - Network activity monitor                         │
│  - File system access monitor                       │
│  - Process spawn monitor                            │
│  - Heuristic rules engine                           │
└─────────────────────────────────────────────────────┘
```

## Directory Structure

```
bridge-validator/
├── api/                    # Express API server
│   ├── server.js          # Main server file
│   ├── routes/
│   │   └── validate.js    # Validation endpoints
│   ├── controllers/
│   │   └── validator.js   # Business logic
│   └── middleware/
│       ├── auth.js        # API key authentication
│       └── ratelimit.js   # Rate limiting
├── worker/                 # Docker container worker
│   ├── Dockerfile         # Container definition
│   ├── install-monitor.sh # Installation script
│   └── analyze.js         # Behavior analysis
├── analyzer/               # Behavior analysis engine
│   ├── network.js         # Network activity analyzer
│   ├── filesystem.js      # File system analyzer
│   ├── process.js         # Process spawn analyzer
│   └── heuristics.js      # Security heuristics
├── cache/                  # Redis caching layer
│   └── cache.js
├── tests/                  # Test suite
│   ├── integration/
│   └── unit/
├── docs/                   # Documentation
│   └── api.md
├── docker-compose.yml      # Local development setup
├── package.json
└── README.md
```

## Data Flow

**Step-by-step validation flow:**

1. **Request arrives**

```json
POST /api/validate
{
  "package": "express",
  "version": "4.18.0",
  "package_manager": "npm"
}
```

2. **Check cache**

```javascript
const cacheKey = "npm:express@4.18.0";
const cached = await redis.get(cacheKey);
if (cached) return JSON.parse(cached); // ⚡ Instant response
```

3. **Spawn sandbox container**

```javascript
const container = await docker.run('bridge-worker', [
  'express',
  '4.18.0'
]);
```

4. **Monitor installation**
    
    - strace captures system calls (file access, network, process spawns)
    - tcpdump captures network packets
    - Logs saved to container volume
5. **Analyze behavior**
    

```javascript
const analysis = await analyzeBehavior({
  networkLog: '/tmp/network.pcap',
  syscallLog: '/tmp/strace.log'
});
```

6. **Generate report**

```javascript
const result = {
  safe: analysis.criticalFlags === 0,
  flags: analysis.flags,
  score: calculateRiskScore(analysis)
};
```

7. **Cache result**

```javascript
await redis.setex(cacheKey, 2592000, JSON.stringify(result)); // 30 days
```

8. **Return to client**

```json
{
  "safe": true,
  "validation_id": "uuid",
  "score": 95,
  "flags": [],
  "cached": false
}
```

## Technology Stack

**API Server:**

- Node.js 20.x
- Express 4.x
- Docker SDK for Node

**Worker Container:**

- Ubuntu 22.04 (minimal)
- Node.js 20.x
- strace (system call monitoring)
- tcpdump (network monitoring)

**Cache:**

- Redis 7.x

**Deployment** (MVP):

- Railway or Fly.io
- Docker Compose for local dev

---

# 🗓️ Development Roadmap

## Overview

**Total timeline**: 12-16 weeks

**Milestones:**

- Week 4: Basic sandbox working
- Week 8: Monitoring & analysis complete
- Week 12: API integration with Bridge-Desktop
- Week 16: MVP launched

## Week-by-Week Breakdown

|Week|Focus|Deliverable|
|---|---|---|
|1|Docker basics, environment setup|Hello World container|
|2|Package installation in container|Can install npm packages in sandbox|
|3|Basic monitoring (strace)|Can capture system calls|
|4|Network monitoring (tcpdump)|Can capture network activity|
|5|Log parsing|Can parse strace/tcpdump output|
|6|Heuristic rules engine|Can detect suspicious patterns|
|7|Risk scoring algorithm|Quantitative risk assessment|
|8|Testing with known malicious packages|Validation that detection works|
|9|Express API setup|Basic POST /validate endpoint|
|10|Redis caching layer|Fast repeated validations|
|11|API authentication & rate limiting|Production-ready security|
|12|Bridge-Desktop integration|Connor tests end-to-end|
|13|Performance optimization|Handle 100 req/min|
|14|Documentation & error handling|Production polish|
|15|Deploy to Railway/Fly.io|Live on internet|
|16|Final testing & launch|MVP complete|

---

# 🔨 Phase 1: Basic Sandbox (Weeks 1-4)

## Week 1: Docker Fundamentals

**Goal**: Get comfortable with Docker, build your first container

### Tasks

**1. Install Docker**

```bash
# Mac
brew install --cask docker

# Linux
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Verify installation
docker --version
docker run hello-world
```

**2. Create your first Dockerfile**

Create `worker/Dockerfile`:

```dockerfile
# Start from official Node.js image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Create a simple script
RUN echo 'console.log("Hello from Docker!");' > hello.js

# Run the script
CMD ["node", "hello.js"]
```

Build and run:

```bash
cd worker
docker build -t bridge-worker .
docker run bridge-worker
```

You should see: `Hello from Docker!`

**3. Experiment with volumes**

Update `Dockerfile`:

```dockerfile
FROM node:20-alpine
WORKDIR /app

# Install npm globally
RUN npm install -g npm@latest

# Create a script that lists files
RUN echo 'const fs = require("fs"); console.log(fs.readdirSync("/data"));' > list.js

CMD ["node", "list.js"]
```

Run with volume mount:

```bash
docker build -t bridge-worker .
docker run -v $(pwd):/data bridge-worker
```

This mounts your current directory into the container at `/data`

**4. Learn container lifecycle**

```bash
# Run container in background
docker run -d --name test bridge-worker sleep 3600

# Execute commands in running container
docker exec test ls /app

# View logs
docker logs test

# Stop container
docker stop test

# Remove container
docker rm test
```

### Checkpoint

✅ You understand how to build Docker images, run containers, and mount volumes.

---

## Week 2: Package Installation in Container

**Goal**: Install npm packages inside a Docker container

### Tasks

**1. Create installation script**

Create `worker/install-package.sh`:

```bash
#!/bin/sh

# Get package name and version from arguments
PACKAGE=$1
VERSION=$2

echo "Installing ${PACKAGE}@${VERSION}..."

# Create temp directory for installation
mkdir -p /tmp/install
cd /tmp/install

# Initialize a package.json
npm init -y

# Install the package
npm install ${PACKAGE}@${VERSION}

# Check if installation succeeded
if [ $? -eq 0 ]; then
  echo "✅ Installation successful"
  exit 0
else
  echo "❌ Installation failed"
  exit 1
fi
```

Make it executable:

```bash
chmod +x worker/install-package.sh
```

**2. Update Dockerfile**

`worker/Dockerfile`:

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Copy installation script
COPY install-package.sh /app/

# Make it executable
RUN chmod +x /app/install-package.sh

# Set the script as entrypoint
ENTRYPOINT ["/app/install-package.sh"]
```

**3. Test installation**

```bash
docker build -t bridge-worker .

# Try installing express
docker run bridge-worker express 4.18.0

# Try installing a specific version of lodash
docker run bridge-worker lodash 4.17.21
```

You should see successful installations!

**4. Handle errors gracefully**

Update `install-package.sh`:

```bash
#!/bin/sh

PACKAGE=$1
VERSION=$2

# Validate inputs
if [ -z "$PACKAGE" ]; then
  echo "❌ Error: Package name required"
  echo "Usage: install-package.sh <package> <version>"
  exit 1
fi

if [ -z "$VERSION" ]; then
  echo "❌ Error: Version required"
  echo "Usage: install-package.sh <package> <version>"
  exit 1
fi

echo "Installing ${PACKAGE}@${VERSION}..."

mkdir -p /tmp/install
cd /tmp/install

npm init -y > /dev/null 2>&1

# Capture npm install output
if npm install ${PACKAGE}@${VERSION} 2>&1 | tee /tmp/install.log; then
  echo "✅ Installation successful"
  exit 0
else
  echo "❌ Installation failed"
  cat /tmp/install.log
  exit 1
fi
```

Test error handling:

```bash
# Should fail gracefully
docker run bridge-worker nonexistent-package 1.0.0
docker run bridge-worker express 999.999.999
```

### Checkpoint

✅ You can install any npm package at any version inside a Docker container.

---

## Week 3: Basic Monitoring (strace)

**Goal**: Capture what the package does during installation

### Tasks

**1. Install strace in container**

Update `Dockerfile`:

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install monitoring tools
RUN apk add --no-cache strace

COPY install-package.sh /app/
RUN chmod +x /app/install-package.sh

ENTRYPOINT ["/app/install-package.sh"]
```

**2. Update installation script to use strace**

`install-package.sh`:

```bash
#!/bin/sh

PACKAGE=$1
VERSION=$2

# Validate inputs
if [ -z "$PACKAGE" ] || [ -z "$VERSION" ]; then
  echo "Usage: install-package.sh <package> <version>"
  exit 1
fi

echo "Installing ${PACKAGE}@${VERSION} with monitoring..."

mkdir -p /tmp/install
cd /tmp/install

npm init -y > /dev/null 2>&1

# Run npm install under strace
# -f = follow forks (child processes)
# -e trace=... = what to trace
# -o = output file
strace \
  -f \
  -e trace=open,openat,connect,execve \
  -o /tmp/strace.log \
  npm install ${PACKAGE}@${VERSION} 2>&1 | tee /tmp/install.log

INSTALL_EXIT_CODE=$?

echo ""
echo "=== STRACE LOG ==="
head -n 20 /tmp/strace.log
echo "..."
echo ""

exit $INSTALL_EXIT_CODE
```

**3. Test monitoring**

```bash
docker build -t bridge-worker .
docker run bridge-worker express 4.18.0
```

You'll see strace output showing:

- Files being opened
- Network connections
- Processes being spawned

**4. Extract strace logs from container**

Run container with volume mount:

```bash
docker run -v $(pwd)/logs:/tmp bridge-worker express 4.18.0
```

Now check `logs/strace.log` - it contains all system calls!

**5. Understand strace output**

Example strace log line:

```
12345 openat(AT_FDCWD, "/etc/resolv.conf", O_RDONLY) = 3
```

Breaking it down:

- `12345` = Process ID
- `openat` = System call (opening a file)
- `AT_FDCWD` = Open relative to current directory
- `"/etc/resolv.conf"` = File being opened (DNS config)
- `O_RDONLY` = Read-only mode
- `= 3` = File descriptor returned

### Checkpoint

✅ You can monitor and log all system calls during package installation.

---

## Week 4: Network Monitoring (tcpdump)

**Goal**: Capture network activity during installation

### Tasks

**1. Install tcpdump**

Update `Dockerfile`:

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install monitoring tools
RUN apk add --no-cache strace tcpdump

COPY install-package.sh /app/
RUN chmod +x /app/install-package.sh

ENTRYPOINT ["/app/install-package.sh"]
```

**2. Capture network traffic**

Update `install-package.sh`:

```bash
#!/bin/sh

PACKAGE=$1
VERSION=$2

if [ -z "$PACKAGE" ] || [ -z "$VERSION" ]; then
  echo "Usage: install-package.sh <package> <version>"
  exit 1
fi

echo "Installing ${PACKAGE}@${VERSION} with full monitoring..."

mkdir -p /tmp/install
cd /tmp/install

npm init -y > /dev/null 2>&1

# Start network capture in background
tcpdump -i any -w /tmp/network.pcap > /dev/null 2>&1 &
TCPDUMP_PID=$!

echo "Network monitoring started (PID: $TCPDUMP_PID)"

# Run npm install under strace
strace \
  -f \
  -e trace=open,openat,connect,execve \
  -o /tmp/strace.log \
  npm install ${PACKAGE}@${VERSION} 2>&1 | tee /tmp/install.log

INSTALL_EXIT_CODE=$?

# Stop network capture
kill $TCPDUMP_PID
wait $TCPDUMP_PID 2>/dev/null

echo ""
echo "=== MONITORING COMPLETE ==="
echo "System calls logged to: /tmp/strace.log"
echo "Network traffic logged to: /tmp/network.pcap"
echo ""

exit $INSTALL_EXIT_CODE
```

**3. Run with network capabilities**

tcpdump needs special permissions:

```bash
docker build -t bridge-worker .

# Run with network admin capability
docker run --cap-add=NET_ADMIN -v $(pwd)/logs:/tmp bridge-worker express 4.18.0
```

**4. Analyze network capture**

Install Wireshark on your local machine:

```bash
brew install --cask wireshark  # Mac
```

Open `logs/network.pcap` in Wireshark to see:

- DNS queries (looking up registry.npmjs.org)
- HTTP/HTTPS connections
- IP addresses contacted

Or use command line:

```bash
# Read pcap file
tcpdump -r logs/network.pcap

# Show HTTP requests
tcpdump -r logs/network.pcap -A | grep -i "GET\|POST"
```

**5. Create summary script**

Create `worker/analyze-network.sh`:

```bash
#!/bin/sh

PCAP_FILE=$1

echo "=== NETWORK ANALYSIS ==="
echo ""

echo "DNS Queries:"
tcpdump -r $PCAP_FILE -n 'port 53' 2>/dev/null | head -n 10

echo ""
echo "Unique IP addresses contacted:"
tcpdump -r $PCAP_FILE -n | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' | sort -u

echo ""
echo "HTTP requests:"
tcpdump -r $PCAP_FILE -A 'tcp port 80' 2>/dev/null | grep -i "Host:" | head -n 10
```

### Checkpoint

✅ You can capture and analyze network activity during package installation.

---

# 🔍 Phase 2: Monitoring & Analysis (Weeks 5-8)

## Week 5: Log Parsing

**Goal**: Convert raw strace/tcpdump logs into structured data

### Tasks

**1. Create analysis script**

Create `worker/analyze.js`:

```javascript
const fs = require('fs');

function parseStraceLog(logPath) {
  const content = fs.readFileSync(logPath, 'utf-8');
  const lines = content.split('\n');
  
  const events = {
    files: [],
    network: [],
    processes: []
  };
  
  for (const line of lines) {
    // Parse file access
    const fileMatch = line.match(/open(?:at)?\([^"]*"([^"]+)"/);
    if (fileMatch) {
      events.files.push(fileMatch[1]);
    }
    
    // Parse network connections
    const connectMatch = line.match(/connect\(.*sin_addr=inet_addr\("([^"]+)"\)/);
    if (connectMatch) {
      events.network.push(connectMatch[1]);
    }
    
    // Parse process spawns
    const execMatch = line.match(/execve\("([^"]+)"/);
    if (execMatch) {
      events.processes.push(execMatch[1]);
    }
  }
  
  return events;
}

function parseNetworkLog(pcapPath) {
  // For now, just check if file exists
  // We'll use tcpdump to analyze it
  if (!fs.existsSync(pcapPath)) {
    return { domains: [], ips: [] };
  }
  
  // In a real implementation, you'd use tcpdump or a library
  // For MVP, we can shell out to tcpdump
  const { execSync } = require('child_process');
  
  try {
    // Extract unique IPs
    const ips = execSync(
      `tcpdump -r ${pcapPath} -n 2>/dev/null | grep -oE '([0-9]{1,3}\\.){3}[0-9]{1,3}' | sort -u`,
      { encoding: 'utf-8' }
    ).split('\n').filter(Boolean);
    
    return { domains: [], ips };
  } catch (error) {
    return { domains: [], ips: [] };
  }
}

function main() {
  const straceLog = process.argv[2] || '/tmp/strace.log';
  const networkLog = process.argv[3] || '/tmp/network.pcap';
  
  console.log('=== ANALYSIS RESULTS ===\n');
  
  const straceEvents = parseStraceLog(straceLog);
  console.log('Files accessed:', straceEvents.files.length);
  console.log('Sample files:', straceEvents.files.slice(0, 5));
  console.log('');
  
  console.log('Network connections:', straceEvents.network.length);
  console.log('Sample connections:', straceEvents.network.slice(0, 5));
  console.log('');
  
  console.log('Processes spawned:', straceEvents.processes.length);
  console.log('Sample processes:', straceEvents.processes.slice(0, 5));
  console.log('');
  
  const networkEvents = parseNetworkLog(networkLog);
  console.log('Unique IPs contacted:', networkEvents.ips.length);
  console.log('IPs:', networkEvents.ips);
}

main();
```

**2. Integrate into container**

Update `Dockerfile`:

```dockerfile
FROM node:20-alpine

WORKDIR /app

RUN apk add --no-cache strace tcpdump

COPY install-package.sh /app/
COPY analyze.js /app/

RUN chmod +x /app/install-package.sh

ENTRYPOINT ["/app/install-package.sh"]
```

Update `install-package.sh` to run analysis at the end:

```bash
# ... after tcpdump stops ...

echo ""
echo "=== RUNNING ANALYSIS ==="
node /app/analyze.js /tmp/strace.log /tmp/network.pcap
echo ""

exit $INSTALL_EXIT_CODE
```

**3. Test parsing**

```bash
docker build -t bridge-worker .
docker run --cap-add=NET_ADMIN bridge-worker express 4.18.0
```

You should see analysis output!

### Checkpoint

✅ You can parse raw monitoring logs into structured data.

---

## Week 6: Heuristic Rules Engine

**Goal**: Detect suspicious patterns in behavior

### Tasks

**1. Define suspicious patterns**

Create `worker/heuristics.js`:

```javascript
const SUSPICIOUS_PATTERNS = {
  files: [
    /\/etc\/passwd/,
    /\/etc\/shadow/,
    /\.ssh\/id_rsa/,
    /\.aws\/credentials/,
    /\.bash_history/,
    /\.zsh_history/
  ],
  
  ips: [
    // Suspicious TLDs
    /\.ru$/,
    /\.cn$/,
    // Known malicious ranges (example)
    /^185\.220\./  // Tor exit nodes
  ],
  
  domains: [
    /pastebin\.com/,
    /discord\.com\/api\/webhooks/,
    /ngrok\.io/,
    /requestbin\.com/
  ],
  
  processes: [
    /\/bin\/bash/,
    /\/bin\/sh/,
    /curl/,
    /wget/,
    /nc$/,  // netcat
    /python/  // Unexpected Python execution
  ]
};

const EXPECTED_PATTERNS = {
  ips: [
    /^104\.16\./,  // Cloudflare (npm registry uses this)
    /^151\.101\./  // Fastly (npm CDN)
  ],
  
  domains: [
    /npmjs\.org$/,
    /npmjs\.com$/,
    /yarnpkg\.com$/,
    /github\.com$/
  ],
  
  processes: [
    /node$/,
    /npm$/,
    /git$/
  ]
};

function isSuspicious(value, patterns) {
  return patterns.some(pattern => pattern.test(value));
}

function isExpected(value, patterns) {
  return patterns.some(pattern => pattern.test(value));
}

function analyzeFiles(files) {
  const flags = [];
  
  for (const file of files) {
    if (isSuspicious(file, SUSPICIOUS_PATTERNS.files)) {
      flags.push({
        severity: 'critical',
        type: 'suspicious_file_access',
        detail: `Accessed sensitive file: ${file}`
      });
    }
  }
  
  return flags;
}

function analyzeNetwork(ips, domains) {
  const flags = [];
  
  for (const ip of ips) {
    if (isExpected(ip, EXPECTED_PATTERNS.ips)) {
      continue; // Expected npm infrastructure
    }
    
    if (isSuspicious(ip, SUSPICIOUS_PATTERNS.ips)) {
      flags.push({
        severity: 'high',
        type: 'suspicious_network',
        detail: `Connected to suspicious IP: ${ip}`
      });
    }
  }
  
  for (const domain of domains) {
    if (isExpected(domain, EXPECTED_PATTERNS.domains)) {
      continue;
    }
    
    if (isSuspicious(domain, SUSPICIOUS_PATTERNS.domains)) {
      flags.push({
        severity: 'high',
        type: 'suspicious_network',
        detail: `Connected to suspicious domain: ${domain}`
      });
    }
  }
  
  return flags;
}

function analyzeProcesses(processes) {
  const flags = [];
  
  for (const proc of processes) {
    if (isExpected(proc, EXPECTED_PATTERNS.processes)) {
      continue;
    }
    
    if (isSuspicious(proc, SUSPICIOUS_PATTERNS.processes)) {
      flags.push({
        severity: 'medium',
        type: 'unexpected_process',
        detail: `Spawned unexpected process: ${proc}`
      });
    }
  }
  
  return flags;
}

module.exports = {
  analyzeFiles,
  analyzeNetwork,
  analyzeProcesses
};
```

**2. Integrate heuristics into analysis**

Update `analyze.js`:

```javascript
const fs = require('fs');
const heuristics = require('./heuristics');

// ... parseStraceLog and parseNetworkLog functions ...

function main() {
  const straceLog = process.argv[2] || '/tmp/strace.log';
  const networkLog = process.argv[3] || '/tmp/network.pcap';
  
  const straceEvents = parseStraceLog(straceLog);
  const networkEvents = parseNetworkLog(networkLog);
  
  // Run heuristic analysis
  const fileFlags = heuristics.analyzeFiles(straceEvents.files);
  const networkFlags = heuristics.analyzeNetwork(
    networkEvents.ips,
    networkEvents.domains
  );
  const processFlags = heuristics.analyzeProcesses(straceEvents.processes);
  
  const allFlags = [...fileFlags, ...networkFlags, ...processFlags];
  
  // Generate report
  console.log('=== SECURITY ANALYSIS ===\n');
  
  if (allFlags.length === 0) {
    console.log('✅ No suspicious behavior detected');
  } else {
    console.log(`⚠️  ${allFlags.length} potential security issues detected:\n`);
    
    allFlags.forEach((flag, i) => {
      console.log(`${i + 1}. [${flag.severity.toUpperCase()}] ${flag.type}`);
      console.log(`   ${flag.detail}\n`);
    });
  }
  
  // Output JSON for programmatic use
  const result = {
    safe: allFlags.filter(f => f.severity === 'critical' || f.severity === 'high').length === 0,
    flags: allFlags,
    events: {
      files: straceEvents.files.length,
      network_connections: networkEvents.ips.length,
      processes: straceEvents.processes.length
    }
  };
  
  fs.writeFileSync('/tmp/analysis.json', JSON.stringify(result, null, 2));
  console.log('\nFull results saved to /tmp/analysis.json');
}

main();
```

**3. Test with benign package**

```bash
docker run --cap-add=NET_ADMIN -v $(pwd)/logs:/tmp bridge-worker lodash 4.17.21

cat logs/analysis.json
# Should show: "safe": true
```

### Checkpoint

✅ You can detect suspicious behavior patterns.

---

## Week 7: Risk Scoring Algorithm

**Goal**: Quantify risk as a numeric score

### Tasks

**1. Create scoring system**

Create `worker/scoring.js`:

```javascript
const SEVERITY_WEIGHTS = {
  critical: 100,
  high: 50,
  medium: 20,
  low: 5
};

const TYPE_MULTIPLIERS = {
  suspicious_file_access: 2.0,
  suspicious_network: 1.5,
  unexpected_process: 1.0
};

function calculateRiskScore(flags) {
  if (flags.length === 0) {
    return 0;
  }
  
  let totalScore = 0;
  
  for (const flag of flags) {
    const baseScore = SEVERITY_WEIGHTS[flag.severity] || 0;
    const multiplier = TYPE_MULTIPLIERS[flag.type] || 1.0;
    const flagScore = baseScore * multiplier;
    
    totalScore += flagScore;
  }
  
  // Normalize to 0-100 scale
  // Max realistic score is ~500 (5 critical file access flags)
  const normalized = Math.min(100, (totalScore / 500) * 100);
  
  return Math.round(normalized);
}

function getRiskLevel(score) {
  if (score === 0) return 'safe';
  if (score < 20) return 'low';
  if (score < 50) return 'medium';
  if (score < 80) return 'high';
  return 'critical';
}

module.exports = {
  calculateRiskScore,
  getRiskLevel
};
```

**2. Add scoring to analysis**

Update `analyze.js`:

```javascript
const scoring = require('./scoring');

function main() {
  // ... existing code ...
  
  const allFlags = [...fileFlags, ...networkFlags, ...processFlags];
  
  const riskScore = scoring.calculateRiskScore(allFlags);
  const riskLevel = scoring.getRiskLevel(riskScore);
  
  console.log('=== SECURITY ANALYSIS ===\n');
  console.log(`Risk Score: ${riskScore}/100 (${riskLevel.toUpperCase()})\n`);
  
  if (allFlags.length === 0) {
    console.log('✅ No suspicious behavior detected');
  } else {
    // ... existing flag output ...
  }
  
  const result = {
    safe: riskLevel === 'safe' || riskLevel === 'low',
    risk_score: riskScore,
    risk_level: riskLevel,
    flags: allFlags,
    events: {
      files: straceEvents.files.length,
      network_connections: networkEvents.ips.length,
      processes: straceEvents.processes.length
    }
  };
  
  fs.writeFileSync('/tmp/analysis.json', JSON.stringify(result, null, 2));
}
```

### Checkpoint

✅ You have a quantitative risk assessment system.

---

## Week 8: Testing with Known Packages

**Goal**: Validate that your detection works

### Tasks

**1. Test with popular safe packages**

```bash
# These should all pass
docker run --cap-add=NET_ADMIN -v $(pwd)/logs:/tmp bridge-worker react 18.2.0
docker run --cap-add=NET_ADMIN -v $(pwd)/logs:/tmp bridge-worker express 4.18.0
docker run --cap-add=NET_ADMIN -v $(pwd)/logs:/tmp bridge-worker lodash 4.17.21

# Check results
cat logs/analysis.json
# All should show: "safe": true, "risk_score": 0
```

**2. Create test suite**

Create `tests/test-packages.js`:

```javascript
const { execSync } = require('child_process');
const fs = require('fs');

const TEST_CASES = [
  {
    name: 'react',
    version: '18.2.0',
    expectedSafe: true,
    expectedScore: 0
  },
  {
    name: 'express',
    version: '4.18.0',
    expectedSafe: true,
    expectedScore: 0
  },
  {
    name: 'lodash',
    version: '4.17.21',
    expectedSafe: true,
    expectedScore: 0
  }
];

function runTest(testCase) {
  console.log(`\nTesting ${testCase.name}@${testCase.version}...`);
  
  try {
    execSync(
      `docker run --cap-add=NET_ADMIN -v $(pwd)/logs:/tmp bridge-worker ${testCase.name} ${testCase.version}`,
      { stdio: 'inherit' }
    );
    
    const result = JSON.parse(fs.readFileSync('logs/analysis.json', 'utf-8'));
    
    const passed = 
      result.safe === testCase.expectedSafe &&
      result.risk_score <= testCase.expectedScore;
    
    if (passed) {
      console.log(`✅ PASS: ${testCase.name}@${testCase.version}`);
    } else {
      console.log(`❌ FAIL: ${testCase.name}@${testCase.version}`);
      console.log(`  Expected: safe=${testCase.expectedSafe}, score<=${testCase.expectedScore}`);
      console.log(`  Got: safe=${result.safe}, score=${result.risk_score}`);
    }
    
    return passed;
  } catch (error) {
    console.log(`❌ ERROR: ${testCase.name}@${testCase.version}`);
    console.log(error.message);
    return false;
  }
}

function main() {
  console.log('=== RUNNING VALIDATOR TESTS ===');
  
  let passed = 0;
  let failed = 0;
  
  for (const testCase of TEST_CASES) {
    if (runTest(testCase)) {
      passed++;
    } else {
      failed++;
    }
  }
  
  console.log('\n=== RESULTS ===');
  console.log(`Passed: ${passed}/${TEST_CASES.length}`);
  console.log(`Failed: ${failed}/${TEST_CASES.length}`);
  
  process.exit(failed > 0 ? 1 : 0);
}

main();
```

Run tests:

```bash
node tests/test-packages.js
```

**3. Document your findings**

Create `docs/validation-examples.md` and add notes as you test packages.

### Checkpoint

✅ You have a working validation system that correctly identifies safe packages.

---

# 🌐 Phase 3: API & Integration (Weeks 9-12)

## Week 9: Express API Setup

**Goal**: Create HTTP API for validation requests

### Tasks

**1. Initialize API project**

```bash
mkdir api
cd api
npm init -y
npm install express cors dotenv
npm install --save-dev nodemon
```

Update `package.json`:

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

**2. Create basic server**

`api/server.js`:

```javascript
const express = require('express');
const cors = require('cors');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(express.json());

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Validation endpoint (placeholder)
app.post('/api/validate', async (req, res) => {
  const { package: pkg, version, package_manager = 'npm' } = req.body;
  
  // Validation
  if (!pkg || !version) {
    return res.status(400).json({
      error: 'Missing required fields: package, version'
    });
  }
  
  // For now, return mock response
  res.json({
    validation_id: 'mock-' + Date.now(),
    package: pkg,
    version,
    safe: true,
    risk_score: 0,
    flags: [],
    cached: false,
    validated_at: new Date().toISOString()
  });
});

app.listen(PORT, () => {
  console.log(`🚀 Bridge Validator API running on port ${PORT}`);
});
```

**3. Test API**

```bash
npm run dev
```

In another terminal:

```bash
# Health check
curl http://localhost:3000/health

# Validation request
curl -X POST http://localhost:3000/api/validate \
  -H "Content-Type: application/json" \
  -d '{"package": "express", "version": "4.18.0"}'
```

**4. Add Docker integration**

Install Docker SDK:

```bash
npm install dockerode
```

Create `api/controllers/validator.js`:

```javascript
const Docker = require('dockerode');
const fs = require('fs');
const path = require('path');
const docker = new Docker();

async function validatePackage(pkg, version, packageManager = 'npm') {
  const validationId = `val-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  const outputDir = path.join(__dirname, '../validation-results', validationId);
  
  // Create output directory
  fs.mkdirSync(outputDir, { recursive: true });
  
  console.log(`Validating ${pkg}@${version}...`);
  
  try {
    // Run container with volume mount
    const container = await docker.run(
      'bridge-worker',
      [pkg, version],
      process.stdout,
      {
        HostConfig: {
          AutoRemove: true,
          CapAdd: ['NET_ADMIN'],
          Binds: [
            `${outputDir}:/tmp:rw`
          ]
        }
      }
    );
    
    // Read analysis results
    const analysisPath = path.join(outputDir, 'analysis.json');
    
    if (!fs.existsSync(analysisPath)) {
      throw new Error('Analysis file not generated');
    }
    
    const analysis = JSON.parse(fs.readFileSync(analysisPath, 'utf-8'));
    
    // Clean up
    fs.rmSync(outputDir, { recursive: true, force: true });
    
    return {
      validation_id: validationId,
      package: pkg,
      version,
      safe: analysis.safe,
      risk_score: analysis.risk_score,
      risk_level: analysis.risk_level,
      flags: analysis.flags,
      events: analysis.events,
      validated_at: new Date().toISOString(),
      cached: false
    };
    
  } catch (error) {
    // Clean up on error
    if (fs.existsSync(outputDir)) {
      fs.rmSync(outputDir, { recursive: true, force: true });
    }
    
    console.error('Validation error:', error);
    throw error;
  }
}

module.exports = { validatePackage };
```

Update `server.js`:

```javascript
const { validatePackage } = require('./controllers/validator');

app.post('/api/validate', async (req, res) => {
  const { package: pkg, version, package_manager = 'npm' } = req.body;
  
  if (!pkg || !version) {
    return res.status(400).json({
      error: 'Missing required fields: package, version'
    });
  }
  
  try {
    const result = await validatePackage(pkg, version, package_manager);
    res.json(result);
  } catch (error) {
    res.status(500).json({
      error: 'Validation failed',
      details: error.message
    });
  }
});
```

### Checkpoint

✅ You have a working API that runs Docker containers and returns validation results.

---

## Week 10: Redis Caching Layer

**Goal**: Make repeat validations instant

### Tasks

**1. Install Redis**

```bash
# Mac
brew install redis
brew services start redis

# Or use Docker
docker run -d -p 6379:6379 redis:7-alpine
```

Install client:

```bash
cd api
npm install redis
```

**2. Create cache service**

`api/cache/cache.js`:

```javascript
const redis = require('redis');

let client = null;

async function connect() {
  if (client) return client;
  
  client = redis.createClient({
    url: process.env.REDIS_URL || 'redis://localhost:6379'
  });
  
  client.on('error', (err) => console.error('Redis error:', err));
  
  await client.connect();
  console.log('✅ Connected to Redis');
  
  return client;
}

function getCacheKey(pkg, version, packageManager) {
  return `validation:${packageManager}:${pkg}@${version}`;
}

async function get(pkg, version, packageManager = 'npm') {
  const client = await connect();
  const key = getCacheKey(pkg, version, packageManager);
  
  const cached = await client.get(key);
  
  if (cached) {
    console.log(`✅ Cache hit: ${key}`);
    return JSON.parse(cached);
  }
  
  console.log(`❌ Cache miss: ${key}`);
  return null;
}

async function set(pkg, version, result, packageManager = 'npm', ttl = 2592000) {
  const client = await connect();
  const key = getCacheKey(pkg, version, packageManager);
  
  // TTL = 30 days by default (2592000 seconds)
  await client.setEx(key, ttl, JSON.stringify(result));
  console.log(`✅ Cached: ${key} (TTL: ${ttl}s)`);
}

async function clear(pkg, version, packageManager = 'npm') {
  const client = await connect();
  const key = getCacheKey(pkg, version, packageManager);
  
  await client.del(key);
  console.log(`✅ Cleared cache: ${key}`);
}

module.exports = { connect, get, set, clear };
```

**3. Integrate caching into validation**

Update `api/server.js`:

```javascript
const cache = require('./cache/cache');
const { validatePackage } = require('./controllers/validator');

// Connect to Redis on startup
cache.connect().catch(console.error);

app.post('/api/validate', async (req, res) => {
  const { package: pkg, version, package_manager = 'npm' } = req.body;
  
  if (!pkg || !version) {
    return res.status(400).json({
      error: 'Missing required fields: package, version'
    });
  }
  
  try {
    // Check cache first
    const cached = await cache.get(pkg, version, package_manager);
    
    if (cached) {
      return res.json({
        ...cached,
        cached: true
      });
    }
    
    // Not in cache, run validation
    const result = await validatePackage(pkg, version, package_manager);
    
    // Cache the result
    await cache.set(pkg, version, result, package_manager);
    
    res.json(result);
    
  } catch (error) {
    res.status(500).json({
      error: 'Validation failed',
      details: error.message
    });
  }
});
```

**4. Test caching**

```bash
# First request - should be slow
time curl -X POST http://localhost:3000/api/validate \
  -H "Content-Type: application/json" \
  -d '{"package": "express", "version": "4.18.0"}'

# Second request - should be instant
time curl -X POST http://localhost:3000/api/validate \
  -H "Content-Type: application/json" \
  -d '{"package": "express", "version": "4.18.0"}'
```

First request: ~10-30 seconds Second request: ~10 milliseconds

### Checkpoint

✅ Repeat validations are now instant via caching.

---

## Week 11: API Authentication & Rate Limiting

**Goal**: Secure the API for production use

### Tasks

**1. Add API key authentication**

Create `api/middleware/auth.js`:

```javascript
const validApiKeys = new Set(
  (process.env.API_KEYS || '').split(',').filter(Boolean)
);

function requireApiKey(req, res, next) {
  const apiKey = req.headers['x-api-key'] || req.query.api_key;
  
  if (!apiKey) {
    return res.status(401).json({
      error: 'Missing API key',
      message: 'Provide X-API-Key header or api_key query parameter'
    });
  }
  
  if (!validApiKeys.has(apiKey)) {
    return res.status(403).json({
      error: 'Invalid API key'
    });
  }
  
  next();
}

module.exports = { requireApiKey };
```

Update `server.js`:

```javascript
const { requireApiKey } = require('./middleware/auth');

// Apply to validation endpoint
app.post('/api/validate', requireApiKey, async (req, res) => {
  // ... existing code ...
});
```

Create `.env`:

```
API_KEYS=test-key-123,connor-desktop-key-456
```

**2. Add rate limiting**

Install:

```bash
npm install express-rate-limit
```

Create `api/middleware/ratelimit.js`:

```javascript
const rateLimit = require('express-rate-limit');

const validationLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 10, // 10 requests per minute per IP
  message: {
    error: 'Too many validation requests',
    message: 'Please wait before making more requests'
  },
  standardHeaders: true,
  legacyHeaders: false
});

module.exports = { validationLimiter };
```

Update `server.js`:

```javascript
const { validationLimiter } = require('./middleware/ratelimit');

app.post('/api/validate', 
  requireApiKey,
  validationLimiter,
  async (req, res) => {
    // ... existing code ...
  }
);
```

**3. Test authentication**

```bash
# Should fail (no API key)
curl -X POST http://localhost:3000/api/validate \
  -H "Content-Type: application/json" \
  -d '{"package": "express", "version": "4.18.0"}'

# Should succeed
curl -X POST http://localhost:3000/api/validate \
  -H "Content-Type: application/json" \
  -H "X-API-Key: test-key-123" \
  -d '{"package": "express", "version": "4.18.0"}'
```

### Checkpoint

✅ API is secured with authentication and rate limiting.

---

## Week 12: Bridge-Desktop Integration

**Goal**: Connor can call your API from Bridge-Desktop

### Tasks

**1. Document the API**

Create comprehensive API documentation in `api/docs/API.md` with:

- Authentication details
- All endpoints
- Request/response examples
- Error codes
- Rate limits

**2. Create example client for Connor**

Create `api/examples/client.js`:

```javascript
const fetch = require('node-fetch');

const API_URL = process.env.API_URL || 'http://localhost:3000';
const API_KEY = process.env.API_KEY || 'test-key-123';

async function validatePackage(pkg, version) {
  console.log(`Validating ${pkg}@${version}...`);
  
  const response = await fetch(`${API_URL}/api/validate`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-API-Key': API_KEY
    },
    body: JSON.stringify({
      package: pkg,
      version
    })
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'Validation failed');
  }
  
  const result = await response.json();
  
  console.log('\n=== VALIDATION RESULT ===');
  console.log(`Package: ${result.package}@${result.version}`);
  console.log(`Safe: ${result.safe ? '✅' : '⚠️'}`);
  console.log(`Risk Score: ${result.risk_score}/100 (${result.risk_level})`);
  console.log(`Cached: ${result.cached ? 'Yes' : 'No'}`);
  
  if (result.flags.length > 0) {
    console.log('\nSecurity Issues:');
    result.flags.forEach((flag, i) => {
      console.log(`${i + 1}. [${flag.severity.toUpperCase()}] ${flag.detail}`);
    });
  }
  
  return result;
}

// Test it
if (require.main === module) {
  validatePackage('express', '4.18.0')
    .then(result => {
      process.exit(result.safe ? 0 : 1);
    })
    .catch(error => {
      console.error('Error:', error.message);
      process.exit(1);
    });
}

module.exports = { validatePackage };
```

**3. Test end-to-end**

**4. Share with Connor**

Send Connor:

- API documentation
- Example client code
- Your API URL (once deployed)
- An API key for Bridge-Desktop

### Checkpoint

✅ Connor can successfully call your API from Bridge-Desktop.

---

# 🚀 Phase 4: Polish & Launch (Weeks 13-16)

## Week 13: Performance Optimization

**Goals:**

- Handle 100 validations/minute
- Reduce container startup time
- Optimize cache usage

**Key tasks:**

- Pre-pull Docker images on server startup
- Implement container pooling (reuse containers)
- Add database for validation history (optional)
- Monitor memory usage

## Week 14: Documentation & Error Handling

**Goals:**

- Comprehensive error messages
- Logging for debugging
- User-facing documentation

**Key tasks:**

- Add Winston or Pino for logging
- Create troubleshooting guide
- Add Sentry for error tracking
- Write deployment guide

## Week 15: Deploy to Production

**Goals:**

- Live on the internet
- Available 24/7
- Monitored

See Deployment Guide below.

## Week 16: Final Testing & Launch

**Goals:**

- Test with real users (Connor + team)
- Fix any bugs
- Announce launch

**Key tasks:**

- Beta test with Bridge-Desktop users
- Collect feedback
- Fix bugs
- Write launch blog post
- Ship it! 🚀

---

# ☁️ Deployment Guide

## Option 1: Railway (Easiest for MVP)

**1. Install Railway CLI**

```bash
npm install -g @railway/cli
```

**2. Login**

```bash
railway login
```

**3. Create project**

```bash
cd api
railway init
railway add
```

**4. Add Redis**

```bash
railway add redis
```

**5. Deploy**

```bash
railway up
```

**6. Set environment variables**

```bash
railway variables set API_KEYS="prod-key-123,connor-key-456"
```

Your API is now live!

## Option 2: Fly.io (More Control)

**1. Install Fly CLI**

```bash
curl -L https://fly.io/install.sh | sh
```

**2. Create app**

```bash
cd api
fly launch
```

**3. Deploy**

```bash
fly deploy
```

## Monitoring

Add uptime monitoring:

- UptimeRobot (free)
- Better Uptime
- Pingdom

---

# 🔧 Troubleshooting Common Issues

|Issue|Solution|
|---|---|
|Docker containers won't start|Make sure Docker daemon is running: `docker ps`|
|tcpdump permission denied|Run container with `--cap-add=NET_ADMIN`|
|Redis connection refused|Check Redis is running: `redis-cli ping`|
|API returns 500 errors|Check logs: `railway logs` or `fly logs`|

---

# 🎯 Next Steps After MVP

Once you have the MVP working:

1. **Add more package managers** (Python/pip, Ruby/gem, etc.)
2. **Improve heuristics** (machine learning for pattern detection)
3. **Build web dashboard** (view validation history)
4. **Add webhooks** (notify on suspicious packages)
5. **Create browser extension** (validate before installing in terminal)

---

# 💬 Getting Help

**When you get stuck** (and you will, that's normal):

1. **Check Docker logs**: `docker logs <container-id>`
2. **Search error messages**: 90% of errors are Googleable
3. **Ask Connor**: He's your boss, he wants you to succeed
4. **Stack Overflow**: Tag questions with `docker`, `node.js`, `security`
5. **Discord communities**:
    - Docker Community Slack
    - Node.js Discord
    - InfoSec communities

---

# 🎉 Final Thoughts

This is a big project, but it's broken down into manageable weekly chunks. Don't try to do everything at once.

**Your goal for Month 1**: Get a basic sandbox working **Your goal for Month 2**: Make it detect something suspicious **Your goal for Month 3**: Make it accessible via API **Your goal for Month 4**: Ship it

You got this. 🚀

The npm ecosystem genuinely needs this. You're building something that will protect thousands of developers from supply chain attacks.

---

> 💼 **EQUITY AGREEMENT**
> 
> Upon completion of this comprehensive build guide and your first commit to the bridge-validator GitHub repository, you will be granted **15% equity** in Bridge (the parent company). This equity **vests immediately** upon your first commit — no cliff, no waiting period.
> 
> This is a founding engineer position. Your work on the validator service is critical infrastructure for Bridge's success. Welcome to the team. 🚀
> 
> - Connor McCoy, Founder & CEO