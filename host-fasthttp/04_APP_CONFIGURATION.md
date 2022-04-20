<link href="tutorial.css" rel="stylesheet" />

**March 2022**  |  **v1.0**

[↩ 回首頁](README.md)

04 設定檔配置
================================
🎯 本節使用 `Host.Init()` 以及 *config.yaml* 來配置 **Host** 服務各種執行環境設定。你將會掌握：

  1. 如何使用 `Host.Init()` 進行 Host 服務配置。
  2. 認識 Config、config.yaml 與 Host 有關的設定。
  3. 環境變數與多環境部署設定。
  4. 了解如何用不同的啟動方式來指定 Host 配置。

----------------------------------------------------------------
#### 目錄
  - [Host 服務配置](#host-服務配置)
  - [認識 Config、config.yaml 與 Host 有關的設定](#認識-configconfigyaml-與-host-有關的設定)
  - [多環境部署設定](#多環境部署設定)
  - [調整環境變數並啟動應用程式](#調整環境變數並啟動應用程式)
----------------------------------------------------------------

## Host 服務配置

⠿ 使用 *host-fasthttp* 開發的應用程式，所使用的 Host 引擎是以 [*valyala/fasthttp*](https://github.com/valyala/fasthttp) 為主體，這個引擎由 *host-fasthttp* 的 `Host` 型別包裝；同時 *host-fasthttp* 亦提供 `Init()` 方法，作為應用程式配置所需的服務設定，比如：通訊埠、服務監聽位址、服務名稱、是否啟用*HTTP*壓縮……等。

⠿ 我們可以在章節 «[01 使用 host-fasthttp 快速建構你的第一個 Web API](01_QUICK_START.md)» 的專案中的 **internal/appContext.go** 的內容找到 **Host.Init()** 方法中配置 Host 服務基礎設定。這個實作的方法，由已完成繫結的 **Config** 物件來，進行 **Host** 物件的配置。

> 💾 **internal/appContext.go**
> ```go
> func (h *Host) Init(conf *Config) {
>   h.Server = &fasthttp.Server{
>     Name:                          conf.ServerName,
>     DisableKeepalive:              true,
>     DisableHeaderNamesNormalizing: true,
>   }
>   h.ListenAddress = conf.ListenAddress
>   h.EnableCompress = conf.EnableCompress
>   h.Version = conf.Version
> }
> ```
>
> ⠿ **在 fasthttp.Host 中設定的欄位：**
> - **Server** `*fasthttp.Server`<br/>
>   繼承 fasthttp.Server 內容。提供相容於 *fasthttp.Server* 的相容功能。
> - **ListenAddress** `string`<br/>
>   應用程式啟動後的監聽位址與連接埠。
> - **EnableCompress** `bool`<br/>
>   應用程式是否啟用 HTTP gzip 壓縮傳送功能。
> - **Version** `string`<br/>
>   應用程式的版本號。
>
> --------
> 🐾 有關 `fasthttp.Server` 的欄位與說明參考：https://pkg.go.dev/github.com/valyala/fasthttp#Server
>
> ✋ <span class="remind">下面 *fasthttp.Server* 設定值<span class="underline">**不保證 *host-fasthttp* 的功能正常運作**</span>，若非得使用時需要特別注意。下列 *fasthttp.Server* 的設定盡量避免使用：</span>
> - `fasthttp.Server.Handler`<br/>
>   這個設定可能導致 *host-fasthttp* 的 **UseResourceManager**、**UseLogging**、**UseXHttpMethod**……等，涉及 *RouteTable* 相關的功能無作用。
> - `fasthttp.Server.ErrorHandler`<br/>
>   這個設定可能導致 **UseUnhandledRequestHandler** 的行為無法控制。

[🔝回目錄](#%e7%9b%ae%e9%8c%84)


----------------

## 認識 Config、config.yaml 與 Host 有關的設定

⠿ 有關 Host 服務的基礎設定，如：通訊埠、服務監聽位址、服務名稱、是否啟用*HTTP*壓縮……等；在章節 «[01 使用 host-fasthttp 快速建構你的第一個 Web API](01_QUICK_START.md)» 的專案中，分別定義於 **internal/appContext.go**、**config.yaml**、**config.local.yaml** 內容中。

> 💾 **internal/appContext.go**
> ```go
>   Config struct {
>     // host-fasthttp server configuration
>     ListenAddress  string `yaml:"address"        arg:"address;the combination of IP address and listen port"`
>     EnableCompress bool   `yaml:"useCompress"    arg:"use-compress;indicates the response enable compress or not"`
>     ServerName     string `yaml:"serverName"`
>     Version        string `resource:".VERSION"`
> 
>     ...
> ```
>
> ⠿ **在 Config 型別中找到與 Host 有關的欄位：**
> - **ListenAddress** `string`<br/>
>   應用程式啟動後的監聽位址與連接埠。這個值可以在 **config.yaml** 中經由指定 `address` 的名稱來設定。
>   
>    > 或是在啟動應用程式時，使用命令列參數 `-address` 設定。
> - **EnableCompress** `bool`<br/>
>   應用程式是否啟用 HTTP gzip 壓縮傳送功能。這個值可以在 **config.yaml** 中經由指定 `useCompress` 的名稱來設定。
>   
>    > 或是在啟動應用程式時，使用命令列參數 `-use-compress` 設定。
> - **ServerName** `string`<br/>
>   應用程式服務名稱，這個名稱會顯示在 HTTP回應標頭中。這個值可以在 **config.yaml** 中經由指定 `serverName` 的名稱來設定。
> - **Version** `string`<br/>
>   應用程式的版本號。這個值會直接讀取 **.VERSION** 檔案的內容。
>
> --------
> 💬 在 **Config** 使用 *golang struct tags* 來繫結 (*binding*) 設定來源，其中
>  - `yaml` 是指 *yaml* 檔案
>  - `arg` 指命令列參數
>  - `resource` 指資源檔案內容。


> 💾 **config.yaml**
> ```yaml
> address: ":80"
> serverName: WebAPI
> withCompress: true
> ```
>
> ⠿ **我們可以在 config.yaml 型別中找到與 Host 有關的名稱：**
> - **address** `string`<br/>
>   應用程式啟動後的監聽位址與連接埠。其設定格式為 `<host>:<port>` 。目前設定為監聽該主機連接埠為 `80` 的任何 *IP*，這個值會被設定到 `Config.ListenAddress`。
> - **useCompress** `bool`<br/>
>   應用程式是否啟用 HTTP gzip 壓縮傳送功能。目前設定為 `true` (啟用 *HTTP*壓縮)，這個值會被設定到 `Config.EnableCompress`。
> - **serverName** `string`<br/>
>   應用程式服務名稱，這個名稱會顯示在 HTTP回應標頭中。目前設定為 `WebAPI`，這個值會被設定到 `Config.ServerName`。

> 💾 **config.local.yaml**
> ```yaml
> address: ":10074"
> ```
>
> ⠿ **我們可以在 config.local.yaml 型別中找到與 Host 有關的名稱：**
> - **address** `string`<br/>
>   應用程式啟動後的監聽位址與連接埠。目前設定為監聽該主機連接埠為 `10074` 的任何 *IP*，這個值會被設定到 `Config.ListenAddress`。
>   
>    > 📝 由於<span class="underline">**設定檔載入順序機制**</span>，這個 `:10074` 將會覆蓋 config.yaml 的 `:80` 的值，成為 `Config.ListenAddress` 最後所定義的值。
>
> 🐾 有關設定檔載入順序機制將在下方 «[設定檔載入順序機制](#設定檔載入順序機制))» 進行更深入的說明。

> 🐾 在章節 «[01 使用 host-fasthttp 快速建構你的第一個 Web API](01_QUICK_START.md)» 的專案中並沒有提供 **.VERSION** 的檔案，由於這個檔案在開發過程非必要存在；若想要了解，可以在專案目錄下建立一個名為 **.VERSION** 的檔案，並填入下方內容。
>
> 💾 **.VERSION**
> ```
> v1.0.0-local
> ```
> 📌 內容僅一行即可，不可以包含行尾的 *\r\n* 等換行字元

[🔝回目錄](#%e7%9b%ae%e9%8c%84)


## 設定檔載入順序機制

⠿ 在現代開發專案時，一定會用到環境變數，像是讀取 `AWS Secret Key` 等等，在部署上面也會透過設定變數讓專案依據不同環境讀取不同環境變數，在此使用 [*joho/godotenv*](https://github.com/joho/godotenv) 套件模擬實現讀取環境變數的設定，可以讓開發者透過 `.env` 檔案動態改變環境變數。

> 我們可以在章節 «[01 使用 host-fasthttp 快速建構你的第一個 Web API](01_QUICK_START.md)» 的專案中找到 **.env** 檔。
>
> 💾 **.env**
> ```yaml
> Environment=local
> ```
> 可見其內容設定 `Environment` 參數的值為 `local` 。

⠿ 設定檔載入順序是在 **app.go** 中，使用 `fasthttp.Startup()` 並透過 `ConfigureConfiguration{}` 設定。
> 
> 💾 **app.go**
> ```go
> func main() {
>   ctx := AppContext{}
>   fasthttp.Startup(&ctx).
>     Middlewares(
>       fasthttp.UseResourceManager(&ResourceManager{}),
>       fasthttp.UseXHttpMethodHeader(),
>     ).
>     ConfigureConfiguration(func(service *config.ConfigurationService) {
>       service.
>       // 載入 `config.yaml`
>       LoadYamlFile("config.yaml").
>       // 搭配`${Environment}`環境變數載入, 例：`config.local.yaml`
>       LoadYamlFile("config.${Environment}.yaml").
>       // 根據輸入參數載入環境變數
>       LoadEnvironmentVariables("").
>       // 根據輸入路徑參數尋找資源檔案內容
>       LoadResource(".").
>       // 根據輸入路徑參數尋找資源檔案內容
>       LoadResource(".conf/${Environment}").
>       // 載入命令啟動列的參數
>       LoadCommandArguments()
>     }).
>     Run()
> }
> ```
> 可見其載入原則為：
>  - 由上而下的載入順序，**最上方為首要載入，並依次順序往下載入**。
>  - 載入過程中，如遇相同參數的設定，則由後載入的值做覆蓋。
> 
> ⚠️ 注意： 載入過的參數就會一直存在，並不會因為後方載入的設定檔中，沒有設定其參數就刪除。

[🔝回目錄](#%e7%9b%ae%e9%8c%84)


----------------
## 多環境部署設定

⠿ 日常開發中，開發者通常會有應用多環境部署的需求，此時會利用環境變數設定，當部署到不同環境上時，讀取不同的設定檔做為該環境的應用程式配置，以下為環境情境的假設：

> ⠿ 環境：
>  - `local` 本機開發者。
>  - `prod` 正式環境部署。

⠿ 在上述的情境下建立兩個對應的設定檔，並設定應用程式啟動後的監聽位址與連接埠。這個值經由指定 `address` 的名稱來設定。

> ⠿ 新增設定檔與設定：
>   - `config.local.yaml`
>   - `config.prod.yaml`
> 
> 以下為依據環境產生並設定的兩個設定檔。
>
> 💾 **config.local.yaml**
> ```yaml
> address: ":10074"
> ```
> 
> 💾 **config.prod.yaml**
> ```yaml
> address: ":20074"
> ```

[🔝回目錄](#%e7%9b%ae%e9%8c%84)


----------------
## 調整環境變數並啟動應用程式


#### **步驟一**：新增 config.prod.yaml 檔，並新增 `address` 的值為 `:20074` 並存檔。
> 💾 **config.prod.yaml**
> ```yaml
> address: ":20074"
> ```


#### **步驟二**：開啟 .env 檔，並修改 `Environment` 的值為 `prod` 並存檔。
> 💾 **.env**
> ```yaml
> Environment=prod
> ```

#### **步驟三**：開啟新的終端機，並使用下面的命令直接啟動應用程式。
```bash
$ gorun
```

#### **步驟四**：啟動完成後，使用下面的命令嘗試連接。
```bash
$ curl -s-v -XPING http://127.0.0.1:20074/
```

> 💡 <span class="remind">另一個作法是，你也可以在命令啟動列中直使用使 `-address` 參數指定應用程式的通信埠，依照載入順序，最終會依據命令列中的參數為最終的設定值。</span>
> ```bash
> $ gorun app.go -address :13994
> ```

[🔝回目錄](#%e7%9b%ae%e9%8c%84)


----------------
### 🎬 What's Next?

  > ### [**05 開發慣例**](05_CODING_CONVENTIONS.md)
  > 本節將介紹 `host-fasthttp` 的專案中的開發慣例，例如專案目錄的規範、命名規範...等等。
