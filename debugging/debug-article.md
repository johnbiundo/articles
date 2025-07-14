### Intro

Debugging NodeJS TypeScript apps is a killer feature for Visual Studio Code (VS Code). The obvious connection between Microsoft's VS Code and TypeScript teams has made VS Code the editor of choice for most TypeScript enthusiasts, and has brought rapid improvements to server side TypeScript debugging, much to the benefit of the NestJS developer.

However, this capability is so robust, and has developed so rapidly, and the possible development/deployment configurations are so numerous, that it's not always easy to find current, unambiguous documentation. Choosing the right way to set up your environment and configure debugging can be frustrating. I've spent a bunch of time looking at and experimenting with different configurations, and reading everything I could find on debugging TypeScript with VS Code. In this article, I'll map out the best, most up-to-date approach, to debugging your NestJS app.

### Configurations

- If you're developing everything locally -- meaning you fire up your Nest app on your local Windows 10 or Linux desktop (e.g., you access your server on `localhost:3000`), the news could not be better. You can literally be source code debugging in about 30 seconds. See []() below.
- If you're running VS Code on Windows 10 and running Nest on WSL ([Windows Subsystem for Linux](...)), your life is only slightly more complicated than the all-local case above. You can be source code debugging your "remote" WSL Nest app in your Windows-hosted VS Code editor in about 60 seconds. See []() below.
- If you're running VS Code on one desktop (any OS), and Nest on a local server, as I do, there are a number of choices that can bring you a productive debugging experience. I'll cover these [here]().
- If you're running VS Code on your desktop, and Nest in a Docker container, you also have a number of choices that can work well. I'll cover this [here]().
- Finally, if you want to debug a Nest app in the cloud from your desktop, I'll show you how you can do this with Azure. The principals should apply to most cloud deployment environments, and if there's a lot of interest, I'll examine some of these in a future article.

Let's get started! You may want to read the **Background** section that follows below, and possibly skim the various scenarios to see if they inspire you to try out a new development configuration, or just choose your specific scenario and jump right to it.

### Background

Until around mid-2018, setting up a debugging environment with VS Code meant building a "launch configuration". This is still a technique that gets most of the attention, and it remains useful to understand the concepts. The main concept of a launch configuration is likely familiar to back-end developers coming from other environments such as Java and .NET. But it can be a little daunting for JavaScript developers, especially those coming from the FE world. The idea is that you capture all the details needed to "launch" the server side app (i.e., your NestJS app) in debug mode, along with the details that VS Code needs to _connect_ to the server process, in one configuration file. VS Code tries to guide you through setting that file up, but there are several "modes" (launch vs. attach) and potentially dozens of variables to set up depending on your scenario. You _can_ make it work, but you may well spend a lot of time and try your patience dearly.

The good news is that unless you're doing something that's a bit out of the ordinary, you probably don't have to worry about launch configurations at all.

In 2019, the VS Code team added a bunch of really cool features to help ease configuration, among them:

1. Integrated support for NPM script browsing/launching
2. "AutoAttach"
3. Remote Development Extensions

Combined, these make debugging in almost any configuration a breeze.

#### Launch Vs Attach

Let's spend another moment on this. Here's the deal: if you launch your NestJS app manually in a terminal with something like `npm run start:debug`, you'll need to _attach_ the VS Code debugger to that running instance. Conversely, if you want a single "push button" approach, you can have VS Code _launch_ your server app for you. I'm going to generally advocate for the "manually launch, then attach" mode, so you'll see that in the recommendations below. Why? Well, I think of them as two separate steps. If you combine them, especially on Windows, you can sometimes "lose track" of a server process running in the background, forcing you to deal with nasty details like firing up the _task manager_ to track down these little escapees. If you're not careful, you may even find yourself testing against one of these runaway rascals, or get a confusing `econn...` error when iterating through code/test/debug cycles.

Plus, with _Auto Attach_, it is utterly painless to _attach_ to a running server.

### Scenario 1: Everything local

Say you're running Windows 10, or Ubuntu **_ with the _** desktop, on your local machine. You fire up VS Code, edit your NestJS source code on the local filesystem, run the app natively on the local machine, and fire up a browser or something like Postman to access the Nest app. Assuming you're running recent versions of VS Code and NestJS (as of this writing, the current versions are ...), the only VS Code feature you need is "Auto Attach".

#### Turn on Auto Attach

#### Use Auto Attach

example 1 - firing up from the integrated terminal
example 2 - firing up from a separate terminal window
example 3 - firing up using npm script editor

### Scenario 2: Windows 10 and WSL

If, like me, you're a big Linux fan but can't quite give up your Windows desktop, maybe you've started playing around with WSL. In principal, you could treat WSL as a "remote" (OK, really more like a "containerized") system, and run VS Code on Windows. Until recently, this was probably more trouble than it was worth. But with the advent of the Remote Development Extension, this configuration is almost completely transparent, including accessing your source code on the Linux filesystem, and debugging a Nest app running under WSL from your Windows VS Code environment.

About all you have to do is install and set up the ... Remote Dev Extension ...

#### Set up the ... Remote Dev Extension ....

#### Use R D Extension

### Scenario 3: VS Code on Desktop, NestJS running on Server

This is my preferred environment. It mostly works fine, but presents a bit more complex setup for debugging. Here's the basic principle to consider if you want to do TypeScript source code debugging: you need access to the TypeScript source code that is running on the server **on your desktop machine** and available to VS Code. You can handle this in a number of ways:

- Shared disk: this is the approach I use, wherein I share my Linux server disk using SAMBA. SAMBA uses the Windows SMB protocol to make a remote server partition look like a local Windows hard drive.
  - Note that there are other disk sharing solutions available, some of which are dependent on your desktop and server machines. For Windows/Linux, you could also consider -- ssh file system --, though I have had trouble with this
- Snapshot copies: in this approach, you literally copy files between the two machines. I haven't done this as I just don't like the idea much -- and save an occasional hiccup, SAMBA works great for me -- but setting up something like rsync could make this reasonably practical, or just using an SFTP client like [winscp]() for small projects could be workable.

Once you figure out how to have synchronized access (copy or sharing) to the source code on both machines, you then need to set up a launch configuration.

#### Create a launch configur

### Setting up debugging

Start by enabling npm script running. Click on File > Preferences > Settings, and start typing `npm: enable`. When it's displayed, check the box labeled `npm: EnablescriptExplorer`.

This feature makes it simple to launch your Nest server in debug mode.

Next, enable "Auto Attach". Press Ctrl + Shift + P to bring up the command pallet, then start typing `Debug: Toggle Auto Attach`. When displayed, click the checkbox to turn the feature on.

This feature allows VS Code to automatically detect when a server is running with the `--inspect` flag, and automatically infer the correct settings to connect to it.

### NestJS settings

If you're running a recent version of NestJS, you'll have the latest npm scripts installed. Take a look in `package.json` and note the value for the `start:debug` script. If it looks like the following, you're all set.

```json
{ "a": 1, "b": 2 }
```

If not, you can safely upgrade the npm scripts without breaking compatibility, which you'll probably want to do. The best way to do this is probably to upgrade your nest installation (do it locally in a test project, with `npx`, if you are worried about compatibility), then scaffold a new app with `nest new test-app`, and cut and paste the npm scripts into your existing project. This will give you the latest and best scripts for running and debugging Nest, but won't upgrade your core Nest libraries.
