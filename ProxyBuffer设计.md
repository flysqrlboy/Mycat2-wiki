# 简介
mycat2.0 设计前后端读写共享同一个buffer。该buffer是可重用的,连续读或者写,当空间不够时compact擦除之前用过的空间。
当前buffer设计是基于单工模式的，即：要么读，要么写.不会存在同时读写的情况。只有数据被操作完成（读完或者写完）后state才能被改变。

## 一、bufer 操作基本场景
对buffer的操作分为以下三个场景
1. 从 channel 向 buffer 写入数据。
2. 从 buffer  中读取数据 进行逻辑处理。    以mycat 为例，会从buffer中读取出mysql报文数据,进行逻辑处理。
3. channel    从 buffer 中读取数据。<br>

注： 以上三个场景区分前端和后端。即：当前buffer 是前端操作还是后端操作。

为满足以上三个场景,设计了ProxyBuffer. ProxyBuffer 共有三个指针,两个状态。
### 三个指针
|#|字段|默认值|说明|
|---|----|-----|------
|1|writeIndex|0|从channel 向 buffer  写入数据的开始位置。
|2|readMark|0|从buffer  向 channel 写入数据的开始位置。
|3|readIndex|0|从buffer  想 channel 写入数据的结束位置。

### 两个状态
|#|字段|默认值|说明|
|---|----|------|------
|1|inReading|false|当前buffer 的读写状态。有两个值 false 代表写入状态;true 代表读取状态。
|2|frontUsing|false|当前buffer 是谁在使用。false 代表后端在使用;true 代表前端在使用。

#### 第一个场景 从 channel 向 proxybuffer 写入数据。
    1. proxybuffer 读写状态。
       从channel 读取数据写入到 ProxyBuffer时, channel 为可读状态,
       proxyBuffer 处于写入状态,即 inReading= false;
    2. 写入的开始结束位置。
       始终是从 writeIndex 开始向proxyBuffer 中写入数据。
       写入结束位置始终是buffer 的capacity。
    3. writeIndex 指针的移动。
       本次写入多少数据writeIndex 就移动相应的长度。
    4. proxybuffer 压缩。
       每次开始向proxybuffer 写入数据前，
       判断当前proxybuffer 容量是否大于总容量的1/3（writeIndex > buffer.capacity() * 1 / 3).
       如果大于 1/3 进行一次 compact。 

#### 第二个场景 从 buffer  中读取数据 进行逻辑处理。
    1. proxybuffer 读写状态。
       从 channel 中读取数据到proxybuffer后，proxybuffer 进入可读状态，即 inReading = true;
    2. 读取数据的开始结束位置。
       数据始终从 readIndex 开始读取数据。
       读取结束位置为 writeIndex。
       即程序从buffer 中读取数据的范围是 readIndex -- writeIndex 之间的数据。
    3. readIndex 指针的移动
       每读取一次数据，readIndex就增加相应的长度。
       当 readIndex == writeIndex 时,代表本次写入到proxybuffer中的数据，全部读取完成。

#### 第三个场景 channel 从 buffer 中读取数据。
    1. proxybuffer 读写状态。
       向 channel 写入数据时,当前proxybuffer 需要确保进入可读状态，即 inReading = true;
    2. 写入到channel中数据的开始结束位置。
       channel 始终从 readMark 开始 读取数据，到 readIndex 结束。
       即：写入到 channel中的数据范围是 readMark---readIndex 之间的数据。
    3. readMark 指针的移动
       将数据写出到channel中后,readMark 对应写出了多少数据。即： writed = channel.write(buffer);
       每次写出数据后，readMark 增加写出数据的长度。即： readMark += writed ;
       readMark默认值为0. 有可能存在 要写出的数据 writed 没有写出去,或者只写出去了一部分的情况。
       下次channel 可写时（通常可写事件被触发），接着从readMark 开始写出数据到channel中。
       当readMark==readIndex 时,代表 数据全部写完。
    4. 读写状态转换
       数据全部写完后,proxybuffer 状态 转换为 可写状态。即  inReading = false;
    5. proxybuffer 压缩。
       每次从proxybuffer读取数据写入到channel前，
       判断当前proxybuffer 已读是否大于总容量的2/3（readIndex > buffer.capacity() * 2 / 3).
       如果大于 2/3 进行一次 compact。 
#### 总结
|#|场景|状态|数据范围|变化指针|
|--|---|----|-------|-----
|1|从channel 向 buffer 写入数据|写入状态|writeIndex---capacity|writeIndex 增加
|2|从buffer 读取数据 进行逻辑处理|可读状态|readIndex---writeIndex|readIndex  增加
|3|channel 从buffer 读取数据时|可读状态|readMark --- readIndex|readMark  增加

## 二、mycat 使用场景
### 2.1 透传 场景
### 2.2 只前端读写、只后端读写场景
### 2.3 没有读取数据，向buffer中写入数据后 直接 write 到 channel的场景