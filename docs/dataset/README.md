# Datasets
Dataset is used to store data for your application in the cloud. You can interact with your data via REST API and SDK's. All data protected with policy autorization. You can establish relation between different Datsets or other part of the platform. You do not need take care of indexes, foring keys, scaling, backups and other headache related to database management. All this will be done by Jexia. Each project has own instance of dataset and other projects can't impact stability and security of your project.   

Datasets has built-in support for validation, schema and schemaless support, dafault values for fields.  

To create Dataset click the button "Go to project" and on the next screen "Create dataset":
![Create Dataset](./create_ds.png)

Name your dataset. The name of the dataset is used as an endpoint (```*.app.jexia.com/ds/dataset_name```) to allow you to communicate with the API. The name of your dataset can contain only Latin characters and digits. The name of the dataset has to start with a character.

## Configuration

### Schema:

The next step is to add fields to your datasets. In order to create a field, click the "Add field" button. In the same window, you can select name, type, and validation for your field. You can also provide a default value for the field. Field name validation parameters can be changed in the future via the edit field but not the field type. If you want to change the type then you can only delete and create the field again. However, by deleting the field you will also lose the data stored in the field. With the Schema approach in responding, you will get the field in specific types:  String, Integer, Float, Date, DateTime, Boolean, JSON or UUID. Data will be validated against the validators added to fields while inserting and updating records.
![Create Field](./create_field.png)

::: tip
If you will insert JSON object which has additional fields versus schema, those fields will be save as schemaless and will be available during fetching data 
:::

### Schemaless:
To apply the schemaless approach just insert your JSON object into JEXIA without creating any fields inside Dataset. Data will be stored automatically with the proper type. Please, note that validations and default values are not applicable to schemaless data. You can convert from Schemaless to Schema when data design for your project is stabilized. Jexia supports String, Integer, Float, Date, DateTime, Boolean, JSON and UUID as field types.

::: warning
Please, keep in mind when you conver field from schemaless to schema data from records will not be transfered to schema fields. You need to do it by your own and control quality of data. 

If you delete fields from schema data will be deleted as well. It will not be converted back to schemaless.

During fetching data next priority is apply for fields with same name: Schema - Schenaless

In case you have related sets you will get next priority for fields with same name: Schema - Related Schema - Schemaless  
:::

### Validation:

Depends on field-type validation parameters will be available for you. The most validators are available for String type. Such as: Required, UpperCase, LowerCase, Alphanumeric, Numeric, Alpha, Min/ Max length, RegEx.

For Float and Integer, there are Required, Min/ Max value.

In the future, we plan to add Date range and other validators. 

You might admit that when you select some validators, another one might be unavailable. It is due to logical exclusion. For example, it is not logical to have Upper and Lower case validators at the same time. To reduce the possibility of human mistakes we decided to prevent selection for some combinations.  

::: tip
Please, keep in mind validation is applicable for schema fields only. Jexia apply same validations for Create and for Update actions.
:::

## Default values
You can setup default values for field. Value will be validated against type and constrins. 

::: warning
Please, keep for String type it is not possible to put default value as empty string '', you will get either some value or **null**
:::

## Create records
To create record in Jexia Dataset you need to have Create action avaialble for your User or API-Key for specific resource. 
I will use User approach as it will be more common use case for record creation.

::: tip
Please, keep in mind in responds you always get back array of records event if you insert only 1 record. With this you can apply same approach for data manipulation on front end.  
:::
<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
import { jexiaClient, dataOperations,UMSModule } from "jexia-sdk-js/node"; 
const ds = dataOperations();
const ums = new UMSModule(); 

jexiaClient().init({
  projectID: "project_id",
}, ds, ums);

const user = await ums.signIn({    
  email: 'Elon@tesla.com',    
  password: 'secret-password'
});  

let orders = [{
    "title":"Order1",
    "total":10,
    "verified":false
}, {
    "title":"Order2",
    "total":100,
    "verified":false
}]
const orders = dataModule.dataset("orders");
const insertQuery = orders.insert(orders);  
insertQuery.subscribe(records => { 
    // you will always get an array of created records, including their 
    //generated IDs (even when inserting a single record) 
  }, 
  error => { 
     // you can see the error info here, if something goes wrong 
});
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

