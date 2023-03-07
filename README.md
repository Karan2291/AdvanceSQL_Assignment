# AdvanceSQL_Assignment
This is my AdvanceSQL Assignment

Below I have explain the approach and shown the output screenshot to my solution:

## Question 1:
#### Write a query that gives an overview of how many films have replacements costs in the following cost ranges
* #### low: 9.99 - 19.99
* #### medium: 20.00 - 24.99
* #### high: 25.00 - 29.99


### Solution:-

```
select distinct
SUM(CASE WHEN f.replacement_cost >=9.99 AND f.replacement_cost<=19.99 THEN 1 ELSE 0 END) over() as low,
SUM(CASE WHEN f.replacement_cost >=20.00 AND f.replacement_cost<=24.99 THEN 1 ELSE 0 END) over() as medium,
SUM(CASE WHEN f.replacement_cost >=25.00 AND f.replacement_cost<=29.99 THEN 1 ELSE 0 END)over()  as high
FROM film f;
```
__Approach:-__
* Here in this particular question we have been asked to give the overview of the films where the replacement_cost is  in the given range categorizing it     into __low,medium and high__
* So I have used window function where I have used CASE statement for filtering the data according to the given range of replacement cost and i categorize   those filtered data for each case statement as low,medium and high
* Logic in the case statement is if the __particular replacement_cost__ is within the __specified range__ for the __particular category__ then returning     __1__ else __0__ which we are summing up using ```SUM()``` function which results in __total number of films__ in that particular range


### OUTPUT:
<img width="683" alt="Screenshot 2023-03-07 at 6 16 50 PM" src="https://user-images.githubusercontent.com/122456892/223426227-ae064981-61b7-4177-9e51-12aadac39a53.png">


## Question 2:
#### Write a query to create a list of the film titles including their film title, film length and film category name ordered descendingly by the film length. Filter the results to only the movies in the category 'Drama' or 'Sports'.
* #### "STAR OPERATION" "Sports"  181
* #### "JACKET FRISCO"  "Drama"   181

### Solution:-

```
SELECT f.title AS film_title,f.length,c.name
FROM film f,film_category f_c,category c
WHERE f.film_id=f_c.film_id
AND f_c.category_id=c.category_id
AND (c.name='Drama' or c.name='Sports')
ORDER BY f.length DESC;
```

__Approach:-__
* Here we have to create a list of the film titles including their __film title__, __film length__ and __film category__ name ordered descendingly by the     film length for the category ```“sports”``` and ```“drama”```
* So for that I have joined three tables i.e., ```film, film_category, category``` and extracted the __film_title__, __film_length__ and __film_category__   by applying the filter for __category name__ to be only ```“Sports”``` and ```“Drama”``` ordering by the __length__ of the film in descending order.

### OUTPUT:
<img width="683" alt="Screenshot 2023-03-07 at 6 27 26 PM" src="https://user-images.githubusercontent.com/122456892/223428740-5dec848a-91b2-4cb6-9411-02aada30bd6e.png">

#### Total Rows Fetched: 136
<img width="1120" alt="Screenshot 2023-03-07 at 5 55 13 PM" src="https://user-images.githubusercontent.com/122456892/223431963-171423dd-4f6c-4f42-b175-599cb3903441.png">

## Question 3:
#### Write a query to create a list of the addresses that are not associated to any customer.
### Solution:-

```
SELECT a.address_id,a.address,a.district,a.city_id,c.customer_id
FROM address a LEFT JOIN  customer c  -- Using left join here to retrieve all the data from address table irrespective
ON a.address_id=c.address_id          -- whether its associated to any customer
WHERE c.customer_id IS NULL;
```
__Approach:-__
* We have been asked to create a list of the addresses that are not associated to any customer
* So For that I have used __two tables__ i.e., ```address``` and ```customer``` to extract __address_id__, __address__, __district__, __city_id__ and   __customer_id__
* I have used __left join__ above to retrieve all the data from the address table __irrespective of whether it is present in the customer table__. So whichever customer_id is not associated with any address will result in __null values__ which i am using in the where clause to filter out the data where __customer_id is null__ which will give me all the address which are not related to any customer.

### OUTPUT:
<img width="683" alt="Screenshot 2023-03-07 at 6 31 41 PM" src="https://user-images.githubusercontent.com/122456892/223432988-0cf63d0e-4c6b-4c5f-8736-edc44d328924.png">

