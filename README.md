# ADCameralink: EPICS AreaDetector support for Cameralink



ADCameralink

The cameralink driver has three parts:

1. ADCameralink- ADDriver for Cameralink image grabbing functions.
2. camLinkSerial - asynPortDriver for serial port embedded into camera link grabber.


Camera Link has two parts: a serial port for camera control, and a high speed 
data link for image data. The Camera Link driver is a class ADCameraLink, which 
inherits ADDriver, and adds functions to interface to any camera link card. There 
are special class for the silicon software MEIV grabber and Dalsa/Coreco grabbers.
Editing the makefile sets which grabber is being used. 

For supporting a specific camera, one writes a driver that inherits ADCameraLink 
and adds the serial port code to control the camera.


CameraLink Serial Port

The serial port on the camera link board shows up in EPICS as an asyn serial 
port driver. The port of this driver is passed to the the camera driver. The 
serial port driver behaves like other asyn serial port drivers as documented in

http://www.aps.anl.gov/epics/modules/soft/asyn/R3-1/asynDriver.html#drvAsynSerialPort




