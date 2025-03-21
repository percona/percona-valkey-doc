# Database caching with Valkey  
   
Valkey is an in-memory key/value store that serves as the caching database for applications. Valkey keeps the result of various queries in memory for a defined period An application receives the result of a repeating query  from memory  directly and doesn’t have to query the database. 

Let’s take an online shop as an example to see how it works. 

When you search a website and add goods or periodically check your cart, you see your order. If you linger too long and don’t proceed with the purchase, your cart is empty. This is done to free up memory.  
   
This guide will give you an idea how to implement a simple value caching and a query result caching with Valkey in your application.  
   
We will use the following software:  
 

* PostgreSQL as the database where you store the data  
* Valkey as the caching database  
* Python for code samples as one of the most programming languages

   
## Simple value caching  
   
Simple value caching is keeping the result of a query that is a simple string-based value in a cache. An example of such values is an employee name in a company.

In Valkey, you operate with simple string-based values using the SET and GET commands.

The workflow is the following:  
   
When a user searches for an employee, your application first queries Valkey to check if the result of this search is already in memory. If it's there, the application retrieves it from Valkey and serves to the user. If it's not there (it has already expired or it is a new query), the application sends the request to the database. The result is served to the user and is cached in Valkey with an expiry time.  
   
Here's what you need to do to implement this workflow:  
 
* Create a table Employees in PostgreSQL and populate it with data

   * Create a database Staff:

      ```sql
      CREATE DATABASE IF NOT EXIST Staff;
	  ```

   * Create a table Employees:

	  ```sql
	  CREATE TABLE Employees (
		 id SERIAL PRIMARY KEY,
		 name VARCHAR(50) NOT NULL
	  );
	  ```

   * Populate the table with data:

	  ```sql
	  INSERT INTO Employees (Name)
	  VALUES 
	  ('Alice Johnson'),
	  ('Bob Smith'),
	  ('Charlie Brown'),
	  ('Diana Evans'),
	  ('Ethan Williams');
      ```

   * Use the following statement to retrieve the data:

      ```sql
      SELECT * FROM Employees where name = '<name>';
      ```


* Write a function to create a SET key in Valkey that equals the `id:name` of the employee and set the expiry time for a key to 30 seconds.

   ```
   valkey-cli:6379> SET user:1 "Alice Johnson" EX 30
   ```

This code sample shows how it looks like:

   
   
## Query result caching  
   
This approach is used for complex queries that return multiple values. For this that can be retrieved from several tables. Instead of using the SET key which is a single string-based value in Valkey, you use a more complex HASH key. Hash keys allow storing multiple values associated with one key. A key to use in this case is the hash that you calculate for the query result.  

Let's extend the previous example and add the information about an employee's department to the search results. This information is stored in a separate table. The result of a query is a joint information from both tables.  
  
The workflow is the same as for the simple value caching: 

A user searches for an employee. The app first checks the result of the search request in Valkey. If it's there, the app retrieves it from Valkey and serves it to the user. If it is a new search or it has already expired and is no longer in memory, the app queries the database, retrieves the result and serves it to the user. The app also creates a hash for the search result and saves it in Valkey with an expiry time of 2 minutes.  
   

Here's what you need to do to implement this workflow: 

* Create a table Department for the Staff database:

   ```sql
   CREATE TABLE Department (
	  id SERIAL PRIMARY KEY,
	  name VARCHAR(50) NOT NULL
   );
   ```

* Populate the table with data:

   ```sql
   INSERT INTO Department (Name, Job_Title, Employee_ID)
   VALUES 
   ('Finance', 'Accountant', 1),
   ('Human Resources', 'HR Manager', 2),
   ('IT', 'Software Engineer', 3),
   ('Sales', 'Sales Representative', 4),
   ('Marketing', 'Marketing Specialist', 5);
   ```

* Use this query to retrieve the employee information:

   ```sql
   SELECT 
       Employees.Name AS Employee_Name,
       Department.Name AS Department_Name,
       Department.Job_Title
   FROM 
       Employees
   JOIN 
       Department ON Employees.ID = Department.Employee_ID;
   ```

* Write a function to create a hash for the query result.  
* Write a HASH key in Valkey that equals to the query hash and has the result of the query as values. Set the expiration time for a key.  
   
Here’s the sample code to implement this  
   
   
   
   