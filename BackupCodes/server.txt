const express = require('express');  // Make sure this is at the top
const bodyParser = require('body-parser');
const cookieSession = require('cookie-session');
const fs = require('fs');
const path = require('path');

const app = express();  // Make sure this comes immediately after requiring express
const port = process.env.PORT || 3000;
const usersDir = path.join(__dirname, 'users');

// Ensure the 'users' directory exists
if (!fs.existsSync(usersDir)) {
    fs.mkdirSync(usersDir);
}

// Middleware
app.use(express.static('public'));
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

app.use(cookieSession({
    name: 'session',
    keys: ['your-secret-key'], // Replace with your own secret key
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
}));

// Function to read user data from a file
function readUserData(username) {
    const filePath = path.join(usersDir, `${username}.txt`);
    if (fs.existsSync(filePath)) {
        const data = fs.readFileSync(filePath, 'utf8');
        return JSON.parse(data);
    }
    return null;
}

// Function to write user data to a file
function writeUserData(username, data) {
    const filePath = path.join(usersDir, `${username}.txt`);
    fs.writeFileSync(filePath, JSON.stringify(data));
}

// Route to serve the main index page
app.get('/', (req, res) => {
    if (!req.session.user) {
        return res.redirect('/login.html');
    }
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// Signup route
app.post('/signup', (req, res) => {
    const { username, password } = req.body;
    const userFilePath = path.join(usersDir, `${username}.txt`);

    if (fs.existsSync(userFilePath)) {
        return res.send('Username already exists. Please <a href="/signup.html">try again</a>.');
    }

    const userData = { username, password, balance: 100, stocks: {} };
    writeUserData(username, userData);
    req.session.user = userData;
    res.redirect('/');
});

// Login route
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    const userData = readUserData(username);

    if (userData && userData.password === password) {
        req.session.user = userData;
        return res.redirect('/');
    }
    res.send('Invalid username or password. Please <a href="/login.html">try again</a>.');
});

// Route to get user data for the logged-in user
app.get('/user-data', (req, res) => {
    if (!req.session.user) {
        return res.status(401).json({ error: 'Not logged in' });
    }
    res.json(req.session.user);
});

// Route to update user data for the logged-in user
app.post('/user-data', (req, res) => {
    if (!req.session.user) {
        return res.status(401).json({ error: 'Not logged in' });
    }

    const { balance, stocks } = req.body;
    req.session.user.balance = balance;
    req.session.user.stocks = stocks;
    writeUserData(req.session.user.username, req.session.user);
    res.json({ success: true });
});

// Logout route
app.get('/logout', (req, res) => {
    req.session = null;
    res.redirect('/login.html');
});

// Start the server
app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});

module.exports = app;
