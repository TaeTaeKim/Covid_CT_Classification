# 페 CT 사진을 이용한 코로나 여부 판별

## 모델링에서 사용한 방법

1. 정확한 모델 평가를 위해 Random seed를 고정 ==> 2022

2. 이미지 사이즈를 조정 ==> 128, 384(최대)
    - 이미지를 Resize로 줄이지 않고 높게 잡을수록 정확도는 향상되는 듯
    - 하지만 이미지가 커질수록 gpu memory에러가 날 확률이 높아 batch size가 줄어들고 학습시간이 늘어남.

3. 최저 validation loss와 Best accuracy모델 두가지를 저장하여 예측결과 도출

4. Img Augmentation

5. 다양한 모델 사용

## Image Augmentation

- 기본적으로 train val 개수는 각각 600,40개정도

- 랜덤한 두 사진을 뽑아 수직, 수평 결함 : https://github.com/TaeTaeKim/Covid_CT_Classification/blob/Covid/make_newimg.ipynb
    - label이 같은 사진을 각각 2장씩 뽑아서 수직결합으로 한장, 수평결합으로 한장을 생성

- 첫 번째 방법
   - 원본 train data 전체를 이용해서 image augment적용,augmetation은 약 20000장정도 진행
 
 - 두 번쨰 방법
   - val에 원본사진 400장정도를 넣고 나머지 200장으로 augment를 진행 train1200장 val400장 정도로 실험을 진행

## 다양한 모델 사용

### Resnet50:
    - 대회 규정에 의해 Pretrain되지 않은 모델을 불러와서 사용
    - 모델을 조금 수정하여 num class를 1로하고 output에 sigmoid함수를 적용 Binary Cross Entropy를 이용한 학습을 진행
    - 20000장의 Augment된 img를 학습하여서 0.8의 성능을 보임. 나머지 조정에서는 이하의 정확도
    - 과적합을 막기위해 bottleneck의 output마다 dropout2d를 0.25씩 줌.

### Alexnet50:
    - 역시 사전학습 안된 모델 사용, num class와 loss를 Resnet50과 동일하게 설정 후 학습
    - 역시 20000장의 학습데이터로 했을 때 가장 좋은 성적을 보이지만 0.76에 그침
    - 20000장의 데이터로 학습시 과적합을 보여 Dropout과 batchnorm을 추가하였지만 규제가 너무 강한지 Train loss도 쉽게 낮아지지 않음.
    
    - GoogLeNet에서 다시 AlexNet의 깊지 않은 모델로 넘어와 학습재개
        1. Alex모델 수정없이 + Test set만 augmetation함. ==> 여전히 과적합으로 validation loss가 올라가지 않음.
        2. Alex모델 수정(batchnorm,dropout) + Test set만 aug

### GoogLeNet:
    - 역시 num class와 loss를 동일하게 설정, aux1,aux2 loss에 0.3의 가중치를 놓고 train loss를 계산
    - 첫번째 시도: 모델을 수정하지 않고 data augmetation 방법 2를 적용해서 진행 ==> Overfitting이 발생했고 validation loss가 떨어지지 않음
    - 두번째 시도: 모델을 수정하지 않고 data augmetation 방법 1을 적용해서 진행 ==> train과 Vadlidation에서 모두 overfitting이 일어나지만 0.83의 정확도를 보임.
    - 두번째 모들의 빠른 overfitting으로 다음 인셉션에 전달되기 전에 모든 값에 dropout 0.3을 부여후 전달. epoch이 많아지니 결국에는 과적합으로 간다. 정확도는 0.7333 -->
    - 세번째 validation data를 image aug에서도 제외하고 train data300장을 가지고 augmetation을 통해 train 1300장 valid 400장 정도를 가지고 실험 ==> dropout과 batchnorm을 모두 적용했지만 overftting이 강하게 나타남 모델자체가 깊어서 그런것 같아 Alexnet으로 실험 재개
 
 ## 그외 파라미터 조정
 
    1.seed값을 조정해봄
 
    2.threshold를 조정해봄 기존 0.5에서 0.4~0.6사이값으로 조정
