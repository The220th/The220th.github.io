---
title: "Удаление предустановленных приложений на Android"
date: 2021-04-18T22:49:36+03:00
draft: false
---

Чтобы увеличить привлекательность смартфонов, производители ставят на них как можно больше разных программ. Это понятно. Просто берём и удаляем ненужное… Стоп.

Оказывается, некоторые программы невозможно удалить. Например, на отдельных моделях Samsung невозможно удалить Facebook (есть только опция 'disable', в итоге будет просто место занимать). Говорят, на Samsung S9 вдобавок предустановлены «неудаляемые» приложения Microsoft.

Всё это надо зачистить.

Впрочем, «неудаляемые» они только теоретически. На практике достаточно открыть ADB (Android Debug Bridge) и запустить пару команд.  
На телефоне должна быть разрешена отладка по USB (семь тапов но номеру сборки в настройках и включаем её в новом пункте меню), а на компьютере установлен USB-драйвер устройства.

Теперь приступаем.

`pm list packages | grep '<OEM/Carrier/App Name>'`


выводит список установленных пакетов.
```
pm list packages | grep 'oneplus'
package:com.oneplus.calculator
package:net.oneplus.weather
package:com.oneplus.skin
package:com.oneplus.soundrecorder
package:com.oneplus.opsocialnetworkhub
package:cn.oneplus.photos
package:com.oneplus.screenshot
package:com.oneplus.deskclock
package:com.oneplus.setupwizard
package:com.oneplus.sdcardservice
package:com.oneplus.security
package:cn.oneplus.nvbackup
package:com.oneplus.wifiapsettings
```

Как вариант, можно установить на телефоне бесплатную программу Инспектор приложений. Она покажет подробную информацию обо всех установленных приложениях, их разрешения. Вдобавок она может извлекать (скачивать) APK-файлы для любого установленного приложения.

Для удаления конкретного пакета запускаем такую команду:

`pm uninstall -k --user 0 <name of package>`


Это работает без рутования.

Для упомянутых в начале статьи «неудаляемых» программ это выглядит так:

Facebook  
`pm uninstall -k –user 0 com.facebook.katana`


Facebook App Installer  
`pm uninstall -k –user 0 com.facebook.appmanager`


Microsoft OneDrive  
`pm uninstall -k –user 0 com.microsoft.skydrive`


Microsoft PowerPoint  
`pm uninstall -k –user 0 com.microsoft.office.powerpoint`


Microsoft OneNote  
`pm uninstall -k –user 0 com.microsoft.office.onenote`


… и так далее.