# PR02 — команды и пояснения

## 1. Три команды Linux, использованные в работе

### Команда 1: `mkdir -p src evidence/pr02`

- **Назначение:** создать каталоги `src/` и `evidence/pr02/`, включая все недостающие родительские каталоги. Флаг `-p` не выдаёт ошибку, если каталог уже существует.
- **Результат:** в корне репозитория появились `src/` и `evidence/pr02/`.

### Команда 2: `source /opt/ros/jazzy/setup.bash`

- **Назначение:** подключить базовое окружение ROS 2 Jazzy в текущий терминал (переменные `ROS_DISTRO`, `AMENT_PREFIX_PATH`, `PATH`, `PYTHONPATH` и др.).
- **Результат:** `printenv ROS_DISTRO` выводит `jazzy`; команды `ros2` доступны.

### Команда 3: `ros2 pkg prefix turtlesim`

- **Назначение:** показать путь установки пакета `turtlesim`.
- **Результат:** `/opt/ros/jazzy` — пакет установлен из системного репозитория ROS 2.

---

## 2. Чем `>` отличается от `|`

- `>` перенаправляет **stdout** команды в файл, **заменяя** его прежнее содержимое. Например: `ros2 doctor --report > evidence/pr01/doctor.txt 2>&1` — весь вывод и ошибки пишутся в файл, старые данные теряются.
- `|` (конвейер) передаёт **stdout** одной команды на **stdin** следующей, не создавая файл. Например: `colcon build ... 2>&1 | tee evidence/pr02/build.txt` — вывод одновременно показывается на экране и сохраняется в файл через `tee`.

---

## 3. Чем `source` отличается от запуска новой программы

- `source script` выполняет скрипт **в текущем shell**. Переменные окружения, установленные скриптом, **сохраняются** в текущем терминале. Именно поэтому `source /opt/ros/jazzy/setup.bash` делает `ros2` доступным в этом же терминале.
- Запуск программы (например, `./script.sh` или `bash script.sh`) создаёт **новый процесс** с собственной копией окружения. Переменные, заданные внутри, **не влияют** на родительский shell после завершения программы.

---

## 4. Запуск launch, проверка графа и остановка

### Запуск (терминал A)

```bash
source /opt/ros/jazzy/setup.bash
source install/setup.bash
export ROS_DOMAIN_ID=16
ros2 pkg prefix turtle_bringup
ls "$(ros2 pkg prefix turtle_bringup)/share/turtle_bringup/launch"
ros2 launch turtle_bringup sim.launch.py
```

- Запустилась нода с черепашкой

### Проверка графа (Терминал B)

```bash
ros2 node list --no-daemon --spin-time 2
```

- Вывело /turtlesim

### Остановка Ctrl+C (Терминал A)

- Завершилась нода, и проверка в терминале B ничего не выводит

---

## 5. Связка команды с движением

### Получите тип позы и начальные координаты (Терминал C)

```bash
ros2 interface show geometry_msgs/msg/Twist
ros2 topic type /turtle1/pose
ros2 topic echo /turtle1/pose --once
```

- Вывод:

```bash
# This expresses velocity in free space broken into its linear and angular parts.

Vector3  linear
        float64 x
        float64 y
        float64 z
Vector3  angular
        float64 x
        float64 y
        float64 z

turtlesim/msg/Pose

x: 5.544444561004639
y: 5.544444561004639
theta: 0.0
linear_velocity: 0.0
angular_velocity: 0.0
---
```

### Отправьте одну команду (Терминал B)

```bash
ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist \
  '{linear: {x: 1.0}, angular: {z: 0.5}}'
```

- Вывод:

```bash
publisher: beginning loop
publishing #1: geometry_msgs.msg.Twist(linear=geometry_msgs.msg.Vector3(x=1.0, y=0.0, z=0.0), angular=geometry_msgs.msg.Vector3(x=0.0, y=0.0, z=0.5))
```

### Повторно получите позу (Терминал C)

```bash
ros2 topic echo /turtle1/pose --once
```

- Вывод:

```bash
x: 6.509308815002441
y: 5.796990871429443
theta: 0.5040000081062317
linear_velocity: 0.0
angular_velocity: 0.0
---
```

## 6. Ошибитесь в имени и докажите исправление

### Запуск издателя с другим именем топика (Терминал B)

