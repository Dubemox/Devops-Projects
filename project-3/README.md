# Project 3: Simple To-Do Application on the MERN Stack on AWS

A hands-on project where I learn DevOps by building and running real infrastructure on AWS, and documenting every step along the way.

## Goals

- Understand RESTful APIs
- Get familiar with web application frameworks
- Explain the difference between relational databases (RDBMS) and NoSQL databases
- Learn the basics of cascading style sheets (CSS)
- Use JavaScript syntax confidently

## The MERN Stack

| Letter | Technology | Role |
|--------|------------|------|
| **M** | MongoDB | NoSQL database that stores the data |
| **E** | Express.js | Web framework for building the backend API |
| **R** | React | Frontend library for building the user interface |
| **N** | Node.js | JavaScript runtime that runs the backend |

## Environment

| Item | Value |
|------|-------|
| Cloud provider | AWS (free tier) |
| Service | EC2 |
| Region | eu-north-1 (Stockholm) |
| OS | Ubuntu `[version]` |
| Instance type | `[e.g. t3.micro]` |
| Database | MongoDB Atlas (free cluster) |
| Connection | SSH from Windows PowerShell |

---

## Step 0: Preparing Prerequisites

To complete this project, I needed the AWS free tier account I created in my [previous projects](link-to-project-1).

1. I launched a new EC2 instance and confirmed it was running:

![EC2 instance in running state](running-state.png)

**Task:** deploy a simple To-Do application.

---

## Step 1: Backend Configuration

### Updating Ubuntu and installing Node.js

I updated the package list:

```bash
sudo apt update
```

![Updating Ubuntu](update-ubuntu.png)

I upgraded the installed packages:

```bash
sudo apt upgrade
```

![Upgrading Ubuntu](upgrade-ubuntu.png)

I installed Node.js (`[if you added the NodeSource repository first, put that command here]`):

```bash
sudo apt-get install -y nodejs
```

![Node.js installation](node-install.png)

I verified the installation:

```bash
node -v
npm -v
```

![Node.js and npm versions](version.png)

### Application code setup

I created a directory for the To-Do project and moved into it:

```bash
mkdir Todo
ls
cd Todo
```

![Todo directory](todo-dir.png)

I initialized the project, which creates a `package.json` file to track dependencies:

```bash
npm init
```

![Initializing the project](init-status.png)

I ran `ls` to confirm that `package.json` was created.

### Installing Express.js

Express is a web framework for Node.js:

```bash
npm install express
```

![Express installation](express-install.png)

I created a file named `index.js` and ran `ls` to confirm it:

```bash
touch index.js
ls
```

I installed the `dotenv` module, which loads environment variables from a `.env` file:

```bash
npm install dotenv
```

![dotenv installation](dotenv-install.png)

I opened `index.js` in the vi editor and pasted in the application code:

```bash
vi index.js
```

![index.js code](index-file-1.png)

I started the server to check that it works:

```bash
node index.js
```

![Server running](node-server.png)

Next, I added an inbound rule for port 5000 to the instance's security group:

![Inbound rule for port 5000](inbound-rule.png)

Then I opened the server in my browser:

```text
http://<public-ip-or-public-dns>:5000
```

![Express server in the browser](express-server.png)

### Routes

The To-Do application needs to do three things:

- Create a new task
- Display a list of all tasks
- Delete a completed task

Each action uses a different HTTP method: `POST`, `GET` and `DELETE`. For each one, I needed to create a route that defines its endpoint. I created a `routes` folder and moved into it:

```bash
mkdir routes
cd routes
```

![Routes folder](routes-0.png)

I created `api.js` for the endpoints and opened it in vi to paste in the code:

```bash
touch api.js
vi api.js
```

![api.js code](api-0.png)

### Models

The application uses MongoDB, so I needed a model that defines the shape of the data. I went back to the `Todo` directory and installed **Mongoose**, a library that makes working with MongoDB much easier:

```bash
cd ..
npm install mongoose
```

Then I created a `models` folder (next to `routes`, not inside it) and moved into it:

```bash
mkdir models
cd models
```

![Mongoose installation and models folder](models-0.png)

In the `models` folder, I created a file called `todo.js` and pasted in the code:

```bash
touch todo.js
vi todo.js
```

![todo.js code](todo.js-code.png)

Then I went back to the `Todo` directory and updated `routes/api.js` to use the model:

```bash
cd ..
vi routes/api.js
```

![Updated api.js](api.js-update.png)

### MongoDB database

The application needs a database to store tasks. I signed up for MongoDB Atlas, created a free cluster, and chose AWS as the cloud provider with a region near me.

![MongoDB Atlas cluster](mongo-1.png)

I allowed access to the database from anywhere. This is **not secure**, but it is fine for testing:

![Network access list](mongo-2.png)

I created a `.env` file in the `Todo` directory to hold the Atlas connection string:

```bash
touch .env
vi .env
```

```text
DB="mongodb+srv://<db_user>:<db_password>@<cluster-address>/"
```

> **Important:** Never commit `.env` to Git or publish your real connection string. Add `.env` and `node_modules` to `.gitignore`.

