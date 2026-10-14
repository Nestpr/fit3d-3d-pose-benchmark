# Comprehensive Evaluation of Monocular and Multi-view 3D Human Pose Estimation Models for Fitness Exercise Analysis

Repository accompanying the research article:

> **Comprehensive Evaluation of Monocular and Multi-view 3D Human Pose Estimation Models for Fitness Exercise Analysis**

The project provides a unified experimental framework for comparing recent monocular and multi-view deep learning approaches for three-dimensional human pose estimation on the **Fit3D** dataset.

Three representative architectures from different 3D human pose estimation paradigms are evaluated:

- **TCPFormer**, representing Transformer-based temporal 2D-to-3D lifting;
- **PoseMamba-L**, representing State Space Model-based temporal 2D-to-3D lifting;
- **MVGFormer**, representing geometry-aware multi-view 3D pose estimation.

The monocular models are evaluated using three different sources of 2D keypoints: projected **ground-truth 2D keypoints**, **RTMPose**, and **YOLO-Pose**. MVGFormer instead operates directly on synchronized RGB images from four calibrated cameras.

The evaluation considers both **spatial accuracy** and **temporal consistency**. Spatial reconstruction accuracy is measured using **PA-MPJPE**, while temporal stability is assessed using the **RMSE of joint velocities** with respect to the ground truth. In addition, three temporal post-processing techniques are evaluated: the **One Euro Filter**, **Savitzky--Golay Filter**, and **Median Filter**.

The study also analyzes model complexity through the number of parameters and multiply--accumulate operations (MACs), as well as the influence of 2D detector quality, skeletal representation, exercise type, and temporal filtering on the final 3D reconstructions.

## Project overview

The experimental pipeline includes:

- preparation of the Fit3D multi-view fitness sequences;
- projection of Fit3D 3D ground-truth joints into a frontal camera for monocular evaluation;
- extraction of 2D keypoints using RTMPose and YOLO-Pose;
- conversion of 2D detections to the Human3.6M 17-joint representation;
- monocular 3D pose reconstruction with TCPFormer and PoseMamba-L;
- multi-view 3D pose reconstruction with MVGFormer using synchronized RGB images and camera calibration;
- conversion of the different skeletal representations to a common joint convention when required;
- calculation of spatial reconstruction accuracy using PA-MPJPE;
- evaluation of temporal consistency using velocity RMSE;
- application of One Euro, Savitzky--Golay, and Median temporal filters;
- comparison of filtered and unfiltered 3D pose sequences;
- per-subject, per-exercise, and per-joint analysis;
- qualitative visualization of reconstructed 3D poses and temporal trajectories.

The main objective is to provide a unified benchmark in which the behavior of monocular temporal models and multi-view geometric reconstruction can be analyzed under the same Fit3D-based evaluation framework.

## Dataset

The dataset is not included in this repository due to its size and licensing conditions.

The experiments use the publicly available **Fit3D** dataset, which was specifically designed for 3D human motion analysis in fitness scenarios.

Fit3D contains:

- **13 subjects** performing fitness exercises;
- **47 exercise types** grouped into warm-up, barbell, dumbbell, and equipment-free exercises;
- **611 multi-view sequences**;
- recordings from **four synchronized RGB cameras**;
- RGB resolution of **900 × 900 pixels**;
- a recording frame rate of **50 fps**;
- 3D ground-truth poses captured using a **12-camera Vicon motion-capture system**;
- ground-truth skeletons containing **25 joints**;
- intrinsic and extrinsic camera calibration parameters.

The dataset defines an official training and test split. Since the 3D ground truth of the test split is withheld for the online benchmark, the experiments in this work use the eight training subjects for which complete annotations are available:

```text
s03, s04, s05, s07, s08, s09, s10, s11

## Experimental setup

The benchmark follows a common evaluation protocol across all methods in order to minimize differences caused by data preparation or evaluation procedures.

Two different 3D pose estimation pipelines are considered.

### Monocular pipeline

For TCPFormer and PoseMamba-L, a single frontal Fit3D camera is used.

The monocular pipeline includes:

1. obtaining 2D keypoints from projected ground truth, RTMPose, or YOLO-Pose;
2. converting the detected keypoints to the Human3.6M 17-joint representation;
3. processing temporal windows of **243 frames**;
4. reconstructing the corresponding 3D pose sequence;
5. aligning the predictions with the Fit3D ground truth for evaluation.

### Multi-view pipeline

MVGFormer directly processes:

1. synchronized RGB images from the four Fit3D cameras;
2. the corresponding intrinsic and extrinsic camera calibration parameters;
3. the multi-view observations for each time instant independently;
4. the resulting 3D pose predictions converted to the common 15-joint representation used for comparison.

Unlike TCPFormer and PoseMamba-L, MVGFormer does not explicitly model temporal continuity between consecutive frames.

### Temporal post-processing

After inference, the reconstructed 3D trajectories can optionally be processed using:

- One Euro Filter;
- Savitzky--Golay Filter;
- Median Filter.

Each filter is applied independently to every coordinate of every joint trajectory.

The same evaluation procedure is then used to compare the original and filtered predictions in terms of spatial accuracy and temporal consistency.

## Model architectures

Three representative 3D human pose estimation architectures are evaluated, covering complementary modeling paradigms.

### TCPFormer

TCPFormer is a monocular 2D-to-3D lifting model based on Transformer architectures.

The model introduces an **implicit pose proxy** that summarizes temporal information and facilitates the modeling of long-range dependencies across the input sequence.

In this benchmark, TCPFormer receives sequences of **243 frames of 2D keypoints** and predicts the corresponding 3D pose sequence.

### PoseMamba-L

PoseMamba-L is a monocular 3D pose estimation architecture based on **State Space Models** and the Mamba framework.

Instead of relying on conventional self-attention, it uses bidirectional spatio-temporal state-space operations to model dependencies across joints and frames.

This design reduces computational complexity while maintaining long-range temporal modeling capabilities.

As with TCPFormer, the model receives sequences of **243 frames of 2D keypoints**.

### MVGFormer

MVGFormer is a multi-view 3D human pose estimation architecture that combines deep visual features with explicit multi-view geometry.

The model integrates:

- a convolutional image backbone;
- Transformer-based pose queries;
- camera calibration information;
- differentiable triangulation.

Unlike the monocular models, MVGFormer receives synchronized RGB images from multiple calibrated cameras and reconstructs the 3D pose directly from the multi-view observations.

The model operates **frame by frame**, without explicitly modeling temporal dependencies between consecutive frames.

### Model summary

| Model | Paradigm | Input | Temporal modeling | Parameters | MACs |
|---|---|---|---|---:|---:|
| TCPFormer | Transformer-based monocular lifting | 2D keypoint sequence (`T = 243`) | Temporal attention | 35.1 M | 449 M/frame |
| PoseMamba-L | State Space Model-based monocular lifting | 2D keypoint sequence (`T = 243`) | Bidirectional state-space modeling | 6.7 M | 115 M/frame |
| MVGFormer | Geometry-aware multi-view reconstruction | Multi-view RGB + camera calibration | None, frame-wise | 37.6 M | 639.7 G* |

\* For MVGFormer, the reported MACs correspond to one frame-wise multi-view inference in the original experimental configuration.

## Evaluation metrics

The benchmark evaluates two complementary aspects of 3D pose estimation performance:

- spatial reconstruction accuracy;
- temporal consistency of the predicted motion.

### PA-MPJPE

Spatial accuracy is measured using the **Procrustes-Aligned Mean Per Joint Position Error (PA-MPJPE)**.

Before computing the error, the predicted pose is aligned with the ground truth through a similarity transformation including:

- translation;
- rotation;
- scale.

The metric is expressed in **millimeters (mm)**, and lower values indicate better 3D reconstruction accuracy.

PA-MPJPE is computed over the corresponding skeletal representation used by each model:

- TCPFormer: 17 Human3.6M joints;
- PoseMamba-L: 17 Human3.6M joints;
- MVGFormer: 15 common joints.

For the per-joint comparison between all three models, the predictions are converted to the shared **COMMON15** representation.

### Velocity RMSE

Temporal consistency is evaluated using the **Root Mean Square Error of joint velocities with respect to the ground truth**.

For each joint, velocity is computed as the Euclidean displacement between consecutive frames:

\[
v_{t,j}(X) = \|X_{t,j} - X_{t-1,j}\|_2
\]

The velocity error is then computed as:

\[
RMSE_{vel} =
\sqrt{
\frac{1}{(T-1)J}
\sum_{t=2}^{T}
\sum_{j=1}^{J}
\left(
v_{t,j}(\hat{Y}) - v_{t,j}(Y)
\right)^2
}
\]

The metric is reported in **mm/frame**, with lower values indicating better temporal consistency.

This metric makes it possible to detect frame-to-frame instability, jitter, or abrupt changes that may not be reflected by frame-wise spatial metrics such as PA-MPJPE.

The temporal evaluation is performed both before and after applying the evaluated post-processing filters.

## Computational evaluation

In addition to reconstruction accuracy and temporal consistency, the benchmark considers the computational complexity of the evaluated 3D pose estimation architectures.

The following indicators are reported:

- number of model parameters;
- multiply--accumulate operations (MACs);
- temporal processing strategy;
- input modality required by each architecture.

The reported values are taken from the original model publications.

For TCPFormer and PoseMamba-L, MACs/frame represent the computational cost per output frame for a 243-frame input sequence.

For MVGFormer, the reported MACs correspond to one frame-wise multi-view inference in the original experimental configuration.

| Model | Parameters | MACs | Input |
|---|---:|---:|---|
| TCPFormer | 35.1 M | 449 M/frame | 2D keypoint sequence |
| PoseMamba-L | 6.7 M | 115 M/frame | 2D keypoint sequence |
| MVGFormer | 37.6 M | 639.7 G* | Multi-view RGB + calibration |

\* MVGFormer MACs correspond to one multi-view frame-wise inference in the configuration reported by the original authors.

PoseMamba-L is considerably lighter than TCPFormer, requiring approximately **five times fewer parameters** and **four times fewer operations per output frame**.

MVGFormer has a substantially higher computational cost because it processes RGB images from multiple cameras and performs image feature extraction together with multi-view geometric reconstruction.

The computational values reported here are intended to characterize model complexity rather than direct execution speed, since runtime also depends on hardware, implementation details, input resolution, and software configuration.

## Analysis and visualization resources

The project includes several resources for quantitative analysis and qualitative visualization of the 3D pose estimation results.

These include:

- **Per-exercise analysis:** comparison of PA-MPJPE across the different Fit3D exercises for TCPFormer, PoseMamba-L, and MVGFormer.
- **2D detector comparison:** analysis of the influence of ground-truth 2D keypoints, RTMPose, and YOLO-Pose on the monocular lifting models.
- **Per-joint analysis:** comparison of reconstruction errors across individual body joints using the shared COMMON15 skeleton.
- **Temporal analysis:** visualization of mean body velocity over time to identify jitter, abrupt frame-to-frame variations, and the effect of temporal filtering.
- **Filter comparison:** quantitative and qualitative comparison of the One Euro, Savitzky--Golay, and Median filters.
- **3D skeleton visualization:** representative frames comparing the ground truth with TCPFormer, PoseMamba-L, and MVGFormer predictions.
- **Video demonstrations:** reconstructed 3D pose sequences showing the temporal behavior of the evaluated models and the effect of post-processing filters.

Representative figures and videos are stored in the repository assets and are also used throughout this README to illustrate the main experimental findings.

## Repository structure

The repository is currently organized around the documentation, experimental results, and visual resources associated with the benchmark:

```text
fit3d-3d-pose-benchmark/
├── assets/                         # Figures, images, GIFs, and videos
│   ├── figures/                    # Figures used in the analysis
│   ├── skeletons/                  # Joint-representation diagrams
│   └── videos/                     # Qualitative video demonstrations
├── docs/                           # Additional methodological documentation
│   ├── dataset.md                  # Fit3D dataset description
│   ├── models.md                   # Evaluated 3D pose estimation models
│   ├── skeleton_mapping.md         # Joint conventions and COMMON15 mapping
│   ├── metrics.md                  # Spatial and temporal evaluation metrics
│   └── temporal_filtering.md       # Temporal post-processing methodology
├── results/                        # Experimental results and analyses
│   ├── spatial_results.md          # PA-MPJPE results
│   ├── temporal_results.md         # Velocity RMSE and filtering results
│   └── qualitative_results.md      # Representative qualitative comparisons
├── LICENSE                         # Project license
└── README.md                       # Main project documentation

