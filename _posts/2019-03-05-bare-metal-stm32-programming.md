---
title: Bare Metal STM32 Programming and a Quadcopters Awakening
tags: programming
---

Last year I got the [Crazepony Mini](http://www.crazepony.com/products/mini.html) quadcopter, and just recently I figured out how to program it. I will show my progress in this post, and it will also serve as a getting started guide for programming STM32 microcontrollers. We will build a minimal working example to blink an LED with only the GNU ARM compiler (gcc) and without any library dependencies.

{% picture bare-metal-stm32-programming/crazepony-mini.jpg --alt Crazepony Mini quadcopter %}

If you are just here for the STM32 programming you can [skip this part](#the-bootloader).

## The Crazepony Mini

You can get the quadcopter on eBay for around 100 €. It ships with a remote control that wirelessly connects to the drone. Like the drone the remote control has no casing, which I find for the drone looks really good, but unfortunately makes holding the remote control difficult. Both devices have firmware by Crazepony installed for which they published the source on [GitHub](https://github.com/Crazepony). They seem to be using the Keil IDE. Although it's mainly made for educational purpose you can totally fly the quadcopter, and I had fun with it for a while.

{% picture bare-metal-stm32-programming/remote-control.jpg --alt Quadcopter remote control %}

Mine came with two extra motors for replacement, and you can get super cheap replacement propellers from [AliExpress](https://www.aliexpress.com/item/5Pairs-75mm-Professional-Plastic-Propeller-Screw-For-DIY-Model-Aircraft-Helicopter-Free-Shipping-Russia/32424000447.html). The flight time is supposed to be up to 6 minutes, so I got some [extra batteries and a charger](https://www.aliexpress.com/item/3-7v-650mah-802540-battery-USB-cable-charger-for-drone-X5C-X5-X5SC-X5SW-X5C-1/32859036671.html) so that I can quickly switch batteries when power runs out.

{% picture bare-metal-stm32-programming/batteries-and-charger.jpg --alt Batteries and charger %}

So let's get into the technical details of the drone. The chip on the quadcopter is a 32-bit ARM Cortex-M processor with 64 KiB flash memory. On board are a wireless module, a 3-axis digital compass which detects orientation using the earth's magnetic field, an altimeter that measures height by air pressure and an accelerometer and gyro sensor combined into one chip. In this post we will be using the LEDs on one of the 4 arms and most importantly the integrated USB to serial bridge to flash our program. You can get the schematic for the quadcopter [here](https://github.com/Crazepony/crazepony.github.io/blob/master/assets/schemtics/crazepony-fc-shematics.pdf).

## The bootloader

There are a few ways to flash firmware onto STM32 microcontrollers. You can use one of the debugging interfaces JTAG or Serial Wire Debug (SWD) which also have support for on-chip debugging. Note that SWD despite it's name does not use the standard serial port of the chip. In fact, using the serial connection also known as UART connection is another and the most basic way to flash the controller. That's what we will be using.

If you want to follow along you should get the reference manual, which is meant for application developers, as well as the datasheet for your processor.  For the processor on my drone the [STM32F103 reference manual](https://www.st.com/content/ccc/resource/technical/document/reference_manual/59/b9/ba/7f/11/af/43/d5/CD00171190.pdf/files/CD00171190.pdf/jcr:content/translations/en.CD00171190.pdf) and the [STM32F103x8 datasheet](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf) apply. You can find them by searching for your processor name together with "reference manual" or "datasheet".

{% picture bare-metal-stm32-programming/boot-table.png mobile: bare-metal-stm32-programming/boot-table-mobile.png --alt Boot modes %}

The STM32 processors have three boot modes as shown in this table from the reference manual. Booting from flash is the default mode and what we will be using to run our program once we uploaded it to flash memory. To flash the microcontroller over UART we will have to boot the processor in system memory mode. System memory refers to a part of ROM on the chip which contains a bootloader since the manufacturing. That means there's no way to brick the controller. We will be speaking to that bootloader over UART and tell it to write our firmware to flash.

## USB to serial converter

If your computer doesn't have a serial port which is likely nowadays you need a USB to serial bridge to connect the board to your PC. Fortunately most STM32 boards like the popular Blue Pill board have a USB to UART bridge with the CP2102 chip on board. Below you can see the bridge in the schematic of my drone. If your board has an USB port it most likely connects to a bridge as well. Otherwise you can get a dongle with the CP2102 chip for around 8 € on [Amazon](https://www.amazon.com/WINGONEER-CP2102-Module-Serial-Converter/dp/B01LRVQIFQ). The chip connects to the 5 volts and ground pins to power the microcontroller.  And the `RXD` and `TXD` serial lines connect to pins `A9` and `A10` (`TXD_BT` and `RXD_BT` in the schematic) on the processor respectively.

{% picture bare-metal-stm32-programming/cp2102-schematic.png mobile: bare-metal-stm32-programming/cp2102-schematic-mobile.png --alt Drone schematic with CP2102 chip %}

From the pin table from the processors datasheet:

{% picture bare-metal-stm32-programming/serial-pins.png mobile: bare-metal-stm32-programming/serial-pins-mobile.png --alt Serial pins on processor %}

Don't get confused with the RX reciever and TX transmitter pins. The pins on the processor are named from the processors perspective and the pins on the bridge are named from the computers perspetive.

## Flashing the processor

Before we can start writing to flash we need to boot the processor in system mode. As you can tell from the boot modes table we need to set the `B00T0` pin to high. On my microcontroller the `BOOT0` pin is actually connected to the CP2102 chip as shown in the schematic so this happens automatically when I upload firmware. On some boards like the Blue Pill you can set the `BOOT0` and `BOOT1` pins using those yellow jumpers.

{% picture bare-metal-stm32-programming/blue-pill.jpg --alt Blue Pill board %}

Once the `BOOT0` pin is set correctly you can connect the board to your computer to power it. The microcontroller boots and a serial device should appear at `/dev`.

{% highlight bash %}
$ ls /dev | grep USB
ttyUSB0
{% endhighlight %}

We will use the [stm32loader](https://github.com/jsnyder/stm32loader) Python script to upload our program. It requires the [pySerial](https://pythonhosted.org/pyserial/) library. A normal user can't write to `/dev/ttyUSB0` so you either need to run it as root or add your user to the `dialout` or `uucp` group depending on your distro which gives a user access to the serial ports. You might need to login again afterwards. If you are having issues with the script on Arch Linux consider trying [my patch](https://github.com/timakro/stm32-quadcopter/commit/693d90b8d5c5718913f91700d757576b4af217ee).

{% highlight bash %}
sudo adduser tim dialout
{% endhighlight %}

The `stm32loader.py` script can download the current firmware so you might want to do that before flashing your own. To upload your own firmware you would pass it the path to the device and the binary file to write. `-e` erases the previous binary `-w` means write and `-v` will download after uploading and verify that the upload worked.

{% highlight bash %}
./stm32loader.py -p /dev/ttyUSB0 -ewv firmware.bin
{% endhighlight %}

## Startup assembly code

For now we will start with a C program that does nothing. We call it `main.c` and it will just stay in an endless while loop.

{% highlight c %}
void main ()
{
    while (1);
}
{% endhighlight %}

Although this alone can't run on the microcontroller yet. The processor starts executing at a very specific entry point in flash memory. We need finer memory control to set this up correctly and therefore we will use assembly. From the reference manual:

> After this startup delay has elapsed, the CPU fetches the top-of-stack value from address `0x0000 0000`, then starts code execution from the boot memory starting from `0x0000 0004`.

The assembly code will go in a file called `startup.s`.

{% highlight nasm %}
.cpu cortex-m3
.thumb

// Stack top address (end of 20K RAM)
.word 0x20005000
.word _reset

.thumb_func
_reset:
    bl main
    b .
{% endhighlight %}

The first two lines configure the assembler for a Cortex-M3 processor with the Thumb instruction set. On ARM a word is 4 bytes. The first `.word` line is writing the value `0x20005000` to the beginning of the output binary. When we flash the binary this will become position `0x0000 0000` in flash memory. Note the quote above, that's the position where the processor reads the stack's top address. As you might know the program stack lives at the very end of memory and grows backwards, the end of memory on my processor is `0x20005000`. You might want to adjust this for your processor, I will get into memory layout later.

The next word we write is the address of the `_reset` function which is defined below. So this address ends up at position `0x0000 0004`, you probably see the pattern now. That is where the processor starts code execution.

The `.thumb_func` directive is required for functions to run on a processor with the Thumb instruction set like the STM32. For our C code this will be implied for all functions by passing a compiler flag. The `_reset` function calls the `main` function defined in the C program and afterwards enters an endless loop.

You will often encounter endless loops in microprocessor programming. They are basically the microprocessor equivalent to `exit(0)`. When you want the program to end or there is some failure you enter an endless loop to stop the control flow. In this state a reset is required to restart the program.

## Linker script

The last file required is a linker script which specifies the processor's memory layout to the linker. You can see the relevant section of the memory map from the datasheet below.

{% picture bare-metal-stm32-programming/memory-map.png --alt Memory map %}

As you can see the flash memory starts at `0x0800 0000` and the SRAM starts at `0x2000 0000`. I could also take from the datasheet that my processor has 64 KiB of flash and 20 KiB of SRAM. This information goes into the linker script called `linker.ld`.

{% highlight none %}
MEMORY
{
  FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 64K
  RAM (xrw)  : ORIGIN = 0x20000000, LENGTH = 20K
}
{% endhighlight %}

Now that we have a C program, the startup assembly code and the linker script ready, we can start compiling the final binary.

## Compiling for the STM32

To compile for ARM processors you can get the GNU ARM toolchain. It provides a C compiler `arm-none-eabi-gcc`, a linker `arm-none-eabi-ld`, etc. to cross-compile for ARM. On Debian the package is called `gcc-arm-none-eabi` and on Arch Linux you need the packages `arm-none-eabi-gcc` and `arm-none-eabi-newlib`.

First we compile the assembly and C code to object files. The `-mcpu=cortex-m3` and `-mthumb` flags are required for the C compiler to compile for STM32 processors.

{% highlight bash %}
arm-none-eabi-as -o startup.o startup.s
arm-none-eabi-gcc -mcpu=cortex-m3 -mthumb -c -o main.o main.c
{% endhighlight %}

When we link both object files we pass the linker script with the `-T` option.

{% highlight bash %}
arm-none-eabi-ld -T linker.ld -o main.elf startup.o main.o
{% endhighlight %}

The ELF format is designed for binaries that run on an operating system. We need to extract the actual raw binary executable, starting with the eight bytes we set in the assembly code. The `objcopy` utility can do that.

{% highlight bash %}
arm-none-eabi-objcopy -O binary main.elf main.bin
{% endhighlight %}

If everything compiled without errors you can now flash your first firmware and hope for the best.

{% highlight bash %}
./stm32loader.py -p /dev/ttyUSB0 -ewv main.bin
{% endhighlight %}

If everything went right *nothing* should happen, hurray!

## Controlling output pins

On my drone the two LEDs on the top and bottom of one of the arms are connected to pin `PA11` of the processor.

{% picture bare-metal-stm32-programming/led-connection.png mobile: bare-metal-stm32-programming/led-connection-mobile.png --alt LED connection to processor pin %}

To turn them on the pin has to be configured as an output pin first. Configuring or toggling any of the `PA` pins is done by writing to the `GPIOA` registers. `PB` pins would be controlled through the `GPIOB` registers and so on. In the system architecture figure from the reference manual below, you can see that the `GPIOA` registers are part of the `APB2` domain.

{% picture bare-metal-stm32-programming/system-architecture.png --alt System architecture with GPIOA register %}

### Clock enable register

To write to the `GPIOA` registers the corresponding clock has to be turned on first. As documented in the reference manual the *`APB2` peripheral clock enable register* is accessible at address `0x4002 1018`. It is 32 bits long and bit 2 controls the `GPIOA` clock.

> Bit 2 **IOPAEN:** IO port A clock enable  
> Set and cleared by software.  
> 0: IO port A clock disabled  
> 1: IO port A clock enabled

So to turn on the `GPIOA` clock we can set the 32 bit value `0x00000004` (bit 2 set) at address `0x4002 1018`.

{% highlight c %}
*(volatile uint32_t *)0x40021018 = 0x00000004;
{% endhighlight %}

The `volatile` keyword tells the compiler that the value may change at any time because it's accessed by other hardware. Without the keyword the compiler could notice that the value is never used by any of our code and optimize the line away. Thanks to [dmitrygr](https://news.ycombinator.com/item?id=19315732) for pointing this out in the comments.

### Port configuration register

To configure the `PA` pins there are the *`GPIOA` port configuration register low* for pins 0 to 7 and the *`GPIOA` port configuration register high* for pins 8 to 15. Since on my drone I want to configure pin `PA11` I need the high register at address `0x4001 0804`. It is 32 bits as well with four bits for each pin. Bits 12 to 15 are for pin `PA11`. The first two bits are specifying the `MODE` and the latter two bits the `CNF` value. From the reference manual:

> **CNFy[1:0]:** Port x configuration bits (y= 8 .. 15)  
> These bits are written by software to configure the corresponding I/O port.  
> Refer to Table 20: Port bit configuration table.  
> **In input mode (MODE[1:0] = 00):**  
> 00: Analog mode  
> 01: Floating input (reset state)  
> 10: Input with pull-up / pull-down  
> 11: Reserved  
> **In output mode (MODE[1:0] > 00):**  
> 00: General purpose output push-pull  
> 01: General purpose output Open-drain  
> 10: Alternate function output Push-pull  
> 11: Alternate function output Open-drain
>
> **MODEy[1:0]:** Port x mode bits (y= 8 .. 15)  
> These bits are written by software to configure the corresponding I/O port.  
> Refer to Table 20: Port bit configuration table.  
> 00: Input mode (reset state)  
> 01: Output mode, max speed 10 MHz.  
> 10: Output mode, max speed 2 MHz.  
> 11: Output mode, max speed 50 MHz.

We want to configure the pin as an output and the frequency doesn't matter for an LED so we go with the lowest 2 MHz. This means the `MODE` value will be `10`. Because our `MODE` value is greater than 0 the second table applies for the `CNF` value. We choose a push-pull output for the LED, so the `CNF` value is `00`.

So the four bits we need for the pin are `0010`. In hexadecimal the four bits of a pin are one digit. Because we don't want to touch the other pins configuration settings we first clear the four bits of pin `PA11` and then set them using a binary or.

{% highlight c %}
*(volatile uint32_t *)0x40010804 &= 0xFFFF0FFF;
*(volatile uint32_t *)0x40010804 |= 0x00002000;
{% endhighlight %}

### Port bit set/reset registers

Finally, to turn on the pin, there is the *`GPIOA` port bit set/reset register* at address `0x4001 0810`. Bits 0 to 15 correspond to the different pins.

> **BSy:** Port x Set bit y (y= 0 .. 15)  
> These bits are write-only and can be accessed in Word mode only.  
> 0: No action on the corresponding ODRx bit  
> 1: Set the corresponding ODRx bit

{% highlight c %}
*(volatile uint32_t *)0x40010810 = (1<<11);
{% endhighlight %}

For turning it off you can use the *`GPIOA` port bit reset register* at `0x4001 0814`.

{% highlight c %}
*(volatile uint32_t *)0x40010814 = (1<<11);
{% endhighlight %}

## Putting it all together

Here is the complete `main.c` program to turn the `PA11` pin on and off. The `stdint.h` header is required for the `uint32_t` type. A for loop is used as a simple way to get some delay.

{% highlight c %}
#include <stdint.h>

void main ()
{
    // Enable I/O port A clock
    *(volatile uint32_t *)0x40021018 = 0x00000004;

    // Configure pin 11 as push-pull output
    *(volatile uint32_t *)0x40010804 &= 0xFFFF0FFF;
    *(volatile uint32_t *)0x40010804 |= 0x00002000;

    while (1) {
        *(volatile uint32_t *)0x40010810 = (1<<11); // Set pin 11
        for (int i = 0; i < 1000000; i++);
        *(volatile uint32_t *)0x40010814 = (1<<11); // Unset pin 11
        for (int i = 0; i < 1000000; i++);
    }
}
{% endhighlight %}

Once you've adjusted everything for your board compile it and flash it. If everything went right the output pin should be powered and if an LED is connected it should start blinking.

You can find the complete source code on [GitHub](https://github.com/timakro/stm32-quadcopter/tree/master/blink_minimal), including a Makefile. All the manuals quoted in this post can also be found in the repository.

If I ever get to writing the next post, we will learn how to use the [CMSIS](https://developer.arm.com/embedded/cmsis) library. It defines pointers with meaningful names for the registeres we saw in this article, so we don't have to hardcode all the addresses. And it provides functions for properly timed delay so we can get rid of the for loops which lack a fixed delay.

You can leave comments on [Hacker News](https://news.ycombinator.com/item?id=19314076). I'd be happy about any feedback and corrections.

<video autoplay loop preload>
  <source src="/public/bare-metal-stm32-programming/blinking-led.mp4" type="video/mp4">
  <source src="/public/bare-metal-stm32-programming/blinking-led.webm" type="video/webm">
</video>
