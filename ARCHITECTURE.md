# mentormatch-global Architecture

## Overview
To democratize access to mentorship by connecting professionals worldwide with relevant mentors tailored to their career goals and aspirations.

### Problem Statement
Lack of accessible and personalized career mentorship, especially for individuals in developing countries or those seeking career transitions and requiring specialized industry knowledge.

### Target Audience
Early to mid-career professionals across various industries globally, particularly those in developing countries or those undergoing career transitions and seeking guidance from experienced mentors.

### Core Entities
Users (Mentors and Mentees), Profiles (Skillsets, Experience, Goals), Mentorship Sessions, Reviews and Ratings, Matching Algorithms, Industries, Career Paths

## Design System

### Typography
- **Font**: Poppins
- **Font Stack**: `'Poppins', sans-serif`
- **Google Fonts**: https://fonts.google.com/specimen/Poppins
- **Rationale**: Poppins is a versatile sans-serif font known for its clean, geometric forms and excellent readability at various sizes and weights. Its modern aesthetic complements the professional nature of the platform. It also supports a wide range of languages, ensuring global accessibility.

### Color Palette
A complementary palette chosen for its balance and professional feel. It prioritizes accessibility with high contrast ratios, ensuring readability for all users. The primary and secondary colors provide a sense of trust and reliability, while the accent color adds a touch of energy and excitement.

- **Primary**: `#007bff` - Main brand color, used for headers, primary actions (buttons), and the primary call to action.  This adheres to WCAG AA contrast ratio requirements.
- **Secondary**: `#6c757d` - Supporting color for secondary elements, such as section headings and less important buttons. Used sparingly to avoid visual clutter.  This adheres to WCAG AA contrast ratio requirements.
- **Accent**: `#ffc107` - Used for call-to-action buttons (e.g., "Connect with Mentor"), highlights, and alerts. A visual cue to draw attention.  This adheres to WCAG AA contrast ratio requirements.
- **Neutral Text**: `#212529` - Primary text color for high contrast and readability on light backgrounds.  Adheres to WCAG AA contrast ratio requirements.
- **Neutral Background**: `#f8f9fa` - Main background color for content areas to provide a clean and professional look.  Provides sufficient contrast for text and other elements.
- **Neutral Border**: `#dee2e6` - For card borders, dividers, and form inputs, providing subtle visual separation.
- **Success**: `#28a745` - For success messages and confirmation, indicating a successful action.
- **Warning**: `#ffc107` - For warnings and non-critical alerts, indicating potential issues.
- **Danger**: `#dc3545` - For error messages and destructive actions, indicating critical errors or potential data loss.

## System Architecture

### 1. Core Components & Rationale
MentorMatch Global utilizes a modular architecture built upon Laravel for the backend and React/Inertia.js with TypeScript for the frontend. This separation of concerns enables independent scaling and development. Key components include:

*   **Backend (Laravel):** Manages the API, business logic, database interactions, authentication, authorization, and real-time events.
*   **Frontend (React/Inertia.js/TypeScript):** Handles the user interface, user interactions, and data presentation. Inertia.js acts as the glue between Laravel and React, simplifying routing and data fetching.
*   **Database (MySQL/PostgreSQL):** Stores user data, profiles, mentorship sessions, reviews, and other relational data.
*   **Redis:** Used for caching, session management, and real-time event broadcasting (with `laravel-echo-server`).
*   **Object Storage (AWS S3/Google Cloud Storage):** Stores user-uploaded files (e.g., profile pictures, documents).

Rationale:

*   **Scalability:** Laravel's queue system and Redis integration facilitate asynchronous processing and horizontal scaling.  React's component-based architecture enables efficient rendering and reuse.
*   **Security:** Laravel's built-in security features, combined with secure coding practices and regular security audits, mitigate OWASP Top 10 vulnerabilities.
*   **Maintainability:** The modular architecture promotes code reusability and simplifies debugging and maintenance.  TypeScript enhances code quality and reduces runtime errors.
*   **User Experience:** React provides a fast and responsive user interface, while Inertia.js simplifies routing and data fetching.

