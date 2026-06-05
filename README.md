# Room Booking Service

A simple .NET 8 Web API to manage meeting room bookings.

## Features

* Create room bookings
* Prevent overlapping bookings
* View room availability
* Delete bookings
* View room utilization reports

## Technologies Used

* ASP.NET Core 8
* Entity Framework Core
* SQL Server
* Swagger

## Setup

### Prerequisites

* .NET 8 SDK
* SQL Server

### Run the Project

```bash
dotnet restore
dotnet build
dotnet run
```

Swagger URL:

```text
http://localhost:5000/swagger
```

## Main APIs

### Get Rooms

```http
GET /rooms
```

### Create Booking

```http
POST /bookings
```

### Get Room Bookings

```http
GET /rooms/{id}/bookings
```

### Delete Booking

```http
DELETE /bookings/{id}
```

### Check Availability

```http
GET /availability
```

## Database

* Rooms Table
* Bookings Table
* SQL Server Stored Procedure for Utilization Report



-------------------------------------------------------
Interview Questions & Answers
1. How do you prevent double booking of a room?

Answer:
I use a SERIALIZABLE transaction while creating a booking. Before inserting a new booking, the system checks whether another booking already exists in the same time slot. If an overlap is found, the API returns a conflict response.

2. How do you detect overlapping bookings?

Answer:
The application checks whether:

existing.StartTime < newEndTime &&
existing.EndTime > newStartTime

If this condition is true, the bookings overlap.

3. Which database is used in the project?

Answer:
The project uses SQL Server as the database and Entity Framework Core 8 as the ORM for database operations.

4. What improvements would you make if you had more time?

Answer:
I would add:

JWT Authentication
Role-based Authorization
Audit Logging
Redis Caching
Pagination
Better Exception Handling and Logging
