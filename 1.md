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
- **Prettier & .prettierignore:** Ensures consistent code formatting  
- **Environment Files:** Manages configurations and keeps sensitive data secure  
- **Public Folder:** Stores assets, favicons, and images  
- **GitKeep in Images Folder:** Ensures Git tracks empty directories  
- **Src Folder:** Organizes core backend functionality  

---

## 🛠️ Setting up Express App  

### 🔹 app.js  
- Initialize Express and export it  
- Configure middleware (cookies, CORS, rate limiting, etc.)  
- Use routers  

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
- **dotenv.config():** Loads environment variables  
- **PORT Configuration:** Prevents port conflicts  
- **Using `.then()`:** Handles async database connection  
- **process.exit(1):** Stops server on DB connection failure  

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
- **Mongoose Connection:** Async MongoDB connection  
- **Environment Variable:** Flexible DB URL configuration  
- **process.exit(1):** Prevents server from running without DB  

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
- **Standardized Errors:** Consistent error structure  
- **Stack Trace:** Better debugging capabilities  
- **Extends Error:** Native error handling benefits  

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
- **Uniform Responses:** Predictable API output  
- **Auto Success Detection:** Based on status code  
- **Clean Structure:** Easy to maintain and extend  

---

## 🧱 Utility Files (utils/)

### 🔹 constants.js

```javascript
export const UserRolesEnum = {
    ADMIN : 'admin',
    PROJECT_ADMIN : 'project_admin',
    MEMBER : 'member'
};

export const AvailableUserRoles = Object.values(UserRolesEnum);

export const TaskStatusEnum = {
    TODO: 'todo',
    IN_PROGRESS: 'in_progress',
    DONE: 'done',
};

export const AvailableTaskRoles = Object.values(TaskStatusEnum);
```

### 📌 Reasoning:  
- **Enums for Roles and Task Status:** Provides centralized, typo-proof references  
- **Object.values usage:** Generates arrays for validation and filtering  

---

### 🔹 asyncHandler.js

```javascript
/**
 * A utility function to handle asynchronous route handlers in Express.
 * This function wraps an asynchronous function and ensures that any errors
 * occurring in the asynchronous code are passed to the `next` middleware,
 * which is typically an error-handling middleware in Express.
 *
 * @param {Function} requestHandler - The asynchronous route handler function
 * @returns {Function} A new function that wraps the original handler and catches errors
 */
const asyncHandler = (requestHandler) => {
    return (req, res, next) => {
        Promise.resolve(requestHandler(req, res, next)).catch((err) => next(err));
    };
};

export { asyncHandler };
```

### 📌 Reasoning:  
- **Simplifies Error Handling:** Eliminates repetitive try-catch blocks  
- **Middleware Integration:** Ensures errors reach Express error handlers  


---

## 🗃️ Model Creation Process

### 🔹 File Creation
Created model files via terminal using:
```powershell
New-Item -ItemType File user.models.js, task.models.js, subtask.models.js, project.models.js, note.models.js projectmember.models.js
```

### 🔹 Base Schema Setup
Initialized all models with basic structure:
```javascript
import mongoose, {Schema} from "mongoose";

const userSchema = new Schema({})

export const User = mongoose.model("User", userSchema)
```
(Copied to all other model files with appropriate name changes)

---

## 📝 ProjectNote Schema Example

### 🔹 Complete Implementation
```javascript
import mongoose, { Schema } from "mongoose";

const projectNoteSchema = new Schema(
  {
    project: {
      type: Schema.Types.ObjectId, // Reference to Project model
      ref: "Project",
      required: true,
    },
    createdBy: {
      type: Schema.Types.ObjectId, // Reference to User model
      ref: "User",
      required: true,
    },
    content: {
      type: String,
      required: true,
    },
  },
  { timestamps: true } // Adds createdAt and updatedAt automatically
);

export const ProjectNote = mongoose.model("ProjectNote", projectNoteSchema);
```

### 📌 Reasoning:  
- **Relationships:** Uses ObjectId references to establish model relationships  
- **Timestamps:** Automatic tracking of document creation/modification times  
- **Required Fields:** Ensures essential data integrity  
- **Modular Structure:** Follows consistent pattern for all models  

---


## 🏷️ Enhanced Model Properties

### 🔹 Field Validation Example
```javascript
name: {
    type: String,
    required: true,
    unique: true,     // Ensures no duplicate values
    trim: true       // Automatically removes whitespace
}
```

### 📌 Reasoning:  
- **Trim:** Automatically sanitizes input by removing spaces  
- **Unique:** Prevents duplicate entries in database  
- **Required:** Enforces mandatory field validation  

---

## 📋 Complete Task Model

