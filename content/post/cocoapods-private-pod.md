+++
date = "2016-04-25T08:43:13+08:00"
title = "CocoaPods - 私有套件"
tags = ["cocoapods", "ios"]
description = "使用CocoapPods管理私有套件"

+++

[CocoaPods][cocoapods]除了可以用來下載和管理第三方套件之外，也能用來管理私有的套件。假設在一個龐大的專案中，功能套件與主要程式分別由不同的人開發，或不同的App必需使用相同的套件的時候，將套件製作成一個pod，便能夠利用[CocoaPods][cocoapods]非常方便地管理。

----
### 文章內容

- 建立私有的pod
- 在其他專案中使用本地的pod
- 將pod利用Git上傳到伺服器（[GitHub][github], [BitBucket][bitbucket],......）
- 在其他專案中使用伺服器上的pod

### 適用對象

- 有使用[CocoaPods][cocoapods]及Git經驗的開發者

### 實作環境

- Macbook Pro Retina, 13″, Early 2015
- OS X El Capitan, 10.11.4

----
### 在本機建立pod專案

打開Terminal，cd進入準備建立專案的路徑

```
cd /Users/hao/Documents/MyPods/
```

建立pod專案

```
pod lib create myFirstPod
```

這時[CocoaPods][cocoapods]會幫你從官方的Git複製一個範本，接下來要回答幾個問題，可以依照自己的情況填寫

```
What language do you want to use?? [ Swift / ObjC ]
> Swift
Would you like to include a demo application with your library? [ Yes / No ]
> Yes
Which testing frameworks will you use? [ Quick / None ]
> Quick
Would you like to do view based testing? [ Yes / No ]
> Yes
```

設定完成後專案會自動開啓，這時基本上就是一個裡面沒有檔案的pod專案，接下來就要對專案做一些設定。如果沒有自動開啓，請到以下路徑開啓

```
myFirstPod/Example/myFirstPod.xcworkspace
```

### 設定pod專案

在專案中，有幾個資料夾是我們在開發pod時會用到，這裡先大略介紹一下，之後會有開發pod的實作範例。

- myFirstPod/Podspec Metadata
    - myFirstPod.podspec：這是管理pod最重要的檔案，在其他專案中使用pod時，CocoaPod會依據.podspec中的資料下載對應的版本及其他相關的pod，我在這裡標注幾個比較需要更改的部份

```
#
# Be sure to run`pod lib lint myFirstPod.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see http://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = "myFirstPod"
  s.version          = "0.1.0"
  s.summary          = "This is my first pod."

  # This description is used to generate tags and improve search results.
  #   * Think: What does it do? Why did you write it? What is the focus?
  #   * Try to keep it short, snappy and to the point.
  #   * Write the description between the DESC delimiters below.
  #   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <MIT（MIT是open source的認證，一般來說pod都是開源軟體，目前我們的pod不會被其他人使用，就先不處理認證的問題）'
  s.author           = { "Hao" => "hao@hao.com" }
  s.source           = { :git =>     "https://github.com//myFirstPod.git（稍後介紹使用Git的時候再一併說明）", :tag =>   s.version.to_s }

  # s.social_media_url = 'https://twitter.com/'
  s.ios.deployment_target = '8.0'
  s.source_files = 'myFirstPod/Classes/**/*'
  s.resource_bundles = {
  'myFirstPod' => ['myFirstPod/Assets/*.png']
  }  
  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  s.dependency 'Alamofire'（如果同時需要使用其他的pod，必需要在這裡加入）

  end
```
- README.md、LICENSE：在公開的pod上，README會簡介和教導pod如何使用，LICENSE則是清楚的表明這份pod的智慧財產權，目前我們都是私有的所以可以暫時略過。
- myFirstPod/Example for myFirstPod：這是自動產生的範例程式，如果有開發iOS App的經驗，對其中的檔案應該不陌生，可以在裡面的class中`import myFirstPod`來測試pod的功能。
- Pods/Development Pods/myFirstPod/myFirstPod/Classes：這裡就是主要開發套件的目錄，會有一個預設檔案ReplaceMe.swift，我們開發的類別都會存在這個資料夾裡。

### 開發過程

接下來我們實作一個專案開發的過程，模擬一下真正開發pod時的流程，首先假設我們需要開發一個使用SVProgressHUD的套件，我們必須先在myFirstPod中引用SVProgressHUD，然後再到Example的範例程式中引用myFirstPod。

#### 加入SVProgressHUD套件

首先我們到`myFirstPod.podspec`檔案裡，加入

```
s.dependency 'SVProgressHUD'
```

然後先關閉Xcode視窗，開啓Terminal，輸入

```
cd /Users/hao/Documents/MyPods/myFirstPod/Example
```

進入myFirstPod目錄底下的Example資料夾，再使用指令

```
pod install
```

或
```
pod update
```

這時你會看到[CocoaPods][cocoapods]正在幫你安裝SVProgressHUD，安裝完成時就可以在專案中使用了。

#### 新增類別

接下來我們到`Pods/Development Pods/myFirstPod/myFirstPod/Classes`的目錄下，把`ReplaceMe.swift`刪除，新增一個`TestClass.swift`，特別注意要在Targets的地方勾選myFirstPod。

![new-file]

在`TestClass.swift`中輸入

```
import Foundation
import SVProgressHUD

public class TestClass{
  public init(){}

  func show(){
    SVProgressHUD.setDefaultMaskType(.Black)
    SVProgressHUD.show()
  }