By using the 'laravel/react-starter-kit', which is a modern framework, the project leverages the best features of both technologies, TypeScript's type safety, and a streamlined development workflow.

### 2. Database Schema Design
The database schema consists of several key tables:

*   **Users:** Stores user authentication information (id, name, email, password, role (mentor/mentee), email_verified_at, created_at, updated_at).
*   **Profiles:** Stores user profile data (id, user_id (FK), bio, skills (JSON), experience (JSON), goals (JSON), industry_id (FK), career_path_id (FK), avatar_url, timezone, location).
*   **MentorshipSessions:** Stores information about mentorship sessions (id, mentor_id (FK), mentee_id (FK), start_time, end_time, status (scheduled, completed, cancelled), notes).
*   **Reviews:** Stores reviews and ratings for mentors (id, mentor_id (FK), mentee_id (FK), rating, comment, created_at, updated_at).
*   **Industries:** Stores a list of industries (id, name).
*   **CareerPaths:** Stores a list of career paths (id, name).
*   **Notifications:** Stores user-specific notifications (id, user_id (FK), type, data (JSON), read_at (nullable timestamp), created_at, updated_at).
*   **PasswordResets:** Stores password reset tokens (email, token, created_at).
*   **FailedJobs:** Stores failed queue jobs (id, connection, queue, payload, exception, failed_at).
*   **PersonalAccessTokens:** Stores API tokens (id, tokenable_type, tokenable_id, name, token, abilities, last_used_at, expires_at, created_at, updated_at).

JSON columns in Profiles (skills, experience, goals) allow for flexible data storage and can be queried using Laravel's JSON column support. Foreign keys ensure data integrity and relationships between tables.  Indexes are used to optimize query performance.

### 3. API Design & Key Endpoints
The API follows a RESTful design using Laravel's resource controllers.  API versioning is implemented using a `/api/v1/` prefix.

Key endpoints include:

*   **Authentication:**
    *   `POST /api/v1/register`: Registers a new user.
    *   `POST /api/v1/login`: Logs in an existing user.
    *   `POST /api/v1/logout`: Logs out the current user.
    *   `POST /api/v1/password/reset`: Sends a password reset email.
    *   `POST /api/v1/password/reset/confirm`: Resets the user's password.
    *   `GET /api/v1/user`: Retrieves the authenticated user's data.
*   **Profiles:**
    *   `GET /api/v1/profiles/{id}`: Retrieves a user profile.
    *   `PUT /api/v1/profiles/{id}`: Updates a user profile.
*   **Mentorship Sessions:**
    *   `GET /api/v1/mentorship-sessions`: Retrieves a list of mentorship sessions for the authenticated user (as a mentor or mentee).
    *   `POST /api/v1/mentorship-sessions`: Creates a new mentorship session.
    *   `GET /api/v1/mentorship-sessions/{id}`: Retrieves a specific mentorship session.
    *   `PUT /api/v1/mentorship-sessions/{id}`: Updates a mentorship session.
    *   `DELETE /api/v1/mentorship-sessions/{id}`: Cancels a mentorship session.
*   **Reviews:**
    *   `POST /api/v1/reviews`: Creates a new review for a mentor.
    *   `GET /api/v1/reviews/{mentor_id}`: Retrieves reviews for a specific mentor.
*   **Matching:**
    *   `GET /api/v1/matches`: Retrieves a list of potential mentors/mentees based on the user's profile and preferences.
*   **Notifications:**
    *   `GET /api/v1/notifications`: Retrieves a list of notifications for the authenticated user.
    *   `POST /api/v1/notifications/{id}/mark-as-read`: Marks a notification as read.

API requests are authenticated using Laravel Sanctum, which provides a lightweight token-based authentication system suitable for SPAs.

