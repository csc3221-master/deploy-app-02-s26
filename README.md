# Transactions 2 API: Need to do myself

## Node + Express + MongoDB + Models + Controllers + Routes + Heroku

This guide creates a second version of the Transactions API.

The previous project, `deploy-app-01-s26`, built a working Transactions API in one main `server.js` file. In this version, we will keep the same general idea but reorganize the code into a more professional back-end structure:

- `models/`
- `controllers/`
- `routes/`
- `config/`

This is often called an MVC-style structure.

In this project:

- **Model** = knows how to talk to the database
- **Controller** = contains the request/response logic
- **Route** = connects a URL to a controller function

---

# Project Goal

Create a back-end Transactions API using:

- Node.js
- Express
- MongoDB
- Docker
- VS Code Dev Containers
- MongoDB Atlas
- Heroku

The API stores credit card transactions.

Transactions are append-only:

- Transactions are created.
- Transactions can be read.
- Transactions are not edited.
- Transactions are not deleted.
- Corrections are made by adding amendment transactions.

---

# Data Model

A transaction will look like this:

```json
{
  "_id": "mongodb-generated-id",
  "creditCardNickname": "Costco Visa",
  "cardType": "Visa",
  "date": "2026-05-12T00:00:00.000Z",
  "amount": 42.75,
  "amendment": false,
  "comment": "Gas",
  "createdAt": "2026-05-17T00:00:00.000Z"
}
```

Supported card types:

```txt
Visa
Master
Amex
Discover
Other
```

---

# API Endpoints

## Basic route

```txt
GET /
```

Returns a simple message showing that the API is running.

## Transaction routes

```txt
POST /api/transactions
GET /api/transactions
GET /api/transactions/:id
```

The `GET /api/transactions` route supports optional query parameters:

```txt
/api/transactions?date=2026-05-12
/api/transactions?startDate=2026-05-01&endDate=2026-05-31
/api/transactions?creditCardNickname=Costco%20Visa
```

This API intentionally does not include:

```txt
PUT
PATCH
DELETE
```

Financial records should not be destructively modified.

---

# Phase 0 — Requirements

You should already have:

- Visual Studio Code
- Docker Desktop
- VS Code Dev Containers extension
- Git
- GitHub account
- MongoDB Atlas account
- Heroku account
- Heroku CLI installed on your computer

Important:

- Docker and VS Code will be used for development.
- Heroku CLI commands should usually be run from your computer terminal, not from inside the container.
- The local database will use MongoDB inside Docker.
- The deployed version will use MongoDB Atlas.

---

# Phase 1 — Create the Repository

Create a new GitHub repository.

Suggested name:

```txt
transactions2-api
```

Clone the repository to your computer:

```bash
git clone https://github.com/YOUR-USERNAME/transactions2-api.git
cd transactions2-api
```

---

# Phase 2 — Create the Project Structure

Create the following folders and files:

```txt
transactions2-api/
├── .devcontainer/
│   └── devcontainer.json
├── config/
│   └── database.js
├── controllers/
│   └── transactionController.js
├── models/
│   └── transactionModel.js
├── routes/
│   └── transactionRoutes.js
├── .env
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── package.json
├── Procfile
├── seed.js
├── server.js
└── README.md
```

You can create folders from the terminal:

```bash
mkdir .devcontainer config controllers models routes
touch .env .gitignore Dockerfile docker-compose.yml package.json Procfile seed.js server.js README.md
touch .devcontainer/devcontainer.json
touch config/database.js
touch controllers/transactionController.js
touch models/transactionModel.js
touch routes/transactionRoutes.js
```

---

# Phase 3 — Create `.gitignore`

Create `.gitignore`:

```gitignore
node_modules
.env
.DS_Store
npm-debug.log
```

Important:

Never commit `.env` to GitHub.

---

# Phase 4 — Create `.env`

Create `.env`:

```env
PORT=3000
MONGODB_URI=mongodb://db:27017/transactions2db
```

This connection string works inside the Docker container because the MongoDB service in `docker-compose.yml` will be named `db`.

---

# Phase 5 — Create `docker-compose.yml`

Create `docker-compose.yml`:

```yaml
services:
  app:
    build: .
    working_dir: /app
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - db
    command: sleep infinity

  db:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

Explanation:

- `app` is the Node/Express container.
- `db` is the MongoDB container.
- The database data is stored in a Docker volume named `mongo-data`.
- Port `3000` is forwarded so the browser can reach the Express app.
- Port `27017` is forwarded so tools can connect to MongoDB.

---

# Phase 6 — Create `Dockerfile`

Create `Dockerfile`:

```dockerfile
FROM node:20-bookworm

