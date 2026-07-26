# MMU Event Management System

A desktop event management portal built with Java Swing for Multimedia University (MMU). It allows students to browse and register for campus events, and lets management/admin staff create, update, and monitor events, registrations, and users — all backed by a lightweight JSON file store (no external database required).

## Features

**Student / Staff**
- Browse all active (non-cancelled) events in a card-based dashboard
- Register for an event with personal details (name, student ID, contact, email, dietary preference, transportation)
- Automatic fee calculation with early-bird pricing, transportation fees, and discount breakdown
- View "My Bookings" with full registration and payment details
- Registration is automatically disabled once an event is full or closed

**Management**
- Create new events (venue, schedule, capacity, fees, early-bird pricing, food/transport provisioning)
- Update existing events
- Open/close registration per event
- Cancel events
- View and export the participant list for an event to CSV

**System Admin**
- All management capabilities
- View events created by any management user (management users only see their own)
- User management: create and delete user accounts, assign roles (student, management, admin)

## Tech Stack

- **Language:** Java (Swing/AWT for the GUI)
- **JSON handling:** [org.json](https://github.com/stleary/JSON-java) (`lib/json-20240303.jar`)
- **Persistence:** Flat JSON files under `data/` (no database/server required)

## Project Structure

```
.
├── MainPage.java              # Entry point — role selection (Student/Staff, Management, System Admin)
├── LoginForm.java             # Login screen, validates credentials against data/users.json
├── SignUpForm.java            # Account creation
│
├── StudentDashboard.java      # Student home — lists active events as cards
├── RegistrationForm.java      # Event registration form (personal + dietary/transport info)
├── FeeCalculationPage.java    # Computes and displays payment breakdown, saves the booking
├── MyBookingsPage.java        # Lists the logged-in student's bookings
├── BookingDetailsPage.java    # Detailed view of a single booking + payment breakdown
│
├── ManagementDashboard.java   # Management/Admin home — lists events (own events, or all for admin)
├── CreateEventForm.java       # Two-part wizard to create a new event
├── UpdateEventForm.java       # Two-part wizard to edit an existing event
├── ParticipantListPage.java   # Table of registrants for an event, exportable to CSV
├── UserManagementPage.java    # Admin-only: create/delete user accounts
│
├── Event.java                 # Event model (POJO + JSON (de)serialization)
├── EventManager.java          # Static helpers for CRUD on data/events.json
├── Booking.java                # Booking model (POJO + JSON (de)serialization)
├── BookingManager.java        # Static helpers for querying data/registrations.json
│
├── data/
│   ├── users.json              # User accounts (username, password, fullname, role)
│   ├── events.json             # All events and their participant lists
│   └── registrations.json      # Full registration/booking records
│
└── lib/
    └── json-20240303.jar       # org.json library
```

## Architecture Notes

- **No database** — all state is persisted directly to JSON files in `data/` using the `org.json` library. Each screen reads the relevant file on load and rewrites it on save.
- **Role-based access** — a single `LoginForm` is reused for all three roles (`student`, `management`, `admin`); the role string gates which dashboard and actions are shown.
- **Swing navigation pattern** — each page is its own `JFrame`; navigating forward disposes the current frame and constructs the next one, passing along the data it needs (username, role, event, booking, etc.).
- **Model classes** (`Event`, `Booking`) wrap `JSONObject` construction/parsing (`toJSON` / `fromJSON`), while `EventManager` / `BookingManager` centralize file I/O for their respective entities.
- **Fee logic** — early-bird pricing applies while `participants.length < early_bird_limit`; transportation fee is added only if the event provides transport and the student opts in.

## Getting Started

### Prerequisites
- JDK 8 or later
- The bundled `org.json` jar in `lib/`

### Compile
```bash
javac -cp lib/json-20240303.jar -d out *.java
```

### Run
```bash
java -cp "out:lib/json-20240303.jar" MainPage
```
On Windows, use a semicolon instead of a colon in the classpath:
```bash
java -cp "out;lib/json-20240303.jar" MainPage
```

The app must be run from the project root so it can find the `data/` folder.

### Default accounts
See `data/users.json` for seeded accounts across the `student`, `management`, and `admin` roles.

## Data Model (JSON)

**`users.json`** — `username`, `password`, `fullname`, `role`

**`events.json`** — `event_id`, `name`, `type`, `venue`, `date`, `start_time`, `end_time`, `capacity`, `has_fee`, `base_price`, `early_bird_price`, `early_bird_limit`, `transport_fee`, `provides_food`, `provides_transportation`, `registration_open`, `cancelled`, `createdBy`, `participants`

**`registrations.json`** — `username`, `event_id`, `fullName`, `studentId`, `contact`, `email`, `dietary`, `transport`, `paid_amount`, `booking_date`

## Known Limitations

- Passwords are stored in plain text in `users.json` — not suitable for production use.
- No concurrent-access protection on the JSON files.
- Payment is manual/offline (bank transfer + WhatsApp receipt confirmation), not integrated with a payment gateway.
