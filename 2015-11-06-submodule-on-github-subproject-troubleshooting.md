---
layout: post
title: 'GitHub上Submodule(Subproject)問題排除'
date: 2015-11-06 12:39
comments: true
categories: [git]
---
##問題描述與解決過程：
*[11/20Update補充說明]
如果是用Submodule的方式來做，會如下圖所示，看不到底下的相關檔案，因此後來是先把資料夾中的.git資料夾整個砍掉，再依下面步驟，回到母層的資料夾的git repo作commit重新push上GitHub。*

今天把自己練習的專案推上GitHub發生如下的問題：
![submodule1.png](http://user-image.logdown.io/user/13199/blog/12437/post/308218/7TdiKboEREyOFgX6Q7XX_submodule1.png)
資料夾無法點選，因此也看不到底下的相關檔案，點擊旁邊的Complete 2 practices進去，拉到最底下，看到：
![submodule2.png](http://user-image.logdown.io/user/13199/blog/12437/post/308218/BB7YpHosTlahtpvdSlTI_submodule2.png)

我對這個Subproject所代表的意義沒有概念，於是Google了一下，發現就是Git Submodule，也在[stack overflow](http://stackoverflow.com/questions/21088781/what-does-subproject-commit-mean)上看到有人遇到類似的問題，試了在專案底下，輸入下面指令，想說這樣應該就可以看到這個submodule repo底下的各個檔案：
`$ git submodule update —-init`
結果得到下面訊息:
`No submodule mapping found in .gitmodules for path ‘Udemycourse'`

看來我根本就沒有建立過這個路徑，然後又用這個訊息google看到另一篇[Stack overflow]
(http://stackoverflow.com/questions/4185365/no-submodule-mapping-found-in-gitmodule-for-a-path-thats-not-a-submodule/13394710#13394710)，於是再輸入下面指令：
`$ git rm --cached Udemycourse`
出現結果訊息 `rm 'Udemycourse'`

再輸入`$ git status`指令確認，出現：
![submodule3.png](http://user-image.logdown.io/user/13199/blog/12437/post/308218/UuOLlqvITZenkheofyz9_submodule3.png)

看來問題解決了，重新commit一次再push上GitHub，想說OK了，結果：
![submodule4.png](http://user-image.logdown.io/user/13199/blog/12437/post/308218/4hYx80yfRNWMqQhVydQf_submodule4.png)
看起來被reject了，但因為這個專案都是一些練習而已，我知道應該不會發生什麼大問題，因此，我用force-pushed bypass掉了這個問題，順利的把專案push上去GitHub。

再上去GitHub看，果然顯示了：
![submodule5.png](http://user-image.logdown.io/user/13199/blog/12437/post/308218/uCJT5z1FSMeWtrnJMUmV_submodule5.png)
但已經可以看到底下的檔案了，Case Closed！
我想以後重要的專案還是避免這樣搞，了解Reject的原因，排除之後，再push會比較好。

##結論：
其實發生問題的原因我其實不是很確定，猜想可能是之前已經在該資料夾生成一個local repo，過了幾天，我又直接在其母層資料夾init repo，然後commit一些新做的練習，push上GitHub才發現這個問題，不過趁這個機會也知道了git submodules這個東西的存在，目前還沒實際操作過，但也收集了一些相關資訊在底下供未來參考囉。

##Submodule參考資料：
Git 工具 - 子模組 (Submodules)
https://git-scm.com/book/zh-tw/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E7%B5%84-Submodules
Git Submodule 介紹與使用
http://blog.wu-boy.com/2011/09/introduction-to-git-submodule/
Git Submodule 用法筆記
http://blog.chh.tw/posts/git-submodule/