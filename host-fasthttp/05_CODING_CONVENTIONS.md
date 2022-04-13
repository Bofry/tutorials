<link href="tutorial.css" rel="stylesheet" />

**March 2022**  |  **v1.0**

[↩ 回首頁](README.md)

05 開發慣例
================================
🎯 本節使用在章節 «[03 Translate 範例](03_TRANSLATE_EXAMPLE.md)» 中所實作的 `API` 內容。你將會掌握：

  1. 專案目錄結構的規範。
  2. 如何找尋專案服務對應的規格文件放置位置。
  3. 如何閱讀專案服務中的規格內容。

----------------------------------------------------------------
#### 目錄
  - [專案目錄結構與命名規範](#專案目錄結構與命名規範)
  - [文件目錄結構與規格範例](#文件目錄結構與規格範例)
----------------------------------------------------------------

### 專案目錄結構與命名規範

⠿ 依照慣例，每個專案資料夾內，應該至少會有這些資料夾與檔案配置。並遵守以下開發慣例。

  - **檔案配罝慣例：**
    ```
    📄 app.go
    📄 go.mod
    📁 internal/
      📄 appContext.go
    📁 model/
      📄 postTransferResult.go
    📁 resource/
      📄 translateResource.go
      📁 args/
        📄 postTranslateArgs.go
    📁 provider/
      📄 googleTranslateProvider.go
    ```
    **檔案配置說明：**

    - [**app.go**](#file-appgo)

      應用程式的進入點。這個檔案定義應用程式的<span class="underline">**程式進入點**</span> `main()`，依據要啟動的 *AppContext* 類型，以及應程式的中介元件 (*middleware*)，進行初始化，並於啟動後開始接收請求。

      > 📌 *Golang* 的應用程式進入點為 `main()` 函式。

      > 📝 *AppContext* 型別指的是 *application context* （應用程式工作環境），通常為一個型別或子模組，這個元件提供應用程式初始化時期，所需要的基礎資源與操作：比如服務引擎、網路協定、應用程式設定值……等。

      > 📝 中介元件（*middleware*）是設計用來提供應用程式增強基礎功能的擴充元件，開發者可視需要添加既有的中介元件或自行實作所需元件。

    - **go.mod**

      `go mod init` 指令所產生的模組管理設定檔。

      > 🐾 有關 `go mod init` 的用法與說明參考：https://golang.org/ref/mod#go-mod-init

    - 📁 **internal/**

      *AppContext* 相關的應用程式核心檔案皆配置於此目錄。

      - [**appContext.go**](#internalappcontextgo)

        應用程式 *AppContext* 型別相關程式碼。

    - 📁 **model/**

      放置共用的模型物件目錄。

      - [**postTransferResult.go**](#)

        第三方資源或服務 *repository* 型別相關程式碼回應的模型物件。

    - 📁 **resource/**

      放置處理 Web API *url path* 請求的 *Resource* 的目錄。

      - [**translateResource.go**](#)

        提供範例使用的 /Translate 呼叫處理的 Resource。

    - 📁 **resource/args/**

      放置處理 Web API *url path* 請求輸入參數 *Args* 物件的子目錄。

      - [**postTranslateArgs.go**](#)

        提供範例使用的 /Translate 呼叫處理的的輸入參數 Args。

    - 📁 **provider/**

      放置實作第三方資源或服務的 *Provider* 的目錄。

      - [**googleTranslateProvider.go**](#)

        提供範例使用第三方的 `Translate` 呼叫處理的 Provider。

  - **命名慣例：**
    1. 進入點檔案一律命名 **app.go**。
    2. **internal** 目錄內，配置專案應該程式所需的基礎核心元件。
    3. 應用程式所使用的**AppContext**，以 `AppContext` 來命名。
        > 🔍 目前在在若干早期專案仍可看到以 `App` 作為 **AppContext** 來命名。這些專案將會在未來陸續更新。
    4. **model** 目錄內，配置作為共用的 *http method* 加 *url path* 的 *Result* 。
    5. **resource** 目錄內，配置作為處理 *url path* 的 *Reource*。
    6. **resource/args** 目錄內，配置作為處理請求輸入參數 *http method* 加 *url path* 的 *Args*。
    7. **repository** 目錄內，配置作為處理第三方資源或服務的 *Repository*。

[🔝回目錄](#%e7%9b%ae%e9%8c%84)


----------------


## 文件目錄結構與規格範例

⠿ 一般情況下，每個專案皆會使用 `Git` 用於做版本控制，而專案內的目錄組成規範如下：
    
1. 每個專案根目錄都會有 `doc` 的目錄，用於存放服務的規格文件、設計文件...等等。
目錄結構範例：
  ```
   📁 DemoService
   📁 doc
     📄 DemoService_API_Specification.md
  ```

2. DemoService API 規格的範例，如下：

-----------------------------------------------------------
#### 目錄
- [API 規格](#api-規格)
  - /Translate 翻譯相關
      - [Get](#get-translate) : 取得翻譯結果
-----------------------------------------------------------

#### API 規格

##### Get /Translate
_取得字串翻譯結果。_

  **路徑**: `/Translate`

  **請求**:
  - **查詢字串**:
    - **SOURCE_LANGUAGE** `required`  : 指示進行當前翻譯內容的語言。    
          
      > ※ 輸入列舉值為 [各國語言代碼表`]()

    - **TARGET_LANGUAGE** `required`  : 指示進行當前翻譯結果的語言。
  - **標頭**:
    - **Content-Type**: `application/json; charset=utf-8`
  - **參數**:
    - **content** `required` : 翻譯內容。

  **回應**:
  - **標頭**:
    - **Content-Type**: `text/plain; charset=utf-8`
  - **欄位**:
    - **translation** : 翻譯結果。

  **錯誤**:

  **範例**:

<details>
  <summary><b>用例: 中文翻譯英文操作 </b></summary>

  - **請求**
    ```json
    GET /Translate?SOURCE_LANGUAGE=zh-CN&TARGET_LANGUAGE=en  HTTP/1.1
    Host: <server>
    Content-Type: application/json; charset=utf-8

    {
      "content": "說明"
    }
    ```

  - **回應**
    ```json
    200 OK
    Content-Type: text/plain; charset=utf-8

    {
      "translation": "illustrate"
    }
    ```
</details>

[🔝回目錄](#目錄)


----------

