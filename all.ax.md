
<img width="1660" height="645" alt="image" src="https://github.com/user-attachments/assets/51d4bdf5-223b-44fe-a070-5036588652ce" />


 # ALL.AX AI Signal Notification Workflow

This n8n workflow monitors **Aristocrat Leisure Limited (ALL.AX)** using price data, ASX 200 market context, financial news feeds, technical indicators, and AI-assisted analysis.

The workflow collects market data, calculates technical indicators, asks OpenAI to analyse the result, and sends a final AI signal report to Gmail.

> This workflow is for research and monitoring only. It does not execute trades and does not provide personalised financial advice.

---

## Workflow Structure

```text
Schedule Trigger
        ↓
Check ASX Trading Day
        ↓
 ┌──────────────────────────────┬──────────────────────────────┬──────────────────────────────┐
 ↓                              ↓                              ↓
ASX Chart HTTP Request      ALL.AX HTTP Request         ASX200 Yahoo RSS Read
 ↓                              ↓                              ↓
Global RSS Read             ASX Investing RSS Read      Other optional feeds
 └──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
                                ↓
                            Merge
                                ↓
                  Code: Merge and Normalise Data
                                ↓
                    Code: Calculate Indicators
                                ↓
                    OpenAI: Message a Model
                                ↓
                    Code: Parse AI Response
                                ↓
                    Gmail: Send a Message
```

---

## Workflow Purpose

The workflow is designed to monitor **ALL.AX** by combining:

* ALL.AX price history
* ASX 200 market movement
* Australian financial news
* Global financial news
* Technical indicators
* AI-assisted signal interpretation

The final output is an email report containing:

* Final signal
* AI signal
* Confidence score
* News sentiment
* Key drivers
* Risk factors
* Rationale
* Disclaimer

---

# 1. Schedule Trigger

## Function

The **Schedule Trigger** starts the workflow automatically.

It is used to run the workflow after the ASX market opens so that the workflow can collect updated market data and news.

## Recommended Parameters

| Parameter | Value                      |
| --------- | -------------------------- |
| Node      | Schedule Trigger           |
| Mode      | Every Day or Every Weekday |
| Time      | 10:15 AM                   |
| Timezone  | Australia/Sydney           |

## Recommended Timing

```text
10:15 AM Australia/Sydney
```

This gives the ASX market time to open and produce early trading data.

---

# 2. Check ASX Trading Day

## Function

This **IF** node checks whether the workflow is running on a weekday.

It prevents the workflow from running on Saturday or Sunday.

## Node Type

```text
IF
```

## Conditions

### Condition 1

| Field    | Value                |
| -------- | -------------------- |
| Value 1  | `{{ $now.weekday }}` |
| Operator | `is not equal to`    |
| Value 2  | `6`                  |

### Condition 2

| Field    | Value                |
| -------- | -------------------- |
| Value 1  | `{{ $now.weekday }}` |
| Operator | `is not equal to`    |
| Value 2  | `7`                  |

## Logic

```text
1 = Monday
2 = Tuesday
3 = Wednesday
4 = Thursday
5 = Friday
6 = Saturday
7 = Sunday
```

The workflow continues only when:

```text
weekday is not Saturday
AND
weekday is not Sunday
```

## Note

This only checks weekends.

It does not check ASX public holidays. A more advanced version can add an ASX holiday calendar check.

---

# 3. ASX Chart HTTP Request

## Function

This node fetches historical price data for the **S&P/ASX 200 Index**.

The ASX 200 is used as the broader Australian market benchmark.

It helps compare:

```text
ALL.AX performance
vs
ASX 200 performance
```

## Node Type

```text
HTTP Request
```

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
%5EAXJO = ^AXJO
```

`^AXJO` is the Yahoo Finance symbol for the S&P/ASX 200 Index.

## Expected Output

The node returns:

```text
chart.result[0].meta
chart.result[0].timestamp
chart.result[0].indicators.quote[0].open
chart.result[0].indicators.quote[0].high
chart.result[0].indicators.quote[0].low
chart.result[0].indicators.quote[0].close
chart.result[0].indicators.quote[0].volume
```

---

# 4. ALL.AX HTTP Request

## Function

This node fetches historical price data for **Aristocrat Leisure Limited (ALL.AX)**.

The data is used to calculate:

* Current price
* Previous close
* One-day return
* Volume ratio
* 20-day moving average
* 50-day moving average
* 200-day moving average
* RSI 14
* Technical signal

## Node Type

```text
HTTP Request
```

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

## Expected Output

The node returns price and volume history for ALL.AX.

Expected ticker inside output:

```text
chart.result[0].meta.symbol = ALL.AX
```

---

# 5. ASX200 Yahoo RSS Read

## Function

This node fetches ASX 200-related financial news from Yahoo Finance.

It provides Australian market context.

## Node Type

```text
RSS Read
```

## Parameters

| Parameter | Value                                                                             |
| --------- | --------------------------------------------------------------------------------- |
| Feed URL  | `https://feeds.finance.yahoo.com/rss/2.0/headline?s=%5EAXJO&region=AU&lang=en-AU` |
| Limit     | Optional, for example `10` or `20`                                                |

