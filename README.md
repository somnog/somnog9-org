# SOMNOG9 Event Management System



## System Architecture

The Somnok Event Management System follows a microservices architecture pattern where each service is independently deployable and responsible for a specific domain. A centralized NextJS frontend consumes APIs from all microservices.

```
┌─────────────────────────────────────────────────────────────┐
│                     NextJS Frontend                          │
│              (Event Discovery, Booking, Profile)             │
└────────────────┬────────────────┬────────────────────────────┘
                 │                │
        ┌────────┴────────┐       │
        │                 │       │
   ┌────▼────┐    ┌──────▼──────┐ │
   │   Auth  │    │   Event     │ │
   │ Service │    │  Service    │ │
   └────┬────┘    └──────┬──────┘ │
        │                │        │
        └────────┬───────┤        │
                 │       │        │
          ┌──────▼──┐    │   ┌────▼────────┐
          │ Booking │◄───┤   │ Notification│
          │ Service │    │   │  Service    │
          └─────────┘    │   └─────────────┘
                         │
                    ┌────▼──────────┐
                    │ Speaker &     │
                    │ Schedule      │
                    │ Service       │
                    └───────────────┘
```

---

## Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Backend Framework | NestJS | Scalable, modular backend framework |
| ORM | Prisma | Type-safe database access |
| Database | PostgreSQL | Relational data storage |
| Frontend | Next.js | React-based frontend framework |
| API Communication | REST | Service-to-service communication |
| Runtime | Node.js | JavaScript runtime environment |

---

## Microservices Overview

### Service Assignment by Group

| Service | Group | Responsibility |
|---------|-------|-----------------|
| Auth Service | Group 1 | User authentication, profile management, access control |
| Event Service | Group 2 | Event creation, track management, event operations |
| Booking Service | Group 3 | Event enrollment, event discovery, user bookings |
| Notification Service | Group 4 | Email notifications, event updates, real-time messaging |
| Speaker & Schedule Service | Group 5 | Speaker/facilitator management, event scheduling |

---

## Service Specifications

### 1. Auth Service (Group 1)

**Description**
The Auth Service is the identity and access control backbone of the system. It handles user authentication, profile management, role-based access control (RBAC), and credential verification.

**Core Responsibilities**
- User registration and login
- JWT token generation and validation
- User profile management (bio, avatar, preferences)
- Role assignment (Student, Facilitator, Admin)
- Access control verification
- Password management and reset

**Database Schema**

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  password  String   // hashed
  firstName String
  lastName  String
  avatar    String?
  bio       String?
  role      Role     @default(STUDENT)
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  bookings  Booking[]
  reviews   Review[]
}

enum Role {
  STUDENT
  FACILITATOR
  ADMIN
}
```

**API Endpoints**

```
POST   /auth/register          - Register new user
POST   /auth/login             - User login (returns JWT)
POST   /auth/refresh-token     - Refresh expired token
POST   /auth/logout            - Logout user
GET    /auth/me                - Get current user profile
PATCH  /auth/profile           - Update user profile
POST   /auth/password-reset    - Initiate password reset
PATCH  /auth/password          - Change password
```

**Example Request/Response**

```json
// POST /auth/register
{
  "email": "student@somnog9.com",
  "password": "SecurePass123",
  "firstName": "Abdi",
  "lastName": "Hassan"
}

