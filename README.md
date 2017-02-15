brickpi-raspberry
==============

BrickPi Nodejs API module

# Introduction

Coupling Dexter Industries amazing BrickiPi board with Nodejs makes for a powerful way to build and program LEGO robots.

## Changes

I re-coded the entire module so as to use pure Javascript to communicate with the BrickPi board, therefore no longer relying on Dexter Industries C librairies. 

Added PID controllers to the motors for smooth operation.

## Install

You'll need a BrickPi compatible Raspian distribution.

Then:

```bash
$ npm install brickpi-raspberry
```

## Examples


### Running Motors and Sensors

```javascript
var brickpi = require('brickpi-raspberry');

var robot = new brickpi.BrickPi();
var motorA = new brickpi.Motor({port: brickpi.PORTS.MA, name: 'motorA'});
var motorB = new brickpi.Motor({port: brickpi.PORTS.MB, name: 'motorB'});
var touchA = new brickpi.Sensor({port: brickpi.PORTS.S1, type: brickpi.SENSOR_TYPE.NXT.TOUCH, name: 'Touch Sensor on upper arm'});

robot.addMotor(motorA).addMotor(motorB).addSensor(touchA).setup();

robot.on('ready', function() {
	motorA.resetPosition();
	motorB.resetPosition();
	robot.run();

	motorA.start(100).moveTo(5000, function(err) {
		// called when motorA has reached 5000 ticks (2500 degrees in rotation)
	});
});

setTimeout(function() {
	motorB.start(50);
}, 3000);

setTimeout(function() {
	motorB.stop();
}, 5000);

robot.on('tick', function() {
	// called at every polling cycle.
	var value = touchA.getValue();
});

setTimeout(function() {
	robot.stop();
}, 10000);

```

## API Documentation

### Main Object

#### Creation

Use the `BrickPi()` object contructor.  An options object can be passed to the constructor, e.g. `{serialPortAddress: "/dev/ttyAMA0", pollingInterval: 20}`.  The `pollingInterval` is the wait time at the end of an updateValue cycle and the next.  default value is 50ms, i.e. about 20 cycles per seconds.  It can be set at 0, although that was not tested.  Works fine at 20 as well. 

#### Adding Sensors and Motors

Use the `addMotor()` and `addSensor()` methods to add motors and sensors.

#### Setup

To initialize the BrickPi object, use the `setup()` method.  This method establishes the serial port connection to the BrickPi board, sets timeouts and configures the sensors.

#### Running the BrickPi

Use the `run()` method the start your robot.  This launches a polling loop that updates motors speeds and encoders and fetches fresh sensor values.  It should be called only once the `ready` event is received.

#### Stopping the robot

The `stop()` method will do that.  This stops the communication polling to the Brickpi, ending all processes and exiting the nodescript.

#### Events

`ready`:  issued when the raspberry pi has established a serial connection to the Brickpi and proper timeout values where set.

### Motor Object

#### Creation

`Motor()` is used to create the motor object.  Specify port aand optionally name:

```javascript
var motorA = new brickpi.motors.Motor({port: brickpi.PORTS.MA, name: 'motor A'});
``` 

#### Methods

`start(speed)` starts the motor at given speed.

`stop()` stops the motor.

`stopIn(ticks, callback)` stops the motor in the given amount of ticks.  Ticks is an absolute value.  The callback is called when the motor reaches the desired position and stops. An `err` parameter is passed to the callback if the position is not reached, e.g. `stop` was called before.

`moveTo(ticks, callback)` moves the motor position to the desired tick value (2 ticks equal 1 degree).  An `err` parameter is passed to the callback if for some reason the desired endpoint was not reached.

```js
motorA.start(150).moveTo(720, function(err) {
  if (!err) {
    console.log('motorA did one revolution');
  }
});

```

`getPosition()` returns current motor encoder position in ticks.

`resetPosition()` zeros up the encoder position.   Calling `getPosition()` right after will return 0 as motor position.

`getActualSpeed()` returns the actual motor speed in ticks/second.

##### PID

When `stopIn` or `moveTo` is used, a PID (proportional, Intergral and Derivative) control loop is applied to the motor speed/power setting. i.e. the motor slows down start as it approaches the desired position.

#### Events
`stop`: issued when the motor has stopped.

```javascript
motorA.on('stop', function() {
    console.log('motorA has stopped');		  
});

```

### Sensor Object

#### Creation

To create a sensor, use `Sensor()` providing a name, port and type.

```javascript
var sensorA = new brickpi.Sensor({port: brickpi.PORTS.S1, type: bricpi.SENSOR_TYPE.NXT.TOUCH, name: 'touch A'});
```
Current SENSOR_TYPE supported are:

NXT.TOUCH, 
NXT.ULTRASONIC.CONT, 
NXT.ULTRASONIC.SS, 
NXT.COLOR.FULL, 
NXT.COLOR.RED,
NXT.COLOR.BLUE,
NXT.COLOR.GREEN,
NXT.COLOR.NONE,

DEXTER.IMU.ACC

#### Methods

`getValue()` returns the sensor value.  In the case of the Dexter IMU sensor, a structure is returned:
`{x: x, y: y, z: z}`

#### Note
For now, the Dexter dIMU sensor will only return the acceleration only in an object ex: {x: 1.2, y:0.1 z:0.2}. And to keep things simple and efficient, one axis is updated per cycle. But considering the can be 50 cycles per seconds, a complete new vector is achieved in 60ms.


## Limitations
This module is improving quickly.  Stay posted.

A limited number of sensors are now supported.  If you require other ones, please let me know.

Sensor port S5 is not implemented.

LED are not implemented.


## ASYNC

Use the `async` module along with `brickpi-raspberry` to create simple, yet powerful LEGO robots.  For example, use `async.series` and `async.parallel` methods to orchestrate the movements of your robot.

[video: LEGO excavator in action](http://youtu.be/n9S490slNUk)

### Examples

```js
async.series([
	function(callback) {
		motorA.start(200).moveTo(720, callback);
	},
	function(callback) {
		motorB.start(150).moveTo(-3000, callback);
	}
	], function() {
	   console.log('Motor A moved to 720 followed by Motor B');
	});
```

