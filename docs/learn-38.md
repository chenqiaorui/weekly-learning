#### 概念说明
chromedp是类似selenium的库，可以操作chrome去做截图等动作。

无头浏览器，即Headless Chrome，它是 Chrome 浏览器的无界面形态,可以在不打开浏览器的前提下，使用所有 Chrome 支持的特性运行您的程序。可以像在其他现代浏览器里一样渲染目标网页，并能进行网页截图，获取cookie，获取html等操作。

#### 场景一：windows下使用chromedp打开百度访问导航页
前提：
- 已安装go
- 本地有chrome浏览器

步骤：
```
mkdir chromedp-demo
cd  chromedp-demo && go mod init chromedp-demo # 创建 go.mod文件
go get -u github.com/chromedp/chromedp # 安装chromedp，生成go.sum

创建main.go文件，内容如下：

package main

import (
    "context"
    "log"

    "github.com/chromedp/chromedp"
)

func main() {
    opts := append(chromedp.DefaultExecAllocatorOptions[:],
        chromedp.Flag("headless", false), // 禁用chrome headless
		chromedp.Flag("enable-automation", false), // 不出现"Chrome正受到测试软件的控制"
    )
    allocCtx, _ := chromedp.NewExecAllocator(context.Background(), opts...)
    
	ctx, _ := chromedp.NewContext(
		allocCtx,
		chromedp.WithLogf(log.Printf),
	)
	_ = chromedp.Run(ctx, chromedp.Tasks{
		chromedp.Navigate("https://www.baidu.com/"),
		chromedp.SendKeys("//*[@id=\"kw\"]", "落花流水春去也", chromedp.BySearch),
	})
}
```
执行：`go run main.go`

#### 场景二：Linux下使用chromedp，只对单个元素(某张图片)截图 + 全网站截图
因为用到cheome，使用docker安装Headless Chrome(无头浏览器)。所谓“无头浏览器”，即可以在不开启UI视图的情况下如完成截图的功能。

编写安装chrome的Dockerfile
```
FROM debian:buster-slim

LABEL name="chrome-headless" maintainer="Ricky Chen <809155736@qq.com>"

COPY sources.list /etc/apt/sources.list

# Install deps + add Chrome Stable + purge all the things
RUN apt-get update && apt-get install -y \
	apt-transport-https \
	ca-certificates \
	curl \
	gnupg \
        net-tools \
        procps \
        vim \
	--no-install-recommends \
	&& curl -sSL https://dl.google.com/linux/linux_signing_key.pub | apt-key add - \
	&& echo "deb https://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list \
	&& apt-get update && apt-get install -y \
	google-chrome-stable \
	fontconfig \
	fonts-ipafont-gothic \
	fonts-wqy-zenhei \
	fonts-thai-tlwg \
	fonts-kacst \
	fonts-symbola \
	fonts-noto \
	fonts-freefont-ttf \
	--no-install-recommends \
	&& apt-get purge --auto-remove -y curl gnupg \
	&& rm -rf /var/lib/apt/lists/*

# Add Chrome as a user
#RUN groupadd -r chrome && useradd -r -g chrome -G audio,video chrome \
#	&& mkdir -p /home/chrome && chown -R chrome:chrome /home/chrome

# Run Chrome non-privileged
#USER chrome

# Expose port 9222
EXPOSE 9222

ENV TZ Asia/Shanghai

# Autorun chrome headless with no GPU
ENTRYPOINT [ "google-chrome" ]
CMD [ "--headless", "--no-sandbox", "--disable-gpu", "--remote-debugging-address=0.0.0.0", "--remote-debugging-port=9222" ]

```
说明：
- `--no-sandbox` 不开启沙盒模式可以减少对服务器的资源消耗,但是服务器安全性降低
- `--remote-debugging-address` 远程调试地址

更换apt为阿里源，编写sources.list
```
deb http://mirrors.aliyun.com/debian/ bullseye main non-free contrib

deb-src http://mirrors.aliyun.com/debian/ bullseye main non-free contrib

deb http://mirrors.aliyun.com/debian-security/ bullseye-security main

deb-src http://mirrors.aliyun.com/debian-security/ bullseye-security main

deb http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib

deb-src http://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib

deb http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib

deb-src http://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib
```

运行容器：
```
docker run -d -p 9222:9222 --rm --name headless-shell chromedp/headless-shell
```
编写main.go
```
// Command screenshot is a chromedp example demonstrating how to take a
// screenshot of a specific element and of the entire browser viewport.
package main

import (
	"context"
	"log"
	"os"

	"github.com/chromedp/chromedp"
)

func main() {
	// create context
	ctx, cancel := chromedp.NewContext(
		context.Background(),
		// chromedp.WithDebugf(log.Printf),
	)
	defer cancel()

	// capture screenshot of an element
	var buf []byte
	if err := chromedp.Run(ctx, elementScreenshot(`https://pkg.go.dev/`, `img.Homepage-logo`, &buf)); err != nil {
		log.Fatal(err)
	}
	if err := os.WriteFile("elementScreenshot.png", buf, 0o644); err != nil {
		log.Fatal(err)
	}

	// capture entire browser viewport, returning png with quality=90
	if err := chromedp.Run(ctx, fullScreenshot(`https://brank.as/`, 90, &buf)); err != nil {
		log.Fatal(err)
	}
	if err := os.WriteFile("fullScreenshot.png", buf, 0o644); err != nil {
		log.Fatal(err)
	}

	log.Printf("wrote elementScreenshot.png and fullScreenshot.png")
}

// elementScreenshot takes a screenshot of a specific element.
func elementScreenshot(urlstr, sel string, res *[]byte) chromedp.Tasks {
	return chromedp.Tasks{
		chromedp.Navigate(urlstr),
		chromedp.Screenshot(sel, res, chromedp.NodeVisible),
	}
}

// fullScreenshot takes a screenshot of the entire browser viewport.
//
// Note: chromedp.FullScreenshot overrides the device's emulation settings. Use
// device.Reset to reset the emulation and viewport settings.
func fullScreenshot(urlstr string, quality int, res *[]byte) chromedp.Tasks {
	return chromedp.Tasks{
		chromedp.Navigate(urlstr),
		chromedp.FullScreenshot(res, quality),
	}
}
```
执行：`go run main.go`
##### 参考：
更多例子：https://github.com/chromedp/examples