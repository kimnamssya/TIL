## 영상 이진화(Binarization)

**영상 이진화**는 이미지의 모든 픽셀을 흑과 백으로만 표현하는 것을 말합니다. 픽셀 데이터의 값을 특정 경계(threshold)를 기준으로, 기준보다 높은 값을 가지면 픽셀의 값을 255(흰색), 낮은 값을 가지면 0(검은색)으로 바꾸어주어 영상 이진화를 실현할 수 있습니다. 



기준이 되는 임계점(threshold point)을 설정하는 방법에 따라 전역 이진화, 지역 이진화 등으로 구분되지만, 오늘은 **모든 픽셀의 값에 같은 임계점을 적용하는** 전역 이진화 방법으로 영상 이진화를 구현해보겠습니다.

---

사실 앞서 실습했던 [Change truecolor image to grayscale image](https://github.com/skadud8951/TIL/blob/main/Image_processing/change_gray2.md) 코드에 간단한 수정을 하면  Binarization을 실현할 수 있습니다.

~~~c
	for (int y = infoHeader.biHeight - 1; y >= 0; y--)
	{
		for (int x = 0; x < infoHeader.biWidth; x++)
		{
			int index = (x * PIXEL_SIZE) + (y * ((infoHeader.biWidth * PIXEL_SIZE) + padding));

			RGBTRIPLE *pixel = (RGBTRIPLE *)&img[index];

			unsigned char gray = (pixel->rgbtBlue + pixel->rgbtGreen + pixel->rgbtRed) / 3;
			
			if (gray <= 128) gray = 0;
			else gray = 255;

			img_output[x + y * (infoHeader.biWidth + padding)] = gray;
		}
	}
~~~

위 코드는 24비트 트루컬러 이미지를 8비트 그레이스케일 이미지로 바꾸는 코드 중 일부입니다. 24비트 이미지의 RGB 평균 값을 그레이 스케일 이미지의 픽셀 데이터 값으로 넣어주게 되는데, 이 때 if문을 사용하여 특정 임계값을 기준으로 픽셀 데이터 값을 0과 255로 변경해주면 됩니다.

~~~C
if (gray <= 128) gray = 0;
	else gray = 255;
~~~

---



## 2. 전체 코드와 실행 결과

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

	fp = fopen("binarization.bmp", "wb");
	if (fp == NULL) return 0;

	for (int y = infoHeader.biHeight - 1; y >= 0; y--)
	{
		for (int x = 0; x < infoHeader.biWidth; x++)
		{
			int index = (x * PIXEL_SIZE) + (y * ((infoHeader.biWidth * PIXEL_SIZE) + padding));

			RGBTRIPLE *pixel = (RGBTRIPLE *)&img[index];

			unsigned char gray = (pixel->rgbtBlue + pixel->rgbtGreen + pixel->rgbtRed) / 3;
			
			if (gray <= 128) gray = 0;
			else gray = 255;

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

<img src="https://user-images.githubusercontent.com/48755185/103415076-f2413e00-4bc3-11eb-9704-0c0bf02db14e.JPG" alt="binary" style="zoom:50%;" />
