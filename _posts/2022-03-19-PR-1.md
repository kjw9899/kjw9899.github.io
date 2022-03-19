---
layout: single
title: "[논문 리뷰]Learing Spatiotemporal Features with 3D Convolution Networks"
excerpt: ""
categories: Classification
tag: [3DCNN, Classification]
use_math: true
---

Paper : https://arxiv.org/pdf/1412.0767.pdf

Pytorch documentation : https://pytorch.org/docs/stable/generated/torch.nn.Conv3d.html



### Abstract

1. 3D ConvNets은 시공간적인 특징을 학습하기에 2D ConvNet보다 적합하다.
2. 구조적인 특징으로는 3x3x3의 크기를 가진 kernel이 모든 layer에 적용된다.
3. 학습된 feature인 단순 선형 분류를 가진 C3D는 최첨단의 방법인 4개의 다른 벤치마크를 능가하며, 다른 2개의 벤치마크와 비교할 수 있다.



### Introduction

 video data는 용량도 크고 양도 많기 때문에 video descriptor에 대한 4가지 특성이 요구 된다.

>  4 properties for an effective video descriptor

* ***It needs to be <span style ="color: red"> generic</span>, so that it can represent different types of videos well while being discriminative.***
  - classification을 수행할 때, 다른 종류들의 무수히 많은 비디오를 잘 표현하기 위해서 충분히 일반적일 필요가 있다. 
* ***The descriptor needs to be <span style ="color: red"> compact</span>.***
  * 진행, 저장, 일처리를 더 큰 범위에서 수행 가능해야한다.
* ***It needs to be <span style ="color: red"> efficient</span> to compute, as thousands of videos are expected to be processed every minute in real world system.***
  * 기존 이미지 분류 작업에서는 time domain이 존재하지 않았다. 하지만 C3D에서는 time이라는 dimension이 더 추가된 것이기 때문에 당연히 연산량이 늘어 날 수 밖에 없다. 그러므로 연산에 대한 효율성을 고려할 필요가 있다.
* ***It must be <span style ="color: red"> simple</span> to implement.*** 
  * 동작 여부에 대해 간단해야 한다.



***PS.*** 시공간을 다루는 대표적인 딥러닝 모델로는 RNN, LSTM 등이 있다. 하지만 모델이 매우 무겁기 때문에 넉넉한 서버용량과 좋은 GPU를 요구한다. C3D는 그들보다 연산량이 적다는 단점이 있기 때문에 모델 경량화에 대한 장점이 있다고 생각한다.



### Dataset



> **UFC101** 

![image-20220319183353536](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220319183353536.png)





Dataset으로는 [UFC101](https://www.crcv.ucf.edu/data/UCF101.php)을 사용했다. 대표적인 행동 인식 데이터셋이며, Youtube에 있는 짧은 동영상들을 101개의 카테고리로 분류한 것이다.



### 3D Convolution

#### Input of 3D Convolution

![image-20220319183947872](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220319183947872.png)



> Input video clips

- $c$ : number of channels
- $l$ : length of number of frames
- $h, w$ : height of number of frames

> Pooling kernel

* $d$ : kernel depth (temporal)
* $k$ : kernel spatial size (spatial)



#### Common Network Setting

![image-20220319184903273](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220319184903273.png)

* kernel size of pool1 : 1x2x2
* Padding : appopriate
* stride : 1
* kernel size : 2x2x2 (except for first layer)
* Mini-batch : 30 clips
* Learning rate : 0.003
* lr is divided every 4 epochs, stopped after 16 epochs



### Experiments

depth-d 실험 : temporal depth of d equal to 1,3,5,7 **(d x k x k)**

* 증가 : 3-3-5-5-7
* 감소 : 7-5-5-3-3

![image-20220319185600822](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220319185600822.png)

증가와 감소 모두 마지맏 pooling layer에서 아웃풋 크기가 같기 때문에 fc에 대한 파라미터 수가 동일하다.

depth-3가 가장 뛰어난 성능을 보인다.



### Conclusion

* video 분석에서 시공간적인 features를 학습하는 것이 가능하다.
* 다양한 비디오 분석에 있어서 외관과 움직임 정보를 동시에 분석할 수 있으며, 2D Convnet을 능가할 것으로 보임.
* Linear classifier를 통해 추출된 C3D feature들이 다른 비디오 분석 벤치마크에서도 현재까지 가장 좋은 방법인 것을 증명했다.
* C3D features는 efficient, compact하고 사용하기에 simple하다.

















