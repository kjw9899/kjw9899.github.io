---
layout: single
title: "[논문리뷰]A Novel Face Inpainting Approach Based on Guided Deep Learning"
excerpt: ""
categories: GAN
tag: [Image_Inpainting, GAN]
use_math: true
---

> ***Abstract***

지난 몇 년간, 딥러닝은 이미지 인페인팅이 발전하는데 굉장히 중요한 역할을 맡아왔다. 하지만 이미지 인페인팅을 여전히 challenge로 남아있고 특히 face inpainting은 그 중 가장 어려운 challenge로 남아있다. 본 논문에서는 face inpainting에 대한 새로운 접근법을 소개한다. capture가 가능하며 이미지  내에서 각 사람의 identity를 보존할 수 있다. 두 단계의 cascaded model이 제안되는데, inpainting model을 따르면서 얼굴의 keypoint에 대한 shape-predictor로 구성된다. 

* shape-predictor
  * 사람의 얼굴 구조를 보존하는 역할을 함.
  * 훈련 전에 미리 알고 있는 지식을 얻음으로써 훼손된 영역을 채워넣는다.
  * CelebA datasets



## 1. Introduction

* context-encoder
  * 전통적인 방법으로는 context-encoder 방법을 사용하여 훼손된 영역을 채워넣었다.
  * global and local discriminator를 사용하여 성능을 높임.
  * 위의 방법들은 인공적인 느낌이 강하고 낮은 해상도의 이미지에서만 활용되는 단점들이 있다.

* partial convolutions
  * irregular random mask에서도 잘 동작함
  * Deepfillv2 : 마스크를 사용하지 않고 사용자가 직접 마스크를 그려 넣는다. 



> Proposed Model Composed of Two-cascaded Nerworks

