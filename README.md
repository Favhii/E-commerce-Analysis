# E-commerce-Analysis

## PREVIEW
- [PROJECT OVERVIEW](project-overview)
- [DATA SOURCE](data-sources)
- [TOOLS](tools)
- [DATA CLEANING](data-cleaning)
- [EXPLORATORY DATA ANALYSIS](exploratory-data-analysis)
- [INSIGHTS](insights)
- [RECOMMENDATION](recommendation)

## PROJECT OVERVIEW
The data contains a record of an e-commerce business. The goal is to understand the sales insights, factors that affect sales and products as well as how customers interact with and review products. 

## DATA SOURCE
The dataset used in this project was provided by Kaggle.

## TOOLS
1.	Excel: The dataset was gotten as a CSV, so I used Excel to open it. Go through the dataset to know the information it contains as well as cleaning.
2.	SQL: The dataset was loaded to SQL for cleaning and querying.
3.	Python: For sentimental analysis
4.	Power BI: This tool was used for visualization.
   
## DATA CLEANING
1.	Confirming the total normal of dataset and columns.
2.	Making sure they all represent their various data type.
3.	Checking for duplicates and null values.

## EXPLORATORY DATA ANALYSIS
~~~ SQL
rename table `product table` to product_table;
rename table `sales table` to sales_table;

-- renaming date table
alter table sales_table
rename column `date` to transcation_date;

-- date type conversion
select Transcation_date,
str_to_date(Transcation_date, '%m/%d/%Y') as Transcation_date
from sales_table;

update sales_table
set Transcation_date = str_to_date(Transcation_date, '%m/%d/%Y');

-- distinct customers
select count(distinct customer_id)
from sales_table;

-- avg quantity per product
select category, round(avg(quantity), 0) as average_quantity
from(
select p.category, s.quantity
from sales_table s
join product_table p
on s.product_id = p.product_id) as h
group by 1
order by 2 desc;

-- avg price per product 
select category, round(avg(price), 2) as average_price
from(
select p.category, s.price
from sales_table s
join product_table p
on s.product_id = p.product_id) as h
group by 1
order by 2 desc;

-- average Customer frequency as well as max, min
select round(avg(frequency), 2), max(frequency), min(frequency)
from(
select customer_id, count(distinct transcation_date) as frequency
from sales_table
group by 1) as h;

-- revenue
select round((quantity * price), 2) as Revenue
from sales_table;

alter table sales_table
add column Revenue int;

update sales_table
set Revenue = round((quantity * price), 2);

-- which category sells the most
select p.category, sum(s.revenue) as total_revenue
from product_table p
join sales_table s
on p.product_id = s.product_id
group by 1
order by 2 desc;

-- RFM
with rfm_base as
(
select 
customer_id,
-- max(transcation_date) as most_recently_purchase_date,
datediff('2025-10-01',max(transcation_date)) as recency,
count(distinct transcation_date) as frequency,
round(sum(price*quantity), 2) as monetary
from sales_table
group by 1
),

rfm_score as
(
select
customer_id, 
recency,
frequency,
monetary,
ntile(5) over (order by recency desc) as R,
ntile(5) over (order by frequency asc) as F,
ntile(5) over (order by monetary asc) as M
from rfm_base
),

rfm_final as
(
select customer_id, 
round((R+F+M)/3, 2) as RFM
-- cast(R as varchar)||
-- cast(F as varchar)||
-- cast(M as varchar) as RFM
from rfm_score
)

select customer_id, RFM,
case
	when RFM >=4.01 then 'Champions'
    when RFM >=3.01 then 'Loyal customers'
    when RFM >=2.01 then 'Potential Loyalists'
    when RFM >=1.51 then 'At Risk'
    else 'Lost customers'
end as Customer_segment
from rfm_final;

-- stock turn over ratio
with inventory as
(
select p.category, 
sum(s.quantity) as quantity_sold,
sum(s.quantity * s.price) as total_revenue,
sum(s.quantity)/2 as average_inventory
from sales_table s
join product_table p
on s.product_id = p.product_id
group by 1
),

stock_turnover_ratio as
(
select category, quantity_sold, total_revenue, average_inventory,
case 
	when average_inventory = 0 then NULL
    else round(total_revenue/average_inventory, 2)
end as stock_turnover_ratio
from inventory
)

