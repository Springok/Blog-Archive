---
layout: post
title: 'Rails101學習筆記(3)-Others'
date: 2015-10-20 00:06
comments: true
categories: [rails]
---
##Router相關

在Rails開發框架中的Router，主要功用有三個，其一是辨識HTTP request的URL網址，再丟到對應的Controller#action去處理，其二，在Rails中有一些慣例，Router就依據這些慣例規則產生路徑helper與URL，也免去在view直接指定URL的瑣事，最後，Router也可以開發者的需求，客製化設定一些路徑與URL，而以上這些都可以透過編輯config/routes.rb這個檔案來達成，如何正確地撰寫這些生成路徑規則與客製化設定，絕對是Rails開發中最需要把握的環節之一。

提供一常見Rails面試[題目](http://5xruby.tw/interview_questions)是來自5倍紅寶石，可用來檢視自己對於相關規則是否了解。

另外愛用`$rake routes`，可以檢查目前設定規則，包含URL helper, URL網址與對應的Controller action。

下面簡單筆記一些Rails101中用到的routes.rb設定

###namespace
Namespace是Scope的一種特定應用，特別適合例如後台介面，這樣就整組controller、網址path、URL helper前置名稱都影響到：

```ruby config/routes.rb
namespace :admin do
  resources :projects
end
```
如此對應的controller會是Admin::ProjectsController，URL網址則如/admin/projects，而URL Helper如admin_projects_path這樣的形式。

### member & collection
如果是要產生路徑對應至某個特定元素操作的自訂Action，像Rails101s中groups#quit與groups#join：

```ruby config/routes.rb
resources :groups do
    member do
      post :quit #退出討論版
      post :join #加入討論版
    end
    resources :posts
  end
```
就可產生網址格式：
- groups/[:id]/quit
- groups/[:id]/join

URL helper：
- quit_groups_path(@group)
- join_groups_path(@group)

而如果是要產生路徑至群集操作的自訂Action，用法如下：

```ruby config/routes.rb
resources :photos do
  collection do
    get :search  #對應到photos#search
  end
end
```
就可產生網址格式：
- photos/search

URL helper：
- search_photos_path

##Gemfile
Rails專案中所需要使用的Gem都列在這個檔案中[(ihower實戰聖經的寫法說明)](https://ihower.tw/rails4/environments-and-bundler.html#bundler--gemfile-)，而每次修改Gemfile後，務必記得執行`bundle install`，bundler就會檢查現有並安裝新增的Gem，同時產生一個 Gemfile.lock 檔案，當中會詳列出使用gem的版本，把這個檔案納入git commit的對象中，則可讓其他開發者與上線版本使用相同的gem版本了。

另外在gem後面加上 "require: false" 代表在跑 rails server 的時候不會去呼叫到這個 gem，幫助你在開發環境啟動 server 時能較快速運作，未來只要新安裝的 gem 若不會用在 server 運作上，都可以加入這段來減少無謂的 loading。