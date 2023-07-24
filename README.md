# business-analytics-with-sql
The given dataset of this project is about a website e-commerce website. A number of queries have been performed to examine some metrics such as revenue, margin, conversion rate and cross-sale selling to investigate the business performance. 

### 1. The increasing number of website sessions and orders in each quarter
Find the total sessions and total orders divided by time. 

MySQL code: 

SELECT 

Year(ws.created_at) as yr, quarter(ws.created_at) as qtr, 

count(distinct ws.website_session_id) as sessions,

count(distinct o.order_id) as orders

from website_sessions as ws

Left join orders as o

On ws.website_session_id=o.website_session_id

Group by 1,2;

Result: 

![image](https://github.com/ththao99/business-analytics-with-sql/assets/135719243/8ebf0758-a197-4406-9d27-1b7619a19f61)

### 2. Order conversion rate and revenue per each order and per each session 

MySQL code: 

SELECT 
Year(ws.created_at) as yr,
quarter(ws.created_at) as qtr,

count(distinct o.order_id)/count(distinct ws.website_session_id) as session_to_order_conv_rate,

sum(price_usd)/count(distinct o.order_id) as revenue_per_order,

SUM(price_usd)/count(distinct ws.website_session_id) as rev_per_session

from website_sessions as ws
 Left join orders as o
 
On ws.website_session_id=o.website_session_id

Group by 1,2;

Result: 
![image](https://github.com/ththao99/business-analytics-with-sql/assets/135719243/b8d3bb73-2836-41b4-afdb-2b762c04d7b5)

### 3. The number of orders by marketing campaigns

MySQL code: 
Create temporary table order_websession_detail

SELECT 
Year(ws.created_at) as yr,
Quarter(ws.created_at) as qtr,
ws.website_session_id,
o.order_id,
ws.utm_source,
ws.utm_campaign,
http_referer

FROM website_sessions as ws
Left join orders as o

On ws.website_session_id=o.website_session_id;

***

SELECT
yr, 
qtr,

count(distinct case when utm_source='gsearch' AND utm_campaign ='nonbrand' then order_id else null end) 
as gsearch_nonbrand_orders,

count(distinct case when utm_source='bsearch' AND utm_campaign='nonbrand' then order_id else null end) 
as bsearch_nonbrand_orders,

count(distinct case when utm_campaign ='brand' then order_id else null end) as brand_search_orders,

count(distinct case when utm_source is null and http_referer is not null then order_id else null end) 
as organic_type_in_orders,

count(distinct case when utm_source is null AND http_referer is null then order_id else null end) 
as direct_type_in_orders

From order_websession_detail

Group by 1,2;

Result: 

![image](https://github.com/ththao99/business-analytics-with-sql/assets/135719243/e44f51c4-ecd8-4d81-a62b-946432e58e20)

### 4. Order conversion rate by the marketing campaigns

MySQL code: 
SELECT
yr, 
qtr,

count(distinct case when utm_source='gsearch' AND utm_campaign ='nonbrand' then order_id else null end)/count(distinct case when utm_source='gsearch' AND utm_campaign ='nonbrand' then website_session_id else null end)  as gsearch_nonbrand_orders_cvr,

count(distinct case when utm_source='bsearch' AND utm_campaign='nonbrand' then order_id else null end)/count(distinct case when utm_source='bsearch' AND utm_campaign='nonbrand' then website_session_id else null end) as bsearch_nonbrand_orders_cvr,

count(distinct case when utm_campaign ='brand' then order_id else null end)/count(distinct case when utm_campaign ='brand' then website_session_id else null end) as brand_search_orders_cvr,

count(distinct case when utm_source is null and http_referer is not null then order_id else null end)/count(distinct case when utm_source is null and http_referer is not null then website_session_id else null end) as organic_type_in_orders_cvr,

count(distinct case when utm_source is null AND http_referer is null then order_id else null end)/count(distinct case when utm_source is null AND http_referer is null then website_session_id else null end) as direct_type_in_orders_cvr

From order_websession_detail

Group by 1,2;

Result: 

![image](https://github.com/ththao99/business-analytics-with-sql/assets/135719243/47cec98f-ff87-41df-83d0-21f774529d4d)

### 5. Revenue and profit divided by products

MySQL code: 

CREATE temporary table product_rev_margin
SELECT 
year(oi.created_at) as yr,
month(oi.created_at) as mo,
Product_name,
oi.product_id, 
price_usd,
cogs_usd

FROM order_items as oi
Left join products as p

On oi.product_id=p.product_id;

*** 

SELECT * from product_rev_margin;
SELECT
yr,mo,

SUM(case when product_name='The Original Mr. Fuzzy' then price_usd else null end) as mrfuzzy_rev,

SUM(case when product_name='The Original Mr. Fuzzy' then price_usd else null end)- SUM(case when product_name='The Original Mr. Fuzzy' then cogs_usd else null end) as mrfuzzy_marg,

SUM(case when product_name='The Forever Love Bear' then price_usd else null end) as lovebear_rev,

SUM(case when product_name='The Forever Love Bear' then price_usd else null end)-SUM(case when product_name='The Forever Love Bear' then cogs_usd else null end) as lovebear_marg,

SUM(case when product_name='The Birthday Sugar Panda' then price_usd else null end) as birthdaybear_rev,

SUM(case when product_name='The Birthday Sugar Panda' then price_usd else null end)-SUM(case when product_name='The Birthday Sugar Panda' then cogs_usd else null end) as birthdaybear_marg,

SUM(case when product_name='The Hudson River Mini bear' then price_usd else null end) as minibear_rev,

SUM(case when product_name='The Hudson River Mini bear' then price_usd else null end)-SUM(case when product_name='The Hudson River Mini bear' then cogs_usd else null end) as minibear_marg,

SUM(price_usd) as total_revenue,

SUM(price_usd)-SUM(cogs_usd) as total_margin

From product_rev_margin

Group by 1,2;

Result: 

![image](https://github.com/ththao99/business-analytics-with-sql/assets/135719243/8ec71c3d-3d41-4e1c-85ad-d387b11b2b72)

### 6. Click-through rate 
MySQL code: 

drop table if exists product_sessions;
create temporary table product_sessions

SELECT 
Year(wp.created_at) as yr,
month(wp.created_at) as mo,
ws.website_session_id,
wp.website_pageview_id as product_pageview_id, 
wp.pageview_url

FROM website_sessions as ws

Left join website_pageviews as wp
On ws.website_session_id=wp.website_session_id

Where wp.pageview_url='/products';

***

SELECT * from product_sessions;

drop table if exists next_pageview_ids;

CREATE temporary table next_pageview_ids

*** 

SELECT 
yr,
mo,
ps.website_session_id,
product_pageview_id,
Min(wp.website_pageview_id) as Next_pageview_id

FROM product_sessions as ps

Left join website_pageviews as wp

On ps.website_session_id=wp.website_session_id

AND product_pageview_id<wp.website_pageview_id

Group by yr,mo,ps.website_session_id,product_pageview_id;

***

SELECT * from next_pageview_ids;

create temporary table product_to_orders
SELECT yr, mo, 
npi.website_session_id, npi.product_pageview_id, npi.next_pageview_id,
o.order_id

FROM next_pageview_ids as 

Left join orders as o

On npi.website_session_id=o.website_session_id;

SELECT * from product_to_orders;

***

select yr, mo, 

count(distinct website_session_id) as session_to_product_page,

count(distinct next_pageview_id) as click_to_next,

count(distinct next_pageview_id)/count(distinct website_session_id) as clickthrough_rate,

count(distinct order_id) as orders,

count(distinct order_id)/count(distinct website_session_id) as products_to_orders_rate

FROM product_to_orders

GROUP by 1,2;

Result: 

![image](https://github.com/ththao99/business-analytics-with-sql/assets/135719243/0ee8e1b2-ed72-4880-8bb3-acc2706c8733)

### 7. The number of cross-sell products, and performance of cross-sell products 

MySQL code: 
drop table if exists xsell_orders;

create temporary table xsell_orders

SELECT 
o.order_id, product_id,

If(is_primary_item='1',oi.product_id,null) as primary_product_id,

If(is_primary_item='0',oi.product_id,null) as secondary_product_id

From orders as o

Left join order_items as oi

On o.order_id=oi.order_id

Where o.created_at>='2014-12-05';

***

drop table if exists final_xsell;
create temporary table final_xsell

SELECT order_id, Max(primary_product_id) as Primary_product_id, Max(secondary_product_id)as secondary_product_id

from xsell_orders

Group by order_id;

***

SELECT

primary_product_id,

count(distinct order_id) as total_xsold_orders, 

count(distinct case when secondary_product_id='1' then order_id else null end) as xsold_p1,

count(distinct case when secondary_product_id='2' then order_id else null end) as xsold_p2,

count(distinct case when secondary_product_id='3' then order_id else null end) as xsold_p3,

count(distinct case when secondary_product_id='4' then order_id else null end) as xsold_p4,

count(distinct case when secondary_product_id='1' then order_id else null end)/count(distinct order_id) as xsold_p1_rt,

count(distinct case when secondary_product_id='2' then order_id else null end)/count(distinct order_id) as xsold_p2_rt,

count(distinct case when secondary_product_id='3' then order_id else null end)/count(distinct order_id) as xsold_p3_rt,

count(distinct case when secondary_product_id='4' then order_id else null end)/count(distinct order_id) as xsold_p4_rt

From final_xsell

Group by primary_product_id; 

Result: 
![image](https://github.com/ththao99/business-analytics-with-sql/assets/135719243/aeefac5c-2f3e-434f-91e1-2a76cb4fdc23)

