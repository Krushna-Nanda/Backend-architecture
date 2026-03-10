# Backend-architecture


**Backend Architecture Review**

I reverse-engineered the backend under server and mapped how requests move through routes, controllers, middleware, models, and integrations.

## 1. Server Entry Point

Main backend entry is server.js.

How server starts:
- Imports Express and creates app instance in server.js.
- Loads environment variables via dotenv side-effect import in server.js.
- Connects MongoDB with top-level await in server.js, using db.js.
- Starts listener with PORT from env or 4000 in server.js.

Framework:
- Express 5.1 is used, confirmed in package.json.

Middleware registration order in server.js:
1. express.json in server.js
2. cors in server.js
3. clerkMiddleware in server.js

Route mounting:
- Health route GET / in server.js
- Inngest endpoint /api/inngest in server.js
- Routers:
  - /api/user → userRotes.js
  - /api/post → postRoutes.js
  - /api/story → storyRoutes.js
  - /api/message → messageRoutes.js

Request lifecycle from entrypoint:
- Incoming request → global middleware (JSON parse, CORS, Clerk auth context)
- Path dispatch to mounted router
- Route-level middleware (upload, protect) runs
- Controller executes business logic
- Controller calls Mongoose model operations
- JSON response returned

For SSE route:
- GET /api/message/:userId bypasses protect and opens streaming response in messageController.js.

---

## 2. Route Structure

All route files:
- userRotes.js
- postRoutes.js
- storyRoutes.js
- messageRoutes.js

No PUT or DELETE HTTP verbs are used. This API uses GET and POST only.

| Endpoint | File | Handler | Purpose |
|---|---|---|---|
| GET / | server.js | inline handler | Health check |
| ALL /api/inngest | server.js | serve from Inngest adapter | Receives Inngest events/functions |
| GET /api/user/data | userRotes.js | getUserData | Return authenticated user profile |
| POST /api/user/update | userRotes.js | updateUserData | Update profile text + profile/cover images |
| POST /api/user/discover | userRotes.js | discoverUsers | Search users by username/email/name/location |
| POST /api/user/follow | userRotes.js | followUser | Follow another user |
| POST /api/user/unfollow | userRotes.js | unfollowUser | Unfollow user |
| POST /api/user/connect | userRotes.js | sendConnectionRequest | Create connection request + trigger reminder workflow |
| POST /api/user/accept | userRotes.js | acceptConnectionRequest | Accept connection request |
| GET /api/user/connections | userRotes.js | getUserConnections | Return connections/followers/following/pending |
| POST /api/user/profiles | userRotes.js | getUserProfiles | Return another profile + their posts |
| GET /api/user/recent-messages | userRotes.js | getUserRecentMessages | Return latest inbound messages |
| POST /api/post/add | postRoutes.js | addPost | Create post with optional images/snippets |
| GET /api/post/feed | postRoutes.js | getFeedPosts | Feed from user + connections + following |
| POST /api/post/like | postRoutes.js | likePost | Toggle like/unlike |
| POST /api/post/delete | postRoutes.js | deletePost | Delete own post |
| POST /api/post/edit | postRoutes.js | editPost | Edit own post content/images/snippets |
| POST /api/post/comment/add | postRoutes.js | addComment | Add comment |
| POST /api/post/comment/delete | postRoutes.js | deleteComment | Delete own comment |
| POST /api/story/create | storyRoutes.js | addUserStory | Create story and schedule auto-delete |
| GET /api/story/get | storyRoutes.js | getStories | Get stories from network |
| GET /api/message/:userId | messageRoutes.js | sseController | Register SSE stream for live messages |
| POST /api/message/send | messageRoutes.js | sendMessage | Send text/image message; push SSE |
| POST /api/message/get | messageRoutes.js | getChatMessages | Fetch conversation and mark seen |

---

## 3. Controllers and Business Logic

Controller files:
- userController.js
- postController.js
- storyController.js
- messageController.js

What each does:
- userController: identity/profile retrieval, profile updates with image upload, user discovery, follow/unfollow, connection workflow, profile lookup.
- postController: create/feed/like/delete/edit posts, plus nested comment add/delete.
- storyController: story creation + retrieval, integrates with Inngest for expiration.
- messageController: SSE connection registry, send messages with optional image upload, conversation fetch, unseen handling.

Core backend logic:
- Social graph and connection logic in userController.js.
- Feed and interaction mechanics in postController.js.
- Realtime messaging in messageController.js.
- Ephemeral story behavior in storyController.js + index.js.

Data flow pattern:
- Route receives request and runs middleware
- Controller reads auth context via req.auth
- Controller validates basic conditions (existence, ownership checks)
- Controller performs Mongoose query/mutation
- Optional external integration (ImageKit, Clerk, Inngest, email)
- Responds with success boolean + payload/message

---

## 4. Database Layer

Database tech:
- MongoDB with Mongoose driver/ODM in db.js and dependency list in package.json.

Model/schema files:
- User.js
- Post.js
- Connection.js
- Story.js
- Message.js

Model structure:
- User uses Clerk user id as String primary key (_id is String), includes follower/following/connection arrays with refs.
- Post embeds commentSchema and codeSnippetSchema, includes likes_count as user id array.
- Connection links two user ids with pending/accepted status.
- Story stores text/image/video with viewer list and timestamps.
- Message stores from/to ids, text/image type, media URL, seen flag.

Where CRUD happens:
- User CRUD mostly in userController.js and Clerk sync functions in index.js.
- Post CRUD in postController.js.
- Story create/read in storyController.js, delete in Inngest function in index.js.
- Message create/read/updateMany in messageController.js.
- Connection create/read/update in userController.js.

---

## 5. Authentication and Security

