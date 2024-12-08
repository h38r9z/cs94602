java cStoring Data in a MongoDB Database
1 Introduction
There are two main categories of databases: relational databases and non-relational databases. Relational databases
such as MySQL, SQL Server, and Postgre, store data in tables containing records of the same type, e.g., a table called
courses would contain records representing all the courses, and a users table would contain all the users of an
application. Records are represented as rows in the tables where each column stores data for attributes speciﬁﬁc to the
type of the table, e.g., the rows in the courses table might have columns such as name, description, startDate, endDate,
etc. Some of the columns might refer, or relate to other records in other tables such as the instructor column in the
courses table might refer to, or relate to a particular row in the users table signifying that that particular user is the
instructor of that particular course. Rows in one table relating to rows in another table is where relational databases get
their name. The structured query language or SQL, is a computer language commonly used to interact with relational
databases. The query in SQL generally means to ask for, or retrieve data that matches some criteria, often written as a
boolean expression or predicate.
More recently there has been a growing interest in representing and storing data using alternative strategies which have
collectively come to be referred to as non relational databases, or NoSQL databases. Non relational databases such as
MongoDB, Firebase, and Couchbase, store their data in collections containing documents which are roughly analogous to
tables and records in their relational counterparts. The biggest difference though is that the columns, or ﬁﬁelds in the rows
in relational databases generally can only contain primitive data types, e.g., simple strings, numbers, dates, and booleans,
whereas the ﬁﬁelds, or properties in non relational documents can be arbitrarily complex data types, e.g., strings, numbers,
booleans, dates as well as combinations of these in complex objects containing arrays of objects of arrays, etc. The other
big difference is that relational databases require the structure, or schema of the data to be explicitly described before
storing any data, whereas non relational databases do not require predeﬁﬁned schemas. Instead, non relational databases
delegate this responsibility to the applications using the database. The structure, or schema in relational databases is
where structured query language gets its name.
In the previous chapter we learned how to create an HTTP server with Node.js and integrated it with a React.js Web user
interface application to store the application state on the server. In this chapter we expand on this idea to store the data to
MongoDB, a popular non relational database. The ﬁﬁrst section demonstrates how to download, install and use a local
instance of the MongoDB database. The next section covers how to use the Mongoose library to integrate and program a
MongoDB database with a Node.js server application. The ﬁﬁnal section describes how to deploy the database to Mongo
Atlas, a remote MongoDB database hosted as a cloud service.
The following ﬁﬁgure illustrates the overall architecture of what we'll be building in this chapter. From right to left, we'll ﬁﬁrst
create a MongoDB database called kanbas where we'll create several collections such as users, courses, modules,
assignments, etc. We'll use the Mongoose library to connect to the database programmatically from a Node server. A
Mongoose schema will describe the structure of the collections in the MongoDB database and a Mongoose model will
implement generic CRUD operations. We'll create higher level functions in Data Access Objects (DAOs) that operate on the
database, and expose those operations through Express routes as RESTful Web APIs. A React Web app will integrate with
the RESTful API through a client that will allow the user interface to interact with the database.2 Working with a Local MongoDB Instance
MongoDB is one of an increasingly popular family of non relational databases. Data is stored in collections of documents
usually formatted as JSON objects which makes it very convenient to integrate with JavaScript based frameworks such
as Node.js and React.js. This section describes how to install, conﬁﬁgure and get started using MongoDB.
2.1 Installing and Conﬁﬁguring MongoDB
To get started, download MongoDB for free selecting the latest version for your operating system, and click Download.
Run the installer and, if given the choice, choose to run the database as a service so that you don't have to bother having to
restart the database sever every time you login or restart your computer. The MongoDB database will automatically start
whenever you start your computer. On Windows, conﬁﬁrm the database is running by searching for MongoDB in the
Services dialog. On macOS, conﬁﬁrm the database is running by
clicking the MongoDB icon in the Systems Settings dialog. The
service dialog gives you controls to start and stop the database, but
it should already be conﬁﬁgured to start automatically when you
restart your computer.
2.1.1 Installing MongoDB Manually (optional)
On macOS you can install MongoDB using brew by typing the following at the command line
brew install mongodb-atlas
atlas setup
Alternatively you can unzip the MongoDB server from the downloaded archive to a local ﬁﬁle system and add the right
commands to your operating system PATH environment variable. On macOS, unzip the ﬁﬁle into /usr/local which creates a
directory such as /usr/local/mongodb-macos-x86_64-5.0.3 (your version might differ). To be able to execute the database
related commands, add the path to the .bash_proﬁﬁle or .zshrc ﬁﬁle located in your home directory. Add the following line in
the conﬁﬁguration ﬁﬁle as shown below. Your actual version might differ.
~/.bash_profile or ~/.zshrc
export PATH="$PATH:/usr/local/mongodb-macos-x86_64-5.0.3/bin"
If the .bash_proﬁﬁle or .zshrc ﬁﬁle does not exist in your home directory, create it as a plain text ﬁﬁle, but with no extensions
and a period in front of it. Conﬁﬁgure it as shown above and then restart your computer.
On Windows, unzip the ﬁﬁle into C:\Program Files. To conﬁﬁgure environment variables on Windows press the Windows + R
key combination to open the Run prompt, type sysdm.cpl and press OK. In the System Properties window that appears,
press the Advanced tab and then the Environment Variables button. In the Environment Variables conﬁﬁguration window
select the Path variable and press the Edit button. Copy and paste the path of the bin directory in the mongodb directory
you unzipped the MongoDB download, e.g., "C:\Program Files\mongodb-macos-x86_64-5.0.3\bin". The actual path might
differ. Press OK and restart the computer.
2.1.2 Starting MongoDB from the Command Line
If you installed MongoDB as a service, it is already running in the background and can be conﬁﬁgured and restarted in
Windows from the Services dialog or from the System Settings on macOS. Alternatively you can start the MongoDB server
from the command line using the mongod executable in the bin directory where you installed MongodDB. First you'll needto create a data folder where the server will store all its data. You can create a data folder in your home directory as shown
below.
cd ~
mkdir data
When you start MongoDB, you'll need to tell it where the data folder is with the dbpath option. If you installed MongoDB on
Windows in C:\Program Files\mongodb-macos-x86_64-5.0.3, you can start MongoDB from your home directory as shown
below.
cd ~
C:\Program Files\mongodb-macos-x86_64-5.0.3\bin\mongod --dbpath data
Make sure to include the dbpath data option to tell MongoDB where to ﬁﬁnd the data directory.
If you installed MongoDB on macOS in /usr/local/mongodb-macos-x86_64-6.0.5, you can start MongoDB from your home
directory as shown below.
cd ~
/usr/local/mongodb-macos-x86_64-6.0.5/bin/mongod --dbpath data
2.2 Using MongoDB Compass to Interact with MongoDB
Your installation should have installed MongoDB Compass, a user interface client to the MongoDB database. If not,
MongoDB Compass can be downloaded from MongoDB's download page. You can start Compass from your applications
folder, or search for it in your operating system's search feature. On
macOS bring up Spotlight by pressing the magnifying glass on the top
right menu bar, or press the Command ( ) and Spacebar. Type
MongoDB Compass in the search bar and select the application from
the result list. On Windows press the Window key to bring up the search
ﬁﬁeld, type MongoDB Compass, and select the application from the
result list. When Compass comes up, conﬁﬁrm that the connection
string mongodb:/ 127.0.0.1:27017 appears in the New Connection
screen, and press Connect to connect to MongoDB.
2.3 Creating a MongoDB Database
Once you are connected to a running MongoDB server, click on Databases on the left side bar and then click the Create
database button on the Databases tab. In the Create Database dialog that appears, name your database kanbas and your
ﬁﬁrst collection as users. Click Create Database to create the kanbas database.2.4 Inserting and Retrieving Data with Compass
In MongoDB, data is organized into collections, which are analogous to tables in relational databases. Data contained in
collections are referred to as documents, which are analogous to records in relational databases. To create, or insert
documents into a collection in a MongoDB database using Compass, select the database on the left sidebar and then
select the collection you want to insert documents into. For instance, select the kanbas database and then the users
collection as shown below on the right. On the right side, selected Add Data and then Insert Document. In the Insert to
Collection kanbas.users dialog that appears, insert the document as shown below. Click Insert to insert the document.
Conﬁﬁrm the document inserted as expected.
You can also import entire JSON ﬁﬁles containing data. Import the courses.json ﬁﬁle we used in earlier assignments under
the Database directory of your React project. To import click ADD DATA, then Import JSON or CSV ﬁﬁle. Navigate to the
location of courses.json, select the ﬁﬁle and click Import. Conﬁﬁrm the courses are imported. Also create the following
collections and import the JSON ﬁﬁles linked to each of the collection names. Conﬁﬁrm all collections are imported:
modules.json, assignments.json, users.json
Note the objects stored in the database have a primary key _id automatically added by MongoDB when they were inserted.
MongoDB primary keys are of type ObjectId and are created automatically by the database so your _id values will differ
from the ones shown in this document.
2.5 Interacting with a MongoDB Database with the Command Line (optional)
Compass is a great graphical user interface to the MongoDB database, but there is also value to knowing how to interact
with the database through a command line interface. At the bottom of the Compass window there's a _MONGOSH window
you can expand to type commands to the database. Let's practice a few commands to retrieve data on the command line.
First select the database we want to interact with.
> use kanbas
switched to db kanbas
All the documents in a collection can be retrieved using the ﬁﬁnd() command on a collection as shown below.
> db.courses.find();
Documents in a collection can be retrieved by pattern matching their properties. The example below illustrates how to
retrieve documents by pattern matching their primary key _id, that is, retrieving the document whose _id ﬁﬁeld matches
ObjectId('6370104926906053f1597ce6'). Your ID will likely be different.
> db.courses.find({_id: ObjectId("654e8c73ea7ead465908d1cc")}){
 _id: ObjectId("654e8c73ea7ead465908d1cc"),
name: 'Web Development',
number: 'CS4550',
startDate: '2023-01-10',
endDate: '2023-05-15',
department: 'K123',
credits: 4
}
We can also pattern match any of the other ﬁﬁelds individually or combined with other ﬁﬁelds. The following example
retrieves a document from the courses collection whose number property is equal to RS4560.
> db.courses.find({number: 'RS4560'})
{
 _id: 'RS102',
name: 'Aerodynamics',
number: 'RS4560',
startDate: '2023-01-10',
endDate: '2023-05-15'
}
Here's another example retrieving courses in the D134 department.
> db.courses.find({department: 'D134'})
{ _id: 'CH101',
name: 'Organic Chemistry', number: 'CH1230',
startDate: '2023-01-10', endDate: '2023-05-15',
department: 'D134', credits: 3
}
{ _id: 'CH102',
name: 'Inorganic Chemistry', number: 'CH1240',
startDate: '2023-01-10', endDate: '2023-05-15',
department: 'D134', credits: 3
}
{ _id: 'CH103',
name: 'Physical Chemistry', number: 'CH1250',
startDate: '2023-01-10', endDate: '2023-05-15',
department: 'D134', credits: 3
}
3 Programming with a MongoDB database
In the previous section we practiced interacting with the MongoDB database through the Compass graphical interface as
well as manually on the command line with MONGOSH. This is all and good to make occasional simple queries to conﬁﬁrm
the data behaves as expected, but to create applications we're going to need to interact with the database
programmatically with libraries such as Mongoose. The following sections describe how to install, conﬁﬁgure, and connect
a Node.js application to a MongoDB database server using the Mongoose library. The ﬁﬁnal section discusses how to
conﬁﬁgure the application to integrate to a MongoDB database hosted in the Atlas cloud service. Do all your work in a new
GitHub branch called a6 in both your React.js and Node.js projects.
3.1 Installing and Connecting to a MongoDB Database
The Mongoose library provides a set of operations and abstractions that enhance a MongoDB database and leverages the
familiarity of the MONGOSH command line client. To use the Mongoose library, install it from the root of the Node.js
project as shown below.
$ npm install mongooseTo connect to the database server programmatically, import the Mongoose library and then use the connect function as
shown below. The URL in the connect function is called the connection string and is currently referring to a MongoDB
server instance running in the localhost machine (your current laptop or desktop) listening at port 27017 and the kanbas
database existing in that server. In a later section we'll revisit the connection string and conﬁﬁgure it to connect to a
database server running in a remote machine hosted by Mongo's Atlas cloud service.
index.js
import express from "express";
import mongoose from "mongoose";
...
const CONNECTION_STRING = "mongodb://127.0.0.1:27017/kanbas"
mongoose.connect(CONNECTION_STRING);
const app = express();
...
// load the mongoose library
// connect to the kanbas database
3.2 Conﬁﬁguring Connection Strings as Environment Variables
Instead of hard coding the connection string in the source code, it's better to conﬁﬁgure it as an environment variable and
then reference it from the code. This will come in handy when the server application is deployed to a remote service such
as Render or Heroku and the connection string can be conﬁﬁgure to reference the online remote database running on Atlas
cloud servive. In a new .env ﬁﬁle, declare th代 写data、SQL
代做程序编程语言e following connection string environment variable.
.env
MONGO_CONNECTION_STRING=mongodb://127.0.0.1:27017/kanbas
Install the dotenv library to read conﬁﬁgurations in the local environment.
$ npm install dotenv
Then in index.js, import the dotenv library to read the connection string as shown below.
index.js
import "dotenv/config";
import express from "express";
import mongoose from "mongoose";
...
const CONNECTION_STRING = process.env.MONGO_CONNECTION_STRING || "mongodb://127.0.0.1:27017/kanbas"
mongoose.connect(CONNECTION_STRING);
const app = express();
app.use(cors());
app.use(express.json());
...
app.listen(process.env.PORT || 4000);
3.3 Implementing Mongoose Schemas and Models
Now that we have the collections setup in our database, let's now discuss how to connect and interact with the collections
in the database using the Mongoose library. We'll create Mongoose Schemas and Models so that we can connect and
interact to the database programmatically.
As mentioned earlier, non relational database do not require specifying the structure, or schema of the data stored in
collections like relational databases do. That responsibility has been delegated to the applications using non relational
databases. Mongoose schemas describe the structure of the data being stored in the database and it's used to validate
the data being stored or modiﬁﬁed through the Mongoose library. The schema shown below describes the structure for the
users collection imported earlier. Create the schema in a Users directory in your Node.js projects.Kanbas/Users/schema.js
import mongoose from "mongoose";
const userSchema = new mongoose.Schema({
username: { type: String, required: true, unique: true },
password: { type: String, required: true },
firstName: String,
email: String,
lastName: String,
dob: Date,
role: {
type: String,
enum: ["STUDENT", "FACULTY", "ADMIN", "USER"],
default: "USER",
},
loginId: String,
section: String,
lastActivity: Date,
totalActivity: String,
},
{ collection: "users" }
);
export default userSchema;
// load the mongoose library
// create the schema
// String field that is required and unique
// String field that in required but not unique
// String fields
// with no additional
// configurations
// Date field with no configurations
// String field
// allowed string values
// default value if not provided
// store data in "users" collection
3.4 Implementing Mongoose Models
In earlier sections we demonstrated using the command line client to interact manually with the MongoDB server using
the ﬁﬁnd command. Mongoose models provide similar functionality to interact with MongoDB programmatically instead of
manually. The functions are similar to the ones found in the mongo shell client: ﬁﬁnd(), create(), updateOne(), removeOne(),
etc. In Users/model.js below, create a Mongoose model from the users schema. The functions provided by Mongoose
models are deliberately generic because they can interact with any collection conﬁﬁgured in the schema. In the next
section we'll create a data access object that implements higher level functions speciﬁﬁc to the domain of kanbas.
Kanbas/Users/model.js
import mongoose from "mongoose";
import schema from "./schema.js";
const model = mongoose.model("UserModel", schema);
export default model;
// load mongoose library
// load users schema
// create mongoose model from the schema
// export so it can be used elsewhere
3.5 Retrieving data from Mongo with Mongoose
The Mongoose model created in the previous section provides low level functions such as ﬁﬁnd, create, updateOne, and
deleteOne, that are deliberately vague since they need to be able to operate on any collection. It is good practice to wrap
these low level generic functions into higher level functions that are speciﬁﬁc to the use cases of the speciﬁﬁc projects. For
instance instead of just using the generic ﬁﬁnd() function, we'd prefer something such as ﬁﬁndUsers(), ﬁﬁndUserById() or
ﬁﬁndUserByUsername(). A previous assignment implemented a data access object using arrays declared in ﬁﬁles. This
assignment refactors the DAOs so they use an actual database. The following Users/dao.js re-implements the CRUD
operations for the users collection written in terms of the low level Mongoose model operations.
Kanbas/Users/dao.js
import model from "./model.js";
import db from "../Database/index.js";
export const createUser = (user) => {} // implemented later
export const findAllUsers = () => model.find();
export const findUserById = (userId) => model.findById(userId);
export const findUserByUsername = (username) => model.findOne({ username: username });
export const findUserByCredentials = (username, password) => model.findOne({ username, password });
export const updateUser = (userId, user) => model.updateOne({ _id: userId }, { $set: user });
export const deleteUser = (userId) => model.deleteOne({ _id: userId });3.6 Implementing APIs to interact with MongoDB from a React client application
DAOs implement an interface between an application and the low level database access, providing a high level API to the
rest of the application hiding the details and idiosyncrasies of using a particular database vendor. Likewise routes
implement an interface between the HTTP network world and the JavaScript object and function world by converting a
stream of bits from a network connection request into a set of objects, maps, and function event handlers that participate
in the client/server architecture of a multi tiered application.
3.6.1 Refactoring Account Routes
Previous assignments implemented account routes such as signin and signup shown below. Since the DAO
implementations used data structures imported from the local ﬁﬁle system, the operations were synchronous. Now that the
DAO is interacting with a database, the operations are asynchronous and must be tagged with the async/await keywords
as shown below. Conﬁﬁrm Signin, Signup, and Proﬁﬁle screens work as before. Following the examples below for the signin
and signup functions, add the async keyword to all other router functions and add the await keyword to all calls to dao
functions.
Kanbas/Users/routes.js
import * as dao from "./dao.js";
export default function UserRoutes(app) {
const signin = async (req, res) => {
const { username, password } = req.body;
const currentUser = await dao.findUserByCredentials(username, password);
if (currentUser) {
req.session["currentUser"] = currentUser;
res.json(currentUser);
} else {
res.status(401).json({ message: "Unable to login. Try again later." });
}
};
const signup = async (req, res) => {
const user = await dao.findUserByUsername(req.body.username);
if (user) {
res.status(400).json({ message: "Username already taken" });
return;
}
const currentUser = await dao.createUser(req.body);
req.session["currentUser"] = currentUser;
res.json(currentUser);
};
...
app.post("/api/users/signin", signin);
app.post("/api/users/signup", signup);
}
3.6.2 Retrieving All Documents from MongoDB with Mongoose
DAOs implement high level data operations based on lower level Mongoose models. The Mongoose model ﬁﬁnd function
retrieves all documents from a collection. The ﬁﬁndAllUsers function below uses ﬁﬁnd to retrieve all the users from the
users collection.
Kanbas/Users/dao.js
import model from "./model.js";
export const findAllUsers = () => model.find();
Routes implement RESTful Web APIs that user interface clients can use to interact with server functionality. The route
implemented below uses the ﬁﬁndAll function implemented by the DAO to retrieve all the users from the database. The
route responds with the collection of users retrieved from the database. Conﬁﬁrm the route works by navigating to
http:/ localhost:4000/api/users with your browser.Kanbas/Users/routes.js
import * as dao from "./dao.js";
let currentUser = null;
export default function UserRoutes(app) {
const findAllUsers = async (req, res) => {
const users = await dao.findAllUsers();
res.json(users);
};
app.get("/api/users", findAllUsers);
...
}
Meanwhile in the React user interface application, in src/Kanbas/Account/ client.ts, implement the ﬁﬁndAllUsers function
shown below to send a GET request request to the server and await for the server's response containing an array of users
in the data property.
src/Kanbas/Account/client.ts
import axios from "axios";
const axiosWithCredentials = axios.create({ withCredentials: true });
export const REMOTE_SERVER = process.env.REACT_APP_REMOTE_SERVER;
export const USERS_API = `${REMOTE_SERVER}/api/users`;
export const findAllUsers = async () => {
const response = await axiosWithCredentials.get(USERS_API);
return response.data;
};
...
To display the array of users from the database, refactor the PeopleTable component to accept an optional users
parameter, instead of retrieving the users from the local ﬁﬁle system. Remove any ﬁﬁlters since it's best done at the server.
src/Courses/People/Table.tsx
import React, { useState, useEffect } from "react";
// import * as db from "../../Database";
// import { useParams } from "react-router-dom";
export default function PeopleTable({ users = [] }: { users?: any[] }) {
// const { cid } = useParams();
// const { users, enrollments } = db;
return (


...

{users
.filter((usr) => enrollments.some((enrollment) =>
enrollment.user === usr._id  enrollment.course === cid))
.map((user: any) => ( ... ))}


);}
Create a new Users screen that fetches the users from the database and displays it with the PeopleTable component as
shown below.
src/Kanbas/Account/Users.tsx
import { useState, useEffect } from "react";
import { useParams } from "react-router";
import PeopleTable from "../Courses/People/Table";
import * as client from "./client";
export default function Users() {
const [users, setUsers] = useState([]);
const { uid } = useParams();
const fetchUsers = async () => {
const users = await client.findAllUsers();setUsers(users);
};
useEffect(() => {
fetchUsers();
}, [uid]);
return (

Users


);}
Add a new Users route to the Account screen as shown below.
src/Kanbas/Account/index.tsx
...

 } />
} />
} />
} />
} />

