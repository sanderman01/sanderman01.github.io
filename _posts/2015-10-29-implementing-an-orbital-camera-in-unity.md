---
layout: post
title: Implementing an orbital camera in Unity
date: 2015-10-29 13:18
author: sanderman0
comments: true
tags: [camera, game design, game development, orbital, software-development, spherical, unity]
---
Currently I am working at a company on a project where it is very important that the player is able to easily inspect and interact with certain objects from various angles by focusing on, zoom and rotating around them. This also required constraints to limit the camera to predefined areas, to guide the player and to prevent clipping. Nothing I could find really satisfied me, so I wrote my own variant.

Orbital camera systems by themselves are quite simple to implement, but things can get a bit tricky when you start to add constraints as a requirement. I noticed many people having problems with this sort of thing, so I finally took the time to explain the way I decided to approach this problem. It's also to help me remember this stuff if I need something similar at a later date. Included is a simplified piece of example code.
<h1>Defining the problem</h1>
<ul>
	<li>We want our camera (or any object really) to orbit around another object, responding to user actions and keeping at a certain distance from the target object. In other words, we want the camera to be driven by some type of a <a href="https://en.wikipedia.org/wiki/Spherical_coordinate_system" target="_blank">Spherical Coordinate System.</a></li>
	<li>We want the ability to control both Pitch (up/down or the local x-axis) and Yaw. (left/right or the local y-axis) We'll ignore the Roll axis.</li>
	<li>For each point that the camera will focus on, we want the ability to define constraints that determine where the camera is allowed to go.</li>
</ul>
<h1>Implementation</h1>
Let's start with defining our focus points and constraints. When people think about rotations, many of them think in degrees or radians. That's why it might seem natural to define the constraints simply as a set of angles like MinPitch, MaxPitch, etc.

There is a different way to do it, which may seem a bit of a weird approach at first, but it makes the entire constraint problem much easier to solve. Instead of setting explicit boundary numbers on each side, we define the constraint as a maximum deviation from a center rotation. The constraint is then defined by the angles YawLimit, PitchLimit and the center rotation. (the center rotation in this case being simply the rotation of the focus point object)

Doing it this way means we can avoid the mess of trying to figure out where we are in relation to the constraints and whether we need to rotate left or right to satisfy them. We can simply calculate the difference between the angles and then use linear interpolation to move the required amount back towards the target rotation without needing to know or care which direction that is. We also don't have to deal with issues such as wrapping around from weird angles like at the 0-360 degrees boundary.

For that reason, instead of storing most of our angles as floats, we store them as Quaternions. Quaternions seem complicated, but they really make some calculations much easier. And if we want to work in degree angles, we simply convert between them.

Finally, we multiply our current pitch and yaw rotations together to get our desired rotation. We could stop at this point, and then we would simply have a camera that rotates on the spot. Useful for e.g. a security camera. To turn it into an orbit camera, we calculate our offset by taking our forward vector and multiplying it with our target rotation, then adding that to the position of our target object. Lastly, we can add some damping for smoother movement.
<h1>Example</h1>
Note that this implementation may not be suitable for everyone, as it largely depends on your use case, but if not then at least it may give you some ideas on how to write your own, suited for your purposes. The example code is also very basic and not optimized so you may want to add a few improvements if you decide to use it.

https://gist.github.com/sanderman01/647f954103c4b3577031

There's many ways this example can be improved upon. The following suggestions are left as an exercise for the reader:
<ul>
	<li>Zooming. (hint: you'll want to look at logarithmic or exponential functions instead of just linearly increasing/decreasing the distance)</li>
	<li>A fancy editor that allows you to see the constraint angles in the scene view. (hint: <a href="http://docs.unity3d.com/ScriptReference/Handles.DrawWireArc.html" target="_blank">Handles.DrawWireArc</a>)</li>
	<li>Can you spot the minor bug? (Well. It may or may not be considered a bug depending on your perspective.)</li>
	<li>Using local space instead of world space allows different camera angles, but also makes your camera more dependent on the transform hierarchy. Useful e.g. if your target object is a spaceship and you want the camera to keep a certain orientation in relation to the ship instead of the world.</li>
	<li>Smooth switching from one target object to another.</li>
</ul>
