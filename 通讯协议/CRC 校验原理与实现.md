# CRC算法简介
循环冗余校验(Cyclic Redundancy Check, CRC)是一种根据数据产生**简短固定位数的校验码**的一种信道编码技术，主要用于检测或校验数据传输或保存后可能出现的错误。利用除法及余数的原理实现错误侦测。

CRC校验计算速度快，检错能力强，易于用编码器等硬件电路实现。

# CRC参数模型
计算一个正确的CRC值，需要知道CRC的参数模型。

一个完整的CRC参数模型应该包含以下信息：
| 参数名称 | 含义                                                                                                     | 备注     |
| -------- | -------------------------------------------------------------------------------------------------------- | -------- |
| NAME     | 参数模型名称                                                                                             |          |
| WIDTH    | 宽度，即生成CRC数据位宽，如CRC-8，则生成的CRC为8位                                                       | 必须参数 |
| POLY     | 十六进制多项式，省略最高位1，如x8+x2+x+1,二进制为1 0000 0111，省略最高位1，转换为十六进制为0x07          | 必须参数 |
| INIT     | CRC初始值，和WIDTH位宽一致                                                                               | 必须参数 |
| REFIN    | true 或 false, 在进行计算之前，原始数据是否进行翻转，如原始数据为0x34(0011 0100),反转后为0x2C(0010 1100) | 必须参数 |
| REFOUT   | true 或 false, 运算完成之后，得到的CRC值是否进行翻转，翻转方式与REFIN一致                                | 必须参数 |
| XOROUT   | 计算结果与此参数进行异或运算后得到最终的CRC值，与WIDTH位宽一致                                           | 必须参数 | 

通常如果只给了一个多项式，没有其他说明，则 INIT = 0x00, REFIN = false, REFOUT = false, XOROUT = 0x00。

常用的 21 个标准CRC参数模型

## 常用的CRC参数模型

| CRC算法名称        | 多项式公式                                                                       | WIDTH | POLY     | INIT     | XOROUT   | REFIN | REFOUT |
| ------------------ | -------------------------------------------------------------------------------- | ----- | -------- | -------- | -------- | ----- | ------ |
| CRC-4/ITU          | x4 +  x + 1                                                                      | 4     | 03       | 00       | 00       | TRUE  | TRUE   |
| CRC-5/EPC          | x5 +  x3 + 1                                                                     | 5     | 09       | 09       | 00       | FALSE | FALSE  |
| CRC-5/ITU          | x5 +  x4 + x2 + 1                                                                | 5     | 15       | 00       | 00       | TRUE  | TRUE   |
| CRC-5/USB          | x5 +  x2 + 1                                                                     | 5     | 05       | 1F       | 1F       | TRUE  | TRUE   |
| CRC-6/ITU          | x6 +  x + 1                                                                      | 6     | 03       | 00       | 00       | TRUE  | TRUE   |
| CRC-7/MMC          | x7 +  x3 + 1                                                                     | 7     | 09       | 00       | 00       | FALSE | FALSE  |
| CRC-8              | x8 +  x2 + x + 1                                                                 | 8     | 07       | 00       | 00       | FALSE | FALSE  |
| CRC-8/ITU          | x8 +  x2 + x + 1                                                                 | 8     | 07       | 00       | 55       | FALSE | FALSE  |
| CRC-8/ROHC         | x8 +  x2 + x + 1                                                                 | 8     | 07       | FF       | 00       | TRUE  | TRUE   |
| CRC-8/MAXIM        | x8 +  x5 + x4 + 1                                                                | 8     | 31       | 00       | 00       | TRUE  | TRUE   |
| CRC-16/IBM         | x16 +  x15 + x2 + 1                                                              | 16    | 8005     | 0000     | 0000     | TRUE  | TRUE   |
| CRC-16/MAXIM       | x16 +  x15 + x2 + 1                                                              | 16    | 8005     | 0000     | FFFF     | TRUE  | TRUE   |
| CRC-16/USB         | x16 +  x15 + x2 + 1                                                              | 16    | 8005     | FFFF     | FFFF     | TRUE  | TRUE   |
| CRC-16/MODBUS      | x16 +  x15 + x2 + 1                                                              | 16    | 8005     | FFFF     | 0000     | TRUE  | TRUE   |
| CRC-16/CCITT       | x16 +  x12 + x5 + 1                                                              | 16    | 1021     | 0000     | 0000     | TRUE  | TRUE   |
| CRC-16/CCITT-FALSE | x16 +  x12 + x5 + 1                                                              | 16    | 1021     | FFFF     | 0000     | FALSE | FALSE  |
| CRC-16/X25         | x16 +  x12 + x5 + 1                                                              | 16    | 1021     | FFFF     | FFFF     | TRUE  | TRUE   |
| CRC-16/XMODEM      | x16 +  x12 + x5 + 1                                                              | 16    | 1021     | 0000     | 0000     | FALSE | FALSE  |
| CRC-16/DNP         | x16 +  x13 + x12 + x11 + x10 + x8 + x6 + x5 +  x2 + 1                            | 16    | 3D65     | 0000     | FFFF     | TRUE  | TRUE   |
| CRC-32             | x32 +  x26 + x23 + x22 + x16 + x12 + x11 + x10 +  x8 + x7 + x5 + x4 + x2 + x + 1 | 32    | 04C11DB7 | FFFFFFFF | FFFFFFFF | TRUE  | TRUE   |
| CRC-32/MPEG-2      | x32 +  x26 + x23 + x22 + x16 + x12 + x11 + x10 +  x8 + x7 + x5 + x4 + x2 + x + 1 | 32    | 04C11DB7 | FFFFFFFF | 0        | FALSE | FALSE  |

