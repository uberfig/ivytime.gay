+++
title= "A little dive into NPR shading in Godot"
date= 2023-07-25
draft= false
description = "I tried my hand at NPR shading with the early Godot 4 releases"
[extra]
has_preview = true
preview_img = "/img/finalResult.png"
+++

When I was playing around with the early Godot 4 releases I stumbled upon a great NPR video from Useless Game Dev and I tried my hand at replicating it in godot

{{ youtube(id="jlKNOirh66E", autoplay=false) }}

as it currently stands I was able to get similar behavior but I didn't quite get the same effect for shadows

![final result](/img/finalResult.png "screenshot of the final result of the shader")

the following shader is handling the sobel filtering for both the depth and normal buffers. I took this combined approach to allow for better drawing of lines and the ability for a material to draw lines over itself when it makes sense

![arm overlap](/img/OverlapArm.png "screenshot highlighting that the depth pass will allow smooth materials to draw lines when overlapping themself like most 2d drawings")

## sobel filtering

for the following to work, it should be applied to a meshinstance with a new quadmesh and the geometry Extra Cull Distance property should be set to max. The shader will handle snapping the mesh to the camera and filtering the buffer behind it

```glsl
shader_type spatial;
render_mode unshaded, cull_disabled;

uniform sampler2D DEPTH_TEXTURE : hint_depth_texture, filter_nearest;
uniform sampler2D SCREEN_TEXTURE : hint_screen_texture, filter_nearest;
uniform sampler2D NORMAL_TEXTURE : hint_normal_roughness_texture, filter_nearest;

uniform vec3 line_color : source_color;

uniform float noise_intensity = 0.5;

//a noise texture shopuld be provided here for the wobbly line effect
uniform sampler2D noise;
uniform bool use_noise = true;

uniform float normalcutoff =0.001;
uniform float depthcutoff =0.018;

//values used for the sobel filter
uniform mat3 sx = mat3(
	vec3 (1.0, 2.0, 1.0), 
	vec3(0.0, 0.0, 0.0), 
	vec3(-1.0, -2.0, -1.0)
);
uniform mat3 sy = mat3(
	vec3 (1.0, 0.0, -1.0), 
	vec3(2.0, 0.0, -2.0), 
	vec3(1.0, 0.0, -1.0)
);


void vertex(){
	VERTEX *= 2.0;
	POSITION = vec4(VERTEX, 1.0);
}

//fragcoord is the xy pixel coord of a frag
//screen uv is is the xy uv coord of a frag (from 0 to 1)

vec2 fragToScreenUV(vec2 fc, vec2 vpsize){
	vec2 uv = vec2(fc.x/vpsize.x, fc.y/vpsize.y);
	return uv;
}

vec2 ScreenUVtoFrag(vec2 uv, vec2 vpsize){
	vec2 fc = vec2(uv.x*vpsize.x, uv.y*vpsize.y);
	return fc;
}

float getDepth(vec2 SC_UV, float proj32, float proj22){
	float depth = texture(DEPTH_TEXTURE, SC_UV).r;
	
	depth = depth*2.0-1.0;
	depth = proj32 / (depth + proj22);
	depth *= 0.01;
	
	return depth;
}

float depthSobel(ivec2 fragcoord, vec2 vp_size, float proj32, float proj22) {
	mat3 I;
	for (int i=0; i<3; i++) {
		for (int j=0; j<3; j++) {
//			vec3 sample  = texelFetch(tex, fragcoord + ivec2(i-1,j-1), 0 ).rgb;
//			I[i][j] = length(sample); 
			I[i][j] = getDepth(fragToScreenUV(vec2(fragcoord + ivec2(i-1,j-1)), vp_size), proj32, proj22); 
		}
	}
	
	float gx = dot(sx[0], I[0]) + dot(sx[1], I[1]) + dot(sx[2], I[2]); 
	float gy = dot(sy[0], I[0]) + dot(sy[1], I[1]) + dot(sy[2], I[2]);
	
	float g = sqrt(pow(gx, 2.0)+pow(gy, 2.0));
	return g;
}

float normalSobel(ivec2 fragcoord, sampler2D tex) {
	mat3 I;
	for (int i=0; i<3; i++) {
		for (int j=0; j<3; j++) {
			vec3 sample  = texelFetch(tex, fragcoord + ivec2(i-1,j-1), 0 ).rgb;
			I[i][j] = length(sample); 
//			I[i][j] = sample.r; 
		}
	}
	
	float gx = dot(sx[0], I[0]) + dot(sx[1], I[1]) + dot(sx[2], I[2]); 
	float gy = dot(sy[0], I[0]) + dot(sy[1], I[1]) + dot(sy[2], I[2]);
	
	float g = sqrt(pow(gx, 2.0)+pow(gy, 2.0));
//	float g = max(gx, gy);
	return g;
}

float sobel(ivec2 fragcoord, sampler2D tex) {
	mat3 I;
	for (int i=0; i<3; i++) {
		for (int j=0; j<3; j++) {
			vec3 sample  = texelFetch(tex, fragcoord + ivec2(i-1,j-1), 0 ).rgb;
			I[i][j] = length(sample); 
		}
	}
	
	float gx = dot(sx[0], I[0]) + dot(sx[1], I[1]) + dot(sx[2], I[2]); 
	float gy = dot(sy[0], I[0]) + dot(sy[1], I[1]) + dot(sy[2], I[2]);
	
	float g = sqrt(pow(gx, 2.0)+pow(gy, 2.0));
//	float g = max(gx, gy);
	return g;
}

void fragment(){
	
//	vec2 sampleFrag = FRAGCOORD.xy + texture(noise,SCREEN_UV).rg;
//	vec2 sampleUV = SCREEN_UV + texture(noise,SCREEN_UV).rg;
	vec2 sampleUV;
	if (use_noise) {
		sampleUV = SCREEN_UV + ((texture(noise,SCREEN_UV).rg - vec2(0.5,0.5)) * noise_intensity / 255.0);
	} else {
		sampleUV = SCREEN_UV;
	}
	float g1 = normalSobel(ivec2(sampleFrag), NORMAL_TEXTURE);
	float g2 = depthSobel(ivec2(sampleFrag), VIEWPORT_SIZE,PROJECTION_MATRIX[3][2],PROJECTION_MATRIX[2][2]);
	
	float depth = getDepth(fragToScreenUV(FRAGCOORD.xy,VIEWPORT_SIZE),PROJECTION_MATRIX[3][2],PROJECTION_MATRIX[2][2]);
	float dcutt = depthcutoff*depth;
	
	float n_cutt = normalcutoff/depth*0.01;
	
	if (g2>dcutt) {
		ALBEDO = line_color;
	} else if (g1>n_cutt) {
		ALBEDO = line_color;
	} else {
		//you can swap between these to see the normal buffers and depth buffers with this shader applied
//		ALBEDO = texture(NORMAL_TEXTURE, sampleUV).rgb;
		ALBEDO = texture(SCREEN_TEXTURE, sampleUV).rgb; 
//		ALBEDO = vec3(depth);
	}
//	ALBEDO = texture(NORMAL_TEXTURE, sampleUV).rgb;
}
```
for the noise texture, I would recommend a new noisetexture2d  with fastnoiselite creating perlin noise with a frequency around 0.049

