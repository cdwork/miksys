# miksys

MIKSYS -- это SoC, разработанная для платы Марсоход2 (http://marsohod.org/prodmarsohod2) с ПЛИС Altera Cyclone III.
Работа выполнена для получения опыта разработки на verilog. Практической ценности не имеет.
Была поставлена цель выжать из платы производительность, достаточную для отображения 3д графики.

Обсуждение проекта: http://marsohod.org/forum/proekty-polzovatelej/4110-realizatsiya-3d-grafiki-na-plate-marsohod2

# Файлы:

* miksys.html - справка по системе команд
* write.sh - загрузить прошивку в марсоход2 (linux)
* write_epcs4.sh - загрузить прошивку в микросхему epcs4 на плате разъемов (linux)
* prepare_port.sh - установить нужные настройки последовательно порта (linux)
* verilog/* - проект quartus
* qt_sim - эмулятор (исходники)
* qt_sim_build - эмулятор (откомпилированный для linux)
* mbftdi - программатор http://marsohod.org/downloads/doc_download/91-programmator-usb-mbftdi-versiya-1-0 (linux)
* miksys_soft/compile.py - транслятор ассемблера (есть зависимость от gcc - используется препроцессор gcc. При запуске в windows потребуется mingw compiler и соответствующая настройка $PATH)
* miksys_soft/pack.py - упаковывает откомпилированную программу для загрузки по последовательному порту
* miksys_soft/pack_usb.py - упаковывает откомпилированную программу для загрузки по USB
* miksys_soft/demo3d - демонстрационная программа с 3д графикой. Режим вращения и перспектива переключаются кнопкой на плате (вторая кнопка - reset). Можно управлять вращением с PS/2 клавиатуры (кнопки WASD) или с PS/2 мыши (управление мышью глючит).
* miksys_soft/mikasm.lang - подсветка синтаксиса ассемблера для gtksourceview
* miksys_soft/ustartup - код загрузчика. После изменения загрузчика нужно откомпилировать (miksys_soft/compile.py), сконвертировать в *.hex (verilog/startup/gen_hex.py) и заменить файл verilog/startup/startup.hex
* miksys_soft/include/std.H - описания и адреса функций, находящихся в загрузчике. После изменения загрузчика следует исправить адреса в этом файле и перекомпилировать использующие его программы.


# Инструкция:

1. Собрать проект verilog/miksys.qsf и записать его в marsohod2.
Можно воспользоваться готовыми *.svf файлами -- verilog/miksys.svf (для записи в ПЛИС) или
write_epcs4_part1.svf, write_epcs4_part2.svf -- для записи в микросхему epcs4 на плате расширения

В линуксе можно использовать write.sh для записи непосредственно в ПЛИС или write_epcs4.sh для записи в epcs4

2. Откомпилировать программу на ассемблере miksys (miksys_soft/compile.py) и упаковать её для загрузки через последовательный порт (miksys_soft/pack.py) или загрузки с флешки (miksys_soft/pack_usb.py). pack.py и pack_usb.py добавляют к файлу контрольную сумму в требуемом формате.

Для демонстрационной программы demo3d готовы файлы miksys_soft/serial_in (для последовательно порта) и miksys_soft/demo3d/demo3d.usb_packed.
В линуксе для пересборки demo3d есть скрипт miksys_soft/demo3d/build.sh

3. (необязательный пункт) Запустить эмулятор qt_sim. Это проект на QT, открывать в qtcreator. При запуске эмулятора содержимое файла miksys_soft/serial_in будет отправлено в последовательный порт эмулированной системы. Т.е. при первом запуске эмулятора должна запуститься программа demo3d.

4. (a) Загрузка через последовательный порт.
Настроить baud_rate виртуального последовательного порта 6MHz (в линуксе делается скриптом prepare_port.sh).
Отправить serial_in в последовательный порт.

4. (b) Загрузка с флешки. Работает не для всех флешек (я использую флешку SanDisk CruzerBlade - для неё работает).
В загрузчике прописаны номера endpoint-ов (их автоматическое распознавание в загрузчик не поместилось): endpoint_in - 1, endpoint_out - 2. Если у флешки другие номера endpoint-ов, придется перекомпилировать загрузчик.

Откомпилированную программу нужно записать на флешку, начиная с блока 2048 (начало 2-го мегабайта).
Это удобно сделать, если настроить таблицу разделов на флешке, чтобы один из разделов начинался с этого адреса. Например:

    Устр-во Загр     Начало       Конец       Блоки   Id  Система
    /dev/sdb1          264192    15633407     7684608   83  Linux
    /dev/sdb2            2048      264191      131072   83  Linux

В этом случае запись на флешку осуществляется (linux) командой `sudo dd if=miksys_soft/demo3d/demo3d.usb_packed of=\dev\sdb2`

# Недоработки:

1) Не реализована работа со звуком

2) Не реализован кэш команд. На данный момент кэш команд жестко привязан к адресам 0x100000-0x101000 оперативной памяти.
Т.е. любой исполняемый код (кроме кода загрузчика) должен лежать в этом кусочке оперативной памяти. Загрузчик размещает программу по адресу 0x100000 и передаёт туда управление.

3) Не реализованы команды LPU0-LPU3
