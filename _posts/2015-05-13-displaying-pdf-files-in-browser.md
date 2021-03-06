---
layout: post
categories: [programming, C-Sharp]
tags: [C#]
author: Simon Childs
comments: true
---

Before I start, I should mention straight away that the solution I propose here is **not free**. This post is mostly intended for those of you lucky enough to work in a company that is willing and able to spend cash on tech.

Displaying PDF files in a browser, as if they are a part of the application or website you're building, has long been a tricky thing to do especially if you take into account cross browser compatibility. The first thing you might think of doing is sticking  
`<iframe href="/some.pdf">` into your page and forgetting about it, right? Well, no. Not really.

The PDF will render differently in every browser, and if you don't have [Adobe Acrobat Reader](https://acrobat.adobe.com/uk/en/products/pdf-reader.html) installed, you won't see anything. Or worse still, you'll see some Adobe reader error.

This can be problematic when your design team have produced a beautiful layout allowing users to upload PDF files and have a preview of their uploaded file displayed to them in the browser, with beautiful sticker type styles laid over the file and no controls what so ever. Not such an easy task now.

This is where [Apitron PDF Rasterizer](http://apitron.com/Product/pdf-rasterizer) comes in. As I said at the begining of this post, this is not a free option - in fact, it's a rather expensive one if you don't have a company to pay for it. It is, however, a fantasic piece of software. You can use the software for free straight away, you'll just get a corner to corner water mark on each image generated informing you it's a trial.

The approach I will describe here involves generating several images from each of the pages of the given PDF file. For this, I will keep it simple and use a standard .NET Console Application to go through it. OK, let's get stuck in.

First of all, you will need to install the Apitron PDF Rasterizer to your project. This is made really easy thanks to a [NuGet](https://www.nuget.org/packages/Apitron.PDF.Rasterizer/) package.

`PM> Install-Package Apitron.PDF.Rasterizer`

Next, you will need to import the Apitron namespaces:

{% highlight C# %}
using Apitron.PDF.Rasterizer;
using Apitron.PDF.Rasterizer.Configuration;
{% endhighlight %}

And create images from a PDF:

{% highlight C# %}
static void Main(string[] args)
{
    const float res = 150f;

    var index = 0;

    var file = new FileStream("C:\\files\\document.pdf", FileMode.Open);

    var document = new Document(file);

    var resolution = new Resolution(res, res);

    var settings = new RenderingSettings { ScaleMode = ScaleMode.PreserveAspectRatio };

    foreach (var page in document.Pages)
    {
        var img = page.Render(resolution, settings);

        var pageName = string.Format("C:\\files\\document_page{0}.jpg", ++index);

        img.Save(pageName, ImageFormat.Jpeg);

        img.Dispose();
    }

    document.Dispose();

    file.Close();

    file.Dispose();
}
{% endhighlight %}

And that's all there is to it! You can see from the code example above that the technique used here follows a few simple steps. These are:

* Load the PDF document from disk and store as a `FileStream`
* Create a new Apitron Document from the file stream - this will give you access to a list of pages from the PDF, as well as other helpful data and meta data
* Iterate through each page, render the page as a `Bitmap` using the given resolution and scale settings and save the newly generated image with the JPG format. You can, of course, use any image format (such as PNG) you like.

With the images saved, you can now display them in a web page simply by showing the images in an ordered list. For example:  

{% highlight html %}

<ul>
    <li>
        <img src="/public/files/document_page1.jpg" />
    <li>
    <li>
        <img src="/public/files/document_page2.jpg" />
    <li>
    <li>
        <img src="/public/files/document_page3.jpg" />
    <li>
    <li>
        <img src="/public/files/document_page4.jpg" />
    <li>
    <!-- etc... -->
</ul>

{% endhighlight %}