As a result you will get next array of objects:
```JSON
[{
    "id": "e0e17683-f494-4f33-8343-ffed792b324e",
    "created_at": "2020-02-15T19:43:39.784342Z",
    "updated_at": "2020-02-15T19:43:39.784342Z",
    "title":"Order1",
    "total":10,
    "verified":false
}, {
    "id": "e0e17683-f494-4f33-9563-dded795e3121",
    "created_at": "2020-02-15T19:43:39.784342Z",
    "updated_at": "2020-02-15T19:43:39.784342Z",
    "title":"Order2",
    "total":100,
    "verified":true
}]
```
## Fetch records
To get your data you need to have Read action available for you for particular resource. You can apply different filters to get specific data. In example I will show example with API-Key usage as the most common approach.

As soon as JS SDK built on top of RxJS you can use power of this libary and re-use all available methods from here.  


<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
// Jexia client
import { jexiaClient, dataOperations, field } from "jexia-sdk-js/node"; 
// jexia-sdk-js/browser;
const ds = dataOperations();

jexiaClient().init({
  projectID: "project_id",
  key: "API_KEY",
  secret: "API_SECRET",
}, ds);

const orders = dataModule.dataset("orders");
const selectQuery = orders
      .select()
      .where(field => field("verified").isEqualTo(true))
   // .where(field("title").isDifferentFrom("test")) 
   // .where(field("total").isBetween(1,30))
   // .where(field("total").isEqualOrGreaterThan(15))
   // .where(field("total").isEqualOrLessThan(7))
   // .where(field("total").isEqualTo(100))
   // .where(field("total").isGreaterThan(57))
   // .where(field("total").isLessThan(100))
   // .where(field("id").isInArray(my_val))   // my_val=[uuid1,uuid2];
   // .where(field("id").isNotInArray(my_val)) // my_val=[uuid1,uuid1];
   // .where(field("title").isLike("Moby dick"))
   // .where(field("title").isNotNull())
   // .where(field("title").isNull())
   // .where(field("title").satisfiesRegexp('a-z0-9'))   
      .pipe(
        // put them into archive!
        switchMap(records => archive.insert(records)),
      );  
selectQuery.subscribe(records => { 
     // you will always get an array of created records, including their 
     //generated IDs (even when inserting a single record) 
  }, 
  error => { 
     // you can see the error info here, if something goes wrong 
});
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

As a result you will get next array of objects:

```JSON
[{
    "id": "e0e17683-f494-4f33-9563-dded795e3121",
    "created_at": "2020-02-15T19:43:39.784342Z",
    "updated_at": "2020-02-15T19:43:39.784342Z",
    "title":"Order2",
    "total":100,
    "verified":true
}]
```

## Delete a record
To delete record you need to have Delete action available for you for particular resource. You can setup it in Policy area. 
I will show examples with Project User usage as it will be common usage. 

When you run Delete update you will get back array of affected records so you can sync changes with front end as well.  

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
import { jexiaClient, UMSModule, dataOperations } from "jexia-sdk-js/node"; 
// to use .where and .outputs
import { field } from "jexia-sdk-js/node"; 

const ds = dataOperations();
const ums = new UMSModule(); 

jexiaClient().init({
  projectID: "project_id",
}, ds, ums);

const user = await ums.signIn({    
  email: 'Elon@tesla.com',    
  password: 'secret-password'
});

const orders = dataModule.dataset("orders");
const deleteQuery = orders
.delete()
.where(field => field("verified").isEqualTo(true));  

// Either way, the response will be an array  
deleteQuery.subscribe(records => { 
     // you will always get an array of created records, including their 
     //generated IDs (even when inserting a single record) 
  }, 
  error => { 
     // you can see the error info here, if something goes wrong 
});
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

As a result you will get next array of objects:

```JSON
[{
    "id": "e0e17683-f494-4f33-9563-dded795e3121",
    "created_at": "2020-02-15T19:43:39.784342Z",
    "updated_at": "2020-02-15T19:43:39.784342Z",
    "title":"Order2",
    "total":100,
    "verified":true
}]
```
## Update records

To update record you need to have Update action available for you for particular resource. You can setup it in Policy area. 
I will show examples with Project User usage as it will be common usage. 

When you run Update action you will get back array of affected records so you can sync changes with front end as well.  

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
import { jexiaClient, UMSModule, dataOperations } from "jexia-sdk-js/node"; 
// to use .where and .outputs
import { field } from "jexia-sdk-js/node"; 

