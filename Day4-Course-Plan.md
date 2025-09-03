# Day 4: Authentication & Authorization for POS System

## Course Overview (8 Hours: 9:00 AM - 5:00 PM)
**Focus**: Implementing user authentication, authorization, and role-based access control for the POS system

## Daily Schedule
- **9:00 - 10:00**: Authentication Fundamentals & JWT Setup (60 min)
- **10:00 - 11:00**: User Model & Registration System (60 min)  
- **11:00 - 12:00**: Login System & Token Management (60 min)
- **12:00 - 13:00**: **LUNCH BREAK**
- **13:00 - 14:00**: Middleware & Route Protection (60 min)
- **14:00 - 15:00**: Role-Based Access Control (RBAC) (60 min)
- **15:00 - 16:00**: Password Security & Account Management (60 min)
- **16:00 - 17:00**: Integration Testing & Security Best Practices (60 min)

## Learning Objectives
By the end of Day 4, students will be able to:
- Understand JWT (JSON Web Tokens) and session-based authentication
- Implement secure user registration and login systems
- Create authentication middleware for route protection
- Design and implement role-based access control (RBAC)
- Implement password hashing and security best practices
- Handle token refresh and logout functionality
- Secure API endpoints based on user roles and permissions
- Test authenticated endpoints using Postman

---

## 9:00 - 10:00: Authentication Fundamentals & JWT Setup (60 minutes)

### Topics Covered:
- **Authentication vs Authorization** (15 minutes)
  - What is authentication (who you are)
  - What is authorization (what you can do)
  - Common authentication methods
  - Why JWT for APIs

- **JWT Deep Dive** (25 minutes)
  - JWT structure (Header.Payload.Signature)
  - How JWT works in APIs
  - JWT vs Sessions comparison
  - Security considerations

- **Project Setup** (20 minutes)
  - Installing required packages (jsonwebtoken, bcryptjs)
  - Environment variables for secrets
  - JWT utility functions setup

### Exercise 1 (15 minutes):
Students install dependencies and create basic JWT utility functions

### Key Code Blocks:

**1. Install Authentication Dependencies:**
```bash
npm install jsonwebtoken bcryptjs express-validator express-rate-limit helmet
```

**2. Update Environment Variables (.env):**
```env
# Server Configuration
PORT=3000
NODE_ENV=development

# CORS Configuration
FRONTEND_URL=http://localhost:3000

# Database Configuration
MONGODB_URI=mongodb://localhost:27017/pos_system

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key-here-make-it-long-and-random
JWT_EXPIRE=7d
JWT_REFRESH_SECRET=your-refresh-token-secret-here
JWT_REFRESH_EXPIRE=30d
```

**3. JWT Utility Functions (utils/jwt.js):**
```javascript
const jwt = require('jsonwebtoken');

// Generate access token
const generateToken = (payload) => {
    return jwt.sign(payload, process.env.JWT_SECRET, {
        expiresIn: process.env.JWT_EXPIRE || '7d'
    });
};

// Generate refresh token
const generateRefreshToken = (payload) => {
    return jwt.sign(payload, process.env.JWT_REFRESH_SECRET, {
        expiresIn: process.env.JWT_REFRESH_EXPIRE || '30d'
    });
};

// Verify access token
const verifyToken = (token) => {
    return jwt.verify(token, process.env.JWT_SECRET);
};

// Verify refresh token
const verifyRefreshToken = (token) => {
    return jwt.verify(token, process.env.JWT_REFRESH_SECRET);
};

// Decode token without verification (for expired tokens)
const decodeToken = (token) => {
    return jwt.decode(token);
};

module.exports = {
    generateToken,
    generateRefreshToken,
    verifyToken,
    verifyRefreshToken,
    decodeToken
};
```

**4. Password Utility Functions (utils/passwordUtils.js):**
```javascript
const bcrypt = require('bcryptjs');

// Hash password
const hashPassword = async (password) => {
    const saltRounds = 12;
    return await bcrypt.hash(password, saltRounds);
};

// Compare password with hash
const comparePassword = async (password, hashedPassword) => {
    return await bcrypt.compare(password, hashedPassword);
};

// Validate password strength
const validatePassword = (password) => {
    const minLength = 8;
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumber = /\d/.test(password);
    const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);
    
    const errors = [];
    
    if (password.length < minLength) {
        errors.push(`Password must be at least ${minLength} characters long`);
    }
    if (!hasUpperCase) {
        errors.push('Password must contain at least one uppercase letter');
    }
    if (!hasLowerCase) {
        errors.push('Password must contain at least one lowercase letter');
    }
    if (!hasNumber) {
        errors.push('Password must contain at least one number');
    }
    if (!hasSpecialChar) {
        errors.push('Password must contain at least one special character');
    }
    
    return {
        isValid: errors.length === 0,
        errors
    };
};

module.exports = {
    hashPassword,
    comparePassword,
    validatePassword
};
```

### Key Learning Points:
- JWT tokens are stateless and self-contained
- Never store sensitive data in JWT payload
- Always use strong secrets for signing
- Understand token expiration and refresh concepts

---

## 10:00 - 11:00: User Model & Registration System (60 minutes)

### Topics Covered:
- **User Model Design** (20 minutes)
  - User schema for POS system
  - Role-based field design
  - Password field security
  - User status and activation

- **Password Security** (25 minutes)
  - Password hashing with bcrypt
  - Salt rounds and security
  - Password validation rules
  - Pre-save middleware for hashing

- **Registration Controller** (15 minutes)
  - User registration logic
  - Duplicate email/username handling
  - Input validation and sanitization
  - Response formatting

### Exercise 2 (20 minutes):
Students create User model and implement registration endpoint

### Key Code Blocks:

