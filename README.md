# shapekey_experiments
Goal is a facial shape manager for Blender, for shapekey based facial rigs. One line at a time.

So far I'm adding functions I'll need down the road. I also added a first, more or less functioning prototype.

In the end it should handle correctives and inbetweens based on their naming, as well as have filtered UI lists, and handle import and export of shapes.

Some terminology:
- hero shapes: simple shape keys with no drivers. Examples: jawOpen, lipCornerPuller, browLowerer, eyesClosed
- combination shapes/correctives/combos: shape keys that are driven by 2 or more hero shapes and are used as a corrective difference in deltas. Examples: jawOpen_lipCornerPuller, browLowerer_eyesClosed
- inbetweens/inbetween shapes: shape keys that are driven by a hero shape and act as an inbetween corrective. Examples: jawOpen_10, eyesClosed_50
- inbetween combos: a combination of inbetweens and combos. Examples: jawOpen_10_lipCornerPuller, eyesClosed_25_browLowerer_50
- puppet: a mesh, typically a head, with shape keys for the expressions.
- FACS: Facial action coding system

How it looks so far - 

![Screenshot 2022-12-19 100918](https://user-images.githubusercontent.com/78473045/208319836-c3cfef4a-6deb-40aa-b4a2-8fc225071fee.png)

TODO:
- clean up/re write code to be more readable and organized as it's becoming a bit messy
- filter the list on the left to only display hero shapes, re-sort alphabetically. Alternatively create a filtered list without values, like the one on the right but only with hero shapes, and different menu with the complete list with values.
- explore other export options, like alembic
- create a build function that would build a puppet based on selected shapes and their naming
- explore the possibility of updating multiple shapes at once based on naming
- explore by state updates/quickset
- explore freezing shapes
- explore multi selection


-----------
Simple flow chart for the update process:

![Untitled Diagram(5)](https://user-images.githubusercontent.com/78473045/207794114-774d9fef-8a2a-42a1-bcd8-1784db41452a.jpg)


More detailed flow chart on how the corrective update process:

![Untitled Diagram(7)](https://user-images.githubusercontent.com/78473045/207978133-cc05a728-c4c4-4e97-89f6-abdedfb2b954.jpg)

More detailed flow chart on how to get a corrective delta difference between a raw combo shape and a tweaked corrective vie shape keys:

![Untitled Diagram(4)](https://user-images.githubusercontent.com/78473045/207691890-0ae56f25-1c5b-4925-8649-5208a7960650.jpg)

Instead of the shape key approach the difference in vert position is calculated vie the difference mesh approach.

--------------

Some expressions for non linear corrective drivers. Credit goes to sourvinos and bandages from blenderartists. - 

(sqrt(s1 * s2)) * (s1 + s2) / 2 .9,.1 - .150 | .5,.5 - .25 | .3,.7 - .229

sin(pi/2 * s1)*sin(pi/2 * s2) .9,.1 - .155 | .5,.5 - .5 | .3,.7 - .405

sin(sqrt(pi/2 * s1))*sin(sqrt(pi/2 * s2)) .9,.1 - .358 | .5,.5 - .6 | .3,.7 - .549


These do look weird if there are correctives as well as inbetweens, the later having a linear activation/deactivation expression. Here's an expression that is probably used by many studios for that reason. While less correct, it's probably more suitable for animation as the result doesn't have hickups while blending deltas.

(((s2>0) * (s1)) + ((s1>0) * (s2)) )/2

-------------

More detailed flow chart on how the inbetween update process:

![Untitled Diagram(6)](https://user-images.githubusercontent.com/78473045/207964779-400980f5-fd57-4919-ad71-85a8e269d51c.jpg)

Little bit of background on the inbetween expression. (Thanks to bandages from blenderartists for that)

Take the case of 3 different keys, one to trigger fully at input 0.33, the next to interpolate in from input 0.33 to maximum at input 0.67, as the first one likewise interpolates out, and then a final one. Might as well interpolate linearly.

First thing you’re doing is a simple remap operation: input 0.33-0.67 -> SK2 0-1. First you find your range: 0.34 (0.67-0.33). You subtract your minimum bound from your input, then you divide by your range. 0.33-0.33/0.34 = 0. 0.67-0.33/0.34 = 1. Clamp it. I think there’s a python clamp(). Maybe saturate(), I dunno what they call it.

That gives you a 0-1 value out of each shapekey. How do you interpolate out? If they’re clamped, most easily by checking them in reverse order: SK2 = SK2 - SK3. SK1 = SK1 - (SK2 + SK3).

So let’s establish a little bit of nomenclature:
i = input value
SK1 = output for shapekey1
a = min bound of SK1
A = max bound of SK1
b = min bound of SK2
etc

To clarify, at i=A, you’re getting SK1 at 1.0 and no other shapekeys. At B, you’re getting SK2 at 1.0 and no other shapekeys. b=A and c=B, it just makes it easier to write.

SK3 = clamp((i-c)/(C-c))
SK2 = clamp((i-b)/(B-b)) - SK3

Etc.

Okay, just checked, no clamp. Use max(min(n, 1.0), 0.0) to clamp in Python I guess.

So SK3 = max(min((i-c)/(C-c), 1.0),0.0)

Extrapolates to however many shapekeys you want, if you can follow the reasoning.

max(min(({}-{})/({}-{}), 1),0)

------

The build puppet workflow is basically the update shape process in a loop, selected to active. The export process is the reverse, but simpler in some aspects. Here's a basic flow chart for the export process - 

![Untitled Diagram](https://user-images.githubusercontent.com/78473045/208230609-c6bde3e4-fe1b-46de-9b55-10950264533b.jpg)







