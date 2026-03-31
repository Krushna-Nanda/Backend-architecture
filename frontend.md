# opus 4.6 review

Now I have all the information needed to create the comprehensive analysis. Let me also check if there's a single post route handler:

Searched for files matching `api/post/:postId|get\(.*/api/post/` in `f:\Pingup-Setup`, 2 results

Searched for files matching `get\(|post\(` in `f:\Pingup-Setup\server\routes`, 22 results

I notice the `SinglePost.jsx` calls `api.get('/api/post/${postId}')` but I don't see a corresponding route for that. Let me check if it's defined elsewhere:

Searched for files matching `postId|:id` in `f:\Pingup-Setup\server`, 11 results

Now I have all the information needed. Let me create the comprehensive analysis document.

---

# **FRONTEND ARCHITECTURE ANALYSIS — PINGUP (ZINGUP) SOCIAL PLATFORM**

---

## **1. FRONTEND ENTRY POINT**

### Framework: **React 19.1.0** with **Vite** as build tool

### Entry File: `client/src/main.jsx`

```
Browser
↓
index.html (line 11 loads /src/main.jsx)
↓
main.jsx → createRoot(document.getElementById('root'))
```

### Boot Sequence (main.jsx lines 17-27):

```jsx
createRoot(document.getElementById('root')).render(
  <ClerkProvider publishableKey={PUBLISHABLE_KEY}>    // Auth provider (outermost)
    <BrowserRouter>                                    // React Router
      <Provider store={store}>                         // Redux store
        <ThemeProvider>                                // Dark/Light theme context
          <App />                                      // Root component
        </ThemeProvider>
      </Provider>  
    </BrowserRouter>    
  </ClerkProvider>
)
```

### Provider Hierarchy (from outer to inner):

| Provider | File | Purpose |
|----------|------|---------|
| `ClerkProvider` | `@clerk/clerk-react` | Authentication (Clerk) |
| `BrowserRouter` | `react-router-dom` | Client-side routing |
| `Provider` | `react-redux` | Redux global state |
| `ThemeProvider` | `context/ThemeContext.jsx` | Dark/light mode |

### Environment Variable Used:
- `VITE_CLERK_PUBLISHABLE_KEY` (main.jsx line 11) — Clerk authentication key

---

## **2. ROUTING STRUCTURE**

### Router Type: React Router v7 (`react-router-dom@7.7.1`)

### Route Configuration: `client/src/App.jsx` (lines 70-82)

| URL Path | File | Component | Purpose |
|----------|------|-----------|---------|
| `/` | `pages/Login.jsx` OR `pages/Layout.jsx` | `Login` (unauthenticated) / `Layout` + `Feed` (authenticated) | Login or Feed page |
| `/` (nested) | `pages/Feed.jsx` | `Feed` | Main feed with posts |
| `/messages` | `pages/Messages.jsx` | `Messages` | List of connections to message |
| `/messages/:userId` | `pages/ChatBox.jsx` | `ChatBox` | 1-on-1 chat view |
| `/connections` | `pages/Connections.jsx` | `Connections` | Manage followers/following |
| `/discover` | `pages/Discover.jsx` | `Discover` | Search and discover users |
| `/profile` | `pages/Profile.jsx` | `Profile` | Current user profile |
| `/profile/:profileId` | `pages/Profile.jsx` | `Profile` | View other user's profile |
| `/create-post` | `pages/CreatePost.jsx` | `CreatePost` | Create new post form |
| `/post/:postId` | `pages/SinglePost.jsx` | `SinglePost` | View single post (public) |

### Route Protection Logic (App.jsx line 71):
```jsx
<Route path='/' element={ !user ? <Login /> : <Layout/>}>
```
- If not authenticated (`!user`): render `Login`
- If authenticated: render `Layout` which contains `Sidebar` + `Outlet` for child routes

---

## **3. PAGES & COMPONENTS**

### **Page Components** (`client/src/pages/`)

