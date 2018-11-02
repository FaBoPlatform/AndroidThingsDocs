# Button

GPIO22に差し込み、GPIO1_IO10を指定

## ソース

```java
package com.example.vssadmin.myapplication;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;

import com.google.android.things.pio.Gpio;
import com.google.android.things.pio.GpioCallback;
import com.google.android.things.pio.PeripheralManager;

import java.io.IOException;

public class MainActivity extends Activity {

    private final static String TAG = "THINGS";
    private Gpio mButtonGpio;
    private String PIN_NAME = "GPIO1_IO10";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        PeripheralManager manager = PeripheralManager.getInstance();
        try {
            mButtonGpio = manager.openGpio(PIN_NAME);
            mButtonGpio.setDirection(Gpio.DIRECTION_IN);
            mButtonGpio.setEdgeTriggerType(Gpio.EDGE_BOTH);
            mButtonGpio.registerGpioCallback(mCallback);
        } catch (IOException e) {
            Log.e(TAG, "Error on PeripheralIO API", e);
        }
    }

    private GpioCallback mCallback = new GpioCallback() {
        @Override
        public boolean onGpioEdge(Gpio gpio) {
            try {
                if(gpio.getValue() == true) {
                    Log.i(TAG, gpio.getName() + "が押された");
                } else {
                    Log.i(TAG, gpio.getName() + "が離された");
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

            return true;
        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mButtonGpio != null) {
            mButtonGpio.unregisterGpioCallback(mCallback);
            try {
                mButtonGpio.close();
            } catch (IOException e) {
                Log.e(TAG, "Error on PeripheralIO API", e);
            }
        }
    }
}

```
