# Project HELIX: Humanoid Assistant for Everyday Life

![Status](https://img.shields.io/badge/status-in%20development-orange)
![Simulation](https://img.shields.io/badge/simulation-MuJoCo-blue)
![Robot](https://img.shields.io/badge/robot-Unitree%20G1-lightgrey)
![License](https://img.shields.io/badge/license-MIT-green)

A humanoid assistant that **sees, listens, plans and acts safely**. HELIX perceives its surroundings, understands natural speech in Arabic and English, turns requests into validated action plans, and moves with a human-friendly presence. It is built **simulation-first** on the Unitree G1, so every capability is designed and stress-tested in a physics simulator before any hardware is risked.

> **Status:** in development. The project is currently in the planning and simulation-setup phase. See the [Roadmap](#roadmap) for progress.

## Table of contents

- [Motivation](#motivation)
- [Key ideas](#key-ideas)
- [Architecture](#architecture)
- [Tech stack](#tech-stack)
- [Quick start (simulation)](#quick-start-simulation)
- [Example: from speech to safe action](#example-from-speech-to-safe-action)
- [Safety design](#safety-design)
- [Planned project structure](#planned-project-structure)
- [Roadmap](#roadmap)
- [Resources](#resources)
- [License](#license)

## Motivation

Older adults living alone, people with mobility challenges, and busy families all need small, constant help: a reminder to take medication, an object fetched from another room, someone noticing when a fall happens. HELIX explores how a humanoid robot can provide this help safely, in the user's own language.

## Key ideas

- **Simulation-first:** develop and test everything in MuJoCo, then transfer to hardware only when proven.
- **The AI proposes, the supervisor disposes:** the language model never sends raw motor commands. It outputs structured plans that a deterministic safety layer checks before anything moves.
- **Multilingual by design:** speech understanding targets Arabic (including dialect-aware prompts) and English.
- **Human-friendly presence:** a soft, removable outer shell concept, slow motion near people, and force-limited manipulation.

## Architecture

```
            HUMAN  (voice, gesture, presence)
                        |
   +--------------------+--------------------+
   |                    |                    |
 HEARING              SEEING              CONTEXT
 Whisper STT       cameras, YOLO,        home map, schedule,
 Arabic/English    pose, faces           memory
   |                    |                    |
   +--------------------+--------------------+
                        |
                   AI PLANNER
        LLM -> schema-validated JSON skill plan
                        |
                SAFETY SUPERVISOR
   skill whitelist, speed and force caps, e-stop
                        |
                 SKILL LIBRARY (ROS 2)
   locate, navigate, grasp, hand-over, remind,
   follow, fall-response
                        |
                 MOTION CONTROL
   RL locomotion, whole-body control, Unitree SDK2
                        |
          +-------------+-------------+
          |                           |
   MuJoCo simulation             Real Unitree G1
     (start here)              (only when proven)
```

## Tech stack

| Domain | Technologies |
|---|---|
| Languages | Python (AI, rapid iteration), C++ (real-time components) |
| Robot middleware | ROS 2, Unitree SDK2 (DDS), unitree_sdk2_python |
| Simulation | MuJoCo, unitree_mujoco, Gazebo; Isaac Sim / Isaac Lab for large-scale RL |
| Learning | PyTorch, reinforcement learning (unitree_rl_mjlab, unitree_rl_lab), imitation learning via LeRobot |
| Perception | OpenCV, YOLO, pose estimation, SLAM |
| Language | Whisper (speech-to-text), a TTS engine, LLM planner with JSON-schema validation |

## Quick start (simulation)

**Requirements:** Ubuntu or WSL2 (Windows), Python 3. No physical robot needed.

```bash
# 1. Create an environment
python3 -m venv g1env
source g1env/bin/activate        # Windows: g1env\Scripts\activate
pip install mujoco pygame

# 2. Sanity check: opens the MuJoCo viewer
python -m mujoco.viewer

# 3. Install the Unitree SDK for Python
git clone https://github.com/unitreerobotics/unitree_sdk2_python.git
cd unitree_sdk2_python
pip install -e .                 # if cyclonedds fails: pip install cyclonedds first
cd ..

# 4. Run the G1 simulator
git clone https://github.com/unitreerobotics/unitree_mujoco.git
cd unitree_mujoco/simulate_python
# edit config.py:  ROBOT = "g1"   INTERFACE = "lo"
python3 unitree_mujoco.py
```

Control programs written against the Unitree SDK connect to the simulator the same way they connect to the real robot.

## Example: from speech to safe action

The planner converts a request into a plan the supervisor can inspect step by step:

```json
// user: "Bring me my water, please"
{
  "intent": "fetch_object",
  "steps": [
    { "skill": "locate",    "target": "cup" },
    { "skill": "navigate",  "to": "kitchen", "max_speed_mps": 0.4 },
    { "skill": "grasp",     "object": "cup",  "max_force_n": 15 },
    { "skill": "navigate",  "to": "user",     "max_speed_mps": 0.3 },
    { "skill": "hand_over", "to": "user",     "mode": "slow" }
  ]
}
```

The values above are illustrative examples of the plan format, not tuned parameters.

## Safety design

1. **The AI proposes, the supervisor disposes.** Every plan step passes a rule-based gate: skill whitelist, speed, force and workspace limits.
2. **One button stops everything.** The emergency-stop path is tested before any other feature and always outranks the planner.
3. **Fail in simulation first.** Fault injection (sensor dropouts, slips, blocked paths) is run repeatedly in MuJoCo before hardware moves.
4. **Soft by default.** Reduced speed near humans, force-limited grippers, and a compliant outer layer concept to protect people on contact.

## Planned project structure

```
helix-humanoid-assistant/
├── README.md
├── requirements.txt
├── docs/          # diagrams, project brief (PDF)
├── perception/    # detection, faces, pose, fall detection
├── language/      # speech-to-text, text-to-speech, LLM planner
├── skills/        # ROS 2 skill library
├── safety/        # safety supervisor
└── sim/           # simulation scripts and configs
```

This is the intended layout; folders are added as each phase is implemented.

## Roadmap

- [ ] **Phase 0:** environment setup, G1 running in MuJoCo, repository documentation
- [ ] **Phase 1:** joint state logging, PD control of a single joint, arm waypoints, soft limits, ROS 2 bridge
- [ ] **Phase 2:** perception (simulated camera stream, YOLO detection, face recognition, pose estimation, fall-detection rule)
- [ ] **Phase 3:** speech (Whisper STT and TTS), LLM planner with schema validation, first four skills
- [ ] **Phase 4:** safety supervisor, emergency stop, fault-injection tests, logging and replay
- [ ] **Phase 5:** learned locomotion policy trained and validated sim-to-sim
- [ ] **Phase 6:** silicone shell prototype and hardware readiness checks
- [ ] **Phase 7:** demo video (medication reminder, fetch a cup, fall response) and technical report

## Resources

- [unitree_mujoco](https://github.com/unitreerobotics/unitree_mujoco)
- [unitree_sdk2_python](https://github.com/unitreerobotics/unitree_sdk2_python)
- [G1 MuJoCo simulator (Hugging Face)](https://huggingface.co/lerobot/unitree-g1-mujoco)
- [LeRobot: Unitree G1 guide](https://huggingface.co/docs/lerobot/unitree_g1)
- [MuJoCo](https://github.com/google-deepmind/mujoco)
- [ROS 2 documentation](https://docs.ros.org)
- [unitree_rl_mjlab](https://github.com/unitreerobotics/unitree_rl_mjlab)
- [unitree_rl_lab](https://github.com/unitreerobotics/unitree_rl_lab)

## License

Released under the [MIT License](LICENSE).