| Page | File | Role | Stateful? | Key Props/Data |
|------|------|------|-----------|----------------|
| **Login** | `Login.jsx` | Auth UI with Clerk SignIn | Presentational | None - uses Clerk's `<SignIn />` |
| **Layout** | `Layout.jsx` | Wrapper with Sidebar + Outlet | Yes (sidebar toggle) | Reads `user` from Redux |
| **Feed** | `Feed.jsx` | Main feed with posts & stories | Yes (feeds, loading) | Fetches posts via API |
| **Messages** | `Messages.jsx` | List connections for messaging | Presentational | Reads `connections` from Redux |
| **ChatBox** | `ChatBox.jsx` | Real-time chat interface | Yes (text, image, user, messages) | Reads/dispatches messages Redux |
| **Connections** | `Connections.jsx` | Manage followers/following/pending | Yes (currentTab) | Reads connections from Redux |
| **Discover** | `Discover.jsx` | Search users | Yes (input, users, loading) | Fetches users via API |
| **Profile** | `Profile.jsx` | User profile display | Yes (user, posts, activeTab, showEdit) | Fetches profile via API |
| **CreatePost** | `CreatePost.jsx` | Post creation form | Yes (content, images, codeSnippets, loading) | Submits post via API |
| **SinglePost** | `SinglePost.jsx` | Display single post | Yes (post, loading) | Fetches post by ID |

### **Reusable Components** (`client/src/components/`)

| Component | File | Role | Stateful? |
|-----------|------|------|-----------|
| **Sidebar** | `Sidebar.jsx` | Navigation sidebar | Presentational (receives props) |
| **MenuItems** | `MenuItems.jsx` | Navigation links | Presentational |
| **PostCard** | `PostCard.jsx` | Post display with interactions | Yes (likes, comments, menus) |
| **StoriesBar** | `StoriesBar.jsx` | Horizontal stories carousel | Yes (stories, modal states) |
| **StoryModal** | `StoryModal.jsx` | Create story form | Yes (mode, text, media, background) |
| **StoryViewer** | `StoryViewer.jsx` | View story fullscreen | Yes (progress) |
| **ProfileModal** | `ProfileModal.jsx` | Edit profile form | Yes (editForm) |
| **EditPostModal** | `EditPostModal.jsx` | Edit post form | Yes (content, images, codeSnippets) |
| **UserCard** | `UserCard.jsx` | User card in discover | Presentational |
| **UserProfileInfo** | `UserProfileInfo.jsx` | Profile header info | Presentational |
| **RecentMessages** | `RecentMessages.jsx` | Recent messages sidebar | Yes (messages) |
| **Notification** | `Notification.jsx` | Toast notification for messages | Presentational |
| **CodeSnippetEditor** | `CodeSnippetEditor.jsx` | Add code to posts | Yes (showForm, language, title, code) |
| **CodeSnippetDisplay** | `CodeSnippetDisplay.jsx` | Display code with syntax | Yes (copied, theme) |
| **ThemeToggle** | `ThemeToggle.jsx` | Dark/light mode button | Presentational (uses context) |
| **Loading** | `Loading.jsx` | Spinner component | Presentational |
| **TypeWriter** | `TypeWriter.jsx` | Animated text typing | Yes (displayedText, index) |

### Component Composition Patterns:

1. **Layout + Outlet Pattern** (`Layout.jsx` line 20):
   ```jsx
   <Outlet />  // React Router renders child routes here
   ```

2. **Render Props via Toast** (`App.jsx` lines 56-58):
   ```jsx
   toast.custom((t) => (
     <Notification t={t} message={message}/>
   ), {position: "bottom-right"})
   ```

3. **Compound Components**: `PostCard` contains `EditPostModal`, `CodeSnippetDisplay`

4. **Controlled Components**: All forms use controlled inputs with `useState`

---

## **4. STATE MANAGEMENT**

### **Global State: Redux Toolkit** (`@reduxjs/toolkit@2.8.2`)

### Store Configuration: `client/src/app/store.js`

```javascript
export const store = configureStore({
    reducer: {
       user: userReducer,           // Current user data
       connections: connectionsReducer,  // Followers, following, connections
       messages: messagesReducer    // Chat messages
    }
})
```

### Redux Slices:

| Slice | File | State Shape | Async Thunks |
|-------|------|-------------|--------------|
| **user** | `features/user/userSlice.js` | `{ value: User | null }` | `fetchUser`, `updateUser` |
| **connections** | `features/connections/connectionsSlice.js` | `{ connections, pendingConnections, followers, following }` | `fetchConnections` |
| **messages** | `features/messages/messagesSlice.js` | `{ messages: [] }` | `fetchMessages` |

### Async State Handling:

