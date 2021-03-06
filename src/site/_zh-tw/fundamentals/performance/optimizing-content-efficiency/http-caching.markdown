---
layout: article
title: "HTTP 快取"
description: "透過網路取得內容的做法不僅緩慢，成本也很高：大型回應需要在用戶端和伺服器之間進行多次往返通訊，因此拖延了瀏覽器可以使用及處理內容的時間，同時也增加了訪客的行動數據傳輸費用。因此，如果能夠快取及重複使用先前取得的資源，對於將效能最佳化將是非常關鍵的一環。"
introduction: "透過網路取得內容的做法不僅緩慢，成本也很高：大型回應需要在用戶端和伺服器之間進行多次往返通訊，因此拖延了瀏覽器可以使用及處理內容的時間，同時也增加了訪客的行動數據傳輸費用。因此，如果能夠快取及重複使用先前取得的資源，對於將效能最佳化將是非常關鍵的一環。"
article:
  written_on: 2014-01-01
  updated_on: 2014-01-05
  order: 5
collection: optimizing-content-efficiency
authors:
  - ilyagrigorik
key-takeaways:
  validate-etags:
    - "伺服器透過 ETag HTTP 標題傳遞驗證權杖"
    - "您可以透過驗證權杖進行高效率的資源更新檢查：如果資源未變更，則不會傳輸任何資料。"
  cache-control:
    - "每個資源都可以透過 Cache-Control HTTP 標題來定義專屬的快取策略"
    - "Cache-Control 指令負責管控可快取回應的使用者、條件和快取時限"
  invalidate-cache:
    - "在資源「過期」之前，將一直使用本地快取的回應"
    - "透過將檔案內容指紋碼嵌入網址，我們可以強制使用者端更新到新版的回應"
    - "為了獲得最佳效能，每個應用程式需要定義專屬的快取階層"
notes:
  webview-cache:
    - "如果在應用程式中使用 Webview 來擷取及顯示網頁內容，可能需要提供額外的配置旗標，以確保啟用 HTTP 快取，並根據用途設定合理的快取大小，同時也可延長快取的效期。請查看平台文件並確認您的設定！"
  boilerplate-configs:
    - "提示：HTML5 Boilerplate 專案包含了所有主流伺服器的<a href='https://github.com/h5bp/server-configs'>設定檔範例</a>，並且為每個配置旗標和設定都提供了詳細的備註：請在清單中找到您喜歡的伺服器，尋找適合的設定，然後複製/確認您的伺服器配置了推薦的設定。"
  cache-control:
    - "Cache-Control 標題的定義詳列於 HTTP/1.1 規範中，取代了之前用來定義回應快取策略的標題 (例如 Expires)。現今所有的瀏覽器都支援 Cache-Control，因此我們有這個標題就夠了。"
---

{% wrap content%}

<style>
  img, video, object {
    max-width: 100%;
  }

  img.center {
    display: block;
    margin-left: auto;
    margin-right: auto;
  }
</style>

{% include modules/toc.liquid %}

好消息是每個瀏覽器都內建了 HTTP 快取！ 我們所要做的就是，確保每個伺服器回應都提供正確的 HTTP 標題指令，以指示瀏覽器快取回應的時機和時限。

{% include modules/highlight.liquid character="{" position="left" title="" list=page.notes.webview-cache %}

<img src="images/http-request.png" class="center" alt="HTTP 請求">

伺服器傳回回應時，還會發出一組 HTTP 標題，用來描述內容類型、長度、快取指令、驗證權杖等。舉例來說，在上圖的交流中，伺服器傳回了一個 1024 位元組的回應，指示用戶端快取回應長達 120 秒，並提供驗證權杖 (「x234dff」)，在回應過期之後，可以用來驗證資源是否遭到修改。


## 使用 ETag 驗證快取的回應

{% include modules/takeaway.liquid list=page.key-takeaways.validate-etags %}

讓我們假設在首次擷取資源 120 秒之後，瀏覽器又對該資源發出新請求。首先，瀏覽器會檢查本地快取並找到之前的回應，但很可惜這個回應現在已經「過期」，無法再使用。此時，瀏覽器也可以直接發出新請求以獲得新的完整回應，但是這樣做效率較低。如果資源未曾變更，我們就沒有理由再去下載已存在於快取中的相同位元組。

這就是 ETag 標題中指定的驗證權杖所要解決的問題：伺服器會產生並傳回一個隨機權杖，通常是檔案內容的雜湊值或者其他指紋碼。用戶端不必瞭解指紋碼是如何產生的，只需要在下一個請求中將其傳送給伺服器：如果指紋碼仍然一致，說明資源未被修改，我們就可以跳過下載步驟。

<img src="images/http-cache-control.png" class="center" alt="HTTP Cache-Control 示例">

