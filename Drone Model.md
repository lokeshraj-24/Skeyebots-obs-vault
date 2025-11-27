
Backbone: YOLOV11n**

Custom Pre-processing steps -> For inference:
	 - Terrain -> Low visibility expected -> Apply [^1]CLAHE  or Histogram equaliser before feeding into YOLO
	 - Frame Stabilization: To account for constant movement and unexpected shake
	 - Cancel out Drone's [^2][[Ego-motion]] (Ht)
	 - Slicing ([[SAHI]] algo) to a definite square-like shape (ex: 512x512)----------------Increase inference time by 2-4x for 4-9 slices, adding an extra 10-20ms

TensorRT Optimization:
-> Faster inference

```
import tensorrt as trt
import pycuda.driver as cuda
import pycuda.autoinit
# Load TensorRT engine and create multiple streams
streams = [cuda.Stream() for _ in range(4)]  # One stream per slice
# Custom logic to queue slices to streams
```

[^1]: Contrast Limited Adaptive histogram Equalization

[^2]: estimation of drone's own movement by analysing the sequence of image


INFERENCE:


1st Train:

```
model.train(
	data='/content/visdrone.yaml',
	epochs=100,
	imgsz=960,
	batch=16,
	workers=2,
	lr0=0.01,
	lrf=0.01,
	momentum=0.937,
	weight_decay=0.0005, # Helps regularization
	warmup_epochs=3, # Warmup for stable initial convergence
	augment=True
	)
```


| Epoch | train/box_loss | train/cls_loss | train/dfl_loss | precision | recall   | mAP50    | mAP50-95 | val/box_loss | val/vls_loss | val/dfl_loss | lr/pg0       | lr/pg1       | lr/pg2      |
| ----- | -------------- | -------------- | -------------- | --------- | -------- | -------- | -------- | ------------ | ------------ | ------------ | ------------ | ------------ | ----------- |
| 95    | 1.39515,       | 1.04741,       | 1.13657,       | 0.4984,   | 0.38482, | 0.40555, | 0.24311, | 1.47437,     | 1.17016,     | 1.17883,     | 4.62898e-05, | 4.62898e-05, | 4.62898e-05 |
| 96    | 1.39537,       | 1.04689,       | 1.13483,       | 0.49546,  | 0.38578, | 0.40628, | 0.24308, | 1.47534,     | 1.17026,     | 1.17868,     | 3.96865e-05, | 3.96865e-05, | 3.96865e-05 |
| 97    | 1.39487,       | 1.0473,        | 1.1343,        | 0.49678,  | 0.3846,  | 0.4067,  | 0.24324, | 1.47581,     | 1.16947,     | 1.17898,     | 3.30832e-05, | 3.30832e-05, | 3.30832e-05 |
| 98    | 1.38979,       | 1.04182,       | 1.13511,       | 0.49464,  | 0.38851, | 0.40627, | 0.24313, | 1.47563,     | 1.16971,     | 1.17905,     | 2.64799e-05, | 2.64799e-05, | 2.64799e-05 |
| 99    | 1.39091,       | 1.03977,       | 1.13322,       | 0.49209,  | 0.38858, | 0.4068,  | 0.24352, | 1.47563,     | 1.16996,     | 1.17878,     | 1.98766e-05, | 1.98766e-05, | 1.98766e-05 |
| 100   | 1.38992,       | 1.03969,       | 1.13128,       | 0.49433,  | 0.38617, | 0.4069,  | 0.24364, | 1.4758,      | 1.16966,     | 1.17881,     | 1.32733e-05, | 1.32733e-05, | 1.32733e-05 |


2nd Training:

```
model2.train(
	data="/content/visdrone.yaml",
	epochs=150, imgsz=960, batch=16, workers=2,
	optimizer="AdamW",
	lr0=0.0018, # AdamW likes a smaller peak
	lrf=0.02, # slightly higher tail helps when underfitting
	weight_decay=0.01, # decoupled WD for AdamW
	warmup_epochs=3,
	mosaic=0.15, mixup=0.0, scale=0.5, degrees=0.0, shear=0.0, perspective=0.0,
	translate=0.05, hsv_h=0.015, hsv_s=0.50, hsv_v=0.40,
	close_mosaic=15, label_smoothing=0.03,
	multi_scale=False, patience=50
	)
```

