# Code-Quest

### API Endpoints

- Auth
  - ` POST /api/auth/register ` – create account
  - ` POST /api/auth/login ` – log in, receive JWT
  - ` POST /api/auth/logout ` – log out (invalidate client token)
  - ` GET /api/auth/me ` – return current user (auth check)
 
- Exam
  - ` POST /api/exams/start ` – returns examId + randomized questions (without answers)
  - ` POST /api/exams/submit ` – submit answers for examId, returns score
  - ` GET /api/exams/:examId/result ` – fetch score + per-question correctness


### Data Models

- User
  - `_id`, `name`, `email` (unique), `passwordHash`, `createdAt`
- Question
  - `_id`, `text`, `options` (array of 4 strings), `answerIndex` (0–3), `difficulty?` (optional), `tags?` (optional)
- ExamAttempt
  - `_id`, `userId`, `startedAt`, `expiresAt` (30-min window),  `questions` (array of `{ questionId, order }`),
    `responses` (array of `{ questionId, selectedIndex }`), `submittedAt?`, `score?`

  #### Rules

  - Do not send `answerIndex` to the client in `start`.
  - `ExamAttempt` freezes the set/order of questions at start.
  - Scoring happens on submit using server-side truth, not client-provided answers.


### Security points

- Use JWT Access Token (short-lived) stored in HttpOnly cookie (prevents XSS token theft).
(If you prefer Bearer in memory/localStorage, justify the trade-offs; cookie is usually safer.)
- All exam endpoints require auth (middleware checks JWT).
- Don’t expose correct answers in any API until after submit/result.
- Validate payloads (e.g., with a validator library) and sanitize inputs.



### Functional Details

- Exam duration: 30 minutes from start
- Backend stores expiresAt. Submissions after expiresAt are rejected or auto-submitted by frontend at zero.
- Frontend shows a countdown. When it hits 0: auto-submit once; disable further changes.
- On refresh, frontend re-fetches active attempt and recomputes remaining time from server timestamps.

