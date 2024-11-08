-- Step 1: Database Creation
CREATE DATABASE IF NOT EXISTS student_db;
USE student_db;
/* 
- Creates a new database if it doesn't exist
- Switches to use that database
*/

-- Step 2: Table Creation
CREATE TABLE Stud_Marks (
    Roll INT PRIMARY KEY,      -- Unique identifier for each student
    name VARCHAR(50),          -- Student name
    total_marks INT           -- Total marks scored
);

CREATE TABLE Result (
    Roll INT,                 -- References Roll from Stud_Marks
    Name VARCHAR(50),         -- Student name
    Class VARCHAR(50),        -- Grade category
    FOREIGN KEY (Roll) REFERENCES Stud_Marks(Roll)  -- Ensures data integrity
);
/*
- Creates two tables: Stud_Marks for student data and Result for grades
- Foreign key ensures each Result record links to valid student
*/

-- Step 3: Sample Data Insertion
INSERT INTO Stud_Marks (Roll, name, total_marks) VALUES
(1, 'John Doe', 1200),    -- Distinction (>990)
(2, 'Jane Smith', 950),   -- First Class (900-989)
(3, 'Bob Wilson', 880),   -- Higher Second Class (825-899)
(4, 'Alice Brown', 1400), -- Distinction (>990)
(5, 'Charlie Davis', 830); -- Higher Second Class (825-899)
/*
- Inserts test data with different mark ranges
- Each student falls into different grade categories
*/

-- Step 4: Stored Procedure Creation
DELIMITER //  -- Changes delimiter to allow ; inside procedure

CREATE PROCEDURE proc_Grade()
BEGIN
    -- Variable declarations
    DECLARE v_finished INTEGER DEFAULT 0;  -- Flag for cursor completion
    DECLARE v_roll INT;                    -- To store roll number
    DECLARE v_name VARCHAR(50);            -- To store name
    DECLARE v_marks INT;                   -- To store marks
    DECLARE v_class VARCHAR(50);           -- To store grade
    
    -- Cursor declaration
    DECLARE stud_cursor CURSOR FOR 
        SELECT Roll, name, total_marks 
        FROM Stud_Marks;
    
    -- Error handler for when no more rows to fetch
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET v_finished = 1;
    
    -- Clear existing results
    DELETE FROM Result;
    
    -- Start processing
    OPEN stud_cursor;  -- Opens the cursor
    
    get_stud: LOOP
        -- Fetch next student record
        FETCH stud_cursor INTO v_roll, v_name, v_marks;
        
        -- Exit loop if no more records
        IF v_finished = 1 THEN
            LEAVE get_stud;
        END IF;
        
        -- Grade calculation logic
        IF v_marks >= 990 AND v_marks <= 1500 THEN
            SET v_class = 'Distinction';
        ELSEIF v_marks >= 900 AND v_marks <= 989 THEN
            SET v_class = 'First Class';
        ELSEIF v_marks >= 825 AND v_marks <= 899 THEN
            SET v_class = 'Higher Second Class';
        ELSE
            SET v_class = 'No Grade';
        END IF;
        
        -- Store result
        INSERT INTO Result (Roll, Name, Class)
        VALUES (v_roll, v_name, v_class);
        
    END LOOP get_stud;
    
    -- Clean up
    CLOSE stud_cursor;
    
    -- Show results
    SELECT * FROM Result;
END //

DELIMITER ;  -- Reset delimiter




Let me explain the code step by step with detailed explanations.

