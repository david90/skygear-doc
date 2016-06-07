Storing data on Skygear is built around the `Record`. 
With Skygear iOS SDK imported, you can interact with the `SKYRecord` class to store and manage your data.

The `Record` data is schemaless, it means you don't need to specify ahead of time what keys exist.


<a name="ios-record-class"></a>
# The `SKYRecord` class
### SKYRecord

`SKYRecord` is a key-value data object which can be stored in a _database_. Each
record has a _type_, which describes what kind of data this record holds.

A record can store whatever values that's JSON-serializable, it include
strings, numbers, booleans, dates, plus several custom type that Skygear
supports.

<a name="ios-container"></a>
# Container
### SKYContainer

`SKYContainer` is the uppermost layer of `SKYKit`. It represents the root of all
resources accessible by an application.

There should be **only one** container in each application. 

In `SKYKit`, the container can be accessed via the singleton
`defaultContainer`:

```obj-c
SKYContainer *container = [SKYContainer defaultContainer];
```

Container provides [User Authentication]({{< relref "user.md" >}}),
[Asset Storage]({{< relref "asset.md" >}}) and access to
[public and private databases]({{< relref "#SKYDatabase" >}}).

<a name="ios-database"></a>
# Database
### SKYDatabase

`SKYDatabase` is the central hub of data storage in `SKYKit`. The main
responsibility of database is to store [records]({{< relref "#SKYRecord" >}}),
the data storage unit in Skygear.

In Skygear, there are two types of databases: private and public database.

Every container has one _pubic database_, which stores data accessible to
every users. Every user also has its own _private database_, which stores data
only accessible to that user alone.

<a name="ios-define-record-type"></a>
# Define the record type
You can define different record types to model your app. It's just like defining table
schema in SQL.

``` obj-c
SKYRecord *note = [SKYRecord recordWithRecordType:@"note"]; // define the note type
SKYRecord *blog = [SKYRecord recordWithRecordType:@"blog"]; // define the blog type
```

<a name="ios-save-records"></a>
# Save record

Let's imagine we are using Skygear to write a To-do app. 

When the user creates a to-do item, we will save it to the server. Sample code as the following:

```obj-c
SKYRecord *todo = [SKYRecord recordWithRecordType:@"todo"];
todo[@"title"] = @"Write documents for Skygear";
todo[@"order"] = @1;
todo[@"done"] = @NO;

SKYDatabase *privateDB = [[SKYContainer defaultContainer] privateCloudDatabase];
[privateDB saveRecord:todo completion:^(SKYRecord *record, NSError *error) {
    if (error) {
        NSLog(@"error saving todo: %@", error);
        return;
    }

    NSLog(@"saved todo with recordID = %@", record.recordID);
}];
```

In the above example, we have performed the following actions:

1. First we created a `todo` _record_ and assigned some attributes to it.
2. We fetched the [_container_]("#ios-container") of our app, and set the variable `privateDB` as a reference to the private
   database of the current user.
3. We saved the `todo` record and registered a block to be executed
   after the saving action is done.
4. By saving a record, it is assigned with a unique `recordID`

<a name="ios-modify-record"></a>
# Modify the record

Just simply call the `saveRecord:completion:` to modify an existing record

```obj-c
SKYRecord *todo = [SKYRecord recordWithRecordType:@"todo"];
todo[@"title"] = @"Write documents for Skygear";
todo[@"order"] = @1;
todo[@"done"] = @NO;

SKYDatabase *privateDB = [[SKYContainer defaultContainer] privateCloudDatabase];
[privateDB saveRecord:todo completion:^(SKYRecord *record, NSError *error) {
    if (error) {
        NSLog(@"error saving todo: %@", error);
        return;
    }

    NSLog(@"saved todo with recordID = %@", record.recordID);
}];
```

By running the code above, you should show see this in your console:

```
2015-09-22 16:16:37.893 todoapp[89631:1349388] saved todo with recordID = <SKYRecordID: 0x7ff93ac37940; recordType = todo, recordName = 369067DC-BDBC-49D5-A6A2-D83061D83BFC>
```

The `recordID` property of the item is an unique identifier of the record in the database. 

You can modify the record later with the `recordID`. For example, if you want to mark an item as done:

```obj-c
SKYRecord *todo = [SKYRecord recordWithRecordType:@"todo" name:@"369067DC-BDBC-49D5-A6A2-D83061D83BFC"];
todo[@"done"] = @YES;

[privateDB saveRecord:todo completion:nil];
```

Note that the data in the returned record in the completion block may be
different from the originally saved record. This is because additional
fields maybe applied on the server side when the record is saved. You may
want to inspect the returned record for any changes applied on the server side.

<a name="ios-fetch-record"></a>
# Fetch existing record
With the an unique `recordID`, we could also fetch the record from database:

```obj-c
SKYRecordID *recordID = [SKYRecordID recordIDWithRecordType:@"todo" name:@"369067DC-BDBC-49D5-A6A2-D83061D83BFC"];
[privateDB fetchRecordWithID:recordID completionHandler:^(SKYRecord *record, NSError *error) {
    if (error) {
        NSLog(@"error fetching todo: %@", error);
        return;
    }

    NSString *title = record[@"title"];
    NSNumber *order = record[@"order"];
    NSNumber *done = record[@"done"];

    NSLog(@"Fetched a note (title = %@, order = %@, done = %@)", title, order, done);
}];
```

<a name="ios-fetch-record"></a>
# Delete record
It requires the `recordID` to delete a record:

```obj-c
SKYRecordID *recordID = [SKYRecordID recordIDWithRecordType:@"todo" name:@"369067DC-BDBC-49D5-A6A2-D83061D83BFC"];
[privateDB deleteRecordWithID:recordID completionHandler:nil];
```

If you wish to delete records in a batch, you can use the
`SKYDatabase-deleteRecordsWithIDs:completionHandler:perRecordErrorHandler:`
method.

<a name="ios-record-queries"></a>
# Queries

You can fetch a record with its `recordID`.

However, in real-world applications, we usually want to list out the items according to
certain criteria. You can query the records with `SKYQuery` in Skygear.


The following example shows how to query and fetch a list of to-do items in an ascending order:

```obj-c
SKYQuery *query = [SKYQuery queryWithRecordType:@"todo" predicate:nil];

// Sort with the order key in ascending order
NSSortDescriptor *sortDescriptor = [NSSortDescriptor sortDescriptorWithKey:@"order" ascending:YES];
query.sortDescriptors = @[sortDescriptor];

[privateDB performQuery:query completionHandler:^(NSArray *results, NSError *error) {
    if (error) {
        NSLog(@"error querying todos: %@", error);
        return;
    }

    NSLog(@"Received %@ todos.", @(results.count));
    for (SKYRecord *todo in results) {
        NSLog(@"Got a todo: %@", todo[@"title"]);
    }
}];
```
- Construct a `SKYQuery` to search for `todo` records. If there is no additional
criterion, you may set the predicate to `nil`. 

- Then assign an `NSSortDescription` to ask the Skygear Server to sort the `todo` records by `order` field ascendingly.

`SKYQuery` utilizes `NSPredicate` to apply filter on the query results. For
an overview of supported operations, please refer to the
[Query Guide]({{< relref "query.md" >}}).
