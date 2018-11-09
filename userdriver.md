# UserDriver

自作したDriverは、UserDriverとしてアプリに組み込む事が可能です。

Adx345.java
```java
package com.gclue.adx345;

import com.google.android.things.pio.I2cDevice;
import com.google.android.things.pio.PeripheralManager;
import java.io.IOException;

public class Adx345 implements AutoCloseable {
    private static final String TAG = Adx345.class.getSimpleName();

    /**
     * I2C slave address of the Adx345.
     */
    public static final int I2C_ADDRESS = 0x53;

    /**
     * Sampling rate of the measurement.
     */
    /** Who_am_i register */
    private byte ADXL345_DEVID_REG = 0x00;
    /** name of adx345 */
    private byte ADXL345_DEVICE_NAME = (byte) 0xE5;
    /** Data Format Control */
    private byte ADXL345_DATA_FORMAT_REG = 0x31;
    /** Power-saving features control */
    private byte ADXL345_POWER_CTL_REG = 0x2D;
    /** Power-saving features control */
    private byte ADXL345_3AIXS = 0x32;
    /** SELF Test ON */
    private byte ADXL345_SELF_TEST_ON = (byte) 0b10000000;
    /** SELF Test OFF */
    private byte ADXL345_SELF_TEST_OFF = 0b00000000;
    /** SELF SPI ON */
    private byte ADXL345_SPI_ON = 0b01000000;
    /** SELF SPI OFF */
    private byte ADXL345_SPI_OFF = 0b00000000;
    /** INT_INVERT ON */
    private byte ADXL345_INT_INVERT_ON = 0b00100000;
    /** INT_INVERT OFF */
    private byte ADXL345_INT_INVERT_OFF = 0b00000000;
    /** FULL_RES ON */
    private byte ADXL345_FULL_RES_ON = 0b00001000;
    /** FULL_RES OFF */
    private byte ADXL345_FULL_RES_OFF = 0b00000000;
    /** JUSTIFY ON */
    private byte ADXL345_JUSTIFY_ON = 0b00000100;
    /** JUSTIFY OFF */
    private byte ADXL345_JUSTIFY_OFF = 0b00000000;
    /** RANGE 2G */
    private byte ADXL345_RANGE_2G = 0b00;
    /** RANGE 4G */
    private byte ADXL345_RANGE_4G = 0b01;
    /** RANGE 8G */
    private byte ADXL345_RANGE_8G = 0b10;
    /** RANGE 16G */
    private byte ADXL345_RANGE_16G = 0b11;
    /** AUTO SLEEP ON */
    private byte ADXL345_AUTO_SLEEP_ON = 0b00010000;
    /** AUTO SLEEP OFF */
    private byte ADXL345_AUTO_SLEEP_OFF = 0b00000000;
    /** AUTO MEASURE ON */
    private byte ADXL345_MEASURE_ON = 0b00001000;
    /** AUTO MEASURE OFF */
    private byte ADXL345_MEASURE_OFF = 0b00000000;
    /** SLEEP ON */
    private byte ADXL345_SLEEP_ON = 0b00000100;
    /** SLEEP OFF */
    private byte ADXL345_SLEEP_OFF = 0b00000000;
    /** WAKEUP 8Hz */
    private byte ADXL345_WAKEUP_8HZ = 0b00;
    /** WAKEUP 4Hz */
    private byte ADXL345_WAKEUP_4HZ = 0b01;
    /** WAKEUP 2Hz */
    private byte ADXL345_WAKEUP_2HZ = 0b10;
    /** WAKEUP 1Hz */
    private byte ADXL345_WAKEUP_1HZ = 0b11;

    private I2cDevice mDevice;

    /**
     * Create a new Adx345 driver connected to the given I2C bus.
     * @param bus
     * @throws IOException
     */
    public Adx345(String bus) throws IOException {
        PeripheralManager pioService = PeripheralManager.getInstance();
        I2cDevice device = pioService.openI2cDevice(bus, I2C_ADDRESS);
        try {
            connect(device);
        } catch (IOException|RuntimeException e) {
            try {
                close();
            } catch (IOException|RuntimeException ignored) {
            }
            throw e;
        }
    }

    /**
     * Create a new Adx345 driver connected to the given I2C device.
     * @param device
     * @throws IOException
     */
    /*package*/ Adx345(I2cDevice device) throws IOException {
        connect(device);
    }

    private void connect(I2cDevice device) throws IOException {
        if (mDevice != null) {
            throw new IllegalStateException("device already connected");
        }
        mDevice = device;
    }


    /**
     * Close the driver and the underlying device.
     */
    @Override
    public void close() throws IOException {
        if (mDevice != null) {
            try {
                mDevice.close();
            } finally {
                mDevice = null;
            }
        }
    }

    /**
     * Who am I .
     * @return find or not find.
     */
    public boolean whoAmI() {
        try {
            byte value = mDevice.readRegByte(ADXL345_DEVID_REG);
            if((value & 0xff) == ADXL345_DEVICE_NAME) {
                return true;
            } else {
                return false;
            }
        } catch (IOException e) {
            return false;
        }
    }

    /**
     * Set configure.
     */
    public void setConfigure() {
        byte conf = ADXL345_SELF_TEST_OFF;
        conf |= ADXL345_SPI_OFF;
        conf |= ADXL345_INT_INVERT_OFF;
        conf |= ADXL345_FULL_RES_OFF;
        conf |= ADXL345_JUSTIFY_OFF;
        conf |= ADXL345_RANGE_16G;
        try {
            mDevice.writeRegByte(ADXL345_DATA_FORMAT_REG, conf);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * Power on.
     */
    public void powerOn() {
        byte power = ADXL345_AUTO_SLEEP_OFF;
        power |= ADXL345_MEASURE_ON;
        power |= ADXL345_SLEEP_OFF;
        power |= ADXL345_WAKEUP_8HZ;
        try {
            mDevice.writeRegByte(ADXL345_POWER_CTL_REG, power);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * Read an accelerometer sample.
     * @return acceleration over xyz axis in G.
     * @throws IOException
     * @throws IllegalStateException
     */
    public float[] readSample() throws IOException, IllegalStateException {
        if (mDevice == null) {
            throw new IllegalStateException("device not connected");
        }
        int length = 6;
        byte axis_buff[] = new byte[length];
        mDevice.readRegBuffer(ADXL345_3AIXS, axis_buff, axis_buff.length);
        int x = (((int)axis_buff[1]) << 8) | axis_buff[0];
        int y = (((int)axis_buff[3]) << 8) | axis_buff[2];
        int z = (((int)axis_buff[5]) << 8) | axis_buff[4];
        return new float[]{
                x, y, z
        };
    }



}
```

