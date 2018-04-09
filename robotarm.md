# RobotARM

6DOF RobotARMの制御

## ソース


```java
package com.gclue.restfullarm;

import android.app.Activity;
import android.os.Handler;
import android.os.Bundle;
import android.util.Log;
import com.google.android.things.pio.Gpio;
import com.google.android.things.pio.I2cDevice;
import com.google.android.things.pio.PeripheralManager;

import java.io.IOException;

import static java.lang.Math.*;

public class MainActivity extends Activity {

    private final static String TAG = "THINGS";
    private Gpio[] mLedGpio = new Gpio[5];
    private String[] PIN_NAME =  {"BCM4", "BCM5", "BCM6", "BCM12","BCM21"};
    private Handler mHandler = new Handler();
    private static final int INTERVAL_BETWEEN_BLINKS_MS = 1000;
    private Boolean trigger = false;
    private I2cDevice mDevice;
    // I2C Device Name
    private static final String I2C_DEVICE_NAME = "I2C1";
    // I2C Slave Address
    private static final double OSC_CLOCK = 25000000.0;
    private static final int I2C_ADDRESS = 0x40;
    private static final int MODE1 = 0x00; // Mode register 1
    private static final int MODE2 = 0x01; // Mode register 2
    private static final int ALL_LED_ON_L = 0xFA; // load all the LEDn_ON registers, byte 0
    private static final int ALL_LED_ON_H = 0xFB; // load all the LEDn_ON registers, byte 1
    private static final int ALL_LED_OFF_L = 0xFC; // load all the LEDn_OFF registers, byte 0
    private static final int ALL_LED_OFF_H = 0xFD; // load all the LEDn_OFF registers, byte 1
    private static final int ALLCALL = 0x01;
    private static final int OUTDRV = 0x04; // 2bit
    private static final int SLEEP = 0x10;
    private static final int PRE_SCALE = 0xFE; // prescaler for PWM output frequency
    private static final int RESTART = 0x80;
    private static final int LED0_ON_L = 0x06;
    private static final int LED0_ON_H = 0x07;
    private static final int LED0_OFF_L = 0x08;
    private static final int LED0_OFF_H = 0x09;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        try {

            for(int i = 0; i < mLedGpio.length; i++) {
                mLedGpio[i] = PeripheralManager.getInstance().openGpio(PIN_NAME[i]);
                mLedGpio[i].setDirection(Gpio.DIRECTION_OUT_INITIALLY_LOW);
            }

            mDevice = PeripheralManager.getInstance().openI2cDevice(I2C_DEVICE_NAME, I2C_ADDRESS);

            init();
            mHandler.post(mBlinkRunnable);
        } catch (IOException e) {
            Log.e(TAG, "Error on PeripheralIO API", e);
        }
    }

    private void init() {
        // 通電時、PCA9685の全channleの値は、サーボ稼働範囲外の4096になっているのでこれを適切な範囲に設定しておく
        int default_value = 300;
        try {
            set_all_pwm(0,0);
            /*mDevice.writeRegByte(ALL_LED_ON_L,(byte)0x00);
            mDevice.writeRegByte(ALL_LED_ON_H,(byte)0x00);
            mDevice.writeRegByte(ALL_LED_OFF_L,(byte)(default_value & (byte)0xFF));
            mDevice.writeRegByte(ALL_LED_OFF_H,(byte)(default_value >> 8));*/
            mDevice.writeRegByte(MODE2,(byte)(OUTDRV));
            mDevice.writeRegByte(MODE1,(byte)(ALLCALL));
            try {
                Thread.sleep((long) 50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            // スリープ状態なら解除する
            byte mode = mDevice.readRegByte(MODE1);
            mode = (byte) (mode & ~SLEEP); // SLEEPビットを除去する

            mDevice.writeRegByte(MODE1, mode);
            try {
                Thread.sleep((long) 50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void set_hz(int hz){
        //int prescale = calc_prescale(hz);

        double prescaleval = 25000000.0;    // # 25MHz
        prescaleval /= 4096.0;       // # 12-bit
        prescaleval /= (float)hz;
        prescaleval -= 1.0;
        int prescale = (int)(Math.floor(prescaleval + 0.5));

        byte oldmode = 0;
        try {
            oldmode = mDevice.readRegByte(MODE1);
            Log.i(TAG, "oldmode:" + oldmode);
            byte newmode = (byte) (oldmode | SLEEP); // sleep
            Log.i(TAG, "newmode:" + newmode);
            mDevice.writeRegByte(MODE1,(byte)newmode);
            Log.i(TAG, "prescale:" + prescale);
            mDevice.writeRegByte(PRE_SCALE, (byte) prescale);
            mDevice.writeRegByte(MODE1,(byte)oldmode);
            Log.i(TAG, "oldmode:" + oldmode);

            try {
                Thread.sleep((long) 50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            mDevice.writeRegByte(MODE1, (byte) (oldmode | RESTART));
            Log.i(TAG, "restart:" + (oldmode | RESTART));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /*private int calc_prescale(int hz){
        return (int) (Math.round(OSC_CLOCK/4096/(float)hz)-1);
    }*/
    private void set_all_pwm(int on, int off){
        try {
            mDevice.writeRegByte(ALL_LED_ON_L, (byte) (on & 0xFF));
            mDevice.writeRegByte(ALL_LED_ON_H, (byte) (on >> 8));
            mDevice.writeRegByte(ALL_LED_OFF_L, (byte) (off & 0xFF));
            mDevice.writeRegByte(ALL_LED_OFF_H, (byte) (off >> 8));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void set_channel_value(int channel, int on, int off) {
        try {

            mDevice.writeRegByte(LED0_ON_L+channel*4, (byte) (on & 0xFF));
            mDevice.writeRegByte(LED0_ON_H+channel*4, (byte) (on >> 8));

            mDevice.writeRegByte(LED0_OFF_L+channel*4, (byte) (off & 0xFF));
            mDevice.writeRegByte(LED0_OFF_H+channel*4, (byte) (off >> 8));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private Runnable mBlinkRunnable = new Runnable() {
        @Override
        public void run() {

            setValue(0, true, 5,100, 2000);
            setValue(1, true, 5,30, 2000);
            setValue(2, true, 10,100, 2000);
            setValue(3, true, 10,300, 2000);
            setValue(4, true, 10,100, 2000);

            setValue(0, false, 5,100, 2000);
            setValue(1, false, 5,30, 2000);
            setValue(2, false, 10,100, 2000);
            setValue(3, false, 10,300, 2000);
            setValue(4, false, 10,100, 2000);


        }
    };

    private void setValue(int channel, boolean turn, int hz, int value, int time) {
        try {

            mLedGpio[channel].setValue(turn);

            try {
                Thread.sleep((long) 50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            //set_channel_value(4,  0, 0);
            set_hz(hz);
            set_channel_value(channel,  0, value);

            try {
                Thread.sleep((long) time);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            set_channel_value(channel,  0, 0);

        } catch (IOException e) {
            Log.e(TAG, "Error on PeripheralIO API", e);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        for(int i = 0; i < mLedGpio.length; i++) {
            if (mLedGpio[i] != null) {
                try {
                    mLedGpio[i].close();
                } catch (IOException e) {
                    Log.e(TAG, "Error on PeripheralIO API", e);
                }
            }
        }
    }
}
```
