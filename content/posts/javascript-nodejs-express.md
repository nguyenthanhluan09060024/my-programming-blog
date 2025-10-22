---
title: "Xây Dựng REST API Với Node.js và Express"
date: 2024-01-10T13:30:00+07:00
draft: false
tags: ['javascript', 'nodejs', 'express', 'api', 'backend']
description: "Hướng dẫn chi tiết cách xây dựng REST API mạnh mẽ với Node.js và Express framework"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowCodeCopyButtons: true
toc: true
---

# Xây Dựng REST API Với Node.js và Express

Node.js và Express là một combo mạnh mẽ để xây dựng REST API. Trong bài viết này, tôi sẽ hướng dẫn bạn cách tạo một REST API hoàn chỉnh từ cơ bản đến nâng cao.

## Tại Sao Chọn Node.js và Express?

- **JavaScript Everywhere**: Sử dụng JavaScript cho cả frontend và backend
- **NPM Ecosystem**: Thư viện phong phú và cộng đồng lớn
- **Performance**: Non-blocking I/O cho hiệu suất cao
- **Scalability**: Dễ dàng scale horizontal
- **Rapid Development**: Phát triển nhanh chóng

## Thiết Lập Dự Án

### 1. Khởi tạo dự án

```bash
mkdir my-api
cd my-api
npm init -y
```

### 2. Cài đặt dependencies

```bash
npm install express cors helmet morgan compression
npm install mongoose bcryptjs jsonwebtoken
npm install express-validator express-rate-limit
npm install dotenv
npm install -D nodemon
```

### 3. Cấu trúc thư mục

```
my-api/
├── src/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   ├── middleware/
│   ├── utils/
│   └── app.js
├── config/
│   └── database.js
├── .env
├── package.json
└── server.js
```

## Tạo Server Cơ Bản

### 1. server.js

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const compression = require('compression');
require('dotenv').config();

const app = require('./src/app');

const PORT = process.env.PORT || 3000;

// Middleware
app.use(helmet());
app.use(cors());
app.use(compression());
app.use(morgan('combined'));
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/users', require('./src/routes/userRoutes'));
app.use('/api/products', require('./src/routes/productRoutes'));

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({
        success: false,
        message: 'Something went wrong!',
        error: process.env.NODE_ENV === 'development' ? err.message : {}
    });
});

// 404 handler
app.use('*', (req, res) => {
    res.status(404).json({
        success: false,
        message: 'Route not found'
    });
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

### 2. app.js

```javascript
const express = require('express');
const app = express();

// Health check endpoint
app.get('/health', (req, res) => {
    res.json({
        success: true,
        message: 'Server is running',
        timestamp: new Date().toISOString()
    });
});

module.exports = app;
```

## Kết Nối Database

### 1. config/database.js

```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
    try {
        const conn = await mongoose.connect(process.env.MONGODB_URI, {
            useNewUrlParser: true,
            useUnifiedTopology: true,
        });
        
        console.log(`MongoDB Connected: ${conn.connection.host}`);
    } catch (error) {
        console.error('Database connection error:', error);
        process.exit(1);
    }
};

module.exports = connectDB;
```

### 2. .env

```env
NODE_ENV=development
PORT=3000
MONGODB_URI=mongodb://localhost:27017/myapi
JWT_SECRET=your-secret-key
JWT_EXPIRE=7d
BCRYPT_ROUNDS=12
```

## Tạo Models

### 1. User Model

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
    username: {
        type: String,
        required: [true, 'Username is required'],
        unique: true,
        trim: true,
        minlength: [3, 'Username must be at least 3 characters'],
        maxlength: [30, 'Username cannot exceed 30 characters']
    },
    email: {
        type: String,
        required: [true, 'Email is required'],
        unique: true,
        lowercase: true,
        match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Please enter a valid email']
    },
    password: {
        type: String,
        required: [true, 'Password is required'],
        minlength: [6, 'Password must be at least 6 characters'],
        select: false
    },
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
    role: {
        type: String,
        enum: ['user', 'admin', 'moderator'],
        default: 'user'
    },
    isActive: {
        type: Boolean,
        default: true
    },
    lastLogin: {
        type: Date
    }
}, {
    timestamps: true
});

// Hash password before saving
userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    
    this.password = await bcrypt.hash(this.password, parseInt(process.env.BCRYPT_ROUNDS));
    next();
});

// Compare password method
userSchema.methods.comparePassword = async function(candidatePassword) {
    return await bcrypt.compare(candidatePassword, this.password);
};

// Remove password from JSON output
userSchema.methods.toJSON = function() {
    const userObject = this.toObject();
    delete userObject.password;
    return userObject;
};

