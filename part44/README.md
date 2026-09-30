# Part 44: Web Scraping - BeautifulSoup & Selenium

## บทนำ

Web Scraping คือกระบวนการดึงข้อมูลจากเว็บไซต์โดยอัตโนมัติ Python เป็นภาษาที่นิยมใช้สำหรับ web scraping เพราะมี libraries ที่ทรงพลังและใช้งานง่าย

**ก่อนเริ่ม: Ethics และ Legal Considerations**
- ตรวจสอบ `robots.txt` ก่อนเสมอ
- อ่าน Terms of Service ของเว็บไซต์
- ไม่ส่ง requests มากเกินไป (rate limiting)
- ใช้ข้อมูลที่ scrape อย่างรับผิดชอบ

---

## 1. Web Scraping Concepts และ Ethics

### HTTP Request/Response Flow

```
Client (Python)          Web Server
    │                        │
    │── GET /page ──────────►│
    │                        │
    │◄── 200 OK + HTML ──────│
    │                        │
```

```python
# ตัวอย่าง basic HTTP concepts
"""
HTTP Methods:
- GET: ดึงข้อมูล
- POST: ส่งข้อมูล (forms, APIs)
- PUT/PATCH: อัพเดทข้อมูล
- DELETE: ลบข้อมูล

HTTP Status Codes:
- 200 OK: สำเร็จ
- 301/302: Redirect
- 403 Forbidden: ไม่มีสิทธิ์
- 404 Not Found: ไม่พบ
- 429 Too Many Requests: ส่งมากเกินไป
- 500 Internal Server Error: Server error
"""

# robots.txt ตัวอย่าง
ROBOTS_TXT_EXAMPLE = """
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/
Crawl-delay: 10

User-agent: Googlebot
Allow: /
"""

# การตรวจสอบ robots.txt ด้วย urllib.robotparser
from urllib.robotparser import RobotFileParser

def check_robots_txt(base_url: str, user_agent: str, target_url: str) -> bool:
    """ตรวจสอบว่า robots.txt อนุญาตให้ scrape URL นั้นหรือไม่"""
    rp = RobotFileParser()
    rp.set_url(f"{base_url}/robots.txt")
    try:
        rp.read()
        return rp.can_fetch(user_agent, target_url)
    except Exception:
        return True  # ถ้าอ่าน robots.txt ไม่ได้ ถือว่า allowed

# ตัวอย่างการใช้
can_scrape = check_robots_txt(
    "https://www.example.com",
    "MyBot/1.0",
    "https://www.example.com/products"
)
print(f"Can scrape: {can_scrape}")
```

---

## 2. HTTP Requests ด้วย requests Library

### ติดตั้ง

```bash
pip install requests beautifulsoup4 lxml
```

### Basic Requests

```python
import requests
from typing import Optional, Dict, Any

def fetch_page(url: str, timeout: int = 30) -> Optional[str]:
    """ดึง HTML content จาก URL"""
    
    headers = {
        'User-Agent': 'Mozilla/5.0 (compatible; MyBot/1.0)',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Accept-Encoding': 'gzip, deflate',
        'Connection': 'keep-alive',
    }
    
    try:
        response = requests.get(url, headers=headers, timeout=timeout)
        response.raise_for_status()  # raise exception สำหรับ 4xx/5xx
        return response.text
    except requests.exceptions.HTTPError as e:
        print(f"HTTP Error: {e}")
    except requests.exceptions.ConnectionError as e:
        print(f"Connection Error: {e}")
    except requests.exceptions.Timeout:
        print(f"Request timed out after {timeout}s")
    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")
    
    return None

# ดึงหน้าเว็บ
html = fetch_page("https://books.toscrape.com")
if html:
    print(f"Retrieved {len(html)} bytes")
    print(html[:500])
```

```python
import requests

# ดู response details
response = requests.get("https://httpbin.org/get")

print(f"Status Code: {response.status_code}")
print(f"Content-Type: {response.headers.get('Content-Type')}")
print(f"Encoding: {response.encoding}")
print(f"URL: {response.url}")

# Response เป็น JSON
data = response.json()
print(f"My IP: {data.get('origin')}")
print(f"Request headers: {data.get('headers')}")
```

---

## 3. HTML Parsing ด้วย BeautifulSoup4

### Basic Parsing

```python
from bs4 import BeautifulSoup

# HTML ตัวอย่าง
html = """
<!DOCTYPE html>
<html>
<head><title>Sample Page</title></head>
<body>
    <h1 id="main-title" class="title">Hello World</h1>
    <div class="content">
        <p class="intro">This is a paragraph.</p>
        <ul id="menu">
            <li class="item active"><a href="/home">Home</a></li>
            <li class="item"><a href="/about">About</a></li>
            <li class="item"><a href="/contact">Contact</a></li>
        </ul>
    </div>
    <div class="content secondary">
        <p>Another paragraph with <strong>bold text</strong></p>
        <img src="/images/photo.jpg" alt="Photo" class="image"/>
    </div>
    <footer>
        <p>Copyright 2024</p>
    </footer>
</body>
</html>
"""

# สร้าง BeautifulSoup object
soup = BeautifulSoup(html, 'html.parser')
# หรือใช้ lxml parser (เร็วกว่า): BeautifulSoup(html, 'lxml')

# ดึง title
print(soup.title.text)          # Sample Page
print(soup.title.string)        # Sample Page

# ดึง h1
h1 = soup.h1
print(h1.text)                  # Hello World
print(h1['id'])                 # main-title
print(h1['class'])              # ['title']
print(h1.get('id'))             # main-title (ปลอดภัยกว่า)
```

### find() และ find_all()

```python
from bs4 import BeautifulSoup

# ใช้ html จากด้านบน
soup = BeautifulSoup(html, 'html.parser')

# find() - หาตัวแรก
first_p = soup.find('p')
print(first_p.text)  # This is a paragraph.

# find กับ class
intro_p = soup.find('p', class_='intro')
print(intro_p.text)  # This is a paragraph.

# find กับ id
menu = soup.find('ul', id='menu')
print(menu)

# find_all() - หาทั้งหมด
all_p = soup.find_all('p')
print(f"Found {len(all_p)} paragraphs")
for p in all_p:
    print(f"  - {p.text.strip()}")

# find_all กับ หลาย tags
all_headings = soup.find_all(['h1', 'h2', 'h3'])

# find_all กับ class
content_divs = soup.find_all('div', class_='content')
print(f"Found {len(content_divs)} content divs")

# find_all กับ string (regex)
import re
paragraphs_with_another = soup.find_all('p', string=re.compile('Another'))
```

### การ Navigate Tree

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html, 'html.parser')

# Parent/Children navigation
body = soup.body
print(f"Body children count: {len(list(body.children))}")

