 # NAFNet-Deblurring
- NAFNet을 이용한 Deblurring 작업 파이프라인 구축 및 튜닝
* Dataset
  ```
    * GoPro Dataset
    * Train Dataset : Blur, Sharp 각각 2103 장 / Test Dataset : Blur, Sharp 각각 1111 장
    * image shape : 1280 x 720
  ```
* Default Configs
  ```
    * device : Cuda (GPU T4 x 2)
    * image_resize : 256x256
    * num_workers : 4
    * batch_size : 8 (Block num = 36 -> parameter 증가로 인해 Cuda Out Of Memory error 발생 -> Batch_size = 8로 감소)
    * epoch : 50 -> 100 -> 200(예정)
  ```
* Transforms for train, valid dataset
  ```
    * resize : 256x256 (bicubic interpolation) -> Center Crop -> Random Crop 
    * random horizontal flip (0.2) -> (0.5)
    * random vertical filp (0.2) -> (0.5)
  ```

## 진행 사항
- 12/02
  [NAFNet-gopro-dataset_v1]
  ```
  * blocks_parameter
      * enc_blks = [1, 1, 1, 1]
      * middle_blks_num = 1
      * dec_blks = [1, 1, 1, 1]
      * total parameter: 4,494,563
  * result
      * {Train Loss: 70.1300, Train PSNR: 29.8700, Train SSIM: 0.9269}
      * {Val Loss: 71.0571, Val PSNR: 28.9429, Val SSIM: 0.9219}
  ```
- 12/04
  [NAFNet-gopro-dataset_v1_2]
  - Validation, Test dataset split 작업 진행 -> Train : Valid : Test = 2103 : 555 : 556
  ```
  * blocks_parameter
      * enc_blks = [1, 1, 1, 1]
      * middle_blks_num = 1
      * dec_blks = [1, 1, 1, 1]
      * total parameter: 4,494,563
  * result
      * {Train Loss: 70.0215, Train PSNR: 29.9785, Train SSIM: 0.9274}
      * {Val Loss: 70.9691, Val PSNR: 29.0309, Val SSIM: 0.9228}
      * {Test Loss: 70.9692, Test PSNR: 29.0308, Test SSIM: 0.9212}
  ```
- 12/06
  [NAFNet-gopro-dataset_v2]
  ```
  * blocks parameter -> NAFNet Default 설정으로 세팅
  * blocks_parameter
      * enc_blks = [1, 1, 1, 28]
      * middle_blks_num = 1
      * dec_blks = [1, 1, 1, 1]
      * total parameter: 17,111,907
  * result (best model's weights load)
      * {Train Loss: 69.5422, Train PSNR: 30.4578, Train SSIM: 0.9311}
      * {Val Loss: 70.7287, Val PSNR: 29.2713, Val SSIM: 0.9251}
      * {Test Loss: 70.8195, Test PSNR: 29.1805, Test SSIM: 0.9232}
  ```
  - Epoch 6에서 Training, Validation Loss값이 대폭 증가 (L2 규제를 추가하여 학습 안정화 필요)
- 12/11
  [NAFNet-gopro-dataset_v3]
  ```
  * blocks_parameter
      * enc_blks = [2, 2, 4, 8]
      * middle_blks_num = 12
      * dec_blks = [2, 2, 2, 2]
      * total parameter: 29,159,715
  * result (best model's weights load)
      * {Train Loss: 68.9812, Train PSNR: 31.0188, Train SSIM: 0.9389}
      * {Val Loss: 69.9356, Val PSNR: 30.0644, Val SSIM: 0.9350}
      * {Test Loss: 69.9329, Test PSNR: 30.0671, Test SSIM: 0.9369}
  ```
  - blocks_parameter 변경 후 가장 좋은 결과 / 마지막 epoch에서도 Validation PSNR, SSIM값 증가 -> 최대 epoch를 늘려야 할것으로 판단
- 12/12
  [NAFNet-gopro-dataset_v4]
  - v3 모델에 Dropout(0.5) layer 추가 / Train, Valid Dataset Transforms : resize (256x256) -> center crop (256x256) 
  - 결과
    ```
    - v3 모델에 비해 성능이 대폭 하락
    - 30 epoch 에서 조기 종료
    - Val SSIM 값이 0.84를 넘지 못함 -> 주요 요인이 transforms 변경인지, Dropout layer 추가인지 확인 필요
    ```
