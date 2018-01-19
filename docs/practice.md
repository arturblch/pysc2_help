## Practice

### Replay info parsing

#### MSC

**Install virtualenv localy with conda** 

Префикс нужен для обозначения пути и имени. Если не указан путь - создает папку в месте открытия консоли.
```
conda create --prefix=env python=3.6
```

Для активации env нужно с той же директории прописать:
```
activate env
```

Для удаления нужно в директории прописать:
```
conda env remove -n env
```

**Install requrements**
```
pip install -r requrements.txt
```
У меня с первого раза не сработало - ошибки при установке scipy и numpy. Пришлость ставить их в ручную. Но для начала нужно обновить setuptools
```
pip install -U setuptools
easy_install scipy
```

После этого находими requrements.txt и устанавливаем все необходимые зависимости командой
```
pip install -r requrements.txt
```

**Prepare replays**
Для начала подберем реплеи и сложим их в отдельную папку (путь - REPLAY_FOLDER_PATH)
После этого создадим папку для сохранения полученой информации по каждому реплею (путь - SAVE_PATH)
Затем нужно вызвать команду:
```
python parse_replay_info.py
  --replays_paths REPLAY_FOLDER_PATH
  --save_path SAVE_PATH
  --n_instance [N_PROCESSES]
  --batch_size [BATCH_SIZE]
```
Например : 
```
(env) D:\temp\MSC-master\MSC-master\preprocess>python parse_replay_info.py 
--replays_paths "D:\temp\K7SB5MO71IUQ1500658470335\Grand Final" 
--save_path "D:\temp\K7SB5MO71IUQ1500658470335\Grand Final\state"
```
В итоги получается примерно такой файл:
```
{"info": "{
\"mapName\": \"\\u0413\\u043b\\u0443\\u0431\\u043e\\u043a\\u043e\\u0432\\u043e\\u0434\\u043d\\u044b\\u0439 \\u0440\\u0438\\u0444 \\u0420\\u0412\",
\"playerInfo\": 
[
    {
        \"playerInfo\": {
            \"playerId\": 1,
            \"raceRequested\": \"Zerg\",
            \"raceActual\": \"Zerg\"
        },
        \"playerResult\": {
            \"playerId\": 1,
            \"result\": \"Defeat\"
        },
        \"playerApm\": 468
    },
    {
        \"playerInfo\": {
            \"playerId\": 2,
            \"raceRequested\": \"Zerg\",
            \"raceActual\": \"Zerg\"
        },
        \"playerResult\": {
            \"playerId\": 2,
            \"result\": \"Victory\"
            },
        \"playerApm\": 438
    }
],
\"gameDurationLoops\": 10415,
\"gameDurationSeconds\": 464.98779296875,
\"gameVersion\": \"3.15.1.54724\",
\"dataBuild\": 54724,
\"baseBuild\": 54518,
\"dataVersion\": \"6EB25E687F8637457538F4B005950A5E\"
}", 
"path": "D:\\temp\\K7SB5MO71IUQ1500658470335\\Grand Final\\Elazer vs Snute Abyssal Reef LE WCS Valencia.SC2Replay"}
```