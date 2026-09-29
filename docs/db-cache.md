# Database caching with Valkey  
   
Valkey is an in-memory key/value store with optional persistence, that serves as the caching layer for database applications. The two typical caching use cases for Valkey are simple value caching and query result caching. Valkey can keep the result of various queries in memory for a period defined by the Valkey object time-to-live(TTL). Once cached, an application receives the result of a query from Valkey directly and doesn't have to query the source database. 

An online shop as a good example to see how it works: 

When you search a website and add goods or periodically check your cart, you see your order. If you linger too long and don't proceed with the purchase, your cart is emptied. This is done to free up memory.  
   
This guide will give you an idea how to implement a simple value caching and a query result caching with Valkey in your application.  
   
We will use the following software:  
 

* PostgreSQL as the database where you store persistent data  
* Valkey as the caching database  
* Python for code samples

   
## Simple value caching  
   
Simple value caching is keeping object values in a cache. Some examples : application configuration values, application reference data and even complete files.

In Valkey, you operate with string datatype values using the SET and GET commands or hash datatype values with HSET and HGET, HGETALL for simple value caching.

To illustrate simple-value caching, we use employee data which typically doesn't change that often so can have a high TTL. 

The workflow is the following:  
   
When a user searches for an employee, the application first checks in Valkey if this employee data has already been cached there. If it's there, the application retrieves it from Valkey and returns it to the user. If it's not there (it has already expired or it is a new search), the application sends the request to the database. This result is stored in Valkey with an expiry time, and then served to the user.  
   
Here's how you can implement this workflow:  
 
1. Create a table Employees in PostgreSQL and populate it with data

    * Create a database `staff`:

       ```sql
       CREATE DATABASE IF NOT EXIST staff;
       ```

    * Create a table `employees`:

       ```sql
       CREATE TABLE employees (
          id SERIAL PRIMARY KEY,
          name VARCHAR(50) NOT NULL
       );
       ```

    * Populate the table with data:

       ```sql
       INSERT INTO Employees (Name) VALUES ('Alice Johnson');
       INSERT INTO Employees (Name) VALUES ('Bob Smith');
       INSERT INTO Employees (Name) VALUES ('Charlie Brown');
       INSERT INTO Employees (Name) VALUES ('Diana Evans');
       INSERT INTO Employees (Name) VALUES ('Ethan Williams');
       ```

2. Use the following statement to retrieve the data: 

    ```sql
    SELECT * FROM employees where name = '<name>';
    ```

3. The application code would look like this:

    ```py
    import psycopg2
    import valkey  # Import the valkey-py library
    import time

    def get_employee_name_with_caching(conn_params, valkey_params, employee_id, ttl=3600):
        """
        Retrieves employee name from PostgreSQL, caching it in Valkey with a TTL.

        Args:
            conn_params (dict): PostgreSQL connection parameters.
            valkey_params (dict): Valkey connection parameters.
            employee_id (int): ID of the employee.
            ttl (int, optional): Time-to-live (in seconds) for the cached data. Defaults to 3600.

        Returns:
            The employee name (str) or None if not found.
        """

        try:
            v = valkey.Valkey(**valkey_params)  # Connect to Valkey
            cached_name = v.get(f"employee:{employee_id}")

            if cached_name:
                return cached_name.decode('utf-8')

            conn = psycopg2.connect(**conn_params)
            cur = conn.cursor()

            cur.execute("SELECT name FROM employees WHERE id = %s", (employee_id,))
            result = cur.fetchone()

            if result:
                employee_name = result[0]
                v.setex(f"employee:{employee_id}", ttl, employee_name)
                return employee_name
            else:
                return None

        except psycopg2.Error as e:
            print(f"PostgreSQL Error: {e}")
            return None
        except valkey.ValkeyError as e:
            print(f"Valkey Error: {e}")
            return None

        finally:
            if 'conn' in locals() and conn:
                cur.close()
                conn.close()

    # Example usage:
    # PostgreSQL connection parameters
    conn_params = {
        "host": "your_host",
        "database": "your_database",
        "user": "your_user",
        "password": "your_password",
    }

    # Valkey connection parameters
    valkey_params = {
        "host": "your_valkey_host",
        "port": 6379,  # Default Valkey port
        "db": 0,       # Default Valkey database
        # Add other Valkey parameters as needed (password, etc.)
    }

    employee_id = 1

    employee_name = get_employee_name_with_caching(conn_params, valkey_params, employee_id)
    if employee_name:
        print(f"Employee name: {employee_name}")
    else:
        print("Employee not found.")
    employee_name_cached = get_employee_name_with_caching(conn_params, valkey_params, employee_id)
    print(f"Employee name (cached): {employee_name_cached}")
    ```
   
   
## Query result caching  
   
This approach is used for complex queries that return multiple values. In this case, the whole query result is cached to offload processing on the backend database. Choosing a suitable TTL for the query result depends on the frequency of change of the tables that are the part of the query. A cached result can also become invalid on a change in underlying table in the backend database.

