---
title: 練習在 WPF 中加入 Entity Framework 資料繫結建立簡單的資料應用程式
published: true
---

參考教學：https://docs.microsoft.com/zh-tw/visualstudio/data-tools/create-a-simple-data-application-with-wpf-and-entity-framework-6?view=vs-2017

## 建立資料庫與範例資料集

1. 在 Visual Studio 中開啟 SQL Server 物件總管視窗。 (SQL Server 物件總管安裝的一部分資料儲存和處理中的工作負載 Visual Studio 安裝程式。)依序展開 SQL Server 節點。 以滑鼠右鍵按一下您的 LocalDB 執行個體，然後選取新的查詢。
2. 查詢編輯器視窗隨即開啟。
3. 複製 Northwind 的 TRANSACT-SQL 指令碼到剪貼簿。 這個 T-SQL 指令碼會從頭建立 Northwind 資料庫，並填入資料。
4. 將 T-SQL 指令碼貼到查詢編輯器，然後選擇 Execute 按鈕。
5. 短時間之後，查詢完成執行，並建立 Northwind 資料庫。
6. 加入新連接 northwind。

![](assets/20190224/20190224-1.png)
![](assets/20190224/20190224-2.png)

## 專案

1. 準備好一個 C# WPF 專案後，接下來，針對 Entity Framework 6 中新增 NuGet 套件。 在 [方案總管] 中，選取 [專案] 節點。 在主功能表中，選擇專案 &gt; 管理 NuGet 套件 &gt; 管理 NuGet 套件 功能表項目

![](assets/20190224/20190224-3.png)

2. 在 NuGet 套件管理員，按一下瀏覽連結。 Entity Framework 可能是最上層的套件清單中。 按一下 安裝右窗格中，並遵循提示。 [輸出] 視窗會告訴您完成安裝。
   Entity Framework NuGet 套件

![](assets/20190224/20190224-4.png)

3. 現在您可以使用 Visual Studio 來建立模型，根據 Northwind 資料庫。

## 建立模型

1. 在方案總管的專案節點上按一下滑鼠右鍵，然後選擇 加入 > 新項目。 在左窗格中，在 C#節點，選擇資料，然後在中間窗格中，選擇 ADO.NET 實體資料模型。

![](assets/20190224/20190224-5.png)

2. 將此模型命名為 Northwind_model，然後選擇 確定。

![](assets/20190224/20190224-6.png)

3. [實體資料模型精靈] 隨即開啟。 選擇資料庫的 EF Designer ，然後按一下下一步。

![](assets/20190224/20190224-7.png)

4. 在下一個畫面中，選擇您的 LocalDB Northwind 連接，然後按一下 [下一步]。
5. 在精靈的下一個頁面中，選擇哪些資料表、 預存程序，以及要包含在 Entity Framework 模型中其他資料庫物件。 展開樹狀檢視中的 [dbo] 節點，然後選擇 Customers、Orders、Order Details。 保留選取預設值，然後按一下完成。

![](assets/20190224/20190224-8.png)

精靈會產生 C#表示 Entity Framework 模型的類別。 類別是純舊 C#類別，而且它們是我們繫結至 WPF 使用者介面。 .Edmx 檔案會描述關聯性和其他中繼資料與資料庫中的物件產生關聯的類別。 .Tt 檔案會產生運作模型，並將變更儲存到資料庫的程式碼的 T4 範本。 您可以看到所有這些檔案中的方案總管 中 Northwind_model 節點下：

![](assets/20190224/20190224-9.png)

設計工具介面 .edmx 檔案可讓您修改一些屬性和模型中的關聯性。 我們不會在此使用設計工具。

6. `.Tt` 檔案是 general purpose，您需要調整其中一個，才能使用 WPF 資料繫結(which requires ObservableCollections)。在 方案總管，展開 Northwind_model 節點，直到您找到 Northwind_model.tt，開啟這個檔案以查看與編輯以下其中一個：

- Replace the two occurrences `ICollection` (line 303, 491)with `ObservableCollection<T>`。
- Replace the first occurrence `HashSet<T>` with `ObservableCollection<T>` around line 51. Do not replace the second occurrence fo HashSet(around 606).
- Replace the only occurrence of `System.Collections.Generic` (arond line 431) with `System.Collections.ObjectModel`.

(有碰過只改第三個還是會建置失敗的狀況，最安全的做法是三個都改)

7. 按下 Ctrl+Shift+B 來建置專案。 When the build finishes, the model classes are visible to the data sources wizard.

![](assets/20190224/20190224-10.png)

現在您已準備好將連結至 XAML 頁面的這個模型，讓您可以檢視、 巡覽及修改資料。

## Databind the model to the XAML page

1. 從主功能表中，選擇專案 > 新增新的資料來源以顯示資料來源組態精靈。

![](assets/20190224/20190224-11.png)

2. 選擇 物件，因為您要繫結模型類別，不是針對資料庫：

![](assets/20190224/20190224-12.png)

3. 選取 客戶。 （Sources fro Orders are automatically generated from the Orders navigation property in Customer.）

![](assets/20190224/20190224-13.png)

4. 按一下 `[完成]`。

![](assets/20190224/20190224-14.png)

此時就已經可以在 MainWindow.cs 中使用 Model class 了

![](assets/20190224/20190224-15.png)

5. 回到 MainWindow.xaml 的 CodeView，把以下內容貼到 `<Grid></Grid>` 中

```xml
<Grid.RowDefinitions>
  <RowDefinition Height="auto"/>
  <RowDefinition Height="auto"/>
  <RowDefinition Height="*"/>
</Grid.RowDefinitions>
```

6. 按下 shift + alt + D 叫出 Data Source，按一下 Customers 右方倒三角按鈕，點選 詳細資料

![](assets/20190224/20190224-16.png)

7. 接著再點一下 Customers，但是按著他拖曳到 MainWindow.xaml 的圖形化視窗中，此時精靈就會自動加入一個編輯元件

![](assets/20190224/20190224-17.png)

8. 試試在程式碼中以 ORM 方式存取客戶資料

![](assets/20190224/20190224-18.png)

9. 按下 F5 執行看輸出

![](assets/20190224/20190224-19.png)
