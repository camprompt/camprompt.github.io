# CamPrompt Annotations for the MultiCamVideo Dataset

Text annotations released with the WACV 2027 paper
**CamPrompt: Camera-Controlled Video-to-Video Generation from Text Prompt**.

These files add **natural-language video captions** and **dense camera-motion
prompts** to every training video of the
[MultiCamVideo dataset](https://huggingface.co/datasets/KlingTeam/MultiCamVideo-Dataset)
introduced by [ReCamMaster](https://arxiv.org/abs/2503.11647). The captions and
dense camera prompts are the annotations used to train CamPrompt. We also
release an alternative **single camera prompt per video**, which was **not used
for training**.

**No videos are included.** Download the videos and camera extrinsics from the
original dataset; the annotations join to it by file path.

## Contents

| File | Rows | Columns |
|---|---|---|
| `captions.csv` | 136,000 | `file_name`, `caption` |
| `camera_prompts_dense.csv` | 136,000 | `file_name`, `segment_1`, `segment_2`, `segment_3`, `segment_4` |
| `camera_prompts_single.csv` (not used for training) | 136,000 | `file_name`, `camera_prompt` |

* **`file_name`** is the video path relative to the root of the MultiCamVideo
  dataset, e.g. `train/f18_aperture10/scene1/videos/cam01.mp4`. All files
  cover exactly the same 136,000 videos: 4 camera settings × 3,400 scenes × 10
  cameras (the full `train` split). The `val` split is not annotated.
* **`caption`** is a paragraph describing the scene, subjects and action of the
  video (median 82 words).
* **`segment_1` … `segment_4`** (in `camera_prompts_dense.csv`) describe the camera motion during four
  consecutive segments (quarters) of the 81-frame clip, in the camera's own
  frame of reference (median 7 words each), e.g. `Move left while pushing in
  and turning right.`
* **`camera_prompt`** (in `camera_prompts_single.csv`) describes the camera
  motion of the whole clip in a single instruction (median 9 words; complex
  trajectories can take a few sentences, up to 50 words), e.g. `Move the camera
  left and slightly forward while turning to the right.` **CamPrompt was not
  trained on these prompts**; they are provided for users who want one prompt
  per video instead of four.

Example rows:

```
file_name,segment_1,segment_2,segment_3,segment_4
train/f35_aperture2.4/scene26/videos/cam02.mp4,Tilt downwards.,Tilt downwards.,Tilt downwards.,Tilt downwards.
train/f35_aperture2.4/scene26/videos/cam04.mp4,Move downward and forward while tilting upwards.,Move downward and forward while tilting upwards.,Move downward while tilting upwards.,Move downward while tilting upwards.
```

```
file_name,camera_prompt
train/f35_aperture2.4/scene26/videos/cam02.mp4,Tilt the camera downwards.
train/f35_aperture2.4/scene26/videos/cam04.mp4,Move the camera downward and slightly forward while tilting it upwards.
```

## Usage

```python
import pandas as pd

captions = pd.read_csv("captions.csv")
prompts = pd.read_csv("camera_prompts_dense.csv")
annotations = captions.merge(prompts, on="file_name")

# Optional: one camera prompt per video (not used to train CamPrompt).
single = pd.read_csv("camera_prompts_single.csv")
annotations = annotations.merge(single, on="file_name")

# Path to the video inside your copy of MultiCamVideo-Dataset:
#   <MultiCamVideo-Dataset root>/<file_name>
```

## How the annotations were made

* **Captions**: each video was captioned automatically with Gemini 2.5 Flash.
* **Camera prompts**: each camera trajectory (from the dataset's
  `camera_extrinsics.json`) was split into 4 consecutive chunks. For each chunk
  we computed the relative 6-DoF camera motion (translation and Euler angles)
  and used Gemini 2.5 Flash to describe that motion in a short natural-language
  instruction. See Sec. 3 and Appendix "Details of Training Data" of the paper.
* **Single camera prompts** (not used for training): Gemini 2.5 Flash
  summarized the camera motion of the whole trajectory in one instruction.

All annotations are machine-generated with Gemini 2.5 Flash.

## Known limitations

* Automatically generated text can contain errors, including hallucinated
  details in captions and imprecise motion descriptions.
* 49 source videos in MultiCamVideo are entirely black; their captions say so
  (e.g. "The video is a black screen.").
* Segments split each clip uniformly in time; a motion change inside a segment
  is summarized in one phrase.
* A single camera prompt summarizes the whole clip and can leave out motion
  that the segment prompts mention (e.g. a roll).

## License

The annotations in this release are licensed under the
[Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)
(see `LICENSE`). The MultiCamVideo videos and camera data are **not** part of
this release and remain under their own license (Apache-2.0, see the
[original dataset](https://huggingface.co/datasets/KlingTeam/MultiCamVideo-Dataset)).

## Citation

If you use these annotations, please cite CamPrompt and ReCamMaster:

```bibtex
@inproceedings{liu2027camprompt,
  title     = {CamPrompt: Camera-Controlled Video-to-Video Generation from Text Prompt},
  author    = {Liu, Xinhao and Chen, Kan and Zhong, Yuan and Jin, Pengchong and Xu, Yan and Chen, Zhuoyuan and Yu, Ting},
  booktitle = {Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision (WACV)},
  year      = {2027}
}

@misc{bai2025recammaster,
  title         = {ReCamMaster: Camera-Controlled Generative Rendering from A Single Video},
  author        = {Jianhong Bai and Menghan Xia and Xiao Fu and Xintao Wang and Lianrui Mu and Jinwen Cao and Zuozhu Liu and Haoji Hu and Xiang Bai and Pengfei Wan and Di Zhang},
  year          = {2025},
  eprint        = {2503.11647},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CV}
}
```

*This is not an officially supported Google product.*
