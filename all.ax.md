
## ASX Trading Signal Analysis and Notification System

<img width="1507" height="295" alt="image" src="https://github.com/user-attachments/assets/a71fb7cf-4e16-4224-91db-8e9f2e603787" />



1)Set up Trigger

<img width="259" height="185" alt="image" src="https://github.com/user-attachments/assets/9cd40d6a-e9f6-4b39-a006-727a0c234658" />

// Setup a trigger at 09:00 AM Sydney time.


<img width="296" height="192" alt="image" src="https://github.com/user-attachments/assets/e6d9d2eb-0c49-47ca-871d-b3c89c67bb28" />


Parameters 
Conditions
{{ $now.weekday }} # is not equal to 6
AND
{{ $now.weekday }} # is not equal to 7 

//Check ASX trading day except Saturday(6) and Sunday(7)


If True // meet the conditions

 # ALL.AX AI Notification Workflow — Data Collection Nodes

This section explains the function and setup parameters for each data-collection node used in the ALL.AX AI notification workflow.

The workflow collects price data, market context, and financial news before passing the data to the AI analysis layer.

---

## Workflow Purpose

This workflow is designed to monitor **Aristocrat Leisure Limited (ALL.AX)** using:

* ALL.AX price history
* ASX 200 market context
* ASX-related news
* Australian financial news
* Global financial-market news

The collected data is later used to calculate technical indicators, analyse market conditions, and generate AI-assisted notifications.

---

## Data Collection Structure

```text
Schedule Trigger
        ↓
Check ASX Trading Day
        ↓
 ┌─────────────────────────────┬─────────────────────────────┬─────────────────────────────┐
 ↓                             ↓                             ↓
ASX Chart HTTP Request     ALL.AX HTTP Request        ASX200 Yahoo RSS Read
 ↓                             ↓                             ↓
Global RSS Read            ASX Investing RSS Read      Other optional news feeds
 └─────────────────────────────┴─────────────────────────────┴─────────────────────────────┘
                               ↓
                    Merge and Normalise Data
                               ↓
                    Calculate Technical Indicators
                               ↓
                         AI Analysis
                               ↓
                     Email Notification
```

---

# 1. ASX Chart HTTP Request

## Node Type

```text
HTTP Request
```

## Function

This node fetches historical price data for the **S&P/ASX 200 Index**.

The ASX 200 is used as the broader Australian market benchmark. It helps compare whether ALL.AX is moving because of company-specific factors or because the entire Australian market is moving.

ALL.AX may rise or fall because of:

* company-specific news
* gaming-sector sentiment
* general ASX market movement
* macroeconomic conditions
* global equity-market direction

By collecting ASX 200 data, the workflow can compare:

```text
ALL.AX movement
vs
ASX 200 movement
```

Example:

```text
ALL.AX falls 3%
ASX 200 falls 2.8%
```

This may suggest broad market weakness.

But:

```text
ALL.AX falls 3%
ASX 200 is flat
```

This may suggest company-specific weakness.

## Parameters

| Parameter             | Value                                                                                                                   |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Method                | `GET`                                                                                                                   |
| URL                   | `https://query1.finance.yahoo.com/v8/finance/chart/%5EAXJO?range=1y&interval=1d&includePrePost=false&events=div,splits` |
| Authentication        | `None`                                                                                                                  |
| Send Query Parameters | `OFF`                                                                                                                   |
| Send Headers          | `ON`                                                                                                                    |
| Header Name           | `User-Agent`                                                                                                            |
| Header Value          | `Mozilla/5.0`                                                                                                           |

## URL Explanation

```text
%5EAXJO
```

is the URL-encoded version of:

```text
^AXJO
```

`^AXJO` is the Yahoo Finance ticker for the S&P/ASX 200 Index.

## Expected Output

The node returns JSON data containing:

* timestamp
* open price
* high price
* low price
* close price
* volume
* market metadata

Example structure:

```json
{
  "chart": {
    "result": [
      {
        "meta": {
          "symbol": "^AXJO",
          "currency": "AUD",
          "regularMarketPrice": 8500.2
        },
        "timestamp": [],
        "indicators": {
          "quote": [
            {
              "open": [],
              "high": [],
              "low": [],
              "close": [],
              "volume": []
            }
          ]
        }
      }
    ]
  }
}
```

---

# 2. ALL.AX HTTP Request

## Node Type

```text
HTTP Request
```

## Function

This node fetches historical price data for **Aristocrat Leisure Limited (ALL.AX)**.

The data is used to calculate technical indicators such as:

