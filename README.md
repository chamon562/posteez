## Issues

- Access to XMLHttpRequest at 'http://localhost:8318/posts' from origin 'http://localhost:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
- FIXED: go to package.json and under "private":true, add a proxy and also the other issue was to go to the serer side as well.

```json
"name": "mern-stack-client",
  "version": "0.1.0",
  "private": true,
  "proxy": "https://localhost:8318",

//   "proxy": "https://memories-project-part4.herokuapp.com/",
```

- Whenever changing package.json file rerun terminal
- Then in server side we specified app.use('/posts', postRoutes) before app.use(cors()) and app.use('/posts',postRoutes) needed to be after app.use(cors())

```js
dotenv.config();
import dotenv from "dotenv";
import express from "express";
import bodyParser from "body-parser";
import mongoose from "mongoose";
import cors from "cors";
import postRoutes from "./routes/posts.js";
const app = express();

app.use("/posts", postRoutes);

app.use(bodyParser.json({ limit: "30mb", extended: true }));
app.use(bodyParser.urlencoded({ limit: "30mb", extended: true }));
app.use(cors());

const CONNECTION_URL = process.env.MONGODB_URI;
const PORT = process.env.PORT || 8000;

mongoose
  .connect(CONNECTION_URL, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(() => {
    app.listen(
      PORT,
      console.log(`🐮 Server running on http://localhost:${PORT}`)
    );
  })
  .catch((error) => {
    console.log(error);
  });

const db = mongoose.connection;

db.once("open", () => {
  console.log(`Connected to MongoDB at ${db.host}, port: ${db.port}`);
});
db.on("error", (error) => {
  console.log(`Database error \n ${error}`);
});

// Changed to
dotenv.config();
import dotenv from "dotenv";
import express from "express";
import bodyParser from "body-parser";
import mongoose from "mongoose";
import cors from "cors";
import postRoutes from "./routes/posts.js";
const app = express();

app.use(bodyParser.json({ limit: "30mb", extended: true }));
app.use(bodyParser.urlencoded({ limit: "30mb", extended: true }));
app.use(cors());

// app.use('/posts', postRoutes) here now
app.use("/posts", postRoutes);
const CONNECTION_URL = process.env.MONGODB_URI;
const PORT = process.env.PORT || 8000;

