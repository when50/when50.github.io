---
layout: post
title: ReSwift Tutorial
date: 2020-01-15 14:28:20 +0800
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tags: [ReSwift, unidirectional data flow, design pattern, Swift]
---

[原文：ReSwift教程：记忆力游戏][5]

随着iOS app数量的增长，MVC模式像“goto”一样逐渐丧失了首选地位。

有了更多可伸缩的架构模式供iOS开发者选择，例如MVVM，VIPER，还有Riblets。看起来这些架构迥然不同，但它们有个相同的目标：把多项数据流拆分成单一职责的代码块。在多向数据流中，不同模块之间数据流向多个方向。

有时候，你不想（需要）多向数据流，换而言之，你想要数据流是单向的，即单向数据流（*unidirectional*）。

ReSwift是啥？

ReSwift是帮你创建类Redux架构的Swift小框架。

ReSwift由4个主要部分组成：

* Views：响应Store变化，把其展示在屏幕上。发送Actions。
* Actions：在app中发起状态改变。Reducer负责处理Action。
* Reducers：直接修改存储在Store中的应用状态。
* Store：存储应用当前的状态。Views之类的模块可以订阅、响应Store的改变。

ReSwift展现了许多有趣的有点：

* 超强的约束：把逻辑并不真正相关的小块代码放在一个方便的位置，是非常有诱惑力的。ReSwift严格约束了（行为）在哪里发生和发生了什么（行为）。
* 单向数据流：多向数据流开发的App很难理解和调试。一个改变会导致程序发送一连串数据的事件。单向数据流更容易设想变化，并且可以大大减少代码的理解成本。
* 便于测试：大部分（业务）逻辑包含在函数式的Reducers当中。
* 平台无关：ReSwift所有组件都是平台无关的。你可以方便的应用到iOS、macOS或者tvOS当中。

对比多向和单向数据流

举个例子来说明前面的数据流。VIPER框架支持组件之间的多向数据流：

