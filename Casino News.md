
<img width="1003" height="677" alt="image" src="https://github.com/user-attachments/assets/65a0cc18-b8e6-45c9-8267-9d63857a8856" />

# Aristocrat, Light & Wonder, and Financial News Monitoring Report Workflow

This workflow collects financial news from multiple RSS feeds, cleans and filters the data, removes duplicate articles, ranks the most relevant items, and produces a structured financial market report.

**AI interprets the collected information and supports informed decision-making.**



## Overview

This workflow automatically collects and processes financial news from multiple trusted RSS sources to build a structured market-monitoring pipeline.

It is designed to gather news related to:

* Aristocrat Leisure
* Light & Wonder
* ASX 200
* Australian equities
* Global financial markets
* Interest rates and central-bank policy
* Inflation and macroeconomic developments
* Selected financial and market themes

The workflow cleans and refines the collected data by:

* Standardising RSS fields
* Removing duplicate articles
* Filtering low-value news
* Preventing previously processed news from being repeated
* Ranking the most relevant stories
* Limiting the final number of articles
* Analysing the selected news using AI
* Delivering the final report through email or Discord

The final output is a concise and prioritised financial report that supports market monitoring and business decision-making.

---

## How It Works

The workflow begins with a **Schedule Trigger**, which automatically runs the process at a selected time or interval.

Five RSS feeds collect financial news related to:

* Light & Wonder
* Aristocrat Leisure
* ASX 200
* Yahoo Finance
* Global financial markets

The five RSS feeds are combined into one dataset using the **Merge** node.

The workflow then:

1. Standardises the RSS fields
2. Removes duplicates within the current execution
3. Filters articles using selected financial keywords
4. Removes articles processed in previous executions
5. Calculates an importance score
6. Sorts articles by importance
7. Limits the number of selected articles
8. Combines the selected articles into one dataset
9. Uses OpenAI to analyse and summarise the news
10. Converts the report into HTML
11. Sends the full report through Microsoft Outlook

---

## Workflow Structure

```text
Schedule Trigger
   ├── RSS Read: Light & Wonder
   ├── RSS Read: Aristocrat
   ├── RSS Read: ASX 200
   ├── RSS Read: Yahoo Finance
   └── RSS Read: Global Finance
                ↓
           Merge — Append
                ↓
            Edit Fields
                ↓
 Remove Duplicates — Current Input
                ↓
          Filter Relevant News
                ↓
Remove Duplicates — Previous Executions
                ↓
      Calculate Importance Score
                ↓
              Sort
                ↓
              Limit
                ↓
            Aggregate
                ↓
 OpenAI — Analyse and Summarise
                ↓
      Markdown to HTML
                ↓
 Microsoft Outlook — Send Email
```

---

# 1. Schedule Trigger

! 

The **Schedule Trigger** starts the workflow automatically.

## Example Configuration

For regular market monitoring:

```text
Mode: Every X Minutes
Minutes: 30
```

For a daily financial report:

```text
Mode: Every Day
Time: 07:00 AM
Timezone: Australia/Sydney
```

## Purpose

The Schedule Trigger removes the need to run the workflow manually.

It ensures that all five RSS sources are collected during the same workflow execution.

---

# 2. RSS Data Collection Layer


This section collects financial news from five RSS sources.

Each RSS source uses a separate **RSS Read** node.

## Light & Wonder RSS Feed

```text
https://feeds.finance.yahoo.com/rss/2.0/headline?s=LNW.AX&region=AU&lang=en-AU
```

## Aristocrat Leisure RSS Feed

```text
https://feeds.finance.yahoo.com/rss/2.0/headline?s=ALL.AX&region=AU&lang=en-AU
```

## ASX 200 RSS Feed

```text
https://feeds.finance.yahoo.com/rss/2.0/headline?s=%5EAXJO&region=AU&lang=en-AU
```

## Yahoo Finance RSS Feed

```text
https://finance.yahoo.com/news/rssindex
```

## Global Financial News RSS Feed

