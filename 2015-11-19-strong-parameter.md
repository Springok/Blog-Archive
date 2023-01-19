---
layout: post
title: '關於Strong parameter'
date: 2015-11-19 21:30
comments: true
categories: 
---
Strong parameter是Rail4 之後內建的一種安全機制，避免有心人士在填資料時，偷塞參數，惡搞資料庫，輕則資料被刪除竄改，重則資料外洩，造成安全問題。

##[Rails的參數(Parameter)](http://stackoverflow.com/questions/6885990/rails-params-explained)

由使用者瀏覽器端送出的請求(Request)包含著參數(parameter)再提供給伺服器端，伺服器端再依據請求中的這些參數做處理，產生頁面回傳，最常見的請求種類是HTTP GET，而參數就夾雜在URL中，舉例來說，如果使用者瀏覽器端發出請求的URL如下：
							http://www.example.com/?foo=1&boo=octopus
在Rails中的`params[:foo]`會是`1`，而`params[:boo]`就是`”octopus”`

在HTTP/HTML中，參數就是一連串的key-value對，key與value都是字串(String)，而Rails中的特殊語法會將這些參數包成Hash在包到另一個Hash當中，舉例來說，如果使用者瀏覽器端發出請求的URL如下：
							http://www.example.com/?vote[item_id]=1&vote[user_id]=2
`params[:vote]`就是一個hash物件，而`params[:vote][:item_id]`就會是`1`而`params[:vote][:user_id]`就會是`2`

##Rails中的[實作方式舉例by ihower實戰聖經](https://ihower.tw/rails4/basic.html#section-2)
```
def create
  @event = Event.new(event_params)
  @event.save

  redirect_to :action => :index
end

private

def event_params
  params.require(:event).permit(:name, :description)
end
```

像上述的例子，在private底下作一個`event_params` method來作Strong parameter，如此一來，當使用者提出請求呼叫controller#create，只能修改:name :description兩個參數，其他參數就算傳送到伺服器端也都會被擋下，無法取代原先在資料庫中的對應欄位資料。

##如何讓 strong_parameter 接受 nested_attributes
有時候因為專案需求會做[nested form](http://guides.rubyonrails.org/form_helpers.html#nested-forms)，而實作Strong parameter的方式步驟，一般而言如下：
	1. 在對應的`model`裡宣告`accepts_nested_attributes_for :nested_model`
	2. 修改新增`controller`裡的`private`底下的`model_params 的 permit ( :model_attr1, :model_attr2, nested_model_attributes: [:nested_model_attr1, :nested_model_attr2])`
參考資料：
     官方API文件說明:http://api.rubyonrails.org/classes/ActionController/Parameters.html
     [GitHub上的Rails原始碼](https://github.com/rails/rails/blob/9ab2d030209d9608a6c866d83210f5b3b7d2319e/actionpack/lib/action_controller/metal/strong_parameters.rb#L585)
     https://rocodev.gitbooks.io/rails-102/content/chapter1-mvc/c/strong_parameters.html
     http://guides.rubyonrails.org/action_controller_overview.html#strong-parameters