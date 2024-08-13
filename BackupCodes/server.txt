const express = require('express');
const bodyParser = require('body-parser');
const cookieSession = require('cookie-session');

const app = express();
const port = process.env.PORT || 3000;

// Middleware
app.use(express.static('public'));
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

// Set up cookie-session
app.use(cookieSession({
  name: 'session',
  keys: ['your-secret-key'], // Replace with your secret key(s)
  maxAge: 24 * 60 * 60 * 1000 // 24 hours
}));

// Your other routes and logic

// Start the server
app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});

module.exports = app;
