---
layout: post
title: 'Rails for Zombies學習筆記'
date: 2015-07-14 09:45
comments: true
categories: [rails]
---
當初看到這個標題，還不知道這標題到底想表達什麼？ 學了內容原來是CodeSchool作者群利用簡化版的Twitter來說明Rails的框架運作，而且發Twitter的不是人而是Zombie~XD，個人覺得這份教材非常有趣，學起來很有幹勁，簡單整理自己的一些心得～

1. 作者群將Rails中的最重要的組成都用了相稱的比喻圖像，有效地幫助記憶與熟悉。
2. 有些練習可能要稍微想一下，但看一下hint基本上都可以解的出來，錯誤也有提示，真的解不出來網路上也google的到～對我而言，盡量不回頭看教材，才知道自己哪種語法不熟，題目都練到可以不看提示，只看需求然後寫對。
3. 不熟的地方就重複多看幾次，徹底了解，這份教材很多概念都是Rails最最基本的章節。

簡單把此份教材中一些學習重點或對新手(小弟本人XD)不易理解的內容整理如下，作為自身後續回憶複習的參考～

#CURD
1. Hash--Series of key value pairs  
```
	Recipe:
	variable = {key: value}  # 新增或更新變數
	variable[:key]  # Read the value
```
[更多的應用參考Ruby Hash API(2.2.2)](http://ruby-doc.org/core-2.2.2/Hash.html)
2. **CURD--Create, Update, Read, Delete**為資料庫操作的基本四種型態，包括資料讀取、新增、更新與刪除，而在Rails中，由Model扮演與資料庫溝通的角色，講義15頁有相關基本語法可供作參考範例。
3. Rails的Model[命名慣例](http://guides.rubyonrails.org/active_record_basics.html)為單字首字大寫。
    
#Model
1. Rails＿app下的Model檔案內容裡的名稱慣例為單字首字大寫(例如：class TodoList < ActiveRecord::Base)，但Model檔案名稱為小寫(例如todo＿list.rb)，而Model與資料庫表格之間的對應關係(Mapping)可參考講義27頁。
2. 在Model也會放入資料的正確性驗證(Valiadtion)，若驗證失敗，資料則無法存進資料庫，相關驗證方式與基本語法可參考講義31、32頁與[Guide](http://guides.rubyonrails.org/active_record_validations.html)。
3. 資料之間的關聯性宣告也屬於Model的管轄範圍，如belongs＿to、has＿many，基本語法可參考講義37、38頁與[Guide](http://guides.rubyonrails.org/association_basics.html)。

#View
1. Rails＿app下的View中存在.erb檔(例如 index.html.erb)，erb為Embedded Ruby的簡寫，Ruby inside HTML，<% %>中為Ruby語法，<%= %>則代表將其中Ruby語法的執行結果(Return)顯示(Print)出來。
2. application.html.erb為HTML與共用區塊的模板(Template)，如顯示notice，而其中 <%= yield %> 所在則是顯示個別頁面內容。
3. 建立連結語法[link＿to](http://api.rubyonrails.org/)參考講義61、64頁與[Guide](http://guides.rubyonrails.org/layouts_and_rendering.html#using-render)。

#Controller
1. Controller在Rails中扮演的角色[(Guide)](http://guides.rubyonrails.org/action_controller_overview.html#what-does-a-controller-do-questionmark)，簡單來說，Controller中定義各種方法(Method)或稱行動(Action)，當接收route傳送來的要求(Request)後便與Model溝通要求所需資料，得到資料後再丟給View來做網頁呈現，基本語法參考講義79頁
2. 在Rails中定義Instance variable如@tweet(But why?)，與Accepting parameters如/tweets/1，其params[:id]就是1，相關參考講義81~84頁與[Guide](http://guides.rubyonrails.org/action_controller_overview.html#parameters)。
3.Strong Parameter，則是一種安全機制，詳細可以參考SDLong大的[解說](http://rails101s.logdown.com/posts/211391-strong-parameters)與[Guide](http://guides.rubyonrails.org/action_controller_overview.html#strong-parameters)。
4. 簡單的權限設定(Authorization)，例如用session抓id加上條件判斷，限定該使用者可執行的Action，參考講義94、95頁。

#Route
1.Route在Rails中扮演的角色([Guide](http://guides.rubyonrails.org/routing.html)與[實戰聖經](https://ihower.tw/rails4/routing.html))，url傳至後端，再由route分配到對應的controller#action，講義中也介紹多種變化如Named Routes、Redirect…等


#Exercise Refresher
Lab 4, Exercise  3 strong parameter
![3A0782C7-96EE-44C6-B37F-0CAE9037518B.png](http://user-image.logdown.io/user/13199/blog/12437/post/285103/fyrTACAbTQKQELknIai1_3A0782C7-96EE-44C6-B37F-0CAE9037518B.png)
Lab 4, Exercise  4
![E1A7C559-9E38-41C2-ADF6-C3E4235945EE.png](http://user-image.logdown.io/user/13199/blog/12437/post/285103/uduUpIDDTLKbavdKNICP_E1A7C559-9E38-41C2-ADF6-C3E4235945EE.png)
