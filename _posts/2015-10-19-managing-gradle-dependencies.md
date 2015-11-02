---
layout: post
title: Managing Gradle Dependencies
image: /assets/images/posts/gradle_dependencies/dependencies_gradle.png
---


Lately, I'm wondering how can I make my gradle build files better. Then I found a way to manage my gradle dependencies version separately from project gradle files that will be really helpful if you separate your project into several parts, for example MVP design pattern.

To achieve this you need to create a new gradle file, in my case I will name it dependencies and I will put it in buildconfig folder.

<img src="/assets/images/posts/gradle_dependencies/dependencies_gradle.png" style="margin-left=auto;margin-right-auto;">

And manage your dependencies in that file

{% highlight java linenos %}
allprojects {
  repositories {
    jcenter()
  }
}

ext {
  //Libraries
  androidSupportLibraryVersion = '23.0.1'
  butterKnifeVersion = '7.0.1'
  rxJavaVersion = '1.0.14'
  rxAndroidVersion = '1.0.1'

  //Test
  espressoVersion = '2.2.1'
  androidTestSupportLibraryVersion = '0.1'

  uiDependencies = [
      androidSupportLibraryV4:     "com.android.support:support-v4:${androidSupportLibraryVersion}",
      androidSupportAppcompatV7:   "com.android.support:appcompat-v7:${androidSupportLibraryVersion}",
      recyclerView:                "com.android.support:recyclerview-v7:${androidSupportLibraryVersion}",
      androidSupportDesign:        "com.android.support:design:${androidSupportLibraryVersion}",
      butterKnife:                 "com.jakewharton:butterknife:${butterKnifeVersion}",
      rxJava:                      "io.reactivex:rxjava:${rxJavaVersion}",
      rxAndroid:                   "io.reactivex:rxandroid:${rxAndroidVersion}",
  ]

  uiTestDependencies = [
      espresso:                    "com.android.support.test.espresso:espresso-core:${espressoVersion}",
      androidTestRunner:           "com.android.support.test:runner:${androidTestSupportLibraryVersion}",
      androidTestRules:            "com.android.support.test:rules:${androidTestSupportLibraryVersion}",
  ]
}
{% endhighlight %} 

And then to integrate this to your build files you need to include this line to your build.gradle file (The top level one, not the project build gradle)

{% highlight java %}
apply from: 'buildconfig/dependencies.gradle'
{% endhighlight %}

And then you can use it in your project build gradle like this

{% highlight java linenos %}
dependencies {
  def uiDependencies = rootProject.ext.uiDependencies
  def uiTestDependencies = rootProject.ext.uiTestDependencies

  compile uiDependencies.androidSupportLibraryV4
  compile uiDependencies.androidSupportAppcompatV7
  compile uiDependencies.recyclerView
  compile uiDependencies.androidSupportDesign
  compile uiDependencies.butterKnife
  compile uiDependencies.rxJava
  compile uiDependencies.rxAndroid

  compile uiTestDependencies.espresso
  compile uiTestDependencies.androidTestRunner
  compile uiTestDependencies.androidTestRules
}
{% endhighlight %}


That's all, I hope my post can help you!
If you have any question, <a href="https://twitter.com/intent/tweet?screen_name=niko_yuwono">send me a tweet</a> or leave a comment below! If you like this post you can share it with your friends using share buttons below.