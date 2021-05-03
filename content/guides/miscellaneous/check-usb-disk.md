---
title: "Проверка флешек на обьём"
date: 2021-04-18T01:09:07+03:00
draft: false
---

После покупки флешек на ali нужно их всегда проверять. Недобросовестные продавцы будут указывать объём больше, чем он есть на самом деле (хотя система будет показывать, что всё ОК). Они это делают (во всяком один из способов), форматируя флешки на низком уровне. Можно на них нажиться, вернуть деньги и оставить флешку, просто реального объёма.

Как вставили флешку запускайте [h2testw](https://www.usbdev.ru/files/h2testw/), выбирайте English, после выберите флэшку в Select target, тыкните all available space и снимите галочку, если стоит, в endless verify. Теперь просто нажать Write+Verify и ждать. Долго ждать.

Если всё хорошо, то будет типа такого вывода:
```
Test finished without errors.
You can now delete the test files *.h2w or verify them again.
Writing speed: 6.41 MByte/s
Reading speed: 21.2 MByte/s
H2testw v1.4 
```

Если же нет, то может быть такая надпись:
```
The media is likely to be defective.
58.5 GByte OK (122845070 sectors)
57 KByte DATA LOST (114 sectors)
Details:0 KByte overwritten (0 sectors)
0 KByte slightly changed (< 8 bit/sector, 0 sectors)
57 KByte corrupted (114 sectors)
0 KByte aliased memory (0 sectors)
First error at offset: 0x0000000171279c00
Expected: 0x0000000171279c00
Found: 0xcea8a293915717a4
H2testw version 1.3
Writing speed: 17.9 MByte/s
Reading speed: 95.0 MByte/s
H2testw v1.4
```
или такое
```
Warning: Only 4052 of 4091 MByte tested.
The media is likely to be defective.
3.9 GByte OK (8252480 sectors)
22.4 MByte DATA LOST (46016 sectors)
Details:2 MByte overwritten (4096 sectors)
0 KByte slightly changed ( 8 bit/sector, 0 sectors)
20.4 MByte corrupted (41920 sectors)
2 MByte aliased memory (4096 sectors)
First error at offset: 0x00000000005f6000
Expected: 0x00000000005f6000
Found: 0xa91360744cd56eb3
H2testw version 1.3
Writing speed: 10.2 MByte/s
Reading speed: 26.6 MByte/s
H2testw v1.4
```
В любом случае окошко станет красно, уже понятно, что что-то не так.

Пишите продавцу, чтобы возвращал деньги. Нужен скрин программы h2testw; фотография (видео), где видно, что проверяемая флешка воткнута в usb порт, нет никаких других. Также на фотографии (видео) должно быть видно вывод программы h2testw (обоих окон) и окно винды, где показана буква вашей флешки. Пусть возвращает деньги, оставляя флешку вам. Если отказывается, то поднимайте спор на уровень выше. Ведь такие флешки несут угрозу вашим данным. Если при записи на флешку исчерпывается её реальный объём, то начинаются затираться ваши файлы. Это не дело.

Теперь решение проблемы. В строке "... GByte OK (xxx sectors)" копируем кол-во этих самых xxx секторов, запускаем программу [MyDiskFix-ENG](https://www.usbdev.ru/files/mydiskfix/) и вбиваем в `Sectors:` во второе окошко эти xxx секторов (32 не трогаем!). Тыкаем Low-Level, выбираем в `Choose Device:` нужную флешку и жмём `START Format`. Может всплыть окошко, но там просто нажимаем ок или типа того. Везде в выводе программы должны стоять `...OK!`. Объём, конечно, уменьшится, но он станет хотя бы правильным.