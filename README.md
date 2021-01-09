# Project2
## 主要功能：
* 输入一张图片后可以通过计算来判断其人脸的比例、可能性。
## 程序实现：
（滤波器数据等都已经给出）
### 函数：
* 第一个函数：
```cpp
 float* conv2(float* input, int rows, int cols, int channels,conv_param& c) {
    float* result;
    result = new float[c.out_channels * (2 + rows/c.stride) * (2 + cols/c.stride)]{ 0 };
    for (int i = 0; i < c.out_channels; i++) {
        for (int x = 0; x < rows/c.stride; x++) {
            for (int y = 0; y < cols/c.stride; y++) {
                for (int a = 0; a < 3; a++) {
                    for (int b = 0; b < 3; b++) {
                        for (int j = 0; j < channels; j++) {
                            result[i * (2 +( rows / c.stride)) * (2 + (cols / c.stride)) + (x + 1) * (2 + cols / c.stride) + y + 1] += input[j * (2 + rows) * (cols + 2) + (x * c.stride + a) * (2 + cols) + y * c.stride + b] * c.p_weight[i * 9 * channels + j * 9 + a * 3 + b];




                        }
                        


                    }
                   
                }
                result[(2 + rows / c.stride) * (2 + cols / c.stride) * i + (1 + x) * (2 + cols / c.stride) + y + 1] += c.p_bias[i];
            }
        
            
        }
           
    }
 
    for (int i = 0; i < c.out_channels * (2 + rows / c.stride) * (2 + cols / c.stride); i++) {

        if (result[i] < 0) { result[i] = 0; }
    }
    delete[]input;
    return result;
    delete[] result;
}


float* conv(float* input, int rows, int cols, int channels, conv_param& c) {
    float* result1 = new float[(rows / c.stride) * (cols / c.stride) * c.out_channels]{ 0 };
    int n = 0;
    for (int i = 0; i < c.out_channels; i++) {
        for (int x = 0; x < rows; x += c.stride) {
            for (int y = 0; y < cols; y += c.stride) {
                for (int j = 0; j < channels; j++) {
                    for (int a = 0; a < 3; a++) {
                        for (int b = 0; b < 3; b++) {
                            result1[n] += input[j * (rows + 2) * (cols + 2) + (x + a) * (cols + 2) + y + b] * c.p_weight[i * channels * 9 + j * 9 + a * 3 + b];
                        }
                    }
                }
                n++;
            }
        }
    }


    
    for (int i = 0; i < c.out_channels; i++) {
        for (int j = 0; j < (rows / c.stride) * (cols / c.stride); j++) {
            result1[i * (rows / c.stride) * (cols / c.stride) + j] += c.p_bias[i];
        }
    }
    for (int i = 0; i < (rows / c.stride) * (cols / c.stride) * c.out_channels; i++) {
        if (result1[i] < 0)result1[i] = 0;
    }

    
    float* result = new float[(rows / c.stride + 2) * (cols / c.stride + 2) * c.out_channels]{ 0 };

    int rows1 = rows / c.stride;
    int cols1 = cols / c.stride;
    for (int i = 0; i < c.out_channels; i++) {
        for (int x = 0; x < rows1; x++) {
            for (int y = 0; y < cols1; y++) {
                result[i * (rows1 + 2) * (cols1 + 2) + (x + 1) * (2  + cols1) + y + 1] = result1[i * rows1 * cols1 + x * cols1+ y];

            }
        }
    }

    
    delete[]result1;
    return result;

}
```
卷积层的函数代码，包括卷积计算、RELu（将小于0的值返回成0），得到结果矩阵。
* 第二个函数：
```cpp
float* pool(float * conv0, int rows,int cols,int channels){
    float* result = new float[(rows / 2 + 2) * (cols / 2 + 2) * channels]{ 0 };
    for (int i = 0; i < rows / 2; i++) {
        for (int j = 0; j < cols / 2; j++) {
            for (int x = 0; x < channels; x++) {
                result[x * (rows / 2 + 2) * (cols / 2 + 2) + (1 + i) * (cols / 2 + 2) + j + 1] = conv0[x * (rows + 2) * (cols + 2) + (1 + 2 * i) * (cols + 2) + 2 * j + 1];
                if (conv0[x * (rows + 2) * (cols + 2) + (1 + 2 * i) * (cols + 2) + 2 * j + 2] > result[x * (rows / 2 + 2) * (cols / 2 + 2) + (1 + i) * (cols / 2 + 2) + j + 1])result[x * (rows / 2 + 2) * (cols / 2 + 2) + (1 + i) * (cols / 2 + 2) + j + 1] = conv0[x * (rows + 2) * (cols + 2) + (1 + 2 * i) * (cols + 2) + 2 * j + 2];
                if (conv0[x * (rows + 2) * (cols + 2) + (2 + 2 * i) * (cols + 2) + 2 * j + 1] > result[x * (rows / 2 + 2) * (cols / 2 + 2) + (1 + i) * (cols / 2 + 2) + j + 1])result[x * (rows / 2 + 2) * (cols / 2 + 2) + (1 + i) * (cols / 2 + 2) + j + 1] = conv0[x * (rows + 2) * (cols + 2) + (2 + 2 * i) * (cols + 2) + 2 * j + 1];
                if (conv0[x * (rows + 2) * (cols + 2) + (2 + 2 * i) * (cols + 2) + 2 * j + 2] > result[x * (rows / 2 + 2) * (cols / 2 + 2) + (1 + i) * (cols / 2 + 2) + j + 1])result[x * (rows / 2 + 2) * (cols / 2 + 2) + (1 + i) * (cols / 2 + 2) + j + 1] = conv0[x * (rows + 2) * (cols + 2) + (2 + 2 * i) * (cols + 2) + 2 * j + 2];
            }
        }
    }
    return result;
}

```
池化层的函数代码，取每个2X2的矩阵中的最大元素为一个元素，组成一个新的矩阵（长宽为原来的二分之一，通道数不变）

