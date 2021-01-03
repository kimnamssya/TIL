## Histogram (미완성)

~~~c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>

#define PIXEL_SIZE 1
#define PIXEL_ALIGN 4

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
} INFOHEADER;

typedef struct RGBQUAD
{
	unsigned char rgbBlue;
	unsigned char rgbGreen;
	unsigned char rgbRed;
	unsigned char rgbReserved;
} RGBQUAD;

int main()
{
	FILE *fp;
	FILEHEADER fileheader;
	INFOHEADER infoheader;
	int histogram[256];
	RGBQUAD rgbquad[256];
	unsigned char *img;
	int size, padding;

	for (int i = 0; i < 256; i++)
	{
		histogram[i] = 0;
	}

	fp = fopen("lena512.bmp", "rb"); //grayscale
	if (fp == NULL) return 0;

	if (fread(&fileheader, sizeof(FILEHEADER), 1, fp) < 1)
	{
		fclose(fp);
		return 0;
	}

	if (fileheader.bfType != 'MB')
	{
		fclose(fp);
		return 0;
	}

	fseek(fp, fileheader.bfOffBits, SEEK_SET);

	
	img = (char*)malloc();
	fread(img, sizeof(char), 256, fp);

}
~~~