## Expected Output Fields

```text
title
link
pubDate
content
contentSnippet
guid
isoDate
```

---

# 6. Global RSS Read

## Function

This node fetches global financial-market news.

It provides international market context, including:

* US market movement
* Federal Reserve news
* Inflation
* Commodities
* Global equity sentiment
* Risk-on or risk-off market conditions

## Node Type

```text
RSS Read
```

## Parameters

| Parameter | Value                                    |
| --------- | ---------------------------------------- |
| Feed URL  | `https://www.investing.com/rss/news.rss` |
| Limit     | Optional, for example `10` or `20`       |

## Expected Output Fields

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

---

# 7. ASX Investing RSS Read

## Function

This node fetches Australian financial-market news from Investing.com Australia.

It complements Yahoo Finance by adding another Australian market-news source.

## Node Type

```text
RSS Read
```

## Parameters

| Parameter | Value                                   |
| --------- | --------------------------------------- |
| Feed URL  | `https://au.investing.com/rss/news.rss` |
| Limit     | Optional, for example `10` or `20`      |

## Expected Output Fields

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

---

# 8. Merge

## Function

The **Merge** node combines the five data sources into one stream.

It combines:

* ASX 200 price data
* ALL.AX price data
* Yahoo ASX 200 news
* Global financial news
* Australian investing news

## Node Type

```text
Merge
```

## Parameters

| Parameter | Value    |
| --------- | -------- |
| Mode      | `Append` |

## Expected Output

Example:

```text
ASX Chart HTTP Request: 1 item
ALL.AX HTTP Request: 1 item
ASX200 Yahoo RSS Read: 10 items
Global RSS Read: 10 items
ASX Investing RSS Read: 10 items

Total after Merge: 32 items
```

---

# 9. Code Node — Merge and Normalise Data

## Function

This Code node separates price-chart data from RSS news data and creates one clean JSON object.

It identifies:

* ALL.AX chart data
* ASX 200 chart data
* RSS news articles

## Node Type

```text
Code
```

## Parameters

| Parameter | Value                    |
| --------- | ------------------------ |
| Mode      | `Run Once for All Items` |
| Language  | `JavaScript`             |

## Code

```javascript
const items = $input.all();

let allChart = null;
let asxChart = null;
const news = [];

for (const item of items) {
  const json = item.json;

  // Detect Yahoo Finance chart response
  if (json.chart?.result?.[0]) {
    const result = json.chart.result[0];
    const symbol = result.meta?.symbol;

    if (symbol === "ALL.AX") {
      allChart = result;
    }

    if (symbol === "^AXJO") {
      asxChart = result;
    }

    continue;
  }

  // Detect RSS news item
  if (json.title || json.link || json.pubDate || json.isoDate) {
    news.push({
      title: json.title || "",
      link: json.link || "",
      pubDate: json.pubDate || json.isoDate || "",
      description:
        json.contentSnippet ||
        json.content ||
        json.description ||
        "",
      source:
        json.creator ||
        json.author ||
        json.source ||
        "RSS Feed",
      guid:
        json.guid ||
        json.link ||
        json.title ||
        ""
    });
  }
}

if (!allChart) {
  throw new Error("ALL.AX chart data was not found.");
}

if (!asxChart) {
  throw new Error("ASX 200 chart data was not found.");
}

return [
  {
    json: {
      symbol: "ALL.AX",
      analysisTime: new Date().toISOString(),
      allChart,
      asxChart,
      news,
      newsCount: news.length
    }
  }
];
```

## Expected Output

```json
{
  "symbol": "ALL.AX",
  "analysisTime": "2026-07-02T06:37:56.901Z",
  "allChart": {},
  "asxChart": {},
  "news": [],
  "newsCount": 30
}
```

---

# 10. Code Node — Calculate Indicators

## Function