```javascript
// userSlice.js - Async thunk pattern (lines 10-15)
export const fetchUser = createAsyncThunk('user/fetchUser', async (token) => {
    const { data } = await api.get('/api/user/data', {
        headers: {Authorization: `Bearer ${token}`}
    })
    return data.success ? data.user : null
})
```

### Local State: React Context API

| Context | File | Purpose |
|---------|------|---------|
| **ThemeContext** | `context/ThemeContext.jsx` | Dark/light mode toggle |

### Custom Hook from Context:
```javascript
// ThemeContext.jsx lines 37-43
export const useTheme = () => {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider')
  }
  return context
}
```

### Data Flow Pattern:

```
User Action → dispatch(asyncThunk) → API Call → Redux State Update → Component Re-render
                                                       ↑
                                               useSelector() hooks
```

### Components Reading Global State:

| Component | Redux State Used |
|-----------|------------------|
| `Layout.jsx` | `state.user.value` |
| `Sidebar.jsx` | `state.user.value` |
| `Messages.jsx` | `state.connections.connections` |
| `ChatBox.jsx` | `state.messages`, `state.connections.connections` |
| `Connections.jsx` | `state.connections` (all) |
| `Profile.jsx` | `state.user.value` |
| `CreatePost.jsx` | `state.user.value` |
| `PostCard.jsx` | `state.user.value` |
| `UserCard.jsx` | `state.user.value` |
| `ProfileModal.jsx` | `state.user.value` |

---

## **5. API INTEGRATION — FRONTEND TO BACKEND CONNECTIONS**

### HTTP Client: `axios@1.11.0`
### API Configuration: `client/src/api/axios.js`

```javascript
const api = axios.create({
    baseURL: import.meta.env.VITE_BASEURL  // Environment variable
})
```

### **Complete API Call Table:**

| Frontend File | Line | Method + Endpoint | Trigger Function/Hook | Backend File | Handler |
|---------------|------|-------------------|----------------------|--------------|---------|
| **USER DATA** |
| `features/user/userSlice.js` | 11 | `GET /api/user/data` | `fetchUser` thunk | `controllers/userController.js` | `getUserData` |
| `features/user/userSlice.js` | 18 | `POST /api/user/update` | `updateUser` thunk | `controllers/userController.js` | `updateUserData` |
| `features/connections/connectionsSlice.js` | 13 | `GET /api/user/connections` | `fetchConnections` thunk | `controllers/userController.js` | `getUserConnections` |
| **DISCOVER/FOLLOW** |
| `pages/Discover.jsx` | 25 | `POST /api/user/discover` | `handleSearch` | `controllers/userController.js` | `discoverUsers` |
| `components/UserCard.jsx` | 20 | `POST /api/user/follow` | `handleFollow` | `controllers/userController.js` | `followUser` |
| `components/UserCard.jsx` | 40 | `POST /api/user/connect` | `handleConnectionRequest` | `controllers/userController.js` | `sendConnectionRequest` |
| `pages/Connections.jsx` | 29 | `POST /api/user/unfollow` | `handleUnfollow` | `controllers/userController.js` | `unfollowUser` |
| `pages/Connections.jsx` | 45 | `POST /api/user/accept` | `acceptConnection` | `controllers/userController.js` | `acceptConnectionRequest` |
| **PROFILE** |
| `pages/Profile.jsx` | 30 | `POST /api/user/profiles` | `fetchUser` (local) | `controllers/userController.js` | `getUserProfiles` |
| **POSTS** |
| `pages/Feed.jsx` | 21 | `GET /api/post/feed` | `fetchFeeds` | `controllers/postController.js` | `getFeedPosts` |
| `pages/CreatePost.jsx` | 41 | `POST /api/post/add` | `handleSubmit` | `controllers/postController.js` | `addPost` |
| `pages/SinglePost.jsx` | 15 | `GET /api/post/${postId}` | `fetchPost` | **MISSING BACKEND ROUTE** | — |
| `components/PostCard.jsx` | 27 | `POST /api/post/like` | `handleLike` | `controllers/postController.js` | `likePost` |
| `components/PostCard.jsx` | 55 | `POST /api/post/comment/add` | `handleAddComment` | `controllers/postController.js` | `addComment` |
| `components/PostCard.jsx` | 76 | `POST /api/post/comment/delete` | `handleDeleteComment` | `controllers/postController.js` | `deleteComment` |
| `components/PostCard.jsx` | 113 | `POST /api/post/delete` | `handleDelete` | `controllers/postController.js` | `deletePost` |
| `components/EditPostModal.jsx` | 32 | `POST /api/post/edit` | `handleSubmit` | `controllers/postController.js` | `editPost` |
| **STORIES** |
| `components/StoriesBar.jsx` | 22 | `GET /api/story/get` | `fetchStories` | `controllers/storyController.js` | `getStories` |
| `components/StoryModal.jsx` | 72 | `POST /api/story/create` | `handleCreateStory` | `controllers/storyController.js` | `addUserStory` |
| **MESSAGES** |
| `features/messages/messagesSlice.js` | 10 | `POST /api/message/get` | `fetchMessages` thunk | `controllers/messageController.js` | `getChatMessages` |
| `pages/ChatBox.jsx` | 44 | `POST /api/message/send` | `sendMessage` | `controllers/messageController.js` | `sendMessage` |
| `components/RecentMessages.jsx` | 18 | `GET /api/user/recent-messages` | `fetchRecentMessages` | `controllers/messageController.js` | `getUserRecentMessages` |
| **SSE (Real-time)** |
| `App.jsx` | 48 | `GET /api/message/:userId` (SSE) | `EventSource` | `controllers/messageController.js` | `sseController` |