Adx345AccelerometerDriver.java
```java
package com.gclue.adx345;

import android.hardware.Sensor;
import android.hardware.SensorManager;

import com.google.android.things.userdriver.UserDriverManager;
import com.google.android.things.userdriver.sensor.UserSensor;
import com.google.android.things.userdriver.sensor.UserSensorDriver;
import com.google.android.things.userdriver.sensor.UserSensorReading;

import java.io.IOException;
import java.util.UUID;

public class Adx345AccelerometerDriver implements AutoCloseable {

    private static final String TAG = Adx345AccelerometerDriver.class.getSimpleName();
    private static final String DRIVER_NAME = "FaBoAccelerometer";
    private static final String DRIVER_VENDOR = "GClue";
    private static final int DRIVER_VERSION = 1;
    private Adx345 mDevice;
    private UserSensor mUserSensor;

    /**
     * Create a new framework accelerometer driver connected to the given I2C bus.
     * The driver emits {@link android.hardware.Sensor} with acceleration data when registered.
     * @param bus
     * @throws IOException
     * @see #register()
     */
    public Adx345AccelerometerDriver(String bus) throws IOException {
        mDevice = new Adx345(bus);
    }

    /**
     * Close the driver and the underlying device.
     * @throws IOException
     */
    @Override
    public void close() throws IOException {
        unregister();
        if (mDevice != null) {
            try {
                mDevice.close();
            } finally {
                mDevice = null;
            }
        }
    }

    /**
     * Register the driver in the framework.
     * @see #unregister()
     */
    public void register() {
        if (mDevice == null) {
            throw new IllegalStateException("cannot registered closed driver");
        }
        if (mUserSensor == null) {
            mUserSensor = build(mDevice);
            UserDriverManager.getInstance().registerSensor(mUserSensor);
        }
    }

    /**
     * Unregister the driver from the framework.
     */
    public void unregister() {
        if (mUserSensor != null) {
            UserDriverManager.getInstance().unregisterSensor(mUserSensor);
            mUserSensor = null;
        }
    }

    static UserSensor build(final Adx345 adx345) {
        return new UserSensor.Builder()
                .setType(Sensor.TYPE_ACCELEROMETER)
                .setName(DRIVER_NAME)
                .setVendor(DRIVER_VENDOR)
                .setVersion(DRIVER_VERSION)
                .setUuid(UUID.randomUUID())
                .setDriver(new UserSensorDriver() {
                    @Override
                    public UserSensorReading read() throws IOException {
                        float[] sample = adx345.readSample();
                        return new UserSensorReading(sample);
                    }

                    @Override
                    public void setEnabled(boolean enabled) throws IOException {
                        if (enabled) {
                            adx345.setConfigure();
                            adx345.powerOn();
                        } else {
                            // ToDo
                        }
                    }
                })
                .build();
    }



}
```

