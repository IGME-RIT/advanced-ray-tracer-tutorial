Documentation Author: Niko Procopi 2019

This tutorial was designed for Visual Studio 2017 / 2019
If the solution does not compile, retarget the solution
to a different version of the Windows SDK. If you do not
have any version of the Windows SDK, it can be installed
from the Visual Studio Installer Tool

Welcome to the Reflection Ray Tracing Tutorial!
Prerequesites: Shadow Ray Tracing

In this tutorial, the only file we are changing is the
fragment shader. After the 1st tutorial, we added lighting
features by adding more rays. In this tutorial, we will add
reflections, by adding even more rays, which check for collision
with all polygons. The amount of power to make this run
is quickly adding up. Thank goodness we don't have 100 triangles.

We will change "trace(...)" again, and we will add a new function
called "addReflectionToPixColor". 

Read the next few lines very carefully
What this function will do: 
reflect geometry off of other geometry
What this function will NOT do:
reflect geometry that is being reflected off of other geometry.

That means after a ray goes from eye to surface, it only
bounces one time to look for geometry to reflect. It will
not bounce more than once. (Looped reflections come next).

For example, the cube in the center of the world will reflect
the blue plane under it, and the blue plane will have a reflection
of the red cube. However, the red cube in the reflection of the
blue plane, will not have a reflection of the blue plane.

In the "trace" function, we change the inside of our "for(j)" loop.

In the last tutorial, if one of the eye's rays hit a polygon,
then more rays would be casted from that polygon to the lights 
(in the direction of the lights), and then we added that color to the 
pixel. 
	vec3 lightColor = addLightColorToPixColor(light[j], dir, i, lightIntensity);

In this tutorial, if one of the eye's rays hit a polygon, we
get color from the lights that shine on the polygon, and then
we get another color, which determines what geometry reflect off
of the polygon that was originally hit by the eye's ray.
	vec3 reflection = addReflectionToPixColor(light[j], dir, i, lightIntensity);

Once we have those two colors, we use "mix", a GLSL command to blend the colors
together, based on a reflectivity variable that we create. A 0.0 reflectivity
will reflect nothing, a 1.0 will act as a perfect mirror, and a 0.5 will be a mixture.
	pixColor += mix(lightColor, reflection, reflectionLevel);
	
Then we return the pixel color, and that goes to the screen.

[New function]
vec3 addReflectionToPixColor(vec3 lightPos, vec3 pointToLight, vec3 dir, hitinfo eyeHitPoint, float lightIntensity)

Just like previous tutorials where we used "reflect" in tutorials from the "More Graphics"
section, and just how we used the "reflect" command in the last tutorial for specular
lighting, we will use it again in this tutorial.

After a ray hits a triangle (in this case, it was from our eye to the triangle),
To reflect a ray off of the triangle that the ray hit,
we use the "reflect" command that GLSL has for us
	reflect(dir, triangles[eyeHitPoint.index].normal);
	
We create a hitInfo to see if this ray hits anything
We check every triangle in the scene to see what it hits
	if(intersectTriangles(eyeHitPoint.point, reflectedRayToPoint, reflectHit))
	
If it hits nothing, return vec3(0), which is the background color.
If it hits a triangle, then we need to render the point on that
triangle, just like how we rendered points that the eye's rays hit 
before reflections
	return addLightColorToPixColor(lightPos, reflectedRayToPoint, reflectHit, lightIntensity);

Then we multiply diffuse by the color of the reflected geometry, and 
we return that pixel color, which then gets "mixed" in "trace(...)", 
and then the result of trace() gets sent to the screen

How to improve:
Make a loop that reflects geometry, off of geometry, off of geometry...
	To do this, rays need to bounce several times (next tutorial)
