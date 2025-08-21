# TODO

> Edit this file to add your answers to each TODO item. It's okay to link to external documentation as well. But ensure that our reviewers have access.
> 
> When you're ready to submit, click **Commit changes...** and save your responses to a new branch titled 'implementation' and create a pull request.
>
> Before you start reading, please configure your environment as described in your GitHub repository's "Before you get started" README section.*

In this task, you will work with [this dataset about a fictional company called InsightEdge](https://console.cloud.google.com/bigquery?ws=!1m4!1m3!3m2!1salva-devskills!2sinsightedge_challange_dataset) to figure out some insights about the company's business.

Here's some useful information about this dataset:

- [Revenue Model](revenue-model.md)
- [Cost Model](cost-model.md)
- [Entity Relationship Diagram](erd.md)

Feel free to explore the dataset and try different queries using the BigQuery interface.

Once you have a query that answers the question, add the code snippet to the specific question and any supportive reasoning/comments you may have.

## Task 1

How much revenue was earned during March 2025?

> TODO: Your answer
select sum(amount) as Rev,
extract(Month FROM date) as Month,
extract(Year from date) as Year
 from  `alva-devskills.insightedge_challange_dataset.revenue` 
where extract(Year from date)= 2025
and extract(Month from date) = 3
group by Year,Month

;


## Task 2

How many customers generated revenue in January 2025?

> TODO: Your answer
select count(distinct customer_id) as No_of_Customers,
extract(Month FROM date) as Month,
extract(Year from date) as Year
 from  `alva-devskills.insightedge_challange_dataset.revenue` 
where extract(Year from date)= 2025
and extract(Month from date) = 1
group by Year,Month

;

## Task 3

Rank the customers by the total amount of revenue they have generated, and return the top 3 customer names (1 being highest).

> TODO: Your answer

with top3_customers AS(

select 
rev.customer_id,
sum(amount) as rev_tot

 from `alva-devskills.insightedge_challange_dataset.revenue` as rev
 group by rev.customer_id
),

rank_top3 as (
select*,
row_number()OVER( ORDER BY top3_customers.rev_tot DESC,top3_customers.customer_id) as rank3_column

from top3_customers
)

select cust.customer_id ,cust.name as customer_name, cast(rank_top3.rev_tot as Numeric) as tot_rev from rank_top3

join
`alva-devskills.insightedge_challange_dataset.customers` as cust
on
 cust.customer_id = rank_top3.customer_id
 where rank_top3.rank3_column <= 3
 order by rank_top3.rank3_column;

## Task 4

Which month’s leads had the highest customer conversion rate? What was the Month, Year, and what was the Conversion rate?

> TODO: Your answer

---
---
with leads as (
select 
extract(year from entered_sales_funnel_at) as year,
extract(month from entered_sales_funnel_at) as month,
count(*) as leads_tot

 from `alva-devskills.insightedge_challange_dataset.leads`

 group by year,month
),

conversion_leads as (

Select 
extract(year from entered_sales_funnel_at) as year,
extract(month from entered_sales_funnel_at) as month,
count(distinct leads.lead_id) as leads_conv

from `alva-devskills.insightedge_challange_dataset.leads` as leads
join `alva-devskills.insightedge_challange_dataset.leads_customers` leads_cust

on leads.lead_id = leads_cust.lead_id

group by year,month

)

select 
leads.year,
leads.month,
conversion_leads.leads_conv,
leads.leads_tot,
safe_divide(conversion_leads.leads_conv,leads.leads_tot) as Conv_rate

from leads

join conversion_leads  using(year,month)
order by Conv_rate desc

limit 3;

-- 

## Task 5

Considering your response to the previous question (Task 4), present arguments both in favor of and against qualifying it as a 'good insight'.

> TODO: Your answer
-- it is a good insight to create that kind of kpi's in a sales data but you would need more parameters to check the trends of those month's over time

## Task 6

Is there any additional data that you believe would enhance the depth of your analysis?

> TODO: Your answer
-- yes more dimension tables like regions, sector's etc and also cost per lead would be nice aswell
## Task 7

If you were to make a single recommendation to the company, what would it be?

> TODO: Your answer
---Whatever that created the revenue that has the highest conversion rate for the months past , monitor and use that strategy in every lead

---

#### Now assume that it's July 1, 2025 (2025-07-01).

## Task 9

The VP of Sales approaches you with a request: "Can you provide an estimate for the number of customers we can expect this month?" How do you reply?

> TODO: Your answer
with a as(

SELECT 
date_trunc(date(date), month) as month_first,
count(distinct customer_id) as No_of_cust

 FROM `alva-devskills.insightedge_challange_dataset.revenue`

group by 1
)

select  cast(round(avg(No_of_cust)) as INT64) as customers_forecast
from (
select a.No_of_cust
from a where month_first < DATE '2025-07-01'
order by month_first DESC
LIMIT 3
);
## Task 10

The CEO asks for an update on the performance of the 'AI Insight' product. How would you respond to this inquiry? (text/media input)

> TODO: Your answer
SELECT 
extract( year from date) as year,
extract( month from date) as month,
sum(amount) as rev_amount,
 level_1, level_2,  FROM `alva-devskills.insightedge_challange_dataset.revenue` 
 where level_2 = 'AI insight'

 group by year,month,level_1,level_2
 ;

 -- based on the data of this product its revenue is growing 30-50% each month since the start of this year but it did decline at the start of this year by also 50 %. 

## Task 11

The CEO asks you, "What is your assessment of our current performance?" How would you formulate your response?

> TODO: Your answer
-- based on our previous analysis and for this month the revenue is up by 10 %, the number of customers is down 10% since last month but up by over 200 % since the start of this year 
--and conversion rate is steady on 15-20%. So i would say we are doing good but need to do more analysis to continue this trend. 
