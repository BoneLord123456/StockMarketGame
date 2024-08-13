const express = require('express');
const bodyParser = require('body-parser');
const session = require('express-session');
const fs = require('fs');
const path = require('path');
const app = express();
const port = 3000;

// Middleware
app.use(express.static('public'));
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json()); // To handle JSON data
app.use(session({
    secret: 'secret-key',
    resave: false,
    saveUninitialized: true
}));

const usersDir = path.join(__dirname, 'users');

// Ensure the 'users' directory exists
if (!fs.existsSync(usersDir)) {
    fs.mkdirSync(usersDir);
}

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

// Signup route
app.post('/signup', (req, res) => {
    const { username, password } = req.body;
    const userFilePath = path.join(usersDir, `${username}.txt`);

    if (fs.existsSync(userFilePath)) {
        res.send('Username already exists. Please <a href="/signup.html">try again</a>.');
    } else {
        const userData = { username, password, balance: 100, stocks: {} };
        writeUserData(username, userData);
        req.session.user = userData;
        res.redirect('/index.html');
    }
});

// Login route
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    const userData = readUserData(username);

    if (userData && userData.password === password) {
        req.session.user = userData;
        res.redirect('/index.html');
    } else {
        res.send('Invalid username or password. Please <a href="/login.html">try again</a>.');
    }
});

// Middleware to check if user is logged in
app.use((req, res, next) => {
    if (!req.session.user && req.path !== '/login.html' && req.path !== '/signup.html' && !req.path.startsWith('/login') && !req.path.startsWith('/signup')) {
        return res.redirect('/login.html');
    }
    next();
});

// Route to get user data (for client-side use)
app.get('/user-data', (req, res) => {
    if (req.session.user) {
        res.json(req.session.user);
    } else {
        res.status(401).json({ error: 'Not logged in' });
    }
});

// Route to update user data
app.post('/user-data', (req, res) => {
    if (req.session.user) {
        const { balance, stocks } = req.body;
        req.session.user.balance = balance;
        req.session.user.stocks = stocks;
        writeUserData(req.session.user.username, req.session.user);
        res.json({ success: true });
    } else {
        res.status(401).json({ error: 'Not logged in' });
    }
});

// Main menu
app.get('/index.html', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

// Shop page
app.get('/shop.html', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'shop.html'));
});

// Logout
app.get('/logout', (req, res) => {
    req.session.destroy();
    res.redirect('/login.html');
});

// Start the server
app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});
