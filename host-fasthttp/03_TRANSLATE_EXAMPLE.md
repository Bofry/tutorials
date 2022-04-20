<link href="tutorial.css" rel="stylesheet" />

**March 2022**  |  **v1.0**

[↩ 回首頁](README.md)

03 Translate 範例
================================
🎯 本節使用 ServiceProvider 提供應用程式與資源互動。你將會掌握：

  1. 如何建立第三方 `provider` 物件和方法。
  2. 如何配置 `ServiceProvider` 與注入。
  3. 如何於 `Resource` 調用 `ServiceProvider` 提供的方法。

----------------------------------------------------------------
#### 目錄
  - [情境](#情境)
  - [實際操作](#實際操作)
----------------------------------------------------------------

## 情境

⠿ 服務將提供翻譯功能，讓使用者呼叫 `GET` request method，並帶入參數進行翻譯：

>**路徑**: `/Translate`
>
>**查詢字串**:
>- **SOURCE_LANGUAGE** `required` : 指示進行當前翻譯內容的語言。
>- **TARGET_LANGUAGE** `required` : 指示進行當前翻譯結果的語言。
>
>
>**參數**:
>- **content** `required` : 翻譯內容


⠿ 使用 httparg 提供的方法，對輸入參數做處理：
- 方法：
  - `ProcessQueryString` ：處理輸入的請求參數。
  - `ProcessContent` ：處理輸入的請求內容。

⠿ 接續透過第三方服務取得翻譯結果並回傳 `translation` 翻譯結果。

[🔝回目錄](#%e7%9b%ae%e9%8c%84)

----------------

## 實際操作

⠿ 本節根據情境使用 **httparg** 處理輸入參數與調用 *ServiceProvider* 的基本步驟。

- **步驟一: 新增 model 目錄**

  ⠿ 於根目錄下新增 `model` 目錄，用於存放共用的回傳結果的 `struct`。
  ```bash
  $ mkdir model
  ```

- **步驟二: 建立 translate result struct**

  ⠿ 於 `model` 目錄下，新增 `postTranslateResult.go` 檔，用於存放結果的 `struct` 其內容如下：
  ```go
  package model
  
  import (
    "encoding/json"
    "fmt"
  )
  
  type PostTranslateResult struct {
    Translation string `json:"translation"`
  }
  
  func (p *PostTranslateResult) UnmarshalJSON(data []byte) error {
    // raw_data = [[["illustrate","說明",null,null,10]],null,"zh-CN",null,null,null,null,[]]
    var tmp []interface{}
    err := json.Unmarshal(data, &tmp)
    if err != nil {
      return err
    }
    if v, ok := tmp[0].([]interface{}); ok {
      if vv, ok := v[0].([]interface{}); ok {
        if str, ok := vv[0].(string); ok {
          p.Translation = str
          return nil
        }
      }
    }
    return fmt.Errorf("can't unmarshal translation")
  }
  ```

- **步驟三: 新增 provider 目錄**

  ⠿ 於根目錄下新增 `provider` 目錄，用於存放呼叫第三方資源或服務的實作。
  ```bash
  $ mkdir provider
  ```

- **步驟四: 建立 google reanslate provider**

  ⠿ 於 `provider` 目錄下，新增 `googleTranslateProvider.go` 檔，其內容如下：
  ```go
  package provider
  
  import (
    "apiservice/model"
    "encoding/json"
    "errors"
    "fmt"
    "github.com/valyala/fasthttp"
    "net/url"
    "time"
  )
  
  type GoogleTranslateProvider struct {
  }
  
  func NewGoogleTranslateProvider() *GoogleTranslateProvider {
    r := &GoogleTranslateProvider{}
    return r
  }
  
  func (p *GoogleTranslateProvider) PostTranslate(
    sourceLanguage string,
    targetLanguage string,
    content string) (*model.PostTranslateResult, error) {
  
    req := fasthttp.AcquireRequest()
    resp := fasthttp.AcquireResponse()
    defer fasthttp.ReleaseRequest(req)
    defer fasthttp.ReleaseResponse(resp)

    url := fmt.Sprintf("https://translate.google.com/translate_a/single?client=at&sl=%s&tl=%s&dt=t&q=%s", sourceLanguage,   targetLanguage, url.QueryEscape(content))
    req.SetRequestURI(url)
    req.Header.SetMethod("POST")
  
    client := &fasthttp.Client{}
  
    req.SetBody([]byte(content))
    err := client.DoTimeout(req, resp, 3*time.Second)
    if err != nil {
      return nil, err
    }
  
    bodyBytes := resp.Body()
    if resp.StatusCode() != 200 {
      return nil, errors.New(string(bodyBytes))
    }
  
    result := &model.PostTranslateResult{}
    err = json.Unmarshal(bodyBytes, &result)
    
    if err != nil {
      return nil, err
    }
  
    return result, nil
  }
  ```

- **步驟五: 將 GoogleTranslateProvider 加入至 ServiceProvider**

  1. 使用編輯器打開 **appContext.go** 檔。
  2. 找到第27行，
      ```go
      ServiceProvider struct {
      }
      ```
      更改為：
      ```go
      ServiceProvider struct {
        GoogleTranslateProvider *provider.GoogleTranslateProvider
      }
      ```
      > 💬 這個動作用於將 `GoogleTranslateProvider` 注入到 `ServiceProvider` 中。
  3. 找到第43行，
      ```go
      func (p *ServiceProvider) Init(conf *Config) {
        // initialize service provider components
      }
      ```
      更改為：
      ```go
      func (p *ServiceProvider) Init(conf *Config) {
        // initialize service provider components
        p.GoogleTranslateProvider = provider.NewGoogleTranslateProvider()
      }
      ```
      > 💬 這個動作是建立 `GoogleTranslateProvider` 實例並設定於 `ServiceProvider` 中的 `GoogleTranslateProvider`。

- **步驟六: 加入的 http request handler**
  1. 使用編輯器打開 **app.go** 檔。
  2. 找到第11行，
      ```go
      type ResourceManager struct {
        *DefaultResource `url:"/"`
      }
      ```
      更改為：
      ```go
      type ResourceManager struct {
        *DefaultResource   `url:"/"`
        *TranslateResource `url:"/Translate"`
      }
      ```
      > 💬 若你使用 IDE，比如 vscode 則可能會標示語法錯誤，請先忽略。
      1. 執行 CLI 命令 `go generate` 來配置 http request handler 檔案。
      ```bash
      $ go generate
      ```
      > 💬 這個動作為幫你產生 **TranslateResource.go** 的檔案，並會放在專案內的 `/resource` 目錄中。
      >
      > 💬 若使用 IDE 且支援 go generate 指令的話，可以在程式碼上方直接按下 `run go generate` 的按鈕，也會有相同的效果。

- **步驟七: 新增 args 目錄**

  ⠿ 於 `resource` 目錄下新增 `args` 目錄，用於存放處理輸入參數的 `struct`。
  ```bash
  $ cd resource
  $ mkdir args
  ```

- **步驟八: 建立 Translate struct**

  ⠿ 於 `resource/args` 目錄下，新增 `postTranslateArgs.go` 檔，用於接收輸入參數的 `struct` 其內容如下：
  ```go
  package args

  type PostTranslateArgs struct {
      SourceLanguage string `query:"*SOURCE_LANGUAGE"`
      TargetLanguage string `query:"*TARGET_LANGUAGE"`
      Content        string `json:"content"`
  }
  ```

- **步驟九: 新增 Translate Get request method**

  1. 使用編輯器打開 **translateResource.go** 檔。
  2. 於建構子內新增 `ServiceProvider *ServiceProvider` 參數。
  > 💬 此處使用 `Dependency Injection` 方式注入並提供呼叫使用。

  > ⚠️ 型別與名稱必為 `ServiceProvider *ServiceProvider` 是不可變動。

  3. 新增 `GET` 方法，並透過 `httparg` 處理輸入參數。
  4. 呼叫 `ServiceProvider` 內的 `GoogleTranslateProvider.PostTranslate` 並帶入參數。
  5. 最終回應翻譯結果，完整內容如下：
  ```go
  package resource
  
  import (
    "encoding/json"
    "github.com/Bofry/host-fasthttp/response"
    "github.com/Bofry/httparg"
    "github.com/valyala/fasthttp"
  
    . "apiservice/internal"
    . "apiservice/resource/args"
  )
  
  type TranslateResource struct {
    // 此處使用 `Dependency Injection` 方式注入。
    ServiceProvider *ServiceProvider
  }
  
  func (r *TranslateResource) Ping(ctx *fasthttp.RequestCtx) {
    response.Success(ctx, "text/plain", []byte("PONG"))
  }
  
  func (r *TranslateResource) Get(ctx *fasthttp.RequestCtx) {
    args := PostTranslateArgs{}
  
    httparg.Args(&args).
      ProcessQueryString(ctx.QueryArgs().String()).
      ProcessContent(ctx.PostBody(), "application/json").
      Validate()
  
    // 呼叫依賴注入的 `ServiceProvider` 內的 `GoogleTranslate.PostTranslate` 的方法。
    rsp, _ := r.ServiceProvider.GoogleTranslateProvider.PostTranslate(args.SourceLanguage, args.TargetLanguage, args.Content)
  
    body, _ := json.Marshal(rsp)
    response.Success(ctx, "text/plain", body)
  }
  ```

- **步驟十: 啟動專案**

  ⠿ 執行 CLI 命令 `gorun` 來啟動專案。
  ```bash
  $ gorun
  ```
  > 💬  CLI 命令 `gorun` 整合 `godotenv` 與 `go run` 命令的工具，透過該命令能夠載入自訂義的環境設定，並啟動專案。
  >
  > 💬 使用 `Ctrl` + `C` 來關閉服務。

- **步驟十一: 檢查**
  ```bash
  $ curl -XGET -sv 'http://127.0.0.1:10074/Translate?SOURCE_LANGUAGE=zh-CN&TARGET_LANGUAGE=en' \
  --header 'Content-Type: application/json' \
  --data-raw '{
      "content": "說明"
  }'
  ```
  將會得到以下的輸出：
  ```bash
  *   Trying 127.0.0.1:10074...
  * TCP_NODELAY set
  * Connected to 127.0.0.1 (127.0.0.1) port 10074 (#0)
  > GET /Translate?SOURCE_LANGUAGE=zh-CN&TARGET_LANGUAGE=en HTTP/1.1
  > Host: 127.0.0.1:10074
  > User-Agent: curl/7.64.1
  > Accept: */*
  > Content-Type: application/json
  > Content-Length: 31
  >
  * upload completely sent off: 31 out of 31 bytes
  < HTTP/1.1 200 OK
  < Server: WebAPI
  < Date: Mon, 07 Mar 2022 09:03:07 GMT
  < Content-Type: text/plain
  < Content-Length: 67
  < Connection: close
  <
  * Closing connection 0
  {"translation":"illustrate"}%
  ```

[🔝回目錄](#%e7%9b%ae%e9%8c%84)

----------------
### 🎬 What's Next?

  > ### [**04 設定檔配置**](04_APP_CONFIGURATION.md)
  > 本節將介紹 `host-fasthttp` 的集中管理設定值的設計，以及如何定義不同環境的設定檔，以及如何切換不同的工作環境。