![img](https://koenig-media.raywenderlich.com/uploads/2017/02/first-480x239.png)

对照ReSwift框架的单向数据流：

![img](https://koenig-media.raywenderlich.com/uploads/2017/02/Image-2017-02-26-at-11.21.46-PM-480x196.png)

数据只能单向流动，所以在app中单向数据流展示数据和追踪问题容易得多。

入门

下载[基础工程][1],其中包含了骨干代码和框架，还包含了ReSwift。

首先，你需要装配ReSwift。你要开始装配应用程序的核心：状态。

打开AppState.swift，创建AppState结构体，实现了StateType协议。

```swift
import ReSwift
struct AppState: StateType {
  
}
```

这个结构体定义了整个app的状态。

在创建包含AppState的State之前，必须要先创建主Reducer。

![img](https://koenig-media.raywenderlich.com/uploads/2017/03/reducer-480x196.png)

Reducers只是用来直接改变Store持有的AppState当前值的代码块。只有Actions可以发起Reducer，从而改变应用程序的当前状态。Reducer根据Action的类型改修改AppState的当前值。

> 注意：App中只有一个Store，也只有一个Reducer

在AppReducer.swift中创建主Reducer

```swift
import ReSwift

func appReducer(action: Action, state: AppState?) -> AppState {
  return AppState()
}
```

appReducer是个函数，输入一个Action，返回改变后的AppState。state是应用的当前状态；这个函数按照收到的action类型相应的改变state。现在它简单的创建了一个新的AppState-在配置Store时，需要返回。

现在可以创建Store了。

![img](https://koenig-media.raywenderlich.com/uploads/2017/03/store-480x196.png)

Store包含你的整个app的当前状态：AppState结构的值。打开AppDelegate.swift，把`import UIKit`替换为：

```swift
import ReSwift

var store = Store<AppState>(reducer: appReducer, state: nil)
```

这步用`appReducer`初始化了全局变量`store`。`appReducer`是Store的主Reducer，包含了store收到一个Action时作出响应的所有操作。这里是初步创建而非迭代修改，你可以给`state`传nil。

构建并且运行，确保app编译正确：

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/05/Simulator-Screen-Shot-May-10-2017-9.24.33-PM-281x500.png)

App路由

现在为app创建第一个真正的状态。从界面导航开始或者说路由。

App路由在所有框架中都是一个挑战，不仅是ReSwift中。在MemoryTunes中，你会用一个非常容易的方法，你把所有（路由）目标定义为一个enum，你的`AppState`持有当前目标。`AppRouter`会响应并改变这个值，并把当前目标显示在屏幕上。

打开AppRouter.swift，把`import UIKit`替换为：

```swift
import ReSwift

enum RoutingDestination: String {
  case menu = "MenuTableViewController"
  case categories = "CategoriesTableViewController"
  case game = "GameViewController"
}
```

这个枚举代表了app中的所有视图控制器。

最后，你需要把应用的状态保存到store。现在只有一个主状态的结构（这个例子中是`AppState`），你可以拆分app的状态到多个子状态，并在主状态中引用他们。

这是个好的编程实践，你要把状态分组到子状态结构中。打开RoutingState.swift，为路由添加下面的子状态结构：

 ```swift
import ReSwift

struct RoutingState: StateType {
  var navigationState: RoutingDestination
  
  init(navigationState: RoutingDestination = .menu) {
    self.navigationState = navigationState
  }
}
 ```

`RoutingState`包含`navigationState`，代表屏幕上正在显示的当前目标。

> 注意:`navigationState`的默认值是`menu`。如果你没有在初始化RoutingState时指定，它隐含指定了应用开始状态的默认值。

在AppState.swift中，添加下面的结构：

```swift
let routingState: RoutingState
```

`AppState`现在包含了`RoutingState`子状态。

构建并运行，你会注意到有问题：

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/04/missingInit.png)

`appReducer`不能编译！因为你给`AppStore`加入了`routingState`，但初始化的时候并没有给出任何它的信息。为了维护`routingState`，你需要一个reducer。

现在我们只有一个主Reducer函数，和状态一样，reducers也要拆分成子reducers。

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/03/sub3.png)

添加RoutingReducer.swift文件：

```swift
import ReSwift

func routingReducer(action: Action, state: RoutingState?) -> RoutingState {
  let state = state ?? RoutingState()
  return state
}
```

和主Reducer类似，`routingReducer`依据收到的action改变state状态，然后返回。目前我们还没有actions，如果`state`是`nil`就赋给它一个新的，并把它返回。

子Reducers负责为对应的子状态初始化一个初始值。

回到AppReducer.swift修复编译警告。编辑`appReducer`的内容为：

```swift
return AppState(routingState: routingReducer(action: action, state: state?.routingState))
```

这里为`AppState`初始化器添加了`routingState`参数。mainReducer把`action`和`state`传入到`routingReducer`得到新状态。这个是常规操作，你会为你创建的子状态和子reducer重复这个操作。

订阅

还记得`RoutingState`的默认值`menu`吗？这正是我们app的当前状态！但是我们没有在任何地方订阅它。

任何类都可以订阅Store，并不限于视图。一旦一个类订阅了Store，他就会在收到app当前状态或者子状态的所有改变信息。我们将会订阅，当AppRouter改变routingState时，进而改变`UINavigationController`在屏幕上中显示的内容。

打开AppRouter.swift，替换成下面的内容：

```swift
final class AppRouter {
  let navigationController: UINavigationController
  
  init(window: UIWindow) {
    navigationController = UINavigationController()
    window.rootViewController = navigationController
    // 1
    store.subscribe(self) {
      $0.select {
        $0.routingState
      }
    }
  }
  
  // 2
  fileprivate func pushViewController(identifier: String, animated: Bool) {
    let viewController = instantiateViewController(identifier: identifier)
    navigationController.pushViewController(viewController, animated: animated)
  }
  
  private func instantiateViewController(identifier: String) -> UIViewController {
    let storyboard = UIStoryboard(name: "Main", bundle: nil)
    return storyboard.instantiateViewController(withIdentifier: identifier)
  }
}

// MARK: StoreSubscriber
// 3
extension AppRouter: StoreSubscriber {
  func newState(state: RoutingState) {
    // 4
    let shouldAnimate = navigationController.topViewController != nil
    // 5
    pushViewController(identifier: state.navigationState.rawValue, animated: shouldAnimate)
  }
}
```

上面的代码，我们给`AppRouter`添加了个扩展。这里仔细看下做了什么：

1. `AppState`现在订阅了全局`Store`。在闭包中，`select`指明专门订阅了routingState的变化。
2. `pushViewController`会实例化并推入给出的视图控制器。它把identifier传递给instantiateViewController，用来加载视图控制器。
3. 使`AppRouter`符合`StoreSubscriber`，从而在`routingState`变化时得到`newState`。
4. 我们不想在根视图控制器应用动画，所以确认当前是否是推入根。
5. 状态改变时，会用`state.navigationState`的`rawValue`作为目标视图控制器的名字，推入`UINavigationController`。

`AppRouter`首先响应并推入初始化的`menu`表示的`MenuTableViewController`到导航控制器。

构建并运行，查看输出：

![ReSwift tutorial ](https://koenig-media.raywenderlich.com/uploads/2017/04/Simulator-Screen-Shot-Apr-30-2017-5.29.59-PM-281x500.png)

我们的app会显示`MenuTableViewController`，暂时还没有内容。下一节我们给它增加选项，以便路由到其它屏幕。

Views

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/03/view-480x196.png)

任何东西都可以是`StoreSubscriber`，绝大部分时间都是视图响应状态变化。我们的目标是让`MenuTableViewController`显示两个不同的菜单项。此时可以进行State/Reducer的常规任务了！

打开MenuState.swift用下面的代码建立菜单状态：

```swift
import ReSwift

struct MenuState: StateType {
  var menuTitles: [String]
  
  init() {
    menuTitles = ["New Game", "Choose Category"]
  }
}
```

`MenuState`包含`menuTitles`，用标题初始化并显示在表视图中。

在MenuReducer.swift中，创建Reducer用下面的代码：

```swift
import ReSwift

func menuReducer(action: Action, state: MenuState?) -> MenuState {
  return MenuState()
}
```

由于`MenuState`是静态的，我们不需要操心如何处理状态改变，所以这里简单返回一个`MenuState`实例。

回到AppState.swift，在`AppState`的下面添加`MenuState`。

```swift
let menuState: MenuState
```

我们再次编辑了默认初始化器导致变异失败。在AppReducer.swift，编辑`AppState`初始化器：

```swift
return AppState(routingState: routingReducer(action: action, state: state?.routingState), menuState: menuReducer(action: action, state: state?.menuState))
```

现在可以订阅`MenuState`，并且用它来渲染菜单视图。

打开MenuTableViewController.swift，把占位代码替换成下面：

```swift
import ReSwift

final class MenuTableViewController: UITableViewController {
  
  // 1
  var tableDataSource: TableDataSource<UITableViewCell, String>?
  
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    // 2
    store.subscribe(self) {
      $0.select {
        $0.menuState
      }
    }
  }
  
  override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    // 3
    store.unsubscribe(self)
  }
}

// MARK: - StoreSubscriber
extension MenuTableViewController: StoreSubscriber {
  func newState(state: MenuState) {
    // 4
    tableViewDataSource = 
      TableDataSource(cellIdentifier:"TitleCell", models:state.menuTitles) { cell, model in
        cell.textLabel?.text = model                                                              			cell.textLabel?.textAlignment = .center                                                   			return cell
      }
    tableView.dataSource = tableDataSource
    tableView.reloadData()
  }
}
```

这个控制器订阅了`MenuState`，直接把其状态和改变渲染到UI。

1. `TableDataSource`是包含了tableView启动和行为的陈述式数据源。
2. 在`viewWillAppear`中订阅`menuState`。在`menuState`发生变化后，都会回调`newState`。
3. 不在需要时，取消订阅。
4. 这是陈述部分。`UITableView`用它生成内容。在代码当中我们可以清楚的知道状态如何转换为视图。

> 注意：你可能注意到了，ReSwift偏好不可编辑，重度使用值类型的结构而非引用类型的对象。这里也鼓励我们创建陈述式的UI代码。这是为啥？
>
> 回调定义在`StoreSubscriber`中的`newState`时，入参是改变后的状态。把捕捉到的状态做属性很有诱惑力，就像：
>
> ```swift
> final class MenuTableViewController: UITableViewController {
> var currentMenuTitlesState: [String]
> ...
> }
> ```
>
> 陈述式UI代码，更容易看出状态是如何转换为视图的，也更容易追踪。例子里的问题是`UITableView`没有陈述式API。所以这里用`TableDataSource`弥补了这个缺陷。有兴趣的话可以查看TableDataSource.swift源码。

构建然后运行，你就会看到菜单项：

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/05/Simulator-Screen-Shot-May-3-2017-9.18.04-PM-281x500.png)

