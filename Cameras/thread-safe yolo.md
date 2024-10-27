https://docs.ultralytics.com/guides/yolo-thread-safe-inference/#thread-safe-example

# Safe: Instantiating a single model inside each thread
```
from ultralytics import YOLO
from threading import Thread


def thread_safe_predict(image_path):
# Instantiate a new model inside the thread
local_model = YOLO("yolov8n.pt")
results = local_model.predict(image_path)
# Process results


# Starting threads that each have their own model instance
Thread(target=thread_safe_predict, args=("image1.jpg",)).start()
Thread(target=thread_safe_predict, args=("image2.jpg",)).start()
```