CRC校验在电子通信领域非常常用，可以说有通信存在的地方就有CRC校验。

# CRC计算
手算一个CRC值，进一步了解CRC计算的原理。

**原始数据为0x34, 使用CRC-8/MAXIM 参数模型，求出CRC值。**
先列出 CRC-8/MAXIM 的参数
```
POLY = 0x31 = 0b00110001
INIT = 0x00
XROUT = 0x00
REFIN = TRUE
REFOUT = TRUE
```
计算步骤如下
1. INIT = 0x00，原始数据高8位和初始值进行异或运算；`0x34^0x00 = 0x34`
2. REFIN 为 TRUE，需要对数据进行反转；`0x34(0011 0100) 反转后得到 0x2C(0010 1100)`
3. 数据左移8位；`0x2C(0010 1100) << 8  = 0x2C00(0010 1100 0000 0000)`
4. 将数据和多项式进行模2除法，求得余数，取余数低八位；
	`0x2C00 除以 0x131(补齐多项式的第八位)`
	`得到 0 1111 1011(0xFB) `
5. XROUT = 0x00, 将上一步求得的余数与XROUT进行异或运算；`0xFB^0x00 = 0xFB`
6. REFOUT 为 TRUE，对结果进行反转得到最终的 CRC-8 值；`0xFB(1111 1011)反转后得到 0xDF(1101 1111)`

**模2除法求余数：**
1111 0000 除以 1101
![](assert/Pasted%20image%2020230426210016.png)
例子中的 0x2C00 / 0x131
![](assert/Pasted%20image%2020230426210845.png)



# 代码实现
```
/******************************************************************************
 * Name:    CRC-8/MAXIM         x8+x5+x4+1
 * Poly:    0x31
 * Init:    0x00
 * Refin:   True
 * Refout:  True
 * Xorout:  0x00
 * Alias:   DOW-CRC,CRC-8/IBUTTON
 * Use:     Maxim(Dallas)'s some devices,e.g. DS18B20
 *****************************************************************************/
uint8_t crc8_maxim(uint8_t *data, uint16_t length)
{
    uint8_t i;
    uint8_t crc = 0;         // Initial value
    while(length--)
    {
        crc ^= *data++;        // crc ^= *data; data++;
        for (i = 0; i < 8; i++)
        {
            if (crc & 1)
                crc = (crc >> 1) ^ 0x8C;        // 0x8C = reverse 0x31
            else
                crc >>= 1;
        }
    }
    return crc;
}
```
1. 与Init进行异或运算`crc ^= *data++;`
2. 



# 参考文章
https://zhuanlan.zhihu.com/p/256487370
https://baike.baidu.com/item/%E6%A8%A12%E9%99%A4%E6%B3%95/10416971?fr=aladdin
https://github.com/whik/crc-lib-c