---
layout: post
title: 'Web開發101-Session與Cookie'
date: 2015-11-25 12:20
comments: true
categories: 
---
##Cookie是什麼？

當使用者開啟某個網站時，瀏覽器就會傳送保存在用戶端的cookie到伺服器端，cookie中包含使用者上次瀏覽該網站的一些活動紀錄與資訊，而檔案大小上限為4kb。

當你瀏覽網站時，瀏覽器會自動下載cookie，用來追蹤你在網站上的一些活動紀錄，而在Rails中，保存cookie的cookie container本身看起來像一個hash，於controller中，就可透過相關的method來讀取其中的資料，許多Rails開發者使用cookie來記錄一些使用者偏好設定與資安方面較不敏感的資料，但千萬不要將重要資訊保存在cookie中，cookies 屬於沒有加密的公開檔案，且是可由使用者任意讀取的。

預設情況下，Rails提供相關的helper可操作cookie container。

舉例來說，Rails中你可以直接操作`cookies`這個特別保留的hash，而當中每個key-value pair分別儲存不同的資訊，如果加入`cookies[:hair-color]=“blonde"`在某個Controller#Action中，當執行時，打開瀏覽器的developer tools就可以看到在cookie中看到key(name)為hair-color，而value為blonde，被儲存在用戶端，而透過`cookies.delete(:hair-color)`就可以刪除該cookie。

Cookies參考說明：
http://skillcrush.com/2012/05/16/cookies/
http://www.theodinproject.com/ruby-on-rails/sessions-cookies-and-authentication

##Session是什麼？

###Session id原理
一般而言，相對於cookie儲存在使用者(Client)端，session則是儲存在伺服器(Server)端，session也需要cookie的輔助才能產生運作，當使用者造訪我們的網站時，我們由伺服器產生 session id (32 byte long MD5 hash value)，並傳送存有這個 session id 的 cookie 給瀏覽器儲存，之後使用者每次造訪我們網站時，只需要比對 cookies 上的 session id 和 session 裡的 session id 就可以知道使用者身份，大部份的網站也是運用此原理實作儲存 User 登入狀態的機制。

換句話說，如果使用者(client)端的cookie清掉，那你當下的登入狀態，也會跟著消失，重整網頁之後會產生一個新的cookie，但就不是你先前的登入狀態了。

###在Rails中使用Session
因為HTTP是一種無狀態的通訊協定，為了能夠讓瀏覽器能夠在跨request之間記住資訊，Rails內建了session功能，像是記住登入的狀態、記住使用者購物車的內容等等，都是用session實作出來的。

在Rails中要操作Session，直接操作session這個Hash變數即可，舉例來說：
`session[:cart_id] = @cart.id`

###Session Storage

然而目前Rails預設是採取CookieStore，是上述的Session id原理不太一樣的做法，CookieStore其實就是直接將session資料透過`config/secrets.yml`的secret_key_base編碼後儲存在使用者端的cookie，好處是對伺服器的效能負擔很低，但檔案大小上限只有4kb，而因為網站每次傳送request都會包含cookie，較大的cookie檔案也意味著較慢的網站效能，如果不小心外洩secret_key_base或被反編碼成功，Session資料就會被解出來，因此安全性較低，不適合存放機密資料。

當然Rails也提供存放Session的其他選項來供開發者做選擇，透過修改config/initializers/session_store.rb來達成包含：
- :active_record_store 資料庫
- :mem_cache_store 使用Memcached快取系統來儲存，適合高流量的網站
- :cache_store 使用Rails快取

一般來說使用預設的CookieStore即可，如果對安全性較高要求，可以使用資料庫。如果希望兼顧效能，可以考慮使用Memcached。


更多Session參考說明：
- General
https://rocodev.gitbooks.io/rails-102/content/chapter2-rails/cookies-and-session.html
http://andikan.github.io/blog/2012/10/03/cookie-and-session/

- Session-Storage
https://ihower.tw/rails4/actioncontroller.html#session-storage
http://dev.housetrip.com/2014/01/14/session-store-and-security/
http://www.justinweiss.com/blog/2015/03/17/how-rails-sessions-work/

- Access the session
http://guides.rubyonrails.org/action_controller_overview.html#accessing-the-session