## Execution environment

The experiments were developed using both local and cloud-based resources.

Local development was performed on:

```text
OS: macOS Catalina 10.15.7
Computer: iMac Retina 5K 27-inch (2020)
CPU: 6-core Intel Core i5 @ 3.1 GHz
RAM: 8 GB
GPU: AMD Radeon Pro 5300
GPU memory: 4 GB
```

Since the local machine does not provide CUDA support, the GPU-dependent experiments were mainly executed in **Google Colab** using the following environment:

```text
Python: 3.12.13
PyTorch: 2.10.0
CUDA: 12.8
GPU: NVIDIA Tesla T4
```

The project uses **PyTorch** as the main deep learning framework for model inference.

Additional libraries used throughout the experimental pipeline include:

- NumPy;
- OpenCV;
- Pandas;
- Matplotlib;
- SciPy.

For MVGFormer, the multi-view RGB images were processed at a resolution of:

```text
960 × 540
```

together with the corresponding camera calibration parameters.

The required Deformable Attention CUDA components were compiled in the Google Colab environment, and the model used the provided T-pose query initialization.

## Installation

The repository is currently focused on the documentation, experimental methodology, results, and qualitative resources associated with the benchmark.

The implementation and processing scripts used to reproduce the experiments are not yet included in the public repository. Therefore, no installation procedure is currently required.

