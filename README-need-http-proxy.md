# Need http proxy, network is very slow.


# selenium-workshop
Play with Selenium using Docker.


# Build docker images(Only need to be done once)
Build selenium-3.4.1

    BUILD_ARGS="--build-arg http_proxy=http://192.168.234.1:8123 --build-arg https_proxy=http://192.168.234.1:8123" \
    docker build $(BUILD_ARGS) -t selenium-3.4.1 ./selenium


Build Selenium Standalone Servers
```bash
git clone https://github.com/SeleniumHQ/docker-selenium.git
cd docker-selenium

# Choose one method to build images:

# (Option 1)Do not need http proxy
VERSION=local make build

# (Option 2)Need http proxy, here http proxy(http://192.168.234.1:8123) should be replaced accordingly.
VERSION=local \
BUILD_ARGS="--build-arg http_proxy=http://192.168.234.1:8123 --build-arg https_proxy=http://192.168.234.1:8123" \
make build
```


# Example
Get github projects treading today for popular languages.

First, run Selenium Standalone Server(using docker images built locally).

Here Chrome with vnc support is selected, 192.168.234.1 is host ip.
Refers https://github.com/SeleniumHQ/docker-selenium#running-the-images.
```bash
# For osx
# Prepare ram_disk first
SIZE_IN_GB=2
diskutil eraseVolume HFS+ "ram_disk" `hdiutil attach -nomount ram://$((2 * 1024 * 1024 * $SIZE_IN_GB))`

docker run -d -p 192.168.234.1:4444:4444 -p 192.168.234.1:5900:5900 -v /Volumes/ram_disk:/dev/shm selenium/standalone-chrome-debug:local


# For linux
docker run -d -p 192.168.234.1:4444:4444 -p 192.168.234.1:5900:5900 -v /dev/shm:/dev/shm selenium/standalone-chrome-debug:local
```

Second, run selenium-3.4.1:

    docker run -it selenium-3.4.1


Test case can be input now:
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
