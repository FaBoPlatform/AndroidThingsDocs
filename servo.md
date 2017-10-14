# Servo

PWMで、サーバを操作する。

## ソース

```java
package com.gclue.myapplication;

import android.os.Handler;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import com.google.android.things.pio.PeripheralManagerService;
import com.google.android.things.pio.Pwm;

import java.io.IOException;

public class MainActivity extends AppCompatActivity {
    Servo mServo;
    private final static String TAG = "THINGS";
    private Pwm mPwm;
    private static final String PWM_NAME = "PWM1";
    private Handler mHandler = new Handler();
    private static final int INTERVAL_BETWEEN_BLINKS_MS = 1000;
    private int freqValue = 100;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        PeripheralManagerService manager = new PeripheralManagerService();
        try {
           mPwm = manager.openPwm(PWM_NAME);
           initializePwm(mPwm);
           mHandler.post(mBlinkRunnable);
        } catch (IOException e) {
            Log.e(TAG, "Unable to access PWM", e);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mPwm != null) {
            try {
                mPwm.close();
                mPwm = null;
            } catch (IOException e) {
                Log.e(TAG, "Unable to close PWM", e);
            }
        }
    }

    public void initializePwm(Pwm pwm) throws IOException {
        pwm.setPwmFrequencyHz(120);
        pwm.setPwmDutyCycle(25);

        pwm.setEnabled(true);
    }

    private Runnable mBlinkRunnable = new Runnable() {
        @Override
        public void run() {

            if (mPwm == null) {
                return;
            }

            try {
                freqValue+=10;
                if(freqValue > 300) {
                    freqValue = 100;
                }
                mPwm.setPwmFrequencyHz(freqValue);
                mHandler.postDelayed(mBlinkRunnable, INTERVAL_BETWEEN_BLINKS_MS);
            } catch (IOException e) {
                Log.e(TAG, "Error on PWM", e);
            }
        }
    };

}
```
