#前言
Quartz 2D是一个二维图形绘制引擎，支持iOS环境和Mac OS X环境。我们可以使用Quartz 2D API来实现许多功能，如基本路径的绘制、透明度、描影、绘制阴影、透明层、颜色管理、反锯齿、PDF文档生成和PDF元数据访问。在需要的时候，Quartz 2D还可以借助图形硬件的功能。

在Mac OS X中，Quartz 2D可以与其它图形图像技术混合使用，如Core Image、Core Video、OpenGL、QuickTime。例如，通过使用 QuickTime的GraphicsImportCreateCGImage函数，可以用 Quartz从一个 QuickTime图形导入器中创建一个图像。

#page
Quartz 2D在图像中使用了绘画者模型(painter’s model)。在绘画者模型中，每个连续的绘制操作都是将一个绘制层(a layer of ‘paint’)放置于一个画布(‘canvas’)，我们通常称这个画布为Page。 Page上的绘图可以通过额外的绘制操作来叠加更多的绘图。Page上的图形对象只能通过叠加更多的绘图来改变。这个模型允许我们使用小的图元来构建复杂的图形。

图1-1展示了绘画者模型如何工作。从图中可以看出不同的绘制顺序所产生的效果不一样。

![](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/painters_model.gif)

Page可以是一张纸(如果输出设备是打印机)，也可以是虚拟的纸张(如果输出设备是PDF文件)，还可以是bitmap图像。这根据实际使用的graphics context而定。

#绘制目标：Graphics Context
Graphics Context是一个数据类型(CGContextRef)，用于封装Quartz绘制图像到输出设备的信息。设备可以是PDF文件、bitmap或者显示器的窗口上。Graphics Context中的信息包括在Page中的图像的图形绘制参数和设备相关的表现形式。Quartz中所有的对象都是绘制到一个Graphics Context中。

我们可以将Graphics Context想像成绘制目标，如图1-2所示。当用Quartz绘图时，所有设备相关的特性都包含在我们所使用的Graphics Context中。换句话说，我们可以简单地给Quartz绘图序列指定不同的Graphics Context，就可将相同的图像绘制到不同的设备上。我们不需要处理任何设备相关的计算；这些都由Quartz替我们完成。

![](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/draw_destinations.gif)

Quartz提供了以下几种类型的Graphics Context。

* Bitmap Graphics Context
* PDF Graphics Context
* Window Graphics Context
* Layer Context
* Post Graphics Context

#Quartz 2D 数据类型
除了 Graphics Context 之外，Quartz 2D API还定义一些数据类型。由于这些API就Core Graphics框架的一部分，所以这些数据类型都是以CG开头的。

Quartz 2D使用这些数据类型来创建对象，通过操作这些对象来获取特定的图形。图1-3例举了三个使用Quartz 2D的绘制操作所获得的图像。

![](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/drawing_primitives.gif)

下面列出了Quartz 2D包含的数据类型：

* CGPathRef：用于向量图，可创建路径，并进行填充或描画(stroke)
* CGImageRef：用于表示bitmap图像和基于采样数据的bitmap图像遮罩。
* CGLayerRef：用于表示可用于重复绘制(如背景)和幕后(offscreen)绘制的绘画层
* CGPatternRef：用于重绘图
* CGShadingRef、CGGradientRef：用于绘制渐变
* CGFunctionRef：用于定义回调函数，该函数包含一个随机的浮点值参数。当为阴影创建渐变时使用该类型
* CGColorRef, CGColorSpaceRef：用于告诉Quartz如何解释颜色
* CGImageSourceRef,CGImageDestinationRef：用于在Quartz中移入移出数据
* CGFontRef：用于绘制文本
* CGPDFDictionaryRef, CGPDFObjectRef, CGPDFPageRef, CGPDFStream, CGPDFStringRef, and CGPDFArrayRef：用于访问PDF的元数据
* CGPDFScannerRef, CGPDFContentStreamRef：用于解析PDF元数据
* CGPSConverterRef：用于将PostScript转化成PDF。在iOS中不能使用。

#图形状态

Quartz通过修改当前图形状态(current graphics state)来修改绘制操作的结果。图形状态包含用于绘制程序的参数。绘制程序根据这些绘图状态来决定如何渲染结果。例如，当你调用设置填充颜色的函数时，你将改变存储在当前绘图状态中的颜色值。

Graphics Context包含一个绘图状态栈。当Quartz创建一个Graphics Context时，栈为空。当保存图形状态时，Quartz将当前图形状态的一个副本压入栈中。当还原图形状态时，Quartz将栈顶的图形状态出栈。出栈的状态成为当前图形状态。