Actions

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/03/action-480x196.png)

现在菜单项有了，可以用它们打开新的界面就更赞了。我们来写第一个Action吧。

Actions发起了Store的变化。一个Action就是个结构，可以包含附加变量：称为行为参数。Reducer依据行为类型和它的参数处理收到的行为。

在RoutingAction.swift中创建一个行为：

```swift
import ReSwift

struct RoutingAction: Action {
  let destination: RoutingDestination
}
```

`RoutingAction`改变当前路由的`destination`。

现在在选中菜单项时，派发`RoutingAction`。

打开MenuTableViewController.swift添加如下代码:

```swift
override func tableView(_ tableView: UITableView, didSelectRowAt: indexPath: IndexPath) {
  var routeDestination: RoutingDestination = .categories
  switch (indexPath.row) {
    case 0: routeDestination = .game
    case 1: routeDestination = .categories
    default: break
  }
  store.dispatch(RoutingAction(destination: routeDestination))
}
```

按照选择的行设置`routeDestination`。把它用作参数，由Store在`RoutingAction`中派发。

行为已经派发，但还有在任何reducer中支持。打开RoutingReducer.swift，把`routingReducer`的内容替换为代码：

```swift
var state = state ?? RoutingState()

switch action {
  case let routingAction as? RoutingAction:
  	state.navigationState = routingAction.destination
  default: break
}

return state
```

