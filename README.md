# Robotics Data Annotation Pipeline

A robotics data annotation workflow for inspecting LeRobot demonstration data, analyzing robot behavior, defining semantic subtasks, performing temporal video annotation with Label Studio, and exporting structured annotation data for downstream dataset development and analysis.

## Overview

This project implements a workflow for transforming raw robot demonstration data into structured temporal action annotations.

The workflow covers:

```text
LeRobot Dataset
      ↓
Dataset & Episode Inspection
      ↓
Multimodal Data Analysis
      ↓
Subtask Taxonomy Definition
      ↓
Label Studio Configuration
      ↓
Temporal Video Annotation
      ↓
Annotation Validation
      ↓
Structured JSON Export
```

The focus is on robot manipulation data and temporal segmentation of a demonstration into meaningful action-level subtasks.

---

## Dataset

The workflow uses the Hugging Face dataset:

**`lerobot/svla_so100_pickplace`**

The analysis focuses on **Episode 0**, a robot pick-and-place demonstration.

### Episode Details

- Frames: **454**
- Duration: **15.13 seconds**
- Frame rate: approximately **30 FPS**
- Top-camera observations
- Wrist-camera observations
- Robot action data
- Robot observation/state data
- Gripper information

The combination of video observations and structured robot data allows the demonstration to be analyzed from both visual and state/action perspectives.

---

## Technologies

- Python
- Google Colab
- Hugging Face Hub
- LeRobot `0.6.1`
- Pandas
- PyArrow
- Matplotlib
- Label Studio `1.23.0`
- JSON

---

## Repository Contents

| File | Description |
|---|---|
| `README.md` | Project documentation, workflow description, annotation taxonomy, and results. |
| `Robotics_annotation.ipynb` | Main notebook containing the end-to-end dataset inspection, robot-data analysis, video processing, Label Studio setup, and annotation workflow. |
| `episode_0.mp4` | Extracted Episode 0 robot demonstration video used for temporal annotation. |
| `episode_0_annotations.json` | Exported Label Studio annotation containing the temporal action labels and frame ranges. |

### `Robotics_annotation.ipynb`

The main notebook contains the workflow used to:

1. Install and validate LeRobot.
2. Inspect the `lerobot/svla_so100_pickplace` dataset.
3. Inspect dataset metadata and episode information.
4. Download the Episode 0 video streams.
5. Extract and visualize video frames.
6. Load Episode 0 robot data from Parquet.
7. Inspect robot action vectors and observation state.
8. Analyze the gripper action signal.
9. Inspect important moments in the demonstration using the available camera views.
10. Configure and launch Label Studio.
11. Serve the video to the Label Studio interface.
12. Create the temporal annotation workflow.
13. Export and validate the completed annotation.

### `episode_0.mp4`

This is the extracted video used for the annotation workflow.

It represents the selected robot manipulation demonstration from Episode 0 and provides the visual reference used to identify the boundaries of the robot's actions.

### `episode_0_annotations.json`

This file contains the completed Label Studio annotation exported from the project.

The annotation consists of five temporal action regions:

| Label | Start Frame | End Frame |
|---|---:|---:|
| `REACH` | 0 | 87 |
| `GRASP` | 88 | 151 |
| `TRANSPORT` | 152 | 205 |
| `PLACE` | 206 | 253 |
| `RETREAT` | 254 | 363 |

The exported JSON contains one completed annotation with five annotation results and no cancelled annotations.

---

## Dataset Inspection

The dataset structure was inspected to identify the available robot, episode, task, metadata, and video information.

Relevant components included:

```text
data/
meta/
videos/
```

Robot data was loaded from Parquet and filtered to the selected episode:

```python
data = pd.read_parquet(data_path)

episode_0_data = data[
    data["episode_index"] == 0
].copy()
```

The following fields were examined:

```text
timestamp
frame_index
action
observation.state
```

This provided a structured representation of the robot's behavior alongside the visual demonstration.

---

## Video Analysis

The selected Episode 0 video was extracted from the dataset and inspected frame-by-frame.

The video was used to identify the temporal boundaries of the robot's manipulation subtasks.

The analysis considered the visible robot motion and interaction with the manipulated object, while the structured robot/action data provided additional context for interpreting the demonstration.

The final short video used for annotation is included in the repository as:

```text
episode_0.mp4
```

---

## Gripper Signal Analysis

The gripper/action signal was examined to help interpret manipulation events such as grasping and releasing.

Observed gripper values ranged approximately from:

```text
Minimum: 0.1065
Maximum: 27.4760
```

The signal was visualized over time to support interpretation of the manipulation sequence and transitions between subtasks.

---

## Subtask Taxonomy

The manipulation demonstration was decomposed into five semantic subtasks:

```text
REACH
  ↓
GRASP
  ↓
TRANSPORT
  ↓
PLACE
  ↓
RETREAT
```

### Action Definitions

