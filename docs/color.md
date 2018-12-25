# Color

203で色を取得

## ソース

```java
package com.gclue.myapplication;

import android.os.Handler;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;

import com.google.android.things.pio.Gpio;
import com.google.android.things.pio.I2cDevice;
import com.google.android.things.pio.PeripheralManagerService;
import java.io.IOException;

public class MainActivity extends AppCompatActivity {
    // I2C Device Name
    private static final String I2C_DEVICE_NAME = "I2C1";
    // I2C Slave Address
    private static final int I2C_ADDRESS = 0x2A;
    private static final String TAG = "THINGS";
    private Handler mHandler = new Handler();
    private static final int INTERVAL_BETWEEN_AXIS_MS = 100;

    private I2cDevice mDevice;
    /** Who_am_i register */
    private byte S11059_CONTROL = 0x00;
    /** Data Format Control */
    private byte S11059_DATA_RED_H = 0x03;
    private byte S11059_CTRL_GAIN  = 0x08;
    private byte S11059_CTRL_MODE  = 0x04;
    private byte S11059_CTRL_TIME_224M = 0x2;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Attempt to access the I2C device
        try {
            PeripheralManagerService manager = new PeripheralManagerService();
            mDevice = manager.openI2cDevice(I2C_DEVICE_NAME, I2C_ADDRESS);
        } catch (IOException e) {
            Log.w(TAG, "Unable to access I2C device", e);
        }

        try {
            // Setting
            byte value = mDevice.readRegByte(S11059_CONTROL);
            value |= S11059_CTRL_GAIN;
            value &= ~(S11059_CTRL_MODE);
            value &= 0xFC;
            value |= S11059_CTRL_TIME_224M;
            value &= 0x3F; // RESET off,SLEEP off
            mDevice.writeRegByte(S11059_CONTROL, value);

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
                int length = 8;
                byte color_buff[] = new byte[length];
                mDevice.readRegBuffer(S11059_DATA_RED_H, color_buff, color_buff.length);
                mHandler.postDelayed(mAxisRunnable, INTERVAL_BETWEEN_AXIS_MS);
                int r = (((int)color_buff[0]) << 8) | color_buff[1];
                int g = (((int)color_buff[2]) << 8) | color_buff[3];
                int b = (((int)color_buff[4]) << 8) | color_buff[5];
                int i = (((int)color_buff[6]) << 8) | color_buff[7];

                Log.i(TAG, "r=" + r + " g=" + g + " b=" + b + " i=" + i);

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
