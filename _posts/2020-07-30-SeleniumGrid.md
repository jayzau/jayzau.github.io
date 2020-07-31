---
layout: post
title: Selenium Grid 搭建、使用
date: 2020-07-30
tags: 博客   
---

[Selenium](https://www.selenium.dev/) 是一个用于Web应用程序测试的工具。Selenium测试直接运行在浏览器中，就像真正的用户在操作一样。

理论上说，只要用户能手动做的事情，都可以通过Selenium来自动化实现。当然，用于数据抓取也可以。“所见即所得”是一件很爽的事情。

### 使用

> 系统：Linux
>
> 语言：Python
>
> 浏览器：Chrome

#### 简单使用

下载`selenium`三方包。

```python
pip install selenium
```

查看`Chrome`版本。

```shell
google-chrome --version
```

[官网下载](http://chromedriver.storage.googleapis.com/index.html)对应版本浏览器驱动。

开始使用。

```python
from selenium import webdriver

with webdriver.Chrome(executable_path="驱动绝对路径") as browser:
    browser.get("https://www.baidu.com/")       # 访问百度
    input()       # 防止程序执行完毕浏览器秒关看不到效果
```

#### 使用Docker + Selenium Grid

> If you want to scale by distributing and running tests on several machines and manage multiple environments from a central point, making it easy to run the tests against a vast combination of browsers/OS, then you want to use Selenium Grid.

有几个好处：

- docker-compose 一行命令搭建。方便。
- 终端不再需要配环境，调用接口就可以操控浏览器。
- 管理方便，易于扩展。
- 浏览器类型(火狐、谷歌、欧朋)和版本改改配置想换就换。

![控制台](/images/posts/SeleniumGrid/Grid_console.png)

默认系统已安装启动`Docker`。

安装`docker-compose`

```python
pip install docker-compose
```

配置文件：

```yaml
# To execute this docker-compose yml file use `docker-compose -f <file_name> up`
# Add the `-d` flag at the end for detached execution
# 主机(selenium-hub.yaml)
version: "2"
services:
  selenium-hub:
    image: selenium/hub:3.141.59
    container_name: selenium-hub
    restart: always
    ports:
      - "7654:4444"   # 4444端口开不了
    environment:
      - GRID_MAX_SESSION=50
      - GRID_NEW_SESSION_WAIT_TIMEOUT=60
      - GRID_TIMEOUT=300
      - START_XVFB=false

  chrome:
    image: selenium/node-chrome:3.141.59-xenon
    restart: always
    volumes:
      - /dev/shm:/dev/shm
    depends_on:
      - selenium-hub
    environment:
      - HUB_HOST=selenium-hub
      - NODE_MAX_INSTANCES=3
      - NODE_MAX_SESSION=3
    
  # firefox:
  #   image: selenium/node-firefox:4.0.0-alpha-6-20200721
  #   volumes:
  #     - /dev/shm:/dev/shm
  #   depends_on:
  #     - selenium-hub
  #   environment:
  #     - HUB_HOST=selenium-hub

  # opera:
  #   image: selenium/node-opera:4.0.0-alpha-6-20200721
  #   volumes:
  #     - /dev/shm:/dev/shm
  #   depends_on:
  #     - selenium-hub
  #   environment:
  #     - HUB_HOST=selenium-hub
```

指定文件启动：

```shell
docker-compose -f selenium-hub.yaml up -d
```

多开3个chrome节点：

```shell
docker-compose -f selenium-hub.yaml up --scale chrome=3 -d
```

扩容配置(另一台服务器)：

```yaml
# To execute this docker-compose yml file use `docker-compose -f <file_name> up`
# Add the `-d` flag at the end for detached execution
# 从机(selenium-node.yaml)
version: "2"
services:
  chrome-node1:    # 需要几个写几个
    image: selenium/node-chrome:3.141.59-xenon
    restart: always
    stdin_open: true
    volumes:
      - /dev/shm:/dev/shm
    ports:
      - "7200:5555"
    environment:
      - HUB_HOST=127.0.0.1    # 主机IP
      - HUB_PORT=7654         # 主机端口
      - NODE_MAX_INSTANCES=5
      - NODE_MAX_SESSION=5
      - REMOTE_HOST=http://127.0.0.1:7200    # 从机IP  端口同 ports
      - GRID_TIMEOUT=300

  chrome-node2:
    image: selenium/node-chrome:3.141.59-xenon
    restart: always
    stdin_open: true
    volumes:
      - /dev/shm:/dev/shm
    ports:
      - "7201:5555"
    environment:
      - HUB_HOST=127.0.0.1
      - HUB_PORT=7654
      - NODE_MAX_INSTANCES=5
      - NODE_MAX_SESSION=5
      - REMOTE_HOST=http://127.0.0.1:7201
      - GRID_TIMEOUT=300

  # firefox:
  #   image: selenium/node-firefox:4.0.0-alpha-6-20200721
  #   volumes:
  #     - /dev/shm:/dev/shm
  #   depends_on:
  #     - selenium-hub
  #   environment:
  #     - HUB_HOST=selenium-hub

  # opera:
  #   image: selenium/node-opera:4.0.0-alpha-6-20200721
  #   volumes:
  #     - /dev/shm:/dev/shm
  #   depends_on:
  #     - selenium-hub
  #   environment:
  #     - HUB_HOST=selenium-hub
```

指定文件启动：

```shell
docker-compose -f selenium-node.yaml up -d
```

关闭：

```shell
docker-compose -f selenium-hub.yaml down
docker-compose -f selenium-node.yaml down
```

使用：

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options    # Chrome
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities


chrome_options = Options()
desired_capabilities = DesiredCapabilities.CHROME
command_executor = 'http://{IP}:{PORT}/wd/hub'.format(IP="", PORT="")
with webdriver.Remote(
    command_executor=command_executor,
    options=chrome_options,
    desired_capabilities=desired_capabilities,
) as browser:
    browser.get("https://www.baidu.com/")       # 访问百度
    print(browser.page_source)
```

由于浏览器运行在服务器上，所以界面是看不见的。

镜像`selenium/node-chrome-debug`可以使用VNC连接查看界面。

或者利用`get_screenshot_as_png()`方法来查看截图。

Github地址：[Selenium Docker](https://github.com/SeleniumHQ/docker-selenium)

#### 一些谷歌浏览器参数

```python
from selenium.webdriver.chrome.options import Options
chrome_options = Options()
chrome_options.add_argument('--proxy-server=http://ip:port')    # 代理
chrome_options.add_argument('--no-sandbox')     # 最高权限启动（服务器）
chrome_options.add_argument('--start-maximized')     # 最大化打开
chrome_options.add_argument('--disable-gpu')      # 一些博文说防止出BUG
chrome_options.add_argument('user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) '
                            'Chrome/73.0.3683.86 Safari/537.36')    # 请求头
chrome_options.add_argument('--headless')       # 无头浏览器（不显示界面）
# 这俩不是无头设了也没用
chrome_options.add_argument('window-size=1920x1080')
chrome_options.add_argument('lang=zh_CN.UTF-8')
# 这俩是无头设了也没用（开发者选项）
chrome_options.add_experimental_option('excludeSwitches', ['--enable-automation'])
chrome_options.add_experimental_option('useAutomationExtension', False)

from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
# 用于查看 Network 记录
desired_capabilities = DesiredCapabilities.CHROME
desired_capabilities["loggingPrefs"] = {'performance': 'ALL'}
chrome_options.add_experimental_option('w3c', False)
# browser.get_log("performance")
```

---

挺好的。