| Label | Description |
|---|---|
| `REACH` | Robot moves toward the object |
| `GRASP` | Robot grasps or picks up the object |
| `TRANSPORT` | Robot moves while carrying the object |
| `PLACE` | Robot places or releases the object |
| `RETREAT` | Robot moves away after placement |

This taxonomy provides a consistent representation of the robot's manipulation sequence and enables temporal segmentation of the demonstration.

---

## Label Studio Configuration

Label Studio was configured for temporal video annotation using the `TimelineLabels` interface.

The annotation configuration was:

```xml
<View>
  <Header value="Robot Demonstration Annotation"/>

  <Video
    name="video"
    value="$video_url"
    timelineHeight="120"
  />

  <TimelineLabels
    name="actions"
    toName="video"
  >
    <Label value="REACH"/>
    <Label value="GRASP"/>
    <Label value="TRANSPORT"/>
    <Label value="PLACE"/>
    <Label value="RETREAT"/>
  </TimelineLabels>
</View>
```

The video was provided to Label Studio through the `video_url` task field.

---

## Temporal Annotation

The demonstration was segmented into five sequential temporal regions.

| Action | Start Frame | End Frame |
|---|---:|---:|
| REACH | 0 | 87 |
| GRASP | 88 | 151 |
| TRANSPORT | 152 | 205 |
| PLACE | 206 | 253 |
| RETREAT | 254 | 363 |

The resulting temporal structure is:

```text
0                                                        363
|----------------------------------------------------------|
|     REACH    |   GRASP   | TRANSPORT | PLACE | RETREAT |
0             87         151         205    253        363
```

The regions are sequential and non-overlapping.

This provides a structured temporal representation of the robot's manipulation sequence.

---

## Annotation Validation

After completing the labeling task, the annotation was submitted through Label Studio and exported as JSON.

The exported annotation was independently inspected to verify:

- Annotation completion
- Expected label set
- Number of annotation regions
- Frame boundaries
- Region ordering
- Temporal overlap
- Association with the source video

The final annotation contained five temporal regions:

```text
REACH       0–87
GRASP       88–151
TRANSPORT   152–205
PLACE       206–253
RETREAT     254–363
```

The exported result contained one completed annotation with five annotation results.

---

## Output

The primary output is **structured annotation data**, rather than a modified video.

The Label Studio JSON export contains the temporal ranges and semantic labels associated with the demonstration.

A simplified representation is:

```json
{
  "label": "REACH",
  "start_frame": 0,
  "end_frame": 87
}
```

The complete export preserves the Label Studio annotation structure and the reference to the associated video.

---

## Workflow Capabilities

The workflow covers several components of a robotics data preparation and annotation process:

- Robotics dataset ingestion
- LeRobot dataset inspection
- Multimodal data exploration
- Robot video analysis
- Robot state/action analysis
- Gripper signal analysis
- Subtask decomposition
- Action taxonomy definition
- Label Studio configuration
- Temporal video annotation
- Human-in-the-loop labeling
- Annotation validation
- Structured JSON export

---

## Further Development

The workflow can be extended toward larger-scale robotics data processing and annotation.

### Automated Annotation Quality Checks

Potential validation checks include:

- Temporal overlap detection
- Missing-segment detection
- Invalid frame-range detection
- Label ordering validation
- Annotation completeness checks

### Model-Assisted Annotation

Vision-language models and language models can be incorporated to propose:

- Action labels
- Temporal boundaries
- Candidate subtask sequences

Human annotators can then review and correct the proposed annotations.

### Dataset-Level Analysis

The resulting annotation data can support:

- Subtask frequency analysis
- Subtask duration analysis
- Task diversity analysis
- Rare behavior identification
- Failure-case analysis
- Annotation coverage analysis

### Language Grounding

The annotation scheme can be extended with natural-language task instructions and descriptions to connect language, visual observations, and robot actions.

---

## Reproducibility

The main workflow is contained in:

```text
Robotics_annotation.ipynb
```

The notebook can be run in a Google Colab environment with the required Python dependencies installed.

The exported annotation is provided separately as:

```text
episode_0_annotations.json
```

The demonstration video used for annotation is provided as:

```text
episode_0.mp4
```

Together, these files provide the notebook workflow, source demonstration, and resulting structured annotation.

---

## Project Structure

```text
robotics-data-annotation-pipeline/
│
├── README.md
├── Robotics_annotation.ipynb
├── episode_0.mp4
└── episode_0_annotations.json
```

---

## Summary

This workflow demonstrates the transformation of a robot manipulation demonstration from raw multimodal data into structured temporal action annotations:

```text
LeRobot Dataset
      ↓
Data Inspection
      ↓
Robot Demonstration Analysis
      ↓
Subtask Taxonomy
      ↓
Label Studio Annotation
      ↓
Validation
      ↓
Structured JSON
```

The resulting structured annotations provide a foundation for robotics dataset curation, temporal behavior analysis, annotation quality checks, and further machine-learning workflows.
