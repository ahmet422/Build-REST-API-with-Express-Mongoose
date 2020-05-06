# Build-REST-API-with-Express-Mongoose
Build-REST-API-with-Express-Mongoose

This tutorial will guide you to build a RESTful API with Node.js, Express, and Mongoose with CRUD functionalities. I expect that you have the basic knowledge of Node.js and JavaScript. If you do, you're good to go!

Prerequisites
These software need to be installed on your machine first:

Node.js
MongoDB
Getting Started
The only thing we need to get started with this project is a blank folder with npm package initialized. So, lets create one!

$ mkdir learn-express
$ cd learn-express
$ npm init -y
Now, lets install some useful packages.

$ npm install express mongoose body-parser
Here, we're installing Express for our web framework, mongoose to interact with our MongoDB database, and body-parser to parse our request body.

We can now start to create index.js and create a simple Express server.

// index.js

const express = require("express")
const app = express()

app.listen(5000, () => {
  console.log("Server has started!")
})
We first import our express package that we've just installed. Then, create a new express instance and put it into app variable. This app variable let us do everything we need to configure our REST API, like registering our routes, installing necessary middlewares, and much more.

Try to run our server by running this command below.

$ node index.js
Server has started!
Alternatively, we can setup a new npm script to make our workflow much more easier.

{
  "scripts": {
    "start": "node index.js"
  }
}
Then, we can run our server by executing npm start.

$ npm start
Server has started!
Setup mongoose
Mongoose is the most preferred MongoDB wrapper for Node.js. It allows us to interact with MongoDB database with ease. We can start connecting our server into our MongoDB database.

// index.js

const express = require("express")
const mongoose = require("mongoose")

mongoose
  .connect("mongodb://localhost:27017/acmedb", { useNewUrlParser: true })
    .then(() => {
      const app = express()
      
      app.listen(5000, () => {
        console.log("Server has started!")
      })
    })
Here, we're importing mongoose package and use it to connect into our database called acmedb, but you can name it whatever you want though. If you haven't created that database, don't worry, mongoose will create it for ya.

The connect method returns a promise, so we can wait until it resolved, and run our Express server.

Run the server again, and make sure there is no error.

$ npm start
Server has started!
Now, we have successfully connect our server with the database, now it's time to create our first model.

Mongoose model
In NoSQL world, every single data stored inside a single document. And multiple documents with the same type can be put together inside a collection.

Model is a class, that lets us interact with a specific collection of a database.

Defining a model also requires us to define a schema. Schema is basically tells the model how our document should look like. Even though in NoSQL world, the document schema is flexible, mongoose helps us to keep our data more consistent.

Let's say we have a blog API. So, we obviously going to have a Post model. And the post model has a schema that contains the fields that can be added into a single document. For this example, we will simply have a title and content field.

So, let's add a new folder in our project called models, and create a file called Post.js inside it.

// models/Post.js

const mongoose = require("mongoose") const schema = mongoose.Schema({ title: String, content: String
}) module.exports = mongoose.model("Post", schema)
Here, we're constructing a schema with mongoose.Schema, and define the fields as well as the data types. Then, we create a new model by using the mongoose.model based on the schema that we've just created.

Get all posts
Here, we can create a new file called routes.js which will contains our Express routes.

// routes.js

const express = require("express")
const router = express.Router() module.exports = router
We also need to import express but this time, we want to use the express.Router. It lets us register the routes and use it in our application (in index.js).

Now, we're ready to create our first route in Express that actually do something!

Let's create a route that can get a list of the existing posts.

// index.js

const express = require("express")
const Post = require("./models/Post") // new
const router = express.Router()

// Get all posts
router.get("/posts", async (req, res) => {
  const posts = await Post.find()
  res.send(posts)
})

module.exports = router
Here, we're importing the Post model and create a new GET route with router.get method. This method will accept the endpoint of the route, and the route handler to define what data should be sent to the client. In this case, we're going to fetch all of our posts with the find from our model and send the result with res.send method.

Because fetching documents from the database is asynchronous, we need to use await to wait until the operation is finished. So, don't forget to tag your function as async. Then, after the data is completely fetched, we can send it to the client.

Now, we can install our routes in our index.js.

// index.js

const express = require("express")
const mongoose = require("mongoose")
const routes = require("./routes") // new

mongoose
  .connect("mongodb://localhost:27017/acmedb", { useNewUrlParser: true })
  .then(() => {
    const app = express()
    app.use("/api", routes) // new

    app.listen(5000, () => {
      console.log("Server has started!")
    })
  })

First, we import the ./routes.js file to get all the routes, and register it with app.use method with the prefix of /api, So, all of our posts can be accessed in /api/posts.

Try to run our server and fetch /api/posts, let's see what we got.

$ curl http://localhost:5000/api/posts
[]
Now, we got an empty array from our server. That's because we haven't create any post yet. So, why not create one?

Create Post
To create a post, we need to accept POST requests from /api/posts.

// routes.js

