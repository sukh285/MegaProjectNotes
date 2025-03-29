# 📌 Package Management Project - Backend Setup 
## 29 March 2025
---

## 🏗️ Project Structure

```
.
│-- .prettierrc          # Prettier config file for code formatting
│-- .prettierignore      # Files to ignore for Prettier
│-- .env                 # Environment variables
│-- .env.example         # Example env file
│-- .env.local           # Local environment variables
│-- public/              # Public assets like images, favicons
│   ├── images/          # Store images here
│   ├── .gitkeep         # Ensures Git tracks empty directories
│-- src/                 # Source code
│   ├── controllers/     # Controllers for request handling
│   ├── db/              # Database connection
│   ├── models/          # Mongoose models
│   ├── middlewares/     # Express middlewares
│   ├── routes/          # Express routes
│   ├── utils/           # Utility functions
│   ├── validators/      # Request validation
│   ├── app.js           # Initializes and exports Express app
│   ├── index.js         # Entry point of the app
```

### 📌 Reasoning:
- **Prettier & .prettierignore:** Ensures consistent code formatting.
- **Environment Files (.env, .env.example, .env.local):** Manages configurations and keeps sensitive data secure.
- **Public Folder:** Stores assets, favicons, and images.
- **GitKeep in Images Folder:** Ensures Git doesn't ignore empty directories.
- **Src Folder:** Organizes core backend functionality.

---

## 🛠️ Setting up Express App

### 🔹 app.js
- Initialize Express and export it.
- Configure middleware (cookies, CORS, rate limiting, etc.).
- Use routers.

### 🔹 index.js

```javascript
import dotenv from 'dotenv';
import app from './app';
import connectDB from './db';

dotenv.config({ path: "./.env" });
const PORT = process.env.PORT || 8000;

connectDB()
    .then(() => {
        app.listen(PORT, () => console.log(`🚀 Server running on port ${PORT}`));
    })
    .catch(error => {
        console.error("Database connection failed", error);
        process.exit(1);
    });
```

### 📌 Reasoning:
- **dotenv.config():** Loads environment variables.
- **PORT Configuration:** Ensures there are no conflicts when multiple services are running.
- **Using `.then()`:** Async functions return a promise, making `.then()` chaining possible.
- **process.exit(1):** Stops the server when database connection fails.

---

## 📦 Database Setup

### 🔹 db/index.js

```javascript
import mongoose from 'mongoose';

const connectDB = async () => {
    try {
        await mongoose.connect(process.env.MONGO_URL);
        console.log("✅ Database Connected Successfully");
    } catch (error) {
        console.error("❌ Database Connection Failed", error);
        process.exit(1);
    }
};

export default connectDB;
```

### 📌 Reasoning:
- **Mongoose Connection:** Handles MongoDB connection asynchronously.
- **Environment Variable Usage:** Uses `process.env.MONGO_URL` for flexibility.
- **process.exit(1):** Ensures that if the database connection fails, the server does not continue running.

---

## 🔥 Error Handling

### 🔹 utils/api-error.js

```javascript
class ApiError extends Error {
    constructor(statusCode, message, errors = [], stack = "") {
        super(message);
        this.statusCode = statusCode;
        this.message = message;
        this.success = false;
        this.errors = errors;
        if (stack) {
            this.stack = stack;
        } else {
            Error.captureStackTrace(this, this.constructor);
        }
    }
}
export default ApiError;
```

### 📌 Reasoning:
- **Extends Error Class:** Standardizes error handling across the application.
- **Fixes Incorrect Property Assignment:** Corrects `this.errors = errors`.
- **Stack Trace Capture:** Helps debug by tracking the error source.

---

## ✅ Standardized API Responses

### 🔹 utils/api-response.js

```javascript
class ApiResponse {
    constructor(statusCode, data, message = "Success") {
        this.statusCode = statusCode;
        this.data = data;
        this.message = message;
        this.success = statusCode < 400;
    }
}
export default ApiResponse;
```

### 📌 Reasoning:
- **Standardized API Response Format:** Ensures consistent structure.
- **Derived Success Property:** Simplifies response success determination.

---

## 📂 Controllers

### 🔹 HealthCheck Controller

```javascript
import ApiResponse from '../utils/api-response.js';

const healthCheck = (req, res) => {
    res.status(200).json(new ApiResponse(200, { uptime: process.uptime() }, "Server is healthy"));
};
export default healthCheck;
```

### 📌 Reasoning:
- **Provides Additional Metadata:** Adds uptime for better monitoring.
- **Uses ApiResponse Class:** Maintains consistency in API responses.

---

## 🔑 Middleware & Security

### 🔹 Adding CORS & Rate Limiting

```javascript
import cors from 'cors';
import rateLimit from 'express-rate-limit';
import healthCheckRouter from './routes/healthcheck.js';

const app = express();
app.use(cors()); // Enable CORS
app.use(express.json());

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // Max requests per window per IP
});
app.use(limiter);

app.use("/api/v1/healthCheck", healthCheckRouter);
export default app;
```

### 📌 Reasoning:
- **CORS:** Allows secure cross-origin resource sharing.
- **Rate Limiting:** Prevents API abuse and explains limits.

---

## ✅ Conclusion

This guide provides a clean structure and setup for a project management backend using Express, MongoDB, and Node.js. Follow these best practices for scalability and maintainability!
