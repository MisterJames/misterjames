title: How to Resize Images Uploaded to Blob Storage with Azure Functions
layout: post
tags:
  - Azure Functions
  - c#
  - 5 days of Azure Functions
categories:
  - Development
authorId: james_chambers
featureImage: https://jcblogimages.blob.core.windows.net:443/img/2016/11/resize-pictures.png
originalUrl: 'http://jameschambers.com/2016/11/Resizing-Images-Using-Azure-Functions/'
date: 2016-11-15 7:04:26
---
Here's the scenario: you have some block of processing that needs to be executed everytime a file is pushed up into your blob storage account.  The solution: use Azure Functions and the integration module for blob-triggers so that you don't have to do any of the heavy lifting.

![](https://jcblogimages.blob.core.windows.net:443/img/2016/11/resize-pictures.png)

In this post we'll look at using a storage account trigger to automatically have an image processed as part of an Azure Function App. Not just to be used for image processing, any type of object can trigger a block of work and it will follow these same mechanics.

<!-- more -->

{% include_markdown _includes/Five-Days-Overview %}

## Start By Creating Your Trigger
The type of Function I'm creating here is based off of the `BlobTrigger-CSharp` template. The interface allows you to create the function and select your storage account settings, including the path to the source of the images. You can trigger Azure Functions from an event in one of your existing storage accounts, or you can use the Azure Functions interface to create a new storage account as I'm doing here when I create my function. 

![](https://jcblogimages.blob.core.windows.net:443/img/2016/11/create-trigger-function.png)

The portal will create a binding for your script that will allow you to process files created at the path specified. The connection string for your storage account will automatically be created and added to your Function App. **There is no magic here.** To see how this is wired up, inspect the `function.json` file in your function through the `View Files` tool pane.

One important note is that the parameter `{imageName}` in the path is what you will need to name the parameter in your method signature. We'll come back to that, but first, we need to add another parameter binding, this time for output, so that we can save out the resized image back to Azure Blob Storage. 

Click on the Integrate menu and add a new Output binding to the list. The type you want to select here is Azure Blob Storage output. Basically, we're taking one blob and saving it out as another blob.

![Settings for the Output of the Script](https://jcblogimages.blob.core.windows.net:443/img/2016/11/function-output-blob.png)

I've selected the same connection string that I created in the first step. We're limited in that we must use a storage account from the same region as our Function App. Note the blob parameter name because we're going to be seeing it as the name of the output stream in our function. Also note the path, where I'm reusing the `{imageName}` parameter. This was the name of the file coming into our function, and we'll save it out as the same filename but to a different path.

Next, use the `View Files` button to reveal your app's assets and then add a new file called `project.json`. This is the current mechanism that Function Apps use to restore packages from NuGet (which we'll cover in a future post).

``` javascript
{
"frameworks": {
  "net46":{
    "dependencies": {
      "ImageResizer": "4.0.4"
    }
  }
 }
}
```

Save the JSON above into your `project.json`, then navigate to `run.csx`, the script file for your endpoint. 

## Piecing it All Together
This is what the code should look like in your `run.csx` file when you're done with it. I'm using the awesome `ImageResizer` library to execute a resize operation with one stream as the source and the other as the output. Here's where all those parameter names come back into play:

``` csharp
#r "System.Drawing"

using ImageResizer;
using System.Drawing;
using System.Drawing.Imaging;

public static void Run(Stream inputImage, string imageName, Stream resizedImage, TraceWriter log)
{
    log.Info($"C# Blob trigger function Processed blob\n Name:{imageName} \n Size: {inputImage.Length} Bytes");

    var settings = new ImageResizer.ResizeSettings{
        MaxWidth = 400,
        Format = "png"
    };

    ImageResizer.ImageBuilder.Current.Build(inputImage, resizedImage, settings);

}
```

The input blob or stream called `inputImage`, the name of the image itself `imageName` which we extract from the path, and the output stream called `resizedImage`. We also get a bonus parameter in there of type `TraceWriter` which is provided to us by the runtime to facilitate logging.

## Testing it Out
Redgate makes this great tool called [Azure Explorer](http://www.red-gate.com/products/azure-development/azure-explorer/) that you should grab. It makes working with Azure Storage much easier. Sign in to the tool, add your storage account to the configuration and you should be off to the races. 

If you created the storage account through the Azure Functions App wizard (as I did above), remember that there is no magic here! This is a slice of a App Service, just the parts needed to execute code built on the Web Jobs SDK. This means that you can use the Azure portal to navigate to the Application Settings as you would for any other app by clicking on the `Function app settings` and drilling into the Application Settings from that page. Note that this is all just Azure, so you could also filter by your storage account name from the portal and drill in to find your connection string. 

## Next Steps

This technique is by no means restricted to resizing images. There are a whole host of other event types that you can elect to leverage as a trigger for your Function App including:
 - Service Bus Queues
 - Service Bus Topics
 - Event Hub
 - Storage Queues
 - Time-based Intervals

Give one of those a shot!

## Other articles in this series
{% include_markdown _includes/Five-Days-Menu %}

Happy coding!  

