# Astro Pi 

The main drivers of the Astro Pi are 3 transducers:
1. LEDScreenTranducer: used to control the 8x8 LED RGB screen and interpret joy stick events
2. IMUTransducer: used to set up the accelerometer, gyroscope and magnetometer as well as interpreting their data
3. EnvSensorTransducer: used to set up the pressure and humidity sensors as well as interpreting their data

In order to read data from the device, first specify what to read in the connect() method inside declareConnections(). Then specify the corresponding listener for the Behavior class to implement. The data read will then be passed to the listener interface's abstract methods. 

### LEDScreenTransducer
1. Controlling the LED screen
Refer to the transducer's javadocs for the list of functionalities supported. 
Each "pixel" of the 8x8 LED Matrix consists of 3 Red,Green and Blue LEDs. The intensity of each LED is set by an integer between 0 and 63.
The position of each pixel is indexed by its row (int between 0-7) and column (int between 0-7) position. 
2. Getting the joystick events
First, call connect(AstroPi.GetJoystick) inside declareConnections() 
The transducer support JoyStickListener interface which has joyStickEvent() method with 5 integer parameters: left, right,up,down,push . The value will be 1 if the corresponding action is detected. 

### IMUTransducer
1. Setting up the sensors
By default, the 3 sensors are turned on with the following settings:
  1. Accelerometer: Scale = +/- 2g, output data rate = 952 Hz
  2. Gyroscope: Scale = 245 dps, output data rate = 119 Hz
  3. Magnetometer: Scale = +/- 4 Gauss, output data rate = 80 Hz
(Note that the output data rate is simply the rate that the hardware samples the data, not the rate that the application is reading it).