```bash
ros2 topic pub --rate 1 --wait-matching-subscriptions 0 \
  /cmd_vel geometry_msgs/msg/Twist \
  '{linear: {x: 1.0}, angular: {z: 0.5}}'
```

Вывод:

```bash
publisher: beginning loop
publishing #1: geometry_msgs.msg.Twist(linear=geometry_msgs.msg.Vector3(x=1.0, y=0.0, z=0.0), angular=geometry_msgs.msg.Vector3(x=0.0, y=0.0, z=0.5))

publishing #2: geometry_msgs.msg.Twist(linear=geometry_msgs.msg.Vector3(x=1.0, y=0.0, z=0.0), angular=geometry_msgs.msg.Vector3(x=0.0, y=0.0, z=0.5))

...
```

### Проверка, пока издатель работает (Терминал C)

```bash
ros2 topic info /cmd_vel --verbose
ros2 topic info /turtle1/cmd_vel --verbose
```

- Вывод:

```bash
Type: geometry_msgs/msg/Twist

Publisher count: 1

Node name: _ros2cli_21845
Node namespace: /
Topic type: geometry_msgs/msg/Twist
Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
Endpoint type: PUBLISHER
GID: 01.0f.c8.4f.55.55.9a.e2.00.00.00.00.00.00.07.03
QoS profile:
  Reliability: RELIABLE
  History (Depth): UNKNOWN
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Subscription count: 0
```

```bash
Type: geometry_msgs/msg/Twist

Publisher count: 0

Subscription count: 1

Node name: turtlesim
Node namespace: /
Topic type: geometry_msgs/msg/Twist
Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
Endpoint type: SUBSCRIPTION
GID: 01.0f.c8.4f.b9.4e.3f.44.00.00.00.00.00.00.1d.04
QoS profile:
  Reliability: RELIABLE
  History (Depth): UNKNOWN
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite
```

### После исправления /cmd_vel на /turtle1/cmd_vel (Повторный запуск в терминале B)

- Проверка в терминале C (ros2 topic info /turtle1/cmd_vel --verbose):

```bash
Type: geometry_msgs/msg/Twist

Publisher count: 1

Node name: _ros2cli_22141
Node namespace: /
Topic type: geometry_msgs/msg/Twist
Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
Endpoint type: PUBLISHER
GID: 01.0f.c8.4f.7d.56.37.36.00.00.00.00.00.00.07.03
QoS profile:
  Reliability: RELIABLE
  History (Depth): UNKNOWN
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite

Subscription count: 1

Node name: turtlesim
Node namespace: /
Topic type: geometry_msgs/msg/Twist
Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
Endpoint type: SUBSCRIPTION
GID: 01.0f.c8.4f.b9.4e.3f.44.00.00.00.00.00.00.1d.04
QoS profile:
  Reliability: RELIABLE
  History (Depth): UNKNOWN
  Durability: VOLATILE
  Lifespan: Infinite
  Deadline: Infinite
  Liveliness: AUTOMATIC
  Liveliness lease duration: Infinite
```

## 7. Сравнение "до / сбой / после"

- До
До сбоя в системе была рабочая связь: turtlesim слушал топик /turtle1/cmd_vel, и когда в этот топик приходила команда Twist, черепашка двигалась. То есть имя топика и тип сообщения совпадали с тем, что ожидала нода.

- Сбой
Во время сбоя издатель публиковал сообщения правильного типа geometry_msgs/msg/Twist, но в топик с неправильным именем — /cmd_vel. Проверка показала: у /cmd_vel был один издатель и ноль подписчиков, а у /turtle1/cmd_vel — ноль издателей и один подписчик (turtlesim). Команды уходили в пустоту: тип сообщения был верным, но имя топика не совпадало с тем, который слушает черепашка, поэтому движения не происходило.

- После
После исправления имя топика изменили на /turtle1/cmd_vel. Проверка показала: теперь у этого топика один издатель и один подписчик (turtlesim). Команды начали доходить, и черепашка стала двигаться. После остановки издателя черепашка остановилась, потому что новых команд больше не поступало.

- Вывод
Правильного типа сообщения недостаточно. Тип Twist был одинаковым и при сбое, и после исправления, но связь работала только тогда, когда совпало имя топика. Имя топика — такая же часть контракта связи, как и тип сообщения: подписчик слушает конкретное имя, и сообщение с верным типом, но в другое имя, до него не дойдёт.