RUN apt-get update && \
    apt-get install -y wget gnupg && \
    wget -qO - https://pgp.mongodb.com/server-7.0.asc | \
    gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg && \
    echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/7.0 main" \
    > /etc/apt/sources.list.d/mongodb-org-7.0.list && \
    apt-get update && \
    apt-get install -y mongodb-mongosh && \
    apt-get clean

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

This installs Node and also installs `mongosh` so you can connect to MongoDB from inside the container.

---

# Phase 7 — Create `.devcontainer/devcontainer.json`

Create `.devcontainer/devcontainer.json`:

```json
{
  "name": "Transactions 2 API",
  "dockerComposeFile": "../docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/app",
  "shutdownAction": "stopCompose",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-azuretools.vscode-docker",
        "mongodb.mongodb-vscode",
        "dbaeumer.vscode-eslint"
      ]
    }
  },
  "forwardPorts": [3000, 27017],
  "remoteUser": "root"
}
```

Then reopen the project in the container:

1. Open the project folder in VS Code.
2. Press `Cmd + Shift + P` on Mac or `Ctrl + Shift + P` on Windows.
3. Choose `Dev Containers: Reopen in Container`.

---

# Phase 8 — Create `package.json`

Create `package.json`:

```json
{
  "name": "transactions2-api",
  "version": "1.0.0",
  "description": "Transactions API using Express, MongoDB, models, controllers, and routes",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon --legacy-watch server.js",
    "seed": "node seed.js"
  },
  "dependencies": {
    "dotenv": "^16.4.7",
    "express": "^4.18.3",
    "mongodb": "^6.8.0"
  },
  "devDependencies": {
    "nodemon": "^3.1.4"
  }
}
```

Then install dependencies inside the container terminal:

```bash
npm install
```

---

# Phase 9 — Create `config/database.js`

Create `config/database.js`:

```js
const { MongoClient } = require("mongodb");

let client;
let db;

async function connectToDatabase() {
  if (db) {
    return db;
  }

  const uri = process.env.MONGODB_URI;

  if (!uri) {
    throw new Error("Missing MONGODB_URI environment variable.");
  }

  client = new MongoClient(uri);

  await client.connect();

  db = client.db();

  console.log("Connected to MongoDB");

  return db;
}

function getDatabase() {
  if (!db) {
    throw new Error("Database has not been initialized.");
  }

  return db;
}

async function closeDatabaseConnection() {
  if (client) {
    await client.close();
    client = null;
    db = null;
  }
}

module.exports = {
  connectToDatabase,
  getDatabase,
  closeDatabaseConnection
};
```

This file is responsible for connecting to MongoDB.

Notice that database connection code is no longer inside `server.js`.

---

# Phase 10 — Create `models/transactionModel.js`

Create `models/transactionModel.js`:

```js
const { ObjectId } = require("mongodb");
const { getDatabase } = require("../config/database");

const COLLECTION_NAME = "transactions";

const validCardTypes = ["Visa", "Master", "Amex", "Discover", "Other"];

function getCollection() {
  const db = getDatabase();
  return db.collection(COLLECTION_NAME);
}

function validateTransaction(body) {
  if (!body.creditCardNickname) {
    return "creditCardNickname is required.";
  }

  if (!body.cardType) {
    return "cardType is required.";
  }

  if (!validCardTypes.includes(body.cardType)) {
    return "Invalid card type.";
  }

  if (!body.date) {
    return "date is required.";
  }

  if (Number.isNaN(Date.parse(body.date))) {
    return "Invalid date.";
  }

  if (body.amount === undefined) {
    return "amount is required.";
  }

  if (typeof body.amount !== "number") {
    return "amount must be numeric.";
  }

  return null;
}

function buildTransaction(body) {
  return {
    creditCardNickname: body.creditCardNickname,
    cardType: body.cardType,
    date: new Date(body.date),
    amount: body.amount,
    amendment: body.amendment === true,
    comment: body.comment || null,
    createdAt: new Date()
  };
}

async function createTransaction(body) {
  const validationError = validateTransaction(body);

  if (validationError) {
    const error = new Error(validationError);
    error.statusCode = 400;
    throw error;
  }

  const transaction = buildTransaction(body);

  const result = await getCollection().insertOne(transaction);

  return {
    ...transaction,
    _id: result.insertedId
  };
}

async function findTransactions(query) {
  const filter = {};

  const { date, startDate, endDate, creditCardNickname } = query;

  if (creditCardNickname) {
    filter.creditCardNickname = creditCardNickname;
  }

  if (date) {
    const start = new Date(date);
    const end = new Date(date);

    end.setDate(end.getDate() + 1);

    filter.date = {
      $gte: start,
      $lt: end
    };
  }

  if (startDate || endDate) {
    filter.date = {};

    if (startDate) {
      filter.date.$gte = new Date(startDate);
    }

    if (endDate) {
      const end = new Date(endDate);
      end.setDate(end.getDate() + 1);
      filter.date.$lt = end;
    }
  }

  return getCollection()
    .find(filter)
    .sort({ date: -1 })
    .toArray();
}

async function findTransactionById(id) {
  if (!ObjectId.isValid(id)) {
    const error = new Error("Invalid id.");
    error.statusCode = 400;
    throw error;
  }

  const transaction = await getCollection().findOne({
    _id: new ObjectId(id)
  });

  return transaction;
}

module.exports = {
  validCardTypes,
  createTransaction,
  findTransactions,
  findTransactionById
};
```