|Epoch|train/box_loss|train/cls_loss|train/dfl_loss|precision|recall|mAP50|mAP50-95|val/box_loss|val/vls_loss|val/dfl_loss|lr/pg0|lr/pg1|lr/pg2|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|**10**|1.48137|1.15463|1.15398|0.48539|0.37088|0.38926|0.23106|1.49661|1.21635|1.18913|0.00060757|0.00060757|0.00060757|
|**11**|1.48299|1.15969|1.15412|0.48784|0.37037|0.38732|0.22901|1.50477|1.21934|1.19206|0.000600967|0.000600967|0.000600967|
|**12**|1.49011|1.16342|1.1528|0.48665|0.37413|0.38631|0.22877|1.49422|1.22169|1.18384|0.000594364|0.000594364|0.000594364|
|**13**|1.47945|1.15084|1.14756|0.56633|0.3749|0.39111|0.23114|1.49231|1.22169|1.18875|0.00058776|0.00058776|0.00058776|
|**14**|1.48572|1.15416|1.15087|0.47336|0.38196|0.39637|0.23503|1.50118|1.19752|1.18421|0.000581157|0.000581157|0.000581157|
|**15**|1.47945|1.14825|1.1496|0.48087|0.38009|0.39125|0.23309|1.49162|1.21244|1.18284|0.000574554|0.000574554|0.000574554|


3rd Training:

```
stage1_args = dict(
	data='/content/visdrone.yaml',
	imgsz=960,
	batch=16,
	epochs=100,
	optimizer="AdamW",
	lr0=0.00075,
	lrf=0.10,  # cosine final LR = 10% of start
	weight_decay=0.01,  # AdamW-style
	warmup_epochs=5,
	nbs=64, # gradient accumulation → effective batch 64
	cos_lr=True,
	amp=True,
	workers=8,
	degrees=5, translate=0.10, scale=0.50, shear=0.0,
	fliplr=0.5, flipud=0.0,
	mosaic=1.0, mixup=0.10,
	hsv_h=0.015, hsv_s=0.70, hsv_v=0.40,
	box=7.0, cls=0.4, # Loss tuning (safe, modest bumps)
	cache=True, # Caching the images into the RAM
	)
```





I just did 100epoch in Adam optimizer itself because I was notiving improvement and loss and mAP too. So after I finished the 100 epoch, just out of curiosity I downloaded the best model and tried it with few sample dataset and it was giving way more false positives than any previous models. Will those be resolved upon training it on SGD optimizer for 200 more epoch or is it a problem to address, like an unexpected outcome?


Totally expected—and fixable.

**What happened:**  
Running only **Adam/AdamW for 100 epochs** often drives recall fast but also **inflates confidence on background** for tiny objects. Ultralytics saves `best.pt` by **mAP50–95**, which can prefer higher recall even if precision drops. So it’s normal to see **more FPs** on ad-hoc samples.

### Will switching to SGD help?

**Yes, usually.** An SGD “polish” with a low LR tail + a bit more weight decay typically:

- reduces overconfident background activations,
    
- tightens boxes (lower dfl/box loss),
    
- improves **precision** at the same recall.
    

### Do these quick sanity checks first (no retrain needed)

1. **Inference thresholds** (your tests vs val):
    
    - Use consistent `conf` and `iou` (start `conf=0.35–0.5`, `iou=0.5`), `agnostic_nms=False`, `max_det` sane (e.g., 300).
        
2. **Pick a threshold from your PR curve**, not a guess:
    
    ```python
    from ultralytics import YOLO
    m = YOLO("best.pt")
    m.val(data="your.yaml", imgsz=960, conf=0.001, iou=0.5, plots=True)  # creates PR/curves per class
    ```
    
    Read the per-class best-F1 thresholds and use those for inference.
    
3. Try an **earlier checkpoint** (higher precision): sometimes `epoch ~40–70` has fewer FPs than the final “best mAP”.
    

### Then do the SGD finish (recommended)

Use your latest checkpoint (best or last), and continue:

```python
from ultralytics import YOLO

m = YOLO("runs/detect/train/weights/last.pt")  # or the best.pt if you prefer
m.train(
    data="your_dataset.yaml",
    imgsz=960,
    batch=16,
    epochs=180,                 # 150–200 is typical
    optimizer="SGD",
    lr0=0.0025,                 # scaled for batch=16
    lrf=0.01,                   # try 0.005 if FPs persist
    momentum=0.937, nesterov=True,
    weight_decay=0.0007,        # try 0.001 if FPs persist
    cos_lr=True,
    cache=True,                 # or 'disk' if RAM is low
    # slightly lighter aug while polishing
    mosaic=0.7, mixup=0.05,
    # tiny-object loss balance (training-time only)
    box=7.0, cls=0.4,           # can lower cls→0.3 if FPs are mostly “wrong class”
)
# Final stability pass (last 10–15 epochs)
m = YOLO("runs/detect/train2/weights/last.pt")
m.train(
    data="your_dataset.yaml", imgsz=960, batch=16, epochs=15,
    optimizer="SGD", lr0=0.0025, lrf=0.01, momentum=0.937, nesterov=True,
    weight_decay=0.0007, cos_lr=True, cache=True,
    mosaic=0.0, mixup=0.0, hsv_h=0.0, hsv_s=0.0, hsv_v=0.0,
    degrees=0.0, translate=0.0, scale=1.0, shear=0.0, fliplr=0.0, flipud=0.0,
    box=7.0, cls=0.4,
)
```

### If FPs still remain after ~50–80 SGD epochs