在上面的例子中，用戶端自動在「If-None-Match」HTTP 請求標題中提供 ETag 權杖，伺服器針對目前的資源檢查權杖，如果未被修改過，則傳回「304 Not Modified」回應，告訴瀏覽器快取中的回應未被修改過，可以再延用 120 秒。請注意，我們不必再次下載回應，因此這可節省時間和頻寬。

網路開發人員應如何利用高效率的重新驗證機制？ 瀏覽器代替我們完成了所有的工作：自動檢測是否已指定了驗證權杖，並會將驗證權杖附加到發出的請求上，根據從伺服器收到的回應，在必要時更新快取時間戳記。**實際上，我們唯一要做的就是確保伺服器提供必要的 ETag 權杖：查看伺服器文件中是否有必要的設定旗標。**

{% include modules/remember.liquid list=page.notes.boilerplate-configs %}


## Cache-Control

{% include modules/takeaway.liquid list=page.key-takeaways.cache-control %}

最好的請求是不必與伺服器進行通訊的請求：透過回應的本機複本，我們可以避免所有的網路延遲以及傳輸資料的數據連線成本。有鑑於此，HTTP 規範允許伺服器傳回[一系列不同的 Cache-Control 指令](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)，控制瀏覽器或者其他中繼快取如何快取某個回應以及快取的期限。

{% include modules/remember.liquid list=page.notes.cache-control %}

<img src="images/http-cache-control-highlight.png" class="center" alt="HTTP Cache-Control 示例">

### "no-cache" 和 "no-store"

「no-cache」表示必須先與伺服器確認傳回的回應是否已變更，然後才能使用該回應來滿足後續對同一個網址的請求。因此，如果存在合適的驗證權杖 (ETag)，no-cache 會發起往返通訊來驗證快取的回應，如果資源沒有任何變更，即可避免下載步驟。

相較之下，「no-store」更加簡單，直接禁止瀏覽器和所有中繼快取儲存傳回的任何回應版本，例如包含個人隱私資料或銀行資料的回應。每次使用者請求該資產時，都會向伺服器發送一個請求，而且每次都會下載完整的回應。

###「public」和「private」

如果回應標記為「public」，即使具備關聯的 HTTP 認證，甚至回應狀態碼無法正常快取，回應也可以供使用者快取。在大多數情況下，「public」並不是必要項目，因為明確的快取資訊 (例如「max-age」) 已表示
回應可供快取。

另一方面，瀏覽器可以快取「private」回應，但是通常只開放給單一使用者快取，因此不允許任何中繼快取對其進行快取，例如使用者瀏覽器可以快取包含使用者私人資訊的 HTML 網頁，但是 CDN 不能快取。

### "max-age"

這個指令指定從目前請求開始，允許擷取的回應重複使用的最長時間 (單位為秒)，例如「max-age=60」表示回應可以再快取及重複使用 60 秒。

## 定義最佳 Cache-Control 政策

<img src="images/http-cache-decision-tree.png" class="center" alt="快取決策樹">

還在為您的應用程式所使用的特定資源或一組資源煩惱嗎？快按照上面的決策樹確定這些資源的最佳快取策略。在理想情況下，您的目標應該是可在用戶端長時間快取最多的回應，並且為每個回應提供驗證權杖，讓重新驗證流程以高效率完成。

<table class="table-2">
<colgroup><col span="1"><col span="1"></colgroup>
<thead>
  <tr>
    <th width="30%">Cache-Control 指令</th>
    <th>說明</th>
  </tr>
</thead>
<tr>
  <td data-th="cache-control">max-age=86400</td>
  <td data-th="說明">瀏覽器和任何中繼快取都可以快取回應 (如果是 "public") 長達一天 (60 秒 x 60 分 x 24 小時)</td>
</tr>
<tr>
  <td data-th="cache-control">private, max-age=600</td>
  <td data-th="說明">用戶端瀏覽器最長只能快取回應 10 分鐘 (60 秒 x 10 分)</td>
</tr>
<tr>
  <td data-th="cache-control">no-store</td>
  <td data-th="說明">不允許快取回應，每個請求必須擷取完整的回應。</td>
</tr>
</table>

