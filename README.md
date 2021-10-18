# drone-development
Содержит инструкции по быстрому развертыванию симулятора и зависимостей для программирования дронов на python

### Описание
Данный проект предназначен для быстрого старта программирования дронов на python. Здесь рассказывается как:
* установить контейнер с ROS для PX4;
* установить и запустить [PX4 SITL](https://docs.px4.io/master/en/simulation/);

* установить библиотеку dronekit и управлять дроном через нее.


### Подготовка 

На компьютере предварительно должны быть установлены git и docker (см. [здесь](https://docs.docker.com/engine/install/)).

1. Загружаем docker-образ, содержащий [ROS-Melodic](http://wiki.ros.org/melodic), [Gazebo](https://www.gazebosim.org/) и зависимости PX4
```bash
docker pull px4io/px4-dev-ros-melodic
```
2. Скачиваем [PX4 SITL](https://github.com/PX4/PX4-Autopilot)
```bash
mkdir src
cd src
git clone https://github.com/PX4/PX4-Autopilot.git
cd ..
```

3. Запускаем контейнер

<details>
<summary> Инструкция для Linux</summary>

Для Linux надо разрешить подключения к X Server для вывода графической информации:  
```bash
xhost+
```  
и запустить контейнер с параметрами:  
```bash
docker run -it --privileged \
--gpus=all \
--runtime=nvidia \
--tmpfs=/tmp \
-v /tmp/.X11-unix:/tmp/.X11-unix:ro \
-e DISPLAY=:0 \
-e QT_X11_NO_MITSHM=1 \
-e USE_NVIDIA=true \
-e NVIDIA_VISIBLE_DEVICES=${NVIDIA_VISIBLE_DEVICES:-all} \
-e NVIDIA_DRIVER_CAPABILITIES=${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics \
-v `pwd`/src/PX4-Autopilot:/src/PX4-Autopilot \
-p 14570:14570/udp \
--name=mycontainer px4io/px4-dev-ros-melodic:latest bash
```  
</details>

<details>
<summary> Инструкция для Windows</summary>

На Windows ситуация другая. Для запуска GUI приложений надо скачать, установить и запустить программу [
VcXsrv Windows X Server](https://sourceforge.net/projects/vcxsrv/).  

После чего запустить docker-контейнер с параметрами:  

```bash  
docker run -it --privileged \
-e DISPLAY={DISPLAY_IP} \
-v `pwd`/src/PX4-Autopilot:/src/PX4-Autopilot \
-p 14570:14570/udp \
--name=mycontainer px4io/px4-dev-ros-melodic:latest bash
```  
где вместо ```{DISPLAY_IP}```  надо указать IP-адрес подключения к дисплею из программы VcXsrv Windows X Server.  Его можно получить внутри контейнера командой (способ описан [здесь](https://github.com/microsoft/WSL/issues/6430)):  
```bash  
grep nameserver /etc/resolv.conf
```  
Пример вывода:  
```bash  
nameserver 127.0.1.1
```  
Тогда при запуске контейнера указывается параметр вида:  
```bash  
-e DISPLAY=127.0.1.1:0 \
```

```-v `pwd`/src/PX4-Autopilot:/src/PX4-Autopilot``` - означает, что мы подключаем папку ``` `pwd`/src/PX4-Autopilot``` как том и в контейнере она будет располагаться по пути ```/src/PX4-Autopilot```.  

Параметр ```--name``` определяет имя контейнера. По умолчанию задается ```mycontainer```, но можно задать и свое.  
</details>  

4. Загрузка подмодулей PX4
После запуска контейнера в PX4 надо догрузить содержимое подмодулей.  
```bash
cd /src/PX4-Autopilot
git submodule update --init --recursive
```

Проверить, что все успешно работает можно, запустив симулятор px4 вместе с gazebo:
```bash
cd /src/PX4-Autopilot
make px4_sitl gazebo
```  
Если все ОК, то должно открыться окно симулятора вместе с дроном.

5. Установка [dronekit](https://dronekit-python.readthedocs.io/en/latest/) в контейнер

```bash
pip install dronekit
```

6. Установка nano  
nano - это консольный текстовый редактор. Он устанавливается так:  
```bash
apt-get update
apt-get install nano
```

7. Если все вышеописанные шаги прошли успешно, то у Вас получился контейнер для разработки ПО для дронов на python. Выходите из контейнера:  
```bash
exit
```

### Запуск и работа
Запуск контейнера:  
```bash
docker start mycontainer
```

Подключение к контейнеру: 
```bash
docker attach mycontainer
```

<details>  
<summary><i>Пример запуска программы на python с dronekit</i></summary>

 
1. Запускаем PX4 SITL и симулятор gazebo:  
```bash
cd /src/PX4-Autopilot
make px4_sitl gazebo
```  
2. Открывает новую консоль и подключаемся к контейнеру:  
```bash  
docker attach mycontainer
```  
3. Создаем python-файл:  
```bash
cd /src
touch example.py
chmod a+x example.py
```  
4. Копируем в него код из [туториала PX4](https://docs.px4.io/master/en/robotics/dronekit.html)  
Для этого открывает файл через nano:  
```bash  
nano example.py
```  
Вставляем код, сохраняем изменения (```Ctrl+O```) и закрываем файл (```Ctrl+X```).  

5. Запускаем код:  
```bash  
python example.py
```  
6. Наслаждаемся полетом дрона по точкам:)  

</details>  

**Примечание.** Контейнер не удалять!!! Иначе придется заново устанавливать dronekit и nano.

### Устранение неполадок

<details>  
<summary><b>Ошибка при запуске симулятора JMAVSim</b></summary>


Текст ошибки:  
```java  
Exception in thread "main" java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.eclipse.jdt.internal.jarinjarloader.JarRsrcLoader.main(JarRsrcLoader.java:61)
Caused by: java.awt.AWTError: Assistive Technology not found: org.GNOME.Accessibility.AtkWrapper
	at java.awt.Toolkit.loadAssistiveTechnologies(Toolkit.java:807)
	at java.awt.Toolkit.getDefaultToolkit(Toolkit.java:886)
	at java.awt.Window.getToolkit(Window.java:1358)
	at java.awt.Window.init(Window.java:506)
	at java.awt.Window.<init>(Window.java:537)
	at java.awt.Frame.<init>(Frame.java:420)
	at java.awt.Frame.<init>(Frame.java:385)
	at javax.swing.JFrame.<init>(JFrame.java:189)
	at me.drton.jmavsim.Visualizer3D.<init>(Visualizer3D.java:107)
	at me.drton.jmavsim.Simulator.<init>(Simulator.java:192)
	at me.drton.jmavsim.Simulator.main(Simulator.java:941)
	... 5 more
```  

**Решение**:  
В контейнере в файле */etc/java-8-openjdk/accessibility.properties* надо закомментировать строчку *assistive_technologies=org.GNOME.Accessibility.AtkWrapper*.  
</details>  



### Полезные ссылки
* [Использование PX4 с docker-контейнерами](https://docs.px4.io/master/en/test_and_ci/docker.html)  
* [Симулятор PX4](https://docs.px4.io/master/en/simulation/)
* [Dronekit вместе с PX4](https://docs.px4.io/master/en/robotics/dronekit.html)  
* [Запуск симулятора черех mavros (для работы с ROS)](https://docs.px4.io/master/en/simulation/ros_interface.html)
