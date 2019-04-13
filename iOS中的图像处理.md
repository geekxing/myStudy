## iOS中的图像处理

  1、Bitmap数据  
  2、Core Graphics  
  3、Core  Image  
  4、GPUImage  
### 图像
![Image Process ghost](https://koenig-media.raywenderlich.com/uploads/2014/03/ghost_tiny.png)  
有一组像素点组成，每个像素就是一个颜色值。  
颜色，如果用32位RGBA来表示的话，即4个字节。每个字节表示一个通道。 
1byte = 8bit，即每个通道可以有2^8=256个值。

* R表示red
* G表示green
* B表示blue
* A表示alpha（透明度）  

### 色彩空间
**RGB就是展示色彩空间的一种方法**   
另外两种常用的**HSV**和**YUV**
![color space](https://koenig-media.raywenderlich.com/uploads/2014/03/rgbvshsv.jpg)
**HUV**的各个字母的含义  

* Hue表示颜色
* Saturation表示饱和度
* Value表示亮度

**YUV**是电视机使用的色彩空间表示方法  
### 坐标系
在iOS系统中，`UIImage`和`UIView`使用左上为原点的坐标系；`Core Image`和`Core Graphics`使用左下为原点的坐标系。  
### 图片压缩
**存储原始图片会占用大量内存空间**  
一张8M个像素的32位RGBA图片，占用了大约32M字节的空间。
JPEG，PNG这些都是图片的压缩格式，可以节省图片解压占用的内存空间。
### 图片的像素
```
输出图片的像素点到日志

// 1.
CGImageRef inputCGImage = [image CGImage];
NSUInteger width = CGImageGetWidth(inputCGImage);
NSUInteger height = CGImageGetHeight(inputCGImage);

// 2.
NSUInteger bytesPerPixel = 4;
NSUInteger bytesPerRow = bytesPerPixel * width;
NSUInteger bitsPerComponent = 8;

UInt32 * pixels;
pixels = (UInt32 *) calloc(height * width, sizeof(UInt32));

// 3.
CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
CGContextRef context = CGBitmapContextCreate(pixels, width, height, bitsPerComponent, bytesPerRow, colorSpace, kCGImageAlphaPremultipliedLast | kCGBitmapByteOrder32Big);

// 4.
CGContextDrawImage(context, CGRectMake(0, 0, width, height), inputCGImage);

// 5. Cleanup
CGColorSpaceRelease(colorSpace);
CGContextRelease(context);
```
可以重点关注下

```
CGContextRef CGBitmapContextCreate(
void *data, 
size_t width,  
size_t height,  
size_t bitsPerComponent,   
size_t bytesPerRow, 
CGColorSpaceRef space, 
uint32_t bitmapInfo);
```
这个API的调用，怎么创建一个Bitmap绘制上下文。其中`void *data`即存储像素的数组
### 图像混合
我们知道，一个像素点有一个颜色值。如果图片有透明度，imageView有背景色，这个时候就会发生**alpha blending**。
混合颜色公式是：

```
NewColor = 
TopColor * TopColor.Alpha + 
BottomColor * (1 - TopColor.Alpha)

optimization

value = TopColor * TopColor.Alpha
NewColor = 
value + 
BottomColor * (1 - TopColor.Alpha)
```
Bitmap->CGImage->UIImage

```
CGImageRef newCGImage = CGBitmapContextCreateImage(context);
UIImage * processedImage = [UIImage imageWithCGImage:newCGImage];
```
### 灰度图片
直接把图片的每个像素的颜色的每个通道的值置为通道的平均值。
### 拓展思考
* 交换图片的r/b通道
* 图片增加10%亮度
* 操作像素来缩ghost放图片
	* 创建一个新的绘制上下文
	* 在这个新上下文中，计算每个像素点应该怎么从原图中拷贝过来
	* 如果您对原始坐标的计算落在像素之间，请尝试在附近像素之间进行插值。
如果您在最近的四个像素之间进行插值，那么您就实现了双线性缩放

### Core Graphics

**Graphics Context**: 持有绘图属性的全局状态机。  

* Bitmap context
* PDF context
* ...  

怎么获取context?

* `-drawRect:`方法中已经有系统创建的context，绘制完直接渲染到view上
* 在其他地方，可以自己创建一个context。比如
	* `CGContextCreate()`; 
	* `UIGraphicsBeginImageContext()`, 然后`UIGraphicsGetCurrentContext()`取出context

其中，第二个方法叫做**离屏渲染**，因为你绘制的东西没有立即展示，他们被存在离屏的buffer里。如果是Core Graphics，你可以从context中获取一个`UIImage`展示；如果是OpenGL，则直接和当前渲染的图片进行交换。

### Core Image
优势： 
  
* Apple的图片处理解决方案
* 相比**操作原始图片像素**和**Core Graphics**，有着更好的性能。CPU和GPU处理，达到实时渲染的性能。
* 提供大量的滤镜，并且可以使用 **Core Image Kernel Language** 来自己写滤镜。

```
- (UIImage *)processUsingCoreImage:(UIImage*)input {
  CIImage * inputCIImage = [[CIImage alloc] initWithImage:input];
  
  // 1. Create a grayscale filter
  CIFilter * grayFilter = [CIFilter filterWithName:@"CIColorControls"];
  [grayFilter setValue:@(0) forKeyPath:@"inputSaturation"];
  
  // 2. Create your ghost filter
  
  // Use Core Graphics for this
  UIImage * ghostImage = [self createPaddedGhostImageWithSize:input.size];
  CIImage * ghostCIImage = [[CIImage alloc] initWithImage:ghostImage];

  // 3. Apply alpha to Ghosty
  CIFilter * alphaFilter = [CIFilter filterWithName:@"CIColorMatrix"];
  CIVector * alphaVector = [CIVector vectorWithX:0 Y:0 Z:0.5 W:0];
  [alphaFilter setValue:alphaVector forKeyPath:@"inputAVector"];
  
  // 4. Alpha blend filter
  CIFilter * blendFilter = [CIFilter filterWithName:@"CISourceAtopCompositing"];
  
  // 5. Apply your filters
  [alphaFilter setValue:ghostCIImage forKeyPath:@"inputImage"];
  ghostCIImage = [alphaFilter outputImage];

  [blendFilter setValue:ghostCIImage forKeyPath:@"inputImage"];
  [blendFilter setValue:inputCIImage forKeyPath:@"inputBackgroundImage"];
  CIImage * blendOutput = [blendFilter outputImage];
  
  [grayFilter setValue:blendOutput forKeyPath:@"inputImage"];
  CIImage * outputCIImage = [grayFilter outputImage];
  
  // 6. Render your output image
  CIContext * context = [CIContext contextWithOptions:nil];
  CGImageRef outputCGImage = [context createCGImage:outputCIImage fromRect:[outputCIImage extent]];
  UIImage * outputImage = [UIImage imageWithCGImage:outputCGImage];
  CGImageRelease(outputCGImage);
  
  return outputImage;
}

```
代码最终达成的效果  
![core image](https://koenig-media.raywenderlich.com/uploads/2014/03/BuildnRun-3.png)

使用Core Image， **就是创建一系列的滤镜，让图片经过每一个滤镜，前一个滤镜的输出作为后一个滤镜的输入，这么链接起来，**最终得到处理好的图。
### GPUImage
隐藏复杂的OpenGL ES方法调用细节，暴露简单的接口，性能在某些方面胜过Core Image。
可以用 OpenGL Shading Language (GLSL) 写shader来自定义滤镜。  

```
- (UIImage *)processUsingGPUImage:(UIImage*)input {
  
  // 1. Create the GPUImagePictures
  GPUImagePicture * inputGPUImage = [[GPUImagePicture alloc] initWithImage:input];
  
  UIImage * ghostImage = [self createPaddedGhostImageWithSize:input.size];
  GPUImagePicture * ghostGPUImage = [[GPUImagePicture alloc] initWithImage:ghostImage];

  // 2. Set up the filter chain
  GPUImageAlphaBlendFilter * alphaBlendFilter = [[GPUImageAlphaBlendFilter alloc] init];
  alphaBlendFilter.mix = 0.5;
  
  [inputGPUImage addTarget:alphaBlendFilter atTextureLocation:0];
  [ghostGPUImage addTarget:alphaBlendFilter atTextureLocation:1];
  
  GPUImageGrayscaleFilter * grayscaleFilter = [[GPUImageGrayscaleFilter alloc] init];
  
  [alphaBlendFilter addTarget:grayscaleFilter];
  
  // 3. Process & grab output image
  [grayscaleFilter useNextFrameForImageCapture];
  [inputGPUImage processImage];
  [ghostGPUImage processImage];
  
  UIImage * output = [grayscaleFilter imageFromCurrentFramebuffer];
  
  return output;
}

```
过程可以用下图展示:  
![gpuimage](https://koenig-media.raywenderlich.com/uploads/2014/03/GPUFilterChain.png)