1. Face-shape predictor : [Histogram of Oriented Gradients(HOG)](https://donghwa-kim.github.io/hog.html)를 만들어내고 훈련하는 것.
2. Image Inpainting Network : HOG feature를 이용하여 훼손된 이미지를 채워넣는 것.



*HOG는 픽셀의 변화량의 각도와 크기를 고려하여 히스토그램 형태의 feature를 추출하는 방법입니다.*



#### HOG (5 stages)

1. **Global image normalization** : 채색(빛)의 효과를 감소시키기 위해서 전체적인 이미지를 normalization한다.
2. **Computing image gradients** : contour(윤곽)와 맥락적인 정보와 같은 first-order gradient를 연산한다.
3. **Computing of gradient histogram** : image content의 인코딩 구조를 생산한다.
4. **Normalization across blocks** : 셀 단위로 nomalization. 셀은 이미지를 구성하는 작은 부분이다. 이 nomalization block은 HOG와 연관된다.

5. **Flattening into a feature vector** : 마지막 structured image를 만들어 내는 모든 block으로 부터의 HOG descriptor의 집합

#### Image Inpainting network

HOG network에서 얻은 local structure로 훼손된 영역을 채워넣는다.



## 2. Related Work

![image-20220415142802200](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220415142802200.png)

### 2.3 Network Architecture

* followed encoder-decoder
* encoder는 두 번의 down sample 역할을 한다. (stride = 2, dilated = 2, 8 residual block)
* decoder transposed convolution을 통해서 원본 이미지의 해상도를 만들어 낸다. 

* Generator의 모든 layer에 ReLU activation function이 사용된다. 

* Face-Shape predictor의 입력 = GT image + masked image
* Inpainting Network의 입력은 masked imge 하나



### 2.4 Loss Function

#### Face-shape Network

* $L1_{FS}$ reconstruction loss + $L_{adv1}$ adversarial loss
* 입력 : 

$$
I_{gt}\ +\ I_m (I_{gt}\ \odot\ (1-M))
$$

$I_{gt}$ : ground truth 이미지로 구성된 입력

$I_{m}$ : masked된 이미지 입력

$M$ : binary mask

$\odot$ : element-wise product operation

1은 훼손된 영역을 뜻하고, 0 은 그렇지 않은 것을 뜻한다.



generator $G_H$는 HOG feature를 생성하고 훼손 영역을 예측한다.

$$
\begin{aligned}
I_{pred}\ =\ G_H(I_m,\ I_{gt})

$$
그 후, descriminator $D_H$는 HOG feature와 GT image를 비교하여 real or fake를 예측한다.
\end{aligned}
$L1_{FS}$와 $L_{adv1}$는 다음과 같은 공식을 따른다.

$$
\begin{aligned}
L1_{FS}\ &=\ ||I_{pred}\ -\ I_{gt}|| \\
L_{adv1}\ &=\ E_{(I_{gt}\, I_{pred})}[\log D(I_{GT},\ I_{pred})]\ +\ E_{(I_{pred})}[1\ -\ D(I_{pred},\ I_{gt})]
\end{aligned}
$$

그러므로 $L_{FS}$는 다음과 같이 나타낼 수 있다.

$$
\begin{aligned}
L_{FS}\ =\ \lambda_1 L1_{FS}\ +\ \lambda_{adv1}L_{adv1}
\end{aligend}
$$
*lambda는 하이퍼파라미터 값이며 1:1의 비율을 가진다.*



#### Inpainting Network

* 4개의 Loss Function으로 구성
* L1 reconstruction loss(L1), adversarial loss ($L_{adv}$), perceptual loss(L_{$L_{prec}$), a style loss($L_{style}$)

* input pair : first model's $I_{pred}$ + masked images $I_{m}$
* output : recovered inpainted image I_{O}

각 Loss Function의 식은 다음과 같이 나타낼 수 있다.

![image-20220415190332328](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220415190332328.png)

* (4) & (5)
  * $L_{FS}$와 비슷한 형태와 연산
* (6)
  * $\phi_{k}$는 미리 학습된 모델의 k번 째 layer의 activation feature map을 뜻한다.
  * relu1-1, relu2-1, relu3-1 and relu4-1을 사용
  * style loss도 같은 VGG16의 activation layer를 활용
* (7)
  * 만들어진 이미지와 원본 이미지의 유사성을 측정한다.
  * 절댓값 안의 분자는 Gram Matrix(직교 행렬)
  * Gram Matrix shape  $C_k$ x $C_k$는 $H_k$ x $W_k$ x $C_k$의 차원을 가진다. <span style='background-color: #fff5b1'>각 레이어들의 autocorrelation 제공(최근 연구에서 매우 중요하다). 이는 transpose convolutional layer를 통해 만들어진 사진이 인공적인 요소를 줄이기 위한 정확한 도구로 사용된다. </span>

위의 Loss Function을 합하면,

![image-20220415192426892](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220415192426892.png)

앞의 계수들은 하이퍼파라미터이며, 다음 값을 따른다.
$$
\eta_1\ =\ 1,\ \eta_{adv}\ =\ \eta_{adv}\ =\ 0.1\ and\ \eta_{style}\ =\ 250
$$

### 2.5 Training Data

* CelebA-img-algin dataset
* 초기에 256 x 256으로 resize, train : validation : test = 65,554 : 20,000 : 20,000
* irregular mask dataset : 12,000 masks, 마스크 이미지도 256 x 256으로 resize



![image-20220415193914331](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220415193914331.png)



결과를 보면 확실이 Deepfillv1과 디테일 뿐만 아니라 생성 이미지의 퀄리티 차이가 난다.



## Conclusions and Future Work

두 단계에 거쳐 이미지를 생성해내는 새로운 network를 제시했다. 첫 번째 네트워크는 training images로부터 HOG feature를 만들어 내는 것이고, 두 번째인 Inpainting Network는 그 feature를 추출하고 보다 더 나은 결과를 도출한다. 

이로써 더 현실적인 이미지를 만들었을 뿐만 아니라, 얼굴의 형태 또한 유지할 수 있었다. 

또한 차후에, high resolution image를 다루는 model도 꾸준히 연구할 것이다.



