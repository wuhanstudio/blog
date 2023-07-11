---
title: "CARLA Simulator - Python API (Intermediate)"
date: 2023-07-11T16:00:41+01:00
draft: false
author: "Han Wu"
categories: "Autonomous Driving"
tags: ["CARLA"]
thumbnail: "images/thumbnail/carla.png"
headline: 
    enabled: True
    background: ""
---

A step-by-step tutorial that uses 3 intermediate examples to help you get familiar with CARLA APIs.

<!--more--> 

## Introduction

In the [previous post](https://blog.wuhanstudio.uk/blog/carla-tutorial-basic/), I introduced how to connect to CARLA Simulator using Python API, the difference between synchronous and asynchronous mode, and how to get images from an RGB camera. In this post, I will introduce other types of cameras and sensors.

More details will be updated soon.

## Example 04: More Cameras

This example introduces how to capture sensor data from multiple cameras. You can get all supported camera types from the blueprint library.

```
# Print all camera types
for bp in bp_lib.filter("camera"):
    print(bp.id)
```

At the time of writing, CARLA supports six types of cameras: RGB, Semantic Segmentation, Instance Segmentation, Depth, DVS, and Optical Flow camera.

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/04_more_cameras.gif)

```
python 04_more_cameras.py
```

We follow the same procedure to spawn six cameras from blueprints, and then attach all cameras to the vehicle.

```
# Create a camera floating behind the vehicle
camera_init_trans = carla.Transform(carla.Location(x=-5, z=3), carla.Rotation(pitch=-20))

# Create a RGB camera
rgb_camera_bp = world.get_blueprint_library().find('sensor.camera.rgb')
rgb_camera = world.spawn_actor(rgb_camera_bp, camera_init_trans, attach_to=vehicle)

# Create a semantic segmentation camera
seg_camera_bp = world.get_blueprint_library().find('sensor.camera.semantic_segmentation')
seg_camera = world.spawn_actor(seg_camera_bp, camera_init_trans, attach_to=vehicle)

# Create a instance segmentation camera
ins_camera_bp = world.get_blueprint_library().find('sensor.camera.instance_segmentation')
ins_camera = world.spawn_actor(ins_camera_bp, camera_init_trans, attach_to=vehicle)

# Create a depth camera
depth_camera_bp = world.get_blueprint_library().find('sensor.camera.depth')
depth_camera = world.spawn_actor(depth_camera_bp, camera_init_trans, attach_to=vehicle)

# Create a DVS camera
dvs_camera_bp = world.get_blueprint_library().find('sensor.camera.dvs')
dvs_camera = world.spawn_actor(dvs_camera_bp, camera_init_trans, attach_to=vehicle)

# Create an optical flow camera
opt_camera_bp = world.get_blueprint_library().find('sensor.camera.optical_flow')
opt_camera = world.spawn_actor(opt_camera_bp, camera_init_trans, attach_to=vehicle)
```