**1. User Model (models/User.js):**
```javascript
const mongoose = require('mongoose');
const { hashPassword } = require('../utils/passwordUtils');

const userSchema = new mongoose.Schema({
    firstName: {
        type: String,
        required: [true, 'First name is required'],
        trim: true,
        maxlength: [50, 'First name cannot exceed 50 characters']
    },
    lastName: {
        type: String,
        required: [true, 'Last name is required'],
        trim: true,
        maxlength: [50, 'Last name cannot exceed 50 characters']
    },
    email: {
        type: String,
        required: [true, 'Email is required'],
        unique: true,
        lowercase: true,
        validate: {
            validator: function(v) {
                return /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v);
            },
            message: 'Please enter a valid email address'
        }
    },
    username: {
        type: String,
        required: [true, 'Username is required'],
        unique: true,
        trim: true,
        minlength: [3, 'Username must be at least 3 characters long'],
        maxlength: [20, 'Username cannot exceed 20 characters'],
        match: [/^[a-zA-Z0-9_]+$/, 'Username can only contain letters, numbers and underscores']
    },
    password: {
        type: String,
        required: [true, 'Password is required'],
        minlength: [8, 'Password must be at least 8 characters long']
    },
    role: {
        type: String,
        enum: ['admin', 'manager', 'cashier', 'staff'],
        default: 'cashier'
    },
    isActive: {
        type: Boolean,
        default: true
    },
    isEmailVerified: {
        type: Boolean,
        default: false
    },
    lastLogin: {
        type: Date,
        default: null
    },
    loginAttempts: {
        type: Number,
        default: 0
    },
    lockUntil: {
        type: Date,
        default: null
    },
    refreshTokens: [{
        token: String,
        createdAt: {
            type: Date,
            default: Date.now,
            expires: 2592000 // 30 days
        }
    }],
    profile: {
        phone: {
            type: String,
            validate: {
                validator: function(v) {
                    return !v || /^\+?[\d\s-()]{8,15}$/.test(v);
                },
                message: 'Please enter a valid phone number'
            }
        },
        avatar: {
            type: String,
            default: null
        },
        department: {
            type: String,
            enum: ['sales', 'inventory', 'management', 'support'],
            default: 'sales'
        }
    }
}, {
    timestamps: true
});

// Virtual for full name
userSchema.virtual('fullName').get(function() {
    return `${this.firstName} ${this.lastName}`;
});

// Virtual for account locked status
userSchema.virtual('isLocked').get(function() {
    return !!(this.lockUntil && this.lockUntil > Date.now());
});

// Hash password before saving
userSchema.pre('save', async function(next) {
    // Only hash if password was modified
    if (!this.isModified('password')) return next();
    
    try {
        this.password = await hashPassword(this.password);
        next();
    } catch (error) {
        next(error);
    }
});

// Remove password from JSON output
userSchema.methods.toJSON = function() {
    const user = this.toObject();
    delete user.password;
    delete user.refreshTokens;
    delete user.loginAttempts;
    delete user.lockUntil;
    return user;
};

// Account lock methods
userSchema.methods.incLoginAttempts = function() {
    // If we have previous lock and it's expired, restart at 1
    if (this.lockUntil && this.lockUntil < Date.now()) {
        return this.updateOne({
            $set: {
                loginAttempts: 1
            },
            $unset: {
                lockUntil: 1
            }
        });
    }
    
    const updates = { $inc: { loginAttempts: 1 } };
    
    // Lock account after 5 failed attempts for 2 hours
    if (this.loginAttempts + 1 >= 5 && !this.isLocked) {
        updates.$set = { lockUntil: Date.now() + 2 * 60 * 60 * 1000 }; // 2 hours
    }
    
    return this.updateOne(updates);
};

// Reset login attempts
userSchema.methods.resetLoginAttempts = function() {
    return this.updateOne({
        $unset: {
            loginAttempts: 1,
            lockUntil: 1
        }
    });
};

module.exports = mongoose.model('User', userSchema);
```

**2. Input Validation Middleware (middleware/validation.js):**
```javascript
const { body, validationResult } = require('express-validator');
const { validatePassword } = require('../utils/passwordUtils');

// Registration validation rules
const validateRegistration = [
    body('firstName')
        .trim()
        .isLength({ min: 1, max: 50 })
        .withMessage('First name must be between 1 and 50 characters')
        .matches(/^[a-zA-Z\s]+$/)
        .withMessage('First name can only contain letters and spaces'),
    
    body('lastName')
        .trim()
        .isLength({ min: 1, max: 50 })
        .withMessage('Last name must be between 1 and 50 characters')
        .matches(/^[a-zA-Z\s]+$/)
        .withMessage('Last name can only contain letters and spaces'),
    
    body('email')
        .isEmail()
        .normalizeEmail()
        .withMessage('Please enter a valid email address'),
    
    body('username')
        .trim()
        .isLength({ min: 3, max: 20 })
        .withMessage('Username must be between 3 and 20 characters')
        .matches(/^[a-zA-Z0-9_]+$/)
        .withMessage('Username can only contain letters, numbers and underscores'),
    
    body('password')
        .isLength({ min: 8 })
        .withMessage('Password must be at least 8 characters long')
        .custom((password) => {
            const validation = validatePassword(password);
            if (!validation.isValid) {
                throw new Error(validation.errors.join(', '));
            }
            return true;
        }),
    
    body('role')
        .optional()
        .isIn(['admin', 'manager', 'cashier', 'staff'])
        .withMessage('Role must be one of: admin, manager, cashier, staff')
];

// Login validation rules
const validateLogin = [
    body('login')
        .trim()
        .isLength({ min: 1 })
        .withMessage('Email or username is required'),
    
    body('password')
        .isLength({ min: 1 })
        .withMessage('Password is required')
];

// Check validation results
const checkValidationResult = (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({
            success: false,
            message: 'Validation failed',
            errors: errors.array().map(error => ({
                field: error.path,
                message: error.msg
            }))
        });
    }
    next();
};

module.exports = {
    validateRegistration,
    validateLogin,
    checkValidationResult
};
```

**3. Registration Controller (controllers/authController.js):**
```javascript
const User = require('../models/User');
const { generateToken, generateRefreshToken } = require('../utils/jwt');
const { comparePassword } = require('../utils/passwordUtils');

// Register new user
const register = async (req, res) => {
    try {
        const { firstName, lastName, email, username, password, role } = req.body;
        
        // Check if user already exists
        const existingUser = await User.findOne({
            $or: [{ email }, { username }]
        });
        
        if (existingUser) {
            return res.status(400).json({
                success: false,
                message: existingUser.email === email ? 
                    'Email already registered' : 
                    'Username already taken'
            });
        }
        
        // Create new user
        const user = new User({
            firstName,
            lastName,
            email,
            username,
            password,
            role: role || 'cashier'
        });
        
        await user.save();
        
        // Generate tokens
        const tokenPayload = {
            userId: user._id,
            email: user.email,
            role: user.role
        };
        
        const accessToken = generateToken(tokenPayload);
        const refreshToken = generateRefreshToken(tokenPayload);
        
        // Save refresh token
        user.refreshTokens.push({ token: refreshToken });
        await user.save();
        
        res.status(201).json({
            success: true,
            message: 'User registered successfully',
            data: {
                user,
                accessToken,
                refreshToken
            }
        });
        
    } catch (error) {
        console.error('Registration error:', error);
        
        // Handle duplicate key errors
        if (error.code === 11000) {
            const field = Object.keys(error.keyPattern)[0];
            return res.status(400).json({
                success: false,
                message: `${field} already exists`
            });
        }
        
        res.status(500).json({
            success: false,
            message: 'Registration failed',
            error: process.env.NODE_ENV === 'development' ? error.message : undefined
        });
    }
};

module.exports = {
    register
};
```

### Key Learning Points:
- Never store plain text passwords
- Use proper password strength validation
- Hash passwords before saving to database
- Handle registration errors gracefully

---

## 11:00 - 12:00: Login System & Token Management (60 minutes)

### Topics Covered:
- **Login Controller Implementation** (25 minutes)
  - Email/password verification
  - Password comparison with bcrypt
  - JWT token generation
  - Login response structure

- **Token Management** (20 minutes)
  - Access token vs refresh token
  - Token expiration handling
  - Token storage best practices
  - Logout implementation

- **Error Handling** (15 minutes)
  - Invalid credentials handling
  - Account locked/disabled scenarios
  - Rate limiting for login attempts
  - Security logging

### Exercise 3 (25 minutes):
Students implement login system with proper token generation and error handling

### Key Code Blocks:

**1. Login Controller (add to controllers/authController.js):**
```javascript
// Login user
const login = async (req, res) => {
    try {
        const { login, password } = req.body;
        
        // Find user by email or username
        const user = await User.findOne({
            $or: [
                { email: login },
                { username: login }
            ],
            isActive: true
        });
        
        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'Invalid credentials'
            });
        }
        
        // Check if account is locked
        if (user.isLocked) {
            return res.status(423).json({
                success: false,
                message: 'Account temporarily locked due to too many failed login attempts'
            });
        }
        
        // Compare password
        const isPasswordValid = await comparePassword(password, user.password);
        
        if (!isPasswordValid) {
            // Increment login attempts
            await user.incLoginAttempts();
            
            return res.status(401).json({
                success: false,
                message: 'Invalid credentials'
            });
        }
        
        // Reset login attempts on successful login
        if (user.loginAttempts > 0) {
            await user.resetLoginAttempts();
        }
        
        // Generate tokens
        const tokenPayload = {
            userId: user._id,
            email: user.email,
            username: user.username,
            role: user.role
        };
        
        const accessToken = generateToken(tokenPayload);
        const refreshToken = generateRefreshToken(tokenPayload);
        
        // Save refresh token and update last login
        user.refreshTokens.push({ token: refreshToken });
        user.lastLogin = new Date();
        await user.save();
        
        res.json({
            success: true,
            message: 'Login successful',
            data: {
                user,
                accessToken,
                refreshToken,
                expiresIn: process.env.JWT_EXPIRE || '7d'
            }
        });
        
    } catch (error) {
        console.error('Login error:', error);
        res.status(500).json({
            success: false,
            message: 'Login failed',
            error: process.env.NODE_ENV === 'development' ? error.message : undefined
        });
    }
};

// Refresh token
const refreshToken = async (req, res) => {
    try {
        const { refreshToken } = req.body;
        
        if (!refreshToken) {
            return res.status(401).json({
                success: false,
                message: 'Refresh token is required'
            });
        }
        
        // Verify refresh token
        const decoded = verifyRefreshToken(refreshToken);
        
        // Find user and check if refresh token exists
        const user = await User.findOne({
            _id: decoded.userId,
            'refreshTokens.token': refreshToken,
            isActive: true
        });
        
        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'Invalid refresh token'
            });
        }
        
        // Generate new tokens
        const tokenPayload = {
            userId: user._id,
            email: user.email,
            username: user.username,
            role: user.role
        };
        
        const newAccessToken = generateToken(tokenPayload);
        const newRefreshToken = generateRefreshToken(tokenPayload);
        
        // Replace old refresh token with new one
        user.refreshTokens = user.refreshTokens.filter(
            token => token.token !== refreshToken
        );
        user.refreshTokens.push({ token: newRefreshToken });
        await user.save();
        
        res.json({
            success: true,
            message: 'Token refreshed successfully',
            data: {
                accessToken: newAccessToken,
                refreshToken: newRefreshToken,
                expiresIn: process.env.JWT_EXPIRE || '7d'
            }
        });
        
    } catch (error) {
        console.error('Token refresh error:', error);
        res.status(401).json({
            success: false,
            message: 'Invalid or expired refresh token'
        });
    }
};

// Logout user
const logout = async (req, res) => {
    try {
        const { refreshToken } = req.body;
        const userId = req.user?.userId;
        
        if (userId && refreshToken) {
            // Remove refresh token from user
            await User.updateOne(
                { _id: userId },
                { $pull: { refreshTokens: { token: refreshToken } } }
            );
        }
        
        res.json({
            success: true,
            message: 'Logout successful'
        });
        
    } catch (error) {
        console.error('Logout error:', error);
        res.status(500).json({
            success: false,
            message: 'Logout failed'
        });
    }
};

// Logout from all devices
const logoutAll = async (req, res) => {
    try {
        const userId = req.user.userId;
        
        // Remove all refresh tokens
        await User.updateOne(
            { _id: userId },
            { $set: { refreshTokens: [] } }
        );
        
        res.json({
            success: true,
            message: 'Logged out from all devices'
        });
        
    } catch (error) {
        console.error('Logout all error:', error);
        res.status(500).json({
            success: false,
            message: 'Logout failed'
        });
    }
};

// Get current user profile
const getProfile = async (req, res) => {
    try {
        const user = await User.findById(req.user.userId);
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            data: { user }
        });
        
    } catch (error) {
        console.error('Get profile error:', error);
        res.status(500).json({
            success: false,
            message: 'Failed to get profile'
        });
    }
};

module.exports = {
    register,
    login,
    refreshToken,
    logout,
    logoutAll,
    getProfile
};
```

**2. Rate Limiting for Auth Routes (middleware/rateLimiter.js):**
```javascript
const rateLimit = require('express-rate-limit');

// Rate limiter for login attempts
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // Limit each IP to 5 login attempts per windowMs
    message: {
        success: false,
        message: 'Too many login attempts, please try again later',
        retryAfter: '15 minutes'
    },
    standardHeaders: true,
    legacyHeaders: false,
    skipSuccessfulRequests: true, // Don't count successful requests
});

// Rate limiter for registration
const registerLimiter = rateLimit({
    windowMs: 60 * 60 * 1000, // 1 hour
    max: 3, // Limit each IP to 3 registration attempts per hour
    message: {
        success: false,
        message: 'Too many registration attempts, please try again later',
        retryAfter: '1 hour'
    },
    standardHeaders: true,
    legacyHeaders: false,
});

// General auth rate limiter
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 20, // Limit each IP to 20 auth requests per windowMs
    message: {
        success: false,
        message: 'Too many authentication requests, please try again later'
    },
    standardHeaders: true,
    legacyHeaders: false,
});

module.exports = {
    loginLimiter,
    registerLimiter,
    authLimiter
};
```

**3. Auth Routes (routes/auth.js):**
```javascript
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');
const { validateRegistration, validateLogin, checkValidationResult } = require('../middleware/validation');
const { loginLimiter, registerLimiter, authLimiter } = require('../middleware/rateLimiter');
const { authenticate } = require('../middleware/auth');

// Apply general rate limiting to all auth routes
router.use(authLimiter);

// Public routes
router.post('/register', 
    registerLimiter,
    validateRegistration, 
    checkValidationResult, 
    authController.register
);

router.post('/login', 
    loginLimiter,
    validateLogin, 
    checkValidationResult, 
    authController.login
);

router.post('/refresh-token', authController.refreshToken);

// Protected routes (require authentication)
router.post('/logout', authenticate, authController.logout);
router.post('/logout-all', authenticate, authController.logoutAll);
router.get('/profile', authenticate, authController.getProfile);

module.exports = router;
```

### Key Learning Points:
- Always verify passwords using bcrypt.compare()
- Generate tokens only after successful authentication
- Include necessary user info in token payload
- Implement proper error messages without revealing system details

---

## 12:00 - 13:00: **LUNCH BREAK** 

---

## 13:00 - 14:00: Middleware & Route Protection (60 minutes)

### Topics Covered:
- **Authentication Middleware** (25 minutes)
  - JWT verification middleware
  - Token extraction from headers
  - Token validation and decoding
  - User object attachment to request

- **Route Protection Implementation** (20 minutes)
  - Protecting existing POS routes
  - Public vs protected endpoints
  - Middleware application strategies
  - Error handling for unauthorized access

- **Testing Protected Routes** (15 minutes)
  - Postman authentication setup
  - Authorization header configuration
  - Testing with valid/invalid tokens

