// server.js
import express from "express";
import cors from "cors";

const app = express();
const PORT = 5000;

// Middleware
app.use(cors());
app.use(express.json());

// Dummy rental homes data (can replace with DB later)
let homes = [
  { id: 1, title: "2BHK Apartment in Bangalore", price: 18000, bedrooms: 2, location: "Bangalore" },
  { id: 2, title: "1BHK Flat in Hyderabad", price: 12000, bedrooms: 1, location: "Hyderabad" },
  { id: 3, title: "3BHK Villa in Chennai", price: 35000, bedrooms: 3, location: "Chennai" },
  { id: 4, title: "2BHK Flat in Pune", price: 22000, bedrooms: 2, location: "Pune" },
];

// API Route: Get all homes with filters
app.get("/api/homes", (req, res) => {
  const { search, price, bedrooms } = req.query;

  let filtered = homes;

  if (search) {
    filtered = filtered.filter(
      (home) =>
        home.title.toLowerCase().includes(search.toLowerCase()) ||
        home.location.toLowerCase().includes(search.toLowerCase())
    );
  }

  if (price) {
    if (price === "low") {
      filtered = filtered.filter((home) => home.price < 15000);
    } else if (price === "mid") {
      filtered = filtered.filter((home) => home.price >= 15000 && home.price <= 25000);
    } else if (price === "high") {
      filtered = filtered.filter((home) => home.price > 25000);
    }
  }

  if (bedrooms && bedrooms !== "all") {
    filtered = filtered.filter((home) => home.bedrooms === parseInt(bedrooms));
  }

  res.json(filtered);
});

// API Route: Add new home (POST request)
app.post("/api/homes", (req, res) => {
  const { title, price, bedrooms, location } = req.body;
  const newHome = {
    id: homes.length + 1,
    title,
    price,
    bedrooms,
    location,
  };
  homes.push(newHome);
  res.status(201).json(newHome);
});

app.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
});
