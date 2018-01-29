
## Парсим json-реплеи в csv

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
col=['map_name', 'race_p1', 'apm_p1', 'race_p2', 'apm_p2', 'win_player', 'game_loops', 'game_seconds', 'game_version', 'path']

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