### Exercise 4 (20 minutes):
Students create authentication middleware and protect existing POS endpoints

### Key Code Blocks:

**1. Authentication Middleware (middleware/auth.js):**
```javascript
const { verifyToken } = require('../utils/jwt');
const User = require('../models/User');

// Authentication middleware
const authenticate = async (req, res, next) => {
    try {
        const authHeader = req.header('Authorization');
        
        if (!authHeader || !authHeader.startsWith('Bearer ')) {
            return res.status(401).json({
                success: false,
                message: 'Access token is required'
            });
        }
        
        const token = authHeader.substring(7); // Remove 'Bearer ' prefix
        
        // Verify token
        const decoded = verifyToken(token);
        
        // Check if user still exists and is active
        const user = await User.findById(decoded.userId);
        if (!user || !user.isActive) {
            return res.status(401).json({
                success: false,
                message: 'User not found or inactive'
            });
        }
        
        // Check if account is locked
        if (user.isLocked) {
            return res.status(423).json({
                success: false,
                message: 'Account is temporarily locked'
            });
        }
        
        // Attach user info to request
        req.user = {
            userId: decoded.userId,
            email: decoded.email,
            username: decoded.username,
            role: decoded.role
        };
        
        next();
        
    } catch (error) {
        console.error('Authentication error:', error);
        
        if (error.name === 'JsonWebTokenError') {
            return res.status(401).json({
                success: false,
                message: 'Invalid token'
            });
        }
        
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({
                success: false,
                message: 'Token expired'
            });
        }
        
        res.status(500).json({
            success: false,
            message: 'Authentication failed'
        });
    }
};

// Optional authentication middleware (for endpoints that can work with or without auth)
const optionalAuth = async (req, res, next) => {
    try {
        const authHeader = req.header('Authorization');
        
        if (authHeader && authHeader.startsWith('Bearer ')) {
            const token = authHeader.substring(7);
            const decoded = verifyToken(token);
            
            const user = await User.findById(decoded.userId);
            if (user && user.isActive && !user.isLocked) {
                req.user = {
                    userId: decoded.userId,
                    email: decoded.email,
                    username: decoded.username,
                    role: decoded.role
                };
            }
        }
        
        next();
        
    } catch (error) {
        // For optional auth, we continue even if token is invalid
        next();
    }
};

module.exports = {
    authenticate,
    optionalAuth
};
```

**2. Updated Protected Routes (routes/products.js):**
```javascript
const express = require('express');
const router = express.Router();
const productController = require('../controllers/productController');
const { authenticate } = require('../middleware/auth');
const { authorize } = require('../middleware/authorize');

// Public routes (no authentication required)
router.get('/', productController.getAllProducts); // Anyone can view products
router.get('/low-stock', authenticate, authorize(['admin', 'manager']), productController.getLowStockProducts);
router.get('/:id', productController.getProductById); // Anyone can view single product

// Protected routes (authentication required)
router.post('/', authenticate, authorize(['admin', 'manager']), productController.createProduct);
router.put('/:id', authenticate, authorize(['admin', 'manager']), productController.updateProduct);
router.patch('/:id/stock', authenticate, authorize(['admin', 'manager', 'cashier']), productController.updateStock);
router.delete('/:id', authenticate, authorize(['admin', 'manager']), productController.deleteProduct);

module.exports = router;
```

**3. Updated Protected Routes (routes/orders.js):**
```javascript
const express = require('express');
const router = express.Router();
const orderController = require('../controllers/orderController');
const { authenticate } = require('../middleware/auth');
const { authorize } = require('../middleware/authorize');

// All order routes require authentication
router.use(authenticate);

// Order routes with role-based access
router.get('/', authorize(['admin', 'manager', 'cashier']), orderController.getAllOrders);
router.get('/reports/daily', authorize(['admin', 'manager']), orderController.getDailySalesReport);
router.get('/:id', authorize(['admin', 'manager', 'cashier']), orderController.getOrderById);
router.post('/', authorize(['admin', 'manager', 'cashier']), orderController.createOrder);
router.patch('/:id/status', authorize(['admin', 'manager', 'cashier']), orderController.updateOrderStatus);

module.exports = router;
```

**4. Updated Protected Routes (routes/customers.js):**
```javascript
const express = require('express');
const router = express.Router();
const customerController = require('../controllers/customerController');
const { authenticate } = require('../middleware/auth');
const { authorize } = require('../middleware/authorize');

// All customer routes require authentication
router.use(authenticate);

// Customer routes with role-based access
router.get('/', authorize(['admin', 'manager', 'cashier']), customerController.getAllCustomers);
router.get('/:id', authorize(['admin', 'manager', 'cashier']), customerController.getCustomerById);
router.post('/', authorize(['admin', 'manager', 'cashier']), customerController.createCustomer);
router.put('/:id', authorize(['admin', 'manager']), customerController.updateCustomer);
router.delete('/:id', authorize(['admin', 'manager']), customerController.deleteCustomer);

module.exports = router;
```

**5. Updated Protected Routes (routes/categories.js):**
```javascript
const express = require('express');
const router = express.Router();
const categoryController = require('../controllers/categoryController');
const { authenticate } = require('../middleware/auth');
const { authorize } = require('../middleware/authorize');

// Public routes
router.get('/', categoryController.getAllCategories); // Anyone can view categories
router.get('/:id', categoryController.getCategoryById);
router.get('/:id/products', categoryController.getCategoryProducts);

// Protected routes (admin and manager only)
router.post('/', authenticate, authorize(['admin', 'manager']), categoryController.createCategory);
router.put('/:id', authenticate, authorize(['admin', 'manager']), categoryController.updateCategory);
router.delete('/:id', authenticate, authorize(['admin', 'manager']), categoryController.deleteCategory);

module.exports = router;
```

**6. Updated App.js with Auth Routes:**
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const morgan = require('morgan');
const connectDB = require('./config/database');
require('dotenv').config();

const app = express();

// Connect to MongoDB
connectDB();

// Security middleware
app.use(helmet());

// CORS configuration
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: {
    error: 'Too many requests from this IP, please try again later.'
  }
});
app.use(limiter);

// Logging
app.use(morgan('dev'));

// Body parsing
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Auth routes (must come before other routes)
app.use('/api/auth', require('./routes/auth'));

// POS API Routes
app.use('/api/categories', require('./routes/categories'));
app.use('/api/products', require('./routes/products'));
app.use('/api/customers', require('./routes/customers'));
app.use('/api/orders', require('./routes/orders'));

// Basic routes
app.get('/', (req, res) => {
    res.json({ 
        message: 'POS System API is running!',
        endpoints: {
            auth: '/api/auth',
            categories: '/api/categories',
            products: '/api/products',
            customers: '/api/customers',
            orders: '/api/orders'
        }
    });
});