This file is the model.

The model knows:

- the collection name
- how to validate a transaction
- how to build a transaction object
- how to insert a transaction
- how to query transactions
- how to find one transaction by id

---

# Phase 11 — Create `controllers/transactionController.js`

Create `controllers/transactionController.js`:

```js
const Transaction = require("../models/transactionModel");

async function createTransaction(req, res, next) {
  try {
    const transaction = await Transaction.createTransaction(req.body);
    res.status(201).json(transaction);
  } catch (error) {
    next(error);
  }
}

async function getTransactions(req, res, next) {
  try {
    const transactions = await Transaction.findTransactions(req.query);
    res.json(transactions);
  } catch (error) {
    next(error);
  }
}

async function getTransactionById(req, res, next) {
  try {
    const transaction = await Transaction.findTransactionById(req.params.id);

    if (!transaction) {
      return res.status(404).json({
        error: "Transaction not found."
      });
    }

    res.json(transaction);
  } catch (error) {
    next(error);
  }
}

module.exports = {
  createTransaction,
  getTransactions,
  getTransactionById
};
```

This file is the controller.

The controller knows:

- how to read `req.body`
- how to read `req.query`
- how to read `req.params`
- how to send JSON responses
- how to send HTTP status codes

The controller does not directly talk to MongoDB. It calls the model.

---

# Phase 12 — Create `routes/transactionRoutes.js`

Create `routes/transactionRoutes.js`:

```js
const express = require("express");
const transactionController = require("../controllers/transactionController");

const router = express.Router();

router.post("/", transactionController.createTransaction);
router.get("/", transactionController.getTransactions);
router.get("/:id", transactionController.getTransactionById);

module.exports = router;
```

This file is the route file.

The route file connects URLs to controller functions.

Because this router will be mounted at `/api/transactions`, these routes become:

```txt
POST /api/transactions
GET /api/transactions
GET /api/transactions/:id
```

---

# Phase 13 — Create `server.js`

Create `server.js`:

```js
require("dotenv").config();

const express = require("express");
const { connectToDatabase } = require("./config/database");
const transactionRoutes = require("./routes/transactionRoutes");

const app = express();

const PORT = process.env.PORT || 3000;

app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    message: "Transactions 2 API is running",
    routes: {
      transactions: "/api/transactions"
    }
  });
});

app.use("/api/transactions", transactionRoutes);

app.use((req, res) => {
  res.status(404).json({
    error: "Route not found."
  });
});

app.use((error, req, res, next) => {
  console.error(error);

  const statusCode = error.statusCode || 500;

  res.status(statusCode).json({
    error: error.message || "Internal server error."
  });
});

async function startServer() {
  await connectToDatabase();

  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
}

startServer().catch((error) => {
  console.error(error);
  process.exit(1);
});
```

This version of `server.js` is much cleaner than the original one-file app.

Its job is only to:

- load environment variables
- create the Express app
- use JSON middleware
- mount routes
- handle errors
- connect to the database
- start the server

---

# Phase 14 — Create `seed.js`

Create `seed.js`:

