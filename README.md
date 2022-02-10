# 페 CT 사진을 이용한 코로나 여부 판별

## 모델링에서 사용한 방법

### 정확한 모델 평가를 위해 Random seed를 고정 ==> 2022

### 이미지 사이즈를 조정 ==> 128, 384(최대)

### lr scheduler를 조정
 * OneCycleLR
 
 *

### Accuracy를 높이기 위한 여러 모델 설정: Resnet50, Alexnet, Custom한 CNN
 * Resnet50: 사전학습 되지 않은 모델을 사용하여 Train함 마지막 num class를 1로하고 sigmoid와 BCELoss를 활용한 학습

 * Alexnet50: 사전학습 되지 않은 모델을 사용하여 Train함 마지막 num class를 1로하고 sigmoid함수와 BCELoss를 활용한 학습

 * Custom CNN : num class를 1로하고 sigmoid와 BCELoss를 활용한 학습
### Img Augmentation : 
* HorizontalFlip

* 랜덤한 두 사진을 뽑아 수직, 수평 결함 : https://github.com/TaeTaeKim/Covid_CT_Classification/blob/Covid/make_newimg.ipynb
 - 1. split된 데이터 셋에서 train에만 랜덤 결합한 사진을 2000정도 늘림
 - 2. 전체 데이터를 20000정도로 나누고 split