# Parent
h1 = soup.find('h1')
print(f"H1 parent: {h1.parent.name}")  # body

# Siblings
first_li = soup.find('li')
print(f"First li: {first_li.text.strip()}")
print(f"Next sibling: {first_li.next_sibling.next_sibling.text.strip()}")

# เดิน traverse tree
for element in soup.body.descendants:
    if hasattr(element, 'name') and element.name:
        print(f"Tag: {element.name}")
```

---

## 4. CSS Selectors

BeautifulSoup รองรับ CSS selectors ผ่าน `select()` และ `select_one()`

```python
from bs4 import BeautifulSoup

html = """
<div class="container">
    <article class="post featured" id="post-1">
        <h2 class="title"><a href="/post/1">First Post</a></h2>
        <div class="meta">
            <span class="author">Alice</span>
            <span class="date">2024-01-01</span>
        </div>
        <p class="excerpt">This is the first post...</p>
        <a href="/post/1" class="read-more">Read More</a>
    </article>
    <article class="post" id="post-2">
        <h2 class="title"><a href="/post/2">Second Post</a></h2>
        <div class="meta">
            <span class="author">Bob</span>
            <span class="date">2024-01-02</span>
        </div>
        <p class="excerpt">This is the second post...</p>
        <a href="/post/2" class="read-more">Read More</a>
    </article>
</div>
"""

soup = BeautifulSoup(html, 'html.parser')

# CSS Selectors

# Element selector
titles = soup.select('h2')
print(f"H2 count: {len(titles)}")

# Class selector
posts = soup.select('.post')
print(f"Posts: {len(posts)}")

# ID selector
post1 = soup.select_one('#post-1')
print(f"Post 1: {post1.find('h2').text}")

# Descendant selector (space)
authors = soup.select('.post .author')
for author in authors:
    print(f"Author: {author.text}")

# Child selector (>)
direct_titles = soup.select('.post > .title')

# Multiple classes
featured = soup.select('.post.featured')
print(f"Featured posts: {len(featured)}")

# Attribute selector
links = soup.select('a[href^="/post/"]')  # href ขึ้นต้นด้วย /post/
for link in links:
    print(f"Link: {link['href']} - {link.text}")

# nth-child
first_post = soup.select('.post:first-child')
```

---

## 5. XPath Basics

XPath ใช้ได้กับ lxml library

```python
from lxml import etree, html as lhtml
import requests

# Parse HTML ด้วย lxml
html_content = """
<html>
<body>
    <div class="products">
        <div class="product" data-id="1">
            <span class="name">Laptop</span>
            <span class="price">999.99</span>
            <a href="/product/1">Details</a>
        </div>
        <div class="product" data-id="2">
            <span class="name">Phone</span>
            <span class="price">599.99</span>
            <a href="/product/2">Details</a>
        </div>
    </div>
</body>
</html>
"""

tree = lhtml.fromstring(html_content)

# XPath Expressions

# // = search anywhere in document
# / = direct child
# @ = attribute
# text() = text content

# หาทุก div.product
products = tree.xpath('//div[@class="product"]')
print(f"Products found: {len(products)}")

for product in products:
    # ดึง data-id
    product_id = product.xpath('@data-id')[0]
    # ดึงชื่อ
    name = product.xpath('.//span[@class="name"]/text()')[0]
    # ดึงราคา
    price = product.xpath('.//span[@class="price"]/text()')[0]
    # ดึง link
    link = product.xpath('.//a/@href')[0]
    
    print(f"ID: {product_id}, Name: {name}, Price: {price}, Link: {link}")

# XPath ที่ซับซ้อนขึ้น
# หา product ที่มี price > 700
expensive = tree.xpath('//div[@class="product"][.//span[@class="price"][number(text()) > 700]]')
print(f"\nExpensive products: {len(expensive)}")
for p in expensive:
    print(f"  {p.xpath('.//span[@class=\"name\"]/text()')[0]}")
```

---

## 6. Handling Pagination

```python
import requests
from bs4 import BeautifulSoup
import time
from typing import Iterator, List, Dict

def scrape_paginated_site(
    base_url: str,
    max_pages: int = 10,
    delay: float = 1.0
) -> Iterator[Dict]:
    """Scrape เว็บที่มี pagination"""
    
    page = 1
    
    while page <= max_pages:
        url = f"{base_url}?page={page}"
        
        print(f"Scraping page {page}: {url}")
        
        try:
            response = requests.get(url, timeout=30)
            response.raise_for_status()
        except requests.exceptions.RequestException as e:
            print(f"Error on page {page}: {e}")
            break
        
        soup = BeautifulSoup(response.text, 'html.parser')
        
        # ดึงข้อมูลจากหน้านี้
        items = soup.select('.item-container .item')
        
        if not items:
            print(f"No items found on page {page}, stopping")
            break
        
        for item in items:
            yield {
                'title': item.select_one('.title').text.strip() if item.select_one('.title') else '',
                'url': item.select_one('a')['href'] if item.select_one('a') else '',
                'page': page
            }
        
        # ตรวจสอบว่ามีหน้าถัดไปหรือไม่
        next_btn = soup.select_one('.pagination .next:not(.disabled)')
        if not next_btn:
            print("No more pages")
            break
        
        page += 1
        time.sleep(delay)  # Polite delay

# ตัวอย่างกับ books.toscrape.com
def scrape_books_toscrape(max_pages: int = 3) -> List[Dict]:
    """Scrape books.toscrape.com - เว็บสำหรับ practice"""
    books = []
    base_url = "https://books.toscrape.com/catalogue"
    url = f"{base_url}/page-1.html"
    page = 1
    
    while url and page <= max_pages:
        print(f"Scraping page {page}...")
        
        response = requests.get(url, timeout=30)
        soup = BeautifulSoup(response.text, 'html.parser')
        
        for article in soup.select('article.product_pod'):
            book = {
                'title': article.select_one('h3 a')['title'],
                'price': article.select_one('.price_color').text.strip(),
                'rating': article.select_one('.star-rating')['class'][1],
                'url': base_url + '/' + article.select_one('h3 a')['href'],
                'in_stock': 'In stock' in article.select_one('.availability').text
            }
            books.append(book)
        
        # หา next page
        next_link = soup.select_one('.pager .next a')
        if next_link and page < max_pages:
            url = f"{base_url}/{next_link['href']}"
            page += 1
        else:
            url = None
        
        time.sleep(0.5)  # Respectful delay
    
    return books

# ใช้งาน
# books = scrape_books_toscrape(max_pages=2)
# print(f"Scraped {len(books)} books")
# for book in books[:5]:
#     print(f"  {book['title']}: {book['price']}")
```

---

## 7. Scrapy Framework เบื้องต้น

Scrapy เป็น web scraping framework ที่ทรงพลังสำหรับ large-scale scraping

### ติดตั้ง

```bash
pip install scrapy
scrapy startproject myproject
```

### Spider พื้นฐาน

```python
# myproject/spiders/books_spider.py
import scrapy
from scrapy import Request
from typing import Generator, Dict, Any

