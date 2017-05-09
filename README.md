# selenium-workshop
Play with Selenium using Docker.

If you need to use http proxy to pull docker images or install packages online, refer [README-need-http-proxy.md](README-need-http-proxy.md).


# Build docker image(Only need to be done once)
Build selenium-3.4.1

    docker build -t selenium-3.4.1 ./selenium


# Example
Get github projects treading today for popular languages.

First, run Selenium Standalone Server.

* There are several versions of Selenium standalone server, Chrome with vnc support is used here
* 192.168.234.1 is host ip
* port 5900 is for vnc, default password of vnc is `secret`
* port 4444 is for selenium api

Refer https://github.com/SeleniumHQ/docker-selenium#running-the-images.

```bash
# For osx
# Prepare ram_disk first
SIZE_IN_GB=2
diskutil eraseVolume HFS+ "ram_disk" `hdiutil attach -nomount ram://$((2 * 1024 * 1024 * $SIZE_IN_GB))`

docker run -d -p 192.168.234.1:4444:4444 -p 192.168.234.1:5900:5900 -v /Volumes/ram_disk:/dev/shm selenium/standalone-chrome-debug:3.4.0-chromium


# For linux
docker run -d -p 192.168.234.1:4444:4444 -p 192.168.234.1:5900:5900 -v /dev/shm:/dev/shm selenium/standalone-chrome-debug:3.4.0-chromium
```

Second, run selenium-3.4.1:

    docker run -it selenium-3.4.1


Test case(which fetch and print out trending projects in github) can be input now:
```python
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

driver = webdriver.Remote(command_executor='http://192.168.234.1:4444/wd/hub',
         desired_capabilities=DesiredCapabilities.CHROME)

def fetch_projects(url):
  driver.get(url)
  li_list = driver.find_elements_by_css_selector(".explore-content .repo-list li")
  projects = []
  for li in li_list:
    projects.append(li.find_element_by_css_selector("h3 a").text)
  return projects

def fetch_trending_urls(url):
  driver.get(url)
  navi_list = driver.find_elements_by_css_selector("div.explore-pjax-container.container.explore-page > div.columns > div.column.one-fourth > ul li a")
  urls = []
  for navi in navi_list:
    urls.append(navi.get_property("href"))
  return urls

urls = fetch_trending_urls("https://github.com/trending")
print urls

projects_map = {}
for url in urls:
  projects_map[url] = fetch_projects(url)

for url in projects_map:
  print url
  for p in projects_map[url]:
    print "  ", p
```