// Health check
app.get('/health', (req, res) => {
  res.json({
    status: 'OK',
    timestamp: new Date().toISOString(),
    database: 'Connected'
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({
    success: false,
    message: 'Route not found'
  });
});

// Error handler
app.use((err, req, res, next) => {
  console.error('Error:', err.stack);
  res.status(500).json({
    success: false,
    message: 'Internal Server Error'
  });
});

module.exports = app;
```

### Key Learning Points:
- Middleware runs before route handlers
- Always validate token signature and expiration
- Attach authenticated user to request object
- Provide clear error messages for authentication failures

---

## 14:00 - 15:00: Role-Based Access Control (RBAC) (60 minutes)

### Topics Covered:
- **RBAC Design for POS** (20 minutes)
  - Role hierarchy (Admin, Manager, Cashier, Staff)
  - Permission-based access control
  - Resource-based permissions
  - Role assignment and management

- **Authorization Middleware** (25 minutes)
  - Role checking middleware
  - Permission validation functions
  - Dynamic role assignment
  - Resource ownership checks

- **Implementing Role-Based Routes** (15 minutes)
  - Admin-only routes (user management, reports)
  - Manager routes (inventory, category management)
  - Cashier routes (order processing, customer management)
  - Read-only access for staff

### Exercise 5 (25 minutes):
Students implement RBAC system and apply role restrictions to POS endpoints

### Key Code Blocks:

**1. Authorization Middleware (middleware/authorize.js):**
```javascript
// Role-based authorization middleware
const authorize = (allowedRoles) => {
    return (req, res, next) => {
        try {
            // Check if user is authenticated
            if (!req.user) {
                return res.status(401).json({
                    success: false,
                    message: 'Authentication required'
                });
            }
            
            // Check if user role is in allowed roles
            if (!allowedRoles.includes(req.user.role)) {
                return res.status(403).json({
                    success: false,
                    message: 'Insufficient permissions',
                    required: allowedRoles,
                    current: req.user.role
                });
            }
            
            next();
            
        } catch (error) {
            console.error('Authorization error:', error);
            res.status(500).json({
                success: false,
                message: 'Authorization failed'
            });
        }
    };
};

// Permission-based authorization
const authorizePermissions = (requiredPermissions) => {
    return async (req, res, next) => {
        try {
            if (!req.user) {
                return res.status(401).json({
                    success: false,
                    message: 'Authentication required'
                });
            }
            
            // Get user permissions based on role
            const userPermissions = getRolePermissions(req.user.role);
            
            // Check if user has all required permissions
            const hasPermissions = requiredPermissions.every(
                permission => userPermissions.includes(permission)
            );
            
            if (!hasPermissions) {
                return res.status(403).json({
                    success: false,
                    message: 'Insufficient permissions',
                    required: requiredPermissions,
                    available: userPermissions
                });
            }
            
            next();
            
        } catch (error) {
            console.error('Permission authorization error:', error);
            res.status(500).json({
                success: false,
                message: 'Authorization failed'
            });
        }
    };
};

// Resource ownership check
const authorizeOwnership = (resourceParam = 'id') => {
    return (req, res, next) => {
        try {
            const resourceId = req.params[resourceParam];
            const userId = req.user.userId;
            
            // Admin and managers can access any resource
            if (['admin', 'manager'].includes(req.user.role)) {
                return next();
            }
            
            // For other roles, check if they own the resource
            // This would need to be implemented per resource type
            // For now, we'll just pass through
            next();
            
        } catch (error) {
            console.error('Ownership authorization error:', error);
            res.status(500).json({
                success: false,
                message: 'Authorization failed'
            });
        }
    };
};

// Get permissions for a role
const getRolePermissions = (role) => {
    const rolePermissions = {
        admin: [
            'users.create', 'users.read', 'users.update', 'users.delete',
            'products.create', 'products.read', 'products.update', 'products.delete',
            'categories.create', 'categories.read', 'categories.update', 'categories.delete',
            'customers.create', 'customers.read', 'customers.update', 'customers.delete',
            'orders.create', 'orders.read', 'orders.update', 'orders.delete',
            'reports.read', 'system.manage'
        ],
        manager: [
            'products.create', 'products.read', 'products.update', 'products.delete',
            'categories.create', 'categories.read', 'categories.update', 'categories.delete',
            'customers.create', 'customers.read', 'customers.update', 'customers.delete',
            'orders.create', 'orders.read', 'orders.update',
            'reports.read', 'inventory.manage'
        ],
        cashier: [
            'products.read',
            'categories.read',
            'customers.create', 'customers.read', 'customers.update',
            'orders.create', 'orders.read', 'orders.update',
            'inventory.update'
        ],
        staff: [
            'products.read',
            'categories.read',
            'customers.read',
            'orders.read'
        ]
    };
    
    return rolePermissions[role] || [];
};

module.exports = {
    authorize,
    authorizePermissions,
    authorizeOwnership,
    getRolePermissions
};
```

**2. User Management Controller (controllers/userController.js):**
```javascript
const User = require('../models/User');
const { generateToken } = require('../utils/jwt');

// Get all users (Admin only)
const getAllUsers = async (req, res) => {
    try {
        const { role, isActive, limit = 50, page = 1 } = req.query;
        
        let query = {};
        
        if (role) query.role = role;
        if (isActive !== undefined) query.isActive = isActive === 'true';
        
        const skip = (page - 1) * limit;
        
        const users = await User.find(query)
            .sort({ createdAt: -1 })
            .limit(parseInt(limit))
            .skip(skip);
        
        const total = await User.countDocuments(query);
        
        res.json({
            success: true,
            data: users,
            pagination: {
                current: parseInt(page),
                total: Math.ceil(total / limit),
                count: users.length,
                totalUsers: total
            }
        });
        
    } catch (error) {
        console.error('Get users error:', error);
        res.status(500).json({
            success: false,
            message: 'Failed to fetch users'
        });
    }
};

// Get single user
const getUserById = async (req, res) => {
    try {
        const user = await User.findById(req.params.id);
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            data: { user }
        });
        
    } catch (error) {
        console.error('Get user error:', error);
        res.status(500).json({
            success: false,
            message: 'Failed to fetch user'
        });
    }
};

// Update user (Admin/Manager)
const updateUser = async (req, res) => {
    try {
        const { firstName, lastName, email, username, role, isActive, profile } = req.body;
        const userId = req.params.id;
        const currentUserRole = req.user.role;
        
        // Only admins can change roles to admin
        if (role === 'admin' && currentUserRole !== 'admin') {
            return res.status(403).json({
                success: false,
                message: 'Only admins can create admin users'
            });
        }
        
        // Managers cannot deactivate admins
        if (isActive === false && currentUserRole === 'manager') {
            const targetUser = await User.findById(userId);
            if (targetUser && targetUser.role === 'admin') {
                return res.status(403).json({
                    success: false,
                    message: 'Managers cannot deactivate admin users'
                });
            }
        }
        
        const updateData = {};
        if (firstName) updateData.firstName = firstName;
        if (lastName) updateData.lastName = lastName;
        if (email) updateData.email = email;
        if (username) updateData.username = username;
        if (role) updateData.role = role;
        if (isActive !== undefined) updateData.isActive = isActive;
        if (profile) updateData.profile = { ...updateData.profile, ...profile };
        
        const user = await User.findByIdAndUpdate(
            userId,
            updateData,
            { new: true, runValidators: true }
        );
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            message: 'User updated successfully',
            data: { user }
        });
        
    } catch (error) {
        console.error('Update user error:', error);
        
        if (error.code === 11000) {
            const field = Object.keys(error.keyPattern)[0];
            return res.status(400).json({
                success: false,
                message: `${field} already exists`
            });
        }
        
        res.status(400).json({
            success: false,
            message: 'Failed to update user',
            error: error.message
        });
    }
};

