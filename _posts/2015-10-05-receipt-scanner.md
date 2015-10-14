---
layout: post
title: Receipt Scanner (Text Detection)
image: /assets/text_detection/result.jpg
---


Text extraction from image becomes more popular these days. For example text translation from images, receipt scanner, paper scanner, etc.  
But the question is, how is it done? First, to recognize what texts inside the image we need to know where the texts are **located** and that's our topic today.

My approaches to detect where the texts are:  

1. Change color space from BGR (or RGB) to Gray  
2. Apply Morphological Gradient  
3. Apply threshold to convert to binary image
4. Apply Closing Morphological Transformation  
5. Find contours and filter it  

Let's take a look at each step in detail.

###Original Image
<img src="/assets/text_detection/original.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Change color space from BGR to Gray.  
In image processing it's pretty common to change the color space from BGR (or RGB) to Gray. One of the reason is luminance is by far more important in distinguishing visual features compared to chrominance ([Good explanation by John Zhang][1]).  
To convert image from BGR (or RGB) to Gray color space in OpenCV is quite simple.  
{% highlight cpp %}
Mat original = imread("receipt.jpg");
Mat gray;
cvtColor(original, gray, CV_BGR2GRAY);
{% endhighlight %}  

Result :  
<img src="/assets/text_detection/gray.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Apply Morphological Gradient.  
Morphological Gradient is a combination from dilation and erosion. Where dilation will act as local maximum operator and erosion as local minimum operator.
{% highlight cpp %}
Mat gradient;
Mat morphStructure = getStructuringElement(MORPH_ELLIPSE, Size(3, 3));
morphologyEx(gray, gradient, MORPH_GRADIENT, morphStructure);
{% endhighlight %}  
Which morphStructure is kernel of structuring element that will define how the gradient will be calculated.  
To understand more about dilaton and erosion you can check really nice demonstration in OpenCV's example [here][2]

Result :  
<img src="/assets/text_detection/gradient.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Apply threshold to convert to binary image
I'm using Otsu algorithm to choose the optimal threshold value to convert the processed image to binary image.
{% highlight cpp %}
Mat binary;
threshold(gradient, binary, 0.0, 255.0, THRESH_BINARY | THRESH_OTSU);
{% endhighlight %}  

Result :  
<img src="/assets/text_detection/binary.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

###Apply Closing Morphological Transformation  
And the last one before we can find the contours we need to do Closing Morphological Transformation which is dilation followed by erosion that will close small holes between words so we can group it into a sentence horizontally.
{% highlight cpp %}
Mat closed;
morphStructure = getStructuringElement(MORPH_RECT, Size(15, 1));
morphologyEx(binary, closed, MORPH_CLOSE, morphStructure);
{% endhighlight %}  

Result :  
<img src="/assets/text_detection/closed.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

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
<img src="/assets/text_detection/result.jpg" height="75%" width="75%" style="margin-left=auto;margin-right-auto;">

That's all! I hope my post can help you to understand how to detect texts inside an image.  
For recognizing the texts I will write a post about it in the future.  
If you have any question, <a href="https://twitter.com/intent/tweet?screen_name=niko_yuwono">send me a tweet!</a> If you like this post you can share it with your friends using share buttons below.

  
[1]: https://www.quora.com/Computer-Vision/If-you-had-to-choose-would-you-rather-go-without-luminance-or-chrominance/answer/John-Zhang
[2]: http://docs.opencv.org/master/d9/d61/tutorial_py_morphological_ops.html#gsc.tab=0