```sql
-- Step 1: Database Creation
CREATE DATABASE IF NOT EXISTS student_db;
USE student_db;
/* 
- Creates a new database if it doesn't exist
- Switches to use that database
*/

-- Step 2: Table Creation
CREATE TABLE Stud_Marks (
    Roll INT PRIMARY KEY,      -- Unique identifier for each student
    name VARCHAR(50),          -- Student name
    total_marks INT           -- Total marks scored
);

CREATE TABLE Result (
    Roll INT,                 -- References Roll from Stud_Marks
    Name VARCHAR(50),         -- Student name
    Class VARCHAR(50),        -- Grade category
    FOREIGN KEY (Roll) REFERENCES Stud_Marks(Roll)  -- Ensures data integrity
);
/*
- Creates two tables: Stud_Marks for student data and Result for grades
- Foreign key ensures each Result record links to valid student
*/

-- Step 3: Sample Data Insertion
INSERT INTO Stud_Marks (Roll, name, total_marks) VALUES
(1, 'John Doe', 1200),    -- Distinction (>990)
(2, 'Jane Smith', 950),   -- First Class (900-989)
(3, 'Bob Wilson', 880),   -- Higher Second Class (825-899)
(4, 'Alice Brown', 1400), -- Distinction (>990)
(5, 'Charlie Davis', 830); -- Higher Second Class (825-899)
/*
- Inserts test data with different mark ranges
- Each student falls into different grade categories
*/

-- Step 4: Stored Procedure Creation
DELIMITER //  -- Changes delimiter to allow ; inside procedure

CREATE PROCEDURE proc_Grade()
BEGIN
    -- Variable declarations
    DECLARE v_finished INTEGER DEFAULT 0;  -- Flag for cursor completion
    DECLARE v_roll INT;                    -- To store roll number
    DECLARE v_name VARCHAR(50);            -- To store name
    DECLARE v_marks INT;                   -- To store marks
    DECLARE v_class VARCHAR(50);           -- To store grade
    
    -- Cursor declaration
    DECLARE stud_cursor CURSOR FOR 
        SELECT Roll, name, total_marks 
        FROM Stud_Marks;
    
    -- Error handler for when no more rows to fetch
    DECLARE CONTINUE HANDLER FOR NOT FOUND 
        SET v_finished = 1;
    
    -- Clear existing results
    DELETE FROM Result;
    
    -- Start processing
    OPEN stud_cursor;  -- Opens the cursor
    
    get_stud: LOOP
        -- Fetch next student record
        FETCH stud_cursor INTO v_roll, v_name, v_marks;
        
        -- Exit loop if no more records
        IF v_finished = 1 THEN
            LEAVE get_stud;
        END IF;
        
        -- Grade calculation logic
        IF v_marks >= 990 AND v_marks <= 1500 THEN
            SET v_class = 'Distinction';
        ELSEIF v_marks >= 900 AND v_marks <= 989 THEN
            SET v_class = 'First Class';
        ELSEIF v_marks >= 825 AND v_marks <= 899 THEN
            SET v_class = 'Higher Second Class';
        ELSE
            SET v_class = 'No Grade';
        END IF;
        
        -- Store result
        INSERT INTO Result (Roll, Name, Class)
        VALUES (v_roll, v_name, v_class);
        
    END LOOP get_stud;
    
    -- Clean up
    CLOSE stud_cursor;
    
    -- Show results
    SELECT * FROM Result;
END //

DELIMITER ;  -- Reset delimiter

```

**Key Components Explained:**

1. **Database Structure**:
   - `Stud_Marks`: Primary table storing student information
   - `Result`: Table storing calculated grades
   - Foreign key relationship ensures data integrity

2. **Grade Categories**:
   - Distinction: 990-1500 marks
   - First Class: 900-989 marks
   - Higher Second Class: 825-899 marks
   - No Grade: Below 825 marks

3. **Procedure Components**:
   - Cursor: Used to process records one by one
   - Variables: Store temporary data during processing
   - Loop: Processes each student record
   - Error Handler: Manages end of data condition

4. **Processing Flow**:
```
Start
↓
Open Cursor
↓
Fetch Student Record
↓
Calculate Grade
↓
Store Result
↓
Repeat until done
↓
Show Results
↓
End
```

5. **Testing the System**:
```sql
-- Execute procedure
CALL proc_Grade();

-- View results
SELECT * FROM Result;
```

The main advantages of this system:
1. Automated grade calculation
2. Data integrity through foreign keys
3. Reusable procedure
4. Error handling included
5. Easy to modify grade boundaries

Would you like me to:
1. Explain any specific part in more detail?
2. Show how to add more grade categories?
3. Demonstrate error handling scenarios?
4. Show how to modify the grade calculation logic?




