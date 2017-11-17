# 3Axis

I2Cに3Axisを接続し、100ms毎に加速度を取得。TextViewをUIに表示する。

## ソース

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView android:text="DX"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/textViewX"
        />

    <TextView android:text="DY"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/textViewY"
        android:layout_below="@id/textViewX"
        />

    <TextView android:text="DZ"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/textViewZ"
        android:layout_below="@+id/textViewY"

        />

</RelativeLayout>
```


```java
package com.gclue.myapplication;

import android.os.Handler;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import com.google.android.things.pio.I2cDevice;
import com.google.android.things.pio.PeripheralManagerService;
import java.io.IOException;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
    // I2C Device Name
    private static final String I2C_DEVICE_NAME = "I2C1";
    // I2C Slave Address
    private static final int I2C_ADDRESS = 0x53;
    private static final String TAG = "THINGS";
    private Handler mHandler = new Handler();
    private static final int INTERVAL_BETWEEN_AXIS_MS = 100;

    private I2cDevice mDevice;
    /** Who_am_i register */
    private byte ADXL345_DEVID_REG = 0x00;
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

    /** TextView */
    TextView mTextViewDX;
    TextView mTextViewDY;
    TextView mTextViewDZ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mTextViewDX = (TextView) findViewById(R.id.textViewX);
        mTextViewDY = (TextView) findViewById(R.id.textViewY);
        mTextViewDZ = (TextView) findViewById(R.id.textViewZ);

        // Attempt to access the I2C device
        try {
            PeripheralManagerService manager = new PeripheralManagerService();
            mDevice = manager.openI2cDevice(I2C_DEVICE_NAME, I2C_ADDRESS);
        } catch (IOException e) {
            Log.w(TAG, "Unable to access I2C device", e);
        }

        try {
            // Who am I.
            byte value = mDevice.readRegByte(0x00);
            if((value & 0xff) == 0xe5) {
                Log.i(TAG, "Device is ADXL345");
            }

            // Configure.
            byte conf = ADXL345_SELF_TEST_OFF;
            conf |= ADXL345_SPI_OFF;
            conf |= ADXL345_INT_INVERT_OFF;
            conf |= ADXL345_FULL_RES_OFF;
            conf |= ADXL345_JUSTIFY_OFF;
            conf |= ADXL345_RANGE_16G;
            mDevice.writeRegByte(ADXL345_DATA_FORMAT_REG, conf);

            // PowerOn.
            byte power = ADXL345_AUTO_SLEEP_OFF;
            power |= ADXL345_MEASURE_ON;
            power |= ADXL345_SLEEP_OFF;
            power |= ADXL345_WAKEUP_8HZ;
            mDevice.writeRegByte(ADXL345_POWER_CTL_REG, power);

            // Handler.
            mHandler.post(mAxisRunnable);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private Runnable mAxisRunnable = new Runnable() {
        @Override
        public void run() {

            if (mDevice == null) {
                return;
            }

            try {
                int length = 6;
                byte axis_buff[] = new byte[length];
                mDevice.readRegBuffer(ADXL345_3AIXS, axis_buff, axis_buff.length);
                mHandler.postDelayed(mAxisRunnable, INTERVAL_BETWEEN_AXIS_MS);
                int x = (((int)axis_buff[1]) << 8) | axis_buff[0];
                int y = (((int)axis_buff[3]) << 8) | axis_buff[2];
                int z = (((int)axis_buff[5]) << 8) | axis_buff[4];

                mTextViewDX.setText("X: " + x);
                mTextViewDY.setText("Y: " + y);
                mTextViewDZ.setText("Z: " + z);

                Log.i(TAG, "x=" + x + " y=" + y + " z=" + z);

            } catch (IOException e) {
                Log.e(TAG, "Error on I2C Device API", e);
            }
        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mDevice != null) {
            try {
                mDevice.close();
                mDevice = null;
            } catch (IOException e) {
                Log.w(TAG, "Unable to close I2C device", e);
            }
        }
    }

}


```