BoardDefault.java
```java
package com.gclue.adx345;

/*
 * Copyright 2016, The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

import android.os.Build;

@SuppressWarnings("WeakerAccess")
public class BoardDefaults {
    private static final String DEVICE_RPI3 = "rpi3";
    private static final String DEVICE_IMX6UL_PICO = "imx6ul_pico";
    private static final String DEVICE_IMX7D_PICO = "imx7d_pico";

    /**
     * Return the preferred I2C port for each board.
     */
    public static String getI2CPort() {
        switch (Build.DEVICE) {
            case DEVICE_RPI3:
                return "I2C1";
            case DEVICE_IMX6UL_PICO:
                return "I2C2";
            case DEVICE_IMX7D_PICO:
                return "I2C1";
            default:
                throw new IllegalStateException("Unknown Build.DEVICE " + Build.DEVICE);
        }
    }
}
```

MainActivity.java
```java
package com.gclue.adx345;

import android.app.Activity;
import android.content.Context;
import android.hardware.Sensor;
import android.hardware.SensorEvent;
import android.hardware.SensorEventListener;
import android.hardware.SensorManager;
import android.os.Bundle;
import android.util.Log;

import java.io.IOException;

/**
 * Skeleton of an Android Things activity.
 * <p>
 * Android Things peripheral APIs are accessible through the class
 * PeripheralManagerService. For example, the snippet below will open a GPIO pin and
 * set it to HIGH:
 *
 * <pre>{@code
 * PeripheralManagerService service = new PeripheralManagerService();
 * mLedGpio = service.openGpio("BCM6");
 * mLedGpio.setDirection(Gpio.DIRECTION_OUT_INITIALLY_LOW);
 * mLedGpio.setValue(true);
 * }</pre>
 * <p>
 * For more complex peripherals, look for an existing user-space driver, or implement one if none
 * is available.
 *
 * @see <a href="https://github.com/androidthings/contrib-drivers#readme">https://github.com/androidthings/contrib-drivers#readme</a>
 */
public class MainActivity extends Activity implements SensorEventListener {
    private Adx345AccelerometerDriver mAdx345AccelerometerDriver;
    private SensorManager mSensorManager;
    private static final String TAG = MainActivity.class.getSimpleName();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
        mSensorManager.registerDynamicSensorCallback(new SensorManager.DynamicSensorCallback() {
            @Override
            public void onDynamicSensorConnected(Sensor sensor) {
                if (sensor.getType() == Sensor.TYPE_ACCELEROMETER) {
                    Log.i(TAG, "Accelerometer sensor connected");
                    mSensorManager.registerListener(MainActivity.this, sensor,
                            SensorManager.SENSOR_DELAY_NORMAL);
                }
            }
        });
        try {
            mAdx345AccelerometerDriver = new Adx345AccelerometerDriver(BoardDefaults.getI2CPort());
            mAdx345AccelerometerDriver.register();
            Log.i(TAG, "Accelerometer driver registered");
        } catch (IOException e) {
            Log.e(TAG, "Error initializing accelerometer driver: ", e);
        }

    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        if (mAdx345AccelerometerDriver != null) {
            mSensorManager.unregisterListener(this);
            mAdx345AccelerometerDriver.unregister();
            try {
                mAdx345AccelerometerDriver.close();
            } catch (IOException e) {
                Log.e(TAG, "Error closing accelerometer driver: ", e);
            } finally {
                mAdx345AccelerometerDriver = null;
            }
        }
    }


    @Override
    public void onSensorChanged(SensorEvent event) {
        Log.i(TAG, "Accelerometer event: " +
                event.values[0] + ", " + event.values[1] + ", " + event.values[2]);
    }

    @Override
    public void onAccuracyChanged(Sensor sensor, int accuracy) {

    }
}
```


AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.gclue.adx345">
    <uses-permission android:name="com.google.android.things.permission.MANAGE_SENSOR_DRIVERS" />
    <uses-permission android:name="com.google.android.things.permission.USE_PERIPHERAL_IO" />

    <application android:label="@string/app_name">
        <uses-library android:name="com.google.android.things" />

        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```