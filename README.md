# GroceryApp-Part2

 Creating a Web App Using Angular 4 MongoDB
 
In this tutorial, you’ll see how to create a web app using Angular 4 & MongoDB. In the first part of this tutorial, you learnt how to get started with creating a web app using Angular 4. In this part of the tutorial, you’ll learn how to write the back end for the Angular 4 grocery manager web app using Node.js and MongoDB.

Source code from this tutorial is available on GitHub.

Other tutorials in Angular tutorial series.

How To Make Asynchronous Calls In Angular
How to Implement Auto Complete In Angular
Creating A Web App Using Angular And MongoDB
How To Create A Web App Using Angular And Firebase
How To Make Login Page Using Angular And Firebase
Getting Started

Let’s get started by cloning the source code from the first part of the tutorial.

git clone https://github.com/jay3dec/GroceryApp.git
Copy
Navigate to the client code and install the required dependencies.

cd GroceryApp/client
npm install
Copy
Once the dependencies are installed, start the server and you should have the application running.

ng serve
Copy
You should have the application running on http://localhost:4200/.

Setting Up the Node Rest API

You’ll be using Express, Node.js web application framework for creating the APIs for your Angular Grocery Web App. Let’ get started by installing express for our application.

npm install express --save
Copy

 
Once you have express installed in your project, let’s start by creating a web server for serving the APIs. Add the following code to the server/app.js file:

const express = require('express')
const app = express()

app.get('/', function (req, res) {
  res.send('Welcome to Grocery Service APIs.')
})

app.listen(3000, function () {
  console.log('Grocery Web app service listening on port 3000!')
})
Copy
Save the above changes and start the server by running it using Node.

node app.js
Copy
You should have the application APIs running at http://localhost:3000.

Creating the Add Grocery API

You’ll start by creating a separate file for keeping the grocery services. Let’s start by creating a separate file called services/groceryService.js. You’ll be following the ES 6 standards when writing the JavaScript code, so create a GroceryService class inside the groceryService.js file. It should have two function defined, addGrocery and getGrocery for use inside the Angular Grocery App. Here is how the groceryService.js file looks:

class GroceryService{
    constructor(req, res){
        this.req = req
        this.res = res
    }

    addGrocery(){
        
    }
    getGrocery(){

    }
}
module.exports = GroceryService
Copy
Next let’s create the APIs in the app.js file by importing the groceryService.js file in the app.js file and exposing the endpoints. Here is how the app.js file looks like:

const express = require('express')
const groceryService  = require('./services/groceryService')
const app = express()

app.post('/api/addGrocery', function (req, res) {
  let groceryServiceObj = new groceryService(req, res)
  groceryServiceObj.getGrocery()
})

app.get('/api/getGrocery', function (req, res) {
  let groceryServiceObj = new groceryService(req, res)
  groceryServiceObj.addGrocery()
})

app.listen(3000, function () {
  console.log('Grocery Web app service listening on port 3000!')
})
Copy
You also need to add the body parser module so that the server is able to parse the data posted to the API endpoints. Install body-parser using npm.

npm install body-parser --save
Copy
Once you have installed the body-parser, add the following code to the app.js to use the body-parser module.

const bodyParser = require('body-parser')
app.use(bodyParser.json())
app.use(bodyParser.urlencoded({ extended : false}))
Copy
You’ll be using the MongoDB Node.js driver to connect from Node.js to MongoDB database. Install it using npm.

npm install mongodb
Copy
Require the MongoDB client inside the groceryService.js file. You’ll be using the MongoDB client to connect to MongoDB database and insert the details to a collection called grocery. Here is how the addGrocery method looks:

const MongoClient = require('mongodb').MongoClient;
const assert = require('assert');
const url = 'mongodb://localhost:27017/groceryDb';

class GroceryService{
    
    constructor(req, res){
        this.req = req
        this.res = res
    }

    insert(groceryItem, db, callback){
        db.collection('grocery').insertOne({
                "item" : groceryItem
        }, function(){
            callback()      
        })
    }

