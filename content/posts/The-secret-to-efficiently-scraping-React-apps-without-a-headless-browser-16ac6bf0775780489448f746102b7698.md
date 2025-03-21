---
title: "The secret to efficiently scraping React apps without a headless browser"
date: "2024-01-22T00:00:00.000-05:00"
lastmod: "2025-01-30T05:12:00.000Z"
draft: false
series: []
authors:
  - "Peter"
tags:
  - "engineering"
categories: []
NOTION_METADATA:
  object: "page"
  id: "16ac6bf0-7757-8048-9448-f746102b7698"
  created_time: "2024-12-28T15:50:00.000Z"
  last_edited_time: "2025-01-30T05:12:00.000Z"
  created_by:
    object: "user"
    id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
  last_edited_by:
    object: "user"
    id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
  cover: null
  icon: null
  parent:
    type: "database_id"
    database_id: "167c6bf0-7757-810a-95f9-cfb3b2adb3e1"
  archived: false
  in_trash: false
  properties:
    series:
      id: "B%3C%3FS"
      type: "multi_select"
      multi_select: []
    draft:
      id: "JiWU"
      type: "checkbox"
      checkbox: false
    published-date:
      id: "%5C%5CDZ"
      type: "date"
      date:
        start: "2024-01-22T00:00:00.000-05:00"
        end: null
        time_zone: null
    authors:
      id: "bK%3B%5B"
      type: "people"
      people:
        - object: "user"
          id: "911a64eb-c8df-432a-b949-560ff0b1ced3"
          name: "Peter"
          avatar_url: "https://s3-us-west-2.amazonaws.com/public.notion-static.com/c7114a\
            e2-5f13-4f26-95a6-9feccbcca818/profile-pic_(1).png"
          type: "person"
          person:
            email: "notion@peterrauscher.com"
    custom-front-matter:
      id: "c~kA"
      type: "rich_text"
      rich_text: []
    tags:
      id: "jw%7CC"
      type: "multi_select"
      multi_select:
        - id: "d3648cb9-022a-42c7-a0bc-6601edae5057"
          name: "engineering"
          color: "green"
    categories:
      id: "nbY%3F"
      type: "multi_select"
      multi_select: []
    summary:
      id: "x%3AlD"
      type: "rich_text"
      rich_text: []
    Name:
      id: "title"
      type: "title"
      title:
        - type: "text"
          text:
            content: "The secret to efficiently scraping React apps without a headless
              browser"
            link: null
          annotations:
            bold: false
            italic: false
            strikethrough: false
            underline: false
            code: false
            color: "default"
          plain_text: "The secret to efficiently scraping React apps without a headless
            browser"
          href: null
  url: "https://www.notion.so/The-secret-to-efficiently-scraping-React-apps-witho\
    ut-a-headless-browser-16ac6bf0775780489448f746102b7698"
  public_url: "https://peterrauscher.notion.site/The-secret-to-efficiently-scrapi\
    ng-React-apps-without-a-headless-browser-16ac6bf0775780489448f746102b7698"
MANAGED_BY_NOTION_HUGO: true

---


If you've ever tried to build a web-scraping project, you've probably run into issues with dynamically rendered content, common in things like single-page applications (SPAs). Powered by technologies like Next.js and React, these SPAs offer seamless user experiences but pose unique challenges for web scrapers.


I'll walk you through how I solved this challenge _**without**_ using a headless browser, allowing me to keep resource costs low and continue web-scraping at scale. I use dead-simple tooling - Python, the requests library, and some string manipulation - to make the magic happen.


## Why are SPAs harder to scrape?


Traditional websites load all of their content synchronously and reload the entire page for each user navigation. Unlike these static sites, SPAs are a breed of modern web applications that load a single HTML page and dynamically update content with Javascript as users interact with them. Even non-SPA modern websites render more and more of their content through AJAX requests nowadays, from APIs and content management systems like Contentful.


The challenge with scraping SPAs stems from the fact that traditional web scraping tools, such as [BeautifulSoup](https://pypi.org/project/beautifulsoup4/) often struggle to capture data from dynamically rendered content, as they do not execute the Javascript embedded in `<script>` tags.


The usual approach is to use a headless browser like [Selenium](https://www.selenium.dev/), [Puppeteer](https://pptr.dev/), or [Playwright](https://playwright.dev/) and scrape after the page has rendered completely. These tools load the entire web page, including executing JavaScript, rendering styles, and making AJAX requests. This approach can be resource-intensive and sluggish, particularly when dealing with unoptimized React code.


![](https://peterrauscher.com/api?block_id=16ac6bf0-7757-80ee-85a0-dbc629f465e3)


## The Solution


We can use a simple GET request to the page’s URL. This initial request fetches the HTML response, which often has the data we’re looking for embedded in the bundled JS.


```python
import requests

url = "https://example.com/spa"
response = requests.get(url)
html_content = response.text

```


### Extracting React/Next.js Data from HTML


Here's where the magic happens. Our goal is to extract React data from the plaintext response. SPAs frequently load data within script tags, and we'll employ regular expressions (regex) to locate and extract these script tags.


```python
import re
import json

def extractor(html_content: str) -> str:
    react_app_script = re.search(
        "<script>document.getElementById.*</script>", html_content
    )
    return (
        "{"
        + react_app_script.group()
        .split("{", 2)[2]
        .split("};window.initilizeAppWithHandoffState", 1)[0]
        + "}"
    )

react_data = json.loads(extractor(html_content))

```


### Handling Authentication and Cookies


In certain scenarios, SPAs may necessitate authentication or rely on cookies to access data. You can adeptly manage authentication and cookies within your scraping script using the session management features provided by the `requests` library. This enables you to maintain a persistent session and send authenticated requests.


```python
# Create a session
session = requests.Session()

# Authenticate (if needed)
login_data = {
    'username': 'your_username',
    'password': 'your_password'
}
session.post('https://example.com/login', data=login_data)

# Send requests with the session
response = session.get('https://example.com/protected_data')
```


## Conclusion


A little string manipulation goes a long way. Harnessing the power of regex you can create an efficient and lightweight approach to extracting data from SPA sites.


### Chrome Extension


When I was building an ingestion engine for an app I was working on, I wanted a quick way to view the hidden JSON blob I for the page I was viewing to see if it held the data I wanted. I built a chrome extension that displays this data and lets you easily copy or export it to help when writing scraping applications. [It’s available on the Chrome Web Store](https://chromewebstore.google.com/detail/nextscraper/kjlhnflincmlpkgahnidgebbngieobod), check it out!


### _Disclaimer_


_Always respect website terms of service and robots.txt files when scraping data from the web._

