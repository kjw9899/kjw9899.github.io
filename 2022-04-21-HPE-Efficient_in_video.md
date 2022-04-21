---
layout: single
title: "[논문리뷰]Detect-and-Track: Efficient Pose Estimation in Videos"
excerpt: ""
categories: HPE
tag: [Pose Estimation in videos, Human Pose Estimation]
use_math: true
---

> ***Abstract***

본 논문은 multi-person video와 같이 복잡한 상황에서의 키포인트 estimation과 tracking에 대한 주장을 한다. 

매우 가볍고 효율적인 접근을 시도하며, Mask R-CNN과 Carreira의 action recognition을 기반으로 한다. 

모델을 두 단계로 구성된다.

* First : 3D Mask R-CNN, 프레임이나 짧은 클립 별로 kp를 측정한다.
* Second : 비디오 영상에서 생성된 전체적인 kp를 연결한다.

제안 모델의 3D extension은 프레임 별로, 견고한 예측을 더욱 잘 만들어내는 작은 클립에 시간적인 정보의 영향력을 행사하게 끔 한다. 또한 본 논문은 제안 모델의 다양한 선택을 평가하기 위해서, multi-person video 에서 [PoseTrack](https://posetrack.net/)에 대한 광범위한 절제 실험을 진행한다.

* Accuracy : 55.2% on MOTA
* PoseTrack keypoint tracking challenge에서 최신 성능 달성 in ICCV



![image-20220420141133146](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220420141133146.png)



## 1. Introduction

* 이미지 기반의 pose estimation에서는 꾸준한 발전과 모델의 성능향상이 계속되어 왔지만 video에서는 그에 비해 발전이 더뎠다.
* 본 논문은 복잡한 비디오에서 human pose estimation의 문제에 집중했다.
* challenge : 시간에 따른 tracking과 estimation / pose 자체 / 가려지거나 overlap되는 시점

그러므로 pose의 tracker를 구체화하려는 노력과 이미지내에서 객체 단계로 기술을 향상시키는 것은 굉장히 중요하다. 이러한 문제를 해결하기 위해 최신 기술인 Mask R-CNN을 활용하면서 새로운 3D CNN architecture를 이용함으로써 시간 정보의 integration을 통해 이를 확장한다.

[Sliding Window](http://blog.skby.net/%EC%8A%AC%EB%9D%BC%EC%9D%B4%EB%94%A9-%EC%9C%88%EB%8F%84%EC%9A%B0sliding-window/) 방법을 착안하여, 짧은 clip 기반으로 프레임 단계 별로 나눠서 estimation이 진행된다. 이것은 3D 모델의 propagete를 용이하게 해주며, 각 프레임 별로 예측을 더 견고하게 해준다.

*제안 모델인 3D model은  기존 2D baseline 모델의 mAP를 2%를 능가하는 성능을 보였으며, 가장 성능이 좋을 때는 100 프레임을 학습하는데 2분 밖에 걸리지 않았다고 합니다.*



## 3. Technical Approach

* Mask R-CNN을 2D에서 3D로 확장하여 시공간적인 정보를 활용한다. 

* short clips을 인풋으로 사용하고, clip에 있는 공간 정보를 통합하면서 모든 사람의 자세를 예측.

* 프레임 당 detection 수와 비디오의 프레임 수와 관련된 지수함수적인 복잡도를 다루기 위해서 간단한 [heuristic](https://kau-algorithm.tistory.com/7) 알고리즘(당연한 건 신경쓰지 않겠다는 이론!)을 활용한다.

![image-20220421122900245](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220421122900245.png)

### 3.1. Two-Stage Approach for Pose Tracking



#### Stage 1 : Spatiotemporal pose estimation over clips.

* 간단한 formulation과 견고한 성능을 고려하여, Mask R-CNN을 사용한다.
* 2d convolutional kernel을 3D kernel로 확장하여 행동 인식을 수행한 I3D로 부터 영감을 받음. 즉, 3D CNN 사용.
* 모델의 인풋은 single Image가 아닌, T라는 길이의 clip.
* 3D CNN을 통과한 output은 모든 프레임을 통틀어서 kp가 있을 법한 위치의 heatmap.
* 그러므로 3D Mask R-CNN은 일련의 kp estimation 가설 tube이다.

##### Base network

* 모든 2D convolution을 3D convolution으로 대체함으로써 3D ResNet으로 확장.
* kernel size($K_T$): 3 x 7 x 7 (channel x width x height)
* padding = 1 $\rightarrow$ $K_T$ = 3, padding = 0 $\rightarrow$ $K_T$ = 1
* stride = 1,  stride가 높을 수록 성능 저하
* 2D filter weights를 가진 3D kernel, center initialization.
* input의 크기가 (T x H x W)일 때, 3D Network의  최종적인 feature map output은  (T x H/8 x W/8)

##### Tube proposal network

* Faster R-CNN의 [RPN](https://techblog-history-younghunjo1.tistory.com/184)을 착안하여 tube proposal network 설계 (tube classification and regression)
* 모든 sliding 위치 마다 A개의 다른 앵커 박스를 사용. (A는 전형적으로 12). 총 (H/8 x W/8 x A)의 앵커 박스를 얻음.
* classification은 proposal tube와 가장 높은 확률로 오버랩되는 앵커 박스를 예측한다. (softmax classification loss 사용)
* regression은 앵커 박스의 좌표를 출력한다. (L1 loss 사용)
* 2D case의 loss 값과 비교하는 것을 유지하기 위해서 regression loss를 T로 나눈다.

##### 3D Mask R-CNN heads

* 위의 과정으로 후보 지역이 주어지면 분류(c;assify)와 회귀(regress)를 통해 사람을 tracking한다.
* 3D region transform operator를 설계함으로써 region feature를 연산한다.
* base network의 output으로부터 시공간적인 특징을 추출하기 위해 RoIAlign을 확장한다.
* feature map의 시간의 크기 == tube candidate (of dimension T)
* tube를 T 2D boxes로 쪼개고, feature map에서 T 차원의 시간 정보를 추출하기 위해 RoIAlign을 사용한다.
* 그 후의 ouput : T x R x R, R은 RoIAlign 연산의 아웃풋 해상도를 나타낸 것.(R은 7로 유지됨)
* 14개의 keypoint
* classification block은 3D ResNet block으로 구성
* keypoint head $\rightarrow$ 8개의 conv layer + 2개의 deconv layer(keypoint heatmap을 만들어 냄)



#### Stage 2 : Linking keypoint perdictions into tracks.

* detection들이 그래프에 보여진다. 프레임 마다, 사람을 나타내는 detection된 bounding box는 node가 된다.
* 각 프레임의 box를 다음 프레임의 모든 박스로 연결하는 edge를 정의한다.
* 각 edge의 cost는 같은 사람이 속해 있는 edge와 연결된 두개의 box의 negative likelihood로 구해진다.
* likelihood 값이 주어지면, 근접한 프레임 쌍을 매칭시켜 단순화함으로써 tracking을 연산한다.
* 첫 번째 프레임의 tracking을 초기화하고 매칭을 사용하여 label을 propagate한다.

##### Likelihood metrics

1. *Visual similarity* : detection에 의해 추출된 CNN feature 사이에 cosine distance를 적용
2. *Location similarity* : 두 개의 detection box의 IoU를 사용. (IoU= 교집합 영역 넓이 / 합집합 영역 넓이)
3. Pose similarity : 두 프레임 의 pose 사이에 PCKh를 사용한다.



## 4. Experiments and Results

![image-20220421141434607](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220421141434607.png)



#### 4.1 Dataset and Evaluation

* PoseTrack : 514개의 비디오(66,374개의 프레임으로 구성). train : validation : test = 300 : 50 : 208
* training video에는 human body keypoint가 라벨링되어 있는 30 프레임이 딸려 있다.
* validation과 test video에는 중간에 30개의 프레임을 제외하고, 매 4번 째 프레임 마다 라벨링 되어 있다.
* 총 23,000rodml 라벨링된 프레임과 153,615 프레임이 포함되어 있다.ㅙ



* 한 사람 당 15개의 keypoint
* 본 논문이 제안하는 접근이 top-down 방식이기 때문에, labeled keypoint를 확장하고 box의 크기를 20% 확장함으로써 bounding box를 연산한다.
* COCOdataset과 호환 가능한 dataset를 만들기 위해서, COCOdatase에 있는 label과 최대한 동등한 label로 변경한다.
* single-frame pose estimation, pose estimation in video $\rightarrow$ mAP
* Pose tracking in the wild $\rightarrow$ MOT
* PCKh 또한 평가지표로 사용.

##### Thresholding initial detections

Table 1을 보면 알수 있듯이, mAP에서 threshold 값을 내리는 것이 false positive(틀린 검출)을 하는 것을 막는 효과가 있다. 반면 MOTA에서는 threshold 값이 높을 수록 높은 값을 보여준다. 따라서 본 논문은 threshold 값을 0.95로 최종 결정했다.



##### Deeper base networks

프레임 단위의 pose estimation에서는 깊은 base model을 사용하는 것이 더 높은 성능을 보인다. 이는 tracking의 성능과도 직결된다. ResNet-50을 ResNet-101로 대체함으로써 MOTA 성능이 2% 정도 향상되었다. 



##### Matching algorithm

본 논문에서는 Hungarian 알고리즘과 greedy 알고리즘을 사용한다.

* Hungarian 알고리즘 : 주어진 edge cost 행렬을 최적으로 매칭하는 연산 수행
* greedy 알고리즘 : object detection과 tracking에 있어서 평가지표로 널리 사용됨

![image-20220421161531175](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220421161531175.png)

위 Table은 Hungarian 앍고리즘과 greedy  알고리즘을 통해 매칭을 평가한 것이다. 본 논문은 매칭 알고리즘으로 Hungarian 알고리즘을 선택했다.



##### Tracking cost criterion

* IoU
  * 이미지 내에서 bounding box에 대해, (교집합의 영역 / 합집합의 영역)
  * 이 측정법은 사람이 한 프레임에서 다음 프레임으로 거의 이동하지 않다는 것을 예측하며, 이것은 인접한 프레임의 matching box들은 대게 오버랩되었다고 간주한다.
* PCKh
  * Percentage of Correct Keypoints head
  * 같은 사람의 자세가 연속적인 프레임에서 거의 바뀌지 않음을 예상한다.
* ***cosine similarity***
  * 두 벡터 간의 코사인 각도를 이용하여 구할 수 있는 두 벡터의 유사도.

![image-20220421161549538](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220421161549538.png)



##### Upper bounds

linking stage에 대한 한 가지 우려점은 비교적 간단한지와 occlusion, missed detection에 대한 것이다. 하지만 본 논문은 명백한 occlusion handling 없이, 제안 모델이 그러한 문제에 영향을 받지 않는다는 것을 알아냈다.

![image-20220421162405890](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220421162405890.png)

##### Final performance 

![image-20220421162445924](https://raw.githubusercontent.com/kjw9899/kjw9899.github.io/master/kjw9899/kjw9899.github.io/assets/images/image-20220421162445924.png)