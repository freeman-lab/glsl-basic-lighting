# glsl-basic-lighting

Minimal GLSL function for applying light sources to a material. Designed for use as a shader component with [`glslify`](https://github.com/stackgl/glslify), with struct types defined by [`glsl-basic-light`](https://github.com/freeman-lab/glsl-basic-light) and [`glsl-basic-material`](https://github.com/freeman-lab/glsl-basic-material).

# install

To make avaialble in your project

```bash
npm install glsl-basic-lighting --save
```

# example

Define the function in your fragment shader

```glsl
#pragma glslify: BasicLighting = require('glsl-basic-lighting')
```

then use it to apply a light to a material

```glsl
vec3 viewpoint = eye - vposition;
vec3 result = BasicLighting(light, material, vnormal, vposition, viewpoint);
gl_FragColor = vec4(result, 1.0);
```

if you have multiple lights, you can use an array to apply them one at a time

```glsl
vec3 viewpoint = eye - vposition;
vec3 result = vec3(0.0)
for (int i = 0; i < N; ++i) {
    result += BasicLighting(lights[i], material, vnormal, vposition, viewpoint);
}
gl_FragColor = vec4(result, 1.0);
```

# algorithm

Standard methods are used to compute the contribution of each light souce as either point, directional, or spot, depending on the parameters. For a good overview, see this [tutorial](http://www.tomdalling.com/blog/modern-opengl/07-more-lighting-ambient-specular-attenuation-gamma/).

The primary parameter of the light is the `position`, a `vec4` in homogenous coordinates. The fourth element determines whether the light is directional or not. 

- If the fourth element is `0.0`, it will produce directional light, and the position vector will be treated as a direction. 
- If the fourth element is `1.0`, it will be treated as a point light source. 

For point light sources, the parameters `target` and `cutoff` can additionally be used to create a spot light, i.e. light restricted to a cone, where `target` is where cone is pointing and `cutoff` is the angular direction. If `cutoff` is `180` there will be no restriction.

The final intensity is determined by adding four `material` `vec3` components: `emissive`, `ambient`, `diffuse`, and `specular`, each scaled appropriately by the calculated coefficients. The diffuse and specular coefficients are computed using [`glsl-diffuse-oren-nayar`](https://github.com/stackgl/glsl-diffuse-oren-nayar) and [`glsl-specular-blinn-phong`](https://github.com/stackgl/glsl-specular-blinn-phong), controlled by the `material` parameters `shininess` and `roughness`. If either the `diffuse` or `specular` material properties are set to `[0.0, 0.0, 0.0]` these properties will be ignored.

# API

#### `pragma glslify: BasicLight = require('glsl-basic-light')`

#### `BasicLight(light, material, vnormal, vposition, viewpoint)`

Inputs
- `light` `struct` instance of `glsl-basic-light`
- `material` `struct` instance of `glsl-basic-material`
- `normal` `vec3` surface normal for the vertex
- `position` `vec3` position of the vertex
- `viewpoint` `vec3` vector from the material surface torwards the camera

Returns
- `vec3` the computed for the material