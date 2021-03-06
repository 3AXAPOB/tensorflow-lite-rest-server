# tensorflow-lite-rest-server
Expose tensorflow-lite models via a rest API. Currently object, face & scene detection is supported. Can be hosted on any of the common platforms including RPi, linux desktop, Mac and Windows. A service can be used to have the server run automatically on an RPi.

## Setup
In this process we create a virtual environment (venv), then install tensorflow-lite [as per these instructions](https://www.tensorflow.org/lite/guide/python) which is platform specific, and finally install the remaining requirements. Note on an RPi (only) it is necessary to manually install pip3, numpy, pillow.

All instructions for mac:
```
python3.7 -m venv venv
source venv/bin/activate
pip3 install https://dl.google.com/coral/python/tflite_runtime-2.1.0.post1-cp37-cp37m-macosx_10_14_x86_64.whl
pip3 install -r requirements.txt
```

## Models
For convenience a couple of models are included in this repo and used by default. A description of each model is included in its directory. Additional models are available [here](https://github.com/google-coral/edgetpu/tree/master/test_data).

If you want to create custom models, there is the easy way, and the longer but more flexible way. The easy way is to use [teachablemachine](https://teachablemachine.withgoogle.com/train/image), which I have done in this repo for the dogs-vs-cats model. The teachablemachine service is limited to image classification but is very straightforward to use. The longer way allows you to use any neural network architecture to produce a tensorflow model, which you then convert to am optimized tflite model. An example of this approach is described in [this article](https://towardsdatascience.com/inferences-from-a-tf-lite-model-transfer-learning-on-a-pre-trained-model-e16e7c5f0ee6), or jump straight [to the code](https://github.com/arshren/TFLite/blob/master/Transfer%20Learning%20with%20TFLite-Copy1.ipynb).

## Usage
Start the tflite-server on port 5000 (default is port 5000):
```
(venv) $ python3 tflite-server.py --port 5000
```

You can check that the tflite-server is running by visiting `http://ip:5000/` from any machine, where `ip` is the ip address of the host (`localhost` if querying from the same machine).

Post an image to detecting objects via cURL:
```
curl -X POST -F image=@tests/people_car.jpg 'http://localhost:5000/v1/vision/detection'
```
Which should return:
```
{
  "predictions": [
    {
      "confidence": 0.93359375, 
      "label": "car", 
      "x_max": 619, 
      "x_min": 302, 
      "y_max": 348, 
      "y_min": 120
    }, 
    {
      "confidence": 0.7890625, 
      "label": "person", 
      "x_max": 363, 
      "x_min": 275, 
      "y_max": 323, 
      "y_min": 126
    },
.
.
.
'success': True}
```

To detect faces:
```
curl -X POST -F image=@tests/faces.jpg 'http://localhost:5000/v1/vision/face'
```

To detect the scene (dogs vs cats model):
```
curl -X POST -F image=@tests/cat.jpg 'http://localhost:5000/v1/vision/scene'
```

## Add tflite-server as a service
You can run tflite-server as a [service](https://www.raspberrypi.org/documentation/linux/usage/systemd.md), which means tflite-server will automatically start on RPi boot, and can be easily started & stopped. Create the service file in the appropriate location on the rpi using: ```sudo nano /etc/systemd/system/tflite-server.service```

Entering the following (adapted for your `tflite-server.py` file location and args):
```
[Unit]
Description=Flask app exposing tensorflow lite models
After=network.target

[Service]
ExecStart=/usr/bin/python3 -u tflite-server.py
WorkingDirectory=/home/pi/github/tensorflow-lite-rest-server
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```

Once this file has been created you can to start the service using:
```sudo systemctl start tflite-server.service```

View the status and logs with:
```sudo systemctl status tflite-server.service```

Stop the service with:
```sudo systemctl stop tflite-server.service```

Restart the service with:
```sudo systemctl restart tflite-server.service```

You can have the service auto-start on rpi boot by using:
```sudo systemctl enable tflite-server.service```

You can disable auto-start using:
```sudo systemctl disable tflite-server.service```

## Deepstack, Home Assistant & UI
This API can be used as a drop in replacement for [deepstack object detection](https://github.com/robmarkcole/HASS-Deepstack-object) and [deepstack face detection](https://github.com/robmarkcole/HASS-Deepstack-face) (configuring `detect_only: True`) in Home Assistant. I also created a UI for viewing the predictions of the object detection model [here](https://github.com/robmarkcole/deepstack-ui).

## Development
I am developing on a mac/pi4 using VScode. On mac use a venv, on pi install system wide.

* First time only, create venv: `python3.7 -m venv venv`
* Activate venv: `source venv/bin/activate`
* Install the dev requirements: `pip3 install -r requirements.txt` & `pip3 install -r requirements-dev.txt`
* Sort requirements: `venv/bin/isort tflite-server.py`
* Black format: `venv/bin/black tflite-server.py`
* Run the `usage.ipynb` notebook: `venv/bin/jupyter notebook`

Unfortunately appears black is not supported on pi4 yet.