## Question 4:
#### Write a query to create a list of the revenue (sum of amount) grouped by a column in the format "country, city" ordered in decreasing amount of revenue.
* #### eg. 	"Poland, Bydgoszcz"		52.88

### Solution:-

```
SELECT DISTINCT CONCAT(c.country,', ',ct.city) AS country_city_name,
SUM(p.amount) OVER(PARTITION BY c.country, ct.city) AS revenue -- finding sum using window function based on country and city
FROM country c,city ct,address ad,customer cu,payment p        -- Joining total 5 tables to get the desired result
WHERE c.country_id=ct.country_id
AND ad.city_id=ct.city_id
AND cu.address_id=ad.address_id
AND p.customer_id=cu.customer_id
ORDER BY revenue DESC;                                         -- Ordering by revenue in descending order

```
__Approach:-__
* Here we need to create a list of the revenue based on __country__, __city__ according to decreasing amount of revenue
* So for getting the desired result I have joined 4 tables i.e., ```country```, ```city```, ```address``` and ```payment``` to fetch __country name__,       __city name__ which I am __concatenating__ using ```CONCAT()``` function
* Also calculated the __revenue__ by using window function which is calculating the sum of amount partitioning by ```Country```,```City``` and then           ordering revenue by __descending__ order

### OUTPUT:
<img width="683" alt="Screenshot 2023-03-07 at 6 52 46 PM" src="https://user-images.githubusercontent.com/122456892/223434650-b9d00f38-1e9c-4efe-8e62-5a52c4127d4c.png">

#### Total Rows Fetched: 597
<img width="1120" alt="Screenshot 2023-03-07 at 5 57 45 PM" src="https://user-images.githubusercontent.com/122456892/223434904-674827c0-b488-4233-a478-83fe953d7ea4.png">

## Question 5:
#### Write a query to create a list with the average of the sales amount each staff_id has per customer.
* #### result: 	2	56.64
* ####	        1	55.91

### Solution:-
```
SELECT DISTINCT d.staff_id,
ROUND(AVG(d.SUM_by_staff_customer) OVER(PARTITION BY d.staff_id),2) AS Avg_sales_each_staff_per_cust
FROM
(
SELECT DISTINCT p.staff_id,p.customer_id,
SUM(p.amount) OVER(PARTITION BY p.staff_id,p.customer_id) AS SUM_by_staff_customer
FROM payment p
) AS d                                         -- d is the alias for this derived table
ORDER BY Avg_sales_each_staff_per_cust DESC;
```
__Approach:-__
* Here we are asked to create a list with the average of sales per __customer__ with each __staff id__
* Here each __customer has multiple transaction with particular staff id__, so first __we need to find total amount of transaction each customer had with   particular staff id__, so I am calculating that In the __derived table__ ```(d)``` above using window function which is calculating the sum of amount     partitioning by __staff_id__,__customer_id__
* Now using __derived table__ I am extracting __staff_id__ and calculating __average__ of the __total_amount__ per customer calculated in derived table to   find out the average amount of sales each staff had
* Now using __derived table__ I am extracting __staff_id__ and calculating __average__ of the __total_amount__ per customer calculated in derived table to   find out the average amount of sales each staff had


### OUTPUT:
<img width="683" alt="Screenshot 2023-03-07 at 6 55 53 PM" src="https://user-images.githubusercontent.com/122456892/223436116-206e6416-de39-4655-b068-7d8f3ad0ebf1.png">

#### Total Rows Fetched: 2

## Question 6:
####  Write a query that shows average daily revenue of all Sundays.

### Solution:-
```
SELECT ROUND(AVG(d2.tot_amt),2) AS avg_daily_of_all_sunday
FROM
(
SELECT d1.date1,SUM(d1.amount) AS tot_amt
	FROM
		(
		SELECT DATE(p.payment_date) AS date1,p.amount 
		FROM payment p
		WHERE DAYNAME(p.payment_date) = 'Sunday'
		) AS d1                                       -- d1 is the alias for inner derived table
GROUP BY date1
) AS d2;                                          -- d2 is the alias for outer derived table
```
__Approach:-__
* We have been asked to show average daily revenue of all Sundays.
* So for that I have used single payment table wherein in the __inner derived table__  I am extracting the __payment_date__ and __amount__ by filtering     the data where the __date is of sunday__ by using ```DAYNAME()``` function in where clause
* In each sunday their can be __multiple transaction__ so in order to find out the __average__, first we need to calculate the __total of all the           transaction in every sunday__.So in __outer derived table__ I am calculating the sum of amount grouping by the date ```(obtain from inner derived table   by filtering out on the basis of sunday)``` which will give me the __total transaction in all sundays.__
* Now that I have got __total amount__ ,so I can find the __average__ of that __total amount__ which will give me the desired result that is average daily   revenue of all sundays.



