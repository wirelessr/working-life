# Blog onto GitHub

GitHub上有一些套件後可以提供user發網誌，能支援markdown實在是很開心的一件事，而且居然可以自定域名！只是免費用戶又不會參與open source，去使用自定域名好像太奢侈。

在SOP開始前，我還是要再前言一下，現在在GitHub上面架blog有很多套件，官方提供的Jekyll或是外包一層去仿wordpress的octopress，還有許多輕量級的，但我選擇hexo (台灣人寫的open source)。
 
當然我不是因為是台灣人寫的才選他，而是因為他最容易上手，在開始之前，GitHub帳號應該是必備的，並且在上面開一個username.github.com的repo，然後把烏龜git(TortoiseGit)裝起來，因為我是烏龜svn的愛用者，所以就選烏龜git了，還有很多選擇，可以選自己喜歡的，但windows上一定要裝git on windows。
 
裝完烏龜git他會提示你去裝，還要安裝node.js，這也很簡單，官網下載windows安裝版即可，以上說的所有安裝都無腦的next按到底就對了。
 
重頭戲開始，在你想要的地方創一個放blog資料夾，然後進去資料夾內部用滑鼠右鍵叫出Git bash：
> npm install -g hexo   
> hexo init   
> npm install    
> hexo generate   
> hexo server

此時在localhost:4000就已經把blog做好了，這樣你們應該都會知道我為什麼要選hexo了吧！非常無腦，我喜歡。
 
接著在`_config.yml`裡面編輯
```
deploy:
  type: github
  repository: https://github.com/username/username.github.com.git
 
  branch: master
```
最後在Git bash內
> hexo generate   
> hexo deploy

期間會要求你輸入帳號密碼甚麼的，照做就對了，等個十分鐘後就可以去username.github.com參觀了，第一次放上去會等十分鐘，之後就沒這麼久。





