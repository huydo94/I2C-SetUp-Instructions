# Setting up an I2C device in FogLight

## Write to I2C device operation 

![alt text](https://github.com/oci-pronghorn/FogLight-API/blob/master/I2CListener/Screen%20Shot%202017-08-09%20at%201.06.41%20PM.png?raw=true "Table 12")

![alt text](https://github.com/oci-pronghorn/FogLight-API/blob/master/I2CListener/Screen%20Shot%202017-08-09%20at%201.06.50%20PM.png?raw=true "Table 13")

Table 12 and 13 shows the protocol of an I2C write from master to slave. The meaning of the master/slave's actions are:
+ ST: a START command initiated by the master
+ SAD + W: the I2C address of the slave device with a write(W) bit (usually 0) appended at the end
+ SAK: slave acknowledge that it received the commands/data 
+ SUB: the address of the register on the I2C device that the master is writing to. Depending on the device, in order to write multiple bytes at a time, the most significant bit of the 8-bit register address will have to be 1 to enable address-auto increment. Check each device's datasheet for details on that.
+ DATA: the data byte being written to the register
+ SP: a STOP command sent by the master

FogLight's underlying API takes care of the ST,SAK and SP commands. It also automatically appends the write(W) bit to the SAD. Thus, the user only needs to specify SAD, SUB and the DATA fields.

The following code shows how to write one byte to a register on an I2C device

```java
FogCommandChannel target;
DataOutputBlobWriter<I2CCommandSchema> i2cPayloadWriter = target.i2cCommandOpen(I2C_ADDRESS); // I2C_ADDRESS is the i2c address of the device, or SAD  
        
i2cPayloadWriter.writeByte(SUB); // address of the register being written to
i2cPayloadWriter.writeByte(DATA); //data to write to the register. 
        
target.i2cCommandClose(); 
target.i2cFlushBatch(); //send the chain of commands on the command channel
```

The following code shows how to write multiple bytes to a register on an I2C device.

```java
FogCommandChannel target;
DataOutputBlobWriter<I2CCommandSchema> i2cPayloadWriter = target.i2cCommandOpen(I2C_ADDRESS); // I2C_ADDRESS is the i2c address of the device, or SAD  
        
i2cPayloadWriter.writeByte(SUB); // address of the register being written to. Make sure to check the device's datasheet to know whether the register address's MSB needs to be 1 for auto-address increment if writing multiple bytes.
i2cPayloadWriter.writeByte(DATA1); //data to write to the register. 
i2cPayloadWriter.writeByte(DATA2); 
i2cPayloadWriter.writeByte(DATA3); 

target.i2cCommandClose(); 
target.i2cFlushBatch(); //send the chain of commands on the command channel
```

### Read from I2C device operation

![alt text](https://github.com/oci-pronghorn/FogLight-API/blob/master/I2CListener/Screen%20Shot%202017-08-09%20at%201.06.59%20PM.png?raw=true "Table 14")

![alt text](https://github.com/oci-pronghorn/FogLight-API/blob/master/I2CListener/Screen%20Shot%202017-08-09%20at%201.07.06%20PM.png?raw=true "Table 15")

Table 14 and 15 shows the protocol of an I2C read from slave. The meaning of the master/slave's actions are:
+ SR: a "repeated start" command issued by the master
+ SAD+R: the I2C address of the slave device with a read(R) bit (usually 1) appended at the end
+ SUB: the address of the register on the slave device that the master is reading from. If multiple bytes are being read from multiple adjacent registers( for example at register 0x21 to 0x26), SUB will be the address of  the lowest register from the group (which is 0x21). Depending on the device, the most significant bit of SUB will have to be 1 to enable address-auto increment. Check each device's datasheet for details on that.
+ DATA : data returned by the slave
+ MAK: master acknowledged that it received the data 
+ NMAK : no master acknowledge

FogLight's underlying API took care of the handshake signals such as ST,SR, SAK,MAK,NMAK and SP. Thus, the user only needs to specify SAD (the slave's I2C addrees), SUB and number of bytes to read. However, setting up I2C read in FogLight is very different from I2C write operation. 

In FogLight, after specifying SAD,SUB and number of bytes to read, the user also set the rate that the master will read from the slave. Once the application starts running, the master will poll the data from the slave at the rate specified indefinitely. 

That is done by implementing an enum as follows:

```java
/*
* To change this license header, choose License Headers in Project Properties.
* To change this template file, choose Tools | Templates
* and open the template in the editor.
*/
package com.ociweb.iot.grove.three_axis_accelerometer_16g;

import com.ociweb.iot.hardware.I2CConnection;
import com.ociweb.iot.hardware.I2CIODevice;
import com.ociweb.iot.maker.FogCommandChannel;

/**
 *
 * @author huydo
 */
public enum ThreeAxisAccelerometer_16gTwig {
    ;
    public enum ThreeAxisAccelerometer_16g implements I2CIODevice {
        GetXYZ(){
            
            @Override
            public I2CConnection getI2CConnection() {
                byte[] REG_ADDR = {ThreeAxisAccelerometer_16g_Constants.ADXL345_DATAX0}; // address of the register being reading. If reading multiple bytes, make sure to check the device's datasheet to know whether the register address's MSB needs to be 1 for auto-address increment.
                byte I2C_ADDR = ThreeAxisAccelerometer_16g_Constants.ADXL345_DEVICE; //I2C address of the slave device
                byte BYTESTOREAD = 6; //number of bytes to read
                byte REG_ID = ThreeAxisAccelerometer_16g_Constants.ADXL345_DATAX0; //just an identifier
                return new I2CConnection(this, I2C_ADDR, REG_ADDR, BYTESTOREAD, REG_ID, null);
            }
        },
        GetTapAct(){
            @Override
            public I2CConnection getI2CConnection() {
                byte[] REG_ADDR = {ThreeAxisAccelerometer_16g_Constants.ADXL345_ACT_TAP_STATUS};
                byte I2C_ADDR = ThreeAxisAccelerometer_16g_Constants.ADXL345_DEVICE;
                byte BYTESTOREAD = 1;
                byte REG_ID = ThreeAxisAccelerometer_16g_Constants.ADXL345_ACT_TAP_STATUS; //just an identifier
                return new I2CConnection(this, I2C_ADDR, REG_ADDR, BYTESTOREAD, REG_ID, null);
            }
        },
        GetInterrupt(){
            @Override
            public I2CConnection getI2CConnection() {
                byte[] REG_ADDR = {ThreeAxisAccelerometer_16g_Constants.ADXL345_INT_SOURCE};
                byte I2C_ADDR = ThreeAxisAccelerometer_16g_Constants.ADXL345_DEVICE;
                byte BYTESTOREAD = 1;
                byte REG_ID = ThreeAxisAccelerometer_16g_Constants.ADXL345_INT_SOURCE; //just an identifier
                return new I2CConnection(this, I2C_ADDR, REG_ADDR, BYTESTOREAD, REG_ID, null);
            }
        };
        @Override
        public boolean isInput() {
            return true;
        }
        
        @Override
        public boolean isOutput() {
            return true;
        }
        @Override
        public int response() {
            return 1000; // send a read command every 1000ms 
        }
        
        @SuppressWarnings("unchecked")
        @Override
        public ThreeAxisAccelerometer_16g_Transducer newTransducer(FogCommandChannel... ch) {
            return new ThreeAxisAccelerometer_16g_Transducer(ch[0]);
        }
        /**

         * /**
         * @return Delay, in milliseconds, for scan. TODO: What's scan?
         */
        @Override
        public int scanDelay() {
            return 0;
        }
        
        /**
         * @return True if this twig is Pulse Width Modulated (PWM) device, and
         *         false otherwise.
         */
        @Override
        public boolean isPWM() {
            return false;
        }
        
        /**
         * @return True if this twig is an I2C device, and false otherwise.
         */
        public boolean isI2C() {
            return true;
        }
        
        /**
         * @return The possible value range for reads from this device (from zero).
         */
        @Override
        public int range() {
            return 256;
        }
        
        /**
         * @return the setup bytes needed to initialized the connected I2C device
         */
        public byte[] I2COutSetup() {
            return null;
        }
        
        /**
         * Validates if the I2C data from from the device is a valid response for this twig
         *
         * @param backing
         * @param position
         * @param length
         * @param mask
         *
         * @return false if the bytes returned from the device were not some valid response
         */
        @Override
        public boolean isValid(byte[] backing, int position, int length, int mask) {
            return true;
        }
        
        /**
         * @return The number of hardware pins that this twig uses.
         */
        @Override
        public int pinsUsed() {
            return 1;
        }
    }
}
```

The 3 nums GetXYZ, GetTapAct and GetInterrupt are the 3 reading operations being set up. Each of them overrides getI2CConnection(), where the user will specify the SAD, SUB and number of bytes to read. The default polling rate for all devices is set at 1Hz by the response() method. 

From the application layer, inside declareConnections() method, calling connect(ThreeAxisAccelerometer_16g.GetXYZ) will activate the reading commands set up by the corresponding enum.

The behavior class/ transducer class can get the I2C read data by implementing an I2CListener/ I2CListenerTransducer, which has the i2cEvent() abstract method. This event triggers whenever the data package requested by the enum is available.

```java
	@Override
	public void i2cEvent(int addr, int register, long time, byte[] backing, int position, int length, int mask) {
		// addr is the i2c address of the slave device
		// register is the address of the register containing the first byte of the data package 
		// backing is a circular buffer, with size = mask containing bytes read from i2c
		// position is the index on backing[] of the first byte of the data package 
		// length is the number of bytes contained in the data package
		if(addr == ADDR_ADC121 && register == REG_ADDR_RESULT){
			short temp = (short)(((backing[(position)&mask]&0x0F) << 8) | (backing[(position+1)&mask]&0xFF));
			System.out.println("The conversion reading is: "+ temp);  
	     }
	}
```
