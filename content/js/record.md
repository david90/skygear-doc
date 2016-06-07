Storing data on Skygear is built around `Record`

For simplicity, most of the examples use success and error callbacks. If you're familiar with JavaScript Promises or would like to learn how to avoid writing nested code, you may use Promises instead.

The `Record` data is schemaless, it means you don't need to specify ahead of time what keys exist.

<a name="js-record-class"></a>
# The `Record` class
A `Record` has the following property:

- It must have a type.
- It is a key-value data object that store at [_database_](#database).
- It belongs to the current user who has logged in. 

<a name="js-container"></a>
# Container

The Skygear Container (`JSSkygear`) is the uppermost layer of skygear. 

In practice, by importing Skygear
`import skygear from 'skygear'` will give you a container instance.

In most of the cases you will only need one container instance.

The first thing you need to interact with the container is setting the `endPoint` and
`accessToken`.

``` javascript
JSSkygear.endPoint = 'http://your-endpoint.skygeario.com'; //Your server endpoint
JSSkygear.configApiKey('SKYGEAR_API_KEY'); //Your Skygear API Key
```

[Sign up](http://portal.skygear.io/) an account at Skygear Portal to obtain the server endpoint and API Key


<a name="js-database"></a>
# Database

In Skygear, there are two types of databases: private and public database.

### Private Database

- Everything in private database is truely private, regardless of what access
control entity you set to the record.

### Public Database

- Record saved at public database is defualt public. To control or restrict the access, you
may set difference access control entity to the record.

### Example
This is a simple example how you can work with the Records. 
You can save `note` to the `publicDB` of the server:

``` javascript
skygear.publicDB.save(new Note({
    'content': 'Hello World!'
})).then((record) => {
    console.log(record);
}, (error) => {
    console.log(error);
});
```

<a name="js-define-record-type"></a>
# Define the record type

You can define different record types to model your app. It's just like defining table
schema in SQL.

``` javascript
const Note = skygear.Record.extend('note'); // define the note type
const Blog = skygear.Record.extend('blog'); // define the blog type
```
<a name="js-modify-record"></a>
# Modify the record

You can use the `save()` method to modify the record.

``` javascript
skygear.publicDB.save(new Note({
    'content': 'Hello World!'
})).then((record) => {
    console.log(record);
}, (error) => {
    console.log(error);
});
```

You can also save multiple records at one time.

``` javascript
let helloNote = new Note({
  content: 'Hello world'
});

let foobarNote = new Note({
  content: 'Foo bar'
});

skygear.publicDB.save([helloNote, foobarNote])
.then((result) => {
  let {
    savedRecords: [savedHelloNote, savedFoobarNote],
    errors:       [helloError, foobarError]
  } = result;

  if (helloError) {
    console.error('Fail to save hello note');
  } else {
    console.log('updated hello note: ', savedHelloNote);
  }

  if (foobarError) {
    console.error('Fail to save foo bar note');
  } else {
    console.log('updated foo bar note: ', savedFoobarNote);
  }
}, (error) => {
  if (error) {
    console.error('Request error', error);
  }
});
```

After saving a record, any attributes modified from the server side will
be updated on the saved record object in place. The local transient fields of
the records are merged with any remote transient fields applied on the server
side.

<a name="js-fetch-record"></a>
# Fetch existing record

You can construct a `Query` object by specifying the Record Type.
Config the query by mutating its state.

``` javascript
var query = new skygear.Query(Blog);
query.greaterThan('popular', 10);
query.addDescending('popular');
query.limit = 10;

skygear.publicDB.query(query).then((records) => {
  console.log(records)
}, (error) => {
  console.log(error);
})
```
<a name="js-delete-record"></a>
# Delete record

``` javascript
skygear.publicDB.delete(record)
.then(() => {
  console.log(record);
}, (error) => {
  console.log(error);
});
```

You can also delete multiple records at one time.

``` javascript
let Note = skygear.Record.extend('note');
let query = new skygear.Query(Note);
query.lessThan('rating', 3);

let foundNotes = [];
skygear.publicDB.query(query)
.then((notes) => {
  console.log(`Found ${notes.length} notes, going to delete them.`);

  foundNotes = notes;
  return skygear.publicDB.delete(notes)
})
.then((errors) => {
  errors.forEach((perError, idx) => {
    if (perError) {
      console.error('Fail to delete', foundNotes[idx]);
    }
  });
}, (reqError) => {
  console.error('Request error', reqError);
});
```
