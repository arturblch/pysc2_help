## Установка StarCraft 2

Если игра установлена не в каталоге по умолчанию, то нужно установить системную переменную SC2PATH. При этом конфигурация корневого каталога игры должна вылядить следующим образом:

- StarCraft II/
    - Battle.net/
    - Maps/
    - Replays/
    - SC2Data/
    - Versions/

## Некоторые особенности Starcraft

### Неопределенность

Starcraft II  почти не имеет рандомных явлений, но у него есть неопределенность, которая определена по косметическим причинам. Двумя основными случайными событиями являются скорость атаки и порядок обновления.

Скорость атаки случайна в из-за задержки юнитов перед выстрелом и составляет от -1 до +2 игровых шагов для почти всех единиц. Идея состоит в том, что группа морпехов может начать стрелять вместе, но задержка +/- для следующего выстрела сделает группу менее роботизированной и предотвратит их синхронную стрельбу. Поэтому справедливый матч (например, 1х1 морпехов) приведет к случайному результату.

Порядок обновления тоже случаен и определяет порядок событий в заданном игровом цикле. Например, если у вас есть два высоких тамплиера, которые бросают обратную связь друг на друга на одном игровом цикле, то заранее неопределено кто нанесет урон первым и выиграет.

Автоматический таргетинг объектов детерминирован, но сложен. Он основан на оценке расстояний стрельбы оружия, угрозы и помощи. Юниты будут считать любых врагов с оружием более приорететными, чем без оружия, и враги, которые могут нанести ответный огонь, будут более приоритетными, чем те, которые не могут. Единицы, которые не могут атаковать этот юнит (например, Ракетная башня против морского пехотинца), но которые атакуют союзную единицу (например, medivac), попросят помощь и увеличат приоритет. Целитель поднимает свой приоритет, когда исцеляет. Если две цели имеют одинаковый приоритет, будет выбран ближайший.

Эта неопределенность может быть убрана или уменьшена путем установки начального значения для генератора псевдослучайных чисел. Повторы работают потому что настройки игры включают начальное значение генератора, а также список действий всех игроков, а затем воспроизводятся действия.


### Расчет APM(Action per minute)

Игра вычисляет APM запутанным способом. Не очевидно, как подсчитать действие перемещения камеры с помощью прокрутки края или сколько действий стоит здание. Вот несколько правил, как это было в игре.

Есть на самом деле два типа сообщается в игре:

 - Действия в минуту (APM): подсчитывает каждое действие.
 - Эффективные действия в минуту (EPM): фильтрует действия которые не имеют никакого влияние (например: повторное выделение).

Различные действия имеют разные коэффициенты:

 - Команды с целью = 2 (например: перемещение, атака, построение здания)
 - Команды без цели = 1 (например: остановка, Железнодорожный блок, выгрузка груза)
 - Smart = 1 (щелкните правой кнопкой мыши)
 - Группы выбора и управления = 1
 - Все остальное = 0 (например: движение камеры)

Пользовательский интерфейс рассчитывает это с двумя различными временными интервалами: средний (средний за всю игру до момента расчета) и текущий (средний за последние 5 секунд). API предоставляет только средний APM.
