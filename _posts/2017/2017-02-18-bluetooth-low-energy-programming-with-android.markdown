---
layout: "post"
title: "Bluetooth Low Energy Programming with Android"
date: "2017-02-18 16:47"
author: "Juntong Liu"
catalog: true
---

## Declare Permission

There are two bluetooth related permissions in Android:

```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
```

If you are using Android Studio, it will automatically notify the permissions required. The only thing you need to do is to apply them when needed.

## Get Bluetooth Manager

`BluetoothManager` is the top level API provided by Android related to Bluetooth. You can get different kinds of API using it. To get the `BluetoothManager`, you need to invoke following code within your `Activity`. Of course, you can pass the `Activity` to other places to invoke it, like a custom `BluetoothHelper`, dealing with all the stuff with Bluetooth.

```java
BluetoothManager btManager =
    (BluetoothManager)getSystemService(BLUETOOTH_SERVICE);
```

## Get Bluetooth Adapter

The `BluetoothAdapter` object is the abstract representation of the bluetooth chips on the device. It provides APIs related to hardware operations. You can get it using following code:

```java
BluetoothAdapter adapter = btManager.getAdapter();
```

One thing to be noticed is the function call will return a `null` if there is no Bluetooth adapter on the device. In such case, you might need to handle such situation. In addition, the adapter might not be enabled. You can check the state by calling `adapter.isEnabled()`. If it is not enabled, the following code might help to request to enable the adapter:


```java
Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
startActivityForResult(enableBtIntent, Activity.RESULT_OK);
```

At this time, the system will open a dialogue to request to enable the adapter. The result of this request can be listened by overriding related functions of your `Activity`.


```java
@Override
protected void onActivityResult(int request, int result, Intent data) {
    if (request == result)
        // Successfully opened
    else
        // Request refused
}
```

## Search For Device

After getting the 'BluetoothAdapter', we can start to search for devices. In old version of Android, there is no independent interface for Bluetooth LE, the only way is to use the following code to search. Here you need a callback which will be called when remote devices are found, to pass the device.

```java
adapter.startLeScan(LeScanCallback callback);
```

In new version of Android, you can use following code instead. The `ScanCallback` is a better implemented callback. In addition to the device, it also passes many other information like the strength of signal, the time stamp of connection, available services and other information within a `ScanResult` object, these will be covered later.

```java
adapter.getBluetoothLeScanner().startScan(ScanCallback callback)
```

The most significant difference between these two implementations is the second one provides more decoded information. If you have to use the first one, you can refer to [this article][1] to decode the information.

## The Structure of Bluetooth LE

After starting the search, we will start to receive some devices, but how to find the device that we want? The easiest method is to filter by device name. You can invoke `bluetoothDevice.getName()` to do this. However, we might want more precise filtering in some cases. To do this, we need to know the structure of Bluetooth LE. Every BLE devices will provide several `Service`s, each `Service` represent one thing this device can do. For example, serial communication (UART), or wireless input. The `Service` can be distinguished by its `UUID`. There are several `UUID`s already defined universally alone with some specified by the manufacturer. Using some apps that will show the Bluetooth LE information will make your life much easier to know everything about one device. Here I uses the `UUID` provided by the manufacturer `"6e400001-b5a3-f393-e0a9-e50e24dcca9e"`

To filter devices according to `Service`, the following code might be useful (Here I uses `ScanCallback`. If you are using `LeScanCallback`, you might need to decode the `UUID`s manually).

```java
UUID UUID_SERVICE = UUID.fromString("6e400001-b5a3-f393-e0a9-e50e24dcca9e");
for (ParcelUuid uuid : result.getScanRecord().getServiceUuids()) {
    if (uuid.getUuid().equals(UUID_SERVICE)) {
        // Some operation
    }
}
```

Here the `result` is the `ScanResult` passed in.

There are some information within every `Service`, which is called `Characteristic`. This is basically the actual content of BLE communication. Two devices can communicate by reading and writing `Characteristic`s. The `Characteristic`s is distinguished by `UUID` as well.

## Establish Connection

After the former section, you might have founded the device to connect. We will start connection in this step. Assuming the device is called `device`, connection can be started using the following code:

```java
BluetoothGatt gatt =
    device.connectGatt(context, false, new MyBluetoothGattCallback());
```

