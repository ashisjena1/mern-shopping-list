#MERN Shopping List
```
Shopping list app built with the MERN stack along with Redux for state management, Reactstrap and react-transition-group.
```

# Quick Start
Add your MONGO_URI to the default.json file. Make sure you set an env var for that and the jwtSecret on deployment

```
# Install dependencies for server
npm install

# Install dependencies for client
npm run client-install

# Run the client & server with concurrently
npm run dev

# Run the Express server only
npm run server

# Run the React client only
npm run client

# Server runs on http://localhost:5000 and client on http://localhost:3000
```


# shopping list
## steps

### Start with backend
1. npm init
2. Install Dependecies
```
npm i express body-parser mongoose concurrently

concurrently can run more than one script at a time(so we can run server and client at a time)
```

3. configure server.js
```
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');

const items = require('./routes/api/items');

const app = express();

app.use(bodyParser.json());

// DB Config
const db = require('./config/keys').mongoURI;

// Connect to Mongo
mongoose
  .connect(db, { 
    useNewUrlParser: true,
    useCreateIndex: true
  }) // Adding new mongo url parser
  .then(() => console.log('MongoDB Connected...'))
  .catch(err => console.log(err));


// Use Routes
app.use('/api/items',items);

const port = process.env.PORT || 5000;

app.listen(port, () => console.log(`Server started on port ${port}`));
```

4. create model
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Create Schema
const ItemSchema = new Schema({
  name: {
    type: String,
    required: true
  },
  date: {
    type: Date,
    default: Date.now
  }
});

module.exports = Item = mongoose.model('item', ItemSchema);
```

5. create api routes
```
const express = require('express');
const router = express.Router();

// Item Model
const Item = require('../../models/Items');

// @route   GET api/items
// @desc    Get All Items
// @access  Public
router.get('/', (req, res) => {
    Item.find()
      .sort({ date: -1 })
      .then(items => res.json(items));
});

// @route   POST api/items
// @desc    Create An Item
// @access  Public
router.post('/', (req, res) => {
    const newItem = new Item({
      name: req.body.name
    });
  
    newItem.save().then(item => res.json(item));
});

// @route   DELETE api/items/:id
// @desc    Delete A Item
// @access  Public
router.delete('/:id', (req, res) => {
    Item.findById(req.params.id)
      .then(item => item.remove().then(() => res.json({ success: true })))
      .catch(err => res.status(404).json({ success: false }));
});
  
  
module.exports = router;
```

### Frontend
1. mkdir client
2. cd 
3. npm i -g create-react-app
4. create-react-app .
5. add below line in package.json
```
"proxy": "http://localhost:5000"
```

6. Install frontend dependecies
```
npm i bootstrap reactstrap uuid react-transition-group
```
7. Install redux and react-redux
```
npm i react react-redux redux-thunk
```
8. Create store
```
import { createStore, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers';

const initialState = {};

const middleware = [thunk];

// createStore(reducer,state,middleware)
const store = createStore(rootReducer, initialState, compose(
    applyMiddleware(...middleware),
    window.__REDUX_DEVTOOLS_EXTENSION__  && window.__REDUX_DEVTOOLS_EXTENSION__()
));

export default store;
```

9. add reducers
itemReducers
```
import uuid from 'uuid';
import { GET_ITEMS, ADD_ITEM,DELETE_ITEM } from '../actions/types';

const initialState = {
    items: [
        { id: uuid(), name:'Egg' },
        { id: uuid(), name:'Milk' },
        { id: uuid(), name:'Steak' },
        { id: uuid(), name:'Water' }
    ]
}

export default function(state = initialState, action){
    switch(action.type){
        case GET_ITEMS:
            return{
                ...state
            }
        case DELETE_ITEM:
            return{
                ...state,
                items: state.items.filter( item => item.id !== action.payload )
            }
        case ADD_ITEM:
            return{
                ...state,
                items: [action.payload, ...state.items]
            }
        default:
            return state
    }
}
```

rootReducer
```
import { combineReducers } from 'redux';
import itemReducer from './itemReducer';

export default combineReducers({
    item: itemReducer
});
```

10. add actions
itemAction
```
import { GET_ITEMS, ADD_ITEM, DELETE_ITEM } from '../actions/types';

export const getItems = () => {
    return {
        type: GET_ITEMS
    };
}; 

export const deleteItem = id => {
    return {
        type: DELETE_ITEM,
        payload: id
    };
};

export const addItem = item => {
    return {
        type: ADD_ITEM,
        payload: item
    };
};
```

11. connect with backend
```
npm i axios
```

itemReducer.js
```
import { GET_ITEMS, ADD_ITEM,DELETE_ITEM,ITEMS_LOADING } from '../actions/types';

const initialState = {
    items: [],
    loading: false
};

export default function(state = initialState, action){
    switch(action.type){
        case GET_ITEMS:
            return{
                ...state,
                items: action.payload,
                loading: false
            };
        case DELETE_ITEM:
            return{
                ...state,
                items: state.items.filter( item => item._id !== action.payload )
            }
        case ADD_ITEM:
            return{
                ...state,
                items: [action.payload, ...state.items]
            }
        case ITEMS_LOADING:
            return {
                ...state,
                loading: true
            }
        default:
            return state
    }
}
```

itemActions.js
```
import axios from 'axios';
import { GET_ITEMS, ADD_ITEM, DELETE_ITEM, ITEMS_LOADING } from '../actions/types';

export const getItems = () => dispatch => {
    dispatch(setItemsLoading());
    axios
        .get('/api/items')
        .then(res => 
            dispatch({
                type: GET_ITEMS,
                payload:res.data
            })
        )
}; 

export const addItem = item => dispatch => {
    dispatch(setItemsLoading());
    axios
        .post('/api/items', item)
        .then(res =>
            dispatch({
                type: ADD_ITEM,
                payload: res.data
            })    
        )
};

export const deleteItem = id => dispatch => {
    axios
        .delete(`/api/items/${id}`)
        .then(res => 
            dispatch({
                type: DELETE_ITEM,
                payload: id
            })
        )
};

export const setItemsLoading = () => {
    return {
        type: ITEMS_LOADING
    }
}
```
### configure for frontend and backend
npm i concurrent
```
"scripts": {
    "client-install": "npm instaall --prefix client",
    "start" : "node server.js",
    "server": "nodemon server.js",
    "client": "npm start --prefix client",
    "dev":"concurrently \"npm run server\" \"npm run client\""
},
```

### uploading to heroku
steps

1. add below command in script
```
"herok-postbuild": "NPM_CONFIG_PRODUCTION=false nppm install --prefix client && npm run build --prefix client"
```

2. add code in server.js for production
```
// Serve static assets if in production
if(process.env.NODE_ENV === 'production'){
  app.use(express.static('client/build'));

  app.get('*',(req,rees) => {
    res.sendFile(path.resolve(__dirname, 'client', 'build', 'index.html'))
  });
} 
```

3. install heroku-cli
4. heroku login
5. heroku create
6. go to heroku dashboard then add remote git
```
heroku git:remote -a afternoon-oasis-93789
```
7. commit and push to heroku
```
git commit -am 'Version 1.0.0
git push heroku master
```


to add confi file in project
```
npm i config

create a folder called config
add a file default.json
put the secret data inside it
```