`switch`检查`action`是否是`RoutingAction`。如果是，用它的`destination`修改`RoutingState`，然后返回。

构建、运行。现在点击菜单项，对应的视图控制器会压入导航控制器的顶部。

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/05/navigation.gif)

更新状态

你可能发现现在的导航有个问题。当点击New Game菜单项，`RoutingState`的`navigationState`会从`menu`变成`game`。但点击导航控制器的返回按钮返回到菜单视图控制器时，`navigationState`并没有改变。

在ReSwift中，保持状态和UI同步非常重要。UIKit管理一些东西让我们很容易忽略这一点，例如导航的返回和UITextField输入两种情况。

修复方式就是：在`MenuTableViewController`出现时更新`navigationState`。

在MenuTableViewController.swift，把下面的代码添加到`viewWillAppear的最后：

```swift
store.dispatch(RoutingAction(destination: .menu))
```

在返回按钮返回时，手动更新store。

再次运行并测试导航。啊...导航状态完全错了。什么都没有完全推送出来，也许你看到了崩溃。

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/04/stickers.gif)

打开AppRouter.swift；接收到navigationState时，重复调用了`pushViewController`。这就表示收到菜单`RoutingDestination`时重复推入了menu。

我们需要动态检查`MenuViewController`在push前是否已经显示。把`pushViewController`替换成：

```swift
let viewController = instantiateViewController(identifier: identifier)
let newViewControllerType = type(of: viewController)
if let currentVc = navigationController.topViewController {
  let currentViewControllerType = type(of: currentVc)
  if currentViewControllerType == newViewControllerType {
    return
  }
}

navigationController.pushViewController(viewController, animated: animated)
```

这里用`type(of:)`判断当前视图控制器和将要push的视图控制器。如果他们类型相同，就直接`return`不需要重复push了。

构建并运行，导航正常了，当弹出栈时，路由状态也是正确的了。

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/05/navigation.gif)

按UI行为更新状态，需要做的动态检查往往很复杂。这是应用ReSwift时需要克服的一个挑战。幸运的是这种情况不多。

Categories

接下来我们实现一个更加复杂的界面：`CategoriesTableViewController`。

用户需要选择音乐的分类，让他们玩游戏的时候有喜欢的乐队。从添加状态开始CategoriesState.swift:

```swift
import ReSwift

enum Category: String {
  case pop = "Pop"
  case electronic = "Electronic"
  case rock = "Rock"
  case metal = "Metal"
  case rap = "Rap"
}

