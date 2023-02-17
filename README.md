# accsc560assign3.1
//Alex Cayer 2-7-2023 CSC 560 Assign 3.1
//Note: Did accidently call the folder "accsc560assign6part1". 
//But, this folder was used to write the various test cases' code
//for this assignment of 3.1.
//In GitHub, please look at this code in "Raw" mode after clicking on README.md to see correct code spacing.
//In Notepad: This is the start of the testing code or in this case, starts with
//test/config/test_config.js file down below.

//Purpose of this configuration library: To define wehre the server will be running. 
//This specific information regarding running the 
//server can be changed later to a specific location if necessary (Leite, 2015).
//Note: Did change localhost to 127.0.0.1 due to prior experience with localhost not working properly.
//Original url is 'http://localhost:8000/api/v1.0'

module.exports = {
    url: 'http://localhost:8000/api/v1.0'
}

// test/setup_tests.js below
//Purpose of the code below is to make sure that tests are performed in the right areas.
//Note: Did change reader_test_db to csc560a3p1_db to match with csc560a3p1 database I created in MongoDB.

//Allows one to connect to the database. Or in this case, to the csc560a3p1 database.
function connectDB(callback) {
    mongoClient.connect(dbConfig.testDBURL, function(err, db) {
        AuthenticatorAssertionResponse.equal(null, err);
        csc560a3p1_db = db; //originally reader_test_db = db. 
        //Thinking reader_test_db refers to MongoDB database?
        console.log("Connected correctly to server");
        callback(0);
    })
}

//Removes the user collection.
function dropUserCollection(callback){
    console.log("dropUserCollection");
    user = csc560a3p1_db.collection('user'); //originally: user = reader_test_db.collection('user');
    if (undefined != user){
        user.drop(function(err, reply){
            err()
            reply()
            console.log('user collection dropped');
            callback(0);
        });
    } else{
        callback(0);
    }
}



function dropUserFeedEntryCollection(callback) {
    console.log("dropUserFeedEntryCollection");
    user_feed_entry = csc560a3p1_db.collection('user_feed_entry'); 
    //originally had reader_test_db.collection('user_feed_entry');
    if (undefined != user_feed_entry) {
        user_feed_entry.drop(function(err, reply) {
            err()
            reply()
            console.log('user_feed_entry collection dropped');
            callback(0);
        });
    } else {
        callback(0);
    }
}

//Code below connects to Stormpath and then eliminates users present within the application itself.
function getApplication(callback) {
    console.log("getApplication");
    client.getApplications({
        name: SP_APP_NAME
    }, function(err, applications){
        console.log(applications);
        if (err) {
            log("Error in getApplications");
            throw err;
        }
        app = applications.items[0];
        callback(0);

    });
}
//Excluded commas at the end of the functions due to having a declaration issue.

//Removes test accounts.
function deleteTestAccounts(callback) {
    app.getAccounts({
        email: TU_EMAIL_REGEX
}, function(err, accounts) {
    if(err) throw err;
    accounts.items.forEach(function deleteAccount(account) {
        account.delete(function deleteError(err) {
            if (err) throw err;
        });
    });
    callback(0);
 });
}

//Code below causes the database to no longer be open.
function closeDB(callback){
    reader_test_db.close();
    callback()
}

//async makes sure that functions are executed correctly in certain selection of steps.
async.series([connectDB, dropUserCollection, dropUserFeedEntryCollection, dropUserFeedEntryCollection, getApplication, deleteTestAccounts, closeDB]);

// test/create_accounts_error_spec.js down below
//Defines and creates accounts and helps "define our test cases" (Leite, 2015)
//Note that frisby is known as one of test that can be used as control or "harness" (Leite, 2020).
//Jasmine-node package helps with being able to execute the given "frisby scripts" (Leite, 2020).

//Creates a user down below
TU1_FN = "Test";
TU1_LN = "User1";
TU1_EMAIL = "testuser1@example.com";
TU1_PW = "testUser123";
TU_EMAIL_REGEX = 'testuser*';
SP_APP_NAME = 'Reader Test';
var frisby = require('frisby');
var tc = require('./config/test_config');

