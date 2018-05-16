---
title: 用pyqt5构建人脸识别界面
date: 2018-03-12 09:00:38
tags:
- pyqt5
- dlib
categories: 工作
---

<font size="5"  color=lightseagreen>  这篇博客是总结之前用Pyqt5、dlib、opencv实现的一个实时人脸识别同时具有人脸录入功能的软件，后续还会增加基于深度学习的人脸识别功能。</font>

<!--more-->



# 一、效果视频

<div style="max-width:960px; margin:0 auto 10px;" >
<div
style="position: relative;
width:100%;
padding-bottom:56.25%;
height:0;">
<iframe style="position: absolute;top: 0;left: 0;width: 100%;height: 100%;"  src="http://player.youku.com/embed/XMzQ2NDg1NDIxNg==" frameborder="0" allowfullscreen></iframe>
</div>
</div>


# 二、源码解析

## 1、UI布局

　　该软件界面只是使用了简单的UI界面，包括按键QPushButton、显示视屏是用的QLabel不断刷新图片来显示视频，录入姓名是用的QTextEdit。
  [参考PyQT教程：](https://zhuanlan.zhihu.com/xdbcb8)
```python
    def set_ui(self):
        # 布局设置
        self.layout_main = QHBoxLayout()  # 整体框架是水平布局
        self.layout_button = QVBoxLayout()  # 按键布局是垂直布局

        # 按钮设置
        self.btn_open_cam = QPushButton('打开相机')
        self.btn_photo = QPushButton('拍照')
        self.btn_input_name = QPushButton('录入人名')
        self.btn_detection_face = QPushButton('人脸检测')
        self.btn_recognition_face = QPushButton('人脸识别')


        self.btn_quit = QPushButton('退出')

        # 显示视频
        self.label_show_camera = QLabel()
        self.label_move = QLabel()
        self.label_move.setFixedSize(100, 200)
        # 显示文本框
        self.text = QTextEdit(self)

        self.label_show_camera.setFixedSize(800, 600)
        self.label_show_camera.setAutoFillBackground(False)

        # 布局
        self.layout_button.addWidget(self.btn_open_cam)
        self.layout_button.addWidget(self.btn_photo)
        self.layout_button.addWidget(self.btn_input_name)
        self.layout_button.addWidget(self.btn_detection_face)
        self.layout_button.addWidget(self.btn_recognition_face)
        self.layout_button.addWidget(self.btn_quit)
        self.layout_button.addWidget(self.label_move)
        self.layout_button.addWidget(self.text)

        self.layout_main.addLayout(self.layout_button)
        self.layout_main.addWidget(self.label_show_camera)

        self.setLayout(self.layout_main)
        self.setGeometry(300, 300, 600, 400)
        self.setWindowTitle("人脸识别软件")
```
## 2、按键函数介绍
　　<font size= 5 color= red >**注意：不同按键切换前需要关闭当前功能。否则操作会出现错误。**</font>
### 1) def btn_open_cam_click(self)
　　打开摄像头函数 btn_open_cam_click(self)，该函数主要是用来打开和关闭摄像头，具体是开启摄像头界面刷新定时器timer_camera.start(30),30ms出发一次信号槽， `self.timer_camera.timeout.connect(self.show_camera)`同时检查摄像头是否正确连接摄像头，如果没有会弹出一个消息提示框。

```python
    def btn_open_cam_click(self):
        if self.timer_camera.isActive() == False:
            flag = self.cap.open(self.cap_num)
            if flag == False:
                msg = QMessageBox.warning(self, u"Warning", u"请检测相机与电脑是否连接正确", buttons=QMessageBox.Ok,
                                          defaultButton=QMessageBox.Ok)
            # if msg==QtGui.QMessageBox.Cancel:
            #                     pass
            else:
                self.timer_camera.start(30)

                self.btn_open_cam.setText(u'关闭相机')
        else:
            self.timer_camera.stop()
            self.cap.release()
            self.label_show_camera.clear()
            self.btn_open_cam.setText(u'打开相机')
```

### 2) def show_camera(self):
　　摄像头显示函数，该函数也相当于主函数了，这里有个很重要的变量`self.btn_flag`该变量用来切换按钮不同的模式，有打开、关闭摄像头、人脸检测、人脸识别等功能。
　　第一是加载人脸级联分类器haarcascade_frontalface_default.xml，该分类器是检测人脸用的分类器。其实该分类器在人脸识别中并未用到。只是在人脸检测中用到。
　　第二是需要用将图片转化才能在QT界面中显示，`self.showImage = QImage(show.data, show.shape[1], show.shape[0], QImage.Format_RGB888)`
　　<font color=red >第三是人脸识别功能要开启线程，这在同时识别多个人时特别关键，特别将是检测不同人脸，并标定不同标签来识别人，放在了不同的线程中。</font>
   
```python
    def show_camera(self):
        harr_filepath = cv2.data.haarcascades + "haarcascade_frontalface_default.xml"  # 系统安装的是opencv-contrib-python
        classifier = cv2.CascadeClassifier(harr_filepath)  # 加载人脸特征分类器
        if self.btn_flag == 0:
            ret, self.image = self.cap.read()
            show = cv2.resize(self.image, (800, 600))
            show = cv2.cvtColor(show, cv2.COLOR_BGR2RGB)  # 这里指的是显示原图
            # opencv 读取图片的样式，不能通过Qlabel进行显示，需要转换为Qimage QImage(uchar * data, int width,
            # int height, Format format, QImageCleanupFunction cleanupFunction = 0, void *cleanupInfo = 0)
            self.showImage = QImage(show.data, show.shape[1], show.shape[0], QImage.Format_RGB888)
            self.label_show_camera.setPixmap(QPixmap.fromImage(self.showImage))
        elif self.btn_flag == 1:  # 人脸检测
            ret_1, self.image_1 = self.cap.read()
            show_0 = cv2.resize(self.image_1, (800, 600))
            show_1 = cv2.cvtColor(show_0, cv2.COLOR_BGR2RGB)
            gray_image = cv2.cvtColor(show_0, cv2.COLOR_BGR2GRAY)
            faces = classifier.detectMultiScale(gray_image, 1.3, 5)  # 1.3和5是特征的最小、最大检测窗口，它改变检测结果也会改变
            for (x, y, w, h) in faces:
                cv2.rectangle(show_1, (x, y), (x + w, y + h), (0, 255, 0), 2)  # 画出人脸
            detect_image = QImage(show_1.data, show_1.shape[1], show_1.shape[0],
                                  QImage.Format_RGB888)
            self.label_show_camera.setPixmap(QPixmap.fromImage(detect_image))

        elif self.btn_flag == 2:  # 人脸识别
            ret_2, self.image_2 = self.cap.read()
            show_2 = cv2.resize(self.image_2, (800, 600))
            self.show_3 = cv2.cvtColor(show_2, cv2.COLOR_BGR2RGB)
            gray_image = cv2.cvtColor(show_2, cv2.COLOR_BGR2GRAY)
            self.dets = self.detector(gray_image, 1)  # 对视频中的人脸进行标定
            self.dist = []  # 声明一个数组
            if not len(self.dets):
                # print('Can`t get face.')
                detect_image = QImage(self.show_3.data, self.show_3.shape[1], self.show_3.shape[0],
                                      QImage.Format_RGB888)
                self.label_show_camera.setPixmap(QPixmap.fromImage(detect_image))
            # 这里开启了一个人脸识别的线程，会自动为for循环分配线程，这里为进一步在同一个视频中，识别不同的人脸准备
            t = threading.Thread(target=self.face_thread, name='Face_Thread')
            t.start()
            t.join()

```
### 3) def face_thread(self):

　　线程函数中特别重要是一点是，
```python
# 计算欧式距离
for i in self.descriptors:  # 遍历之前提取的候选人的128D向量
     # 计算欧式距离，有多少个候选人就有多少个距离,放到dist数组中。
     dist_ = np.linalg.norm(i - d_test)
     self.dist.append(dist_)
     # 候选人和距离组成一个dict字典
     c_d = dict(zip(self.candidate, self.dist))
#注意这里必须把dist[]之前的数据pop出去，才能确保每次不同线程计算的不同人的向量。从而不同线程识别不同的人。
#因为pop()只是把list中的最后一个pop出去，所以需要一个循环。
for i in range(0, len(self.candidate)):
    self.dist.pop()
```

```python
 def face_thread(self):  # 这里多线程是解决，同时识别不同人脸的关键

        print('thread %s is running...' % threading.current_thread().name)
        for k, face in enumerate(self.dets):  # 遍历视频中所有人脸 ，

            print('thread %s >>> %s' % (threading.current_thread().name, k))
            shape = self.landmark(self.show_3, face)  # 检测人脸特征点
            face_descriptor = self.facerec.compute_face_descriptor(self.show_3, shape)  # 计算人脸的128D向量
            d_test = np.array(face_descriptor)
            x1 = face.top() if face.top() > 0 else 0
            y1 = face.bottom() if face.bottom() > 0 else 0
            x2 = face.left() if face.left() > 0 else 0
            y2 = face.right() if face.right() > 0 else 0
            cv2.rectangle(self.show_3, (x2, x1), (y2, y1), (255, 0, 0), 3)
            # print(x2, x1, y2, y1)
            # 计算欧式距离
            for i in self.descriptors:  # 遍历之前提取的候选人的128D向量
                dist_ = np.linalg.norm(i - d_test)  # 计算欧式距离，有多少个候选人就有多少个距离,放到dist数组中。
                self.dist.append(dist_)
                # 候选人和距离组成一个dict字典
            c_d = dict(zip(self.candidate, self.dist))
            for i in range(0, len(self.candidate)):  # 注意这里必须把dist[]之前的数据pop出去，才能确保每次不同线程计算的不同人的向量。
                self.dist.pop()
            # sorted将dict字典中数排序，按key顺序（第二个关键字）
            cd_sorted = sorted(c_d.items(), key=lambda d: d[1])
            if cd_sorted[0][1] > 0.32:
                cv2.putText(self.show_3, 'Unknown', (x2, x1), cv2.FONT_HERSHEY_SIMPLEX, 1.0,
                            (0, 255, 255), 2)
            else:
                cv2.putText(self.show_3, cd_sorted[0][0], (x2, x1), cv2.FONT_HERSHEY_SIMPLEX, 1.0,
                        (0, 255, 255), 2)
            # 各参数依次是：照片/添加的文字/左上角坐标/字体/字体大小/颜色/字体粗细
            print("\n The person is: ", cd_sorted[0][0])
            detect_image = QImage(self.show_3.data, self.show_3.shape[1], self.show_3.shape[0],
                                  QImage.Format_RGB888)
            self.label_show_camera.setPixmap(QPixmap.fromImage(detect_image))
```

### 4) def photo_face(self):
　 拍照函数：以系统当前时间命名。使用的是`datetime.now().strftime("%Y%m%d%H%M%S")`
```python
    def photo_face(self):

        # photo_save_path = '/home/dx/Desktop/detect_face_pyqt5/candidate-faces'

        photo_save_path = os.path.join(os.path.dirname(os.path.abspath('__file__')),
                                       'candidate-faces/')
        # self.time_flag.append(datetime.now().strftime("%Y%m%d%H%M%S"))
        self.showImage.save(photo_save_path + datetime.now().strftime("%Y%m%d%H%M%S") + ".jpg")
        QMessageBox.information(self, "Information",
                                self.tr("拍照成功!"))
```

### 5) def input_name(self):
　　实现功能是录入人名，保存人名到name.txt文件
```python
    def input_name(self):
        name_path = os.path.join(os.path.dirname(os.path.abspath('__file__')),  # 将人名文件放在candidate-faces文件夹下
                                 'candidate-faces/')
        file_name_path = os.path.join(self.faces_folder_path, 'name.txt')
        if self.input_flag == 0:
            self.input_flag = 1
            self.fname = QFileDialog.getOpenFileName(self, '打开文件', name_path, "Text Files(*.txt)")
            if self.fname[0]:  # self.fname[0] 是/home/dx/Desktop/detect_face_pyqt5/candidate-faces/name.txt
                with open(self.fname[0], 'r', encoding='gb18030', errors='ignore') as f:
                    self.text.setText(f.read())
            self.btn_input_name.setText('保存人名')
        elif self.input_flag == 1:
            self.input_flag = 0
            self.cont = self.text.toPlainText()  # 获取文本框内容
            f = open(file_name_path, 'w')
            f.write(self.cont)
            #print(self.cont)
            self.btn_input_name.setText('录入人名')
```

### 6) dlib_para_init(self): def readfile(self):
　
　　该函数作用是加载，人脸特征提取器，人脸识别分类，同时加载候选人文件夹，从name.txt加载候选人信息，并按照时间顺序排序，这里很关键。这样才能把照片和人名有序的联系起来。

```python
# 人脸关键点检测器
        #  face_landmark_path    = '/home/dx/Desktop/detect_face_pyqt5/shape_predictor_68_face_landmarks.dat'
        face_landmark_path = os.path.join(os.path.dirname(os.path.abspath('__file__')),
                                          'shape_predictor_68_face_landmarks.dat')

        # 人脸识别模型
        #  face_recognize_path   = '/home/dx/Desktop/detect_face_pyqt5/dlib_face_recognition_resnet_model_v1.dat'
        face_recognize_path = os.path.join(os.path.dirname(os.path.abspath('__file__')),
                                           'dlib_face_recognition_resnet_model_v1.dat')

        # 候选人文件夹
        #  faces_folder_path     = '/home/dx/Desktop/detect_face_pyqt5/candidate-faces'
        self.faces_folder_path = os.path.join(os.path.dirname(os.path.abspath('__file__')),
                                         'candidate-faces/')
        # 1.加载正脸检测器
        self.detector = dlib.get_frontal_face_detector()
        # 2.加载人脸关键点检测器
        self.landmark = dlib.shape_predictor(face_landmark_path)
        # 3. 加载人脸识别模型
        self.facerec = dlib.face_recognition_model_v1(face_recognize_path)

        # 候选人脸描述子list
        self.descriptors = []
        # 对文件夹下的每一个人脸进行:

        # 1.人脸检测
        # 2.关键点检测
        # 3.描述子提取
        time_flag = []  # 获取照片中时间
        file_glob = os.path.join(self.faces_folder_path, "*.jpg")
        file_list = []
        file_list.extend(glob.glob(file_glob))
        print(file_list)
        for i in range(0, len(file_list)):
            tmp = str(file_list[i])
            tmp_1 = tmp[51:65]  # 截取字符串,截取时间
            time_flag.append(tmp_1)
        # print(time_flag)
        cand_d = dict(zip(file_list, time_flag))
        cand_sorted = sorted(cand_d.items(), key=lambda d: d[1])  # 按字典的第二个关键字排序
        # print(cand_sorted)

        for f in range(0, len(cand_sorted)):

            print("Processing file: {}".format(cand_sorted[f][0]))
            self.img = io.imread(cand_sorted[f][0])
            # win.clear_overlay()
            # win.set_image(img)

            # 1.人脸检测
            dets = self.detector(self.img, 1)  # 人脸标定
            print("Number of faces detected: {}".format(len(dets)))

            for k, d in enumerate(dets):
                # 2.关键点检测
                shape = self.landmark(self.img, d)
                # 画出人脸区域和和关键点
                # win.clear_overlay()
                # win.add_overlay(d)
                # win.add_overlay(shape)

                # 3.描述子提取，128D向量
                face_descriptor = self.facerec.compute_face_descriptor(self.img, shape)

                # 转换为numpy array
                v = np.array(face_descriptor)
                self.descriptors.append(v)

        # 对需识别人脸进行同样处理

        # 提取描述子，不再注释

        # 候选人名单

       # self.candidate = ['dwh', 'whr', 'zjr', 'dx', 'dx', 'whr', 'dx']
        self.candidate = self.readfile()  # 读取候选人,从name.txt中
        print(self.candidate)

    def readfile(self):
        file_name_path = os.path.join(self.faces_folder_path, 'name.txt')
        with open(file_name_path, 'r') as f:
            content = f.read().splitlines()  # 去掉行尾的换行符\n 并保存为list
            #content = [line.rstrip('\n') for line in f]
            return content
```

















