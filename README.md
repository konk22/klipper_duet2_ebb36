Для обновления MCU duet2 wifi необходимо:  

1. компилируем прошивку:  
    1.1 подключаемся через ssh;  
    1.2 запускаем KIAUH  
    ```bash  
    ./kiauh/kiauh.sh
    ```  
    1.3 выбираем пункт "4) Advanced"  
    1.4 выбираем пункт "4) [Build + Flash]"  
    1.5 выставляем настройки:  
    ```bash  
    [ ] Enable extra low-level configuration options
        Micro-controller Architecture (SAM3/SAM4/SAM E70 (Due and Duet))  --->
        Processor model (SAM4e8e (Duet Wifi/Eth))  --->
        Communication interface (USB)  --->
    ```  
    1.6 выходим, начинается компиляция. В процессе отвечаем на вопросы и прошиваем  

Для подготовке к работе и прошивки MCU EBB36+U2C необходимо:  

1. ПРИ НЕОБХОДИМОСТИ(обычно не нужно) подключаем U2C через usb к raspberry pi с зажатой кнопкой boot, после подключения отпускаем кнопку:  
    1.1 скачиваем прошивку под свой процессор:  
    ```bash
    wget -P ~/ https://github.com/bigtreetech/U2C/blob/master/firmware/{ВСТАВИТЬ НЕОБХОДИМЫЙ ФАЙЛ ПРОШИВКИ}
    lsusb
        Bus 001 Device 005: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
    sudo dfu-util -a 0 -D ~/{ВСТАВИТЬ СКАЧАННЫЙ ФАЙЛ ПРОШИВКИ} --dfuse-address 0x08000000:force:mass-erase:leave -d {ВСТАВИТЬ ID ИЗ КОМАНДЫ lsusb}
    ```  
    1.2 начинается прошивка, ждем строку ```File Downloaded Sucessfully```  
2. Подключаем EBB36 через usb к raspberry pi, зажимаем кнопку ```boot```, затем кликаем кнопку ```reset``` и отпускаем ```boot```(необходимо поставить перемычку для питания EBB36 от USB)  
    2.1 скачиваем katapult:  
    ```bash
    git clone https://github.com/Arksine/katapult
    cd katapult
    make menuconfig
    ```
    2.2 выставляем настройки:  
    ```bash
    Micro-controller Architecture (STMicroelectronics STM32)  --->
    Processor model (STM32G0B1)  --->
    Build Katapult deployment application (8KiB bootloader)  --->
    Clock Reference (8 MHz crystal)  --->
    Communication interface (CAN bus (on PB0/PB1))  --->
    Application start offset (8KiB offset)  --->
    (1000000) CAN bus speed
    ()  GPIO pins to set on bootloader entry
    [*] Support bootloader entry on rapid double click of reset button
    [ ] Enable bootloader entry on button (or gpio) state
    [*] Enable Status LED
    (PA13)  Status LED GPIO Pin
    ```  
    2.3 выходим с сохранением  
    2.4 компилируем  
    ```bash
    make clean
    make
    ```  
    2.5 прошиваем загрузчик CAN в EBB36:
    ```bash
    lsusb
        Bus 001 Device 005: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
    sudo dfu-util -a 0 -D ~/katapult/katapult.bin --dfuse-address 0x08000000:force:mass-erase:leave -d {ВСТАВИТЬ ID ИЗ КОМАНДЫ lsusb}
    ```  
    2.6 отключаем EBB36 от USB;  
    2.7 соединяем U2C и EBB36 проводами CAN;  
    2.8 подключаем U2C к USB. Если питание 24в в U2C не подключено или EBB36 подключен к U2C только проводами CAN(без питания), то подаем питание EBB36 через любой USB;  
    2.9 создаем CAN устройство в klipper:  
    ```bash
    sudo nano /etc/network/interfaces.d/can0
        allow-hotplug can0
        iface can0 can static
            bitrate 1000000
            up ifconfig $IFACE txqueuelen 256
            pre-up ip link set can0 type can bitrate 1000000
            pre-up ip link set can0 txqueuelen 256

    ```  
    2.10 