*(The variable name must match what `index.js` reads from `process.env`.)*

I updated `index.js` so it connects to the database:

```bash
vi index.js
```

![Updated index.js](index-update-0.png)

I started the server again to check that it works:

```bash
node index.js
```

![Server connected to the database](server-check-0.png)

### Testing the API with Postman

I tested the API endpoints in Postman.

`POST` request working:

![POST request in Postman](post.png)

`GET` request working:

![GET request in Postman](get.png)

`DELETE` request working:

![DELETE request in Postman](delete.png)

The backend is complete.

---

## Step 2: Frontend Creation

I created a new folder called `client` for the React code, using Vite (run from the `Todo` directory):

```bash
npm create vite@latest client -- --template react
```

From the `Todo` directory, I installed two development dependencies:

```bash
# Runs more than one command at the same time from one terminal window
npm install concurrently --save-dev

# Runs the server and restarts it automatically when files change
npm install nodemon --save-dev
```

![Installing dependencies](depend-1.png)

I updated the scripts in the `package.json` file in the `Todo` directory:

```bash
vi package.json
```

```json
"scripts": {
  "start": "node index.js",
  "start-watch": "nodemon index.js",
  "dev": "concurrently \"npm run start-watch\" \"cd client && npm run dev\""
}
```

![Updated package.json](package-1.png)

I added a proxy configuration so the frontend can call the backend directly, without the full server address:

```text
proxy: {
    '/api': 'http://localhost:5000'
}
```

![Proxy configuration](proxy-0.png)

I ran the app:

```bash
npm run dev
```

![Running the app](server-run-1.png)

![App running in the terminal](server-run.png)

### Creating the components

I moved into the `client` directory, installed **Axios** (a promise-based HTTP client for the browser and Node.js), then moved into `src` and created a `components` folder:

```bash
cd client
npm install axios
cd src
mkdir components
```

![Axios installation](axios.png)

In the `components` folder, I created three files: `Input.jsx`, `ListTodo.jsx` and `Todo.jsx`:

```bash
cd components
touch Input.jsx ListTodo.jsx Todo.jsx
```

Then I pasted the code into each file.

`Input.jsx`:

```bash
vi Input.jsx
```

![Input.jsx code](input.png)

`ListTodo.jsx`:

```bash
vi ListTodo.jsx
```

![ListTodo.jsx code](listtodo.png)

`Todo.jsx`:

```bash
vi Todo.jsx
```

![Todo.jsx code](todo.png)

I updated `App.jsx` to display the To-Do application:

```bash
# Go back up to the src directory
cd ..

vi App.jsx
```

![App.jsx code](app.png)

### Styling the application

I added the styles for the application to `App.css`:

```bash
vi App.css
```

```css
.App {
  text-align: center;
  font-size: calc(10px + 2vmin);
  width: 60%;
  margin: 0 auto;
}

input {
  height: 40px;
  width: 50%;
  border: none;
  border-bottom: 2px #101113 solid;
  background: none;
  font-size: 1.5em;
  color: #787a80;
}

input:focus {
  outline: none;
}

button {
  width: 25%;
  height: 45px;
  border: none;
  margin-left: 10px;
  font-size: 25px;
  background: #101113;
  border-radius: 5px;
  color: #787a80;
  cursor: pointer;
}

button:focus {
  outline: none;
}

ul {
  list-style: none;
  text-align: left;
  padding: 15px;
  background: #171a1f;
  border-radius: 5px;
}

li {
  padding: 15px;
  font-size: 1.5em;
  margin-bottom: 15px;
  background: #282c34;
  border-radius: 5px;
  overflow-wrap: break-word;
  cursor: pointer;
}

@media only screen and (min-width: 300px) {
  .App {
    width: 80%;
  }

  input {
    width: 100%;
  }

  button {
    width: 100%;
    margin-top: 15px;
    margin-left: 0;
  }
}

@media only screen and (min-width: 640px) {
  .App {
    width: 60%;
  }

  input {
    width: 50%;
  }

  button {
    width: 30%;
    margin-left: 10px;
    margin-top: 0;
  }
}
```

I also updated `index.css`:

```bash
vi index.css
```

![index.css code](index.css.png)

### Verifying the application

I went back to the `Todo` directory and ran the whole app:

```bash
# Go back to the Todo directory
cd ../..

# Run the app
npm run dev
```

To open the frontend in my browser, I added an inbound rule for port 5173, the port Vite's development server uses:

```text
http://<public-ip-or-public-dns>:5173
```

![To-Do application working in the browser](complete-todo.png)

---

## Troubleshooting & Lessons Learned

- `[Problem you hit, how you diagnosed it, and the fix]`
- Directory location matters: `routes` and `models` must be inside the `Todo` directory, and commands such as `npm install` must be run from there.

## Cleanup

To avoid unexpected charges and exposure:

- Stop or terminate the EC2 instance when I'm not using it.
- Remove the "allow access from anywhere" rule in MongoDB Atlas, or restrict it to my instance's IP.
- Close the inbound ports (5000 and 5173) in the security group when they are no longer needed.

## Author

Dubemox · Lagos, Nigeria.