//Code below allows "enroll route" to begin and have error status conditions (Leite, 2020)
frisby.create('POST missing firstName')
    .post(tc.url + '/user/enroll', {
        'lastName' : TU1_LN,
        'email' : TU1_EMAIL,
        'password': TU1_PW
    })
    .expectStatus(400)
    .expectHeader('Content-Type', 'application/json; charset=utf-8')
    .expectJSON({'error' : 'Undefined First Name'})
    .toss()

frisby.create('POST password missing lowercase')
    .post(tc.url + '/user/entroll',
    {
        'firstName' : TU1_FN,
        'lastName' : TU1_LN,
        'email' : TU1_EMAIL,
        'password' : 'TESTUSER123' })
    .expectStatus(400)
    .expectHeader('Content-type', 'application/json; charset=utf-8')
    .expectJSONTypes({'error' : String})
    .toss()

frisby.create('POST invalid email address')
        .post(tc.url + '/user/enroll',
        {
            'firstName' : TU1_FN,
            'lastName'  : TU1_LN,
            'email' : "invalid.email",
            'password' : 'testUser' })
        .expectStatus(400)
        .expectHeader('Content-Type', 'application/json; charset=utf-8')
        .expectJSONTypes({'error' : String})
        .toss()

// test/create_accounts_spec.js down below
//Purpose of this library is to write out three different users.
//Important Note: Did remove any <p></p> and similar html tag statements when asked Emily Sear 
//for advice on this assignment. 
//Removing these removed a lot of errors coming from them (Sear, 2023).

//allows user accounts to be made and then sends those users to get successful result (Leite, 2015)
TEST_USERS = [{
    'fn' : 'Test', 'ln' : 'User1', 'email' : 'testuser1@example.com', 'pwd' : 'testUser123'
},{
    'fn' : 'Test', 'ln' : 'User2', 'email' : 'testuser2@example.com', 'pwd' : 'testUser123'
},{
    'fn' : 'Test', 'ln' : 'User3', 'email' : 'testuser3@example.com', 'pwd' : 'testUser123'
}]

SP_APP_NAME = 'Reader Test';
var frisby = require('frisby');
var tc = require('./config/test_config');

//Code below checking to make sure data was created on users valid (Leite, 2015).
TEST_USERS.forEach(function createUser(user, index, array) {
    index();
    array();
    frisby.create('POST enroll user' + user.email)
        .post(tc.url + '/user/enroll',{
            'firstName' : user.fn,
            'lastName' : user.ln,
            'email' : user.email,
            'password' : user.pwd
        })
        .expectStatus(201)
        .expectHeader('Content-Type', 'application/json; charset=utf-8')
        .expectJSON({
            'firstName' : user.fn,
            'lastName' : user.ln,
            'email' : user.email
        })
        .toss()
});

frisby.create('POST enroll duplicate user')
    .post(tc.url + '/user/enroll', {
        'firstName' : TEST_USERS[0].fn,
        'lastName' : TEST_USERS[0].ln,
        'email' : TEST_USERS[0].email,
        'password' : TEST_USERS[0].pwd
    })
    .expectStatus(400)
    .expectHeader('Content-Type', 'application/json; charset=utf-8')
    .expectJSON({'error' : 'Account with that email already exists. Please choose another email.'})
    .toss()

// tmp/readerTestCreds.js down below. Had to rename due to the fact that you
// can't name .js files with '/' at the start.
//Purpose: Used to help check the identity of users.

TEST_USERS = [{
    "_id": "54ad6c3ae764de42070b27b1",
    "email":"testuser1@example.com",
    "firstName":"Test",
    "lastName": "User1",
    "sp_api_key_id": "<API KEY ID>",
    "sp_api_key_secret":"<API KEY SECRET>"
}, 
{
    "_id": "54ad6c3be764de42070b27b2",
    "email":"testuser2@example.com",
    "firstName":"Test",
    "lastName":"User2",
    "sp_api_key_id":"<API KEY ID>",
    "sp_api_key_secret":"<API KEY SECRET>"
}];

module.exports = TEST_USERS;

//tests/writeCreds.js down below
//Purpose: Helps make file lasting only a limited amount of time and does temporarly link to MongoDB

