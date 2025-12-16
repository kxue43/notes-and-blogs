---
author: Ke Xue
title: Selenium vs. Playwright
date: 2022-02-05T17:31:35-05:00
draft: false
layout: docs
description: Selenium vs. Playwright for web automation.
tags: 
---

Recently I developed a Python program that scrapes a Single-Page Application with [Selenium][Selenium-Python],
and then reimplemented it with [Playwright][Playwright-Python]. This gave me firsthand knowledge of the difference
between the two. In this article, I shall explain my preference of Playwright over Selenium, and share some general
experience of scraping single-page applications.

## The reason and result of the reimplementation

My scraping program needs to click an anchor element on the target webpage to trigger file download by the browser,
wait for the file download to finish, and then do something with the downloaded file. With Selenium, as surprising as
it may seem, waiting for file download to finish is not straightforward. According to its
[official documentation][Selenium-file-download]: 

> Whilst it is possible to start a download by clicking a link with a browser under Selenium's control,
> the API does not expose download progress.

I did find a workaround from an ingenious answer on Stack Overflow (more on this later), but it imposes two limitations
on the program: (1) the workaround only works with Chrome in the headed mode, making it harder to port to a container;
(2) I had to make a `time.sleep(3)` call after every download to let Chrome finish virus scan on the downloaded file,
and such sleep calls are a well-known source of flakiness in web automation.

Limitation No. 2 is tolerable for my current use case. Being a cloud service enthusiast, it is No. 1 that drove me
to search for a headless solution incessantly. After watching a brilliant [presentation][Playwright-presentation-1]
of Playwright on YouTube, I decided to give it a try. Playwright offers a convenient
[download API][Playwright-download-api], and I was thereby able to remove all equivalents of sleep system calls
from my code and yet definitively get file downloads done right. The Playwright version of the program doubles
the speed of the Selenium version and runs in headless mode without any observed flakiness.

## Selenium vs. Playwright

The presenter in the aforementioned video explains very nicely the difference in *architecture* between Selenium and
Playwright, and its consequence on their respective capabilities. To put it simply, Selenium is a small
[WebDriver][WebDriver] that serves as the intermediary between the application program and the browser.
This limits what information the application program and the browser can communicate with each other,
and it is why Selenium does not currently provide a file download API. Playwright, on the other hand,
utilizes the Chrome DevTools Protocol to give the application program almost full control of the browser.
Besides file download, Playwright offers other APIs that can only be found in extension packages with Selenium,
if they exist at all. Monitoring and modifying the browser's network traffic is one such example.

More concretely, to use Selenium with the Chrome browser, we need to first find the version number of Chrome,
and then go to <https://sites.google.com/chromium.org/driver/downloads> to download the ChromeDriver
of the same version. The downloaded executable `chromedriver.exe` is only 11 MB in size. When the application program
runs, Selenium drives through `chromedriver.exe` the very same Chrome instance that we use for daily web browsing.
With Playwright, we use its CLI command `playwright install chromium` to install a separate Chromium browser
(the open-source part of Chrome), which is over 100 MB in size. When the program runs, the Playwright Python library
drives the separate Chromium browser, which displays its own bluish logo on the task bar. From this we get a first
impression of the architectural difference between Selenium and Playwright --- a small driver versus a full browser.

References:

- [Another presentation of the Playwright architecture][Playwright-presentation-2].

## Waiting for file download

In this section, we compare the different ways that Selenium and Playwright wait for a file download to finish.

### The Selenium version

Since Selenium doesn't offer a download API, I must write a [custom wait condition][Selenium-custom-wait] by myself.
Essentially, it is a callable that takes an instance of the Selenium Python package's WebDriver class as
the only argument, performs some check, and returns True or False based on whether the download is completed or not.
The application program waits for the custom condition to be met by calling this function every 500ms until
it returns True (i.e. polling).

An ingenious [Stack Overflow answer][SO-answer] leads to the following solution.

At the beginning of the scraping run, open two Chrome browser tabs. Go to the target webpage's URL on the first tab
and the special URL `chrome://downloads` on the second. The second tab shows the download history view of
the Chrome browser. After triggering a file download from the first tab, switch to the download history tab
and wait for the following custom wait function to return True.

```python
def download_finishes(driver: WebDriver) -> bool:
    return driver.execute_script(
        """
        let items = document.querySelector(
            'downloads-manager'
        ).shadowRoot.getElementById('downloadsList').items;
        return items[0].state === 'COMPLETE';
        """
    )
```

The `driver.execute_script()` method call injects a piece of JavaScript code to the Chrome browser and lets it execute
the code in the download history tab. The first statement of the JavaScript code finds the custom HTML element
`downloads-manager`, goes into its [Shadow DOM][shadow-DOM], finds the element with id `downloadsList`, gets the `items`
attribute of this element, and assigns it to the variable `items`. The variable `items` is an array of objects,
each of which represents a file that is being downloaded or has been downloaded. `items[0]` is always the latest file,
i.e., the file whose download we just triggered on the first tab. The meaning of the return statement should be
self-evident. The reader can try the triple-quoted JavaScript code easily from the Chrome Developer Console.

After the above function returns True, I still had to make a `time.sleep(3)` call to let Chrome finish virus scan,
before conducting subsequent operations on the downloaded file. This is because Selenium drives the same Chrome
that I use for daily browsing, and I didn't figure out how to turn the virus scan off.

### The Playwright version

Playwright offers a very idiomatic download API. The following code is taken from the Playwright
[documentation][Playwright-download-snippet] directly.