// Response (201)
{
  "id": "user_123",
  "email": "student@somnog9.com",
  "firstName": "Abdi",
  "lastName": "Hassan",
  "role": "STUDENT",
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

---

### 2. Event Service (Group 2)

**Description**
The Event Service manages the creation, configuration, and lifecycle of events and their associated tracks. It defines what events are happening at Somnok and how they're organized by tracks.

**Core Responsibilities**
- Create and manage events
- Define event tracks (Software Development, Design, Business, etc.)
- Manage sub-tracks within events
- Set event metadata (date, time, location, capacity)
- Track event status (planned, active, completed)
- Handle event cancellation and rescheduling

**Database Schema**

```prisma
model Event {
  id          String   @id @default(cuid())
  title       String
  description String
  startDate   DateTime
  endDate     DateTime
  location    String
  capacity    Int
  status      EventStatus @default(PLANNED)
  createdBy   String
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  tracks      Track[]
  bookings    Booking[]
  speakers    Speaker[]
}

model Track {
  id          String   @id @default(cuid())
  name        String   // e.g., "Software Development"
  eventId     String
  event       Event    @relation(fields: [eventId], references: [id])
  description String?
  capacity    Int
  
  sessions    Session[]
}

enum EventStatus {
  PLANNED
  ACTIVE
  COMPLETED
  CANCELLED
}
```

**API Endpoints**

```
POST   /events                  - Create new event
GET    /events                  - List all events
GET    /events/:id              - Get event details
PATCH  /events/:id              - Update event
DELETE /events/:id              - Cancel event
GET    /events/:id/tracks       - Get all tracks for event
POST   /events/:id/tracks       - Create track
PATCH  /tracks/:id              - Update track
```

**Example Request/Response**

```json
// POST /events
{
  "title": "SOMNOG9 2026 - Software Development Track",
  "description": "Annual technology seminar in Somalia",
  "startDate": "2026-12-01T08:00:00Z",
  "endDate": "2026-12-03T18:00:00Z",
  "location": "Mogadishu Convention Center",
  "capacity": 500
}

// Response (201)
{
  "id": "event_456",
  "title": "SOMNOG9 2026 - Software Development Track",
  "status": "PLANNED",
  "tracks": []
}
```

---

### 3. Booking Service (Group 3)

**Description**
The Booking Service manages student enrollment in events and sessions. It handles seat availability, booking confirmations, and provides event discovery/search functionality.

**Core Responsibilities**
- Allow students to book/enroll in events
- Manage seat availability and capacity
- Search and filter events
- Track booking status (confirmed, waitlisted, cancelled)
- Provide personalized event recommendations
- Handle booking cancellations

**Database Schema**

```prisma
model Booking {
  id        String    @id @default(cuid())
  userId    String
  user      User      @relation(fields: [userId], references: [id])
  eventId   String
  event     Event     @relation(fields: [eventId], references: [id])
  trackId   String
  status    BookingStatus @default(CONFIRMED)
  bookedAt  DateTime  @default(now())
}

enum BookingStatus {
  CONFIRMED
  WAITLISTED
  CANCELLED
  COMPLETED
}
```

**API Endpoints**

```
POST   /bookings                 - Create new booking
GET    /bookings/my-bookings     - Get user's bookings
GET    /bookings/:id             - Get booking details
PATCH  /bookings/:id             - Update booking status
DELETE /bookings/:id             - Cancel booking
GET    /events/search            - Search events by keyword/track
GET    /events/:id/availability  - Check seat availability
POST   /events/:id/check-status  - Check if user already booked
```

**Example Request/Response**

```json
// POST /bookings
{
  "userId": "user_123",
  "eventId": "event_456",
  "trackId": "track_789"
}

// Response (201)
{
  "id": "booking_101",
  "userId": "user_123",
  "eventId": "event_456",
  "status": "CONFIRMED",
  "bookedAt": "2026-11-15T10:30:00Z"
}

// GET /events/search?keyword=microservices&track=software-dev
{
  "results": [
    {
      "id": "event_456",
      "title": "Microservices Architecture Workshop",
      "track": "Software Development",
      "availableSeats": 45
    }
  ]
}
```

---

### 4. Notification Service (Group 4)

**Description**
The Notification Service is responsible for all user communications. It sends email notifications for bookings, event updates, reminders, and system alerts to keep users informed throughout their journey.

**Core Responsibilities**
- Send confirmation emails for bookings
- Send event reminder emails
- Publish updates when events change
- Send facilitator messages to students
- Handle notification preferences
- Log all notifications for audit purposes

**Database Schema**

```prisma
model Notification {
  id        String   @id @default(cuid())
  userId    String
  type      NotificationType
  subject   String
  message   String
  email     String
  status    NotificationStatus @default(PENDING)
  sentAt    DateTime?
  createdAt DateTime @default(now())
  
  eventId   String?
  bookingId String?
}

model NotificationTemplate {
  id      String @id @default(cuid())
  name    String
  subject String
  body    String
  type    NotificationType
}

enum NotificationType {
  BOOKING_CONFIRMED
  BOOKING_CANCELLED
  EVENT_REMINDER
  EVENT_UPDATED
  EVENT_CANCELLED
  MESSAGE_FROM_FACILITATOR
  SCHEDULE_CHANGE
}

enum NotificationStatus {
  PENDING
  SENT
  FAILED
  BOUNCED
}
```

**API Endpoints**

```
POST   /notifications/send              - Send notification
GET    /notifications                   - List user notifications
PATCH  /notifications/:id/mark-read     - Mark notification as read
POST   /notifications/email-trigger     - Trigger email send
POST   /notifications/templates         - Create notification template
GET    /notifications/preferences/:id   - Get user preferences
PATCH  /notifications/preferences/:id   - Update preferences
```

**Example Request/Response**

```json
// POST /notifications/send
{
  "userId": "user_123",
  "type": "BOOKING_CONFIRMED",
  "email": "student@somnog9.com",
  "subject": "Booking Confirmation - SOMNOG9 2026",
  "message": "Your booking for the Software Development track has been confirmed."
}

// Response (201)
{
  "id": "notif_999",
  "status": "PENDING",
  "createdAt": "2026-11-15T10:35:00Z"
}
```

---

### 5. Speaker & Schedule Service (Group 5)

**Description**
The Speaker & Schedule Service manages facilitator/speaker information and event scheduling. It maintains speaker bios, session details, and coordinates the overall event timeline.

**Core Responsibilities**
- Manage speaker/facilitator profiles
- Create and schedule sessions
- Assign speakers to sessions
- Handle session timings and rooms
- Manage session capacity
- Track speaker availability

**Database Schema**

```prisma
model Speaker {
  id          String   @id @default(cuid())
  userId      String   @unique
  bio         String?
  expertise   String   // e.g., "Microservices, Cloud Architecture"
  avatar      String?
  company     String?
  eventId     String
  event       Event    @relation(fields: [eventId], references: [id])
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  sessions    Session[]
}

model Session {
  id          String   @id @default(cuid())
  title       String
  description String?
  speakerId   String
  speaker     Speaker  @relation(fields: [speakerId], references: [id])
  eventId     String
  event       Event    @relation(fields: [eventId], references: [id])
  trackId     String
  startTime   DateTime
  endTime     DateTime
  location    String
  capacity    Int
  sessionType SessionType @default(WORKSHOP)
}

enum SessionType {
  KEYNOTE
  WORKSHOP
  PANEL_DISCUSSION
  BREAKOUT
  NETWORKING
}
```

**API Endpoints**

```
POST   /speakers                 - Register speaker
GET    /speakers                 - List all speakers
GET    /speakers/:id             - Get speaker details
PATCH  /speakers/:id             - Update speaker profile
POST   /sessions                 - Create new session
GET    /sessions                 - List all sessions
GET    /sessions/:id             - Get session details
PATCH  /sessions/:id             - Update session
GET    /events/:id/schedule      - Get event schedule
POST   /speakers/:id/availability - Set speaker availability
```

**Example Request/Response**

```json
// POST /speakers
{
  "userId": "user_500",
  "bio": "Senior Software Architect with 10 years experience",
  "expertise": "Microservices, Cloud Native, DevOps",
  "company": "TechCorp Solutions",
  "eventId": "event_456"
}

// Response (201)
{
  "id": "speaker_123",
  "userId": "user_500",
  "expertise": "Microservices, Cloud Native, DevOps",
  "eventId": "event_456"
}

// GET /events/event_456/schedule
{
  "event": "SOMNOG9 2026",
  "sessions": [
    {
      "id": "session_1",
      "title": "Microservices Architecture",
      "speaker": "Ahmed Mohamed",
      "startTime": "2026-12-01T09:00:00Z",
      "endTime": "2026-12-01T10:30:00Z",
      "location": "Hall A",
      "capacity": 100
    }
  ]
}
```

---

## Service Communication Flow

Services communicate via REST APIs. Here's how the key workflows happen:

### Booking Flow
```
1. User (NextJS Frontend) → Auth Service: Login
2. User → Booking Service: Search events
3. Booking Service → Event Service: Get event details
4. User → Booking Service: Create booking
5. Booking Service → Auth Service: Verify user role
6. Booking Service → Notification Service: Send confirmation email
```

### Event Creation Flow
```
1. Facilitator (NextJS Frontend) → Auth Service: Verify admin role
2. Facilitator → Event Service: Create event with tracks
3. Event Service → Speaker & Schedule Service: Create sessions
4. Speaker & Schedule Service → Auth Service: Get facilitator details
5. Facilitator → Notification Service: Broadcast event announcement
```

### Session Attendance Flow
```
1. Student → Booking Service: View booked sessions
2. Booking Service → Speaker & Schedule Service: Get session details
3. Booking Service → Auth Service: Get student profile
4. After Event → Notification Service: Send attendance/certificate email
```

---

## Frontend

A single NextJS application serves as the user interface for the entire system. It consumes APIs from all microservices and handles authentication, event browsing, bookings, and user interactions.

**Environment Configuration**
```env
NEXT_PUBLIC_AUTH_API=http://localhost:3001
NEXT_PUBLIC_EVENT_API=http://localhost:3002
NEXT_PUBLIC_BOOKING_API=http://localhost:3003
NEXT_PUBLIC_NOTIFICATION_API=http://localhost:3004
NEXT_PUBLIC_SPEAKER_API=http://localhost:3005
```

Frontend implementation details and component structure will be discussed during development.

---

## Getting Started

### Prerequisites
- Node.js 16+ 
- PostgreSQL 12+
- npm or yarn

### Installation

1. **Clone repositories** (one per service)
```bash
git clone https://github.com/somnog9/auth-service.git
git clone https://github.com/somnog9/event-service.git
git clone https://github.com/somnog9/booking-service.git
git clone https://github.com/somnog9/notification-service.git
git clone https://github.com/somnog9/speaker-schedule-service.git
git clone https://github.com/somnog9/frontend.git
```

2. **Install dependencies for each service**
```bash
cd auth-service
npm install
```

3. **Set up environment files** (.env)
```env
DATABASE_URL=postgresql://user:password@localhost:5432/somnog9_auth
JWT_SECRET=your_jwt_secret
MAIL_SERVICE=gmail
MAIL_USER=noreply@somnog9.com
MAIL_PASSWORD=your_app_password
```

4. **Run Prisma migrations**
```bash
npx prisma migrate dev
```

5. **Start the service**
```bash
npm run start:dev
```

6. **Start all services** on different ports:
- Auth Service: `http://localhost:3001`
- Event Service: `http://localhost:3002`
- Booking Service: `http://localhost:3003`
- Notification Service: `http://localhost:3004`
- Speaker & Schedule Service: `http://localhost:3005`
- NextJS Frontend: `http://localhost:3000`

---

## Deployment

Each microservice can be deployed independently using Docker and container orchestration platforms (Kubernetes, Docker Swarm, etc.).

**Docker Build**
```bash
docker build -t somnog9/auth-service:1.0.0 .
docker push somnog9/auth-service:1.0.0
```

**Environment Variables for Production**
- Database credentials (secure vault)
- JWT signing keys
- Email service credentials
- API gateway URL
- CORS origins

---

## API Gateway (Optional)

For production, consider implementing an API Gateway (Kong, Ambassador, AWS API Gateway) to:
- Route requests to appropriate services
- Handle rate limiting
- Manage API versioning
- Provide centralized authentication

---

## Notes for Development Teams

### Code Standards
- Follow NestJS best practices
- Write unit tests for all services (Jest)
- Use Prisma migrations for schema changes
- Implement proper error handling with consistent response formats
- Add API documentation comments

### Communication Between Teams
- Maintain clear API contracts
- Use semantic versioning for services
- Update this README when adding/changing endpoints
- Coordinate database schema changes

### Key Principles
- Each service owns its database (no cross-service database queries)
- Services communicate via REST APIs only
- All responses should follow consistent JSON structure
- Implement proper authentication/authorization checks

---

## Support & Questions

For questions about the architecture or service specifications, reach out to the facilitators of the software development track.

---

**Last Updated:** November 2026
**Version:** 1.0.0
**Maintained by:** SOMNOG9 Development Team