class BooksSpider(scrapy.Spider):
    """Spider สำหรับ scrape books.toscrape.com"""
    
    name = "books"
    allowed_domains = ["books.toscrape.com"]
    start_urls = ["https://books.toscrape.com/"]
    
    custom_settings = {
        'DOWNLOAD_DELAY': 1,  # รอ 1 วินาทีระหว่าง requests
        'CONCURRENT_REQUESTS': 1,
        'ROBOTSTXT_OBEY': True,
    }
    
    def parse(self, response) -> Generator:
        """Parse หน้าหลัก"""
        
        # ดึงข้อมูล books แต่ละเล่ม
        for article in response.css('article.product_pod'):
            book_url = article.css('h3 a::attr(href)').get()
            
            # Follow link ไปยังหน้า detail
            yield Request(
                response.urljoin(book_url),
                callback=self.parse_book,
                meta={'title': article.css('h3 a::attr(title)').get()}
            )
        
        # Follow pagination
        next_page = response.css('li.next a::attr(href)').get()
        if next_page:
            yield response.follow(next_page, callback=self.parse)
    
    def parse_book(self, response) -> Dict[str, Any]:
        """Parse หน้า detail ของ book"""
        
        yield {
            'title': response.meta['title'],
            'price': response.css('.price_color::text').get().strip(),
            'rating': response.css('.star-rating::attr(class)').get().split()[-1],
            'description': response.css('#product_description ~ p::text').get('').strip(),
            'upc': response.css('table tr:first-child td::text').get(),
            'availability': response.css('.availability::text').getall()[-1].strip() if response.css('.availability::text').getall() else '',
            'url': response.url,
        }

# รัน: scrapy crawl books -o books.json
```

### Scrapy Items

```python
# myproject/items.py
import scrapy

class BookItem(scrapy.Item):
    title = scrapy.Field()
    price = scrapy.Field()
    rating = scrapy.Field()
    description = scrapy.Field()
    upc = scrapy.Field()
    availability = scrapy.Field()
    url = scrapy.Field()

# myproject/pipelines.py
import sqlite3
from datetime import datetime

class DatabasePipeline:
    """Pipeline สำหรับบันทึกข้อมูลลง SQLite"""
    
    def open_spider(self, spider):
        self.conn = sqlite3.connect('books.db')
        self.cursor = self.conn.cursor()
        self.cursor.execute('''
            CREATE TABLE IF NOT EXISTS books (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT,
                price TEXT,
                rating TEXT,
                url TEXT,
                scraped_at TEXT
            )
        ''')
        self.conn.commit()
    
    def close_spider(self, spider):
        self.conn.close()
    
    def process_item(self, item, spider):
        self.cursor.execute('''
            INSERT INTO books (title, price, rating, url, scraped_at)
            VALUES (?, ?, ?, ?, ?)
        ''', (
            item.get('title'),
            item.get('price'),
            item.get('rating'),
            item.get('url'),
            datetime.now().isoformat()
        ))
        self.conn.commit()
        return item
```

---

## 8. Selenium WebDriver

Selenium ใช้สำหรับ scrape เว็บที่ใช้ JavaScript rendering

### ติดตั้ง

```bash
pip install selenium webdriver-manager
```

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager
import time

def create_driver(headless: bool = True) -> webdriver.Chrome:
    """สร้าง Chrome WebDriver"""
    
    options = Options()
    
    if headless:
        options.add_argument('--headless')
    
    options.add_argument('--no-sandbox')
    options.add_argument('--disable-dev-shm-usage')
    options.add_argument('--window-size=1920,1080')
    options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36')
    
    # ปิด images เพื่อเพิ่มความเร็ว
    prefs = {"profile.managed_default_content_settings.images": 2}
    options.add_experimental_option("prefs", prefs)
    
    service = Service(ChromeDriverManager().install())
    driver = webdriver.Chrome(service=service, options=options)
    
    return driver

def scrape_with_selenium(url: str) -> str:
    """Scrape หน้าเว็บที่ใช้ JavaScript"""
    
    driver = create_driver(headless=True)
    
    try:
        driver.get(url)
        
        # รอ element โหลด
        wait = WebDriverWait(driver, 10)
        wait.until(EC.presence_of_element_located((By.TAG_NAME, 'body')))
        
        # รอ JavaScript execute
        time.sleep(2)
        
        return driver.page_source
    
    finally:
        driver.quit()
```

### Selenium กับ Dynamic Content

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys
import time

def scrape_infinite_scroll(url: str, scroll_times: int = 5) -> list:
    """Scrape เว็บที่ใช้ infinite scroll"""
    
    driver = create_driver()
    items = []
    
    try:
        driver.get(url)
        time.sleep(2)
        
        for i in range(scroll_times):
            print(f"Scroll {i+1}/{scroll_times}")
            
            # ดึงข้อมูลก่อน scroll
            elements = driver.find_elements(By.CSS_SELECTOR, '.item')
            for el in elements:
                text = el.text.strip()
                if text and text not in [item['text'] for item in items]:
                    items.append({'text': text})
            
            # Scroll ลงไปด้านล่าง
            driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
            time.sleep(2)  # รอ load
        
        return items
    
    finally:
        driver.quit()

def fill_and_submit_form(url: str, form_data: dict) -> None:
    """กรอกและ submit form"""
    
    driver = create_driver(headless=False)
    
    try:
        driver.get(url)
        
        wait = WebDriverWait(driver, 10)
        
        # กรอก username
        username_field = wait.until(
            EC.element_to_be_clickable((By.NAME, 'username'))
        )
        username_field.clear()
        username_field.send_keys(form_data.get('username', ''))
        
        # กรอก password
        password_field = driver.find_element(By.NAME, 'password')
        password_field.send_keys(form_data.get('password', ''))
        
        # Click submit button
        submit_btn = driver.find_element(By.CSS_SELECTOR, 'button[type="submit"]')
        submit_btn.click()
        
        # รอ redirect หลัง login
        wait.until(EC.url_changes(url))
        print(f"Current URL after login: {driver.current_url}")
    
    finally:
        driver.quit()

def click_and_wait(driver, locator_type, locator_value: str, wait_time: int = 10):
    """Click element และรอจนเสร็จ"""
    
    wait = WebDriverWait(driver, wait_time)
    
    element = wait.until(
        EC.element_to_be_clickable((locator_type, locator_value))
    )
    element.click()
    
    # รอ page load
    wait.until(EC.staleness_of(element))

