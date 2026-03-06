# Chat Conversation

Note: _This is purely the output of the chat conversation and does not contain any raw data, codebase snippets, etc. used to generate the output._

### User Input

The CustomArticleCrawler and GithubCrawler inherit from BaseCrawler. The classes LinkedInCrawler and MediumCrawler inherit from BaseSeleniumCrawler. What does this tell me?

*Listed directory [crawlers](file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers) *

*Viewed [base.py](file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py) *

*Viewed [custom_article.py](file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/custom_article.py) *

### Planner Response

The fact that [CustomArticleCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/custom_article.py:11:0-53:65) and `GithubCrawler` inherit directly from [BaseCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:16:0-20:55), while `LinkedInCrawler` and `MediumCrawler` inherit from [BaseSeleniumCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:23:0-65:31) (which in turn inherits from [BaseCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:16:0-20:55)), reveals some important architectural and operational details about how your application interacts with these different platforms:

### 1. Different Scraping Strategies (Static vs. Dynamic)
*   **Static/API-based ([BaseCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:16:0-20:55))**: [CustomArticleCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/custom_article.py:11:0-53:65) and `GithubCrawler` can extract their data using straightforward, lightweight HTTP requests. They don't need to render JavaScript or interact with the page. For instance, [CustomArticleCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/custom_article.py:11:0-53:65) uses `AsyncHtmlLoader` (which relies on `aiohttp` or `requests`) to just fetch the raw HTML and `Html2TextTransformer` to parse it. GitHub might even use an API under the hood depending on your implementation.
*   **Dynamic, JS-Rendered ([BaseSeleniumCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:23:0-65:31))**: `LinkedInCrawler` and `MediumCrawler` require a full, headless Google Chrome browser (via Selenium WebDriver) to extract data. This is because LinkedIn and Medium are typical Single Page Applications (SPAs) that heavily rely on client-side JavaScript to load content asynchronously after the initial page load. A simple `requests.get()` would likely just return a skeleton HTML file without the actual article content or profile data.

### 2. Simulating User Behavior
If you look at [BaseSeleniumCrawler](cci:2://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:23:0-65:31) in [base.py](cci:7://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:0:0-0:0), it includes methods like [scroll_page()](cci:1://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:54:4-65:31) and [login()](cci:1://file:///home/bhsham/LLM-Engineers-Handbook/llm_engineering/application/crawlers/base.py:51:4-52:12). 
*   **Medium and LinkedIn** often require you to scroll down the page dynamically to trigger "lazy loading" of comments, full articles, or subsequent posts. 
*   **Anti-Bot Protections**: Platforms like LinkedIn are notoriously difficult to scrape and have strong anti-bot protections. Using a real browser instance (with options like `--disable-blink-features=AutomationControlled` natively or patched via undetected-chromedriver) is often the only reliable way to bypass these checks.

### 3. Performance and Resource Implications
*   **Speed**: Github and Custom Article crawlers will be **significantly faster**. They just send a request, get text back, and process it.
*   **Computing Power**: The `LinkedInCrawler` and `MediumCrawler` will be much heavier and slower. Every time they run, they must spin up a Chrome process in memory, download all page assets (images, CSS, heavy JS bundles), execute the React/Angular/Vue scripts, and potentially wait (`time.sleep()`) for elements to appear in the DOM.

In summary, this inheritance structure nicely abstract away the complexity: it tells you that the system was designed to handle both "easy" traditional web scraping (BaseCrawler) and "hard" modern web scraping that mimics human browser interaction (BaseSeleniumCrawler), delegating the heavy browser dependency only to the platforms that actually require it.