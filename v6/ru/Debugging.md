---
title: Отладка
layout: default
---
# Отладка #

При загрузке приложение iikoFront выполняет поиск плагинов и для каждого из них запускает процесс-контейнер `Resto.Front.Api.Host.exe`, в контексте которого плагин и будет работать. При всех своих эксплуатационных преимуществах этот подход создаёт ряд неудобств на этапе отладки:

- не подключается автоматически отладчик Visual Studio (не работают breakpoints),
- после каждой остановки плагина и внесения исправлений необходимо перезапускать приложение iikoFront (долго).

## Visual Studio
Для отладки плагина в Visual Studio можно запускать тот же самый хост-процесс `Resto.Front.Api.Host.exe`, указав в свойствах проекта плагина следующие параметры отладки:

- Start Action: Start external program: `C:\Program Files\iiko\iikoRMS\Front.Net\Resto.Front.Api.Host.exe`.
- Start Options: Command line arguments: `/a=C:\Projects\Front.Api Sdk\Resto.Front.Api.SamplePlugin\bin\Debug\Resto.Front.Api.SamplePlugin.dll /c=Resto.Front.Api.SamplePlugin.SamplePlugin /m=21005108 /l=C:\Projects\Front.Api Sdk\Logs\sample-plugin.log`

Аргументы командной строки:

- `-a`, `--assembly`=file — полный путь к файлу плагина,
- `-c`, `--class`=name    — полное имя класса плагина,
- `-l`, `--log`=log       — полный путь к файлу лога,
- `-m`, `--module`=id     — модуль лицензии, указываемый в атрибуте `PluginLicenseModuleId` ([Лицензирование](Licensing)),
- `-n`, `--nowindow`      — не показывать консольное окно.

Далее достаточно один раз запустить приложение iikoFront (не устанавливая плагин в `Plugins`), и можно будет многократно запускать и останавливать плагин прямо из Visual Studio (`F5` / `Shift+F5`), не перезапуская iikoFront. Отладчик будет подключаться автоматически, соответственно, будут работать breakpoints, а также отладка будет прерываться при возникновении исключений (в Visual Studio можно настроить, для каких типов исключений надо или не надо прерывать отладку). Записи в лог (`PluginContext.Log`) будут дублироваться в консольном окне, если оно не скрыто ключом `--nowindow`, и в окне Output (`Ctrl+Alt+O`), если подключен отладчик.

Для отладки плагина таким способом необходима лицензия разработчика плагинов.

## Альтернативные варианты
Если лицензии разработчика плагинов нет, придётся устанавливать плагин в `Plugins`, чтобы приложение iikoFront запускало его штатным способом, а при каждом изменении плагина приложение придётся останавливать и запускать снова. При этом можно свести неудобства к минимуму следующим образом:

- В Visual Studio в настройках проекта на вкладке `Build` в разделе `Output` указать в качестве `Output path` путь к подпапке плагина в папке `Plugins`. Благодаря этому после каждой сборки плагин сразу будет публиковаться в эту папку, его не придётся копировать вручную.
- В начале конструктора класса плагина добавить вызов `Debugger.Launch()`, чтобы получать предложение подключить отладчик. Можно, конечно, подключить его самому, выбрав в меню `Debug → Attach to Process`, но это потребует больше кликов, кроме того, придётся выбирать из нескольких процессов-контейнеров с одинаковым названием.