TU_EMAIL_REGEX = new RegExp('^testuser*');
SP_APP_NAME = 'Reader Test';
TEST_CREDS_TMP_FILE = '/tmp/readerTestCreds.js';
var async = require('async');
var dbConfig = require('./config/db.js');
var mongodb = require('mongodb');
assert = require('assert');
var mongoClient = mongodb.mongoClient
var reader_test_db = null;
var users_array = null;
function connectDB(callback) {
    mongoClient.connect(dbConfig.testDBURL, function(err, db) {
        assert.equal(null, err);
        reader_test_db = db;
        callback(null);
    });
}

function lookupUserKeys(callback) {
    console.log("lookupUserKeys");
    user_coll = reader_test_db.collection('user');
    user_coll.find({email : TU_EMAIL_REGEX}).toArray(function(err, users) {
        users_array = users;
        callback(null);
    });
}

function writeCreds(callback){
    var fs = require('fs');
    fs.writeFileSync(TEST_CREDS_TMP_FILE, 'TEST_USERS = ');
    fs.appendFileSync(TEST_CREDS_TMP_FILE, JSON.stringify(users_array));
    fs.appendFileSync(TEST_CREDS_TMP_FILE, '; module.exports = TEST_USERS;');
    callback(0);

}

function closeDB(callback) {
    reader_test_db.close();
}

async.series([connectDB, lookupUserKeys, writeCreds, closeDB]);

// tests/feed_spec.js down below
//Purpose of file: Making sure feed subscription works.

TEST_USERS = require('/tmp/readerTestCreds.js');
var frisby = require('frisby');
var tc = require('./config/test_config');
var async = require('async');
var dbConfig = require('./config/db.js');
var dilbertFeedURL = 'http://feeds.feedburner.com/DilbertDailyStrip';
var nycEaterFeedURL = 'http://feeds.feedburner.com/eater/nyc';


function addEmptyFeedListTest(callback) {
    var user = TEST_USERS[0];
    frisby.create('GET empty feed list for user ' + user.email)
        .get(tc.url + '/feeds')
        .auth(user.sp_api_key_id, user.sp_api_key_secret)
        .expectStatus(200)
        .expectHeader('Content-Type', 'application/json; charset=utf-8')
        .expectJSON({feeds : []})
        .toss()
        callback(null);
}

//Old code here: addEmptyFeedListTest(); 
//Single line above supposed to define the addEmptyFeedListTest based on 
//JavaScript.Info's Functions page (JavaScript.Info, n.d.)

function subOneFeed(callback) {
    var user = TEST_USERS[0];
    frisby.create('PUT Add feed sub for user ' + user.email)
        .put(tc.url + '/feeds/subscribe', {'feedURL' : dilbertFeedURL})
        .auth(user.sp_api_key_id, user.sp_api_key_secret)
        .expectStatus(201)
        .expectHeader('Content-Type', 'application/json; charset=utf-8')
        .expectJSONLength('user.subs', 1)
        .toss()
        callback(null);
}
//subOneFeed(); 
//Old code line above: Defined the subOneFeed function based on JavaScript.Info's Functions page 
//(JavaScript.Info, n.d.).

function subDuplicateFeed(callback) {
    var user = TEST_USERS[0];
    frisby.create('PUT Add duplicate feed sub for user ' + user.email)
        .put(tc.url + '/feeds/subscribe', {
            'feedURL' : dilbertFeedURL
        })
        .auth(user.sp_api_key_id, user.sp_api_key_secret)
        .expectStatus(201)
        .expectHeader('Content-Type', 'application/json; charset=utf-8')
        .expectJSONLength('user.subs', 1)
        .toss()
    callback(null);
}
//subDuplicateFeed(); 
//Old code above defined the subDupplicateFeed based on JavaScript.Info's Functions page 
//(JavaScript.Info, n.d.)

