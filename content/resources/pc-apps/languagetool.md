---
title: "LanguageTool - Локальный сервер проверки орфографии  "
date: 2021-06-15T17:21:25+03:00
draft: false
authors: ["WexCore"]
---

LanguageTool — программное обеспечение для проверки грамматики, орфографии и стиля. А то, что описано в этой статье, является сервером, через который совершают проверку орфографии другие утилиты, к примеру расширения для браузера или VSCode.

## Установка
Рекомендую проводить установку через ваш любимый менеджер пакетов на Linux 
``` bash
sudo apt install languagetool
# Для Ubuntu возможно также потребуется пакет libreoffice-java-common
sudo pacman -S languagetool
```
или через [Chocolatey]({{< ref "/resources/pc-apps/choco.md" >}}) для Windows.
``` powershell
choco install languagetool
```
Если же менеджер пакетов вам не подходит, то вы можете вручную [загрузить архив с сервером](https://languagetool.org/download/LanguageTool-stable.zip) и распаковать его в удобную вам папку.

## Использование
Для запуска сервера перейдите в директорию с сервером и запустите команду
``` bash
java -cp languagetool-server.jar org.languagetool.server.HTTPServer --port 8081 --allow-origin "*"
```