This node calculates technical indicators and market context.

It calculates:

* Current price
* Previous close
* One-day return
* ASX 200 one-day return
* Relative performance vs ASX 200
* SMA20
* SMA50
* SMA200
* RSI14
* Volume ratio
* Technical score
* Technical signal

## Node Type

```text
Code
```

## Parameters

| Parameter | Value                    |
| --------- | ------------------------ |
| Mode      | `Run Once for All Items` |
| Language  | `JavaScript`             |

## Code

```javascript
const item = $input.first().json;

const allChart = item.allChart;
const asxChart = item.asxChart;

function extractSeries(chart) {
  const timestamps = chart.timestamp || [];
  const quote = chart.indicators?.quote?.[0] || {};

  const closes = quote.close || [];
  const volumes = quote.volume || [];

  return timestamps
    .map((ts, i) => ({
      date: new Date(ts * 1000).toISOString().slice(0, 10),
      close: closes[i],
      volume: volumes[i],
    }))
    .filter(row => row.close !== null && row.close !== undefined);
}

function sma(values, period) {
  if (values.length < period) return null;

  const slice = values.slice(-period);
  const sum = slice.reduce((acc, value) => acc + value, 0);

  return Number((sum / period).toFixed(4));
}

function rsi(values, period = 14) {
  if (values.length <= period) return null;

  const recent = values.slice(-(period + 1));

  let gains = 0;
  let losses = 0;

  for (let i = 1; i < recent.length; i++) {
    const change = recent[i] - recent[i - 1];

    if (change > 0) {
      gains += change;
    } else {
      losses += Math.abs(change);
    }
  }

  const avgGain = gains / period;
  const avgLoss = losses / period;

  if (avgLoss === 0) return 100;

  const rs = avgGain / avgLoss;
  return Number((100 - 100 / (1 + rs)).toFixed(2));
}

function percentChange(current, previous) {
  if (!current || !previous) return null;

  return Number((((current - previous) / previous) * 100).toFixed(2));
}

const allSeries = extractSeries(allChart);
const asxSeries = extractSeries(asxChart);

const allCloses = allSeries.map(row => row.close);

const allLatestRow = allSeries.at(-1);
const allPreviousRow = allSeries.at(-2);

const asxLatestRow = asxSeries.at(-1);
const asxPreviousRow = asxSeries.at(-2);

const currentPrice =
  allLatestRow?.close ??
  allChart.meta?.regularMarketPrice ??
  null;

const previousClose =
  allPreviousRow?.close ??
  allChart.meta?.previousClose ??
  null;

const asxCurrent =
  asxLatestRow?.close ??
  asxChart.meta?.regularMarketPrice ??
  null;

const asxPreviousClose =
  asxPreviousRow?.close ??
  asxChart.meta?.previousClose ??
  null;

const sma20 = sma(allCloses, 20);
const sma50 = sma(allCloses, 50);
const sma200 = sma(allCloses, 200);
const rsi14 = rsi(allCloses, 14);

const allReturn1d = percentChange(currentPrice, previousClose);
const asxReturn1d = percentChange(asxCurrent, asxPreviousClose);

const relativePerformance1d =
  allReturn1d !== null && asxReturn1d !== null
    ? Number((allReturn1d - asxReturn1d).toFixed(2))
    : null;

const volumes = allSeries
  .map(row => row.volume)
  .filter(v => v !== null && v !== undefined);

const latestVolume = allLatestRow?.volume ?? null;

const avgVolume20 =
  volumes.length >= 20
    ? Number((volumes.slice(-20).reduce((a, b) => a + b, 0) / 20).toFixed(0))
    : null;

const volumeRatio =
  latestVolume && avgVolume20
    ? Number((latestVolume / avgVolume20).toFixed(2))
    : null;

let technicalScore = 0;

if (currentPrice && sma50 && currentPrice > sma50) technicalScore += 1;
if (currentPrice && sma200 && currentPrice > sma200) technicalScore += 1;
if (sma50 && sma200 && sma50 > sma200) technicalScore += 2;

if (rsi14 !== null && rsi14 >= 50 && rsi14 <= 70) technicalScore += 1;
if (rsi14 !== null && rsi14 > 75) technicalScore -= 1;

if (volumeRatio !== null && volumeRatio > 1.3) technicalScore += 1;

if (relativePerformance1d !== null && relativePerformance1d > 1) {
  technicalScore += 1;
}

if (relativePerformance1d !== null && relativePerformance1d < -1) {
  technicalScore -= 1;
}

let technicalSignal = "HOLD";

if (technicalScore >= 4) {
  technicalSignal = "BUY";
} else if (technicalScore <= -3) {
  technicalSignal = "SELL";
}

return [
  {
    json: {
      symbol: item.symbol,
      analysisTime: item.analysisTime,

      price: {
        currentPrice,
        previousClose,
        oneDayReturnPercent: allReturn1d,
        latestVolume,
        averageVolume20: avgVolume20,
        volumeRatio,
      },

      marketContext: {
        asxCurrent,
        asxPreviousClose,
        asxOneDayReturnPercent: asxReturn1d,
        relativePerformance1d,
      },

      indicators: {
        sma20,
        sma50,
        sma200,
        rsi14,
      },

      technicalSignal,
      technicalScore,

      news: item.news,
      newsCount: item.newsCount,
    },
  },
];
```

