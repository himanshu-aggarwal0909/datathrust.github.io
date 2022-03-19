---
layout: post
title: Running SQL Queries on postgresql/redshift using python.
# author: "Himanshu Aggarwal"
# description: "Python utility to Run SQL queries in redshift"
date: 2022-03-01
categories: data engineer code wall
---


Installing psycopg2
```sh
python3 -m pip install psycopg2-binary
```

Simple Code Template

```python

import psycopg2


def get_connection(redshift_connection_details):
	"""
	This function is used to set up a connection on the passed postgresql database.
	:param: redshift_connection_details (connection details dictionary required keys (host, user, password, database, port))

	:return: Connection object
	"""
	try: 
		#Setting up connection
		conn = psycopg2.connect(**redshift_connection_details)
		return conn
	except Exception as e:
		print("Error in establishing the connection Error Message :{}".format(str(error)))
		raise e


def run_sql_query(connection, sql_query, return_results=True):
	"""
	This function is used to run sql query using the provided connection object
	:param: connection DB Connection object
	:param: sql_query  SQL Query to run on the database
	:param: return_results if True,return the results of the executed query 

	:return: None when return_results=False otherwise results
	"""
	try:
		#Creating the cursor
		cursor = connection.cursor()
		#Executing Query
		cursor.execute(sql_query)
		if return_results:
			results = cursor.fetchall()
			return results
	except Exception as e:
		print("Error in running sql query Error Message : {}".format(str(e)))

REDSHIFT_CONNECTION_DETAILS = {
	"host"     : "dummy-hostname",
	"user"     : "dummy-user",
	"password" : "dummy-password",
	"database" : "dummy-database",
	"port"     : "dummy-port"
}

connection = get_connection(REDSHIFT_CONNECTION_DETAILS)

sql_query = "SELECT VERSION();"

results = run_sql_query(connection, sql_query)

connection.close()
```