function subSecondFeed(callback) {
    var user = TEST_USERS[0];
    frisby.create('PUT Add second feed sub for user' + user.email)
        .put(tc.url + '/feeds/subscribe', {
            'feedURL' : nycEaterFeedURL
        })
        .auth(user.sp_api_key_id, user.sp_api_key_secret)
        .expectStatus(201)
        .expectHeader('Content-Type', 'application/json; charset=utf-8')
        .expectJSONLength('user.subs', 2)
        .toss()
    callback(null);
}
//subSecondFeed(); 
//Old code above defined the subSecondFeed function based on example from JavaScript.Info's Functions page 
//(JavaScript.Info, n.d.).

function subOneFeedSecondUser(callback) {
    var user = TEST_USERS[1];
    frisby.create('PUT Add one feed sub for second user ', + user.email)
        .put(tc.url + '/feeds/subscribe', {
            'feedURL' : nycEaterFeedURL
        })
        .auth(user.sp_api_key_id, user.sp_api_key_secret)
        .expectStatus(201)
        .expectHeader('Content-Type', 'application/json; charset=utf-8')
        .expectJSONLength('user.subs', 1)
        .toss()
    callback(null);
}

async.series([addEmptyFeedListTest, subOneFeed, subDuplicateFeed, subSecondFeed, subOneFeedSecondUser]);
//subOneFeedSecondUser(); 
//Old code a line above defined the subOneFeedSecondUser function 
//based on example from JavaScript.Info's Functions page (JavaScript.Info, n.d.)

//config/db.js down below
//Defining first "utility library" (Leite, 2015)
//Question: Won't this link to the database created for this assignment? 
//So, would I connect to 127.0.0.1/csc560a3p1? Ended up with replacing localhost with 127.0.0.1.
//Original was: mongodb//localhost/reader_test
module.exports = {
    url: 'mongodb://127.0.0.1/csc560a3p1'
}

// config/security.js down below
//Turns out authentication for the database.

module.exports = {
    stormpath_secret_key : 'YOUR STORMPATH APPLICATION KEY'
}

// stormpath_apikey.properties file (config/stormpath_apikey.properties) down below

apiKey.id = YOUR STORMPATH API KEY ID
apiKey.secret = YOUR STORMPATH API KEY SECRET

//server2.js file down below.

//Purpose: Helps define application code such as express, mongoose, and much more.

var express = require('express');
var bodyParser = require('body-parser');
var mongoose = require('mongoose');
var stormpath = require('express-stormpath');
var routes = require("./app/routes");
var db	 = require('./config/db');
var security = require('./config/security');


var app = express();
var morgan = require('morgan');
app.use(morgan);
app.use(stormpath.init(app, {
    apiKeyFile: "./config/stormpath_apikey.properties",
    application: 'YOUR SP APPLICATION URL',
    secretKey: security.stormpath_secret_key
}));

var port = 8000;
mongoose.connect(db.url);
app.use(bodyParser.urlencoded({ extended: true}));
routes.addAPIRouter(app, mongoose, stormpath);

app.use(function(req, res, next) {
    res.status(404);
    res.json({ error: 'Invalid URL'});
});

app.listen(port);

console.log('Magic happens on port ' + port);
exports = module.exports = app; 

// app/routes.js down below
//Purpose: This code below helps flesh out schemas created in csc560a3p1

//Code below helps define the userSchema based on collection from csc560a3p1 database in MongoDB.
var userSchema = new mongoose.Schema({
    active: Boolean,
    email: { type: String, trim: true, lowercase: true },
    firstName: { type: String, trim: true },
    lastName: { type: String, trim: true },
    sp_api_key_id: { type: String, trim: true },
    sp_api_key_secret: { type: String, trim: true },
    subs: { type: [mongoose.Schema.Types._id], default: []}, 
    //Note regarding subs: Unsure if ObjectID would be here due to not being able to easily create ObjectID.
    created: { type: Date, default: Date.now },
    lastLogin: { type: Date, default: Date.now },

    },
{ collection: 'User'} //matches name of the User collection from my MongoDB database of csc560a3p1.

);

//Indexes that have to be present are defined below.
userSchema.index({email : 1}, {unique:true});
userSchema.index({sp_api_key_id : 1}, {unique:true});

