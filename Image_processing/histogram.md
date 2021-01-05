## Histogram (미완성)

~~~c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>

#define PIXEL_SIZE 1
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
#pragma pack(pop)

int main()
{
	FILE *fp;
	FILEHEADER fileheader;
	INFOHEADER infoheader;
	RGBQUAD rgbQuad[256];
	float histogram[256];
	unsigned char *result;
	unsigned char *img;
	int size, padding;

	for (int i = 0; i < 256; i++)
	{
		histogram[i] = 0;
	}

	fp = fopen("gray.bmp", "rb"); //grayscale
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
	
	if (fread(&infoheader, sizeof(INFOHEADER), 1, fp) < 1)
	{
		fclose(fp);
		return 0;
	}

	if (fread(&rgbQuad, sizeof(RGBQUAD), 256, fp) < 256)
	{
		fclose(fp);
		return 0;
	}

//	fseek(fp, fileheader.bfOffBits, SEEK_SET);
	
	padding = (PIXEL_ALIGN - ((infoheader.biWidth * PIXEL_SIZE) % PIXEL_ALIGN)) % PIXEL_ALIGN;
	size = (infoheader.biWidth + padding) * infoheader.biHeight;

	img = (char*)malloc(size);
	result = (char*)malloc(256 * 256);

	if (fread(img, size, 1, fp) < 1)
	{
		fclose(fp);
		free(img);
		return 0;
	}
	float sum = 0;
	for (int y = infoheader.biHeight - 1; y >= 0; y--)
	{
		for (int x = 0; x < infoheader.biWidth; x++)
		{
			int index = img[x + y * (infoheader.biHeight + padding)];
			//printf("%d\n", index);
			histogram[index]++;
			sum++;
		}
	}
	float scale = (255.0 / sum);
	for (int i = 0; i < 256; i++) 
	{
		histogram[i] = histogram[i] * scale;
	}
	for (int i = 0; i < 256; i++)
	{
		printf("%d\n", (int)histogram[i]);
	}
	for (int x = 0; x < 256; x++)
	{
		int y = 0;
		while((int)histogram[x] != y)
		{
			result[x + y * (infoheader.biHeight + padding)] = 0;
			y++;
		}
		while (y != 256)
		{
			result[x + y * (infoheader.biHeight + padding)] = 255;
			y++;
		}
	}

	fclose(fp);
	
	fp = fopen("result.bmp", "wb");

	infoheader.biHeight = 256;
	infoheader.biWidth = 256;
	fileheader.bfSize = infoheader.biWidth * (infoheader.biHeight + padding) + fileheader.bfOffBits;

	fwrite(&fileheader, sizeof(FILEHEADER), 1, fp);
	fwrite(&infoheader, sizeof(INFOHEADER), 1, fp);
	fwrite(&rgbQuad, sizeof(RGBQUAD), 256, fp);
	fwrite(result, 1, 256 * 256, fp);
	
	fclose(fp);
	free(img);
	free(result);
	return 0;

}
~~~
