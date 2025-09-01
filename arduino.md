# Arduino基础

以arduino uno为例子记录如何在Linux系统上准备开发环境和点亮第一个led邓。

## 准备开发环境

#### 方式一：Arduino IDE

最简单的方式就是直接安装arduino ide软件。

```shell
sudo apt install arduino
```

#### 方式二：AVR tools

使用avr tools命令行工具也可以完成开发过程。

```shell
sudo apt install gcc-avr avr-libc avrdude

# 使用以下命令确认工具安装正常
avr-gcc --version
avrdude -v
```



arduino ide简单直观，接下来所有内容都是“方式二”相关的操作。

## 准备模拟器

使用模拟器可以更快速和安全的测试代码行为是否正常。simulide软件提供了arduino模拟器和常用器件。

```shell
sudo apt install simulide
```

## 点亮第一个LED灯

#### 点亮arduino电路板上自带的led灯：

```c
#include <avr/io.h>
#include <util/delay.h>

int main()
{
    // Set built-in LED pin (13) as output
    DDRB |= (1 << DDB5);
    while (1) {
        PORTB |=  (1 << PB5);   // LED on
        _delay_ms(500);
        PORTB &= ~(1 << PB5);   // LED off
        _delay_ms(500);
    }
    return 0;
}
```

#### 编译源代码

```shell
avr-gcc blink.c -o blink.elf -mmcu=atmega328 -DF_CPU=16000000UL -Os
avr-objcopy blink.elf -O ihex blink.hex
```

#### 使用simulide进行测试

运行simulide，添加arduino uno模拟器。为模拟器加载hex文件。点击开始模拟按钮，电路板图标上的led灯开始闪烁。

除此之外，还可以添加电阻、led灯、groud。将这些零件串联到arduino的13号引脚上，此时，外接的led灯也会同步闪烁。

#### 上传程序到硬件中

准备号硬件，上传程序。

```shell
avrdude -c arduino -p m328p -U flash:w:"blink.hex":a -P /dev/ttyACM0
```

