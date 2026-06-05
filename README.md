# Meeting Room Booking API

## Overview

This project is a Meeting Room Booking REST API developed using .NET 8 and SQL Server. The application allows employees to book meeting rooms while ensuring that no room can be double-booked for overlapping time periods.

The API supports room management, booking creation and cancellation, room availability search, and utilization reporting.

---

## Technology Stack

* .NET 8 Web API
* Entity Framework Core
* SQL Server
* xUnit for Unit Testing

---

## Features

* List available meeting rooms
* Filter rooms by minimum capacity
* Create room bookings
* Prevent overlapping bookings for the same room
* View bookings for a room on a specific date
* Cancel existing bookings
* Find the earliest available time slot within a given window
* Generate room utilization reports using raw SQL
* Input validation and proper HTTP status codes
* Unit tests for critical business logic

---

## Database Design

### Room

| Column   | Type     |
| -------- | -------- |
| Id       | int      |
| Name     | nvarchar |
| Capacity | int      |
| Location | nvarchar |

### Booking

| Column    | Type     |
| --------- | -------- |
| Id        | int      |
| RoomId    | int      |
| Title     | nvarchar |
| StartTime | datetime |
| EndTime   | datetime |
| BookedBy  | nvarchar |

### Constraints

* Primary Keys on Room and Booking
* Foreign Key from Booking.RoomId to Room.Id
* Required fields enforced through database constraints
* Validation to ensure EndTime is greater than StartTime

### Index

```sql
CREATE INDEX IX_Bookings_RoomId_StartTime_EndTime
ON Bookings(RoomId, StartTime, EndTime);
```

Reason:

This index improves the performance of overlap checks, room booking searches, and availability calculations by allowing SQL Server to efficiently filter bookings based on room and time range.

---

## Running the Application

### 1. Clone Repository

```bash
git clone <repository-url>
```

### 2. Update Connection String

Modify the connection string in:

```json
appsettings.json
```

### 3. Apply Database Migrations

```bash
dotnet ef database update
```

### 4. Run the Application

```bash
dotnet run
```

Swagger UI will be available at:

```text
https://localhost:<port>/swagger
```

---

## API Endpoints

### Rooms

| Method | Endpoint              | Description              |
| ------ | --------------------- | ------------------------ |
| GET    | /rooms                | Get all rooms            |
| GET    | /rooms?minCapacity=10 | Filter rooms by capacity |

### Bookings

| Method | Endpoint                             | Description             |
| ------ | ------------------------------------ | ----------------------- |
| POST   | /bookings                            | Create booking          |
| GET    | /rooms/{id}/bookings?date=YYYY-MM-DD | Get bookings for a room |
| DELETE | /bookings/{id}                       | Cancel booking          |

### Availability

| Method | Endpoint      | Description                  |
| ------ | ------------- | ---------------------------- |
| GET    | /availability | Find earliest available slot |

Example:

```text
/availability?roomId=1&durationMinutes=30&from=2026-06-05T09:00:00&to=2026-06-05T18:00:00
```

### Utilization

| Method | Endpoint                           | Description                 |
| ------ | ---------------------------------- | --------------------------- |
| GET    | /rooms/utilization?date=YYYY-MM-DD | Get room utilization report |

---

## Raw SQL Query

The utilization endpoint uses a handwritten SQL query as required.

```sql
SELECT
    r.Id,
    r.Name,
    COUNT(b.Id) AS BookingCount,
    ISNULL(SUM(DATEDIFF(MINUTE, b.StartTime, b.EndTime)),0) AS TotalBookedMinutes
FROM Rooms r
LEFT JOIN Bookings b
    ON r.Id = b.RoomId
    AND CAST(b.StartTime AS DATE) = @Date
GROUP BY r.Id, r.Name
ORDER BY r.Name;
```

This query returns:

* Room Name
* Number of Bookings
* Total Booked Minutes

for a given date.

---

## Concurrency Handling

To prevent double booking, booking creation is executed inside a database transaction using the Serializable isolation level.

The overlap check and booking insertion occur within the same transaction. This guarantees that two concurrent requests cannot both pass the overlap validation and insert conflicting bookings for the same room.

### Limitation

Serializable transactions can reduce throughput under heavy concurrent load because transactions may need to wait for locks to be released. For a larger distributed system, additional strategies such as distributed locking or database-specific exclusion constraints may be considered.

---

## Availability Algorithm

The availability endpoint identifies the earliest available slot of a specified duration within a given time window.

Approach:

1. Retrieve all bookings for the room within the requested window.
2. Sort bookings by StartTime.
3. Merge overlapping or adjacent bookings.
4. Scan gaps between bookings.
5. Return the earliest gap that satisfies the requested duration.
6. Return No Content (204) if no suitable slot exists.

### Time Complexity

```text
O(n log n)
```

Sorting dominates the overall execution time.

---

## Validation Rules

The API validates the following:

* EndTime must be greater than StartTime.
* Duration must be greater than zero.
* Room must exist.
* Overlapping bookings are rejected.
* Invalid requests return appropriate HTTP status codes and error messages.

---

## Unit Tests

Unit tests have been implemented for the most critical business rules.

### Overlap Validation Tests

* Reject overlapping bookings.
* Allow non-overlapping bookings.
* Allow bookings that start exactly when another booking ends.

### Availability Logic Tests

* Find gap before first booking.
* Find gap between bookings.
* Find gap after last booking.
* Handle adjacent bookings correctly.
* Return no result when no suitable gap exists.

---

# Written Answers

## Question 1: Concurrency

I implemented concurrency protection using a database transaction with Serializable isolation level.

The overlap validation and booking creation execute within the same transaction. This ensures that while one booking request is checking for conflicts, another concurrent request cannot insert a conflicting booking for the same room. As a result, double booking is prevented.

One limitation is that Serializable transactions can reduce performance under high concurrency because of locking and blocking between transactions.

---

## Question 2: Scale

As the Bookings table grows to approximately one million rows, the overlap-check query and availability search query are likely to degrade first because they frequently filter data by RoomId and time range.

To improve performance, I created an index on:

```sql
(RoomId, StartTime, EndTime)
```

This allows SQL Server to efficiently locate bookings for a specific room and time period instead of performing table scans.

For further scaling, partitioning historical booking data and introducing caching could be considered.

---

## Question 3: Trade-Off

Given the time-boxed nature of the exercise, I focused on correctness of business logic, concurrency handling, API design, and test coverage.

I intentionally did not implement:

* Authentication and authorization
* Audit logging
* API versioning
* Pagination
* Structured logging and monitoring
* Docker deployment

With additional time, I would add authentication, centralized exception handling, integration tests, logging, monitoring, and deployment support to make the application production-ready.

---

## Assumptions

* Booking times are stored in UTC.
* A booking ending at a specific time and another booking starting at the same time are considered valid and non-overlapping.
* Room names are not required to be unique.
* Users are identified by username strings only.
* Authentication is out of scope for this assignment.

---

## Conclusion

This solution demonstrates REST API development using .NET 8, relational database design, concurrency handling, raw SQL querying, algorithmic problem solving, and automated testing. The primary focus was correctness, maintainability, and ensuring that room bookings remain consistent even under concurrent requests.
