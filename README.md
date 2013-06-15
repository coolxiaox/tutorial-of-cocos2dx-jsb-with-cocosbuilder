Cocos2d-x JavaScript Binding结合CocosBuilder跨平台游戏开发简明教程
==========================================

##作者：[刘松](http://weibo.com/loosen)  [转载务必注明作者和原文链接]

##前言


由国人自己研发的开源游戏引擎Cocos2d-x，在很短的时间内就风靡全球，来自多个国家的开发者均参与贡献代码，其优秀毋庸置疑。但是，年轻的Cocos2d-x正处于快速发展阶段，围绕其周边的配套工具、文档、教程等等均尚不完善。我本人在学习的过程中就体会过把Google、StackOverflow、github以及官方论坛等等翻个底朝天都找不到相关资料的窘境。这也是本篇教程诞生的根本原因。

因我本人亦是刚刚加入Cocos2d阵营学习不久，加上自身能力有限，教程之中若有错误，还望不吝指出，我将及时更新修复错误。另外，本人热爱开源，因此本教程亦设定为“`开源教程`”。希望能借此抛砖引玉，让更多的朋友参与到这个教程的编写过程中来，共同学习和完善相关内容，让更多人后来者受益。

另外，本教程中多屏幕适配相关的知识曾得到[杨世玲](http://young40.github.io/)老兄的大力指导，在此深表感谢:)

##一、Cocos2d-x JavaScript Binding简明介绍

众所周知，Cocos2d-x的主要开发语言是C++，而Cocos2d-x JavaScript Binding则是基于Cocos2d-x用JavaScript语法来编写游戏。Cocos2d-x JavaScript Binding又常常被简称为JSB，其实现原理可简单地总结为：Cocos2d-x内置了一套JavaScript的解析引擎SpiderMonkey，通过SpiderMonkey在引擎内部将JavaScript代码“映射”为C++代码，从而实现了用JavaScript语法调用Cocos2d-x的API来完成游戏的逻辑的编写。这意味着，JSB开发模式在依然保持了C++原生应用同样的性能这个优点之外，还大大降低了开发语言的门槛。

另外，由于Cocos2d家族的另一支新秀Cocos2d-HTML5也继承了Cocos2d家族相同的API结构，这就使得我们可以将由JSB编写的代码很轻松地移植到Cocos2d-HTML5，从而实现了横跨原生客户端和浏览器的全新的游戏开发模式。这一特性优势，也是目前与JSB并行存在的Cocos2d-x Lua Binding所不能比拟的。虽然相比Lua Binding而言JavaScript Binding目前还显得不太成熟，但我认为后者一定是未来。

在讲述Cocos2d-x JavaScript Binding开发实践之前，我们不得不提到CocosBuilder。相信很多人都至少听说过这款功能强大的工具，其集成了所见即所得的场景编辑器、动画编辑器、智能纹理打包以及JavaScript绑定相关配置功能，让开发工程师不再需要跟任何视觉相关的环节打交道，从而专注于游戏的逻辑控制部分，使得游戏开发变得更加简单。

下面就将结合我的实际使用经历来讲解Cocos2d-x JavaScript Binding结合CocosBuilder的跨平台游戏开发实践。


##二、开发环境配置

本节内容我大量参考和翻译了国外的一篇文章[《Setup a cross platform project with CocosBuilder and Cocos2d-x》](http://2sa-studio.blogspot.com/2013/04/setup-cross-platform-project-with.html)(须翻墙)，这篇文章里所描述的项目目录结构较为合理，我个人认为非常适合用作跨平台开发和调试的通用结构。我将这篇文章的主要内容摘录翻译下来，并结合这篇教程的需要做了一些修改，同时也方便了英文不好的朋友。

###1、在Mac终端下载Cocos2d-x最新版本并解压(此时最新版本为2.1.4)。

我更习惯用命令行，参考下面：

    $  cd #回到家目录，也可以放在别的目录
    $  wget https://cocos2d-x.googlecode.com/files/cocos2d-x-2.1.4.zip
    $  unzip cocos2d-x-2.1.4.zip

*`提示`：以上过程解压后的目录应该是cocos2d-x-2.1.4，你可能改下目录名，也可以保持不变。下面的内容将会把这个目录名用{cocos2d-x}代替。比如，我的{cocos2d-x}实际指代的是/Users/liusong/cocos2d-x*

###2、使用cocos2d-x自带的工具创建工程。

    $  cd {cocos2d-x}/tools/project-creator
    $  ./create_project.py -project MyGame -package com.MyCompany.AwesomeGame -language javascript  #创建一个名叫MyGame的跨平台工程，会在{cocos2d-x}/projects/目录下自动生成MyGame工程代码
    $  cd ../../projects/MyGame  #切换到工程目录
    $  rm -rf Resources/     #删除默认生成的资源目录，后面会用到CocosBuilder生成的资源目录

*`提示`：这里创建的工程名称MyGame会在后续教程中频繁出现，如果你不是使用这个名称，那么请注意在后面自行更改*

###3、下载、安装、配置CocosBuilder的最新版（此时最新版为3.0 alpha4）。

在这里我推荐使用Git来clone源码的方式来下载CocosBuilder，因为目前CocosBuilder尚在发展中，并不太完善和稳定，下载源代码的好处是今后遇到问题时可以自己通过追踪源代码来解决，也可以及时合并最新的补丁而不用等官方发布新版。

    $  cd #回到家目录，也可以放在别的目录
    $  git clone https://github.com/cocos2d/CocosBuilder.git
    $  cd CocosBuilder
    $  git submodule update --init --recursive
    $  open ./CocosBuilder/CocosBuilder.Xcodeproj  #打开CocosBuilder的Xcode工程

在打开的Xcode工程中，确认左上角的scheme是CocosBuilder而不是其他，直接点击“Run”，经过一段时间的编译运行后CocosBuilder将会被打开。这时候，你可以在CocosBuilder根目录下的build文件夹中看到Xcode生成的CocosBuilder.app文件。今后如果有需要启动CocosBuilder，就可以直接打开这个来启动，无须再在Xcode中编译运行，除非是你修改过CocosBuilder的源码。这个我们后面会提到。

在打开的CocosBuilder界面中新建一个CocosBuilder工程（File->New->New Project），注意这里的命名一定和之前的项目名称一样，并且要保存在工程目录。如下图所示：

![tutorial-01.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-01.png)

保存时会提示是否覆盖，请点击“Replace”确认覆盖。如下图：

![tutorial-02.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-02.png)

*`提示`：如果你操作时没有出现这个提示，那么一定是你的名称或保存目录位置有错，请删除刚才创建的内容重新操作。*

以上操作完毕后，你会看到在MyGame目录下新增了MyGame.ccbproj文件和Resources目录。MyGame.ccbproj是CocosBuilder的工程文件，而Resources则是默认的资源存放路径。还记得我们之前创建项目工程后删掉的Resources目录吗？这里又重新补上了，因为我们下来都要用CocosBuilder来组织和生成游戏资源。注意这个资源路径设置仅仅是默认的，为了方便教学我们不做修改，如果你需要修改，请打开菜单File->Project Settings进行修改。打开Resources目录可以看到，里面已经有一些文件。这是CocosBuilder在创建工程时默认为我们添加的，我们这个教程将直接使用这些默认的资源来进行演示，所以我们予以保留。如下图：

![tutorial-08.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-08.png)

接下来，点击菜单（File->Publish Settings）打卡发布设置界面，激活所有平台。默认情况下，iOS和HTML5的发布设置都是选中状态，我们只需要再勾选上中间的Android。请注意每个选项中以Published-开头的设置，那代表着你用CocosBuilder制作的游戏资源或代码将会被存储到那些对应的目录里。图示：

![tutorial-03.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-03.png)

最后点击菜单（File->Publish）进行发布，这个发布行为会自动生成上面提到的三个Published-开头的文件夹。打开看看，会发现里面有一些文件，那都是刚才由CocosBuilder自动生成的。仔细观察会发现，不同平台目录下的文件会有所差异，这就是CocosBuilder的强大之处，它可以根据你的设置来针对不同平台的特征自动处理资源兼容和适配问题。另外，原来资源目录Resources下的ccb文件均被发布成了ccbi的二进制版本，以供程序代码直接调用。

![tutorial-04.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-04.png)

###4、修改工程配置以连接CocosBuilder

####修改AppDelegate.cpp

    $  cd {cocos2d-x}/projects/MyGame #切换到项目根目录
    $  vim Classes/AppDelegate.cpp

最后，修改applicationDidFinishLaunching的内容如下：

```
bool AppDelegate::applicationDidFinishLaunching()
{
  // initialize director
	CCDirector *pDirector = CCDirector::sharedDirector();
	pDirector->setOpenGLView(CCEGLView::sharedOpenGLView());
    
	CCSize designSize = CCSizeMake( 480,320);
	CCSize resourceSize = CCSizeMake( 960,640);
	CCSize screenSize = CCEGLView::sharedOpenGLView()->getFrameSize();
    
	std::vector<std::string> resDirOrders;
    
	TargetPlatform platform = CCApplication::sharedApplication()->getTargetPlatform();
	if (platform == kTargetIphone || platform == kTargetIpad)
	{
		std::vector<std::string> searchPaths = CCFileUtils::sharedFileUtils()->getSearchPaths();
		searchPaths.insert(searchPaths.begin(), "Published-iOS");
		CCFileUtils::sharedFileUtils()->setSearchPaths(searchPaths);
		if (screenSize.height == 1536)//iPad-hd
		{
            CCLog("resources-ipadhd");
			resourceSize = CCSizeMake(2048, 1536);
			resDirOrders.push_back("resources-ipadhd");
		}
		else if (screenSize.height == 768)//iPad
		{
            CCLog("resources-ipad");
			resourceSize = CCSizeMake(1024, 768);
			resDirOrders.push_back("resources-ipad");
		}
		else if (screenSize.height == 640)//iPhone 4/4s/5
		{
            CCLog("resources-iphonehd");
            resourceSize = CCSizeMake( 960,640);
			resDirOrders.push_back("resources-iphonehd");
		}
		else//iPhone 3GS
		{
            CCLog("resources-iphone");
			resourceSize = CCSizeMake(480, 320);
			resDirOrders.push_back("resources-iphone");
		}
	}
	else if (platform == kTargetAndroid || platform == kTargetWindows)
	{
		if (screenSize.height > 720)
		{
			resourceSize = CCSizeMake( 960,640);
			resDirOrders.push_back("resources-large");
		}
		else if (screenSize.height > 568)
		{
			resourceSize = CCSizeMake(480, 720);
			resDirOrders.push_back("resources-medium");
		}
		else
		{
			resourceSize = CCSizeMake(320, 568);
			resDirOrders.push_back("resources-small");
		}
	}
	
	CCFileUtils::sharedFileUtils()->setSearchResolutionsOrder(resDirOrders);
	pDirector->setContentScaleFactor(resourceSize.width/designSize.width);
	CCEGLView::sharedOpenGLView()->setDesignResolutionSize(designSize.width, designSize.height, kResolutionNoBorder);
	
	// turn on display FPS
	pDirector->setDisplayStats(true);
	
	// set FPS. the default value is 1.0/60 if you don't call this
	pDirector->setAnimationInterval(1.0 / 60);
	
	ScriptingCore* sc = ScriptingCore::getInstance();
	sc->addRegisterCallback(register_all_cocos2dx);
	sc->addRegisterCallback(register_all_cocos2dx_extension);
	sc->addRegisterCallback(register_all_cocos2dx_extension_manual);
	sc->addRegisterCallback(register_cocos2dx_js_extensions);
	sc->addRegisterCallback(register_CCBuilderReader);
	sc->addRegisterCallback(jsb_register_chipmunk);
	sc->addRegisterCallback(jsb_register_system);
	sc->addRegisterCallback(JSB_register_opengl);
	sc->addRegisterCallback(MinXmlHttpRequest::_js_register);
    sc->addRegisterCallback(register_jsb_websocket);
	
	sc->start();
	
	CCScriptEngineProtocol *pEngine = ScriptingCore::getInstance();
	CCScriptEngineManager::sharedManager()->setScriptEngine(pEngine);
	ScriptingCore::getInstance()->runScript("main.js");
    
	return true;
}
```
全部修改完成后保存，退出编辑。

仔细对比不难发现，这里所做的修改，是在应用启动的时候根据设备类型和屏幕分辨率来设置资源搜索路径。这个步骤的设置常常会被忽略掉，我本人也曾经在这里浪费掉很多时间。究其原因，是CocosBuilder给人留下了“自动适配多种分辨率”印象，从而下意识地认为这个适配工作将会由引擎来自动完成，而实际情况却不是这样。CocosBuilder可以以某个分辨率下的资源为准，通过自动缩放来为生成适配其他分辨率的资源，但游戏中如何加载这些不同的资源却需要手工设置规则。

####修改iOS工程配置

    $  open proj.ios/MyGame.Xcodeproj/ #打开iOS工程
iOS工程打开后，分别做以下三件事：

* 可以看到Resources分组下的main.js, res, src是显示红色的，这些文件或文件夹在此前的步骤里已经连带Resources目录删除而不存在，但其引用关系依然在，所以我们将其选中并右键点选“delete”删除。

![tutorial-05.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-05.png)

* 将此前由CocosBuilder自动生成的Published-iOS文件夹拖拽到Xcode中。注意要放在MyGame的根节点下，并且要在弹出的窗口中去掉第一个"Copy items into …"选项前面的勾选状态，然后选中“Create folder references for any added folders”。这两个操作非常重要，其含义是在MyGame的Xcode工程中对Published-iOS文件夹建立一个“引用”关系，而不是将此文件夹的内容复制到工程中。这样的好处是，今后Published-iOS目录中有任何变动都会及时地反应到MyGame，而无须再次手动添加到工程。

![tutorial-06.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-06.png)

* 注意Xcode左上角的scheme区域的左侧，如果显示的是cocos2dx，请点击该按钮将其手动改为MyGame。如果本就是MyGame则无须修改。另外，右边的iOS Device也需要重新点选为一个具体的模拟器，如图，我选择的是iPhone 6.1 Simulator。

![tutorial-07.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-07.png)

完成以上步骤后，点击左上角Run按钮，Xcode将会开始编译并自动启动模拟器来启动和运行MyGame。看看效果，就是此前在CocosBuilder所生成的默认画面。

####修改Android工程配置[待更新，欢迎提交]

####配置HTML5环境

HTML5版本中加载资源需要通过HTTP请求来完成，所以必须要有一个WebServer。你可以将Published-HTML5中的所有文件都上传到你已有的服务器，也可以在本地搭建一个。本教程将不会涉及WebServer的内容，所以采用本地环境来搭建。Python中有个快速搭建的命令：

	$  cd {cocos2d-x}/projects/MyGame/Published-HTML5
    $  python -m SimpleHTTPServer &
*`提示`：python -m SimpleHTTPServer会在本机启动一个WebServer，监听8000端口，其默认目录即是执行命令所在的当前目录。因此注意要切换到你的Published-HTML5目录下执行。*

当屏幕出现类似如下提示时表示WebServer启动成功：

    Serving HTTP on 0.0.0.0 port 8000 ...
这时候打开浏览器，输入`http://localhost:8000`即可在浏览器中启动由CocosBuilder为我们自动生成的游戏。

至此，我们的开发环境已经全部搭建完成，从此，我们就可以开始一种全新的、完全在CocosBuilder中进行游戏开发的开发体验了。接下来，我们会以CocosBuilder为MyGame工程默认创建的资源为基础，通过示例讲解如何在CocosBuilder中进行JSB的开发。

##三、在CocosBuilder中进行游戏开发

打开MyGame的CocosBuilder工程（也可以用图形界面的菜单选项打开File->Open）：

    $  open {cocos2d-x}/projects/MyGame/MyGame.ccbproj
其界面大致介绍如下图：

![tutorial-09.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-09.png)

在左侧的资源列表区，我们可以看到两个以MainScene开头的文件，一个是ccb，一个是js。

ccb文件类似PhotoShop的PSD文件，我们开发游戏过程中的场景、精灵、动画等都可以在ccb文件中进行编辑。另一个js文件无须多讲，这就是游戏的逻辑代码，以JavaScript来编写。

MainScene.ccb是CocosBuilder的默认入口，即游戏启动后第一个加载的场景。如果你想用其他文件作为默认入口，只需打开工程设置（File->Project settings）修改“Start ccb-file name”即可。在工程设置窗口，还可以看到其他选项，第一个“Resources paths”前面已经讲过，设置资源路径，第二个默认勾选的“JavaScript based project”表示是否使用JSB开发模式，如果你的游戏不采用JSB，就可以去掉这个勾选。第三个默认入口已经讲过，最后一个选项是设置游戏支持的屏幕旋转方向，默认是横屏。

我们来到CocosBuilder界面中下面的编辑区，“Default Timeline”表示这是当前ccb的默认时间轴，点击这里会弹出新建和编辑timeline等选项，每个ccb文件都可以创建多个timeline，并且可以将多个timeline连接起来，具体可见点击界面底部的“No chained timeline”的弹出菜单。

![tutorial-10.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-10.png)

点击选中CCLayer节点，右侧会显示这个层的各种属性设置选项。我们注意到最顶部的“Code Connections”中“JS Controller”，这就是ccb文件和JavaScript自动连接的重要选项，它为当前这个层定义了一个JavaScript的控制器类的名称，这样我们就可以在程序代码中来直接操作这个层里面的资源或节点。

再往下，点击选中CCLabelTTF节点，右侧的“Code Connections”中“JS Controller”被禁用，第二个选项则定义了这个文本控件的名称helloLabel。类似地，我们再选中最后一个CCMenuItemImage节点，这是一个按钮控件，右侧下面的CCMenuItem区域也已经为这个按钮定义好了其点击时触发的方法名称onPressButton，这样我们也只需在JavaScript代码中定义此方法即可。

在左侧资源列表区双击打开MainScene.js，这里的内容就可以看到MainScene的类定义、按钮点击时调用的onPressButton的方法定义及成员变量helloLabel的用法。到这里，我们应该就基本明白了在CocosBuilder中进行游戏开发的大致流程，我简单总结如下：

#####1）在ccb中组织美术资源，创建动画、特效
#####2）根据需要设置相关节点的属性以备代码连接
#####3）编写JavaScript程序实现游戏逻辑
#####4）发布，然后分别在Xcode、Eclipse和浏览器中调试结果

*`提示`：以上第4步的调试工作也可以在CocosPlayer中进行，但CocosPlayer仅支持iOS和Android模拟器，考虑到HTML5平台调试的问题，本教程将暂不涉及CocosPlayer。*

通常情况下，一个游戏开发过程中有关场景、动画和特效等视觉上的工作都是由团队中的美术来负责完成的，因此ccb这些文件理论上来说应该由美术来完成编辑再交由开发工程师来调用，但我更推荐工程师也要学会使用CocosBuilder中场景、动画和特效编辑等功能的用法，或者索性由二者共同完成。因为CocosBuilder目前尚不完善，美术在学习和使用过程中可能会遇到一些问题，另外资源的组织方式也是会对游戏的性能产生影响的，这时候就需要工程师能够协助把关，共同实现最优的解决方案。

看完了默认的示例，我们接下来将基于这个例子的资源和代码逐步深入讲解一些游戏开发中常见问题的实现。

###1、响应触屏事件

触屏事件是游戏开发中必不可少的，是玩家和游戏产生交互的基础。我们将以MainScene为例来讲解如何响应玩家的触屏事件，目标是实现玩家点击屏幕任意区域后在helloLabel这个文本区域内显示其点击位置的坐标。

双击打开MainScene.ccb，选中根节点（CCLayer），我们在右侧最下面的CCLayer属性设置中可以看到四个选项，其各自的含义解释如下图：

![tutorial-11.png](https://github.com/loosen/tutorial-of-cocos2dx-jsb-with-cocosbuilder/raw/master//images/tutorial-11.png)

我们预期的是想让MainScene响应触屏事件，因此默认选项不用更改即可，其他三项鼠标事件、重力感应事件和键盘事件没有用到则暂且不管。

现在来到代码中，双击打开MainScene.js，在文件最后一行下面增加如下代码：

```
MainScene.prototype.onDidLoadFromCCB = function()
{
	cc.log('MainScene ccb file has been loaded!');
	
	this.rootNode.onTouchesBegan = function( touches, event) {
		// 将触屏开始事件转发给控制器 (this)
        this.controller.onTouchesBegan(touches, event);
        return true;
    };
    
	this.rootNode.onTouchesMoved = function( touches, event) {
		// 将触屏移动事件转发给控制器 (this)
        this.controller.onTouchesMoved(touches, event);
        return true;
    };
    
	this.rootNode.onTouchesEnded = function( touches, event) {
		// 将触屏结束事件转发给控制器 (this)
        this.controller.onTouchesEnded(touches, event);
        return true;
    };
};

MainScene.prototype.onTouchesBegan = function(touches, event)
{
	// 修改文本内容
	this.helloLabel.setString("TOUCH START: "+parseInt(touches[0].getLocation().x)+", "+parseInt(touches[0].getLocation().y));
};

MainScene.prototype.onTouchesMoved = function(touches, event)
{
	// do some staff here
};

MainScene.prototype.onTouchesEnded = function(touches, event)
{
	// do some staff here
};

```

以上代码中，onDidLoadFromCCB方法会在ccb文件加载完成后自动调用，类似iOS中的viewDidLoad。我们在onDidLoadFromCCB中分别将三种触屏事件转发给控制器自定义的方法去处理。在触屏开始事件的处理方法onTouchesBegan中，我们将触屏位置信息转换成文本显示在屏幕上，实现了我们的目标。另外两个onTouchesMoved和onTouchesEnded暂时没有用上，但我们应该能就此理解它们的用法了。

###2、HTTP网络请求

为了更好地演示XMLHttpRequest的用法，我们利用前面配置HTML5环境时开启的WebServer来提供一个HTTP服务接口，该接口直接返回一段JSON格式的字符串，JavaScript将在接收到接口返回的数据时进一步解析处理。

    $  cd {cocos2d-x}/projects/MyGame/Published-HTML5
    $  echo '{"data":"I am from the remote server"}' > test.json
以上操作在WebServer的根目录创建了一个test.json文件，内容是一段JSON格式的文本。我们的目标是玩家点击屏幕上的按钮时向`http://localhost:8000/test.json`发起一个HTTP请求，并将返回数据中data字段的内容打印在屏幕上。

同样，双击打开MainScene.js，将MainScene.prototype.onPressButton的方法修改如下：


```
// Create callback for button
MainScene.prototype.onPressButton = function()
{	
    // Rotate the label when the button is pressed
    //this.helloLabel.runAction(cc.RotateBy.create(1,360));
    
    //新增以下代码
    var theHelloLabel = this.helloLabel;
    var xhr = new XMLHttpRequest();
	xhr.onreadystatechange = function() {
		if (xhr.readyState==4) {// 4 = "loaded"
			if (xhr.status==200) {// 200 = "OK"
				var response = JSON.parse(xhr.responseText);
				cc.log(response);
				theHelloLabel.setString(response.data);
			} else {
				cc.log("Problem retrieving JSON data:" + xhr.statusText);
			}
		}
	};
	//发起一个GET请求
	xhr.open("GET", "http://localhost:8000/test.json");
    xhr.send(null);
};
```

###3、多分辨率适配方案[待更新，欢迎提交]
###4、WebSocket[待更新，欢迎提交]
###5、Plugin-X整合绑定[待更新，欢迎提交]
###6、自定义第三方库的绑定[待更新，欢迎提交]
###7、在线资源更新解决方案[待更新，欢迎提交]