* current price
* 20-day moving average
* 50-day moving average
* 200-day moving average
* daily return
* volume trend
* relative performance against the ASX 200


This is the core price-data node for the workflow.

The AI analysis should not rely only on news. It should also consider technical and price-based evidence.

This node allows the workflow to check whether ALL.AX is:

* above or below key moving averages
* trending upward or downward
* showing unusual volume
* outperforming or underperforming the broader ASX market

## Parameters

| Parameter             | Value                                                                                                                  |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Method                | `GET`                                                                                                                  |
| URL                   | `https://query1.finance.yahoo.com/v8/finance/chart/ALL.AX?range=1y&interval=1d&includePrePost=false&events=div,splits` |
| Authentication        | `None`                                                                                                                 |
| Send Query Parameters | `OFF`                                                                                                                  |
| Send Headers          | `ON`                                                                                                                   |
| Header Name           | `User-Agent`                                                                                                           |
| Header Value          | `Mozilla/5.0`                                                                                                          |

## URL Explanation

```text
ALL.AX
```

is the Yahoo Finance ticker for Aristocrat Leisure Limited on the Australian Securities Exchange.

## Expected Output

The node returns historical price and volume data.

The next Code node can extract:

```javascript
const result = $json.chart.result[0];

const timestamps = result.timestamp;
const quote = result.indicators.quote[0];

const closes = quote.close;
const volumes = quote.volume;
const currentPrice = result.meta.regularMarketPrice;
const previousClose = result.meta.previousClose;
```

## Data Used Later

This node provides the base data for:

* moving average calculation
* RSI calculation
* trend classification
* price-versus-market comparison
* AI signal generation

---

# 3. ASX200 Yahoo RSS Read

## Node Type

```text
RSS Read
```

## Function

This node fetches ASX 200-related financial news from Yahoo Finance.

It provides market-level news that may affect Australian equities generally.


ALL.AX does not move in isolation. Australian market sentiment can affect most ASX-listed shares.

This node helps the workflow understand the broader Australian market narrative.

Examples of useful news include:

* ASX 200 rises or falls
* Australian shares react to inflation data
* RBA expectations affect market sentiment
* resources or bank shares influence the index
* global risk sentiment affects the ASX

## Parameters

| Parameter | Value                                                                             |
| --------- | --------------------------------------------------------------------------------- |
| Node Type | `RSS Read`                                                                        |
| URL       | `https://feeds.finance.yahoo.com/rss/2.0/headline?s=%5EAXJO&region=AU&lang=en-AU` |
| Limit     | Optional, for example `20`                                                        |

## URL Explanation

```text
%5EAXJO
```

means:

```text
^AXJO
```

This is the Yahoo Finance symbol for the S&P/ASX 200 Index.

## Expected Output Fields

Typical RSS output fields include:

```text
title
link
pubDate
content
contentSnippet
guid
isoDate
```

## Example Output

```json
{
  "title": "Australian shares rise as investors assess RBA outlook",
  "link": "https://finance.yahoo.com/...",
  "pubDate": "2026-06-18T06:32:23.000Z",
  "contentSnippet": "Australian shares moved higher...",
  "guid": "article-id"
}
```

## Data Used Later

This node provides news context for:

* ASX market sentiment
* macroeconomic themes
* RBA-related news
* broad Australian market direction

---

# 4. Global RSS Read

## Node Type

```text
RSS Read
```

## Function

This node fetches global financial-market news.

It helps the workflow understand overseas market conditions that may influence Australian equities.

## Why This Node Is Needed

Australian shares are often influenced by global market movements, especially from:

* United States markets
* European markets
* global interest-rate expectations
* oil and commodity prices
* geopolitical risk
* technology-sector movement
* risk-on or risk-off sentiment

For ALL.AX, global context is useful because Aristocrat has significant international exposure, including gaming and digital segments.

## Parameters

| Parameter | Value                                    |
| --------- | ---------------------------------------- |
| Node Type | `RSS Read`                               |
| URL       | `https://www.investing.com/rss/news.rss` |
| Limit     | Optional, for example `20`               |

## Expected Output Fields

Typical RSS output fields include:

```text
title
link
pubDate
creator
content
contentSnippet
guid
isoDate
```

## Example Output

```json
{
  "title": "Global stocks ease as investors assess Fed rate outlook",
  "link": "https://www.investing.com/...",
  "pubDate": "2026-06-18T05:25:32.000Z",
  "creator": "Reuters",
  "contentSnippet": "Global stocks moved lower...",
  "guid": "article-id"
}
```

## Data Used Later

