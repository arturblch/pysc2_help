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