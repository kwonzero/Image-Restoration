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
    * batch_size : 16 (Block num = 36 -> parameter 증가로 인해 Cuda Out Of Memory error 발생 -> Batch_size = 8로 감소)
    * epoch : 50
  ```
* Transforms
  ```
    * resize : 256x256 (bicubic interpolation) -> center crop
    * random horizontal flip (0.2)
    * random vertical filp (0.2)
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
  - 결과 : v3 모델에 비해 성능이 대폭 하락
    - 30 epoch 에서 조기 종료
    - Val SSIM 값이 0.84를 넘지 못함 -> 주요 요인이 transforms 변경인지, Dropout layer 추가인지 확인 필요

## 진행 예정 사항
```
 1. enc_blks, middle_blks_num, dec_blks 변경 후 결과 확인 (default setting의 경우 50 epoch -> 약 3시간 20분 소요)
 2. Scheduler (stepLR, cosineAnnealingLR 등) 적용 후 성능 비교
 3. dropout (0.5) 적용 -> 과적합 방지
 4. transforms - resize 대신 crop 후 결과 확인 (정보 손실 최소화)
 5. Test-time Local Converter 적용
 6. SimpleGate -> SwiGLU, GeGLU, ReGLU 변경
```
