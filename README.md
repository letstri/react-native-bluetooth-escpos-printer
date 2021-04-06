# @brooons/react-native-bluetooth-escpos-printer

> Cloned and changed from https://github.com/januslo/react-native-bluetooth-escpos-printer

[![npm version](https://badge.fury.io/js/%40brooons%2Freact-native-bluetooth-escpos-printer.svg)](https://www.npmjs.com/package/@brooons/react-native-bluetooth-escpos-printer)

React-Native plugin for the bluetooth ESC/POS & TSC printers.

Any questions or bug please raise a issue.

## Features

- Still under development
- May support Android only
- Typescript support

## Table of Contents

- [Installation](#installation)
- [Usage and APIs](#usage-and-apis)
  - [BluetoothManager](#bluetoothmanager)
  - [BluetoothTscPrinter](#bluetoothtscprinter)
  - [BluetoothEscposPrinter](#bluetoothescposprinter)

## Installation

### Step 1

Install via NPM [Check In NPM](https://www.npmjs.com/package/@brooons/react-native-bluetooth-escpos-printer)

```bash
npm install @brooons/react-native-bluetooth-escpos-printer --save
```

### Step 2

Link the plugin to your RN project

```bash
react-native link @brooons/react-native-bluetooth-escpos-printer
```

### Manual linking (Android)

Ensure your build files match the following requirements:

1. (React Native 0.59 and lower) Define the *`react-native-bluetooth-escpos-printer`* project in *`android/settings.gradle`*:

```
include ':react-native-bluetooth-escpos-printer'
project(':react-native-bluetooth-escpos-printer').projectDir = new File(rootProject.projectDir, '../node_modules/@brooons/react-native-bluetooth-escpos-printer/android')
```
2. (React Native 0.59 and lower) Add the *`react-native-bluetooth-escpos-printer`* as an dependency of your app in *`android/app/build.gradle`*:
```
...
dependencies {
  ...
  implementation project(':react-native-bluetooth-escpos-printer')
}
```

3. (React Native 0.59 and lower) Add *`import cn.jystudio.bluetooth.RNBluetoothEscposPrinterPackage;`* and *`new RNBluetoothEscposPrinterPackage()`* in your *`MainApplication.java`* :

### Step 3

Refers to your JS files

```javascript
import { BluetoothManager, BluetoothEscposPrinter, BluetoothTscPrinter } from '@brooons/react-native-bluetooth-escpos-printer';
```

## Usage and APIs

### BluetoothManager

BluetoothManager is the module for Bluetooth service management, supports Bluetooth status check, enable/disable Bluetooth service, scan devices, connect/unpair devices.

#### checkBluetoothEnabled(): Promise<void>

Async function, checks whether Bluetooth service is enabled.

> // TODO: consider to return the the devices information already bound and paired here..

```javascript
const isEnabled = await BluetoothManager.checkBluetoothEnabled();

console.log(isEnabled); // true/false
```

#### enableBluetooth(): Promise<void>

Async function, enables the bluetooth service, returns the devices information already bound and paired.

```javascript
const devices = await BluetoothManager.enableBluetooth();

return devices
  .reduce((acc, device) => {
    try {
      return [...acc, JSON.parse(device)];
    } catch (e) {
      return acc;
    }
  }, [])
  .filter((device) => device.address);
```

#### disableBluetooth(): Promise<void>

Async function, disables the bluetooth service.

```javascript
await BluetoothManager.disableBluetooth();
```

#### scanDevices(): Promise<void>

Async function, scans the bluetooth devices, returns devices found and paired after scan finish. Event [BluetoothManager.EVENT_DEVICE_ALREADY_PAIRED] would be emitted with devices bound; event [BluetoothManager.EVENT_DEVICE_FOUND] would be emitted (many time) as long as new devices found.

samples with events:
```javascript
DeviceEventEmitter.addListener(
  BluetoothManager.EVENT_DEVICE_ALREADY_PAIRED, (response) => {
    console.log(response);
    // response.devices would returns the paired devices array in JSON string.
  }
);
DeviceEventEmitter.addListener(
  BluetoothManager.EVENT_DEVICE_FOUND, (response) => {
    console.log(response);
    // response.devices would returns the found device object in JSON string
  }
);
```

Sample with scanDevices function

```javascript
const {
  found,
  paired,
}: {
  found: Array<{ address: string; name?: string }>;
  paired: Device[];
} = JSON.parse(await BluetoothManager.scanDevices());

return {
  paired,
  found: found.filter((device) => !!device.name),
};
```

#### connect(address: string): Promise<void>
async function, connects the specified device, if not bound, bound dailog prompts.

```javascript
await BluetoothManager.connect(address);
```

#### disconnect(address: string): Promise<void>
async function, disconnects the specified device.

```javascript
await BluetoothManager.disconnect(address);
```

#### getConnectedDeviceAddress(): Promise<void>
async function, Return the address of the currently connected device .

```javascript
await BluetoothManager.getConnectedDeviceAddress();
```

#### unpair(address: string): Promise<void>
async function, disconnects and unpairs the specified devices

```javascript
await BluetoothManager.unpair(address);
```

#### Events of BluetoothManager module

| Name/KEY | DESCRIPTION |
|---|---|
| EVENT_DEVICE_ALREADY_PAIRED | Emits the devices array already paired |
| EVENT_DEVICE_DISCOVER_DONE | Emits when the scan done |
| EVENT_DEVICE_FOUND | Emits when device found during scan |
| EVENT_CONNECTION_LOST | Emits when device connection lost |
| EVENT_UNABLE_CONNECT | Emits when error occurs while trying to connect device |
| EVENT_CONNECTED | Emits when device connected |
| EVENT_BLUETOOTH_NOT_SUPPORT | Emits when device not support bluetooth(android only) |

### BluetoothTscPrinter

The printer for label printing.

#### printLabel(): Promise<void>

Async function that performs the label print action.

```javascript
await BluetoothTscPrinter.printLabel(options);
```

##### Options

```typescript
interface IPrintLabelOptions {
  width?: number;
  height?: number;
  gap?: number;
  direction?: DIRECTION;
  reference?: [number, number];
  tear?: TEAR,
  sound?: number,
  text?: Array<{
    text: string;
    x: number;
    y: number;
    fonttype: FONTTYPE;
    rotation: ROTATION;
    xscal: FONTMUL;
    yscal: FONTMUL;
  }>;
  qrcode?: Array<{
    x: number;
    y: number;
    level: EEC;
    width: number;
    rotation: ROTATION;
    code: string;
  }>;
  barcode?: Array<{
    x: number;
    y: number;
    type: BARCODETYPE;
    height: number;
    readable: number;
    rotation: ROTATION;
    code: string;
  }>;
  image?: Array<{
    x: number;
    y: number;
    mode: BITMAP_MODE,
    width: number,
    image: string;
  }>;
}
```

##### Demo of printLabel()

```javascript
const base64Image = 'some_base_64';

const options = {
  width: 40,
  height: 30,
  gap: 20,
  direction: BluetoothTscPrinter.DIRECTION.FORWARD,
  reference: [0, 0],
  tear: BluetoothTscPrinter.TEAR.ON,
  sound: 0,
  text: [
    {
      text: 'I am a testing txt',
      x: 20,
      y: 0,
      fonttype: BluetoothTscPrinter.FONTTYPE.SIMPLIFIED_CHINESE,
      rotation: BluetoothTscPrinter.ROTATION.ROTATION_0,
      xscal:BluetoothTscPrinter.FONTMUL.MUL_1,
      yscal: BluetoothTscPrinter.FONTMUL.MUL_1,
    },
    {
      text: '你在说什么呢?',
      x: 20,
      y: 50,
      fonttype: BluetoothTscPrinter.FONTTYPE.SIMPLIFIED_CHINESE,
      rotation: BluetoothTscPrinter.ROTATION.ROTATION_0,
      xscal:BluetoothTscPrinter.FONTMUL.MUL_1,
      yscal: BluetoothTscPrinter.FONTMUL.MUL_1,
    },
  ],
  qrcode: [
    {
      x: 20,
      y: 96,
      level: BluetoothTscPrinter.EEC.LEVEL_L,
      width: 3,
      rotation: BluetoothTscPrinter.ROTATION.ROTATION_0,
      code: 'show me the money',
    },
  ],
  barcode: [
    {
      x: 120,
      y: 96,
      type: BluetoothTscPrinter.BARCODETYPE.CODE128,
      height: 40,
      readable: 1,
      rotation: BluetoothTscPrinter.ROTATION.ROTATION_0,
      code: '1234567890',
    },
  ],
  image: [
    {
      x: 160,
      y: 160,
      mode: BluetoothTscPrinter.BITMAP_MODE.OVERWRITE,
      width: 60,
      image: base64Image,
    },
  ],
};
```
### BluetoothEscposPrinter

the printer for receipt printing, following ESC/POS command.

#### printerInit(): Promise<void>

Init the printer.

#### printAndFeed(feed: number): Promise<void>

Printer the buffer data and feed (feed lines).

#### printerLeftSpace(space: number): Promise<void>

Set the printer left spaces.

#### printerLineSpace(space: number): Promise<void>

Set the spaces between lines.

#### printerUnderLine(line: number): Promise<void>

Set the underline of the text.

- 0 - off
- 1 - on
- 2 - deeper

#### printerAlign(align: number): Promise<void>

Set the printer alignment.

Constansts:
- BluetoothEscposPrinter.ALIGN.LEFT
- BluetoothEscposPrinter.ALIGN.CENTER
- BluetoothEscposPrinter.ALIGN.RIGHT

Does not work on printPic() method.

#### printText(text: string, options: object): Promise<void>

Print text.

Options as following:

- encoding - text encoding, default `GBK`.
- codepage - codepage using, default `0`.
- widthtimes - text font mul times in width, default `0`.
- heigthTimes - text font mul times in height, default `0`.
- fonttype - text font type, default `0`.

#### printColumn(columnWidths: Array<number>, columnAligns: Array<number>, columnTexts: Array<string>, options: object: Promise<void>

Print texts in column, Parameters as following:

- columnWidths - int arrays, configs the width of each column, calculate by english character length. ex:the width of "abcdef" is 5 ,the width of "中文" is 4.
- columnAligns - arrays, alignment of each column, values is the same of printerAlign().
- columnTexts - arrays, the texts of each colunm to print.
- options - text print config options, the same of printText() options.

#### setWidth(width: number): Promise<void>

Sets the width of the printer.

Constansts:
- BluetoothEscposPrinter.DEVICE_WIDTH.WIDTH_58
- BluetoothEscposPrinter.DEVICE_WIDTH.WIDTH_80

#### printPic(base64encodeStr: string, options: object): Promise<void>

Prints the image which is encoded by base64, without schema.

Options contains the params that may use in printing pic: "width": the pic width, basic on devices width(dots,58mm-384); "left": the left padding of the pic for the printing position adjustment.

#### selfTest(): Promise<void>

Printer self test.

#### rotate(): Promise<void>

Sets the rotation of the line.

#### setBlob(weight: number): Promise<void>

Sets blob of the line.

#### printQRCode(content: string, size: number, correctionLevel: number): Promise<void>

Prints the QRCode.

#### printBarCode(str: string, nType: number, nWidthX: number, nHeight: number, nHriFontType: number, nHriFontPosition: number): Promise<void>

Prints the barcode.

#### cutOnePoint(): Promise<void>

Cut a paper.

#### Demos of printing a receipt

```javascript
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.CENTER);
await BluetoothEscposPrinter.setBlob(0);
await BluetoothEscposPrinter.printText("广州俊烨\n\r",{
  encoding: 'GBK',
  codepage: 0,
  widthtimes: 3,
  heigthtimes: 3,
  fonttype: 1,
});
await BluetoothEscposPrinter.setBlob(0);
await BluetoothEscposPrinter.printText("销售单\n\r",{
  encoding: 'GBK',
  codepage: 0,
  widthtimes: 0,
  heigthtimes: 0,
  fonttype: 1,
});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.LEFT);
await BluetoothEscposPrinter.printText("客户：零售客户\n\r",{});
await BluetoothEscposPrinter.printText("单号：xsd201909210000001\n\r",{});
await BluetoothEscposPrinter.printText("日期："+(dateFormat(new Date(), "yyyy-mm-dd h:MM:ss"))+"\n\r",{});
await BluetoothEscposPrinter.printText("销售员：18664896621\n\r",{});
await BluetoothEscposPrinter.printText("--------------------------------\n\r",{});
let columnWidths = [12,6,6,8];
await BluetoothEscposPrinter.printColumn(columnWidths,
  [BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.CENTER,BluetoothEscposPrinter.ALIGN.CENTER,BluetoothEscposPrinter.ALIGN.RIGHT],
  ["商品",'数量','单价','金额'],{});
await BluetoothEscposPrinter.printColumn(columnWidths,
  [BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.CENTER,BluetoothEscposPrinter.ALIGN.RIGHT],
  ["React-Native定制开发我是比较长的位置你稍微看看是不是这样?",'1','32000','32000'],{});
await BluetoothEscposPrinter.printText("\n\r",{});
await BluetoothEscposPrinter.printColumn(columnWidths,
  [BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.CENTER,BluetoothEscposPrinter.ALIGN.RIGHT],
  ["React-Native定制开发我是比较长的位置你稍微看看是不是这样?",'1','32000','32000'],{});
await BluetoothEscposPrinter.printText("\n\r",{});
await BluetoothEscposPrinter.printText("--------------------------------\n\r",{});
await BluetoothEscposPrinter.printColumn([12,8,12],
  [BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.LEFT,BluetoothEscposPrinter.ALIGN.RIGHT],
  ["合计",'2','64000'],{});
await BluetoothEscposPrinter.printText("\n\r",{});
await BluetoothEscposPrinter.printText("折扣率：100%\n\r",{});
await BluetoothEscposPrinter.printText("折扣后应收：64000.00\n\r",{});
await BluetoothEscposPrinter.printText("会员卡支付：0.00\n\r",{});
await BluetoothEscposPrinter.printText("积分抵扣：0.00\n\r",{});
await BluetoothEscposPrinter.printText("支付金额：64000.00\n\r",{});
await BluetoothEscposPrinter.printText("结算账户：现金账户\n\r",{});
await BluetoothEscposPrinter.printText("备注：无\n\r",{});
await BluetoothEscposPrinter.printText("快递单号：无\n\r",{});
await BluetoothEscposPrinter.printText("打印时间："+(dateFormat(new Date(), "yyyy-mm-dd h:MM:ss"))+"\n\r",{});
await BluetoothEscposPrinter.printText("--------------------------------\n\r",{});
await BluetoothEscposPrinter.printText("电话：\n\r",{});
await BluetoothEscposPrinter.printText("地址:\n\r\n\r",{});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.CENTER);
await BluetoothEscposPrinter.printText("欢迎下次光临\n\r\n\r\n\r",{});
await BluetoothEscposPrinter.printerAlign(BluetoothEscposPrinter.ALIGN.LEFT);
await BluetoothEscposPrinter.cutOnePoint();
```

#### Demo for opening the drawer

```javascript
BluetoothEscposPrinter.openDrawer(0, 250, 250);
```