whatever you wind up doing, I would recommend using soft and gradual noise to give the lines a more gradual waves in the final output

## solid color shader


when working with curved geometry, the sobel filter will overreact and draw over it more than nessary. we can eliminate this by setting the solid norm property which will set the normal map to a solid color. this may make your lighting act a bit strange, but it shouldn't be too noticeable. Once godot gets support for a stencil buffer, this could be solved in a better way, but if anyone has an idea for a cleaner workaround, feel free to open an issue on my blog's github repo [here](https://github.com/uberfig/ivytime.gay/issues)

pictured is solid norm on the left and without it on the right
![comparison](/img/comparison.png "the solid norm has lines as what would be expected and the example without it has a large amount of line artifacts because of variations in proper smooth shading")

```glsl
shader_type spatial;
render_mode diffuse_burley;

uniform vec3 color : source_color = vec3(1.0);
uniform bool useLineart = false;
uniform bool useSolidNorm = false;
uniform sampler2D lineart : filter_nearest, source_color;
uniform float cutoff;
uniform float solid_norm;

void fragment() {
	
	if (useLineart) {
		float line_intensity = texture(lineart,UV).r;
		if (line_intensity < cutoff) {
			ALBEDO = vec3(0.07);
			NORMAL = vec3(0.1);
		} else {
			ALBEDO = color;
			NORMAL = vec3(solid_norm);
		}
		
	} else {
		ALBEDO = color;
		if (useSolidNorm){
			NORMAL = vec3(solid_norm);
		}
	}
	
}

uniform float cutt1 = 0.1;
uniform float cutt2 = 0.05;
uniform float cutt3 = 0.01;
void light() {
	vec3 diffused = DIFFUSE_LIGHT + dot(NORMAL, LIGHT) * ATTENUATION * ALBEDO;
	if (length(diffused) > cutt1) {
		DIFFUSE_LIGHT = vec3(1);
	} else if (length(diffused) > cutt2) {
		DIFFUSE_LIGHT = vec3(0.5);
	} else if (length(diffused) > cutt3) {
		DIFFUSE_LIGHT = vec3(0.2);
	} else {
		DIFFUSE_LIGHT = vec3(0);
	}
}
```

## 2d billboard shader

this is a useful little shader that you can use to incorporate 2d sprites into the scene. If you keep your lineart and coloring on seperate layers in your drawing platform of choice, you can punch these both into the texture and lineart properties and it will draw the lineart onto the sprite like the 3d lineart. This shader will also make the sprite turn to face the camera along the z axis, there seems to be a bit of an issue with looking at it from behind but this could be worked around by making a copy and rotating it 180° so that one is always visible 

```glsl
shader_type spatial;
render_mode unshaded, cull_disabled;

uniform sampler2D _texture : filter_nearest, source_color;
uniform sampler2D _lineart : filter_nearest, source_color;
uniform sampler2D _normal : hint_normal;

uniform vec4 albedo : source_color;

uniform bool useLineart = true;

uniform float cutoff : hint_range(0.0, 1.0) = 0.5; 

void vertex() {
	MODELVIEW_MATRIX = VIEW_MATRIX * mat4(vec4(normalize(cross(vec3(0.0, 1.0, 0.0), INV_VIEW_MATRIX[2].xyz)), 0.0), vec4(0.0, 1.0, 0.0, 0.0), vec4(normalize(cross(INV_VIEW_MATRIX[0].xyz, vec3(0.0, 1.0, 0.0))), 0.0), MODEL_MATRIX[3]);
}

void fragment() {
	NORMAL_MAP = texture(_normal, UV).xyz * vec3(2.0,2.0,1.0) - vec3(1.0,1.0,0.0);
	NORMAL_MAP_DEPTH = 1.0; //float
	vec3 tex1 =texture(_texture,UV).rgb;
	vec3 tex2 =texture(_lineart,UV).rgb;
	vec3 color;
	if (useLineart) {
		if (length(tex2)<length(vec3(cutoff))) {
//			color = tex2;
			color = vec3(0);
		} else {
			color = tex1;
		}
	} else {
		color = tex1;
	}
	ALBEDO = color;
	ALPHA = texture(_texture,UV).a;
}

```
