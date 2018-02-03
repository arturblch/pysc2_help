## Начало работы с S2Protocol

Данная либо поддерживает только **python 2**. Есть ветка для поддержки **python 3** но на момент написания документации ей никто не занимался. Поэтому я решил форкнуться и дописать что надо.

Исходный репозиторий найдешь [тут](https://github.com/Blizzard/s2protocol/tree/py3-conversion)

### Установка

Для установки я использовал ветку своего репозитория (копия оригинала с небольшими дополнениями)

Чтобы склонировать весь репозиторий и при импорте использовать именно этот репозиторий нужно:

- Перейти в нужную деректорию. У меня все github-модули лежат в site-pakages в папке src (pip сам создает папку src)
- Прописать в консоли:
    
        pip install -e git+git://github.com/arturblch/s2protocol.git@py3-conversion#egg=s2protocol

    Где `-e` скачает всю репу целиком, `@py3-conversion` скачает из ветки *py3-conversion*, `#egg=s2protocol` присвоит имя *s2protocol*

### Пример Использования

    import mpyq

    # Using mpyq, load the replay file.
    archive = mpyq.MPQArchive('Elazer vs Snute Abyssal Reef LE WCS Valencia.SC2Replay')
    contents = archive.header['user_data_header']['content']

    # Now parse the header information.
    from s2protocol import versions
    header = versions.latest().decode_replay_header(contents)

### Практическое применение

Использую этот модуль на реплеях 3.16, которые предоставлялись дл PySC2, я понял следующие вещи:

- Можно удалять файлы реплея, но бакапы остаются, пусть и не в целом виде
- Игра использует какие-то еще данные, которые я не нашел в реплее, для визуацлизации(поле скинов было затерто в этих реплеях, но при воспроизведении они есть)
- Нашел как вытащить некоторую информацию, отражающие опыт игрока:
    - об прокачаных уровнях расс
    - высшей достигнутой лиги
    - награды (неизвестно как переводить в строку)
    - предпочитаемую расу
    - рейтинг игрока
- Так и не нашел индефикато скинов

 В основном все новые данные беруться с файла initData.backup по следующему алгоритму:

- Запускаем скрипт s2protocol.s2_cli.py из библиотеки папки(не настроена косвенная аддресация) с атрибутами:
    `python s2_cli.py absolute_replay_path.SC2Replay --initdata_backup --output initdata.txt`
- Структура файла состоит из вывода pprint (json не хочет парсить битовые строки)

Нужная информация хранится по следующим ключам: 

- **Лист наград** 
    - initdata['m_syncLobbyState']
        - ['m_lobbyState']
            - ['m_slots'][0/1(В зависимости от слота игрока)]
                - ['m_rewards']
- **Предпочитаемая раса** 
    - initdata['m_syncLobbyState']
        - ['m_lobbyState']
            - ['m_slots'][0/1(В зависимости от слота игрока)]
                - ['m_racePref']['m_race']
- **Прокачаные уровни классов**
    - initdata['m_syncLobbyState']
        - ['m_userInitialData'][0/1(В зависимости от слота игрока)]
            - ['m_combinedRaceLevels']
- **Высшая достигнутая лига**
    - initdata['m_syncLobbyState']
        - ['m_userInitialData'][0/1(В зависимости от слота игрока)]
            - ['m_highestLeague']
- **Рейтинг**
    - initdata['m_syncLobbyState']
        - ['m_userInitialData'][0/1(В зависимости от слота игрока)]
            - ['m_scaledRating']