### 4. Frontend Structure (TypeScript)
The frontend is built using React, Inertia.js, and TypeScript.  The directory structure is organized as follows:

*   `resources/ts/`: Root directory for frontend code.
    *   `Components/`: Reusable React components (e.g., buttons, form fields, profile cards).
    *   `Layouts/`: Application layouts (e.g., main layout, guest layout).
    *   `Pages/`: React components representing individual pages (e.g., Home, Profile, Mentorship Sessions).
    *   `Services/`: TypeScript classes or functions for interacting with the API (e.g., AuthService, ProfileService).
    *   `Types/`: TypeScript type definitions for data models (e.g., User, Profile, MentorshipSession).
    *   `Utils/`: Utility functions (e.g., date formatting, string manipulation).
    *   `hooks/`: Custom React hooks to reuse component logic.
    *   `app.tsx`: Entry point for the React application.
    *   `inertia.ts`: Inertia.js configuration.

Each React component is written in TypeScript (.tsx extension) and uses functional components with React Hooks. State management is handled using React's built-in `useState` and `useContext` hooks. Complex state management can be managed using a state management library like Zustand or Jotai, if necessary.  Component styling is done using CSS Modules or a CSS-in-JS library like Styled Components.

The 'Pages' directory maps directly to Laravel routes, thanks to Inertia.js. Data is passed from Laravel controllers to React components as props.  Form submissions are handled using Inertia.js's `useForm` hook.

### 5. Real-time & Events Architecture
Real-time functionality is implemented using Laravel's event broadcasting system and Redis, along with `laravel-echo-server`. The architecture includes:

*   **Laravel Events:** Define events that occur in the application (e.g., `MentorshipSessionCreated`, `MessageSent`).
*   **Event Listeners:** Listen for specific events and perform actions, such as broadcasting the event to a Redis channel.
*   **Redis:** Acts as the message broker for real-time events.
*   **`laravel-echo-server`:** A Node.js server that listens for events on Redis channels and broadcasts them to connected clients via WebSockets.
*   **Frontend (React):** Uses the `laravel-echo` JavaScript library to subscribe to specific channels and listen for events. Updates the UI in real-time when new events are received.

Example:

When a new message is sent in a mentorship session, a `MessageSent` event is fired. The event listener broadcasts the event to a private channel (`mentorship-sessions.{session_id}`). The React component responsible for displaying the chat messages subscribes to this channel using `laravel-echo` and updates the UI in real-time when a new message is received.

Channels are secured using authentication and authorization checks to ensure that only authorized users can subscribe to specific channels.  Private channels require authentication, while presence channels allow tracking of users who are currently subscribed to a channel.

### 6. Authentication & Authorization Flow
Authentication is handled using Laravel Sanctum, providing a lightweight token-based authentication system for SPAs.

1.  **Registration:** A user submits their registration information (name, email, password).
2.  **Backend:** The Laravel backend validates the input, creates a new user account, and generates a Sanctum API token.
3.  **Response:** The API token is returned to the frontend.
4.  **Frontend:** The frontend stores the API token in local storage or a cookie.
5.  **Subsequent Requests:** For subsequent API requests, the frontend includes the API token in the `Authorization` header (`Bearer {token}`).
6.  **Backend:** The Laravel backend authenticates the request using the API token and retrieves the associated user.

Authorization is handled using Laravel's built-in authorization features, including policies and gates. Policies define which users are allowed to perform specific actions on specific resources (e.g., update a profile, cancel a mentorship session). Gates provide a simple way to define authorization rules that are not tied to a specific resource.

Middleware is used to protect routes and ensure that only authenticated and authorized users can access specific resources.

Role-based access control (RBAC) can be implemented by adding a `role` column to the `users` table and using policies to restrict access based on user roles (e.g., administrator, mentor, mentee).

### 7. Deployment & Scalability Plan
The application will be deployed using a containerized approach with Docker and orchestrated with Kubernetes or Docker Compose. This allows for consistent deployments across different environments and simplifies scaling.

