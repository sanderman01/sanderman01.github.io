---
layout: post
title: Stuck Detection
date: 2013-02-19 17:18
author: sanderman0
comments: true
tags: [size matters, game development, mathematics, trigonometry]
---
A while ago I took the task of improving the 'stuck detection' code in our game codenamed <em>Size Matters</em>, and I want to share this solution in case someone out there finds it useful.

This game features a player character which is essentially a circle with the ability to roll in both directions and traverse a level. What makes our game relatively unique is that the player also has the ability to grow and shrink. This allows various interesting twists to what would otherwise be a fairly standard platformer type game.

<img class="size-full wp-image-298" alt="sizematters-cart" src="http://sanderman0.files.wordpress.com/2013/02/sizematters-cart.png" width="625" height="434" />
The player can enter this vehicle by shrinking into the gaps and growing to fill the space in the middle. The player can provide torque to turn the wheels.

This introduces the issue of knowing when the player is allowed to grow. Obviously if the player is encased on all sides by walls or other colliders, then he should not be able to grow as that would cause spazzing. <small>(spazzing is the official technical term for confusing the heck out of the physics engine, and clipping through objects)</small> The player might even end up breaking the level design by bypassing entire areas. If the player is only partially encased, then he may be able to grow, and pop out of a hole if needed. So we needed a way to determine when the player is considered stuck and should not be a allowed to grow any further.

Our first approach was conceived during the game jam where the first prototype of this game was built. It was a hasty, naive approach which took contact-points from current collisions and compared the directions using the dot product. If the value exceeded a certain threshold, we considered them opposite and the player was considered stuck. This worked for the most obvious cases such as pipes, though there were still quite a few situations where it failed. It would not work with the vehicle in the image above.

For commercial release I wanted something that actually worked reliably. I took a step back and reflected on the situation. What information do we have:
<ul>
	<li>A circle</li>
	<li>Several contactpoints on the circle</li>
	<li>We might be stuck if we have at least three contactpoints. (We can ignore cases with two or fewer contactpoints)</li>
</ul>
Play around with this on a scratchpad or in a program like Cinderella and you'll come to the following conclusion:
<ul>
	<li>The player is stuck if any triangle possible with the current contactpoints overlaps the midpoint of the circle.</li>
</ul>
<a href="http://sanderman0.files.wordpress.com/2013/02/circumcircle.png"><img class="aligncenter size-full wp-image-300" alt="circumcircle" src="http://sanderman0.files.wordpress.com/2013/02/circumcircle.png" width="479" height="450" /></a>

The next question then becomes, how do we figure out if the midpoint lies within the triangle? The answer lies in high-school <a href="http://en.wikipedia.org/wiki/Circumcircle">geometry</a>.
<blockquote>
<ul>
	<li>If and only if a triangle is acute (all angles smaller than a right angle), the circumcenter lies inside the triangle.</li>
	<li>If and only if it is obtuse (has one angle bigger than a right angle), the circumcenter lies outside the triangle.</li>
</ul>
</blockquote>
So we only need to check if one of the angles of the triangle is larger than a right angle. The easiest way to do that is to take the dot product of the directions of the two line segments. If the dot product is negative, that means the angle is greater than 90 degrees.

But wait.. How do we deal with more than three contact-points? We simply take the first contact points, and iterate over the rest in pairs. If any resulting triangle overlaps the midpoint, then the player is stuck.
