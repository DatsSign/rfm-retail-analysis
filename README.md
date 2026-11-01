# rfm-retail-analysis


## A convenient way to categorize clients for future interactions

This script calculates three key metrics for each customer:
* **Recency (R):** Days since the last order (Fewer days = higher score).
* **Frequency (F):** Total number of orders (More orders = higher score).
* **Monetary (M):** Total revenue generated (More revenue = higher score).

Scores range from 1 to 5, where 5 is the best possible behavior. 

## Key Segments (Examples):
* **555 (Champions / VIP):** Bought recently, buy often, and spend the most. 
* **155 (At-Risk High Spenders):** Great past customers who haven't purchased in a while. (Needs a win-back email/promo).
* **511 (Newcomers):** Bought very recently, but only once. (Needs a welcome campaign).

``` sql
with flatt as (

	select
		s."CustomerKey",
		s."Order Number",
		s."Order Date",
		(s."Quantity" * p."Unit Price USD" ) as rev
	from sales s
	left join products p
		on s."ProductKey" = p."ProductKey" 
),

	rfm_absolute as (
		-- Calculate raw stats
		select
			"CustomerKey",
			max(max("Order Date")) over () - max("Order Date") as date_diff,
			count(distinct "Order Number") as ord_quant,
			sum(rev) as rev
		from flatt
		group by "CustomerKey"
), 
	last_step as (
		-- Graduate the raw stats
		select 
			"CustomerKey",
			NTILE(5) over (order by date_diff DESC) as r,
			NTILE(5) over (order by ord_quant) as f,
			NTILE(5) over (order by rev) as m
		from rfm_absolute
)

select
	"CustomerKey",
	concat(r, '' , f , '' , m) as "R|F|M"
from last_step
order by 2 desc

```