In Valkey, you now use a more complex HASH key. Hash keys allow storing multiple values associated with one key. A key to use in this case is the hash that you calculate for the query result.

Let's extend the previous example and add the information about an employee's department to the search results. This information is stored in a separate table. The result of a query is a joint information from both tables.  
  
The workflow is the same as for the simple value caching: 

A user searches for an employee. The app first checks the result of the search request in Valkey. If it's there, the app retrieves it from Valkey and serves it to the user. If it is a new search or it has already expired and is no longer in memory, the app queries the database, retrieves the result and serves it to the user. The app also creates a hash for the search result and saves it in Valkey with an expiry time of 2 minutes.  
   

Here's what you need to do to implement this workflow: 

1. Create a table `departments` for the `staff` database:

     ```sql
     CREATE TABLE departments (
        id SERIAL PRIMARY KEY,
        name VARCHAR(50) NOT NULL
     );
     ```

2. Create the `employee` table with department id

    ```sql
    CREATE TABLE employees (
      id SERIAL PRIMARY KEY,
      name VARCHAR(50) NOT NULL,
      job_title VARCHAR(50) NOT NULL,
      dept_id INT NOT NULL
    );
    ```

3. Populate the `departments` table with data:

    ```sql
    INSERT INTO departments (Name) VALUES ('Finance');
    INSERT INTO departments (Name) VALUES ('Human Resources');
    INSERT INTO departments (Name) VALUES ('IT');
    INSERT INTO departments (Name) VALUES ('Sales');
    INSERT INTO departments (Name) VALUES ('Marketing');
    ```

4. And now insert some data to the `employees` table:

    ```sql
    INSERT INTO Employees (Name, Job_title, Dept_id) VALUES ('Alice Johnson','Accountant', 1);
    INSERT INTO Employees (Name, Job_title, Dept_id) VALUES ('Bob Smith', 'HR Manager', 2);
    INSERT INTO Employees (Name, Job_title, Dept_id) VALUES ('Charlie Brown', 'Software Engineer', 3);
    INSERT INTO Employees (Name, Job_title, Dept_id) VALUES ('Diana Evans', 'Sales Representative', 4);
    INSERT INTO Employees (Name, Job_title, Dept_id) VALUES ('Ethan Williams', 'Marketing Specialist', 5);
    ```

5. Use this query to retrieve the employee information:

    ```sql
    SELECT 
        employees.name AS employee_name,
        departments.name AS department_name,
        employees.job_title
    FROM 
        employees
    INNER JOIN 
        departments
    ON
        employees.dept_id = departments.id;
    ```
  
Here’s the sample code to implement this:  
   
```py
import psycopg2     
import valkey  
import hashlib Here’s the sample code to implement this  
import json 

def get_employee_department_data(conn_params, valkey_params, query, ttl=3600):   
    """  
    Executes a SQL query, caches the result in Valkey using a query hash as the key,   
    and returns the result.   
    Args:   
        conn_params (dict): PostgreSQL connection parameters.  
        valkey_params (dict): Valkey connection parameters. 
        query (str): The SQL query to execute.  
        ttl (int, optional): Time-to-live (in seconds) for the cached data. Defaults to 3600.   
    Returns:   
        A list of dictionaries representing the query result, or None if an error occurs. 
    """  
    try: 
        v = valkey.Valkey(**valkey_params)   
        query_hash = hashlib.sha256(query.encode()).hexdigest() # Hash the query 
        cached_result = v.get(query_hash) 
        if cached_result:  
            return json.loads(cached_result.decode('utf-8'))   
        conn = psycopg2.connect(**conn_params)  
        cur = conn.cursor()   
        cur.execute(query) 
        columns = [desc[0] for desc in cur.description] #Get column names  
        result = [dict(zip(columns, row)) for row in cur.fetchall()] 
        v.setex(query_hash, ttl, json.dumps(result)) #Store as JSON in Valkey 
        return result   
    except (psycopg2.Error, valkey.ValkeyError) as e: 
        print(f"Error: {e}")  
        return None  
    finally:   
        if 'conn' in locals() and conn:   
            cur.close() 
            conn.close()   
# Example usage:  
conn_params = {   
    "host": "your_host",   
    "database": "your_database", 
    "user": "your_user",   
    "password": "your_password", 
}  
valkey_params = { 
    "host": "your_valkey_host",  
    "port": 6379, 
    "db": 0,   
}  
sql_query = """   
SELECT   
    employees.name AS employee_name,   
    departments.name AS department_name,  
    employees.job_title 
FROM  
    employees  
INNER JOIN  
    departments   
ON 
    employees.dept_id = departments.id;   
"""   
data = get_employee_department_data(conn_params, valkey_params, sql_query) 
if data: 
    print(json.dumps(data, indent=2)) # Print the result in a readable JSON format. 
else: 
    print("Failed to retrieve data.")  
#Example of retrieving cached data  
cached_data = get_employee_department_data(conn_params, valkey_params, sql_query)   
print(json.dumps(cached_data, indent=2))     
```   
  
   
   