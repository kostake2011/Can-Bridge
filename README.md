# Can-Bridge

Мануал расчитывает на некоторые базовые знания, как перевести октопус в дфу. Как нажать Y если система того просит и т.д. Я привел два примера под октопус на 446 и на 723м проце. Если у вас 723й и вы уверены в своем кабеле, то скорость CAN шины можете попробовать в 1000000. Если же 446й, то везде где дальше увидите строчку 1000000 меняйте на 500000.
PS: все на свой страх и риск, автор данного текста никакой ответственности не несет.

1)Начало, обновляем систему и до устанавливаем пакеты
sudo apt update
sudo apt upgrade
sudo apt install python3 python3-pip python3-can
sudo pip3 install pyserial
2) Ставим катапульт

git clone https://github.com/Arksine/katapult
cd katapult
make clean


3) Создаем прошивки 
make menuconfig

446 проц

 
Если 723й
 
make

И переносим прошивку в другую папку
mkdir ~/firmware	
mv ~/katapult/out/katapult.bin ~/firmware/octopus_katapult.bin

4) Вводим октопус в режим DFU и вводим команду lsusb

 

Видим устройство в DFU, копируем его ID, в моем случае это было 0483:df11 и вставляем в следующую команду

Прошиваем октопус следующей командой, подставив свои ID

sudo dfu-util -a 0 -D ~/firmware/octopus_katapult.bin --dfuse-address 0x08000000:force:mass-erase:leave -d 0483:df11


Проверяем командой lsusb и видим плату на STM723, все ок.
 


5) Подключаем EBB36 в режиме DFU к PI через кабель USB , предварительно установим перемычку для питания от USB (микрофит отключен), проверяем также командой lsusb

 
Видим наш октопус (у меня это stm723hxx) и опять устройство в DFU.

Собираем прошивку для EBB36

cd katapult
make clean
make menuconfig

 

мake

переносим прошивку и прошиваем аналогично октопусу

mv ~/katapult/out/katapult.bin ~/firmware/ebb_katapult.bin

и прошиваем ebb, подставив также ID уже ее. Если они отличные от моих.

sudo dfu-util -a 0 -D ~/firmware/ebb_katapult.bin --dfuse-address 0x08000000:force:mass-erase:leave -d 0483:df11


6) Создаем Can интерфейс

cd / 
sudo nano /etc/network/interfaces.d/can0

откроется текстовый редактор, вставляем туда следующие строчки и сохраняемся: 

allow-hotplug can0
iface can0 can static
  bitrate 1000000
  up ifconfig $IFACE txqueuelen 1024

 
Сохраняем файл

7) Собираем прошивку для режима кан бридж октопуса
Если вам нужно, чтобы при включении октопус поднимал пин PS_on, прописываем его на данном этапе

cd ~/klipper
make clean
make menuconfig

446
 
723
 

make
mv ~/klipper/out/klipper.bin ~/firmware/octopus_klipper.bin



8)Собираем прошивку для EBB

make clean
make menuconfig

 

make
mv ~/klipper/out/klipper.bin ~/firmware/ebb_klipper.bin

8) Опрос  устройств

ls -al /dev/serial/by-id

 
Видим строчку с октопусом. 
cd ~/katapult/scripts
pip3 install pyserial

В следующую команду подставить свой Serial  октопуса полученный на скрине выше.
python3 flash_can.py -f ~/firmware/octopus_klipper.bin -d /dev/serial/by-id/usb-katapult_stm32h723xx_390025000D51313236343430-if00

 

Октопус прошит, после чего выключаем пишку командой

sudo shutdown now

Отключаем питание. Подключаем EBB . не забываем перемычку 120ohm 

9) Заново заходим в путти и выполняем команды (вводить сразу обе)

cd ~/katapult/scripts
python3 flash_can.py -i can0 –q

Если не сработает, то команда
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0

Видим обе платы. Немного радуемся 
 
На данном этапе с пометкой Katapult . это EBB36. Берем ее uuid и вставляем в следующую команду и прошиваем ebb 

python3 flash_can.py -i can0 -u 28c5348109be -f ~/firmware/ebb_klipper.bin
  прошивка завершена
10)Вновь опрашиваем платы 

python3 flash_can.py -i can0 -q

 
копируем uuid в конфиги - и радуемся 





