# Подглава 10.9 OpenNI и Skeleton Tracking

Возможно, самым ранним и самым известным робототехническим приложением, использующим глубинную камеру, является отслеживание скелета. Пакет ROS [openni\_tracker](https://wiki.ros.org/openni_tracker) может использовать данные глубины из Kinect или Asus Xtion для отслеживания положения суставов человека, стоящего перед камерой. Используя эти данные, можно запрограммировать робота на выполнение жестовых команд, сигнализируемых пользователем. Один пример того, как сделать это с помощью ROS и Python, можно найти в пакете [pi\_tracker](http://wiki.ros.org/pi_tracker). 

Хотя мы не будем вдаваться в подробности использования отслеживания скелета, давайте хотя бы взглянем на основы.

_**10.9.1 Установка NITE и openni\_tracker для ROS Indigo**_ 

На момент написания этой статьи ни двоичные файлы NITE, ни ROS-пакет openni\_tracker не были доступны в качестве пакета Debian для ROS Indigo, поэтому нам необходимо установить их вручную. 

Чтобы установить двоичные файлы NITE, выполните следующие инструкции: 

1. Загрузите двоичный пакет NiTE v1.5.2.23 по одной из следующих ссылок в зависимости от того, используете ли вы 32-разрядную или 64-разрядную установку Ubuntu:

a. 32-bit: [http://www.openni.ru/wp-content/uploads/2013/10/NITE-Bin-Linuxx86-v1.5.2.23.tar.zip](http://www.openni.ru/wp-content/uploads/2013/10/NITE-Bin-Linuxx86-v1.5.2.23.tar.zip) 

b. 64-bit: [http://www.openni.ru/wp-content/uploads/2013/10/NITE-Bin-Linuxx64-v1.5.2.23.tar.zip](http://www.openni.ru/wp-content/uploads/2013/10/NITE-Bin-Linuxx64-v1.5.2.23.tar.zip)

2. Разархивируйте и распакуйте архив в любое место по вашему выбору \(например, ~/tmp\). 

3. Распакованный архив - это фактически другой архив в формате bz2, поэтому разархивируйте и извлеките архив в то же место, что и на шаге 2.

4. Полученная папка должна называться NITE-Bin-Dev-Linux-x64-v1.5.2.23 \(64-разрядная версия\) или NITE-Bin-Linux-x86-v1.5.2.23 \(32-разрядная версия\). Перейдите в эту папку и запустите сценарий uninstall.sh, а затем сценарии install.sh. Для 64-битной версии это будет выглядеть следующим образом:

`$ cd ~/tmp/NITE-Bin-Dev-Linux-x64-v1.5.2.23` 

`$ sudo ./uninstall.sh` 

`$ sudo ./install.sh`

5. Чтобы завершить установку NITE, выполните команду:

`$ sudo niLicense -l 0KOIk2JeIBYClPWVnMoRKn5cdY4=`

Это должно сделать это. Теперь мы готовы установить пакет openni\_tracker следующим образом:

`$ cd ~/catkin_ws/src` 

`$ git clone` [`https://github.com/ros-drivers/openni_tracker.git`](https://github.com/ros-drivers/openni_tracker.git) ``

`$ cd ~/catkin_ws` 

`$ catkin_make` 

`$ rospack profile`

_**10.9.2 Просмотр скелетов в RViz**_ 

Пакет ROS [openni\_tracker](https://wiki.ros.org/openni_tracker) подключается к устройству PrimeSense, такому как Kinect или Asus Xtion, и транслирует преобразование кадра ROS для каждого скелета, обнаруженного перед камерой. Tf-преобразования определяются относительно openni\_depth\_frame, который встроен в камеру за датчиком глубины. Чтобы просмотреть каркасные рамки в RViz, выполните следующие действия. Во-первых, подключите камеру Kinect или Asus, а в случае с Kinect убедитесь, что она также имеет питание. Обязательно прекратите запуск любых файлов openni, которые у вас уже есть. Затем выполните команду openni\_tracker:

`$ rosrun openni_tracker openni_tracker`

\(Не беспокойтесь, если вы не видите начального вывода этой команды. Сообщения о состоянии появятся позже, когда вы будете стоять перед камерой и ваши суставы отслеживаются.\) 

Теперь запустите RViz с включенным файлом конфигурации skeleton\_frames.rviz:

`$ rosrun rviz rviz -d rospack find rbx1_vision/skeleton_frames.rviz`

Следите за RViz и отойдите от камеры как минимум на 5 или 6 футов, принимая «Psi pose». \(cм. cтраницу [openni\_tracker](https://wiki.ros.org/openni_tracker) для примера позы Psi.\) Как только трекер зафиксируется на вас, вы должны увидеть, что ваши скелетные кадры TF появляются в RViz, как показано ниже.

![](.gitbook/assets/image%20%287%29.png)

В этот момент вы можете передвигаться как угодно перед камерой, и скелет в RViz должен имитировать ваши действия. 

_**10.9.3 Доступ к каркасным фреймам в ваших программах**_ 

Так как узел openni\_tracker делает скелетные соединения доступными как ROS tf-кадры, мы можем использовать tf TransformListener, чтобы найти текущую позицию данного соединения. Пример того, как это работает, можно найти в пакете [skeleton\_markers](http://wiki.ros.org/skeleton_markers). Вы можете установить его в свой личный каталог ROS catkin, используя следующие команды:

`$ cd ~/catkin_ws/src` 

`$ git clone -b` [`https://github.com/pirobot/skeleton_markers.git`](https://github.com/pirobot/skeleton_markers.git) ``

`$ cd skeleton_markers` 

`$ git checkout indigo-devel` 

`$ cd ~/catkin_ws` 

`$ catkin_make` 

`$ rospack profile`

Давайте попробуем это, прежде чем смотреть на код. Сначала завершите все экземпляры openni\_tracker и RViz, которые вы, возможно, запускали в предыдущем разделе. Затем выполните следующие две команды:

`$ roslaunch skeleton_markers markers_from_tf.launch`

и

`$ rosrun rviz rviz -d rospack find \ skeleton_markers/markers_from_tf.rviz`

Теперь примите «Psi Pose» перед камерой, следя за RViz, пока калибровка не будет завершена и не начнется отслеживание. Как только трекер закрепится на вас, вы должны увидеть зеленые маркеры скелета в RViz. В этот момент вы можете передвигаться как угодно перед камерой, а скелет в RViz должен следовать вашим действиям, как показано на рисунке ниже:

![](.gitbook/assets/image%20%285%29.png)

Прежде чем мы рассмотрим код, давайте удостоверимся, что понимаем файл запуска markers\_from\_tf.launch, указанный ниже:

```text
<launch>
  <node pkg="openni_tracker" name="openni_tracker" type="openni_tracker" output="screen">
      <param name="fixed_frame" value="openni_depth_frame" />
  </node>
           
  <node pkg="skeleton_markers" name="markers_from_tf" type="markers_from_tf.py" output="screen">
      <rosparam file="$(find skeleton_markers)/params/marker_params.yaml" command="load" />
      
  </node>
</launch>
```

Сначала мы запускаем узел openni\_tracker с фиксированным кадром, установленным на глубину кадра камеры. \(Это строго по умолчанию, так что указывать параметр в файле запуска не обязательно.\) Затем мы запускаем наш скрипт markers\_from\_tf.py и загружаем файл параметров marker\_params.yaml из каталога params. Этот файл определяет параметры, описывающие внешний вид маркеров, а также список каркасных фреймов, которые мы хотим отслеживать. 

Давайте теперь посмотрим на скрипт markers\_from\_tf.py. Наша общая стратегия будет заключаться в использовании библиотеки tf для нахождения преобразования между каждым каркасным кадром и кадром с фиксированной глубиной. Все, что нам нужно для наших целей - это координаты начала каждого кадра относительно кадра глубины. Это позволяет нам разместить маркер визуализации в этом месте, чтобы представить положение соответствующего скелетного сустава в пространстве. 

Ссылка на источник: [markers\_from\_tf.py](https://github.com/pirobot/skeleton_markers/blob/indigo-devel/nodes/markers_from_tf.py)

```text
#!/usr/bin/env python

import rospy

from visualization_msgs.msg import Marker
from geometry_msgs.msg import Point
import tf

class SkeletonMarkers():
    def __init__(self):
        rospy.init_node('markers_from_tf')
                
        rospy.loginfo("Initializing Skeleton Markers Node...")
        
        rate = rospy.get_param('~rate', 20)
        r = rospy.Rate(rate)
        
        # There is usually no need to change the fixed frame from the default
        self.fixed_frame = rospy.get_param('~fixed_frame', 'openni_depth_frame')
        
        # Get the list of skeleton frames we want to track
        self.skeleton_frames = rospy.get_param('~skeleton_frames', '')

        # Initialize the tf listener
        tf_listener = tf.TransformListener()
        
        # Define a marker publisher
        marker_pub = rospy.Publisher('skeleton_markers', Marker, queue_size=5)
        
        # Intialize the markers
        self.initialize_markers()
        
        # Make sure we see the openni_depth_frame
        tf_listener.waitForTransform(self.fixed_frame, self.fixed_frame, rospy.Time(), rospy.Duration(60.0))
        
        # A flag to track when we have detected a skeleton
        skeleton_detected = False
        
        # Begin the main loop
        while not rospy.is_shutdown():
            # Set the markers header
            self.markers.header.stamp = rospy.Time.now()
                        
            # Clear the markers point list
            self.markers.points = list()
            
            # Check to see if a skeleton is detected
            while not skeleton_detected: 
                # Assume we can at least see the head frame               
                frames = [f for f in tf_listener.getFrameStrings() if f.startswith('head_')]
                
                try:
                    # If the head frame is visible, pluck off the
                    # user index from the name
                    head_frame = frames[0]
                    user_index = head_frame.replace('head_', '')
                    
                    # Make sure we have a transform between the head
                    # and the fixed frame
                    try:
                        (trans, rot)  = tf_listener.lookupTransform(self.fixed_frame, head_frame, rospy.Time(0))
                        skeleton_detected = True

                    except (tf.Exception, tf.ConnectivityException, tf.LookupException):
                        skeleton_detected = False
                        rospy.loginfo("User index: " + str(user_index))
                        r.sleep()
                except:
                    skeleton_detected = False
            
            # Loop through the skeleton frames
            for frame in self.skeleton_frames:
                # Append the user_index to the frame name
                skel_frame = frame + "_" + str(user_index)

                # We only need the origin of each skeleton frame
                # relative to the fixed frame
                position = Point()
                                        
                # Get the transformation from the fixed frame
                # to the skeleton frame
                try:
                    (trans, rot)  = tf_listener.lookupTransform(self.fixed_frame, skel_frame, rospy.Time(0))
                    position.x = trans[0]
                    position.y = trans[1]
                    position.z = trans[2]
                                                            
                    # Set a marker at the origin of this frame
                    self.markers.points.append(position)
                except:
                    pass
                
            # Publish the set of markers
            marker_pub.publish(self.markers)
                              
            r.sleep()
            
    def initialize_markers(self):
        # Set various parameters
        scale = rospy.get_param('~scale', 0.07)
        lifetime = rospy.get_param('~lifetime', 0) # 0 is forever
        ns = rospy.get_param('~ns', 'skeleton_markers')
        id = rospy.get_param('~id', 0)
        color = rospy.get_param('~color', {'r': 0.0, 'g': 1.0, 'b': 0.0, 'a': 1.0})
        
        # Initialize the marker points list
        self.markers = Marker()
        self.markers.header.frame_id = self.fixed_frame
        self.markers.ns = ns
        self.markers.id = id
        self.markers.type = Marker.POINTS
        self.markers.action = Marker.ADD
        self.markers.lifetime = rospy.Duration(lifetime)
        self.markers.scale.x = scale
        self.markers.scale.y = scale
        self.markers.color.r = color['r']
        self.markers.color.g = color['g']
        self.markers.color.b = color['b']
        self.markers.color.a = color['a']
        
if __name__ == '__main__':
    try:
        SkeletonMarkers()
    except rospy.ROSInterruptException:
        pass
        
```

Давайте рассмотрим ключевые строки:

`18 # There is usually no need to change the fixed frame from the default` 

`19 self.fixed_frame = rospy.get_param('~fixed_frame', 'openni_depth_frame')` 

`20` 

`21 # Get the list of skeleton frames we want to track` 

`22 self.skeleton_frames = rospy.get_param('~skeleton_frames', '')`

Здесь мы устанавливаем фиксированный кадр по умолчанию, используемый узлом openni\_tracker. Мы также читаем в списке каркасных фреймов, которые мы хотим отслеживать, как указано в файле параметров marker\_params.yaml. Файл параметров выглядит следующим образом:

```text
# The fixed reference frame 
fixed_frame: 'openni_depth_frame' 

# Update rate 
rate: 20 

# Height and width of markers in meters 
scale: 0.07 

# Duration of markers in RViz; 0 is forever 
lifetime: 0 

# Marker namespace 
ns: 'skeleton_markers' 

# Marker id 
id: 0 

# Marker color 
color: { 'r': 0.0, 'g': 1.0, 'b': 0.0, 'a': 1.0 } 

skeleton_frames: [ 
head, 
neck, 
torso, 
left_shoulder, 
left_elbow, 
left_hand, 
left_hip, 
left_knee, 
left_foot, 
right_shoulder, 
right_elbow, 
right_hand, 
right_hip, 
right_knee, 
right_foot 
]

```

Файл параметров используется для определения частоты обновления, появления маркеров для отображения в RViz и списка кадров скелета, которые мы хотим отслеживать. 

Возвращаясь к сценарию markers\_from\_tf.py:

`24 # Initialize the tf listener` 

`25 tf_listener = tf.TransformListener()` 

`26` 

`27 # Define a marker publisher` 

`28 marker_pub = rospy.Publisher('skeleton_markers', Marker, queue_size=5)`

Здесь мы создаем TransformListener из библиотеки ROS tf и настраиваем издателя для маркеров визуализации.

`31 self.initialize_markers()`

Эта строка вызывает функцию \(определенную позже в скрипте\) для инициализации маркеров. Мы не будем обсуждать маркеры в этом томе, но вы, вероятно, можете довольно легко следовать функции инициализации. Вы также можете посмотреть [учебное пособие по маркерам](https://wiki.ros.org/rviz/Tutorials/Markers:%20Basic%20Shapes) в ROS Wiki, хотя примеры приведены на C++.

`34 tf_listener.waitForTransform(self.fixed_frame, self.fixed_frame, rospy.Time(), rospy.Duration(60.0))`

Прежде чем мы начнем искать каркасные кадры, мы должны убедиться, что, по крайней мере, видим фиксированный кадр камеры, и у нас есть 60 секунд до истечения времени ожидания.

`37 skeleton_detected = False`

Флаг, указывающий, виден ли скелет.

`39 # Begin the main loop`

`40 while not rospy.is_shutdown():`

 `41 # Set the markers header` 

`42 self.markers.header.stamp = rospy.Time.now()` 

`43` 

`44 # Clear the markers point list` 

`40 self.markers.points = list()`

Теперь мы входим в основной цикл скрипта. Сначала мы отметим время в списке маркеров и очистим все координаты маркера.

`48 while not skeletondetected:` 

`49 # Assume we can at least see the head frame` 

`50 frames = [f for f in tf_listener.getFrameStrings() if f.startswith('head')]`

Затем мы используем слушатель tf, чтобы получить список всех доступных кадров и проверить, можем ли мы видеть головной кадр.

`52 try:` 

`53 # If the head frame is visible, pluck off the` 

`54 # user index from the name` 

`55 head_frame = frames[0]`

`56 userindex = head_frame.replace('head', '')` 

`57` 

`58 # Make sure we have a transform between the head 59 # and the fixed frame` 

`60 try:` 

`61 (trans, rot) = tf_listener.lookupTransform(self.fixed_frame, head_frame, rospy.Time(0))` 

`62 skeleton_detected = True` 

`63` 

`64 except (tf.Exception, tf.ConnectivityException, tf.LookupException):` 

`65 skeleton_detected = False` 

`66 rospy.loginfo("User index: " + str(user_index))` 

`67 r.sleep()` 

`68 except:` 

`69 skeleton_detected = False`

Если у нас есть рамка заголовка, внешний блок try-except будет успешным, и мы ищем преобразование между рамкой головы и фиксированной рамкой. Если поиск успешен, мы можем быть уверены, что обнаружили скелет, и мы установили флаг в True, чтобы мы могли выйти из внешнего цикла while.

`72 for frame in self.skeleton frames:` 

`73 # Append the user_index to the frame name` 

`74 skel_frame = frame + "" + str(user_index)` 

`75` 

`76 # We only need the origin of each skeleton frame` 

`77 # relative to the fixed frame` 

`78 position = Point()` 

`79` 

`80 # Get the transformation from the fixed frame` 

`81 # to the skeleton frame` 

`82 try:` 

`83 (trans, rot) = tf_listener.lookupTransform(self.fixed_frame, skel_frame, rospy.Time(0))` 

`84 position.x = trans[0] 85 position.y = trans[1]` 

`86 position.z = trans[2]` 

`87` 

`88 # Set a marker at the origin of this frame` 

`89 self.markers.points.append(position)` 

`90 except:` 

`91 pass`

Теперь мы можем перебрать все каркасные кадры и попытаться найти преобразование между каждым кадром и фиксированным кадром. Если поиск выполнен успешно, перевод и ротация возвращаются отдельно. Мы заботимся только о компоненте перевода \(расположении источника кадра относительно фиксированного кадра\), поэтому мы храним компоненты x, y и z в переменной position, которую мы инициализировали ранее как тип Point. Этот пункт затем добавляется в список маркеров.

`94 marker_pub.publish(self.markers)` 

`95`

`77 r.sleep()`

Как только у нас есть маркер для каждого кадра, мы публикуем весь набор маркеров и затем ждем один цикл оценки, прежде чем начинать новый цикл.