# เลื่อน scroll ไปยัง element แล้ว click
def scroll_to_and_click(driver, element):
    """Scroll ไปยัง element แล้ว click"""
    
    # Scroll element ให้เข้า viewport
    driver.execute_script("arguments[0].scrollIntoView(true);", element)
    time.sleep(0.5)
    
    # Click ด้วย ActionChains
    actions = ActionChains(driver)
    actions.move_to_element(element).click().perform()
```

---

## 9. Playwright for Python

Playwright เป็นทางเลือกที่ทันสมัยกว่า Selenium

### ติดตั้ง

```bash
pip install playwright
playwright install
```

```python
from playwright.sync_api import sync_playwright, Page, Browser
import time

def scrape_with_playwright(url: str) -> str:
    """Scrape ด้วย Playwright"""
    
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        
        # Set user agent
        page.set_extra_http_headers({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        })
        
        page.goto(url)
        
        # รอ element
        page.wait_for_selector('body')
        
        # ดึง HTML
        content = page.content()
        
        browser.close()
        return content

def scrape_dynamic_content(url: str) -> list:
    """Scrape เว็บที่ใช้ JavaScript rendering"""
    
    results = []
    
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        
        # Intercept network requests
        def handle_response(response):
            if '/api/data' in response.url and response.status == 200:
                try:
                    data = response.json()
                    results.extend(data.get('items', []))
                except:
                    pass
        
        page.on('response', handle_response)
        
        page.goto(url)
        page.wait_for_load_state('networkidle')
        
        # Scroll เพื่อ trigger lazy loading
        for _ in range(3):
            page.evaluate("window.scrollTo(0, document.body.scrollHeight)")
            page.wait_for_timeout(1000)
        
        browser.close()
    
    return results

# Async Playwright
from playwright.async_api import async_playwright
import asyncio

async def scrape_multiple_pages(urls: list) -> list:
    """Scrape หลายหน้าพร้อมกัน"""
    
    results = []
    
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)
        
        # สร้าง multiple pages
        pages = []
        for _ in range(min(len(urls), 5)):  # max 5 concurrent
            page = await browser.new_page()
            pages.append(page)
        
        # Distribute URLs across pages
        async def scrape_url(page: Page, url: str):
            await page.goto(url)
            await page.wait_for_selector('body')
            
            # ดึง title
            title = await page.title()
            return {'url': url, 'title': title}
        
        # Run concurrently
        tasks = [
            scrape_url(pages[i % len(pages)], url)
            for i, url in enumerate(urls)
        ]
        
        page_results = await asyncio.gather(*tasks)
        results.extend(page_results)
        
        await browser.close()
    
    return results

# ใช้งาน async
# results = asyncio.run(scrape_multiple_pages([
#     "https://example.com",
#     "https://python.org"
# ]))
```

---

## 10. Handling JavaScript-rendered Pages

```python
import requests
from bs4 import BeautifulSoup
import json

def scrape_with_api(base_url: str, api_endpoint: str) -> list:
    """บางเว็บ load ข้อมูลผ่าน API - ดึงจาก API โดยตรงแทน"""
    
    headers = {
        'User-Agent': 'Mozilla/5.0',
        'Accept': 'application/json',
        'X-Requested-With': 'XMLHttpRequest',
        'Referer': base_url,
    }
    
    items = []
    page = 1
    
    while True:
        params = {
            'page': page,
            'per_page': 20,
            'format': 'json'
        }
        
        response = requests.get(api_endpoint, headers=headers, params=params)
        
        if response.status_code != 200:
            break
        
        data = response.json()
        
        if not data.get('items'):
            break
        
        items.extend(data['items'])
        
        if not data.get('has_more'):
            break
        
        page += 1
        import time
        time.sleep(0.5)
    
    return items

# ตรวจสอบ network requests ใน DevTools เพื่อหา API endpoints
def find_api_calls_pattern():
    """
    วิธีหา API endpoints:
    1. เปิด Chrome DevTools (F12)
    2. ไปที่ Network tab
    3. Filter by XHR/Fetch
    4. โหลดหน้าเว็บ
    5. ดู requests ที่มี data (JSON responses)
    
    Common patterns:
    - /api/v1/products
    - /graphql
    - /_next/data/.../products.json (Next.js)
    - /wp-json/wp/v2/posts (WordPress)
    """
    pass

# ดึงข้อมูลจาก JavaScript variables ใน HTML
def extract_js_data(html: str, variable_name: str) -> dict:
    """ดึงข้อมูลที่ embed อยู่ใน JavaScript"""
    
    soup = BeautifulSoup(html, 'html.parser')
    
    for script in soup.find_all('script'):
        if script.string and variable_name in str(script.string):
            # หา JSON data จาก JavaScript
            import re
            # Pattern: var DATA = {...}
            pattern = rf'{variable_name}\s*=\s*({{.*?}});'
            match = re.search(pattern, str(script.string), re.DOTALL)
            
            if match:
                try:
                    return json.loads(match.group(1))
                except json.JSONDecodeError:
                    pass
    
    return {}

# ตัวอย่างหน้า Next.js ที่มี __NEXT_DATA__
def scrape_nextjs_page(url: str) -> dict:
    """ดึงข้อมูลจาก Next.js page"""
    
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # Next.js เก็บ data ใน <script id="__NEXT_DATA__">
    next_data_script = soup.find('script', id='__NEXT_DATA__')
    
    if next_data_script and next_data_script.string:
        return json.loads(next_data_script.string)
    
    return {}
```

---

## 11. Rate Limiting และ Politeness

```python
import time
import random
import requests
from typing import Optional, Callable, TypeVar
from functools import wraps
import threading

T = TypeVar('T')

class RateLimiter:
    """Rate limiter สำหรับควบคุมความเร็วในการส่ง requests"""
    
    def __init__(self, max_calls: int, period: float):
        """
        max_calls: จำนวน calls สูงสุดใน period วินาที
        period: ช่วงเวลาเป็นวินาที
        """
        self.max_calls = max_calls
        self.period = period
        self.calls = []
        self.lock = threading.Lock()
    
    def __call__(self, func: Callable[..., T]) -> Callable[..., T]:
        @wraps(func)
        def wrapper(*args, **kwargs):
            with self.lock:
                now = time.time()
                # ลบ calls ที่หมดอายุ
                self.calls = [c for c in self.calls if now - c < self.period]
                
                if len(self.calls) >= self.max_calls:
                    sleep_time = self.period - (now - self.calls[0])
                    if sleep_time > 0:
                        print(f"Rate limit: sleeping {sleep_time:.2f}s")
                        time.sleep(sleep_time)
                
                self.calls.append(time.time())
            
            return func(*args, **kwargs)
        
        return wrapper

# ใช้งาน decorator
rate_limiter = RateLimiter(max_calls=5, period=1.0)  # 5 requests per second