```js
require("dotenv").config();

const { MongoClient } = require("mongodb");

const MONGODB_URI = process.env.MONGODB_URI;

if (!MONGODB_URI) {
  console.error("Missing MONGODB_URI in .env");
  process.exit(1);
}

const cardNicknames = [
  "Costco Visa",
  "Amazon Visa",
  "Travel Master",
  "Blue Amex",
  "Discover Cashback",
  "Everyday Card"
];

const cardTypes = [
  "Visa",
  "Master",
  "Amex",
  "Discover",
  "Other"
];

const comments = [
  "Gas",
  "Groceries",
  "Restaurant",
  "Coffee",
  "Flight",
  "Hotel",
  "Books",
  "Online Purchase",
  "Utilities",
  "Streaming Service",
  "Pharmacy",
  "Electronics",
  "School Supplies",
  "Parking",
  "Car Maintenance"
];

function randomElement(array) {
  return array[Math.floor(Math.random() * array.length)];
}

function randomAmount(min, max) {
  return Number((Math.random() * (max - min) + min).toFixed(2));
}

function randomDateWithinLast90Days() {
  const now = new Date();
  const daysAgo = Math.floor(Math.random() * 90);
  const date = new Date(now);

  date.setDate(now.getDate() - daysAgo);

  return date;
}

function generateTransaction() {
  const amendment = Math.random() < 0.1;

  const amount = amendment
    ? -randomAmount(5, 250)
    : randomAmount(5, 250);

  return {
    creditCardNickname: randomElement(cardNicknames),
    cardType: randomElement(cardTypes),
    date: randomDateWithinLast90Days(),
    amount,
    amendment,
    comment: amendment ? "Amendment Transaction" : randomElement(comments),
    createdAt: new Date()
  };
}

async function seed() {
  const client = new MongoClient(MONGODB_URI);

  try {
    await client.connect();

    console.log("Connected to MongoDB");

    const db = client.db();
    const collection = db.collection("transactions");

    await collection.deleteMany({});

    const transactions = [];

    for (let i = 0; i < 100; i++) {
      transactions.push(generateTransaction());
    }

    const result = await collection.insertMany(transactions);

    console.log(`Inserted ${result.insertedCount} transactions`);
  } catch (error) {
    console.error(error);
  } finally {
    await client.close();
    console.log("Disconnected from MongoDB");
  }
}

seed();
```

This script deletes existing transactions and inserts 100 new sample transactions.

Run it with:

```bash
npm run seed
```

---

# Phase 15 — Create `Procfile`

Create `Procfile`:

```txt
web: node server.js
```

Heroku uses this file to know how to start the application.

---

# Phase 16 — Start the App Locally

Inside the VS Code container terminal, run:

```bash
npm install
npm run dev
```

You should see something like:

```txt
Connected to MongoDB
Server running on port 3000
```

Open the browser:

```txt
http://localhost:3000
```

Expected result:

```json
{
  "message": "Transactions 2 API is running",
  "routes": {
    "transactions": "/api/transactions"
  }
}
```

---

# Phase 17 — Seed the Database

Open a second container terminal.

Run:

```bash
npm run seed
```

Expected result:

```txt
Connected to MongoDB
Inserted 100 transactions
Disconnected from MongoDB
```

---

# Phase 18 — Test with `curl`

## Get all transactions

```bash
curl http://localhost:3000/api/transactions
```

## Create a transaction

```bash
curl -X POST http://localhost:3000/api/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "creditCardNickname": "Costco Visa",
    "cardType": "Visa",
    "date": "2026-05-12",
    "amount": 42.75,
    "comment": "Gas"
  }'
```

## Get transactions by date

```bash
curl "http://localhost:3000/api/transactions?date=2026-05-12"
```

## Get transactions by date range

```bash
curl "http://localhost:3000/api/transactions?startDate=2026-05-01&endDate=2026-05-31"
```

## Get transactions by card nickname

```bash
curl "http://localhost:3000/api/transactions?creditCardNickname=Costco%20Visa"
```

## Get one transaction by id

First get all transactions:

```bash
curl http://localhost:3000/api/transactions
```

Copy one `_id`, then run:

```bash
curl http://localhost:3000/api/transactions/PASTE_ID_HERE
```

---

# Phase 19 — Test with Postman

Create a new Postman collection called:

```txt
Transactions 2 API
```

Add these requests:

```txt
GET http://localhost:3000/
GET http://localhost:3000/api/transactions
GET http://localhost:3000/api/transactions?creditCardNickname=Costco%20Visa
POST http://localhost:3000/api/transactions
```

For the `POST` request:

1. Select `Body`.
2. Select `raw`.
3. Select `JSON`.
4. Paste this body:

```json
{
  "creditCardNickname": "Costco Visa",
  "cardType": "Visa",
  "date": "2026-05-12",
  "amount": 42.75,
  "comment": "Gas"
}
```

