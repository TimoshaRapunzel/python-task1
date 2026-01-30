# Student-Room Database Analysis

A Python application that loads student and room data from JSON files into a PostgreSQL database and performs analytical queries to generate insights about student distribution and demographics.

## Overview

This project demonstrates ETL (Extract, Transform, Load) principles by:
- **Extracting** data from JSON files containing student and room information
- **Transforming** the data using SQL analytics to derive meaningful insights
- **Loading** results into structured JSON or XML output formats

## Features

- **High-Performance Data Loading**: Uses PostgreSQL's `COPY` command for fast bulk data insertion
- **Stream Processing**: Leverages `ijson` for memory-efficient JSON parsing of large files
- **Transaction Management**: Implements context managers for safe database operations with automatic rollback on errors
- **Multiple Output Formats**: Supports both JSON and XML output formats
- **Comprehensive Analytics**: Runs 4 different analytical queries to extract insights

## Analytical Queries

The application performs the following analyses:

1. **Student Count by Room**: Counts students in each room, ordered by population
2. **Top 5 Rooms with Lowest Average Age**: Identifies rooms with the youngest average student age
3. **Top 5 Rooms with Biggest Age Difference**: Finds rooms with the largest age spread between oldest and youngest students (minimum 2 students)
4. **Mixed-Sex Rooms**: Lists all rooms that contain both male and female students

## Project Structure

```
python-task1/
│
├── main.py              # Entry point - orchestrates the entire workflow
├── config.py            # Database configuration settings
├── db_connector.py      # Database connection handler with context manager
├── data_services.py     # Data loading and SQL query execution logic
├── formatter.py         # Output formatting (JSON/XML) with decimal handling
│
├── students.json        # Input: Student data (id, name, birthday, room, sex)
├── rooms.json           # Input: Room data (id, name)
│
└── results.json/xml     # Output: Analysis results in selected format
```

## Technologies Used

- **Python 3.x**
- **PostgreSQL** - Relational database
- **psycopg2** - PostgreSQL adapter for Python
- **ijson** - Streaming JSON parser for handling large files
- **xml.etree.ElementTree** - XML generation

## Prerequisites

1. **Python 3.7+** installed
2. **PostgreSQL** server running locally
3. Required Python packages (install via pip):

```bash
pip install psycopg2-binary ijson
```

## Configuration

Edit `config.py` to match your PostgreSQL database settings:

```python
DB_CONFIG = {
    "host": "localhost",
    "database": "Task-1",
    "user": "postgres",
    "password": "your_password",
    "port": "5432"
}
```

## Database Setup

Create the required tables in your PostgreSQL database:

```sql
CREATE TABLE "Rooms" (
    "id" INTEGER PRIMARY KEY,
    "name" VARCHAR(255) NOT NULL
);

CREATE TABLE "Students" (
    "id" INTEGER PRIMARY KEY,
    "name" VARCHAR(255) NOT NULL,
    "birthday" DATE NOT NULL,
    "room" INTEGER REFERENCES "Rooms"("id"),
    "sex" CHAR(1) CHECK ("sex" IN ('M', 'F'))
);
```

## Usage

1. Ensure your PostgreSQL database is running and configured correctly
2. Place your `students.json` and `rooms.json` files in the project root
3. Run the application:

```bash
python main.py
```

4. When prompted, enter your desired output format:
   - Type `json` for JSON output
   - Type `xml` for XML output

5. Check the generated `results.json` or `results.xml` file for the analysis results

## Input Data Format

### rooms.json
```json
[
    {"id": 1, "name": "Room 101"},
    {"id": 2, "name": "Room 102"}
]
```

### students.json
```json
[
    {
        "id": 1,
        "name": "John Doe",
        "birthday": "2005-03-15",
        "room": 1,
        "sex": "M"
    }
]
```

## Output Format

### JSON Example
```json
{
    "student_count": [
        {"room_name": "Room 101", "students_count": 25}
    ],
    "lowest_avg_age": [
        {"room_name": "Room 102", "avg_age": 18.5}
    ]
}
```

### XML Example
```xml
<results>
    <student_count>
        <item>
            <room_name>Room 101</room_name>
            <students_count>25</students_count>
        </item>
    </student_count>
</results>
```

## Key Components

### DatabaseConnector
Implements Python context manager protocol (`__enter__`, `__exit__`) for:
- Automatic connection management
- Transaction commit on success
- Automatic rollback on errors
- Safe resource cleanup

### DataService
- **load_data_from_json()**: Streams JSON data and bulk loads into PostgreSQL using `COPY`
- **run_analysis_queries()**: Executes all analytical SQL queries and returns structured results
- **_prepare_buffer()**: Converts Python data to tab-delimited format for `COPY` command

### OutputFormatter
- **format()**: Routes to appropriate formatter based on user selection
- **_to_json()**: Handles decimal serialization for JSON output
- **_to_xml()**: Builds hierarchical XML structure from query results