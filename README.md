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

Для обновления MCU EBB36 необходимо:  

1. компилируем прошивку:  
    1.1 запускаем KIAUH  
    ```bash 
    ./kiauh/kiauh.sh
    ```  
    1.2 выбираем пункт "4) Advanced"  
    1.3 выбираем пункт "2) [Build]"  
    1.4 выставляем настройки:  
    ```bash
    [*] Enable extra low-level configuration options
        Micro-controller Architecture (STMicroelectronics STM32)  --->
        Processor model (STM32G0B1)  --->
        Bootloader offset (8KiB bootloader)  --->
        Clock Reference (8 MHz crystal)  --->
        Communication interface (USB to CAN bus bridge (USB on PA11/PA12))  --->
        CAN bus interface (CAN bus (on PB0/PB1))  --->
        USB ids  --->
    (1000000) CAN bus speed
    ()  GPIO pins to set at micro-controller startup (NEW)
    ```  
    1.5 выходим, сохраняем, выходим из KIAUH  
    1.6 прошиваем EBB36 прошивкой klipper:  
    ```bash
    python3 ~/katapult/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u {ВСТАВЛЯЕМ UUID}
    ```

Для подготовки к работе и прошивки MCU EBB36+U2C необходимо:  

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
    2.10 получаем UUID EBB36:  
    ```bash
    ~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
    ```
    или  
    ```bash
    ~/katapult/scripts/flash_can.py -i can0 -q
    ```  
    ответ должен быть таким:  
    ```bash
    Found canbus_uuid=XXXXXXXXXX, Application: Katapult
    ```  
    2.11 компилируем прошивку:   
    2.11.1 запускаем KIAUH  
    ```bash 
    ./kiauh/kiauh.sh
    ```  
    2.11.2 выбираем пункт "4) Advanced"  
    2.11.3 выбираем пункт "2) [Build]"  
    2.11.4 выставляем настройки:  
    ```bash
    [*] Enable extra low-level configuration options
        Micro-controller Architecture (STMicroelectronics STM32)  --->
        Processor model (STM32G0B1)  --->
        Bootloader offset (8KiB bootloader)  --->
        Clock Reference (8 MHz crystal)  --->
        Communication interface (USB to CAN bus bridge (USB on PA11/PA12))  --->
        CAN bus interface (CAN bus (on PB0/PB1))  --->
        USB ids  --->
    (1000000) CAN bus speed
    ()  GPIO pins to set at micro-controller startup (NEW)
    ```  
    2.11.5 выходим, сохраняем, выходим из KIAUH  
    2.12 прошиваем EBB36 прошивкой klipper:  
    ```bash
    python3 ~/katapult/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u {ВСТАВЛЯЕМ UUID}
    ```  

Настройка Mainsail для работы с EBB36  

1. Открываем printer.cfg:  
    1.1 добавляем блок инициализации MCU:  
    ```
    [mcu EBBCan]
    canbus_uuid: {ВСТАВЛЯЕМ UUID}
    ```  
    1.2 подключаем кулеры:  
    ```
    [fan]
    pin: EBBCan: PA1 #fan2

    [heater_fan heatbreak_cooling_fan]
    pin: EBBCan: PA0 #fan1
    ```  
    1.3 подключаем датчик температуры EBB36:  
    ```
    [temperature_sensor EBBCan]
    sensor_type: temperature_mcu
    sensor_mcu: EBBCan
    min_temp: 0
    max_temp: 100
    ```  
    1.4 подключаем акселерометр:  
    ```
    [adxl345]
    cs_pin: EBBCan:PB12
    spi_software_sclk_pin: EBBCan:PB10
    spi_software_mosi_pin: EBBCan:PB11
    spi_software_miso_pin: EBBCan:PB2
    axes_map: x,z,y # поменять в зависимости от установки на голову

    [resonance_tester]
    accel_chip: adxl345
    probe_points: 150,100,20
    ```  
    1.5 подключаем драйвер мотора:  
    ```
    [tmc2209 extruder]
    uart_pin: EBBCan: PA15
    run_current: 0.35
    sense_resistor: 0.051
    ```  
    1.6 настраиваем мотор и термистор экструдера:  
    ```
    [extruder]
    step_pin: EBBCan:PD0
    dir_pin: !EBBCan:PD1
    enable_pin: !EBBCan:PD2
    heater_pin: EBBCan: PB13
    sensor_pin: EBBCan: PA3
    ```  
    1.7 подключаем bltouch согласно с pinout:  
    ```
    [bltouch]
    sensor_pin: EBBCan:PB8
    control_pin: EBBCan:PB9
    ```  
2. Созранить и перезугрузить. Ошибок быть не должно, а в ```System Loads``` должен появиться ```mcu EBBCan(stm32g0b1xx)```  