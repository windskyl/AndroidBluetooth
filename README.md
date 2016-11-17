# AndroidBluetooth
1、安卓蓝牙聊天通迅 2、安卓BLE通信
## 说明
    1、在ble包中，是谷歌官方中提供的方式
    2、在BleConnectionActivity 中，是依据 ble包中的信息提取出来的 BLE设备连接通迅的
    3、在MainActiviy中 为扫描BLE 设备方法
    4、DeviceModel 为设置Model

#BLE协议通信
##检查蓝牙是否支持BLE协议
     boolean isBleUsed = getPackageManager().hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE);

## 初始化操作
     //获取 BluetoothManager
     mBluetoothManager = (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
     //获取 BluetoothAdapter
     mBluetoothAdapter = mBluetoothManager.getAdapter();
     //判断是否支持
     if (mBluetoothAdapter == null) {
          Log.e("ble","设备不支持蓝牙");
          Toast.makeText(this,"设备不支持蓝牙",Toast.LENGTH_SHORT).show();
      }else {
            Log.d("ble","设备支持蓝牙");
       }


## 开启扫描
###  开始扫描与停止
        /**
         * 扫描BLE设备
         */
        private boolean isScanBleDevice = false;
        private void scanBleDevice(){
            if (!isScanBleDevice) {
                //开始扫描
                mBluetoothAdapter.startLeScan(mScanCallback);
            }else {
                //停止扫描
                mBluetoothAdapter.stopLeScan(mScanCallback);
            }

        }
###  扫描到BLE设备回调方法
    //扫描到BLE设备的回调
        private BluetoothAdapter.LeScanCallback mScanCallback = new BluetoothAdapter.LeScanCallback() {

            @Override
            public void onLeScan(BluetoothDevice device, int rssi, byte[] scanRecord) {
                if (Looper.myLooper() == Looper.getMainLooper()) {
                    // Android 5.0 及以上

                } else {
                    // Android 5.0 以下
                    mHandler.post(new Runnable() {
                        @Override
                        public void run() {

                        }
                    });


                }
            }
        };


##连接BLE设备
###获取BLE设备
    点击列表跳转到设备连接页面，将BLE设备信息传递

    //对应的BLE蓝牙设备
    private BluetoothDevice mBluetoothDevice;

    //创建连接

    //连接服务对应的Gatt
    private BluetoothGatt mBluetoothGatt;

    mBluetoothGatt = mBluetoothDevice.connectGatt(this, true, mGattCallback);

    //连接服务的回调响应

    private final BluetoothGattCallback mGattCallback = new BluetoothGattCallback() {
        @Override
        public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
            if (newState == BluetoothProfile.STATE_CONNECTED) {

                Log.i(TAG, "Connected to GATT server.");
                Log.i(TAG, "Attempting to start service discovery:" +
                        mBluetoothGatt.discoverServices());
                isConnect = true;
            } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                isConnect = false;
                Log.i(TAG, "Disconnected from GATT server.");
            }
        }

        @Override
        public void onServicesDiscovered(BluetoothGatt gatt, int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                Log.w(TAG, "onServicesDiscovered gatt succeess received: " + status);
                //处理服务数据
                dataInitFunction();
            } else {
                Log.w(TAG, "onServicesDiscovered gatt other received: " + status);
            }
        }

        @Override
        public void onCharacteristicRead(BluetoothGatt gatt,
                                         BluetoothGattCharacteristic characteristic,
                                         int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {


            }

            byte[] value = characteristic.getValue();

            Log.d("ble--", "ble -- " + "收到消息 " + new String(value));
        }

        @Override
        public void onCharacteristicChanged(BluetoothGatt gatt,
                                            BluetoothGattCharacteristic characteristic) {

        }
    };

    当连接成功后，会回调方法 onServicesDiscovered
    当状态信息为 status == BluetoothGatt.GATT_SUCCESS 的时候，可以获取到外围设备中提供的服务

### 处理获取扫描到的外围设备的服务
    private void dataInitFunction() {
            //获取 外围设备中提供的服务
            List<BluetoothGattService> mBluetoothGattServices = mBluetoothGatt.getServices();

            if (mBluetoothGattServices != null) {
                //获取每个服务中
                for (BluetoothGattService mBluetoothGattService : mBluetoothGattServices) {
                    //获取服务的UUID
                    String uuid = mBluetoothGattService.getUuid().toString();

                    Log.e("ble--", "ble-- " + uuid);

                    //获取服务中对应的 所有的characteristic
                    List<BluetoothGattCharacteristic> characteristics = mBluetoothGattService.getCharacteristics();

                    for (BluetoothGattCharacteristic bluetoothGattCharacteristic : characteristics) {
                        //获取characteristic的信息
                        String charaUUid = bluetoothGattCharacteristic.getUuid().toString();
                        Log.e("ble--", "ble-- --- " + charaUUid);
                    }
                }
            }
        }

### 发送消息到 外围设备中
        private String mServiceUUIDString = "50b168cf-85fa-43e5-9665-a0faefb42a89";
        private String mServiceUUID = "beb07058-7edd-46de-af18-a7d4ae069e53";

        private boolean isConnect = true;

        //写入BLE 消息
        private void writeBluetooh() {
            if (mBluetoothGatt != null) {
                if (!isConnect) {
                    Log.e("ble", "连接断开，重新连接");
                    //连接已断开
                    mBluetoothGatt.connect();
                    return;
                }
                //获取指定类型的服务
                BluetoothGattService service = mBluetoothGatt.getService(UUID.fromString(mServiceUUIDString));
                if (service != null) {
                    //获取指定的 characteristic
                    BluetoothGattCharacteristic characteristic = service.getCharacteristic(UUID.fromString(mServiceUUID));
                    if (characteristic != null) {
                        //设置数据
                        characteristic.setValue("this is a bluetooth test ".getBytes());
                        //写入
                        mBluetoothGatt.writeCharacteristic(characteristic);
                    }
                }
            } else {
                Log.e("ble", "Gatt 为 null 重新创建");
                mBluetoothGatt = mBluetoothDevice.connectGatt(this, false, mGattCallback);
            }

        }
### 读取外围设备中的信息
        private String mServiceUUIDString = "50b168cf-85fa-43e5-9665-a0faefb42a89";
        private String mServiceUUID = "beb07058-7edd-46de-af18-a7d4ae069e53";

        private boolean isConnect = true;
        //读取BLE消息
        private void readBluetoohMsgFunction() {
            if (mBluetoothGatt != null) {
                 //获取指定类型的服务
                 BluetoothGattService service = mBluetoothGatt.getService(UUID.fromString(mServiceUUIDString));
                    if (service != null) {
                        //获取指定的 characteristic
                        BluetoothGattCharacteristic characteristic = service.getCharacteristic(UUID.fromString(mServiceUUID));
                        if (characteristic != null) {
                            //读取数据
                            boolean b = mBluetoothGatt.readCharacteristic(characteristic);
                            if (b) {
                                Log.e(TAG, "readBluetoohMsgFunction: 读取成功");
                            } else {
                                Log.e(TAG, "readBluetoohMsgFunction: 读取失败");
                            }
                        }
                    }
             }
        }

        那么会在gattcallback方法中回调 onCharacteristicRead，以获取读到的消息
        @Override
        public void onCharacteristicRead(BluetoothGatt gatt,
                                       BluetoothGattCharacteristic characteristic,
                                       int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {


            }

            byte[] value = characteristic.getValue();

             Log.d("ble--", "ble -- " + "收到消息 " + new String(value));
         }