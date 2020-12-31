# Change truecolor image to grayscale image 2 (24bits -> 8bits)

[Change truecolor image to grayscasle image 1](https://github.com/skadud8951/TIL/blob/main/Image_processing/change_gray.md)에서는 24bits 비트맵 트루컬러 이미지를 24bits 비트맵 그레이스케일 이미지로 변환하였습니다(그레이스케일은 8bits로도 표현이 가능함에도 불구하고). 따라서 용량의 낭비를 줄이기 위해 24bits 트루컬러 이미지를 8bits 그레이스케일로 바꾸어 보도록 하겠습니다. 



## 1. 8bits bitmap file의 구조 

![비트맵구조2](https://user-images.githubusercontent.com/48755185/103414183-38949e00-4bc0-11eb-91f3-12f649b08f07.jpg)  
출처: C언어 코딩 도장

8비트 비트맵 파일의 경우 24비트일 때와 다르게 **색상테이블**이 추가되어 있습니다. 24비트 비트맵 파일은 픽셀 하나당 24비트를 사용하여 원하는 색상을 표현하고, 8비트 비트맵 파일은 픽셀 하나당 8비트를 사용하여 원하는 색상을 표현합니다. 즉, 2<sup>8</sup> = 256가지의 색상만을 표현할 수 있습니다. 따라서 표현하고자 하는 256가지의 색상을 색상 테이블에 저장해두고, 픽셀 데이터에서는 사용하고자 하는 색상의 인덱스만을 적어 색상을 표현합니다. 



## 2. 색상 테이블 표현을 위한 구조체 RGBQUAD

~~~c
typedef struct RGBQUAD
{
	unsigned char rgbBlue;
	unsigned char rgbGreen;
	unsigned char rgbRed;
	unsigned char rgbReserved;
} RGBQUAD;
~~~

색상 테이블의 표현을 위한 구조체는 위와 같습니다. B, G, R 값이 순서대로 들어있고 rgbReserved는 항상 0의 값이 저장되어 있습니다. 앞선 예제에서 RGB값을 모두 같게 만들어 그레이스케일을 구현할 수 있었습니다. 따라서 이번 예제에서는 RGBQUAD 구조체를 256개 생성하고, 인덱스와 동일하게 RGB 값을 채워주어 그레이스케일을 위한 색상 테이블을 만들어 보겠습니다. 



## 3. 문제 해결 순서

RGB값을 채우는 것도 중요하지만, 24비트 이미지가 8비트로 바뀌면서 파일헤더와 정보헤더의 각종 값들이 바뀌어야만 한다는 것을 인지해야 합니다. 그러므로 문제 해결 순서를 다음과 같이 정리할 수 있습니다.

1. 그레이 스케일을 위한 색상 테이블 RGBQUAD 만들기
2. Output image를 위한 포인터 선언(픽셀 데이터 값을 저장합니다.)
3. 24비트 이미지의 R, G, B의 평균 값을 픽셀 데이터 값으로 넣기
4. 파일헤더, 정보헤더에서 바뀌어야 할 값 계산하여 저장



## 4. 문제 해결

~~~c
int main()
{
	FILE *fp;
	FILEHEADER fileHeader;
	INFOHEADER infoHeader;
	RGBQUAD rgbQuad[256];

	unsigned char *img, *img_output;
	int padding, size, size_output;

	for (int i = 0; i < 256; i++)
	{
		rgbQuad[i].rgbBlue = i;
		rgbQuad[i].rgbGreen = i;
		rgbQuad[i].rgbRed = i;
		rgbQuad[i].rgbReserved = 0;
	}
~~~

* RGBQUAD 구조체를 main 함수 위에 선언한 뒤, main 함수에서 RGBQUAD의 값을 채워줍니다. 그레이 스케일 이므로 RGB값을 모두 동일하게 설정해주면 됩니다(rgbQuad[0]에는 RGB값이 모두 0, rgbQuad[1]에는 RGB값이 모두 1). 

-------------



~~~C
fp = fopen("lena512.bmp", "rb");
	if (fp == NULL) return 0;

	if (fread(&fileHeader, sizeof(FILEHEADER), 1, fp) < 1)
	{
		fclose(fp);
		return 0;
	}
	if (fileHeader.bfType != 'MB')
	{
		fclose(fp);
		return 0;
	}
	
	if (fread(&infoHeader, sizeof(INFOHEADER), 1, fp) < 1)
	{
		fclose(fp);
		return 0;
	}
~~~

* Input image에서 파일헤더, 정보헤더를 읽어오기 위한 작업을 해줍니다. 

---------

~~~c
	padding = (PIXEL_ALIGN - ((infoHeader.biWidth * PIXEL_SIZE) % PIXEL_ALIGN)) % PIXEL_ALIGN;
	size = (infoHeader.biWidth * PIXEL_SIZE + padding) * infoHeader.biHeight;
	size_output = (infoHeader.biWidth * 1 + padding) * infoHeader.biHeight;

	img = (char*)malloc(size);
	img_output = (char*)malloc(size_output);

	if (fread(img, size, 1, fp) < 1)
	{
		fclose(fp);
		free(img);
		return 0;
	}
	fclose(fp);

	fp = fopen("gray.bmp", "wb");
	if (fp == NULL) return 0;
~~~

* padding과 size를 계산해줍니다. size만큼의 동적할당을 하기 위해 필요합니다. padding에 대한 개념은 [Change truecolor image to grayscale image 1](https://github.com/skadud8951/TIL/blob/main/Image_processing/change_gray.md) 에 설명되어 있습니다.
* 24비트 이미지를 8비트로 만들어 주는 것이므로 input과 output의 픽셀데이터 사이즈가 다릅니다. input(24비트) 이미지는 픽셀 하나당 24비트를 사용하고 output(8비트) 이미지는 픽셀 하나당 8비트를 사용하기 때문입니다. 따라서 output의 size를 따로 계산하고, output 픽셀 데이터의 저장을 위한 포인터를 동적할당 합니다.
*  이후 input 이미지에서 픽셀 데이터를 읽어와 img 포인터가 가리키게 하고, output의 write를 위한 파일 포인터를 새롭게 열어줍니다.
* 이 때, 주의할 점은 read이던 write이던 "rb", "wb" 를 사용해야 한다는 점입니다. "r"과 "w"는 텍스트 파일을 읽고 쓸 때 사용하는 것이므로, 바이너리 데이터를 읽고 쓸 때에는 오류가 발생할 수 있습니다.

-------

~~~c
	for (int y = infoHeader.biHeight - 1; y >= 0; y--)
	{
		for (int x = 0; x < infoHeader.biWidth; x++)
		{
			int index = (x * PIXEL_SIZE) + (y * ((infoHeader.biWidth * PIXEL_SIZE) + padding));

			RGBTRIPLE *pixel = (RGBTRIPLE *)&img[index];

			unsigned char gray = (pixel->rgbtBlue + pixel->rgbtGreen + pixel->rgbtRed) / 3;

			img_output[x + y * (infoHeader.biWidth + padding)] = gray;
		}
	}
~~~

* 픽셀 데이터 RGB 값의 평균을 구해 img_output(output의 픽셀 데이터 부분)에 저장하는 단계입니다. 
* RGB값의 평균을 구하는 코드는 이전과 동일하고, img_output에는 해당 픽셀이 표현하는 색상의 인덱스가 저장되도록 코드를 구성합니다. 앞서 만들었던 그레이 스케일을 위한 RGBQUAD를 보면, 인덱스와 RGB값이 동일하게 설정되어 있습니다. 즉, RGB값의 평균(gray)이 곧 색상의 인덱스 입니다.
* input 이미지는 24비트 이므로 for문 한번을 돌 때 index가 3씩 커져야 하지만, output 이미지는 8비트 이므로  index가 1씩 커져야 한다는 것만 주의해서 코드를 만들면 됩니다.

----

~~~C
	fileHeader.bfOffBits = sizeof(FILEHEADER) + sizeof(INFOHEADER) + sizeof(RGBQUAD) * 256;
	fileHeader.bfSize = infoHeader.biWidth * infoHeader.biHeight + fileHeader.bfOffBits;
	infoHeader.biBitCount = 8;
	infoHeader.biClrUsed = 256;

	fwrite(&fileHeader, sizeof(FILEHEADER), 1, fp);
	fwrite(&infoHeader, sizeof(INFOHEADER), 1, fp);
	fwrite(&rgbQuad, sizeof(RGBQUAD), 256, fp);
	fwrite(img_output, size_output, 1, fp);

	fclose(fp);
	free(img);
	free(img_output);

	return 0;
}
~~~

* 파일헤더와 정보헤더에서 바뀌어야 할 부분을 수정해주어야 합니다. 바뀌어야 할 부분은 다음과 같습니다.
  * bfOffBits: 색상테이블이 추가되었으므로 픽셀데이터 시작의 오프셋이 달라졌습니다. 
  * bfSize: 당연히 파일의 사이즈가 달라집니다. 
  * biBitCount: 픽셀 당 비트 수를 나타냅니다. 24에서 8로 수정되어야 합니다.
  * biClrused: 사용하는 색상의 개수를 나타냅니다. 24비트 트루컬러 이미지 일 때는 색상 테이블을 사용하지 않으므로 0이 저장되어 있지만, 8비트에서는 256가지의 색상을 사용하므로 256이 들어가야 합니다.
* 바뀌어야 할 부분을 수정한 뒤, fwrite를 통해 비트맵 파일을 작성합니다. 

-----



## 5. 전체 코드

~~~c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>

#define PIXEL_SIZE 3
#define PIXEL_ALIGN 4

#pragma pack(push, 1)
typedef struct FILEHEADER
{
	unsigned short bfType;
	unsigned long bfSize;
	unsigned short bfReserved1;
	unsigned short bfReserved2;
	unsigned long bfOffBits;
} FILEHEADER;

typedef struct INFOHEADER
{
	unsigned long biSize;
	long biWidth;
	long biHeight;
	unsigned short biPlanes;
	unsigned short biBitCount;
	unsigned long biCompression;
	unsigned long biSizeImage;
	long biXPelsPerMeter;
	long biYPelsPerMeter;
	unsigned long biClrUsed;
	unsigned long biClrImportant;
} INFOHEADER;

typedef struct RGBTRIPLE
{
	unsigned char rgbtBlue;
	unsigned char rgbtGreen;
	unsigned char rgbtRed;
} RGBTRIPLE;

typedef struct RGBQUAD
{
	unsigned char rgbBlue;
	unsigned char rgbGreen;
	unsigned char rgbRed;
	unsigned char rgbReserved;
} RGBQUAD;
#pragma pack(pop)

int main()
{
	FILE *fp;
	FILEHEADER fileHeader;
	INFOHEADER infoHeader;
	RGBQUAD rgbQuad[256];

	unsigned char *img, *img_output;
	int padding, size, size_output;

	for (int i = 0; i < 256; i++)
	{
		rgbQuad[i].rgbBlue = i;
		rgbQuad[i].rgbGreen = i;
		rgbQuad[i].rgbRed = i;
		rgbQuad[i].rgbReserved = 0;
	}
	
	fp = fopen("lena512.bmp", "rb");
	if (fp == NULL) return 0;

	if (fread(&fileHeader, sizeof(FILEHEADER), 1, fp) < 1)
	{
		fclose(fp);
		return 0;
	}
	if (fileHeader.bfType != 'MB')
	{
		fclose(fp);
		return 0;
	}
	
	if (fread(&infoHeader, sizeof(INFOHEADER), 1, fp) < 1)
	{
		fclose(fp);
		return 0;
	}

	padding = (PIXEL_ALIGN - ((infoHeader.biWidth * PIXEL_SIZE) % PIXEL_ALIGN)) % PIXEL_ALIGN;
	size = (infoHeader.biWidth * PIXEL_SIZE + padding) * infoHeader.biHeight;
	size_output = (infoHeader.biWidth * 1 + padding) * infoHeader.biHeight;

	img = (char*)malloc(size);
	img_output = (char*)malloc(size_output);

	if (fread(img, size, 1, fp) < 1)
	{
		fclose(fp);
		free(img);
		return 0;
	}
	fclose(fp);

	fp = fopen("gray.bmp", "wb");
	if (fp == NULL) return 0;
	
	for (int y = infoHeader.biHeight - 1; y >= 0; y--)
	{
		for (int x = 0; x < infoHeader.biWidth; x++)
		{
			int index = (x * PIXEL_SIZE) + (y * ((infoHeader.biWidth * PIXEL_SIZE) + padding));

			RGBTRIPLE *pixel = (RGBTRIPLE *)&img[index];

			unsigned char gray = (pixel->rgbtBlue + pixel->rgbtGreen + pixel->rgbtRed) / 3;

			img_output[x + y * (infoHeader.biWidth + padding)] = gray;
		}
	}
	
	fileHeader.bfOffBits = sizeof(FILEHEADER) + sizeof(INFOHEADER) + sizeof(RGBQUAD) * 256;
	fileHeader.bfSize = infoHeader.biWidth * infoHeader.biHeight + fileHeader.bfOffBits;
	infoHeader.biBitCount = 8;
	infoHeader.biClrUsed = 256;

	fwrite(&fileHeader, sizeof(FILEHEADER), 1, fp);
	fwrite(&infoHeader, sizeof(INFOHEADER), 1, fp);
	fwrite(&rgbQuad, sizeof(RGBQUAD), 256, fp);
	fwrite(img_output, size_output, 1, fp);

	fclose(fp);
	free(img);
	free(img_output);

	return 0;
}
~~~