### OUTPUT:
<img width="696" alt="Screenshot 2023-03-07 at 7 05 04 PM" src="https://user-images.githubusercontent.com/122456892/223437633-5659c95e-b4f8-40ea-944e-0fe0822f11cb.png">


#### Total Rows Fetched: 1

## Question 7:
####  Write a query to create a list that shows how much the average customer spent in total (customer life-time value) grouped by the different districts.

### Solution:-
```
SELECT DISTINCT d.district,Round(AVG(d.total_sum) OVER(PARTITION BY d.district),2) as average_amount
FROM
(
SELECT DISTINCT c.customer_id,a.district,
SUM(p.amount) OVER(PARTITION BY c.customer_id) AS total_sum -- using the window function to calculate sum partition by customer_id
FROM customer c,address a,payment p
WHERE c.address_id = a.address_id
AND c.customer_id=p.customer_id
) AS d;
```
__Approach:-__
* Here we need to find out the __average amount__ of all customer spent in total based on __each district__
* For that I have used __three tables__ i.e., ```customer```, ```address``` and ```payment``` and extracted __customer_id__, __district__ and the __total   amount__ for each customer as each customer have __multiple transaction in each district__
* Now making above as __derived table__ I extracted __district__ and __Average of total_sum calculated for each customer in the derived table__ partitioning by the __district__.So this way I am getting the __district and the average of customers total spent grouped by each district__




### OUTPUT:
<img width="696" alt="Screenshot 2023-03-07 at 7 09 27 PM" src="https://user-images.githubusercontent.com/122456892/223439103-0f333003-eb9c-41a6-9ae2-1a9a681bc39e.png">


#### Total Rows Fetched: 376
<img width="1120" alt="Screenshot 2023-03-07 at 5 59 43 PM" src="https://user-images.githubusercontent.com/122456892/223439238-4f37d267-af6d-4738-8d9c-09a072dde298.png">


## Question 8:
####  Write a query to list down the highest overall revenue collected (sum of amount per title) by a film in each category. Result should display the film title, category name and total revenue.
* #### eg. 	"FOOL MOCKINGBIRD"		  "Action"	    175.77
* ####    	"DOGMA FAMILY"			    "Animation"	  178.7
* ####    	"BACKLASH UNDEFEATED"	  "Children"	  158.81

### Solution:-

```
SELECT d2.title,d2.name,d2.Max_revenue
FROM
(
SELECT DISTINCT d1.title,d1.name,d1.overall_revenue_by_title,
MAX(d1.overall_revenue_by_title) OVER(PARTITION BY d1.name) AS Max_revenue
FROM
	(
		SELECT DISTINCT f.title,c.name,
		SUM(p.amount) OVER(PARTITION BY f.title) AS overall_revenue_by_title 	-- using window function to find overall revenue per title
		FROM film f,film_category f_c,category c,inventory i,rental r,payment p
		WHERE f.film_id=f_c.film_id
		AND f_c.category_id = c.category_id
		AND f.film_id=i.film_id
		AND i.inventory_id=r.inventory_id
		AND r.rental_id = p.rental_id
	) AS d1                                  -- d1 is an alias for inner derived table
) AS d2                                      -- d2 is an alias for outer derived table
WHERE overall_revenue_by_title=Max_revenue;  -- filtering the data where overall revenue is equal to max_revenue
```


__Approach:-__
* Here we need to list out the __film_title__ and __highest overall revenue collected__ ```sum of amount per title``` by a film for __each category__
  For this I have used __6 tables__ i.e., ```film```, ```film_category``` ,```category```, ```inventory```, ```rental```, ```payment``` and from that I     have extracted __film_title__, __film category__ and __sum of amount partition by film_title__, so this will give me the __overall revenue__ for each     film title
