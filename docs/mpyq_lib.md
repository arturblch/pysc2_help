## mpyq

[github.com/eagleflo/mpyq](https://github.com/eagleflo/mpyq) - Репозиторий библиотеки

### Использование скриптов

Команду mpyq можно запустить с консоли, при этом можно указать следующие аргументы:
```
 - h, --help           Отображает меню помощи
 - I, --headers        Печатает загаловок архива
 - H, --hash-table     Печатает хэш таблицу
 - b, --block-table    Печатает таблицу блоков
 - s, --skip-listfile  Пропуск чтения списка файлов
 - t, --list-files     Печатает список файлов архива
 - x, --extract        Распаковывает архив
```

Пример использования скрипта для отобрадения списка файлов в архиве
```python 
$ mpyq -t game.SC2Replay

Files
-----
replay.attributes.events            580 bytes
replay.details                      443 bytes
replay.game.events                42859 bytes
replay.initData                    1082 bytes
replay.load.info                     96 bytes
replay.message.events                94 bytes
replay.smartcam.events             1444 bytes
replay.sync.events                  765 bytes
```