struct CategoriesState: StateType {
  let categories: [Category]
  var currentCategorySelected: Category
  
  init(currentCategory: Category) {
    categories = [.pop, .electronic, .rock, .metal, .rap]
    currentCategorySelected = currentCategory
  }
}
```

Category枚举定义了集中音乐分类。`CategoriesState`包含全部分类`categories`，还有追踪当前分类的`currentCategorySelected`。

在changeCategoryAction.swift中，添加如下代码：

```swift
import ReSwift

struct ChangeCategoryAction: Action {
  let categoryIndex: Int
}
```

这个行为用来改变`CategoriesState`，用`categoryIndex`引用音乐分类。

现在需要实现`Reducer`接收`ChangeCategoryAction`存储新的状态。打开CategoriesReducer.swift添加代码：

```swift
import ReSwift

private struct CategoriesReducerConstants {
  static let userDefaultsCategoryKey = "currentCategoryKey"
}

private typealias C = CategoriesReducerConstants

func categoriesReducer(action: Action, state: CategoriesState?) -> CategoriesState {
  let currentCategory: Category
  // 1
  if let loadedCategory = getCurrentCategoryStateFromUserDefaults() {
    currentCategory = loadedCategory
  } else {
    currentCategory = .pop
  }
  var state = state ?? CategoriesState(currentCategory: currentCategory)
  
  switch action {
    case let changeCategoryAction as ChangeCategoryAction:
    //
    let newCategory = state.categories[changeCategoryAction.categoryIndex]
    state.currentCategorySelected = newCategory
    saveCurrentCategoryStateToUserDefautls(category: newCategory)
    default: break
  }
  
  return state
}

private func getCurrentCategoryStateFromUserDefautls() -> Category? {
  let userDefaults = UserDefaults.standard
  let rawValue = userDefaults.string(forKey: C.userDefaultsCategoryKey)
  if let rawValue = rawValue {
    return Category(rawValue: rawValue)
  } else {
    return nil
  }
}

private func saveCurrentCategoryStateToUserDefaults(category: Category) {
  let userDefaults = UserDefautls.standard
  userDefaults.set(category.rawValue, forKey: C.userDefaultsCategoryKey)
  userDefaults.synchronize()
}
```

和其它reducer一样，这个完全地实现了从行为到状态更新。在这里，我们持久化选中的分类到`UserDefautls`。下面详细说下做了什么：

1. 从`UserDefaults`中加载可用的分类，如果当前AppState没有分类状态的话，就用分类初始化一个。
2. 响应`ChangeCategoryAction`：更新`state`，把分类保存到`UserDefaults`。
3. `getCurrentCategoryStateFromUserDefaults`辅助方法从`UserDefaults`之中加载分类。
4. `saveCurrentCategoryStateToUserDefaults`辅助方法把分类保存到`UserDefautls`。

辅助方法都是全局方法。我们可以把他们放到一个类或一个结构中，但他们总该是纯粹的。

我们需要给AppState加入新状态。打开AppState.swift把下面的代码添加进结构：

```swift
let categoriesState: CategoriesState
```

`categoriesState`现在是`AppState`的一部分了，你应该可以处理这个了！

打开AppReducer.swift，编辑返回值如下：

```swift
return AppState(
	routingState: routingReducer(action: action, state: state?.routingState),
	menuState: menuReducer(action: action, state: state?.menuState),
	categoriesState: categoriesReducer(action: action, state: state?.categoriesState))
```

这里我们在appReducer中添加了`categoriesState`，并传入了`action`和`categoriesState`。

现在可以创建分类界面了，和`MenuTableViewController`一样。我们订阅Store并使用`TableDataSource`。

打开CategoriesTableViewController.swift，替换成如下：

```swift
import ReSwift

final class CategoriesTableViewController: UITableViewController {
  var tableDataSource: TableDataSource<UITableViewCell, Category>?
  
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    // 1
    store.subscribe(self) {
      $0.select {
        $0.categoriesState
      }
    }
  }
  
  override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    store.unsubscribe(self)
  }
  
  override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
    // 2
    store.dispatch(ChangeCategoryAction(categoryIndex: indexPath.row))
  }
}

