---
title: "Выключаем дефрагментацию в простое"
date: 2021-04-18T01:15:36+03:00
draft: false
---

Если у вас windows 8 и выше, то отойдя на мин 5-8 можно услышать как ваш HHD надрывается, шумит и кричит о помощи. Но как только тронуть мышкой, то всё прекращается. 

Вирус? Нет (да), это автоматическая дефрагментация. Конечно, дефрагментация - это хорошо. Но нельзя ли её включать, когда этого хочет сам пользователь. Процесс, отвечающий за это безобразие - это `svchost.exe` (`defragsvc`). Чтобы выключить:
1) Нажать win + r
2) Ввести туда dfrgui (здесь же можно провести дефрагментация ваших жёсткий дисков вручную)
3) Нажать внизу "Изменить параметры"
4) Убрать галочку "Выполнять по расписанию (рекомендуется)"
5) Нажать ОК и всё.

Говорят, что эта херня работает и для SSD, что вообще недопустимо (да и зачем вообще). Так что если у вас SSD, то это необходимо выключить