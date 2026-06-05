Meeting Room Booking API

Overview

This project is a Meeting Room Booking REST API built using .NET 8 Web API and SQL Server. It allows users to create and manage meeting room bookings while preventing overlapping reservations for the same room.

Technology Used

* .NET 8 Web API
* Entity Framework Core
* SQL Server
* xUnit

Features

* List all rooms
* Filter rooms by minimum capacity
* Create bookings
* Prevent overlapping bookings
* View room bookings by date
* Cancel bookings
* Find earliest available slot
* Room utilization report using raw SQL
* Input validation and error handling

Database

Tables:

* Rooms
* Bookings

Index Used:
IX_Bookings_RoomId_StartTime_EndTime

Reason:
Improves performance for overlap checks and availability searches.

How to Run

1. Update the SQL Server connection string in appsettings.json
2. Run database migration:

dotnet ef database update

3. Run the application:

dotnet run

4. Open Swagger UI

Written Answers

1. Concurrency

I used a database transaction with Serializable isolation level. The overlap check and booking creation are performed within the same transaction, preventing two concurrent requests from creating overlapping bookings for the same room.

Limitation:
Serializable transactions may reduce performance when many users create bookings simultaneously.

2. Scale

As the Bookings table grows, overlap checking and availability searches will be affected first because they filter by RoomId and time range.

To improve performance, I added an index on:

(RoomId, StartTime, EndTime)

This helps SQL Server quickly locate relevant bookings.

3. Trade-Off

Due to the time limit, I focused on core functionality, concurrency handling, and unit testing.

I did not implement:

* Authentication
* Logging
* Pagination
* API Versioning
* Docker support

With additional time, I would add these features and increase test coverage.

Tests

Unit tests are included for:

* Booking overlap validation
* Availability calculation logic

Assumptions

* Booking times are stored in UTC.
* A booking ending at 10:00 and another starting at 10:00 are not considered overlapping.
* Authentication is out of scope for this task.
