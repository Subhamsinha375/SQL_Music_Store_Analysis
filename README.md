# SQL_Music_Store_Analysis

create database musicstore;
use musicstore;
select * from album;
select * from artist;
select * from customer;
select * from employee;
select * from genre;
select * from invoice;
select * from invoice_line;
select * from media_type;
select * from playlist;
select * from playlist_track;
select * from track;


/* Who is the senior most employee based on job title */

select * from employee order by levels desc limit 1;

/* Which countries have the most Invoices? */

select billing_country,count(*) from invoice group by billing_country order by count(*) desc limit 1;

/* What are top 3 values of total invoice? */

select total from invoice order by total desc limit 3;

/* Which city has the best customers? We would like to throw a promotional Music Festival in the city we made the most money. 
   Write a query that returns one city that has the highest sum of invoice totals. 
   Return both the city name & sum of all invoice totals */
   
select billing_city,sum(total) as total_amount from invoice group by billing_city order by total_amount desc limit 1;

/* Who is the best customer? The customer who has spent the most money will be declared the best customer. 
   Write a query that returns the person who has spent the most money.*/

select c.customer_id,sum(i.total) as total_amount
from customer as c 
inner join 
invoice as i 
on c.customer_id=i.customer_id
group by c.customer_id
order by total_amount desc
limit 1;

/* Write query to return the email, first name, last name, & Genre of all Rock Music listeners. 
   Return your list ordered alphabetically by email starting with A. */

select c.email,c.first_name,c.last_name,g.name 
from customer as c join invoice as i on c.customer_id=i.customer_id
join invoice_line as il on i.invoice_id= il.invoice_id
join track as t on il.track_id=t.track_id
join genre as g on t.genre_id=g.genre_id
where c.email like 'A%'and g.name='Rock';

/* Let's invite the artists who have written the most rock music in our dataset. 
Write a query that returns the Artist name and total track count of the top 10 rock bands. */

select ar.name,count(t.track_id) as track_count
from artist as ar join album as al on ar.artist_id=al.artist_id
join track as t on al.album_id=t.album_id
join genre as g on t.genre_id=g.genre_id
where g.name='Rock'
group by name
order by track_count desc;

/* Return all the track names that have a song length longer than the average song length. 
   Return the Name and Milliseconds for each track. Order by the song length with the longest songs listed first. */

select name, milliseconds from track where milliseconds>(select avg(milliseconds) from track) order by milliseconds desc;

/* Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent */

select c.first_name,ar.name as artist_name,sum(i.total) as total
from customer as c join invoice as i on c.customer_id=i.customer_id
join invoice_line as il on i.invoice_id=il.invoice_id
join track as t on il.track_id=t.track_id
join album as a on t.album_id=a.album_id
join artist as ar on a.artist_id=ar.artist_id
group by c.first_name,ar.name
order by total desc;


/* We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre 
   with the highest number of purchases. Write a query that returns each country along with the top Genre. For countries where 
   the maximum number of purchases is shared return all Genres. */

WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1;

/* Write a query that determines the customer that has spent the most on music for each country. 
   Write a query that returns the country along with the top customer and how much they spent. 
   For countries where the top amount spent is shared, provide all customers who spent this amount. */

with cte as (select c.customer_id,c.first_name,c.country,sum(i.total) as amount , 
row_number() over(partition by c.country order by sum(i.total)) as ranking 
from customer as c join invoice i on i.customer_id=c.customer_id
group by 1,2,3 
order by ranking)

select * from cte where ranking=1;