Then click `Send`.

You should receive a `201 Created` response.

---

# Phase 20 — Use Postman Variables

Postman can store a value and reuse it.

Create a Postman environment variable:

```txt
base_url
```

Set the initial value to:

```txt
http://localhost:3000
```

Now you can write requests like:

```txt
{{base_url}}/api/transactions
```

Later, when the app is deployed to Heroku, you only need to change `base_url`.

Example deployed value:

```txt
https://YOUR-HEROKU-APP-NAME.herokuapp.com
```

---

# Phase 21 — Copy an ID in Postman

To test `GET /api/transactions/:id`:

1. Send:

```txt
GET {{base_url}}/api/transactions
```

2. In the JSON response, find a transaction.
3. Copy the value of `_id`.

Example:

```json
{
  "_id": "6654d6b0d4c15c882b3e57cc",
  "creditCardNickname": "Costco Visa"
}
```

4. Create a new request:

```txt
GET {{base_url}}/api/transactions/6654d6b0d4c15c882b3e57cc
```

5. Send the request.

A good habit is to create another Postman variable:

```txt
transaction_id
```

Then use:

```txt
{{base_url}}/api/transactions/{{transaction_id}}
```

---

# Phase 22 — Check MongoDB with `mongosh`

From inside the container terminal:

```bash
mongosh "mongodb://db:27017/transactions2db"
```

Inside `mongosh`, run:

```js
show collections
db.transactions.find().limit(5)
db.transactions.countDocuments()
```

Exit:

```js
exit
```

---

# Phase 23 — Understand the MVC Flow

When a request comes in:

```txt
GET /api/transactions
```

The flow is:

```txt
server.js
  ↓
routes/transactionRoutes.js
  ↓
controllers/transactionController.js
  ↓
models/transactionModel.js
  ↓
MongoDB
```

For example:

```txt
GET /api/transactions
```

is matched in:

```js
router.get("/", transactionController.getTransactions);
```

Then the controller calls:

```js
Transaction.findTransactions(req.query);
```

Then the model calls MongoDB:

```js
getCollection().find(filter).sort({ date: -1 }).toArray();
```

This separation helps keep the code easier to understand and easier to maintain.

---

# Phase 24 — Prepare MongoDB Atlas

For deployment, the local Docker MongoDB database will not be used.

The deployed Heroku app needs a cloud database.

Use MongoDB Atlas.

General steps:

1. Log in to MongoDB Atlas.
2. Create a cluster.
3. Create a database user.
4. Allow network access.
5. Copy the connection string.
6. Replace `<username>` and `<password>` with your database user credentials.
7. Make sure the database name is included at the end.

Example:

```txt
mongodb+srv://USERNAME:PASSWORD@cluster0.xxxxx.mongodb.net/transactions2db
```

Important:

If your password has special characters, Atlas may encode them for you. Copy the connection string carefully.

---

# Phase 25 — Test Atlas Locally Before Heroku

Before deploying to Heroku, test your app locally with Atlas.

In `.env`, temporarily replace:

```env
MONGODB_URI=mongodb://db:27017/transactions2db
```

with your Atlas connection string:

```env
MONGODB_URI=mongodb+srv://USERNAME:PASSWORD@cluster0.xxxxx.mongodb.net/transactions2db
```

Then restart the server:

```bash
npm run dev
```

Seed Atlas:

```bash
npm run seed
```

Test:

```bash
curl http://localhost:3000/api/transactions
```

If this works locally, Heroku is more likely to work.

Before continuing, commit your code.

Do not commit `.env`.

---

# Phase 26 — Commit to Git

Run:

```bash
git status
git add .
git commit -m "Create transactions2 API with models controllers and routes"
```

Push to GitHub:

```bash
git push origin main
```

---

# Phase 27 — Deploy to Heroku

Run these commands from your computer terminal, not from inside the container.

Login:

```bash
heroku login
```

Create the app:

```bash
heroku create YOUR-APP-NAME
```

Example:

```bash
heroku create transactions2-carlos
```

Add the Heroku remote if needed:

```bash
heroku git:remote -a YOUR-APP-NAME
```

Set the MongoDB Atlas connection string:

```bash
heroku config:set MONGODB_URI="mongodb+srv://USERNAME:PASSWORD@cluster0.xxxxx.mongodb.net/transactions2db"
```

Deploy:

```bash
git push heroku main
```

Open the app:

```bash
heroku open
```

View logs:

```bash
heroku logs --tail
```

---