```text
https://www.investing.com/rss/news.rss
```

## Purpose

The RSS layer collects diverse financial information from Australian and global sources.

This provides broader coverage than relying on only one news provider.

---

# 3. Merge Layer


The **Merge** node combines the output of all five RSS Read nodes.

## Recommended Setting

```text
Mode: Append
```

The Append mode adds all RSS articles into one continuous dataset.

## Example

```text
Light & Wonder: 20 items
Aristocrat: 20 items
ASX 200: 20 items
Yahoo Finance: 42 items
Global Finance: 10 items
```

Combined result:

```text
Total: 112 items
```

## Purpose

The Merge node creates one unified stream of financial news for cleaning, filtering, and analysis.

---

# 4. Edit Fields and Data Normalisation


The **Edit Fields** node standardises the data structure from different RSS sources.

Different RSS feeds may use different field names.

For example:

```text
title
headline
link
url
pubDate
isoDate
creator
author
content
contentSnippet
```

The Edit Fields node converts these variations into one standard structure.

## Recommended Fields

### Title

Field name:

```text
title
```

Value:

```javascript
{{ $json.title || $json.headline || "" }}
```

### Link

Field name:

```text
link
```

Value:

```javascript
{{ $json.link || $json.url || "" }}
```

### Publication Date

Field name:

```text
pubDate
```

Value:

```javascript
{{ $json.isoDate || $json.pubDate || $json.date || "" }}
```

### Description

Field name:

```text
description
```

Value:

```javascript
{{ $json.contentSnippet || $json.description || $json.content || "" }}
```

### GUID

Field name:

```text
guid
```

Value:

```javascript
{{ $json.guid || $json.id || $json.link || "" }}
```

### Source

Field name:

```text
source
```

Value:

```javascript
{{ $json.creator || $json.author || $json.source || "RSS Feed" }}
```

## Final Standard Structure

```json
{
  "title": "Article title",
  "link": "https://example.com/article",
  "pubDate": "2026-06-18T05:25:32.000Z",
  "description": "Article description",
  "guid": "unique-article-id",
  "source": "Reuters"
}
```

## Purpose

The normalisation step ensures that all later nodes use the same field names.

This improves reliability and reduces errors.

---

# 5. Remove Duplicates Within the Current Input

 

The first **Remove Duplicates** node removes repeated articles found during the same workflow execution.

## Recommended Settings

```text
Operation: Remove Items Repeated Within Current Input
Compare: Selected Fields
Field: link
```

You may also compare:

```text
guid
```

## Preferred Priority

```text
guid
→ link
→ title
```

## Purpose

The same article may appear in several RSS feeds.

This node prevents duplicate articles from continuing through the workflow.

---

# 6. Financial News Filtering Layer

 

The **Filter** node keeps only news related to selected financial topics.

## Value 1

```javascript
{{ [
  $json.title,
  $json.source,
  $json.description
].filter(Boolean).join(' ').toLowerCase() }}
```

## Operator

```text
String → Matches Regex
```

## Value 2

```regex
\b(all\.ax|aristocrat|lnw\.ax|light\s*(?:&|and)?\s*wonder|asx|australian shares?|rba|reserve bank|federal reserve|fed|interest rates?|cash rate|inflation|cpi|stock market|shares?|earnings|dividend|oil|bitcoin|nasdaq|s&p 500)\b
```

## Topics Included

* Aristocrat
* ALL.AX
* Light & Wonder
* LNW.AX
* ASX
* Australian shares
* Reserve Bank of Australia
* Federal Reserve
* Interest rates
* Cash rate
* Inflation
* CPI
* Stock market
* Earnings
* Dividends
* Oil
* Bitcoin
* Nasdaq
* S&P 500

## Purpose

The filter reduces noise and removes articles that are not relevant to the monitoring strategy.

For example, an unrelated technology or lifestyle article will be discarded unless it matches one of the selected financial topics.

---

# 7. Remove Duplicates from Previous Executions

 
The second **Remove Duplicates** node prevents articles from being processed repeatedly across different workflow runs.