可使用函数CGContextSaveGState来保存图形状态，CGContextRestoreGState来还原图形状态。
注意：并不是当前绘制环境的所有属性都是图形状态的元素。如，图形状态不包含当前路径(current path)。

#Quartz 2D 坐标系统
坐标系统定义是被绘制到Page上的对象的位置及大小范围，如图1-4所示。我们在用户空间坐标系统(user-space coordination system，简称用户空间)中指定图形的位置及大小。坐标值是用浮点数来定义的。

![](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/quartz_coordinates.gif)

由于不同的设备有不同的图形功能，所以图像的位置及大小依赖于设备。例如，一个显示设备可能每英寸只能显示少于96个像素，而打印机可能每英寸能显示300个像素。如果在设备级别上定义坐标系统，则在一个设备上绘制的图形无法在其它设备上正常显示。

Quartz通过使用当前转换矩阵(current transformation matrix， CTM)将一个独立的坐标系统(user space)映射到输出设备的坐标系统(device space)，以此来解决设备依赖问题。 CTM是一种特殊类型的矩阵(affine transform, 仿射矩阵)，通过平移(translation)、旋转(rotation)、缩放(scale)操作将点从一个坐标空间映射到另外一个坐标空间。

CTM还有另外一个目的：允许你通过转换来决定对象如何被绘制。例如，为了绘制一个旋转了45度的盒子，我们可以在绘制盒子之前旋转Page的坐标系统。Quartz使用旋转过的坐标系统来将盒子绘制到输出设备中。

用户空间的点用坐标对(x, y)来表示，(0, 0)表示坐标原点。Quartz中默认的坐标系统是：沿着x轴从左到右坐标值逐渐增大；沿着y轴从下到上坐标值逐渐增大。

有一些技术在设置它们的graphics context时使用了不同于Quartz的默认坐标系统。相对于Quartz来说，这些坐标系统是修改的坐标系统(modified coordinate system)，当在这些坐标系统中显示Quartz绘制的图形时，必须进行转换。最常见的一种修改的坐标系统是原点位于左上角，而沿着y轴从上到下坐标值逐渐增大。我们可以在如下一些地方见到这种坐标系统：

* 在Mac OS X中，重写过isFlipped方法以返回yes的NSView类的子类
* 在iOS中，由UIView返回的绘图上下文
* 在iOS中，通过调用UIGraphicsBeginImageContextWithOptions函数返回的绘图上下文

如果应用程序想以相同的绘制程序在一个UIView对象和PDF Graphics Context上进行绘制，需要做一个变换以使PDF Graphics Context使用与UIView相同的坐标系。要达到这一目的，只需要对PDF的上下文的原点做一个平移(移到左上角)和用-1对y坐标值进行缩放。图1-5显示了这种变换操作：

![](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/Art/flipped_coordinates.jpg)

我们的应用程序负责调整Quartz调用以确保有一个转换应用到上下文中。例如，如果你想要一个图片或PDF正确的绘制到一个Graphics Context中，你的应用程序可能需要临时调整Graphics Context的CTM。在iOS中，如果使用UIImage对象来包裹创建的CGImage对象，可以不需要修改CTM。UIImage将自动进行补偿以适用UIKit的坐标系统。

重要：如果你打算在iOS上开发与Quartz相关的程序，了解以上所讨论的是很有用的，但不是必须的。在iOS 3.2及后续的版本中，当UIKit为你的应用程序创建一个绘图上下文时，也对上下文进行了额外的修改以匹配UIKit的约定。特别的，patterns和shadows(不被CTM影响)单独进行调整以匹配UIKit坐标系统。在这种情况下，没有一个等价的机制让CTM来转换Quartz和UIKit的上下文。我们必须认识到在什么样的上下文中进行绘制，并调整行为以匹配上下文的预期。

#内存管理：对象所有权

Quartz使用Core Foundation内存管理模型(引用计数)。所以，对象的创建与销毁与通常的方式是一样的。在Quartz中，需要记住如下一些规则：

* 如果创建或拷贝一个对象，你将拥有它，因此你必须释放它。通常，如果使用含有”Create”或“Copy”单词的函数获取一个对象，当使用完后必须释放，否则将导致内存泄露。
* 如果使用不含有”Create”或“Copy”单词的函数获取一个对象，你将不会拥有对象的引用，不需要释放它。
* 如果你不拥有一个对象而打算保持它，则必须retain它并且在不需要时release掉。可以使用Quartz 2D的函数来指定retain和release一个对象。例如，如果创建了一个CGColorspace对象，则使用函数CGColorSpaceRetain和CGColorSpaceRelease来retain和release对象。同样，可以使用Core Foundation的CFRetain和CFRelease，但是注意不能传递NULL值给这些函数。