根據 HTTP Archive 指出，在排名最高的 300,000 個網站中 (Alexa 排名)，[幾乎有半數的下載回應其實都可供瀏覽器快取] (http://httparchive.org/trends.php#maxage0)，對於重複性網頁瀏覽量和造訪來說，這可省下相當可觀的成本！ 當然，這並不表示您的特定應用程式有 50% 的資源可以快取：有些網站可以快取 90% 以上的資源， 而有些網站則因為包含許多私人或具時效性的資料，根本無法快取。

**審查您的網頁，確定哪些資源可供快取，並確認這些資源可傳回正確的 Cache-Control 和 ETag 標題。**


## 作廢及更新已快取的回應

{% include modules/takeaway.liquid list=page.key-takeaways.invalidate-cache %}

瀏覽器發出的所有 HTTP 請求會先傳送到瀏覽器的快取，以查看其中是否已快取了可滿足請求的有效回應。如果有相符的回應，就會直接從快取中讀取回應，藉此避免了網路延遲以及傳輸過程產生的行動數據傳輸費用。**不過，如果我們希望更新或作廢已快取的回應，該怎麼辦？**

舉例來說，假設我們已經告訴訪客快取 CSS 樣式表最長不要超過 24 小時  (max-age=86400)，但是設計人員剛剛提交了一個更新，而我們希望所有使用者都能使用。我們該如何通知所有訪客快取的 CSS 副本已過時，需要更新快取？ 這是一個棘手的問題。實際上，至少在不變更資源網址的情況下，我們做不到。

瀏覽器快取回應後，在快取的版本過期以前都會一直使用 (這個行為是由 max-age 或者 expires 指定的)，或者直到因為某些原因從快取中刪除，例如使用者清除了瀏覽器快取。因此，在建構網頁時，各個使用者可能使用的檔案版本都不同；剛剛擷取該資源的使用者將使用新版本，而曾經快取舊版本 (但是依然有效) 的使用者將繼續使用舊版本的回應。

**所以，我們如何才能魚和熊掌兼得，同時達成用戶端快取和快速更新？** 很簡單，在資源內容變更時，我們可以變更資源的網址，強制使用者下載新回應。一般來說，只要在檔案名稱中嵌入檔案的指紋碼或版本號碼，例如 style.**x234dff**.css 即可。

<img src="images/http-cache-hierarchy.png" class="center" alt="快取階層">

藉由為每個資源定義快取策略的功能，我們可以定義「快取階層」。如此一來，不但可以控制每個回應的快取時間，還可以控制訪客看到新版本的速度。讓我們一起分析上面的示例：

* HTML 標記為「no-cache」，這表示瀏覽器在每次請求時都會重新驗證文件，如果內容變更，就會擷取最新版本。同時，在 HTML 標記中，我們在 CSS 和 JavaScript 資源的網址中嵌入指紋碼：如果這些檔案的內容變更，網頁的 HTML 也會隨之變更，並將下載 HTML 回應的新副本。
* 允許瀏覽器和中繼快取 (例如 CDN) 快取 CSS，期限設定為 1 年。請注意，我們可以放心使用 1 年的「遠期期限」，因為我們在檔案名稱中嵌入了檔案指紋碼：如果 CSS 更新，網址也會隨之變更。
* JavaScript 期限也設定為 1 年，但是被標記為「private」，也許是因為其中包含了不適合 CDN 快取的使用者私人資料。
* 快取圖片時不包含版本或唯一指紋碼，期限設定為 1 天。

混合使用 ETag、Cache-Control 和唯一網址，我們可以提供最佳的方案，例如較長的期限，控制可以快取回應的位置，以及視需要隨時更新。

## 快取檢查表

天底下沒有所謂的最佳快取策略。根據您的流量模式、提供的資料類型以及應用特定的資料更新要求，您必須定義及設置每個資源最適合的設定和整體的「快取階層」。

在定義快取策略時，請記住下列技巧和方法：

1. **使用一致的網址：**如果您在不同的網址上提供相同的內容，將會多次取得及儲存該內容。提示：請注意[網址區分大小寫](http://www.w3.org/TR/WD-html40-970708/htmlweb.html)！
2. **確認伺服器提供驗證權杖 (ETag)：**透過驗證權杖，如果伺服器上的資源未曾變更，就不必傳輸相同的位元組。
3. **確定中繼快取可以快取哪些資源：**對所有使用者的回應完全相同的資源很適合由 CDN 或其他中繼快取進行快取。
4. **確定每個資源的最佳快取效期：**不同的資源可能有不同的更新要求。審查並確定每個資源適合的 max-age。
5. **確定網站的最佳快取階層：**對 HTML 文件組合使用包含內容指紋碼的資源網址以及短時間或 no-cache 的效期，可以控制用戶端取得更新的速度。
6. **更新內容最小化：**有些資源的更新頻率比其他資源高。如果資源的特定部分 (例如 JavaScript 函式或一組 CSS 樣式) 會經常更新，請考慮將其程式碼當做單獨的檔案提供。如此一來，每次擷取更新時，剩餘內容 (例如不會頻繁更新的程式庫程式碼) 可以從快取中擷取，讓需要下載的內容量降到最低。


{% include modules/nextarticle.liquid %}

{% endwrap %}

