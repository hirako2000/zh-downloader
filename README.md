# Zh-Downloader

This is a Chrome (and Firefox) plug-in to perform video sniffing and download. Supports download as TS format (MPEG2-TS format) or MP4 format.


[![Chrome Web Store](https://developer.chrome.com/webstore/images/ChromeWebStore_BadgeWBorder_v2_206x58.png)](https://chrome.google.com/webstore/detail/zh-downloader/gcknejnpflbcggigdinhahgngfhaomik?utm_source=chrome-ntp-icon)



# Dev

After downloading this code, run  `npm install && npm run build` and select "Load unpacked extension in chrome `chrome://extensions/` select the `dist` directory.


# Dependencies

 Vue，[Kocal/vue-web-extension](https://github.com/Kocal/vue-web-extension)

+ [Vue2.0](https://vuejs.org/)
+ [Element](http://element.eleme.io/#/zh-CN)
+ [m3u8-parser](https://github.com/videojs/m3u8-parser)
+ [mpegts_to_mp4 (https://github.com/shellvon/mpegts)
+ [mux.js](https://github.com/videojs/mux.js)
+ [vuex-webextensions](https://github.com/MitsuhaKitsune/vuex-webextensions)

# Usage

General steps:

-  使用 [chrome.webRequest.onBeforeRequest](https://developer.chrome.com/webRequest) 监听所有知乎发起的视频请求，根据简单抓包可以看到知乎当有视频的时候会向这个地址:[https://v.vzuu.com/video/ZHIHU_VIDEO_ID](https://v.vzuu.com/video) 发起请求，其中`ZHIHU_VIDEO_ID`即为知乎视频ID. 之后利用知乎 API: https://lens.zhihu.com/api/videos/ZHIHU_VIDEO_ID 找到视频的 m3u8 地址。

- 利用 [M3U8-Parser](https://github.com/videojs/m3u8-parser) 解析上述 API 返回的数据，并提交数据至 Popup 页面进行查看。

- 提交数据利用了 Vue 的 Store 状态管理，由于 Popup 生命周期的原因，因此将 Store 也存入了 [chrome.storage](https://developer.chrome.com/apps/storage) 进行持久化, 插件是 `vuex-webextensions`。

- Popup(前端)接受用户下载请求，利用 `chrome.runtime.connect` 连接 Port 与 Background.js (后端)进行双向通讯，通知其下载请求。

- Background.js 遍历所有 TS 数据包并将起合并为一个 TS 包，如果发现下载的是 mp4 格式，则利用 `mpegts_to_mp4` 或则 `mux.js` 进行数据转化。

- 利用 Port 实时通知 Popup 页的下载进度以及下载结果。

> ⚠️ 注意:  MP4 尝试了`mpegts_to_mp4`, `mux.js`, `videoconverter.js` 效果均不是很理想。因此不建议下载MP4。

1. mpegts_to_mp4: ~~读取 SPS 信息的时候宽度/高度信息错误~~ (调研发现是没有去掉 [Emulation Prevention Bytes](http://blog.51cto.com/danielllf/1758115). 目前已经修复))
2. mux.js 能正常读取宽/高，但是无法正常解析Duration(See [videojs/mux.js#210](https://github.com/videojs/mux.js/issues/210))，另一个有趣的问题是部分知乎用户的视频没有音频，因此不会触发 mux.js 的 `data` 事件(See [videojs/mux.js#194](https://github.com/videojs/mux.js/issues/194))，因此需要分开处理音频/视频。
3. videoconverter.js  Node 直接就爆内存错误

# 限制

1. 下载过程中不能关闭 Popup 页，否则后端无法与之通信然后通知下载结果
2. 不知道视频链接过期时间, 因此下载过程中会出现403，这时候可以点击ID进入知乎将自动刷新
3. 转化的MP4格式宽高不正确，因此普通视频播放器可能难以播放, 请尝试用 `mplayer` 播放。 或者下载 TS 之后用 `ffmpeg` 或者 [https://cloudconvert.com/](https://cloudconvert.com/) 在线转换

# TODO: 

- [x] Customizable settings
- [x] Added open function for downloaded videos
- [x] Automatically delete videos collected over a certain period of time (time / strategy?)
- [ ] The user ignores some conditions of video capture (such as size / definition / author / video name)? ~~ (It seems useless, this function is not intended to be implemented)
- [x] Directly search for Zhihu videos (I don't know if there is an API) Zhihu recommended videos (can't search)
- [x] Fixed the problem of exporting MP4 format, whether it is `mux.js` or` mpegts_to_mp4`, either
- [x] Extension for Google Chrome 商店[Install from Google Chrome Store](https://chrome.google.com/webstore/detail/zh-downloader/gcknejnpflbcggigdinhahgngfhaomik?utm_source=chrome-ntp-icon)

# Change Logs

关于本项目的 Change Logs 您可以访问 http://von.sh/zh-downloader/#change-logs 查看详情 或者可以查看 `docs/changelog.json` 文件。
