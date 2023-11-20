---
title: "The CARLA Coordinate System"
date: 2023-11-19T10:57:39Z
draft: false
author: "Han Wu"
categories: "Autonomous Driving"
tags: ["CARLA"]
thumbnail: "images/thumbnail/coordinate.png"
math: true
headline: 
    enabled: True
    background: ""
---

A summary of CARLA Coordinate System (Global, Camera, Image).

<!--more--> 

## Introduction

To project the location of a 3D vehicle $O_{world}=(x, y, z, 1)$ onto the 2D camera image $O_{image}=(u, v, w)$, we need to be familiar with the CARLA coordinate system.

$$O_{image} = K[R|t] O_{world}$$

- Global Coordinates: $O_{world}$ is available using the API **vehicle.get_location()**;
- Extrinsic Matrix: $[R|t]$ is provided by the API **camera.get_transform().get_inverse_matrix()**;
- Intrinsic Matrix: $K$ can be constructed from camera properties **(w, h, fov)**.

![](../../images/carla-coordinate/overview.png)

## Global Coordinate

CARLA is developed using the Unreal Engine, which uses a coordinate system of **x-front , y-right , z-up** (left-handed).

We can get the world location (x, y, z, 1) of a vehicle using the API `vehicle.get_location()`.

```
>> vehicle.get_location()
<carla.libcarla.Location object at 0x0000020E1CF58A50>
x: -110.14263153076172
y: -7.242839813232422
z: -0.005238113459199667
```

## Extrinsinc Matrix

The extrinsic matrix calculates the position of a vehicle relative to the camera.

$$O_{camera} = [R|t] O_{world}$$

```
# Get the extrinsic matrix 
world_2_camera = np.array(camera.get_transform().get_inverse_matrix())
```

## Intrinsic Matrix

We also need the intrinsic matrix K to project the relative position $O_{camera}$ to image coordinates $O_{camera} = (u, v, w)$.

$$O_{image} = K O_{camera}$$

Intrinsic Matrix: 

$$
K = 
\begin{bmatrix}
      f & 0 & \frac{w}{2} \\\\ 
      0 & f & \frac{h}{2} \\\\ 
      0 & 0 & 1
\end{bmatrix}
$$

```
def build_projection_matrix(w, h, fov, is_behind_camera=False):
    focal = w / (2.0 * np.tan(fov * np.pi / 360.0))
    K = np.identity(3)

    if is_behind_camera:
        K[0, 0] = K[1, 1] = -focal
    else:
        K[0, 0] = K[1, 1] = focal

    K[0, 2] = w / 2.0
    K[1, 2] = h / 2.0
    return K
```

## Full Example:

> Example 07: https://github.com/wuhanstudio/carla-tutorial

```
def get_image_point(loc, K, w2c):
    # Calculate 2D projection of 3D coordinate

    # Format the input coordinate (loc is a carla.Position object)
    point = np.array([loc.x, loc.y, loc.z, 1])
    # transform to camera coordinates
    point_camera = np.dot(w2c, point)

    # New we must change from UE4's coordinate system to an "standard"
    # (x, y ,z) -> (y, -z, x)
    # and we remove the fourth componebonent also
    point_camera = np.array(
        [point_camera[1], -point_camera[2], point_camera[0]]).T

    # now project 3D->2D using the camera matrix
    point_img = np.dot(K, point_camera)

    # normalize
    point_img[0] /= point_img[2]
    point_img[1] /= point_img[2]

    return point_img
```
