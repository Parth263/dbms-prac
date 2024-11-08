-- 1. Create table to store radius and area
CREATE TABLE AREAS(
    RADIUS FLOAT,   -- To store the radius values
    AREA FLOAT      -- To store the calculated areas
);

-- 2. Create procedure that calculates areas
CREATE PROCEDURE FIND_CIRCLE_AREA()
BEGIN
    DECLARE R INT;      -- Declare variable for radius
    DECLARE A FLOAT;    -- Declare variable for area
    SET R = 5;          -- Initialize radius to 5
    
    loop_LABEL: LOOP    -- Start a labeled loop
        IF R > 9 THEN   -- Exit condition
            LEAVE loop_LABEL;
        END IF;
        
        SET A = 3.14 * R * R;    -- Calculate area (πr²)
        
        -- Store radius and area in table
        INSERT INTO AREAS(RADIUS, AREA) VALUES(R, A);
        
        SET R = R + 1;  -- Increment radius
    END LOOP;
END;

-- The procedure will:
-- 1. Calculate areas for circles with radius 5 to 9
-- 2. Store each radius and its corresponding area in the AREAS table