# Day 3: MongoDB Models & POS System Backend Development

## Course Overview (8 Hours: 9:00 AM - 5:00 PM)
**Focus**: Creating MongoDB models, database connections, and building core POS system features

## Daily Schedule
- **9:00 - 10:00**: MongoDB Setup & Connection (60 min)
- **10:00 - 11:00**: Understanding Mongoose & Schemas (60 min)  
- **11:00 - 12:00**: Product Model Development (60 min)
- **12:00 - 13:00**: **LUNCH BREAK**
- **13:00 - 14:00**: Category Model & Relationships (60 min)
- **14:00 - 15:00**: Customer Model Development (60 min)
- **15:00 - 16:00**: Order Model Foundation (60 min)
- **16:00 - 17:00**: Advanced Order Management & Testing (60 min)

## Learning Objectives
By the end of Day 3, students will be able to:
- Set up MongoDB connection in Node.js/Express application
- Create and understand Mongoose schemas and models
- Implement CRUD operations for POS system entities
- Handle database relationships between models
- Validate data input and handle errors
- Test API endpoints using Postman or similar tools

---

## 9:00 - 10:00: MongoDB Setup & Connection (60 minutes)

### Topics Covered:
- **MongoDB Atlas setup** (15 minutes)
  - Creating free cluster
  - Database user creation
  - Network access configuration
  - Connection string generation

- **Node.js MongoDB Connection** (25 minutes)
  - Installing mongoose package
  - Creating database connection file
  - Environment variables for database URL
  - Connection error handling

- **Project Structure Setup** (20 minutes)
  - Creating models folder
  - Setting up database configuration
  - Basic Express app with MongoDB connection

### Exercise 1 (15 minutes):
Students connect their Express app to MongoDB Atlas and verify connection

### Key Code Blocks:

**1. Install Dependencies:**
```bash
npm init -y
npm install express mongoose dotenv cors
npm install -D nodemon
```

**2. Environment Configuration (.env):**
```env
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/pos_system
PORT=5000
NODE_ENV=development
```

**3. Database Connection (config/database.js):**
```javascript
const mongoose = require('mongoose');
require('dotenv').config();

const connectDB = async () => {
    try {
        const conn = await mongoose.connect(process.env.MONGODB_URI);
        console.log(`MongoDB Connected: ${conn.connection.host}`);
    } catch (error) {
        console.error('Database connection error:', error.message);
        process.exit(1);
    }
};

module.exports = connectDB;
```

**4. Basic Express App (app.js):**
```javascript
const express = require('express');
const cors = require('cors');
const connectDB = require('./config/database');
require('dotenv').config();

const app = express();

// Connect to MongoDB
connectDB();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Basic route
app.get('/', (req, res) => {
    res.json({ message: 'POS System API is running!' });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

**5. Package.json Scripts:**
```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js",
    "test": "echo \"No tests yet\""
  }
}
```

---

## 10:00 - 11:00: Understanding Mongoose & Schemas (60 minutes)

### Topics Covered:
- **Mongoose Fundamentals** (20 minutes)
  - What is Mongoose and why use it
  - Schema vs Model concept
  - Data types in Mongoose
  - Schema validation basics

- **Creating First Schema** (25 minutes)
  - Product schema creation
  - Field types and validation
  - Required fields and defaults
  - Timestamps functionality

- **Model Creation & Export** (15 minutes)
  - Converting schema to model
  - Proper module exports
  - Naming conventions

### Exercise 2 (20 minutes):
Students create a basic Product schema with validation rules

### Key Code Blocks:

**1. Basic Product Schema (models/Product.js):**
```javascript
const mongoose = require('mongoose');

// Define Product Schema
const productSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'Product name is required'],
        trim: true,
        maxlength: [100, 'Product name cannot exceed 100 characters']
    },
    description: {
        type: String,
        trim: true,
        maxlength: [500, 'Description cannot exceed 500 characters']
    },
    price: {
        type: Number,
        required: [true, 'Price is required'],
        min: [0, 'Price cannot be negative']
    },
    stock: {
        type: Number,
        required: [true, 'Stock quantity is required'],
        min: [0, 'Stock cannot be negative'],
        default: 0
    },
    isActive: {
        type: Boolean,
        default: true
    }
}, {
    timestamps: true // Automatically add createdAt and updatedAt
});

// Create and export the model
const Product = mongoose.model('Product', productSchema);
module.exports = Product;
```

**2. Schema Validation Examples:**
```javascript
// Different validation types
const exampleSchema = new mongoose.Schema({
    // String validations
    name: {
        type: String,
        required: true,        // Field is mandatory
        unique: true,          // Must be unique in collection
        trim: true,            // Remove whitespace
        lowercase: true,       // Convert to lowercase
        minlength: 3,          // Minimum length
        maxlength: 50,         // Maximum length
        match: /^[a-zA-Z]+$/   // Regex pattern
    },
    
    // Number validations
    price: {
        type: Number,
        required: true,
        min: 0,                // Minimum value
        max: 10000             // Maximum value
    },
    
    // Enum validation
    status: {
        type: String,
        enum: ['active', 'inactive', 'discontinued'],
        default: 'active'
    },
    
    // Custom validation
    email: {
        type: String,
        validate: {
            validator: function(v) {
                return /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v);
            },
            message: 'Please enter a valid email'
        }
    }
});
```

---

## 11:00 - 12:00: Product Model Development (60 minutes)

### Topics Covered:
- **Complete Product Schema** (25 minutes)
  - All required fields for POS products
  - Price handling and validation
  - Stock/inventory tracking
  - Category relationships

- **Product CRUD Operations** (25 minutes)
  - Create product endpoint
  - Read/Get products (all and by ID)
  - Update product information
  - Delete product (soft delete concept)

- **Error Handling** (10 minutes)
  - Validation error responses
  - Database error handling
  - Proper HTTP status codes

### Exercise 3 (30 minutes):
Students implement complete Product CRUD with validation and test with Postman

### Key Code Blocks:

**1. Complete Product Schema (models/Product.js):**
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
        trim: true,
        maxlength: [500, 'Description cannot exceed 500 characters']
    },
    price: {
        type: Number,
        required: [true, 'Price is required'],
        min: [0, 'Price cannot be negative']
    },
    stock: {
        type: Number,
        required: [true, 'Stock quantity is required'],
        min: [0, 'Stock cannot be negative'],
        default: 0
    },
    category: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Category',
        required: [true, 'Category is required']
    },
    barcode: {
        type: String,
        unique: true,
        sparse: true // Allows multiple null values
    },
    image: {
        type: String, // URL to product image
        default: null
    },
    isActive: {
        type: Boolean,
        default: true
    }
}, {
    timestamps: true
});

// Add index for better search performance
productSchema.index({ name: 'text', description: 'text' });

module.exports = mongoose.model('Product', productSchema);
```

