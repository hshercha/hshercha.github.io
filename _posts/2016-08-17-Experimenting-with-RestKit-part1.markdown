---
title:  Experimenting with RestKit (Part 1)
date:   2016-08-18 22:02:00
description: Scratching the surface
---

As I am growing as a Software Engineer, I am understanding the value of frameworks. Striking the right balance between using and avoiding it can be difficult. Frameworks can help developers bypass potential overhead that incurs when writing boilerplate code. On the otherhand, frameworks are usually tied to dependencies. Minor issues in those dependencies could propogate and therefore have a snowball effect on the rest of the software. With that being said, I approach frameworks with just as much excitement as I approach it with caution.

Recently, I started to experiment with RestKit on iOS. To understand RestKit, one has to understand REST.  

## What is REST?
REST (Representational State Transfer) is an architecture style upon which maintainable and scalable web services(RESTful services) are built. REST has become one of the most important frameworks for designing web applications. RESTful services are language(Java, Python, etc) and platform(Unix, Mac, etc) independent. Much like other web services, RESTful services rely on HTTP protocols to perform CRUD (Create, Read, Update and Delete) operations.

## What is RestKit?
RestKit is a framework for Objective-C that allows for seamless integration with iOS and RESTful web services. Its object-mapping technology provides mapping data responses from web directly into user-defined objects or core data. One of its core dependencies is AFNetworking, so all mapping of HTTP requests and responses are performed on top of AFNetworking. To understand more about the RestKit framework, check out the [RestKit Github Project](https://github.com/RestKit/RestKit).

## How to integrate RestKit into your iOS project?
RestKit can be integrated via cocoapods. You can also pull it directly into your project from their repository. The cocoapods options is less of a hassle in comparison. In case you don't know what cocoapods are, check out [RayWinderlich.com](https://www.raywenderlich.com/97014).They have amazing resources on iOS related topics. In fact, I followed their post on [RestKit](https://www.raywenderlich.com/58682/introduction-restkit-tutorial) to attain a deeper understanding.

Open up a terminal and type the following
{% highlight bash %}
$ sudo gem install cocoapods
$ pod setup
$ cd [Project Directory]
$ touch Podfile
{% endhighlight %}

Copy and paste the following in your Podfile.
{% highlight bash %}
target "YOUR PROJECT" do
    platform :ios, '7.0'
    # Or platform :osx, '10.7'
    pod 'RestKit', '~> 0.24.0'
end
{% endhighlight %}

As you can see from the code above that dependencies for varying target files can be managed effortlessly.

1. Open the terminal > Navigate to your project directory > Type "pod install".
2. Sit there and watch the wizardy unfold! 
3. Once the pod is successfully installed, open .xcworkspace file instead of the .xcodeproj file.
4. Open AppDelegate.m file.
5. Add the following code and build the project.

{% highlight C %}
#import <RestKit/RestKit.h>
{% endhighlight %}

---
**ISSUE #1**
{% highlight bash %}
TypeError - Unable to convert Ruby value `"AFNetworking"' into a CFTypeRef.
{% endhighlight %}

I ran into this issue while integrating RestKit via the podfile. This issue pertains to the current Ruby version in your computer. Installing any version of Ruby >= 2.2.3 should fix it.

**ISSUE #2**
{% highlight bash %}
"RKObjectMapping.h" file not found
{% endhighlight %}

This compile error that occured while building the project. Downgrading your cocoapods is an option. However, there is an easier work around for this issue. 

1. Select the project file.
2. Go to Build Settings > Select "Header Search Paths".
3. Set all the paths to be recursive.

{% highlight bash %}
${PODS_ROOT}/Headers/Public
${PODS_ROOT}/Headers/Public/AFNetworking
${PODS_ROOT}/Headers/Public/Bolts
${PODS_ROOT}/Headers/Public/RKValueTransformers
${PODS_ROOT}/Headers/Public/RestKit 
${PODS_ROOT}/Headers/Public/SOCKit
${PODS_ROOT}/Headers/Public/ISO8601DateFormatterValueTransformer
{% endhighlight %}

---
Now that we have configured RestKit, my next post will discuss extracting data from the 
[Unofficial Airbnb API](http://airbnbapi.org/) 