module.exports = mongoose.model('User', userSchema);
```

### 2. Product Model

```javascript
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'Product name is required'],
        trim: true,
        maxlength: [100, 'Product name cannot exceed 100 characters']
    },
    description: {
        type: String,
        required: [true, 'Product description is required'],
        maxlength: [500, 'Description cannot exceed 500 characters']
    },
    price: {
        type: Number,
        required: [true, 'Product price is required'],
        min: [0, 'Price cannot be negative']
    },
    category: {
        type: String,
        required: [true, 'Product category is required'],
        enum: ['electronics', 'clothing', 'books', 'home', 'sports']
    },
    stock: {
        type: Number,
        required: [true, 'Stock quantity is required'],
        min: [0, 'Stock cannot be negative'],
        default: 0
    },
    images: [{
        type: String,
        validate: {
            validator: function(v) {
                return /^https?:\/\/.+\.(jpg|jpeg|png|gif|webp)$/i.test(v);
            },
            message: 'Invalid image URL'
        }
    }],
    tags: [{
        type: String,
        trim: true
    }],
    isActive: {
        type: Boolean,
        default: true
    },
    createdBy: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true
    }
}, {
    timestamps: true
});

// Index for search
productSchema.index({ name: 'text', description: 'text', tags: 'text' });

module.exports = mongoose.model('Product', productSchema);
```

## Tạo Controllers

### 1. User Controller

```javascript
const User = require('../models/User');
const jwt = require('jsonwebtoken');

// Generate JWT Token
const generateToken = (id) => {
    return jwt.sign({ id }, process.env.JWT_SECRET, {
        expiresIn: process.env.JWT_EXPIRE
    });
};

// @desc    Register user
// @route   POST /api/users/register
// @access  Public
const registerUser = async (req, res) => {
    try {
        const { username, email, password, firstName, lastName } = req.body;

        // Check if user exists
        const existingUser = await User.findOne({
            $or: [{ email }, { username }]
        });

        if (existingUser) {
            return res.status(400).json({
                success: false,
                message: 'User already exists with this email or username'
            });
        }

        // Create user
        const user = await User.create({
            username,
            email,
            password,
            firstName,
            lastName
        });

        const token = generateToken(user._id);

        res.status(201).json({
            success: true,
            message: 'User registered successfully',
            data: {
                user,
                token
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error registering user',
            error: error.message
        });
    }
};

// @desc    Login user
// @route   POST /api/users/login
// @access  Public
const loginUser = async (req, res) => {
    try {
        const { email, password } = req.body;

        // Check if user exists and include password
        const user = await User.findOne({ email }).select('+password');

        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'Invalid credentials'
            });
        }

        // Check password
        const isPasswordValid = await user.comparePassword(password);

        if (!isPasswordValid) {
            return res.status(401).json({
                success: false,
                message: 'Invalid credentials'
            });
        }

        // Update last login
        user.lastLogin = new Date();
        await user.save();

        const token = generateToken(user._id);

        res.json({
            success: true,
            message: 'Login successful',
            data: {
                user,
                token
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error logging in',
            error: error.message
        });
    }
};

// @desc    Get current user
// @route   GET /api/users/me
// @access  Private
const getMe = async (req, res) => {
    try {
        const user = await User.findById(req.user.id);

        res.json({
            success: true,
            data: user
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error getting user profile',
            error: error.message
        });
    }
};

// @desc    Get all users
// @route   GET /api/users
// @access  Private/Admin
const getUsers = async (req, res) => {
    try {
        const page = parseInt(req.query.page) || 1;
        const limit = parseInt(req.query.limit) || 10;
        const skip = (page - 1) * limit;

        const users = await User.find()
            .select('-password')
            .skip(skip)
            .limit(limit)
            .sort({ createdAt: -1 });

        const total = await User.countDocuments();

        res.json({
            success: true,
            data: users,
            pagination: {
                page,
                limit,
                total,
                pages: Math.ceil(total / limit)
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error getting users',
            error: error.message
        });
    }
};

module.exports = {
    registerUser,
    loginUser,
    getMe,
    getUsers
};
```

### 2. Product Controller

```javascript
const Product = require('../models/Product');

// @desc    Get all products
// @route   GET /api/products
// @access  Public
const getProducts = async (req, res) => {
    try {
        const page = parseInt(req.query.page) || 1;
        const limit = parseInt(req.query.limit) || 10;
        const skip = (page - 1) * limit;
        
        const { category, search, minPrice, maxPrice, sort } = req.query;
        
        // Build filter object
        const filter = { isActive: true };
        
        if (category) {
            filter.category = category;
        }
        
        if (search) {
            filter.$text = { $search: search };
        }
        
        if (minPrice || maxPrice) {
            filter.price = {};
            if (minPrice) filter.price.$gte = parseFloat(minPrice);
            if (maxPrice) filter.price.$lte = parseFloat(maxPrice);
        }
        
        // Build sort object
        let sortObj = { createdAt: -1 };
        if (sort) {
            switch (sort) {
                case 'price-asc':
                    sortObj = { price: 1 };
                    break;
                case 'price-desc':
                    sortObj = { price: -1 };
                    break;
                case 'name-asc':
                    sortObj = { name: 1 };
                    break;
                case 'name-desc':
                    sortObj = { name: -1 };
                    break;
            }
        }
        
        const products = await Product.find(filter)
            .populate('createdBy', 'username firstName lastName')
            .skip(skip)
            .limit(limit)
            .sort(sortObj);
        
        const total = await Product.countDocuments(filter);
        
        res.json({
            success: true,
            data: products,
            pagination: {
                page,
                limit,
                total,
                pages: Math.ceil(total / limit)
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error getting products',
            error: error.message
        });
    }
};

// @desc    Get single product
// @route   GET /api/products/:id
// @access  Public
const getProduct = async (req, res) => {
    try {
        const product = await Product.findById(req.params.id)
            .populate('createdBy', 'username firstName lastName');
        
        if (!product) {
            return res.status(404).json({
                success: false,
                message: 'Product not found'
            });
        }
        
        res.json({
            success: true,
            data: product
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error getting product',
            error: error.message
        });
    }
};

// @desc    Create product
// @route   POST /api/products
// @access  Private
const createProduct = async (req, res) => {
    try {
        const productData = {
            ...req.body,
            createdBy: req.user.id
        };
        
        const product = await Product.create(productData);
        
        res.status(201).json({
            success: true,
            message: 'Product created successfully',
            data: product
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error creating product',
            error: error.message
        });
    }
};

module.exports = {
    getProducts,
    getProduct,
    createProduct
};
```

## Middleware

### 1. Authentication Middleware

```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const protect = async (req, res, next) => {
    try {
        let token;
        
        if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
            token = req.headers.authorization.split(' ')[1];
        }
        
        if (!token) {
            return res.status(401).json({
                success: false,
                message: 'Access denied. No token provided.'
            });
        }
        
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        const user = await User.findById(decoded.id);
        
        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'Token is not valid.'
            });
        }
        
        req.user = user;
        next();
    } catch (error) {
        res.status(401).json({
            success: false,
            message: 'Token is not valid.'
        });
    }
};

