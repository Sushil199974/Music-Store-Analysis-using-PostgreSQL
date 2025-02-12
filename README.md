# 🎵 Music Store Data Analysis 🎶-using-PostgreSQL
This project focuses on analyzing a Music Store Database using SQL queries to extract valuable business insights. By answering a structured set of Easy, Moderate, and Advanced questions, we uncover trends related to sales, customer behavior, and music preferences.


# SQL Project - Music Store Data Analysis

## 📌 Project Overview
This project analyzes a **Music Store** dataset using **PostgreSQL**, answering business-related questions and deriving insights from the data. The dataset includes information about customers, invoices, tracks, artists, and genres.

## 📊 Key Questions & Queries
The queries are categorized into three difficulty levels: **Easy, Moderate, and Advanced**.

---
## 🟢 Question Set 1 - Easy

### 1️⃣ Who is the senior-most employee based on job title?
```sql
SELECT title, last_name, first_name
FROM employee
ORDER BY levels DESC
LIMIT 1;
```

### 2️⃣ Which countries have the most invoices?
```sql
SELECT COUNT(*) AS invoice_count, billing_country
FROM invoice
GROUP BY billing_country
ORDER BY invoice_count DESC;
```

### 3️⃣ What are the top 3 values of total invoice amounts?
```sql
SELECT total
FROM invoice
ORDER BY total DESC
LIMIT 3;
```

### 4️⃣ Which city has the highest total invoice amount?
```sql
SELECT billing_city, SUM(total) AS total_invoice_amount
FROM invoice
GROUP BY billing_city
ORDER BY total_invoice_amount DESC
LIMIT 1;
```

### 5️⃣ Who is the best customer (highest spender)?
```sql
SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spent
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spent DESC
LIMIT 1;
```

---
## 🔵 Question Set 2 - Moderate

### 1️⃣ Rock Music Listeners (Email, Name, Genre)
```sql
SELECT DISTINCT email, first_name, last_name, genre.name AS genre_name
FROM customer
JOIN invoice ON invoice.customer_id = customer.customer_id
JOIN invoiceline ON invoiceline.invoice_id = invoice.invoice_id
JOIN track ON track.track_id = invoiceline.track_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name = 'Rock'
ORDER BY email;
```

### 2️⃣ Top 10 Rock Artists by Number of Tracks
```sql
SELECT artist.name, COUNT(track.track_id) AS track_count
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name = 'Rock'
GROUP BY artist.name
ORDER BY track_count DESC
LIMIT 10;
```

### 3️⃣ Tracks Longer than the Average Song Length
```sql
SELECT name, milliseconds
FROM track
WHERE milliseconds > (SELECT AVG(milliseconds) FROM track)
ORDER BY milliseconds DESC;
```

---
## 🔴 Question Set 3 - Advanced

### 1️⃣ Amount Spent by Each Customer on Artists
```sql
WITH best_selling_artist AS (
    SELECT artist.artist_id, artist.name, SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
    FROM invoice_line
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN album ON album.album_id = track.album_id
    JOIN artist ON artist.artist_id = album.artist_id
    GROUP BY artist.artist_id
    ORDER BY total_sales DESC
    LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY c.customer_id, c.first_name, c.last_name, bsa.artist_name
ORDER BY amount_spent DESC;
```

### 2️⃣ Most Popular Genre per Country
```sql
WITH popular_genre AS (
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name,
           ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo
    FROM invoice_line
    JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
    JOIN customer ON customer.customer_id = invoice.customer_id
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN genre ON genre.genre_id = track.genre_id
    GROUP BY customer.country, genre.name
)
SELECT country, name AS top_genre, purchases
FROM popular_genre
WHERE RowNo = 1;
```

### 3️⃣ Highest-Spending Customer in Each Country
```sql
WITH customer_spending AS (
    SELECT customer.customer_id, first_name, last_name, billing_country, SUM(total) AS total_spent,
           ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo
    FROM invoice
    JOIN customer ON customer.customer_id = invoice.customer_id
    GROUP BY customer.customer_id, first_name, last_name, billing_country
)
SELECT * FROM customer_spending WHERE RowNo = 1;
```

---


---