**2. Product Routes (routes/products.js):**
```javascript
const express = require('express');
const router = express.Router();
const productController = require('../controllers/productController');

// Product routes
router.get('/', productController.getAllProducts);
router.get('/low-stock', productController.getLowStockProducts);
router.get('/:id', productController.getProductById);
router.post('/', productController.createProduct);
router.put('/:id', productController.updateProduct);
router.patch('/:id/stock', productController.updateStock);
router.delete('/:id', productController.deleteProduct);

module.exports = router;
```

**2b. Complete Product Controller (controllers/productController.js):**
```javascript
const Product = require('../models/Product');

// Get all products with search and filtering
const getAllProducts = async (req, res) => {
    try {
        const { search, category, lowStock, isActive = true, limit = 50, page = 1 } = req.query;
        
        let query = { isActive };
        
        // Search by name or description
        if (search) {
            query.$text = { $search: search };
        }
        
        // Filter by category
        if (category) {
            query.category = category;
        }
        
        // Filter for low stock products
        if (lowStock === 'true') {
            query.stock = { $lt: 10 };
        }
        
        const skip = (page - 1) * limit;
        
        const products = await Product.find(query)
            .populate('category', 'name color')
            .sort({ createdAt: -1 })
            .limit(parseInt(limit))
            .skip(skip);
        
        res.json({
            success: true,
            data: products,
            count: products.length
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching products',
            error: error.message
        });
    }
};

// Get single product by ID
const getProductById = async (req, res) => {
    try {
        const product = await Product.findById(req.params.id)
            .populate('category', 'name description');
        
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
            message: 'Error fetching product',
            error: error.message
        });
    }
};

// Create new product
const createProduct = async (req, res) => {
    try {
        const product = new Product(req.body);
        const savedProduct = await product.save();
        
        res.status(201).json({
            success: true,
            message: 'Product created successfully',
            data: savedProduct
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error creating product',
            error: error.message
        });
    }
};

// Update product
const updateProduct = async (req, res) => {
    try {
        const product = await Product.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        );
        
        if (!product) {
            return res.status(404).json({
                success: false,
                message: 'Product not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Product updated successfully',
            data: product
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error updating product',
            error: error.message
        });
    }
};

// Update product stock
const updateStock = async (req, res) => {
    try {
        const { stock, operation } = req.body; // operation: 'add' or 'set'
        
        const product = await Product.findById(req.params.id);
        if (!product) {
            return res.status(404).json({
                success: false,
                message: 'Product not found'
            });
        }
        
        if (operation === 'add') {
            product.stock += parseInt(stock);
        } else {
            product.stock = parseInt(stock);
        }
        
        const updatedProduct = await product.save();
        
        res.json({
            success: true,
            message: 'Stock updated successfully',
            data: updatedProduct
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error updating stock',
            error: error.message
        });
    }
};

// Delete product (soft delete)
const deleteProduct = async (req, res) => {
    try {
        const product = await Product.findByIdAndUpdate(
            req.params.id,
            { isActive: false },
            { new: true }
        );
        
        if (!product) {
            return res.status(404).json({
                success: false,
                message: 'Product not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Product deleted successfully'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error deleting product',
            error: error.message
        });
    }
};

// Get low stock products
const getLowStockProducts = async (req, res) => {
    try {
        const threshold = req.query.threshold || 10;
        
        const lowStockProducts = await Product.find({
            stock: { $lt: parseInt(threshold) },
            isActive: true
        }).populate('category', 'name');
        
        res.json({
            success: true,
            data: lowStockProducts,
            count: lowStockProducts.length
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching low stock products',
            error: error.message
        });
    }
};

module.exports = {
    getAllProducts,
    getProductById,
    createProduct,
    updateProduct,
    updateStock,
    deleteProduct,
    getLowStockProducts
};
```

**3. Updated App.js with Product Routes:**
```javascript
const express = require('express');
const cors = require('cors');
const connectDB = require('./config/database');
require('dotenv').config();

const app = express();

// Connect to MongoDB
connectDB();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/products', require('./routes/products'));

// Basic route
app.get('/', (req, res) => {
    res.json({ message: 'POS System API is running!' });
});

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({
        success: false,
        message: 'Something went wrong!',
        error: process.env.NODE_ENV === 'development' ? err.message : {}
    });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

---

---

## 12:00 - 13:00: **LUNCH BREAK** 

---

## 13:00 - 14:00: Category Model & Relationships (60 minutes)

### Topics Covered:
- **Category Schema Design** (20 minutes)
  - Category model structure
  - Hierarchical categories (optional)
  - Category validation

- **Model Relationships** (25 minutes)
  - Referencing categories in products
  - Population concept in Mongoose
  - One-to-many relationships

- **Category CRUD Implementation** (15 minutes)
  - Category management endpoints
  - Relationship handling in queries

### Exercise 4 (25 minutes):
Students create Category model and establish relationship with Products

### Key Code Blocks:

**1. Category Schema (models/Category.js):**
```javascript
const mongoose = require('mongoose');

const categorySchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'Category name is required'],
        unique: true,
        trim: true,
        maxlength: [50, 'Category name cannot exceed 50 characters']
    },
    description: {
        type: String,
        trim: true,
        maxlength: [200, 'Description cannot exceed 200 characters']
    },
    color: {
        type: String,
        default: '#007bff', // Bootstrap primary color
        match: [/^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/, 'Please enter a valid hex color']
    },
    isActive: {
        type: Boolean,
        default: true
    }
}, {
    timestamps: true
});

module.exports = mongoose.model('Category', categorySchema);
```

**2. Category Routes (routes/categories.js):**
```javascript
const express = require('express');
const router = express.Router();
const categoryController = require('../controllers/categoryController');

// Category routes
router.get('/', categoryController.getAllCategories);
router.get('/:id', categoryController.getCategoryById);
router.get('/:id/products', categoryController.getCategoryProducts);
router.post('/', categoryController.createCategory);
router.put('/:id', categoryController.updateCategory);
router.delete('/:id', categoryController.deleteCategory);

module.exports = router;
```

**2b. Category Controller (controllers/categoryController.js):**
```javascript
const Category = require('../models/Category');
const Product = require('../models/Product');

// Get all categories
const getAllCategories = async (req, res) => {
    try {
        const categories = await Category.find({ isActive: true })
            .sort({ name: 1 });
        
        res.json({
            success: true,
            data: categories,
            count: categories.length
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching categories',
            error: error.message
        });
    }
};

// Get single category
const getCategoryById = async (req, res) => {
    try {
        const category = await Category.findById(req.params.id);
        
        if (!category) {
            return res.status(404).json({
                success: false,
                message: 'Category not found'
            });
        }
        
        res.json({
            success: true,
            data: category
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching category',
            error: error.message
        });
    }
};

// Get category with products
const getCategoryProducts = async (req, res) => {
    try {
        const category = await Category.findById(req.params.id);
        
        if (!category) {
            return res.status(404).json({
                success: false,
                message: 'Category not found'
            });
        }
        
        const products = await Product.find({ 
            category: req.params.id, 
            isActive: true 
        });
        
        res.json({
            success: true,
            data: {
                category,
                products,
                productCount: products.length
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching category products',
            error: error.message
        });
    }
};

// Create new category
const createCategory = async (req, res) => {
    try {
        const category = new Category(req.body);
        const savedCategory = await category.save();
        
        res.status(201).json({
            success: true,
            message: 'Category created successfully',
            data: savedCategory
        });
    } catch (error) {
        if (error.code === 11000) {
            res.status(400).json({
                success: false,
                message: 'Category name already exists'
            });
        } else {
            res.status(400).json({
                success: false,
                message: 'Error creating category',
                error: error.message
            });
        }
    }
};

// Update category
const updateCategory = async (req, res) => {
    try {
        const category = await Category.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        );
        
        if (!category) {
            return res.status(404).json({
                success: false,
                message: 'Category not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Category updated successfully',
            data: category
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error updating category',
            error: error.message
        });
    }
};

// Delete category
const deleteCategory = async (req, res) => {
    try {
        // Check if category has products
        const productCount = await Product.countDocuments({ 
            category: req.params.id, 
            isActive: true 
        });
        
        if (productCount > 0) {
            return res.status(400).json({
                success: false,
                message: `Cannot delete category. ${productCount} products are using this category.`
            });
        }
        
        const category = await Category.findByIdAndUpdate(
            req.params.id,
            { isActive: false },
            { new: true }
        );
        
        if (!category) {
            return res.status(404).json({
                success: false,
                message: 'Category not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Category deleted successfully'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error deleting category',
            error: error.message
        });
    }
};

module.exports = {
    getAllCategories,
    getCategoryById,
    getCategoryProducts,
    createCategory,
    updateCategory,
    deleteCategory
};
```

**3. Population Examples:**
```javascript
// Simple population
const products = await Product.find().populate('category');

// Population with field selection
const products = await Product.find().populate('category', 'name color');

// Population with conditions
const products = await Product.find().populate({
    path: 'category',
    match: { isActive: true },
    select: 'name description color'
});

// Multiple populations
const products = await Product.find()
    .populate('category', 'name color')
    .populate('supplier', 'name contact');
```

## 14:00 - 15:00: Customer Model Development (60 minutes)

### Topics Covered:
- **Customer Schema Design** (20 minutes)
  - Customer information fields
  - Contact information validation
  - Customer type (regular, VIP, etc.)

- **Customer CRUD Operations** (25 minutes)
  - Customer registration
  - Customer information management
  - Search functionality

- **Data Validation & Sanitization** (15 minutes)
  - Email validation
  - Phone number formatting
  - Data sanitization techniques

### Exercise 5 (25 minutes):
Students build Customer model with proper validation and search functionality

### Key Code Blocks:

**1. Customer Schema (models/Customer.js):**
```javascript
const mongoose = require('mongoose');

const customerSchema = new mongoose.Schema({
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
        unique: true,
        sparse: true, // Allows multiple null values
        lowercase: true,
        validate: {
            validator: function(v) {
                return !v || /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v);
            },
            message: 'Please enter a valid email address'
        }
    },
    phone: {
        type: String,
        required: [true, 'Phone number is required'],
        validate: {
            validator: function(v) {
                return /^\+?[\d\s-()]{8,15}$/.test(v);
            },
            message: 'Please enter a valid phone number'
        }
    },
    address: {
        street: { type: String, trim: true },
        city: { type: String, trim: true },
        state: { type: String, trim: true },
        zipCode: { type: String, trim: true },
        country: { type: String, default: 'USA', trim: true }
    },
    customerType: {
        type: String,
        enum: ['regular', 'vip', 'wholesale'],
        default: 'regular'
    },
    discount: {
        type: Number,
        min: 0,
        max: 100,
        default: 0 // Percentage discount
    },
    isActive: {
        type: Boolean,
        default: true
    },
    notes: {
        type: String,
        maxlength: [500, 'Notes cannot exceed 500 characters']
    }
}, {
    timestamps: true
});

