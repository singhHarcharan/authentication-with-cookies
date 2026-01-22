# MERN Authentication System

This project demonstrates two approaches to authentication in a MERN (MongoDB, Express.js, React, Node.js) application:
1. JWT with localStorage
2. Cookie-based authentication

## Table of Contents
1. [Authentication Overview](#authentication-overview)
2. [JWT with localStorage](#jwt-with-localstorage)
   - [Signup Flow](#signup-flow)
   - [Signin Flow](#signin-flow)
   - [Authenticated Requests](#authenticated-requests)
3. [Cookie-based Authentication](#cookie-based-authentication)
   - [What are Cookies?](#what-are-cookies)
   - [Cookie Properties](#cookie-properties)
   - [Implementation](#implementation)
4. [Security Considerations](#security-considerations)
5. [Setup and Usage](#setup-and-usage)

## Authentication Overview

Authentication is the process of verifying a user's identity, typically through credentials like username/password. This project demonstrates two modern approaches to handle authentication in web applications:

1. **JWT with localStorage**:
   - Stateless authentication using JSON Web Tokens
   - Tokens stored in browser's localStorage
   - Requires manual token management

2. **Cookie-based Authentication**:
   - Uses HTTP-only cookies for secure token storage
   - Automatic token management by the browser
   - Better protection against XSS attacks

## JWT with localStorage

### Signup Flow
1. Client sends user credentials to `/signup` endpoint
2. Server validates and creates user in database
3. Server generates JWT containing user info
4. JWT is sent back to client and stored in localStorage

```javascript
// Server-side
const newUser = await User.create({ username, email, password });
const token = generateJWT({ userId: newUser.id, email: newUser.email });
res.json({ token });
```

### Signin Flow
1. Client sends credentials to `/signin` endpoint
2. Server verifies credentials
3. If valid, generates and returns JWT
4. Client stores JWT in localStorage

```javascript
// Server-side
const user = await User.findOne({ email });
if (!user || !user.validatePassword(password)) {
  return res.status(401).json({ error: 'Invalid credentials' });
}
const token = generateJWT({ userId: user.id, email: user.email });
res.json({ token });
```

### Authenticated Requests
For subsequent requests, include the JWT in the Authorization header:

```javascript
// Client-side
const token = localStorage.getItem('token');
fetch('/protected-route', {
  headers: { 'Authorization': `Bearer ${token}` }
});
```

## Cookie-based Authentication

### What are Cookies?
- Small pieces of data stored by the browser
- Automatically sent with every request to the server
- Ideal for session management and authentication

### Cookie Properties
- **Secure**: Only sent over HTTPS
- **HttpOnly**: Inaccessible to JavaScript (prevents XSS)
- **SameSite**: Controls cross-site request behavior
  - `Strict`: Only same-site requests
  - `Lax`: Allows top-level navigation (default)
  - `None`: Allows cross-site requests (requires Secure)

### Implementation

#### Server Setup
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');
const cors = require('cors');
const jwt = require('jsonwebtoken');

const app = express();
app.use(cookieParser());
app.use(express.json());
app.use(cors({
    credentials: true,
    origin: "http://localhost:5173"
}));
```

#### Signin Endpoint
```javascript
app.post("/signin", (req, res) => {
    const { email, password } = req.body;
    // Validate credentials
    const token = jwt.sign({ id: 1 }, process.env.JWT_SECRET);
    res.cookie("token", token, { 
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'lax'
    });
    res.send("Logged in!");
});
```

#### Protected Route
```javascript
app.get("/user", (req, res) => {
    const token = req.cookies.token;
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    res.json({ userId: decoded.id });
});
```

## Security Considerations

### JWT with localStorage
- **Pros**:
  - Stateless authentication
  - Works well with mobile apps
- **Cons**:
  - Vulnerable to XSS attacks
  - Requires manual token management

### Cookie-based Authentication
- **Pros**:
  - More secure against XSS (with HttpOnly)
  - Automatic CSRF protection with SameSite
  - Built-in session management
- **Cons**:
  - Requires CSRF protection if not using SameSite=Strict
  - Slightly more complex setup

### Best Practices
1. Always use HTTPS in production
2. Implement proper CORS policies
3. Use secure cookie flags (HttpOnly, Secure, SameSite)
4. Implement rate limiting on auth endpoints
5. Use short-lived access tokens with refresh tokens

## Setup and Usage

### Backend Setup
```bash
cd backend
npm install
npm start
```

### Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

### Environment Variables
Create a `.env` file in the backend directory:
```
JWT_SECRET=your-secret-key
NODE_ENV=development
```

## Conclusion

This project demonstrates two common authentication approaches in modern web applications. While both methods have their use cases, cookie-based authentication with proper security headers is generally recommended for web applications due to its built-in security features and better protection against common web vulnerabilities.