```python
# Start waiting for the download
with page.expect_download() as download_info:
    # Perform the action that initiates download
    page.click("button#delayed-download")
download = download_info.value
# Wait for the download process to complete
path = download.path()
```

Playwright's `page` object is where most interactions with the webpage are initiated. Its use is similar to Selenium's
`driver` object, although `page` represents a browser tab instead of the webdriver. `download_info` is a context manager
that represents the download event. The page interaction that triggers the file download is placed inside
the with statement. The `download.path()` method call returns the path to the downloaded file after the
download completes. Playwright does not perform virus scan on the downloaded file, and names the file by a UUID
with no extension. Clearly, Playwright considers the proper handling of downloaded files a responsibility of
the application program.

It should be quite evident that Playwright does a better job at file downloading with much simpler code.
This is all thanks to its architecture of controlling the browser via the Chrome DevTools Protocol.

The comparison between Selenium and Playwright is over. The rest of the sections contain some general experience of
scraping single-page applications with Playwright.

## Wait for authentication to complete

Nowadays, more and more SPAs are adopting OpenID Connect Authorization Code flow with PKCE to conduct user
authorization/authentication (abbreviated as "auth" hereafter) when necessary. Typically, after a user logs in by MFA
and the webpage loads, the SPA's JavaScript code makes an HTTP POST request to the auth server with
`code` and `code_verifier` in the payload in exchange for an access token. The auth process is NOT complete until
the SPA receives the full response of this POST request.

In such case, the application program should be careful NOT to reload the webpage before the auth process is complete.
Otherwise, it causes the browser to reload the HTML, CSS and JavaScript files, which might interrupt the auth process
and cause the logged-in session to be terminated immediately. A reload operation may exist implicitly when the scraping
operation is iterative and the application program tries to set up the initial state of each iteration by something
like `page.goto(starting_url)`. It is this command in the very first iteration that may interrupt the auth process.

In general, the application program should wait for the auth process to complete before starting the actual scraping
operations. There are several ways to do this with Playwright, and it is a good idea to use them together.

Firstly, we can wait until there are no network connections for at least 500ms, by which time the auth process is
likely to have completed.

```python
page.wait_for_load_state(state='networkidle')
```

Secondly, we can wait for a DOM element whose visibility implies the completion of the auth process.

```python
page.locator('xpath=//some_dom_element').wait_for(state='visible')
```

Finally, we can "poke around" in the SPA using Chrome Developer Console. For example, if it is found that the auth
information is stored in `window.localStorage`, some analogue of the following code can be used.

```python
page.wait_for_function(
    """
    () => {
        session = window.localStorage;
        return (
            session.hasOwnProperty('authenticated') &&
            session.authenticated.hasOwnProperty('access_token')
        );
    }
    """
)
```

It is better to carefully observe and analyze the SPA and figure out the specific wait conditions than to blindly
use `page.wait_for_timeout()`, as the latter is expressly discouraged by the Playwright
[documentation][Playwright-wait-for-timeout].

References:

- [OAuth2.0][Okta-1], on which OpenID Connect is based.
- [OpenID Connect Authorization Code flow with PKCE][Okta-2]

## Learn the XPath syntax

It is not uncommon for a front-end framework/library to create `id` attributes for a lot of DOM elements in an HTML
that it dynamically generates. React and Ember both do this. In this case, the XPath selector that we get by
`right-click => Copy => Copy XPath` from the Chrome Developer Console often starts with such an `id` from the closest
ancestor of our target element. This type of selector is useless for web scraping, since the `id` value is different
for every generation of the HTML. It is better to analyze the SPA and hard code the XPaths by ourselves.

References:

- [XPath Syntax][xpath-syntax].

## Do not wait by sleeping

When the application program waits by making sleep system calls, it might cause the OS scheduler to push the entire
process back on the ready queue. This is especially the case when the the program runs in the cloud and is scheduled
together with many other container instances.

Playwright, in addition, uses asynchronous logic under the hood, so `time.sleep()` is simply incompatible.
`page.wait_for_timeout()` should only be used for debugging. It is always better to wait by actively polling using
methods such as `page.wait_for_function()`.

[Selenium-Python]: https://selenium-python.readthedocs.io/
[Playwright-Python]: https://playwright.dev/python/
[Selenium-file-download]: https://www.selenium.dev/documentation/test_practices/discouraged/file_downloads/
[Playwright-presentation-1]: https://www.youtube.com/watch?v=_Jla6DyuEu4
[Playwright-download-api]: https://playwright.dev/python/docs/downloads
[WebDriver]: https://www.selenium.dev/documentation/webdriver/
[chromium-logo]: https://en.wikipedia.org/wiki/File:Chromium_11_Logo.svg
[Playwright-presentation-2]: https://www.youtube.com/watch?v=PXTspGn1im0&t=482s
[Selenium-custom-wait]: https://selenium-python.readthedocs.io/waits.html#explicit-waits
[SO-answer]: https://stackoverflow.com/questions/48263317/selenium-python-waiting-for-a-download-process-to-complete-using-chrome-web/48267887
[shadow-DOM]: https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM
[Playwright-download-snippet]: https://playwright.dev/python/docs/downloads
[Playwright-wait-for-timeout]: https://playwright.dev/python/docs/api/class-page#page-wait-for-timeout
[Okta-1]: https://www.youtube.com/watch?v=996OiexHze0
[Okta-2]: https://developer.okta.com/blog/2019/05/01/is-the-oauth-implicit-flow-dead
[xpath-syntax]: https://www.w3schools.com/xml/xpath_syntax.asp
