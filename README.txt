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
reflect geometry that is being reflect off of other geometry.

For example, the cube in the center of the world will reflect
the blue plane under it, and the blue plane will have a reflection
of the red cube. However, the red cube in the reflection of the
blue plane, will not have a reflection of the blue plane.

If we want to go the extra mile, we can have:
reflections of reflections of reflections of reflections of reflections...
We can have it go infinitely, but for now let's just keep it simple

This tutorial is reflections of geometry, on other geometry.

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

In the Bumpy Skybox tutorial of the "more graphics" section, you might remember we reflected
vectors that were from the camera to the point of an object, and then the reflected value
would project into the skybox, and return a color (which was from the skybox).
We used a command "reflect(dir, normal)" to reflect the direction FROM camera TO skybox,
over the normal vector of the object. Now with Ray Tracing, while we're actually shooting rays
through the world, we can do this again.

To reflect a ray off of the triangle that the ray hit (in this case, the eye's ray hitting a triangle),
we use the "reflect" command that GLSL has for us
	reflect(dir, triangles[eyeHitPoint.index].normal);
	
We create a hitInfo to see if this ray hits anything
We check every triangle in the scene to see what it hits
	if(intersectTriangles(eyeHitPoint.point, reflectedEyeToPoint, reflectHit))
	
If it hits nothing, return vec3(0), which is the background color.
If it hits a triangle, get the color of the triangle
	triangles[reflectHit.index].color
However, this doesn't give us any of the light information or shadow
data of the geometry that is reflected off of geometry.

So, for the geometry that is reflected off of geometry,
we calculate lighting data:
	vec3 reflectPointToLight = lightPos - reflectHit.point;
	float diffuse = max(0, dot(triangles[reflectHit.index].normal, reflectPointToLight)) / pow(length(reflectPointToLight), 2);

Then we multiply diffuse by the color of the reflected geometry, and 
we return that pixel color, which then gets "mixed" in "trace(...)", 
and then the result of trace() gets sent to the screen
 
How to improve:
Many ways to improve:

More Reflections
	Try to have reflections, of reflections, of reflections...

Uniform buffers
	Keep the triangles array, make it empty
	Put the plane, and each cube in a seperate uniform buffer
	Give them each a model matrix
	Use a compute shader to move them, each with their own model matrix
	Have the compute shader write to the triangles array, just like
	how the compute shader wrote the the vertex buffer in Basic Compute Particles
	
Optimization:
	Give each model a spherical radius, so that rays check for collision
	with the spheres, before checking for collision with each polygon that
	is inside the sphere

Textures
	This was between lines 329 and 343 in Reflection FragmentShader.glsl
	Between float lightIntensity = 6.0; and vec3 pixColor = ...

	i.point is the 3D coordinate that a ray hits a triangle
	triangles[i.index].a is one point on the triangle
	triangles[i.index].b is one point on the triangle
	triangles[i.index].c is one point on the triangle
	UV coordinates dont exist yet
	Use barycentric coordinates to get (a+b+c=1) variables
	to compare points of triangle to point intersecting
	Then use (a+b+c=1) to compare points of triangle's UVs
		to get the UV coordinate of the point you hit
	For Debugging, do pixColor = uv * 0.1
		Then do vec3 pixColor = sample(texture, uv) * 0.1
	However
		This will not change the color of squares in reflection
		At line 285 and 296, triangles[eyeHitPoint.index].color
		needs to be replaced with reflectHit.point (like i.point)
		to calculate UVs all over again
		
Skybox
	On lines 274, in Reflection FragmentShader.glsl
	When you see if(intersectTriangles ...
	Should have "else ..." to get skybox color
	We already have direction, we just need color from cubemap
	We do NOT change the if(intersectTriangles ... at line 241,
	because that is for shadows
	On line 346, change "return vec4(0,0,0,1)" to 
	return the skybox color. This will be the actual background
	wallpaper skybox
