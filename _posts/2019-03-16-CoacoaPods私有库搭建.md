---
layout: post
title: 'CocoaPods私有库搭建'
subtitle: '如何创建一个私有的Spec Repo和Pod'
categories: 技术
tags: CocoaPods iOS
---

## 创建Spec Repo

Sepc Repo相当于Pods的索引，所有的公开的Pods的引用文件都在上面。实际上它是一个在搭建在Github上的Git仓库。

现在到Git服务器上，创建一个Git仓库。

目前Github的私有库也已经免费了，所以可以直接在Github上创建，比如[GinhoorSpecs](https://github.com/ginhoor/GinhoorSpecs)。



然后将这个Spec Repo添加到你的本地CoacoaPods的Repo中

```shell
$ pod repo add GinhoorSpecs https://github.com/ginhoor/GinhoorSpecs.git
```

成功之后，在本地的`~/.cocoaPods/repos`路径下，就能看到刚才添加的库了，这样第一步创建Spec Repo就成功了。

## 创建Pod项目

### 使用命令创建项目

这里推荐用CocoaPods的命令来直接创建项目。

```shell

# 进入要创建项目的路径中
$ cd ~/iOS/Workspace

# 创建项目
$ pod lib create GHApplicationMediator

# 执行命令后，根据提示选择选项
Cloning `https://github.com/CocoaPods/pod-template.git` into `GHApplicationMediator`.
Configuring GHApplicationMediator template.

------------------------------

To get you started we need to ask a few questions, this should only take a minute.

If this is your first time we recommend running through with the guide:
 - https://guides.cocoapods.org/making/using-pod-lib-create.html
 ( hold cmd and click links to open in a browser. )


What platform do you want to use?? [ iOS / macOS ]
 > iOS

What language do you want to use?? [ Swift / ObjC ]
 > Objc

Would you like to include a demo application with your library? [ Yes / No ]
 > Yes

Which testing frameworks will you use? [ Specta / Kiwi / None ]
 >
specta
Would you like to do view based testing? [ Yes / No ]
 > No

What is your class prefix?
 > GH

Running pod install on your new library.

Analyzing dependencies
Fetching podspec for `GHApplicationMediator` from `../`
Downloading dependencies
Installing Expecta (1.0.6)
Installing GHApplicationMediator (0.1.0)
Installing Specta (1.0.7)
Generating Pods project
Integrating client project

[!] Please close any current Xcode sessions and use `GHApplicationMediator.xcworkspace` for this project from now on.
Sending stats
Pod installation complete! There are 3 dependencies from the Podfile and 3 total pods installed.

 Ace! you're ready to go!
 We will start you off by opening your project in Xcode
  open 'GHApplicationMediator/Example/GHApplicationMediator.xcworkspace'

To learn more about the template see `https://github.com/CocoaPods/pod-template.git`.
To learn more about creating a new pod, see `http://guides.cocoapods.org/making/making-a-cocoapod`.

#命令执行完成后，项目会自动打开。
```

### 主要结构

```shell
GHApplicationMediator						# 项目名称
├── Example                        			# Demo项目
├── GHApplicationMediator                   # 组件库
│   ├── Assets                            	# 组件库资源文件
│   └── Classes                             # 组件库类文件
├── GHApplicationMediator.podspec			# 主要配置文件
├── README.md								# 说明文件
└── LICENSE                                	# 开源协议
```

## 添加组件库文件

1.  将组件库文件放入路径`/你的库名称/你的库名称/Classes`中
2.  在终端上进入路径`/你的库文件/Example`，在此路径下，执行Pod update，这样Example项目就会加载你的库文件。
3.  以后每次更新库文件，都需要执行步骤二。

## 制作Demo

Demo是一个组件的面子，组件做的再牛，没有Demo，大家也还是不会用。

打开Example，写几个简单的使用案例。

写完之后配置下项目的git仓库，commit一下。

```shell
$ cd ~/iOS/Workspace/GHApplicationMediator			# 进入组件库根目录
$ git add .
$ git commit -s -m "Initial Commit"
$ git remote add origin https://github.com/ginhoor/GHApplicationMediator.git
													# 添加远端仓库
$ git push origin master     						# 提交
```

如果你的git仓库已经存在master，则使用下面的命令进行push

```shell
$ git pull origin master --allow-unrelated-histories 
```

pod是通过tag来关系组件库版本号的，所以还需要打上tag。

```shell
$ git tag -m "first release" "0.1.0"
$ git push --tags
```

## 配置podspec

```ruby
Pod::Spec.new do |s|
  # 组件库名称
  s.name             = 'GHApplicationMediator'
  # 组件库当前版本，也就是tag指定的
  s.version          = '0.1.0'
  # 简介
  s.summary          = 'make AppDelegate lighter.'
  # 详细描述
  s.description      = <<-DESC
				  	   manage appDelegate modules easier
                       DESC
  # 组件库首页
  s.homepage         = 'https://github.com/ginhoor/GHApplicationMediator'
  # 组件库开源协议
  s.license          = { :type => 'Apache License', :file => 'LICENSE' }
  # 作者
  s.author           = { 'ginhoor' => 'ginhoor@gmail.com' }
  # Git仓库地址
  s.source           = { :git => 'https://github.com/ginhoor/GHApplicationMediator.git', :tag => s.version.to_s }
  # 依赖的iOS版本
  s.ios.deployment_target = '10.0'
  # 源文件地址
  s.source_files = 'GHApplicationMediator/Classes/**/*'
  # 依赖资源地址
  # s.resource_bundles = {
  #   'GHApplicationMediator' => ['GHApplicationMediator/Assets/*.png']
  # }
  # 头文件地址
  # s.public_header_files = 'Pod/Classes/**/*.h'
  # 依赖系统库
  # s.frameworks = 'UIKit', 'MapKit'
  # 依赖的第三方库
  # s.dependency 'AFNetworking', '~> 2.3'
end
```

更多设置请参考[帮助文档](https://guides.cocoapods.org/syntax/podspec.html)

## 验证podspec

这里完成以后，需要验证下配置的正确性。

```shell
$ pod lib lint

=================================
 -> GHApplicationMediator (0.1.0)

GHApplicationMediator passed validation.
# 验证通过
```

## 上传组件库

将组件库上传到你的Spec repo上。

```shell
$ pod repo push GinhoorSpecs GHApplicationMediator.podspec

=================================
Validating spec
 -> GHApplicationMediator (0.1.0)

Updating the `GinhoorSpecs' repo

Already up to date.

Adding the spec to the `GinhoorSpecs' repo

 - [Add] GHApplicationMediator (0.1.0)

Pushing the `GinhoorSpecs' repo
# 上传成功
```

之后就可以通过pod search来找到找你的组件库了。

```shell
$ pod search GHApplicationmediator

==================================
-> GHApplicationMediator (0.1.0)
   make AppDelegate lighter.
   pod 'GHApplicationMediator', '~> 0.1.0'
   - Homepage: https://github.com/ginhoor/GHApplicationMediator
   - Source:   https://github.com/ginhoor/GHApplicationMediator.git
   - Versions: 0.1.0 [GinhoorSpecs repo]
```



## 引用组件库

由于这个组件库，用了我们自己的Spec Repo，所以在引用的时候需要同时注明官方的Source源与自己的Source源。

```ruby
source 'https://github.com/ginhoor/GinhoorSpecs.git'
source 'https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git'

target 'FrameworkDemo' do
    pod 'GHApplicationMediator', '~> 0.1.0'
    inhibit_all_warnings!
end
```

