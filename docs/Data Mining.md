
## Парсим json-реплеи в csv

В репе MSC было предложено следующее решение:
```python
import json
from google.protobuf.json_format import Parse
from s2clientprotocol import sc2api_pb2 as sc_pb

with open(REPLAY_INFO_PATH) as f:
    info = json.load(f)
REPLAY_PATH = info['path']
REPLAY_INFO_PROTO = Parse(info['info'], sc_pb.ResponseReplayInfo())
```


```python
%matplotlib inline
```


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


```python
for i in tqdm_notebook(range(1000), desc='hi'):
    pass
```


<p>Failed to display Jupyter Widget of type <code>HBox</code>.</p>
<p>
  If you're reading this message in the Jupyter Notebook or JupyterLab Notebook, it may mean
  that the widgets JavaScript is still loading. If this message persists, it
  likely means that the widgets JavaScript library is either not installed or
  not enabled. See the <a href="https://ipywidgets.readthedocs.io/en/stable/user_install.html">Jupyter
  Widgets Documentation</a> for setup instructions.
</p>
<p>
  If you're reading this message in another frontend (for example, a static
  rendering on GitHub or <a href="https://nbviewer.jupyter.org/">NBViewer</a>),
  it may mean that your frontend doesn't currently support widgets.
</p>



    
    


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


<p>Failed to display Jupyter Widget of type <code>HBox</code>.</p>
<p>
  If you're reading this message in the Jupyter Notebook or JupyterLab Notebook, it may mean
  that the widgets JavaScript is still loading. If this message persists, it
  likely means that the widgets JavaScript library is either not installed or
  not enabled. See the <a href="https://ipywidgets.readthedocs.io/en/stable/user_install.html">Jupyter
  Widgets Documentation</a> for setup instructions.
</p>
<p>
  If you're reading this message in another frontend (for example, a static
  rendering on GitHub or <a href="https://nbviewer.jupyter.org/">NBViewer</a>),
  it may mean that your frontend doesn't currently support widgets.
</p>



    
    

Race:
0. NoRace 
1. Terran
2. Zerg
3. Protoss
4. Random


```python
games_list[0]
```




    apm_p1                                                        386
    apm_p2                                                        384
    game_loops                                                  20887
    game_seconds                                               932.52
    game_version                                         3.16.1.55958
    map_name                                             Меха-депо РВ
    path            D:\Program Files (x86)\StarCraft II\Replays\3....
    race_p1                                                         3
    race_p2                                                         2
    win_player                                                      2
    dtype: object




```python
games_df = pd.concat(games_list, axis=1).transpose()
```


```python
def race(num):
    if num ==1:
        return('Terran')
    if num ==2:
        return('Zerg')
    if num ==3:
        return('Protoss')

games_df['race_p1'] = games_df['race_p1'].map({1:'Terran', 2: 'Zerg', 3: 'Protoss'})

```


```python
games_df.win_player.value_counts(normalize=True)
```




    2    0.500272
    1    0.496949
    3    0.002780
    Name: win_player, dtype: float64




```python
games_df.map_name.value_counts(normalize=True)
```




    Одиссея РВ              0.169330
    Путь на Айур РВ         0.168491
    Меха-депо РВ            0.145306
    Незваный гость РВ       0.143567
    Глубоководный риф РВ    0.137293
    Аколит РВ               0.133473
    Каталлена РВ (Void)     0.102539
    Name: map_name, dtype: float64




```python
def mutchup(row):
    p = sorted([row.race_p1,row.race_p2])
    return (p[0][0] + 'v' + p[1][0])            
                
games_df['mutchup'] = games_df.apply(mutchup, axis=1)
```


```python
filter_games = games_df[(games_df.apm_p1 > 30) & (games_df.apm_p2 > 30)]
```


```python
filter_games = filter_games[filter_games.game_seconds > 60]
```


```python
filter_games[(filter_games.race_p1 =='Terran') & (filter_games.win_player ==1) 
           | (filter_games.race_p2 =='Terran') & (filter_games.win_player == 2)]
```
