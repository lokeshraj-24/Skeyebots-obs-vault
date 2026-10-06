(Slicing Aided Hyper Inference)


Final inference: best ot implemetn


Drawbacks to account for:

-> CANNOT USE SAHI WITH YOLO INBUILT TRACKING ------ sol below
-> Increased Latency

Workaround: Do SAHI, get yolo output and merge it back, and feed the merged frame to a tracker manually
Ex:
```
from ultralytics.trackers import ByteTrack  # Or BOT_SORT

tracker = ByteTrack()  # Initialize once outside loop

# In frame loop
sahi_result = get_sliced_prediction(...)  # As above
detections = sahi_result.to_ultralytics_detections()  # Convert to YOLO format if needed

tracked_objects = tracker.update(detections)  # Returns tracked boxes with IDs
```


Reduce Latenncy:

Parallel slicing
SAHI allows parallel slicin -> feeding sliced images parallely through multiple CPU threads so the model can process the slices parallely

**DeepStream Integration**:

- NVIDIA’s DeepStream SDK, tailored for Jetson, allows you to build a pipeline where slices are processed in parallel via a batching mechanism or custom plugins.
- DeepStream can queue slices as a mini-batch, leveraging NVDEC for decoding and TensorRT for inference, optimizing the entire pipeline for 30fps.