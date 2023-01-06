# adversarial-filter
초해상화 모델에 적대적 학습을 수행하여 AI 모델에 대한 적대적 공격을 상쇄하는 필터를 개발한 프로젝트입니다. 


## 목차
* [프로젝트 설명](#프로젝트-설명)
* [모델 구조](#모델-구조)
* [실험환경](#실험환경)
* [학습방법](#학습방법)
* [결과](#결과)
* [개발환경](#개발환경)
* [코드 및 사용법](#코드-및-사용법)
* [관련연구](#관련연구)

## 프로젝트 설명
<p align="center">
<img src="https://user-images.githubusercontent.com/71579787/210082116-a4736c6b-18d4-44c7-b132-8de5d2dc716a.png" width="650">
</p>
본 프로젝트는 적대적 공격이 적용된 이미지에서 적대적 공격을 상쇄하는 일종의 '필터'를 개발하는 프로젝트입니다.  <br>
<b>적대적 공격</b>이란, 의도적으로 이미지를 정교하게 조작하여 AI 분류 모델의 오동작을 유발하는 공격입니다. (ex)판다를 책상으로 분류) <br>
위의 이미지(Attacked)와 같이, 공격을 가해도 사람의 눈에는 큰 차이가 없지만 AI 모델은 오분류하게 됩니다. <br>
적대적 학습이나, 전처리를 통한 방어와 같이 적대적 공격에 대한 방어 방법들이 제시되었지만, <b>기존 방법들은 다음과 같은 문제</b>가 있습니다.  
 <br><br>

 * 적대적 학습<br>
 AI 모델에 적대적 공격을 가한 이미지를 추가로 학습시키는 방법입니다. <br>
 그러나 적대적 학습은 각 <u>모델마다 별도로 추가 학습</u>이 필요하다는 단점이 있습니다.. <br>
 * 전처리를 통한 방어<br>
 Randomization이나 초해상화(SR)를 통해 이미지를 전처리함으로써, 공격 상쇄 효과가 있다는 것이 확인되었습니다. <br>
 그러나 고도화된 적대적 공격(DI-<sup>2</sup>-FGSM, M-DI<sup>2</sup>-FGSM)에 대해서는 낮은 방어율을 보입니다.  
 또한 공격받지 않은 이미지에 대해서는 오히려 정확도를 저하시킵니다. 

본 프로젝트에서는 두 방식의 장점을 결합하고 단점을 상쇄하도록 초해상화 모델에 적대적 학습을 수행하여, <b>적대적 공격을 상쇄해주는 필터</b>를 개발하였습니다.  
이러한 방식을 통해 다음과 같은 <b>장점</b>을 얻을 수 있습니다.

 * 사용 중이던 AI 모델을 별도로 학습할 필요가 없습니다.
 * 기존의 전처리를 적용하는 방식보다 고도화된 적대적 공격에 대해서도 향상된 방어율을 보입니다.
 * 기존의 전처리를 적용하는 방식보다 공격받지 않은 이미지에 대한 정확도 저하가 적습니다. 
 

 
## 모델 구조
본 프로젝트에서 개발한 적대적 필터는 다음과 같은 구조를 갖습니다.  

①입력 이미지에 대한 다운샘플링을 수행하고,  
②다운샘플된 이미지를 적대적 학습된 초해상화 모델로 다시 복원합니다. 
<p align = "center">
<img src = "https://user-images.githubusercontent.com/71579787/209918632-f8729954-6549-4dc6-9a29-dc5f005c8398.png" width = "750" ></p>

각 단계는 다음과 같은 역할을 수행합니다.
 * 다운샘플링: 다운샘플링을 통한 적대적 공격 상쇄 효과가 있습니다. 다만 이미지 자체에도 손상이 발생합니다.
 * 초해상화: 다운샘플링 과정에서 손실된 정보를 복원하고, 적대적 공격을 추가로 상쇄합니다. 

## 실험환경
### 학습 및 검증에 사용한 데이터셋: <b>NIPS-DEV</b> dataset
* Train: 0~799
* Validation: 800~899
* Test: 900~999
 
### 사용한 공격 종류
* 학습: DI<sup>2</sup>-FGSM
* 학습 및 검증 : DI<sup>2</sup>-FGSM, M-DI<sup>2</sup>-FGSM

## 학습방법
적대적 공격을 수행한 이미지를 다운 샘플링한 이미지(LR)를 SR 모델로 복원한 결과와 원본 이미지(HR)를 AI 분류 모델이 같게 인식하도록 SR 모델을 학습하였습니다.  
이때의 <b>loss function</b>은 다음과 같습니다. 여러 분류 모델에 일반화가 잘되도록 <b>앙상블 기법을 적용</b>하였습니다. <br><br>

$$ L = \Sigma_{k=1}^{N}{MSE(C_K(HR), C_K(F(LR)))} $$  

<p align="center">(※ C<sub>k</sub>: K번째 분류 모델, F: SR 모델)</p>

## 결과
### 결과이미지
<p align="center">
<img width="800" alt="image" src="https://user-images.githubusercontent.com/71579787/210080637-0b7b5b76-cbf4-41e0-b7d7-9fa96a798032.png">
</p>
시각적으로도 적대적 공격에 의해 추가된 일종의 노이즈들이 제거되는 것을 확인할 수 있습니다.  


### 방어율
원본 이미지를 포함해 3가지 공격 기법에 대해 3가지 방어 기법을 적용했을 때의 방어율은 다음과 같습니다.  
이때 <b>방어율</b>은 각 공격 기법에 방어 기법을 적용한 후, 분류 모델에게 입력했을 때 올바르게 분류한 비율을 의미하며, 학습에 사용되지 않은 3가지 분류 모델을 통해 검증한 결과입니다. 
  
<p align = "center">
<img src = "https://user-images.githubusercontent.com/71579787/210076973-d6be5029-0a1c-4ddc-b4d6-9576ac333b48.png" width="600">
</p>
<p align = "right">(※행: 사용한 공격 기법, 열: 사용한 방어 기법)</p>



### 고찰
#### 장점
* 고도화된 적대적 공격에 대해 기존 방법인 SR보다 30~40% 이상 높은 방어율을 보입니다. 
* 공격받지 않은 이미지에 대해서도 기존 방법인 SR보다 더 높은 방어율을 보입니다.
* 본 모델을 통해 이미지를 전처리하면 별도의 학습없이 방어 효과를 얻을 수 있습니다.
#### 개선점
* 기존 연구보다는 좋은 결과를 보였지만, 여전히 원본 이미지에 대한 방어율 저하 문제를 개선해야 합니다. 
* 마찬가지로 더욱 높은 방어율이 필요합니다. 

## 개발환경
* 언어 - python
* 프레임워크 - pytorch
* Google Colab Pro

## 코드 및 사용법
* train.ipynb: 학습 코드
* test.ipynb : 검증 코드([결과](#결과))
* model_implementation.py : 모델 사용법
* models, model_zoo/IMDN/IMDN_x2.pth : [IDMN](https://github.com/Zheng222/IMDN)

model_zoo/ours/adv_ens_imdn_v1_best.pt 파일을 통해 사전학습된 모델을 사용할 수 있습니다. (model_implementation.py 참고)

## 관련연구
* Mustafa, Aamir, et al. "Image super-resolution as a defense against adversarial attacks." IEEE Transactions on Image Processing 29 (2019): 1711-1724.
* C. Xie, J. Wang, Z. Zhang, Z. Ren, and A. Yuille, “Mitigating adversarial effects through randomization,” in International Conference on Learning Representations, 2018.
* N. Das, M. Shanbhogue, S.-T. Chen, F. Hohman, S. Li, L. Chen, M. E. Kounavis, and D. H. Chau, “Shield: Fast, practical defense and vaccination for deep learning using jpeg compression,” arXiv preprint arXiv:1802.06816, 2018.
* A. Prakash, N. Moran, S. Garber, A. DiLillo, and J. Storer, “Deflecting adversarial attacks with pixel deflection,” in Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, 2018, pp. 8571–8580
* C. Guo, M. Rana, M. Cisse, and L. van der Maaten, “Countering adversarial images using input transformations,” arXiv preprint arXiv:1711.00117, 2017



 
