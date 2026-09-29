# Project HELIX: Humanoid Assistant for Everyday Life

A humanoid assistant robot that perceives its surroundings, understands speech (Arabic and English), plans safe actions, and helps with daily tasks. Built **simulation-first** on the Unitree G1.

**Status:** in development (simulation phase)

## Overview

HELIX combines four layers:

- **Perception:** object and person detection, face recognition, pose estimation, fall detection
- **Language:** speech-to-text (Whisper), text-to-speech, LLM-based planning
- **Planning:** the LLM outputs schema-validated JSON plans, never raw motor commands
- **Motion:** ROS 2 skills on top of the Unitree SDK, developed and tested in MuJoCo

A deterministic **Safety Supervisor** sits between the planner and the robot: skill whitelist, speed and force caps, and an emergency stop.

## Quick start (simulation)

Requires Ubuntu or WSL2 with Python 3.

```bash
python3 -m venv g1env && source g1env/bin/activate
pip install mujoco pygame

git clone https://github.com/unitreerobotics/unitree_sdk2_python.git
cd unitree_sdk2_python && pip install -e . && cd ..

git clone https://github.com/unitreerobotics/unitree_mujoco.git
cd unitree_mujoco/simulate_python
# edit config.py: ROBOT = "g1", INTERFACE = "lo"
python3 unitree_mujoco.py
```

## Roadmap

- [ ] Phase 0: environment setup and G1 running in MuJoCo
- [ ] Phase 1: joint state logging, PD control, arm waypoints
- [ ] Phase 2: perception (YOLO, faces, pose, fall detection)
- [ ] Phase 3: speech, LLM planner, first four skills
- [ ] Phase 4: safety supervisor and fault-injection tests
- [ ] Phase 5: learned locomotion (sim-to-sim)
- [ ] Phase 6: embodiment prototype and hardware readiness
- [ ] Phase 7: demo video and technical report

## Resources

- [unitree_mujoco](https://github.com/unitreerobotics/unitree_mujoco)
- [unitree_sdk2_python](https://github.com/unitreerobotics/unitree_sdk2_python)
- [MuJoCo](https://github.com/google-deepmind/mujoco)
- [ROS 2 docs](https://docs.ros.org)

## License

MIT