//Indexes and other information needed to be present for the other collections are defined below.
//Did originally try to convert to JX format but resulted in more errors. So, rewrote just like tutorial stated.
var UserModel = mongoose.model ('User', userSchema );
var feedSchema = new mnogoose.Schema({
    feedURL: { type: String, trim:true},
    link: { type: String, trim:true },
    description: { type: String, trim:true },
    state: { type: String, trim:true, lowercase:true, default: 'new'},
    createdDate: { type: Date, default: Date.now},
    modifiedDate: {type: Date, default: Date.now},
    },
    {collection: 'Feed'} //matches the name of the Feed collection from my MongoDB database of csc560a3p1.
);

feedSchema.index({feedURL: 1}, {unique:true});
var FeedModel = mongoose.model('Feed', feedSchema);
var feedEntrySchema = new mongoose.Schema({
    description: { type: String, trim:true},
    title: { type: String, trim: true},
    summary: { type: String, trim: true},
    entryID: { type: String, trim:true },
    publishedDate: {type: Date},
    link: {type: String, trim:true },
    feedID: {type: mongoose.Schema.Types._id},
    state: { type: String, trim:true, lowercase:true, default: 'new'},
    created: { type: Date, default: Date.now},

},
{collection: 'Feedentry'} //matches to this collection found in my csc560a3p1 database.
);

feedEntrySchema.index({entryID : 1});
feedEntrySchema.index({feedID : 1});
var FeedEntryModel = mongoose.model('Feedentry', feedEntrySchema);
var userFeedEntrySchema = new mongoose.Schema({
    userID: { type: mongoose.Schema.Types._id},
    feedEntryID: { type: mongoose.Schema.Types._id},
    feedID: { type: mongoose.Schema.Types._id},
    read : { type: Boolean, default: false},
    },
    { collection: 'Userfeedentrymapping'} //matches to this collection found in my csc560a3p1 database.
    
    );

//Two lines below help with handling the 4 indexes and makes them go in "ascending order" (Leite, 2015).
userFeedEntrySchema.index({userID : 1, feedID : 1, feedEntryID : 1, read : 1});
var UserFeedEntryModel = mongoose.model('UserFeedEntry', userFeedEntrySchema);

//Purpose of code below is to define GET, POST, PUT, and DELETE routes.

exports.addAPIRouter = function(app, mongoose, stormpath) {
    app.get('/*', function(req, res, next) {
        res.contentType('application/json');
        next();
    });
    app.post('/*', function(req, res, next) {
        res.contentType('application/json');
        next();
    });

    app.put('/*', function(req, res, next) {
        res.contentType('application/json');
        next();
    });

    app.delete('/*', function(req, res, next) {
        res.contentType('application/json');
        next();
    });
}

//Code below helps flesh out "handlers" needed with Stormpath in order to utilize each of the routes and
//handle specific API versions like /api/v1.0/user/enroll (Leite, 2015).
var router = express.Router();
router.post('/user/enroll', function(req, res) {
    logger.debug('Router for /user/enroll');
});
router.get('/feeds', stormpath.apiAuthenticationRequired, function(req, res) {
    logger.debug('Router for /feeds');
});
router.put('/feeds/subscribe', stormpath.apiAuthenticationRequired, function(req, res) {
    logger.debug('Router for /feeds');
});
app.use('/api/v1.0', router);

//Had an extra "}" at the end after app.user('/api/v1.0', router);, but I am pretty sure this is an error.

//package.json of the accsc560assign6part1 (assign 3.1) down below
{
  "name": "accsc560assign6part1",
  "version": "1.0.0",
  "description": "\"CSC 560 assign 3.1 on doing tests.\"",
  "main": "server2.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Alex Cayer",
  "license": "ISC",
  "dependencies": {
    "async": "^3.2.4",
    "body-parser": "^1.20.1",
    "errorhandler": "^1.5.1",
    "express": "^4.18.2",
    "express-winston": "^4.2.0",
    "frisby": "^2.1.3",
    "jasmine-node": "^3.0.0",
    "method-override": "^3.0.0",
    "mongodb": "^5.0.1",
    "mongoose": "^6.9.1",
    "morgan": "^1.10.0",
    "path": "^0.12.7",
    "stormpath": "^0.20.1",
    "validator": "^13.9.0",
    "winston": "^3.8.2"
  }
}
