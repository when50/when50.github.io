---
layout: post
title: IGListKit教程
date: 2020-02-17 11:51:20 +0800
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tags: [UICollectionView, IGListKit]

---

IGListKit教程-更好的的UICollectionViews

在IGListKit教程当中，你会学习使用Instagram数据驱动框架，更好、更加动态的UICollectionViews。

每个App都以相似的方式开始：少量屏幕、一些按钮、一两个列表。但App随着时间增长，功能缓慢的进入。你的清晰的数据源在产品经理的截止日期压力下崩塌。过了一会，你会离开巨大的视图控制器这堆难以维护的废墟。你的运气来了，有个方案解决这个问题。

Instagram创建的[IGListKit][1]和`UICollectionView`协同工作时让功能缓慢成长的巨大的视图控制器不复存在。用IGListKit构造表格，你可以用彼此隔离的组件构建app，炫酷的更新支持任何数据类型。

在这个教程中，你会用IGListKit重构一个基本的`UICollectionView`，然后扩大这个应用使之成为真正有用的app。



### 起步

你是NASA的顶尖软件工程师，为最新的火星任务值班。团队已经构建Marslink app的首个版本。

用教程的[下载][2]按钮下载这个版本。下载之后，打开Marslink.xcworkspace，构建并运行应用。

