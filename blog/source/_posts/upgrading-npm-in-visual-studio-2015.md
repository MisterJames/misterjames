title: Upgrading NPM in Visual Studio 2015
tags:
  - node
  - npm
  - Visual Studio 2015
id: 7621
categories:
  - Code Dive
date: 2015-09-02 03:15:17
---

Visual Studio 2015 (<u>[download here](https://www.visualstudio.com/?Wt.mc_id=DX_MVP4038205)</u>) ships with it’s own version of several external tools, such as grunt, node and npm.&nbsp; If you are wanting to take advantage of newer versions of these tools, you have three options:

1.  Wait for VS 2015 to upgrade the tooling and ship an update.
2.  Hack the tooling proxies used by Visual Studio.
3.  Use the built-in external tool path editor to slip your updated versions in.

Waiting for updates is no fun. Let’s hack a little.

## Wait a minute! Why are we doing this?

[![image](http://jameschambers.com/wp-content/uploads/2015/09/image_thumb.png "image")](http://jameschambers.com/wp-content/uploads/2015/09/image.png)For me the primary motivator was the path length limitations in Windows. Nested node_modules folders buried 19 levels deep is no fun when you hit the max path length. For me, I was trying to share the files on OneDrive and hit 255 characters pretty quickly.

Older versions of npm resolved package dependencies by pulling in a package, creating a node_modules folder inside of it, then putting all the packages in there. Except, of course, if one of those packages contained more dependencies, then we were into the recursive bits of package resolution and very deep paths, ultimately toppling a lot of Windows tooling.

The latest major version of npm – version 3.0.x and above – creates a flat store of packages (very similar to what we know in NuGet) and only pulls one copy of each required version of each required package. Much nicer. So, back to the dicing!

## Hacking up the VS Tooling Proxies

These are pretty straightforward, once you find them. For me, they were located in the following directory:

C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\Extensions\Microsoft\Web Tools\External

For example, here the entire contents of npm.cmd:
<pre class="csharpcode"><span class="str">@"%~dp0\node\node"</span> <span class="str">"%~dp0\npm\node_modules\npm\bin\npm-cli.js"</span> %*</pre><style type="text/css">.csharpcode, .csharpcode pre
{
	font-size: small;
	color: black;
	font-family: consolas, "Courier New", courier, monospace;
	background-color: #ffffff;
	/*white-space: pre;*/
}
.csharpcode pre { margin: 0em; }
.csharpcode .rem { color: #008000; }
.csharpcode .kwrd { color: #0000ff; }
.csharpcode .str { color: #006080; }
.csharpcode .op { color: #0000c0; }
.csharpcode .preproc { color: #cc6633; }
.csharpcode .asp { background-color: #ffff00; }
.csharpcode .html { color: #800000; }
.csharpcode .attr { color: #ff0000; }
.csharpcode .alt 
{
	background-color: #f4f4f4;
	width: 100%;
	margin: 0em;
}
.csharpcode .lnum { color: #606060; }
</style>

The %~dp0 is the old command line way of bringing the current drive letter (the d in the command), the path (the letter p here) and the current directory of the executing script (represented by 0) into context. So, basically, “start from where you’re running”. It’s a very hard-to-read version of “.” in most other notations. So, the command is running node (which is an exe), passing in the VS version of npm, and pushing into it the rest of the parameters that were passed along. So, when VS issues an “npm install”, this command kicks in, runs npm via node and passes “install” as the command to npm. 

With that knowledge, we can simply update the call that is proxied through to our current version. I installed node (which includes npm), then updated npm to the latest version (thanks to [<u>this module</u>](https://www.npmjs.com/package/npm-windows-upgrade)) and updated my npm.cmd to the following:
<pre class="csharpcode"><span class="str">@"C:\Program Files (x86)\nodejs\node.exe"</span> <span class="str">"C:\Program Files (x86)\nodejs\node_modules\npm\bin\npm-cli.js"</span> %*</pre><style type="text/css">.csharpcode, .csharpcode pre
{
	font-size: small;
	color: black;
	font-family: consolas, "Courier New", courier, monospace;
	background-color: #ffffff;
	/*white-space: pre;*/
}
.csharpcode pre { margin: 0em; }
.csharpcode .rem { color: #008000; }
.csharpcode .kwrd { color: #0000ff; }
.csharpcode .str { color: #006080; }
.csharpcode .op { color: #0000c0; }
.csharpcode .preproc { color: #cc6633; }
.csharpcode .asp { background-color: #ffff00; }
.csharpcode .html { color: #800000; }
.csharpcode .attr { color: #ff0000; }
.csharpcode .alt 
{
	background-color: #f4f4f4;
	width: 100%;
	margin: 0em;
}
.csharpcode .lnum { color: #606060; }
</style>

Of course, here be dragons: I have no idea how stable this will be with updates to VS, and/or how badly you may be crippling features if you mess this up. So, make sure you take a backup of your scripts before modifying them.&nbsp; This will be super-handy if you have some other requirement – like the order of params on tooling changes – but otherwise likely isn’t needed. Thankfully, there is a UI-way of doing this, too.

## Not Hacking Your Visual Studio Tooling

Probably a more pleasing solution for your boss.

This one is pretty straightforward as well, and can be done by right-clicking on the “Dependencies” node in Solution Explorer, or by typing “external web tools” in the QuickLaunch bar. 

From here, just add a new entry and move it to the top. For me, npm is located in the nodejs install directory, and this is good enough to get VS to see it first.

[![image](http://jameschambers.com/wp-content/uploads/2015/09/image_thumb1.png "image")](http://jameschambers.com/wp-content/uploads/2015/09/image1.png)

Note, I did seem to have some issues with caching and/or gremlins here, so you may need to restart Visual Studio for the tooling paths to be picked up.

## What I Don’t Like

Couple of things here that I don’t care for:

1.  **Not consistent between team members**: there seems to be no way to put in your solution/project a hint at the version of tooling you wish to use. In my case, a developer with npm 2 trying to run an install off of OneDrive would fail.
2.  **Visual Studio external tools are internal**: yeah, you read that right. I’m not a fan of the way these projects are packed in, in such a way that the path to update them or pick versions is non-obvious.

## Next Steps

Not too much to do, but if you run into long paths, nested node_modules kicking your butt or other out-of-date tooling, this should get you on your way.

Make sure you grab your <u>[copy of VS 2015](https://www.visualstudio.com/?Wt.mc_id=DX_MVP4038205)</u> and start diving into the next phase of our careers!

Happy coding! ![Smile](http://jameschambers.com/wp-content/uploads/2015/09/wlEmoticon-smile.png)