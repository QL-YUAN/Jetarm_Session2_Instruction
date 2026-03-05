## ROS Image Capture and Manual Saving Tool

This Python script implements a **ROS-based image capture utility** that allows users to manually save images from a camera topic by clicking on the displayed image. It is primarily intended for tasks such as **camera calibration**, **dataset collection**, and **debugging vision pipelines**.

The script subscribes to an RGB camera topic, displays the live image stream using OpenCV, and saves snapshots when the user clicks the left mouse button.

---

## Purpose and Use Cases

This tool is useful for:

- Capturing calibration images (e.g., chessboard images)
- Collecting training or testing datasets
- Verifying camera image quality and alignment
- Debugging ROS camera pipelines

---

## Dependencies

The following packages are required:

- ROS (with `rospy`)
- OpenCV (`cv2`)
- `cv_bridge`
- `sensor_msgs`

Ensure that `cv_bridge` is properly installed and sourced in your ROS environment.

---

## Subscribed Topic

The node listens to the following ROS topic, which is available once you correctly launch the JetArm Robot:

```text
/rgbd_cam/color/image_rect_color
````

This topic is expected to publish messages of type:

```text
sensor_msgs/Image
```

---

## Program Workflow

The script follows this execution flow:

1. Initialize a ROS node named `image_capture_node`
2. Create an OpenCV window to display the camera feed
3. Register a mouse callback function for image saving
4. Wait for incoming images from the camera topic
5. Convert ROS image messages to OpenCV format using `cv_bridge`
6. Display the image in real time
7. Save the current image when the user clicks the **left mouse button**
8. Exit the program gracefully when the `q` key is pressed

---

## Image Saving Mechanism

* Each left mouse click saves the currently displayed image
* Images are saved in the **current working directory**
* Filenames are automatically generated as:

```text
0.png, 1.png, 2.png, ...
```

The image counter increments after each save to prevent overwriting files.

---

## User Instructions

1. Start the ROS camera driver so that the image topic is publishing
2. Run the script:

   ```bash
   python3 image_capture.py
   ```
3. A window titled **"ROS Image Feed"** will appear
4. Click anywhere inside the image window to save the current frame
5. Press **`q`** to exit the program

---

## Mouse and Keyboard Controls

| Action           | Description            |
| ---------------- | ---------------------- |
| Left mouse click | Save the current image |
| `q` key          | Quit the program       |

---

## Notes and Limitations

* Images are saved in **BGR format**, as used by OpenCV
* The script saves images continuously until manually exited
---

## Summary

This script provides a simple and effective way to manually capture images from a ROS camera topic using OpenCV. Its interactive design makes it especially suitable for calibration and data collection tasks in robotics and computer vision applications.

## Python code

```
#!/usr/bin/env python3
import cv2
import rospy
from sensor_msgs.msg import Image
from cv_bridge import CvBridge, CvBridgeError

# Global variable to store the latest image

current_image = None
count=0
# Initialize CvBridge
bridge = CvBridge()

# Define the callback function for mouse clicks
def save_image(event, x, y, flags, param):
    global count
    if event == cv2.EVENT_LBUTTONDOWN and current_image is not None:
        # Save the current frame when the left mouse button is clicked
        filename = str(count)+".png"
        cv2.imwrite(filename, current_image)
        print(f"Image saved as {filename}")
        count+=1

def capture_and_save_image():
    rospy.init_node('image_capture_node', anonymous=True)
    
    # Wait for a single message on the /rgbd_cam/color/image_rect_color topic
    rospy.loginfo("Waiting for image message...")
    # Create an OpenCV window and set the mouse callback
    cv2.namedWindow("ROS Image Feed")
    cv2.setMouseCallback("ROS Image Feed", save_image)
        
    while True:
        msg = rospy.wait_for_message('/rgbd_cam/color/image_rect_color', Image)
        
        # Convert the ROS image message to OpenCV format
        try:
            global current_image
            current_image = bridge.imgmsg_to_cv2(msg, "bgr8")
            rospy.loginfo("Image received successfully!")
        except CvBridgeError as e:
            rospy.logerr(f"Error converting ROS image: {e}")
            return
    

        # Display the image once received
        cv2.imshow("ROS Image Feed", current_image)
        
        # Wait for a mouse click
        print("Click on the image to save it.")    
        key = cv2.waitKey(1) & 0xFF
        
        # Exit the loop when 'q' is pressed
        if key == ord('q'):
            break
    
    # Close OpenCV window
    cv2.destroyAllWindows()

if __name__ == "__main__":
    capture_and_save_image()
```

