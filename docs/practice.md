## Replay info parsing

### MSC

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


## Data Mining

В репе [MSC](https://github.com/wuhuikai/MSC/blob/master/instructions/HardWay.md#parse-replay-info) описан процесс парсинга реплеев в json-файл. Для чтения json-файлов описан следующий алгоритм:
```python
import json
from google.protobuf.json_format import Parse
from s2clientprotocol import sc2api_pb2 as sc_pb

with open(REPLAY_INFO_PATH) as f:
    info = json.load(f)
REPLAY_PATH = info['path']
REPLAY_INFO_PROTO = Parse(info['info'], sc_pb.ResponseReplayInfo())
```
**Цель** данной работы - **обработать** все существующие *json-реплеи* и сохранить их в виде *таблицы в csv-формате* для дальнейшего анализа


```python
import pandas as pd
import json
from google.protobuf.json_format import Parse
from s2clientprotocol import sc2api_pb2 as sc_pb
import os
from tqdm import tnrange, tqdm_notebook

REPLAY_INFOS = 'D:\\temp\MSC-master\\MSC-master\\replays_infos'
```


```python
col=['map_name',
     'race_p1',
     'apm_p1',
     'race_p2',
     'apm_p2',
     'win_player',
     'game_loops',
     'game_seconds',
     'game_version',
     'path']

games_df = pd.DataFrame(columns=col)
```

Одиночный реплей парсим следующим образом


```python
REPLAY = '0000e057beefc9b1e9da959ed921b24b9f0a31c63fedb8d94a1db78b58cf92c5.SC2Replay'
REPLAY_INFO_PATH = os.path.join(REPLAY_INFOS, REPLAY)
with open(REPLAY_INFO_PATH) as f:
    info = json.load(f)
replay_info = Parse(info['info'], sc_pb.ResponseReplayInfo())
game = {
        'map_name': replay_info.map_name, 
        'race_p1': replay_info.player_info[0].player_info.race_actual, 
        'apm_p1': replay_info.player_info[0].player_apm, 
        'race_p2': replay_info.player_info[1].player_info.race_actual, 
        'apm_p2': replay_info.player_info[1].player_apm , 
        'win_player': replay_info.player_info[0].player_result.result, 
        'game_loops': replay_info.game_duration_loops, 
        'game_seconds': replay_info.game_duration_seconds, 
        'game_version': replay_info.game_version, 
        'path': info['path']
    }
games_df = games_df.append(game, ignore_index=True)
```

Алгоритм для всех *json-реплеев*


```python
games_list = []
for REPLAY in tqdm_notebook(os.listdir(REPLAY_INFOS), desc="Work"):
    REPLAY_INFO_PATH = os.path.join(REPLAY_INFOS, REPLAY)
    with open(REPLAY_INFO_PATH) as f:
        info = json.load(f)
    replay_info = Parse(info['info'], sc_pb.ResponseReplayInfo())
    game = {
        'map_name': replay_info.map_name, 
        'race_p1': replay_info.player_info[0].player_info.race_actual, 
        'apm_p1': replay_info.player_info[0].player_apm, 
        'race_p2': replay_info.player_info[1].player_info.race_actual, 
        'apm_p2': replay_info.player_info[1].player_apm , 
        'win_player': replay_info.player_info[0].player_result.result, 
        'game_loops': replay_info.game_duration_loops, 
        'game_seconds': replay_info.game_duration_seconds, 
        'game_version': replay_info.game_version, 
        'path': info['path']
    }
    games_list.append(pd.Series(game))
```


```python
games_list[1]
```


```python
games_df = pd.concat(games_list, axis=1).transpose()
games_df.to_csv('test_example.csv', encoding='utf-8', sep=';')
```