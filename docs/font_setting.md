tag: Visual code markdown preview font setting for Chinese characters

```shell
fc-list :lang=zh
```

可以找到系统中已有的中文字体，可以看到有：

```shell
/usr/share/fonts/truetype/wqy-zenhei.ttc: WenQuanYi Zen Hei Sharp,文泉驛點陣正黑,文泉驿点阵正黑:style=Regular
/usr/share/fonts/truetype/wqy-zenhei.ttc: WenQuanYi Zen Hei Mono,文泉驛等寬正黑,文泉驿等宽正黑:style=Regular
```

快捷键 'ctrl+,'  调出visual code的setting界面，搜索markdown.preview.fontFamily，修改变量，参考已有格式，在最后添加“WenQuanYi Zen Hei Mono”，这样我们就能让markdown preview显示正确的中文。


如果想要添加其他字体，比如YaHei Consolas Hybrid，可以参考该[repo](https://github.com/GitHubNull/YaHei-Consolas-Hybrid-1.12)下载安装该字体，然后同样的在visual code的设置中修改markdown preview的字体即可。