# Phase 28 — Seed the Heroku/Atlas Database

The Heroku app uses Atlas.

You can seed the Atlas database from Heroku by running:

```bash
heroku run npm run seed
```

Then test the deployed API:

```bash
curl https://YOUR-APP-NAME.herokuapp.com/api/transactions
```

Or update your Postman `base_url` variable to:

```txt
https://YOUR-APP-NAME.herokuapp.com
```

Then send:

```txt
GET {{base_url}}/api/transactions
```

---

# Phase 29 — Common Problems

## Problem: `Missing MONGODB_URI environment variable`

Check local `.env`:

```bash
cat .env
```

Check Heroku config:

```bash
heroku config
```

You should see `MONGODB_URI`.

---

## Problem: `getaddrinfo ENOTFOUND db`

This usually means you are trying to use:

```txt
mongodb://db:27017/transactions2db
```

outside the Docker Compose network.

That connection string only works inside the container.

Use this locally inside Docker:

```env
MONGODB_URI=mongodb://db:27017/transactions2db
```

Use Atlas for Heroku:

```env
MONGODB_URI=mongodb+srv://USERNAME:PASSWORD@cluster0.xxxxx.mongodb.net/transactions2db
```

---

## Problem: Heroku app crashes

Run:

```bash
heroku logs --tail
```

Look for:

- missing environment variables
- MongoDB connection problems
- syntax errors
- wrong `Procfile`
- wrong start script

---

## Problem: POST request gives `amount must be numeric`

Make sure the JSON body uses a number, not a string.

Correct:

```json
{
  "amount": 42.75
}
```

Incorrect:

```json
{
  "amount": "42.75"
}
```

---

## Problem: Postman request body is not working

Check:

1. Body is set to `raw`.
2. Body type is set to `JSON`.
3. Header includes:

```txt
Content-Type: application/json
```

Postman usually adds this automatically when you select JSON.

---

# Phase 30 — Suggested Git Tags

You can create tags after each major phase:

```bash
git tag phase-0-setup
git tag phase-1-mvc-structure
git tag phase-2-local-db
git tag phase-3-routes-working
git tag phase-4-atlas
git tag phase-5-heroku
git push origin --tags
```

Suggested meanings:

```txt
phase-0-setup        Docker, Dev Container, package setup
phase-1-mvc-structure Models, controllers, routes folders
phase-2-local-db     Local MongoDB working
phase-3-routes-working API routes tested with curl/Postman
phase-4-atlas        Atlas connection tested
phase-5-heroku       Heroku deployment working
```

---

# Phase 31 — Student Reflection Questions

Answer these in your `README.md`:

1. What problem does the model file solve?
2. What problem does the controller file solve?
3. What problem does the route file solve?
4. How is this project different from the previous Transactions API?
5. Why should financial transactions avoid destructive edits?
6. What is the purpose of the `seed.js` file?
7. What is the difference between the local MongoDB URI and the Atlas MongoDB URI?
8. What was the most difficult part of this assignment?
9. Include the URL of your deployed Heroku application.
10. Include screenshots of successful Postman requests.

---

# Complete Source Code Summary

Below is the complete list of files that contain source/configuration code.

---

## `.env`

```env
PORT=3000
MONGODB_URI=mongodb://db:27017/transactions2db
```

---

## `.gitignore`

```gitignore
node_modules
.env
.DS_Store
npm-debug.log
```

---

## `docker-compose.yml`

```yaml
services:
  app:
    build: .
    working_dir: /app
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - db
    command: sleep infinity

  db:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

---

## `Dockerfile`

```dockerfile
FROM node:20-bookworm

RUN apt-get update && \
    apt-get install -y wget gnupg && \
    wget -qO - https://pgp.mongodb.com/server-7.0.asc | \
    gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg && \
    echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/debian bookworm/mongodb-org/7.0 main" \
    > /etc/apt/sources.list.d/mongodb-org-7.0.list && \
    apt-get update && \
    apt-get install -y mongodb-mongosh && \
    apt-get clean

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

---

## `.devcontainer/devcontainer.json`

```json
{
  "name": "Transactions 2 API",
  "dockerComposeFile": "../docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/app",
  "shutdownAction": "stopCompose",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-azuretools.vscode-docker",
        "mongodb.mongodb-vscode",
        "dbaeumer.vscode-eslint"
      ]
    }
  },
  "forwardPorts": [3000, 27017],
  "remoteUser": "root"
}
```

---

## `package.json`

