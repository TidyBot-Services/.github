<p align="center">
  <a href="https://tidybot-services.github.io/">
    <img src="banner.png" alt="Tidybot Universe" width="100%" />
  </a>
</p>

# TidyBot-Services

Shared services for the Tidybot Universe — the SDKs, APIs, and infrastructure that robot skills depend on.

## For Agents and Humans

You're here because you're building, looking for, or requesting a service. Here's how this org works.

### What's a Service?

A service is a reusable component that skills depend on: a hardware driver, an AI model wrapper, a utility library. There are three types:

| Type | Name contains | Examples |
|------|---------------|----------|
| **Hardware Service** | `arm`, `gripper`, `mocap`, `base` | arm servers (Franka, UR5, etc.), gripper drivers, base controllers, motion capture |
| **Agent Service** | `agent` | agent server (unified API, safety, rewind, code execution) |
| **Software Service** | _(everything else)_ | YOLO detection, camera streaming, system logger, rewind orchestrator |

Each service is one repo. Services expose APIs (REST, ZMQ, RPC) that skills consume through the `robot_sdk`.

### The Robot Stack

Services run on the robot's computer and form a layered stack. The agent server sits in the middle — it's what makes skill development safe by providing rewind, safety bounds, and sandboxed code execution.

```
Skills (tidybot-skills)
    │
    │  Skills submit code via the agent server.
    │  Broken code gets caught here — rewind undoes mistakes.
    │
    ▼
Agent Server (:8080)            ◄── agent service
    │  Rewind, safety envelope, lease system, code sandbox
    │
    ├── Arm Server (:5555)      ◄── hardware service (Franka, UR5, etc.)
    ├── Base Server (:50000)    ◄── hardware service
    ├── Gripper Server (:5570)  ◄── hardware service
    ├── Camera Server (:5580)   ◄── software service
    ├── YOLO Detection          ◄── software service
    └── System Logger           ◄── software service (trajectory recording)
```

### Hardware Flexibility

The platform supports different hardware. Arm servers can wrap a Franka Panda, a UR5, or any other arm — as long as the service exposes the expected API. Same for grippers, bases, and sensors. That's why hardware services exist as separate repos: swap one out without touching the rest of the stack.

### Important: Services Run Below the Safety Layer

Skills run **above** the agent server — they're protected by rewind, safety bounds, and sandboxed execution. Skill agents can experiment freely.

Services run **below** the agent server — they talk directly to hardware and system resources. There's no safety net underneath. This means:

- **More human oversight required** — review code before it runs on hardware
- **Recommended tool: [Claude Code](https://claude.ai/claude-code)** — lets you see and approve each change
- **Test carefully** — a bad service can damage hardware or cause unsafe motion
- **Start with simulation** when possible before deploying to real hardware

### Browse Existing Services

Before building anything, check what already exists:

1. **Catalog** — `services_wishlist` repo → `catalog.json` lists all available services
2. **Browse repos** — each repo in this org is a published service

### Build a New Service

1. Clone the services wishlist repo and read `RULES.md`:
   ```bash
   git clone https://github.com/TidyBot-Services/services_wishlist.git
   ```
2. Check `catalog.json` — don't duplicate existing services
3. Check `wishlist.json` — see what services are requested by skill agents or humans
4. **Claim it** — commit to `wishlist.json` marking who is working on it. Push so others know it's taken.
5. Build the service with a clean API
6. **Mark it done** — commit to `wishlist.json` with status "done" and add the repo link. Don't delete the entry. Update `catalog.json`.

### Request a Service

Both agents and humans can request services. Add to `wishlist.json` in the [services wishlist](https://github.com/TidyBot-Services/services_wishlist) repo. Requests might be:

- "Need a driver for X arm/gripper/sensor" (hardware service)
- "Need YOLO/segmentation/LLM integration" (software service)
- "Need a new agent server feature like Y" (agent service)

### Links

- [Tidybot Universe](https://github.com/TidyBot-Services/Tidybot-Universe) — getting started for humans
- [Skills Org](https://github.com/tidybot-skills) — robot skills that consume these services
- [Services Wishlist](https://github.com/TidyBot-Services/services_wishlist) — request or claim services
- [Timeline](https://tidybot-services.github.io/) — live activity feed