router.post("/posts", async (req, res) => {
  const post = new Post({ title: req.body.title, content: req.body.content })
  await post.save() res.send(post)
})
Here, we're creating a new Post object and populate the fields from the req.body property. The req object contains the client request data, and the body is one of them.

Then, we also need to save our record with the save method. Saving data is also asynchronous, so we need to use async/await syntax.

By default, Express doesn't know how to read the request body. That's why we need to use body-parser to parse our request body into a JavaScript object.

// index.js

const express = require("express")
const mongoose = require("mongoose")
const routes = require("./routes")
const bodyParser = require("body-parser")

mongoose
  .connect("mongodb://localhost:27017/acmedb", { useNewUrlParser: true })
    .then(() => {
      const app = express()
      app.use(bodyParser.json())
      app.use("/api", routes)
      app.listen(5000, () => {
      	console.log("Server has started!")
      })
    })
Here, we're using the body-parser library as a middleware to parse the JSON body so that we can access it via req.body in our route handler.

Let's test out the create post feature that we have just created!

$ curl http://localhost:5000/api/posts \
    -X POST \
    -H "Content-Type: application/json" \
    -d '{"title":"Post 1", "content":"Lorem ipsum"}'
{
    "_v": 0,
    "_id": <OBJECT_ID>,
    "title": "Post 1",
    "content": "Lorem ipsum"
}
You also can test it with Postman as well.

Get individual post
To grab individual post, we need to create a new route with GET method.

// routes.js

router.get("/posts/:id", async (req, res) => {
  const post = await Post.findOne({ _id: req.params.id })
  res.send(post)
})
Here, we're registering a new route with the endpoint of /posts/:id. This is called the URL parameter, it lets us grab the id of our post in our route handler. Because, every single document that we stored into our database has their own uniqe identifier called ObjectID. And we can find it using the findOne method and pass the id from req.params object.

Cool, now try to fetch a single blog post with our HTTP client.

$ curl http://localhost:5000/api/posts/<OBJECT_ID>
{
  "_id": <OBJECT_ID>,
  "title": "Post 1",
  "content": "Lorem ipsum"
}
Looks like its working, but there is one thing though.

If we to the this route and pass the wrong ObjectID, our server is crashed. And the reason why its not working is because when we fetch a single post with an ObjectID that doesn't exist, the promise rejects and our application is stop working.

To prevent this, we can wrap our code with try/catch block, so that we can send a custom error whenever the client request a data that doesn't exist.

// routes.js

router.get("/posts/:id", async (req, res) => {
  try {
    const post = await Post.findOne({ _id: req.params.id })
    res.send(post)
  } catch {
    res.status(404)
    res.send({ error: "Post doesn't exist!" })
  }
})
Now, if we try to fetch a post that doesn't exist, our server still behaves as it should be.

$ curl http://localhost:5000/api/posts/<OBJECT_ID>
{
  "error": "Post doesn't exist!"
}
Update post
Usually, the preferred HTTP method to do an update operation into a single record is PATCH. So, let's create one!

// routes.js

router.patch("/posts/:id", async (req, res) => {
  try {
    const post = await Post.findOne({ _id: req.params.id })

    if (req.body.title) {
      post.title = req.body.title
    }

    if (req.body.content) {
      post.content = req.body.content
    }

    await post.save()
    res.send(post)
  } catch {
    res.status(404)
    res.send({ error: "Post doesn't exist!" })
  }
})
Our update post route relatively similar to the get single post route. We're looking for a post by based on the id, and throw a custom error if the post doesn't exist. But this time, we also update every single field of the post object by populating it with the data provided by the client inside the req.body.

We also want to save our post object with save method, and send the update post data to the client.

Now, we can run a PATCH method to our /api/posts/<OBJECT_ID> endpoint.

$ curl http://localhost:5000/api/posts/<OBJECT_ID> \
    -X PATCH \
    -H "Content-Type: application/json" \
    -d '{"title":"Updated Post", "content":"Updated post content"}'
{
    "__v": 0,
    "_id": <OBJECT_ID>,
    "title": "Updated Post"
    "content": "Updated Post content",
}
Delete post
Finally, our last step is to finish the CRUD feature by add the delete functionality.

// routes.js

router.delete("/posts/:id", async (req, res) => {
  try {
    await Post.deleteOne({ _id: req.params.id })
    res.status(204).send()
  } catch {
    res.status(404)
    res.send({ error: "Post doesn't exist!" })
  }
})
In the delete post route, we basically just run the delete operation directly to the database with deleteOne method and pass the document id. And we return nothing to the user.

$ curl http://localhost:5000/posts/<OBJECT_ID> -X DELETE -I
HTTP/1.0 204 NO CONTENT
...

original source : https://www.codementor.io/@rahmanfadhil/build-rest-api-with-express-mongoose-1538dd6c58?utm_content=posts&utm_source=sendgrid&utm_medium=email&utm_term=post-1538dd6c58&utm_campaign=newsletter2020408
