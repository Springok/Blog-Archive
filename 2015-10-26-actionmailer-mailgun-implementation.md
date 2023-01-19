---
layout: post
title: 'ActionMailer與Mailgun實作'
date: 2015-10-26 21:18
comments: true
categories: [rails, gem]
---
最近練習專案中有用到ActionMailer，簡單記錄一下使用方式與連接的第三方SMTP寄信服務。

利用E-mail發送各種資訊給網站會員或電子報訂閱者，已經是各網站必備的服務功能，而Rails中的ActionMailer也幫我們做了很好的整合。

一旦我們做好網站寄送Email的相關規劃，如寄送時機、內容等等，就可以利用ActionMailer來自動發信，舉例來說，當消費者在購物網站下單後，寄送訂單確認信，或是用戶註冊後，寄送歡迎信等等。

透過下面幾個步驟就可以讓我們設定好網站寄信的功能：

  1. 建立Mailer
  2. 編輯對應樣版(Template)
  3. 設定寄信者
  4. 在本機預覽Mail與套用CSS格式
  5. 連結第三方SMTP寄信服務(Mailgun)

###Step1. 建立Mailer
跟建立Controller很類似，首先我們先建立一個Mailer，而這個Mailer類別(class)中的一個方法(method)也會對應到一個樣板(template)，舉例來說，產生一個OrderMailer的指令如下：
```
$ rails g mailer ordermailer
```
接著編輯ordermailer，例如新增寄發訂單確認信的方法(method)：
``` ruby app/mailer/order_mailer.rb

def notify_order_place(order)  # 當這個方法被呼叫時，我們希望可以得到order的資訊
     @order = order
     @user = order.user
     @order_items =@order.items
     # 從order裡萃取出等等所需要用的各個實例變數(instance variable)

     mail(to: @user.email, subject: "[ECstore] 感謝您的訂單，以下是您這次的訂單商品明細 #{order.id}")
     # 最後設定收件者與標題並寄送Email
end
```

###Step2. 編輯對應樣版(Template)

``` html app/views/order_mailer/notify_order_placed.html.erb

<div class="row">
  <div class="col-md-12">
    <h2>
      訂單明細 <br>
      <small>
        <%= link_to("訂單連結", order_url(@order.id)) %>
      </small>
    </h2>
    <table class="table table-bordered">
      <thead>
        <tr>
          <th width="70%">商品明細</th>
          <th>單價</th>
          <th>數量</th>
          <th>小記</th>
        </tr>
      </thead>
      <tbody>

        <% @order_items.each do |order_item| %>
          <tr>
            <td>
              <%= order_item.product_name %>
            </td>
            <td>
              <%= order_item.price %>
            </td>
            <td>
              <%= order_item.quantity %>
            </td>
            <td>
              <%= order_item.quantity * order_item.price %>
            </td>
          </tr>
        <% end %>

      </tbody>
    </table>
   </div>
</div>
```
### Step3. 設定寄信者
```ruby app/mailer/application_mailer.rb
     default from: “servicenoreply@service.com"
     # 可將預設的寄信者設定改掉，以符合使用需求
```

### Step4. 在本機預覽Mail與套用CSS格式
接下來，我們希望能在本機開發端預覽Mail，確認Mail的內容與格式，同時也確認OrderMailer這個Actionmailer運作正常，為了達到預覽目的，需求搭配使用一個Gem叫letter_opener。
首先安裝Gem：
``` ruby Gemfile

group :development do
     gem "letter_opener"
end
```
接下來跑`bundle install`，然後在開發環境的config設定：
``` ruby config/environments/development.rb
…(略)
config.action_mailer.default_url_options = { host: 'localhost:3000’ } #本機端
config.action_mailer.delivery_method = :letter_opener

```
接下來在Rails c底下輸入：
```
>> OrderMailer.notify_order_placed(Order.last).deliver!
#抓出最後一筆訂單資料，並寄送訂單確認信
```
如此一來，瀏覽器應該會就開一個新分頁，顯示發送mail的內容如下：
<div class = "center" >
<img src ="http://user-image.logdown.io/user/13199/blog/12437/post/306492/Fxg8NHVQCiTDpPg9kKcO_mailer1.png">
</div>
但可以看出信件格式好像怪怪的，是因為Mailer的樣板並沒有套用CSS的關係，這封mail如先前編輯的是一份HTML格式的文件，如果要讓收信者能看到相關CSS的呈現，勢必也要讓其擁有相關的CSS設定，在這邊可借助`Roadie`這個Gem達到目的。

首先安裝Gem
``` ruby Gemfile

group :development do
     gem "letter_opener"
 +   gem “roadie"
end
```
接下來跑`bundle install`，在樣式底下加入
``` ruby app/views/order_mailer/notify_order_placed.html.erb
+    <link rel="stylesheet" type="text/css" href="/assets/application.css" />
     # 加入CSS檔案的所在位置連結
<div class="row">
  <div class="col-md-12">
    <h2>
      訂單明細 <br>
      <small>
        <%= link_to("訂單連結", order_url(@order.id)) %>
      </small>
…(略)
```
再寄送一次試試：

<img src="http://user-image.logdown.io/user/13199/blog/12437/post/306492/21vt3vxeT7WfGmL7oURq_mailer2.png">

已經套用了基本的表格了。

###Step5. 連結第三方SMTP寄信服務(Mailgun)

而寄信方式的選項有:test, :sendmail, :smtp。
其中[SMTP (Simple Mail Transport Protocol)](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol)又稱"簡單郵件傳輸協議" 是電子郵件傳輸的標準協定，是比較推薦的寄信方式，用Gmail就可以[實作](https://ihower.tw/rails4/actionmailer.html#actionmailer)，但通常網站服務都需要大量寄信，自架mail server通常不會是個人選項，這時候就可以利用一些[第三方的寄信服務](http://www.yogoeasy.com/mandrill-mailgun-guide/)，這篇則是以Mailgun說明。

首先，到[官網](https://mailgun.com/signup)去申請帳號，免費帳號每個月的前一萬封mail免費，註冊完之後，點擊上方Domains進入後，可看到一預設的domain，在點擊後，進入如下的顯示各資訊的畫面，利用這些資訊在Rails中做相關設定。
<div class = "center" >
<img src="http://user-image.logdown.io/user/13199/blog/12437/post/306492/Ot3BwWApTfmQx59c6HPt_mailer3.png">
</div>
到Rails環境底下做mail寄送的設定：
``` ruby config/environments/development
…(略)

config.action_mailer.default_url_options = {  host: 'localhost:3000’ }
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
    address:              'smtp.mailgun.org',
    port:                  587,
    domain:               'sandboxb50419f43d7a4fe590dda7e73ea0896a.mailgun.org',
    user_name:             ENV['SMTP_USERNAME'],
    password:              ENV['SMTP_PW'],
    authentication:       'plain',
    }
…(略)

```
相關帳密利用(1)環境變數設定或(2)設定在yml檔中，並將該yml檔加入.gitignore，避免帳密外洩。

這樣就設定完成了，把收件者設成自己的Email，然後寄信試試看吧～

PS:在部署環境(Production)下的設定方式也相同，而帳密利用環境變數做設定的，要記得在部署環境中加入相關變數，才讀得到值。

Ref:
ActionMailer
https://ihower.tw/rails4/actionmailer.html
http://guides.rubyonrails.org/action_mailer_basics.html
Letter_Opener
https://github.com/ryanb/letter_opener
Roadie
https://github.com/Mange/roadie
第三方SMTP寄信服務介紹
http://www.yogoeasy.com/mandrill-mailgun-guide/