```json
{
  "name": "transactions2-api",
  "version": "1.0.0",
  "description": "Transactions API using Express, MongoDB, models, controllers, and routes",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon --legacy-watch server.js",
    "seed": "node seed.js"
  },
  "dependencies": {
    "dotenv": "^16.4.7",
    "express": "^4.18.3",
    "mongodb": "^6.8.0"
  },
  "devDependencies": {
    "nodemon": "^3.1.4"
  }
}
```

---

## `config/database.js`

```js
const { MongoClient } = require("mongodb");

let client;
let db;

async function connectToDatabase() {
  if (db) {
    return db;
  }

  const uri = process.env.MONGODB_URI;

  if (!uri) {
    throw new Error("Missing MONGODB_URI environment variable.");
  }

  client = new MongoClient(uri);

  await client.connect();

  db = client.db();

  console.log("Connected to MongoDB");

  return db;
}

function getDatabase() {
  if (!db) {
    throw new Error("Database has not been initialized.");
  }

  return db;
}

async function closeDatabaseConnection() {
  if (client) {
    await client.close();
    client = null;
    db = null;
  }
}

module.exports = {
  connectToDatabase,
  getDatabase,
  closeDatabaseConnection
};
```

---

## `models/transactionModel.js`

```js
const { ObjectId } = require("mongodb");
const { getDatabase } = require("../config/database");

const COLLECTION_NAME = "transactions";

const validCardTypes = ["Visa", "Master", "Amex", "Discover", "Other"];

function getCollection() {
  const db = getDatabase();
  return db.collection(COLLECTION_NAME);
}

function validateTransaction(body) {
  if (!body.creditCardNickname) {
    return "creditCardNickname is required.";
  }

  if (!body.cardType) {
    return "cardType is required.";
  }

  if (!validCardTypes.includes(body.cardType)) {
    return "Invalid card type.";
  }

  if (!body.date) {
    return "date is required.";
  }

  if (Number.isNaN(Date.parse(body.date))) {
    return "Invalid date.";
  }

  if (body.amount === undefined) {
    return "amount is required.";
  }

  if (typeof body.amount !== "number") {
    return "amount must be numeric.";
  }

  return null;
}

function buildTransaction(body) {
  return {
    creditCardNickname: body.creditCardNickname,
    cardType: body.cardType,
    date: new Date(body.date),
    amount: body.amount,
    amendment: body.amendment === true,
    comment: body.comment || null,
    createdAt: new Date()
  };
}

async function createTransaction(body) {
  const validationError = validateTransaction(body);

  if (validationError) {
    const error = new Error(validationError);
    error.statusCode = 400;
    throw error;
  }

  const transaction = buildTransaction(body);

  const result = await getCollection().insertOne(transaction);

  return {
    ...transaction,
    _id: result.insertedId
  };
}

async function findTransactions(query) {
  const filter = {};

  const { date, startDate, endDate, creditCardNickname } = query;

  if (creditCardNickname) {
    filter.creditCardNickname = creditCardNickname;
  }

  if (date) {
    const start = new Date(date);
    const end = new Date(date);

    end.setDate(end.getDate() + 1);

    filter.date = {
      $gte: start,
      $lt: end
    };
  }

  if (startDate || endDate) {
    filter.date = {};

    if (startDate) {
      filter.date.$gte = new Date(startDate);
    }

    if (endDate) {
      const end = new Date(endDate);
      end.setDate(end.getDate() + 1);
      filter.date.$lt = end;
    }
  }

  return getCollection()
    .find(filter)
    .sort({ date: -1 })
    .toArray();
}

async function findTransactionById(id) {
  if (!ObjectId.isValid(id)) {
    const error = new Error("Invalid id.");
    error.statusCode = 400;
    throw error;
  }

  const transaction = await getCollection().findOne({
    _id: new ObjectId(id)
  });

  return transaction;
}

module.exports = {
  validCardTypes,
  createTransaction,
  findTransactions,
  findTransactionById
};
```

---

## `controllers/transactionController.js`

```js
const Transaction = require("../models/transactionModel");

async function createTransaction(req, res, next) {
  try {
    const transaction = await Transaction.createTransaction(req.body);
    res.status(201).json(transaction);
  } catch (error) {
    next(error);
  }
}

async function getTransactions(req, res, next) {
  try {
    const transactions = await Transaction.findTransactions(req.query);
    res.json(transactions);
  } catch (error) {
    next(error);
  }
}

async function getTransactionById(req, res, next) {
  try {
    const transaction = await Transaction.findTransactionById(req.params.id);

    if (!transaction) {
      return res.status(404).json({
        error: "Transaction not found."
      });
    }

    res.json(transaction);
  } catch (error) {
    next(error);
  }
}

module.exports = {
  createTransaction,
  getTransactions,
  getTransactionById
};
```