1.  **Dockerization:** The Laravel backend and `laravel-echo-server` are packaged as Docker containers.
2.  **Container Orchestration:** Kubernetes or Docker Compose is used to manage the deployment and scaling of the containers.
3.  **Database:** A managed database service (e.g., AWS RDS, Google Cloud SQL, Azure Database for MySQL/PostgreSQL) is used for the database. This provides automatic backups, scaling, and high availability.
4.  **Redis:** A managed Redis service (e.g., AWS ElastiCache, Google Cloud MemoryStore, Azure Cache for Redis) is used for caching, session management, and real-time event broadcasting.
5.  **Object Storage:** AWS S3 or Google Cloud Storage is used for storing user-uploaded files.
6.  **Load Balancing:** A load balancer (e.g., AWS ALB, Google Cloud Load Balancing) is used to distribute traffic across multiple backend instances.
7.  **Continuous Integration/Continuous Deployment (CI/CD):** A CI/CD pipeline (e.g., GitHub Actions, GitLab CI, Jenkins) is used to automate the build, test, and deployment process.  This ensures that code is automatically tested and deployed to production whenever changes are committed to the repository.

Scaling is achieved by horizontally scaling the backend instances and the `laravel-echo-server` instances.  The database and Redis instances can also be scaled as needed. Load testing should be conducted regularly to identify bottlenecks and ensure that the application can handle the expected traffic.

### 8. Testing Strategy
A comprehensive testing strategy is implemented to ensure the quality and reliability of the application.

*   **Backend (Laravel):**
    *   **Unit Tests (Pest):** Test individual units of code (e.g., models, controllers, services) in isolation.  Focus on testing the business logic and ensuring that each unit of code behaves as expected.
    *   **Feature Tests (Pest):** Test the application's features from an end-to-end perspective.  Simulate user interactions and verify that the application behaves as expected.
    *   **Integration Tests (Pest):** Test the interaction between different components of the application (e.g., controllers and models).
*   **Frontend (React/TypeScript):**
    *   **Unit Tests (Vitest):** Test individual React components and utility functions in isolation. Use a mocking library (e.g., Jest Mock) to mock dependencies.
    *   **Component Tests (Vitest & React Testing Library (RTL)):** Test React components in a realistic environment.  Use RTL to interact with the components as a user would.
    *   **End-to-End (E2E) Tests (Cypress/Playwright):** Test the entire application from an end-to-end perspective.  Simulate user interactions across multiple pages and verify that the application behaves as expected.
*   **Code Coverage:** Code coverage tools are used to measure the percentage of code that is covered by tests.  Aim for high code coverage to ensure that most of the code is tested.
*   **Continuous Integration:** Tests are automatically run as part of the CI/CD pipeline to ensure that code changes do not introduce regressions.  Failed tests should block the deployment process.

### 9. Security & Hardening Plan
A 'security-first' approach is implemented to mitigate OWASP Top 10 vulnerabilities and other security risks.

