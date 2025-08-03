 Rental Home Finder Project
 Features
Users can search for rental homes by location, price, number of rooms, property type.

Users can view property details with images.

Admin/owners can add, update, and delete properties.

Secure login/signup for users and admins.

REST API for handling property and user data.

Database to store all listings and users.

🛠 Tech Stack
Frontend: React + Tailwind CSS

Backend: Node.js + Express

Database: MongoDB (Mongoose ORM)

Authentication: JWT (JSON Web Token)

🔹 Backend (Node.js + Express + MongoDB)
1. Setup
bash
Copy
Edit
mkdir rental-home-finder
cd rental-home-finder
npm init -y
npm install express mongoose bcryptjs jsonwebtoken cors dotenv
2. server.js
javascript
Copy
Edit
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
require("dotenv").config();

const app = express();
app.use(express.json());
app.use(cors());

// Routes
const userRoutes = require("./routes/userRoutes");
const propertyRoutes = require("./routes/propertyRoutes");

app.use("/api/users", userRoutes);
app.use("/api/properties", propertyRoutes);

// MongoDB Connection
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => app.listen(5000, () => console.log("Server running on port 5000")))
.catch(err => console.log(err));
3. Models
models/User.js
javascript
Copy
Edit
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  isAdmin: { type: Boolean, default: false }
});

module.exports = mongoose.model("User", userSchema);
models/Property.js
javascript
Copy
Edit
const mongoose = require("mongoose");

const propertySchema = new mongoose.Schema({
  title: { type: String, required: true },
  location: { type: String, required: true },
  price: { type: Number, required: true },
  bedrooms: { type: Number, required: true },
  propertyType: { type: String, enum: ["Apartment", "House", "Villa"], required: true },
  description: { type: String },
  image: { type: String } // URL to image
}, { timestamps: true });

module.exports = mongoose.model("Property", propertySchema);
4. Controllers
controllers/userController.js
javascript
Copy
Edit
const User = require("../models/User");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");

exports.register = async (req, res) => {
  try {
    const { name, email, password } = req.body;
    const existingUser = await User.findOne({ email });
    if (existingUser) return res.status(400).json({ message: "User already exists" });

    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({ name, email, password: hashedPassword });
    await user.save();

    res.status(201).json({ message: "User registered successfully" });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    if (!user) return res.status(400).json({ message: "User not found" });

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) return res.status(400).json({ message: "Invalid credentials" });

    const token = jwt.sign({ id: user._id, isAdmin: user.isAdmin }, process.env.JWT_SECRET, { expiresIn: "1d" });
    res.json({ token, user: { id: user._id, name: user.name, isAdmin: user.isAdmin } });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};
controllers/propertyController.js
javascript
Copy
Edit
const Property = require("../models/Property");

exports.addProperty = async (req, res) => {
  try {
    const property = new Property(req.body);
    await property.save();
    res.status(201).json(property);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};

exports.getProperties = async (req, res) => {
  try {
    const { location, minPrice, maxPrice, bedrooms, propertyType } = req.query;
    let filter = {};
    if (location) filter.location = { $regex: location, $options: "i" };
    if (minPrice) filter.price = { ...filter.price, $gte: minPrice };
    if (maxPrice) filter.price = { ...filter.price, $lte: maxPrice };
    if (bedrooms) filter.bedrooms = bedrooms;
    if (propertyType) filter.propertyType = propertyType;

    const properties = await Property.find(filter);
    res.json(properties);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};
5. Routes
routes/userRoutes.js
javascript
Copy
Edit
const express = require("express");
const { register, login } = require("../controllers/userController");
const router = express.Router();

router.post("/register", register);
router.post("/login", login);

module.exports = router;
routes/propertyRoutes.js
javascript
Copy
Edit
const express = require("express");
const { addProperty, getProperties } = require("../controllers/propertyController");
const router = express.Router();

router.post("/add", addProperty);
router.get("/", getProperties);

module.exports = router;
🔹 Frontend (React + Tailwind)
1. Setup
bash
Copy
Edit
npx create-react-app rental-frontend
cd rental-frontend
npm install axios react-router-dom tailwindcss
npx tailwindcss init
Add Tailwind config in index.css.

2. Pages
App.js
javascript
Copy
Edit
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Home from "./pages/Home";
import Login from "./pages/Login";
import Register from "./pages/Register";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
pages/Home.js
javascript
Copy
Edit
import { useEffect, useState } from "react";
import axios from "axios";

export default function Home() {
  const [properties, setProperties] = useState([]);
  const [filters, setFilters] = useState({ location: "", minPrice: "", maxPrice: "", bedrooms: "" });

  useEffect(() => {
    fetchProperties();
  }, []);

  const fetchProperties = async () => {
    const res = await axios.get("http://localhost:5000/api/properties", { params: filters });
    setProperties(res.data);
  };

  return (
    <div className="p-6">
      <h1 className="text-3xl font-bold mb-4">🏡 Rental Home Finder</h1>

      {/* Filter Form */}
      <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
        <input placeholder="Location" className="border p-2" onChange={e => setFilters({ ...filters, location: e.target.value })} />
        <input type="number" placeholder="Min Price" className="border p-2" onChange={e => setFilters({ ...filters, minPrice: e.target.value })} />
        <input type="number" placeholder="Max Price" className="border p-2" onChange={e => setFilters({ ...filters, maxPrice: e.target.value })} />
        <input type="number" placeholder="Bedrooms" className="border p-2" onChange={e => setFilters({ ...filters, bedrooms: e.target.value })} />
      </div>
      <button onClick={fetchProperties} className="bg-blue-500 text-white px-4 py-2 rounded">Search</button>

      {/* Property List */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mt-6">
        {properties.map(p => (
          <div key={p._id} className="border p-4 rounded shadow">
            <img src={p.image || "https://via.placeholder.com/150"} alt={p.title} className="w-full h-40 object-cover rounded" />
            <h2 className="text-xl font-semibold mt-2">{p.title}</h2>
            <p>{p.location}</p>
            <p>₹{p.price} | {p.bedrooms} BHK</p>
            <p className="text-gray-500">{p.propertyType}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
