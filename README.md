# 编程规范

C语言和Python、Arduino封库编程规范, 编写库的时候应该在满足功能的前提下尽可能采用让人易阅读易懂的编写方式

## 索引
* [变量及函数](#变量及函数)
* [宏](#宏)
* [类](#类)
* [缩进](#缩进)
* [{}](#{})
* [if-else](#if-else)
* [switch-case](#switch-case)
* [运算符](#运算符)
* [Readme中多个参数函数写法](#readme中多个参数函数写法)
* [ino文件头部写法](#ino文件头部写法)
* [封库其他注意事项](#封库其他注意事项)
* [套件类例程](#套件类例程)
* [Python](#Python)
* [C应用驱动](#C应用驱动)
* [版本](#版本)

## 变量及函数

在文件中所有有具体意义的常量应使用宏表示.<br>
所有类中的函数遵循驼峰命名法.<br>
类之外的函数名前面应当加上模块前缀.<br>
<br>

对于变量的使用限制:<br>
尽量避免大的临时变量和全局变量的创建, 在需要大型 buffer 的情况下, 需要思考这个 buffer 是否可以拆分.<br>
比如有一个统计总线上 IIC 设备个数和地址的函数, 不应在全局变量中创建一个 125 字节的 buffer 存储所有可能的地址, 而应采用如下的结构.<br>
```c++
class DFRobot_ScanIIC
{
  public:
    uint8_t scan();     // 每次调用 scan 便从 _scanMask 开始扫描, 每发现一个 IIC 设备便返回该设备的地址并将 _scanMask 赋值为该设备地址, _count 加一, 直到扫描到地址 0x7f
    uint8_t reset();    // 重置
    bool isCompelete(); // 返回 _compelete
    ...                 // 其他函数
  protected:
    uint8_t _scanMask = 0, _count = 0;
    bool _compelete = false;
};

// 在 ino 中提示用法

DFRobot_ScanIIC scan();

void setup()
{
  uint8_t currentAddr = scan.scan();
  Serial.begin(115200);
  while(scan.isCompelete() != true) {
    uint8_t addr = scan.scan();
    if(addr) {
      Serial.print("scan addr: ");
      Serial.println(addr);
    }
  }
}

```

使用 typedef 重定义的变量应加上后缀 _t .<br>
<br>
例:<br>
```c++
typedef uint16_t color_t;
```

普通变量(除结构体、共用体、枚举、重定义变量、指针外)的命名应遵循驼峰命名法, 专有名词应缩写.<br>
给模块用的变量, 变量前应加上模块的前缀和下划线.<br>
<br>
例:<br>
```c++
int lineWidth; // 普通变量
int SPI_speed; // 给 SPI 模块用的变量, 这里如果写为 SPISpeed, 与类名的命名规则重复.
```

结构体、共用体、枚举变量的创建应结合 typedef. 并在首字母前添加标识 s(struct), u(union), e(enum), 标识后的变量名首字母大写并遵循驼峰命名法.<br>
<br>
例:<br>
```c++
typedef struct {
  int var1;
} sStruct_t;

// 由于 struct, union, ecum 是独立于 class 的, 所以大多数情况下应当在定义时加上模块缩写和下划线, 下划线后的内容遵循驼峰命名法

typedef struct {
  int var1;
} sDFModule_struct_t;

typedef union {
  int i;
  char ch[4];
} uUnion_t;

// 由于枚举变量也是一个常量, 所以枚举值应当像宏定义一样基本所有字母都大写

typedef enum {
  eENUM1;
} eEnum_t;
```

指针变量在命名时应当在命名前加上 p. 多级指针应当有同数量个 p. * 号应当在变量前面<br>
<br>
例:<br>
```c++
int *pVar1, *pVar2;
int **ppVar1;

// 对于结构体、联合体指针, 应除了后缀 _t 之外保留其他字符
sStruct_t *psStruct;
```

函数指针变量应在开始加上 pf + 模块缩写 指明函数指针.<br>
<br>
例:<br>
```c++
typedef void (*pfModule_foo_t)();
```

为了防止子类在编写函数时传入的参数名和父类中的变量名冲突。<br>
类中的所有变量都应以下划线开始并在下划线后遵循驼峰命名法。<br>
<br>
例:<br>
```c++
class DFRobot_XXXX
{
  public:
    void foo(uint16_t var); // 
  protected:
    uint16_t _var; // 避免命名冲突
};

class DFRobot_child : public DFRobot_XXXX
{
  public:
    void foo1(uint16_t var);
};
```

## 宏

所有字母都要大写, 所有的带参宏必须加完全的括号. (类似于函数用法的宏例外并不用所有字母大写)<br>
<br>
例:<br>
```c++
#define AA          1000

#define ADD(x, y)   ((x) + (y))

#define swap_int(a, b)  { int c = b; a = b; b = c; }
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

## 封库其他注意事项

封库应当尽量避免使用定时器和注意外设和其他库联合使用时的冲突, 在条件允许的情况下应做外设状态检查, 在外设使用完成后保持外设状态不变. 如果一定要使用, 必须在相关文件里说明.<br>
<br>
以下建议作为用作减少阅读理解难度.<br>
在封库的类中, 需要将类的主要逻辑放入类中, 它的通信方式使用另一个子类(这个子类如果代码量不多, 可以和父类放在用一个文件中)来实现.<br>
<br>
例:<br>
```c++
class DFRobot_BME680 {
  protected:
    virtual void writeReg(uint8_t reg, uint8_t data) = 0;  // 这里使用虚函数, 让子类来实现具体寄存器操作
}

class DFRobot_BME680_SPI : public DFRobot_BME680 {
  protected:
    void writeReg(uint8_t reg, uint8_t data);  // 重写父类的虚函数, 实现寄存器操作
}
```

如果在MCU和目标模块大小端模式一样, 在传输数据时, 如果能明确知道传输数据的意义, 直接使用相匹配的数据类型来读取数据, 不使用以字节为单位的 buffer.<br>
<br>
例:想要读目标模块中一段16位的数据<br>
```c++
void readBytes(uint8_t addr, uint8_t reg, uint8_t *pBuf, uint8_t len);

uint16_t readObj16()
{
  uint16_t objName = 0;
  readBytes(addr, reg, (uint8_t*) &objName, sizeof(objName));
  objName >>= objOffset;  // 可能需要的数据移位
  return objName;
}
```
例:想要读取目标模块中一段相邻的8位、16位、24位、16位数据.<br>
```c++
typedef struct {
  uint8_t     v1;
  uint16_t    v2;
  uint8_t     v3[3];
  uint16_t    v4;
} sModule_t;

void readBytes(uint8_t addr, uint8_t reg, uint8_t *pBuf, uint8_t len);

sModule_t readObj()  // 在使用返回的结构体进行逻辑处理时需要注意到 v3 是一个 buffer, 暂时未想到其他方式规避
{
  sModule_t sModule;
  readBytes(addr, reg, (uint8_t*) &sModule, sizeof(sModule));
  return sModule;
}
```

## 套件类例程

例如浇花套件.<br>
为了方便迭代, 明确 ino 文件的作用是只做常见的配置和各个模块之间的连接, 不做主要工作流程, 主要工作流程应当在独立其他cpp文件中执行, 由 ino 文件调用.

## Python

Python 的库应将 demo 和库分开.<br>
类名需要与遵循与 Arduino 一致的原则, 类中的变量如果不会暴露给用户, 则需要在变量名前加下划线, 同样遵循驼峰命名法.<br>
函数名如果不暴露给用户, 则也需要在函数名前加下划线.

## C应用驱动

为了方便迭代, 明确 main.c 文件的作用是只做常见的配置和各个模块之间的连接, 不做主要工作流程, 主要工作流程应当独立在其他c文件中执行, 由main文件调用.<br>
(以下内容为推荐写法, 不强制)<br>
所有能够进行 IAP 操作的 MCU (例如OLED的5.5寸屏stm32更新驱动)在设计迭代产品更新固件的方法时都应使用 IAP 的方式. (或者其他类似于一键载入, 选择串口后下载的方式)<br>
<br>
所有可能需要对象的模块需要使用结构体来抽象对象.<br>
<br>
例:<br>
```c++
typedef struct {
  int var1;
} sModule_t;

void Module_init(sModule_t *psModule, ...); // 传入对象的变量和其他初始化参数
```
所有模块应将能抽象的部分(比如SPI通信, IO操作等)进行函数指针抽象. 模块的主要 C 文件只包含逻辑控制.<br>
<br>
例:<br>
```c++
typedef void (*pfTransfer_t)(uint8_t addr, uint8_t reg, uint8_t *pBuf, uint8_t len);

typedef struct {
  pfTransfer_t pfRead;
  pfTransfer_t pfWrite;
} sModule_t;

void Module_init(sModule_t *psModule);
```

## 版本

-version v0.0.1<br>
-date 2018-12-6<br>
-author xiaowo<br>
-remark first version<br>
