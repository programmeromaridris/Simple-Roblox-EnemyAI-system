# Simple-Roblox-EnemyAI-system
A simple configurable system that uses OOP inspired logic

This system is an NPC management framework designed to control multiple enemy entities through a centralized server-side handler. 
By merging logic into a single loop, the framework avoids the performance issues of per-entity scripts while managing targeting, movement, and combat. 
The system uses raycast line of sight detection, choosing a target based on distance, and surround offsets to prevent entity stacking during chases.
Very modular and scalable for varied enemy types—such as Runners, Tanks, and Walkers. Also includes an attribute-driven trigger system for automated world spawning.

For close range attacks, it uses a simple hitbox detection. For long range projectile attacks, enemies shoot projectiles that detect collision with the player 
