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
