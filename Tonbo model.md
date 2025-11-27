
Software:

"Animesh Khare" <animesh.khare@tonboimaging.com>
"BRIJ Bhushan" <brij.bhushan@tonboimaging.com>
"Himanshu Arora" <harora@tonboimaging.com>


"Apoorva Parihar" <apoorva.parihar@tonboimaging.com>
"Deep Singh" <deep.singh@tonboimaging.com>

Documentation on [[Prowl AD Camera]]


[[Information, mail notes]]

##### Data pipeline (possibilities):
- Receive Tonbo raw data from sensors, detection in AI, send BB to Tonbo and let them do the tracking
- Receive Tonboo processed data with embedded BB into the image, do the detection again and then do the send information of the Tonbo again for tracking
- Receive Tonbo raw image + annotated, secondary detection in AI node, and then send info back to Tonbo sens for tracking





## Ways to improve model performance using Hard Negatives: 
#### Case A – Adding hard negatives during normal training

If you integrate hard negatives into your training dataset (with correct _no-object_ annotations):
- **Improves precision** (reduces false positives).  
    The model learns to suppress detections on confusing backgrounds.
    
-  **May reduce recall** slightly.  
    Because the model becomes more conservative; borderline true positives might get suppressed too.
    
-  **Best practice:**
    
    - Mix ~10–30 % hard negatives into your training set.
        
    - Keep _normal positives_ well represented so the model still learns target diversity.
        
    - Use `cls` loss weight tuning or focal loss to prevent imbalance.


#### Case B – Fine-tuning an already trained model **only** on hard negatives

That’s the situation you asked about: you already have a well-trained model → now fine-tune on negatives only from your operational terrain.

What happens internally

YOLO and similar detectors train by assigning labels:

- Positive boxes contribute to object (cls + box) losses.
    
- Unlabeled regions implicitly act as background (no-object) loss.
    

If your fine-tuning dataset has no positive boxes, the model only sees "everything = background", which:

- strengthens its no-object confidence in those terrains (good for suppressing FPs),
    
- but also erodes sensitivity to real targets (if LR or epochs are too high),
    
- because gradients drive class and objectness predictions toward zero everywhere.


#### Works well if:
- You use a very low learning rate (e.g., 1e-5 to 5e-6).
    
- You freeze most of the backbone, train only heads (`model.model[-1].requires_grad=True` type setup).
    
- You fine-tune for few epochs (1–5) or early-stop when FP loss plateaus.
    
- Your negatives are domain-matched (same sensor, lighting, terrain).
    
- Optionally, you blend a small fraction (~5–10 %) of true positives during this run.
    

Then you get the best of both worlds:

- sharper discrimination for your deployment terrain,
    
- minimal drift or forgetting of target features.
- 
#### Hurts if:
	You fine-tune too long or with normal LR → catastrophic forgetting.  
    The model “forgets” what targets look like, because it only optimizes background confidence.
    
- You have too few negative frames → overfitting to a small patch of terrain.
    
- You don’t freeze earlier layers → the feature extractor itself shifts.

There are a few potential pitfalls to be aware of:
	Catastrophic forgetting: If you fine-tune your model on a new dataset without including any of the original training data, the model may "forget" what it has learned previously. This is known as catastrophic forgetting, and it can lead to a significant drop in performance on the original task.
	Overfitting: If your dataset of hard negatives is too small, the model may overfit to that specific data and fail to generalize to new examples.






Meeting agenda:


1) ASk about Inbuilt tracking
2) How does Tonbo allow bi-directional comms between camera and AI node 
3) What is the interface/protocol used for comms - Ethernet - RTSP stream
4) If they have the Ethernet capabilities or what mode of comms
5) Security protocol used by Tonbo
6) If Tonbo have any AI analysis in built in system, if so what is it 
7) They expect user input for tracking
8) precision issues with tracking, 
9) API dock: LRF, motor controls
10) LRF: specs, cycle period, any cooldown, the expected latency
11) Format of the data received
12) Status of the camera, how to get this info
13) How to assign sweep parameters, zoom capabilities
14) Image specs, RGB/thermal, if its in same channel, or whats the format of the image received
15) If there's any encoding done
16) How fast is the zoom


Image data:
1) Image dimention
2) How much pixels(dimentions) are expected for a OoI respective to the img dimension befroe adn after zoom
	1) zoom time period
3) Format of thermal img(grayscale or heatmap(RGB) both day nad night
4) Frame rate
5) How many frames() needed fro training
6) 6hr vid, from different locations
7) 

Prowl - SR -> 3-5km
Avenger - LR

Shouldnt deploy our ode into their device


Difference in format of prowler adn avenger




