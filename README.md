# Library-Management-System-Advanced-SQL-Project
![Library_project](https://github.com/vikash-013/Library-Management-System-Advanced-SQL-Project/blob/main/library%20img.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.



## Project Structure

### 1. Database Setup
![ERD](https://github.com/vikash-013/Library-Management-System-Advanced-SQL-Project/blob/main/Diagram.png)

**Table Creation**
```sql
CREATE DATABASE library_db;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);



-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);



-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);

```
**Q 1. Create a New Book Record**
```sql
INSERT INTO books(isbn, books_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mocklingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B Lippincott & Co.');
SELECT * FROM books;

```
**Q 2: Update Member's Address**
```sql
UPDATE members
SET member_address = '125 main 5t'
WHERE member_id = 'C101';
SELECT * FROM members;
```

**Q 3: Delete a Record from the Issued Status Table**
```sql
SELECT * FROM issued_status;
DELETE FROM issued_status
WHERE issued_id = 'IS121';
```

**Q 4: Retrieve All Books Issued by a Specific Employee**
```sql
SELECT*FROM issued_status
WHERE issued_emp_id = 'E101';
```
**Q 5: List Members Who Have Issued More Than One Book**
```sql
SELECT issued_emp_id FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT (issued_id)>1;
```

### 3. CTAS (Create Table As Select)
**Q.6**
```sql
CREATE TABLE book_cnts
AS
SELECT 
    b.isbn,
	b.books_title,
	COUNT(ist.issued_id) as no_issued
FROM books as b 
JOIN 
issued_status as ist 
ON 
ist.issued_book_isbn = b.isbn
GROUP BY 1, 2;

SELECT* FROM book_cnts;
```
### 4. Data Analysis & Findings

 **Q 7.Retrieve All Books in a Specific Category**
 ```sql
 SELECT * FROM books 
WHERE category ='Classic';


```
 **Q 8: Find Total Rental Income by Category**
 ```sql
SELECT 
   b.category,
   SUM(b.rental_price),
   COUNT(*)
FROM books as b
JOIN 
issued_status as ist 
ON ist.issued_book_isbn = b.isbn
GROUP BY 1 
```

**Q 9 :List Members Who Registered in the Last 180 Days**
```sql
SELECT*  FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days'
```
**Q 10 :List Employees with Their Branch Manager's Name and their branch details**
```sql
SELECT 
    e1.*,
	b.manger_id,
	e2.emp_name as manger

FROM employees as e1 
JOIN 
branch as b 
ON b.branch_id = e1.branch_id
JOIN 
employees as e2
ON b.manger_id = e2.emp_id
```
**Q 11. Create a Table of Books with Rental Price Above a Certain Threshold**
```sql

CREATE TABLE books_price_greater_than_seven 
AS
SELECT * FROM books 
WHERE rental_price >7

SELECT * FROM 
books_price_greater_than_seven
```
 **Q 12: Retrieve the List of Books Not Yet Returned**
 ```sql
SELECT
    DISTINCT ist,issued_book_name
FROM issued_status as ist 
LEFT JOIN 
return_status as rs 
ON ist.issued_id = rs.issued_id
WHERE rs.return_id IS NULL
```


## Advanced SQL Operations
**Q 13: Identify Members with Overdue Books**  
```sql
SELECT
   ist.issued_member_id,
   m.member_name,
   bk.books_title,
   ist.issued_date,
    CURRENT_DATE -  ist.issued_date as over_dues_days
 FROM issued_status as ist
 JOIN 
 members as m 
 ON m.member_id = ist.issued_member_id
JOIN 
books as bk
ON bk.isbn = ist.issued_book_isbn
LEFT JOIN 
return_status as rs 
ON rs.issued_id = ist.issued_id
WHERE 
rs.return_date IS NULL 
AND 
 (CURRENT_DATE -  ist.issued_date) >30

ORDER BY 1
```
**Q 14: Branch Performance Report**  
```sql
CREATE TABLE branch_reports
AS
SELECT
     b.branch_id,
	 b.manger_id,
	 COUNT(ist.issued_id) as number_book_issued,
	 COUNT(rs.return_id) as number_of_book_return,
	 SUM(bk.rental_price) as total_revenue
FROM issued_status as ist
JOIN 
employees as e 
ON e.emp_id = ist.issued_emp_id
JOIN 
branch as b 
ON e.branch_id = b.branch_id
 LEFT JOIN 
 return_status as rs 
 ON rs.issued_id = ist.issued_id 
 JOIN 
 books as bk 
 ON ist.issued_book_isbn = bk.isbn
 GROUP BY 1,2;

SELECT * FROM branch_reports;
```
**Q 15: CTAS: Create a Table of Active Members**  
```sql
CREATE TABLE active_members
AS 
SELECT * FROM members 
WHERE member_id IN (SELECT
                     DISTINCT issued_member_id
					 FROM issued_status
					 WHERE 
					   issued_date >= CURRENT_DATE - INTERVAL '2 month '
					   
);
SELECT*FROM active_members;
```
**Q 16: Find Employees with the Most Book Issues Processed**  
```sql
SELECT 
      e.emp_name,
	  b.*,
	  COUNT(ist.issued_id) as no_book_issued
FROM issued_status as ist 
JOIN 
employees as e
ON e.emp_id = ist.issued_emp_id
JOIN 
branch as b 
ON e.branch_id = b.branch_id 
GROUP BY 1,2
```
**Q 17: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
```sql
CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(30), p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v_status VARCHAR(10);

BEGIN
    SELECT 
        status 
        INTO
        v_status
    FROM books
    WHERE isbn = p_issued_book_isbn;

    IF v_status = 'yes' THEN

        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES
        (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        UPDATE books
            SET status = 'no'
        WHERE isbn = p_issued_book_isbn;

        RAISE NOTICE 'Book records added successfully for book isbn : %', p_issued_book_isbn;


    ELSE
        RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;
    END IF;
END;
$$

SELECT * FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT * FROM issued_status;

CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'


```
## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.







