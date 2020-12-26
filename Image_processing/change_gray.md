# Change truecolor image to grayscale image (컬러사진 -> 흑백사진) 

### 1. Bitmap 파일 구조 알기
bitmap file은 다음과 같은 구조로 이루어져 있습니다.
![비트맵구조](https://user-images.githubusercontent.com/48755185/103137050-edcede00-4708-11eb-885f-4b267a0bb09f.JPG)
  
실습을 위한 사진 파일은 24비트 트루컬러 이미지이므로, 오른쪽처럼 색상 테이블이 없는 구조를 가집니다.    

> 트루컬러 이미지는 색상표현을 위해 24비트(또는 32비트)를 사용하는 이미지입니다. 각각 8비트를 가지는 R, G, B의 조합으로 표현되며 총 2^24개의 색상을 표현할 수 있습니다.

#### 비트맵 파일 헤더(BITMAPFILEHEADER)  
비트맵 파일 헤더는 다음과 같이 **파일에 대한** 정보를 포함합니다.
|멤버| 바이트| 정보|
|---|:----:|---|
|bfType | 2 | 비트맵 파일이 맞는지 확인('BM'을 표현하는 16진수 0x42 0x4D가 저장)|
|bfSize | 4 | 파일의 크기 정보(Byte)|
|bfReserved1 | 2 | 나중을 위해 사용하지 않는 공간(must be zero)| 
|bfReserved2 | 2 | 나중을 위해 사용하지 않는 공간(must be zero)| 
|bfOffBits | 4 | 픽셀 데이터의 시작 위치|

#### 비트맵 정보 헤더(BITMAPINFOHEADER)  
비트맵 정보 헤더는 다음과 같이 **이미지에 대한** 정보를 포함합니다.  
|멤버| 바이트| 정보|
|---|:----:|---|
|biSize|4|비트맵 정보 헤더의 크기|
|biWidth|4|비트맵 이미지의 가로 크기|
|biHeight|4|비트맵 이미지의 세로 크기|
|biPlanes|2|사용하는 색상판의 수(must be set to 1)|
|biBitCount|2|픽셀 당 비트 수(8비트 R, G, B로 이루어져 있으므로 24비트)|
|biCompression|4|압축 방식|
|biSizeImage|4|픽셀 데이터의 크기|
|biXPelsPerMeter|4|가로 해상도|
|biYPelsPerMeter|4|세로 해상도|
|biClrUsed|4|색상 테이블의 사이즈|
|biClrImportant|4|색상 인덱스의 수|

 #### 픽셀 데이터
|멤버| 바이트| 정보|
|---|:----:|---|
|rgbtBlue|1|blue 0~255|
|rgbtGreen|1|green 0~255|
|rgbtRed|1|red 0~255|
