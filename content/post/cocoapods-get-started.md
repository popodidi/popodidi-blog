+++
date = "2016-04-23T11:46:30+08:00"
tags = ["cocoapods", "ios"]

title = "CocoaPods - 入門"
description = "CocoaPods入門"

+++
### 文章內容

- [CocosPods][cocoapods]簡介
- [CocosPods][cocoapods]安裝
- 匯入及使用第三方套件（pod）

### 適用對象

- iOS/OS X App開發者

### 實作環境

- Macbook Pro Retina, 13”, Late 2013
- OS X El Capitan, 10.11.3

----

### 什麼是[CocosPods][cocoapods]？

[CocosPods][cocoapods]最主要的功能就是管理Xcode專案中的第三方套件。在App開發專案中，一般並不會從頭到尾開發每個功能，更多的時候我們會使用好用的第三方套件，例如用來做Http Request的[Alamofire][alamofire]、產生讀取動畫的[SVProgressHUD][svprogresshud]⋯⋯，節省開發成本和時間，也能更專注於開發產品的核心功能，而[CocosPods][cocoapods]就是針對Xcode開發者設計的第三方套件管理軟體，可以讓開發者輕鬆地在專案中匯入不同的第三方套件，[CocosPods][cocoapods]上也累積了許多好用的第三方套件，大部分都是開源軟體，使用MIT授權（The MIT License），簡單來說，MIT授權的開源軟體不限制使用者對程式碼做任何事，包括使用、複製、修改或發行，因此我們可以很放心地在專案中使用這些套件。

### 安裝CocoaPods

[CocosPods][cocoapods]是使用Ruby寫成的第三方套件，在Mac中就內建有Ruby和gem（Ruby的套件管理軟體），所以我們並不需要額外安裝，所以我們要做的就是打開Terminal，再輸入

```
sudo gem install cocoapods
```

`sudo`代表以管理者權限，使用`gem`安裝`cocoapods`套件，接下來會有個輸入密碼的提示，輸入之後就會開始安裝（這個步驟可能會有點緩慢，就放著等一下，千萬不要以為是當機就把他關了），安裝完成之後別急著把Terminal關掉，我們等等還會用到它。

### 在專案中匯入pod套件

首先，開啓一個新的iOS空白專案，當然也可以使用原本的專案（如果你不怕把他玩壞的話），然後開啓你的Terminal，假設你的專案開在文件資料夾裡，就輸入

```
cd /Users/hao/Documents/myProject
```

`cd`就意思就是到達以下的路徑，接下來我們要再輸入

```
pod init
```

這個指令會在`myProject`資料夾底下產生一個`Podfile`的檔案，這個就是主要用來管理匯入套件的檔案，現在用一個你喜歡的文字編輯器打開他，我大部份都使用Xcode，這時候可以再來練習一下使用Terminal，輸入

```
open -a Xcode Podfile
```

打開之後我們會看到

```
# Uncomment this line to define a global platform for your project
# platform :ios, '8.0'
# Uncomment this line if you're using Swift
# use_frameworks!

target 'myProject' do
end
target 'myProjectTests' do
end
target 'myProjectUITests' do
end
```

假設我們的專案是支援iOS 8以上並且使用Swift開發，就把最前面改成

```
# Uncomment this line to define a global platform for your project
platform :ios, '8.0'
# Uncomment this line if you're using Swift
use_frameworks!
```

然後匯入我們的第一個套件[SVProgressHUD][svprogresshud]，把檔案下方改成

```
target 'myProject' do
  pod 'SVProgressHUD'
end
target 'myProjectTests' do
end
target 'myProjectUITests' do
end
```

存檔後關閉，暫時把Xcode的視窗關閉，包括`myProject`的視窗也關閉，再度開啓Terminal，如果剛才沒有關的話，你應該會在同一個路徑底下，也可以跟之前一樣用`cd`指令到`Podfile`所在的路徑，輸入

```
pod install
```

這時候你就會看到顯示安裝`SVProgressHUD`，安裝完成之後在`myProject`的資料夾底下會多出幾個檔案

- Podfile.lock
- Pods資料夾
- myProject.xcworkspace

其中`myProject.xcworkspace`就是我們之後開發會開啓的檔案，我們馬上就可以把他打開看看，或是你是Terminal狂人，也可以輸入

```
open -a Xcode myProject.xcworkspace
```

打開之後你會發現，除了原本的`myProject`之後還多了一個`Pods`專案，但基本上你可以完全不要理會他，這時候我們已經可以在專案中使用新匯入的套件了！

### 使用SVProgressHUD套件

[SVProgressHUD][svprogresshud]是一個產生讀取動畫的套件，我們就在`ViewController.swift`中，加入以下的程式碼

