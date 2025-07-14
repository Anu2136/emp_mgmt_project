# emp_mgmt_project
Here's a comprehensive solution for the Employee Management and Attendance Tracker project:
# Step 1: Create Tables

<img width="1301" height="1078" alt="Screenshot 2025-07-07 183412" src="https://github.com/user-attachments/assets/59bd599c-e10f-4106-8df8-907231abf9fb" />

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
# result set :-
<img width="859" height="936" alt="Screenshot 2025-07-14 175939" src="https://github.com/user-attachments/assets/b933ebd1-af30-4fe1-9908-5384250c5a1a" />

-- Insert dummy data into Departments table
INSERT INTO Departments (DepartmentName)
VALUES ('Sales'), ('Marketing'), ('IT'), ('HR'), ('Finance');
# result set :-
<img width="804" height="1007" alt="Screenshot 2025-07-14 180543" src="https://github.com/user-attachments/assets/d45fb171-3957-45b5-83a8-6a096d01830a" />

-- Insert dummy data into Roles table
INSERT INTO Roles (RoleName)
VALUES ('Manager'), ('Developer'), ('Salesperson'), ('HR Specialist'), ('Accountant');
# result set :-
<img width="761" height="1021" alt="Screenshot 2025-07-14 180612" src="https://github.com/user-attachments/assets/9376db9f-0edc-4298-a9e0-3518a2e8a1f4" />

-- Insert dummy data into Attendances table
INSERT INTO Attendances (EmployeeID, AttendanceDate, ClockInTime, ClockOutTime, Status)
VALUES 
(1, '2022-01-01', '08:00:00', '17:00:00', 'Present'),
(1, '2022-01-02', '08:30:00', '17:00:00', 'Late'),
(2, '2022-01-01', '09:00:00', '18:00:00', 'Present'),
-- Add more attendance records...
# result set :-
<img width="975" height="945" alt="Screenshot 2025-07-14 180644" src="https://github.com/user-attachments/assets/aa7095b4-4a3c-4c76-96b0-d76c7011cc72" />

# Step 3: Write Queries for Monthly Attendance and Late Arrivals
-- Query for monthly attendance
SELECT 
    EXTRACT(MONTH FROM AttendanceDate) AS Month,
    COUNT(*) AS TotalAttendance
FROM 
    Attendances
GROUP BY 
    EXTRACT(MONTH FROM AttendanceDate);
 # result set :-
   <img width="430" height="390" alt="Screenshot 2025-07-14 180900" src="https://github.com/user-attachments/assets/bb865049-7191-4c8c-a1fb-b6e1059c60d0" />

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
# result set :-
<img width="666" height="789" alt="Screenshot 2025-07-14 180950" src="https://github.com/user-attachments/assets/d5dc533b-1b2f-4b27-a008-20743a67dc36" />

# Step 4: Add Triggers for Timestamp and Status
-- Create trigger function for timestamp
<img width="664" height="633" alt="Screenshot 2025-07-14 183114" src="https://github.com/user-attachments/assets/fae0cfe1-16de-4a27-a44a-679f37974219" />

-- Create trigger function for status
<img width="615" height="738" alt="Screenshot 2025-07-14 183239" src="https://github.com/user-attachments/assets/4383e980-af46-4e42-b139-22998255408a" />

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

# --with loop to select particular set of records:-
do $$
declare 
  employeeid integer;
begin
	for employeeid in 1..30 loop 
		raise notice 'emp_ID :% ,total work hours : %',employeeid,calculate_work_hours(employeeid,'01-01-2022','01-06-2022');
	end loop;
end $$;
# result set :-
<img width="561" height="817" alt="Screenshot 2025-07-14 184940" src="https://github.com/user-attachments/assets/b9ae7f32-e145-4f77-a236-e7382509313f" />


# using a SELECT statement with a table :-
select emlpoyeeid,
calculate_work_hours(employeeid,'01-01-2022','01-06-2022')
as total_work_hours
from employees;
# result set :-
<img width="630" height="940" alt="Screenshot 2025-07-14 184249" src="https://github.com/user-attachments/assets/9dee0c6e-acf4-4ef4-910c-2d9b3b90b960" />




    
