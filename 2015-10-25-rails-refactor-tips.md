---
layout: post
title: 'Rails的一些Refactor小技巧'
date: 2015-10-25 19:44
comments: true
categories: [rails]
---
##Controller中什麼時機可用before_action
before_action最常用於準備跨action共用的資料，或是使用者權限驗證等等，此動作可以接受Code block、一個Symbol方法名稱或是一個物件(Rails會呼叫此物件的filter方法)。

像是如果使用Devise這個gem，會給你的使用者權限驗證的helper- [authenticate_user!](https://github.com/plataformatec/devise#controller-filters-and-helpers)，通常就會加到before_action。

另外也可以指定before_action所搭配的action，如：
```
before_action :require_login, only: [:create, :edit, :update, :destory]
```

如果需要取消從父類別繼承過來的Filter，也可以使用skip_before_action：
```
skip_before_action :filter_method_name
```

另外注意的是當有多個before_action的時候，Rails是由上往下依序執行的。

Ref:
https://ihower.tw/rails4/actioncontroller.html#filters

##View中什麼時機要用 Helper, 什麼時機要用 partial
兩者都可以協助整理View程式碼，提升可讀性與可維護性。

Partial通常用在HTML Template中需要重複使用的大塊Template區域，通常超過三行以上，例如像是表/選單，nav_bar， footer。

Helper則通常用於只有三到五行左右的HTML，有點像一個小單元而又需要重複使用的情況，或是未來格式可能會變動的物件，例如像是連結、按鈕、時間、圖片等等。

Ref:
使用時機：
http://blog.xdite.net/posts/2011/12/04/misunderstanding-about-render
Partial說明 :
https://ihower.tw/rails4/actionview.html#partials
http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials
Helper說明：
https://ihower.tw/rails4/actionview-helpers.html#helper
http://blog.xdite.net/posts/2011/12/09/how-to-design-helpers

##什麼時機可用 Service Object

當需要執行較複雜的機械式程式腳本，可能還有跨model的action時，會考慮將相關流程包裝成Service Object，可大大增加程式碼的可讀性與可維護性。

其他像是，需要使用到外部服務時，像是到其他社群網站發表貼文，也是使用時機。

儘管service object裡面可能會處理一連串的程式執行，但慣例上，一個Service object就是執行一個商業功能邏輯，如商品下訂、開立發票、寄密碼提醒信，通常是在controller裡呼叫一些instance variable輸入service object回傳結果。

使用與應用時機可參考：
[Gourmet Service Objects](http://brewhouse.io/blog/2014/04/30/gourmet-service-objects.html)
[Service objects in Rails will help you design clean and maintainable code. Here's how.](https://netguru.co/blog/service-objects-in-rails-will-help)
[Service Objects: What They Are, and When to Use Them](http://stevelorek.com/service-objects.html)