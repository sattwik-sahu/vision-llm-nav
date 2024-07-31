---
tags:
  - articles/research
  - notes
url: https://arxiv.org/abs/2312.03275
code: true
---
> [!help] Resources 
> 📄 **Article**
> [Yokoyama, VLFM: Vision-Language Frontier Maps for Zero-Shot Semantic Navigation](https://arxiv.org/abs/2312.03275)
> 
> :luc_github: GitHub
> [bdaiinstitute/vlfm]([https://github.com](https://github.com/bdaiinstitute/vlfm))

> [!quote] Abstract
> Understanding how humans leverage semantic knowledge to navigate unfamiliar environments and decide where to explore next is pivotal for developing robots capable of human-like search behaviors. We introduce a ==zero-shot navigation approach==, Vision-Language Frontier Maps (VLFM), which is inspired by human reasoning and designed to *navigate towards unseen semantic objects in novel environments*. VLFM **builds occupancy maps from depth observations** to identify frontiers, and leverages **RGB observations** and a pre-trained **vision-language model** to ==generate a language-grounded value map==. VLFM then uses this map to identify the **most promising frontier** to explore for *finding an instance of a given target object category*. We evaluate VLFM in photo-realistic environments from the Gibson, Habitat-Matterport 3D (HM3D), and Matterport 3D (MP3D) datasets within the Habitat simulator. Remarkably, VLFM achieves state-of-the-art results on all three datasets as measured by [success weighted by path length (SPL)](<https://eval.ai/web/challenges/challenge-page/97/evaluation#:~:text=Success%20rate%20weighted%20by%20(normalized,Success%20Rate%20against%20Trajectory%20Length.>) for the Object Goal Navigation task. Furthermore, we show that VLFM’s ==zero-shot nature== enables it to be **readily deployed on real-world** robots such as the Boston Dynamics Spot mobile manipulation platform. We deploy VLFM on Spot and demonstrate its capability to efficiently navigate to target objects within an office building in the real world, ==without any prior knowledge of the environment==. The accomplishments of VLFM underscore the promising potential of vision-language models in advancing the field of semantic navigation.

# Introduction

## Human Navigation:person_with_probing_cane: 

- Human navigation in unfamiliar environmets involves complex processes:
	- Explicit maps
	- **Internal knowledge:** Accumulation of semantic knowledge. Used to infer-
		- Layout of a space
		- Locations of specific objects
		- Geometric configurations
- **e.g.** We know toilets and showers are located together in the bathrooms, which are often located near bedrooms.
- Natural language can be used to enhance this semantic knowledge.

## Robots Capable of Human-Like Navigation 🤖

==Foundation models== which can learn to mimic this **human reasoning process** are invaluable.

### Zero-Shot Methods

- **[[bs-thesis/notes/LLM-Based Zero-Shot Object Navigation#Language-Based Zero-Shot Object Navigation|Zero-Shot Methods :luc_link_2:]]** utilize these models to facilitate semantic navigation *wihout any task-specific training or fine-tuning*.
- Zero shot methods are convenient because:
	- **Easily adaptable** for new tasks.
	- Repurposed for **future robotics systems** performing complex tasks.
	- Provide **intermediate representations** which improves *interpretability*.

> The remarkable performance of large language models (LLMs) and vision-language models (VLMs) has facilitated ==task-independent solutions== for the *semantic inference of out-of-view scene information*.

[[bs-thesis/articles/VLFM-Zero-Shot-Semantic-Navigation.pdf#page=1&selection=165,31,168,51|VLFM-Zero-Shot-Semantic-Navigation, page 1]]

## Proposed Idea 💡


> [!important] Vision Language Frontier Maps
> - It is a **zero-shot** approach for **target-driven semantic navigation**
> - The **goal** is to navigate to an *unseen object* in a *novel environment*.
> 
> ### Occupancy Maps & Frontier Identification
> 
> - VLFM builds [occupancy maps](https://en.wikipedia.org/wiki/Occupancy_grid_mapping) from **depth observations** to identify **frontiers** of the *explored map region*.
> - To find the **semantic target objects**, VLFM **prompts a pre-trained VLM** *"Which of these frontiers is most likely to lead to the semantic target?"*

### Difference from Previous Methods

- In contrast to prior language-base zero-shot semantic navigation methods, VLFM **does not rely on object detectors and language models** to evaluate frontiers using *text-only semantic reasoning*.
- VLFM uses a **vision-language model** to directly extract semantic values from RGB images in the form of a *cosine similarity score* with a text-prompt involving the target object.
- These scores are used to create a **language-ground value map**, which is used to identify the most promising frontier to explore.

> This spatially- grounded joint vision-language-based semantic reasoning increases computational inference speed and overall semantic navigation performance.

[[bs-thesis/articles/VLFM-Zero-Shot-Semantic-Navigation.pdf#page=2&selection=33,45,44,23|VLFM-Zero-Shot-Semantic-Navigation, page 2]]

![[bs-thesis/assets/vlfm__intro.png]]

# Methodology 🛠️

## Problem Formulation 🤔

### Task :obs_paste: 

**ObjectNav** - Robot is tasked with searching an instance of a *target object category* in a *previously unseen environment*.

### High-Level Understanding 🧠

This approach encourages the robot to understand the environment on a higher level, semantically - *type of room?* based on the objects seen; rather than solely relying on geometric cues.

### Sensors 📷

1. Egocentric RGB-D camera.
2. Odometry sensor providing current `(x, y)` and `heading` relative to starting pose.
### Action Space :luc_navigation: 

1. `move_forward(0.25)`
2. `turn_left(30)`
3. `turn_right(30)`
4. `look_up(30)`
5. `look_down(30)`
6. `stop()`


> [!success] Success
> An episode is defined to be completed sucessfully ✅ if `stop()` is called within $1\; m$ of the target object in $\le 500$ steps.  

## The Algorithm :luc_feather:

![[bs-thesis/assets/vlfm__maps-generation.png]]

### Frontier Waypoint Generation :luc_locate: 

1. Convert depth image to point cloud 
2. Classify points as obstacle or not obstacle
3. Transform all points to global frame (based on current odometry), then project onto a 2D grid
4. Identify each boundary separating the *explored* and *unexplored* areas, identify its **midpoint** as a ==potential frontier waypoint==.

#### Frontiers and Exploration

- As the robot explores the area, the positions and quantity of these frontier wayopints will vary until the *entire environment* has been explored, at which points there will be *no more frontiers*.
- If the target object *has not been found* at this point, the robot simply triggers `stop()`, ending the episode unsuccessfully ☹️

### Value Map Generation 🗺️

- Value map is a ==2D grid== similar to the frontier map.
- This map assigns a *value to each cell within the explored area*, quantifying the **semantic relevance/similarity** to the target object.
- The value map is used to *evaluate each frontier*, and the *highest value* is chosed as the ==next waypoint to explore==.

#### Comparison to Frontier Map

| Frontier Map                                                                  | Value Map                                                                            |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Uses depth and odometry values to build a top-down map iteratively            | Same                                                                                 |
| Has a single channel representing the "obstacle"-ness of the cell in the grid | Has **two channels**, representing the *semantic value score* and *confidence score* |

### Achieveing Human-Like Reasoning 🚶‍♀️

- Humans derive cues directly from :luc_eye: *visual observation* (lighting, room type, room size, navigability to other rooms, etc.), rather than attempting to first convert current visuals to text :luc_file_text:
- Here, a pre-trained vision language model ([BLIP-2](https://arxiv.org/abs/2301.12597)) is used to obtain a **cosine similarity score** from the robot's *current RGB observation* and a *text-prompt* containing information about the *target object*.

### Semantic Value Channel

> :luc_image: + :luc_file_text: (*"Seems like there is a `target_object` ahead"*)
>:obs_down_arrow_with_tail:
>==BLIP-2==
>:obs_down_arrow_with_tail: 
>*Similarity (to :luc_file_text:) score* for each pixel in :luc_image:

### Confidence Channel

> The confidence channel aims to determine how a pixel’s value in the *semantic value channel should be updated* if it has a value assigned from a *previous time step* and is within the robot’s field-of-view (FOV) at the current time step

[[bs-thesis/articles/VLFM-Zero-Shot-Semantic-Navigation.pdf#page=3&selection=88,0,91,56|VLFM-Zero-Shot-Semantic-Navigation, page 3]]

- Value of pixel in semantic value channel not affect if the given pixel was not seen until the current time step
- Confidence of pixels along the optical axis have a value of $1$ and those at the left and right edges of the image have $0$.

$$c_{\text{pixel}} = \cos^2\big(\frac{\theta}{\theta_{\text{fov}} / 2} \cdot \frac{\pi}{2}\big)$$
where
- $\theta$ is the angle between the *pixel and the optical axis*
- $\theta_{\text{fov}}$ is the *horizontal FOV of the robot's camera*

![[bs-thesis/assets/vlfm__confidence-score-usage.png]]

### Using both the Channels

#### Update Semantic Value Channel $v^{\text{new}}_{i, j}$

$$v^{\text{new}}_{i, j} = \frac{c^{\text{curr}}_{i, j} v^{\text{curr}}_{i, j} + c^{\text{prev}}_{i, j} v^{\text{prev}}_{i, j}}{c^{\text{curr}}_{i, j} + c^{\text{prev}}_{i, j}}$$

#### Update Confidence Channel $c^{\text{new}}_{i, j}$

$$c^{\text{new}}_{i, j} = \frac{(c^{\text{curr}}_{i, j})^2 + (c^{\text{prev}}_{i, j})^2}{c^{\text{curr}}_{i, j} + c^{\text{prev}}_{i, j}}$$

![[bs-thesis/assets/vlfm__update-value-map.png]]

### Object Detection :luc_box_select:

Different types of pre-trained object detectors which infer bounding boxes with semantic labels are used:
1. **[YOLO](https://docs.ultralytics.com/):** For target objects that fall within the COCO dataset.
2. **[Grounding-DINO](https://arxiv.org/abs/2303.05499):** For all other categories. [:luc_github: IDEA-Research/GroundingDINO](https://github.com/IDEA-Research/GroundingDINO)
3. **[Mobile-SAM:](https://arxiv.org/pdf/2306.14289)** Once the target object is detected, the contours are extracted from the bounding box using the Mobile-SAM model.

> [!done] Target Waypoint Navigation
> - This contour along with the **depth image** is used to point out the object that is closest to the robot's current position (there may be *many instances* of the target object in the environment).
> - Once the robot closes in to a *distance less than the success radius*, it calls `stop()`, ending the episode successfully ✅

### Waypoint Navigation 🚗

**TLDR;**

> Our preference for the **PointNav** policy stems from its speed and ease-of- use; it alleviates several concerns associated with traditional mapping-based approaches, particularly when the waypoint resides outside the navigable area (e.g., when the waypoint is on a target object, which itself may also be on a different obstacle), as navigability of the goal does not affect the policy or its observations.

[[bs-thesis/articles/VLFM-Zero-Shot-Semantic-Navigation.pdf#page=5&selection=25,42,34,27|VLFM-Zero-Shot-Semantic-Navigation, page 5]]