const authorize = (...roles) => {
    return (req, res, next) => {
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({
                success: false,
                message: `User role ${req.user.role} is not authorized to access this route`
            });
        }
        next();
    };
};

module.exports = { protect, authorize };
```

### 2. Validation Middleware

```javascript
const { body, validationResult } = require('express-validator');

const handleValidationErrors = (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        return res.status(400).json({
            success: false,
            message: 'Validation failed',
            errors: errors.array()
        });
    }
    next();
};

const validateUser = [
    body('username')
        .isLength({ min: 3, max: 30 })
        .withMessage('Username must be between 3 and 30 characters'),
    body('email')
        .isEmail()
        .withMessage('Please provide a valid email'),
    body('password')
        .isLength({ min: 6 })
        .withMessage('Password must be at least 6 characters'),
    body('firstName')
        .notEmpty()
        .withMessage('First name is required'),
    body('lastName')
        .notEmpty()
        .withMessage('Last name is required'),
    handleValidationErrors
];

const validateProduct = [
    body('name')
        .notEmpty()
        .withMessage('Product name is required'),
    body('description')
        .notEmpty()
        .withMessage('Product description is required'),
    body('price')
        .isNumeric()
        .withMessage('Price must be a number'),
    body('category')
        .isIn(['electronics', 'clothing', 'books', 'home', 'sports'])
        .withMessage('Invalid category'),
    handleValidationErrors
];

module.exports = {
    validateUser,
    validateProduct,
    handleValidationErrors
};
```

## Routes

### 1. User Routes

```javascript
const express = require('express');
const router = express.Router();
const { protect, authorize } = require('../middleware/auth');
const { validateUser } = require('../middleware/validation');
const {
    registerUser,
    loginUser,
    getMe,
    getUsers
} = require('../controllers/userController');

router.post('/register', validateUser, registerUser);
router.post('/login', loginUser);
router.get('/me', protect, getMe);
router.get('/', protect, authorize('admin'), getUsers);

module.exports = router;
```

### 2. Product Routes

```javascript
const express = require('express');
const router = express.Router();
const { protect, authorize } = require('../middleware/auth');
const { validateProduct } = require('../middleware/validation');
const {
    getProducts,
    getProduct,
    createProduct
} = require('../controllers/productController');

router.get('/', getProducts);
router.get('/:id', getProduct);
router.post('/', protect, validateProduct, createProduct);

module.exports = router;
```

## Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: {
        success: false,
        message: 'Too many requests from this IP, please try again later.'
    }
});

const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // limit each IP to 5 requests per windowMs
    message: {
        success: false,
        message: 'Too many authentication attempts, please try again later.'
    }
});

module.exports = { limiter, authLimiter };
```

## Testing API

### 1. Register User

```bash
curl -X POST http://localhost:3000/api/users/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "johndoe",
    "email": "john@example.com",
    "password": "password123",
    "firstName": "John",
    "lastName": "Doe"
  }'
```

### 2. Login User

```bash
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "password123"
  }'
```

### 3. Get Products

```bash
curl -X GET "http://localhost:3000/api/products?page=1&limit=10&category=electronics"
```

## Kết Luận

Với Node.js và Express, bạn có thể xây dựng REST API mạnh mẽ và scalable. Những kiến thức này bao gồm:

- Thiết lập server và middleware
- Kết nối database với Mongoose
- Authentication và authorization
- Validation và error handling
- Rate limiting và security
- Pagination và filtering

Trong bài viết tiếp theo, tôi sẽ hướng dẫn về testing và deployment! 🚀