- **Tighten the tail:** `lrf=0.005` (lower end LR), **or** `weight_decay=0.001`.
    
- **Loss balance:** `cls=0.3`, keep `box≈7.0`, `obj≈2.0` if your build exposes it.
    
- **Data fixes (biggest win):**
    
    - Add **pure background images** (no labels) from your domains.
        
    - Sweep **hard negatives** that look like humans/vehicles at distance (rocks, shadows, poles) and include them without boxes.
        
    - Make sure labels include **all** true objects—missing positives teach the model to ignore real ones and over-trust background.
        

### Quick checklist to avoid “unexpected outcomes”

- Compare **precision/recall at a fixed conf** across checkpoints (not just mAP).
    
- Keep **heavy aug off** near the end (you already do this in the stability pass).
    
- Use **consistent letterboxing/imgsz** between train & test (960 in both).
    
- Don’t rely solely on `best.pt`; also evaluate `last.pt` and some mid-epoch weights.
    

**Bottom line:** Your spike in FPs after 100 Adam epochs is common for tiny-object YOLO. Continuing with **SGD for ~150–200 epochs** (plus a short no-aug tail) almost always pulls precision back up. If some FPs remain, tighten LR/WD and add background/hard-negative data—those two changes fix the stubborn cases.



## YOLOv11-P2-P5 head

First 100 epoch - ADam optimizer, paramters used:

```
stage1_args = dict(
	data='/content/visdrone.yaml',
	imgsz=960,
	batch=16,
	epochs=100,
	optimizer="AdamW",
	lr0=0.00075,
	lrf=0.10, # cosine final LR = 10% of start
	weight_decay=0.01, # AdamW-style
	warmup_epochs=5,
	nbs=64, # gradient accumulation → effective batch 64
	cos_lr=True,
	amp=True,
	workers=8,
	# Tiny-object friendly aug (conservative geometry)
	degrees=5, translate=0.10, scale=0.50, shear=0.0,
	fliplr=0.5, flipud=0.0,
	mosaic=1.0, mixup=0.10,
	hsv_h=0.015, hsv_s=0.70, hsv_v=0.40,
	# Loss tuning (safe, modest bumps)
	box=7.0, cls=0.4,
	# Caching helps if RAM allows
	cache=True,
	)
```

results:![[results_Adam.csv]]
Inference:
-> Steady increase but hit plateau p soon, final val:
	box_loss = 1.38
	cls_loss = 1.039
	dfl_loss = 1.13
	Precision = 0.49
	recall = 0.38
	mAP50 - 0.40
	mAP50-95 - 0.24


109 epoch, SGB optimizer, parameters used:

```
stage2_args = dict(
	data='/content/visdrone.yaml',
	imgsz=960,
	batch=16,
	epochs=200,
	#resume=False, # using a fresh YOLO() from last.pt, so resume=False is fine
	optimizer="SGD",
	lr0=0.0025, # 0.0025 for batch=16
	lrf=0.01, # cosine final LR = 1% of start
	momentum=0.937,
	weight_decay=0.0007,
	nbs=64,
	cos_lr=True,
	amp=True,
	workers=8,
	# keep aug but slightly lighter now
	mosaic=0.7, mixup=0.05,
	# keep the same loss knobs for continuity
	box=7.0, cls=0.4,
	cache=True,
	)
```

![[results (1).csv]]

Inference:
-> Again steady increase but hit plateau at around 60-70 epoch
	box_loss = 1.32
	cls_loss = 0.88
	dfl_loss = 1.09
	precision = 0.52
	recall = 0.39
	mAP50 = 0.42
	mAP50-95 = 0.25

3rd STAGE, 
parameters:

```
stage3_args = dict(
	data='/content/visdrone.yaml',
	imgsz=960,
	batch=16,
	epochs=140, # 100–150 is fine; you can early-stop on val recall
	optimizer="SGD",
	nbs=64,
	# LR & regularization: a touch more drive, gentler decay
	lr0=0.0035, # +0.001 over Stage 2 to shake out of the plateau
	lrf=0.05, # keep useful LR late (not as tiny as 0.01)
	momentum=0.95, # a bit more smoothing helps keep FPs in check
	weight_decay=0.0004, # relax WD to help recall
	warmup_epochs=5,
	cos_lr=True,
	workers=8,
	# Augs tuned for tiny objects (more zoom-in, fewer synthetic artifacts late)
	mosaic=0.9, # slightly stronger than Stage 2
	mixup=0.0, # turn off to avoid diluting tiny instances
	degrees=0.0, translate=0.08, scale=0.20, shear=0.0, perspective=0.0,
	fliplr=0.4, flipud=0.0,
	  
	# Make late training “look real” to lift both recall & precision
	close_mosaic=10, # disable mosaic for final 20 epochs
	  
	# Loss balance: keep box strong; slight cls bump to protect precision
	box=7.0, cls=0.50,
	cache=True,
	)
```

