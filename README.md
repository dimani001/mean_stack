# MEAN Stack Book Management App — Full Setup & Usage Guide


## Table of Contents

1. [About MEAN Stack](#about-mean-stack)  
2. [Project Overview](#project-overview)  
3. [Prerequisites](#prerequisites)  
4. [Basic Linux Commands for Beginners](#basic-linux-commands-for-beginners)  
5. [Step 1: Launch an AWS EC2 Instance](#step-1-launch-an-aws-ec2-instance)  
6. [Step 2: Configure Security Group (Open Ports)](#step-2-configure-security-group-open-ports)  
7. [Step 3: Connect to Your EC2 via SSH](#step-3-connect-to-your-ec2-via-ssh)  
8. [Step 4: Install Node.js](#step-4-install-nodejs)  
9. [Step 5: Install MongoDB (Optional)](#step-5-install-mongodb-optional)  
10. [Step 6: Prepare Project Directory & Files](#step-6-prepare-project-directory--files)  
11. [Step 7: Run the Node.js Server](#step-7-run-the-nodejs-server)  
12. [Additional Notes](#additional-notes)  


## About MEAN Stack

The **MEAN Stack** is a popular full-stack JavaScript framework consisting of:

- **MongoDB:** NoSQL database to store application data  
- **Express.js:** Backend web framework running on Node.js  
- **AngularJS:** Frontend framework for building dynamic user interfaces  
- **Node.js:** JavaScript runtime environment used on the server  

This stack allows developers to write both frontend and backend code in JavaScript, making development efficient and streamlined.


## Project Overview

This project is a simple **Book Management App** that lets you:

- Display a list of books  
- Delete a book by its ISBN  
- Interact with a RESTful API built with Express.js  
- Use AngularJS for the frontend UI  


## Prerequisites

- An AWS account  
- Basic knowledge of terminal commands  
- A downloaded `.pem` key file for your EC2 instance  


## Basic Linux Commands for Beginners

| Command                | Description                        | Example                           |
|------------------------|----------------------------------|---------------------------------|
| `pwd`                  | Show current directory            | `$ pwd`                         |
| `ls`                   | List files/folders                | `$ ls`                          |
| `cd <directory>`       | Change directory                  | `$ cd Books`                    |
| `mkdir <directory>`    | Create new directory              | `$ mkdir Books`                 |
| `nano <file>`          | Edit file in terminal text editor| `$ nano server.js`              |
| `cat <file>`           | Display file content              | `$ cat README.md`               |
| `chmod 400 <file>`     | Set permissions (for .pem keys)  | `$ chmod 400 mykey.pem`         |
| `exit`                 | Exit SSH session or terminal      | `$ exit`                       |


## Step 1: Launch an AWS EC2 Instance

1. Log in to AWS Console → Navigate to **EC2 Dashboard** → Click **Launch Instances**.  
2. Choose **Ubuntu Server 22.04 LTS** (or latest LTS).  
3. Select instance type (e.g., **t3.micro** for free tier).  
4. Configure instance details as needed.  
5. Add storage (default is fine).  
6. **Configure Security Group** (see next step).  
7. Review and Launch. Download the `.pem` key file securely.


## Step 2: Configure Security Group (Open Ports)

Ensure your EC2 security group allows:

| Type       | Protocol | Port Range | Source           |
|------------|----------|------------|------------------|
| SSH        | TCP      | 22         | Your IP (recommended) or 0.0.0.0/0 |
| Custom TCP | TCP      | 3300       | 0.0.0.0/0 (for app access) |


## Step 3: Connect to Your EC2 via SSH

1. Open terminal and navigate to your key file location:

```bash
cd path/to/keyfile
chmod 400 your-key.pem
```

2. Connect using SSH:

```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

3.  Update the Server

```bash
sudo apt update && sudo apt upgrade -y
```

## Step 4: Install Node.js and NPM

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v    # Confirm Node.js installed
npm -v     # Confirm npm installed
```

## Step 5: Install MongoDB 

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
mongo --eval 'db.runCommand({ connectionStatus: 1 })'
mongo --version
```

## Step 6: Prepare Project Directory & Files

Inside the Books folder, create the following structure:

```bash
Books/
├── apps/
│   ├── models/
│   │   └── book.js
│   └── routes.js
├── public/
│   ├── index.html
│   └── script.js
└── server.js
```

 Use nano or your favorite editor to create and paste the respective code into these files.

  . server.js: Main Express server file
  . apps/routes.js: API routes (GET, DELETE for books)
  . public/script.js: AngularJS frontend code


1. Create your project directory and navigate to it:

```bash
mkdir Books
cd Books
```

2.  Initialize Node Project & Install Dependencies

```bash
npm init -y
npm install express body-parser
```

3. Create Project Structure

```bash
mkdir -p apps public
cd apps
nano routes.js
```

Paste the following in routes.js:

```js
const express = require('express');
const router = express.Router();

let books = [
  { isbn: '1234567890', title: 'Book One', author: 'Author A' },
  { isbn: '0987654321', title: 'Book Two', author: 'Author B' }
];

router.get('/book', (req, res) => {
  res.json(books);
});

router.delete('/book/:isbn', (req, res) => {
  const isbnToDelete = req.params.isbn;
  const originalLength = books.length;
  books = books.filter(book => book.isbn !== isbnToDelete);

  if (books.length === originalLength) {
    return res.status(404).json({ message: 'Book not found' });
  }

  res.json({ message: `Book with ISBN ${isbnToDelete} deleted.` });
});

module.exports = router;
```

4. ## Create Book Schema

```bash
mkdir models
cd models
sudo nano book.js
```

Paste the following:

```js
var mongoose = require('mongoose');

var bookSchema = new mongoose.Schema({
  name: String,
  isbn: { type: String, index: true },
  author: String,
  pages: Number
});

module.exports = mongoose.model('Book', bookSchema);
```


Then:

```bash
cd ../public
nano index.html
```

Paste:

```html
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>
        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>
          <td><input type="button" value="Delete" ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```

Now:

```bash
sudo nano script.js
```

Paste:

```js
var app = angular.module('myApp', []);

app.controller('myCtrl', function($scope, $http) {
  // Fetch books from server
  $http({
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.error('Error fetching books:', response);
  });

  // Delete book by ISBN
  $scope.del_book = function(book) {
    $http({
      method: 'DELETE',
      url: '/book/' + book.isbn
    }).then(function successCallback(response) {
      alert(response.data.message);
      // Update local list of books without the deleted one
      $scope.books = $scope.books.filter(b => b.isbn !== book.isbn);
    }, function errorCallback(response) {
      console.error('Error deleting book:', response);
    });
  };
});
```


5.  Create server.js in Root

```bash
cd ~/Books
nano server.js
```

Paste:

```js
const express = require('express');
const app = express();
const port = 3300;  // Use your chosen port

const routes = require('./apps/routes');

app.use(express.json());
app.use(express.static('public'));

app.use('/', routes);

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```


  ## Step 7: Run the Node.js Server

1. Install npm dependencies (if package.json exists):

```bash
  npm install
  ```
2. Start your server:

```bash
node server.js
```
Access the app in your browser:

http://<your-ec2-public-ip>:3300








