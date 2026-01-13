# LHFire

Particle effect renderer used for the explosions, plasma shots, and other sprites in the mod, can make .spr, .spr32, and .tga output.


## Usage

```shell
lhfire script.txt
```


## Script Commands


### Required

`render width height filename`

Sets the frame width and height, and output name, if the filename ends in .spr it makes a normal 8bit quake sprite, if it ends in .spr32 it will make a 32bit quake sprite (perfect image quality, but the engine must support my 'sprite32' format), if it ends in .tga it will save out targas, if saving multiple frames as targa be sure to put `%02d` in the filename where you want the frame number (or `%03d` for 3 digits)

`frames starttime endtime startframe endframe`

Sets the beginning and end time for the scene, and the start and end frame numbers that time range maps to

`particles number`

(required) Sets the particle limit, setting it higher than necessary doesn't hurt rendering time much, only this many particles can exist at once


### Optional

`passes motionblurquality`

Sets the number of rendering passes per frame, 4 or more is needed for smooth motion

`spritetype value`

Sets the type of sprite to create, can be `vp_parallel_upright`, `facing_upright`, `vp_parallel`, `oriented`, or `vp_parallel_oriented` (default: `vp_parallel`)

> **ⓘ Note**
>
> GLQuake only supports vp_parallel (e.g., explosion) and oriented (e.g., bullet holes on walls), DarkPlaces and software Quake support all types.


### Others

All commands below this point are normal script commands and can be used at any time.

`imagebrightness min max`

Sets the image brightness range for the current frame (this is not per pass), for instance `-0.2 0.8` would offset the image brightness toward black, removing some of the darker shades, this is useful to brighten or darken the image before it is saved into the file, it is inadvisable to set min above 0, but otherwise no real limits.

`camera x y z`

Sets the location of the camera, usually x and y are best left 0, and z should be negative, this is an offset from the scene, for instance if you increase the frame size and want something to appear at the same scale, move the camera back farther as well.

`gravity x y z`

Sets the gravity for the scene, affecting all particles, this is an acceleration, experiment to see how to use it.

`airfriction value`

Sets the air friction of the scene, affecting all particles with a mass > 0, this is a deceleration, experiment to see how to use it, 1.0 airfriction * 1.0 mass means the particle will mostly stop moving after 1.0 units of time.

`worldbox minx miny minz maxx maxy maxz`

Defines a box that the particle simulation takes place in, particles will bounce off the sides of this box (losing no energy), this can be useful for some effects.


### Control Commands for Looping

`label name`

Sets a label in the script, this can be jumped to by `goto`, tell me if you need gosub or anything else more advanced (loops for example).

`goto name`

Jumps to a label of the same name, useful for simple loops, note this has no count and never ends if used for a loop.

`wait howlong`

Waits this many 'time units' (whatever your time scale might be).

`time timetowaitfor`

Waits for this time to be reached, not useful in loops as it will do nothing once this time is reached.


### Particle Commands

`rendertype value`

Sets the type of particle rendering to use (affects only particles spawned after it, not the entire scene), types are: 0 = circle (no shading, simply a circle), 1 = globe (circle that fades linearly from full alpha to none at edge), 2 = glow (has to be seen to be understood, large radius but very rapid falloff, this is realistic rendering for fire effects, this lights the entire image, and is often used with imagebrightness to force the dark areas to black, ensuring a black edge around the image).

`area x y z minradius maxradius`

Particles are spawned within the specified radius (distance range actually, if minradius is not zero it can make rings and such) of the specified location.

`velocity x y z minradius maxradius`

Same except this defines how far the particles can move in 1.0 time units (whatever your time scale happens to be), capable of ring waves and such.

`decay start end`

Sets the brightness decay of particles, 1.0 0.0 would make particles that start at 100% size, and end at 0.0 size, you can also do strange things like 0.5 1.0...

`life min max`

How many time units the particles will live for, they will simply vanish when this expires, has no effect on the flare size.

`size min max`

How large the particles are, the best settings depend on the camera distance and desired effect...  play around with it to figure it out.

`color red green blue alpha`
`color1 red green blue alpha`
`color2 red green blue alpha`

Defines the color of the spawned particles, normally these are in the range 0-256, the last (alpha) is special, it determines the opacity of the particle (how solid it is), usually 128 to 256, but can be just about anything, this does not affect the color, only how it is blended with the background.

The difference between color, color1, and color2 is this: color sets the color directly, color1 and color2 define a range of colors (very useful for fire effects), each particle will be a random blend between them (complete color blend, not per component, just one random number scaler for the whole thing).

> **ⓘ Note**
>
> when saving as a normal quake 8bit sprite, pixels that are mostly transparent are replaced by the quake transparency color, 32bit sprites and targa images are full image quality)

`pressure start end`

Defines the pressure over the course of the particle's life, this pushes other particles away from it (warning: this slows down physics, using it on large numbers of particles is inadvisable), note that this is limited to above 0 (so if it goes negative, it is clamped to 0, making it have no effect, this behavior is useful for tricks).

`weight start end`

Defines the weight over the course of the particle's life, this pulls other particles toward it (warning: this slows down physics, using it on large numbers of particles is inadvisable), this is subtracted from the pressure, and is limited to above 0 just like pressure, again useful for tricks.

> **ⓘ Note**
>
> The result of pressure - weight is not clamped to 0, only the values themselves are, so if using pressure 100 -100 weight 20 20, it will fade from 100 to -100 over the course of the particle life, and when that becomes negative it is forced to 0 instead, resulting in the particle's pressure effect fading from 80 (pushing) to -20 (pulling), and staying -20 for slightly more than half of the particle's life).

`mass min max`

How much the particle is influenced by weight/pressure effects, this is a scale value, defaults to range 1.0 to 1.0.

`seed randomnumberseed`

This command will set the random number generator seed to the value, ensuring predictable results until the next seed command, very very useful for repeating effects (looping burning animations for example), although this is not 'required', I highly recommend using it before spawning any particles.  note: neglecting to set this does not make random results.

`spawn numberofparticles`

Spawns the specified number of particles using the current settings

`boxspawn xparticles yparticles zparticles minx miny minz maxx maxy maxz`

Creates a regular grid of particles (number of particles on each axis is controlled by `<xyz>` particles parameters) inside the specified box (min to max).

`clear`

Destroys all particles instantly, mainly useful if you wish to render multiple scenes with one script.


If any other commands are desired, let me know.


## Examples

Here is a one-line bash command to run all the example script configurations:

```bash
find examples -iname "*.txt" -exec sh -c 'name=$(basename -s .txt {}); mkdir -p "examples/$name"; cd "examples/$name"; ../../lhfire ../$name.txt' \;
```
