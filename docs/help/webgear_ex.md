<!--
===============================================
vidgear library source-code is deployed under the Apache 2.0 License:

Copyright (c) 2019 Abhishek Thakur(@abhiTronix) <abhi.una12@gmail.com>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
===============================================
-->

# WebGear Examples

&thinsp;

## Using WebGear with RaspberryPi Camera Module

Because of WebGear API's flexible internal wapper around VideoGear, it can easily access any parameter of CamGear and PiGear videocapture APIs.

!!! info "Following usage examples are just an idea of what can be done with WebGear API, you can try various [VideoGear](../../gears/videogear/params/), [CamGear](../../gears/camgear/params/) and [PiGear](../../gears/pigear/params/) parameters directly in WebGear API in the similar manner."

Here's a bare-minimum example of using WebGear API with the Raspberry Pi camera module while tweaking its various properties in few lines of python code:

!!! new "Backend PiGear API now fully supports the newer [`picamera2`](https://github.com/raspberrypi/picamera2) python library under the hood for Raspberry Pi :fontawesome-brands-raspberry-pi: camera modules. Follow this [guide ➶](../../installation/pip_install/#picamera2) for its installation."

!!! warning "Make sure to [complete Raspberry Pi Camera Hardware-specific settings](https://www.raspberrypi.com/documentation/accessories/camera.html#installing-a-raspberry-pi-camera) prior using this backend, otherwise nothing will work."


=== "New Picamera2 backend"

    ```python linenums="1" hl_lines="22"
    # import libs
    import uvicorn
    from libcamera import Transform
    from vidgear.gears.asyncio import WebGear

    # various WebGear_RTC performance 
    # and Picamera2 API tweaks
    options = {
        "frame_size_reduction": 40,
        "jpeg_compression_quality": 80,
        "jpeg_compression_fastdct": True,
        "jpeg_compression_fastupsample": False,
        "queue": True,
        "buffer_count": 4,
        "controls": {"Brightness": 0.5, "ExposureValue": 2.0},
        "transform": Transform(hflip=1),
        "auto_align_output_config": True,  # auto-align camera configuration
    }

    # initialize WebGear app
    web = WebGear(
        enablePiCamera=True, resolution=(640, 480), framerate=60, logging=True, **options
    )

    # run this app on Uvicorn server at address http://localhost:8000/
    uvicorn.run(web(), host="localhost", port=8000)

    # close app safely
    web.shutdown()
    ```
    
