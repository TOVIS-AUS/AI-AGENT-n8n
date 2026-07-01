
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

HTTP Request  

ALL.AX
20-day moving average
50-day moving average
200-day moving average
daily price movement
volume trend

Parrameters: 
Method: Get
https://query1.finance.yahoo.com/v8/finance/chart/ALL.AX?range=1y&interval=1d

Name: User-Agent
Value: Mozilla/5.0

