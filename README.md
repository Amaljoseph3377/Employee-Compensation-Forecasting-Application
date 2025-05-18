🧾 Employee Compensation Forecasting Application

Objective

This Proof of Concept (PoC) demonstrates a basic **Employee Compensation Forecasting Application** for a mid-sized organization. It enables HR and business stakeholders to:

* Analyze current employee compensation
* Simulate compensation adjustments
* Visualize workforce distribution
* Export filtered employee data

Tools & Technologies Used
 Component         Technology                                        
 Backend         | SQL Server                                        
 Programming     | C# (.NET Framework)                               
 Frontend        | Windows Forms / ASP.NET MVC / WPF (choose one)    
 Charts          | Chart.js / System.Windows.Forms.DataVisualization 
 Database Access | ADO.NET (using Stored Procedures)                 
 File Export     | CSV via .NET Libraries                            

Database Setup
1. Table Creation Scripts
Creates normalized relational tables including:
Table: Role
CREATE TABLE Role (
    RoleID INT PRIMARY KEY IDENTITY(1,1),
    RoleName VARCHAR(100) NOT NULL
);

Table: Location
CREATE TABLE Location (
    LocationID INT PRIMARY KEY IDENTITY(1,1),
    LocationName VARCHAR(100) NOT NULL
);

Table: Employee
CREATE TABLE Employee (
    EmployeeID INT PRIMARY KEY IDENTITY(1,1),
    Name VARCHAR(100),
    RoleID INT,
    LocationID INT,
    ExperienceYears DECIMAL(4,2),
    Compensation DECIMAL(18,2),
    IsActive BIT,
    FOREIGN KEY (RoleID) REFERENCES Role(RoleID),
    FOREIGN KEY (LocationID) REFERENCES Location(LocationID)
);

2. Stored Procedures
Includes procedures such as:
Procedure: FilterEmployees
CREATE PROCEDURE FilterEmployees
    @RoleID INT = NULL,
    @LocationID INT = NULL,
    @IncludeInactive BIT = 0
AS
BEGIN
    SELECT e.Name, r.RoleName, l.LocationName, e.Compensation, e.IsActive
    FROM Employee e
    JOIN Role r ON e.RoleID = r.RoleID
    JOIN Location l ON e.LocationID = l.LocationID
    WHERE (@RoleID IS NULL OR e.RoleID = @RoleID)
      AND (@LocationID IS NULL OR e.LocationID = @LocationID)
      AND (e.IsActive = 1 OR @IncludeInactive = 1)
END;

Procedure: CalculateAverageCompensation
CREATE PROCEDURE CalculateAverageCompensation
    @LocationID INT = NULL
AS
BEGIN
    SELECT l.LocationName, AVG(e.Compensation) AS AvgCompensation
    FROM Employee e
    JOIN Location l ON e.LocationID = l.LocationID
    WHERE (@LocationID IS NULL OR e.LocationID = @LocationID)
    GROUP BY l.LocationName
END;

Procedure: GetEmployeesByExperienceRange
CREATE PROCEDURE GetEmployeesByExperienceRange
AS
BEGIN
    SELECT 
        CASE 
            WHEN ExperienceYears < 1 THEN '0-1'
            WHEN ExperienceYears BETWEEN 1 AND 2 THEN '1-2'
            WHEN ExperienceYears BETWEEN 2 AND 5 THEN '2-5'
            ELSE '5+'
        END AS ExperienceRange,
        COUNT(*) AS EmployeeCount
    FROM Employee
    GROUP BY 
        CASE 
            WHEN ExperienceYears < 1 THEN '0-1'
            WHEN ExperienceYears BETWEEN 1 AND 2 THEN '1-2'
            WHEN ExperienceYears BETWEEN 2 AND 5 THEN '2-5'
            ELSE '5+'
        END
END;

Procedure: ApplyGlobalIncrement
CREATE PROCEDURE ApplyGlobalIncrement
    @IncrementPercent DECIMAL(5,2)
AS
BEGIN
    UPDATE Employee
    SET Compensation = Compensation * (1 + @IncrementPercent / 100)
END;

3. Sample Data Import
Provided Excel file is parsed and inserted into the database via an import utility.

How to Run the Application

1. Clone this repository**
    bash
   git clone https://github.com/yourusername/employee-compensation-forecasting.git
   

2. Set up the SQL Server Database**
Run `/SQLScripts/TableCreation.sql` and `/SQLScripts/StoredProcedures.sql` in SQL Server Management Studio (SSMS)

3. Import Excel Data**
Use the provided import tool or run the insert script after converting the Excel data to CSV

4. Build & Run the C# Application**
   * Open the solution in Visual Studio
   * Update the connection string in `App.config` or `Web.config`
   * Run the application

User Stories & Features
public List<EmployeeDto> FilterEmployees(int? roleId, int? locationId, bool includeInactive)
{
    var result = new List<EmployeeDto>();
    using (var conn = new SqlConnection("YourConnectionString"))
    using (var cmd = new SqlCommand("FilterEmployees", conn))
    {
        cmd.CommandType = CommandType.StoredProcedure;
        cmd.Parameters.AddWithValue("@RoleID", (object)roleId ?? DBNull.Value);
        cmd.Parameters.AddWithValue("@LocationID", (object)locationId ?? DBNull.Value);
        cmd.Parameters.AddWithValue("@IncludeInactive", includeInactive);
        conn.Open();
        var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            result.Add(new EmployeeDto
            {
                Name = reader.GetString(0),
                Role = reader.GetString(1),
                Location = reader.GetString(2),
                Compensation = reader.GetDecimal(3),
                IsActive = reader.GetBoolean(4)
            });
        }
    }
    return result;
}

public void ApplyIncrement(decimal percent)
{
    using (var conn = new SqlConnection("YourConnectionString"))
    using (var cmd = new SqlCommand("ApplyGlobalIncrement", conn))
    {
        cmd.CommandType = CommandType.StoredProcedure;
        cmd.Parameters.AddWithValue("@IncrementPercent", percent);
        conn.Open();
        cmd.ExecuteNonQuery();
    }
}

public void ExportToCsv(List<EmployeeDto> data, string filePath)
{
    using (var sw = new StreamWriter(filePath))
    {
        sw.WriteLine("Name,Role,Location,Experience,Compensation,Status");
        foreach (var e in data)
        {
            sw.WriteLine($"{e.Name},{e.Role},{e.Location},{e.ExperienceYears},{e.Compensation},{e.Status}");
        }
    }
}

📁 Repository Structure

📁 EmployeeCompensationApp/
├── 📁 SQLScripts/
│   ├── TableCreation.sql
│   └── StoredProcedures.sql
├── 📁 UI/                 # C# frontend project
│   ├── Forms / Pages
│   └── Charts / Export
├── 📁 DataAccess/
│   └── DBConnection.cs, DAL.cs
├── 📁 Models/
│   └── Employee.cs, Role.cs, etc.
├── 📁 Services/
│   └── CompensationService.cs
├── Program.cs / Startup.cs
└── README.md
