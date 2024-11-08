-- PART 1: TABLE CREATION
CREATE TABLE Borrower (
    Roll_no NUMBER PRIMARY KEY,  -- Unique identifier for each student
    Name VARCHAR2(50),          -- Student name
    Date_of_Issue DATE,         -- When the book was borrowed
    Name_of_Book VARCHAR2(100), -- Book name
    Status CHAR(1) DEFAULT 'I'  -- 'I' for Issued, 'R' for Returned
);

CREATE TABLE Fine (
    Roll_no NUMBER,            -- References Borrower table
    Fine_Date DATE,            -- When fine was charged
    Amt NUMBER(10,2),          -- Fine amount with 2 decimal places
    FOREIGN KEY (Roll_no) REFERENCES Borrower(Roll_no)  -- Ensures referential integrity
);

-- PART 2: MAIN PROCEDURE
CREATE OR REPLACE PROCEDURE process_book_return (
    p_roll_no IN NUMBER,        -- Input parameter for Roll number
    p_book_name IN VARCHAR2     -- Input parameter for Book name
) IS
    -- Local variable declarations
    v_issue_date DATE;          -- To store issue date
    v_days NUMBER;              -- To store number of days
    v_fine_amount NUMBER := 0;  -- Initialize fine amount to 0
    v_student_name VARCHAR2(50); -- To store student name
    v_status CHAR(1);           -- To store current status
    
    -- Custom exception declarations
    e_invalid_book EXCEPTION;
    e_already_returned EXCEPTION;
    
BEGIN
    -- STEP 1: Validate and get book issue details
    SELECT Date_of_Issue, Name, Status
    INTO v_issue_date, v_student_name, v_status
    FROM Borrower
    WHERE Roll_no = p_roll_no 
    AND Name_of_Book = p_book_name;
    
    -- STEP 2: Check if already returned
    IF v_status = 'R' THEN
        RAISE e_already_returned;
    END IF;
    
    -- STEP 3: Calculate days since issue
    v_days := TRUNC(SYSDATE - v_issue_date);
    
    -- STEP 4: Calculate fine using CASE statement
    CASE 
        WHEN v_days BETWEEN 15 AND 30 THEN
            -- Rs.5 per day after 15 days
            v_fine_amount := (v_days - 15) * 5;
            
        WHEN v_days > 30 THEN
            -- Rs.5 per day from day 15-30 (15 days * 5)
            -- Plus Rs.50 per day after 30 days
            v_fine_amount := (15 * 5) + ((v_days - 30) * 50);
            
        ELSE
            -- No fine for returns within 15 days
            v_fine_amount := 0;
    END CASE;
    
    -- STEP 5: Update book status
    UPDATE Borrower 
    SET Status = 'R'
    WHERE Roll_no = p_roll_no 
    AND Name_of_Book = p_book_name;
    
    -- STEP 6: Handle fine if applicable
    IF v_fine_amount > 0 THEN
        -- Insert fine record
        INSERT INTO Fine (Roll_no, Fine_Date, Amt)
        VALUES (p_roll_no, SYSDATE, v_fine_amount);
        
        -- Display fine details
        DBMS_OUTPUT.PUT_LINE('Fine Details:');
        DBMS_OUTPUT.PUT_LINE('Student: ' || v_student_name);
        DBMS_OUTPUT.PUT_LINE('Days delayed: ' || v_days);
        DBMS_OUTPUT.PUT_LINE('Fine Amount: Rs.' || v_fine_amount);
    ELSE
        DBMS_OUTPUT.PUT_LINE('Book returned within time limit. No fine.');
    END IF;
    
    -- STEP 7: Commit transaction
    COMMIT;
    
-- STEP 8: Exception handling
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        -- Handle invalid roll number or book name
        RAISE_APPLICATION_ERROR(-20001, 'Invalid Roll Number or Book Name');
        
    WHEN e_already_returned THEN
        -- Handle already returned book
        RAISE_APPLICATION_ERROR(-20002, 'Book already returned');
        
    WHEN OTHERS THEN
        -- Handle any other errors
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
        RAISE;
END process_book_return;
/

-- PART 3: HELPER PROCEDURE FOR VIEWING FINES
CREATE OR REPLACE PROCEDURE view_fine_details (
    p_roll_no IN NUMBER
) IS
    -- Cursor declaration for fine details
    CURSOR c_fine IS
        SELECT f.Fine_Date, f.Amt, b.Name, b.Name_of_Book
        FROM Fine f
        JOIN Borrower b ON f.Roll_no = b.Roll_no
        WHERE f.Roll_no = p_roll_no;
        
    v_total_fine NUMBER := 0;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Fine History for Roll No: ' || p_roll_no);
    DBMS_OUTPUT.PUT_LINE('----------------------------------------');
    
    -- Process each fine record
    FOR fine_rec IN c_fine LOOP
        DBMS_OUTPUT.PUT_LINE(
            'Date: ' || fine_rec.Fine_Date || 
            ' | Amount: Rs.' || fine_rec.Amt ||
            ' | Book: ' || fine_rec.Name_of_Book
        );
        v_total_fine := v_total_fine + fine_rec.Amt;
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('----------------------------------------');
    DBMS_OUTPUT.PUT_LINE('Total Fine Amount: Rs.' || v_total_fine);
END view_fine_details;
/