### Real-time Connection (Server-Sent Events):
```javascript
// App.jsx lines 47-64
const eventSource = new EventSource(import.meta.env.VITE_BASEURL + '/api/message/' + user.id);
eventSource.onmessage = (event) => {
    const message = JSON.parse(event.data)
    // Dispatch to Redux or show notification
}
```

---

## **6. FORMS & USER INPUT**

### Form Library: **Native React** (no form library used)

| Form | File | Component | Data Collected | Validation | API Trigger |
|------|------|-----------|----------------|------------|-------------|
| **Login** | `Login.jsx` | `<SignIn />` | Email/Password | Clerk handles | Clerk handles |
| **Create Post** | `CreatePost.jsx` | Native form | content, images[], codeSnippets[] | Line 25-27: checks if at least one field | `POST /api/post/add` (line 41) |
| **Edit Post** | `EditPostModal.jsx` | Native form | content, images[], codeSnippets[] | Line 18-19 | `POST /api/post/edit` (line 32) |
| **Edit Profile** | `ProfileModal.jsx` | Native form | username, bio, location, full_name, social_media, profile_picture, cover_photo | None explicit | `POST /api/user/update` (via Redux thunk) |
| **Create Story** | `StoryModal.jsx` | Native form | text, media (image/video), background_color | Line 60-62: text required for text mode | `POST /api/story/create` (line 72) |
| **Send Message** | `ChatBox.jsx` | Input + file | text, image | Line 36: requires text or image | `POST /api/message/send` (line 44) |
| **Add Comment** | `PostCard.jsx` | Input form | content | Line 49: empty check | `POST /api/post/comment/add` (line 55) |
| **Search Users** | `Discover.jsx` | Input | input (search query) | None | `POST /api/user/discover` (line 25) |
| **Code Snippet** | `CodeSnippetEditor.jsx` | Native form | title, language, code | Line 34: code required | Parent component handles |

### Controlled Input Pattern Example:
```jsx
// ChatBox.jsx lines 107-108
<input 
  type="text" 
  onChange={(e) => setText(e.target.value)} 
  value={text}
  onKeyDown={e => e.key === 'Enter' && sendMessage()} 
/>
```

---

## **7. AUTHENTICATION FLOW (FRONTEND)**

### Auth Provider: **Clerk** (`@clerk/clerk-react@5.36.0`)

### Login UI Location:
- File: `client/src/pages/Login.jsx`
- Component: Clerk's `<SignIn />` (line 65)

### Token Storage:
- **Clerk handles token management internally** (not localStorage/sessionStorage)
- Access via `useAuth().getToken()` hook

### Protected Route Enforcement:
```jsx
// App.jsx line 71
<Route path='/' element={ !user ? <Login /> : <Layout/>}>
```
- Uses Clerk's `useUser()` hook to determine auth state
- No separate `PrivateRoute` component needed

### Auth State Sharing:
| Hook | Source | Usage |
|------|--------|-------|
| `useUser()` | `@clerk/clerk-react` | Returns `{ user }` object |
| `useAuth()` | `@clerk/clerk-react` | Returns `{ getToken }` function |
| `useClerk()` | `@clerk/clerk-react` | Returns `{ signOut }` function |

