# RGB-D Tree Trunk Diameter Mapping Pipeline

This repository contains the code used to estimate tree trunk diameters from synchronized RGB images and depth data collected with an Intel RealSense camera. The pipeline combines object detection, segmentation, and 3D reconstruction to estimate trunk diameter from depth imagery.

The code was developed for research on spatial variability and tree structural measurements in commercial tart cherry orchards, but the workflow can be applied to other orchard systems or structured tree rows.

---

## Processing Pipeline

```text
RGB + Depth Images
        │
        ▼
YOLO Object Detection
        │
        ▼
SAM2 Trunk Segmentation
        │
        ▼
Depth → 3D Point Cloud
        │
        ▼
Trunk Diameter Estimation
```

---

# Pipeline Overview

The processing workflow consists of the following stages:

```text
0_Capture_Images
1_Crop_Images_and_Clean_Dataset
2_Object_Detection
3_Trunk_Segmentation
4_Trunk_Diameter_Estimation
```

---

# Step 0 – Capture Images

A Raspberry Pi connected to an Intel RealSense depth camera captures synchronized RGB images and depth arrays. Image capture automatically begins when the camera is connected and aligns frames with external distance measurements.

**Output**

- RGB images (`.png`)
- Depth arrays (`.npy`)

---

# Step 1 – Crop Images and Clean Dataset

Images are cropped to remove the upper portion of the frame that does not contain trunks. Depth arrays are cropped using the same coordinates to maintain alignment with the RGB images.

The dataset is also filtered so that only frames associated with valid sensor measurements are retained.

**Output**

- Cropped RGB images  
- Cropped depth arrays  

---

# Step 2 – Object Detection

YOLO is used to detect tree trunks in each image. When multiple bounding boxes are detected, the algorithm selects the candidate trunk based on:

- bounding box height (taller boxes preferred)
- average depth inside the bounding box (closer objects preferred)

Detection outputs are saved in COCO annotation format.

Optional analysis scripts compute detection success rates and other metrics for reporting.

**Output**

- YOLO annotations (`.json`)
- Annotated images (optional)
- Detection summary tables

---

# Step 3 – Trunk Segmentation

Segment Anything Model 2 (SAM2) is used to generate pixel-level trunk masks from the YOLO bounding boxes.

Masks are saved in batches to reduce memory usage during processing.

Optional scripts visualize the segmentation masks overlaid on the original images.

**Output**

- Trunk segmentation masks (JSON batches)

---

# Step 4 – Trunk Diameter Estimation

Depth values from segmented trunk pixels are converted to 3D coordinates using the RealSense camera intrinsics.

The following processing steps are applied:

1. Generate point clouds from depth pixels within the trunk mask  
2. Remove statistical outliers  
3. Apply DBSCAN clustering to isolate the trunk  
4. Extract a horizontal slice at a specified height  
5. Estimate trunk diameter using the width of the slice  
6. Apply Savitzky-Golay smoothing to stabilize estimates  

**Output**

- Trunk diameter estimates  
- Optional point clouds (`.ply`)  
- Final results table  

---

# Input Data Requirements

The pipeline expects paired RGB and depth data with matching filenames:
image_name.png
image_name.npy

Depth arrays should contain depth values aligned with the RGB image.

---

# Software Requirements

Major dependencies include:

- Python
- OpenCV
- NumPy
- Pandas
- Open3D
- PyTorch
- Ultralytics YOLO
- Segment Anything Model 2 (SAM2)
- scikit-learn
- Intel RealSense SDK

A full environment file or requirements list can be added for reproducibility.

---

# Notes

Large intermediate outputs such as segmentation masks, point clouds, and generated datasets are intentionally excluded from this repository using `.gitignore`.

Users should provide their own input imagery and depth data when running the pipeline.

---

# Citation

If you use this code in research, please cite the associated publication describing the trunk diameter mapping method.
