import { useState } from "react";

export default function RentalHomeFinder() {
  const [search, setSearch] = useState("");
  const [price, setPrice] = useState("all");
  const [bedrooms, setBedrooms] = useState("all");

  const homes = [
    { id: 1, title: "2BHK Apartment in Bangalore", price: 18000, bedrooms: 2, location: "Bangalore" },
    { id: 2, title: "1BHK Flat in Hyderabad", price: 12000, bedrooms: 1, location: "Hyderabad" },
    { id: 3, title: "3BHK Villa in Chennai", price: 35000, bedrooms: 3, location: "Chennai" },
    { id: 4, title: "2BHK Flat in Pune", price: 22000, bedrooms: 2, location: "Pune" },
  ];

  const filteredHomes = homes.filter((home) => {
    const matchesSearch = home.title.toLowerCase().includes(search.toLowerCase()) ||
                          home.location.toLowerCase().includes(search.toLowerCase());

    const matchesPrice = price === "all" ||
      (price === "low" && home.price < 15000) ||
      (price === "mid" && home.price >= 15000 && home.price <= 25000) ||
      (price === "high" && home.price > 25000);

    const matchesBedrooms = bedrooms === "all" || home.bedrooms === parseInt(bedrooms);

    return matchesSearch && matchesPrice && matchesBedrooms;
  });

  return (
    <div className="p-6 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">🏡 Rental Home Finder</h1>

      {/* Search & Filters */}
      <div className="flex gap-3 mb-5">
        <input
          type="text"
          placeholder="Search by city or title..."
          value={search}
          onChange={(e) => setSearch(e.target.value)}
          className="border rounded-lg p-2 flex-1"
        />
        <select value={price} onChange={(e) => setPrice(e.target.value)} className="border rounded-lg p-2">
          <option value="all">All Prices</option>
          <option value="low">Below ₹15,000</option>
          <option value="mid">₹15,000 - ₹25,000</option>
          <option value="high">Above ₹25,000</option>
        </select>
        <select value={bedrooms} onChange={(e) => setBedrooms(e.target.value)} className="border rounded-lg p-2">
          <option value="all">Any Bedrooms</option>
          <option value="1">1 BHK</option>
          <option value="2">2 BHK</option>
          <option value="3">3 BHK</option>
        </select>
      </div>

      {/* Homes List */}
      <div className="grid gap-4">
        {filteredHomes.length > 0 ? (
          filteredHomes.map((home) => (
            <div key={home.id} className="border rounded-xl p-4 shadow-sm hover:shadow-lg transition">
              <h2 className="font-semibold text-lg">{home.title}</h2>
              <p className="text-gray-600">📍 {home.location}</p>
              <p className="text-gray-800 font-medium">💰 ₹{home.price}</p>
              <p className="text-gray-600">🛏 {home.bedrooms} BHK</p>
            </div>
          ))
        ) : (
          <p className="text-gray-500">No homes found matching your criteria.</p>
        )}
      </div>
    </div>
  );
}