mongoose
  .connect(CONNECTION_URL, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(() => {
    app.listen(
      PORT,
      console.log(`🐮 Server running on http://localhost:${PORT}`)
    );
  })
  .catch((error) => {
    console.log(error);
  });

const db = mongoose.connection;

db.once("open", () => {
  console.log(`Connected to MongoDB at ${db.host}, port: ${db.port}`);
});
db.on("error", (error) => {
  console.log(`Database error \n ${error}`);
});
```

- Cannot destructure property 'data' of '(intermediate value)' as it is undefined
- to FIX must go into the export const fetchPosts function

```js
export const fetchPosts = () => {
  axios.get(url);
};

// change do either
export const fetchPosts = () => axios.get(url);
// ..with the implicit return from your arrow function, I think it should work. Alternatively, you could just explicitly return the result of axios.get:
const fetchPosts = () => {
  return axios.get(url);
};

// https://stackoverflow.com/questions/66340174/cannot-destructure-property-data-of-intermediate-value-as-it-is-undefined
```

## implementing the like post

1. start in server side and in the routes folder/posts.js create a router.patch() request

```js
import express from "express";
import {
  getPosts,
  createPost,
  updatePost,
  deletePost,
  likePost,
} from "../controllers/posts.js";
// this is the route for likeing a post
// why is a patch request if we said that patch request is used for updating?
// that is because liking something is updating it th elikes that the post has
// then add the function of likePost, then import from controllers
router.patch("/:id/likePost", likePost);
```

2. go to controllers folder and posts.js

```js
export const likePost = async (req, res) => {
  const { id } = req.params;
  // check whether its valid
  if (!mongoose.Types.ObjectId.isValid(id))
    return res.stats(404).send("No like with this id");

  // find the post
  const post = await PostMessage.findById(id);
  // for the 2nd parameter remember to pass in our updates so that will be an object
  // increment the likeCount: post.likeCount the post is the post were fetching
  // and then + 1, then with update request we specify the third parameter
  // where we say new: true
  const updatedPost = await PostMessage.findByIdAndUpdate(
    id,
    {
      likeCount: post.likeCount + 1,
    },
    { new: true }
  );

  res.json(updatedPost);
};
```

3. frontend add the api call

- in api folder/index.js

```js
export const likePost = (id) => axios.patch(`${url}/${id}/likePost`);
```

4. frontend go to actions folder/posts.js, create the like action

```js
export const likePost = (id) => async (dispatch) => {
  try {
    // get newly updated post which will be the similar as the update
    // but for the api.likePost(id) only need id not the post because were only liking it
    // we dont have to provide what we want to do with it.
    const { data } = await api.likePost(id);
    dispatch({ type: "UPDATE", payload: data });
  } catch (error) {
    console.log(error);
  }
};
```

5. Go to reducers/posts.js and in there do the logic for liking posts, and the logic is similar as the UPDATE. first we map over the posts, then we check what is the one post that changed. post.\_id === action.payload.\_id or what is the one post that was liked and then return the post with the change action.payload or
   if the post is not liked then just return the post.

```js
// we can copy the same case for UPDATE and rename it like
 case "UPDATE":
      // what is the output method of map it is an array
      // if post._id === action.payload which is the updated post
      // then return the action.payload because it is the updated action.payload else return the regular post
      return posts.map((post) =>
        post._id === action.payload._id ? action.payload : post
      );
    case "LIKE":
      return posts.map((post) =>
        post._id === action.payload._id ? action.payload : post
      );
//  but since they do the same thing we can just put the case under one another like so with LIKE sharing the same logic as UPDATE
case 'UPDATE':
case 'LIKE':
  return posts.map((post)=> post._id === action.payload._id ? action.payload : post)
```

6. then go to Post/Post.js

```js
import {deletePost, LikePost} "../../../actions/posts"
// in the jsx
// dispatch the likePost and pass in the post._id
 <Button size="small" color="primary" onClick={() => dispatch(likePost(post._id))}>
          <Icons.ThumbUpAlt /> Like
        </Button>

// to put a space between the icon and like use &nbsp;
  <Button size="small" color="primary" onClick={() => dispatch(likePost(post._id))}>
          <Icons.ThumbUpAlt /> &nbsp;
          Like {post.likeCount}
        </Button>
```

7. with the current implementation one user can like the post as many times as they want. if we want to give users teh ability to like the post only once per account, that means we will have to work in the full authentication system, register, login, allow users to reset password and create account.

## Redux work flow

// REDUX work flow
// first we get to the actual form
// the form is the actual component in Auth/Auth.js
// once we fill in all the inputs we want to dispatch something
// dispatch immediately has something to do with redux
// their dispatching an action this action is called signup for example
// so were dispatching a signin action and were giving it 2 differnt things fromData and, history

```js
 if(isSignup){
    // do the logic to sign up the user
    // dispatch an action specifically a signup action pass in formData, and history
    // we pass the formData to have it in our databse, and the hisotry object is there
    // to help navigate once something happens
    dispatch(signup(formData, history))
  } else {
    dispatch(signin(formData, history))
    // if were not in the signup do the logic to sign in the user
  }
  };
