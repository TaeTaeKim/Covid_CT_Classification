# 페 CT 사진을 이용한 코로나 여부 판별

## 모델링에서 사용한 방법

### 정확한 모델 평가를 위해 Random seed를 고정 ==> 2022

### 이미지 사이즈를 조정 ==> 128, 384(최대)

### 최저 validation loss와 Best accuracy모델 두가지를 저장하여 예측결과 도출

### Img Augmentation : 


#### 랜덤한 두 사진을 뽑아 수직, 수평 결함 : https://github.com/TaeTaeKim/Covid_CT_Classification/blob/Covid/make_newimg.ipynb

#### 기본적으로 train val 개수는 각각 600,40개정도

 #### 첫 번째 :
  - 원본 train data 만을 이용해서 image augment적용,augmetation은 약 2000장정도 진행
  - 하지만 train data만 많이 늘어나서 그런지 과적합이 빨리 일어났음.
 
 #### 두 번쨰:
 - val에 원본사진 400장정도를 넣고 나머지 200장으로 augment를 진행 train1200장 val400장 정도로 실험을 진행

### Accuracy를 높이기 위한 여러 모델 설정: Resnet50, Alexnet, Custom한 CNN
 #### Resnet50: 사전학습 되지 않은 모델을 사용하여 Train함 마지막 num class를 1로하고 sigmoid와 BCELoss를 활용한 학습

 #### Alexnet50: 사전학습 되지 않은 모델을 사용하여 Train, num class를 1로하고 sigmoid함수와 BCELoss를 활용한 학습
 - image agument를 이용, 학습 데이터를 2만개로 늘리고 학습했을 때 가장 높은 accuracy를 이용 ==> 0.8의 정확도
 - 과적합을 막기위해 bottleneck의 최종 conv이후에 dropouit2d 0.25를 이용,(downsample layer에는 미적용) 규제가 너무 강했는지 Train data loss도 쉽게 낮아지지 않았음.

 #### Custom CNN :
 - 기존 baseline모델을 그대로 사용했을 경우에는 0.76의 성능을 보임.
 - batch norm이나 dropout없이
 
 #### GoogLeNet:
 - num class를 1로 하고 모델에 맞게 Train함수를 수정함 <u>3개의 loss중에서 aux1, aux2 loss에 0.3의 가중치를 두고 loss를 계산.</u>
 - 첫번째 시도에서는 모델 num class만 수정하고 모델을 그대로 진행 ==> 과적합이 일어나고 val loss가 떨어지지 않음
 - 두번째 시도에서는 dataset을 전체 15000개까지 upsample하고 split하여서 진행 ==> 8.33의 정확도달성
 - 할것 ! inception사이에서 전해주기전에 dropout을 적용 후 train upsample dataset으로 돌리기
 - 할것 ! dropout된 모델을 full upsample dataset으로 돌리기
 - 할것 ! image aug를 사각형으로 만들기
 - 최종 pred는 output을 이용한 계산.
 
 ### 그외 파라미터 조정
 #### seed값을 조정해봄
 #### threshold를 조정해봄 기존 0.5에서 0.4~0.6사이값으로 조정