* Now from __inner derived table__ I have the data of __film_title__, __film category__ and __overall revenue generated from each film__.So in __outer       derived table__ I extracted the __film_title__, __category__ and __Maximum of overall revenue__ for each category by using window function for finding     __MAX of overall_revenue partition by category__.
* Now from __outer derived table__ I extracted the __film_title__,__category name__ and the __max revenue__ where my ``` max_revenue= overall_revenue```.   This way I am getting those __film title__ who has generated __highest revenue__ in each category





### OUTPUT:
<img width="696" alt="Screenshot 2023-03-07 at 7 17 54 PM" src="https://user-images.githubusercontent.com/122456892/223441242-35e4153c-93de-46fc-80d1-795613d7f659.png">


#### Total Rows Fetched: 16

## Question 9:
####  Modify the table "rental" to be partitioned using PARTITION command based on ‘rental_date’ in below intervals:

```
	<2005
        between 2005–2010
	between 2011–2015
	between 2016–2020
	>2020 - Partitions are created yearly
```


### Solution:-

```
ALTER TABLE rental 
PARTITION BY RANGE(YEAR(rental_date))
(
PARTITION rental_less_than_2005 VALUES LESS THAN (2005),
PARTITION rental_between_2005_2010 VALUES LESS THAN (2011),
PARTITION rental_between_2011_2015 VALUES LESS THAN (2016),
PARTITION rental_between_2016_2020 VALUES LESS THAN (2021),
PARTITION rental_greater_than_2020 VALUES LESS THAN MAXVALUE
);
```


__Approach:-__
* Here In this particular question first I have to __drop the foreign keys__ as __partitioning was not supporting the foreign key constraints in MYSQL__ as it can change the underlying structure of the table
* Next I __partitioned__ the rental table by __rental_date__ with the specified __range of years__.
* __First Partition__ ```“rental_less_than_2005”``` will contain those values where __year is less than 2005__
* __Second Partition__ ```“rental_between_2005_2010”``` will contain those values where __year is between 2005 and 2010__
* __Third Partition__  ```“rental_between_2011_2015”``` will contain those values where __year is between 2011 and 2015__
* __Fourth Partition__  ```“rental_between_2016_2020”``` will contain those values where __year is between 2016 and 2020__
* And __everything after 2020__ will be stored in the __partition__ ```“rental_greater_than_2020”```




## Question 10:
####  Modify the table "film" to be partitioned using PARTITION command based on ‘rating’ from below list. Further apply hash sub-partitioning based on ‘film_id’ into 4 sub-partitions.

* #### partition_1 - "R"
* #### partition_2 - "PG-13", "PG"
* #### partition_3 - "G", "NC-17"



### Solution:-

```
ALTER TABLE film
PARTITION BY LIST(rating)
SUBPARTITION BY HASH(film_id) SUBPARTITIONS 4
(
PARTITION PR values('R'),
PARTITION Pgs values('PG-13', 'PG'),
PARTITION GNC values('G', 'NC-17')
);
```


__Approach:-__
* Here first I have used ```PARTITION BY LIST``` based on __rating__
* Then I have used ```SUBPARTITION BY HASH``` based on __film_id__


## Question 11:
####  Write a query to count the total number of addresses from the “address” table where the ‘postal_code’ is of the below formats. Use regular expression.

* #### ```9*1**, 9*2**, 9*3**, 9*4**, 9*5**```


### Solution:-

```
SELECT count(postal_code) 
FROM address
WHERE postal_code REGEXP '^9[0-9][1-5][0-9]{2}';  -- Using regular expression to retrieve all the postal code with the givrn pattern
```


__Approach:-__
* Here I have used the __address table__ to extract __postal_code__ of the specific pattern by using a __regular expression__ in the where clause to         filter out the data
* Next I used the __Aggregate function__ ```COUNT``` to count the number of __postal code__ following the specific pattern




### OUTPUT:
<img width="696" alt="Screenshot 2023-03-07 at 7 27 28 PM" src="https://user-images.githubusercontent.com/122456892/223443561-38d0b68f-564d-40c1-8e93-5f1d8d9f4e72.png">

#### Total Rows Fetched: 1

## Question 12:
####  Write a query to create a materialized view from the “payment” table where ‘amount’ is between(inclusive) $5 to $8. The view should manually refresh on demand. Also write a query to manually refresh the created materialized view.


### Solution:-