const ds = dataOperations();
const ums = new UMSModule(); 

jexiaClient().init({
  projectID: "project_id",
}, ds, ums);

const user = await ums.signIn({    
  email: 'Elon@tesla.com',    
  password: 'secret-password'
});

const orders = dataModule.dataset("orders");
const updateQuery = orders.update([{ verified: true }]);  
.where(field => field("verified").isEqualTo(false));  

// Either way, the response will be an array  
deleteQuery.subscribe(records => { 
     // you will always get an array of created records, including their 
     //generated IDs (even when inserting a single record) 
  }, 
  error => { 
     // you can see the error info here, if something goes wrong 
});
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

As a result you will get next array of objects:

```JSON
[{
    "id": "e0e17683-f494-4f33-8343-ffed792b324e",
    "created_at": "2020-02-15T19:43:39.784342Z",
    "updated_at": "2020-02-16T13:15:00.784342Z",
    "title":"Order1",
    "total":10,
    "verified":true
}]
```

## Related records
If you created multiple Datasets you can establish relations between them under Relations menu. Currently, we support 1-1, 1-m, m-1 relation types. 

When do you need this? For example you can keep orders header and order details separatly. Cool think about Jexia that you do not need to care about external keys and index optimizations to organise all off this. All managed by Jexia in optimal way. 

Another cool think as soon as you setup relation you insert object which has parrent / chield data and Jexia automatically put data in proper place with keeping relation between them. For example I have Dataset: Order and Dataset: Items with 1 to many relations between them. So I can insert into Order next object and data will land in proper place: 

When I do fetching I can specify if I want to get result with related data or not. 

::: tip
Please, keep in mind currently is not possible to make relation with dataset itself. It is common case to build parent-chiled records. From another side nobody stops you to store chield in separate dataset and have 1-m relation between them.  
:::

::: tip
Please, keep in mind in result json you always will have id for related data
:::

```json
{
    "title":"Order1",
    "total":10,
    "verified":false,
    "items": [
        {
            "name":"Item 1",
            "qty":2
        },
        {
            "name":"Item 3",
            "qty":20
        }
    ]
}
```


### Get related data
<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
ds.dataset("orders")
  .select()
  .related("items")
  .subscribe(orderWithItem => {
    //you will get same JSON as above
  });
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

If you want to get related data you just need add them to request, Jexia will do matching automatically and send it back in result JSON. Easy than GaphQL, yeh?

### Get specific fields from related data
<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
dom.dataset("orders")
  .select()
  .related("items", items => items.fields("qty"))
  .subscribe(ordersWithComments => {
    console.log(ordersWithComments);
  });
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

In some cases you might want to get only specific fields from related data. You can do it. Example request show you how. 

### Many related resources
<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
dom.dataset("orders")
  .select()
  .related("items", items => items
    .fields("name", "qty")
    .related("stock"), stock => stock
      .fields("available")
    )
  )
  .subscribe()
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

In some cases you might have many dependancies between data. not a problem as well. You can add as many related data as you need. 

### Attach and detach records
<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
dom.dataset("posts")
    .where(field("id").isEqualTo('my_uuid'));
    .attach("items", [
      "b4961b6a-85a2-4ee8-b946-9001c978c801",
      "e199d460-c88f-4ab9-8373-1d6ad0bd0acb",
    ])
    .subscribe();
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

If you would need to related already exisintg data you can use `.attach()` and `.detach()` methods.
For this you need to specify to which parrent you want to attach chield record. In example I am adding to order with id = my_uuid
two items with id's "b4961b6a-85a2-4ee8-b946-9001c978c801" and "e199d460-c88f-4ab9-8373-1d6ad0bd0acb".

## Real-Time notification
If you want to have real-time update about changes on you Dataset you can use real-time notifications which is build-in into Dataset.
This is Pro function so you need to have subscribtion for this. 

After action you will get notification with records id which was modified. After can reflect this on front end and if needed select specific record to get updated data. We send only ID due to security reason, as it might be situation that many users will be subscribed to such changes but they should not have access to data itself. With this you can decide if you want refresh data or not.  

You can use `watch()` method to subscribe for notifications. Actions can be provided either as arguments or as an array. Allowed values are:
1. created
2. updated
3. deleted
4. all (used by default)