Similarly, we use the **Queue** data structure to store sensor data. The only difference is how we handle different types of sensor data. As this tutorial focuses on getting familiar with APIs, **it is recommended to refer to the official documentation for comprehensive details on [sensor data](https://carla.readthedocs.io/en/latest/core_sensors/#cameras)**.

```
# Define camera callbacks                       
def rgb_camera_callback(image, rgb_image_queue):
    rgb_image_queue.put(np.reshape(np.copy(image.raw_data), (image.height, image.width, 4)))

def seg_camera_callback(image, seg_image_queue):
    image.convert(carla.ColorConverter.CityScapesPalette)
    seg_image_queue.put(np.reshape(np.copy(image.raw_data), (image.height, image.width, 4)))

def ins_camera_callback(image, ins_image_queue):
    ins_image_queue.put(np.reshape(np.copy(image.raw_data), (image.height, image.width, 4)))

def depth_camera_callback(image, depth_image_queue):
    image.convert(carla.ColorConverter.LogarithmicDepth)
    depth_image_queue.put(np.reshape(np.copy(image.raw_data), (image.height, image.width, 4)))

def dvs_camera_callback(data, dvs_image_queue):
    dvs_events = np.frombuffer(data.raw_data, dtype=np.dtype([
        ('x', np.uint16), ('y', np.uint16), ('t', np.int64), ('pol', np.bool_)
    ]))

    dvs_img = np.zeros((data.height, data.width, 4), dtype=np.uint8)

    # Blue is positive, red is negative
    dvs_img[dvs_events[:]['y'], dvs_events[:]['x'], dvs_events[:]['pol'] * 2] = 255

    dvs_image_queue.put(dvs_img)

def opt_camera_callback(data, opt_image_queue):
    image = data.get_color_coded_flow()
    height, width = image.height, image.width
    image = np.frombuffer(image.raw_data, dtype=np.dtype("uint8"))
    image = np.reshape(image, (height, width, 4))

    opt_image_queue.put(image)
```

**Congratulations!**  By now, you have learned how to spawn vehicles and attach sensors to vehicles. You have also learned how to receive and display sensor data from CARLA simulator. The remaining examples explore various intriguing topics that are independent of each other. **Feel free to jump directly to the section that interests you the most.**

<br />

<hr />

## Example 05: Open3D Lidar

This example is available in the official CARLA [Video Tutorials](https://www.youtube.com/watch?v=om8klsBj4rc).

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/05_open3d_lidar.gif)

```
python 05_open3d_lidar.py
```

<br />

<hr />

## Example 06: Traffic Manager

This example is available in the official CARLA [Tutorials](https://carla.readthedocs.io/en/0.9.14/tuto_G_traffic_manager/).

In [example 01](#example-01-get-started), we learned how to set a vehicle to autopilot mode. You may wonder if we can control the behavior of an autopiloting vehicle. The answer is YES: The Traffic Manager (TM) controls vehicles in autopilot mode. For example, the traffic manager can change the probability of running a red light or set a pre-defined path.

```
# Autopilot
vehicle.set_autopilot(True) 
```



![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/06_traffic_manager.gif)

```
python 06_trafic_manager.py
```

After connecting to the CARLA server that listens on port 2000, we can use `client.get_trafficmanager()` to get a traffic manager.

```
# Connect to the client and retrieve the world object
client = carla.Client('localhost', 2000)
world = client.get_world()

# Set up the TM (default port: 8000)
traffic_manager = client.get_trafficmanager()
```

{{< hint info >}}
**Info**  
It is worth noting that the **traffic manager runs on the client side**, which means it will listen on port 8000 on your local computer rather than on the server that runs the CARLA simulator.

{{< /hint >}}

As mentioned before, the traffic manager only manages vehicles in **autopilot mode**. Thus, we need to ensure the vehicle is in autopilot mode before the traffic manager takes over. Next, we can set the probability of running a red light using the API `traffic_manager.ignore_lights_percentage()`.

```
vehicle.set_autopilot(True)

# Randomly set the probability that a vehicle will ignore traffic lights
traffic_manager.ignore_lights_percentage(vehicle, random.randint(0,50))
```

By default, there is no goal for vehicles in autopilot mode, which means their path is endless. They randomly choose a direction at junctions. Using traffic manager, we can specify a path from a point list.

```
spawn_points = world.get_map().get_spawn_points()

# Create route 1 from the chosen spawn points
route_1_indices = [129, 28, 124, 33, 97, 119, 58, 154, 147]
route_1 = []
for ind in route_1_indices:
    route_1.append(spawn_points[ind].location)

traffic_manager.set_path(vehicle, route_1)
```

<img src="https://carla.readthedocs.io/en/0.9.14/img/tuto_G_traffic_manager/set_paths.png" width=40% />

We can also use the traffic manager to save computational resources. By default, physics calculations are enabled for all vehicles, which can be computationally expensive. If we enable the **Hybrid physics mode**, not all autopilot vehicles will do physics calculations.

```
# Enable hybrid physics mode to save computational resources.
TrafficManager.set_hybrid_physics_mode(True)
```

In hybrid physics mode, whether or not physics calculations are enabled depends on the distance between the autopilot vehicle and the hero vehicle, a vehicle tagged with `role_name='hero'`. Autopilot vehicles outside a certain radius of the hero vehicle will disable physics calculation, **which means they can teleport** (very interesting). Of course, the hero vehicle (usually the ego agent) won't see them teleporting, so during autonomous driving simulation, we won't see vehicles disappear and reappear through the camera attached to the hero vehicle, making the simulation more realistic while saving some computational resources.

You can find more details about traffic manager in the [official documentation](https://carla.readthedocs.io/en/latest/adv_traffic_manager/).

<br />

<hr />

## References

- Carla Official Documentation: https://carla.readthedocs.io/en/0.9.14/tutorials/