// Delete user (soft delete - Admin only)
const deleteUser = async (req, res) => {
    try {
        const userId = req.params.id;
        const currentUserId = req.user.userId;
        
        // Cannot delete yourself
        if (userId === currentUserId) {
            return res.status(400).json({
                success: false,
                message: 'Cannot delete your own account'
            });
        }
        
        const user = await User.findByIdAndUpdate(
            userId,
            { isActive: false },
            { new: true }
        );
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            message: 'User deactivated successfully'
        });
        
    } catch (error) {
        console.error('Delete user error:', error);
        res.status(500).json({
            success: false,
            message: 'Failed to delete user'
        });
    }
};

// Change user role (Admin only)
const changeUserRole = async (req, res) => {
    try {
        const { role } = req.body;
        const userId = req.params.id;
        const currentUserId = req.user.userId;
        
        // Cannot change your own role
        if (userId === currentUserId) {
            return res.status(400).json({
                success: false,
                message: 'Cannot change your own role'
            });
        }
        
        const user = await User.findByIdAndUpdate(
            userId,
            { role },
            { new: true, runValidators: true }
        );
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            message: 'User role updated successfully',
            data: { user }
        });
        
    } catch (error) {
        console.error('Change role error:', error);
        res.status(400).json({
            success: false,
            message: 'Failed to change user role',
            error: error.message
        });
    }
};

module.exports = {
    getAllUsers,
    getUserById,
    updateUser,
    deleteUser,
    changeUserRole
};
```

**3. User Management Routes (routes/users.js):**
```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { authenticate } = require('../middleware/auth');
const { authorize } = require('../middleware/authorize');

// All user management routes require authentication
router.use(authenticate);

// User management routes (Admin and Manager access)
router.get('/', authorize(['admin', 'manager']), userController.getAllUsers);
router.get('/:id', authorize(['admin', 'manager']), userController.getUserById);
router.put('/:id', authorize(['admin', 'manager']), userController.updateUser);
router.delete('/:id', authorize(['admin']), userController.deleteUser);
router.patch('/:id/role', authorize(['admin']), userController.changeUserRole);

module.exports = router;
```

**4. Role-Based Route Protection Examples:**
```javascript
// Example of different protection levels

// Public access - no authentication required
router.get('/api/categories', categoryController.getAllCategories);

// Authenticated access - any logged-in user
router.get('/api/profile', authenticate, authController.getProfile);

// Role-based access - specific roles only
router.post('/api/products', 
    authenticate, 
    authorize(['admin', 'manager']), 
    productController.createProduct
);

// Permission-based access - specific permissions
router.get('/api/reports/sales', 
    authenticate, 
    authorizePermissions(['reports.read']), 
    reportController.getSalesReport
);

// Hierarchical access - admin can do everything, manager limited access
router.delete('/api/users/:id', 
    authenticate, 
    authorize(['admin']), // Only admins can delete users
    userController.deleteUser
);

// Conditional access based on resource ownership
router.put('/api/orders/:id', 
    authenticate,
    authorize(['admin', 'manager', 'cashier']),
    authorizeOwnership('id'), // Check if user can modify this specific order
    orderController.updateOrder
);
```

### Key Learning Points:
- Roles should follow principle of least privilege
- Separate authentication from authorization
- Design flexible permission systems
- Consider hierarchical role inheritance

---

## 15:00 - 16:00: Password Security & Account Management (60 minutes)

### Topics Covered:
- **Password Management** (20 minutes)
  - Password change functionality
  - Password reset with tokens
  - Password history prevention
  - Account lockout mechanisms

- **User Profile Management** (20 minutes)
  - Profile update endpoints
  - Email change verification
  - Account activation/deactivation
  - User preference settings

- **Security Features** (20 minutes)
  - Login attempt tracking
  - Session management
  - Account security audit logs
  - Two-factor authentication concepts

### Exercise 6 (25 minutes):
Students implement password change and user profile management features

### Key Learning Points:
- Always require current password for sensitive changes
- Use secure token-based password reset
- Log security-related events
- Implement account lockout to prevent brute force attacks

---

## 16:00 - 17:00: Integration Testing & Security Best Practices (60 minutes)

### Topics Covered:
- **Complete System Testing** (25 minutes)
  - End-to-end authentication flow
  - Role-based access testing
  - Error scenario validation
  - Performance testing with auth

- **Security Best Practices** (20 minutes)
  - Token security guidelines
  - API security headers
  - Input validation and sanitization
  - CORS configuration for auth

- **Production Considerations** (15 minutes)
  - Environment variable management
  - SSL/HTTPS requirements
  - Token refresh strategies
  - Monitoring and logging

### Exercise 6 (25 minutes):
Students implement password change and user profile management features

### Key Code Blocks:

**1. Password Change Controller (add to controllers/authController.js):**
```javascript
// Change password
const changePassword = async (req, res) => {
    try {
        const { currentPassword, newPassword } = req.body;
        const userId = req.user.userId;
        
        // Get user with password
        const user = await User.findById(userId).select('+password');
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        // Verify current password
        const isCurrentPasswordValid = await comparePassword(currentPassword, user.password);
        
        if (!isCurrentPasswordValid) {
            return res.status(400).json({
                success: false,
                message: 'Current password is incorrect'
            });
        }
        
        // Validate new password
        const validation = validatePassword(newPassword);
        if (!validation.isValid) {
            return res.status(400).json({
                success: false,
                message: 'Password validation failed',
                errors: validation.errors
            });
        }
        
        // Check if new password is different from current
        const isSamePassword = await comparePassword(newPassword, user.password);
        if (isSamePassword) {
            return res.status(400).json({
                success: false,
                message: 'New password must be different from current password'
            });
        }
        
        // Update password
        user.password = newPassword; // Will be hashed by pre-save middleware
        await user.save();
        
        res.json({
            success: true,
            message: 'Password changed successfully'
        });
        
    } catch (error) {
        console.error('Change password error:', error);
        res.status(500).json({
            success: false,
            message: 'Failed to change password'
        });
    }
};