### Auth Flow Diagram:
```
Login.jsx
  ↓
<SignIn /> (Clerk component)
  ↓
Clerk authenticates user → user object populated
  ↓
App.jsx: useUser() returns user
  ↓
Route renders Layout instead of Login
  ↓
App.jsx useEffect (lines 30-40):
  - getToken() from Clerk
  - dispatch(fetchUser(token)) → GET /api/user/data
  - dispatch(fetchConnections(token)) → GET /api/user/connections
```

### Token Injection Pattern:
```javascript
// Every API call includes Bearer token
const { data } = await api.get('/api/post/feed', {
  headers: { Authorization: `Bearer ${await getToken()}` }
})
```

### Logout:
```jsx
// Sidebar.jsx line 38
<LogOut onClick={signOut} />  // signOut from useClerk()
```

---

## **8. STYLING & UI SYSTEM**

### Framework: **Tailwind CSS v4** with Vite plugin

### Configuration Files:
| File | Purpose |
|------|---------|
| `tailwind.config.js` | Dark mode (`class`), content paths |
| `index.css` | Tailwind imports, custom CSS |
| `vite.config.js` | Tailwind Vite plugin integration |

### Dark Mode Implementation:
```javascript
// tailwind.config.js line 2
darkMode: 'class'

// ThemeContext.jsx line 16
const html = document.documentElement
if (theme === 'dark') {
  html.classList.add('dark')
}
```

### Global Styles (`index.css`):
- Custom font: `Outfit` (Google Fonts)
- Hidden scrollbar utility: `.no-scrollbar::-webkit-scrollbar`
- Animated gradient text: `.hero-gradient-text`

### Icon Library: **Lucide React** (`lucide-react@0.525.0`)

### Design System / Component Library: **None** (custom components)

### Responsive Design:
```jsx
// Layout.jsx responsive sidebar (line 16)
<div className={`w-60 xl:w-72 ... max-sm:absolute ... 
  ${sidebarOpen ? 'translate-x-0' : 'max-sm:-translate-x-full'}`}>
```

Breakpoint prefixes used:
- `sm:` (≥640px)
- `md:` (≥768px)
- `lg:` (≥1024px)
- `xl:` (≥1280px)
- `max-sm:`, `max-md:`, `max-xl:` (max-width variants)

### Reusable UI Primitives:
| Component | File | Purpose |
|-----------|------|---------|
| `Loading` | `components/Loading.jsx` | Spinner |
| `ThemeToggle` | `components/ThemeToggle.jsx` | Dark/light toggle button |
| `TypeWriter` | `components/TypeWriter.jsx` | Animated typing effect |

### Color Palette (from usage patterns):
- Primary: `indigo-500` → `purple-600` gradient
- Background: `slate-50` (light), `gray-900` (dark)
- Text: `gray-900` (light), `white/gray-200` (dark)

---

## **9. FRONTEND ARCHITECTURE MAP**

```
Browser
    ↓
index.html (line 11)
    ↓
main.jsx
    ↓
┌─────────────────────────────────────────────────────┐
│  PROVIDER HIERARCHY                                  │
│  ┌──────────────────────────────────────────────┐   │
│  │ ClerkProvider (Auth)                          │   │
│  │  ┌────────────────────────────────────────┐  │   │
│  │  │ BrowserRouter (Routing)                 │  │   │
│  │  │  ┌──────────────────────────────────┐  │  │   │
│  │  │  │ Provider (Redux Store)            │  │  │   │
│  │  │  │  ┌────────────────────────────┐  │  │  │   │
│  │  │  │  │ ThemeProvider (Context)    │  │  │  │   │
│  │  │  │  │                            │  │  │  │   │
│  │  │  │  │  ┌──────────────────────┐  │  │  │  │   │
│  │  │  │  │  │ App.jsx              │  │  │  │  │   │
│  │  │  │  │  │                      │  │  │  │  │   │
│  │  │  │  │  │ Routes               │  │  │  │  │   │
│  │  │  │  │  │  ├─ Login            │  │  │  │  │   │
│  │  │  │  │  │  └─ Layout           │  │  │  │  │   │
│  │  │  │  │  │       ├─ Sidebar     │  │  │  │  │   │
│  │  │  │  │  │       └─ Outlet      │  │  │  │  │   │
│  │  │  │  │  │            ├─ Feed   │  │  │  │  │   │
│  │  │  │  │  │            ├─ ...    │  │  │  │  │   │
│  │  │  │  │  └──────────────────────┘  │  │  │  │   │
│  │  │  │  └────────────────────────────┘  │  │  │   │
│  │  │  └──────────────────────────────────┘  │  │   │
│  │  └────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
    ↓
Page Component (Feed.jsx, Profile.jsx, etc.)
    ↓
Child Components (PostCard, StoriesBar, etc.)
    ↓
Hooks (useAuth, useSelector, useState)
    ↓
API Call (axios via api/axios.js)
    ↓
Backend REST API (Express server)
    ↓
UI Update (Redux state → re-render)
```

