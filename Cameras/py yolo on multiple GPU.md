
https://github.com/ultralytics/yolov5/issues/6735

https://github.com/ultralytics/yolov5/issues/2995#issuecomment-1737833987


MeatyAri commented on Sep 27, 2023
Apparently you can run YOLO on AMD GPUs using onnxruntime_directml here's how:
I did this on yolov8 ON WINDOWS and I believe it should work with any other yolo version out there:

1- install the DirectML version of ONNX. It's crucial to choose ONNX DirectML over any other variants or versions. The Python package you need is aptly named "onnxruntime_directml". Feel free to use:

pip install onnxruntime_directml

2- render your YOLO model into the ONNX format.

from ultralytics import YOLO

model = YOLO('yolov8n.pt')
model.export(format='onnx')

3- Add the 'DmlExecutionProvider' string to the providers list: this is lines 133 to 140 in "venv\Lib\site-packages\ultralytics\nn\autobackend.py":

133        elif onnx:  # ONNX Runtime
134        LOGGER.info(f'Loading {w} for ONNX Runtime inference...')
135        # check_requirements(('onnx', 'onnxruntime-gpu' if cuda else 'onnxruntime'))
136        import onnxruntime
137        providers = ['CUDAExecutionProvider', 'CPUExecutionProvider'] if cuda else ['DmlExecutionProvider', 'CPUExecutionProvider']
138        session = onnxruntime.InferenceSession(w, providers=providers)
139        output_names = [x.name for x in session.get_outputs()]
140        metadata = session.get_modelmeta().custom_metadata_map  # metadata
✨ Comment check_requirements line 135 ✨ Add the 'DmlExecutionProvider' string to the providers list

4- enjoy the 100% boot in terms of the model performance

---

???
https://github.com/ultralytics/ultralytics/issues/3084