@rate_limiter
def fetch_url(url: str) -> Optional[str]:
    response = requests.get(url, timeout=10)
    return response.text if response.status_code == 200 else None

# Exponential backoff สำหรับ retry
def fetch_with_retry(
    url: str,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0
) -> Optional[requests.Response]:
    """Fetch URL ด้วย exponential backoff retry"""
    
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=30)
            
            if response.status_code == 429:  # Too Many Requests
                retry_after = int(response.headers.get('Retry-After', base_delay * 2**attempt))
                print(f"Rate limited. Waiting {retry_after}s...")
                time.sleep(retry_after)
                continue
            
            response.raise_for_status()
            return response
            
        except requests.exceptions.HTTPError as e:
            if e.response.status_code >= 500:
                # Server error - retry
                delay = min(base_delay * 2**attempt + random.uniform(0, 1), max_delay)
                print(f"Server error on attempt {attempt+1}. Retrying in {delay:.2f}s...")
                time.sleep(delay)
            else:
                raise  # Client error - ไม่ retry
        
        except requests.exceptions.ConnectionError:
            delay = min(base_delay * 2**attempt, max_delay)
            print(f"Connection error on attempt {attempt+1}. Retrying in {delay:.2f}s...")
            time.sleep(delay)
    
    return None

# Polite scraper
class PoliteScraper:
    """Scraper ที่มี rate limiting และ politeness"""
    
    def __init__(
        self,
        min_delay: float = 1.0,
        max_delay: float = 3.0,
        user_agent: str = "PoliteScraper/1.0"
    ):
        self.min_delay = min_delay
        self.max_delay = max_delay
        self.session = requests.Session()
        self.session.headers.update({'User-Agent': user_agent})
        self._last_request_time = 0
    
    def get(self, url: str, **kwargs) -> requests.Response:
        """Fetch URL ด้วย polite delay"""
        
        # ตรวจสอบ robots.txt
        from urllib.parse import urlparse
        from urllib.robotparser import RobotFileParser
        
        parsed = urlparse(url)
        base_url = f"{parsed.scheme}://{parsed.netloc}"
        
        # Apply delay
        elapsed = time.time() - self._last_request_time
        delay = random.uniform(self.min_delay, self.max_delay)
        
        if elapsed < delay:
            sleep_time = delay - elapsed
            time.sleep(sleep_time)
        
        self._last_request_time = time.time()
        
        response = self.session.get(url, timeout=30, **kwargs)
        return response
```

---

## 12. robots.txt Compliance

```python
from urllib.robotparser import RobotFileParser
from urllib.parse import urlparse
import requests
from functools import lru_cache

class RobotsChecker:
    """ตรวจสอบ robots.txt compliance"""
    
    def __init__(self, user_agent: str = "*"):
        self.user_agent = user_agent
        self._parsers = {}
    
    @lru_cache(maxsize=100)
    def _get_parser(self, base_url: str) -> RobotFileParser:
        """โหลดและ cache robots.txt parser"""
        
        parser = RobotFileParser()
        robots_url = f"{base_url}/robots.txt"
        
        try:
            parser.set_url(robots_url)
            parser.read()
        except Exception as e:
            print(f"Could not fetch robots.txt from {base_url}: {e}")
        
        return parser
    
    def can_fetch(self, url: str) -> bool:
        """ตรวจสอบว่า URL นี้ allowed หรือไม่"""
        
        parsed = urlparse(url)
        base_url = f"{parsed.scheme}://{parsed.netloc}"
        
        parser = self._get_parser(base_url)
        return parser.can_fetch(self.user_agent, url)
    
    def get_crawl_delay(self, url: str) -> float:
        """ดึง crawl delay จาก robots.txt"""
        
        parsed = urlparse(url)
        base_url = f"{parsed.scheme}://{parsed.netloc}"
        
        parser = self._get_parser(base_url)
        delay = parser.crawl_delay(self.user_agent)
        
        return delay if delay else 1.0  # default 1 second

# ใช้งาน
checker = RobotsChecker("MyBot/1.0")

urls_to_check = [
    "https://www.google.com/search",
    "https://www.google.com/",
    "https://books.toscrape.com/",
]

for url in urls_to_check:
    allowed = checker.can_fetch(url)
    delay = checker.get_crawl_delay(url)
    print(f"{url}: allowed={allowed}, crawl_delay={delay}s")
```

---

## 13. Data Extraction Patterns

```python
from bs4 import BeautifulSoup
import re
from typing import List, Dict, Optional
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float
    currency: str = "USD"
    rating: Optional[float] = None
    review_count: int = 0
    url: str = ""
    image_url: str = ""
    categories: List[str] = field(default_factory=list)

def extract_price(price_text: str) -> tuple[float, str]:
    """Parse price string เป็น float และ currency"""
    
    # ลบ whitespace
    price_text = price_text.strip()
    
    # Currency symbols
    currency_map = {
        '$': 'USD',
        '€': 'EUR',
        '£': 'GBP',
        '¥': 'JPY',
        '฿': 'THB',
    }
    
    currency = 'USD'
    for symbol, code in currency_map.items():
        if symbol in price_text:
            currency = code
            price_text = price_text.replace(symbol, '')
            break
    
    # ลบ comma และแปลงเป็น float
    price_text = re.sub(r'[^\d.]', '', price_text)
    
    try:
        return float(price_text), currency
    except ValueError:
        return 0.0, currency

def extract_rating(rating_element) -> Optional[float]:
    """Extract rating จาก star rating element"""
    
    if not rating_element:
        return None
    
    # Class-based ratings (เช่น "star-rating Three")
    classes = rating_element.get('class', [])
    rating_words = {
        'one': 1, 'two': 2, 'three': 3, 'four': 4, 'five': 5
    }
    
    for cls in classes:
        if cls.lower() in rating_words:
            return float(rating_words[cls.lower()])
    
    # Numeric rating (เช่น "4.5/5")
    text = rating_element.get_text()
    match = re.search(r'([\d.]+)\s*/\s*\d+', text)
    if match:
        return float(match.group(1))
    
    # Aria label (เช่น "4.5 out of 5")
    aria_label = rating_element.get('aria-label', '')
    match = re.search(r'([\d.]+)', aria_label)
    if match:
        return float(match.group(1))
    
    return None

def clean_text(text: str) -> str:
    """Clean และ normalize text"""
    if not text:
        return ""
    
    # ลบ extra whitespace
    text = re.sub(r'\s+', ' ', text)
    # ลบ special characters ที่ไม่จำเป็น
    text = text.strip()
    return text

def extract_emails(text: str) -> List[str]:
    """Extract email addresses จาก text"""
    
    pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
    return re.findall(pattern, text)

