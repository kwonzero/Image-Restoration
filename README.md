# NAFNet을 활용한 Image-Restoration 성능 개선

* 최종결과
  - 진행 중
    
* 데이터셋
  - SIDD (Smartphone Image Denoising Dataset)
    - 링크 : [[https://dacon.io/competitions/official/236006/overview/description](https://paperswithcode.com/dataset/sidd)](https://www.kaggle.com/datasets/rajat95gupta/smartphone-image-denoising-dataset)


* 진행 사항
  - 2024/11/22(금)
    ```
    - BaseLine Model 생성 (nafnet_v1)
    - Augmentation
      - Horizontal Flip(물리적 증강) , Vertical Flip(물리적 증강)
   
  - 2024/12/5(목)
    ```
    - sidd challenge dataset으로 변경 후 실험 진행
     
      1. Baseline model
         - LR = 1e-4 / 150 Epoch
         - PSNR: 32.0821 / SSIM: 0.8392
    
      2. enhanced model 1 (01_data_x12.ipynb)
         - PSNR: 36.531 / SSIM: 0.936
         - PSNR 13.9%  / SSIM 11.5 % 개선
         - Augemtation : 128장에서 1536장으로 12배 증가시킴
           1. Horizontal Flip(물리적 증강) , Vertical Flip(물리적 증강)
           2. 2x2 patch
         - 한계 : 학습 초반부터 과적합 / 과도한 데이터 수 증가로 인해 과적합이 발생한다고 판단
          
      3. enhanced model 2 (02_data_transform.ipynb)
         - PSNR: 32.9756 / SSIM: 0.8724
         - PSNR 2.8%  / SSIM 4.0 % 개선
         - Augmentation 수정 (horizontal Flip, Vertical Flip을 통한 물리적인 이미지 증강에서 이미지 변형으로 변경)
         - 한계 : 과적합 문제는 해결 되었지만 적어진 데이터 수로 인해 충분한 훈련이 되지 않아 enhanced model 1에 비해 성능이 매우 떨어짐

      4. enhanced model 3 (03_architecture_improvement.ipynb)
         - PSNR: 32.9179 / SSIM: 0.8709
         - PSNR 2.6%  / SSIM 3.8 % 개선
         - 모델 구조 개선(복잡도 증가) - enc_blks = [1, 1, 1, 1] -> [2, 2, 4, 8] / middle_blk_num = 1 -> 12 / dec_blks = [1, 1, 1, 28] -> [2, 2, 2, 2]
         - 모델 파라미터 증로 성능 개선 시도
         - 한계 : 모델 파라미터 증가보다 물리적인 데이터 개수가 중요하다 판단 됨
        
      6. enhanced model 4 (04_data_x12_dropout.ipynb)
         - PSNR: 37.5522 / SSIM: 0.9454
         - PSNR 17.1%  / SSIM 12.7% 개선
         - enhanced model 1에 dropout=0.5 을 추가하여 과적합 유의미한 개선
         - 한계 : enhanced model 1에비해 과적합이 감소하였지만 더 개선할 여지가 있음
    ```
      
  - 2024/12/6(금)
    ```
    - enhanced model 5
         - PSNR: 37.7398  / SSIM: 0.9396
         - PSNR 17.6%  / SSIM 12.0 % 개선
         - 기존 방법 : Raw Image(3000, 5328) -> Resize(256, 256)     -> 2x2 Patch(128, 128) -> Resize(256, 256)
         - 개선 방법 : Raw Image(3000, 5328) -> RandomCrop(512, 512) -> 2x2 Patch(256, 256)
         - 2번의 Resize로 인한 보간 과정에서 정보 손실을 줄이고 이미지 디테일을 살림
    ```

  - 2024/12/18(수)
    ```
    - enhanced model 6
         - PSNR: 37.7751 / SSIM: 0.9451
         - PSNR 17.7%  / SSIM 12.6 % 개선
         - 개선 방법 : GaussNoise 증강을 추가해 노이즈 강한 데이터에 대한 성능 향상
         - 대부분의 데이터의 노이즈가 강하지 않아 노이즈가 강한 데이터를 잘 훈련하지 못하여 Validation에서 성능 향상의 한계에 부딛힌다고 판단하여 적용함
         - 한계 : Validation
    ```
    
# 진행 예정
- 성능 개선-개선 방법 2