```
//
//  ViewController.swift
//  myProject
//
//  Created by Hao on 4/23/16.
//  Copyright © 2016 Hao. All rights reserved.
//
import UIKit
import SVProgressHUD

class ViewController: UIViewController {
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
    SVProgressHUD.setDefaultMaskType(.Black)
    SVProgressHUD.show()
  }
}
```

如果在`import`的時候出現錯誤的話，可以先按`⌘B`讓他Build，上面一切都準備好就可以讓專案跑起來試試看囉，這時候如果一切順利的話，你應該會看到

![svprogress-demo]

一般來說，我們會把[SVProgressHUD][svprogresshud]使用搭配GDC來使用，在main queue裡處理UI，在global queue執行其他的工作，例如

```
SVProgressHUD.show()
dispatch_async(dispatch_get_global_queue(QOS_CLASS_USER_INITIATED, 0)) {
  //Do you tasks here
  //Downloading, calculating, parsing something... etc

  dispatch_async(dispatch_get_main_queue()) {
    //When tasks are done
    SVProgressHUD.dismiss()
  }
}
```

在上面的程式碼中，我們先開始[SVProgressHUD][svprogresshud]的動畫，然後進入global queue做其他的工作，當工作完成之後，再跳回main queue結束動畫。

更多的使用方法就可以參考每個pod的文件或網頁上的說明。

### Podfile
`Podfile`除了單純的指定使用的pod之外，也可以指定特定的版本和來源，例如

```
# 指定版本 0.1.0
pod 'onePod', '0.1.0'

# 指定版本 0.1.0，更新版本小於且不包含 0.2
pod 'onePod', '~> 0.1.0'

# 指定版本 0.1，更新版本小於且不包含 1.0
pod 'onePod', '~> 0.1'

# 指定git來源
pod 'onePod', :git => 'https://github.com/hao/ondPod.git'

# 指定本地端來源
pod 'onePod', :path => '/Users/hao/Documents/onePod'
```

還有更多進階的用法，可以參考官方的說明。

### 刪除pod
要刪除pod，只需要把`Podfile`裡引用的程式碼刪除或註解，像這樣

```
# Uncomment this line to define a global platform for your project
platform :ios, '8.0'
# Uncomment this line if you're using Swift
use_frameworks!

target 'myProject' do
# 就是把這裡刪掉
# pod 'SVProgressHUD'
end
target 'myProjectTests' do
end
target 'myProjectUITests' do
end
```

然後再打開Terminal執行

```
pod install
```

你就會看到Terminal裡顯示pod被刪除。

### 更新pod

每一個pod只有在第一次安裝的時候會執行下載，並根據`Podfile`裡的指示檢查版本，並不會自動更新版本，因此有需要時，我們必須手動更新pod的版本，這時候要再度打開Terminal，跟前面一樣用指令`cd`到`myProject`的資料夾底下，執行

```
pod update
```

你一樣會看在Terminal中顯示你更新了哪個pod，更新到什麼版本。

### install與update的差別

- install：只檢查是否下載，意思是下載`Podfile`裡新增的pod，刪除原本有下載但Podfile裡沒有的pod，但不檢查版本更新。
- update：只檢查是否有版本需要更新。

因此使用上，安裝新的pod就使用install指令，更新舊有的pod就用update指令，如此也可以避免在安裝新的pod時，因為更新而產生的版本不同，所造成的API錯誤。

如此一來，就能在[CocosPods][cocoapods]的網站上搜尋各式各樣的pod來幫助開發，常見的pod有前面提過的[Alamofire][alamofire]，還有用來Decode json的[SwiftyJSON][swiftyjson]，除了功能性的pod，當然也有像[SVProgressHUD][svprogresshud]這樣的UI套件，在使用pod之前，別忘了先觀察授權、文件的完整性和維護的頻率，下載數量也是不錯的參考，以免在使用後卻發現套件處處漏洞，或是已經停止更新，造成自己維護專案時的困難。

[CocosPods][cocoapods]除了下載第三方套件之外，也能用來管理自己製作的套件，例如在龐大的專案分工下，把API的函式庫做成pod，可以明確地管理API版本，也能方便地在不同的專案中使用相同的API，這部份的功能就留在接下來的文章中繼續分享囉！

----

### 參考資料

- [CocoaPods][cocoapods]

[cocoapods]: https://cocoapods.org
[alamofire]: https://github.com/Alamofire/Alamofire
[swiftyjson]: https://github.com/SwiftyJSON/SwiftyJSON
[svprogresshud]: https://github.com/SVProgressHUD/SVProgressHUD
[svprogress-demo]: /images/post/cocoapods-private-pod/svprogress-demo.png