def extract_phone_numbers(text: str) -> List[str]:
    """Extract phone numbers จาก text"""
    
    # Thai phone numbers
    thai_pattern = r'(?:0[0-9]{1,2}[-\s]?[0-9]{3,4}[-\s]?[0-9]{4})'
    # International
    intl_pattern = r'(?:\+[1-9]\d{1,14})'
    
    phones = re.findall(thai_pattern, text) + re.findall(intl_pattern, text)
    return [re.sub(r'[\s-]', '', p) for p in phones]

def extract_structured_data(html: str) -> dict:
    """ดึง JSON-LD structured data จากหน้าเว็บ"""
    
    import json
    soup = BeautifulSoup(html, 'html.parser')
    
    structured_data = []
    for script in soup.find_all('script', type='application/ld+json'):
        try:
            data = json.loads(script.string)
            structured_data.append(data)
        except json.JSONDecodeError:
            pass
    
    return structured_data
```

---

## 14. Storing Scraped Data (CSV, JSON, DB)

```python
import csv
import json
import sqlite3
from typing import List, Dict, Any
from pathlib import Path
from datetime import datetime

class DataStorage:
    """Helper class สำหรับบันทึกข้อมูลที่ scrape ได้"""
    
    def __init__(self, output_dir: str = "scraped_data"):
        self.output_dir = Path(output_dir)
        self.output_dir.mkdir(exist_ok=True)
    
    def save_csv(self, data: List[Dict], filename: str) -> Path:
        """บันทึกเป็น CSV"""
        
        if not data:
            return None
        
        filepath = self.output_dir / filename
        
        with open(filepath, 'w', newline='', encoding='utf-8') as f:
            writer = csv.DictWriter(f, fieldnames=data[0].keys())
            writer.writeheader()
            writer.writerows(data)
        
        print(f"Saved {len(data)} records to {filepath}")
        return filepath
    
    def save_json(self, data: Any, filename: str, indent: int = 2) -> Path:
        """บันทึกเป็น JSON"""
        
        filepath = self.output_dir / filename
        
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(data, f, indent=indent, ensure_ascii=False, default=str)
        
        print(f"Saved to {filepath}")
        return filepath
    
    def load_json(self, filename: str) -> Any:
        """โหลด JSON"""
        
        filepath = self.output_dir / filename
        
        with open(filepath, 'r', encoding='utf-8') as f:
            return json.load(f)

class SQLiteStorage:
    """SQLite storage สำหรับ scraped data"""
    
    def __init__(self, db_path: str = "scraped.db"):
        self.db_path = db_path
        self.conn = sqlite3.connect(db_path)
        self.conn.row_factory = sqlite3.Row
        self._setup()
    
    def _setup(self):
        self.conn.executescript('''
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                price REAL,
                currency TEXT DEFAULT 'USD',
                rating REAL,
                review_count INTEGER DEFAULT 0,
                url TEXT UNIQUE,
                image_url TEXT,
                scraped_at TEXT NOT NULL
            );
            
            CREATE TABLE IF NOT EXISTS categories (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT UNIQUE NOT NULL
            );
            
            CREATE TABLE IF NOT EXISTS product_categories (
                product_id INTEGER REFERENCES products(id),
                category_id INTEGER REFERENCES categories(id),
                PRIMARY KEY (product_id, category_id)
            );
            
            CREATE INDEX IF NOT EXISTS idx_products_url ON products(url);
            CREATE INDEX IF NOT EXISTS idx_products_price ON products(price);
        ''')
        self.conn.commit()
    
    def upsert_product(self, product: Dict) -> int:
        """Insert หรือ update product"""
        
        cursor = self.conn.execute('''
            INSERT INTO products (name, price, currency, rating, review_count, url, image_url, scraped_at)
            VALUES (:name, :price, :currency, :rating, :review_count, :url, :image_url, :scraped_at)
            ON CONFLICT(url) DO UPDATE SET
                price = excluded.price,
                rating = excluded.rating,
                review_count = excluded.review_count,
                scraped_at = excluded.scraped_at
        ''', {
            'name': product.get('name', ''),
            'price': product.get('price'),
            'currency': product.get('currency', 'USD'),
            'rating': product.get('rating'),
            'review_count': product.get('review_count', 0),
            'url': product.get('url', ''),
            'image_url': product.get('image_url', ''),
            'scraped_at': datetime.now().isoformat()
        })
        
        self.conn.commit()
        return cursor.lastrowid
    
    def get_products(self, min_price: float = None, max_price: float = None) -> List[Dict]:
        """ดึง products ตาม criteria"""
        
        query = "SELECT * FROM products WHERE 1=1"
        params = []
        
        if min_price is not None:
            query += " AND price >= ?"
            params.append(min_price)
        
        if max_price is not None:
            query += " AND price <= ?"
            params.append(max_price)
        
        query += " ORDER BY price ASC"
        
        cursor = self.conn.execute(query, params)
        return [dict(row) for row in cursor.fetchall()]
    
    def close(self):
        self.conn.close()
```

---

## 15. ตัวอย่างโปรแกรมจริง: Product Scraper

```python
import requests
from bs4 import BeautifulSoup
import json
import time
import re
from datetime import datetime
from typing import List, Dict, Optional
from dataclasses import dataclass, asdict

@dataclass
class Book:
    title: str
    price: float
    rating: str
    availability: str
    url: str
    scraped_at: str = ""
    
    def __post_init__(self):
        if not self.scraped_at:
            self.scraped_at = datetime.now().isoformat()

