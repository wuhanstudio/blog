---
title: "CARLA Simulator - Python API (Basic)"
date: 2023-06-15T14:54:26+01:00
draft: false
author: "Han Wu"
categories: "Autonomous Driving"
tags: ["CARLA"]
thumbnail: "images/thumbnail/carla.png"
headline: 
    enabled: true
    background: ""
---

A Step-By-Step Tutorial that uses 3 Simple Examples to help you get familiar with CARLA APIs.

<!--more--> 

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/carla.gif)




## CARLA Installation

[**CARLA**](https://carla.org/)  **(Unreal Engine)** is an open-source simulator for autonomous driving research. You can find the latest version of CARLA on the GitHub release page. At the time of writing, the latest version is v0.9.14.

https://github.com/carla-simulator/carla/releases

The installation of CARLA is straightforward: download the package that matches your operating system (Linux / Windows) and then extract the package.

```
Release 0.9.14:

- [Ubuntu] CARLA_0.9.14.tar.gz
- [Ubuntu] AdditionalMaps_0.9.14.tar.gz

- [Windows] CARLA_0.9.14.zip
- [Windows] AdditionalMaps_0.9.14.zip
```

CARLA provides 12 Town maps in total, while **Town06**, **Town07**, and **Town10** are released in separate tar/zip files. You can install additional maps by extracting the **AdditionalMaps.zip** file to the CARLA root folder. A full installation of CARLA costs **~20GB**, including additional maps.



{{< hint info >}}
**Info**  
It is also possible to install CARLA via `sudo apt-get install carla-simulator`. But I prefer to use the latest release on GitHub.

{{< /hint >}}



The CARLA Simulator can be launched using:

```
# Start the CARLA Simulator (high quality)
./CarlaUE4.sh -quality-level=epic -resx=800 -resy=600

# If you don't have a powerful GPU (low quality)
./CarlaUE4.sh -quality-level=low -resx=800 -resy=600
```

By default, the simulator launches a **spectator** with an empty world (no pedestrian, no driving vehicle). You can navigate around using the keyboard (<kbd>W</kbd><kbd>A</kbd><kbd>S</kbd><kbd>D</kbd><kbd>Q</kbd><kbd>E</kbd>) and mouse. 

![](https://carla.readthedocs.io/en/latest/img/tuto_G_getting_started/flying_spectator.gif)

Next, let's spawn some pedestrians and vehicles using the python client library.



## Python Client Library

The python client library is available on **[pypi](https://pypi.org/project/carla/)**, which can be installed using `pip`.

```
pip install carla==0.9.14
```

CARLA provides implementations of several autonomous driving agents that can do planning and control. To use the **agent** module, we need to update environment variables.

```
# Append to ~/.bashrc
export CARLA_ROOT=/home/wuhanstudio/CARLA_0.9.14
export PYTHONPATH="${CARLA_ROOT}/PythonAPI/carla/"
```

Next, let's write some Python code to interact with CARLA Simulator.

{{< hint info >}}
**Info**  
The following examples use **CARLA v0.9.14**, but they also work with other versions that have compatible APIs. The source code is available on GitHub: [Source Code](https://github.com/wuhanstudio/carla-tutorial).

{{< /hint >}}

<br />

<hr />


## Example 01: Get Started

First, let's clone the GitHub repo.

```
git clone https://github.com/wuhanstudio/carla-tutorial
```

The first example `01_get_started.py` (~10 lines of code) spawns a vehicle in autopilot mode, and the spectator automatically follows the vehicle.

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/01_get_started.gif)

```
python 01_get_started.py
```

The implementation is quite simple. We first connect to the CARLA Simulator listening on port 2000.

```
# Connect to CARLA
client = carla.Client('localhost', 2000)
world = client.get_world()
```

Then we get the blueprint of a vehicle from the library. If you are an Unreal game developer, you should find it familiar.

```
# Get a vehicle from the library
bp_lib = world.get_blueprint_library()
vehicle_bp = bp_lib.find('vehicle.lincoln.mkz_2020')
```

Finally, we can spawn a vehicle at a desired location and set the vehicle to be in autopilot mode. The CARLA Simulator provides a series of suggested spawn points which can be obtained using the function `get_spawn_points()`.

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

The last thing I would like to mention is that: **Do not forget to destroy the vehicle before terminating the program**. 

```
vehicle.destroy()
```

If you forget to destroy the vehicle, the street can get crowded fast.

<br />

<hr />

## Example 02: Synchronous Mode

In the first example, CARLA runs in **asynchronous mode**, which means the simulator runs as fast as possible, without waiting for the client.

The first example updates the viewpoint of the spectator in a `while` loop. If the client runs so slow that the viewpoint is updated every one second (`time.sleep(1)`). We can clearly see that the update of the viewpoint lags behind the vehicle.

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/async_mode.gif)

In other words, if the simulation runs faster than the client in asynchronous mode, the client can be too slow to follow the vehicle.

We can use the **synchronous mode** to solve this problem. In synchronous mode, the server waits for a client tick before updating the simulation step, which means the vehicle won't update until the client updates the viewpoint and sends a tick signal. 

As you can see in the video below, the client's viewpoint synchronously follows the vehicle.

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/02_sync_mode.gif)

```
python 02_sync_mode.py
```

{{< hint warning>}}
**Warning**  
In a multiclient architecture, only one client should tick. Many client ticks will create inconsistencies between server and clients.

{{< /hint >}}

We can change the simulation mode by updating world settings:

```
# Set up the simulator in synchronous mode
settings = world.get_settings()
settings.synchronous_mode = True # Enables synchronous mode
settings.fixed_delta_seconds = 0.05
world.apply_settings(settings)
```

**Don't forget to change the mode back to synchronous before terminating the program.** Because if the simulation runs in synchronous mode, and no client sends out tick signals, the spectator will stop the simulation and freeze.

```
settings = world.get_settings()
settings.synchronous_mode = False # Disables synchronous mode
settings.fixed_delta_seconds = None
world.apply_settings(settings)
```

<br />

<hr />

## Example 03: RGB Camera

In the first example, we covered how to spawn and follow a vehicle, while the second example focused on the difference between asynchronous and synchronous modes. In autonomous driving applications, accurate object detection and tracking play a vital role in perception that help the vehicle understand the environment. Now, let's attach a camera to the vehicle and utilize OpenCV to display the captured images.

![](https://wuhanstudio.nyc3.cdn.digitaloceanspaces.com/blog/carla_tutorial/03_RGB_camera.gif)

```
python 03_RGB_camera.py
```

Similar to the vehicle spawning, we first get the blueprint of an RGB camera from the library. CARLA offers support for various camera types, and we'll explore additional options in the next example. The camera floats behind the vehicle (`carla.Location(x=-5, z=3)`) and looks downward (`carla.Rotation(pitch=-20)`). During camera spawning, we can specify its attachment to the vehicle using the parameter attach_to=vehicle.

```
## Part 2

# Create a camera floating behind the vehicle
camera_init_trans = carla.Transform(carla.Location(x=-5, z=3), carla.Rotation(pitch=-20))

# Create a RGB camera
rgb_camera_bp = world.get_blueprint_library().find('sensor.camera.rgb')
camera = world.spawn_actor(rgb_camera_bp, camera_init_trans, attach_to=vehicle)
```

It is also possible to change the camera's attributes, such as FOV (Field of View) and shutter speed (see [Documentations](https://carla.readthedocs.io/en/latest/ref_sensors/#rgb-camera)). We can get the camera's resolution using the function `get_attribute()`.

```
# Get gamera dimensions and initialise dictionary                       
image_w = rgb_camera_bp.get_attribute("image_size_x").as_int()
image_h = rgb_camera_bp.get_attribute("image_size_y").as_int()
```

To receive image data from the camera, we need to define a callback function. This function is invoked whenever a new image arrives and is responsible for handling the data. Within the callback function, we store the image data using the **Queue** data structure. The function `put()` allows us to add an image to the Queue, while 'get()' enables retrieving the image data.

```
# Callback stores sensor data in a dictionary for use outside callback                         
def camera_callback(image, rgb_image_queue):
    rgb_image_queue.put(np.reshape(np.copy(image.raw_data), (image.height, image.width, 4)))

# Start camera recording
rgb_image_queue = queue.Queue()
camera.listen(lambda image: camera_callback(image, rgb_image_queue))
```

Lastly, in the `while` loop, we can retrieve the image data from the Queue and displays the image using `cv2.imshow()`.

```
while True:
	# Display RGB camera image
    cv2.imshow('RGB Camera', rgb_image_queue.get())

    # Quit if user presses 'q'
    if cv2.waitKey(1) == ord('q'):
    	clear()
    	break
```

**Similarly, do not forget to stop and destroy the camera before quitting the program.**

```
# Clear the spawned vehicle and camera
def clear():

    vehicle.destroy()
    print('Vehicle Destroyed.')
    
    camera.stop()
    camera.destroy()
    print('Camera Destroyed. Bye!')

    for actor in world.get_actors().filter('*vehicle*'):
        actor.destroy()

    cv2.destroyAllWindows()
```

<br />

<hr />

## References

- Carla Official Documentation: https://carla.readthedocs.io/en/0.9.14/tutorials/

