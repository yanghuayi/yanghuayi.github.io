## js-16进制数据取反

> 最近在做小程序蓝牙控制的时候，连接蓝牙鉴权需要进行16进制取反，再次记录一下

#### 1. 先进行数据拆分
```
  const hex = 0x1d23f231c84ea31
  let buffer = new Uint8Array(hex.match(/[\da-f]{2}/gi));
```
  上面`hex`是16进制数据的变量，先进行正则匹配成uint8的数组格式

#### 2. 进行循环取反
```
  let revList = '';
  buffer.map((item, index) => {
    buffer[index] = ~item;
    revList += buffer[index].toString(16);
  });
```
  循环buffer数据，对每个值进行取反，`~`符号在js里面代表取反，进行取反后，再用`toString(16)`转换为16进制的字符串拼接到变量`revList`变量，最终循环走完，`revList`的值就是`hex`16进制数据取反后的结果