  public func publicShow(){
    show()
  }
}
```

先按`⌘B`來Build專案可以解決在引入SVProgressHUD時現的錯誤，另外必須要加入public的類別（變數、方法）才有辦法被引用的專案呼叫。

#### 範例程式

接下來到`myFirstPod/Example for myFirstPod`目錄底下的`ViewController.swift`，新增以下的程式碼

```
//
//  ViewController.swift
//  myFirstPod
//
//  Created by Hao on 4/25/16.
//  Copyright © 2016 Hao. All rights reserved.
//
import UIKit
import myFirstPod

class ViewController: UIViewController {
  let a = TestClass()
  override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
  }
  override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    // Dispose of any resources that can be recreated.
  }

  override func viewDidAppear(animated: Bool) {
    super.viewDidAppear(animated)
    a.publicShow()
  }
}
```
接下來我們就可以開啓虛擬機來測試了，如果沒有什麼意外，我們應該會看到

![svprogress-demo]

### 在其他專案中使用本地端的myFirstPod

好不容易完成開發，別忘了我們最開始的目的是在不同的專案中使用pod，所以現在終於可以開始做正事了，讓我們開啓一個隨便的專案，還記得怎麼在專案中使用pod嗎？如果不記得可以參考之前的文章CocoaPods 入門，在`Podfile`中加入

```
pod 'myFirstPod', :path => '~/Documents/MyPods/myFirstPod'
```

再使用Terminal執行

```
pod install
```

完成之後就可以跟其他的pod一樣引用本地端的`myFirstPod`。

這時候你可能會發現，在Xcode左邊的目錄裡，`myFirstPod`並不是跟其他的pod一樣放在`Pods`目錄底下，而是有另一個`Development Pods`的目錄，這跟我們在`myFirstPod`的專案裡情況非常類似，如果你再細細觀察`myFirstPod/Example/Podfile`這個檔案，你會發現裡面有

```
pod 'myFirstPod', :path => '../'
```

基本上就是引用了本地端的`myFirstPod`，因此就像在`myFirstPod`的專案裡一樣，我們在別的專案中，如果更改了`myFirstPod`的檔案，會直接更改原始的檔案。

### 在伺服器上建立套件的Git資料庫

把套件跟專案分離，做成pod最大的好處就是可以讓不同的人在不同的專案使用，假設我們在某個專案中開發了一份iOS的API，另一個專案的開發人員也需要使用同一份API，我們就在Server上開啓一個Git資料庫，再把pod專案上傳，就可以很容易地讓其他人一起使用，也能利用[CocoaPods][cocoapods]來做版本管理。

在伺服器上建立Git資料庫，可能是[GitHub][github]、[BitBucket][bitbucket]⋯⋯
開啓Terminal，cd到myFirstPod的資料夾底下

```
cd /Users/hao/Documents/MyPods/myFirstPod
git add .
git commit -m 'Initial commit'
git tag 0.1.0
```

在myFirstPod裡的Git在我們之前建立專案時就已經自動建立了，我們在這裡加入所有的檔案，然後commit並加上個標籤0.1.0，[CocoaPods][cocoapods]會自動用tag來做版本控制，這由要特別注意tag上的版本必須要跟podspec檔案中的版本一致。

```
s.version          = "0.1.0"
```

繼續使用Terminal

```
git remote add origin git@XX.XXX.X.XX:/myFirstPod.git
git push --set-upstream origin master
```

這裡我們加入一個遠端的Git資料庫，並把剛剛的更改push上資料庫，這裡的範例是使用自己的server，當然你可以用[GitHub][github]或是[BitBucket][bitbucket]，例如

```
git remote add origin https://hao@bitbucket.org/hao/myFirstPod.git
git push -u origin master
```

別忘了同時也要修改`podspec`裡的設定（實際測試時，這部份不改其實也可以使用，因為我們在`Podfile`裡就利用`:path`或`:git`定義了來源，但如果是發佈在[CocoaPods][cocoapods]上的pod，這裡就必須確實更改，通常公開的pod會使用[GitHub][github]資料庫）

```
s.source = { :git => "https://github.com/hao/myFirstPod", :tag => s.version.to_s }
```

### 在其他專案中使用遠端的myFirstPod

這時候讓我們再度回到那個隨便的專案，開啓`Podfile`，把剛剛的引入pod的程式碼改成

```
pod 'myFirstPod', :git => 'git@XX.XXX.X.XX:/myFirstPod.git'
```

然後再開啓Terminal，執行

```
pod install
```

就可以快樂地使用自己的pod啦！

使用 :git 預設是使用資料庫裡的master branch，除此之外，在`Podfile`裡還可以有更多進階的設定，例如

```
pod 'myFirstPod', :git => 'git@XX.XXX.X.XX:/myFirstPod.git', :branch => 'master'
pod 'myFirstPod', :git => 'git@XX.XXX.X.XX:/myFirstPod.git', :tag => '0.2.1'
pod 'myFirstPod', :git => 'git@XX.XXX.X.XX:/myFirstPod.git', :commit => '000x0000xx'
```

現在就可以把已經做好的套件，做成pod放在伺服器上，利用[CocoaPods][cocoapods]來管理套件，就不會每次更新library都是一場華麗又心驚膽顫的冒險，版本管理也更方便了。

----

### 參考資料

- [CocoaPods][cocoapods]
- [GitHub][github]
- [Bitbucket][bitbucket]


[cocoapods]: https://cocoapods.org
[github]: https://github.com
[bitbucket]: https://bitbucket.org
[new-file]: /images/post/cocoapods-private-pod/new-file.png
[svprogress-demo]: /images/post/cocoapods-private-pod/svprogress-demo.png
