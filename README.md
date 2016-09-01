# FRDIntent

[![Language](https://img.shields.io/badge/language-Swift%202-orange.svg)
](https://developer.apple.com/swift/)
[![iOS](https://img.shields.io/badge/iOS-8.0-green.svg)]()

**FRDIntent** 包括两部分 Intent 和 URLRouter。它们分别可以用于处理应用内和应用外的 ViewController 调用。

## 安装

### Install Cocoapods

[CocoaPods](http://cocoapods.org) is a dependency manager for Objective-C and Swift. You can install it with the following command:

```bash
$ gem install cocoapods
```

### Podfile

只使用 FRDInent/Intent：

```ruby
target 'TargetName' do
  pod 'FRDIntent/Intent', '~> 0.8.0'
end
```

使用 FRDIntent/Intent 和 FRDIntent/URLRouter：

```ruby
target 'TargetName' do
  pod 'FRDIntent', '~> 0.8.0'
end
```

注意：`pod FRDInent` 和 `pod FRDIntent/URLRouter` 将引入相同的代码。这是因为 FRDIntent/URLRouter 依赖于 FRDIntent/Intent。

然后，命令行运行：

```bash
$ pod install
```

## Intent

FRDIntent/Intent 是一个消息传递对象，用于启动 UIViewController。可以认为它是对 Android 系统中的 [Intent](https://developer.android.com/guide/components/intents-filters.html) 的模仿。当然，FRDIntent/Intent 对 Android Intent 而言，做了极度简化。这是因为 FRDIntent/Intent 的使用场景更为简单：只处理应用内的  ViewController 间跳转。

直接使用 iOS 系统方法完成各 ViewController 之间的跳转，各 ViewController 代码会耦合得很紧密。跳转时，一个 ViewController 需要知道下一个 ViewController 是如何创建的各种细节。这造成了 ViewController 之间的依赖。使用 FRDIntent/Intent 传递 ViewController 跳转信息。解除各 ViewController 代码的耦合。如果需要对项目进行模块化，重要的一步就是解除各 ViewController 代码的耦合。在这方面，FRDIntent 是一个可以考虑的方案。

FRDIntent/Intent 有如下优势：

- 充分解耦。调用者和被调用者完全隔离，调用者只需要依赖协议：`IntentReceivable`。一个 ViewControlller 符合该协议即可被启动。
- 对于“启动一个页面，并从该页面获取结果”这种较普遍的需求提供了一个通用的解决方案。具体查看方法：startControllerForResult。这是对 Android 中 startActitiyForResult 的模仿和简化。
- 支持自定义转场动画。
- 可以传递复杂数据对象。

### 使用

主要通过类 `ControllerManager` 使用 FRDIntent/Intent。它提供了三个方法：`register` 用于注册，`startController` 和 `startControllerForResult` 用于启动页面。

#### 注册

```Swift
  let controllerManager = ControllerManager.sharedInstance
  controllerManager.register(NSURL(string: "/frodo/firstViewController")!, clazz: FirstViewController.self)
```

#### 通过指定类名启动 ViewController

```Swift
  let intent = Intent(clazz: SecondViewController.self)
  let manager = ControllerManager.sharedInstance
  manager.startController(source: self, intent: intent)
```

#### 通过 URL 启动 ViewController

```Swift
  let intent = Intent(uri: NSURL(string: "/frodo/firstViewController")!)
  let manager = ControllerManager.sharedInstance
  manager.startController(source: self, intent: intent)
```

#### 启动一个会返回结果的 ViewController

调用页面，也是接受返回结果的页面需要符合协议 `IntentForResultSendable`：

```Swift
  extension ViewController: IntentForResultSendable {

    func onControllerResult(requestCode requestCode: Int, resultCode: ResultCode, data: Intent) {
      if (requestCode == RequestText) {
        if (resultCode == .Ok) {
          let text = data.extra["text"]
          print("Successful confirm get from destination : \(text)")
        } else if (resultCode == .Canceled) {
          let text = data.extra["text"]
          print("Canceled get from destination : \(text)")
        }
      }
    }

  }
```

被调用页面需要符合协议 `IntentForResultReceivable`, 该协议是 `IntentReceivable` 的子协议。在 `IntentReceivable` 基础上，多了两个实例变量定义：

```Swift
  var data: [String: Any]?
  var requestCode: Int?
```

通过 `startControllerForResult` 启动页面：

```Swift
  let intent = Intent(clazz: ThirdViewController.self)
  intent.putExtra(name: "text", data: "Text From Source")
  let manager = ControllerManager.sharedInstance
  manager.startControllerForResult(source: self, intent: intent, requestCode: RequestText)
```

#### 自定义转场动画

在 FRDIntent 中，转场动画被抽象为协议：`ControllerDisplay`，并且已提供了两个转场动画的实现：`PushDisplay` 和 `PresentationDisplay`。自定义转场动画的实现需要符合该协议。

在启动其他页面时，将自定义的转场动画对象赋给 `Intent` 的实例变量 `controllerDisplay` 即可。

如果不指定转场动画，通过 `startController` 启动页面使用的是 `PushDisplay`；通过 `startControllerForResult` 启动页面使用的是 `PresentationDisplay`。


## URLRouter

FRDIntent/URLRouter 是一个 URL Router。通过 FRDIntent/URLRouter 可以用 URL 调起一个注册过的 block。

iOS 系统为各个应用间的 ViewController 相互跳转提供了一种基于 URL 的处理方案。即应用可以声明自己可以处理某些有特定 scheme 和 host 的 URL。其他应用就可以通过调用这些 URL 而跳转到该应用的某些页面。

FRDIntent/URLRouter 是为了使得 iOS 系统中这种基于 URL 的应用间调用的处理更为简单。所以 FRDIntent/URLRouter 和社区已经存在的诸多 URL Routers 的功能和目的差别不大。FRDIntent 实现 URLRouter 是为了使 FRDIntent/URLRouter 可以和 FRDIntent/Intent 配合解决应用内和应用外 ViewController 的调用。

### 使用

#### 向系统暴露应用可以接收的 URL

在 Xcode 中选择你的项目的 Target, 点击 Info, 添加一项 URL Types。
例如：

- Identifier: com.frdintent
- URL Schemes: frdintent
- Role: Editor
- Icon:

#### 接管应用的 URL 处理

````Swift
  func application(app: UIApplication, openURL url: NSURL, options: [String : AnyObject]) -> Bool {
    return URLRouter.sharedInstance.route(url: url)
  }
```

#### 注册


注册一个 ViewControler。在第三方应用调起该 URL 时，会该启动该 ViewController。该 ViewController 的进入动画为 Push 横滑进入方式。

```Swift
  let router = URLRouter.sharedInstance
  router.register(url: NSURL(string: "/story/:storyId")!, clazz: SecondViewController.self)
```

注册一个 block handler。下面例子中的 block handler 中，用注册时的 URL 构造了一个 Intent，并将该 Intent 送出。ControllerManager 会处理这个 Intent。看是否有合适的 ViewController 可以被启动。

如果，需要定制 ViewController 的转场动画，可以使用该方法注册 URL。

```Swift
  let router = URLRouter.sharedInstance
  router.register(url: NSURL(string: "/user/:userId")!) { (params: [String: Any]) in
    let intent = Intent(url: params[URLRouter.URLRouterURL] as! NSURL)
    if let topViewController = UIApplication.topViewController() {
      ControllerManager.sharedInstance.startController(source: topViewController, intent: intent)
    }
  }
```

## URLRouter 和 Intent

FRDIntent/URLRouter 和 FRDIntent/Intent 可以配合使用的。Intent 处理内部 ViewController 跳转；URLRouter 负责外部调用。在 FRDIntent/URLRouter 的实现中，FRDIntent/URLRouter 只是起了暴露外部调用入口，接收外部调用的作用。在应用内，仍然是通过 FRDIntent/Intent 启动 ViewController。

这么做其实是为了隔离了外部调用和内部调用，做这个区分会带来一些好处：

iOS 提供的通过 URL 调用另外一个 ViewController 的方案更适合用在应用间的外部调用。iOS 系统中应用之间的隔离是清晰而明确的，通过 URL 传递信息是合适的。但如果内部调用也使用 URL 传递信息，就会会带来诸多限制。Intent 更适合内部调用的场景。通过 Intent，可以传递复杂数据对象，可以较容易地定义转场动画。

区分了外部调用和内部调用。我们就可以选择是否要将一个内部调用给暴露外部使用。这就避免了使用 URL 的方案中，本质上是内部额调用也暴露给应用外部了。

## FRDIntentDemo

FRDIntentDemo 对 FRDIntent 各种使用方法都做了演示。

对于外部调用的演示，可以在模拟器的 Safari 的地址栏中输入 `frdintent://frdintent.com/user/123`。正常情况下，访问该 URL 将会启动 FRDIntentDemo，并进入 FirstViewController。

## 单元测试

FRDIntentTests 文件夹包含了 FRDIntent 单元测试代码。单元测试不仅是对代码正确性的验证，也是查看如何使用 FRDIntent 的良好示例。

## License

FRDIntent is released under the MIT license. See LICENSE for details.
