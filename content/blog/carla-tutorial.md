---
title: "Carla Simulator Tutorial (Python API)"
date: 2023-06-15T14:54:26+01:00
draft: false
author: ""
categories: "Autonomous Driving"
tags: ["Carla"]
thumbnail: "images/thumbnail/carla.png"
headline: 
    enabled: true
    background: ""
---

A step-by-step tutorial that uses 9 examples to help you get familiar with Carla APIs.

<!--more--> 

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/carla.gif)




## Carla Installation

[**CARLA**](https://carla.org/) is an open-source simulator **(Unreal Engine)** for autonomous driving research. You can find the latest version of Carla on the GitHub release page. At the time of writing, the latest version is v0.9.14.

https://github.com/carla-simulator/carla/releases

The installation of Carla is straightforward: download the package that matches your operating system (Linux / Windows) and then extract the package.

```
Release 0.9.14:

- [Ubuntu] CARLA_0.9.14.tar.gz
- [Ubuntu] AdditionalMaps_0.9.14.tar.gz

- [Windows] CARLA_0.9.14.zip
- [Windows] AdditionalMaps_0.9.14.zip
```

Carla provides 12 Town maps in total, while **Town06**, **Town07**, and **Town10** are released in separate tar/zip files. You can install additional maps by extracting the **AdditionalMaps.zip** file to the Carla root folder. A full installation of Carla costs **~20GB**, including additional maps.



{{< hint info >}}
**Info**  
It is also possible to install Carla via `sudo apt-get install carla-simulator`. But I prefer to use the latest release on GitHub.

{{< /hint >}}



The Carla Simulator can be launched using:

```
# Start the Carla Simulator (high quality)
./CarlaUE4.sh -quality-level=epic -resx=800 -resy=600

# If you don't have a powerful GPU (low quality)
./CarlaUE4.sh -quality-level=low -resx=800 -resy=600
```

By default, the simulator launches a **spectator** with an empty world (no pedestrian, no driving vehicle). You can navigate around using the keyboard (<kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd><kbd>Q</kbd><kbd>E</kbd>) and mouse. 

![](https://carla.readthedocs.io/en/latest/img/tuto_G_getting_started/flying_spectator.gif)

We can spawn pedestrians and vehicles using the python client library.



## Python Client Library

The python client library is available on **[pypi](https://pypi.org/project/carla/)**, which can be installed using `pip`.

```
pip install carla==0.9.14
```

Carla provides implementations of several autonomous driving agents that can do planning and control. To use the **agent** module, we need to update environment variables.

```
# Append to ~/.bashrc
export CARLA_ROOT=/home/wuhanstudio/CARLA_0.9.14
export PYTHONPATH="${CARLA_ROOT}/PythonAPI/carla/"
```

Next, let's write some Python code to interact with Carla Simulator.

{{< hint info >}}
**Info**  
The following examples use **Carla v0.9.14**, but they also work with other versions that have compatible APIs. The source code is available on GitHub: [Source Code](https://github.com/wuhanstudio/carla-tutorial).

{{< /hint >}}

<hr />


## Example 01: Get Started

First, let's clone the GitHub repo.

```
git clone https://github.com/wuhanstudio/carla-tutorial
```

The first example `01_get_started.py` (~10 lines of code) spawns a vehicle in autopilot mode, and the spectator automatically follows the vehicle.

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/01_get_started.gif)

The implementation is quite simple. We first connect to the Carla Simulator listening on port 2000.

```
# Connect to Carla
client = carla.Client('localhost', 2000)
world = client.get_world()
```

Then we get the blueprint of a vehicle from the library. If you are an Unreal game developer, you should find it familiar.

```
# Get a vehicle from the library
bp_lib = world.get_blueprint_library()
vehicle_bp = bp_lib.find('vehicle.lincoln.mkz_2020')
```

Finally, we can spawn a vehicle at a desired location and set the vehicle to be in autopilot mode. The Carla Simulator provides a series of suggested spawn points which can be obtained using the function `get_spawn_points()`.

```
# Get a spawn point
spawn_points = world.get_map().get_spawn_points()

# Spawn a vehicle
vehicle = world.try_spawn_actor(vehicle_bp, random.choice(spawn_points))

# Autopilot
vehicle.set_autopilot(True) 
```

Since the spawned vehicle is driving around, we need to update the spectator viewpoint constantly in a `while` loop to follow the vehicle.

```
# Get world spectator
spectator = world.get_spectator() 

while True:
        transform = carla.Transform(vehicle.get_transform().transform(carla.Location(x=-4,z=2.5)),vehicle.get_transform().rotation) 
        spectator.set_transform(transform) 
        time.sleep(0.005)
```

The last thing I would like to mention is that: Do not forget to destroy the vehicle before terminating the program. 

```
vehicle.destroy()
```

If you forget to destroy the vehicle, the street can get crowded fast.



## Example 02: Synchronous Mode





## Example 03: RGB Camera





## Example 04: More Cameras



![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/sensors.gif)





## Example 05: Open3D Lidar





## Example 06: Traffic Manager





## Example 07: 3D Bounding Boxes





## Example 08: Draw Waypoints





## Example 09: Basic Navigation


