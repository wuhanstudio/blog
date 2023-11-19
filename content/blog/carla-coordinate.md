---
title: "The CARLA Coordinate System"
date: 2023-11-19T10:57:39Z
draft: false
author: "Han Wu"
categories: "Autonomous Driving"
tags: ["CARLA"]
thumbnail: "images/thumbnail/coordinate.png"
headline: 
    enabled: True
    background: ""
---

A summary of CARLA Coordinate System (Global, Camera, Image).

<!--more--> 

## Introduction

Using the CARLA simulator, we can use Python API to obtain ground truth bounding boxes of each vehicle, thus saving the labeling time. To project a 3D vehicle onto the 2D image captured by the camera, we need to be familiar with the CARLA coordinate system 

![](../../images/carla-coordinate/overview.png)

## Global Coordinate

CARLA is developed using the Unreal Engine, which uses a coordinate system of **x-front , y-right , z-up**.

We can get the world location (x, y, z) of a vehicle using the API `vehicle.get_location()`.

```
>> vehicle.get_location()
<carla.libcarla.Location object at 0x0000020E1CF58A50>
x: -110.14263153076172
y: -7.242839813232422
z: -0.005238113459199667
```

## Sensor Coordinate

To get the relative position of a vehicle to the camera, we need to do a transform:

```
# Get the camera matrix 
world_2_camera = np.array(camera.get_transform().get_inverse_matrix())
```

This is also known as the **extrinsic matrix**.


## Image Coordinate (OpenCV)

To project the vehicle to image coordinates (u, v, w), we also need the **intrinsic matrix** K.

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

Finally, we can do the projection:

![](../../images/carla-coordinate/math.png)

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
