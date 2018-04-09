# Button

GPIO4にLEDブリックを接続し、1秒ごとにLEDを点滅。

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
    private String PIN_NAME = "BCM4";
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
