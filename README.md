# EU Pay QA Engineer Test Assignment

A Node.js + Express based payment API completed as part of the EU Pay QA Engineer technical assessment.

The project includes authentication, payment processing, webhook handling, table orders, and e-invoicing APIs. The intentional bugs provided in the assignment were identified through manual API testing, fixed, and verified using Postman.

## Tech Stack

- Node.js
- Express.js
- JavaScript (ES Modules)
- JWT
- bcryptjs
- UUID
- Socket.IO
- Axios
- Supertest
- Node.js built-in test runner

## Project Structure

```text
src/
├── db/
│   └── database.js
├── middleware/
│   └── auth.middleware.js
├── routes/
│   ├── auth.routes.js
│   ├── payment.routes.js
│   ├── webhook.routes.js
│   ├── table-order.routes.js
│   └── einvoice.routes.js
└── server.js

scripts/
└── verify-bugs.js
```

## Installation

Clone the repository and install the dependencies:

```
npm install
```

## Run the Application

Start the server:

```
npm start
```

The API runs at:

```
http://localhost:3000
```

For development with automatic restart:

```
npm run dev
```

## API Endpoints

### Authentication

```
POST /api/auth/register
POST /api/auth/login
```

### Payments

```
GET  /api/payments
GET  /api/payments/:id
POST /api/payments
```

### Webhooks

```
POST /api/webhooks/payment-status
POST /api/webhooks/e-invoice
```

### Table Orders

```
GET   /api/table-orders
POST  /api/table-orders
PATCH /api/table-orders/:id/status
```

### E-Invoicing

```
POST /api/einvoice/submit
GET  /api/einvoice/status/:invoice_id
```

### Bugs Identified and Fixed

The assignment contained seven intentional bugs. All seven bugs were identified and fixed.

## Bug A — Payment Amount Data Type

The payment amount was originally returned as a string instead of a numeric value.
The issue was fixed so that the API returns the payment amount as a number.
Example:

```
{
  "amount": 100
}
```

instead of:

```
{
  "amount": "100"
}
```

## Bug B — Maximum Payment Amount

The API did not properly enforce the maximum allowed payment amount.
A maximum transaction amount of €50,000 was added.
Amounts above €50,000 are rejected with:

```
400 Bad Request
```

Amounts equal to or below €50,000 are accepted when all other validations pass.

## Bug C — Email Validation

The original email validation accepted invalid email addresses such as:

```
user@domain
```

The email validation was improved to require a valid domain and top-level domain.
For example:

```
user@example.com
```

is accepted, while invalid email formats are rejected.

## Bug D — Webhook Signature Verification

The webhook endpoint did not properly verify the webhook signature.
HMAC-SHA256 signature verification was implemented using the configured webhook secret.
The API checks the:

```
x-webhook-signature
```

header.
Requests with a missing or invalid signature return:

```
401 Unauthorized
```

Valid signatures are accepted and processed.

## Bug E — Webhook Idempotency

The webhook endpoint could process the same webhook event multiple times.
Idempotency was implemented using the:

```
event_id
```

field.
When the same event_id is received again, the event is not processed a second time.
The API returns:

```
{
  "success": true,
  "message": "Already processed"
}
```

## Bug F — Duplicate Table Orders

The table-order endpoint allowed multiple pending orders for the same merchant and table.
A duplicate check was added for:

- Merchant
- Table number
- Pending order status
  If a pending order already exists, the API returns:

```
409 Conflict
```

with an appropriate error message.

## Bug G — Duplicate E-Invoice Submission

The e-invoice endpoint allowed an invoice to be submitted more than once.
A validation check was added to prevent resubmission when an invoice has already been submitted or accepted.
Duplicate submissions return:

```
409 Conflict
```

## Manual API Testing

The APIs were manually tested using Postman.
The following scenarios were tested:

- User registration
- User login
- Payment creation
- Payment amount validation
- Maximum payment amount validation
- Currency validation
- Email validation
- Webhook signature verification
- Missing webhook signature
- Invalid webhook signature
- Valid webhook signature
- Webhook idempotency
- Duplicate table order prevention
- E-invoice submission
- Duplicate e-invoice submission

