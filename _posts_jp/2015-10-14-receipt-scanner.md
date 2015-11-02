Text extraction from image becomes more popular these days. 最近では画像からの文章抽出は日常生活の中で身近な物となっています。For example text translation from images, receipt scanner, paper scanner, etc.例えば、画像、レシートスキャン、ペーパースキャンなどがあります。
But the question is, how is it done? しかし、これはどのようにおこなわれているのでしょうか？First, to recognize what texts inside the image we need to know where the texts are located and that's our topic today.まずは、画像の中にどんな文章があるのかを認識するために、どこに文章があるのかを認識することが今日のトピックです。

My approaches to detect where the texts are: 私が考える「文章がどこにあるか」を検出するアプローチは：

1. Change color space from BGR (or RGB) to Gray  BGR(RGB)からグレーへ色を変える
2. Apply Morphological Gradient 形態的勾配を適用する 
3. Apply threshold to convert to binary image バイナリーイメージに変換するため「しきい値」を適用する　
4. Apply Closing Morphological Transformation 形態学的なクロージング変換を適用する 
5. Find contours and filter it 輪郭を検出しフィルターする 

Let's take a look at each step in detail.では、それぞれのステップを詳細に見ていきましょう。

###Original Image　元画像
<img src="/assets/images/posts/text_detection/original.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Change color space from BGR to Gray.  色をBGRからグレーに変える
In image processing it's pretty common to change the color space from BGR (or RGB) to Gray. One of the reason is luminance is by far more important in distinguishing visual features compared to chrominance ([Good explanation by John Zhang][1]).  画像処理において色をBGRからグレーに変換することは一般的です。一つの理由として、輝度はクロミナンスと比較して視覚特徴を区別する際にはるかに重要だからです。
To convert image from BGR (or RGB) to Gray color space in OpenCV is quite simple.  OpenCVでBGR(もしくはRGB)からグレーへイメージを変換することはとてもシンプルです。
{% highlight cpp %}
Mat original = imread("receipt.jpg");
Mat gray;
cvtColor(original, gray, CV_BGR2GRAY);
{% endhighlight %}  

Result :  
<img src="/assets/images/posts/text_detection/gray.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Apply Morphological Gradient.  形態的勾配を適用する
Morphological Gradient is a combination from dilation and erosion, which dilation will act as local maximum operator and erosion as local minimum operator.形態的勾配とは収縮(Erosion)と膨張(dilation)から成るもので、dilationはlocal maximum operatorとして、erosionはlocal minimum operatorとして機能します。
{% highlight cpp %}
Mat gradient;
Mat morphStructure = getStructuringElement(MORPH_ELLIPSE, Size(3, 3));
morphologyEx(gray, gradient, MORPH_GRADIENT, morphStructure);
{% endhighlight %}  
Which morphStructure is kernel of structuring element that will define how the gradient will be calculated. morphStructureは要素を構成する核でどのように勾配を計算するのかを定義します。 
To understand more about dilaton and erosion you can check really nice demonstration in OpenCV's example [here][2]dilationとerosionをもっと理解するためにはOpenCVの例の中にあるとても良いデモンストレーションを見て頂きましょう。

Result :  
<img src="/assets/images/posts/text_detection/gradient.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Apply threshold to convert to binary image　バイナリーイメージに変換するため「しきい値」を適用する
I'm using Otsu algorithm to choose the optimal threshold value to convert the processed image to binary image.
{% highlight cpp %}
Mat binary;
threshold(gradient, binary, 0.0, 255.0, THRESH_BINARY | THRESH_OTSU);
{% endhighlight %}  

Result :  
<img src="/assets/images/posts/text_detection/binary.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Apply Closing Morphological Transformation  
And the last one before we can find the contours we need to do Closing Morphological Transformation which is dilation followed by erosion that will close small holes between words so we can group it into a sentence horizontally.
{% highlight cpp %}
Mat closed;
morphStructure = getStructuringElement(MORPH_RECT, Size(15, 1));
morphologyEx(binary, closed, MORPH_CLOSE, morphStructure);
{% endhighlight %}  

Result :  
<img src="/assets/images/posts/text_detection/closed.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Find contours and filter it  
After all the processing it's time to find contours and filter it so we finally can locate where the texts are located.
{% highlight cpp %}
// Find contours
Mat mask = Mat::zeros(bw.size(), CV_8UC1);
vector<vector<Point>> contours;
vector<Vec4i> hierarchy;
findContours(connected, contours, hierarchy, CV_RETR_CCOMP, CV_CHAIN_APPROX_SIMPLE, Point(0, 0));
// Filter contours
for(int i = 0; i >= 0; i = hierarchy[i][0])
{
    Rect rect = boundingRect(contours[i]);
    // Ignore if the rect is too small
    if ((rect.height < 10 || rect.width < 10)) {
        continue;
    }
    Mat maskROI(mask, rect);
    maskROI = Scalar(0, 0, 0);
    drawContours(mask, contours, i, Scalar(255, 255, 255), CV_FILLED);
    // Calculate ratio of non-zero pixels in the filled region
    double r = (double)countNonZero(maskROI)/(rect.width*rect.height);

    // If the ration is bigger than 45% we assume it contains texts
    if (r > 0.45)
    {
        rectangle(original, rect, Scalar(0, 255, 0), 2);
    }
}
{% endhighlight %}  

Result :  
<img src="/assets/images/posts/text_detection/result.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

That's all! I hope my post can help you to understand how to detect texts inside an image.  
For recognizing the texts I will write a post about it in the future.  
If you have any question, <a href="https://twitter.com/intent/tweet?screen_name=niko_yuwono">send me a tweet</a> or leave a comment below! If you like this post you can share it with your friends using share buttons below.

  
[1]: https://www.quora.com/Computer-Vision/If-you-had-to-choose-would-you-rather-go-without-luminance-or-chrominance/answer/John-Zhang
[2]: http://docs.opencv.org/master/d9/d61/tutorial_py_morphological_ops.html#gsc.tab=0