Implemented auth model:
- Clerk-based auth verification.
- No local password hashing/JWT issuance/session storage in backend code.
- Backend trusts Clerk token through Authorization Bearer header and req.auth context.

Where enforced:
- Global Clerk middleware in server.js.
- Route-level protect middleware in auth.js.
- Most APIs require protect, except:
  - user profiles endpoint in userRotes.js
  - SSE connect endpoint in messageRoutes.js

How login works:
- Login happens on frontend with Clerk SDK in App.jsx.
- Frontend gets token via Clerk and sends Authorization header in userSlice.js, connectionsSlice.js, messagesSlice.js.
- Backend does not create tokens; it validates Clerk-authenticated context.

Security observations:
- CORS is open default call in server.js, no origin allowlist.
- No centralized input validation library is present.
- Ownership checks exist in post edit/delete and comment delete logic.
- SSE endpoint currently does not enforce auth.

---

## 6. Middleware System

Middleware used:
- express.json for body parsing in server.js
- cors in server.js
- Clerk middleware in server.js
- protect auth middleware in auth.js
- multer upload middleware in multer.js, applied per route for file fields/single/array

Middleware in lifecycle:
- Global middleware adds parsing/auth context
- Route-level multer parses multipart form and provides req.file/req.files
- protect blocks unauthenticated calls before controllers execute
- No global error-handling middleware is defined; controllers do local try/catch and return JSON errors

---

## 7. Configuration and Environment

Config files:
- DB: db.js
- Image upload provider: imageKit.js
- File upload parsing: multer.js
- Email transport: nodeMailer.js
- Runtime loading of env: server.js

Environment variables used:
- PORT
- MONGODB_URI
- IMAGEKIT_PUBLIC_KEY
- IMAGEKIT_PRIVATE_KEY
- IMAGEKIT_URL_ENDPOINT
- SMTP_USER
- SMTP_PASS
- SENDER_EMAIL
- FRONTEND_URL

Deployment config:
- Vercel serverless routing in vercel.json sends all paths to server.js.

---

## 8. Backend Architecture Map

Primary REST path:
- Client request from axios.js
- Route file postRoutes.js
- Controller postController.js
- Model Post.js
- MongoDB via Mongoose db.js
- JSON response to client

Messaging realtime path:
- Client EventSource in App.jsx
- Route messageRoutes.js
- SSE registry controller messageController.js
- On send, message persisted via Message.js
- Server pushes SSE event to connected recipient
- Client dispatches Redux update/notification

Async workflow path:
- Controller emits event via Inngest client in userController.js or storyController.js
- Inngest endpoint mounted in server.js
- Functions execute in index.js
- Side effects: email send, delayed story deletion, Clerk sync operations

---

## 9. Backend Concept Extraction

| Concept | File location | Short explanation |
|---|---|---|
| REST API routing | routes | Express routers expose resource endpoints under /api prefixes |
| MVC-like separation | routes, controllers, models | Routes delegate to controllers; models encapsulate schema/data access |
| Middleware pipeline | server.js, auth.js | Global and route-level middleware compose request processing |
| Authentication via external IdP | server.js, auth.js | Clerk validates identity; req.auth provides user id |
| Authorization checks | postController.js | Ownership checks prevent editing/deleting others’ content |
| File uploads | multer.js, route files | Multer parses multipart files for posts/stories/messages/profile |
| Cloud media storage | imageKit.js, controller files | Uploaded images/video are pushed to ImageKit and transformed |
| Realtime communication | messageController.js | SSE keeps per-user response streams and pushes incoming messages |
| Async event-driven workflows | index.js | Inngest handles delayed jobs and event-based automation |
| Scheduled jobs | index.js | Daily cron sends unseen message notifications |
| Email notification system | nodeMailer.js, index.js | SMTP transporter sends transactional and reminder emails |
| Database modeling | models | Mongoose schemas define relations and embedded subdocuments |
| CRUD operations | Controller files | Create/read/update/delete logic implemented in controller methods |
| Async programming | All controller files | async/await with Promise.all and delayed workflows |
| Error handling pattern | Controller files | Per-controller try/catch with JSON error payloads |
| CORS handling | server.js | Cross-origin requests enabled for frontend communication |
| Env-based configuration | config files + server.js | Secrets and endpoints injected via process.env |

---

## 10. Learning Analysis

A. Backend concepts you can study without frontend knowledge:
1. Express app bootstrapping and middleware order in server.js.
2. Router/controller/model separation across routes, controllers, models.
3. Mongoose schema design and querying in model/controller files.
4. Auth guard middleware pattern in auth.js.
5. File upload ingestion with Multer + external media storage.
6. SSE server implementation in messageController.js.
7. Event-driven background processing in index.js.
8. SMTP integration for notifications in nodeMailer.js.
9. Deployment routing setup in vercel.json.

B. Parts that interact with React/Tailwind ecosystem:
1. API base URL contract via axios.js and backend route prefixes.
2. Clerk token acquisition in App.jsx and Redux slices sending Authorization headers.
3. SSE connection from React app to backend message stream in App.jsx matching messageRoutes.js.
4. Inngest email links use frontend URLs from FRONTEND_URL in index.js.
5. Tailwind itself does not affect backend architecture directly.

C. Recommended learning order:
1. Start at server.js to understand startup and middleware order.
2. Read all route files to map API surface and middleware composition.
3. Study auth.js and how req.auth is consumed in controllers.
4. Learn data models in models before diving deep into controller logic.
5. Walk through one vertical slice end-to-end:
   - user data endpoint
   - post creation
   - message send/fetch
6. Study media upload path with Multer and ImageKit in config + controllers.
7. Study realtime SSE in message controller.
8. Study Inngest workflows for delayed/scheduled backend work.
9. Review env/config files and deployment config last.

