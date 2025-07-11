# emp_mgmt_project
Here's a comprehensive solution for the Employee Management and Attendance Tracker project:
# Step 1: Create Tables
-- Create Employees table
CREATE TABLE Employees (
    EmployeeID SERIAL PRIMARY KEY,
    Name VARCHAR(255) NOT NULL,
    DepartmentID INTEGER NOT NULL,
    RoleID INTEGER NOT NULL,
    Email VARCHAR(255) UNIQUE NOT NULL);

-- Create Departments table
CREATE TABLE Departments (
    DepartmentID SERIAL PRIMARY KEY,
    DepartmentName VARCHAR(255) NOT NULL);

-- Create Roles table
CREATE TABLE Roles (
    RoleID SERIAL PRIMARY KEY,
    RoleName VARCHAR(255) NOT NULL);

-- Create Attendance table
CREATE TABLE Attendances (
    AttendanceID SERIAL PRIMARY KEY,
    EmployeeID INTEGER NOT NULL,
    AttendanceDate DATE NOT NULL,
    ClockInTime TIMESTAMP NOT NULL,
    ClockOutTime TIMESTAMP,
    Status VARCHAR(50) NOT NULL CHECK(Status IN ('Present', 'Absent', 'Late')));

## Step 2: Insert Dummy Data
-- Insert dummy data into Employees table
DO $$
begin 
  for i in 1..200 loop
    insert into employees (Name ,DepartmentID,RoleId ,Email) 
	values 
	('employee'||i,(i % 5)+ 1,(i % 5)+1,'employee'||i||'@example.com');
  end loop;
end $$;

-- Insert dummy data into Departments table
INSERT INTO Departments (DepartmentName)
VALUES ('Sales'), ('Marketing'), ('IT'), ('HR'), ('Finance');

-- Insert dummy data into Roles table
INSERT INTO Roles (RoleName)
VALUES ('Manager'), ('Developer'), ('Salesperson'), ('HR Specialist'), ('Accountant');

-- Insert dummy data into Attendance table
INSERT INTO Attendances (EmployeeID, AttendanceDate, ClockInTime, ClockOutTime, Status)
VALUES 
(1, '2022-01-01', '08:00:00', '17:00:00', 'Present'),
(1, '2022-01-02', '08:30:00', '17:00:00', 'Late'),
(2, '2022-01-01', '09:00:00', '18:00:00', 'Present'),
-- Add more attendance records...

# Step 3: Write Queries for Monthly Attendance and Late Arrivals
-- Query for monthly attendance
SELECT 
    EXTRACT(MONTH FROM AttendanceDate) AS Month,
    COUNT(*) AS TotalAttendance
FROM 
    Attendances
GROUP BY 
    EXTRACT(MONTH FROM AttendanceDate);

-- Query for late arrivals
SELECT 
    E.Name,
    COUNT(*) AS LateArrivals
FROM 
    Attendances A
JOIN 
    Employees E ON A.EmployeeID = E.EmployeeID
WHERE 
    A.Status = 'Late'
GROUP BY 
    E.Name;

# Step 4: Add Triggers for Timestamp and Status
-- Create trigger function for timestamp
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.ClockInTime = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger for timestamp
CREATE TRIGGER update_timestamp_trigger
BEFORE INSERT ON Attendances
FOR EACH ROW
EXECUTE FUNCTION update_timestamp();

-- Create trigger function for status
CREATE OR REPLACE FUNCTION update_status()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.ClockInTime > '09:00:00' THEN
        NEW.Status = 'Late';
    ELSE 
        NEW.Status = 'Present';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger for status
CREATE TRIGGER update_status_trigger
BEFORE INSERT ON Attendance
FOR EACH ROW
EXECUTE FUNCTION update_status();

# Step 5: Create Functions to Calculate Total Work Hours
CREATE OR REPLACE FUNCTION calculate_work_hours(p_employee_id INTEGER, p_start_date DATE, p_end_date DATE)
RETURNS DECIMAL(10, 2) AS $$
DECLARE
    total_hours DECIMAL(10, 2);
BEGIN
    SELECT 
        SUM(EXTRACT(EPOCH FROM (ClockOutTime - ClockInTime))) / 3600
    INTO 
        total_hours
    FROM 
        Attendances
    WHERE 
        EmployeeID = p_employee_id
        AND AttendanceDate BETWEEN p_start_date AND p_end_date;
    RETURN total_hours;
END;
$$ LANGUAGE plpgsql;





    
