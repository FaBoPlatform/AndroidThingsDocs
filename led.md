# LED

GPIO22にLEDブリックを接続し、GPIO1_IO10を指定
1秒ごとにLEDを点滅。

## ソース

```java

import android.app.Activity;
import android.os.Handler;
import android.os.Bundle;
import android.util.Log;
import com.google.android.things.pio.Gpio;
import com.google.android.things.pio.PeripheralManager;

import java.io.IOException;

public class MainActivity extends Activity {

    private final static String TAG = "THINGS";
    private Gpio mLedGpio;
    private String PIN_NAME = "GPIO1_IO10";
    private Handler mHandler = new Handler();
    private static final int INTERVAL_BETWEEN_BLINKS_MS = 1000;
    private Boolean trigger = false;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        try {
            mLedGpio = PeripheralManager.getInstance().openGpio(PIN_NAME);
            mLedGpio.setDirection(Gpio.DIRECTION_OUT_INITIALLY_LOW);

            mHandler.post(mBlinkRunnable);
        } catch (IOException e) {
            Log.e(TAG, "Error on PeripheralIO API", e);
        }
    }

    private Runnable mBlinkRunnable = new Runnable() {
        @Override
        public void run() {

            if (mLedGpio == null) {
                return;
            }

            try {
                trigger = !trigger;
                mLedGpio.setValue(!mLedGpio.getValue());

                mHandler.postDelayed(mBlinkRunnable, INTERVAL_BETWEEN_BLINKS_MS);
            } catch (IOException e) {
                Log.e(TAG, "Error on PeripheralIO API", e);
            }
        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mLedGpio != null) {
            try {
                mLedGpio.close();
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
