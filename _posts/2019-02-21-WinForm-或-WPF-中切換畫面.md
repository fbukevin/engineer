---
title: WinForm 或 WPF 中切換畫面
published: true
---

Windows 的視窗程式最陽春的擴充視窗空間方式是跳出新的視窗(sub-window)，如果想要在同一個視窗中做切換要怎麼做呢? 參考這個影片[教學](https://www.youtube.com/watch?v=NwfHAsq5GHw&t=356s)，這裡我用 WPF demo

建立一個空 WPF 專案後，在 MainWindow 中加入一個 frame，名稱改為 mainFrame，屬性 NavigationUIVisibility 設為 Hidden，Content 建議設為空

![](assets/20190221/20190221-1.png)

把 frame 的配置 (layout) 設定為全部填滿 (full area)

![](assets/20190221/20190221-2.png)

在專案中新增兩個 頁面(WPF)，英文是 Page，名稱分別是 Page1.xaml 和 Page2.xaml，並在其中各加入一個 label 和 button
![](assets/20190221/20190221-3.png)
![](assets/20190221/20190221-4.png)

## 撰寫控制程式碼

**MainWindow.xaml.cs**

```csharp
public MainWindow()
{
    InitializeComponent();

    // 設定初始化載入的 Page 為 Page 1
    Page1 p1 = new Page1();
    mainFrame.NavigationService.Navigate(p1);
}
```

**Page1.xaml.cs**
雙點擊按鈕以便自動建立 click 事件

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
    Page2 p2 = new Page2();
    this.NavigationService.Navigate(p2);
}
```

**Page2.xaml.cs**
雙點擊按鈕以便自動建立 click 事件

```cs
private void Button_Click(object sender, RoutedEventArgs e)
{
    Page1 p1 = new Page1();
    this.NavigationService.Navigate(p1);
}
```

![](assets/20190221/20190221-5.gif)

接著就可以執行看看效果了：
![](assets/20190221/20190221-6.gif)

如果前面沒有隱藏 NavigationUIVisibility