```

// once we dispatch that we come to our actions/auth.js
// and that is the export const signin = (formdata)

```js
export const signin = (formData, history) => async (dispatch) => {
  // function block get
  // in here we get what we passed into our function which was the formData
  // and history
  try {
    // send data to backend so that it knows to sign in the user
    // right dont have backend endpoints.
    // Log in user
    const { data } = await api.signIn(formData);
    // history will be used to navigate the user to the home page
    history.push("/");
  } catch (error) {
    console.log(error);
  }
};
```

This action makes another call to our api which is the api/index.js

```js
export const signIn = (formData) => API.post("/users/signin", formData);
// this is making a post request saying hey database get me some data and return it to me
```

And we write in our action thats what we do in actions/auth.js

```js
const { data } = await api.signIn(formData);
```

Finally from our action creator we can finally dispatch some things and then were coming into our reducers

```js
// inside of here were getting the profile and setting it to the local storage

// Similar to the posts.js
// import some of the constants
// add these action types AUTH , LOGOUT to the constants folder actionTypes.js
// this helps with mispelling. it will trigger some kind of vs code error if these do not exist
import { AUTH, LOGOUT } from "../constants/actionsTypes";

// reducers are functions
// more specifically functions that are accepting the state and an action
const authReducer = (state = { authData: null }, action) => {
  // so if our action.type is equal to AUTH: what do we want to do
  // if we remember we are getting all the details inside of the
  // action.payload or more specific we called it data
  // so lets first console to see what were getting
  // in the switch case were looking for action.type
  switch (action.type) {
    case AUTH:
      // try to log out action.data to see if we get anything
      // using the option statement operator because that data may not always be there
      console.log(action?.data);
      //   want the token to be saved to the local storage
      // so once we refresh the page the browser will know that were logged in
      // setting our profile with the data filled for the user JSON.stringify({...})
      // and use in the Navbar.js and in there we already have our user set to null
      localStorage.setItem("profile", JSON.stringify({ ...action?.data }));
      return { ...state, authData: action?.data };
    case LOGOUT:
      // this will clear our localStorage .clear
      localStorage.clear();
      // and return the same as above but the authData will stay null
      return { ...state, authData: null };
    default:
      return state;
  }
};
// use in /reducers/index.js
export default authReducer;
```

## Deploy backend to heroku

- Log into heroku (https://dashboard.heroku.com/apps)[url]
- click top right New button and create a new app
  - name your app and click create app
- instruction for deploying in case forget on herokju (https://dashboard.heroku.com/apps/mern-posty/deploy/heroku-git)[url]
- in the server side package.json change the "scripts"{"start": "node index.js"} intead of nodemon
- add a .Procfile in the root of your server folder
  and place in there :
  - web: npm run start
- to know if the server is running go to index.js and create the app.get('/'(req,res) =>{
  res.send('APP IS RUNNING')
  })
  - this is just a message will get in the browser so will know that our app is running.
- MAKE SURE YOU ARE IN SERVER DIRECTORY FOLDER
- now in server terminal type:
  - heroku login
    - it will open a tab on the web browser where you click to login
- or it might say:
  - Clone the repository Use Git to clone mern-posty's source code to your local machine.
  - $ heroku git:clone -a mern-posty
  - $ cd mern-posty
- Create a new Git repository Initialize a git repository in a new or existing directory
  - $ cd my-project/
  - $ git init
  - $ heroku git:remote -a mern-posty
- Deploy your changes, Make some changes to the code you just cloned and deploy them to Heroku using Git.
  - $ git add .
  - $ git commit -am "make it better"
  - $ git push heroku master
- then on the (https://dashboard.heroku.com/apps/mern-posty)[url]
  - click open app
  - shold say APP IS RUNNING because of the app.get we did in the index.js
- now we can take this url and connect our frontend to it (https://mern-posty.herokuapp.com/)[url]
- go back to client side src/api/index.js

```js
// in the baseURL past this is https://mern-posty.herokuapp.com/
const API = axios.create({ baseURL: "https://mern-posty.herokuapp.com/" });
```
- in the public folder add a file called: _redirects
  - inside _redirects file type : 
    - /* /index.html 200
- cd into client side
- build out entire application by typing:
  - npm run build
- a red folder will pop up that says build

- backend deployed on heroku frotnend deployed on netlify
- website https://posteez.netlify.app/posts