# ADCameralink: EPICS AreaDetector support for Cameralink



ADCameralink

The cameralink driver has three parts:

1. ADCameralink- ADDriver for Cameralink image grabbing functions.
2. camLinkSerial - asynPortDriver for serial port embedded into camera link grabber.


Camera Link has two parts: a serial port for camera control, and a high speed 
data link for image data. The Camera Link driver is a class ADCameraLink, which 
inherits ADDriver, and adds functions to interface to any camera link card. There 
are special class for the silicon software ME-4 (microEnable) grabber and Dalsa/Coreco grabbers.
Editing the cameralinkApp/src/Makefile sets which grabber is being used. 
A softwareGrabber is also supported that generates test images when no real hardware
grabber is present.


For supporting a specific camera, one writes a driver that inherits ADCameraLink 
and adds the serial port code to control the camera.


CameraLink Serial Port

The serial port on the camera link board shows up in EPICS as an asyn serial 
port driver. The port of this driver is passed to the the camera driver. The 
serial port driver behaves like other asyn serial port drivers as documented in

http://www.aps.anl.gov/epics/modules/soft/asyn/R3-1/asynDriver.html#drvAsynSerialPort


The asynDriver for the camera link serial port is in camLinkSerial.h/cpp. The callable
function in the IOC shell is
drvCamlinkSerialConfigure()

 



How grabbers are supported

Every grabber has its own commercial esoteric code library with calls specific to that 
particular grabber. To allow ADCameralink to use the grabber a programmer must create two
classes; a class that implements serial port functions, and a class that implements image
grab functions. 

Image grabbing support is accomplished by creating a class that inherits and implements all
functuions in grabberInterface.h. Your header fill will look like grabberInterface, but functions
are not set to 0, as in an interface class. The asssociated cpp fill will implem,ent these functions
calling the commercial library. 

Also, a #define variable, such as USE_XXXCORP, where XXXCORP is the name of the grabber should 
defined in the cameralinkApp/src/Makefile. All code for this grabber, header, and cpp, should be in an 
#ifdef/#endif so it only compiles if we wish to build for that particular grabber. Currently
we cuild ADCameralink for one grabber at a time. That is, if you switch grabbers, then you need
to rebuild and link to the proper commercial library. See the siswGrabber.h and .cpp for examples.

The serial port for Cameralink grabbers can be implemented in two ways. Some grabbers 
have built in support for COM ports. That is, when the grabber is installed, its serial port
appears as a Windows COM port, or Linux tty port. In this case, writing and readig from the
serial interface is the same as reading/writing a standard RS232 port. The Dalsa grabber
uses this method. To use this meathod, the code cl_com_port.h/cpp is uised. When writing
to the serial port, asynPort driver, a sepearate one is ised. the idea is that in the det3ector
st.cmd file, we call the detector driver initializalitaion and serial port init function. The
ASYN port of the serial driver is passed to the detector driver on startup. The serial port
of the grabber appears as a separate ASYN device in thsi way. All serial port code must inherit
from comportInterface.h and implement all the functions. 

The 2nd method of accesing a Cameralink grabber serial port is to use the clserxxxxx.dll (.so)
liobrary. In this method, there are standard dll functions for reading/writing/configuring of 
the serial port, as defined in the camera link spec. The siswSerialPort.h file is an example 
of this. To implement another vendor, it would be easiest to copy the siswSerialPort code
and rename the class to your vendor. Then you woudl use pretty much all the same code to
open up the proper .dll file, and link all the standard functions. Probably, there should 
be a single h/cpp code for all serial ports, but a different dll library is opened based on 
the vendor. 


Once the comportInterface and grabberInterface code is implemented, the programmer must edit
ADCameralink.cpp to actually instantiate the proper objects. Search for USE_SAP in the file
and see the required edits. The two places to edit are in

1)void ADCameralink::loadCCF() , where the proper camera grabber code is instantiated.
2) ADCameralink::ADCameralink(), where a parameter is set defining which grabber is used.
3)camLinkSerial::camLinkSerial(), where the serial port code is instantiated.



Silicon Software Support

The ADCameralink module supports the Micro Enable 4 card from Silicon Software. Currently
Windows is supported. Linux support builds but is not thoroughly tested. 
The software must build against the Silicon software header files, and link against 
Silicon Software libraries.


The headers and link libraries are included with ADCameralink under 
ADCameralink/siliconSoftwareSupport. This code was extracted from the distributable
libries. An issue with Windows, is that software works better when installed properly with
the commercial installer. The included headers and libs make the build easier. 

You should download in install two pieces of code from Silicon Software
1) Silicon software device driver. 
2) Silicon software ME software.This is an application that allows configuring and testing
the Silicon Software card.

You must #define USE_SISW in the cameralinkApp/src/Makefile. This tells the compiler to build for 
Silicon Software



Dalsa/Coreco Support

the Dalsa grabbers only run on Windows, and require an expensive software license for a code
called Sapera.Also device drivers must be downloaded for the card. The license for the drivers 
comes with purchasing the card itself. CamExpert is a code from Dalsa for testing and 
configuring the grabber. 

Once the Dalsa software and drivers are installed and licensed, iyou can build ADCameralink
for this card by #defining USE_SAP   in the  cameralinkApp/src/Makefile. This will compile and link
the Dalsa code. The cpp/h code is in coreco.h/cpp. The serial port is defined as  standard
COM port, so cl_com_port is used.

Because the Dalsa software requires a license, it is not distributed with ADCamralink.


Using ADCameralink

ADCameralink cannot be directly called at IOC run time. There uis no ADCamerlink config function
that is callable fromthe IOC shell. Rather, it is a code library that contains asynDriver
support and needed PVs for controlling a grabber in EPICS. A specific camera implementation
must be written, which is an ADDriver, that bulds and links against ADCameralink.

