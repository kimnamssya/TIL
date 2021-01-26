## 1. RCNN

[Rich feature hierarchies for accurate object detection and semantic segmentation](https://arxiv.org/pdf/1311.2524.pdf)



#### **R-CNN process**

1. **Region proposal**

   object가 있을만 한 지역을 selective search 기법을 통해 proposal한다. 

   > selective search - segmentation을 우선적으로 수행하고 이를 통해 후보가 될 object를 추출하는 방법

2. **CNN**

   (1)에서 제안된 region을 잘 튜닝된 CNN에 각각 통과시켜 Fixed-length feature vector를 추출한다.

3. **Classify**

   추출된 Fixed-length feature vector를 입력으로 하는 분류기(SVM)를 통해 분류한다.



그러나 제안된 Region마다 CNN을 수행하므로 속도가 느리고, 비용이 크다는 단점이 존재한다. 그래서 제안된 방법이 Fast R-CNN 이다.



## 2. Fast R-CNN

[Fast R-CNN](https://arxiv.org/pdf/1504.08083.pdf)



#### 1. Contributions

1. R-CNN, SPPnet에 비해 더 높은 detection quality
2. Single-stage로 학습(proposal과 detection을 한번에)
3. 학습 시 모든 layer를 학습(SPPnet은 CNN layer를 학습시킬 수 없음)
4. Feature caching에 요구되는 디스크 storage가 없음



![fast_rcnn](https://user-images.githubusercontent.com/48755185/105841944-d1300980-6018-11eb-93f3-48f07d721f67.JPG)



#### 2. Fast R-CNN architecture and training

1. 전체 이미지와 Object proposal set을 입력으로 받는다.

2. Conv, max pooling layer에 전체 이미지를 입력으로 넣어 feature map을 추출한다. 

3. 각각의 Object proposal에 대해 RoI pooling layer가 fixed-length feature vector를 추출한다.

4. 두 개의 output layer로 분기되는 fully connected layer에 fixed-length feature vector를 입력으로 넣는다.

5. 하나의 output은 softmax 확률을 제공한다.

6. 다른 하나의 output은 네 개의 real value를 제공하고, 이는 bounding box의 좌표이다.

   

![pooling](https://user-images.githubusercontent.com/48755185/105841951-d4c39080-6018-11eb-80c2-29153e8ee240.JPG)


##### 2-1. RoI pooling layer

feature map에 RoI(h X w)를 투영해서 H X W 크기의 고정된 작은 feature map으로 바꾼다. 기존의 R-CNN에서는 제안된 Region이 각각 CNN을 통과한 뒤 SVM으로 들어가고, bounding box regressor에는 CNN을 거치기 전의 region proposal 데이터가 입력으로 들어가므로 연산이 공유되지 않았다. 하지만 Fast R-CNN에서는 RoI pooling layer를 통과한 같은 feature map이 SVM, boungding box regressor의 입력으로 들어간다(연산을 공유한다.). 이러한 특징으로 모델을 end-to-end로 한번에 학습시킬 수 있다.

##### 2-2. Initializing from pre-trained networks

pre-trained 된 networks가 Fast R-CNN network를 초기화 할 때, 세 가지 변환이 존재한다.

1. last max pooling layer가 RoI pooling layer로 대체된다. 
2. last fully connected layer와 softmax가 softmax와 bounding box regressor로 대체된다.
3. 두 가지 input(image list와 RoI list)을 가지는 network로 대체된다.

#### 3. Fast R-CNN detection

network는 이미지와 대략 2000개의 object proposlas을 입력으로 받는다.

##### 3-1. Truncated SVD for faster detection

Large fully connected layer는 truncated SVD를 통해 압축함으로써 가속화 될 수 있다.



하지만 여전히 Region을 생성하는 selective search 과정이 병목현상의 원인이 되므로, RoI 생성마저 CNN 내부에서 이루어지는 Faster R-CNN이 나오게 된다.



## 3. Faster R-CNN

[Faster R-CNN](https://arxiv.org/pdf/1506.01497.pdf)

![faster_rcnn](https://user-images.githubusercontent.com/48755185/105841962-d8efae00-6018-11eb-9853-adce07e2b4b0.JPG)

 ##### 1. Introduction

Fast R-CNN이 real-time에 가까운 성취를 이뤘지만(proposal 과정을 제외하고), selective search 방식을 사용하는 region proposal은 여전히 많은 running time을 소비한다. region proposal 방법은 CPU에서 수행되기 때문에 매우 느리다. 따라서 본 논문에서는 region proposal을 deep convolutional neural network에서 계산되도록 만들어  거의 cost-free 하도록 만든다. 최신 object detection network와 convolution layer를 공유하는(feature map을 공유한다.) 이 network를 region proposal networks(RPN)이라 부른다.  즉, Faster R-CNN은 RPN과 Fast R-CNN의 합으로 이루어져 있다.

##### 2. Region Proposal Networks

RPN은 직사각형의 이미지를 input으로 받고 직사각형의 output을 낸다. 각각의 proposal은 objectness score를 가진다. 이 과정은 fully convolutional network에 의해 이루어진다(convolution layer에 의해 생성된 feature  map에 fully convolutional network를 slide 시킨다.).

![anchors](https://user-images.githubusercontent.com/48755185/105841970-dbea9e80-6018-11eb-98ea-fbf61388c930.JPG)

##### 3. Anchors

slide된 window(nxn)는 각 위치마다 최대 k개의 proposal을 만들어 낸다. 따라서 reg layer(regression layer)에는 4개의 좌표값 * k = 4k개의 output이 나오고, cls layer(classification layer)에는 2k개의 output이 발생한다(object일 확률과 not object일 확률). 각각의 proposal이 anchor이며 WxH의 feature map에서 WHk개의 anchor가 발생한다.

##### 4. Loss funtion

RPN을 학습할 때는 IOU 가 0.7 이상일 때 positive로 레이블링하고, 0.3 이하일 때 negative로 레이블링 한다. 따라서 0.3과 0.7 사이는 학습에 영향을 끼치지 못한다. 

## 4. YOLO(You Only Look Once)

[YOLO](https://arxiv.org/pdf/1506.02640.pdf)

 ##### 1. Introduction

기존의 R-CNN은 region proposal 을 통해 bounding box를 제안하므로 느리고 각 절차가 독립적이므로 최적화 하기에도 힘들다. YOLO는 bounding box, class probabilities를 하나의 파이프라인으로 빠르게 구해준다(end to end). 따라서 이미지를 한 번만 보면 객체 검출이 가능하므로 YOLO 라는 이름이 붙게 되었다.

YOLO는 sliding window나 region proposal 방식과 달리 이미지 전체를 보기 때문에 주변 정보까지 처리가 가능하다. 하지만 region proposal 방식을 사용하는 Fast R-CNN 모델은 주변 정보를 처리하지 못하기 때문에 배경에 반점, 노이즈가 있으면 그것 또한 물체로 인식하게 된다(background error). 

하지만 YOLO는 속도가 빠른 대신, SOTA 모델(최신 객체 검출 모델)에 비해서 정확성이 떨어진다는 단점이 있다. 

##### 2. Unified Defection

YOLO는 입력 이미지를 S x S grid로 나누고 각 grid에 대해서 confidence score를 예측한다. confidence score는 bounding box가 물체를 포함하는 것을 얼마나 믿을 수 있고 정확한지에 대한 score이며, 실제 box와 예측 box의 합집합과 교집합의 비(IOU)가 같을 때 confidence score는 1로 나타난다. 

각각의 bounding box는 x, y, w, h, confidence score로 나타낸다. x, y는 w, h가 1이라고 했을 때 box의 중심을 상대 좌표로 나타낸 것이고, w, h는 이미지 전체의 높이, 너비가 1이라고 했을 때, 상대적인 높이와 너비이다.

각각의 grid는 conditional class probabilities 를 계산한다. 이는 객체가 grid안에 있다는 조건 하에 그 객체의 class에 대한 확률이다. 이 conditional class probabilities에 confidence score를 곱해 각각의 box에 대한 class specific confidence score을 구한다. 이는 box안에 나타나는 class의 확률과 예측된 box가  얼마나 객체에 잘 들어맞는지에 대한 score이다.

##### 2.1 Network Design

YOLO의 신경망 구조는 GoogLeNet에서 따왔고 convolution 계층에서 이미지의 특징을 추출하고, fully connected layer에서 클래스 확률, bounding box의 좌표를 예측한다.

##### 2.2 Training

YOLO 신경망의 최종 예측값은 class probabilites와 bounding box의 x, y, w, h 정보이다. 마지막 활성화 함수 계층에는 leaky ReLU를 사용한다. leaky ReLU는 0 이하의 값일 때 작은 음수 값을 가진다(ReLU는 0 이하의 값일 때 0). 손실 함수는 SSE(sum-squared error)를 기반으로 하여 최적화 하기 쉽게 만들었으나 mAP를 높이는 것과 완벽하게 맞지는 않다. localization error와 classification error에 대해 동일하게 가중치를 부여하기 때문이다. 또한, 대부분의 grid에는 객체가 없기 때문에 confidence score가 0으로 학습되어 모델이 불안정하게 될 수 있다. 

이를 해결하기 위해 예측된 bounding box의 좌표로 부터의 loss 가중치를 증가시키고, 객체가 존재하지 않는 box에 대한 confidence 값의 loss 가중치는 감소시켰다. 또한 SSE는 크기가 큰 box와 크기가 작은 box에 동일한 가중치를 부여한다. 크기가 작은 box는 크기가 큰 box보다 변동이 크기 때문에, 박스의 w와 h값에 square root를 적용해 box의 크기가 커질수록 증가율은 작아지도록 만들었다. 

YOLO는 하나의 객체 당 하나의 bounding box가 매칭되도록 만들어야 하므로, 여러 예측 bounding box 중 실제 bounding box와의 IOU가 가장 큰 것을 선택한다. 

##### 2.3 Inference

YOLO의 grid에서는 하나의 객체를 여러 grid가 동시에 검출하는 multiple detection 문제가 발생할 수 있다.  따라서 이를 억제하기 위해 non-maximal suppression 방법을 사용하여 mAP를 향상 시켰다.

##### 2.4 Limitations of YOLO

YOLO는 하나의 grid가 오직 하나의 객체만 검출하므로, 하나의 grid에 두 개 이상의 객체가 존재하는 경우 이를 잘 검출하지 못한다는 단점이 있다. 또한 Training 단계에서 학습하지 못했던 새로운 aspect ratio를 보면 고전한다. 마지막으로,  큰 bounding box와 작은 bounding box의 loss에 동일한 가중치를 두기 때문에 부정확한 localization이 발생한다(square root를 통해 개선하긴 했지만).
