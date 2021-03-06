# graphql-query-builder-v2

Since graphql-query-builder seems to no longer be actively maintained, I have forked it. Nothing much has changed except the ability to use `null` and `undefined` values in the arguments. You are also allowed to not execute `.find`

API is unchanged.

### Working examples in v2
```javascript
let query =
    new QueryBuilder(
        "functionName",
        {
            name: null
        }
    )
```

**Output**

`functionName( name: null )`

#### With .find

```javascript
let query =
    new QueryBuilder(
        "functionName",
        {
            name: null
        }
    )
query.find([
    "_id",
    "name"
])
```

**Output**

```javascript
functionName( name: null ) {
    _id,
    name
}
```

#### With enums

```javascript
let query =
    new QueryBuilder(
        "functionName",
        {
            status: QueryBuilder.Enum('draft')
        }
    )
```

**Output**

```javascript
functionName( status: draft )
```

#### Combining to create graphql-upload example

As I have recently started using graphql-upload, I am sharing how this package can also be used to create a valid FormData query.

```javascript
const QueryBuilder = require("graphql-query-builder-v2");

// The wrapping "mutation"
let mutation = new QueryBuilder("mutation");
mutation.filter({
	$image: QueryBuilder.Enum("Upload")
});

// Nested data within actual_query
const nested = {
	data1: "stuff",
	image: QueryBuilder.Enum("$image")
};
// The actual query we want to run
let actual_query = new QueryBuilder("imageUpload");
actual_query.filter({
	_id: 1,
	display_name: "example",
	nested
});
// putting the actual query into the wrapping mutation
mutation.find([
	actual_query
]);

let form_data = new FormData();

const operations = {
    query: mutation_query.toString(),
    variables: {
        image: null
    }
};
form_data.append("operations", JSON.stringify(operations));

const map = {
    "0": ["variables.image"]
};
form_data.append("map", JSON.stringify(map));

form_data.append("0", image_file); // image_file is variable that is holding your File/Blob
```

**Output**

mutation output
```
mutation($image: Upload) {
	imageUpload(
		_id: 1, 
		display_name: "example", 
		nested: {
			data1: "stuff", 
			image: $image
		})
}
```

# Install

Use this instead of the install below. 

`npm install graphql-query-builder-v2`

<hr>
<hr>


# graphql-query-builder

a simple but powerful graphQL query builder

**info:**

[![npm version](https://badge.fury.io/js/graphql-query-builder.svg)](https://badge.fury.io/js/graphql-query-builder)
[![License](http://img.shields.io/:license-mit-blue.svg)](http://doge.mit-license.org)
[![pull requests welcome](https://img.shields.io/badge/Pull%20requests-welcome-pink.svg)](https://github.com/codemeasandwich/graphql-query-builder/pulls)
[![GitHub stars](https://img.shields.io/github/stars/codemeasandwich/graphql-query-builder.svg?style=social&label=Star)](https://github.com/codemeasandwich/graphql-query-builder)


**tests:**

[![build](https://api.travis-ci.org/codemeasandwich/graphql-query-builder.svg)](https://travis-ci.org/codemeasandwich/graphql-query-builder)
[![Coverage Status](https://coveralls.io/repos/github/codemeasandwich/graphql-query-builder/badge.svg?branch=master)](https://coveralls.io/github/codemeasandwich/graphql-query-builder?branch=master)


**quality:**

[![Code Climate](https://codeclimate.com/github/codemeasandwich/graphql-query-builder/badges/gpa.svg)](https://codeclimate.com/github/codemeasandwich/graphql-query-builder)
[![bitHound Overall Score](https://www.bithound.io/github/codemeasandwich/graphql-query-builder/badges/score.svg)](https://www.bithound.io/github/codemeasandwich/graphql-query-builder)
[![Issue Count](https://codeclimate.com/github/codemeasandwich/graphql-query-builder/badges/issue_count.svg)](https://codeclimate.com/github/codemeasandwich/graphql-query-builder)
[![Known Vulnerabilities](https://snyk.io/test/npm/graphql-query-builder/badge.svg)](https://snyk.io/test/npm/graphql-query-builder)

### If this was helpful, [★ it on github](https://github.com/codemeasandwich/graphql-query-builder)

*tested on [**NodeJS**](https://nodejs.org) and [**Webpack**](https://webpack.github.io)*


## [Demo / Sandbox](https://tonicdev.com/codemeasandwich/57a0727c80254315001cb366) :thumbsup:

# Install

`npm install graphql-query-builder`

# Api

``` js
const Query = require('graphql-query-builder');
```

### constructor
query/mutator you wish to use, and an alias or filter arguments.

| Argument (**one** to **two**)  | Description
|--- |---
| String | the name of the query function
| * String / Object | (**optional**) This can be an `alias` or `filter` values 

``` js
let profilePicture = new Query("profilePicture",{size : 50});
``` 

### setAlias
set an alias for this result.

| Argument | Description
|--- |---
| String | The alias for this result

``` js
profilePicture.setAlias("MyPic");
``` 

### filter
the parameters to run the query against.

| Argument | Description
|--- |---
| Object | An object mapping attribute to values

``` js
profilePicture.filter({ height : 200, width : 200});
``` 

### find
outlines the properties you wish to be returned from the query.

| Argument (**one** to **many**) | Description
|--- |---
| String or Object |  representing each attribute you want Returned
| ... | *same as above*

``` js
    profilePicture.find( { link : "uri"}, "width", "height");
``` 

### toString
return to the formatted query string

``` js
  // A (ES6)
  `${profilePicture}`;
  // B
  profilePicture+'';
  // C
  profilePicture.toString();
``` 

## run samples

``` bash
node example/simple.js
```

# Example

``` js 
var Query = require('graphql-query-builder');

// example of nesting Querys
let profilePicture = new Query("profilePicture",{size : 50});
    profilePicture.find( "uri", "width", "height");
    
let user = new Query("user",{id : 123});
    user.find(["id", {"nickname":"name"}, "isViewerFriend",  {"image":profilePicture}])
    
    console.log(user)
    /*
     user( id:123 ) {
    id,
    nickname : name,
    isViewerFriend,
    
    image : profilePicture( size:50 ) {
        uri,
        width,
        height
    }
  }
    */
    
// And another example

let MessageRequest = { type:"chat", message:"yoyo",
                   user:{
                            name:"bob",
                            screen:{
                                    height:1080,
                                    width:1920
                                    }
                    },
                    friends:[
                             {id:1,name:"ann"},
                             {id:2,name:"tom"}
                             ]
                    };
                    
let MessageQuery = new Query("Message","myPost");
    MessageQuery.filter(MessageRequest);
    MessageQuery.find({ messageId : "id"}, {postedTime : "createTime" });
    
    console.log(MessageQuery);
    
    /*
    myPost:Message( type:"chat",
                    message:"yoyo",
                    user:{name:"bob",screen:{height:1080,width:1920}},
                    friends:[{id:1,name:"ann"},{id:2,name:"tom"}])
        {
            messageId : id,
            postedTime : createTime
        }
    */

    // Simple nesting
    
    let user = new Query("user");
        user.find([{"profilePicture":["uri", "width", "height"]}])
    
    /* 
    user {
      profilePicture {
        uri,
        width,
        height
       }
     }
    */ 
    
    // Simple nesting with rename
    
    let user = new Query("user");
        user.find([{"image":{"profilePicture":["uri", "width", "height"]}}])
    
    /* 
    user {
      image : profilePicture {
        uri,
        width,
        height
       }
     }
    */ 
```