=== "Legacy Picamera backend"

    ??? info "Under the hood, Backend PiGear API _(version `0.3.3` onwards)_ prioritizes the new [`picamera2`](https://github.com/raspberrypi/picamera2) API backend."

        However, the API seamlessly switches to the legacy [`picamera`](https://picamera.readthedocs.io/en/release-1.13/index.html) backend, if the `picamera2` library is unavailable or not installed.
        
        !!! tip "It is advised to enable logging(`logging=True`) to see which backend is being used."

        !!! failure "The `picamera` library is built on the legacy camera stack that is NOT _(and never has been)_ supported on 64-bit OS builds."

        !!! note "You could also enforce the legacy picamera API backend in PiGear by using the [`enforce_legacy_picamera`](../../gears/pigear/params) user-defined optional parameter boolean attribute."

    ```python linenums="1" hl_lines="21"
    # import libs
    import uvicorn
    from vidgear.gears.asyncio import WebGear

    # various webgear performance and Picamera API tweaks
    options = {
        "frame_size_reduction": 40,
        "jpeg_compression_quality": 80,
        "jpeg_compression_fastdct": True,
        "jpeg_compression_fastupsample": False,
        "hflip": True,
        "exposure_mode": "auto",
        "iso": 800,
        "exposure_compensation": 15,
        "awb_mode": "horizon",
        "sensor_mode": 0,
    }

    # initialize WebGear app
    web = WebGear(
        enablePiCamera=True, resolution=(640, 480), framerate=60, logging=True, **options
    )

    # run this app on Uvicorn server at address http://localhost:8000/
    uvicorn.run(web(), host="localhost", port=8000)

    # close app safely
    web.shutdown()
    ```

&nbsp;

## Using WebGear with real-time Video Stabilization enabled
 
Here's an example of using WebGear API with real-time Video Stabilization enabled:

```python linenums="1" hl_lines="14"
# import libs
import uvicorn
from vidgear.gears.asyncio import WebGear

# various webgear performance tweaks
options = {
    "frame_size_reduction": 40,
    "jpeg_compression_quality": 80,
    "jpeg_compression_fastdct": True,
    "jpeg_compression_fastupsample": False,
}

# initialize WebGear app  with a raw source and enable video stabilization(`stabilize=True`)
web = WebGear(source="foo.mp4", stabilize=True, logging=True, **options)

# run this app on Uvicorn server at address http://localhost:8000/
uvicorn.run(web(), host="localhost", port=8000)

# close app safely
web.shutdown()
```

&nbsp;


## Display Two Sources Simultaneously in WebGear

In this example, we'll be displaying two video feeds side-by-side simultaneously on browser using WebGear API by defining two separate frame generators: 

??? new "New in v0.2.2" 
    This example was added in `v0.2.2`.

**Step-1 (Trigger Auto-Generation Process):** Firstly, run this bare-minimum code to trigger the [**Auto-generation**](../../gears/webgear/overview/#auto-generation-process) process, this will create `.vidgear` directory at current location _(directory where you'll run this code)_:

```python linenums="1" hl_lines="6"
# import required libraries
import uvicorn
from vidgear.gears.asyncio import WebGear

# provide current directory to save data files
options = {"custom_data_location": "./"}

# initialize WebGear app
web = WebGear(source=0, logging=True, **options)

# close app safely
web.shutdown()
```

**Step-2 (Replace HTML file):** Now, go inside `.vidgear` :arrow_right: `webgear` :arrow_right: `templates` directory at current location of your machine, and there replace content of `index.html` file with following:

```html hl_lines="5-6"
{% extends "base.html" %}
{% block content %}
  <h1 class="glow">WebGear Video Feed</h1>
   <div class="rows">
     <img src="/video" alt="Feed"/>
     <img src="/video2" alt="Feed"/>
   </div>
{% endblock %}
```

**Step-3 (Build your own Frame Producers):** Now, create a python script code with OpenCV source, as follows:

```python linenums="1"  hl_lines="9 15-38 42-65 68-77 81 84-86"
# import necessary libs
import uvicorn, asyncio, cv2
from vidgear.gears.asyncio import WebGear
from vidgear.gears.asyncio.helper import reducer
from starlette.responses import StreamingResponse
from starlette.routing import Route

# provide current directory to load data files
options = {"custom_data_location": "./"}

# initialize WebGear app without any source
web = WebGear(logging=True, **options)

# create your own custom frame producer
async def my_frame_producer1():

   # !!! define your first video source here !!!
   # Open any video stream such as "foo1.mp4"
   stream = cv2.VideoCapture("foo1.mp4")
   # loop over frames
   while True:
       # read frame from provided source
       (grabbed, frame) = stream.read()
       # break if NoneType
       if not grabbed:
           break

       # do something with your OpenCV frame here

       # reducer frames size if you want more performance otherwise comment this line
       frame = await reducer(frame, percentage=30)  # reduce frame by 30%
       # handle JPEG encoding
       encodedImage = cv2.imencode(".jpg", frame)[1].tobytes()
       # yield frame in byte format
       yield (b"--frame\r\nContent-Type:video/jpeg2000\r\n\r\n" + encodedImage + b"\r\n")
       await asyncio.sleep(0.00001)
   # close stream
   stream.release()


# create your own custom frame producer
async def my_frame_producer2():

   # !!! define your second video source here !!!
   # Open any video stream such as "foo2.mp4"
   stream = cv2.VideoCapture("foo2.mp4")
   # loop over frames
   while True:
       # read frame from provided source
       (grabbed, frame) = stream.read()
       # break if NoneType
       if not grabbed:
           break

       # do something with your OpenCV frame here

       # reducer frames size if you want more performance otherwise comment this line
       frame = await reducer(frame, percentage=30)  # reduce frame by 30%
       # handle JPEG encoding
       encodedImage = cv2.imencode(".jpg", frame)[1].tobytes()
       # yield frame in byte format
       yield (b"--frame\r\nContent-Type:video/jpeg2000\r\n\r\n" + encodedImage + b"\r\n")
       await asyncio.sleep(0.00001)
   # close stream
   stream.release()


async def custom_video_response(scope):
   """
   Return a async video streaming response for `my_frame_producer2` generator
   """
   assert scope["type"] in ["http", "https"]
   await asyncio.sleep(0.00001)
   return StreamingResponse(
       my_frame_producer2(),
       media_type="multipart/x-mixed-replace; boundary=frame",
   )


# add your custom frame producer to config
web.config["generator"] = my_frame_producer1

# append new route i.e. new custom route with custom response
web.routes.append(
    Route("/video2", endpoint=custom_video_response)
    )

# run this app on Uvicorn server at address http://localhost:8000/
uvicorn.run(web(), host="localhost", port=8000)

# close app safely
web.shutdown()
``` 

!!! success "On successfully running this code, the output stream will be displayed at address http://localhost:8000/ in Browser."


&nbsp;