```
DELIMITER $$
CREATE EVENT refresh_payment_between_5_8      -- Creating the event which refresh the view every day.
ON SCHEDULE EVERY 1 DAY
DO
BEGIN
  CREATE OR REPLACE VIEW payment_between_5_8 AS
  SELECT *
  FROM payment
  WHERE amount BETWEEN 5 AND 8;
END$$
DELIMITER ;

-- APPLYING SELECT CLAUSE IN VIEW CREATED ABOVE WHICH CONTAINS THE DATA WHERE AMOUNT IS BEWTWEEN $5 AND $8
SELECT * FROM payment_between_5_8;
```


__Approach:-__
* As __materialized view__ was not supported by __MYSQL__ so I have created the event name ```“refresh_payment_between_5_8”``` which is scheduled to         __refresh in every one day__ and inside that I have created the __normal view which stores all the data of the payment table where amount is between $5   and $ 8__


### OUTPUT:
<img width="696" alt="Screenshot 2023-03-07 at 7 30 26 PM" src="https://user-images.githubusercontent.com/122456892/223444631-f0801d87-b738-4f3a-8271-c0e42682f25e.png">

#### Total Rows Fetched: 3100
<img width="1120" alt="Screenshot 2023-03-07 at 6 03 03 PM" src="https://user-images.githubusercontent.com/122456892/223444534-faba9dd7-b4b0-4352-95b7-1cab94e2047f.png">


## Question 13:
####  Write a query to list down the total sales of each staff with each customer from the ‘payment’ table. In the same result, list down the total sales of each staff i.e. sum of sales from all customers for a particular staff. Use the ROLLUP command. Also use GROUPING command to indicate null values.



### Solution:-

```

SELECT p.staff_id,p.customer_id,
GROUPING(p.staff_id) as staff,
GROUPING(p.customer_id) as customer,sum(p.amount) as sum_of_sales
FROM payment p
GROUP BY p.staff_id,p.customer_id
WITH ROLLUP;
```


__Approach:-__
* Here we have been asked to list down all the __total sales of each staff with each customer__. Also we need list down the total sales of each staff i.e.   __sum of sales from all customers for a particular staff.__
* So for that I have used the __payment table__ to extract __staff_id__,__customer_id__ and applied __grouping__ on both __customer_id and staff_id__ to     handle __null values__ and clearly indicate the ```ROLLUP``` sum for the particular column



### OUTPUT:
<img width="696" alt="Screenshot 2023-03-07 at 7 34 21 PM" src="https://user-images.githubusercontent.com/122456892/223445293-d4ea29a7-e045-40a5-a7d9-a21994864e86.png">

#### Total Rows Fetched: 1201
<img width="1120" alt="Screenshot 2023-03-07 at 6 03 39 PM" src="https://user-images.githubusercontent.com/122456892/223445378-02862cf3-7da5-4634-b19a-c12829c134ee.png">


## Question 14:
####   Write a single query to display the customer_id, staff_id, payment_id, amount, amount on immediately previous payment_id, amount on immediately next payment_id ny_sales for the payments from customer_id ‘269’ to  staff_id ‘1’.


### Solution:-

```
select customer_id,payment_id,staff_id,
lead(amount) over(order by payment_id) next_payment,
lag(amount) over(order by payment_id) previous_amount,
lead(amount) over(Partition by customer_id,staff_id ORDER BY payment_id) as ny_sales,
lag(amount) over(Partition by customer_id,staff_id ORDER BY payment_id) as py_sales
from payment
where customer_id=269 and staff_id=1;
```


__Approach:-__
* Here we have been asked to display the __customer_id__, __staff_id__, __payment_id__, __amount__, __amount on immediately previous payment_id__, __amount on immediately next payment_id__ 
* For this we have used the __payment table__ and used the window function ```LEAD``` to __extract the next payment__ and ```LAG``` to __extract the previous payment order by payment_id__ 
* Next we need to find the __ny_sales__ -  __the amount of the next payment made by the same customer to the same staff member__ and __py sales__ - __the amount of the previous payment made by the same customer to the same staff member__.For this as well I have used ```lead``` and ```lag``` function partitioned by __customer_id and staff_id__
* Used where clause to filter out the data where ```customer_id=269``` and ```staff_id=1```




### OUTPUT:
<img width="696" alt="Screenshot 2023-03-07 at 7 39 22 PM" src="https://user-images.githubusercontent.com/122456892/223446478-9d7920bf-ac97-4828-9eaa-c1d6f30b67e5.png">
#### Total Rows Fetched: 15








