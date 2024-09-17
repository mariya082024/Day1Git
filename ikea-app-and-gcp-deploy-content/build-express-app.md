//CREATE EXPRESS APP server side
Objectives
Build a Node Express API
Create CRUD endpoints for a User resource
The app will also serve up the React App
**Getting Started - Initialize Express App**
Let's build our own Node/Express API and test it out with Postman. We will use a SQLite datastore and use a package called Sequelize to connect to the database.

There is a Postman collection in this folder that contains the endpoints we'll build: ikea-app-and-gcp-deploy-content/IKEA-Express-Users-App.postman_collection.json

1.In your VM, open your Terminal and create a new folder in your users directory: mkdir ikea-users-app

2.Change into that directory: cd ikea-users-app

3.Open the directory in VS Code: code .

4.Run npm init -y.

        This will create a package.json file to initialize our node application.
package.json is a JSON file that lives in the root directory of your project. Your package.json holds important information about the project. It contains human-readable metadata about the project (like the project name and description) as well as functional metadata like the package version number and a list of dependencies required by the application.

5.This will install a few node packages for us: npm install express nodemon cors sequelize sqlite3

express -Express is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.
nodemon - This will automatically restart the node application when file changes in the directory are detected.
cors - (helps with potential cross-origin resource sharing issues)
Sequelize will convert Javascript methods to raw SQL in order to connect to the database.
sqlite3 - This package is a light-weight database file that is great for development.

6. touch index.js - Running this command will create a file to serve as the entrypoint of our app.

**Create a .gitignore file**
1.touch .gitignore
2. It's not a good practice to check in node_modules folder since each developer can download their own version of the modules locally. Add this to the file: node_modules

**Add code to index.js**
1.Open the index.js file in your editor and add the following content. This creates a basic server and a single route.

// imports the express npm module
const express = require("express");
// imports the cors npm module
const cors = require("cors");
// Creates a new instance of express for our app
const app = express();

// .use is middleware - something that occurs between the request and response cycle.
app.use(cors());
 // We will be using JSON objects to communcate with our backend, no HTML pages.
app.use(express.json());
// This will serve the React build when we deploy our app
app.use(express.static("react-frontend/dist"));

// This route will return 'Hello Ikea!' when you go to localhost:8080/ in the browser
app.get("/", (req, res) => {
    res.json({ data: 'Hello Ikea!' });
});

// This tells the express application to listen for requests on port 8080
const port = process.env.PORT || 8080;
app.listen(port, async () => {
    console.log(`Server started at ${port}`);
});
2. Add a start command to package.json. We'll use nodemon to serve our app. This way we don't have to stop and restart the server each time we make a change to our code.

"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon index.js" // ADD THIS
}, 
3.Run npm run start from the Terminal and go to http://localhost:8080/ in the browser.

**Set-up our Database**
1.At the top of index.js add these imports:
const { Sequelize, Model, DataTypes } = require('sequelize');

2.Next, initialize a new Sequelize instance.

// Create Sequelize instance
const sequelize = new Sequelize({
    dialect: 'sqlite',
    storage: './database.sqlite'
});

3.Create a User model.

// Define User model
class User extends Model {}
User.init({
    name: DataTypes.STRING,
    isAdmin: DataTypes.BOOLEAN,
}, { sequelize, modelName: 'user' });

// Sync models with database
sequelize.sync();
4. Hardcode a users array with name and isAdmin properties. You can also use the vscode-faker extention.

const users = [
    { name: "John Doe",  isAdmin: false },
    { name: "Jane Smith", isAdmin: false },
    { name: "Mike Johnson", isAdmin: false  },
    { name: "Sarah Williams", isAdmin: false  },
    { name: "David Brown", isAdmin: false  }
];

Note - Feel free to add some more users or add/change your fields. However, if you do add/change your fields then make sure they align with the fields defined in the User.init step above ^^. Try deleting your database.sqlite file if you encounter any issues.

1.Create an /api/seeds endpoint so that we can quickly populate our database with some users.

app.get('/api/seeds', async (req, res) => {
    users.forEach(u => User.create(u));
    res.json(users);
});
2.Run npm run start to start your app, then go to http://localhost:8080/api/seeds in the browser.

**GET /api/users**
1.Create a GET route. Take a minute to consider what this code is doing.

app.get('/api/users', async (req, res) => {
    const users = await User.findAll();
    res.json(users);
});

2.Go to http://localhost:8080/api/users in the browser.
------------------------------------------------------------
**GET /api/users/:id**
1. Create a GET route to return a single user. Take a minute to consider what this code is doing.

app.get("/api/users/:id", async (req, res) => {
    const user = await User.findByPk(req.params.id);
    res.json(user);
});
2. Go to http://localhost:8080/api/users/1 in the browser.

**POST /api/users**
1.Create a POST route. We'll test this out in Postman later.

app.post('/api/users', async (req, res) => {
    const user = await User.create(req.body);
    res.json(user);
});

**PUT /api/users/:id**
1.Create a PUT route. We'll test this out in Postman later.

app.put("/api/users/:id", async (req, res) => {
    const { name, isAdmin } = req.body;

    const user = await User.findByPk(req.params.id);
    await user.update({ name, isAdmin });
    await user.save();
    res.json(user);
});

**DELETE /api/users/:id**
1.Create a DELETE route. We'll test this out in Postman later.

app.delete('/api/users/:id', async (req, res) => {
    const user = await User.findByPk(req.params.id);
    await user.destroy();
    res.json({data: `The user with id of ${req.params.id} is removed.`});
});