## Recommended Settings

```text
Operation: Remove Items Processed in Previous Executions
Keep Items Where: Value Is New
```

## Value to Dedupe On

```javascript
{{ $json.guid || $json.link || $json.title }}
```

## Purpose

The first duplicate node removes duplicates within the current workflow run.

The second duplicate node remembers previously processed articles and prevents them from appearing again in future reports.

## Testing Note

During testing, the node may remove all items because it has already processed them.

To reset the history temporarily:

```text
Operation: Clear Deduplication History
```

Execute the node once, then change it back to:

```text
Remove Items Processed in Previous Executions
```

---

# 8. Importance Score Calculation

 

A **Code** node calculates an importance score for each article.

The score helps determine which articles should appear at the top of the final report.

 

## Purpose

The importance score prioritises articles based on their relevance to selected companies, markets, and macroeconomic topics.

---

# 9. Sort Layer

 

The **Sort** node arranges articles from the highest importance score to the lowest.

## Recommended Settings

```text
Type: Simple
Field Name: importanceScore
Order: Descending
```

## Example

Before sorting:

```text
Article A: 4
Article B: 9
Article C: 6
```

After sorting:

```text
Article B: 9
Article C: 6
Article A: 4
```

## Purpose

The most important financial articles appear first.

---

# 10. Limit Layer

 

The **Limit** node controls the maximum number of articles used in the final report.

## Recommended Settings

```text
Max Items: 10
Keep: First Items
```

You may use:

```text
10 items
15 items
20 items
```

depending on the desired report length.

## Important Note

The Limit node does not create additional items.

For example:

```text
Input: 4 items
Limit: 10
Output: 4 items
```

```text
Input: 25 items
Limit: 10
Output: 10 items
```

## Purpose

The Limit node prevents the AI prompt and final report from becoming unnecessarily long.

---

# 11. Aggregate Layer

 

The **Aggregate** node combines multiple news items into one item before sending them to OpenAI.

## Recommended Setting

```text
Aggregate: Individual Fields
```

## Fields to Aggregate

```text
title
link
pubDate
description
source
guid
importanceScore
```

## Example Output

```json
{
  "title": [
    "Article 1",
    "Article 2"
  ],
  "link": [
    "https://example.com/article-1",
    "https://example.com/article-2"
  ],
  "description": [
    "Description 1",
    "Description 2"
  ],
  "pubDate": [
    "2026-06-18",
    "2026-06-17"
  ]
}
```

## Purpose

The Aggregate node converts multiple news items into one structured dataset.

This allows OpenAI to create one combined financial report instead of generating a separate report for every article.

---

# 12. OpenAI Analysis Layer

 

The **OpenAI — Message a Model** node analyses the aggregated news and produces the financial report.

## Recommended Action

```text
OpenAI
→ Text
→ Message a Model
```

## Recommended Model

Use a cost-efficient model suitable for summarisation and financial analysis.

Example:

```text
GPT-5 Mini
```

## Recommended Prompt

The Prompt field should be set to **Expression mode**.

```javascript
{{ `
You are a professional financial-market analyst.

Analyse the following financial news and produce one concise daily market report.

Use these sections:

1. Executive Summary
2. Global Market Developments
3. Australian Market Developments
4. Interest Rates, Inflation and Central Banks
5. Aristocrat and Light & Wonder Updates
6. Important Company and Sector News
7. Key Risks
8. What to Watch Next
9. Sources

Requirements:

- Use only the supplied financial-news data.
- Prioritise the highest-importance articles.
- Combine related stories.
- Avoid repeating the same information.
- Separate confirmed facts from interpretation.
- Explain likely market impact cautiously.
- Do not invent facts.
- Do not provide personalised financial advice.
- Include relevant source links.
- Return only the completed report.

Financial news data:

${JSON.stringify($json)}
` }}
```

## Purpose

OpenAI interprets the selected articles and transforms them into a readable and structured financial market report.

---

# 13. Markdown to HTML Conversion


