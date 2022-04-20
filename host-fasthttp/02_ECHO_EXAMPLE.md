<link href="tutorial.css" rel="stylesheet" />

**March 2022**  |  **v1.0**

[↩ 回首頁](README.md)

02 Echo 範例
================================
🎯 本節使用 httparg 來處理輸入的參數。你將會掌握：

  1. 認識如何使用 `httparg`。
  2. 如何使用 `ProcessQueryString` 處理輸入的請求參數。
  3. 如何使用 `ProcessContent` 處理輸入的請求內容。
  4. 如何使用 `Validate` 驗證。
  5. 認識 `httparg` 的相關特性。

----------------------------------------------------------------
#### 目錄
  - [情境](#情境)
  - [實際操作](#實際操作)
  - [你必須知道的 httparg 常用事項](#你必須知道的-httparg-常用事項)
----------------------------------------------------------------

## 情境

建立經典的 `ECHO` 例子，讓使用者呼叫 `ECHO` request method，帶入兩個參數 `VOICE` 與 `yell`，並串接字串做迴響：

>**路徑**: `/`
>
>**查詢字串**:
>- **VOICE** `required` : 聲音
>
>
>**參數**:
>- **yell** `required` : 大叫

本章節將使用 httparg 提供的方法，對輸入參數做處理，並回應輸入的參數內容：
- 方法：
  - `ProcessQueryString` ：處理輸入的請求參數。
  - `ProcessContent` ：處理輸入的請求內容。
  - `Validate` ：驗證請求內容。
  > ⚠️ 關於 Validate 失敗的 Error 處理將會在進階內容章節中介紹。

[🔝回目錄](#%e7%9b%ae%e9%8c%84)


----------------

## 實際操作

⠿ 本節根據情境使用 **httparg** 處理輸入參數的基本步驟。

- **步驟ㄧ: 新增 Args 目錄**

  ⠿ 於 `resource` 目錄下新增 `args` 目錄，用於存放處理輸入參數的 `struct`。
  ```bash
  $ cd resource
  $ mkdir args
  ```

- **步驟三: 建立 Echo struct**

  ⠿ 於 `resource/args` 目錄下，新增 `echoArgs.go` 檔，用於接收輸入參數的 `struct` 其內容如下：
  ```go
  package args
  
  type EchoArgs struct {
    // 查詢字串使用 tag:`query`，`*`表示必填。
    Voice string `query:"*VOICE"`
    // 請求內容使用 tag:`json`，`*`表示必填。
    Yell  string `json:"*yell"`
  }

  // 此為使用 httparg.Validate() 所呼叫建立的參數驗證方法
  func (args *EchoArgs) Validate() error {
    err := assert.Assert(
      // 輸入參數 Voice 不得為空字串。
      assert.NonEmptyString(args.Voice, "voice"),
      // 輸入參數 Yell 不得為空字串。
      assert.NonEmptyString(args.Yell, "yell"),
    )
    return err
  }
  ```

- **步驟四: 新增 Echo request method**

  1. 使用編輯器打開 **defaultResource.go** 檔。
  2. 新增 `Echo` 方法，並透過 `httparg` 處理輸入參數，並將字串串接回應；內容如下：
  ```go
  package resource
  
  import (
    . "apiservice/resource/args"
    "github.com/Bofry/host-fasthttp/response"
    "github.com/Bofry/httparg"
    "github.com/valyala/fasthttp"
  )
  
  type DefaultResource struct{}
  
  func (r *DefaultResource) Ping(ctx *fasthttp.RequestCtx) {
    response.Success(ctx, "text/plain", []byte("PONG"))
  }
  
  func (r *DefaultResource) Echo(ctx *fasthttp.RequestCtx) {
    args := EchoArgs{}
  
    httparg.Args(&args).
      // 使用 ProcessQueryString 處理輸入的請求參數
      ProcessQueryString(ctx.QueryArgs().String()).
      // 使用 ProcessContent 處理輸入的請求內容
      ProcessContent(ctx.PostBody(), "application/json").
      // 使用 Validate 呼叫參數驗證
      Validate()
      
    // 輸入參數字串串接
    str := args.Voice + args.Yell
  
    response.Success(ctx, "text/plain", []byte(str))
  }
  ```

- **步驟五: 啟動專案**

  ⠿ 執行 CLI 命令 `gorun` 來啟動專案。
  ```bash
  $ gorun
  ```
  > 💬  CLI 命令 `gorun` 整合 `godotenv` 與 `go run` 命令的工具，透過該命令能夠載入自訂義的環境設定，並啟動專案。
  >
  > 💬 使用 `Ctrl` + `C` 來關閉服務。

- **步驟六: 檢查**
  ```bash
  $ curl -XECHO -sv 'http://127.0.0.1:10074/?VOICE=Hello' \
  --header 'Content-Type: application/json' \
  --data-raw '{
      "yell":"World"
  }'
  ```
  將會得到以下的輸出：
  ```bash
  *   Trying 127.0.0.1:10074...
  * TCP_NODELAY set
  * Connected to 127.0.0.1 (127.0.0.1) port 10074 (#0)
  > ECHO /?VOICE=Hello HTTP/1.1
  > Host: 127.0.0.1:10074
  > User-Agent: curl/7.64.1
  > Accept: */*
  > Content-Type: application/json
  > Content-Length: 22
  >
  * upload completely sent off: 22 out of 22 bytes
  < HTTP/1.1 200 OK
  < Server: WebAPI
  < Date: Wed, 09 Mar 2022 02:27:13 GMT
  < Content-Type: text/plain
  < Content-Length: 10
  < Connection: close
  <
  * Closing connection 0
  HelloWorld%
  ```

[🔝回目錄](#%e7%9b%ae%e9%8c%84)

----------------


## 你必須知道的 httparg 常用事項

1. 必填(可用`*`)
2. 輸入參數為 **數字** 可以自動轉換為 **字串**。
3. 字串內若會合法數字可自動轉換。
4. 支援 json.RawContent
5. 支援時間  5h, 3m, 15s,.. 轉換為 time.Duration
6. 可接受陣列 (逗點分隔）。
7. query string 若有帶, 可以自動轉為 boolean ; 如 http://localhost?ENABLED=true 同等 http://localhost?ENABLED。

[🔝回目錄](#%e7%9b%ae%e9%8c%84)

----------------
### 🎬 What's Next?

  > ### [**03 Translate 範例**](03_TRANSLATE_EXAMPLE.md)
  > 本節將介紹 `ServiceProvider` 如何將共用資源注入，並讓應用程式調取使用。