### Actual File Flow (Login → Dashboard):
```
main.jsx
    ↓
App.jsx (useEffect line 30-40 triggers data fetch)
    ↓
!user → Login.jsx
    ↓
<SignIn /> (Clerk handles login)
    ↓
user populated → Layout.jsx
    ↓
Layout renders: Sidebar.jsx + <Outlet/>
    ↓
Outlet renders: Feed.jsx (index route)
    ↓
Feed.jsx useEffect (line 34-36) → fetchFeeds()
    ↓
api.get('/api/post/feed') (line 21)
    ↓
Backend: postController.getFeedPosts
    ↓
setFeeds(data.posts) → re-render with PostCard components
```

---

## **10. FRONTEND–BACKEND CONNECTION MAP**

### **AUTH FEATURE**
```
[pages/Login.jsx : 65]
  → Clerk <SignIn /> component handles auth externally

[App.jsx : 33-35]
  → dispatch(fetchUser(token))
  → dispatch(fetchConnections(token))
  → Backend: userController.getUserData, getUserConnections
  → Purpose: Load user data on app mount
```

### **USER PROFILE FEATURE**
```
[features/user/userSlice.js : 11]
  → GET /api/user/data
  → Backend: controllers/userController.js : getUserData
  → Purpose: Fetch current user profile

[features/user/userSlice.js : 18]
  → POST /api/user/update
  → Backend: controllers/userController.js : updateUserData
  → Purpose: Update profile (name, bio, photos, social)

[pages/Profile.jsx : 30]
  → POST /api/user/profiles
  → Backend: controllers/userController.js : getUserProfiles
  → Purpose: Fetch any user's profile and posts
```

### **CONNECTIONS FEATURE**
```
[features/connections/connectionsSlice.js : 13]
  → GET /api/user/connections
  → Backend: controllers/userController.js : getUserConnections
  → Purpose: Get followers, following, connections, pending

[pages/Discover.jsx : 25]
  → POST /api/user/discover
  → Backend: controllers/userController.js : discoverUsers
  → Purpose: Search users by name, username, bio, location

[components/UserCard.jsx : 20]
  → POST /api/user/follow
  → Backend: controllers/userController.js : followUser
  → Purpose: Follow a user

[components/UserCard.jsx : 40]
  → POST /api/user/connect
  → Backend: controllers/userController.js : sendConnectionRequest
  → Purpose: Send connection request

[pages/Connections.jsx : 29]
  → POST /api/user/unfollow
  → Backend: controllers/userController.js : unfollowUser
  → Purpose: Unfollow a user

[pages/Connections.jsx : 45]
  → POST /api/user/accept
  → Backend: controllers/userController.js : acceptConnectionRequest
  → Purpose: Accept pending connection request
```

### **POSTS FEATURE**
```
[pages/Feed.jsx : 21]
  → GET /api/post/feed
  → Backend: controllers/postController.js : getFeedPosts
  → Purpose: Get posts from user + connections + following

[pages/CreatePost.jsx : 41]
  → POST /api/post/add
  → Backend: controllers/postController.js : addPost
  → Purpose: Create new post with text, images, code snippets

[components/PostCard.jsx : 27]
  → POST /api/post/like
  → Backend: controllers/postController.js : likePost
  → Purpose: Like/unlike a post

[components/PostCard.jsx : 55]
  → POST /api/post/comment/add
  → Backend: controllers/postController.js : addComment
  → Purpose: Add comment to post

[components/PostCard.jsx : 76]
  → POST /api/post/comment/delete
  → Backend: controllers/postController.js : deleteComment
  → Purpose: Delete own comment

[components/PostCard.jsx : 113]
  → POST /api/post/delete
  → Backend: controllers/postController.js : deletePost
  → Purpose: Delete own post

[components/EditPostModal.jsx : 32]
  → POST /api/post/edit
  → Backend: controllers/postController.js : editPost
  → Purpose: Edit own post
```

