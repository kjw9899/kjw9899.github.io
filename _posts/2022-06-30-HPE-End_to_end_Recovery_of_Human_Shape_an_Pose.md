---
layout: single
title: "[논문리뷰]End-to-end Recovery of Human Shape and Pose"
excerpt: ""
categories: what is the category's name? 
tag: [3DHPE, HPE]
use_math: true
---

![image-20220630131044514](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220630131044514.png)

> Abstract

* 본 논문에서 소개하는 SMPL 기반의 Human Mesh Recovery는 single RGB 이미지를 body keypoint 3D mesh로  재구성한다. 이때, 다른 가지 framework 없이, end-to-end의 종단간 학습을 통해 1 stage로 진행된다.
* 이 모델의 주된 목적으로는 keypoints loss를 최소화하는 것이며, in-the-wild한 사진에서 사용하는 것을 목표로 하고 있다.

