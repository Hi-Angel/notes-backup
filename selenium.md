# Setup

Besides the API part *(in whatever programming language)* this also makes use of two binaries: the browser and the browser driver. E.g.: `chrome` and `chromedriver`. The binaries must have matching version.

Now, the problem is there's also a thing called `selenium-manager`. It's an insidious program that tries to download a driver even if there is one in your system. In my experience it also downloads the wrong version, so good luck trying to figure out why tests do not run later *(a version mismatch won't produce meaningful error message)*. Needless to say, this "selenium-manager" is a very problematic default, in particular because suddenly downloading random binaries from the internet while you run tests is a completely unwanted event, and especially so if the code is part of CI.

To avoid this *(tested as of Selenium 4)* you have to initialize `chromedriver` path while creating webdriver. So e.g. *(error checking omitted)*:

```python
from selenium.webdriver.chrome.service import Service
from shutil import which
[…]
webdriver.Chrome(options=options, service = Service(which('chromedriver')))
```