### **STORIES FEATURE**
```
[components/StoriesBar.jsx : 22]
  → GET /api/story/get
  → Backend: controllers/storyController.js : getStories
  → Purpose: Get stories from connections/following

[components/StoryModal.jsx : 72]
  → POST /api/story/create
  → Backend: controllers/storyController.js : addUserStory
  → Purpose: Create new story (text, image, video)
```

### **MESSAGING FEATURE**
```
[features/messages/messagesSlice.js : 10]
  → POST /api/message/get
  → Backend: controllers/messageController.js : getChatMessages
  → Purpose: Get chat history with a user

[pages/ChatBox.jsx : 44]
  → POST /api/message/send
  → Backend: controllers/messageController.js : sendMessage
  → Purpose: Send message (text or image)

[components/RecentMessages.jsx : 18]
  → GET /api/user/recent-messages
  → Backend: controllers/messageController.js : getUserRecentMessages
  → Purpose: Get recent messages for sidebar

[App.jsx : 48]
  → GET /api/message/:userId (SSE EventSource)
  → Backend: controllers/messageController.js : sseController
  → Purpose: Real-time message notifications
```

---

## **11. FRONTEND CONCEPT EXTRACTION**

| Concept | File Location | Explanation |
|---------|---------------|-------------|
| **Component Composition** | `Layout.jsx` → `Sidebar` + `Outlet` | Parent wraps children, Outlet renders nested routes |
| **Controlled Inputs** | `CreatePost.jsx:77`, `ChatBox.jsx:107-108` | Input values tied to state, `onChange` updates state |
| **Custom Hooks** | `context/ThemeContext.jsx:37-43` | `useTheme()` - context consumer hook |
| **Code Splitting / Lazy Loading** | Not implemented | All components eagerly loaded |
| **Optimistic UI Updates** | `PostCard.jsx:31-37` | Local `likes` state updated before API confirms |
| **Error Boundaries** | Not implemented | Uses toast notifications for errors |
| **Data Fetching Patterns** | Redux Thunks (`userSlice.js:10-15`) | `createAsyncThunk` for async API calls |
| **Context Propagation** | `ThemeContext.jsx` | Theme state shared across app via context |
| **Memoization** | Not extensively used | No `useMemo`, `useCallback`, or `React.memo` found |
| **Refs for DOM** | `ChatBox.jsx:21, 75-76` | `messagesEndRef` for auto-scroll |
| **Server-Sent Events** | `App.jsx:48-64` | `EventSource` for real-time messages |
| **Environment Variables** | `main.jsx:11`, `axios.js:4` | `import.meta.env.VITE_*` |
| **Form Submission** | `ProfileModal.jsx:68-70` | `toast.promise()` for loading state UX |
| **File Upload** | `StoryModal.jsx:22-55` | FileReader with validation for size/duration |
| **URL State** | `Profile.jsx:21` | `useParams()` for `:profileId` |
| **Navigation** | `UserCard.jsx:36`, `Notification.jsx:24` | `useNavigate()` hook |
| **Conditional Rendering** | `App.jsx:71`, `Layout.jsx:14` | Ternary and `&&` patterns |
| **Array Rendering** | `Feed.jsx:44-46` | `.map()` with key prop |
| **Event Handling** | `ChatBox.jsx:108` | `onKeyDown` for enter key submit |
| **CSS-in-JS (Tailwind)** | All components | Utility classes for styling |
| **Dark Mode** | `ThemeContext.jsx`, `tailwind.config.js` | Class-based dark mode |
| **Toast Notifications** | `PostCard.jsx:30, 61` | `react-hot-toast` for feedback |
| **Time Formatting** | `PostCard.jsx:140` | `moment().fromNow()` |
| **Clipboard API** | `PostCard.jsx:105`, `CodeSnippetDisplay.jsx:17` | `navigator.clipboard.writeText()` |
| **Object URL for Preview** | `CreatePost.jsx:84`, `StoryModal.jsx:42` | `URL.createObjectURL(file)` |
| **FormData for Uploads** | `CreatePost.jsx:33-39` | `new FormData()` with file appending |

