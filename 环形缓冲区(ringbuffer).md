# 环形缓冲区(ringbuffer)

## 环形缓冲区概念

> 环形缓冲区是一个数据暂存区域，但与一般的线性区域不同，ringbuffer是首尾相接的环形区域，采用ringbuffer进行数据暂存可以避免线性缓冲区大小不足而溢出的问题，从而使得我们可以源源不断的向缓冲区中填充数据而不用担心缓冲区溢出或数据阻塞写入的问题

 ## 环形缓冲区的实现原理

1. **环形缓冲区基本结构**

   - **缓冲区**：用于存储数据的固定大小数组
   - **读索引值**：数据从读索引值位置开始读取
   - **写索引值**：数据从写索引值位置开始写入
   - **数据项计数**：记录当前缓冲区中存储的有效数据数量

2. **环形缓冲区循环存储原理**

   当写索引值到达缓冲区边界的时候，索引值回到缓冲区开头，继续写入，从而实现循环存储的作用，同时需要注意，写索引值追上读索引值且缓冲满的时候，后面再进行写入，需要调整读索引值的位置，跟着写索引值一起移动，从而保证数据有效范围的正确性

   **环形缓冲区的数据有效范围**：从读索引值到写索引值的区域内，是环形缓冲区数据的有效范围

3. **示例**

   初始缓冲区：_ _ _ _ _ _  读索引值 ：0  写索引值 ：0

   **写入操作**

   - 写入部分数据 1 2 3 4 _ _  写索引值：4 读索引值 ：0 有效数据：1 2 3 4
   - 继续写入数据 1 2 3 4 5 6 写索引值：0 读索引值 ：0 有效数据：1 2 3 4 5 6
   - 当缓冲区满，继续写入数据，覆盖最旧的数据 7 8 3 4 5 6 写索引值：2 读索引值：2 有效数据：7 8 3 4 5 6

   **读取操作**

   - 读取数据：3 4   写索引值：2  读索引值：4 有效数据：5 6 7 8

   ![image-20241211143539643](C:\Users\Administrator\Desktop\image-20241211143539643.png)

1. **环形缓冲区的操作**
   - 写入操作
   - 读取操作
   - 缓冲区状态判断
   - 缓冲区初始化



## 环形缓冲区与线性缓冲区的区别

### 环形缓冲区的特点：

1. 缓冲区大小固定，但**循环储存**
2. 当缓冲区满的时候，**最旧的数据**会被覆盖，新的数据会被写入，覆盖掉最早写入的部分
3. **有效数据区域**:环形缓存区的有效数据区域是从读索引值到写索引值之间

### 线性缓冲区的特点

1. 固定大小的缓冲区，数据**线性存储**
2. 当缓冲区满的时候，通常会**阻塞写入**或者**丢弃数据**
3. **有效数据区域**：从缓冲区的起始位置到当前写索引值所在的位置

## 环形缓冲区的核心目的

环形缓冲区的关键设计目标，并不是每一个数据都能够得到永久的保存，而是**持续流式数据的处理**。它的核心作用是**在读写速度不匹配，数据流较大的时候，能保证用户始终能拿到最新的数据进行处理，在固定大小的缓存区内，允许最新的数据持续写入并尽可能的避免丢失**，自动覆盖最旧的数据

### ringbuffer应用层面的意义

- **适应数据流速度的差异**，假设有一个数据源会源源不断的产生数据，但处理这个数据的消费者无法及时处理所有的数据，那么环形缓存区起到的作用就是**丢弃旧数据**，帮助消费者拿到**最新的数据**
- **实时性**：在许多实时系统中，数据的**新鲜性**更重要，在这种情况下，你可能并不关心几秒前采集的数据，而更关心当前数据的状态，**旧数据被覆盖**并不会导致系统出现问题，因为处理系统只关心**最新的数据**
- **解决内存问题**：环形缓存区的设计并不依赖于无限大的内存，如果需要处理的数据量非常大，环形缓冲区可以通过固定的内存大小来解决**内存占用过大的问题**，不需要担心缓冲区溢出
- **适合高吞吐量的场景**：在网络数据流，串口通信，音视频流传输等场景下，数据量可能远远超过消费者的处理能力，在这种情况下，环形缓冲区能有效的解决**处理速度与数据到达速度不匹配**的问题，即使数据传输的很快，环形缓冲区也能保证拿到最新的数据而不会丢失数据流



## 环形缓冲区与线性缓冲区的应用差异

### 当读写速度匹配时

当数据的读写速度处理速度跟得上数据的写入速度时，线性缓冲区与环形缓冲区没有太大的区别，此时两者都能保证数据不会溢出且最新数据可以持续写入

### 当读写速度不匹配时

当数据的处理速度跟不上数据的写入速度时，环形缓冲区的关键优势就体现出来了

此时，线性缓冲区大小固定，而无法循环存储，所以当数据快速写入但消费者无法及时处理的时候，最新的数据会被阻塞写入或直接被丢弃，只有当缓冲区内的数据被处理之后，新的数据才能继续写入，这无法保证消费者能拿到最新的数据



而环形缓存区采用循环存储的方式，能够在处理速度跟不上写入速度的时候，依然能够使得最新的数据能持续写入，而不会发生数据阻塞的情况，保证了数据的实时有效性

## 总结 - 如何选择使用哪种缓冲区

### 环形缓冲区：

适合高频数据流，确保**实时性**和**最新数据**

- 数据流不断且连续 写入速度大于读取速度
- 只关心最新的数据，可以容忍丢失旧数据
- 需要避免阻塞，即使读取速度较慢，也要保证写入能持续进行
- 适用于实时数据流处理