// Virtual for full name
customerSchema.virtual('fullName').get(function() {
    return `${this.firstName} ${this.lastName}`;
});

// Index for search functionality
customerSchema.index({ 
    firstName: 'text', 
    lastName: 'text', 
    email: 'text',
    phone: 'text'
});

module.exports = mongoose.model('Customer', customerSchema);
```

**2. Customer Routes (routes/customers.js):**
```javascript
const express = require('express');
const router = express.Router();
const customerController = require('../controllers/customerController');

// Customer routes
router.get('/', customerController.getAllCustomers);
router.get('/:id', customerController.getCustomerById);
router.post('/', customerController.createCustomer);
router.put('/:id', customerController.updateCustomer);
router.delete('/:id', customerController.deleteCustomer);

module.exports = router;
```

**2b. Customer Controller (controllers/customerController.js):**
```javascript
const Customer = require('../models/Customer');

// Get all customers with search and filtering
const getAllCustomers = async (req, res) => {
    try {
        const { search, type, limit = 50, page = 1 } = req.query;
        let query = { isActive: true };
        
        // Search functionality
        if (search) {
            query.$text = { $search: search };
        }
        
        // Filter by customer type
        if (type) {
            query.customerType = type;
        }
        
        const skip = (page - 1) * limit;
        
        const customers = await Customer.find(query)
            .sort({ lastName: 1, firstName: 1 })
            .limit(parseInt(limit))
            .skip(skip);
        
        res.json({
            success: true,
            data: customers,
            count: customers.length
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching customers',
            error: error.message
        });
    }
};

// Get single customer
const getCustomerById = async (req, res) => {
    try {
        const customer = await Customer.findById(req.params.id);
        
        if (!customer) {
            return res.status(404).json({
                success: false,
                message: 'Customer not found'
            });
        }
        
        res.json({
            success: true,
            data: customer
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching customer',
            error: error.message
        });
    }
};

// Create new customer
const createCustomer = async (req, res) => {
    try {
        const customer = new Customer(req.body);
        const savedCustomer = await customer.save();
        
        res.status(201).json({
            success: true,
            message: 'Customer created successfully',
            data: savedCustomer
        });
    } catch (error) {
        if (error.code === 11000) {
            res.status(400).json({
                success: false,
                message: 'Email address already exists'
            });
        } else {
            res.status(400).json({
                success: false,
                message: 'Error creating customer',
                error: error.message
            });
        }
    }
};

// Update customer
const updateCustomer = async (req, res) => {
    try {
        const customer = await Customer.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }
        );
        
        if (!customer) {
            return res.status(404).json({
                success: false,
                message: 'Customer not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Customer updated successfully',
            data: customer
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error updating customer',
            error: error.message
        });
    }
};

// Delete customer (soft delete)
const deleteCustomer = async (req, res) => {
    try {
        const customer = await Customer.findByIdAndUpdate(
            req.params.id,
            { isActive: false },
            { new: true }
        );
        
        if (!customer) {
            return res.status(404).json({
                success: false,
                message: 'Customer not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Customer deleted successfully'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error deleting customer',
            error: error.message
        });
    }
};

module.exports = {
    getAllCustomers,
    getCustomerById,
    createCustomer,
    updateCustomer,
    deleteCustomer
};
```

**3. Search and Filter Examples:**
```javascript
// Text search across multiple fields
const customers = await Customer.find({
    $text: { $search: "john smith" }
});

// Filter by customer type
const vipCustomers = await Customer.find({
    customerType: 'vip',
    isActive: true
});

// Combined search and filter
const searchResults = await Customer.find({
    $and: [
        { $text: { $search: req.query.search } },
        { customerType: req.query.type },
        { isActive: true }
    ]
});

// Regex search for partial matches
const phoneSearch = await Customer.find({
    phone: { $regex: req.query.phone, $options: 'i' }
});
```

---

## 15:00 - 16:00: Order Model Foundation (60 minutes)

### Topics Covered:
- **Order Schema Design** (30 minutes)
  - Order structure and fields
  - Order status management
  - Date/time handling
  - Customer reference

- **Order Item Schema** (20 minutes)
  - Embedded vs referenced approach
  - Product quantity and pricing
  - Order totals calculation

- **Basic Order Operations** (10 minutes)
  - Create new order
  - Order status updates
  - Basic order retrieval

### Exercise 6 (20 minutes):
Students create Order and OrderItem schemas with proper relationships

### Key Code Blocks:

**1. Order Schema (models/Order.js):**
```javascript
const mongoose = require('mongoose');

// OrderItem subdocument schema
const orderItemSchema = new mongoose.Schema({
    product: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Product',
        required: true
    },
    name: {
        type: String,
        required: true // Store product name at time of order
    },
    price: {
        type: Number,
        required: true // Store price at time of order
    },
    quantity: {
        type: Number,
        required: true,
        min: 1
    },
    subtotal: {
        type: Number,
        required: true
    }
}, { _id: true });

// Calculate subtotal before saving
orderItemSchema.pre('save', function() {
    this.subtotal = this.price * this.quantity;
});

