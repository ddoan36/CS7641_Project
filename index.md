---
layout: default
title: Real2Craft
---

<div style="margin-bottom: 25px;">
  <a href="#introduction--background" class="mc-btn">Introduction</a>
  <a href="#problem-definition" class="mc-btn">Problem Definition</a>
  <a href="#methods" class="mc-btn">Methods</a>
  <a href="#results-and-discussion" class="mc-btn">Results & Discussion</a>
  <a href="#references" class="mc-btn">References</a>
</div>

---

## Introduction / Background
3D generative models offer a powerful approach to recreating realistic Minecraft worlds. World-GAN [[1]](#ref1) generates arbitrarily sized world chunks from a single example by learning block embeddings (block2vec). VoxelCNN [[2]](#ref2) uses convolutional neural networks to generate houses one block at a time. World2Minecraft [[3]](#ref3) produces detailed indoor scenes directly from RGB images, while DreamCraft [[4]](#ref4) leverages neural radiance fields (NeRFs) for text-guided generation of Minecraft environments. To our knowledge, however, no existing work takes multi-modal imagery as input, such as satellite, ego-vehicle, drone, or hand-held photos.

Several Minecraft-specific sources also exist for recreating real-world locations. Arnis [[5]](#ref5) deterministically converts OpenStreetMap data into Minecraft worlds, though with limited fine-grained detail. BuildTheEarth [[6]](#ref6) is a community-driven project containing thousands of human-made reconstructions of real-world locations.

By drawing on multiple image sources, we hypothesize that our approach can generate more detailed Minecraft worlds than single-source methods, particularly across diverse geographic regions.

---

## Problem Definition
Creating an accurate 3D representation of the world has broad applications including, but not limited to: disaster mapping, robotic simulation, urban planning and geospatial surveying. Given a set of images (RGB, spectral, or depth) from satellite, ego-vehicle, drone, or hand-held photos, our objective is to train a neural network to generate the corresponding 3D scene in Minecraft.

There are many challenges to 3D reconstruction of the open world. Classical photogrammetry is able to reconstruct the visible portion of a scene, but unable to perform scene completion for occluded regions. Faithful reconstruction of any 3D environment requires good camera coverage and using any of the mentioned modalities in isolation leaves out information that is better captured by another format. For example, ego-vehicles may capture ground-level content while drone imagery captures global structure. Additionally, creating a 3D representation of any real-world location from disparate images across various collection modalities is challenging because of open-world conditions and low signal-to-noise ratio in the form of high-frequency dynamic content and low-frequency structural change in environments. Cross-view alignment with large viewpoint variation and scale differences also requires careful mapping to a unified 3D coordinate frame.

We decided to use Minecraft because the game includes a physics engine on top of an in-game structure that is essentially a stylized variation of voxels, a well-studied 3D representation. Minecraft also includes a full embodied-AI stack that is ready to use with our model output. Community driven efforts [[5](#ref5)],[[6](#ref6)] have also created a possible supervision signal for our scene representation, allowing us to perform supervised learning. Voxels with a fixed vocabulary and predefined size (1 m<sup>3</sup>) also allow us to perform more controlled studies.

---

## Methods
**Data Preprocessing:** (1) Timestamp matching: satellite and ego vehicle images are captured at different times, so each satellite tile is paired with spatially overlapping ground images of Manhattan from a short time window. (2) Camera pose: ground images carry only GPS position and heading, so 6 DoF poses are recovered through cross view geolocalization. (3) Dynamic content: pedestrians, vehicles, and construction are absent from the ground truth and are flagged by a vision language model. (4) Image quality: noisy or low quality frames are scored and removed by the same model. (5) Ground truth: voxels are derived from Arnis [[5]](#ref5) and BuildTheEarth [[6]](#ref6), whose builders take creative liberties; we filter misaligned regions and consolidate the block vocabulary.

**Data Visualization:** We will compare block class histograms across training and test splits to verify consistency, and project block2vec embeddings [[1]](#ref1) with PCA and t-SNE to examine whether structural blocks (cobblestone, wood) separate from decorative blocks (flowers, fences).

**Model:** Cross-View Splatter [[7]](#ref7) is a feed forward network that fuses satellite and geotagged ground images to predict pixel aligned Gaussian splats in a shared 3D frame. We retain its pretrained encoder and cross view fusion, initialized from VGGT [[8]](#ref8), and replace only the Gaussian head with a sparse 3D CNN voxel decoder (Figure 1). The decoder predicts block class probabilities over a 64 x 64 x 32 grid of 1 m³ voxels. Following Atlas [[9]](#ref9), class weighted cross entropy is computed only over observed voxels, excluding building interiors. Larger scenes are reconstructed by merging overlapping chunks.

---

## Potential Results and Discussion
**Metrics:** We evaluate against held out ground truth voxels using (1) occupancy IoU for geometry, (2) mean IoU across block classes for semantics, and (3) per class precision, recall, and F1 to expose rare block failures. A human rating study will score rendered worlds against reference photos.

**Goals and Expected Results:** We expect the fused model to outperform satellite only and ground only ablations on all metrics, with the largest gains on facades and occluded regions. Reusing the pretrained encoder limits compute and energy use. Filtering pedestrians and vehicles protects privacy, and we will note the bias of training on Manhattan alone.

<p align="center">
  <img src="assets\images\Screenshot 2026-10-06 163030.png" alt="Proposed Model Architecture" width="85%" style="border: 3px solid #000; box-shadow: inset -2px -2px 0px 0px #373737, inset 2px 2px 0px 0px #8e8e8e;">
  <br>
  <em>Figure 1: Proposed Model Architecture.</em>
</p>

---

## References
1. <a id="ref1"></a>M. Awiszus, F. Schubert, & B. Rosenhahn, "World-GAN: a Generative Model for Minecraft Worlds," *IEEE Conference on Games (CoG)*, arXiv:2106.10155, 2021.
2. <a id="ref2"></a>Z. Chen et al., "Order-Aware Generative Modeling Using the 3D-Craft Dataset," *2019 IEEE/CVF International Conference on Computer Vision (ICCV)*, Seoul, Korea (South), 2019, pp. 1764-1773, doi: 10.1109/ICCV.2019.00185.
3. <a id="ref3"></a>L. Zhang, H. Xu, J. Gong, X. Wang, Y. Xie, & X. Tan, "World2Minecraft: Occupancy-Driven Simulated Scenes Construction," *2026 International Conference on Learning Representations (ICLR)*, Rio de Janeiro, Brazil, arXiv:2604.27578, 2026.
4. <a id="ref4"></a>S. Earle, F. Kokkinos, Y. Nie, J. Togelius, & R. Raileanu, "DreamCraft: Text-Guided Generation of Functional 3D Environments in Minecraft," arXiv:2404.15538, 2024.
5. <a id="ref5"></a>L. Erbkamm, "Arnis: Bring the Real World into Minecraft," Arnis. [Online]. Available: https://arnismc.com/.
6. <a id="ref6"></a>"Building The Earth In Minecraft," BuildTheEarth. [Online]. Available: https://buildtheearth.net/.
7. <a id="ref7"></a>M. Turkulainen et al., "Cross-View Splatter: Feed-Forward View Synthesis with Georeferenced Images," *2026 Conference on Computer Vision and Pattern Recognition (CVPR)*, Denver, CO, USA, arXiv:2605.19656, 2026.
8. <a id="ref8"></a>J. Wang et al., "VGGT: Visual Geometry Grounded Transformer," *2025 Conference on Computer Vision and Pattern Recongiiton (CVPR)*, Nashville, TN, USA, arXiv:2503.11651, 2025.
9. <a id="ref9"></a>Z. Murez et al., "Atlas: End-to-End 3D Scene Reconstruction from Posed Images," *2020 The European Conference on Computer Vision (ECCV)*, doi: 10.1007/978-3-030-58571-6_25.

---

## Gantt Chart

---

## Contribution Table