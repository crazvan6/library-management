Library Management System - MVP Specs
=====================================

## Business requirements
1. Users can register/login.
2. Roles matter: STUDENT, LIBRARIAN, ADMIN see/do different things.
3. Books must have categories, quantities, and availability kept accurate.
4. Searching books must allow filters (title/author/category/availability) with paging/sorting.
5. Students can place/cancel reservations; holds expire automatically if unused.
6. Checkouts are done by librarians/admins; due dates are set automatically.
7. Returns must update availability and trigger fines if overdue.
8. Fines must be tracked, payable, and waivable (admins).
9. All actions should be validated and logged; forbidden/unauthorized access must be blocked cleanly.
10. Scheduled jobs must run for reservation expiry, overdue updates, and daily fine recalculation.

## Entity relationships (summary)
- **User**
  - One-to-many with Loan (User ↔ Loan.user)
  - One-to-many with Reservation (User ↔ Reservation.user)
  - One-to-many with Fine (User ↔ Fine.user)
  - Also referenced as checkedOutBy/returnedBy/processedBy in Loan/Fine (many-to-one back to User)
- **Book**
  - Many-to-many with Category (join table book_categories; Book.categories ↔ Category.books)
  - One-to-many with Loan (Book ↔ Loan.book)
  - One-to-many with Reservation (Book ↔ Reservation.book)
- **Category**
  - Many-to-many with Book (Category.books ↔ Book.categories)
- **Reservation**
  - Many-to-one to User (Reservation.user)
  - Many-to-one to Book (Reservation.book)
  - One-to-one to Loan (Reservation.loan, mapped by Loan.reservation)
- **Loan**
  - Many-to-one to User (Loan.user)
  - Many-to-one to Book (Loan.book)
  - One-to-one to Reservation (Loan.reservation)
  - One-to-one to Fine (Loan.fine, mapped by Fine.loan)
  - Many-to-one to User as checkedOutBy / returnedBy (staff references)
- **Fine**
  - One-to-one to Loan (Fine.loan)
  - Many-to-one to User (the fined user, Fine.user)
  - Many-to-one to User as processedBy (staff who processed payment/waiver)

## MVP features
1. **Auth & Roles**: Registration, login, JWT-based session, profile management; endpoint-level RBAC; password validation; JWT filter; global exception handling.
2. **Catalog (Books & Categories)**: CRUD for books and categories; book–category many-to-many; availability tracking; search with filters; pagination/sorting; list by category; list available.
3. **Reservations**: Students create/cancel reservations; view own and pending; automatic 48h expiry via scheduler; staff can view reservations per book.
4. **Loans (Checkout)**: Librarian/Admin checkout (with or without reservation); due-date calculation; students view their loans; staff list active/overdue; fetch by reservation.
5. **Returns & Fines**: Process returns transactionally (availability + loan status); overdue fine calculation (€0.50/day); pay fines; waive fines (admin); see pending/all; daily fine recomputation; block borrow/reserve when above threshold.

## Example flow with endpoints
1. **Register & login**
   - `POST /api/v1/auth/register` → get JWT
   - `POST /api/v1/auth/login` → get JWT
2. **Browse catalog**
   - `GET /api/v1/books/search?title=clean&category=software&available=true`
   - `GET /api/v1/categories`
3. **Reserve a book (student)**
   - `POST /api/v1/reservations/` with `{ "bookId": <id> }`
   - `GET /api/v1/reservations/my-reservations`
4. **Checkout (librarian/admin)**
   - `POST /api/v1/loans/checkout` with `{ "bookId": <id>, "userId": <studentId> }`
   - `GET /api/v1/loans/my-loans` (student) or `/api/v1/loans/active` (staff)
5. **Return & fines**
   - `POST /api/v1/loans/return` with `{ "loanId": <id> }`
   - `GET /api/v1/fines/my-fines` (student) or `/api/v1/fines/pending` (staff)
   - `POST /api/v1/fines/pay` or `/api/v1/fines/waive` (admin)
