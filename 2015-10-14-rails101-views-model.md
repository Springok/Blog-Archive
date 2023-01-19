---
layout: post
title: 'Rails101學習筆記(1)-關於Model'
date: 2015-10-14 11:59
comments: true
categories: [rails]
---

*這篇是筆記型式，紀錄我在練習實作Rails101時遇到的一些問題或是觀念上卡關的地方，以供爾後複習時，快速回憶之用～*

##資料庫欄位名稱打錯了怎麼辦？
如果不小心在建立資料庫表格時，column名字取錯或打錯，可參考下面的方法更改：

Step1. 新增一新的migration
`rails g migration FixColumnName`

Step2. 在 db/migrate/xxxxxxxxxx_fix_column_name.rb中用rename_column的方式修改成正確的column name
```
class FixColumnName < ActiveRecord::Migration
  def change
    rename_column :table_name, :old_column, :new_column
  end
end
```
step3. 再跑rake db:migrate

`rake db:migrate`

http://stackoverflow.com/questions/1992019/how-can-i-rename-a-database-column-in-a-rails-migration
上面連結中的回答有提到兩種方式，我後來選了第二個方法，用了 `rake db:rollback`指令（輸入rake -T 可看相關指令），回溯資料庫後，重新產生一個新的Model覆蓋，但資料就無法保留，過程中，也會問你要不要overwrite原來的/app/models/xxxx.rb的檔案，之後有幾個選擇，可按h看進一步解說，這部分選擇就看個人了囉。

##Model關聯物件的新增刪除

model之間的關聯設定完成後，就可以利用Rails提供的method來作關聯物件新增刪除操作，這邊簡單記錄一些容易搞混的method。

###build vs. new vs. create

而new跟build基本上沒有差別，build只是new的alias，而create則是new完之後還會執行後續的SQL指令將資料寫入資料庫(就是save)。

若以Rails101中的程式碼說明，前兩行程式碼，group使用了build新增post，再把post的author給currentuser，而此時資料尚未存入資料庫(save動作是在按下頁面表格中Submit按鈕後才執行)。

而後續if conditional判斷式，則是作為是否成功執行save的判斷。

```ruby app/controllers/posts_controller.rb
def create
    @post = @group.posts.build(post_params)
    @post.author = current_user

  if @post.save
      redirect_to group_path(@group), notice: "新增文章成功！"
    else
      render :new
    end
end
```
###destroy vs. delete
執行delete會直接從Model中移除該關聯物件，並將其foreign keys設為null。
而destory除移除關聯物件外，還會執行相關的[Callback動作](http://guides.rubyonrails.org/active_record_callbacks.html#destroying-an-object)。

##Model關聯的設定選項（Association Reference）

model之間的關聯設定完成後，有時候做一些額外的設定，來輔助實作，簡單說明一些之前卡住的觀念題～。

###關於Foreign_key設定的運用時機

在建立belongs_to關聯性後，慣例上，Rails自動會將該model的foreign_key設為關聯Model的名字加底線id，如下面例子中Order model的foreign_key就應該是叫customer_id，但需要注意的是foreign_key欄位必須由自己手動建立，也就是在Order model中加入customer_id的欄位。
```
class Order < ActiveRecord::Base
  belongs_to :customer
end
```
但有時候，如下面的例子，會使用class_name設定Patron model作為實際關聯對象。

雖然一樣可以使用customer_id，但卻不太直覺，這時候可以客製化foreign_key的名稱為patron_id，然後把Order model中customer_id欄位名稱更改為patron_id，如此一來就直覺多囉。
```
class Order < ActiveRecord::Base
  belongs_to :customer, class_name: "Patron",
                        foreign_key: "patron_id"
end
```
###Counter_Cache
這個功能可以自動更新model的關聯物件數量(新增或移除物件時，會自動加一減一)，常常我們需要使用這個資訊來實作功能，藉由這個方式，就不用每次跑大量的SQL count查詢。

首先必須先自己在Model中手動新增一個欄位，以下面的例子來說，依照慣例，將新增欄位命名為orders_count再加入counter_cache: true，如此一來， @customer.order.size就會變成自動去讀orders_count的值，不會傻傻地再跑一堆SQL count查詢啦。

```
class Order < ActiveRecord::Base
  belongs_to :customer, counter_cache: true
end
class Customer < ActiveRecord::Base
  has_many :orders
end
```
如果需要的話，你也可以客製化counter_cache對應的欄位名稱，如下：
```
class Order < ActiveRecord::Base
  belongs_to :customer, counter_cache: :count_of_orders
end
class Customer < ActiveRecord::Base
  has_many :orders
end
```