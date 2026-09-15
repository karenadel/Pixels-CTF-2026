Forbidden Brownie I — Writeup

**Type**: Web Application

## TL;DR

menu.php?category= builds its SQL statement by simply inserting the value we provide in the middle of the query, without any sanitization. We exploited the injection using a single quote, then utilized UNION-based injection technique to gather info about the DB schema, list tables/columns, and retrieve a hidden menu entry.

## Recon

Looked at the application it seems to be an order website. There are 7 category tabs (Burgers, Pizza, Sandwiches, Drinks, Desserts, Snacks, Sides), and all of them point to menu.php?category=<name> URL. No mentions of "Forbidden Brownie" in the visible menu.

# Step 1 — Confirmation of the injection

Injected a single quote into the category parameter:

menu.php?category=Burgers'

Got the database error in response to our request, confirming that our payload ends up unescaped in the SQL statement.

# Step 2 — Identify the number of columns

Employed ORDER BY to get an idea about the number of columns in the underlying query:

menu.php?category=Burgers' ORDER BY 5-- -
menu.php?category=Burgers' ORDER BY 6-- -

5 was just fine, 6 failed — hence, there are exactly 5 columns in the query, which is the number required for UNION.

# Step 3 — Database fingerprinting

Instead of retrieving static values from a query, decided to extract database-related details using UNION SELECT:

menu.php?category=Burgers' UNION SELECT 1,@@version,3,4,5-- -
menu.php?category=Burgers' UNION SELECT 1,database(),3,4,5-- -

From here we were able to identify database version and the name of the current database, thus confirming that we are working with MySQL/MariaDB.

# Step 4 — Dump table names

Queried information_schema.tables via the injection point to enumerate all tables from the current database:

menu.php?category=Burgers' UNION SELECT 1,table_name,3,4,5 FROM information_schema.tables WHERE table_schema=database()-- -

Got the table 'menu_items'.

# Step 5 – Dump column names

Similar process, this time against information_schema.columns, but scoped to the above mentioned table:

menu.php?category=Burgers' UNION SELECT 1,column_name,3,4,5 FROM information_schema.columns WHERE table_name='menu_items'-- -

The query revealed all columns, including a suspicious column called "secret_note," which had no reason for being included in a publicly accessible menu.

# Step 6 – Dump hidden row

Based on the above, dumped all the relevant data directly, including hidden rows (that is, where is_visible != 1):

menu.php?category=Burgers' UNION SELECT id,name,category,secret_note,price FROM menu_items WHERE is_visible=0-- -

The item named "Forbidden Brownie" was found, not present on any of the 7 categories tabs; flag in secret_note column.

## Root cause
Any user input was immediately appended to the SQL statement as a raw string without any sanitizing, escaping, or parametrizing anything we put into the category parameter simply appeared in the real query that the database executed. After gaining control over the SQL injection, we could use the UNION SELECT method to execute an entirely different query in parallel to the intended one, complete with its own WHERE conditions, reading the internal metadata of the database (tables, columns) and then any data from any tables.