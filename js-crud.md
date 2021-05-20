# Building CRUD Actions in a JS REST API

Generally speaking, implementing a REST API will allow clients to:

- see all items
- search for items
- retrieve an item
- add new items
- edit an item
- delete an item

## CRUD Actions

- **C** Create
- **R** Read
- **U** Update
- **D** Delete

As such, a CRUD app supports all four actions.

We can draw a parallel with CRUD and some major HTTP verbs

**CRUD** | **HTTP**
--- | ---
Create | Post
Read | Get
Update | Patch
Delete | Delete 

Note that REST APIs *are* CRUD-- but that is not the case. RESTful architecture is broader.

### Postman

We can use Postman to simulate front-end clients to interact with/send custom data API requests. For example, we would send a POST request to add a new entry which we specify in the body of `request`.

### Update

In a node.js setting, we can tap into `Express` and use the `:<parameter_name>` syntax to tap into a parameter within the URL path and assign it a variable name. For example, in the route specified by `products/:id`, whatever is passed after `/products` can be accessed by the variable name `id`.

## Connecting the Front-End

Now we will hook up our front-end with the API. Note from the PS demo project, both the front-end and API reside in the same project directory which simplifies some things. In reality, they can be completely indepedent projects in which case we would need to specify the path for the front-end to communicate with the API.

### Read/Get

The following code snippet shows how we can get a particular item requested by the client:

```javascript
document.getElementById("load").onclick = function() {
    const req = new XMLHttpRequest();
    req.open('GET', '/api/products');
    req.onload = function(){
        const data = JSON.parse(req.response);
        addList({data});
    }

    req.send();
}
```

### Axios

We can use Axios which is a library that simplifies XMLHttpRequests. See below for the Axios version of the above code snippet:

```javascript
document.getElementById("load").onclick = function() {
    axios.get('/api/products').then(addList);
}
```

That's a lot of simplification! Notice that Axios handles the data, we do not need to explicitly pass it.

### Reading/Getting a Single Record

To route to a specfic read, that is, a specific item, then we can modify the code as follows:

```javascript
document.getElementById("load").onclick = function() {
    const value = document.getElementById("product-id").value;
    if (value === "") {
        // general read: no item specified, so get all the items
        axios.get('/api/products').then(addList);
    } else {
        // use string interpolation to pass specific item id to the API
        axios.get(`/api/products/${value}`).then(addSingle);
    }
}
```

### Handling Missing Records

If we don't address users attempting to retrieve items that don't exist, the result will be confusing to say the least. The general strategy involves two steps:

1. In our API, we will configure the read/get request (for a single record) to send the data if the request is valid (e.g., in our case, a valid entry in the DB) or a 404 status if not.
2. The front-end will then receive a valid response or an error (i.e., 404 not found)

We will further handle what to do when it receives a 404 by producing a "product not found" message in the area where the user expects a result.

### Adding a New Record

Adding a new record involves sending a POST request with the relevant details within the body. With vanilla JS (and similarly with popular front-end frameworks with a few steps removed) we can capture the data in a form and pass that into the POST request.

### Updating a Record

- Patch: partial update
- Put: Complete object update

## Searching Data

We greatly simplified reading a record from the previous read/get action by passing an ID to the API to retrieve. While it certainly works, in reality it is impractical for users to know the ID of any given item. Rather, more common searches would be for the item name, a certain price range, the list goes on. But this presents a new problem: do we create as many API endpoints as there are search parameters? That, too, would be impractical!

We can approach it two other ways: create a search function or custom queries.

## Validating Data

### API Validation

Never assume the client performs data validation. As such, all APIs should perform data validation before processing a query. When receiving a request that fails validation, the API should send back a 4xx status code. Which 4xx status code to send is up for debate (the top three in this category are 400, 403, 422).

## Questions

When updating a record, are we omitting data that is unchanged to decrease data size of what we're sending to the API to update? How is that any better than patching all the fields?