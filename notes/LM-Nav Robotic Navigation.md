---
tags:
  - articles/research
  - notes
---
> [!help] Resources
> **Article**
> [[LM-Nav_Shah-Osinski.pdf|Shah, LM-Nav: Robotic Navigation with Large Pre-Trained Models of Language, Vision, and Action]]


> [!quote] Abstract
> - Goal-conditioned policies for robotic navigation can be trained on large, unannotated datasets, providing for good generalization to real-world settings.
> - However in **vision-based** settings, where specifying goals requires an image, this is an unnatural interface.
> - **Language** provides a more convenient modality for communication with robots.
> - Contemporary methods require extensive supervision, in the form of *trajectories annotated with language descriptions*.
> - **LM-Nav** provides a **high-level interface** to the user.
> - Instead of using a labelled dataset, it is shown that such a system can be constructed entirely out of:
> 	- Pre-trained models for **navigation** (*ViNG*)
> 	- **Image-Language** association (*CLIP*)
> 	- **Language modelling** (*GPT-3*)
> - This does not require any *fine-tuning* or *language-annotated robot data*.
> - **Overview of method**
> 	- Extract **landmark names** from instruction
> 	- **Grounds landmarks in real-world** using image-language model
> 	- **Navigate** to the landmarks using vision-only navigation model

---
# [[LM-Nav_Shah-Osinski.pdf#page=1&selection=60,0,60,12|🔗]] Introduction 

## [[LM-Nav_Shah-Osinski.pdf#page=1&selection=62,0,84,78|🔗]] Current Challenges

- Enable robots to perform wide variety of tasks on **command**.
- Follow **high-level** instructions from human.

### Requirements

- Robots need to **understand** human instructions.
- Need a large collection of **diverse behaviours** to execute these instructions.

### Prior Work

- Annotated trajectories, can understand text instructions, but impedes adoption due to annotation cost.
- Goal-conditioned policies trained with self-supervision utilize large unlabeled datsets to train vision-based controllers via *hindsight-relabeling*.
- They provide stability, robustness and generalization but involve a clunky mechanism for goal specification, using location or images.

> This work aims to combine the strengths of both approaches to enable robotic navigation to execute natural language instructions.

## [[LM-Nav_Shah-Osinski.pdf#page=1&selection=85,0,100,94|🔗]] Off-the-Shelf Pre-Trained Models

> Our main observation is that we can utilize off-the-shelf pre-trained models trained on large corpora of visual and language datasets — that are **widely available** and show great **few-shot generalization capabilities** — to create this interface for ==**embodied instruction following**==
>
> [[LM-Nav_Shah-Osinski.pdf#page=1&selection=85,0,91,79|LM-Nav_Shah-Osinski, page 1]]

### Combination of Models

- To create the interface mentioned above, the authors suggest the combination of ==2 Robot-Agnostic Models== + ==1 Navigation Model== (Both pre-trained).
- **Vision-Navigation Model (VNM):** Create a topological *mental map* of the environment, using robot's observation from **prior exploration of the environment.**
- **Language Model (LM):** Decode **free-form text instructions** into a **sequence of textual landmarks**.
- **Vision Model:** *Grounding* the textual landmarks in the topological map, inferring a joint-likelihood over the landmarks and the nodes.

> Reducing a language task to a combination of grounding and subgoal selection discards a lot of useful cues like relations and verbs.