class BookScraper:
    """Scraper สำหรับ books.toscrape.com"""
    
    BASE_URL = "https://books.toscrape.com/catalogue"
    
    RATING_MAP = {
        'One': 1, 'Two': 2, 'Three': 3, 
        'Four': 4, 'Five': 5
    }
    
    def __init__(self, delay: float = 0.5):
        self.delay = delay
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'BookScraper/1.0 (Educational Purpose)'
        })
        self._books: List[Book] = []
    
    def _fetch(self, url: str) -> Optional[BeautifulSoup]:
        try:
            time.sleep(self.delay)
            response = self.session.get(url, timeout=30)
            response.raise_for_status()
            return BeautifulSoup(response.text, 'html.parser')
        except requests.exceptions.RequestException as e:
            print(f"Error fetching {url}: {e}")
            return None
    
    def _parse_price(self, price_text: str) -> float:
        cleaned = re.sub(r'[^\d.]', '', price_text)
        return float(cleaned) if cleaned else 0.0
    
    def _parse_rating(self, article) -> str:
        rating_el = article.select_one('.star-rating')
        if rating_el:
            classes = rating_el.get('class', [])
            for cls in classes:
                if cls in self.RATING_MAP:
                    return cls
        return 'Unknown'
    
    def scrape_catalogue_page(self, url: str) -> tuple[List[Book], Optional[str]]:
        """Scrape หน้า catalogue และ return books + next page URL"""
        
        soup = self._fetch(url)
        if not soup:
            return [], None
        
        books = []
        
        for article in soup.select('article.product_pod'):
            title = article.select_one('h3 a').get('title', '')
            price_text = article.select_one('.price_color').text
            price = self._parse_price(price_text)
            rating = self._parse_rating(article)
            availability_text = article.select_one('.availability').text.strip()
            relative_url = article.select_one('h3 a')['href']
            full_url = f"{self.BASE_URL}/{relative_url.lstrip('../')}"
            
            book = Book(
                title=title,
                price=price,
                rating=rating,
                availability=availability_text,
                url=full_url
            )
            books.append(book)
        
        # หา next page
        next_link = soup.select_one('.pager .next a')
        next_url = None
        if next_link:
            next_href = next_link['href']
            current_dir = url.rsplit('/', 1)[0]
            next_url = f"{current_dir}/{next_href}"
        
        return books, next_url
    
    def scrape(self, max_pages: int = 5) -> List[Book]:
        """Scrape หลายหน้า"""
        
        url = f"{self.BASE_URL}/page-1.html"
        page = 1
        
        while url and page <= max_pages:
            print(f"Scraping page {page}...")
            books, next_url = self.scrape_catalogue_page(url)
            
            self._books.extend(books)
            print(f"  Found {len(books)} books (total: {len(self._books)})")
            
            url = next_url
            page += 1
        
        return self._books
    
    def save_results(self, filename: str = "books.json") -> None:
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(
                [asdict(book) for book in self._books],
                f, indent=2, ensure_ascii=False
            )
        print(f"Saved {len(self._books)} books to {filename}")
    
    def get_top_rated(self, min_rating: str = 'Four') -> List[Book]:
        rating_order = ['One', 'Two', 'Three', 'Four', 'Five']
        min_idx = rating_order.index(min_rating)
        
        return [
            book for book in self._books
            if book.rating in rating_order and 
            rating_order.index(book.rating) >= min_idx
        ]
    
    def get_cheapest(self, n: int = 10) -> List[Book]:
        return sorted(self._books, key=lambda b: b.price)[:n]

# ใช้งาน
# scraper = BookScraper(delay=0.5)
# books = scraper.scrape(max_pages=3)
# scraper.save_results("books.json")
# 
# top_rated = scraper.get_top_rated('Four')
# print(f"Top rated books: {len(top_rated)}")
# 
# cheapest = scraper.get_cheapest(5)
# for book in cheapest:
#     print(f"{book.title}: £{book.price:.2f}")
```

### News Aggregator

```python
import requests
from bs4 import BeautifulSoup
from datetime import datetime
from typing import List, Dict
import feedparser  # pip install feedparser

class NewsAggregator:
    """Aggregate news จากหลาย sources"""
    
    RSS_FEEDS = {
        'techcrunch': 'https://techcrunch.com/feed/',
        'bbc': 'http://feeds.bbci.co.uk/news/technology/rss.xml',
        'hacker_news': 'https://hnrss.org/frontpage',
    }
    
    def __init__(self):
        self.articles = []
    
    def fetch_rss(self, source: str, url: str) -> List[Dict]:
        """ดึง articles จาก RSS feed"""
        
        try:
            feed = feedparser.parse(url)
            
            articles = []
            for entry in feed.entries[:10]:
                article = {
                    'source': source,
                    'title': entry.get('title', ''),
                    'url': entry.get('link', ''),
                    'summary': entry.get('summary', ''),
                    'published': entry.get('published', ''),
                }
                
                # Clean HTML จาก summary
                if article['summary']:
                    soup = BeautifulSoup(article['summary'], 'html.parser')
                    article['summary'] = soup.get_text()[:500]
                
                articles.append(article)
            
            return articles
        
        except Exception as e:
            print(f"Error fetching {source}: {e}")
            return []
    
    def aggregate(self) -> List[Dict]:
        """ดึง news จากทุก sources"""
        
        all_articles = []
        
        for source, url in self.RSS_FEEDS.items():
            print(f"Fetching from {source}...")
            articles = self.fetch_rss(source, url)
            all_articles.extend(articles)
            print(f"  Got {len(articles)} articles")
        
        self.articles = all_articles
        return all_articles

# ใช้งาน
# aggregator = NewsAggregator()
# articles = aggregator.aggregate()
# print(f"Total articles: {len(articles)}")
```

### Price Monitor

```python
import requests
from bs4 import BeautifulSoup
import json
import sqlite3
from datetime import datetime
from typing import Optional

class PriceMonitor:
    """Monitor ราคาสินค้าและแจ้งเตือนเมื่อราคาลด"""
    
    def __init__(self, db_path: str = "price_history.db"):
        self.conn = sqlite3.connect(db_path)
        self._setup_db()
    
    def _setup_db(self):
        self.conn.executescript('''
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                url TEXT UNIQUE NOT NULL,
                target_price REAL,
                created_at TEXT NOT NULL
            );
            
            CREATE TABLE IF NOT EXISTS price_history (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                product_id INTEGER REFERENCES products(id),
                price REAL NOT NULL,
                currency TEXT DEFAULT 'USD',
                checked_at TEXT NOT NULL
            );
        ''')
        self.conn.commit()
    
    def add_product(self, name: str, url: str, target_price: float = None) -> int:
        cursor = self.conn.execute('''
            INSERT OR IGNORE INTO products (name, url, target_price, created_at)
            VALUES (?, ?, ?, ?)
        ''', (name, url, target_price, datetime.now().isoformat()))
        self.conn.commit()
        return cursor.lastrowid
    
    def record_price(self, product_id: int, price: float, currency: str = "USD"):
        self.conn.execute('''
            INSERT INTO price_history (product_id, price, currency, checked_at)
            VALUES (?, ?, ?, ?)
        ''', (product_id, price, currency, datetime.now().isoformat()))
        self.conn.commit()
    
    def get_price_history(self, product_id: int) -> list:
        cursor = self.conn.execute('''
            SELECT price, currency, checked_at
            FROM price_history
            WHERE product_id = ?
            ORDER BY checked_at DESC
            LIMIT 30
        ''', (product_id,))
        return cursor.fetchall()
    
    def check_price(self, product_id: int, current_price: float) -> bool:
        """ตรวจว่าราคา drop ถึง target หรือยัง"""
        
        product = self.conn.execute(
            "SELECT target_price FROM products WHERE id = ?", 
            (product_id,)
        ).fetchone()
        
        if product and product[0] is not None:
            return current_price <= product[0]
        
        return False
    
    def get_min_price(self, product_id: int) -> Optional[float]:
        result = self.conn.execute('''
            SELECT MIN(price) FROM price_history WHERE product_id = ?
        ''', (product_id,)).fetchone()
        return result[0] if result else None

