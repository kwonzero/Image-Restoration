# NAFNet-Deblurring
- NAFNet을 이용한 Deblurring 작업 파이프라인 구축 및 튜닝
  
* Dataset
    * GoPro Dataset
    * train : 2103 장 / test : 1111 장 (차후 validation : test = 1 : 1 비율로 나누어 테스트 예정)
    * image shape : 1280 x 720

* Default Configs
    * device : Cuda (GPU T4 x 2)
    * image_resize : 256x256
    * num_workers = 4
    * batch_size = 16
 
* Transforms
    * resize : 256x256 (bicubic interpolation)
    * random horizontal flip (0.2)
    * random vertical filp (0.2)

## 진행 사항

- 12/02
  * learning rate : 1e-3
  * epoch = 50
  * param : set 1
  * optimizer : Adam
  * scheduler : None
  * result
      * {Train Loss: 70.1300, Train PSNR: 29.8700, Train SSIM: 0.9269}
      * {Val Loss: 71.0571, Val PSNR: 28.9429, Val SSIM: 0.9219}
 
- 12/04
  * Validation, Test dataset split 작업 진행 -> Train : Valid : Test = 2103 : 555 : 556
  * learning rate : 1e-3
  * epoch = 50
  * param : set 1
  * optimizer : Adam
  * scheduler : None
  * result
      * {Train Loss: 70.0215, Train PSNR: 29.9785, Train SSIM: 0.9274}
      * {Val Loss: 70.9691, Val PSNR: 29.0309, Val SSIM: 0.9228}
      * {Test Loss: 70.9692, Test PSNR: 29.0308, Test SSIM: 0.9212}
