# LLM Based Zero Shot Object Navigation

Can Embodied AI find your "Cat-Shaped Mug"?
by *Dorbala, Mullen, Manocha* - UMD

---

# Goal

> [!important] Object Navigation
> Find a **uniquely described target** in a **previously unseen environment**.

---

# Problems

---

## Descriptive targets

- AI trained on images in surroundings
- ==Target== for robot in training => ==Image==
- **Real-World**
	- **Human:** *Gives description of target obejct*
	- **Robot:** 🥲 Sorry

---

## Unknown Environments

- Often robot needs to
	- **Navigate** unknown environments
	- **Locate** previously unseen/uncommon objects *e.g.* "Cat-Shaped Mug"
- Cannot perform well in such cases 😕

---


> [!info] What the AI needs to do
> AI needs to figure out ==what the goal object must look like==, from the **text description** and navigate to find it

---
# What is L-ZSON?

The problem here belongs to a class of problems called =="Language-Based Zero-Shot Object Navigation"==

---

## Language-Based

Uses language models with often variable modalities (text 📝, image 🏞️, audio 📢)

---

## Zero-Shot

The agent is able to ==find objects it has seen _zero_ times previously== in ==environments it has navigated _zero_ times previously==. 🙈

---

![[first-time-meme.jpg]]

---


## Object Navigation

Given the ==description of the _object_==, the agent needs to ==_navigate_== to it.

---

![[everywhere-i-go.jpg]]

---

# Solution

==Language Guided Exploration== (LGX)

![[llm-zson__how-to-find-cat-shaped-mug.png]]

---

- **Large Language Models:** The **commonsense and reasoning capbility** of LLMs are used to make ==sequential navigation decisions==.
- **Vision-Language grounding models:** Generalized target-object detection; *bridges the gap* between what the ==robot sees in the surrounding== and the ==description of what the robot wants to find==.

---

### Breaking it down

L-ZSON consists of two parts:

| Sequential Decision Making          | Target Object Grounding                                                      |
| ----------------------------------- | ---------------------------------------------------------------------------- |
| ==Exploratory decisions== on-the-go | ==Locate== the target object from _agent-perception_ (what the agent *sees*) |

---

# Contributions

---

### L-ZSON with Unconstrained Language

- Can locate target objects ==described by unconstrained (freeform) language==
- Leverage the ==semantic connections== and ==relationships between objects== learned by **language models**

---

### Scene Descriptions into Prompts

- Robot sees 👁️ -> 🏞️
- 🏞️ -> **VLM** -> 📝 Description of what is being seen
- Navigation done based on this description and the target object's description

---

![[binocular-meme.jpg]]

---

# Methodology

---

![[llm-zson__methodology.png]]

---

## Overview

![[llm-zson-method-flow.png]]

---

# Scene Understanding

---

## Environment Observation

- $r$ = Scene image resolution (degree)
- $I_r$ = RGB images of scene
- $I_d$ = Depth images of scene

$$n_{img} = \frac{360 \degree}{r}$$

- 🎨 **RGB Images** used to form ==contextual cues==
- 📏 **Depth Images** used to construct ==2D costmap== for navigation

---

### Context Cues

![[rgb_image-flow.png]]

---

> [!important] Importance of Rotation-In-Place
> - Allows the agent to ==fully observe surroundings==, giving the **LLM enough information** to make a ==full informed navigation decision== from the agent's current position in the environment.
> - Without this step, the agent would proceed towards seen objects over unknown space, even if **none of the objects in the scene are related to the target object**.

---

### 2D Costmap

While performing the [[#Environment Observation|rotate-in-place]], the agent uses the depth images $I_d$ from the scene to construct a costmap of the environment. *RTABMAP is used here*.

> RTABMAP uses **visual correspondence** along with **depth information** to compute the 2D costmap.

---

### Navigation

The agent uses the LLM's decision and the 2D Costmap to navigate and explore 


![[costmap-nav-loop.png]]

---

### Target Object Grounding

> GLIP is used for target object grounding

![[object-grounding.png]]

---

# Intelligent Exploration with Language Models

---

## Object Detection

**YOLO -> LLM:**
- YOLO -> List of objects $O$
- $O$ used to synthesize a **prompt**
- Prompt passed to LLM to make **navigation decision -> ==OBJECT to go towards==**
- $r =$ lower value here

---

## Image Captioning

**BLIP -> LLM:**
- BLIP -> Image captions $C$
- Captions passed in prompt to LLM to make **navigation decision -> ==DIRECTION to go in==**
- $r = 90 \degree$ here to allow the agent to explore in `left, right, front, back` directions

---


> [!NOTE] What to do when no matches?
> If output from LLM **does not match** any of the expected outcomes, the agent chooses a random direction and proceeds with the point choosing step

---

![[no-matches.jpg]]

---

![[random-bs-go.jpg]]

---

# Analysis

---

## GLIP

Grounded Language-Image Pre-Training

$$\{o_{t, i}, b_{t, i}\} = GLIP(I_t, P_o)$$

---

## LLM Prompts

Different LLM prompting strategies are studied in the article. The types of prompts are *Robot-Prompt, I-Prompt, 3rd-Person-Prompt, First-Prompt, Get-Closest-Prompt, ONE-word-First-Prompt, BLIP-Prompt*

---

> [!NOTE] BLIP Prompt
> “I want to find a $o_g$ in my house. In Front of you there is caption. To your Right, there is  caption. Behind you there is caption. To your Left there is  caption. Which direction from Front, Right, Behind, Left should I go towards? Reply in ONE word.”

---

### Prompt Success Rate

$$PSR = \frac{p_{success}}{p_{total}}$$
- $p_{success}$ = No. of instances where valid response chosen by LLM
- $p_{total}$ = Total no. of times agent prompts the LLM

---

# Results

---

![[l-zson__compare.png]]

---

![[l-zson__prompts_success.png]]

---

![[l-zson__object_success.png]]

---

# Thank you