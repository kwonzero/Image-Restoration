# NAFNet을 활용한 Image-Restoration 성능 개선

* 최종결과
  - 진행 중
  - 
* 데이터셋
  - SIDD (Smartphone Image Denoising Dataset)
    - 링크 : [https://dacon.io/competitions/official/236006/overview/description](https://paperswithcode.com/dataset/sidd)
    - 데이터 구조
        - 아래 데이터 중 gt_images_256.npy과 noisy_images_256.npy 사용
        - 데이터 분할 -> train : 128 장 / validation : 16장 / test : 16장
          ```
          - SIDD
            - gt_images.npy
            - gt_images_128.npy
            - gt_images_256.npy (160장)
            - noisy_images.npy
            - noisy_images_128.npy
            - noisy_images_256.npy (160장)
          ```

        
  - Gopro
    - 링크 : https://www.kaggle.com/datasets/jishnuparayilshibu/a-curated-list-of-image-deblurring-datasets/data
    - 데이터 구조
      ```
      -  DBlur
          ㄴCelebA
          ㄴGopro
            ㄴtrain
              ㄴblur (2103장)
              ㄴsharp (2103장)
            ㄴtest
              ㄴblur (1111장)
              ㄴsharp (1111장)
          ㄴHIDE
          ㄴHelen
          ㄴRealBlur_J
          ㄴRealBlur_R
          ㄴTextOCR
          ㄴWider-Face
      ```
    - 데이터 중 Gopro의 train, test 사용 (test는 validataion : test 절반씩 분할)
      - train      : blur:2103 / sharp:2103
      - validation : blur:556 / sharp:556
      - test       : blur:555 / sharp:555

     
* 진행 사항
  - 2024/11/22(금)
    ```
    - BaseLine Model 생성 (nafnet_v1)
    - Augmentation
      - Horizontal Flip(물리적 증강)
      - Vertical Flip(물리적 증강)
    ```
        
  - 2024/11/25(월)
    ```
    - nafnet_v2, v3, v4
    - 기존 Augmentation 유지
    - Architecture 수정 (유의미한 성능 개선 X)
      - enc_blks       = [1, 1, 1, 28] -> [1, 1, 1, 1] -> [2, 2, 4, 8] 
      - middle_blk_num = 1             ->  1 x 2       ->  12
      - dec_blks       = [1, 1, 1, 1]  -> [1, 1, 1, 1] -> [2, 2, 2, 2]
  
    ```
    
  - 2024/12/2(월)
    ```
    - nafnet_v5
    - Augemntation
        - Train : 기존 Origial(128), Horizontal Flip(128), Vertical Flip(128)을 2X2 Patch로 나눔. 기존 데이터 수 384장에서 1536장으로 4배 증가
        - Validation, Test : 2X2 Patch 증강
    - 성능개선 : Loss: 50.690 / PSNR: 49.301 / SSIM: 0.996으로 매우 높은 성능 
    - 한계 : 과적합 (30 epoch 약간 앞에서 Train Loss와 Val Loss 교차)
    ```

  - 2024/12/4(수)
    ```
      1. sidd challenge dataset으로 바꾸어 실험 진행 및 결과 비교 (*참고: https://abdokamel.github.io/sidd/ , https://competitions.codalab.org/competitions/22230)
      2. PSNR: 36.531071186065674 / SSIM: 0.9363919422030449
     ```
    
  - 2024/12/5(목)
    ```
    - sidd challenge dataset으로 변경 후 실험 진행
     
      1. Baseline model
         - LR = 1e-4 / 150 Epoch
         - PSNR: 32.0821 / SSIM: 0.8392
    
      2. enhanced model 1 (01_data_x12.ipynb)
         - LR = 1e-4 / 60 Epoch / horizontal Flip / Vertical Flip / 2x2 Patch 증강 (완료)
         - PSNR: 36.531 / SSIM: 0.936
         - train 이미지를 horizontal Flip, Vertical Flip와 2x2 patch를 통해 128장에서 1536장으로 12배 증가시킴
         - 한계 : 학습 초반부터 과적합 / 과도한 데이터 수 증가로 인해 과적합이 발생한다고 판단
          
      3. enhanced model 2 (02_data_transform.ipynb)
         - LR = 1e-4 / 300 Epoch / horizontal Flip / Vertical Flip / 2x2 Patch 증강
         - PSNR: 32.9756 / SSIM: 0.8724
         - horizontal Flip, Vertical Flip을 통한 물리적인 데이터 수를 증강 대신 이미지 변형으로 변경
         - 한계 : 과적합 문제는 해결 되었지만 enhanced model 1에 비해 성능이 매우 떨어짐

      4. enhanced model 3 (03_architecture_improvement.ipynb)
         - LR = 1e-4 / 300 Epoch / horizontal Flip / Vertical Flip / 2x2 Patch 증강
         - PSNR: 32.9179 / SSIM: 0.8709
         - 모델 구조 개선(복잡도 증가) - enc_blks = [1, 1, 1, 1] -> [2, 2, 4, 8] / middle_blk_num = 1 -> 12 / dec_blks = [1, 1, 1, 28] -> [2, 2, 2, 2]
         - 모델 파라미터 증로 성능 개선 시도
         - 한계 : PSNR은 약간 증가 SSIM은 하락 / 물리적인 데이터 개수가 중요하다 판단 됨
        
      6. enhanced model 4 (04_data_x12_dropout.ipynb)
         - LR = 1e-4 / 60 Epoch / horizontal Flip / Vertical Flip / 2x2 Patch 증강 / dropout=0.5
         - PSNR: 37.5522 / SSIM: 0.9454
         - enhanced model 2에 dropout=0.5 을 추가하여 과적합 유의미한 개선
         - 한계 : enhanced model 2에비해 과적합이 감소하였지만 더 개선할 여지가 있음
    ```
      
  - 2024/12/6(금)
    ```
    - enhanced model 4 개선: 성능 향상 및 과적합 개선 방안 모색

        1. 성능 개선
           - 기존 방법 : Raw Image(3000, 5328) -> Resize(256, 256)     -> 2x2 Patch(128, 128) -> Resize(256, 256)
           - 개선 방법 1 : Raw Image(3000, 5328) -> RandomCrop(512, 512) -> 2x2 Patch(256, 256)
               - 2번의 Resize로 인한 보간 과정에서 정보 손실을 줄이고 이미지 디테일을 살림
               - PSNR: 37.7398  / SSIM: 0.9396
               - 기대보다 성능 개선이 이루어지지 않음
           
           - 개선 방법 2 : Raw Image(3000, 5328) -> sliding window(256, 256)  


        3. 과적합 개선
           - LR_Scheduler
           - RL 감소
           - L2 규제 등
    ```
 
# 진행 예정
- 성능 개선-개선 방법 2