const orderSchema = new mongoose.Schema({
    orderNumber: {
        type: String,
        unique: true,
        required: true
    },
    customer: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Customer',
        default: null // null for walk-in customers
    },
    customerName: {
        type: String,
        required: true // Store customer name even for walk-ins
    },
    items: [orderItemSchema],
    subtotal: {
        type: Number,
        required: true,
        default: 0
    },
    tax: {
        type: Number,
        required: true,
        default: 0
    },
    discount: {
        type: Number,
        default: 0,
        min: 0
    },
    total: {
        type: Number,
        required: true,
        default: 0
    },
    status: {
        type: String,
        enum: ['pending', 'processing', 'completed', 'cancelled'],
        default: 'pending'
    },
    paymentMethod: {
        type: String,
        enum: ['cash', 'card', 'mobile'],
        required: true
    },
    paymentStatus: {
        type: String,
        enum: ['pending', 'paid', 'refunded'],
        default: 'pending'
    },
    notes: {
        type: String,
        maxlength: 500
    }
}, {
    timestamps: true
});

// Generate order number before saving
orderSchema.pre('save', async function(next) {
    if (this.isNew) {
        const today = new Date();
        const dateStr = today.toISOString().slice(0, 10).replace(/-/g, '');
        const count = await this.constructor.countDocuments({
            createdAt: {
                $gte: new Date(today.getFullYear(), today.getMonth(), today.getDate()),
                $lt: new Date(today.getFullYear(), today.getMonth(), today.getDate() + 1)
            }
        });
        this.orderNumber = `ORD-${dateStr}-${String(count + 1).padStart(3, '0')}`;
    }
    next();
});

// Calculate totals before saving
orderSchema.pre('save', function() {
    this.subtotal = this.items.reduce((sum, item) => sum + item.subtotal, 0);
    this.total = this.subtotal + this.tax - this.discount;
});

module.exports = mongoose.model('Order', orderSchema);
```

**2. Basic Order Routes (routes/orders.js):**
```javascript
const express = require('express');
const router = express.Router();
const orderController = require('../controllers/orderController');

// Order routes
router.get('/', orderController.getAllOrders);
router.get('/reports/daily', orderController.getDailySalesReport);
router.get('/:id', orderController.getOrderById);
router.post('/', orderController.createOrder);
router.patch('/:id/status', orderController.updateOrderStatus);

module.exports = router;
```

**2b. Basic Order Controller Functions (controllers/orderController.js - basic version):**
```javascript
const Order = require('../models/Order');
const Product = require('../models/Product');

// Get all orders (basic version)
const getAllOrders = async (req, res) => {
    try {
        const orders = await Order.find()
            .populate('customer', 'firstName lastName phone')
            .populate('items.product', 'name')
            .sort({ createdAt: -1 })
            .limit(50);
        
        res.json({
            success: true,
            data: orders,
            count: orders.length
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching orders',
            error: error.message
        });
    }
};

// Get single order
const getOrderById = async (req, res) => {
    try {
        const order = await Order.findById(req.params.id)
            .populate('customer')
            .populate('items.product', 'name category');
        
        if (!order) {
            return res.status(404).json({
                success: false,
                message: 'Order not found'
            });
        }
        
        res.json({
            success: true,
            data: order
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching order',
            error: error.message
        });
    }
};

// Create new order (basic version - without stock management)
const createOrder = async (req, res) => {
    try {
        const { items, customer, customerName, paymentMethod, notes } = req.body;
        
        // Validate and prepare order items
        const orderItems = [];
        for (let item of items) {
            const product = await Product.findById(item.product);
            if (!product) {
                return res.status(400).json({
                    success: false,
                    message: `Product not found: ${item.product}`
                });
            }
            
            orderItems.push({
                product: product._id,
                name: product.name,
                price: product.price,
                quantity: item.quantity,
                subtotal: product.price * item.quantity
            });
        }
        
        const order = new Order({
            customer: customer || null,
            customerName: customerName || 'Walk-in Customer',
            items: orderItems,
            paymentMethod,
            notes
        });
        
        const savedOrder = await order.save();
        
        res.status(201).json({
            success: true,
            message: 'Order created successfully',
            data: savedOrder
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error creating order',
            error: error.message
        });
    }
};

