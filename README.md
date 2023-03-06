# Image Explorer

基於Server Directory Listing功能製作的圖片瀏覽器

搭配[RCX - Rclone for Android](https://play.google.com/store/apps/details?id=io.github.x0b.rcx&hl=zh_TW&gl=US)使用

# Try it out!

Go to https://k8188219.github.io/image-explorer/

# 主要功能

* 透過Server Directory Listing切換資料夾
* 將當前資料夾內圖片直條排列顯示
* 圖片lazy loading
* 縮放圖片 [Ctrl + 滾輪]、雙指手勢
* 可辨認圖片副檔名 jpg jpeg jfif jfi jif png bmp gif svg webp

# Server Directory Listing

example:
http://archive.ubuntu.com/ubuntu/

目前可用

* Apache `Options +Indexes`
* Nginx `autoindex`
* Rclone `rclone serve http`
* Live Server