---

## `routes/transactionRoutes.js`

```js
const express = require("express");
const transactionController = require("../controllers/transactionController");

const router = express.Router();

router.post("/", transactionController.createTransaction);
router.get("/", transactionController.getTransactions);
router.get("/:id", transactionController.getTransactionById);

module.exports = router;
```

---

## `server.js`

```js
require("dotenv").config();

const express = require("express");
const { connectToDatabase } = require("./config/database");
const transactionRoutes = require("./routes/transactionRoutes");

const app = express();

const PORT = process.env.PORT || 3000;

app.use(express.json());

app.get("/", (req, res) => {
  res.json({
    message: "Transactions 2 API is running",
    routes: {
      transactions: "/api/transactions"
    }
  });
});

app.use("/api/transactions", transactionRoutes);

app.use((req, res) => {
  res.status(404).json({
    error: "Route not found."
  });
});

app.use((error, req, res, next) => {
  console.error(error);

  const statusCode = error.statusCode || 500;

  res.status(statusCode).json({
    error: error.message || "Internal server error."
  });
});

async function startServer() {
  await connectToDatabase();

  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
}

startServer().catch((error) => {
  console.error(error);
  process.exit(1);
});
```

---

## `seed.js`

```js
require("dotenv").config();

const { MongoClient } = require("mongodb");

const MONGODB_URI = process.env.MONGODB_URI;

if (!MONGODB_URI) {
  console.error("Missing MONGODB_URI in .env");
  process.exit(1);
}

const cardNicknames = [
  "Costco Visa",
  "Amazon Visa",
  "Travel Master",
  "Blue Amex",
  "Discover Cashback",
  "Everyday Card"
];

const cardTypes = [
  "Visa",
  "Master",
  "Amex",
  "Discover",
  "Other"
];

const comments = [
  "Gas",
  "Groceries",
  "Restaurant",
  "Coffee",
  "Flight",
  "Hotel",
  "Books",
  "Online Purchase",
  "Utilities",
  "Streaming Service",
  "Pharmacy",
  "Electronics",
  "School Supplies",
  "Parking",
  "Car Maintenance"
];

function randomElement(array) {
  return array[Math.floor(Math.random() * array.length)];
}

function randomAmount(min, max) {
  return Number((Math.random() * (max - min) + min).toFixed(2));
}

function randomDateWithinLast90Days() {
  const now = new Date();
  const daysAgo = Math.floor(Math.random() * 90);
  const date = new Date(now);

  date.setDate(now.getDate() - daysAgo);

  return date;
}

function generateTransaction() {
  const amendment = Math.random() < 0.1;

  const amount = amendment
    ? -randomAmount(5, 250)
    : randomAmount(5, 250);

  return {
    creditCardNickname: randomElement(cardNicknames),
    cardType: randomElement(cardTypes),
    date: randomDateWithinLast90Days(),
    amount,
    amendment,
    comment: amendment ? "Amendment Transaction" : randomElement(comments),
    createdAt: new Date()
  };
}

async function seed() {
  const client = new MongoClient(MONGODB_URI);

  try {
    await client.connect();

    console.log("Connected to MongoDB");

    const db = client.db();
    const collection = db.collection("transactions");

    await collection.deleteMany({});

    const transactions = [];

    for (let i = 0; i < 100; i++) {
      transactions.push(generateTransaction());
    }

    const result = await collection.insertMany(transactions);

    console.log(`Inserted ${result.insertedCount} transactions`);
  } catch (error) {
    console.error(error);
  } finally {
    await client.close();
    console.log("Disconnected from MongoDB");
  }
}

seed();
```

---

## `Procfile`

```txt
web: node server.js
```

---

# Final Check

Before submitting, verify:

```bash
npm install
npm run seed
npm run dev
```

Then test:

```txt
GET http://localhost:3000/
GET http://localhost:3000/api/transactions
POST http://localhost:3000/api/transactions
GET http://localhost:3000/api/transactions/:id
```

Then deploy and test:

```txt
GET https://YOUR-HEROKU-APP-NAME.herokuapp.com/
GET https://YOUR-HEROKU-APP-NAME.herokuapp.com/api/transactions
POST https://YOUR-HEROKU-APP-NAME.herokuapp.com/api/transactions
```