This node provides global context for:

* Federal Reserve policy
* US market movement
* global equity sentiment
* commodity prices
* risk sentiment
* macroeconomic conditions

---

# 5. ASX Investing RSS Read

## Node Type

```text
RSS Read
```

## Function

This node fetches Australian financial-market news from Investing.com Australia.

It complements the Yahoo ASX 200 RSS feed by adding another Australian market-news source.


Using more than one news source reduces the risk of missing important market stories.

This node may capture Australian market stories not included in the Yahoo Finance ASX 200 feed.

Examples of relevant topics include:

* Australian equities
* ASX market movement
* RBA commentary
* company earnings
* macroeconomic indicators
* commodity and currency impact on Australian shares

## Parameters

| Parameter | Value                                   |
| --------- | --------------------------------------- |
| Node Type | `RSS Read`                              |
| URL       | `https://au.investing.com/rss/news.rss` |
| Limit     | Optional, for example `20`              |

## Expected Output Fields

Typical RSS output fields include:

```text
title
link
pubDate
creator
content
contentSnippet
guid
isoDate
```

## Example Output

```json
{
  "title": "Australian shares fall as investors digest hawkish Fed comments",
  "link": "https://au.investing.com/...",
  "pubDate": "2026-06-18T05:25:32.000Z",
  "creator": "Reuters",
  "contentSnippet": "Australian shares declined...",
  "guid": "article-id"
}
```

## Data Used Later

This node supports:

* Australian market analysis
* ASX-related filtering
* macroeconomic news detection
* AI market interpretation

---

# Data Collection Layer Summary

The five nodes collect two main types of data.

## Market Price Data

| Node                   | Data Collected                   |
| ---------------------- | -------------------------------- |
| ASX Chart HTTP Request | ASX 200 price history            |
| ALL.AX HTTP Request    | Aristocrat Leisure price history |

## Market News Data

| Node                   | Data Collected                     |
| ---------------------- | ---------------------------------- |
| ASX200 Yahoo RSS Read  | ASX 200 and Australian market news |
| Global RSS Read        | Global financial-market news       |
| ASX Investing RSS Read | Australian financial-market news   |

---

# Parameter Summary Table

| Node                   | Type         | Method | URL                                                                                                                     | Authentication | Header                    |
| ---------------------- | ------------ | ------ | ----------------------------------------------------------------------------------------------------------------------- | -------------- | ------------------------- |
| ASX Chart HTTP Request | HTTP Request | GET    | `https://query1.finance.yahoo.com/v8/finance/chart/%5EAXJO?range=1y&interval=1d&includePrePost=false&events=div,splits` | None           | `User-Agent: Mozilla/5.0` |
| ALL.AX HTTP Request    | HTTP Request | GET    | `https://query1.finance.yahoo.com/v8/finance/chart/ALL.AX?range=1y&interval=1d&includePrePost=false&events=div,splits`  | None           | `User-Agent: Mozilla/5.0` |
| ASX200 Yahoo RSS Read  | RSS Read     | N/A    | `https://feeds.finance.yahoo.com/rss/2.0/headline?s=%5EAXJO&region=AU&lang=en-AU`                                       | None           | N/A                       |
| Global RSS Read        | RSS Read     | N/A    | `https://www.investing.com/rss/news.rss`                                                                                | None           | N/A                       |
| ASX Investing RSS Read | RSS Read     | N/A    | `https://au.investing.com/rss/news.rss`                                                                                 | None           | N/A                       |

---

# Notes

## Why Use HTTP Request for Price Data?

HTTP Request nodes are used because price-history data is returned as structured JSON.

This format is suitable for calculating:

* moving averages
* price returns
* volume ratios
* technical signals
* relative performance

## Why Use RSS Read for News?

RSS Read nodes are used because financial news is naturally distributed as feed items.

This format is suitable for collecting:

* headlines
* article links
* publication dates
* article summaries
* source information

## Why Separate Price Data and News Data?

Price data and news data have different structures.

Price data is numerical and time-series based.

News data is text-based and event-driven.

The workflow collects them separately first, then merges and normalises them later for AI interpretation.

---

# Next Step After Data Collection

After these data-collection nodes, the next recommended node is:

```text
Code Node — Merge and Normalise Data
```

This node combines:

* ALL.AX price data
* ASX 200 price data
* ASX-related news
* global market news
* Australian financial news

into one clean JSON object.

The cleaned object can then be passed to:

```text
Code Node — Calculate Technical Indicators
```

and then to:

```text
OpenAI — AI Analysis
```

---