select category,
case 
	when stock_turnover_ratio >= 1018 then 'Fast moving'
    when stock_turnover_ratio >= 1009.215 then 'Moderate moving'
    else 'Slow moving'
end as product_movement
from stock_turnover_ratio;

~~~

## INSIGHTS
### OVERVIEW INSIGHT
1.	There are 630 unique customers.
2.	 Total revenue is £8.40M.
3.	The average price per product is £516.43.
4.	The average quantity purchased per transaction is 26.
5.	The best-selling product category is Sports, generating £1.83M in revenue, followed by:
   - Home products: £1.70M
   - Clothing: £1.69M
   - Electronics: £1.67M
   - Books: £1.52M
6.	 Customer Reviews:
- 346 customers gave positive reviews.
- 147 customers gave negative reviews.
- 137 customers gave neutral reviews.
7.	Sales Velocity:
  - Sports products are the fastest-moving.
  - Books, Electronics, and Home products are moderately moving.
  - Clothing products are slow-moving.

### SALES INSIGHTS
2023:
1.	309 customers
2.	Total revenue: £3.93M
3.	Monthly growth: +6.70%
4.	Best-selling categories:
   - Clothing – £880,000
   - Electronics – £860,000
   - Home products – £840,000
   - Sports – £720,000
   - Books – £630,000

2024:
1.	209 customers
2.	Total revenue: £3.01M
3.	Monthly growth: -2.13%
4.	Best-selling categories:
   - Books – £670,000
   - Home products – £650,000
   - Sports – £630,000
   - Clothing – £540,000
   - Electronics – £520,000

2025 (January – October, Current year):
1.	112 customers 
2.	Total revenue: £1.46M
3.	Monthly growth: -11.07%
4.	Only profitable months: February, March, May, July
5.	Best-selling categories:
   - Sports – £470,000
   - Electronics – £280,000
   - Clothing – £270,000
   - Books – £220,000
   - Home products – £210,000

### CUSTOMER SEGMENT INSIGHTS
1.	169 Potential Loyalists – New customers with promising purchase patterns.
2.	166 Loyal Customers – Repeat buyers who shop frequently.
3.	116 Champions – High-value customers who spend the most.
4.	111 At-Risk Customers – Customers who haven’t purchased in a while.
5.	68 Lost Customers – Previously active customers who have stopped buying.

## RECOMMENDATION
1.	CUSTOMER RETENTION & ENGAGEMNET STRATEGIES
  - Offer targeted discounts and loyalty rewards to bring back lost customers.
  - Send personalized re-engagement emails and promotions to the 111 At-Risk Customers.
  - Introduce exclusive perks for Loyal Customers and Champions to encourage continued spending.
  - Offer VIP memberships with early access to sales and premium customer support to Loyal Customers and Champions
  - For Potential Loyalists, give first-time buyers a discount on their next purchase. Also implement a referral program to encourage them to bring in new customers.

2.	ADRESS DECLINING REVENUE TREND
  - Conduct a customer survey to understand why people stopped buying.
  - To improve Slow-Moving Categories (Clothing): Run flash sales and limited-time offers, Improve marketing efforts on social media for fashion-related audiences.
  - Focus on restocking and bundling best-sellers.
  - Use customer testimonials in marketing campaigns.
    
3.	IMPROVE PRODUCT & CATEGORY PERFORMANCE
  - Optimize Slow-Moving Clothing Sales by launching seasonal campaigns (e.g., summer/winter collections).
  - Offer "Buy One, Get One" (BOGO) deals to increase order volume.
  - Capitalize on Fast-Moving Sports Products by creating bundles with related products.
  - Optimize Home & Electronics Categories by promoting them as bundles.
  - Highlight warranty and durability in marketing to build trust.
    
4.	IMPROVE CUSTOMER SACTISFACTION & REVIEW
   - Address Negative Reviews (147 Negative, 137 Neutral) by analyzing negative feedback and fix recurring issues (product quality, delivery delays, etc.).
   - Respond to dissatisfied customers with support and refund/replacement options.
   - Encourage happy customers to leave reviews to boost overall ratings.