You can unsubscribe from notifications any time by calling `.unsubscribe()` method

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
import { jexiaClient, dataOperations, realTime } from "jexia-sdk-js/node";
const ds = dataOperations();
const rtc = realTime();

// initialize jexia client
jexiaClient().init(credentials, ds, rtc);

const subscription = dataModule.dataset("orders")
  .watch("created", "deleted")
  .subscribe(messageObject => {
    console.log("Realtime message received:", messageObject.data);
  }, error => {
    console.log(error);
  });


subscription.unsubscribe();
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>


## Filtering
Optional filtering parameters to specify which records to select. If it is not provided the request is applied on all records.
All filters build with help of 3 variables: field, comparator, value.

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
import { field } from "jexia-sdk-js/node"; 
....
   .where(field("title").isDifferentFrom("test")) 
   .where(field("total").isBetween(1,30))
   .where(field("total").isEqualOrGreaterThan(15))
   .where(field("total").isEqualOrLessThan(7))
   .where(field("total").isEqualTo(100))
   .where(field("total").isGreaterThan(57))
   .where(field("total").isLessThan(100))
   .where(field("id").isInArray(my_val))   // my_val=[uuid1,uuid2];
   .where(field("id").isNotInArray(my_val)) // my_val=[uuid1,uuid1];
   .where(field("title").isLike("Moby dick"))
   .where(field("title").isNotNull())
   .where(field("title").isNull())
   .where(field("title").satisfiesRegexp('a-z0-9')) 

const isAuthorTom = field("author_name").isEqualTo("Tom");  
const isAuthorDick = field("author_name").isEqualTo("Dick");  

const isAuthorTomOrDick = isAuthorTom.or(isAuthorDick);  
const isAuthorTomOrDick = isAuthorTom.and(isAuthorDick);  
// In order to use these conditions, they need to be added to a query through `.where` method  

dataModule.dataset("posts")  
 .select()
 .where(isAuthorTomOrDick)
 .subscribe(records => // posts of Tom and Dick); 
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>


Below you can find examples for available comparators:

Comparator group | comparator| value | Applicable field types
-----------------|-----------|-------|-----------------------
Equality comparators|“>”, “<”, “=”, “!=”, “>=”, “<=”|single value matching field type|all fields   
Emptiness comparators|“null”|true or false|all fields
Range comparator|“between”|[start, end]|textual, numbers, date/times
Array comparators|“in”, “not in”|Array of allowed values (matching field type)|all fields
Pattern comparator|“like”|string value|textual
Regex comparator|“regex”|string value|textual

There are next operator available to combine filters: "and", "&&", "or" or "||"

## Response fields
Sometimes you want to show specific fields from record versus all record. With Jexia you can have this. During data selection you need to specify what fields to get back. It is applicable for related data as well. 

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
const orders = dataModule.dataset("orders");

orders.select()
  .fields("title", "items.name") // you can also pass an array of field names 
  .subscribe(records => {}); // you will get array of {id, title, author} records (id is always returned));
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

## Limit & Offset
You can use limit and offset on a Query to paginate your records. They can be used separately or together. Only setting the limit (to a value X) will make the query operate on the first X records. Only setting the offset will make the query operate on the last Y records, starting from the offset value.

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
const orders = dataModule.dataset("orders");

orders.select()
  .limit(2)
  .offset(5)
  .subscribe(records => // paginatedPosts will be an array of 2 records, starting from position 5);
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

## Sorting
To sord output you can apply sort methods of Jexia, you can apply `Asc` and `Desc` directions. 

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
const orders = dataModule.dataset("orders");

posts
  .select()
  .sortAsc("total")
//.sortDesc("total")
  .subscribe(records => { 
    // you've got sorted records here 
  });
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>



## Aggregation functions
<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
const posts = dataModule.dataset("posts");

posts.select()
  .fields("author", { fn: "max", field: "likes" })
  .subscribe(result => {
    /* the result is:
       [
         { author: "Tom", "max": 10 },
         { author: "Harry", "max": 24 },
       ]
     */
  });
```
</template>
<template v-slot:ts>

``` bash
```

</template>
</CodeSwitcher>

There are a few aggregation functions you are able to use in order to do some calculations before obtaining data:
1. max
2. min
3. sum
4. avg
5. count

You can combine output with other fields. 