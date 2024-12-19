# NAFNet을 활용한 Image-Restoration 성능 개선

* 최종결과
  - 진행 중
    
* 데이터셋
  - SIDD (Smartphone Image Denoising Dataset)
  - 링크 : [[https://dacon.io/competitions/official/236006/overview/description](https://paperswithcode.com/dataset/sidd)](https://www.kaggle.com/datasets/rajat95gupta/smartphone-image-denoising-dataset)


* 진행 사항
  - 2024/12/5(목)
    ```
    - sidd challenge dataset으로 변경 후 실험 진행
     
    - Baseline model
         - LR = 1e-4 / 150 Epoch
         - PSNR: 32.0821 / SSIM: 0.8392
    
    - enhanced model 1 (01_data_x12.ipynb)
         - PSNR: 36.531 / SSIM: 0.936
         - PSNR: 13.9%  / SSIM: 11.5% 개선
         - Augemtation : 128장에서 1536장으로 12배 증가시킴
           1. Horizontal Flip(물리적 증강) , Vertical Flip(물리적 증강)
           2. 2x2 patch
         - 한계 : 학습 초반부터 과적합 / 과도한 데이터 수 증가로 인해 과적합이 발생한다고 판단
          
    - enhanced model 2 (02_data_transform.ipynb)
         - PSNR: 32.976 / SSIM: 0.872
         - PSNR: 2.8%   / SSIM: 4.0% 개선
         - Augmentation 수정 (horizontal Flip, Vertical Flip을 통한 물리적인 이미지 증강에서 이미지 변형으로 변경)
         - 한계 : 과적합 문제는 해결 되었지만 적어진 데이터 수로 인해 충분한 훈련이 되지 않아 enhanced model 1에 비해 성능이 매우 떨어짐

    - enhanced model 3 (03_architecture_improvement.ipynb)
         - PSNR: 32.918 / SSIM: 0.871
         - PSNR: 2.6%   / SSIM: 3.8% 개선
         - 모델 구조 개선(복잡도 증가) - enc_blks = [1, 1, 1, 1] -> [2, 2, 4, 8] / middle_blk_num = 1 -> 12 / dec_blks = [1, 1, 1, 28] -> [2, 2, 2, 2]
         - 모델 파라미터 증로 성능 개선 시도
         - 한계 : 모델 파라미터 증가보다 물리적인 데이터 개수가 중요하다 판단 됨

    - enhanced model 4 (04_data_x12_dropout.ipynb)
         - PSNR: 37.552 / SSIM: 0.945
         - PSNR: 17.1%  / SSIM: 12.7% 개선
         - enhanced model 1에 dropout=0.5 을 추가하여 과적합 유의미한 개선
         - 한계 : enhanced model 1에비해 과적합이 감소하였지만 더 개선할 여지가 있음
    ```
      
  - 2024/12/6(금)
    ```
    - enhanced model 5 (05_data-x12_dropout_randomcrop.ipynb)
         - PSNR: 37.740 / SSIM: 0.940
         - PSNR: 17.6%  / SSIM: 12.0% 개선
         - 기존 방법 : Raw Image(3000, 5328) -> Resize(256, 256)     -> 2x2 Patch(128, 128) -> Resize(256, 256)
         - 개선 방법 : Raw Image(3000, 5328) -> RandomCrop(512, 512) -> 2x2 Patch(256, 256)
         - 2번의 Resize로 인한 보간 과정에서 정보 손실을 줄이고 이미지 디테일을 살림
    ```

  - 2024/12/18(수)
    ```
    - enhanced model 6 (06_add_gaussnoise.ipynb)
         - PSNR: 37.775 / SSIM: 0.945
         - PSNR: 17.7%   / SSIM: 12.6% 개선
         - 개선 방법 : GaussNoise 증강을 추가해 노이즈 강한 데이터에 대한 성능 향상
         - 대부분의 데이터의 노이즈가 강하지 않아 노이즈가 강한 데이터를 잘 훈련하지 못하여 Validation에서 성능 향상의 한계에 부딛힌다고 판단하여 적용함
         - 한계 : 시간 관계상 Epoch를 60으로 두고 진행하였는데 PSNR이 작지만 꾸준히 감소하는 모습을 보였음, Epoch를 길게 두고 진행해볼만 함
    ```
    
  - 2024/12/19(목)
    ```
    enhanced model 7
         - PSNR: 37.834 / SSIM: 0.945
         - PSNR: 17.9%  / SSIM: 12.6%개선
         - 개선 방법 : 150 Epoch로 증가, 모델 구조 개선(복잡도 증가), Optimizer 변경(Adam -> AdamW) , LR Scheduler 추가(CosineAnnealingLR) T_max=20으로 두어 Warm Restart
         - 지금까지 LR를 1e-3으로 고정해두고 실험을 진행한 결과 LR가 너무 커 최적점에 도달하지 못하는 것으로 보여 Scheduler를 추가하고 Optimizer를 교체함
         - 한계 : 모델 구조 개선로 모델이 복잡해짐에 따라 과적합 발생 / GPU Memory 한계로 batch_size를 8로 둘 수 밖에 없어 성능 일반화가 쉽지 않음

    enhanced model 8 (08_modify_T_max_weight_decay.ipynb)
         - PSNR: 37.960 / SSIM: 0.946
         - PSNR: 18.3%  / SSIM: 12.8% 개선
         - 개선 방법 : enhanced model 7에 epoch 80, CosineAnnealingLR의 T_max=100, AdamW의 weight_decay=1e-4로 수정
         - 초반 학습 개선을 위해 T_max 값을 100으로 설정, 모델이 깊어짐에 따른 과적합 개선을 위해 weight_decay를 추가
         - 한계 : 50 epoch 이후로 성능 개선이 크게 없음 , weight decay를 더 낮추고 ealry stopping 추가할 필요가 있음
    ```
