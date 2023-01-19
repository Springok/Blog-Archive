---
layout: post
title: 'Rails如何使用Carrierwave上傳圖片'
date: 2015-10-21 16:19
comments: true
categories: [rails, gem]
---
###這篇介紹如何使用Carrierwave實作圖⽚上傳功能與並以mini_magick產生不同尺寸的數張圖片。

目前[carrierwave](http://www.rubydoc.info/gems/carrierwave/0.10.0)是上傳圖片時普遍選用的gem，而且有很多功能可以玩，例如可整合另一個自動裁切圖片大小的gem叫mini_magick，下面簡單記錄我實作的一些步驟與設定。

另外要使用[mini_magick](https://github.com/minimagick/minimagick)前，須事先安裝image magick，如果是mac環境底下，應該跑一下下面指令就可以完成安裝了。
```
$ brew install imagemagick
```
不知道自己有沒有裝，跑一下
```
$ convert -version
```
OK的話就會出現類似下面的畫面：
```
$ convert -version
Version: ImageMagick 6.8.9-7 Q16 x86_64 2014-09-11 http://www.imagemagick.org
Copyright: Copyright (C) 1999-2014 ImageMagick Studio LLC
Features: DPC Modules
Delegates: bzlib fftw freetype jng jpeg lcms ltdl lzma png tiff xml zlib
```

##情境說明：

###User story：
賣方在網站上架商品時，要可以上傳實物照片。

###Model：
- Product
- Photo

##實作步驟：

1. 安裝Gem
2. 設定model與建立關係
3. 在Rails console確認關係
4. 設定上傳圖片時，同時切成各種尺寸的圖片供使用
5. 設定.gitignore (選擇性步驟，但建議作)
6. 修改controller與頁面
7. 上傳看看吧~

###*Step1. 安裝Gem*
安裝跟其他gem一樣，在Gemfile新增兩行
``` ruby Gemfile
gem 'carrierwave'
gem 'mini_magick'
```
再執行
```
$ bundle install
```
然後新增一個Uploader，命名為image（舉例說明，可依需求命名）
```
$ rails generate uploader image
```

###*Step2. 設定model與建立關係*

產生一個model, photo用來存放照片
```
$ rails g model photo product_id:integer image:string
# product_id 是foreign_key, image則是之後給mount_uploader的欄位
```
跑migration
```
$ rake db:migrate
```
接著Photo中加入關係與mount_uploader

``` ruby photo.rb
class Photo < ActiveRecord::Base
  belongs_to :product

  mount_uploader :image, ImageUploader
end
```
在Product中也做相關宣告
``` ruby product.rb
class Product < ActiveRecord::Base
  has_many :photos, dependent: :destroy

  accepts_nested_attributes_for :photos  
  #之後我們要做nested form，先在這邊設定接受變更Photo底下的attributes
end
```

###*Step3. 在Rails console確認關係*

首先需要修正無法在 rails c 讀取到 uploader.rb 的問題:
``` ruby config/appliaction.rb
module Artstore
  class Application < Rails::Application
   …(略)
    config.active_record.raise_in_transactional_callbacks = true
+    config.autoload_paths += %W(#{config.root}/app/uploaders)
  end
end
```

接下來在Rails c輸入Product.first.photos應該就可以確認結果，如下結果就是成功，但因為目前我們還沒上傳任何圖片，回傳的是[ ]，而到目前為止圖片上傳功能的Model端設定已經OK。
![9B07205E-31AE-4B76-8290-CEBD08888A06.png](http://user-image.logdown.io/user/13199/blog/12437/post/305754/NK53swIQR4mTFEFre00S_9B07205E-31AE-4B76-8290-CEBD08888A06.png)

###*Step4. 設定上傳圖片時，同時切成各種尺寸的圖片供使用*

然後編輯你的image_uploader.rb 讓MiniMagick可以用還可將照片切成各種尺寸
``` ruby app/uploaders/image_uploader.rb
class ImageUploader < CarrierWave::Uploader::Base
…(略)

+ include CarrierWave::MiniMagick #設定使用minimagick
…(略)

+  process resize_to_fit: [800, 800] #圖片上傳後，自動切成你要的size

+ version :thumb do #設同時切其他size的版本-thumb
+    process resize_to_fill: [200,200]
+  end

+ version :medium do #設同時切其他size的版本-medium
+    process resize_to_fill: [400,400]
+  end

end
```

###*Step5. 設定.gitignore (選擇性步驟，但建議作)*
上傳處理後的圖片自動存放在public/uploads底下，而建議將public/uploaders 資料夾放入 .gitignore，因為commit這些上傳圖片的變更，事實上也不太有用。
``` ruby .gitignore
   …(略)
+ public/uploads
```

###*Step6. 修改Controller與頁面*
完成以上步驟後，接下來就只要在Controller與頁面加入對應的上傳欄位就可以啦～
以我練習的專案來說，是在新增商品的步驟做圖片上傳，所以在Product#new底下，除了原本的@product之外，多宣告一個@photo，並且在product_params的底下允許存取image這個欄位，像這樣：

```ruby
 def new
    @product = Product.new
    @photo = @product.photos.new
  end

…(略)

private
 def product_params
    params.require(:product).
    permit(:title, :description, :quantity, :price, photos_attributes: [:image]) 
    # 配合先前在Product model中的設定，使用nested_attributes的設定方式，通過驗證。

  end
```
然後在對應的new.html.erb，加入該上傳欄位：
```html new.html.erb
…(略)

  <div class="form-group">
    <%= f.simple_fields_for :photos do |c| %>
      <%= c.input :image, as: :file %>
    <% end %>
  </div>
# 這邊是用我自己在練習的專案作說明，使用simple_form，
# 然後是在已有的表單底下，增加額外的欄位，所以用的是fields_for指令，
# 而要從本地端電腦上傳圖片，所以是as: :file。
```

或者你也可以給定圖片所在的遠端URL，來上傳圖片，CarrierWave已經很貼心的幫我們做好整合，也有固定的命名方式，所以只要更改上傳欄位的名稱為 :remote_xxx_url，並在product_params的底下允許存取remote_xxxx_url這個欄位，就OK啦，像這樣：
```html new.html.erb
…(略)

  <div class="form-group">
    <%= f.simple_fields_for :photos do |c| %>
      <%= c.input :image, as: :file %> # 使用本地端上傳，手動更改type為File
      <%= c.input :remote_image_url%>  # 使用遠端url上傳，使用預設type即可(String)
    <% end %>
  </div>

```
產生的頁面會長這樣：
![Screenshot 2015-10-21 20.02.36.png](http://user-image.logdown.io/user/13199/blog/12437/post/305754/07pM2I2T3WAX5LTROIBc_Screenshot%202015-10-21%2020.02.36.png)

###*Step7. 上傳看看吧~*
在頁面中選擇其中一種方式上傳後，專案底下public/uploads/photos/image應該就會有三張不同尺寸的圖片囉～

之後如果要在頁面中使用圖片路徑，像下面使用就可以指向圖片所在位置把對應圖片叫出來囉～

```ruby
Photo.first.image.url        =>"/uploads/photo/image/1/xxx.jpg"        # size: 800x800
Photo.first.image.thumb.url   =>"/uploads/photo/image/1/thumb_xxx.jpg" # size: 200x200
Photo.first.image.medium.url =>"/uploads/photo/image/1/medium_xxx.jpg"  # size: 400x400
#以Photo的第一張圖片舉例。
```

以上簡單整理供參考，有問題也可一起討論～感謝！
##參考資料
Railscast的carrierwave教學影片
https://www.youtube.com/watch?v=YpF_4uciMvg
Carrierwave裁切各種不同版本的圖片設定
https://github.com/carrierwaveuploader/carrierwave#adding-versions
Carrierwave如何設定遠端上傳圖片
https://github.com/carrierwaveuploader/carrierwave#uploading-files-from-a-remote-location