// MARK: - StoreSubscriber
extension CategoriesTableViewController: StoreSubscriber {
  func newState(state: CategoriesState) {
    tableDataSource = TableDataSource(cellIdentifier:"CategoryCell", models: state.categories) { cell, model in
      cell.textLabel?.text = model.rawValue
    	// 3
    	cell.accessoryType = (state.currentCategorySelected == model) ? .checkmark : .none
			return cell                                                                              			}
    
    self.tableView.dataSource = tableDataSource
    self.tableView.reloadData()
  }
}
```

看起来和`MenuTableViewController`十分相似。有一些亮点：

1. 在`viewWillAppear`中订阅`categoriesState`，在`viewWillDisappear`中取消订阅。
2. 选中一个cell时派发`ChangeCategoryAction`。
3. 在`newState`中，用勾选标记当前选中的分类。

一切都设置完毕。现在可以选择分类了。构建运行app，选择**Choose Category**自行查看。

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/05/Simulator-Screen-Shot-May-7-2017-3.06.05-PM-281x500.png)

异步任务

[异步编程很难][2]，果真如此吗？在ReSwift中并非如此。

我们要从[Itunes API][3]中加载图片作为记忆力卡片。我们首先建立游戏状态，reducer和相关的行为。

打开GameState.swift，这里的`MemoryCard`结构表示一张游戏卡。它用`imageUrl`包含会在卡片上显示的图片链接。用`isFlilpped`标记卡片是否可见，`isAlreadyGuessed`代表卡片是否已经匹配。

我们在文件中添加游戏状态。在顶部引入ReSwift：

```swift
import ReSwift
```

把这些代码添加到文件底部：

```swift
struct GameState: StateType {
  var memoryCards: [MemoryCard]
  // 1
  var showLoading: Bool
  // 2
  var gameFinished: Bool
}
```

这些定义了游戏的状态。除了`memoryCards`包含可用的卡片外，这里的属性标示了：

1. 是否正在显示加载指示器。
2. 游戏是否结束。

在GameReducer.swift中添加代码：

```swift
import ReSwift
func gameReducer(action: Action, state: GameState?) -> GameState {
  let state = state ?? GameState(memoryCards: [], showLoading: false, gameFinished: false)
  
  return state
}
```

现在仅新建了一个`GameState`。稍后回来补充完整：

在AppState.swift中，在`AppState`下面添加`gameState`：

```swift
let gameState: GameState
```

在**AppReducer.swift**，最后一次更新初始化器：

```swift
return AppState(
	routingState: routingReducer(action: action, state: state?.routingState),
  menuState: menuReducer(action: action, state: state?.menuState),
  categoriesState: categoriesReducer(action: action, state: state?.categoriesState),
  gameState: gameReducer(action: action, state: state?.gameState))
```

> 注意：注意到可以预测，在几次使用Action/Reducer/State后，所有的东西都是简单和熟悉的。程序员友好得益于自然的单向数据和模块间的严格约束。我们已经学到了，只有Reducer可以改变app Store，只有Action可以触发这种改变。我们立即可以知道去哪里可以看，去哪里可以添加代码。

现在定义更新卡片的行为，打开SetCardsAction.swift：

```swift
import ReSwift

struct SetCardsAction: Action {
  let cardImageUrls: [String]
}
```

这个行为为`GameState`中的卡片设置图片链接。

我们现在来创建首个异步行为。打开**FetchTunesAction.swift**，添加代码：

```swift
import ReSwift

func fetchTunes(state: AppState, store: Store<AppState>) -> FetchTunesAction {
	iTunesAPI.searchFor(category: state.categoriesState.currentCategorySelected.rawValue) { imageUrls in
    store.dispatch(SetCardsAction(cardImageUrls: imageUrls))
  }
  
  return FetchTunesAction()
}

struct FetchTunesAction: Action {
}
```

`fetchTunes`用`iTunesAPI`来加载图片（链接）。在闭包中派发`SetCardsAction`包含加载结果。异步任务在ReSwift里如此容易：在完成时派发行为。仅此而已。

`fetchTunes`返回`FetchTunesAction`表示开始加载了。

打开**GameReducer.swift**添加支持两个新行为的代码。把`gameReducer`内容替换成：

```swift
var state = state ?? GameState(memoryCards: [], showLoading: false, gameFinished: false)

