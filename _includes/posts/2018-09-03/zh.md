### js-10进制-2进制-16进制数据间的相互转化

> 最近蓝牙前端开发，涉及了大量的数据格式转化，在此记录下。
```
export default class BleUtil {
  /**
   * 十进制转16进制
   */
  static intToHex(int) {
    let hex = int.toString(16);
    if (int < 16) {
      return 0x00 + hex;
    } else {
      return hex;
    }
  }
  /**
   * 十六进制转int
   */
  static hexToInt(string) {
    return parseInt(string, 16);
  }

  // 字符串转二进制数据
  static getBytes(string) {
    let array = new Uint8Array(string.length);
    for (var i = 0, l = string.length; i < l; i++) {
      array[i] = string.charCodeAt(i);
    }
    return array;
  }
  /**
   * 接收后封装
   */
  static bytesToString(buffer) {
    return String.fromCharCode.apply(null, new Uint8Array(buffer));
  }
  /**
   * buffer转16进制
   */
  static bytesToHex(buffer) {
    let str = this.bytesToString(buffer);
    let hexStr = '';
    for (let i = 0; i < str.length; i++) {
      hexStr += BleUtil.intToHex(str.charCodeAt(i));
    }
    return hexStr;
  }
  /**
   * 字符串转16进制
   * @param str
   */
  static string2Hex(str) {
    return BleUtil.intToHex(parseInt(str));
  }
  /**
   * 字符串为空
   */
  static isBlank(str) {
    if (typeof str == undefined || str == null) {
      return true;
    } else {
      return str.length == 0;
    }
  }
  /**
   * 16进制转字符串
   */
  static hexCharCodeToStr(hexCharCodeStr) {
  　　var trimedStr = hexCharCodeStr.trim();
  　　var rawStr =
  　　trimedStr.substr(0,2).toLowerCase() === "0x"
  　　?
  　　trimedStr.substr(2)
  　　:
  　　trimedStr;
  　　var len = rawStr.length;
  　　if(len % 2 !== 0) {
     console.log("Illegal Format ASCII Code!");
  　　　　return "";
  　　}
  　　var curCharCode;
  　　var resultStr = [];
  　　for(var i = 0; i < len;i = i + 2) {
  　　　　curCharCode = parseInt(rawStr.substr(i, 2), 16); // ASCII Code Value
  　　　　resultStr.push(String.fromCharCode(curCharCode));
  　　}
  　　return resultStr.join("");
  }
  /**
   * 16进制转buffer
   */
  static hexToBUffer(hex) {
    const typedArray = new Uint8Array(hex.match(/[\da-f]{2}/gi).map(function (h) {
      return parseInt(h, 16)
    }));
    return typedArray;
  }
  /**
   * 16进制取反
   */
  static hexReverse(hex) {
    let buffer = new Uint8Array(hex.match(/[\da-f]{2}/gi));
    let revList = '';
    buffer.map((item, index) => {
      buffer[index] = ~item;
      revList += buffer[index].toString(16);
    });
    return revList;
  }
  /*
  16进制转二进制字符串
  */
  static hexToTwoString(hexString) {
    let inte = parseInt(hexString, 16);
    let bytesString = (inte).toString(2);
    let length = 8 - bytesString.length;
    if (bytesString.length < 8) {
      for (let i = 0; i < length; i++) {
        bytesString = '0' + bytesString;
      }
    }
    return bytesString;
  }
}
```