## TextKit
### iOS 文本功能演化史
>iOS 2 : UILabel、UITextField、UITextView，只支持纯文本展示和输入  
>
>iOS 3 : 新增复制粘贴选择功能，Data Detector  
>
>iOS 3.2  CoreText、UITextInput
> 
>iOS 6 引入Attributed Strings，在这之前只能使用Core Text处理富文本，用UIWebView展示 
>
>iOS 7 **Text Kit**  
 
![ios6text](http://beyondvincent.com/images/2013/11/24.jpg)  
Figure 1  Text Framework before iOS 7
![textkitDiagram](http://beyondvincent.com/images/2013/11/23.jpg)  
Figure 2  Text Kit Framework Position
### 文本属性
Text Kit处理三种文本属性：字符属性，段落属性和文档属性。
  
* 字符属性  NSAttributedString中的attributes
	* 字体
	* 字体颜色
	* ... 
* 段落属性  NSParagraphStyle
	* 对齐
	* 缩进
	* 换行模式
	* 行间距
	* ... 
* 文档属性 
	* RTF
	* HTML
	* ...  


### 功能
* Dynamic Types  
* 对文字进行分页或多列排版
* 支持调整字间距、行间距、文字大小、字体、连字，断行和对齐等排版
* 支持凸版印刷效果(letterpress)  
* 支持数据类型的检测(例如链接、附件等)

### TextKit 中的类
**NSTextStorage**  
负责管理存储所有与文本属性相关的信息，例如字体或段落信息将要呈现的文本存储为属性字符串，并通知布局管理器文本内容的任何更改。是NSMutableAttributedString子类。  
**NSLayoutManager**  
获取存储的文本并将其呈现在屏幕上，排版布局引擎。  
**NSTextContainer** 
描述应用呈现文本的屏幕区域的几何形状。与UITextView关联。可以子类
化NSTextContainer自定义复杂形状。  
![textkitConcepts](https://koenig-media.raywenderlich.com/uploads/2013/09/TextKitStack.png)  
### TextKit 中的MVC
**Model**:  NSTextStorage、NSTextContainer  
**View**: UITextView  
**Controller**: NSLayoutManager  
![mvc](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Art/model_view_controller_2x.png)
### Reference
[Text Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009542-CH1-SW1)  
[TextKit WWDC13  Introducing Text Kit](https://developer.apple.com/videos/play/wwdc2013/210/)  
[TextKit WWDC18 TextKit Best Practices](https://developer.apple.com/videos/play/wwdc2018/221/)