---

## **12. LEARNING ANALYSIS**

### **A. Frontend Concepts to Study Without Backend**

| Concept | Files to Study | What You'll Learn |
|---------|----------------|-------------------|
| Component Structure | `components/Loading.jsx`, `components/TypeWriter.jsx` | Simple functional components |
| State Management | `context/ThemeContext.jsx` | React Context pattern |
| Tailwind Styling | `index.css`, `components/ThemeToggle.jsx` | Utility-first CSS, dark mode |
| Redux Setup | `app/store.js`, `features/*/Slice.js` | Redux Toolkit patterns |
| Props & Children | `components/MenuItems.jsx`, `pages/Layout.jsx` | Data passing, composition |
| React Hooks | `components/StoryViewer.jsx` | `useState`, `useEffect` for timers |
| Form Handling | `components/CodeSnippetEditor.jsx` | Controlled forms without library |
| Animations | `components/TypeWriter.jsx` | Timer-based text animation |

### **B. Best Starting Points for Frontend-Backend Integration**

| File | Why Start Here |
|------|----------------|
| `api/axios.js` | See how base URL is configured, understand the HTTP client setup |
| `features/user/userSlice.js` | See complete async thunk pattern with API call, error handling, state update |
| `pages/Feed.jsx` | Simple GET request, async/await, loading state, conditional rendering |
| `components/PostCard.jsx` | Multiple API calls (like, comment, delete), optimistic updates |
| `App.jsx` lines 46-65 | Real-time SSE connection, complex useEffect |

### **C. Recommended Reading Order**

#### **Step 1: Understand App Bootstrap (5 files)**
1. `index.html` — Entry HTML
2. `main.jsx` — Provider hierarchy
3. `App.jsx` — Routes and global effects
4. `app/store.js` — Redux store setup
5. `context/ThemeContext.jsx` — Context pattern

#### **Step 2: Authentication Flow (3 files)**
6. `pages/Login.jsx` — Clerk integration
7. `pages/Layout.jsx` — Protected layout wrapper
8. `components/Sidebar.jsx` — Auth-dependent UI (signout)

#### **Step 3: Data Fetching (4 files)**
9. `api/axios.js` — HTTP client config
10. `features/user/userSlice.js` — Redux async thunk
11. `features/connections/connectionsSlice.js` — Multiple state fields
12. `pages/Feed.jsx` — Local state data fetching

#### **Step 4: Complete User Flow (6 files)**
13. `pages/Discover.jsx` — Search + display users
14. `components/UserCard.jsx` — User actions (follow, connect)
15. `pages/Connections.jsx` — Tab UI, multiple actions
16. `pages/Profile.jsx` — URL params, conditional fetching
17. `components/ProfileModal.jsx` — Complex form, file upload
18. `components/PostCard.jsx` — Rich interactions

#### **Step 5: Real-Time Features (3 files)**
19. `pages/ChatBox.jsx` — Messaging UI
20. `features/messages/messagesSlice.js` — Message state
21. `App.jsx` lines 46-65 — SSE for real-time

#### **Step 6: Advanced Patterns (3 files)**
22. `components/StoriesBar.jsx` — Complex state (stories, modals)
23. `components/StoryModal.jsx` — File validation, FormData
24. `components/CodeSnippetDisplay.jsx` — Local theme toggle

---

## **Summary**

This is a **modern React social platform** with:
- **Authentication**: Clerk (zero-config auth)
- **State**: Redux Toolkit for global state, Context for theme
- **Routing**: React Router v7 with protected routes
- **Styling**: Tailwind CSS v4 with dark mode
- **API**: Axios with JWT Bearer token auth
- **Real-time**: Server-Sent Events for messages
- **Backend**: Express + MongoDB (Mongoose)

**Notable Architectural Decisions:**
1. No form library — uses native controlled inputs
2. No component library — custom Tailwind components
3. Uses Clerk for auth instead of rolling custom JWT
4. SSE instead of WebSockets for real-time (simpler, one-directional)
5. Redux only for truly global state (user, connections, messages)

**Missing/Potential Issues Found:**
1. `SinglePost.jsx` calls `GET /api/post/:postId` but no backend route exists for it
2. No error boundaries for component failures
3. No memoization (could optimize with `React.memo` for `PostCard`)
4. `RecentMessages.jsx` has interval that doesn't clear properly (line 47)
