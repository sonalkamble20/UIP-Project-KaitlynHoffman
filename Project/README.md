# 🌐 Social Media Platform — UIP Project

> A full-stack social media web application with user profiles, posts, activity (comments), and a React frontend backed by a Node.js/Express API and MongoDB.

**Built by:** Sonal Kamble  
**Course:** User Interface Programming (UIP) — Kaitlin Hoffman  
**Stack:** Node.js · Express.js · MongoDB · Mongoose · React · Bootstrap 5

---

## 📖 What Is This App?

This is a **simplified social media platform** where users can:

- **Register** a new account with a hashed password (bcrypt)
- **Log in** and navigate to a personal profile page
- **View their profile** — including follower/following counts, posts, and activity
- **Create posts** (text content published to their profile)
- **Delete posts** they've authored
- **Add activity comments** tied to posts
- **Delete activity/comments** from their feed

It is a **MERN-style** full-stack app:
- **M** — MongoDB (local, via Mongoose)
- **E** — Express.js
- **R** — React (Create React App, v19)
- **N** — Node.js

---

## 🗂️ Project Structure

```
Project/
├── index.js                        # 🚀 Express server entry point (port 5000)
├── .env                            # Environment variables (MongoDB URL, port)
├── package.json                    # Root dependencies (server-side)
├── ERD.png                         # Entity Relationship Diagram
├── mongod.log                      # MongoDB server log
│
├── server/                         # Backend API
│   ├── models/
│   │   ├── user.js                 # User schema + CRUD (register, login, updatePassword, deleteUser, getUserProfile)
│   │   ├── post.js                 # Post schema + CRUD (uploadPost, openPost, updatePost, deletePost)
│   │   └── activity.js            # Activity/Comment schema + CRUD
│   └── routes/
│       ├── user.js                 # /user — login, register, update, delete, profile
│       ├── post.js                 # /post — open, upload, update, delete
│       └── activity.js            # /activity — createComment, comments by user, updateComment, deleteComment
│
└── client/                         # React frontend (Create React App)
    ├── package.json                # Client dependencies (React 19, Bootstrap 5, react-router-dom 7)
    └── src/
        ├── index.js                # ReactDOM.render root
        ├── App.js                  # Router setup with BrowserRouter + Routes
        ├── App.css
        └── components/
            └── pages/
                ├── Navbar.js       # 🔗 Top navigation (Home, Login, Register links)
                ├── LoginForm.js    # 🔐 Login form → navigates to /profile/:username
                ├── RegisterForm.js # 📝 Registration form
                ├── Profile.js      # 👤 Full profile view (posts + activities)
                └── Profile.css     # Profile page styling
```

---

## ⚙️ How It Works

### 1. User Authentication

No JWT or session tokens are used — authentication is done per-request by username credential matching, and the session state is managed by React state (username stored in URL).

```
Register → POST /user/register → bcrypt hashes password → creates User in MongoDB
Login    → POST /user/login    → bcrypt.compare() validates → returns user object (no password)
                              → React navigates to /profile/:username
```

**Password Security:** ✅ bcryptjs with salt rounds of 10.

### 2. Profile Page

The `Profile.js` React component is the heart of the app. When a user logs in, they land at `/profile/:username`. On mount, it:

1. Fetches `GET /user/profile/:username`
2. The server returns `{ user, posts, activities }` in one call
3. Renders follower count, following count, posts list, and activities list

From the profile page, users can:
- Submit the "Write something new..." form → `POST /post/upload`
- Click "Delete" on a post → `DELETE /post/delete` (with `postid`)
- Submit the "Add a comment/activity..." form → `POST /activity/createComment`
- Click "Delete" on an activity → `DELETE /activity/deleteComment`

All changes update React state optimistically (re-renders instantly without refetching the entire profile).

### 3. Data Models (MongoDB / Mongoose)

#### `User`
```
username: String (unique, required)
password: String (hashed with bcrypt)
followers: [String]
following:  [String]
```