### 🔹 Task Schema Implementation
```javascript
import mongoose, { Schema } from "mongoose";
import { AvailableTaskStatus, TaskStatusEnum } from "../utils/constants";

const taskSchema = new Schema(
  {
    title: {
      type: String,
      required: true,
      trim: true,
    },
    description: {
      type: String,
    },
    project: {
      type: Schema.Types.ObjectId,
      ref: "Project",
      required: [true, "Project reference is required"],
    },
    assignedTo: {
      type: Schema.Types.ObjectId,
      ref: "User",
      required: true,
    },
    assignedBy: {
      type: Schema.Types.ObjectId,
      ref: "User",
      required: true,
    },
    status: {
      type: String,
      enum: AvailableTaskStatus,  // Restricts to predefined values
      default: TaskStatusEnum.TODO,
    },
    attachments: {    //array for multiple files
      type: [{
        url: String,
        mimetype: String,
        size: Number,
      }],
      default: [],
    },
  },
  { timestamps: true },
);

export const Task = mongoose.model("Task", taskSchema);
```

### 📌 Reasoning:  
- **Constants Usage:** Centralizes status values for consistency  
- **Attachment Structure:** Flexible array for multiple file references  
- **Reference Fields:** Establishes relationships between models  
- **Custom Error Messages:** Improves validation feedback  

---

## 🔐 Authentication Implementation

### 🔹 Password Hashing (pre-save hook)
```javascript
userSchema.pre("save", async function(next) {
    if(!this.isModified("password")) return next();
    this.password = await bcrypt.hash(this.password, 10);
    next();
});
```

### 📌 Reasoning:  
- **Security:** Hashes passwords before storage  
- **Efficiency:** Only hashes when password changes  
- **Cost Factor:** 10 rounds balances security & performance  

---

## 🔑 Model Methods

### 🔹 Password Verification
```javascript
userSchema.methods.isPasswordCorrect = async function(password) {
    return await bcrypt.compare(password, this.password);
};
```

### 🔹 Token Generation
```javascript
userSchema.methods.generateAccessToken = function() {
    return jwt.sign(
        {
            _id: this._id,
            email: this.email,
            username: this.username,
        },
        process.env.ACCESS_TOKEN_SECRET,
        { expiresIn: process.env.ACCESS_TOKEN_EXPIRY }
    );
};

userSchema.methods.generateRefreshToken = function() {
    return jwt.sign(
        {
            _id: this._id,
        },
        process.env.REFRESH_TOKEN_SECRET,
        { expiresIn: process.env.REFRESH_TOKEN_EXPIRY }
    );
};
```

### 🔹 Temporary Token Generation
```javascript
userSchema.methods.generateTemporaryToken = function() {
    const unhashedToken = crypto.randomBytes(20).toString("hex");
    const hashedToken = crypto
        .createHash("sha256")
        .update(unhashedToken)
        .digest("hex");
    
    const tokenExpiry = Date.now() + (20 * 60 * 1000); // 20 minutes
    
    return { hashedToken, unhashedToken, tokenExpiry };
};
```

### 📌 Reasoning:  
- **Separation of Concerns:** Authentication logic stays in model  
- **Reusability:** Methods can be called from anywhere in application  
- **Security:**  
  - Refresh tokens contain minimal identifying info  
  - Temporary tokens expire automatically  
  - Cryptographic hashing for all sensitive operations  

---

## 📦 Required Packages
```bash
npm install bcryptjs jsonwebtoken crypto
```

> **Implementation Note:** Remember to add all token secrets and expiry values to your `.env` file with appropriate security measures.


---

## 🚀 Controller & Route Setup

### 🔹 File Creation
Created controller and route files via terminal:
```powershell
New-Item -ItemType File auth.controllers.js, note.controllers.js, project.controllers.js, task.controllers.js
New-Item -ItemType File healthcheck.routes.js, auth.routes.js, notes.routes.js, project.routes.js, task.routes.js
```

---

## 🩺 HealthCheck Implementation

### 🔹 Controller (healthcheck.controller.js)
```javascript
import { ApiResponse } from "../utils/api-response";

const healthCheck = (req, res) => {
  res.status(200).json(
    new ApiResponse(200, { message: "Server Running" })
  );
};

export { healthCheck };
```

### 📌 Reasoning:  
- **Consistent Responses:** Uses custom ApiResponse utility  
- **Status Codes:** Proper HTTP status code (200)  
- **Minimal Logic:** Simple endpoint for service monitoring  

---

### 🔹 Routes (healthcheck.routes.js)
```javascript
import { Router } from "express";
import { healthCheck } from "../controllers/healthcheck.controller";

const router = Router();
router.get("/", healthCheck);

export default router;
```

### 📌 Reasoning:  
- **Express Router:** Standard Express routing practice  
- **Minimal Route:** Simple GET endpoint  
- **Separation of Concerns:** Route only handles routing logic  

---

### 🔹 App Integration (app.js)
```javascript
import express from "express";
import healthCheckRouter from "./routes/healthcheck.routes";

const app = express();

// API versioning
app.use("/api/v1/healthcheck", healthCheckRouter);

export default app;
```

### 📌 Reasoning:  
- **Versioning:** `/api/v1/` prefix for API version control  
- **Modular Design:** Routes imported and mounted  
- **Export Pattern:** Consistent with Express app setup  

---

> **Best Practice:** This pattern can be replicated for all other routes and controllers following the same structure.
