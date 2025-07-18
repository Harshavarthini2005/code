st { createProxyMiddleware } = require("http-proxy-middleware");

// This proxy redirects requests to /api endpoints to
// the Express server running on port 3001 when we're
// in `NODE_ENV=development` mode.
module.exports = function(app) {
  app.use(
    "/api",
    createProxyMiddleware({
      target: "http://localhost:3001"
    })
  );
};

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  padding: 1rem;
}

.card {
  background: #f2f2f2;
  border-radius: 12px;
  padding: 1rem;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  cursor: pointer;
  transition: transform 0.2s;
}

.card:hover {
  transform: scale(1.02);
}

@media screen and (max-width: 768px) {
  .grid {
    grid-template-columns: 1fr;
  }
}

const express = require("express");
const cors = require("cors");
const path = require("path");
const fs = require("fs");

const app = express();

const NODE_ENV = process.env.NODE_ENV;
let port;

app.use(cors());

// Test route
app.get("/api/ping", (req, res) => {
  console.log("❇️ Received GET request to /api/ping");
  res.send("pong!");
});

// Load movie data
const movies = JSON.parse(fs.readFileSync(path.join(__dirname, "./movies_metadata.json"), "utf-8"));

// List all movies
app.get("/api/movies", (req, res) => {
  console.log("❇️ Received GET request to /api/movies");
  const simplified = movies.map((m) => ({
    id: m.id,
    title: m.title,
    tagline: m.tagline,
    vote_average: parseFloat(m.vote_average),
  }));
  res.json(simplified);
});

// Get movie by ID
app.get("/api/movies/:id", (req, res) => {
  const movie = movies.find((m) => m.id === req.params.id);
  if (movie) {
    res.json(movie);
  } else {
    res.status(404).json({ message: "Movie not found" });
  }
});

// Production: serve frontend
if (NODE_ENV === "production") {
  port = process.env.PORT || 3000;
  app.use(express.static(path.join(__dirname, "../build")));
  app.get("*", (req, res) => {
    res.sendFile(path.join(__dirname, "../build", "index.html"));
  });
} else {
  port = 3001;
  console.log("⚠️ Not seeing your changes as you develop?");
  console.log("⚠️ Do you need to set 'start': 'npm run development' in package.json?");
}

// Start the server
app.listen(port, () => {
  console.log(`❇️ Express server is running on port ${port}`);
});


{
  "name": "starter-cra-and-react",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^16.12.0",
    "react-dom": "^16.12.0",
    "react-scripts": "^3.4.0",
    "express": "^4.17.1",
    "concurrently": "^5.0.2",
    "http-proxy-middleware": "^1.0.0"
  },
  "scripts": {
    "start": "npm run development",
    "development": "NODE_ENV=development concurrently --kill-others \"npm run client\" \"npm run server\"",
    "production": "npm run build && NODE_ENV=production npm run server",
    "server": "node server/server.js",
    "client": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "engines": {
    "node": "10.x"
  },
  "eslintConfig": {
    "extends": "react-app"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "license": "MIT"
}

import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
