# Project2
## 主要功能：
* 输入一张图片后可以通过计算来判断其人脸的比例、可能性。
## 程序实现：
（滤波器数据等都已经给出）
### 函数：
* 第一个函数：
```cpp
float* conv(float* input, int rows, int cols, int channels,conv_param& c) {
    float* result;
    result = new float[c.out_channels * (2 + rows/c.stride) * (2 + cols/c.stride)]{ 0 };
    for (int i = 0; i < c.out_channels; i++) {
        for (int x = 0; x < rows/c.stride; x++) {
            for (int y = 0; y < cols/c.stride; y++) {
                for (int j = 0; j < 3; j++) {
                    for (int ptr = 1; ptr < 1+channels; ptr++) {
                        result[(2 + rows/c.stride) * (2 + cols/c.stride) * i + (1 + x) * (2 + cols/c.stride) + y + 1] += input[(2 + cols) *c.stride* x+(channels-ptr)*( rows+2)*(cols+2)+y*c.stride+j]*c.p_weight[i*9*c.in_channels+j+(ptr-1)*9];
                        result[(2 + rows/c.stride) * (2 + cols/c.stride) * i + (1 + x) * (2 + cols/c.stride) + y + 1] += input[(2 + cols) * (c.stride * x+1) + (channels - ptr) * (rows + 2) * (cols + 2) + c.stride*y+j] * c.p_weight[i * 9 * c.in_channels + j+3 + (ptr - 1) * 9];
                        result[(2 + rows / c.stride) * (2 + cols / c.stride) * i + (1 + x) * (2 + cols / c.stride) + y + 1] += input[(2 + cols) * (c.stride * x + 2) + (channels - ptr) * (rows + 2) * (cols + 2) + y * c.stride + j] * c.p_weight[i * 9 * c.in_channels + j + 6 + (ptr - 1) * 9];
                    }
                }
           
                result[(2 + rows / c.stride) * (2 + cols / c.stride) * i + (1 + x) * (2 + cols / c.stride) + y + 1] += c.p_bias[i];
            }
        
            
        }
    }
 
    int n = c.out_channels *(2+ rows / c.stride) *( 2+ cols / c.stride);
    for (int i = 0; i < n; i++) {   

        if (result[i] < 0) { result[i] = 0; }
    }
    delete[]input;
    return result;
    delete[] result;
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
int main() {
	Mat image = imread("bg.jpg");
    float* input0_0;
   
    input0_0 = new float[(image.rows + 2) * (2 + image.cols) * 3]{ 0 };
 


    if (image.data == nullptr)
    {
        std::cerr << "图片文件不存在" << std::endl;
        return 0;
    }

     for (size_t y = 0; y < image.rows; ++y) {

      
        unsigned char* row_ptr= image.ptr<unsigned char>(y);
        for (size_t x = 0; x < image.cols; ++x) {
            
            unsigned char* data_ptr = &row_ptr[x*image.channels()];
            
            
            input0_0[(x + 1) * (2 + image.cols) + y + 1] = float(image.at<Vec3b>(x, y)[0]) / (float)255.0;
            input0_0[(image.rows + 2) * (2 + image.cols) * 1 + (x + 1) * (2 + image.cols) + y + 1] = float(image.at<Vec3b>(x, y)[1]) / (float)255.0;
            input0_0[(image.rows + 2) * (2 + image.cols) * 2 + (x + 1) * (2 + image.cols) + y + 1] = float(image.at<Vec3b>(x, y)[2]) / (float)255.0;
            
        }
    }

    float* topool;
    topool = conv(input0_0, 128, 128, 3, conv_params[0]);
    input0_0 = pool(topool, 64, 64, 16);
    topool = conv(input0_0, 32, 32, 16, conv_params[1]);
    input0_0 = pool(topool, 32, 32, 32);
    topool = conv(input0_0, 16, 16, 32, conv_params[2]);

    float* result = new float[2]{ 0 };

    for (int i = 0; i < 2048; i++) {
        result[0] += fc_params[0].p_weight[i] * topool[i];
        result[1] += fc_params[0].p_weight[i + 2048] * topool[i];
    }
    result[0] += fc0_bias[0];
    result[1] += fc0_bias[1];

    std::cout << "bg score: " << result[0] << std::endl;
    std::cout << "face score: " << result[1] << std::endl;
    
  

	return 0;
}
```
先读取图片转换为MAT矩阵，在转换为以为矩阵，同时缩小每个元素的大小，使其处于（0，1）区间，之后进行识别。

## 结果：
* 人脸：

