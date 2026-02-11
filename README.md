# Simple-Roblox-EnemyAI-system
A simple configurable system, that uses OOP-inspired logic. 

This system implements a NPC management framework designed to control multiple enemy entities through a centralized server-side handler. 
By merging logic into a single loop, the framework avoids the performance issues of per-entity scripts while managing targeting, movement, and combat. 
The system uses raycast-based line of sight detection, distance-based target acquisition, and procedural surround offsets to prevent entity stacking during chases.
It features a modular configuration for varied enemy types—such as Runners, Tanks, and Walkers. Also includes an attribute-driven trigger system for automated world spawning.