// Update order status
const updateOrderStatus = async (req, res) => {
    try {
        const { status } = req.body;
        
        const order = await Order.findByIdAndUpdate(
            req.params.id,
            { status },
            { new: true, runValidators: true }
        );
        
        if (!order) {
            return res.status(404).json({
                success: false,
                message: 'Order not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Order status updated successfully',
            data: order
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error updating order status',
            error: error.message
        });
    }
};

// Placeholder for daily sales report (will be enhanced in Hour 7)
const getDailySalesReport = async (req, res) => {
    try {
        const today = new Date();
        const startOfDay = new Date(today.setHours(0, 0, 0, 0));
        const endOfDay = new Date(today.setHours(23, 59, 59, 999));
        
        const orders = await Order.find({
            createdAt: { $gte: startOfDay, $lte: endOfDay },
            status: { $ne: 'cancelled' }
        });
        
        const totalOrders = orders.length;
        const totalRevenue = orders.reduce((sum, order) => sum + order.total, 0);
        
        res.json({
            success: true,
            data: {
                date: startOfDay.toISOString().split('T')[0],
                totalOrders,
                totalRevenue
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error generating sales report',
            error: error.message
        });
    }
};

module.exports = {
    getAllOrders,
    getOrderById,
    createOrder,
    updateOrderStatus,
    getDailySalesReport
};
```

**3. Order Creation Examples:**
```javascript
// Example order creation request body
const sampleOrder = {
    customer: "647abc123def456789012345", // Optional customer ID
    customerName: "John Doe",
    items: [
        {
            product: "647def456abc789012345678",
            quantity: 2
        },
        {
            product: "647ghi789def012345678901",
            quantity: 1
        }
    ],
    paymentMethod: "card",
    notes: "Customer requested extra napkins"
};

// Order validation before creation
const validateOrderItems = async (items) => {
    for (let item of items) {
        const product = await Product.findById(item.product);
        if (!product) {
            throw new Error(`Product not found: ${item.product}`);
        }
        if (product.stock < item.quantity) {
            throw new Error(`Insufficient stock for ${product.name}`);
        }
    }
};
```

---

## 16:00 - 17:00: Advanced Order Management & Integration Testing (60 minutes)

### Topics Covered:
- **Complete Order CRUD** (25 minutes)
  - Create order with multiple items
  - Update order status
  - Calculate order totals
  - Order history

- **Inventory Management** (20 minutes)
  - Stock reduction on order
  - Stock validation
  - Low stock alerts

- **Order Reporting Basics** (10 minutes)
  - Daily sales summary
  - Order statistics
  - Simple reporting queries

- **Integration Testing** (25 minutes)
  - Testing all endpoints with Postman
  - Complete POS workflow testing
  - Error handling validation

### Exercise 7 (20 minutes):
Students implement complete order processing with inventory management and test the full system

### Key Code Blocks:

**1. Order Controller (controllers/orderController.js):**
```javascript
const Order = require('../models/Order');
const Product = require('../models/Product');

// Create new order with inventory management
const createOrder = async (req, res) => {
    try {
        const { items, customer, customerName, paymentMethod, tax, discount, notes } = req.body;
        
        // Validate and prepare order items with stock check
        const orderItems = [];
        const stockUpdates = [];
        
        for (let item of items) {
            const product = await Product.findById(item.product);
            if (!product) {
                return res.status(400).json({
                    success: false,
                    message: `Product not found: ${item.product}`
                });
            }
            
            if (product.stock < item.quantity) {
                return res.status(400).json({
                    success: false,
                    message: `Insufficient stock for ${product.name}. Available: ${product.stock}, Requested: ${item.quantity}`
                });
            }
            
            orderItems.push({
                product: product._id,
                name: product.name,
                price: product.price,
                quantity: item.quantity,
                subtotal: product.price * item.quantity
            });
            
            // Prepare stock update
            stockUpdates.push({
                productId: product._id,
                newStock: product.stock - item.quantity
            });
        }
        
        // Create order
        const order = new Order({
            customer: customer || null,
            customerName: customerName || 'Walk-in Customer',
            items: orderItems,
            tax: tax || 0,
            discount: discount || 0,
            paymentMethod,
            notes
        });
        
        const savedOrder = await order.save();
        
        // Update product stock
        for (let update of stockUpdates) {
            await Product.findByIdAndUpdate(
                update.productId,
                { stock: update.newStock }
            );
        }
        
        res.status(201).json({
            success: true,
            message: 'Order created successfully',
            data: savedOrder
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error creating order',
            error: error.message
        });
    }
};

// Get all orders with filtering
const getAllOrders = async (req, res) => {
    try {
        const { status, startDate, endDate, customer, limit = 50, page = 1 } = req.query;
        
        let query = {};
        
        // Filter by status
        if (status) {
            query.status = status;
        }
        
        // Filter by date range
        if (startDate || endDate) {
            query.createdAt = {};
            if (startDate) query.createdAt.$gte = new Date(startDate);
            if (endDate) query.createdAt.$lte = new Date(endDate);
        }
        
        // Filter by customer
        if (customer) {
            query.customer = customer;
        }
        
        const skip = (page - 1) * limit;
        
        const orders = await Order.find(query)
            .populate('customer', 'firstName lastName phone email')
            .populate('items.product', 'name category')
            .sort({ createdAt: -1 })
            .limit(parseInt(limit))
            .skip(skip);
        
        const total = await Order.countDocuments(query);
        
        res.json({
            success: true,
            data: orders,
            pagination: {
                current: parseInt(page),
                total: Math.ceil(total / limit),
                count: orders.length,
                totalOrders: total
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching orders',
            error: error.message
        });
    }
};

// Get daily sales report
const getDailySalesReport = async (req, res) => {
    try {
        const { date } = req.query;
        const targetDate = date ? new Date(date) : new Date();
        
        const startOfDay = new Date(targetDate.setHours(0, 0, 0, 0));
        const endOfDay = new Date(targetDate.setHours(23, 59, 59, 999));
        
        const orders = await Order.find({
            createdAt: { $gte: startOfDay, $lte: endOfDay },
            status: { $ne: 'cancelled' }
        });
        
        const report = {
            date: startOfDay.toISOString().split('T')[0],
            totalOrders: orders.length,
            totalRevenue: orders.reduce((sum, order) => sum + order.total, 0),
            averageOrderValue: orders.length > 0 ? 
                orders.reduce((sum, order) => sum + order.total, 0) / orders.length : 0,
            paymentMethods: {
                cash: orders.filter(o => o.paymentMethod === 'cash').length,
                card: orders.filter(o => o.paymentMethod === 'card').length,
                mobile: orders.filter(o => o.paymentMethod === 'mobile').length
            },
            orderStatus: {
                pending: orders.filter(o => o.status === 'pending').length,
                processing: orders.filter(o => o.status === 'processing').length,
                completed: orders.filter(o => o.status === 'completed').length
            }
        };
        
        res.json({
            success: true,
            data: report
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error generating sales report',
            error: error.message
        });
    }
};

// Update order status
const updateOrderStatus = async (req, res) => {
    try {
        const { status, paymentStatus } = req.body;
        
        const order = await Order.findByIdAndUpdate(
            req.params.id,
            { 
                status: status || undefined,
                paymentStatus: paymentStatus || undefined
            },
            { new: true, runValidators: true }
        ).populate('customer', 'firstName lastName');
        
        if (!order) {
            return res.status(404).json({
                success: false,
                message: 'Order not found'
            });
        }
        
        res.json({
            success: true,
            message: 'Order updated successfully',
            data: order
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error updating order',
            error: error.message
        });
    }
};

module.exports = {
    createOrder,
    getAllOrders,
    getDailySalesReport,
    updateOrderStatus
};
```

**2. Product Controller (controllers/productController.js):**
```javascript
const Product = require('../models/Product');

// Get all products with search and filtering
const getAllProducts = async (req, res) => {
    try {
        const { search, category, lowStock, isActive = true, limit = 50, page = 1 } = req.query;
        
        let query = { isActive };
        
        // Search by name or description
        if (search) {
            query.$text = { $search: search };
        }
        
        // Filter by category
        if (category) {
            query.category = category;
        }
        
        // Filter for low stock products
        if (lowStock === 'true') {
            query.stock = { $lt: 10 }; // Less than 10 items
        }
        
        const skip = (page - 1) * limit;
        
        const products = await Product.find(query)
            .populate('category', 'name color')
            .sort({ createdAt: -1 })
            .limit(parseInt(limit))
            .skip(skip);
        
        res.json({
            success: true,
            data: products,
            count: products.length
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching products',
            error: error.message
        });
    }
};

// Create new product
const createProduct = async (req, res) => {
    try {
        const product = new Product(req.body);
        const savedProduct = await product.save();
        
        res.status(201).json({
            success: true,
            message: 'Product created successfully',
            data: savedProduct
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error creating product',
            error: error.message
        });
    }
};

// Update product stock
const updateStock = async (req, res) => {
    try {
        const { stock, operation } = req.body; // operation: 'add' or 'set'
        
        const product = await Product.findById(req.params.id);
        if (!product) {
            return res.status(404).json({
                success: false,
                message: 'Product not found'
            });
        }
        
        if (operation === 'add') {
            product.stock += parseInt(stock);
        } else {
            product.stock = parseInt(stock);
        }
        
        const updatedProduct = await product.save();
        
        res.json({
            success: true,
            message: 'Stock updated successfully',
            data: updatedProduct
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            message: 'Error updating stock',
            error: error.message
        });
    }
};

// Get low stock alert
const getLowStockProducts = async (req, res) => {
    try {
        const threshold = req.query.threshold || 10;
        
        const lowStockProducts = await Product.find({
            stock: { $lt: parseInt(threshold) },
            isActive: true
        }).populate('category', 'name');
        
        res.json({
            success: true,
            data: lowStockProducts,
            count: lowStockProducts.length
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: 'Error fetching low stock products',
            error: error.message
        });
    }
};

module.exports = {
    getAllProducts,
    createProduct,
    updateStock,
    getLowStockProducts
};
```

**3. Updated Routes with Controllers (routes/orders.js):**
```javascript
const express = require('express');
const router = express.Router();
const orderController = require('../controllers/orderController');

// Order routes
router.get('/', orderController.getAllOrders);
router.post('/', orderController.createOrder);
router.patch('/:id/status', orderController.updateOrderStatus);
router.get('/reports/daily', orderController.getDailySalesReport);

module.exports = router;
```

---

### Final Integration Testing:
- **Complete System Test** - Students test the entire POS workflow
- **Error Handling Validation** - Test various error scenarios
- **Performance Review** - Check all endpoints work correctly
- **Code Review** - Ensure proper MVC structure and best practices

### Day 3 Wrap-up:
- **System Demo** - Each student demonstrates their working POS API
- **Q&A Session** - Address any remaining questions
- **Day 4 Preview** - Authentication and user management overview

### Key Code Blocks:

**1. Complete App.js with All Routes:**
```javascript
const express = require('express');
const cors = require('cors');
const connectDB = require('./config/database');
require('dotenv').config();

const app = express();

// Connect to MongoDB
connectDB();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/products', require('./routes/products'));
app.use('/api/categories', require('./routes/categories'));
app.use('/api/customers', require('./routes/customers'));
app.use('/api/orders', require('./routes/orders'));

// Basic route
app.get('/', (req, res) => {
    res.json({ 
        message: 'POS System API is running!',
        endpoints: {
            products: '/api/products',
            categories: '/api/categories',
            customers: '/api/customers',
            orders: '/api/orders'
        }
    });
});

// Error handling middleware
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({
        success: false,
        message: 'Something went wrong!',
        error: process.env.NODE_ENV === 'development' ? err.message : {}
    });
});

// Handle 404 routes
app.use('*', (req, res) => {
    res.status(404).json({
        success: false,
        message: 'Route not found'
    });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
    console.log(`Environment: ${process.env.NODE_ENV || 'development'}`);
});

module.exports = app;
```

**2. Postman Testing Collection Examples:**
```javascript
// 1. Create Category
POST http://localhost:5000/api/categories
Content-Type: application/json

{
    "name": "Beverages",
    "description": "All drinks and beverages",
    "color": "#28a745"
}

// 2. Create Product
POST http://localhost:5000/api/products
Content-Type: application/json

{
    "name": "Coca Cola",
    "description": "Refreshing cola drink",
    "price": 2.50,
    "stock": 100,
    "category": "647abc123def456789012345",
    "barcode": "123456789"
}

// 3. Create Customer
POST http://localhost:5000/api/customers
Content-Type: application/json

{
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@email.com",
    "phone": "+1234567890",
    "address": {
        "street": "123 Main St",
        "city": "New York",
        "state": "NY",
        "zipCode": "10001"
    },
    "customerType": "regular"
}

// 4. Create Order
POST http://localhost:5000/api/orders
Content-Type: application/json

{
    "customer": "647def456abc789012345678",
    "customerName": "John Doe",
    "items": [
        {
            "product": "647ghi789def012345678901",
            "quantity": 2
        }
    ],
    "paymentMethod": "card",
    "tax": 0.50,
    "discount": 0,
    "notes": "Customer requested receipt"
}
```

**3. Error Handling Examples:**
```javascript
// Global error handler middleware (middleware/errorHandler.js)
const errorHandler = (err, req, res, next) => {
    let error = { ...err };
    error.message = err.message;

    console.log(err);

    // Mongoose bad ObjectId
    if (err.name === 'CastError') {
        const message = 'Resource not found';
        error = { message, statusCode: 404 };
    }

    // Mongoose duplicate key
    if (err.code === 11000) {
        const message = 'Duplicate field value entered';
        error = { message, statusCode: 400 };
    }

    // Mongoose validation error
    if (err.name === 'ValidationError') {
        const message = Object.values(err.errors).map(val => val.message);
        error = { message, statusCode: 400 };
    }

    res.status(error.statusCode || 500).json({
        success: false,
        message: error.message || 'Server Error'
    });
};

module.exports = errorHandler;
```

**4. Database Seeder (optional - for testing):**
```javascript
// scripts/seeder.js
const mongoose = require('mongoose');
const Category = require('../models/Category');
const Product = require('../models/Product');
const Customer = require('../models/Customer');
require('dotenv').config();

const seedData = async () => {
    try {
        await mongoose.connect(process.env.MONGODB_URI);
        
        // Clear existing data
        await Category.deleteMany();
        await Product.deleteMany();
        await Customer.deleteMany();
        
        // Create categories
        const categories = await Category.insertMany([
            { name: 'Beverages', description: 'Drinks and beverages', color: '#007bff' },
            { name: 'Snacks', description: 'Chips and snacks', color: '#28a745' },
            { name: 'Electronics', description: 'Electronic items', color: '#ffc107' }
        ]);
        
        // Create products
        await Product.insertMany([
            {
                name: 'Coca Cola',
                description: 'Refreshing cola drink',
                price: 2.50,
                stock: 100,
                category: categories[0]._id
            },
            {
                name: 'Potato Chips',
                description: 'Crispy potato chips',
                price: 3.99,
                stock: 50,
                category: categories[1]._id
            }
        ]);
        
        console.log('Sample data inserted successfully');
        process.exit(0);
    } catch (error) {
        console.error('Error seeding data:', error);
        process.exit(1);
    }
};

seedData();
```

**5. Testing Checklist for Students:**
```javascript
// Testing workflow checklist:

// 1. Server Setup
// □ Server starts without errors
// □ Database connection successful
// □ All routes accessible

// 2. Category Testing
// □ Create category
// □ Get all categories
// □ Update category
// □ Delete category (with validation)

// 3. Product Testing
// □ Create product with category
// □ Get all products
// □ Search products
// □ Update product stock
// □ Low stock alerts

// 4. Customer Testing
// □ Create customer
// □ Search customers
// □ Update customer info
// □ Customer validation (email, phone)

// 5. Order Testing
// □ Create order with multiple items
// □ Stock reduction after order
// □ Order status updates
// □ Payment status tracking
// □ Daily sales report

// 6. Error Testing
// □ Invalid product ID
// □ Insufficient stock
// □ Duplicate category names
// □ Invalid email formats
// □ Missing required fields
```

---

## Daily Exercises Summary

### Exercise 1: MongoDB Connection
- Set up MongoDB Atlas
- Connect Node.js app to database
- Verify connection success

### Exercise 2: Basic Schema
- Create Product schema with validation
- Test schema validation
- Export model properly

### Exercise 3: Product CRUD
- Implement all Product CRUD operations
- Add proper error handling
- Test with Postman

### Exercise 4: Categories & Relationships
- Create Category model
- Establish Product-Category relationship
- Test population queries

### Exercise 5: Customer Management
- Build Customer model with validation
- Implement customer search
- Test customer operations

### Exercise 6: Order Foundation
- Design Order and OrderItem schemas
- Create basic order structure
- Test order creation

### Exercise 7: Complete Order System
- Implement full order processing
- Add inventory management
- Create basic reporting

### Final Exercise: System Integration
- Test complete POS workflow
- Verify all relationships work
- Handle error scenarios

---

## Required Dependencies
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "mongoose": "^7.5.0",
    "dotenv": "^16.3.0",
    "cors": "^2.8.5",
    "nodemon": "^3.0.0"
  }
}
```

## File Structure
```
pos-backend/
├── models/
│   ├── Product.js
│   ├── Category.js
│   ├── Customer.js
│   └── Order.js
├── controllers/
│   ├── productController.js
│   ├── categoryController.js
│   ├── customerController.js
│   └── orderController.js
├── routes/
│   ├── products.js
│   ├── categories.js
│   ├── customers.js
│   └── orders.js
├── config/
│   └── database.js
├── middleware/
│   └── errorHandler.js
├── .env
├── app.js
└── server.js
```

## Assessment Criteria
- **Code Quality**: Clean, readable, and well-commented code
- **Functionality**: All CRUD operations working correctly
- **Validation**: Proper data validation and error handling
- **Relationships**: Correct model relationships and population
- **Testing**: Successful API testing with various scenarios
- **Understanding**: Ability to explain code and concepts

## Homework/Extended Learning
- Practice creating additional models (Suppliers, Employees)
- Implement advanced search and filtering
- Create more detailed reporting features
- Add data validation middleware
- Explore MongoDB aggregation for analytics

## Notes for Instructors
- **Authentication is Day 4** - Don't introduce JWT, login, or user management
- Emphasize **MVC pattern** with controllers separating business logic
- Focus on **practical POS features** that students can relate to
- Use **real-world examples** throughout (inventory management, sales tracking)
- Encourage **testing each feature** immediately after implementation
- **Copy-paste friendly** - All code blocks are complete and ready to use