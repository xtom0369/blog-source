---
title: Game Physics Introduction & Acknowledgements【译】
date: 2021-11-24 21:32:11
tags:
- Physics Engine
categories:
- Physics Engine
- Physics
---

这是由顽皮狗的工程师[周明倫](http://allenchou.net/game-physics-series/)写的有关物理引擎的系列，非常赞，条理清晰，考虑到国内有关物理引擎的资料比较稀缺，遂尝试翻译，有英语基础的可直接阅读原文，原汁原味。


# 游戏物理: 介绍 & 致谢

[原文网页](http://allenchou.net/2013/12/game-physics-introduction/)

I shifted from being the graphics programmer to being the physics programmer for my DigiPen game team half a year ago as we started developing our junior project. Knowledge on game physics is considered “tribal knowledge”, in that there is no formal book or publication that covers every aspect of it; also, cutting-edge information is typically passed down through online articles, forum discussions, and gamedev community activities such as GDC lectures.

I felt like beginning tagging along this tradition and decided to start a game physics series on my blog, sharing what I have learned and perhaps my own thoughts as I carry on studying game physics. In this very first post of the series, I will go through what constitutes a constraint-based rigid body physics engine. Details on individual topics will be covered in future posts.


Terminologies
First, let’s go through some terminologies that will be used throughout this series.

Rigid Body – an individual object that is simulated within the physics engine. As the name suggests, a rigid body is “rigid”, in that all parts belonging to the same rigid body do not distort, move, nor rotate relatively. A typical rigid body data structure would contain several physics properties: mass, inertia tensor (or just moment of inertia in 2D), position, orientation, and velocity (both linear and angular). Usually, you would also need the reciprocals (inverses) of mass and inertia tensor, so it is a good idea to precalculate them once and store them for each rigid body.

Collider – a “part” of a rigid body. For instance, a dumbbell would consist of two weights and a rod connecting them; this counts as three colliders. Some might argue that the dumbbell as a whole is a single collider. This argument is valid; however, it is convenient to model a rigid body as multiple convex colliders “pieced” together rather then a single concave collider, the reason of which would be discussed later. (I’m using Unity‘s terminology here. In Box2D they are called “fixtures” instead of “colliders”.)

Collision – Two colliders are considered colliding if they physically overlap. If two spheres are touching at one point, they collide. If two spheres are intersecting, they collide.

Contact – Collision between two colliders is typically modeled as a collection of contact points, or just contacts for short.

Contact Manifold – The collection of contacts between two colliders is commonly know as the collider pair’s “contact manifold”. This is a simplification of the geometry that defines the intersection of two colliders.

Components of A Physics Engine
Next, let’s look at what constitutes a physics engine. A single time step during physics simulation involves several phases. They are listed below in the typical order of execution:

Broadphase – A broadphase is the algorithm of choice to efficiently decide whether it is possible for two colliders to collide. If there is no chance for a collision to happen (e.g. two colliders are at two extreme ends of a game level), then just skip the relative more expensive collision detection for the collider pair. With no broadphase, a physics engine would compare every possible collider pair every time step; some people still consider this a broadphase and call it the “N-squared broadphase”. Commonly seen broadphases include explicit grid, implicit grid, sweep-and-prune, and dynamic AABB tree. False positive pairs are acceptable; they will just be rejected as not colliding by the collision detection phase that happens later.

Collision Detection – For each collider pair spit out by the broadphase due to potential collision, the collision detection logic would decide if the colliders are actually colliding. For detecting collision between simple regular primitive shapes like spheres and boxes, a physics engine would often have special-case implementation for each potential combination (e.g. sphere-sphere, sphere-box, and box-box). However, algorithms like GJK (Gilbert-Johnson-Keerthi) and MPR (Minkowski Portal Refinement) are usually used to handle collision detection between general convex shapes that are not accounted for by special-case implementations, so the physics programmers don’t have to implement special-case collision detection for all possible combinations of spheres, boxes, capsules, quads, pyramids, cones, polyhedrons, etc.. Note that both GJK and MPR only work with convex shapes; that’s why I mentioned earlier that it is a good idea to split concave colliders into multiple convex colliders.

Resolution – This is the last step of a physics time step. Collisions, as well as other various “physical limits” (such as springs, hinges, pins), are “resolved” at this stage. That means during the resolution phase, the physics engine tries to fix “violations”, such as two overlapping colliders resulting in collision, two objects tied together by a spring being too far apart, two objects supposed to be pinned together at a point being broken apart, etc.. Such attempt by the physics engine would involve modifying the position, orientation, and velocity (both linear and angular) of rigid bodies.

Note that during the collision detection phase, unlike what would normally happen in a graphics engine, the process does not stop at the point where two colliders are determined to be colliding or not. The collision detection logic needs to generate extra information about contacts (e.g. contact positions, contact normals, and penetration depths), so that violations can be resolved during the resolution phase. MPR alone includes logic for generating the required contact information. However, GJK only tells you whether two colliders collide; thus, GJK is usually used along with EPA (Expanding Polytope Algorithm), which generates the required contact information.

Sequential Impulse
Popularized by Erin Catto (author of Box2D), Sequential Impulse is an efficient and effective implementation of Gauss Seidel LCP method, which is used to resolve violated constraints. Collision contacts and physical limits (springs, hings, pins, etc.), are modeled as “equality constraints”. Constraints are solved in a unified way where impulses are applied to rigid bodies, as a fix to the “violations”. A geometric analogy can be drawn between Sequential Impulse and finding intersection of multiple planes through a series of point projections. You might feel a little bit confused and lost with all this abstract description of Sequential Impulse. Don’t worry. There will be a dedicated post just for this topic.

End of Introduction
Alright, that’s all I have to say as an introduction. I will be writing more posts on individual topics in greater details. I hope this post has helped you gain a decent overall idea of how game physics engines work.

Acknowledgements
I’d like to give my thanks to the people who helped me learn about game physics and provided references (listed in the order of last names).

People From DigiPen Institute of Technology
Andrew Alvarez for his help on various 3D physics issues.
Nathan Carlson for his help on general physics problems.
Joshua Davis for his help on fixing 3D physics bugs.
Randy Gaul for his exchange of experience in 2D & 3D game physics development.
Prof. Xin Li and Prof. Natalia Solorzano for teaching us the basics of motion dynamics and its application in games.

People From The Industry
Gino van den Bergen for his book Collision Detection in Interactive 3D Environments.
William Bittle for his website on game physics, Code Zealot.
Erin Catto for his opensource 2D physics engine, Box2D, and his GDC presentations.
Daniel Chappuis for his paper, Constraint Derivation for Rigid Body Simulation in 3D.
Erwin Coumans for his opensource 3D physics engine, Bullet.
Christer Ericson for his book Real-Time Collision Detection.
Dirk Gregorious for his GDC presentations.
Ian Milliongton for his book Game Physics Engine Development.

