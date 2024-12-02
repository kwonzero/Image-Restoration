﻿# NAFNet을 활용한 Image-Restoration 성능 개선
* 최종결과
  - 진행 중 
* 데이터셋
  - SIDD (Smartphone Image Denoising Dataset)
  - 링크 : [https://dacon.io/competitions/official/236006/overview/description](https://paperswithcode.com/dataset/sidd)
* 훈련 데이터 구조
  - 다운 받은 압축 파일을 풀면 아래와 같은 구조가 나옴
  - 아래 데이터 중 gt_images_256.npy과 noisy_images_256.npy 사용
  ```
  - SIDD
    + gt_images.npy
    + gt_images_128.npy
    + gt_images_256.npy
    - noisy_images.npy
    - noisy_images_128.npy
    - noisy_images_256.npy
  ```
* 진행 사항
  - 2024/11/22(금)
    - BaseLine Model 생성 (NAFNet_v1)
    - Augmentation
      - Horizontal Flip
      - Vertical Flip
  - 2024/11/25(월)
    - NAFNet_v2, v3, v4
    - Architecture 수정 (유의미한 성능 개선 X)
      - enc_blks = [1, 1, 1, 28] -> [1, 1, 1, 1] -> [2, 2, 4, 8] 
      - middle_blk_num = 1       -> 1 x 2        -> 12
      - dec_blks = [1, 1, 1, 1] ->  [1, 1, 1, 1] -> [2, 2, 2, 2]
  - 2024/12/2(월)
    - NAFNet_v5
    - Augemntation
        - Train : 기존 Origial(128), Horizontal Flip(128), Vertical Flip(128)을 2X2 Patch로 나눔. 기존 데이터 수 384장에서 1536장으로 4배 증가
        - Validation, Test : 2X2 Patch 증강
        - 성능개선 : 실험 중
