# dynamodb-php-wrapper
Access AWS DynamoDB through simpler interface in PHP

This module, dynamodb-php-wrapper, allows you to access DynamoDB more easily through interfaces shown below. 

Each interface is just a wrapper, so accepts same arguments original one does and you can get that information from [the document](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//Welcome.html) provided by AWS if needed.

This section assumes that:
* There is a table 'User', which has hash key 'UserId' as Number.
* There is a table 'Friend', which has hash key 'UserId' as Number and range key 'FriendUserId' as Number.

## init
First of all, DynamoDBWrapper needs to be instantiated like this: 
```php
<?php

require_once '/path/to/aws-autoloader.php'; // provided by AWS
require_once '/path/to/DynamoDBWrapper.php'; // this module

$ddb = new DynamoDBWrapper(array(
    'key'    => 'YOUR_KEY',
    'secret' => 'YOUR_SECRET_KEY',
    'region' => 'SOME_REGION'
));
```
then, you can access DynamoDB through this instance.

## get
get is a wrapper of [GetItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_GetItem.html) and will be used like this:
```php
<?php
$key = array(
    'UserId' => 2,
);
$result = $ddb->get('User', $key);
/*
Array
(
    [Name] => User2
    [UserId] => 2
)
*/
```
## batchGet
batchGet is a wrapper of [BatchGetItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_BatchGetItem.html) and will be used like this:
```php
<?php

$keys = array(
    array(
        'UserId' => 2,
    ),
    array(
        'UserId' => 3,
    ),
);
$result = $ddb->batchGet('User', $keys);
/*
Array
(   
    [0] => Array
        (   
            [Name] => User 2
            [UserId] => 2
        )
    [1] => Array
        (   
            [Name] => User 3
            [UserId] => 3
        )
)
*/
```
batchGet can get more than 25 items by one call. This wrapper sends multiple requests to DynamoDb if needed, and merge them as a single result.
## query
query is a wrapper of [Query](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_Query.html) and will be used like this:
```php
<?php

$keyConditions = array(
    'UserId' => 2,
);
$result = $ddb->query('Friend', $keyConditions);
/*
Array
(
    [0] => Array
        (
            [UserId] => 2
            [FriendUserId] => 3
        )
    [1] => Array
        (
            [UserId] => 2
            [FriendUserId] => 4
        )
    ...
)
*/
```
query can accept options as $options, such as Limit, ExclusiveStartKey, ScanIndexForward and IndexName. If you use SLI, IndexName must be included in options.
## scan
scan is a wrapper of [Scan](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_Scan.html) and will be used like this:
```php
<?php

$filter = array(
    'UserId'       => array('GT', 1),
    'FriendUserId' => array('IN', array(4, 5)),
);
$result = $ddb->scan('Friend', $filter);
/*
Array
(
    [0] => Array
        (
            [UserId] => 2
            [FriendUserId] => 4
        )
    [1] => Array
        (
            [UserId] => 2
            [FriendUserId] => 5
        )
    ...
)
*/
```

## count
count is a wrapper of [Query](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_Query.html) with _Select_ param as COUNT. This return the number of items query fetches and will be used like this:
```php
<?php

$keyConditions = array(
    'UserId' => 2,
);
$result = $ddb->count('Friend', $keyConditions);
// 5
```
## put
put is a wrapper of [putItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_PutItem.html) and will be used like this:
```php
<?php

$item = array(
    'UserId' => 1,
    'Name'   => 'User N',
);
$result = $ddb->put('User', $item);
// true, if success
```
If operation needs to be conditional, you can specify the expectation like this:
```php
<?php

$item = array(
    'UserId' => 1,
    'Name'   => 'User 1',
);
$expected = array(
    'UserId' => array('Exists' => false)
);
$result = $ddb->put('User', $item, $expected);
// false, if UserId = 1 exists
```
## batchPut
batchPut is a wrapper of [BatchWriteItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_BatchWriteItem.html) with put requests and will be used like this:
```php
<?php

$items = array(
    array(
        'UserId' => 2,
        'Name'   => 'User2',
    ),
    array(
        'UserId' => 3,
        'Name'   => 'User3',
    ),
);
$result = $ddb->batchPut('User', $items);
// true, if success
```
If the number of requests is more than 25, this module divides the requests and send them continuously until all of requests are completed.
## update
update is a wrapper of [UpdateItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_UpdateItem.html) and will be used like this:
```php
<?php

$key = array(
    'UserId' => 2,
);
$update = array(
    'Name' => array('PUT', 'New User Name')
);
$result = $ddb->update('User', $key, $update);
/*
Array
(
    [Name] => New User Name
)
*/
```
If operation succeeded, the updated value will be returned.
If operation needs to be conditional, you can specify the expectation as 3rd arg.
## delete
delete is a wrapper of [DeleteItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_DeleteItem.html) and will be used like this:
```php
<?php

$key = array(
    'UserId'       => 2,
    'FriendUserId' => 4,
);
$result = $ddb->delete('Friend', $key);
/*
Array
(
    [UserId] => 2
    [FriendUserId] => 4
)
*/
```
If operation succeeded, the deleted item will be returned.
## batchDelete
batchDelete is a wrapper of [BatchWriteItem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_BatchWriteItem.html) with put requests and will be used like this:
```php
<?php

$keys = array(
    array(
        'UserId'       => 2,
        'FriendUserId' => 3,
    ),
    array(
        'UserId'       => 2,
        'FriendUserId' => 4,
    ),
    ...
);
$result = $ddb->batchDelete('Friend', $keys);
// true, if success
```
If the number of requests is more than 25, this module divides the requests and send them continuously until all of requests are completed.
## createTable
createTable is a wrapper of [CreateTable](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference//API_CreateTable.html) and will be used like this:
```php
<?php

$result = $ddb->createTable('User', 'UserId');
// Create a table 'User', which has a hash key 'UserId' as STRING.
 
$result = $ddb->createTable('User', 'UserId::N');
// Create a table 'User', which has a hash key 'UserId' as NUMBER.
 
$result = $ddb->createTable('Friend', 'UserId', 'FriendUserId');
// Create a table 'Friend', which has a hash key 'UserId' as STRING and a range key 'FriendUserId' as STRING.
```
If LSI is needed, it can be added like this:
```php
<?php

$options = array(
    'LocalSecondaryIndexes' => array(
        array('name' => "UserName",
              'type' => 'S',
              'projection_type' => 'KEYS_ONLY'
        ),
        array('name' => "CreatedAt",
              'type' => 'S',
              'projection_type' => 'KEYS_ONLY'
        )
    )
);
$result = $ddb->createTable('Friend', 'UserId', 'FriendUserId', $options);
// Create a table 'Friend', which has a hash key 'UserId' as STRING and a range key 'FriendUserId' as STRING and Local Secondary Index 'UserName' and 'CreatedAt' as both STRING.
```
After requested, it will wait for completion.
