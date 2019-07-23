# shopping lis
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