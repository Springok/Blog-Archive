---
layout: post
title: '在Heroku部署Rails專案結合AWS S3存放上傳圖片'
date: 2015-10-29 15:30
comments: true
categories: [deploy, heroku]
---
##關於Heroku
Heroku是一種PaaS（Platform as a Service）服務，特點是部署簡單，藉由Git push即可實作。

一般而言，對於用量不大的App，使用Heroku提供的免費方案就已足夠，而對於用量較大的用戶，Heroku提供的擴充方案，是透過購買額外Dynos，可以想像Dyno就是一台用來跑你專案的機器，多個dynos就讓專案同時可以處理更多的流量與需求。

另外當你的專案一段時間都沒人去用的時候，Heroku會自動關閉跑你專案的dyno，進入睡覺模式，而當你再次使用時，就會有一段前置的等待時間，大約一分鐘，如果不希望你的專案睡覺，可以額外購買不會關閉的dyno或使用一些第三方服務（像是NewRelic），定時去ping你的專案網站。

##將專案部署至Heroku
這部分在[RailsBrigde](http://railsbridge-docs-zh-tw.herokuapp.com/%E8%A3%9D%E6%A9%9F%E8%B6%B4-installfest/%E9%96%8B%E6%96%B0_heroku_%E5%B8%B3%E8%99%9F)與[Rails Girls Guides](http://railsgirls.tw/heroku/)都有圖文並茂的解說，強力推薦參考。

**Step0. 完成前置作業：**
- 安裝與設定Git
- 建立SSH key
- 申請Heroku帳號、安裝Heroku toolbelt並將SSH key加入

**Step1. 確認Rails專案的Gemfile設定**

推薦設定前先完成git commit，之後配合部署的更動再使用另一個commit，方便除錯。

``` ruby Gemfile
group :development, :test do
  gem 'sqlite3'
end

group :production do
  gem 'pg'
  gem 'rails_12factor'
end
```
Rails預設下，可依development、test、production環境不同使用不同的資料庫，Heroku用的資料庫是PostgreSQL，因此配合在production環境底下安裝上述兩個Gem，並將development環境下使用的SQLite區隔，而同時在同一個環境下使用`sqlite3`與`pg`會發生錯誤。

**Step2. 確認Rails專案完成Git commit**

**Step3. `$ heroku create`**
指令跑完git設定的remote repo會多一個heroku，才可進行下一步push

**Step4. `$ git push heroku master`**
把你的專案push到Heroku，同時Heroku會確認專案中的Ruby版本與跑｀bundle install｀

**Step5. `$ heroku run rake db:migrate`**
在Heroku端建立起資料庫

**Step6. `$ heroku open`**
一切OK之後，輸入上述指令，Heroku就會啟動dyno來跑專案，並自動在瀏覽器打開你的專案

##常用的Heroku設定與指令

你可以(1)透過CLI或是(2)直接上Heroku網站的管理後台做一些跟專案有關的Heroku環境或參數設定，像是更改專案名稱、加入環境變數、管理SSH keys、設定第三方服務等等。

個人兩種方式都用過，舉例來說，更改專案名稱，Heroku在`heroku create`時會自動為專案命名，部署完成後，我們就可以更改專案名稱，如果用CLI做設定，只要key入`$ heroku apps:rename newname`，如此一來就完成更動，連Git remote heroku的設定也會做更新。

而根據個人經驗，如果你是在管理後台更改專案名稱為newname，回過頭來，就還要在本地端手動更正git remote heroku的位置，不然之後push新的專案版本的時候就會發生錯誤。
`$ git remote remove heroku`
`$ git remote add heroku git@heroku.com:newname.git `

而我現階段做法都是用CLI指令設定後，再上管理後台做double check，確保更動無虞。

另外，通常在Heroku上跑rails的相關指令，都需要在前面加上`heroku run`
- `$ heroku run rake db:migrate`
- `$ heroku run console` #使用上要非常小心，因為你處理的是實際在運作的資料庫
- `$ heroku logs`
- `$ heroku restart`

更多的指令可以參考[官方指南](https://devcenter.heroku.com/categories/command-line)

##部署上常見錯誤(errors)與除錯
Heroku上的HTTP router如果遇到unstyled HTML就會丟503(Service Unavailable)，這種情況發生在專案執行時遇到系統性錯誤(system-level error)或是maintenance模式被啟動中，而如果是遇到404或500，則是呈現部署專案中的錯誤頁面。

錯誤發生，通常都會去看[歷史紀錄](https://devcenter.heroku.com/articles/logging#types-of-logs)(`$ heroku log`)，確認發生了什麼事，而常見的問題如下：

在部署時發生問題，Heroku底下，Rails運行的是production模式，所以專案中production環境設定的相關設定檔，如`config/environments/production.rb` 或是`Gemfile`中Gem的設定就要特別小心。

專案運行時，遇到status 500系列的錯誤，如果是在使用第三方服務出現問題，檢查看看是不是串連的第三方服務API的環境變數漏了加在Heroku，或是應該API端配合的設定沒有做或忘了變更成production的設定。

##結合AWS S3存放上傳圖片
如果專案需要使用到圖片上傳或類似相簿的功能，由於Heroku目前無法支援，而且空間容量也不夠大，因此通常會搭配其他服務如[AWS S3](http://aws.amazon.com/s3/)來實作這些檔案上傳的功能，而Heroku也提供了[操作指南](https://devcenter.heroku.com/articles/s3#overview)供參考。

而這邊紀錄一下個人的實作經驗，我自己的專案是想實作圖片上傳的功能，在開發時，是把檔案存在本地端，也就是public/uploads底下，而部署之後希望能放到S3上，實作步驟大概如下：

**Step1. 申請amazon帳號與啟用S3服務並新增一個Bucket**
申請完 AWS 帳號後到帳號名稱的下拉選單裡面點「Security Credentials」，就可以看到有個「＋Access Keys」這邊可以申請Access key ID，專案中的設定需要用到。

**Step2. Gemfile加入gem 'fog'與執行`bundle install`**

**Step3. 如果使用carrier wave實作上傳功能，相關設定可直接參考[這篇](https://github.com/carrierwaveuploader/carrierwave#fog)**

而個人練習專案中的設定如下
```ruby image_uploader.rb
 …(略)
  if Rails.env.production?
  storage :fog
  else
  storage :file
  end
 …(略)
```
```ruby carrier_wave.rb
CarrierWave.configure do |config|
  if Rails.env.production?
    config.fog_credentials = {
      provider:              'AWS',
      aws_access_key_id:     ENV['S3_KEY'],
      aws_secret_access_key: ENV['S3_SECRET'],
      region:                'ap-northeast-1'
    }
    config.fog_directory  = ENV['S3_BUCKET']
  else
    config.storage :file
  end
end
```

值得一提的是region的設定方式，並不是直接看Amazon S3上bucket的region,像選tokyo,但程式裡的region設定應該是`ap-northeast-1`，可參考[這份文件](http://docs.aws.amazon.com/general/latest/gr/rande.html)

**Step4. 如果是使用env的方式設定amazon帳密，記得上Heroku管理後台去新增更新env，或在CLI直接下指令也行。**
`$ heroku config:set AWS_ACCESS_KEY_ID=xxx AWS_SECRET_ACCESS_KEY=yyy`

**Step5. 這樣應該就OK了，到專案網址試著上傳圖片後，再到AWS確認看看吧～**

參考資料:
關於Heroku:
https://devcenter.heroku.com/articles/how-heroku-works
http://www.openfoundry.org/tw/tech-column/8573-heroku-the-best-cloud-platform-on-ruby-language

Heroku官方使用指南：
https://devcenter.heroku.com/articles/getting-started-with-ruby#introduction

實作AWS上傳文章參考：
http://yulin-learn-web-dev.logdown.com/posts/259970-rails-app-how-to-access-aws-s3-service