### 主程序：
```cpp
int main(){
    Mat image = imread("bg.jpg");
    float* input0_0;
    input0_0 = new float[(image.rows + 2) * (image.cols + 2) * 3]{ 0 };
    for (int i = 0; i < image.rows; i++) {
        for (int j = 0; j < image.cols; j++) {
            input0_0[(i+1) * (image.cols + 2) + j + 1] = (float)image.at<Vec3b>(i, j)[2] / (float)255;
            input0_0[(i+1) * (image.cols + 2) + j + 1+ (image.rows + 2) * (image.cols + 2)*1] = (float)image.at<Vec3b>(i, j)[1] / (float)255;
            input0_0[(i+1)* (image.cols + 2) + j + 1+ (image.rows + 2) * (image.cols + 2)*2] = (float)image.at<Vec3b>(i, j)[0] / (float)255;
        }
 }


    if (image.data == nullptr)
    {
        std::cerr << "图片文件不存在" << std::endl;
        return 0;
    }
    



    float* topool;
    topool = conv2(input0_0, 128, 128, 3, conv_params[0]);
    input0_0 = pool(topool, 64, 64, 16);
    topool = conv2(input0_0, 32, 32, 16, conv_params[1]);
    input0_0 = pool(topool, 32, 32, 32);
    topool = conv2(input0_0, 16, 16, 32, conv_params[2]);
    
   
    
   

    float* result = new float[2]{ 0 };
    for (int i = 0; i < 32; i++) {
        for (int x = 0; x < 8; x++) {
            for (int y = 0; y < 8; y++) {
                result[0] += fc_params[0].p_weight[i*64+8*x+y] * topool[i * 100 + (x + 1) * 10 + y + 1];
                result[1] += fc_params[0].p_weight[i * 64 + 8 * x + y+2048] * topool[i * 100 + (x + 1) * 10 + y + 1];
            }
        }
    }
    
   

    
    
    result[0] += fc_params[0].p_bias[0];
    result[1] += fc_params[0].p_bias[1];
    std::cout << "bg score: " << exp(result[0]) / (exp(result[0]) + exp(result[1])) << std::endl;
    std::cout << "face score: " << exp(result[1]) / (exp(result[0]) + exp(result[1])) << std::endl;
    
    return 0;
}
```
先读取图片转换为MAT矩阵，在转换为以为矩阵，同时缩小每个元素的大小，使其处于（0，1）区间，之后进行识别。

## 结果：
* 人脸：
![]
(https://github.com/JOJO-debug622/Project2/blob/main/%E6%95%B0%E6%8D%AE2.PNG)
* 背景：
![]
(https://github.com/JOJO-debug622/Project2/blob/main/%E6%95%B0%E6%8D%AE1.PNG)

