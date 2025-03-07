# MERN Stack CRUD Application (Full Project with Notes)

## Project Structure
```
/mern-crud
 ├── /client (React Frontend)
 ├── /server (Node.js + Express + MongoDB Backend)
 │   ├── /config
 │   │   ├── db.js
 │   ├── /controllers
 │   │   ├── itemController.js
 │   ├── /models
 │   │   ├── Item.js
 │   ├── /routes
 │   │   ├── itemRoutes.js
 │   ├── /middlewares
 │   │   ├── errorMiddleware.js
 │   ├── /server.js
 ├── package.json
 ├── README.md
```

---

# Backend (Node.js + Express + MongoDB)

## 1. Install Dependencies
Run inside `/server` folder:
```bash
npm init -y
npm install express mongoose cors dotenv nodemon
```

## 2. Database Connection (`config/db.js`)
```javascript
import mongoose from "mongoose";
import dotenv from "dotenv";

dotenv.config();

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log("MongoDB Connected");
  } catch (error) {
    console.error("Database connection failed", error);
    process.exit(1);
  }
};

export default connectDB;
```

## 3. Define Data Model (`models/Item.js`)
```javascript
import mongoose from "mongoose";

const itemSchema = mongoose.Schema({
  name: { type: String, required: true },
}, { timestamps: true });

const Item = mongoose.model("Item", itemSchema);
export default Item;
```

## 4. Controller Logic (`controllers/itemController.js`)
```javascript
import Item from "../models/Item.js";

export const getItems = async (req, res) => {
  const items = await Item.find();
  res.json(items);
};

export const createItem = async (req, res) => {
  const { name } = req.body;
  if (!name) return res.status(400).json({ message: "Name is required" });

  const item = new Item({ name });
  const savedItem = await item.save();
  res.status(201).json(savedItem);
};

export const updateItem = async (req, res) => {
  const { name } = req.body;
  const item = await Item.findById(req.params.id);

  if (!item) return res.status(404).json({ message: "Item not found" });

  item.name = name;
  const updatedItem = await item.save();
  res.json(updatedItem);
};

export const deleteItem = async (req, res) => {
  const item = await Item.findById(req.params.id);
  if (!item) return res.status(404).json({ message: "Item not found" });

  await item.deleteOne();
  res.json({ message: "Item deleted" });
};
```

## 5. Define Routes (`routes/itemRoutes.js`)
```javascript
import express from "express";
import { getItems, createItem, updateItem, deleteItem } from "../controllers/itemController.js";

const router = express.Router();

router.get("/", getItems);
router.post("/", createItem);
router.put("/:id", updateItem);
router.delete("/:id", deleteItem);

export default router;
```

## 6. Middleware for Error Handling (`middlewares/errorMiddleware.js`)
```javascript
const errorHandler = (err, req, res, next) => {
  const statusCode = res.statusCode === 200 ? 500 : res.statusCode;
  res.status(statusCode).json({ message: err.message });
};

export default errorHandler;
```

## 7. Setup Express Server (`server.js`)
```javascript
import express from "express";
import dotenv from "dotenv";
import cors from "cors";
import connectDB from "./config/db.js";
import itemRoutes from "./routes/itemRoutes.js";
import errorHandler from "./middlewares/errorMiddleware.js";

dotenv.config();
connectDB();

const app = express();
app.use(cors());
app.use(express.json());

app.use("/api/items", itemRoutes);

app.use(errorHandler);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

---

# Frontend (React)

## 1. Install Dependencies
Run inside `/client` folder:
```bash
npx create-react-app .
npm install axios react-router-dom
```

## 2. Create API Service (`client/services/itemService.js`)
```javascript
import axios from "axios";

const API_URL = "http://localhost:5000/api/items";

export const fetchItems = async () => {
  const res = await axios.get(API_URL);
  return res.data;
};

export const createItem = async (name) => {
  await axios.post(API_URL, { name });
};

export const updateItem = async (id, name) => {
  await axios.put(`${API_URL}/${id}`, { name });
};

export const deleteItem = async (id) => {
  await axios.delete(`${API_URL}/${id}`);
};
```

## 3. Create React Components

### `client/src/components/ItemList.js`
```javascript
import { useEffect, useState } from "react";
import { fetchItems, createItem, updateItem, deleteItem } from "../services/itemService";

function ItemList() {
  const [items, setItems] = useState([]);
  const [name, setName] = useState("");

  useEffect(() => {
    loadItems();
  }, []);

  const loadItems = async () => {
    const data = await fetchItems();
    setItems(data);
  };

  const handleAdd = async () => {
    await createItem(name);
    setName("");
    loadItems();
  };

  const handleDelete = async (id) => {
    await deleteItem(id);
    loadItems();
  };

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={handleAdd}>Add</button>
      <ul>
        {items.map((item) => (
          <li key={item._id}>{item.name} <button onClick={() => handleDelete(item._id)}>Delete</button></li>
        ))}
      </ul>
    </div>
  );
}

export default ItemList;
```

## 4. Run Application

### Start Backend
```bash
cd server
npm run dev
```

### Start Frontend
```bash
cd client
npm start
```

---

This is a complete **MERN CRUD app** with **MVC architecture**. 🚀