#### `Post`
```
username:  String (required)
postid:    String (unique UUID, auto-generated)
content:   String
likes:     Number (default: 0)
comments:  [String]
createdAt: Date (Mongoose timestamps)
updatedAt: Date
```

#### `Activity` (Comments)
```
username:  String (required)
postid:    String (required — links to a Post)
comment:   String (required)
createdAt: Date (Mongoose timestamps)
```

### 4. React Routing

```
/            → <Navbar />  (home landing with links)
/login       → <LoginForm />
/register    → <RegisterForm />
/profile/:username → <Profile />
```

Client-side routing is handled by **react-router-dom v7** (BrowserRouter). The React dev server proxies API calls to `http://localhost:5000` (configured in `package.json`).

---

## 📡 API Reference

### User Routes (`/user`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/user/login` | Login: validates credentials, returns user (no password) |
| POST | `/user/register` | Register: hashes password, creates user |
| PUT | `/user/update` | Update password (requires id in body) |
| DELETE | `/user/delete` | Delete user account (requires id in body) |
| GET | `/user/profile/:username` | Get user + their posts + their activities |

### Post Routes (`/post`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/post/open` | Fetch a single post by `postid` |
| POST | `/post/upload` | Create a new post (requires content + username) |
| PUT | `/post/update` | Update post content by `postid` |
| DELETE | `/post/delete` | Delete a post by `postid` |

### Activity Routes (`/activity`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/activity/createComment` | Create a comment (username, comment, postid) |
| GET | `/activity/comments/:username` | Get all comments by a user |
| PUT | `/activity/updateComment` | Update a comment by `activityId` |
| DELETE | `/activity/deleteComment` | Delete a comment by `activityId` |

---

## 🗄️ Database

- **Type:** MongoDB (local)
- **URI:** `mongodb://localhost:27017/social-media`
- **Collections:** `users`, `posts`, `activities`

The `mongo-data/` folder in the parent workspace stores the WiredTiger engine files backing this database — 6 active collections with indexes.

---

## 🚀 Getting Started

### Prerequisites
- Node.js (v18+)
- MongoDB (running locally on port 27017)

### 1. Install Server Dependencies

```bash
cd Project
npm install
```

### 2. Install Client Dependencies

```bash
cd client
npm install
```

### 3. Configure `.env` (root of `Project/`)

```env
dbURL=mongodb://localhost:27017/social-media
PORT=5000
```

### 4. Run the App (both server + client together)

From the `Project/` directory:

```bash
npm run dev
```

This runs both:
- **Server** (`nodemon index.js`) on **http://localhost:5000**
- **React client** (`npm start --prefix client`) on **http://localhost:3000**

> The React dev server proxies all API requests to port 5000.

### 5. Access the App

Open **http://localhost:3000** in your browser.

---

## 🛠️ Tech Stack Details

| Layer | Technology |
|-------|-----------|
| Server | Node.js + Express v5 |
| Database | MongoDB + Mongoose ODM |
| Auth | bcryptjs (password hashing) |
| Frontend | React 19 (Create React App) |
| UI Components | Bootstrap 5 |
| Routing | react-router-dom v7 |
| Dev Tools | nodemon, concurrently |
| API proxy | CRA proxy → localhost:5000 |

---

## 📌 Known Improvements / Future Work

- [ ] Add JWT or session-based authentication to protect API routes
- [ ] Implement follow/unfollow functionality (schema fields exist: `followers`, `following`)
- [ ] Add likes functionality (schema field exists: `likes` on Post)
- [ ] Add real-time updates using WebSockets or polling
- [ ] Implement a global feed (all public posts, not just profile posts)
- [ ] Deploy client and server separately (e.g., Vercel for client, Railway for server + Atlas for MongoDB)
- [ ] Add image upload support for posts
- [ ] Improve error handling and add form validation feedback

---

## 👩‍💻 Author

**Sonal Kamble**  
*Graduate CS Student — SUNY New Paltz*  
Course: User Interface Programming — Instructor: Kaitlin Hoffman