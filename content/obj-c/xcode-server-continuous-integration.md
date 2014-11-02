Title: Set up XCode for continuous integration using OSX Server
Date: 2014-11-01 10:20
Category: obj-c
Tags: xcode, continuous integration, testing
Slug: xcode-server-continuous-integration
Authors: jandro_es
Summary: Setting up XCode for continuous integration using OSX Server.

Since last year, Apple made available to all developers (with a paid membership) their OSX Server for free, among other functionalities, the one which concerns us now is to use it for Continuous Integration in our XCode's applications (iOS, OSX, Objective-C or Swift).

###Downloading OSX Server with redeem code.

Usually **OSX Server** has a prize tag of around 17€, but if you have a paid developer account (for iOS or OSX) you can get it for free. First you need to login in your developer account and access to the [download section](https://developer.apple.com/devcenter/ios/index.action). Almost at the bottom of the page you have the link to get the download code for **OS X Server 4.0**.

![OS X Server Download link](https://www.diigo.com/file/image/reeqooozcdpsbqadozboqppaes/OSX+Server+Download+link.jpg)

Once you click the link, it will give you a *redeem code* and open the App Store application so you can enter it. Afterwards it will start downloading OSX Server and the installation process will start automatically.

![OS X Server 4.0 installation screen](https://www.diigo.com/file/image/reeqooozcdpsbqrqazboqppbdr/Installation.jpg)

Following a simple installation process you should end with a screen similar to this one:

![OS X Server](https://www.diigo.com/file/image/reeqooozcdpsbredazboqppbqp/Server.jpg)

Your OS X Server is already up and running.

###Set up XCode Server.

Now we need to set up our OS X Server to function as a XCode Server. In the left panel, we click over the item labeled **xcode**. The first time we install the server, it will ask for the Xcode installation we want to use.

![XCode installation](https://www.diigo.com/file/image/reeqooozcdpsbrsqqzboqppcee/XCode+Installation.jpg)

We select our XCode installation (XCode 6 or superior) and we wait for the configuration to complete. Once configured we'll see a screen like this:

![XCode Server running](https://www.diigo.com/file/image/reeqooozcdpseasrczboqpqrpa/running.jpg)

We start/stop the XCode Server using the switch in the top left corner. To allow the server to access the profiles, provisioning files and certificates of our development team, we need to add at least one development team (the same server can be in several teams). We click the **add** button and we enter our credentials (we need to be admin or agent inside the team).

![Credentials](https://www.diigo.com/file/image/reeqooozcdpsebdodzboqpqrro/credentials.jpg)

Now the server will register itself on the teams we selected, it'll take a while. Once finished we can create bots for any of our projects.

###Creating bots.

We open our project in XCode and in the menu *product* we select *Create bot...*

![Bot creation](https://www.diigo.com/file/image/reeqooozcdpsebssazboqpqscs/bot1.jpg)

enter the name of the bot, and the first time we need to add a server, we select the new server we've just configured. It'll ask for your credentials (the same as you computer by default, if you have any problem you can create an specific user inside the server control panel).

Now we configure the **bot** schedule, we can schedule it every hour, day, week, etc.. Usually the *on commit* option is the one selected, meaning that every time a team member commit some changes to the repository, the bot will run.

![Bot schedule](https://www.diigo.com/file/image/reeqooozcdpsecosezboqpqsop/Bot2.jpg)

Next step is to configure the execution environment, we can select to run only in simulators, in all of them, or only on some of them (depending in the project compatibility requirements) and if we have some devices provisioned and connected to the server, we can also schedule the execution in them:

![Bot execution environment](https://www.diigo.com/file/image/reeqooozcdpsecqdazboqpqspq/bot3.jpg)

Now we configure the bot's **triggers**, we can schedule actions to be executed before the integration, as well as afterwards. Also we can configure who receive notifications if an integration fails.

![Bot triggers](https://www.diigo.com/file/image/reeqooozcdpsecsdazboqpqsrc/bot4.jpg)

And that's all, XCode will ask us to submit the changes to the repository, and once updated, it will run the first integration and show us the results:

![Test running](https://www.diigo.com/file/image/reeqooozcdpsedbbrzboqpqsrs/tests.jpg)

You already have a **Continuous Integration Server** operational.

