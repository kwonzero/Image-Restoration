# NAFNet을 활용한 Image-Restoration 성능 개선

 
* 최종결과
  - 진행 중 
* 데이터셋
  - SIDD (Smartphone Image Denoising Dataset)
    - 링크 : [https://dacon.io/competitions/official/236006/overview/description](https://paperswithcode.com/dataset/sidd)
  - Gopro
    - 링크 : https://www.kaggle.com/datasets/jishnuparayilshibu/a-curated-list-of-image-deblurring-datasets/data
* 데이터 구조
  1. SIDD
     - 아래 데이터 중 gt_images_256.npy과 noisy_images_256.npy 사용
    ```
    - SIDD
      - gt_images.npy
      - gt_images_128.npy
      - gt_images_256.npy
      - noisy_images.npy
      - noisy_images_128.npy
      - noisy_images_256.npy
    ```
  2. Gopro
     - dd
      ```
      -  DBlur
        - CelebA
        - Gopro
          + train
          + test
        - HIDE
        - Helen
        - RealBlur_J
        - RealBlur_R
        - TextOCR
        - Wider-Face
      CelebA

Gopro

HIDE

Helen

RealBlur_J

RealBlur_R

TextOCR

Wider-Face
      ```
* 진행 사항
  - 2024/11/22(금)
    - BaseLine Model 생성 (nafnet_v1)
    - Augmentation
      - Horizontal Flip(물리적 증강)
      - Vertical Flip(물리적 증강)
        
  - 2024/11/25(월)
    - nafnet_v2, v3, v4
    - 기존 Augmentation 유지
    - Architecture 수정 (유의미한 성능 개선 X)
      ```
      - enc_blks       = [1, 1, 1, 28] -> [1, 1, 1, 1] -> [2, 2, 4, 8] 
      - middle_blk_num = 1             ->  1 x 2       ->  12
      - dec_blks       = [1, 1, 1, 1]  -> [1, 1, 1, 1] -> [2, 2, 2, 2]
      ```  
  - 2024/12/2(월)
    - nafnet_v5
    - Augemntation
        - Train : 기존 Origial(128), Horizontal Flip(128), Vertical Flip(128)을 2X2 Patch로 나눔. 기존 데이터 수 384장에서 1536장으로 4배 증가
        - Validation, Test : 2X2 Patch 증강
    - 성능개선 : Loss: 50.690 / PSNR: 49.301 / SSIM: 0.996으로 매우 높은 성능 
    - 한계 : 과적합 (30 epoch 약간 앞에서 Train Loss와 Val Loss 교차)
    - 추후 진행 : Learning Rate Scheduler를 추가하여 Validation Loss가 수렴하기 전에 최적점을 찾아 성능 개선

  - 2024/12/3(화)
    - 