## Dataset 구축 작업 변경 이후 학습 진행 사항
- 12/13
**Train, Validation, Test Dataset 구축 작업 변경점**
    ```
    기존 : 학습, 검증, 테스트 데이터셋에 모두 Transforms (Resize or Center Crop, Horizontal & Vertical Filp)
    변경 : 학습, 검증 데이터셋에만 Transforms 적용 / 테스트 데이터셋은 Raw Image (1280 x 720)
    ```
  ```
  (12/16 ~)
  
  * blocks_parameter = 'default'
  
  # Test 1
  * 변경점
    - data transforms : resize(256x256) -> center crop(256x256) 
    - epoch 100
    - Dropout(0.5) 추가
  * 실험 결과
      * {Test Loss: 73.0543, Test PSNR: 26.9457, Test SSIM: 0.8771}
    - 학습 초기부터 과적합 발생 (augmentation 확률 증가 필요)
    - 학습의 속도가 느림(논문에서는 1000~3000 epoch로 학습 진행)
  * 추가 예정 사항
    - TLC (Test-time Local Converter) : Full size(1280 x 720) Test 진행 시 성능 하락 방지

  # Test 2
  * 변경점
    - Test 1에서 저장된 모델에 TLC 적용 후 추론 진행
  * 실험 결과
      * {Test Loss: 72.5421, Test PSNR: 27.4579, Test SSIM: 0.8865}
    - Test_1 SSIM : 0.8771 에서 Test_2 SSIM : 0.8865로 유의미한 상승값 확인
  * 추가 예정 사항
    - Transforms : Center Crop -> Random Crop (256 x 256) & Increase Flip Probablity (0.5)
    - Optimizer 변경 : Adam -> AdamW (lr = 1e-3, betas = (0.9, 0.9), weight decay = 1e-3)
    - L2 규제 추가

  # Test 3
  * 변경점
     - Data Transforms
         - Resize(256x256) -> Center Crop(256x256) -> Random Crop(256x256)
         - Horizontal, Vertical Flip 확률 0.5로 증가
     - Overfitting 방지
         - Optimizer 변경 : Adam -> AdamW (lr = 1e-3, betas = (0.9, 0.9), weight decay = 1e-3) -> Weight decay (L2 Regularization) 적용
         - Learning Rate Scheduler(CosineAnnealingLR) 적용 -> 초기에는 높은 lr값으로 빠른 학습 후 점차적으로 학습률을 줄여가며 안정적인 학습 진행 -> 과적합 방지
  * 실험 결과
       * Test Loss: 71.7600, Test PSNR: 28.2400, Test SSIM: 0.8987
     - Test 2와 비교했을 때 학습 초반 과적합이 해결되었으며, 안정적인 학습 양상을 보임 / PSNR, SSIM 모두 상승
     - SSIM, PSNR Graph : 100 epoch에서도 점진적으로 증가 -> 추가 학습을 진행해 볼 필요가 있음
  * 추가 예정 사항
     - 100 epoch 추가 학습 진행
  ```
**# Test 3 Results**

<img src = "https://github.com/user-attachments/assets/25ff6c1f-fcaf-416a-93ce-1b8d10d53998" width="250" height="250">
<img src = "https://github.com/user-attachments/assets/53ef68ab-47f4-4b83-9151-e1f65bec5667" width="250" height="250">
<img src = "https://github.com/user-attachments/assets/42825487-1c74-454b-a942-1ebcadaebe06" width="250" height="250">

  ```
  # Test 4
  * 변경점
     - Test 3의 모델을 200 epoch 까지 학습 진행 -> early stopping in 180 epoch
  * 실험 결과
       * Test Loss: 70.8535, Test PSNR: 29.1465, Test SSIM: 0.9113
     - Training, Validation 모두 학습 진행이 느려짐 (CosineAnnealingLR 추가로 인해 lr값이 1 epoch당 6e-5씩 감소)
     - Local Minimum에 갇힌것으로 판단됨
     - CosineAnnealingLR scheduler의 T_max값 설정에 문제가 있는것을 파악
  * 추가 예정 사항
     - T_max값 변경 후 다시 500 epoch 학습 진행

  # Test 5
  * 변경점
     - T_max : Max epoch (500) 설정 후 학습 진행
  * 실험 결과
       * 
     - 
  * 추가 예정 사항
     - 
  ```

## 진행 예정 사항
```
 1. 약 500 epoch까지 학습 진행
 2. SimpleGate 구조 변경 -> SwiGLU, GeGLU, ReGLU
 3. Input Image Concatenate 작업 진행 (Fast-Fourier Transform)
 4. NAF Block, NAFNet 구조 변경 (AdaRevD, CGNet..)
```
