Meeting Room Booking API
Overview

This project is a Meeting Room Booking REST API built using .NET 8 Web API and SQL Server.

The system allows employees to:

View available meeting rooms
Create room bookings
Prevent overlapping bookings
View room bookings by date
Cancel bookings
Find room availability within a given time window
View room utilization statistics
Technology Stack
.NET 8 Web API
Entity Framework Core
SQL Server
xUnit (Unit Testing)
Database

Database used: SQL Server

Entities
Room
Id
Name
Capacity
Location
Booking
Id
RoomId
Title
StartTime
EndTime
BookedBy
Index
CREATE INDEX IX_Bookings_RoomId_StartTime_EndTime
ON Bookings(RoomId, StartTime, EndTime);

Reason:
This index improves performance when checking booking overlaps and searching bookings for a specific room and time range.

Running the Project
Update Connection String

Update the connection string in:

appsettings.json
Apply Migration
dotnet ef database update
Run Application
dotnet run

API Swagger will be available at:

https://localhost:<port>/swagger
API Endpoints
Method	Endpoint	Description
GET	/rooms	Get all rooms
POST	/bookings	Create booking
GET	/rooms/{id}/bookings	Get bookings by room and date
DELETE	/bookings/{id}	Cancel booking
GET	/availability	Find earliest available slot
GET	/rooms/utilization	Room utilization report
Utilization Query (Raw SQL)
SELECT
    r.Id,
    r.Name,
    COUNT(b.Id) AS BookingCount,
    ISNULL(SUM(DATEDIFF(MINUTE, b.StartTime, b.EndTime)), 0) AS TotalBookedMinutes
FROM Rooms r
LEFT JOIN Bookings b
    ON r.Id = b.RoomId
    AND CAST(b.StartTime AS DATE) = @Date
GROUP BY r.Id, r.Name
ORDER BY r.Name;
Written Answers
1. Concurrency

To prevent double booking, I used a database transaction with a high isolation level (Serializable).

When a booking request arrives, the system checks for overlapping bookings and inserts the new booking within the same transaction. The Serializable isolation level ensures that concurrent transactions cannot insert overlapping bookings for the same room while the check is being performed.

Limitation:

Serializable transactions can reduce throughput under heavy load because transactions may wait for locks to be released. This approach works well for a small to medium booking system but may require a more scalable strategy in a highly distributed environment.

2. Scale

With approximately 1 million booking records, the overlap-check query and availability search query would be the first areas to experience performance degradation because they frequently filter bookings by room and time range.

To address this, I created an index on:

(RoomId, StartTime, EndTime)

This allows SQL Server to quickly locate bookings for a specific room and time interval without scanning the entire table.

For larger scale systems, partitioning historical booking data or introducing caching for frequently queried availability data could also be considered.

3. Trade-Off Made

Due to the time constraint, I focused on implementing the core business requirements and correctness of the booking logic.

I intentionally did not implement:

Authentication and authorization
Audit logging
Global exception middleware
Pagination for large result sets
API versioning

With an additional day, I would add authentication, centralized exception handling, API versioning, and more comprehensive integration tests to improve production readiness.

Testing

Unit tests have been implemented for:

Booking overlap validation.
Availability search logic.

These tests verify critical business rules such as:

Preventing overlapping bookings.
Allowing back-to-back bookings.
Finding the earliest available slot.
Handling adjacent bookings correctly.
Returning no availability when no valid gap exists.