switch action {
  // 1
  case _ as FetchTunesAction:
  	state = GameState(memoryCards: [], showLoading: true, gameFinished: false)
  // 2
  case let setCardsAction as SetCardsAction:
  	state.memoryCards = generateNewCards(with: setCardsAction. cardImageUrls)
  	state.showLoading = false
  default: break
}

return state
```

我们把`state`改为变量，然后实现了视action`执行：

1. 如果是`FetchTunesAction`，设置`showLoading`为`true`。
2. 如果是`SetCardsAction`，打乱卡片顺序，并设置showLoading为false。`generateNewCards`在**MemoryGameLogic.swift**文件里，里面包含了启动器。

现在可以在`GameViewController`中绘制卡片了。从设置cell开始。

打开CardCollectionViewCell.swift，把下面的方法添加到底部：

```swift
func configureCell(with cardState: MemoryCard) {
  let url = URL(string: cardState.imageUrl)
  // 1
  cardImageView.kf.setImage(with: url)
  // 2
  cardImageView.alpha = cardState.isAlreadyGuessed || cardState.isFlipped ? 1 : 0
}
```

`configureCell`中做了下面2点：

1. 用[Kingfisher][4]这个非常棒的库缓存图片。
2. 显示已经猜中和翻开的图片。

接下来我们用colletion view显示卡片。像table views一样，这里有个命名为`CollectionDataSource`的数据源。

打开**GameViewController.swift**首先替换`UIKit`引入：

```swift
import ReSwift
```

在`GameViewController`中，在`showGameFinishedAlert`上方添加：

```swift
var collectionDataSource: CollectionDataSource<CardCollectionViewCell, MemoryCard>?

override func viewWillAppear(_ animated: Bool) {
  super.viewWillAppear(animated)
  store.subscribe(self) {
    $0.select {
      $0.gameState
    }
  }
}

override func viewWillDisappear(_ animated: Bool) {
  super.viewWillDisappear(animated)
  store.unsubscribe(self)
}

override func viewDidLoad() {
  // 1
  store.dispatch(fetchTunes)
  collectionView.delegate = self
  loadingIndicator.hidesWhenStopped = true
  
  // 2
  collectionDataSource = CollectionDataSource(cellIdentifier: "CardCell", models: [], configureCell: { (cell, model) -> CardCollectionViewCell in
    cell.configureCell(with: model)
                                                                                                        return cell
  })
  collectionView.dataSource = collectionDataSource
}
```

注意这里会有一点编译警告，稍后我们适配了`StoreSubscriber`就没问题了。这个视图控制器在viewWillAppear中订阅`gameState`，在viewWillDisappear中取消订阅。在`viewDidLoad`中做了如下工作：

1. 派发`fetchTunes`，开始从iTunes API中加载图片链接。
2. 用`CollectionDataSource`配置cells，用与`configureCell`对应的的`model`。

现在我们需要添加一个扩展来实现`StoreSubscriber`。把下面的代码添加到文件最后:

```swift
// MARK: - StoreSubscriber
extension GameViewController: StoreSubscriber {
  func newState(state: GameState) {
    collectionDataSource?.models = state.memoryCards
    collectionView.reloadData()
    
    // 1
    state.showLoading ? loadingIndicator.startAnimating() : loadingIndicator.stopAnimating()
    
    // 2
    if state.gameFinished {
      showGameFinishedAlert()
      store.dispatch(fetchTunes)
    }
  }
}
```

这里`newState处理了状态变化。更新了数据源以及：

1. 按照状态更新加载指示器显示状态。
2. 游戏结束时，重启游戏并显示弹窗。

构建并运行游戏，选择**New Game**，就能看到记忆力卡片了。

游戏逻辑

游戏的逻辑允许用户翻转两张卡片，如果他们是一样的，他们就不会翻转回去了；否则，他们重新隐藏。玩家的目标就是用最少的步数解开所有卡片。

我们需要翻转行为来做到这些。打开**FlipCardAction.swift**添加代码：

```swift
import ReSwift