    addGrocery(){
        let self = this;
        let groceryItem = this.req.body.groceryItem;
        try{
            MongoClient.connect(url, function(err, db) {
                assert.equal(null, err);
                self.insert(groceryItem, db, function(){
                    db.close()
                    return self.res.status(200).json({
                        status: 'success'
                    })
                })
            });
        }
        catch(error){
            return self.res.status(500).json({
                status: 'error',
                error: error
            })
        }
    }
}
module.exports = GroceryService
Copy
As seen in the above code, you have used the insertOne API to insert a document to the MongoDB collection.

Creating the Get Grocery API

Now you need to create an API to get the grocery items from the MongoDB database collection. You’ll be making use of the db.collection(‘grocery’).find() API to get all item from the MongoDB collection. Here is the getGrocery method would look:

getGrocery(){
    let self = this;
    try{
        MongoClient.connect(url, function(err, db) {
            assert.equal(null, err);
            let groceryList = []
            let cursor = db.collection('grocery').find();

            cursor.each(function(err, doc) {
              assert.equal(err, null);
              if (doc != null) {
                groceryList.push(doc)
              } else {
                return self.res.status(200).json({
                    status: 'success',
                    data: groceryList
                })
              }
            });
        });
    }
    catch(error){
        return self.res.status(500).json({
            status: 'error',
            error: error
        })
    }
}
Copy
Making API Calls From Angular
Now since the APIs are ready, you need to make the API calls from the Angular UI. Modify the addGrocery method in the common.service.ts file to make a call to the /api/addGrocery API. Here is how it looks:

addGrocery(item){
    return this.http.post('/api/addGrocery',{
        groceryItem : item
    })
}
Copy
Also add a method called getGrocery inside the common.service.ts file to make the API call to get all grocery items. Here is how it looks:

getGrocery(){
    return this.http.post('/api/getGrocery',{})
}
Copy
Modify the GroceryListComponent to subscribe and fetch data from the MongoDB database each time a new data is added. Here is how the GroceryListComponent looks:

import { Component, OnInit } from '@angular/core';
import { CommonService } from '../common/common.service'
import { Grocery } from '../groceryadd/grocery.model'

@Component({
  selector: 'grocery-list',
  templateUrl: './grocerylist.component.html',
  styleUrls: ['./grocerylist.component.css']
})
export class GroceryListComponent implements OnInit {

    private groceryList:Grocery[]

    constructor(private commonService:CommonService){

    }

    ngOnInit(){

        this.getAllGrocery()

        this.commonService.add_subject.subscribe(response => {
            this.getAllGrocery()
        })

    }

    getAllGrocery(){
        this.commonService.getGrocery().subscribe(res =>{
            this.groceryList  = []
            res.json().data.map(e => {
                this.groceryList.push(new Grocery(e.item,false));
            })
        })
    }

}
Copy
Modify the GroceryAddComponent to invoke the subscriber to fetch grocery list from MongoDB. Here is how the GroceryAddComponent looks:

import { Component, OnInit } from '@angular/core';
import { Grocery } from './grocery.model'
import { CommonService } from '../common/common.service'

@Component({
    selector: 'grocery-add',
    templateUrl: './groceryadd.component.html',
    styleUrls: ['./groceryadd.component.css']

})
export class GroceryAddComponent implements OnInit {

    private groceryItem


    constructor(private commonService:CommonService) {

    }

    addGrocery(){
        this.commonService.addGrocery(this.groceryItem).subscribe(res => {
            this.commonService.add_subject.next()
        })

        this.groceryItem = ''
    }

    ngOnInit() {

    }
}

Copy
Wrapping It Up

In this tutorial, you saw how to create a Web App Using Angular 4 & MongoDB. In this part you learnt how to save the data from Angular 4 into MongoDB database. You learnt how to make API calls from Angular 4.

How was you experience trying to learn web app using Angular 4 & MongDB ? Do let us know your thoughts, suggestions or any corrections in the comments below.

Source code from this tutorial is available on GitHub.
