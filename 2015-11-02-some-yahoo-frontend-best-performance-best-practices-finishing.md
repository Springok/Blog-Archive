---
layout: post
title: '一些Yahoo Frontend Best Performance 最佳實務整理'
date: 2015-11-02 17:05
comments: true
categories: 
---
之前上課的時候，Xdite介紹很多實務上提升網站效能的方式，其中也包含廣為人知的[Yahoo Frontend Best Performance](https://developer.yahoo.com/performance/rules.html)，總共有35條，並分類為Content, Server, Cookie, CSS, Javascript, Images, Mobile，而Rails中的Asset pipeline就已經包含其中的一些實作。
以下簡單翻譯整理最常見的其中五條：

- Minimize HTTP Requests
- Use a Content Delivery Network
- Put Stylesheets at the Top
- Put Scripts at the Bottom
- Gzip Components

###1. Minimize HTTP Requests (Content)

藉由減少HTTP請求數量來調校網站的執行效能是個不錯的開始，根據[Tenni Theurer's blog post Browser Cache Usage - Exposed!](http://yuiblog.com/blog/2007/01/04/performance-research-part-2/)的文章中提到，每日40~60％網站造訪者是empty cache，這也意味著增加網頁讀取速度是帶給使用者良好體驗的關鍵所在。

Combined files方法是透過將所有Script整合成一份Script，達到減少HTTP請求數量的目的，如網站中所有需要的CSS都彙整成一份CSS檔案，HTTP就可以減少為一次，減少Response時間，但當每個網頁所需要的CSS或Script不同時，就會增加實作上的難度。

- CSS Sprites是常見用來減少image requests的實作方式，結合各背景圖片(background image)成單一張圖片，並以CSS background-image 與background-position properties來呈現所需的網頁效果。

- Image maps則是整合多張圖片成單一圖片，檔案總大小不變，但減少了HTTP請求數量，加速了網頁讀取，但此實作方式僅適用contigous形式的網頁如Navigation bar。定義Image map的對應座標，實作上相當繁瑣乏味且容易出錯。實務上並不推薦使用。

Inline images使用[data: URL scheme](http://tools.ietf.org/html/rfc2397)將圖片資料嵌入實際頁面中，此舉將增加HTML文件的檔案大小，可減少HTTP請求數量，但並非所有瀏覽器都支援inline images。

###2.Use a Content Delivery Network(Server)

約80~90%的用戶端的response time是花費在下載網頁所需的各種物件上包含images, stylesheets, scripts, Flash等等，與其花費時間重新設計網站架構增加效能，倒不如將一些static content分散式儲存，透過CDN來實現，可大幅降低response time。

content delivery network (CDN) 是透過分散各地儲存相同內容的web server更有效率傳送相關內容到用戶端，通常都依據用戶端所在計算選用對應的web server來發送內容，例如選用透過較少的network hops或是具最快response time的web server。

大公司通常都會擁有自己的CDN，一般新創公司或小型私人網站，實行CDN通常是一筆可觀投資，但當用戶數成長與來自全球各區域的客戶增加，CDN就是必要採取的效能提升方式，以yahoo為例，其透過CDN改善了用戶端的response time達20%以上，選用CDN是相較於其他程式優化之下較簡單的實作方式，而且可以有效改善網站效能。

###3.Put Stylesheets at the Top(CSS)

當在Yahoo!中研究網站效能時，團隊發現到將stylesheet移到HTML文件HEAD位置時，網頁具有較快的讀取速度，這是因為此舉可使呈現頁面上更按部就班。
前端工程師希望HTML頁面的呈現順序是header,the navigation bar, the logo at the top等等向下進行，如此一來使用者等待網頁讀取的同時也能接收到一些回饋，可改善使用者體驗。
此外將stylesheet移到HTML文件移至底下時，會限制頁面上對應效果的呈現，可能就會造成頁面出現空白區塊。

###4.Put Scripts at the Bottom(Javascript)

Script文件所造成的效能問題在於，下載Script時會阻斷其他並行的檔案下載，而直到Script文件下載完畢前，都不會下載網頁中的其他物件。

但在某些情況下，這實作方式會遇到困難，例如，可能在網頁中需呈現一些Script中的效果（像是document.write），若此時找不到Script文件就會造成[Scoping issue](https://en.wikipedia.org/wiki/Scope_(computer_science)#JavaScript)。此時有個選擇是使用deferred scripts，此類含DEFER attribute的scripts不包含document.write，這樣網頁會略過此動作而繼續呈現頁面，但Firefox不支援DEFER attribute。若使用deferred scripts就可以將Script文件下載動作移至網頁文件最後執行，可增加縮短網頁讀取速度。

Ref:
Explaining JavaScript Scope And Closures(http://robertnyman.com/2008/10/09/explaining-javascript-scope-and-closures/)

###5.Gzip Components( server)

一般而言，傳送HTTP請求到取得回應所花費的時間可由前端上做大幅度的改善，當然其他因素如：使用者的網路頻寬速度、網際網路服務商等等也會造成影響，但這些都不在開發團隊的管轄，但還有其他變數，像是壓縮(Compression)可縮小HTTP請求的檔案大小進一步減少Response time。

從HTTP/1.1開始，在HTTP請求中加入Accept-Encoding header可使Client端可支援壓縮，指令如下：
Accept-Encoding: gzip, deflate

當Server端接收到含有此header的請求，就會使用Client端指定的方式中選定一個執行壓縮，而Server端則在回應加入一行Content-Encoding header 指令：
Content-Encoding: gzip

Gzip是目前最常見與最具效率的壓縮方式，一般而言可減少約70％左右的response檔案大小，現今聲稱90%網路中透過瀏覽器傳送的請求與回應都支援Gzip

Server根據檔案類型決定是否做Gzip，大多數網站對HTML文件做Gzip，包含scripts, stylesheet或是其他text response像是XML, JSON，而對image與PDF則不做Gzip，因為這些檔案本身已經做過壓縮。

盡可能對可執行Gzip的文件實作Gzip是一種簡單降低page weight與提升使用者體驗的方式。