...
Add a Users link to the Accounts Navigation sidebar that navigates to the Users screen only if the logged in user has an
ADMIN role. To test, use Compass to update the role of an existing user or create a new user with ADMIN role, signin as
the ADMIN user, and navigate to the Users screen. Conﬁﬁrm that all users are displayed.
src/Account/Navigation.tsx
import { Link } from "react-router-dom";
import { useSelector } from "react-redux";
export default function AccountNavigation() {
const { currentUser } = useSelector((state: any) => state.accountReducer);
const links = currentUser ? ["Profile"] : ["Signin", "Signup"];
const active = (path: string) => (pathname.includes(path) ? "active" : "");
const { pathname } = useLocation();
return (

{links.map((link) => (
 {link} 
))}
{currentUser  currentUser.role === "ADMIN"  (
 Users  )}

);}
Logged in as an ADMIN, navigate to the new Users screen and conﬁﬁrm it displays all the users as shown below.3.6.3 Retrieving Documents by Predicate from MongoDB with Mongoose
In the User's DAO, implement ﬁﬁndUsersByRole that ﬁﬁlters the users collection by the role property as shown below.
Mongoose model's ﬁﬁnd function takes as argument a JSON object used to pattern match documents in the collection.
The {role: role} object means that documents will be ﬁﬁltered by their role property that matches the value role.
Kanbas/Users/dao.js
export const findUsersByRole = (role) => model.find({ role: role }); // or just model.find({ role })
In the User routes, refactor the ﬁﬁndAllUsers function so that it parses the role from the query string, and then uses the
DAO to retrieve users with that particular role.
Kanbas/Users/routes.js
const findAllUsers = async (req, res) => {
const { role } = req.query;
if (role) {
const users = await dao.findUsersByRole(role);
res.json(users);
return;
}
const users = await dao.findAllUsers();
res.json(users);
};
In the React user interface application, add ﬁﬁndUserByRole in the client so that it encodes the role in the query string of
the URL as shown below.
src/Kanbas/Account/client.ts
export const findUsersByRole = async (role: string) => {
const response = await
axios.get(`${USERS_API}?role=${role}`);
return response.data;
};
In the Users screen, add a dropdown that invokes a ﬁﬁlterUsers event handler function with the selected role. The function
updates a role state variable and requests from the server the list of users ﬁﬁltered by their role. Conﬁﬁrm that selecting
various roles actually ﬁﬁlters the users by their role.
src/Kanbas/Account/Users.tsx
export default function Users() {
const [users, setUsers] = useState([]);
const [role, setRole] = useState("");
const filterUsersByRole = async (role: string) => {
setRole(role);
if (role) {
const users = await client.findUsersByRole(role);
setUsers(users);
} else {
fetchUsers();
}
};
const fetchUsers = async () => { ... };
useEffect(() => { ... }, []);
return (

Users
filterUsersByRole(e.target.value)}
className="form-select float-start w-25 wd-select-role" >
All Roles Students
Assistants Faculty
Administrators
...

);}
Now practice ﬁﬁltering users by their ﬁﬁrst or lastName by creating a regular expression used to pattern match the ﬁﬁrstName
or lastName ﬁﬁelds of the documents in the users collection.
Kanbas/Users/dao.js
export const findUsersByPartialName = (partialName) => {
const regex = new RegExp(partialName, "i"); // 'i' makes it case-insensitive
return model.find({
$or: [{ firstName: { $regex: regex } }, { lastName: { $regex: regex } }],
});
};
In the the User routes, parse a name parameter from the query string and use it to ﬁﬁnd users whose ﬁﬁrst or last names
partially match the name parameter.
Kanbas/Users/routes.js
const findAllUsers = async (req, res) => {
const { role, name } = req.query;
if (role) { ... }
if (name) {
const users = await dao.findUsersByPartialName(name);
res.json(users);
return;
}
const users = await dao.findAllUsers();
res.json(users);
};
In the React client application, implement the ﬁﬁndUserByPartialName client function as shown below which encodes a
name in the query string the server can use to ﬁﬁlter users by their ﬁﬁrst and lastName.
src/Kanbas/Account/client.ts
export const findUsersByPartialName = async (name: string) => {
const response = await axios.get(`${USERS_API}?name=${name}`);
return response.data;
};
In the Users screen, create a new name state variable and corresponding input ﬁﬁeld used to invoke the
ﬁﬁndUsersByPartialName client function and update the users state variable with a subset of users that match the name.
Conﬁﬁrm that typing a name in the input ﬁﬁeld actually ﬁﬁlters the users by their ﬁﬁrst or last name. Note that the current
implementation does not consider a combination of ﬁﬁltering by role and by name. Feel free to explore how you         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
