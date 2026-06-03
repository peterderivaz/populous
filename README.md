# "Populous: The Beginning" frame breakdown

## Background

<img width="800" height="600" alt="Target frame" src="https://github.com/user-attachments/assets/c3be5c8d-2a11-437c-ae93-67821ab32436" />

[Populous: The Beginning](https://en.wikipedia.org/wiki/Populous:_The_Beginning) was released in 1998.  I had really enjoyed the original Populous game, and this 3d sequel looked, sounded, and played amazingly well.  It is still fun to play today, so give it a go if you have not tried it before.

At the time, few people had 3d graphics cards, so the game uses a number of tricks to run smoothly with CPU rendering.

This document explains the graphical tricks I have found so far.

We will gradually try and build up to the frame shown above (taken from the actual game).

## Map

<img width="387" height="386" alt="level_data" src="https://github.com/user-attachments/assets/9017c5b1-a884-4ff3-94f0-cda83eba0753" />

The map is a 128 by 128 byte array of heights.  This is levl2079.dat, and the red circle shows the part from the screenshot.  

0 represents sea level (black in the image), up to 255 for the highest mountains (white in the image).

The two white dots within the circle show the high altitude parts and are the mountains in the screenshot.

Note that the entire game (just like the original populous) is played on a square map.

If you move off the left of the map, you appear on the right.

If you move off the top of the map, you appear on the left.

However, in the game, it feels like you are playing on a sphere.  This is due to a clever choice of projection described below.

## 3d projection

<img width="800" height="600" alt="wireframe2" src="https://github.com/user-attachments/assets/dee613ed-cfd4-482a-82b6-2cc3b95c9411" />

This shows a wireframe of the terrain vertices using a perspective view.

At this point, we are still clearly playing on a square map.

## Cylindrical distortion

### Left-right

<img width="800" height="600" alt="zoombend1" src="https://github.com/user-attachments/assets/5519cf55-b649-4a9d-899e-d2ff6d8f2cbb" />

We can get closer to the target image by bending the level left-right around a cylinder.

<img width="800" height="600" alt="bend1" src="https://github.com/user-attachments/assets/95c31439-7a89-4c39-8aac-04b8e7f0c997" />

When zoomed out, we can see that the terrain only goes halfway round the cylinder.  In other words, we would need two copies of the terrain to go all around the cylinder.

### Front-back

<img width="800" height="600" alt="zoombend2" src="https://github.com/user-attachments/assets/9ca2daa1-bef2-45d3-a86d-083d30a7b481" />

We can also bend it front-back around a cylinder.

<img width="800" height="600" alt="bend2" src="https://github.com/user-attachments/assets/0c46f410-fefc-4fbe-a4b8-5fa5466d5d29" />

As before, this only goes halfway round.

### Both left-right and front-back

<img width="800" height="600" alt="zoombend3" src="https://github.com/user-attachments/assets/191b8d6b-8226-4f2d-bfb4-ecf8dd5e09eb" />

With a physical object, like a piece of paper, we can only choose one of these, or the paper will crumble.

However, with our vertices there is nothing to stop us applying both distortions at the same time.

<img width="800" height="600" alt="bend3" src="https://github.com/user-attachments/assets/7bef2a07-2069-4a35-8db7-363312550407" />

The distortions caused by applying both bends get worse further away from the centre.  The trick that Populous uses is to recompute the bends for each frame based on moving the centre of the bend to the current bit of the terrain that is being viewed.  This way, we only ever see the part of the terrain that is being gently stretched, so all is well.

## Sky rendering

### Sky cylinder

<img width="800" height="600" alt="sky_wire" src="https://github.com/user-attachments/assets/7c25a010-c31d-493f-a951-9fef38b0168b" />

The sky is a bit simpler.  It is a grid with just the left-right cylindrical distoration applied.  (It probably uses far fewer triangles than shown here.)

### Sky tilt

<img width="800" height="600" alt="sky_tilt" src="https://github.com/user-attachments/assets/d312f82a-e64d-4597-a31c-f202389f3d6d" />

The sky is also tilted down so more of the sky is visible.  Above is a zoomed out side view of the tilted sky.

As for the terrain, the effective 3d location of the sky is adjusted on every frame to be bent left-right and tilted based on the current camera position and orientation.

### Sky texturing

<img width="384" height="386" alt="sky" src="https://github.com/user-attachments/assets/894cc5d0-901c-4b1b-8334-8046af26afeb" />

The sky texture is a 512 by 512 byte array, this is `skyj-0.dat`.

<img width="800" height="600" alt="sky_texture" src="https://github.com/user-attachments/assets/349115df-9f33-4212-a584-6ea9dc322b64" />

The distorted grid is rendered using nearest neighbour texture lookup into this texture.

The uv coordinates used for this texturing are moved faster than they should compared to the 3d location of the points.

This means that when the player moves the camera, the sky moves twice as fast as the terrain.  This compensates for the way that the cylindrical curvature is too low and helps the illusion that you are playing on a spherical world.

An artifact of this is that if you turn 360 degrees in the game, then the sky looks completely different.

## Terrain rendering

The CPU precomputes separate 32x32 textures for each square on the terrain grid.  These texures will bake in the displacement mapping and lighting effects from the sun, and contain eroded trails based on footsteps from the workers.

These textures are then drawn onto the screen as triangles using nearest neighbour interpolation (plus some extra lookups to apply lighting effects).

When the terrain changes (via magic spells or workers flattening the ground), the precomputed textures are regenerated.

### Altitude base colour

<img width="800" height="600" alt="height_colour" src="https://github.com/user-attachments/assets/346e9ad2-dd08-4d8b-999e-0d58687ef716" />

The height at each of the 32x32 positions within a grid square is bilinearly interpolated from the heights at the corners.

This height is then used in a 256 by 1152 2d lookup table `bigf0-j.dat` to find the final colour.

<img width="196" height="832" alt="bigf0" src="https://github.com/user-attachments/assets/9c7bce6d-6ced-4cb3-be4d-537866ff313c" />

The y coordinate is given by the interpolated height.  The x coordinate corresponds to lighting (in this screenshot we are using a fixed value for x).

### Static lighting

<img width="800" height="600" alt="lit_height" src="https://github.com/user-attachments/assets/eb621368-0589-41b4-b366-7b16f40c243f" />

The gradient at each vertex is computed based on the horizontal and vertical height differences between adjacent terrain vertices.

This gradient is then combined with the sun direction (via a dot product) to give the light level for each vertex.

There is a boost to the light level proportional to altitude (this makes the tops of the mountains shine brighter).

There is a reduction to light level if the grid square contains a tree (to emulate shadows).

The light level is bilinearly interpolated across the grid square and used as the x coordinate in the 2d lookup.

Note that the sun is a fixed direction from the point of view of the original square 2d grid.  None of the cylindrical distortions apply to the lighting.  If the sundial says it is elevenses in London, it will also say it is elevenses in Australia.

### Dithering

<img width="800" height="600" alt="dithered" src="https://github.com/user-attachments/assets/96519776-55dd-4b03-9c0b-4b7be9ab3081" />

Dithering is added into the rendered textures with no computation cost. (This was not seen in earlier screenshots as they were displayed with an undithered fade table.)

<img width="860" height="387" alt="dither" src="https://github.com/user-attachments/assets/129daada-8cb1-4f0e-ba14-1517f502357f" />

This shows a closeup of the 2d fade table.  The table includes dithering prebaked into the lookup table.  

### Dynamic lighting

<img width="800" height="600" alt="dithered_fade" src="https://github.com/user-attachments/assets/4a295786-6da4-46bb-8635-5fc4e3d115b8" />

As the triangle is being rendered onto the screen, an additional lookup is performed so that more distant pixels are dimmer.

This dynamic lighting is also used for other purposes such as shading on the sea and UI effects.

### Displacement map change height

<img width="800" height="600" alt="displacement" src="https://github.com/user-attachments/assets/b872d765-732e-4bff-81fa-b7f1af6be1d8" />

A 256 by 256 displacement texture 'disp0-j.dat' (this one looking like the surface of the moon) is used to give fine detail to the static lighting.

<img width="384" height="388" alt="disp0" src="https://github.com/user-attachments/assets/079591ee-b7b9-4d58-84fa-6a560810e41c" />

This contains signed bytes that are used to offset the y coordinate of the 2d fade lookup.

### Displacement map change lighting

<img width="800" height="600" alt="disp_light" src="https://github.com/user-attachments/assets/a6fd72df-b4a5-4fa8-815d-ba17f50b4b99" />

The same displacement is also used to apply an offset to the x coordinate of the 2d fade lookup (i.e. to adjust the light level).

However, in this case, the offset is based on the difference between the displacement map value, and the displacement map value one texel to the lower-right.  This approximates a gradient based diffuse component to the lighting.

### Adaptive split

To make the terrain a bit smoother, an adaptive split is used to change the grid squares into pairs of triangles.

There are two ways a square can be split into two triangles.  In each split there will be two shared vertices.  Choosing the way that minimises the altitude difference between the shared vertices gives a smoother mesh.

#### Non adaptive split
<img width="800" height="600" alt="adaptive_split" src="https://github.com/user-attachments/assets/9f33a0f1-9e1f-45a8-8ccb-2f3acb68cd35" />

The screenshot above shows what happens if a uniform split is used.  Note that the far mountain becomes quite jagged.

#### With adaptive split

<img width="800" height="600" alt="disp_light" src="https://github.com/user-attachments/assets/a6fd72df-b4a5-4fa8-815d-ba17f50b4b99" />

(The adaptive split method has been used in the previous screenshots, so this image is the same as before.)

## Tree rendering

<img width="800" height="600" alt="trees" src="https://github.com/user-attachments/assets/d84986a8-5a6f-4370-91b2-625492da3541" />

Objects are rendered using standard nearest neighbour texture mapping plus the dynamic lighting palette effect to make them dim with distance.

The texture map includes a transparent colour (allowing fine details of leaves to be painted onto objects).

<img width="383" height="338" alt="textures" src="https://github.com/user-attachments/assets/cb1f183d-52d8-453a-87a6-c4c8e707bdda" />

A 256 by 1024 texture map `bl320-j.dat` contains all the textures for the objects. (Only part of the textures shown here.)

## Sea rendering

The sea is rendered using the same static lighting algorithm, but the altitudes of the sea vertices and the dynamic lighting are different.

The dynamic lighting is based on the adjusted altitudes instead of being distance based.

The altitudes of the sea vertices is adjusted based on the sea texture in `watdisp.dat`.

<img width="192" height="191" alt="watdisp2" src="https://github.com/user-attachments/assets/e741b76b-aeb1-4459-aaba-2272b2ed5112" />

The height of the sea is given by sampling this sea texture, from a position which scrolls over time, combined with a second sample of the same sea texture, but from a position that scrolls in the opposite direction over time.

This is an interesting approach that avoids a prevailing wind direction.  Sampling from the sea texture once would look very bad, with a fixed wave pattern moving in a fixed direction.  Sampling twice combines the textures in such a way that there is no overall motion in any direction, but lots of local swirls and seemingly non-uniform motion.

## Beach rendering

<img width="800" height="600" alt="beach" src="https://github.com/user-attachments/assets/363de7a1-1a72-409f-9ddb-25a5f637cc71" />

If you compare the final screenshot to the map data, the island seems a lot bigger than it was in the map.  This is because a beach is added around each island.

Beach vertices have 0 height in the map, but are next to a vertex that has a positive height.  The beach vertices are coloured turqoise in the screenshot above, while sea vertices (0 height and all neighbours are also 0 height) are coloured blue.

When rendering, the beach vertices keep their zero height (and are not adjusted by the sea altitude changes), but the sea vertices are given a negative altitude displacement.  This gives a pleasant surrounding shallow beach around each island. 

# Populous remastered

<img width="1672" height="941" alt="populous_concept" src="https://github.com/user-attachments/assets/49910ecd-1588-4f88-9da6-602cdde94ea8" />

Maybe one day we will get a remastered version?

