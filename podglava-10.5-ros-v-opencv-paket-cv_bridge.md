# Подглава 10.5 ROS в OpenCV: пакет cv\_bridge

После того, как драйверы камер запущены и работают, нам нужен способ обработки видеопотоков ROS с использованием OpenCV. ROS предоставляет пакет [cv\_bridge](http://wiki.ros.org/cv_bridge) для преобразования между ROS и OpenCV и форматами изображений. Скрипт Python cv\_bridge.demo.py в каталоге rbx1\_vision/node демонстрирует, как использовать cv\_bridge. Прежде чем мы посмотрим на код, вы можете попробовать его следующим образом. 

Если у вас Kinect или Xtion, убедитесь, что вы сначала запустили соответствующий драйвер, если он еще не запущен. 

_Для Microsoft Kinect:_

`$ roslaunch freenect_launch freenect-registered-xyzrgb.launch`

_Для камер Asus Xtion, Xtion Pro или Primesense 1.08/1.09:_

`$ roslaunch openni2_launch openni2.launch depth_registration:=true`

или для веб-камеры:

`$ roslaunch rbx1_vision usb_cam.launch video_device:=/dev/video0`

\(Необходимо изменить видеоустройство.\) 

Теперь запустите узел cv\_bridge\_demo.py:

`$ rosrun rbx1_vision cv_bridge_demo.py`

После небольшой задержки вы увидите два окна с изображениями. Окно в верхней части показывает живое видео после того, как оно было преобразовано в оттенки серого, а затем отправлено через OpenCV размытие и краевые фильтры. Окно внизу показывает изображение глубины серого, где белые пиксели находятся дальше, а темно-серые пиксели ближе к камере. \(Это окно останется пустым, если вы используете обычную веб-камеру.\) Чтобы выйти из демонстрации, либо введите букву «q» с помощью мыши над одним из окон, либо нажмите Ctrl-C в терминале, где вы запустили демонстрационный скрипт. 

Давайте теперь посмотрим на код, чтобы увидеть, как он работает.

Ссылка на источник: [cv\_bridge\_demo.py](https://github.com/pirobot/rbx1/blob/indigo-devel/rbx1_vision/nodes/cv_bridge_demo.py)

```text
#!/usr/bin/env python 

import rospy
import sys
import cv2
import cv2.cv as cv
from sensor_msgs.msg import Image, CameraInfo
from cv_bridge import CvBridge, CvBridgeError
import numpy as np

class cvBridgeDemo():
    def __init__(self):
        self.node_name = "cv_bridge_demo"
        
        rospy.init_node(self.node_name)
        
        # What we do during shutdown
        rospy.on_shutdown(self.cleanup)
        
        # Create the OpenCV display window for the RGB image
        self.cv_window_name = self.node_name
        cv.NamedWindow(self.cv_window_name, cv.CV_WINDOW_NORMAL)
        cv.MoveWindow(self.cv_window_name, 25, 75)
        
        # And one for the depth image
        cv.NamedWindow("Depth Image", cv.CV_WINDOW_NORMAL)
        cv.MoveWindow("Depth Image", 25, 350)
        
        # Create the cv_bridge object
        self.bridge = CvBridge()
        
        # Subscribe to the camera image and depth topics and set
        # the appropriate callbacks
        self.image_sub = rospy.Subscriber("input_rgb_image", Image, self.image_callback, queue_size=1)
        self.depth_sub = rospy.Subscriber("input_depth_image", Image, self.depth_callback, queue_size=1)
        
        rospy.loginfo("Waiting for image topics...")
        rospy.wait_for_message("input_rgb_image", Image)
        rospy.loginfo("Ready.")

    def image_callback(self, ros_image):
        # Use cv_bridge() to convert the ROS image to OpenCV format
        try:
            frame = self.bridge.imgmsg_to_cv2(ros_image, "bgr8")
        except CvBridgeError, e:
            print e
        
        # Convert the image to a numpy array since most cv2 functions
        # require numpy arrays.
        frame = np.array(frame, dtype=np.uint8)
        
        # Process the frame using the process_image() function
        display_image = self.process_image(frame)
                       
        # Display the image.
        cv2.imshow(self.node_name, display_image)
        
        # Process any keyboard commands
        self.keystroke = cv2.waitKey(5)
        if self.keystroke != -1:
            cc = chr(self.keystroke & 255).lower()
            if cc == 'q':
                # The user has press the q key, so exit
                rospy.signal_shutdown("User hit q key to quit.")
                
    def depth_callback(self, ros_image):
        # Use cv_bridge() to convert the ROS image to OpenCV format
        try:
            # Convert the depth image using the default passthrough encoding
            depth_image = self.bridge.imgmsg_to_cv2(ros_image, "passthrough")
        except CvBridgeError, e:
            print e

        # Convert the depth image to a Numpy array since most cv2 functions require Numpy arrays.
        depth_array = np.array(depth_image, dtype=np.float32)
                
        # Normalize the depth image to fall between 0 (black) and 1 (white)
        cv2.normalize(depth_array, depth_array, 0, 1, cv2.NORM_MINMAX)
        
        # Process the depth image
        depth_display_image = self.process_depth_image(depth_array)
    
        # Display the result
        cv2.imshow("Depth Image", depth_display_image)
          
    def process_image(self, frame):
        # Convert to greyscale
        grey = cv2.cvtColor(frame, cv.CV_BGR2GRAY)
        
        # Blur the image
        grey = cv2.blur(grey, (7, 7))
        
        # Compute edges using the Canny edge filter
        edges = cv2.Canny(grey, 15.0, 30.0)
        
        return edges
    
    def process_depth_image(self, frame):
        # Just return the raw image for this demo
        return frame
    
    def cleanup(self):
        print "Shutting down vision node."
        cv2.destroyAllWindows()   
    
def main(args):       
    try:
        cvBridgeDemo()
        rospy.spin()
    except KeyboardInterrupt:
        print "Shutting down vision node."
        cv.DestroyAllWindows()

if __name__ == '__main__':
    main(sys.argv)
```

Давайте посмотрим на ключевые строки в этом скрипте.

`5. import cv2` 

`6. import cv2.cv as cv` 

`7. from sensor_msgs.msg import Image, CameraInfo` 

`8. from cv_bridge import CvBridge, CvBridgeError` 

`9. import numpy as np`

Все наши скрипты OpenCV будут импортировать библиотеку cv2, а также более старые функции pre-cv2, найденные в cv2.cv. Нам также обычно нужны типы сообщений ROS Image и CameraInfo из пакета sensor\_msgs. Для узла, которому необходимо преобразовать формат изображения ROS в OpenCV, нам нужны классы CvBridge и CvBridgeError из пакета ROS cv\_bridge. Наконец, OpenCV выполняет большую часть своей обработки изображений с использованием массивов Numpy, поэтому нам почти всегда нужен доступ к модулю Python numpy.

`20.        # Create the OpenCV display window for the RGB image` 

`21.        self.cv_window_name = self.node_name` 

`22.        cv.NamedWindow(self.cv_window_name, cv.CV_WINDOW_NORMAL)` 

`23.        cv.MoveWindow(self.cv_window_name, 25, 75)` 

`24.`       

 `25.        # And one for the depth image` 

`26.        cv.NamedWindow("Depth Image", cv.CV_WINDOW_NORMAL)` 

`27.        cv.MoveWindow("Depth Image", 25, 350)`

Если вы уже знакомы с OpenCV, вы узнаете эти операторы, которые создают именованные окна отображения для мониторинга видеопотоков; в этом случае одно окно для обычного видео RGB и одно для изображения глубины, если мы используем камеру RGB-D. Мы также перемещаем окна так, чтобы окно глубины находилось ниже окна RGB.

`30.         self.bridge = CvBridge()`

Вот как мы создаем объект CvBridge для последующего использования для преобразования изображений ROS в формат OpenCV.

`34.         self.image_sub = rospy.Subscriber("/camera/rgb/image_color",` 

`35.                            Image, self.image_callback)` 

`36.        self.depth_sub = rospy.Subscriber("/camera/depth_registered/image_rect",` 

`37.                            Image, self.depth_callback)`

Это два ключевых подписчика, один для потока изображений RGB и один для изображения глубины. Обычно мы не будем жестко кодировать названия тем, чтобы их можно было переназначить в соответствующий файл запуска, но для демонстрации мы будем использовать имена тем по умолчанию, используемые узлом openni. Как и для всех подписчиков, мы назначаем функции обратного вызова для выполнения реальной работы с изображениями.

`41.    def image_callback(self, ros_image):` 

`42.        # Use cv_bridge() to convert the ROS image to OpenCV format` 

`43.        try:`

`44.            frame = self.bridge.imgmsg_to_cv2(ros_image, "bgr8")` 

`45.        except CvBridgeError, e:` 

`46.            print e`

Это начало нашей функции обратного вызова для изображения RGB. ROS предоставляет изображение в качестве первого аргумента, который мы назвали ros\_image. Затем блок try-exc использует функцию imgmsg\_to\_cv2 для преобразования изображения в формат OpenCV, предполагая сине-зеленое / красное 8-битное преобразование.

`50. frame = np.array(frame, dtype=np.uint8)` 

`51.`        

`52.        # Process the frame using the process_image() function` 

`53.        display_image = self.process_image(frame)` 

`54.`                       

`55.        # Display the image.` 

`56.        cv2.imshow(self.node_name, display_image`

Большинство функций OpenCV требуют преобразования изображения в массив Numpy, поэтому мы выполняем преобразование здесь. Затем мы отправляем массив изображений в функцию process\_image \(\), которую мы опишем ниже. Результатом является новое изображение с именем display\_image, которое отображается в нашем ранее созданном окне отображения с помощью функции OpenCV imshow \(\).

`58.        # Process any keyboard commands` 

`59.        self.keystroke = cv.WaitKey(5)` 

`60.        if 32 <= self.keystroke and self.keystroke < 128:` 

`61.            cc = chr(self.keystroke).lower()` 

`62.            if cc == 'q':` 

`63.                # The user has press the q key, so exit`

`64.                rospy.signal_shutdown("User hit q key to quit.")`

Наконец, мы ищем ввод с клавиатуры от пользователя. В этом случае мы ищем только нажатие клавиши «q», которая сигнализирует о том, что мы хотим завершить сценарий.

`76.        depth_array = np.array(depth_image, dtype=np.float32)` 

`77.`                

`78.        # Normalize the depth image to fall between 0 and 1`

`79.        cv2.normalize(depth_array, depth_array, 0, 1, cv2.NORM_MINMAX)`

Поскольку OpenCV хранит изображения в виде массивов Numpy, мы преобразуем необработанное изображение глубины в массив float32. Затем мы нормализуем массив к интервалу \[0, 1\], поскольку функция OpenCV imshow \(\) может отображать изображения в градациях серого, когда значения пикселей лежат между 0 и 1.

`82.         depth_display_image = self.process_depth_image(depth_array)` 

`83.`    

`84.        # Display the result` 

`85.        cv2.imshow("Depth Image", depth_display_image)`

Массив глубины отправляется в функцию process\_depth\_image \(\) для дальнейшей обработки \(при желании\), и результат отображается в окне изображения глубины, созданном ранее.

`87.     def process_image(self, frame):` 

`88.        # Convert to grayscale` 

`89.        grey = cv2.cvtColor(frame, cv.CV_BGR2GRAY)` 

`90.`        

`91.        # Blur the image` 

`92.        grey = cv2.blur(grey, (7, 7))` 

`93.`        

`94.        # Compute edges using the Canny edge filter` 

`95.        edges = cv2.Canny(grey, 15.0, 30.0)` 

`96.`        

`97.        return edges`

Напомним, что функция обратного вызова изображения, в свою очередь, вызвала функцию process\_image \(\) на тот случай, если мы хотим манипулировать изображением перед его отображением пользователю. Для целей этой демонстрации здесь мы конвертируем изображение в градации серого, размываем его, используя фильтр Gaussian, с дисперсией 7 пикселей в измерениях x и y, а затем вычисляем края Canny по результату. Изображение края возвращается к обратному вызову, где оно отображается пользователю.

`99.     def process_depth_image(self, frame):` 

`100.        # Just return the raw image for this demo` 

`101.        return frame`

В этой демонстрации мы ничего не делаем с изображением глубины и просто возвращаем исходный кадр. Не стесняйтесь добавлять свой собственный набор фильтров OpenCV здесь.

