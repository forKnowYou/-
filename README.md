# 编程规范

C语言 Arduino封库编程规范

## 索引
* [简介](#简介)
* [变量及函数命名](#变量及函数命名)
* [宏](#宏)
* [结构体](#结构体)
* [共用体](#共用体)
* [枚举](#枚举)
* [指针](#指针)
* [类](#类)
* [缩进](#缩进)
* [{}](#{})
* [if-else](#if-else)
* [switch-case](#switch-case)
* [运算符](#运算符)
* [Readme中多个参数函数写法](#readme中多个参数函数写法)
* [ino文件头部写法](#ino文件头部写法)

## 变量及函数命名

*首字母小写, 下个单词首字母大写 <br>
*专有名词要大写

正确的写法

```C++
int sensorValue;
int SDCard,IIS,I2C;
```

错误的写法

```C++
int sensor_value;
float sensorvalue;
byte sensor_Value;
```

为了各个库之间的互相兼容，必须做到以下几点。<br>

  1. 库的 cpp 文件中除个别情况外不能有全局变量，所有相关变量应定义在 class 中
  2. 库相关的变量定义要么定义在 class 中，要么加上库名作为前缀
  3. 所有有特殊需求的外设通信（比如说要把IIC通信速率提高到800KHZ），在每次开始通信前必须进行相关配置，通信完成后必须把更改了的配置恢复到默认状态

例: <br>

.h 文件:

```cpp

#define MODULE_OK   0
#define MODULE_ERR  -1

typedef enum {
  eModuleStatusOK,
  eModuleStatusErr = -1
} eModuleStatus_t;

class DFRobot_Module {

  typedef enum {
    eOK,
    eERR
  } eStatus_t;

  eStatus_t writeReg(uint8_t reg, uint8_t *pBuf, uint16_t len);

}
```

.cpp 文件:

```cpp

DFRobot_Module::eStatus_t DFRobot_Module::writeReg(uint8_t reg, uint8_t *pBuf, uint16_t len)
{
  SPI.begin(SPISettings(32000000, MSBFIRST, SPI_MODE0));  // 特殊设定
  // do something
  SPI.begin(SPISettings(14000000, MSBFIRST, SPI_MODE0));  // 恢复默认设定
}

```

## 宏

作为简化记忆功能的宏所有字母都要大写
正确的写法
```C++
#define MAX 1000
#define SENSOR_REG1 1
```

错误的写法：
```C++
#define Max 1000
#define sensorReg1 1
```
带参数的宏,参数要加括号
```C++
#define ADD(x, y)   ((x) + (y))
```
类似函数用法的宏，采用函数的编程规范
```C++
#define swapInt(a, b)  { int c = b; a = b; b = c; }
```
## 结构体

使用typedef，前面要加s，后面首字母大写

```C++
typedef struct
{
  int age;
  char name[20];
}sTeacher_t;
```

## 共用体

使用typedef，前面要加u，后面首字母大写

```C++
typedef union {
  uint32_t value;
  uint8_t b[4];
} uValue_t;
```

## 枚举

前面要加e，后面首字母大写
```C++
typedef enum{
  eSunday=0,
  eMonday,
  eTuesday,
  eWednesday,
  eThursday,
  eFriday,
  eSaturday,
}eDay_t;
```
## 指针

指针前面要加 * 显式说明这是一个指针

```C++
int *pVar1, *pVar2;
int **ppVar3;
sTeacher_t *psTeacher,*psT1,*psT2;
uValue_t *puValue,*puV1,*puV2;
eDay_t *peDay, *peD, *peD1,*peD2;
```

指向函数的指针，使用pf做前缀

```C++
typedef void (*pfTeacherWork_t)();
```

## 类

以DFRobot_开头,后面接ClassName,ClassName首字母要大些
正确的写法
```C++
class DFRobot_SIM7000;
class DFRobot_BME280;
class DFRobot_Lora;
```
错误的写法
```C++
class DFRobot_sim7000;
class Lora;
class DFRobotLora;
```
成员变量要加 _ 前缀

```C++
class DFRobot_BME280
{
public:
  DFRobot_BME280(int addr)
    :_addr(addr)
    {}
private:
  int _addr;
}
```

## 缩进

全部使用空格缩进，不要使用tab缩进（除了特殊用途，比如keywords makefile里），请用notepad++检查

## {}
在函数中，{}顶格写，例如

```C++
void func()
{
  //do something
}
```

## if-else

{写在语句后面，}写在新行,这样很方便调试。例如
```C++
if(conditon1){
  //do something
}else if(conditon2){
  //do something
}else{
  //do something
}
```

## switch-case

case与switch对其，{跟在switch语句后面，例如
```C++
switch(value){
case x:
  //do something
case y:
  //do something
default:
  //do something
}
```

## 运算符

前后需要留空格(此条根据具体情况，不强制)

```C++
for(int x = 0; x <= 10; x++)
```

## Readme中多个参数函数写法
采用doxygen注释规则描述参数

```C++
    /*!
     *  @brief Set operational mode to VL53L0X
     *
     *  @param mode  Work mode settings
     *      Single : Single mode
     *      Continuous : Back-to-back mode
     *  @param precision  Set measurement precision
     *      High：High precision(0.25mm)
     *      Low: Low precision(1mm)
     *  @return  true if execute successfully, false otherwise.
     */
    bool setMode(uint8_t mode, uint8_t precision);
```

## ino文件头部写法
```C++
 /*!
  * file 文件名（不能使用中文）
  *
  * 如何做这个实验，描述实验步骤（只需要下载程序就能肉眼观测到的简单小实验例如blink，这步可以不写）（不能使用中文）
  * @n 实验现象是什么（不能使用中文）
  *
  * Copyright   [DFRobot](http://www.dfrobot.com), 2016
  * Copyright   GNU Lesser General Public License
  *
  * version  V1.0
  * date  2017-10-9
  */
  ```