Once the source code is released, the repository will include the corresponding dependency files and setup instructions for:

- TCPFormer;
- PoseMamba-L;
- MVGFormer;
- RTMPose;
- YOLO-Pose;
- evaluation utilities;
- temporal filtering utilities.

Specific compatibility notes will also be provided for MVGFormer, since its original implementation requires additional CUDA components for Deformable Attention.

## Configuration

The public repository is currently focused on the experimental documentation, results, and visualization resources associated with the benchmark.

The configuration files and executable scripts used during the experiments are not yet included in the repository.

The experimental setup described in this repository follows the configuration used in the associated study, including:

- Fit3D subjects: `s03`, `s04`, `s05`, `s07`, `s08`, `s09`, `s10`, and `s11`;
- monocular temporal window length: `243` frames;
- monocular 2D input sources: projected ground truth, RTMPose, and YOLO-Pose;
- MVGFormer input: four synchronized RGB views with camera calibration;
- MVGFormer image resolution: `960 × 540`;
- spatial evaluation metric: PA-MPJPE;
- temporal evaluation metric: velocity RMSE;
- temporal filters: One Euro, Savitzky--Golay, and Median.

The corresponding configuration files and command-line options will be documented when the implementation is publicly released.

## Usage

The repository includes the Jupyter notebooks used to reproduce the main experimental pipelines described in the study.

The notebooks cover:

- Fit3D data preparation and preprocessing;
- 2D pose extraction with RTMPose and YOLO-Pose;
- TCPFormer inference;
- PoseMamba-L inference;
- MVGFormer multi-view inference;
- skeletal representation conversion and joint mapping;
- PA-MPJPE evaluation;
- velocity RMSE evaluation;
- temporal filtering;
- quantitative result aggregation;
- qualitative visualization and figure generation.

Each notebook contains the configuration and processing steps required for the corresponding experiment.

Because the evaluated models rely on different original implementations and dependency requirements, the notebooks are organized separately rather than being executed through a single unified command.

Additional usage instructions and dependency notes are provided within the corresponding notebooks.

## Third-party code and attribution

This project uses and adapts components from the official implementations of the models and tools evaluated in the study, including:

- **TCPFormer**;
- **PoseMamba**;
- **MVGFormer**;
- **RTMPose**;
- **YOLO-Pose**.

The original pretrained models, source code, and external dependencies remain subject to their respective licenses and terms of use.

The **Fit3D** dataset is not redistributed as part of this repository and must be obtained separately from its official source under the corresponding dataset license.

Detailed references to the original repositories, publications, and third-party licenses are provided in the project documentation and in the `NOTICE` file.

---

## License

This project is licensed under the **Apache License 2.0**.

See the `LICENSE` file for more information.

Third-party code, pretrained models, datasets, and other external resources are not covered by this repository's Apache 2.0 license unless explicitly stated, and remain subject to their original licenses and terms of use.
