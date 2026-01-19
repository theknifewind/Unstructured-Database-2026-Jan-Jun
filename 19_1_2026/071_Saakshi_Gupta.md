Lab Assignment - 19/01/2026
Work Done:
SET Key value
```
SET counter 10

```
GET Key value
```
GET counter

```
INCR
```
INCR counter

```
EXPIRE
```
EXPIRE counter 60

```

Node Server Timed Rate Limiter:
```
const express = require("express");
const { createClient } = require("redis");

const app = express();

const redisClient = createClient({
  url: "redis://localhost:6379"
});

redisClient.connect();

// rate limit config
const WINDOW_SIZE = 10;   // seconds
const MAX_REQUESTS = 5;

app.use(async (req, res, next) => {
  try {
    const ip = req.ip;
    const key = `rate:${ip}`;

    const current = await redisClient.incr(key);

    if (current === 1) {
      await redisClient.expire(key, WINDOW_SIZE);
    }

    if (current > MAX_REQUESTS) {
      return res.status(429).json({
        error: "Too many requests. Try again later."
      });
    }

    next();
  } catch (err) {
    console.error(err);
    res.status(500).send("Server error");
  }
});

app.get("/", (req, res) => {
  res.send("Request allowed");
});

app.listen(3000, () => {
  console.log("Rate limiter running on port 3000");
});

```