// Update profile
const updateProfile = async (req, res) => {
    try {
        const userId = req.user.userId;
        const { firstName, lastName, profile } = req.body;
        
        const updateData = {};
        if (firstName) updateData.firstName = firstName;
        if (lastName) updateData.lastName = lastName;
        if (profile) updateData.profile = { ...updateData.profile, ...profile };
        
        const user = await User.findByIdAndUpdate(
            userId,
            updateData,
            { new: true, runValidators: true }
        );
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Profile updated successfully',
            data: { user }
        });
        
    } catch (error) {
        console.error('Update profile error:', error);
        res.status(400).json({
            success: false,
            message: 'Failed to update profile',
            error: error.message
        });
    }
};
```

### Final Exercise (20 minutes):
Students test complete authenticated POS system with different user roles and scenarios

### Key Code Blocks:

**1. Complete Postman Collection for Authentication (POS-Auth-API.postman_collection.json):**
```json
{
  "info": {
    "name": "POS System Authentication API",
    "description": "Complete authentication and authorization API collection for POS System",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Authentication",
      "item": [
        {
          "name": "Register User",
          "request": {
            "method": "POST",
            "header": [
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"firstName\": \"John\",\n  \"lastName\": \"Doe\",\n  \"email\": \"john.doe@example.com\",\n  \"username\": \"johndoe\",\n  \"password\": \"SecurePass123!\",\n  \"role\": \"cashier\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/auth/register",
              "host": ["{{baseUrl}}"],
              "path": ["api", "auth", "register"]
            }
          }
        },
        {
          "name": "Login User",
          "event": [
            {
              "listen": "test",
              "script": {
                "exec": [
                  "if (pm.response.code === 200) {",
                  "    const response = pm.response.json();",
                  "    pm.environment.set('accessToken', response.data.accessToken);",
                  "    pm.environment.set('refreshToken', response.data.refreshToken);",
                  "    pm.environment.set('userId', response.data.user._id);",
                  "}"
                ]
              }
            }
          ],
          "request": {
            "method": "POST",
            "header": [
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"login\": \"johndoe\",\n  \"password\": \"SecurePass123!\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/auth/login",
              "host": ["{{baseUrl}}"],
              "path": ["api", "auth", "login"]
            }
          }
        },
        {
          "name": "Refresh Token",
          "request": {
            "method": "POST",
            "header": [
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"refreshToken\": \"{{refreshToken}}\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/auth/refresh-token",
              "host": ["{{baseUrl}}"],
              "path": ["api", "auth", "refresh-token"]
            }
          }
        },
        {
          "name": "Get Profile",
          "request": {
            "method": "GET",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              }
            ],
            "url": {
              "raw": "{{baseUrl}}/api/auth/profile",
              "host": ["{{baseUrl}}"],
              "path": ["api", "auth", "profile"]
            }
          }
        },
        {
          "name": "Change Password",
          "request": {
            "method": "PUT",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              },
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"currentPassword\": \"SecurePass123!\",\n  \"newPassword\": \"NewSecurePass456!\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/auth/change-password",
              "host": ["{{baseUrl}}"],
              "path": ["api", "auth", "change-password"]
            }
          }
        },
        {
          "name": "Update Profile",
          "request": {
            "method": "PUT",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              },
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"firstName\": \"John\",\n  \"lastName\": \"Smith\",\n  \"profile\": {\n    \"phone\": \"+1234567890\",\n    \"department\": \"sales\"\n  }\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/auth/profile",
              "host": ["{{baseUrl}}"],
              "path": ["api", "auth", "profile"]
            }
          }
        },
        {
          "name": "Logout",
          "request": {
            "method": "POST",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              },
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"refreshToken\": \"{{refreshToken}}\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/auth/logout",
              "host": ["{{baseUrl}}"],
              "path": ["api", "auth", "logout"]
            }
          }
        },
        {
          "name": "Logout All Devices",
          "request": {
            "method": "POST",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              }
            ],
            "url": {
              "raw": "{{baseUrl}}/api/auth/logout-all",
              "host": ["{{baseUrl}}"],
              "path": ["api", "auth", "logout-all"]
            }
          }
        }
      ]
    },
    {
      "name": "User Management (Admin/Manager)",
      "item": [
        {
          "name": "Get All Users",
          "request": {
            "method": "GET",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              }
            ],
            "url": {
              "raw": "{{baseUrl}}/api/users?role=cashier&isActive=true",
              "host": ["{{baseUrl}}"],
              "path": ["api", "users"],
              "query": [
                {
                  "key": "role",
                  "value": "cashier"
                },
                {
                  "key": "isActive",
                  "value": "true"
                }
              ]
            }
          }
        },
        {
          "name": "Get User by ID",
          "request": {
            "method": "GET",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              }
            ],
            "url": {
              "raw": "{{baseUrl}}/api/users/{{userId}}",
              "host": ["{{baseUrl}}"],
              "path": ["api", "users", "{{userId}}"]
            }
          }
        },
        {
          "name": "Update User",
          "request": {
            "method": "PUT",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              },
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"firstName\": \"Jane\",\n  \"lastName\": \"Smith\",\n  \"role\": \"manager\",\n  \"isActive\": true\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/users/{{userId}}",
              "host": ["{{baseUrl}}"],
              "path": ["api", "users", "{{userId}}"]
            }
          }
        },
        {
          "name": "Change User Role",
          "request": {
            "method": "PATCH",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              },
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"role\": \"manager\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/users/{{userId}}/role",
              "host": ["{{baseUrl}}"],
              "path": ["api", "users", "{{userId}}", "role"]
            }
          }
        },
        {
          "name": "Deactivate User",
          "request": {
            "method": "DELETE",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              }
            ],
            "url": {
              "raw": "{{baseUrl}}/api/users/{{userId}}",
              "host": ["{{baseUrl}}"],
              "path": ["api", "users", "{{userId}}"]
            }
          }
        }
      ]
    },
    {
      "name": "Protected POS Routes",
      "item": [
        {
          "name": "Create Product (Manager/Admin Only)",
          "request": {
            "method": "POST",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              },
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"name\": \"Secured Product\",\n  \"description\": \"This product requires manager/admin access\",\n  \"price\": 19.99,\n  \"stock\": 50,\n  \"category\": \"{{categoryId}}\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/products",
              "host": ["{{baseUrl}}"],
              "path": ["api", "products"]
            }
          }
        },
        {
          "name": "Create Order (Cashier+ Access)",
          "request": {
            "method": "POST",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              },
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"customerName\": \"Authenticated Customer\",\n  \"items\": [\n    {\n      \"product\": \"{{productId}}\",\n      \"quantity\": 2\n    }\n  ],\n  \"paymentMethod\": \"card\",\n  \"tax\": 2.00\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/orders",
              "host": ["{{baseUrl}}"],
              "path": ["api", "orders"]
            }
          }
        },
        {
          "name": "Get Daily Sales Report (Manager/Admin Only)",
          "request": {
            "method": "GET",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              }
            ],
            "url": {
              "raw": "{{baseUrl}}/api/orders/reports/daily",
              "host": ["{{baseUrl}}"],
              "path": ["api", "orders", "reports", "daily"]
            }
          }
        },
        {
          "name": "Update Stock (Cashier+ Access)",
          "request": {
            "method": "PATCH",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              },
              {
                "key": "Content-Type",
                "value": "application/json"
              }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"stock\": 100,\n  \"operation\": \"set\"\n}"
            },
            "url": {
              "raw": "{{baseUrl}}/api/products/{{productId}}/stock",
              "host": ["{{baseUrl}}"],
              "path": ["api", "products", "{{productId}}", "stock"]
            }
          }
        }
      ]
    },
    {
      "name": "Security Tests",
      "item": [
        {
          "name": "Access Without Token (Should Fail)",
          "request": {
            "method": "GET",
            "header": [],
            "url": {
              "raw": "{{baseUrl}}/api/orders",
              "host": ["{{baseUrl}}"],
              "path": ["api", "orders"]
            }
          }
        },
        {
          "name": "Access With Invalid Token (Should Fail)",
          "request": {
            "method": "GET",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer invalid_token_here"
              }
            ],
            "url": {
              "raw": "{{baseUrl}}/api/orders",
              "host": ["{{baseUrl}}"],
              "path": ["api", "orders"]
            }
          }
        },
        {
          "name": "Access Admin Route as Cashier (Should Fail)",
          "request": {
            "method": "DELETE",
            "header": [
              {
                "key": "Authorization",
                "value": "Bearer {{accessToken}}"
              }
            ],
            "url": {
              "raw": "{{baseUrl}}/api/users/{{userId}}",
              "host": ["{{baseUrl}}"],
              "path": ["api", "users", "{{userId}}"]
            }
          }
        }
      ]
    }
  ],
  "variable": [
    {
      "key": "baseUrl",
      "value": "http://localhost:3000",
      "type": "string"
    },
    {
      "key": "accessToken",
      "value": "",
      "type": "string"
    },
    {
      "key": "refreshToken",
      "value": "",
      "type": "string"
    },
    {
      "key": "userId",
      "value": "",
      "type": "string"
    },
    {
      "key": "categoryId",
      "value": "paste_category_id_here",
      "type": "string"
    },
    {
      "key": "productId",
      "value": "paste_product_id_here",
      "type": "string"
    }
  ]
}
```

**2. Security Testing Checklist:**
```javascript
// Security Testing Scenarios for Day 4

