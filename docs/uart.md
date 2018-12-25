# UART

## ソース

```java
package io.fabo.uartexample;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;

import com.google.android.things.pio.PeripheralManager;
import com.google.android.things.pio.UartDevice;
import com.google.android.things.pio.UartDeviceCallback;

import java.io.IOException;
import java.util.List;

public class MainActivity extends Activity {
    // UART Device Name

    private UartDevice mDevice;

    private static final String TAG = "UART";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Attempt to access the UART device
        try {
            PeripheralManager manager = PeripheralManager.getInstance();
            mDevice = manager.openUartDevice(BoardDefaults.getUartName());
            configureUartFrame(mDevice);
            mDevice.registerUartDeviceCallback(mUartCallback);

            // Send command.
            byte[] buffer = {(byte)0x63};
            writeUartData(mDevice, buffer);
        } catch (IOException e) {
            Log.w(TAG, "Unable to access UART device", e);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mDevice != null) {
            try {
                mDevice.close();
                mDevice = null;
            } catch (IOException e) {
                Log.w(TAG, "Unable to close UART device", e);
            }
        }
    }

    /**
     * Configure the UART port.
     * @param uart
     * @throws IOException
     */
    public void configureUartFrame(UartDevice uart) throws IOException {
        uart.setBaudrate(9600);
        uart.setDataSize(8);
        uart.setParity(UartDevice.PARITY_NONE);
        uart.setStopBits(1);
    }

    /**
     * Callback.
     */
    private UartDeviceCallback mUartCallback = new UartDeviceCallback() {
        @Override
        public boolean onUartDeviceDataAvailable(UartDevice uart) {
            try {
                readUartBuffer(uart);
            } catch (IOException e) {
                Log.w(TAG, "Unable to access UART device", e);
            }
            return true;
        }

        @Override
        public void onUartDeviceError(UartDevice uart, int error) {
            Log.w(TAG, uart + ": Error event " + error);
        }
    };


    /**
     * Read uart buffer.
     * @param uart
     * @throws IOException
     */
    public void readUartBuffer(UartDevice uart) throws IOException {
        final int maxCount = 8;
        byte[] buffer = new byte[maxCount];

        int count;
        while ((count = uart.read(buffer, buffer.length)) > 0) {
            Log.d(TAG, "Read " + count + " bytes from peripheral");
        }

        for(int i = 0; i < buffer.length; i++) {
            Log.d(TAG, i + ":0x" + Integer.toHexString(buffer[i]&0xff));
        }
    }

    /**
     * Write uart.
     * @param uart
     * @param command
     * @throws IOException
     */
    public void writeUartData(UartDevice uart, byte[] command) throws IOException {
        int count = uart.write(command, command.length);

        for(int i = 0; i < count; i++) {
            Log.d(TAG, i + ":0x" + Integer.toHexString(command[i]&0xff));
        }
    }
}

```

##  AndroidManifest
>     <uses-permission android:name="com.google.android.things.permission.USE_PERIPHERAL_IO" />

