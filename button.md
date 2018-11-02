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

AndroidManifest.xmlに<uses-permission android:name="com.google.android.things.permission.USE_PERIPHERAL_IO" /> を追加 

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.vssadmin.myapplication">
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
