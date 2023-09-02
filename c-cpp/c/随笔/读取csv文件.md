

##### 读取 csv 文件

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include "fnorm.h"

size_t getRow(const char *filePath)
{
    size_t row = 0;
    char line[1024];
    FILE *FP = fopen(filePath, "r");
    while (fgets(line, 1024, FP))
    {
        row++;
    }
    fclose(FP);
    return row;
}

size_t getCol(const char *filePath)
{
    size_t col = 0;
    char line[2048];
    FILE *FP = fopen(filePath, "r");
    fgets(line, 1024, FP);

    char *token = strtok(line, ",");
    while (token)
    {
        col++;
        token = strtok(NULL, ",");
    }
    fclose(FP);
    return col;
}

void readMatrix(const char *filePath, double *A)
{
    FILE *fp = NULL;
    char *line, *numStr;
    char buffer[2048];
    int index = 0;
    if ((fp = fopen(filePath, "r")) != NULL)
    {
        while ((line = fgets(buffer, sizeof(buffer), fp)) != NULL)
        {
            numStr = strtok(line, ",");
            while (numStr != NULL)
            {
                A[index++] = atof(numStr);
                numStr = strtok(NULL, ",");
            }
        }
        fclose(fp);
        fp = NULL;
    }
    else
    {
        printf("file open failure!\n");
    }
}

int main(int argc, char **argv)
{
    const char *filePath = argv[1];

    size_t row, col;
    row = getRow(filePath);
    col = getCol(filePath);
    double *A = (double *)malloc(sizeof(double) * (row * col));
    readMatrix(filePath, A);
    double ans = fnorm(A, row, col);
    printf("%lf\n", ans);
    
    return 0;
}
```