// 1. Authentication Tests
// ✓ Register with valid data
// ✓ Register with invalid password (too weak)
// ✓ Register with existing email/username
// ✓ Login with correct credentials  
// ✓ Login with incorrect password
// ✓ Login with non-existent user
// ✓ Login after account lockout
// ✓ Token refresh with valid token
// ✓ Token refresh with invalid token

// 2. Authorization Tests
// ✓ Access protected route without token
// ✓ Access protected route with invalid token
// ✓ Access protected route with expired token
// ✓ Admin accessing admin-only routes
// ✓ Manager accessing manager routes
// ✓ Cashier accessing cashier routes
// ✓ Staff accessing read-only routes
// ✓ Cashier trying to access admin routes (should fail)
// ✓ Staff trying to modify data (should fail)

// 3. Role-Based Access Tests
// ✓ Create products as admin (should succeed)
// ✓ Create products as manager (should succeed)
// ✓ Create products as cashier (should fail)
// ✓ View sales reports as admin (should succeed)
// ✓ View sales reports as manager (should succeed)
// ✓ View sales reports as cashier (should fail)

// 4. Security Edge Cases
// ✓ Multiple failed login attempts (account lockout)
// ✓ Rate limiting on auth endpoints
// ✓ Password change with wrong current password
// ✓ Token usage after logout
// ✓ Accessing deactivated user account
```

### Day 4 Wrap-up:
- **System Demo** - Demonstrate role-based POS system access
- **Security Review** - Discuss security implications and best practices
- **Q&A Session** - Address authentication/authorization questions
- **Next Steps** - Frontend integration and advanced security topics

---

## Daily Exercises Summary

### Exercise 1: JWT Setup
- Install authentication dependencies
- Create JWT utility functions
- Set up environment variables for secrets

### Exercise 2: User Registration
- Design User model with role-based fields
- Implement password hashing with bcrypt
- Create registration endpoint with validation

### Exercise 3: Login System
- Build login controller with password verification
- Implement JWT token generation
- Add proper error handling and security

### Exercise 4: Route Protection
- Create authentication middleware
- Protect existing POS endpoints
- Test authentication with Postman

### Exercise 5: Role-Based Access Control
- Design role hierarchy for POS system
- Implement authorization middleware
- Apply role restrictions to specific endpoints

### Exercise 6: Account Management
- Add password change functionality
- Implement user profile management
- Add security features like login tracking

### Final Exercise: Complete Integration
- Test full authentication flow
- Verify role-based access control
- Validate security measures and error handling

---

## POS System Roles & Permissions

### **Admin Role:**
- **Full System Access**
- User management (create, update, delete users)
- System configuration and settings
- All reports and analytics
- Database maintenance operations
- Role assignment and permission management

### **Manager Role:**
- **Business Operations Management**
- Product management (create, update, delete products)
- Category management
- Inventory management and stock updates
- Customer management
- Order management and refunds
- Sales reports and analytics
- Staff performance monitoring

### **Cashier Role:**
- **Point of Sale Operations**
- Process orders and payments
- Customer lookup and basic info updates
- Product lookup and pricing
- Daily sales summary access
- Inventory level viewing (read-only)
- Basic order history access

### **Staff Role:**
- **Limited Read Access**
- View products and categories
- View customer information
- View basic inventory levels
- Access to training materials
- Personal profile management

---

## Required Dependencies
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^7.5.0",
    "dotenv": "^16.3.0",
    "cors": "^2.8.5",
    "jsonwebtoken": "^9.0.2",
    "bcryptjs": "^2.4.3",
    "express-rate-limit": "^8.0.1",
    "helmet": "^7.0.0",
    "express-validator": "^7.0.1"
  }
}
```

## Updated File Structure
```
pos-backend/
├── models/
│   ├── Product.js
│   ├── Category.js
│   ├── Customer.js
│   ├── Order.js
│   └── User.js (NEW)
├── controllers/
│   ├── productController.js
│   ├── categoryController.js
│   ├── customerController.js
│   ├── orderController.js
│   ├── authController.js (NEW)
│   └── userController.js (NEW)
├── routes/
│   ├── products.js
│   ├── categories.js
│   ├── customers.js
│   ├── orders.js
│   ├── auth.js (NEW)
│   └── users.js (NEW)
├── middleware/
│   ├── errorHandler.js
│   ├── auth.js (NEW)
│   ├── authorize.js (NEW)
│   └── validation.js (NEW)
├── utils/
│   ├── jwt.js (NEW)
│   └── passwordUtils.js (NEW)
├── config/
│   └── database.js
├── .env
├── app.js
└── server.js
```

## Assessment Criteria
- **Security Implementation**: Proper password hashing, JWT handling, and validation
- **Authentication Flow**: Complete registration, login, and logout functionality
- **Authorization Logic**: Correct role-based access control implementation
- **Code Quality**: Clean, secure, and well-documented authentication code
- **API Security**: Proper error handling and security headers
- **Testing**: Successful authentication testing with various scenarios
- **Understanding**: Ability to explain security concepts and implementation choices

## Homework/Extended Learning
- Implement two-factor authentication (2FA)
- Add OAuth integration (Google, GitHub)
- Create user activity logging and audit trails
- Implement advanced session management
- Add API rate limiting based on user roles
- Explore advanced JWT features (refresh tokens, token blacklisting)

## Notes for Instructors
- **Focus on Security**: Emphasize security best practices throughout
- **Real-world Examples**: Use common security vulnerabilities as teaching moments
- **Hands-on Testing**: Ensure students test all security scenarios
- **Role-based Thinking**: Help students understand business logic behind permissions
- **Production Ready**: Code should be production-ready with proper error handling
- **Interactive Learning**: Encourage students to try breaking the authentication system

## Common Security Pitfalls to Address
1. **Storing passwords in plain text** - Always hash passwords
2. **Weak JWT secrets** - Use strong, random secrets
3. **No token expiration** - Implement proper token lifecycle
4. **Client-side role checking only** - Always verify on server
5. **Revealing system information in errors** - Use generic error messages
6. **No rate limiting on auth endpoints** - Implement brute force protection
7. **Insecure token storage** - Educate on secure storage practices
8. **Missing authorization checks** - Verify permissions on every protected route

This comprehensive Day 4 plan builds upon the existing POS system from Day 3 and adds enterprise-level authentication and authorization features that students can immediately apply in real-world projects.