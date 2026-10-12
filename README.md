# Робототехника

---

## ПР01. Окружение и граф ROS 2

### Цель работы
Запустить готовую систему, увидеть обмен сообщениями, сохранить состав графа и версии окружения. Затем разорвать связь разными доменами и доказать её восстановление тем же наблюдением.

---

### Что сделать

#### 1. Подготовка терминалов
Откройте три Bash-терминала в одной среде. В каждом подключите свою дистрибуцию:
```bash
source /opt/ros/jazzy/setup.bash
export ROS_DOMAIN_ID=16
```

#### 2. Запуск и описание исправного графа
- **Терминал A (симулятор):**
  ```bash
  ros2 run turtlesim turtlesim_node
  ```
- **Терминал B (управление):**
  ```bash
  ros2 run turtlesim turtle_teleop_key
  ```
- **Терминал C (наблюдение):**
  ```bash
  ros2 node list --no-daemon --spin-time 2
  ros2 topic list -t
  ros2 node info /turtlesim
  ros2 topic type /turtle1/pose
  ros2 topic echo /turtle1/pose --once
  ros2 topic hz /turtle1/pose
  ```

#### 3. Разорвать связь (изоляция доменов)
- В **Терминале B** перезапуск teleop в домене 17:
  ```bash
  export ROS_DOMAIN_ID=17
  ros2 run turtlesim turtle_teleop_key
  ```
- В **Терминале C** попытка чтения позы в домене 17 (ожидается таймаут 5 с, код завершения 124):
  ```bash
  export ROS_DOMAIN_ID=17
  timeout 5s ros2 topic echo /turtle1/pose turtlesim/msg/Pose --once
  ```

#### 4. Восстановить связь
- В **Терминале B** возврат в домен 16:
  ```bash
  export ROS_DOMAIN_ID=16
  ros2 run turtlesim turtle_teleop_key
  ```
- В **Терминале C** чтение позы в домене 16 (поза успешно получена, код 0):
  ```bash
  export ROS_DOMAIN_ID=16
  timeout 5s ros2 topic echo /turtle1/pose turtlesim/msg/Pose --once
  ```

---

### Автоматическая проверка
Проверка соответствия формату сдачи:
```bash
python3 .course-kit/v1/tools/check_practice.py PR01 --submission .
```
