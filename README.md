# 网络解锁器 API

[![宣传图](https://github.com/bright-cn/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/) 

[网络解锁器](https://www.bright.cn/products/web-unlocker) 是一个强大的网页抓取 API，可绕过复杂的反爬机制并访问任意网站。只需一次 API 调用，无需管理复杂的反机器人基础设施，即可获取干净的 HTML 或 JSON 响应。

## 目录
- [功能](#功能)
- [开始使用](#开始使用)
  - [直接使用 API](#直接使用-api)
  - [使用原生代理访问](#使用原生代理访问)
- [实用示例：抓取 G2 评论](#实用示例抓取-g2-评论)
  - [基础请求（不使用 Web Unlocker）](#基础请求不使用-web-unlocker)
  - [增强请求（使用 Web Unlocker）](#增强请求使用-web-unlocker)
    - [直接使用 API](#直接使用-api-1)
    - [代理模式访问](#代理模式访问)
    - [等待特定元素](#等待特定元素)
    - [移动端 User-Agent 指定](#移动端-user-agent-指定)
    - [地理定位指定](#地理定位指定)
    - [调试请求](#调试请求)
    - [成功率统计](#成功率统计)
- [最后说明](#最后说明)

## 功能
Web Unlocker 提供了全面的网页抓取能力：
- 自动代理管理与 CAPTCHA 识别
- 模拟真实用户行为
- 内置 JavaScript 渲染
- 全球地理位置定向
- 自动重试机制
- 按成功次数付费的定价模式

## 开始使用
在使用 Web Unlocker 前，请先根据此[快速开始指南](https://docs.brightdata.com/scraping-automation/web-unlocker/quickstart)完成对应配置。

### 直接使用 API
这是集成 Web Unlocker 的推荐方式。

**示例：cURL 命令**  
```bash
curl -X POST "https://api.brightdata.com/request" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer INSERT_YOUR_API_TOKEN" \
-d '{
  "zone": "INSERT_YOUR_WEB_UNLOCKER_ZONE_NAME",
  "url": "http://lumtest.com/myip.json",
  "format": "raw"
}'
```

1. API 端点（Endpoint）：`https://api.brightdata.com/request`  
2. Authorization 请求头：使用从 Web Unlocker API 区域生成的 [API 令牌](https://docs.brightdata.com/scraping-automation/web-unlocker/send-your-first-request#generating-your-bright-data-api-token)  
3. 主要请求体参数：  
   - `zone`: 你的 Web Unlocker 区域名称  
   - `url`: 目标访问地址  
   - `format`: 响应格式（使用 `raw` 以获取网站源响应）

**示例：Python 脚本**  
```python
import requests

API_URL = "https://api.brightdata.com/request"
API_TOKEN = "INSERT_YOUR_API_TOKEN"
ZONE_NAME = "INSERT_YOUR_WEB_UNLOCKER_ZONE_NAME"
TARGET_URL = "http://lumtest.com/myip.json"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_TOKEN}"
}

payload = {
    "zone": ZONE_NAME,
    "url": TARGET_URL,
    "format": "raw"
}

response = requests.post(API_URL, headers=headers, json=payload)

if response.status_code == 200:
    print("Success:", response.text)
else:
    print(f"Error {response.status_code}: {response.text}")
```

### 使用原生代理访问
这是利用代理路由的另一种可选方式。

**示例：cURL 命令**  
```bash
curl "http://lumtest.com/myip.json" \
--proxy "brd.superproxy.io:33335" \
--proxy-user "brd-customer-<CUSTOMER_ID>-zone-<ZONE_NAME>:<ZONE_PASSWORD>"
```

必填凭据：
1. 客户 ID（Customer ID）：可在 [帐户设置](https://www.bright.cn/cp/setting/customer_details) 中查看  
2. Web Unlocker 区域名称：可在概览（Overview）页面找到  
3. Web Unlocker 密码：也可在概览页面找到

**示例：Python 脚本**  
```python
import requests

customer_id = "<customer_id>"
zone_name = "<zone_name>"
zone_password = "<zone_password>"

host = "brd.superproxy.io"
port = 33335
proxy_url = f"http://brd-customer-{customer_id}-zone-{zone_name}:{zone_password}@{host}:{port}"

proxies = {"http": proxy_url, "https": proxy_url}

response = requests.get("http://lumtest.com/myip.json", proxies=proxies)

if response.status_code == 200:
    print(response.json())
else:
    print(f"Error: {response.status_code}")
```

## 实用示例：抓取 G2 评论
以下示例展示如何抓取 [G2.com](https://www.g2.com/) 上的评论。G2 使用了 Cloudflare 等严格的反爬机制。

### 基础请求（不使用 Web Unlocker）
先使用一个简单的 Python 脚本，尝试抓取 [MongoDB 在 G2 上的评论](https://www.g2.com/products/mongodb/reviews)：
```python
import requests
from bs4 import BeautifulSoup

url = 'https://www.g2.com/products/mongodb/reviews'
response = requests.get(url)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = soup.find_all('h2')
    
    if headings:
        print("\nHeadings Found:")
        for heading in headings:
            print(f"- {heading.get_text(strip=True)}")
    else:
        print("No headings found")
else:
    print("Request blocked")
```

**结果：** 由于 Cloudflare 的反机器人措施阻拦，此脚本会失败并返回 `403` 错误。

### 增强请求（使用 Web Unlocker）
要绕过此类限制，可使用 Web Unlocker。以下是一个 Python 实现示例：

#### 直接使用 API
```python
import requests
from bs4 import BeautifulSoup

API_URL = "https://api.brightdata.com/request"
API_TOKEN = "INSERT_YOUR_API_TOKEN"
ZONE_NAME = "INSERT_YOUR_ZONE"
TARGET_URL = "https://www.g2.com/products/mongodb/reviews"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {API_TOKEN}"
}
payload = {"zone": ZONE_NAME, "url": TARGET_URL, "format": "raw"}

response = requests.post(API_URL, headers=headers, json=payload)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = [h.get_text(strip=True) for h in soup.find_all('h2')]
    print("\nExtracted Headings:", headings)
else:
    print(f"Error {response.status_code}: {response.text}")
```
**结果：** 成功绕过防护并返回 `200`，能获取所需内容。

#### 代理模式访问
也可以使用代理模式：
```python
import requests
from bs4 import BeautifulSoup

proxy_url = "http://brd-customer-<customer_id>-zone-<zone_name>:<zone_password>@brd.superproxy.io:33335"
proxies = {"http": proxy_url, "https": proxy_url}

url = "https://www.g2.com/products/mongodb/reviews"
response = requests.get(url, proxies=proxies, verify=False)

if response.status_code == 200:
    soup = BeautifulSoup(response.text, "lxml")
    headings = [h.get_text(strip=True) for h in soup.find_all('h2')]
    print("\nExtracted Headings:", headings)
else:
    print(f"Error {response.status_code}: {response.text}")
```

**注意：** 如果遇到 SSL 证书警告，可通过以下方式禁用：
```python
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
```

#### 等待特定元素
可以通过 `x-unblock-expect` 请求头来等待特定元素或文本出现：
```python
headers["x-unblock-expect"] = '{"element": ".star-wrapper__desc"}'
# 或
headers["x-unblock-expect"] = '{"text": "reviews"}'
```

👉 完整代码可参考 [g2_wait.py](https://github.com/bright-cn/web-unlocker-api/blob/main/src/g2_wait.py)

#### 移动端 User-Agent 指定
如果想使用移动端 User-Agent，而非桌面端，可以在用户名中加上 `-ua-mobile`：
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-ua-mobile"
```
👉 完整代码可参考 [g2_mobile.py](https://github.com/bright-cn/web-unlocker-api/blob/main/src/g2_mobile.py)

#### 地理定位指定
Web Unlocker 会自动选择最佳 IP 位置，但也可以自行指定目标位置：
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-country-us"
username = f"brd-customer-{customer_id}-zone-{zone_name}-country-us-city-sanfrancisco"
```

👉 详情可查看 [地理定位高级用法](https://docs.brightdata.com/api-reference/proxy/geolocation-targeting)。

#### 调试请求
可加上 `-debug-full` 来获取详细的调试信息：
```python
username = f"brd-customer-{customer_id}-zone-{zone_name}-debug-full"
```
👉 完整代码可参考 [g2_debug.py](https://github.com/bright-cn/web-unlocker-api/blob/main/src/g2_debug.py)

#### 成功率统计
可监控特定域名在 API 中的成功率：
```python
import requests

API_TOKEN = "INSERT_YOUR_API_TOKEN"

def get_success_rate(domain):
    url = f"https://api.brightdata.com/unblocker/success_rate/{domain}"
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {API_TOKEN}"
    }
    response = requests.get(url, headers=headers)
    print(response.json() if response.status_code == 200 else response.text)

get_success_rate("g2.com") # 获取特定域名统计
get_success_rate("g2.*")   # 获取该域名所有顶级后缀统计
```

## 最后说明
通过 Web Unlocker，你可以轻松抓取最具防护性的网站。以下几点需要注意：

1. **不兼容以下环境：**  
   - 浏览器（Chrome、Firefox、Edge 等）  
   - 反指纹浏览器（Adspower、Multilogin 等）  
   - 自动化工具（Puppeteer、Playwright、Selenium 等）  

2. **使用 Scraping Browser：**  
   如果需要在浏览器环境执行自动化，可使用 Bright Data 的 [抓取浏览器](https://www.bright.cn/products/scraping-browser)。

3. **高级域名（Premium Domains）：**  
   对于难度较高的网站，可使用 [Premium Domain](https://docs.brightdata.com/scraping-automation/web-unlocker/features#web-unlocker-api-premium-domains) 功能。

4. **CAPTCHA 解决：**  
   默认自动识别，也可选择[关闭此功能](https://docs.brightdata.com/scraping-automation/web-unlocker/features#disable-captcha-solving)。更多信息请参考 Bright Data 的 [CAPTCHA Solver](https://www.bright.cn/products/web-unlocker/captcha-solver)。

5. **自定义请求头与 Cookie：**  
   可以自行添加，以便请求特定站点版本。更多内容请见[此文档](https://docs.brightdata.com/scraping-automation/web-unlocker/features#manual-headers-and-cookies)。

更多详情可参阅 [官方文档](https://docs.brightdata.com/scraping-automation/web-unlocker/introduction)。