The `context` here can be you `Activity`, the second parameter is whether to connect automatically when available, the third parameter is a customized `BluetoothGattCallback`, which is used to listen to many event in the connection to be mentioned later. We will use `GattCallback` as a simplification.

The function call will return an object of `BluetoothGatt`, this object can be a representation of the connection. This does not mean the connection has been established successfully while it only means the device is trying to connect. Usually the process will cost a very short time within 1 second, but things might go wrong in some special cases. For example the remote device has been occupied or the device is not ready. The events will be passed to the `GattCallback`. There are some operations you might want to do to establish the communication.

#### `onConnectionStateChange`

When the connection state changes (including connected and disconnected), this method will be fired. The following code might be of help:

```java
@Override
public void onConnectionStateChange(BluetoothGatt gatt,
        int status, int newState) {
    if (newState == BluetoothProfile.STATE_CONNECTED) {
        gatt.discoverServices();  // important
        // Connected
    } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
        // Disconnected
    }
}
```

The most important thing to do is to search for services here. Otherwise, no `Service` can be found by the `gatt` even though with connection is established.

#### `onServicesDiscovered`

Right after starting discover of services, this method will be fired. Here we need to register the listener of some services.

```java
@Override
public void onServicesDiscovered(BluetoothGatt gatt, int status) {
    final UUID UUID_SERVICE =
            UUID.fromString("6e400001-b5a3-f393-e0a9-e50e24dcca9e");
    final UUID UUID_RX =
            UUID.fromString("6e400003-b5a3-f393-e0a9-e50e24dcca9e");
    final UUID CHARACT_CONFIG =
            UUID.fromString("00002902-0000-1000-8000-00805f9b34fb");
    service = gatt.getService(UUID_SERVICE);  // get Service
    BluetoothGattCharacteristic characteristic =
            service.getCharacteristic(UUID_RX);  // get Characteristic
    BluetoothGattDescriptor descriptor =
            characteristic.getDescriptor(CHARACT_CONFIG);  // get Descriptor
    // Register listener to remote device
    descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
    gatt.writeDescriptor(descriptor);  // send operation
    // register locally
    gatt.setCharacteristicNotification(characteristic, true);
}
```

The `UUID`s here comes documentation of the manufacturer as well. At this step you need to both register listener to remote device and locally. Otherwise you will not be informed when data changed. In other words, `onCharacteristicChanged` will not be fired.

## Receive information

After registering listeners in the last step, the method `onCharacteristicChanged` will start to receive information passed in. The following code applies some check to the `Characteristic`, then reads the data. You might want to convert the data to `String`. You can use `new String(bytes, Charset.fromString("UTF-8"))` to do the trick.

```java
@Override
public void onCharacteristicChanged(BluetoothGatt gatt,
        BluetoothGattCharacteristic characteristic) {
    if (characteristic.getService().getUuid().equals(UUID_SERVICE)) {
        if (characteristic.getUuid().equals(UUID_RX)) {
            final byte[] bytes = characteristic.getValue();
            // Apply operations
        }
    }
}
```

## Send information

Since sending message is an active operation, this is slightly easier to do. The following code is given for reference:

```java
BluetoothGattCharacteristic characteristic =
        service.getCharacteristic(UUID_TX);
if (characteristic != null) {
    characteristic.setValue(chunk);
    gatt.writeCharacteristic(characteristic);
}
```

There is nearly no restriction of when to invoke it, checking the state of connection will make it safer. One thing to be noticed is to store the reference of objects to call the function whenever you want.

## Disconnect

I would highly suggest you to close the connection when you are finished. At least on my device, you the connection is not fully closed, the connection next time will cost more time to establish. The following code can be used.

```java
gatt.disconnect();
gatt.close();
```

Currently, I'm using this code in the 'onDestroy' of the `Activity`. You can put them wherever you want.

## Conclusion

This article describes the essential steps to establish a BLE connection. Due to the limit of the length, I'm not going to describe everything of the mechanism. Hopefully this can help starters to get started. For a full implementation, you can refer to [my helper class][2], or read the source code of [Adafruit BLE Application][3].

[1]: http://stackoverflow.com/questions/24003777/read-advertisement-packet-in-android
[2]: https://github.com/HelloRobotics/CarController/blob/master/app/src/main/java/io/github/hellorobotics/carcontroller/utils/HelperBle.java
[3]: https://github.com/adafruit/Adafruit_Android_BLE_UART
