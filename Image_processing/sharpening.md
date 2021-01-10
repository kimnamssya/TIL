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

typedef struct RGBTRIPLE
{
	unsigned char rgbtBlue;
	unsigned char rgbtGreen;
	unsigned char rgbtRed;
} RGBTRIPLE;
#pragma pack(pop)

int main()
{
	FILE *ifp, *ofp;
	FILEHEADER fileheader;
	INFOHEADER infoheader;
	RGBTRIPLE **pixel, **newPixel;
	int width, height, padding, size;
	int padding_size;

	ifp = fopen("lena512.bmp", "rb");
	if (ifp == NULL) return 0;

	if (fread(&fileheader, sizeof(FILEHEADER), 1, ifp) < 1)
	{
		fclose(ifp);
		return 0;
	}
	if (fileheader.bfType != 'MB')
	{
		fclose(ifp);
		return 0;
	}

	if (fread(&infoheader, sizeof(INFOHEADER), 1, ifp) < 1)
	{
		fclose(ifp);
		return 0;
	}

	width = infoheader.biWidth;
	height = infoheader.biHeight;

	pixel = (RGBTRIPLE**)malloc(height * sizeof(RGBTRIPLE*));
	newPixel = (RGBTRIPLE**)malloc(height * sizeof(RGBTRIPLE*));
	for (int i = 0; i < height; i++)
	{
		pixel[i] = (RGBTRIPLE*)malloc(width * sizeof(RGBTRIPLE));
		newPixel[i] = (RGBTRIPLE*)malloc(width * sizeof(RGBTRIPLE));
		fread(pixel[i], sizeof(RGBTRIPLE) * width, 1, ifp);
	}

	fseek(ifp, fileheader.bfOffBits, SEEK_SET);
	for (int i = 0; i < height; i++)
	{
		fread(newPixel[i], sizeof(RGBTRIPLE) * width, 1, ifp);
	}
	unsigned char a = 10;
	unsigned char b = 20;
	b = a - b;
	printf("%d\n", b);
	for (int y = 0; y < height; y++)
	{
		for (int x = 0; x < width; x++)
		{
			if (x > 1 && x < width - 1 && y>0 && y < height - 1)
			{
				if (((5 * pixel[x][y].rgbtBlue) - pixel[x + 1][y].rgbtBlue - pixel[x][y + 1].rgbtBlue - pixel[x - 1][y].rgbtBlue - pixel[x][y - 1].rgbtBlue) > 0)
				{
					if(((5 * pixel[x][y].rgbtBlue) - pixel[x + 1][y].rgbtBlue - pixel[x][y + 1].rgbtBlue - pixel[x - 1][y].rgbtBlue - pixel[x][y - 1].rgbtBlue) <= 255)
					newPixel[x][y].rgbtBlue = ((5 * pixel[x][y].rgbtBlue) - pixel[x + 1][y].rgbtBlue - pixel[x][y + 1].rgbtBlue - pixel[x - 1][y].rgbtBlue - pixel[x][y - 1].rgbtBlue);
					else newPixel[x][y].rgbtBlue = 255;
				}
				else newPixel[x][y].rgbtBlue = 0;

				if (((5 * pixel[x][y].rgbtGreen) - pixel[x + 1][y].rgbtGreen - pixel[x][y + 1].rgbtGreen - pixel[x - 1][y].rgbtGreen - pixel[x][y - 1].rgbtGreen) > 0)
				{
					if(((5 * pixel[x][y].rgbtGreen) - pixel[x + 1][y].rgbtGreen - pixel[x][y + 1].rgbtGreen - pixel[x - 1][y].rgbtGreen - pixel[x][y - 1].rgbtGreen) <= 255)
					newPixel[x][y].rgbtGreen = ((5 * pixel[x][y].rgbtGreen) - pixel[x + 1][y].rgbtGreen - pixel[x][y + 1].rgbtGreen - pixel[x - 1][y].rgbtGreen - pixel[x][y - 1].rgbtGreen);
					else newPixel[x][y].rgbtGreen = 255;
				}
				else newPixel[x][y].rgbtGreen = 0;

				if (((5 * pixel[x][y].rgbtRed) - pixel[x + 1][y].rgbtRed - pixel[x][y + 1].rgbtRed - pixel[x - 1][y].rgbtRed - pixel[x][y - 1].rgbtRed))
				{
					if(((5 * pixel[x][y].rgbtRed) - pixel[x + 1][y].rgbtRed - pixel[x][y + 1].rgbtRed - pixel[x - 1][y].rgbtRed - pixel[x][y - 1].rgbtRed) <=255)
					newPixel[x][y].rgbtRed = ((5 * pixel[x][y].rgbtRed) - pixel[x + 1][y].rgbtRed - pixel[x][y + 1].rgbtRed - pixel[x - 1][y].rgbtRed - pixel[x][y - 1].rgbtRed);
					else newPixel[x][y].rgbtRed = 255;
				}
				else newPixel[x][y].rgbtRed = 0;
				//printf("%d\n", newPixel[x][y].rgbtRed);
			}
		}
	}

	fclose(ifp);

	ofp = fopen("sharpening.bmp", "wb");
	fwrite(&fileheader, sizeof(FILEHEADER), 1, ofp);
	fwrite(&infoheader, sizeof(INFOHEADER), 1, ofp);
	for (int i = 0; i < height; i++)
	{
		fwrite(newPixel[i], sizeof(RGBTRIPLE) * width, 1, ofp);
	}

	fclose(ofp);

	for (int i = 0; i < height; i++)
	{
		free(pixel[i]);
	}
	free(pixel);

	return 0;
}

~~~
