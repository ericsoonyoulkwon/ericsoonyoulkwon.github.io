# Selecting data transacted on previous business day

#### Category: SQL

---

### CASE and WHEN in WHERE clause

When there is a report you have to run on a daily basis for picking up the data transacted one previous business day, what would you do?

### What I was not happy with the current SOP?

According to the service level agreement with our clients, they send us data every business day. The current SOP for the daily report said I need to update the hard-coded date in the WHERE clause before I run the SQL script. I am sure that I frowned as soon as I saw a “manual” step during the process. I asked myself a question whether the parameter in WHERE statement should really be static.

### Date of transaction data = Report date – 1 business day

It is always true that the report is run on one business day after the date the records are received. In other words, I always want to pick up data from one business day ago. It sounds like I can just use SYSDATE – 1 to grab what I want. (SYSDATE is a function to get today’s date in Oracle SQL, just like TODAY function in Excel. Depending on the engine, it could be GETDATE() if you are using MS SQL or MySQL.) However, there always are exceptions and I believe these were the reasons why they have kept updating hard-coded dates in the script every day.

To classify the exceptional cases, I asked myself when I didn’t run the report daily.

- Obviously weekend: SYSDATE – 3 should replace -1 to capture the data received on Friday evening should be included in the report created on Monday.
- Statutory Holidays: I don’t log in to run the report. Clients also don’t send us any data on the holidays. The holidays usually fall on either Friday or Monday where I live.
  - If Monday is a holiday and I run the report on Tuesday, the report should be able to capture the data received on Friday. Pretending it is Tuesday now, I need to capture the data for last Friday (SYSDATE – 4) if yesterday (SYSDATE – 1) was Monday.
  - If last Friday (SYSDATE – 3) was a holiday and I ran the report on Monday, I need to capture the data received on the previous Thursday (SYSDATE – 3).
- When I am out of the office such as on vacation: I don’t need to worry about this case as I have a coworker who plays the same role I do and we cover each other when one of us is away. I can ask him to run a macro that I built.

To handle scenarios, there are two options in Oracle SQL: IF vs CASE. They serve the same logic. Although I found IF useful in many cases such as nesting conditions (if inside if) and compatibility with other scripts because CASE is used in Oracle SQL only, I prefer to use CASE for the sake of readability.

### Let’s put these cases into SQL script

Let’s handle the weekend case first. TO_CHAR(SYSDATE-1, ‘d’) tells what day it was yesterday. In Oracle SQL, a week starts with Sunday which is 1, incrementing one for each day, and finally 7 for Saturday. So, I am picking up the data transacted on SYSDATE – 3 When it was Sunday yesterday or I run the report on Monday. When I run the report between Tuesday and Friday, it is a standard SYSDATE – 1 case. I am not supposed to prepare the report during the weekend so SYSDATE – 1 cannot be 6 or 7.

```SQL
CASE 
    WHEN TO_CHAR(SYSDATE-1, 'd') = 1 
        THEN TO_CHAR(SYSDATE-3, 'YYYY-MM-DD')
    WHEN TO_CHAR(SYSDATE-1, 'd') IN (2,3,4,5) 
        THEN TO_CHAR(SYSDATE-1, 'YYYY-MM-DD')
END
```

Then I need to handle the holidays. I’ve considered creating a temporary table for holidays and referencing it for the exceptions but I guess there are not many benefits compared to the time I need to spend on the fancy feature. So, I compromised to keep the list of holidays in the script and maintain it on a yearly basis. I combined the two holiday-cases of either Friday or Monday with the OR statement because they both result in capturing the date 4 days ago.

```SQL
CASE
    WHEN TO_CHAR(SYSDATE-1, 'YYYY-MM-DD') 
        IN ('2023-02-20', '2023-05-22', '2023-07-03', '2023-08-07', '2023-09-04',
            '2023-10-09', '2023-12-25')
        OR TO_CHAR(SYSDATE-3, 'YYYY-MM-DD') IN ('2023-04-07') 
    THEN TO_CHAR(SYSDATE-4, 'YYYY-MM-DD')  
END
```

### Final WHERE clause

Combining all cases together, below is what the final WHERE clause looks like. So, it, depending on the scenario, takes a desirable SYSDATE – x as a parameter for the date of the record received that exists in the other subquery.

```SQL
WHERE TO_CHAR(SUBQUERY_THAT_INCLUDES_RECORD_DATE.DATE_COLUMN, 'YYYY-MM-DD') =
CASE
    WHEN TO_CHAR(SYSDATE-1, 'YYYY-MM-DD') 
        IN ('2023-02-20', '2023-05-22', '2023-07-03', '2023-08-07', '2023-09-04',
            '2023-10-09', '2023-12-25')
        OR TO_CHAR(SYSDATE-3, 'YYYY-MM-DD') IN ('2023-04-07') 
    THEN TO_CHAR(SYSDATE-4, 'YYYY-MM-DD')  
    WHEN TO_CHAR(SYSDATE-1, 'd') = 1 
        THEN TO_CHAR(SYSDATE-3, 'YYYY-MM-DD')
    WHEN TO_CHAR(SYSDATE-1, 'd') IN (2,3,4,5) 
        THEN TO_CHAR(SYSDATE-1, 'YYYY-MM-DD')
END
```

#### So, I won’t need to hardcode the date that I am querying every day when it is defined in the query itself what date it should look for. This doesn’t really save my time, I would say 2 minutes per day. However, without eliminating the manual hardcoding of the date in the process, [automating the daily task]([https://ericsoonyoulkwon.github.io/2023/05/21/automating-daily-report-with-VBA.html](https://ericsoonyoulkwon.github.io/2023/05/21/automating-daily-report-with-VBA.html)) wouldn’t have been achieved.