### 线性缓冲区：

适合低速数据流，能够容忍**丢失数据**和**阻塞写入**

- 数据量较小或者读取速度跟得上写入速度
- 需要保存所有数据，但可以容忍**阻塞写入**或**丢弃数据**

## 环形缓冲区代码实现

.c部分

```c
#include "ringbuffer.h"
#include "string.h"
/**
 * @brief ringbuffer初始化
 * @param ringbuffer_t *ringbuffer_user
 * @retval none
 */
void ringbuffer_init(ringbuffer_t *ringbuffer_user)
{
    //保证读索引值和写索引值均为0
    ringbuffer_user->ringbuffer_read = 0;
    ringbuffer_user->ringbuffer_write = 0;
    //初始化有效值为0
    ringbuffer_user->ringbuffer_ecount = 0;
    //初始化环形缓冲区数组
    memset(ringbuffer_user->buffer,0,RINGBUFFER_SIZE);
}
/**
 * @brief 环形缓冲区状态判断函数
 * @param ringbuffer_t *ringbuffer_user
 * @retval FULL  or EMPTY or HAVE
 */
uint8_t ringbuffer_status(ringbuffer_t *ringbuffer_user)
{
    if(ringbuffer_user->ringbuffer_ecount == RINGBUFFER_SIZE)
    return FULL;//缓冲区满
    else if(ringbuffer_user->ringbuffer_ecount == 0)
    return EMPTY;//缓冲区空
    else
    return HAVE;//缓冲区有数据
}
/**
 * @brief ringbuffer 写入操作
 * @param  ringbuffer_t *ringbuffer_user
 * @param uint8_t *array
 * @param uint32_t num
 * @retval  RB_RIGHT or RB_ERROR
 */
uint8_t ringbuffer_w(ringbuffer_t *ringbuffer_user,uint8_t *array,uint32_t num)
{
    //防止空指针访问
    if (ringbuffer_user == NULL || array == NULL)
    return RB_ERROR;
    //将数据写入环形缓冲区
    while(num--)
    {
        ringbuffer_user->buffer[ringbuffer_user->ringbuffer_write] = *array++;
        //更新写索引值
        ringbuffer_user->ringbuffer_write = (ringbuffer_user->ringbuffer_write+1)%RINGBUFFER_SIZE;
        //缓冲区满 边界处理，调整读索引值
       if(ringbuffer_user->ringbuffer_ecount == RINGBUFFER_SIZE) 
       {
        ringbuffer_user->ringbuffer_read = (ringbuffer_user->ringbuffer_read + 1) % RINGBUFFER_SIZE;
       } 
       else
       {
        //更新缓冲区有效值数量
         ringbuffer_user->ringbuffer_ecount++;
       }

    }
    return RB_RIGHT;
}
/**
 * @brief ringbuffer 读取操作
 * @param  ringbuffer_t *ringbuffer_user
 * @param uint8_t *array
 * @param uint32_t num
 * @retval RB_ERROR or RB_RIGHT
 */
uint8_t ringbuffer_r(ringbuffer_t *ringbuffer_user,uint8_t *array,uint32_t num)
{
    //防止非法访问
    if(num>ringbuffer_user->ringbuffer_ecount || ringbuffer_status(ringbuffer_user) == EMPTY)
    return RB_ERROR;
    //循环读取缓冲区数据
    while(num--)
    {
        *array++ = ringbuffer_user->buffer[ringbuffer_user->ringbuffer_read];
        //更新读索引值
        ringbuffer_user->ringbuffer_read = (ringbuffer_user->ringbuffer_read+1)%RINGBUFFER_SIZE;
        //更新缓冲区有效值数量
        ringbuffer_user->ringbuffer_ecount--;
    }
    return RB_RIGHT;
}

```

.h部分

```c
#ifndef __RINGBUFFER_H
#define __RIINGBUFFER_H

#include "main.h"
#define RINGBUFFER_SIZE 30 //环形缓冲区大小
/*定义环形缓冲区结构体*/
typedef struct{
    uint16_t ringbuffer_read;//读索引值
    uint16_t ringbuffer_write;//写索引值
    uint8_t buffer[RINGBUFFER_SIZE];
    uint32_t ringbuffer_ecount;//有效值数量
}ringbuffer_t;
/*定义环形缓冲区状态变量*/
enum ringbuffer_status{
    FULL = 0,
    EMPTY,
    HAVE,
    RB_ERROR,
    RB_RIGHT
};
void ringbuffer_init(ringbuffer_t *ringbuffer_user);
uint8_t ringbuffer_status(ringbuffer_t *ringbuffer_user);
uint8_t ringbuffer_w(ringbuffer_t *ringbuffer_user,uint8_t *array,uint32_t num);
uint8_t ringbuffer_r(ringbuffer_t *ringbuffer_user,uint8_t *array,uint32_t num);
#endif


```

## 环形缓冲区读取数据注意问题

###  **数据帧不完整**

- **问题描述**：如果数据帧的大小与缓冲区的大小不成倍数关系，同时读取和写入速度不匹配，可能会导致读取到的数据帧被截断，从而读到混合数据
- **常见原因**：写入和读取操作速度不匹配且缓冲区大小与数据帧大小不成倍数关系，或者数据帧没有进行边界检查。
- **解决方案**：根据数据流特性和缓冲区大小，合理安排读取数据的频率，确保完整帧的读取，或根据数据帧大小调整缓冲区大小，使其读取时无论何时都能完整读到一帧数据而不会发生两帧数据或多帧数据混合