*   **Input Validation:** All user inputs are validated on both the client-side and the server-side to prevent injection attacks (e.g., SQL injection, Cross-Site Scripting (XSS)).  Laravel's built-in validation features are used for server-side validation.
*   **Output Encoding:** Output is encoded to prevent XSS attacks.  Laravel's templating engine automatically escapes output by default.
*   **Authentication & Authorization:** Strong authentication and authorization mechanisms are implemented to protect sensitive data and functionality. Laravel Sanctum is used for API authentication, and policies and gates are used for authorization.
*   **Password Hashing:** User passwords are hashed using bcrypt to protect them from being compromised in the event of a data breach. Laravel's built-in hashing functions are used.
*   **Cross-Site Request Forgery (CSRF) Protection:** CSRF protection is enabled to prevent malicious websites from making unauthorized requests on behalf of authenticated users.  Laravel's CSRF protection middleware is used.
*   **Rate Limiting:** Rate limiting is implemented to prevent brute-force attacks and denial-of-service (DoS) attacks.  Laravel's built-in rate limiting middleware is used.
*   **Regular Security Audits:** Regular security audits are conducted to identify and address potential security vulnerabilities.  Penetration testing is also performed to simulate real-world attacks.
*   **Dependency Management:** Dependencies are regularly updated to address known security vulnerabilities.  A dependency management tool (e.g., Composer, npm) is used to manage dependencies and ensure that they are up-to-date.
*   **Security Headers:** Security headers (e.g., Content Security Policy (CSP), X-Frame-Options, Strict-Transport-Security) are configured to protect against various attacks.
*   **Data Encryption:** Sensitive data (e.g., API keys, database passwords) is encrypted at rest and in transit.  Laravel's encryption features and HTTPS are used.
*   **OWASP ZAP:** Integration of OWASP ZAP into the CI/CD pipeline to automatically scan for vulnerabilities with each build. This helps to catch security issues early in the development lifecycle.  See [https://www.zaproxy.org/](https://www.zaproxy.org/) for more details.

### 10. Logging & Observability
A comprehensive logging and observability strategy is implemented to monitor the health and performance of the application.

*   **Logging:**
    *   **Application Logs:** Application logs are generated using Laravel's logging facade.  Logs are written to the `stderr` channel for centralized logging in containerized environments. The `stderr` channel outputs log messages to the standard error stream, which is typically collected by container orchestration platforms.
    *   **Access Logs:** Access logs are generated by the web server (e.g., Nginx, Apache) to track incoming requests.
    *   **Database Logs:** Database logs are enabled to track database queries and performance.
*   **Monitoring:**
    *   **Application Performance Monitoring (APM):** An APM tool (e.g., New Relic, Datadog, Sentry) is used to monitor the performance of the application and identify bottlenecks.  APM tools provide insights into request latency, error rates, and other key metrics.
    *   **Infrastructure Monitoring:** Infrastructure monitoring tools (e.g., Prometheus, Grafana) are used to monitor the health and performance of the infrastructure (e.g., servers, databases, Redis).
    *   **Error Tracking:** An error tracking tool (e.g., Sentry, Bugsnag) is used to track and report errors that occur in the application.  This allows developers to quickly identify and fix errors.
*   **Alerting:** Alerts are configured to notify developers when critical events occur (e.g., high error rates, slow response times).  Alerts can be sent via email, Slack, or other channels.
*   **Centralized Logging:** A centralized logging system (e.g., ELK Stack, Splunk) is used to collect and analyze logs from all components of the application.  This allows developers to easily search for and analyze logs across multiple systems.

Key metrics to monitor include:

*   **Request latency:** The time it takes to process a request.
*   **Error rates:** The percentage of requests that result in an error.
*   **CPU utilization:** The percentage of CPU resources that are being used.
*   **Memory utilization:** The percentage of memory resources that are being used.
*   **Database query performance:** The time it takes to execute database queries.
*   **Redis cache hit rate:** The percentage of requests that are served from the Redis cache.

Logging levels should be used appropriately (e.g., `debug`, `info`, `warning`, `error`) to provide sufficient context for debugging and troubleshooting.

## Feature Implementation Roadmap

### 1. Advanced Matching Algorithm
Matches mentors and mentees based on a weighted combination of skills, experience, career goals, industry, and career path, ensuring highly relevant connections.

**Database Changes:**
- Add 'weight' columns to skills, experience, goals tables in Profiles table to indicate their importance.
- Create a 'matching_preferences' table (user_id (FK), industry_weight, career_path_weight, skills_weight, experience_weight, goals_weight).

**Backend Components:**
- A new 'MatchingService' class to encapsulate the matching algorithm logic.
- Update the ProfileService to handle matching preferences.
- Create a MatchingController to expose matching functionality.

**Frontend Components:**
- A new '/matching-preferences' page component (`Pages/MatchingPreferences/Index.tsx`) to allow users to configure matching preferences.
- Update the Home page to display recommended mentor/mentee matches.
- Create 'ProfileCard.tsx' component to display the matching percentage.

**API Endpoints:**
- GET /api/v1/matches: Returns a list of potential mentors/mentees based on the user's profile and preferences.
- GET /api/v1/matching-preferences: Retrieves the matching preferences for the authenticated user.
- PUT /api/v1/matching-preferences: Updates the matching preferences for the authenticated user.

**Real-time Events:**
- None applicable.

**Testing Requirements:**
- Unit tests for `MatchingService` to ensure the algorithm calculates scores correctly based on different weights and user profiles.
- Feature tests to verify that the matching API endpoint returns the correct matches based on different scenarios.
- Component tests for '/matching-preferences' to ensure users can modify their preferences.


### 2. Video Conferencing and Screen Sharing
Enables mentors and mentees to conduct virtual mentorship sessions with integrated video conferencing and screen sharing capabilities.

**Database Changes:**
- Add 'room_id' (string) to MentorshipSessions table to store the identifier for the video conference room (generated by the chosen video conferencing provider).

**Backend Components:**
- Integration with a third-party video conferencing provider (e.g., Zoom, Jitsi, Twilio).  A dedicated service class (e.g., 'VideoConferencingService') would abstract away the provider-specific API calls.
- A MentorshipSessionsController update to handle generating/retrieving video conferencing room IDs.

**Frontend Components:**
- A 'VideoConference.tsx' component that embeds the video conferencing interface (using the chosen provider's SDK).
- Update the MentorshipSessions details page to include a "Join Session" button.

**API Endpoints:**
- POST /api/v1/mentorship-sessions/{id}/start-video-conference: Initiates a video conference for the given mentorship session (generates a room ID if one doesn't exist).
- GET /api/v1/mentorship-sessions/{id}/video-conference-url: Returns the URL for the video conference room.

**Real-time Events:**
- A 'VideoConferenceStarted' event broadcast to the mentorship session channel when the video conference is initiated, notifying both mentor and mentee.

**Testing Requirements:**
- Integration tests to verify that video conference sessions can be created and started successfully.
- E2E tests to simulate a full mentorship session with video conferencing.


### 3. Secure Messaging and File Sharing
Provides a secure channel for mentors and mentees to communicate, share files, and collaborate on documents.

**Database Changes:**
- Create a 'messages' table (id, mentorship_session_id (FK), user_id (FK), content, file_path (nullable), created_at, updated_at).
- If file sharing is supported, create a 'files' table (id, user_id (FK), file_name, file_path, file_size, file_type, created_at).

**Backend Components:**
- A 'MessageService' class to handle message creation and retrieval.
- A 'FilesService' class to handle file uploads and downloads (integrated with object storage).
- MessagesController to expose message-related endpoints.

**Frontend Components:**
- A 'Chat.tsx' component to display the messaging interface and handle sending/receiving messages.
- A 'FileUpload.tsx' component to handle file uploads.
- Integration with the Mentorship Session details page to show the chat interface.

**API Endpoints:**
- GET /api/v1/mentorship-sessions/{id}/messages: Retrieves messages for a specific mentorship session.
- POST /api/v1/mentorship-sessions/{id}/messages: Creates a new message in a specific mentorship session.
- POST /api/v1/files/upload: Uploads a new file.
- GET /api/v1/files/{id}/download: Downloads a specific file.

**Real-time Events:**
- A 'MessageSent' event broadcast on a private channel `mentorship-sessions.{session_id}` when a new message is sent.  This includes the message data and the user who sent it.

**Testing Requirements:**
- Unit tests for `MessageService` to ensure messages are created and retrieved correctly.
- Component tests for `Chat.tsx` to verify that messages are displayed and sent correctly.
- Integration tests for file upload and download functionality.


### 4. Progress Tracking and Goal Setting
Allows mentees to set goals, track their progress, and receive feedback from mentors.

**Database Changes:**
- Create a 'goals' table (id, user_id (FK), mentorship_session_id (FK), description, due_date, status (open, in_progress, completed), created_at, updated_at).
- Create a 'progress_updates' table (id, goal_id (FK), description, created_at, updated_at).

**Backend Components:**
- A 'GoalsService' class to handle goal creation, retrieval, and updates.
- A 'ProgressUpdateService' class to handle progress update creation and retrieval.
- GoalsController and ProgressUpdatesController to expose API endpoints.

**Frontend Components:**
- A 'GoalsList.tsx' component to display a list of goals.
- A 'GoalForm.tsx' component to create and edit goals.
- A 'ProgressUpdateForm.tsx' component to add progress updates to a goal.
- Integration with the Mentorship Session details page to display goals and progress updates.

**API Endpoints:**
- GET /api/v1/goals: Retrieves a list of goals for the authenticated user.
- POST /api/v1/goals: Creates a new goal.
- GET /api/v1/goals/{id}: Retrieves a specific goal.
- PUT /api/v1/goals/{id}: Updates a specific goal.
- DELETE /api/v1/goals/{id}: Deletes a specific goal.
- POST /api/v1/goals/{id}/progress-updates: Adds a new progress update to a goal.
- GET /api/v1/goals/{id}/progress-updates: Retrieves progress updates for a specific goal.

**Real-time Events:**
- A 'GoalCreated' event broadcast on a private channel `users.{user_id}` when a new goal is created.
- A 'GoalUpdated' event broadcast on a private channel `users.{user_id}` when a goal is updated.
- A 'ProgressUpdateCreated' event broadcast on a private channel `goals.{goal_id}` when a new progress update is created.

**Testing Requirements:**
- Unit tests for `GoalsService` and `ProgressUpdateService` to ensure data is correctly handled.
- Component tests for `GoalsList.tsx` and `GoalForm.tsx` to ensure goal management functionality.
- Feature tests to confirm that goal creation and tracking work as expected from an end-to-end perspective.


### 5. Mentor Verification and Background Checks
Implements a robust system to verify mentor identities and perform background checks to ensure safety and quality assurance.

**Database Changes:**
- Add 'is_verified' (boolean) and 'background_check_status' (enum: pending, approved, rejected) columns to the Users table.
- Create a 'verification_documents' table (id, user_id (FK), document_type (e.g., ID, certifications), file_path, status (pending, approved, rejected), created_at, updated_at).

**Backend Components:**
- A 'VerificationService' class to handle document uploads, verification checks (manual or via a third-party API), and status updates.
- Update UserService to handle changes to 'is_verified' status.
- Update UserController to handle display of verification status.

**Frontend Components:**
- A '/verify' page component (`Pages/Verify/Index.tsx`) for mentors to upload verification documents.
- An admin interface (if applicable) to review verification documents and update mentor status.
- A badge/icon on mentor profiles to indicate verified status.

**API Endpoints:**
- POST /api/v1/verify/upload: Uploads a verification document.
- GET /api/v1/verify/status: Returns the verification status of the authenticated user.
- PUT /api/v1/admin/verify/{user_id}: Admin endpoint to approve/reject verification (requires admin role).

**Real-time Events:**
- A 'VerificationStatusChanged' event broadcast on a private channel `users.{user_id}` when the verification status of a user changes.

**Testing Requirements:**
- Unit tests for `VerificationService` to ensure document uploads and status updates are handled correctly.
- Feature tests for the mentor verification workflow, including document upload and admin approval.
- Component tests for `Pages/Verify/Index.tsx` to ensure the verification process is working.


## Metadata
- **Generated**: 2025-06-18T08:48:58.799804+00:00
- **Project Type**: Laravel React Starter Kit
- **Architecture Version**: 1.0.0

---
*This architecture document is maintained by the CodeWebMobile AI system and should be the source of truth for all development decisions.*
