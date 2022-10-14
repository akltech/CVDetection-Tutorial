# Create an OpenCV Program for Vehicle Detection in a Video 

This tutorial provides a step-by-step introduction to object detection using OpenCV-Python, the Python API of OpenCV, a computer vision library. To create a vehicle detection program, complete the following steps:

## Objectives

  * [Before you Begin](#before-you-begin)

  * [Step 1: Read, display, and write a video](#step-1-read-display-and-write-a-video)

  * [Step 2: Apply image preprocessing techniques](#step-2-apply-image-preprocessing-techniques)

  * [Step 3: Use a Haar Cascade Classifier to detect vehicles](#step-3-use-a-haar-cascade-classifier-to-detect-vehicles)

  * [Common Errors](#common-errors)


## Requirements

OpenCV and Python are required to run the code sample locally. If you are on Windows and do not already have OpenCV installed see <a href="https://medium.com/@pranav.keyboard/installing-opencv-for-python-on-windows-using-anaconda-or-winpython-f24dd5c895eb" target="_blank">Installing OpenCV for Python.</a> Familiarity with computer vision and machine learning terms is helpful but not required. 


## Before you begin 

* Install ```OpenCV-contrib```. The package contains a Haar cascade classifier, the object detection algorithm used in this tutorial. If you installed OpenCV-Python with Anaconda, you must install contrib to retrieve the Cascades directory. 
  <br></p>
To install ```OpenCV-contrib```, open the Anaconda prompt and use the ```pip``` <a href="https://pypi.org/project/opencv-contrib-python/" target="_blank">command:</a>
    ```
    pip install opencv-contrib-python
    ```

*  Download the ```samplevideo.mp4``` file and the ```cars.xml``` file from <a href="https://github.com/akltech/Vehicle-Detection/blob/94c946f07dfba7fcd958893d59c074fbe26fe91a/Program%20Files" target="_blank">Program Files</a> into your code directory. The ```samplevideo.mp4``` file contains a sample video from a highway surveillance camera.
<br></p>
The XML file is a pre-trained classifier for cars. Training refers to feeding the machine learning algorithm data so that it can learn to detect specific objects. The ```cars.xml``` file contains features classified as a car or non-car. 

## Step 1: Read, display and write a video.

OpenCV's HighGUI API provides methods to access your computer hardware and file system and display video streams in the High-level GUI (Graphical User Interface). A GUI allows you to interact with a computer system through visual graphics, such as clickable file icons on a Windows OS. With the HighGUI module, you can read and write videos into a given directory and view them in real time.

**To load, view, and write a video, follow these steps:**

  1. Import the dependencies: ```cv2``` and ```datetime```

  2. To capture a video from your file system, call the ```VideoCapture()``` method. Pass a parameter to this method to capture a video file or a camera on your computer. For example, you can usually input ```0``` as an argument to retrieve your webcam feed. The following code sample shows how to use the ```VideoCapture()``` method to capture a video from your file system:
  <br></p>
    ```
    cap = cv2.VideoCapture('video path')
    ```
    
  3. Set the output video's frame size by calling the ``` get()``` method. To obtain the default resolution of the frames in a video stream, you can pass an enumerator as a parameter to the ``` get()``` method--for example, ```CAP_PROP_FRAME_WIDTH```. The default resolutions of your system will apply. 
    <br></p>
Alternatively, you can modify your output video's frame size by passing a number as a parameter to the ```get()``` method, as follows in the code sample below. For more information about other enumerators, see <a href="https://docs.opencv.org/3.4/d4/d15/group__videoio__flags__base.html" target="_blank">Enumerations.</a>
<br></p>
     ```
     frame_width = int(cap.get(3))
     frame_height = int(cap.get(4))
     ```
    
  4. To create a video file, make an instance of the ``` VideoWriter()``` class. The first parameter is the filename or the path to the output video file. Next, you must provide the fourcc codec, a four-character code. A codec is a hardware or software that compresses or decompresses a digital video–for example, you can compress an MP4 with the MPEG4 codec```mp4v```. This way, the output video frames will be compressed to a smaller file size suitable for viewing from your directory. To view a list of fourcc codes, see <a href="https://learn.microsoft.com/en-us/windows/win32/medfound/video-fourccs" target="_blank">Video FOURCCs.</a>
  <br></p>
The final parameters of this method are, respectively, frames per second (FPS) and the frame size of the output video. Setting an FPS that is too fast or slow will interfere with your detection algorithm's accuracy. A good starting point for the FPS is ten seconds. The following code sample shows how to call the ``` VideoWriter()``` method to write a video file in a given directory with today’s date:
  <br></p>
    ```
    out = cv2.VideoWriter(f'C:/detect2/test{datetime.date.today()}.mp4',``` <br></p>
    ```
    cv2.VideoWriter_fourcc('m', 'p', '4', 'v'), 10,  (frame_width,frame_height)) 
    ```
    
  5. Initialize a loop with a return value ```ret``` to indicate if you have captured the video frames. To read each frame, invoke the ```read()``` function.
 <br></p>
     ```
     while True:
     ret, frame = cap.read()
     if ret:
     ```
     
  6. Next, display the video frames in the HighGUI window using the ```imshow()``` method. 
  <br></p>
    ```
    cv2.imshow('frame', frame)
    ```
  7. Pass the frames into the video file by calling the ```write()``` function.
  <br></p>
    ```
    out.write(frame)
    ```
  8. To process OpenCV's HighGUI event messages, such as initilaizing the GUI window and closing it after timer invoke the ```waitKey()``` function. Pass this method a parameter to tell it the number of milliseconds to wait to close the window—for example, ```waitKey(20)``` waits up to twenty milliseconds to stop processing the GUI events. Regardless of the timeout value, ```waitKey()``` returns instantly with key input. However, an additional condition is required to stop a video with a key event, as shown in the following code sample:
<br></p>
    ```
    if cv2.waitKey(25) & 0xFF == ord('q'):
    ```
    
  9. Close the video capture object and all frames by executing the ```release()``` function.
  <br></p>
    ```
    cap.release()
    ```
  10. Destroy the GUI window by calling the ```destroyAllWindows()``` method. The GUI window will usually close when the program terminates, but if you are executing it in Python from the terminal rather than a script, it can remain open until you quit Python. Calling ```destroyAllWindows()```  is a good practice to prevent an open GUI window from lingering.
  <br></p>
    ```
    cv2.destroyAllWindows()
    ```
    
## Step 2: Apply image preprocessing techniques

To better detect objects in an image, you must process the image before feeding it to the detection algorithm. Applying techniques with OpenCV, such as gaussian blurring and dilation, reduces the algorithm's complexity. You will begin by applying a gaussian blur to reduce the noise level and smooth the image. For more information about image preprocessing methods in OpenCV, see <a href="https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html" target="_blank">Image Filtering.</a>

**Apply image filtering:**

  1. Blur each frame with the ``` GausianBlur()``` method.
The intensity of the blur depends on the kernel size; the more significant the kernel size, the more intense the blur effect.
 <br></p>
  ```frame = cv2.GaussianBlur(frame, (5, 5), 0)```
  
  2. Accentuate the features of each frame and further blur the image by calling the ```Dilate()``` function.
  ```cv2.Dilate(frame, (5,5), 2)```
  
  3. Convert the frames to grayscale using the ```cvtColor()``` method.  Color is usually not helpful in identifying features such as edges in computer vision applications. Though, there are exclusions. For example,  if you want to detect objects of a specific color, such as red cars on a gray highway, you need information about the colors. Because the colors on the vehicles in the sample video vary greatly, you will convert the images to grayscale.
    <br></p>
OpenCV-Python provides color conversion codes or flags for converting images to different color spaces. For BGR to gray conversion, input the flag ```BGR2GRAY```  to the ```cvtColor()``` function, as shown in the following code. For a list of color conversion codes, see <a href="https://docs.opencv.org/4.x/df/d9d/tutorial_py_colorspaces.html/." target="_blank">Changing Colorspaces.</a>
   <br></p>
  ```gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)```
  
  4. Optional: Convert the video to binary format by calling the ```Laplacian()``` method. A binary image usually consists of only two pixels, usually black and white. This way, the computer can more accurately identify the objects. 
   <br></p>
The ```Laplacian()``` method is an edge detection algorithm that returns a binary image. Sudden changes in pixel intensity result in black and white egde pixels. An image's Laplacian highlights areas where the pixel intensity changes rapidly, making it an excellent method to call for identifying edges in a video. For more information about ```Laplacian()```, see <a href="https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#gad78703e4c8fe703d479c1860d76429e6">Image Filtering.</a>
  <br></p>
To apply Laplacian edge detection, modify the following code sample:
    <br></p>
    ```
    frame = cv2.Laplacian(src=frame, ddepth=cv2.CV_8U, ksize=3)
    ```
  <br />
  <img width="440" height="430" src="Laplacian_edge_detection.jpg">
  Figure 1. A highway surveillance video converted to binary format with Laplacian edge detection.
</p>
<br />
 

## Step 3: Use a Haar Cascade Classifier to detect vehicles

**Detect vehicles in the frames and draw the boundary boxes:**

  1. To tell the classifier to read the pre-trained model for cars, call the ```CascadeClassifier()``` method and pass in the ```cars.xml``` file as an argument.  In the following code sample, the pre-trained model for cars is read using the ```CascadeClassifier()``` method.
    <br></p>
    ```
    car_cascade = cv2.CascadeClassifier('cars.xml')
    ```
  
  2. Detect objects of different sizes in each frame and return a list of rectangles by calling the ```detectMultiScale()``` method. The list of rectangles will be processed in the next step to draw the boundary boxes. 
<br></p>
    ```
    cars = car_cascade.detectMultiScale(gray, 1.1, 1)
    ```
  
  3. Draw rectangles or boundary boxes around the vehicles by applying the ```rectangle()`` function. The function uses the points collected in the previous line of code to print rectangle outlines on the cars in each frame. 
    <br></p>
     ```
     for (x, y, w, h) in cars:
         cv2.rectangle(frame, (x,y), (x+w,y+h), (0,0,255), 2)
     ```
 
**Modify the following code sample to create a vehicle detection program:**

    import cv2
    import datetime
    
    cap = cv2.VideoCapture('C:/cardetect/samplevideo.mp4')
    frame_width = int(cap.get(3))
    frame_height = int(cap.get(4))
    # Write the video file 
    out = cv2.VideoWriter(f'C:/cardetect/test{datetime.date.today()}.mp4',
    cv2.VideoWriter_fourcc('m', 'p', '4', 'v'), 10, (frame_width,frame_height))

    while True:
    # Capture each frame of the video
        ret, frame = cap.read()
        if ret:    

            frame = cv2.GaussianBlur(frame, (5, 5), 0)
            gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
            frame = cv2.Laplacian(src=frame, ddepth=cv2.CV_8U, ksize=3)

            # Load the cars.xml file into the classifier
            car_cascade = cv2.CascadeClassifier('C:/cardetect/cars.xml')
            # Detect objects of different sizes in each frame 
            # and return a list of rectangles
            cars = car_cascade.detectMultiScale(gray, 1.1, 1)
            # Read the list of rectangles to draw rectangle boundaries 
            # around the cars in each frame
            for (x, y, w, h) in cars:
                cv2.rectangle(frame, (x,y), (x+w,y+h), (0,0,255), 2)

            cv2.imshow("frame", frame)
            out.write(frame)
            # Break the loop with a key event
            if cv2.waitKey(25) & 0xFF == ord('q'):
                break
        else:
            break
            
    cap.release()
    cv2.destroyAllWindows()

## Common Errors 

This section lists errors common to the OpenCV-Python API.

**This item was encoded in a format that's not supported**

One possible solution is to download the <a href="https://www.videolan.org/vlc/download-windows.html" target="_blank">VLC media player for Windows.</a>
Then, right click on the video and go to ```Properties```, ```Change...```, and select ```VLC Media Player```

<br />

**SystemError: <class 'cv2.CascadeClassifier'> returned a result with an error set***
  
Ensure ```opencv-contrib-python``` is installed. Return to [Before you Begin](#before-you-begin) for installation instructions. 