![IGListKit](https://koenig-media.raywenderlich.com/uploads/2016/10/Simulator-Screen-Shot-Oct-31-2016-5.32.34-PM-281x500.png)

到这里，应用显示宇航员刊物列表。

你的任务是在应用中添加全体工作人员需要的功能。你自己需要熟悉项目，打开`ClassicFeedViewController.swift`并查看。

如果你使用过UICollectionView，你看到了十分标准的实现：

* `ClassicFeedViewController`是`UIViewController`的子类，并且在扩展中实现了`UICollectionViewDataSource`。
* `viewDidLoad()`创建了一个`UICollectionView`，注册单元格，设置数据源并把它添加到了视图层级中。
* `loader.entries`数组提供了sections的数量，每个有两个cell（一个日期，一个文字）。
* 日期cell包含火星时间，文字实体cell包含刊物文字。
* `colletionView(_:layout:sizeForItemAt:)`返回固定尺寸的日期cell和实体文字相符的计算结果尺寸。

一切看起来工作得都很好，但任务指挥想出一些迫切的产品需求：

> 一个宇航员要在火星上值守。我们需要天气模块和实时交谈。你有48小时。

![img](https://koenig-media.raywenderlich.com/uploads/2016/11/modules.png)

喷气推进实验室（JPL）工程师做了一些工作，不过他们需要你把这些添加到应用之中。

如果一个宇航员家庭带来的压力不足，NASA设计者把app每个子系统动画更新的需求交给你，就是说不能用`reloadData()`。

你设想到底如何把这些新模块整合到app中，还要使他们带有过渡动画？宇航员只有吃不完的马铃薯。

### IGListKit简介

`UICollectionView`是个极度有力的工具，极大的威力就有极大的责任。保持数据源和视图同步极为重要，但断开连接通常会导致崩溃。



[IGListKit][1]是Instagram团队为`UICollectionView`开发的数据驱动框架。在这个框架下，你提供`UICollectionView`显示需要的对象数组。对每种类型的对象，一个**adapter**创建一个称为**section controller**的对象，后者拥有用于创建cell的所有信息。

![IGListKit](https://koenig-media.raywenderlich.com/uploads/2016/11/flowchart-650x357.png)

变化出现时，IGListKit自动比较对象差异并处理批量更新动画。因此你永远不需要自己动手写批量更新，避免[这里][3]告诫的问题列表。



###为UICollectionView添加IGListKit

IGListKit承担了所有困难的工作，标记集合中变化并用动画更新响应的行。它也被精心设计可以处理多个section具有不同的数据和UI。考虑到这一点，IGListKit是新需求的完美解决方案-所以是时候来实践了。

在打开的**Marslink.xcworkspace**中，右键点击**ViewController**组，选择**New File**。添加一个**Cocoa Touch类**，命名为**FeedViewController**，确保语音为**Swift**。

打开**AppDelegate.swift**找到`application(:didFinishLaunchingWithOptions:)`。找到把`ClassicFeedViewController()`压入导航控制器这行，替换成：

```swift
nav.pushViewController(FeedViewController(), animated: false)
```

现在`FeedViewController`是根试图控制器了。你需要保留`ClassicFeedViewController`做参考，不过`FeedViewController`才是你实践IGListKit驱动集合视图的地方。

构建并运行，确保一个新的、空的视图控制器出现在视图上。

![IGListKit](https://koenig-media.raywenderlich.com/uploads/2016/11/Simulator-Screen-Shot-Nov-1-2016-9.46.31-PM-281x500.png)

### 添加期刊加载器

打开**FeedViewController.swift**，把下面的代码添加到`FeedViewController`类的顶部：

``````swift
let loader = JournalEntryLoader()
``````

`JournalEntryLoader`这个类加载硬编码的刊物实体到一个数组当中。

把代码添加到`viewDidLoad()`的底部：

``````swift
loader.loadLatest()
``````

`loadLatest()`是`JournalEntryLoader`的一个方法，它加载信心的刊物实体。

### 添加集合视图

现在可以添加IGListKit特有的控制逻辑到视图控制器中了。首先需要引入这个框架。在**FeedViewController.swift**顶部附近添加新的引入：

``````swift
import IGListKit
``````

> 注意：教程中的项目使用[CocoaPods][4]管理依赖。IGListKit是用Objective-C编写的，所以如果你手动添加到项目，你需要在你的[桥接文件][5]中插入`#import`语句。

在`FeedViewController`顶部添加常量`collectionView`并初始化：

```swift
// 1
let collectionView = {
  // 2
	let view = UICollectionView(
		frame: .zero,
		collectionViewLayout: UICollectionViewFlowLayout())
  // 3
	view.backgroundColor = .black
	return view
}()
```

上面的代码作用是：

1. IGListKit使用一个正常的`UICollectionView`，并在顶部添加它自己的功能，后面你会看到。
2. 从尺寸为零的矩形开始，因为视图尚未创建。和`ClassicFeedViewController`一样，它使用`UICollectionViewFlowLayout`布局。
3. 把背景色设置成NASA标准的黑色。

把下面的代码添加到`viewDidLoad()`的末尾：

```swift
view.addSubview(collectionView)
```

这里把新建的`collectionView`加到控制器的视图中。

把下面的代码添加到`viewDidLoad()`下面：

```swift
override func viewDidLayoutSubviews() {
	super.viewDidLayoutSubviews()
	collectionView.frame = view.bounds
}
```

这里重写了`viewDidLayoutSubviews()`，把`collectionView`的大小设置成 `view`一样的大小。

### ListAdapter和数据源

对`UICollectionView`，你需要相似的**数据源**适配`UICollectionViewDataSource`。它负责返回section和行数还有独特的cells。

在IGListKit中，你用一个`ListAdapter`控制集合视图。你还需要实现了`ListAdapterDataSource`一个数据源，这里不返回数量和cells，你需要提供一个**section controller**的数组。

对新手来说，在**FeedViewController.swift**中，`FeedViewController`顶部添加代码：

```swift
lazy var adapter: ListAdapter = {
	return ListAdapter(
	updater: ListAdapterUpdater(),
	viewController: self,
	workingRangeSize: 0)
}()
```

这里创建并初始化了一个变量`ListAdapter`。初始化需要3个参数：

1. `updater`是一个遵循`ListUpdatingDelegate`的对象，用来处理row和section的更新。`ListAdapterUpdater` 这个默认的实现刚好合用。
2. `viewController`是个`UIViewController`作为adapter的宿主。IGListKit后面会用这个视图控制器导航到其他的视图控制器。
3. `workingRangeSize`是[workingSize][6]的大小，用于准备可视范围外的section的内容。

> 注意：Working ranges等高级话题这里不在这个教程里。[IGList库][1]里面有充足的文档和实例。

在`viewDidLoad()`底部添加代码：

```swift
adapter.collectionView = collectionView
adapter.dataSource = self
```

把`collectionView`连接到`adapter`。还把`self`设置成`adapter`的数据源-编译的话会出错，因为你还没有遵循`ListAdapterDataSource`。

通过扩展`FeedViewController`适配`ListAdapterDataSource`可以修复这个错误。在文件底部添加：

```swift
// MARK: - ListAdapterDataSource
extension FeedViewController: ListAdapterDataSource {
	// 1
	func objects(for listAdapter: ListAdapter) -> [ListDiffable] {
		return loader.entries
	}
	
	// 2
	func listAdapter(_ listAdapter: ListAdapter sectionControllerFor object: Any) -> ListSectionController {
		return ListSectionController()
	}
	
	// 3
	func emptyView(for listAdapter: ListAdapter) -> UIView? {
		return nil
	}
}
```

> 注意：IGListKit大量使用协议方法。尽管你用返回nil实现了最后一个方法，你不用担心在运行时缺失方法。这使得IGListKit很难搞砸。

`FeedViewController`现在遵循了`ListAdapterDataSource`实现了3个必须方法：

* `objects(for:)`返回将要展示在集合视图当中的数据。你在这里提供了`loader.entries`包含的刊物实体。
* 对每一个数据对象，`listAdapter(_:sectionControllerFor:)`必须返回一个**section controller**实例。这里返回了一个简单的。
* `emptyView(for:) `中返回的view会在集合视图没有数据时显示。NASA有点缺时间，所以他们没有个功能的预算。



### 创建你的首个section controller

一个**section controller**是一个抽象，对于给出的**数据对象**，配置和控制集合视图当中section中的cell。这个概念和现有的视图模型配置视图相似：数据对象就是视图模型，cell就是视图。section控制器在中间起胶水作用。

在IGListKit中，你为不同的数据和行为建立一个新的section控制器。JPL工程师已经构建`JournalEntry`模型，所以你需要创建可以处理它的section控制器。

右键点击**SectionsControllers**文件夹选择**新建文件**。创建一个新的**Cocoa Touch Class**，这个类是**ListSectionController**的子类，命名为**JournalSectionController**。

![img](https://koenig-media.raywenderlich.com/uploads/2018/11/ig1-650x233.png)

Xcode不会自动引入三方框架，所以需要在**JournalSectionController.swift**的顶部添加代码：

```swift
import IGListKit
```

把这些属性添加到`JournalSectionController`的顶部：

```swift
var entry: JournalEntry!
let solFormater: SolFormater()
```

`JournalEntry`是你实现数据源中的模型类。`SolFormater`提供了把日期转换为Sol格式的方法。你马上就会用到它们。

`JournalSectionController`当中重写`init()` 方法：

```swift
override init() {
	super.init()
	inset = UIEdgeInstes(top: 0, left: 0, bottom: 15, right: 0)
}
```

如果没这么做，sections相邻的cell之间就没有间隔了。这一步在`JournalSectionController`对象底部添加了15点填充。

你的section控制器需要重写 `ListSectionController`的四个方法，才能和实际数据一起工作。添加扩展代码如下：

```swift
// MARK：- DataProvider
extension JournalSectionController {
	override func numberOfItems() -> Int {
		return 2
	}
	
	override func sizeForItem(at index: Int) -> CGSize {
		return .zero
	}
	
	override func cellForItem(at index: Int) -> UICollectionViewCell {
		return UICollectionViewCell()
	}
	
	override func didUpdate(to object: Any) {
	}
}
```

除了`numberOfItems()`，所有的方法实现都是不完整的，这里简单的为日期和文字返回了数字2。如果你回头看**ClassicFeedViewController.swift**，就会发现对每一个section在`collectionView(_:numberOfItemsInSection:)`也返回了2个项。这其实是一样的。

在`didUpdate(to:)`中添加代码：

```swift
entry = object as? JournalEntry
```

IGListKit会调用section控制器的`didUpdate(to:)`方法来处理一个对象。注意这个方法在cell协议方法前调用。这一步你把对象保存到`entry`中。

> 注意：在section控制器的生命周期中对象会多次改变。这些会在你解锁更多IGListKit更多高级功能后发生，例如[定制模型差异][7]。在这个教程中你不需要担心差异相关内容。

现在你有了一些数据，要开始配置cell了。把`cellForItem(at:)`中的占位符替换为：

```swift
// 1
let cellClass: AnyClass = index == 0 ? JournalEntryDateCell.self : JournalEntryCell.self
// 2
let cell = collectionContext!.deqeueReusableCell(of: cellClass, for: self, at: index)
// 3
if let cell = cell as? JournalEntryDateCell {
	cell.label.text = "SOL \(solFormatter.sols(fromDate: entry.date))"
} else if cell = cell as? JournalEntryCell {
	cell.label.text = entry.text
}
return cell
```

在section需要cell的时候，IGListKit会调用`cellForItem(at:)`并传入索引。代码是这样工作的：

1. 如果索引是第一个，就使用`JournalEntryDateCell`，否则使用`JournalEntryCell`。刊物实体总是显示为一个日期后面跟着文字。
2. 使用类型、section控制器、索引，从重用池当中取得cell。
3. 基于cell类型，用`didUpdate(to:)`中早先设置的`JournalEntry`来配置它。

接下来，替换`sizeForItem(at:)`中的占位符：

```swift
// 1
guard 
	let context = collectionContext,
	let entry = entry
	else {
		return .zero
	}
// 2
let width = context.containerSize.width
// 3
if index == 0 {
	return CGSize(width: width, height: 30)
} else {
	return JournalEntryCell.cellSize(width: width, text: entry.text)
}
```

代码工作：

1. `collectionContext`是个可以为空的弱引用变量。不过它不应该为`nil`，swift提供了便利的`guard`来防备这种情况。
2. `ListCollectionContext`是adapter的包含信息的上下文对象，在section控制器中用到的集合视图还有视图控制器信息都在上下文当中。这里你得到了容器宽度。
3. 如果是首个索引（日期cell），返回容器宽度加30点高度的尺寸。否则，使用cell提供的助理方法动态计算cell中文字的尺寸。

这些模式：重用不同类型的cell，配置和返回不同尺寸，所有的一切和之前使用集合视图看起来都很相似。再声明一下，你可以返回去看看`ClassicFeedViewController`，可以看到大部分代码都一样。

现在你有个接受`JournalEntrty`对象，并返回两个cell以及尺寸的section控制器了。是时候把它们放到一起了。

回到**FeedViewController.swift**，用下面的代码替换`listAdapter(_:sectionControllerFor:)`中的内容：

```swift
return JournalSectionController()
```

每次IGListKit调用这个方法，你都会得到一个新的刊物section控制器。

构建并运行app，你会看到刊物实体列表：

![IGListKit](https://koenig-media.raywenderlich.com/uploads/2016/11/Simulator-Screen-Shot-Nov-5-2016-3.22.23-PM-281x500.png)

### 添加消息

你重构代码速度之快让JPL的工程师很开心，不过他们迫切需要和滞留宇航员建立联系。他们请求你尽快合并消息模块。

添加视图前，首先需要数据。

打开**FeedViewController.swift**，在`FeedViewController`类的顶部添加属性：

```swift
let pathfinder = Pathfinder()
```

`PathFinder()`充当了消息系统。

在`ListAdapterDataSource`扩展当中找到`objects(for:)`，把内容替换为：

```swift
var items: [ListDiffable] = pathfinder.messages
items += loader.entries as [ListDiffable]
return items
```

你会重新调用`ListAdapter`的这个方法，来提供数据对象。我们为新的section控制器添加了消息，首先用`pathfinder.messages`把消息加入到了`items`中。

> 注意：你需要把`entries`转型，以便通过swift编译。这些对象已经符合了`IGListDiffable`协议。

右键点击**SectionControllers**组，创建一个名为**MessageSectionController**的`ListSectionController`的子类。在文件顶部添加IGListKit引入语句：

```swift
import IGListKit
```

为了通过编译，暂时先不修改其他内容。

返回**FeedViewController.swift**更新`ListAdapterDataSource`扩展当中的方法`listAdapter(_:sectionControllerFor:)`为：

```swift
if object is Message {
	return MessageSectionController()
} else {
  return JournalSectionController()
}
```

这里如果对象是Message类型就会返回消息section控制器了。

JPL团队想要你按需求对`MessageSectionController`做设置：

* 接收一条消息。
* 底部填充15点间隔。
* 使用`MessageCell.cellSize(width:text:)`方法返回单个cell的尺寸。
* 重用`MessageCell`，使用`Message`的`text`还有`user.name`配置cell：为其中的label填充内容。
* 在所有标题位置显示`Message`的`user.name`。

试一试！如果你需要的话团队起草了一个解决方案如下：

```swift
import IGListKit

class MessageSectionController: ListSectionController {
	var message: Message!
	
	override init() {
		super.init()
		inset = UIEdgeInstes(top: 0, left: 0, bottom: 15, right: 0)
	}
}

// MARK: - Data Provider
extension MessageSectionController {
	override func numberOfItems() -> Int {
		return 1
	}
	
	override func sizeForItem(at index: Int) -> CGSize {
		guard let context = collectionContext else {
			return .zero
		}
		return MessageCell.cellSize(width: context.containerSize.width, text: message.text)
	}
	
	override func cellForItem(at index: Int) -> UICollectionViewCell {
		let cell = collectionContext?.dequeueReusableCell(of: MessageCell.self, for: self, at: index)
		cell.messageLabel.text = message.text
		cell.titleLabel.text = message.user.name.uppercased()
		return cell
	}
	
	override func didUpdate(to object: Any) {
		message = object as? Message
	}
}
```

你准备好了之后，构建并运行就可以看到消息合并到feed当中了！

![IGListKit](https://koenig-media.raywenderlich.com/uploads/2016/11/Simulator-Screen-Shot-Nov-6-2016-1.22.14-AM-281x500.png)

### 火星天气预报

你的宇航员需要了解当前的天气，以便导航避开诸如尘暴之类的障碍。JPL构建了一个可以显示天气的模块。这里包含了大量的信息，所以他们要求在点击时才显示天气。

![IGListKit](https://koenig-media.raywenderlich.com/uploads/2016/11/weather.gif)

创建最后一个section控制器，命名为**WeatherSectionController**。这个类仅包含一些变量和初始化器：

```swift
import IGListKit

class WeatherSectionController: ListSectionController {
	// 1
	var weather: Weather!
	// 2
	var expanded = false
	
	override init() {
		super.init()
		// 3
		inset = UIEdgeInsets(top: 0, left: 0, bottom: 15, right: 0)
	}
}
```

他们做了下面的工作：

1. 这个section控制器在`didUpdate(to:)`方法中接受一个`Weather`对象。
2. `expanded`这个`Bool`变量追踪了宇航员是否打开了天气section。你用`false`初始化，所以详情cell不存在。
3. 和其他section一样，底部保留15点填充。

现在添加`WeatherSectionController`扩展，重写3个方法：

```swift
// MARK: - Data Provider
extension WeatherSectionController {
	// 1
	override func didUpdate(to object) {
		weather = object as? Weather
	}
	// 2
	override func numberOfItems() -> Int {
		return expanded ? 5 : 1
	}
	// 3
	override func sizeForItem(at index: Int) -> CGSize {
		guard let context = collectionContext else {
			return .zero
		}
		let width = context.containerSize.width
		if index == 0 {
			return CGSize(width: width, height: 70)
		} else {
			return CGSize(width: width, height: 40)
		}
	}
}
```

做的工作：

1. 在`didUpdate(to:)`方法里，保存传入的`Weather`对象。
2. 如果你显示的展开后的天气，`numberOfItems()`返回5个cell来显示天气的不同部分数据。如果没展开，你仅需要显示占位的单个cell。
3. 第一个cell比其他的大，显示为一个头块。这里无需检查`expanded`状态，因为所有情况下，头块都是第一个cell。

接下来你需要实现`cellForItem(at:)`来配置天气cell。下面是需求的细节：

* 第一个cell必须是`WeatherSummaryCell`类型，其他的是`WeatherDetailCell`类型。
* 用`cell.setExpanded(_:)`配置天气概要cell。
* 用不同的标题和天气字段配置4个不同天气细节cell：
  * "Sunrise"和`weather.sunrise`
  * "Sunset"和`weather.sunset`
  * "High"和`\(weather.high) C`
  * "Low"和`\(weather.low) C`

给出设置cell的快照。解决方案如下：

```swift
override func cellForItem(at index: Int) -> UICollectionViewCell {
	let cellClass: AnyClass = index == 0 ? WeatherSummaryCell.self : WeatherDetailCell.self
	let cell = collectionContext?.dequeueReusableCell(of: cellClass, for: self, at: index)
	if let cell = cell as? WeatherSummaryCell {
		cell.setExpanded(expended)
	} else if let cell = cell as? WeatherDetailCell {
		let title: String, detial: String
		switch index {
		case 1:
			title = "SUNRISE"
			detail = weather.sunrise
		case 2:
			title = "SUNSET"
			detail = weather.sunset
		case 3:
			title = "HIGH"
			detail = "\(weather.high) C"
		case 4:
			title = "LOW"
			detail = "\(weather.low) C"
		default:
			title = "n/a"
			detail = "n/a"
		}
		cell.titleLabel.text = title
		cell.detailLabel.text = detail
	}
	return cell
}
```

最后一件事就是在点击时切换section的`expanded`。重写来自于`ListSectionController`的另一个方法：

```swift
override func didSelectItem(at index: Int) {
	collectionContext?.performBatch(animated: true, updates: { batchContext in
		self.expanded.toggle()
		batchContext.reload(self)
	}, completion: nil)
}
```

`performBatch(animated:updates:completion:)`在一个交易中批量处理section中的更新。你可以像这样在section控制器中处理内容或者单元格数量的变化。因为你在`numberOfItems()`中使用了切换后的展开状态，这会基于`expanded`标志添加或者删除单元格。

回到**FeedSectionController.swift**在头部附近添加一个属性：

```swift
let wxScanner = WxScanner()
```

`WxScanner`是天气情况的模型。

接着，更新`ListAdapterDataSource`扩展中的方法`objects(for:)`:

```swift
// 1
var items: [ListDiffable] = [wxScanner.currentWeather]
items += loader.entries as [ListDiffable]
items += pathfinder.messages as [ListDiffable]
// 2
return items.sorted { (left: Any, right: Any) -> Bool in
	guard let left = left as? DateSortable, let right = right as? DateSortable else {
		return false
	}
	return left.date > right.date
}
```

你必须更新数据源来包含天气数据`currentWeather`。下面是具体说明：

1. 添加`currentWeather`到items数组。
2. 所有的数据都符合`DateSortable`协议，所以这里就利用协议来排序。这样就确保了数据按时间排序。

最后，更新`listAdapter(_:sectionControllerFor:)`如下：

```swift
if object is Message {
	return MessageSectionController()
} else if object is Weather {
	return WeatherSectionController()
} else {
	return JournalSectionController()
}
```

这里在出现一个`Weather`对象时，返回一个`WeatcherSectionController`。

再次构建并运行。你会看到顶部的天气对象。点击顶部section展开或者收起它。

![IGListKit](https://koenig-media.raywenderlich.com/uploads/2016/11/Simulator-Screen-Shot-Nov-6-2016-1.00.22-AM-281x500.png)

### 处理变更

![img](https://koenig-media.raywenderlich.com/uploads/2018/11/people_astronaut-320x320.png)

JPL赞叹你的进度！

在你工作的同时，NASA指挥协调了宇航员的救援行动，要求他发射并拦截另一架飞船！这次发射有很多困难，所以他们必须在精确的时间发射。

JPL工程师扩展了消息模块，增加了实时聊天，并要求你合并这个功能。

打开**FeedViewController.swift**添加下面的行到`viewDidLoad()`当中：

```swift
pathfinder.delegate = self
pathfinder.connect()
```

`Pathfinder`模块打包了支持实时功能。你要做的就是连接这个单元，并响应代理事件。

在文件最后添加扩展：

```swift
// MARK: - PathfinderDelegate
extension FeedViewController: PathfinderDelegate {
	func pathfinderDidUpdateMessages(pathfinder: Pathfinder) {
		adapter.performUpdates(animated: true)
	}
}
```

`FeedViewController`现在符合了`PathfinderDelegate`。`performUpdates(animated:)`用来告知`ListAdapter`向数据源请求新对象然后更新UI。这里处理包括：删除、更新、移动或者插入。

构建并运行就可以看到船长的消息更新了！所有你需要做的就是添加一个方法探测数据源变化，当数据到达时动画显示。

![IGListKit](https://koenig-media.raywenderlich.com/uploads/2016/11/realtime.gif)

你现在要做的就是传送最新的构建给宇航员，他就要回家了。干的漂亮！

### 接下来干什么

你可以下载项目的[完整版本][2]。

除了带滞留的宇航员回家，你还学习了IGListKit的很多功能：section控制器，适配器、还有怎样把它们组合到一起。IGListKit还有支持视图、显示事件等其他重要的功能。

你可以阅读查看更多Instagram中IGListKit起源[Realm][8]的话题。这些话题涉及大量通用的`UICollectionView`问题，这些问题来自于他们很多大型复杂app当中的经验。

如果你想为IGListKit出份力，IGListKit团队在GitHub上设置了一些[新手任务][9]标签，你可以从这里开始。





[1]: https://github.com/Instagram/IGListKit "IGListKit"
[ 2 ]: https://koenig-media.raywenderlich.com/uploads/2019/01/MarsLink_Materials.zip "MarsLink"
[ 3 ]: https://www.raizlabs.com/dev/2014/02/animating-items-in-a-uicollectionview/ "here"
[ 4 ]: https://cocoapods.org/ "CocoaPods"
[ 5 ]: https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift "bridging header"
[ 6 ]: https://instagram.github.io/IGListKit/getting-started.html#working-range "working range"
[ 7 ]: https://github.com/Instagram/IGListKit/blob/master/Guides/Getting%20Started.md#diffing "custom model diffing"
[ 8 ]: https://realm.io/news/tryswift-ryan-nystrom-refactoring-at-scale-lessons-learned-rewriting-instagram-feed/ "Realm"
[ 9 ]: https://github.com/Instagram/IGListKit/issues?q=is%3Aissue+is%3Aopen+label%3Astarter-task "starter-task"