struct FlipCardAction: Action {
  let cardIndexToFlip: Int
}
```

翻转一张卡片时，`FlipCardAction`用`cardIndexToFlip`更新`GameState`。

修改`gameReducer`支持`FlipCardAction`，处理游戏算法逻辑。打开**GameReducer.swift**在`default`前插入如下代码：

```swift
case let flipCardAction as FlipCardAction:
	state.memoryCards = flipCard(index: flipCardAction.cardIndexToFlip, memoryCards: state.memoryCards)
	state.gameFinished = hasFinishedGame(cards: state.memoryCards)
```

`flipCard`按照`FlipCardAction`中的`cardIndexToFlip`修改记忆卡片的状态以及游戏逻辑。`hasFinishedGame`被用于确定游戏结束和更新相应的状态。这两个方法在**MemoryGameLogic.swift**文件中。

App拼图的最后一片就是选中一张卡片时派发翻转行为。这会触发游戏逻辑以及相关的状态变化。

在**GameVIewController.swift**中，找到`UICollectionViewDelegate`扩展。把下面的代码添加到`collectionView(_:didSelectItemAt:)`:

```swift
store.dispatch(FlipCardAction(cardIndexToFlip: IndexPath.row))
```

当选中了collection view中的一张卡片时，对应的`indexPath.item`会和`FlipCardAction`一起发送。

运行游戏，现在你可以玩了。玩尽兴了，然后回来。

![ReSwift tutorial](https://koenig-media.raywenderlich.com/uploads/2017/05/Simulator-Screen-Shot-May-10-2017-10.31.23-PM-281x500.png)

从这出发到哪去？

还有好多学习ReSwift的资源。

* **Middleware**：当前Swift中没有一个处理[交叉关注][6]的好方法。在ReSwift中，你可以免费得到！使用ReSwift的[Middleware][7]功能，你可以实现各种各样的关注。它让我们可以方便的在行为中包装关注（日志，统计，缓存）。
* **Routing**：我们为MemoryTunes应用实现了自己的路由。其实可以利用[ReSwift-Router][6]这种更通用的路由。它仍然有开放的问题，或许你可以参与解决呢？
* **Testing**：ReSwift对于测试大概是最简单的框架。Reducers包含了所有需要测试的代码，并且它们都是纯函数式的。纯函数对于相同输入总是产生相同的结果，不依赖app状态也没有副作用。
* **Debugging**：ReSwift的状态是单向数据流下的一个结构，调试起来非常容易。我们可以记录每步的状态直到崩溃。
* **Persistence**：因为app全部状态存在一个地方，可以很容的序列化和持久化。缓存离线内容是个架构性的问题，但在ReSwift中不存在，这基本上没有代价。
* **Other Implementations**：[Katana][10], [ReduxKit][11]

* **还有一些学习链接**：[ReSwift talk][12] [ReSwift Repo][13] [Christian TieTze's Blog][14]



[1]: https://koenig-media.raywenderlich.com/uploads/2017/07/MemoryTunesStarterApp_starter.zip	"Base Project"

[2]: https://ashfurrow.com/blog/comparative-asynchronous-programming/ "Asynchronous Programming is hard"
[3]: https://affiliate.itunes.apple.com/resources/documentation/itunes-store-web-service-search-api/ "ITunes API"
[4]: https://github.com/onevcat/Kingfisher "Kingfisher"

[5]: https://www.raywenderlich.com/516-reswift-tutorial-memory-game-app "ReSwift tutorial"
[6]: https://en.wikipedia.org/wiki/Cross-cutting_concern "cross-cutting concern"
[7]: https://reswift.github.io/ReSwift/master/getting-started-guide.html#middleware "middleware"
[8]: https://github.com/ReSwift/ReSwift-Router "Router"
[9]: https://github.com/ReSwift/ReSwift-Recorder "Recorder"
[10]: https://github.com/BendingSpoons/katana-swift "katana-swift"
[11]: https://github.com/ReduxKit/ReduxKit "ReduxKit"
[12]: https://realm.io/news/benji-encz-unidirectional-data-flow-swift/ "ReSwift talk"
[13]: https://github.com/ReSwift/ReSwift "ReSwift Repo"
[14]: http://christiantietze.de/posts/2016/01/reswift-level-indirection/ "Christian TieTze's Blog"