## Automated Testing

The project uses the Node.js built-in test runner for automated testing.
Run the test suite using:

```
npm test
```

The automated tests cover the API behaviour and the identified bugs.

## Webhook Security

Payment-status webhooks use HMAC-SHA256 signature verification.
The signature is provided through:

```
x-webhook-signature
```

For the test environment, the webhook secret is:

```
test-webhook-secret-key
```

In a production environment, secrets should not be hardcoded. They should be stored securely using environment variables or a dedicated secrets manager.

## Authentication

Protected API endpoints use JWT authentication.
After successful login, the returned token is sent using:

```
Authorization: Bearer <JWT_TOKEN>
```

Example:

```
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
```

## Idempotency

Webhook events use event_id to prevent duplicate processing.
For example:

```
event_id = event-001
```

If the event is received for the first time, it is processed.
If the same event is received again, it is detected as already processed and is not processed again.
This prevents duplicate webhook processing.

## Database

The assignment uses an in-memory database implementation.
The database contains data for:

- Users
- Payments
- Sessions
- Table orders
- Invoices
- Processed webhook events
  Because the database is in memory, the data is reset whenever the server is restarted.
  For a production application, a persistent database such as PostgreSQL or another suitable database system would be required.

## Testing Approach

The testing approach included multiple types of API testing:

## Functional Testing

Verified that API endpoints behave correctly for valid requests.

## Negative Testing

Tested invalid inputs and invalid requests such as:

- Invalid email
- Invalid payment amount
- Missing webhook signature
- Invalid webhook signature
- Duplicate requests

## Boundary Testing

Tested important limits such as:

```
50000
```

and values above the maximum amount.

## Security Testing

Tested webhook signature validation and authentication behaviour.

## Idempotency Testing

Sent the same webhook event multiple times and verified that it was processed only once.

## Duplicate Request Testing

Tested duplicate table orders and duplicate e-invoice submissions.

## Postman Testing

Postman was used for manual API verification during development.
The main flow used during testing was:

```
Register User
      ↓
Login
      ↓
Receive JWT Token
      ↓
Create Payment
      ↓
Test Payment Validation
      ↓
Test Webhook
      ↓
Test Idempotency
      ↓
Test Table Orders
      ↓
Test E-Invoice
```

## Error Handling

The API returns appropriate HTTP status codes for different failure conditions.
Examples include:

```
200 OK
201 Created
400 Bad Request
401 Unauthorized
404 Not Found
409 Conflict
```

The response body contains an error message to help identify the reason for the failure.

## Webhook Flow

The payment webhook flow is:

```
Payment Service
      ↓
Webhook Request
      ↓
Check Signature
      ↓
Validate event_id
      ↓
Check Duplicate Event
      ↓
Find Payment
      ↓
Update Payment Status
      ↓
Store event_id
      ↓
Return Success Response
```

If the signature is invalid, the request is rejected before the webhook is processed.
If the event was already processed, the API returns an idempotent response instead of processing it again.

## Security Considerations

The following security areas were considered during testing and implementation:

- JWT authentication
- Password hashing using bcrypt
- Webhook HMAC signature verification
- Duplicate webhook protection
- Input validation
- Payment amount limits
- Duplicate request handling
- Secure handling of webhook secrets
  For a production environment, additional security measures would also be required, including:
- HTTPS
- Secure secret management
- Rate limiting
- Proper database constraints
- Request logging and monitoring
- Protection against replay attacks
- Stronger authorization controls

## Limitations

This project uses an in-memory database because it is part of a test assignment.
As a result:

- Data is lost after server restart.
- Database-level unique constraints are not available.
- Concurrency behaviour is limited compared with a production database.
- Production-grade persistence and transaction handling would require a real database.

## Test Commands

Install dependencies:

```
npm install
```

Start the application:

```
npm start
```

Run automated tests:

```
npm test
```

Run the repository bug verification script:

```
npm run verify-bugs
```

The verify-bugs script is provided by the assignment repository and is intended to verify the original intentional bugs in the codebase. The final correctness of the fixes was validated separately through manual API testing and automated tests.