The OpenAI report may contain Markdown headings, bullet points, and lists.

The **Markdown** node converts the AI report into HTML for email delivery.

## Recommended Settings

```text
Mode: Markdown to HTML
```

## Markdown Input

```javascript
{{ $json.output[0].content[0].text }}
```

## Destination Key

```text
data
```

## Example Output

```json
{
  "data": "<h1>Daily Market Report</h1><p>...</p>"
}
```

## Purpose

The HTML format improves the appearance and readability of the report in Microsoft Outlook.

---

# 14. Microsoft Outlook Delivery Layer


The **Microsoft Outlook** node sends the full financial report by email.

## Recommended Action

```text
Resource: Message
Operation: Send
```

## To

```text
your-email@example.com
```

## Subject

```javascript
Daily Financial Market Report - {{ $now.setZone("Australia/Sydney").toFormat("dd LLL yyyy") }}
```

## Message

```javascript
{{ $json.data }}
```

## Body Type

```text
HTML
```

## Purpose

Email is recommended for full financial reports because it provides:

* Complete report delivery
* Proper formatting
* Searchable history
* Easy forwarding
* No short message-length limit
* Better readability on mobile and desktop

 
 

# Node Summary

| Step | Node              | Purpose                                 |
| ---- | ----------------- | --------------------------------------- |
| 1    | Schedule Trigger  | Starts the workflow automatically       |
| 2    | RSS Read          | Collects financial-news articles        |
| 3    | Merge             | Combines all RSS sources                |
| 4    | Edit Fields       | Standardises field names                |
| 5    | Remove Duplicates | Removes duplicates in the current input |
| 6    | Filter            | Keeps relevant financial news           |
| 7    | Remove Duplicates | Removes articles processed previously   |
| 8    | Code              | Calculates importance scores            |
| 9    | Sort              | Orders articles by importance           |
| 10   | Limit             | Keeps the top articles                  |
| 11   | Aggregate         | Combines articles into one dataset      |
| 12   | OpenAI            | Analyses and summarises the news        |
| 13   | Markdown          | Converts the report to HTML             |
| 14   | Microsoft Outlook | Sends the full report by email          |
| 15   | Discord           | Sends optional alerts or notifications  |

---

# Key Benefits

* Automated financial-news collection
* Multi-source RSS monitoring
* Focus on Aristocrat and Light & Wonder
* Australian and global market coverage
* Duplicate-article prevention
* Topic-based filtering
* Importance-based ranking
* AI-powered analysis
* Full HTML email delivery
* Optional Discord notifications
* Extendable workflow architecture

---

# Security Considerations

* Keep OpenAI API keys private
* Keep Discord webhook URLs private
* Use Microsoft OAuth credentials
* Avoid placing credentials directly in Code nodes
* Restrict access to the n8n workspace
* Use strong account passwords
* Enable multi-factor authentication
* Review workflow execution logs regularly

---

# Possible Extensions

This workflow can be extended with:

* PDF report generation
* SharePoint report storage
* OneDrive file storage
* Google Sheets news logging
* PostgreSQL news-history storage
* Power BI integration
* Stock-price API integration
* Sentiment analysis
* Email attachment analysis
* Company-specific risk monitoring
* Automatic buy-and-sell signal research

---

# Example Final Report Structure

```text
Daily Financial Market Report

1. Executive Summary

2. Global Market Developments

3. Australian Market Developments

4. Interest Rates, Inflation and Central Banks

5. Aristocrat and Light & Wonder Updates

6. Important Company and Sector News

7. Key Risks

8. What to Watch Next

9. Sources
```

---

# Conclusion

This n8n workflow provides a structured and scalable system for monitoring financial news related to Aristocrat Leisure, Light & Wonder, the ASX, and global financial markets.

It combines automated RSS collection, data cleaning, duplicate control, relevance filtering, importance scoring, AI-powered analysis, and email delivery.

The workflow reduces the time required to manually review multiple financial-news sources and produces a concise report that can support market monitoring and informed decision-making.