# ใช้งาน
# monitor = PriceMonitor()
# product_id = monitor.add_product("Laptop", "https://example.com/laptop", target_price=800)
# monitor.record_price(product_id, 999.99)
# monitor.record_price(product_id, 899.99)
# 
# min_price = monitor.get_min_price(product_id)
# print(f"Minimum price recorded: ${min_price}")
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Basic BeautifulSoup

```python
# ดึงข้อมูลจาก HTML ที่กำหนด

from bs4 import BeautifulSoup

html = """
<html>
<body>
  <div class="news">
    <article id="article-1">
      <h2><a href="/news/1">Python 3.13 Released</a></h2>
      <p class="meta">By <span class="author">John</span> on <span class="date">2024-10-01</span></p>
      <p class="summary">Python 3.13 brings many improvements...</p>
      <div class="tags">
        <span class="tag">python</span>
        <span class="tag">programming</span>
      </div>
    </article>
    <article id="article-2">
      <h2><a href="/news/2">FastAPI vs Django</a></h2>
      <p class="meta">By <span class="author">Jane</span> on <span class="date">2024-10-02</span></p>
      <p class="summary">Comparing popular Python web frameworks...</p>
      <div class="tags">
        <span class="tag">fastapi</span>
        <span class="tag">django</span>
        <span class="tag">web</span>
      </div>
    </article>
  </div>
</body>
</html>
"""

def parse_articles(html_content: str) -> list:
    """Parse articles จาก HTML"""
    soup = BeautifulSoup(html_content, 'html.parser')
    articles = []
    
    for article in soup.select('article'):
        data = {
            'id': article.get('id'),
            'title': article.select_one('h2 a').text,
            'url': article.select_one('h2 a')['href'],
            'author': article.select_one('.author').text,
            'date': article.select_one('.date').text,
            'summary': article.select_one('.summary').text,
            'tags': [tag.text for tag in article.select('.tag')]
        }
        articles.append(data)
    
    return articles

result = parse_articles(html)
for article in result:
    print(f"Title: {article['title']}")
    print(f"Author: {article['author']}, Date: {article['date']}")
    print(f"Tags: {', '.join(article['tags'])}")
    print()
```

### แบบฝึกหัดที่ 2: ดึงข้อมูลจาก quotes.toscrape.com

```python
import requests
from bs4 import BeautifulSoup
from typing import List, Dict

def scrape_quotes(max_pages: int = 3) -> List[Dict]:
    """Scrape quotes จาก quotes.toscrape.com"""
    
    quotes = []
    url = "https://quotes.toscrape.com/"
    
    for page in range(1, max_pages + 1):
        page_url = f"{url}page/{page}/"
        response = requests.get(page_url, timeout=30)
        soup = BeautifulSoup(response.text, 'html.parser')
        
        for quote_div in soup.select('.quote'):
            quote = {
                'text': quote_div.select_one('.text').text.strip('""“”'),
                'author': quote_div.select_one('.author').text,
                'tags': [tag.text for tag in quote_div.select('.tag')],
                'author_url': "https://quotes.toscrape.com" + 
                             quote_div.select_one('a')['href']
            }
            quotes.append(quote)
        
        # Check next page
        if not soup.select_one('.next'):
            break
    
    return quotes

# ใช้งาน
# quotes = scrape_quotes(2)
# print(f"Total quotes: {len(quotes)}")
# for q in quotes[:3]:
#     print(f'"{q["text"]}" - {q["author"]}')
```

### แบบฝึกหัดที่ 3: Rate-limited scraper

```python
import time
import random
from typing import Callable, TypeVar

T = TypeVar('T')

class Scraper:
    """Rate-limited scraper"""
    
    def __init__(self, min_delay: float = 1.0, max_delay: float = 3.0):
        self.min_delay = min_delay
        self.max_delay = max_delay
        self._request_count = 0
        self._start_time = time.time()
    
    @property
    def requests_per_minute(self) -> float:
        elapsed = time.time() - self._start_time
        return (self._request_count / elapsed) * 60 if elapsed > 0 else 0
    
    def polite_sleep(self) -> None:
        delay = random.uniform(self.min_delay, self.max_delay)
        time.sleep(delay)
    
    def get(self, url: str) -> requests.Response:
        self.polite_sleep()
        self._request_count += 1
        
        response = requests.get(url, timeout=30)
        print(f"Request #{self._request_count}: {url} ({response.status_code})")
        return response

# สร้าง scraper แล้วใช้งาน
scraper = Scraper(min_delay=0.5, max_delay=1.5)
print(f"Scraper ready. Rate: ~{60/(scraper.min_delay+scraper.max_delay)*2:.1f} req/min")
```

### แบบฝึกหัดที่ 4-8 (โค้ดสรุป)

```python
# แบบฝึกหัดที่ 4: CSS Selectors Practice
# แบบฝึกหัดที่ 5: XPath Practice  
# แบบฝึกหัดที่ 6: JSON API Scraping
# แบบฝึกหัดที่ 7: Save to CSV/JSON/SQLite
# แบบฝึกหัดที่ 8: Complete product scraper

# 4: CSS Selectors
soup = BeautifulSoup(html, 'html.parser')
# ดึง h2 ทุกตัวใน article
titles = soup.select('article h2')
# ดึง a ที่อยู่ใน h2 ใน article
links = soup.select('article h2 a')
# ดึง article ที่มี id ขึ้นต้นด้วย "article-"
articles = soup.select('article[id^="article-"]')

# 7: บันทึกข้อมูล
import csv, json

data = [{'title': 'Python', 'url': '/python'}, {'title': 'Scrapy', 'url': '/scrapy'}]

# CSV
with open('data.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=['title', 'url'])
    writer.writeheader()
    writer.writerows(data)

# JSON
with open('data.json', 'w') as f:
    json.dump(data, f, indent=2)

print("Data saved!")
```

---

## สรุป

| Tool | ใช้เมื่อ |
|------|--------|
| `requests` + `BeautifulSoup` | Static HTML pages |
| `requests` + XPath (lxml) | Complex HTML navigation |
| `Scrapy` | Large-scale scraping, crawling |
| `Selenium` | JavaScript-rendered pages, form interaction |
| `Playwright` | Modern alternative to Selenium, async support |
| Direct API calls | เมื่อเว็บมี API ให้ใช้ |

**Best Practices:**
1. ตรวจ `robots.txt` ก่อนเสมอ
2. ใช้ polite delays (1-3 วินาที)
3. Cache ข้อมูลที่ดึงมาแล้ว
4. Handle errors อย่างรัดกุม
5. ระบุ User-Agent ที่ชัดเจน
6. ไม่ scrape ข้อมูลส่วนตัวโดยไม่ได้รับอนุญาต
