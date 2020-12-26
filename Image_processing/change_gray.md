# Change truecolor image to grayscale image (컬러사진 -> 흑백사진) 

## 1. Bitmap 파일 구조 알기
bitmap file은 다음과 같은 구조로 이루어져 있습니다.
![비트맵구조](https://user-images.githubusercontent.com/48755185/103137050-edcede00-4708-11eb-885f-4b267a0bb09f.JPG)
  
실습을 위한 사진 파일은 24비트 트루컬러 이미지이므로, 오른쪽처럼 색상 테이블이 없는 구조를 가집니다.    

>*트루컬러 이미지는 색상표현을 위해 24비트(또는 32비트)를 사용하는 이미지입니다. 각각 8비트를 가지는 R, G, B의 조합으로 표현되며 총 2^24개의 색상을 표현할 수 있습니다.*

### 비트맵 파일 헤더(BITMAPFILEHEADER)  
비트맵 파일 헤더는 다음과 같이 **파일에 대한** 정보를 포함합니다.
|멤버| 바이트| 정보|
|---|:----:|---|
|bfType | 2 | 비트맵 파일이 맞는지 확인('BM'을 표현하는 16진수 0x42 0x4D가 저장)|
|bfSize | 4 | 파일의 크기 정보(Byte)|
|bfReserved1 | 2 | 나중을 위해 사용하지 않는 공간(must be zero)| 
|bfReserved2 | 2 | 나중을 위해 사용하지 않는 공간(must be zero)| 
|bfOffBits | 4 | 픽셀 데이터의 시작 위치|

### 비트맵 정보 헤더(BITMAPINFOHEADER)  
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
|biXPelsPerMeter|4|수평 해상도|
|biYPelsPerMeter|4|수직 해상도|
|biClrUsed|4|색상 테이블의 사이즈|
|biClrImportant|4|색상 인덱스의 수|

### 픽셀 데이터  
픽셀 데이터에는 다음과 같이 R, G, B 값이 저장되어 있습니다.
|멤버| 바이트| 정보|
|---|:----:|---|
|rgbtBlue|1|blue 0~255|
|rgbtGreen|1|green 0~255|
|rgbtRed|1|red 0~255|

## 2.바이너리 편집기로 분석하기
![바이너리](https://user-images.githubusercontent.com/48755185/103153688-ac583480-47d5-11eb-8451-e3036820437a.JPG)  
>*Visual Studio -> 파일 -> 열기 -> 파일 -> 파일 선택 -> 열기 옆 화살표 -> 바이너리 편집기*

실습 이미지를 바이너리 편집기로 열면 위와 같은 화면이 나타납니다.    

>*인텔 계열의 CPU는 little endian 방식으로 데이터를 메모리에 올립니다. 이는 메모리의 낮은 주소에 데이터의 하위 바이트를 저장하는 방식입니다. 따라서 올바르게 정보를 읽기 위해서는 데이터의 바이너리 값이 역순으로 저장되어 있어야, 데이터를 메모리에 올려 정보를 읽을 때 올바른 값이 나오게 됩니다.*

|16진수| 해석|
|---|---|
|42 4D|10진수 변환 시 아스키 코드로 'B'와 'M'을 나타냅니다. 이 정보를 통해 Bitmap file임을 인지할 수 있고, 메모리에 올라갈 때는 little endian 방식에 의해 'MB' 순으로 올라가게 됩니다.|
|36 00 0C 00|파일의 크기가 786,486 byte이고 이를 16진수 변환하면 00 0C 00 36 입니다. 역순으로 저장해야하므로 36 00 0C 00이 저장되어 있습니다.|
|00 00 00 00|Reserve된 공간이고 zero 값으로 세팅되어 있습니다.|
|36 00 00 00|pixel data의 시작 위치가 비트맵 파일 헤더로부터 54 Byte 떨어져 있다는 의미입니다(10진수 변환하면). 실제로 54 Byte 떨어진 39 16 ~ 부터 pixel data가 시작됩니다. |
|28 00 00 00|비트맵 정보 헤더의 크기가 40 Byte 라는 의미입니다. |
|00 02 00 00|이미지의 가로 크기입니다(픽셀 단위). 역순으로 표현하면 00 00 02 00 이고 이를 10진수 변환하면 512이므로 가로 512 pixel의 이미지 임을 알 수 있습니다.|
|00 02 00 00|이미지의 세로 크기입니다(픽셀 단위).|
|01 00|must be set to 1|
|18 00|10진수 변환 시 24를 의미하고 한 픽셀당 24Bits(8bits x 3)가 사용됨을 알 수 있습니다.|
|00 00 00 00|보통 Bmp 파일은 압축을 하지 않기 때문에 0이 저장됩니다.|
|00 00 00 00|이미지의 사이즈를 나타내지만, 압축을 하지 않았을 경우 0이 저장됩니다.|
|12 0B 00 00|수평 해상도를 의미합니다.|
|12 0B 00 00|수직 해상도를 의미합니다.|
|00 00 00 00|색상 테이블이 존재하지 않으므로 0이 저장됩니다.|
|00 00 00 00|마찬가지로 0이 저장됩니다.|

## 3. 문제 해결 순서  
파일을 읽어온다는 것은 해당 파일의 바이너리 코드를 읽어오는 것입니다. 그렇다면 해당 파일의 바이너리 코드를 읽어오고, 비트맵 파일 헤더와 비트맵 정보 헤더의 정보를 통해 어떤 이미지인지 파악한 뒤, 픽셀 데이터의 R, G, B 값을 수정해준다면 문제 해결이 가능할 것 입니다. 이를 다음과 같이 간단하게 정리할 수 있습니다.  

1. 비트맵 파일과 같은 모양의 구조체 작성
2. 작성한 구조체에 비트맵 파일 헤더, 비트맵 정보 헤더, 픽셀 데이터 읽어와서 저장
3. 픽셀 데이터 구조체의 값 수정(Grayscale: R, G, B값을 세 값의 평균으로 바꾸면 흑백 이미지가 됩니다.)
4. 수정된 구조체의 값을 통해 파일 생성

## 3. 문제 해결
앞서 정리한 Bitmap file의 구조체를 다음과 같이 작성할 수 있습니다. 
~~~C
#pragma pack(push, 1)
typedef struct _BITMAPFILEHEADER
{
	unsigned short   bfType;
	unsigned int   bfSize;         
	unsigned short bfReserved1;   
	unsigned short bfReserved2;     
	unsigned int   bfOffBits;
} BITMAPFILEHEADER;

typedef struct _BITMAPINFOHEADER   
{
	unsigned int   biSize;         
	int            biWidth;         
	int            biHeight;      
	unsigned short biPlanes;     
	unsigned short biBitCount;     
	unsigned int   biCompression;   
	unsigned int   biSizeImage;      
	int            biXPelsPerMeter; 
	int            biYPelsPerMeter; 
	unsigned int   biClrUsed;        
	unsigned int   biClrImportant; 
} BITMAPINFOHEADER;

typedef struct _RGBTRIPLE           
{
	unsigned char rgbtBlue;         
	unsigned char rgbtGreen;        
	unsigned char rgbtRed;         
} RGBTRIPLE;
#pragma pack(pop)
~~~
pragma pack이란? 
>*32bits CPU는 구조체 내부의 변수에 메모리를 할당할 때 4Byte 단위로 할당하게 됩니다. 즉, BITMAPFILEHEADER의 경우 short + int + short + short + int = 14 byte가 할당될 것 같지만 실제로는 padding을 포함해 16 Byte가 할당됩니다. 따라서 1 Byte 단위로 할당할 수 있게 처리를 해주어야 하는데, 그 역할을 하는 명령어가 pragma pack 입니다.*

~~~C
int main() 
{
	FILE *fpBmp;
	FILE *fpResult;
	BITMAPFILEHEADER fileHeader;
	BITMAPINFOHEADER infoHeader;

	unsigned char *image;
	int size;
	int width, height;
	int padding;
~~~
* 파일의 헤더 정보를 담을 수 있게 fileHeader, infoHeader를 선언하고 각종 변수를 선언합니다.

~~~C
	fpBmp = fopen("lena512.bmp","rb");

	if (fpBmp == NULL) return 0;

	if (fread(&fileHeader, sizeof(BITMAPFILEHEADER), 1, fpBmp) < 1) 
	{
		fclose(fpBmp);
		return 0;
	}

	if (fileHeader.bfType != 'MB')
	{
		fclose(fpBmp);
		return 0;
	}

	if (fread(&infoHeader, sizeof(BITMAPINFOHEADER), 1, fpBmp) < 1)
	{
		fclose(fpBmp);
		return 0;
	}
~~~
* 파일 포인터는 파일의 시작 위치에 존재합니다. 
* 이 때, fread 함수를 통해 BITMAPFILEHEADER의 사이즈만큼 1번 read하여 fileHeader에 저장합니다. 
* fread의 return 값은 읽기에 성공한 항목의 수 입니다. 1번 read 했으므로 return 값은 1이 되어야 합니다.
* 읽어온 fileHeader의 멤버 bfType의 값은 'MB'이어야 합니다. 파일에 'BM' 순으로 저장되어 있었으므로, 메모리에 올라왔을 때는 little endian 방식에 의해 역순인 'MB'로 올라오기 때문입니다.
* 마찬가지 방식으로 BITMAPINFOHEADER를 읽어옵니다.

~~~C
	size = infoHeader.biSizeImage;
	width = infoHeader.biWidth;
	height = infoHeader.biHeight;
	padding = (PIXEL_ALIGN - ((width * PIXEL_SIZE) % PIXEL_ALIGN)) % PIXEL_ALIGN;
~~~
* 편의를 위해 이미지의 size, width, height를 새로운 변수로 지정하고 padding을 계산합니다.

padding이란?
>*32bits CPU는 데이터를 처리할 때 4byte 단위로 접근합니다. 따라서 비트맵 포맷은 효율적인 데이터 처리를 위해 픽셀 데이터의 가로 크기가 4의 배수가 아니라면 남는 공간에 0을 채워 4의 배수로 만들어 저장합니다. 이 남는 공간을 padding이라고 하고, 픽셀 데이터를 읽기 위해서는 padding이 얼마나 채워졌는지 알아야 합니다.*

예를들어 pixel이 9개 존재한다면, pixel당 3byte(R, G, B)가 사용되므로 27Byte의 공간을 차지하게 되고, 이는 4의 배수가 아닙니다. 따라서 이를 4로 나눈 나머지인 3으로 남는 공간이 4 - 3 = 1byte임을 알 수 있습니다(1Byte를 padding으로 채우면 4의 배수가 된다는 뜻입니다). 
~~~C
padding = PIXEL_ALIGN - (width * PIXEL_SIZE) % PIXEL_ALIGN;
~~~
하지만 pixel이 12개 존재한다면, pixel당 3byte(R, G, B)가 사용되므로 36byte의 공간을 차지하게 되고, 이는 4의 배수입니다. 따라서 이를 4로 나눈 나머지는 0 이므로 위 식에 의하면 padding이 4가되는 오류가 발생합니다. 따라서 위에서 구해진 padding 값을 다시 4로 나누어 나머지가 0일 때의 오류를 수정합니다.
~~~C
padding = (PIXEL_ALIGN - ((width * PIXEL_SIZE) % PIXEL_ALIGN)) % PIXEL_ALIGN;
~~~

~~~C
	if (size == 0)   
	{
		size = (width * PIXEL_SIZE + padding) * height;
	}
~~~
위에서 바이너리 코드를 해석할 때, 압축을 하지 않았을 때는 biSizeImage 값이 0임을 알 수 있었습니다. 따라서 이러한 경우에 대비하여 size를 계산하는 코드를 추가로 작성합니다. 

~~~C
	image = (char*)malloc(size);

	if (fread(image, size, 1, fpBmp) < 1)
	{
		fclose(fpBmp);
		return 0;
	}

	fclose(fpBmp);

	fpResult = fopen("lena512_gray.bmp", "w");
	if (fpResult == NULL)
	{
		free(image);
		return 0;
	}
~~~
픽셀 데이터 저장을 위한 image에 메모리를 픽셀 데이터 크기만큼 동적 할당하고, fread를 통해 바이너리 코드를 읽어줍니다. 또한, grayscale로 변경된 이미지를 저장하기 위한 파일 포인터 또한 열어줍니다.

~~~C
	for (int y = height - 1; y >= 0; y--)
	{
		for (int x = 0; x < width; x++)
		{
			int index = (x * PIXEL_SIZE) + (y * (width * PIXEL_SIZE)) + (padding * y);

			RGBTRIPLE *pixel = (RGBTRIPLE *)&image[index];

			unsigned char blue = pixel->rgbtBlue;
			unsigned char green = pixel->rgbtGreen;
			unsigned char red = pixel->rgbtRed;

			unsigned char gray = (red + green + blue) / PIXEL_SIZE;

			pixel->rgbtBlue = gray;
			pixel->rgbtGreen = gray;
			pixel->rgbtRed = gray;
		}
	}
~~~
* 실제로 픽셀 데이터 값을 변경하는 코드입니다. 중요한 점은 bitmap image는 상하 반전이 된 채로 저장이 되어 있으므로 읽어올 때 마지막 줄부터 읽어와야 한다는 점입니다. 
* 픽셀은 R, G, B 각각 1byte씩, 총 3byte이므로 index는 3byte 단위로 이동해야합니다. 
* image 포인터에 RGBTRIPLE 구조체 포인터로의 형 변환을 수행하여 blue, green, red 전체에 접근할 수 있도록 합니다.
* 해당 픽셀의 blue, green, red 값을 모두 세 값의 평균 값으로 변경합니다.

**왜 구조체 포인터를 사용하지 않고 image 포인터를 사용해서 형 변환을 할까?**
>*픽셀 데이터를 읽어올 때, 구조체 포인터를 사용한다면 픽셀의 갯수만큼 구조체 포인터 배열를 만들어야 합니다. 하지만 C에서 배열을 선언할 때는 상수값이 들어가야하므로 애초에 선언이 불가합니다. 반면에 image 포인터를 동적 할당하여 사용한다면, size에 상관없이 모든 픽셀 데이터를 받을 수 있고 픽셀 데이터 하나하나에 인덱스로 접근이 가능하다는 이점이 있습니다.*

**image 포인터를 RGBTRIPLE 구조체 포인터로 형 변환했을 때 일어나는 일**
>*RGBTRIPLE 구조체 포인터는 3byte의 크기를 가집니다. 따라서 형 변환 하였을 때 image 포인터가 가리키는 지점으로부터 3byte만큼의 정보가 RGBTRIPLE 구조체의 멤버로 각각 들어가게 됩니다.*

~~~C
	fwrite(&fileHeader, sizeof(BITMAPFILEHEADER), 1, fpResult);
	fwrite(&infoHeader, sizeof(BITMAPINFOHEADER), 1, fpResult);
	fwrite(image, size, 1, fpResult);

	fclose(fpResult);   

	free(image);
	return 0;
}
~~~
fileHeader와 infoHeader의 정보는 변함이 없고, image가 가리키는 픽셀 데이터 정보는 앞선 코드에 의해 수정이 되어있습니다. 따라서 fwrite 함수를 통해 출력 파일에 바이너리 코드를 써주면 grayscale로 수정된 형태의 bmp 파일이 생성됩니다.
