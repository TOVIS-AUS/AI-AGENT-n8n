# AI-AGENT-n8n
The AI agent collects and analyses financial news from the United States, Europe, Asia, and Australia to produce a comprehensive market report.



<img width="1516" height="362" alt="Image" src="https://github.com/user-attachments/assets/055fc195-afd5-4517-bedc-409d23cc6214" />

### Overview

This workflow automatically generates a daily financial market briefing using multiple trusted news sources. It delivers concise, actionable insights by combining up-to-date financial information with AI-powered analysis.

**How It Works**

The workflow aggregates financial news from the Yahoo Finance, CoinDesk, and Federal Reserve RSS feeds into a unified dataset. It removes duplicate and low-value articles, retaining only content relevant to key macroeconomic and market topics, including inflation, cryptocurrency, interest rates, and financial markets.

The 15 most relevant news items are then organised and analysed using Google Gemini. The AI generates a clear and actionable market summary, which is formatted and delivered through Discord.

If the AI service fails, the workflow automatically sends a fallback error notification.

 



**Data Collection and Merge Layer**

<img width="294" height="299" alt="Image" src="https://github.com/user-attachments/assets/22ebd1e0-4752-465e-a097-cc31b3e33ee8" />

This section collects financial news and economic updates from multiple trusted sources:

Yahoo Finance — general financial market news
Federal Reserve — monetary policy updates and macroeconomic indicators

Goal: To collect diverse, reliable, and high-quality financial information for further analysis.





**Data Cleaning and Filtering Layer**

<img width="398" height="119" alt="Image" src="https://github.com/user-attachments/assets/758b2096-1c62-460b-9e0a-375627db8daf" />
 
const items = $input.all();
const seen = new Set();

function cleanUrl(value) {
  if (!value) return "";

  try {
    const url = new URL(value);
    url.search = "";
    url.hash = "";

    return url
      .toString()
      .replace(/\/$/, "")
      .toLowerCase();
  } catch {
    return String(value)
      .trim()
      .toLowerCase();
  }
}

function cleanTitle(value) {
  return String(value ?? "")
    .trim()
    .toLowerCase()
    .replace(/\s+/g, " ");
}

return items.filter(item => {
  const guid = String(item.json.guid ?? "")
    .trim()
    .toLowerCase();

  const link = cleanUrl(item.json.link);
  const title = cleanTitle(item.json.title);

  const keys = [
    guid ? `guid:${guid}` : "",
    link ? `link:${link}` : "",
    title ? `title:${title}` : ""
  ].filter(Boolean);

  if (keys.length === 0) {
    return false;
  }

  const isDuplicate = keys.some(key => seen.has(key));

  if (isDuplicate) {
    return false;
  }

  keys.forEach(key => seen.add(key));
  return true;
});


This section improves data quality by:

Removing duplicate news articles published within the past 24 hours
Retaining only articles related to key financial topics, including cryptocurrency, inflation, financial markets, Federal Reserve policy, and oil prices
Filtering out irrelevant, repetitive, and low-value content

Goal: To retain only high-value financial information for further analysis.





**Scoring and Sorting the most important news(Scoring from 0 -5)**  5 is the highest

<img width="384" height="89" alt="Image" src="https://github.com/user-attachments/assets/20ff98d6-5be1-4573-adc5-fe21bfffe7a0" />
 

const items = $input.all();

return items.map(item => {
  const text = `
    ${item.json.title ?? ""}
    ${item.json.contentSnippet ?? ""}
    ${item.json.content ?? ""}
  `.toLowerCase();

  let score = 0;

  if (text.includes("rba")) score += 5;
  if (text.includes("federal reserve") || text.includes("fed")) score += 5;
  if (text.includes("interest rate")) score += 4;
  if (text.includes("inflation")) score += 4;
  if (text.includes("asx")) score += 3;
  if (text.includes("s&p 500")) score += 3;
  if (text.includes("nasdaq")) score += 3;
  if (text.includes("earnings")) score += 2;
  if (text.includes("oil")) score += 2;
  if (text.includes("bitcoin")) score += 2;

  item.json.importanceScore = score;

  return item;
})

 




**Limit 15 news and Aggregate** 
The goal is getting the most relevant and valuable news

<img width="379" height="111" alt="Image" src="https://github.com/user-attachments/assets/7fbad37f-c655-461a-8267-92e0d6a7d21e" />






**Interpreting the collected information and interpretation that will be a crucial for a decision making.** 
<img width="174" height="83" alt="Image" src="https://github.com/user-attachments/assets/d6aa3091-4ee6-4586-a9bf-b18a1d018712" />