## Expected Output

```json
{
  "symbol": "ALL.AX",
  "price": {
    "currentPrice": 61.53,
    "previousClose": 59.98,
    "oneDayReturnPercent": 2.58,
    "volumeRatio": 0.99
  },
  "marketContext": {
    "asxOneDayReturnPercent": 0.02,
    "relativePerformance1d": 2.56
  },
  "indicators": {
    "sma20": 55.736,
    "sma50": 51.7488,
    "sma200": 55.0886,
    "rsi14": 74.49
  },
  "technicalSignal": "HOLD",
  "technicalScore": 3
}
```

---

# 11. OpenAI — Message a Model

## Function

This node sends the calculated indicators and news data to OpenAI.

The AI analyses the data and returns a structured signal response.

## Node Type

```text
OpenAI
```

## Parameters

| Parameter  | Value                                                       |
| ---------- | ----------------------------------------------------------- |
| Resource   | `Text`                                                      |
| Operation  | `Message a Model`                                           |
| Credential | `n8n free OpenAI API credits` or your OpenAI API credential |
| Model      | `gpt-5-mini` or supported mini model                        |
| Role       | `User`                                                      |
| Tools      | Leave empty                                                 |

## Prompt

Use **Expression mode** and paste:

```javascript
{{ `
You are a cautious financial-market analyst.

Analyse the following ALL.AX data and return a structured JSON response only.

Do not provide personal financial advice.
Do not invent facts.
Use only the supplied data.

Return JSON in this exact structure:

{
  "aiSignal": "BUY" | "HOLD" | "SELL",
  "confidence": 0-100,
  "newsSentiment": "POSITIVE" | "NEUTRAL" | "NEGATIVE",
  "keyDrivers": ["driver 1", "driver 2"],
  "riskFactors": ["risk 1", "risk 2"],
  "rationale": "short explanation"
}

Decision rules:
- Prefer HOLD if evidence is mixed.
- BUY requires positive technical evidence and no serious negative news.
- SELL requires clear technical weakness or serious negative news.
- Confidence above 70 should only be used when evidence is strong.

Input data:

${JSON.stringify($json)}
` }}
```

## Expected Output

The OpenAI node returns text containing JSON, for example:

```json
{
  "aiSignal": "HOLD",
  "confidence": 60,
  "newsSentiment": "NEUTRAL",
  "keyDrivers": [
    "Price is above key moving averages",
    "One-day return is stronger than ASX 200"
  ],
  "riskFactors": [
    "RSI is near overbought territory",
    "Volume confirmation is limited"
  ],
  "rationale": "The technical setup is positive, but confidence is limited due to overbought RSI and mixed news."
}
```

---

# 12. Code Node — Parse AI Response

## Function

This node parses the AI response and prepares an email-friendly HTML report.

It extracts:

* AI signal
* Confidence
* News sentiment
* Key drivers
* Risk factors
* Rationale
* Final signal
* Email body

## Node Type

```text
Code
```

## Parameters

| Parameter | Value                    |
| --------- | ------------------------ |
| Mode      | `Run Once for All Items` |
| Language  | `JavaScript`             |

## Code

