Here is the solution tailored for terminal execution, with each block separated to run sequentially:

### Step 1: Create the Database and Tables
Run each command separately.

```sql
CREATE DATABASE IF NOT EXISTS library_db;
USE library_db;
```

```sql
CREATE TABLE borrower (
    rollno INT PRIMARY KEY,
    name VARCHAR(50),
    dateofissue DATE,
    name_of_book VARCHAR(100),
    status CHAR(1) DEFAULT 'I'  -- 'I' for issued, 'R' for returned
);
```

```sql
CREATE TABLE fine (
    rollno INT,
    date DATE,
    amount DECIMAL(10,2),
    FOREIGN KEY (rollno) REFERENCES borrower(rollno)
);
```

### Step 2: Insert Sample Data into the Borrower Table

```sql
INSERT INTO borrower (rollno, name, dateofissue, name_of_book) VALUES 
(1, 'Prathamesh', DATE_SUB(CURDATE(), INTERVAL 40 DAY), 'Database Systems'),
(2, 'Arjun', DATE_SUB(CURDATE(), INTERVAL 20 DAY), 'Java Programming'),
(3, 'Mohit', DATE_SUB(CURDATE(), INTERVAL 10 DAY), 'Python Basics'),
(4, 'Kshitij', DATE_SUB(CURDATE(), INTERVAL 25 DAY), 'Web Development');
```

### Step 3: Create Procedure for Fine Calculation with Exception Handling

1. Set delimiter for procedure:

    ```sql
    DELIMITER //
    ```

2. Create the `calculate_fine` procedure:

    ```sql
    CREATE PROCEDURE calculate_fine(IN p_rollno INT, IN p_name_of_book VARCHAR(100))
    BEGIN
        DECLARE v_issue_date DATE;
        DECLARE v_days INT;
        DECLARE v_fine_amount DECIMAL(10,2);
        DECLARE v_book_status CHAR(1);
        
        -- Custom exception for incorrect book
        DECLARE EXIT HANDLER FOR SQLEXCEPTION
        BEGIN
            SELECT 'Error: Roll number or book name does not exist or has already been returned.' AS error;
            ROLLBACK;
        END;
        
        -- Start Transaction
        START TRANSACTION;
        
        -- Get book issue details
        SELECT dateofissue, status INTO v_issue_date, v_book_status
        FROM borrower
        WHERE rollno = p_rollno AND name_of_book = p_name_of_book;
        
        -- Check if book is already returned
        IF v_book_status = 'R' THEN
            SELECT 'Book already returned' AS message;
            ROLLBACK;
            RETURN;
        END IF;

        -- Calculate days from issue date
        SET v_days = DATEDIFF(CURDATE(), v_issue_date);

        -- Determine fine amount based on days
        IF v_days <= 15 THEN
            SET v_fine_amount = 0;
        ELSEIF v_days <= 30 THEN
            SET v_fine_amount = (v_days - 15) * 5;
        ELSE
            SET v_fine_amount = (15 * 5) + ((v_days - 30) * 50);
        END IF;

        -- Insert fine record if applicable
        IF v_fine_amount > 0 THEN
            INSERT INTO fine (rollno, date, amount)
            VALUES (p_rollno, CURDATE(), v_fine_amount);
        END IF;

        -- Update book status to 'returned'
        UPDATE borrower
        SET status = 'R'
        WHERE rollno = p_rollno AND name_of_book = p_name_of_book;

        -- Commit transaction
        COMMIT;

        -- Display results
        SELECT 
            b.rollno,
            b.name,
            b.name_of_book,
            v_days AS days_kept,
            v_fine_amount AS fine_amount
        FROM borrower b
        WHERE b.rollno = p_rollno;
    END //
    ```

3. Reset delimiter:

    ```sql
    DELIMITER ;
    ```

### Step 4: Test the `calculate_fine` Procedure

Run the procedure for a specific borrower, providing `rollno` and `name_of_book`.

```sql
CALL calculate_fine(1, 'Database Systems');
```

### Step 5: Create Procedure to View All Fines

1. Set delimiter:

    ```sql
    DELIMITER //
    ```

2. Create the `view_all_fines` procedure:

    ```sql
    CREATE PROCEDURE view_all_fines()
    BEGIN
        SELECT 
            b.rollno,
            b.name,
            b.name_of_book,
            b.dateofissue,
            b.status,
            f.amount AS fine_amount,
            f.date AS fine_date
        FROM borrower b
        LEFT JOIN fine f ON b.rollno = f.rollno
        ORDER BY b.rollno;
    END //
    ```

3. Reset delimiter:

    ```sql
    DELIMITER ;
    ```

### Step 6: View All Fines

Call the `view_all_fines` procedure to view all borrowers and their fines, if any:

```sql
CALL view_all_fines();
```

This setup includes exception handling within the procedure, updates the book's status upon return, and calculates and records fines when appropriate.