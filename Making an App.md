# Making an App

### Server-Set Up

1. Create github repo & README

2. NPM Init

3. NPM Install 

   * express (or other server)

   * pg (PostgreSQL: or other DB) ( https://www.postgresql.org/)

   * knex (http://knexjs.org/)

   * body-parser

     <u>Additional/Possible NPM Packages:</u>

   * cookie-parser

   * bcrypt (or other password encription)(https://www.npmjs.com/package/bcrypt)

   * jsonwebtoken (if using web tokens) (https://www.npmjs.com/package/jsonwebtoken)

   * Morgan (marks request time in terminal — optional)

   * dotenv (loades environment veriables from a .env file into process.env)

4. Validation Options:

   * Mocha & Chai (https://mochajs.org/) (http://chaijs.com/)
   * Jasmine (https://jasmine.github.io/)
   * Joi - object schema validation (https://github.com/hapijs/joi)

5. .gitignore file

   * node_modules
   * .env file
   * others

6. Create .env file

7. Migrations - folder

8. Seed - folder

9. create knex.js file

10. create knexfile.js

### Database Set Up

1. Createdb name_dev - for your development

2. Createdb name_test - for testing

3. Update knex.js

   ```const environment = process.env.NODE_ENV || &#39;development&#39;;
   const knexConfig = require('./knexfile')[environment];
   const knex = require('knex')(knexConfig);
   module.exports = knex;
   ```

4. Add db names to knexfile.js

   Example:

   ```
   development: {
   		client: 'pg'
   		connection:'postgress://localhost/dbname_dev'
   	},
   	test: {
   		client: 'pg'
   		connection: 'postgres://localhost/dbname_test'
   	},
   	production: {
   		client: 'pg' (or other db)
   		connection: process.end.DATABASE_URL
   	}
   };
   ```

5. Create migration files - up & down for each table

   Example:

       exports.up = function(knex, Promise) {
         return knex.schema.createTable('users', function(table) {
       	table.increments();
       	table.string('username').notNullable().unique();
       	table.specificType('hashed_password', 'char(60)').notNullable();
       	table.boolean('admin').defaultTo('false');
       	table.timestamps(true, true);
         });
       };
       exports.down = function(knex, Promise) {
         return knex.schema.dropTable('users');
       };

6. Create seed files

   Example:

       exports.seed = function(knex, Promise) {
       
        return knex('users').del()
       .then(function () {
         return Promise.all([
           // Inserts seed entries in an array of objects
           knex('users').insert({
             id: 1,
             username: 'annamefford',
             hashed_password:'$2a$08$GRP5vwq.R2ruRY/r8dH7UOT3PFhJ3MRNxw7eyZjmDkFszq3H4xnLq',
             admin: true
           })
         ]);
       })
       .then(function(){
        return knex.raw("SELECT setval('users_id_seq', (SELECT MAX(id) FROM users))");
         });
       };

### Server

1. Require and use apps.

   Example:

   ```
   const express = require('express');
   const path = require('path');
   app.use(express.static(path.join('public')));
   ```

2. Listen on Port

   Example:

   ```
   const port = process.end.PORT || 3000;

   app.listen(port, () => {
     console.log('Listening on port:', port);
   });
   ```

3. Require and use files

   Example:

   ```
   const users = require('./routes/users');
   app.use(users);
   ```

4. Export app

   Example:

   ```
   module.exports = app;
   ```

   ​

### Create Route File(s)

1. Require express, express.Router(), knex, and others as needed.

2. app.get 

   Example:

   ```
   router.get('/users/:userid', (req, res, next) => {
     let userId = parseInt(req.params.userid);

      knex.from('users')
      .where({
        users_id: userId
      })
     .then((results) => {
       res.send(results);
       console.log(results);
     })
     .catch((err) => {
       next(err);
     });
   });
   ```

3. app.post

   ```
   router.post('/users/:id', ev(validations.post), (req, res, next) => {
     let userId = req.params.id;
    
       knex('users')
         .insert({
           user_id: userId,
           name: req.body.name
         })
         .then(
           res.send('update comment')
         )
         .catch((err) => {
           next(err);
         });
   });
   ```

4. app.patch

   ```
   router.patch('/users/:id', (req, res, next) => {
     var id = req.params.id;
     knex('users')
     .where({
       id: id
     })
     .first()
     .then((user) => {
       if (!user || !req.body.user){
         return next();
       } return knex('users')
       .update({
         name: req.body.name
       }, '*')
       .where({'id': id});
     })
     .then((users) => {
       res.send(users[0]);
     })
     .catch ((err) => {
       next(err);
     });
   });
   ```

   ​

5. app.delete

   ```
   router.delete('/users/:id', (req, res, next) => {
     var id = req.params.id;
     let user;

     knex('users')
     .where('id', id)
     .first()
     .then ((result) => {
       if (!result) {
         return next();
       }
     user = result;

     return knex('users')
       .del()
       .where('id', id);
     })
     .then(() => {
       delete user.id;
       res.send(user);
     })
     .catch((err) => {
       next(err);
     });
   });
   ```

6. Export routes

   Example:

   ``` 
   module.exports = router;
   ```

   ​

### Create Test & Validation Files

1. Create validations for each route

   Example:

   ```
   const Joi = require('joi');
   module.exports.post = {
     body: {
       user_id: Joi.string()
       .label('user_id')
       .required()
       .trim(),
       
       observation_id: Joi.string()
       .label("observation_id")
       .required()
       .trim(),
       
       comment: Joi.string()
       .label("comment")
       .required()
       .trim(),
     }
   };
   ```

2. Create tests as needed

   Example:

   ```
   const code = require('../main.js'); //this is the js file being tested
   const expect = require('chai').expect; //require chai

   describe("Greeting", () => {
     it("Should accept one argument", () => {
       expect(code.greeting()).to.equal('invalid argument length');
       expect(code.greeting("arg1", "arg2")).to.equal('invalid argument length');
     });
       it ("Should accept a string", () => {
       expect(code.greeting(123)).to.equal("invalid input type");
       expect(code.greeting(["hello"])).to.equal("invalid input type");
       expect(code.greeting({"object":"item"})).to.equal("invalid input type");
     });
   ```

   Example of code being tested:

   ```
   greeting(arg){
       if (arguments.length !== 1){
         return ("invalid argument length");
       } else if (typeof arg !== 'string'){
         return ("invalid input type");
       }
       return "Hello " + arg;
     }
   ```

### Client Side Files

1. HTML (index.html)

2. CSS (styles.css)

3. JS File

4. Images File

5. Include any Libaries/Frameworks

   * Materialize or Bootstrap (http://materializecss.com/) (http://getbootstrap.com/)
   * JQuery (https://jquery.com/)
   * Others as needed (there are so many out there!)

6. Link to your JS in your HTML Files

   Example:

   ```
   <script src="js/main.js"></script>
   ```

   I recommend creating a quick `console.log("Made it to the js file");` to make sure your script is running.

7. Create AJAX Requests for Routes as needed *(Note: this is in your js file. Using $.ajax is a JQuery function so you need JQuery in your app)*

   Example:

   ```
   function getComments(data) {
           $.ajax({
               url: '/users',
               contentType: "application/json",
               dataType: 'json',
               data: JSON.stringify(data),
               type: 'get',
               success: function(data) {
                   console.log('success'); //do something here if success
               },
               error: function() {
                   console.log('error'); //do something here if error
               }
           });
       }
   ```