```javascript
const item = $input.first().json;

// If parsed fields already exist, use them.
// If not, try to parse directly from OpenAI output.
let aiData = item.rawAiResponse;

if (!aiData) {
  const rawText = item.output?.[0]?.content?.[0]?.text;

  if (!rawText) {
    throw new Error("AI response text was not found.");
  }

  try {
    aiData = JSON.parse(rawText);
  } catch (error) {
    throw new Error("AI response was not valid JSON: " + rawText);
  }
}

const aiSignal = aiData.aiSignal || "UNKNOWN";
const confidence = aiData.confidence ?? "N/A";
const newsSentiment = aiData.newsSentiment || "N/A";
const keyDrivers = aiData.keyDrivers || [];
const riskFactors = aiData.riskFactors || [];
const rationale = aiData.rationale || "No rationale provided.";

let finalSignal = "HOLD";

if (aiSignal === "BUY" && confidence >= 70) {
  finalSignal = "BUY";
}

if (aiSignal === "SELL" && confidence >= 70) {
  finalSignal = "SELL";
}

const decisionReason =
  finalSignal === "HOLD"
    ? "Final signal is HOLD because the AI signal is HOLD or confidence is below 70."
    : `Final signal is ${finalSignal} because the AI signal is ${aiSignal} with confidence ${confidence}%.`;

function makeList(items) {
  if (!Array.isArray(items) || items.length === 0) {
    return "<li>No data provided.</li>";
  }

  return items.map(item => `<li>${item}</li>`).join("");
}

const emailBody = `
<h2>ALL.AX AI Signal Report</h2>

<h3>Final Signal</h3>
<p><strong style="font-size: 20px;">${finalSignal}</strong></p>

<h3>Summary</h3>
<ul>
  <li><strong>AI Signal:</strong> ${aiSignal}</li>
  <li><strong>Confidence:</strong> ${confidence}%</li>
  <li><strong>News Sentiment:</strong> ${newsSentiment}</li>
</ul>

<h3>Decision Reason</h3>
<p>${decisionReason}</p>

<h3>Key Drivers</h3>
<ul>
  ${makeList(keyDrivers)}
</ul>

<h3>Risk Factors</h3>
<ul>
  ${makeList(riskFactors)}
</ul>

<h3>Rationale</h3>
<p>${rationale}</p>

<hr>

<p><em>
This is an automated research signal generated from market data and AI-assisted analysis.
It is not personalised financial advice. No trade has been executed.
</em></p>
`;

return [
  {
    json: {
      finalSignal,
      aiSignal,
      confidence,
      newsSentiment,
      keyDrivers,
      riskFactors,
      rationale,
      decisionReason,
      rawAiResponse: aiData,
      emailBody
    }
  }
];
```

## Expected Output

```json
{
  "finalSignal": "HOLD",
  "aiSignal": "HOLD",
  "confidence": 60,
  "newsSentiment": "NEUTRAL",
  "keyDrivers": [],
  "riskFactors": [],
  "rationale": "",
  "emailBody": "<h2>ALL.AX AI Signal Report</h2>..."
}
```

---

# 13. Gmail — Send a Message

## Function

This node sends the final AI signal report by email.

## Node Type

```text
Gmail
```

## Parameters

| Parameter  | Value                    |
| ---------- | ------------------------ |
| Credential | `Gmail OAuth2 API`       |
| Resource   | `Message`                |
| Operation  | `Send`                   |
| To         | `your-email@example.com` |
| Email Type | `HTML`                   |

## Subject

```javascript
ALL.AX AI Signal - {{ $json.finalSignal }} - {{ $now.setZone("Australia/Sydney").toFormat("dd LLL yyyy") }}
```

## Message

```javascript
{{ $json.emailBody }}
```

## Expected Email Output

The email includes:

* Final signal
* AI signal
* Confidence score
* News sentiment
* Decision reason
* Key drivers
* Risk factors
* Rationale
* Disclaimer

---

# Final Workflow Summary

| Step | Node                     | Purpose                                |
| ---- | ------------------------ | -------------------------------------- |
| 1    | Schedule Trigger         | Runs the workflow automatically        |
| 2    | IF                       | Checks weekday trading condition       |
| 3    | ASX Chart HTTP Request   | Fetches ASX 200 price data             |
| 4    | ALL.AX HTTP Request      | Fetches ALL.AX price data              |
| 5    | ASX200 Yahoo RSS Read    | Fetches ASX-related news               |
| 6    | Global RSS Read          | Fetches global financial news          |
| 7    | ASX Investing RSS Read   | Fetches Australian financial news      |
| 8    | Merge                    | Combines all raw data                  |
| 9    | Merge and Normalise Data | Creates one clean data object          |
| 10   | Calculate Indicators     | Calculates technical indicators        |
| 11   | OpenAI Message a Model   | Generates AI signal analysis           |
| 12   | Parse AI Response        | Parses AI output and builds email body |
| 13   | Gmail Send Message       | Sends final email report               |

---


