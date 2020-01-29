# Shipt Coding Challenge

Create a very basic API application using the Go language, where we can retrieve product information.

## Included contents

 - Source code (`src/myPkg/product_info.go`)
 - Unit test code (`src/myPkg/product_info_test.go`)
 - Readme file (`readme.md`)

### Building and running the project

 - Unzip into a new folder
 - In a command line prompt, make sure the environment variable `GOPATH` is set to the new folder
 - Execute `go get -v github.com/aws/aws-sdk-go/service/dynamodb` - this will download the AWS SDK as it is too large to be zipped and sent via email
 - `cd` into `src/myPkg`
 - `go build`
 - `go run product_info.go`
 - Unit tests can be executed by running `go test`

## Assumptions

 - The API assumes it will only access the one database that was created as data for this exercise.
 - There is no authentication required to access this API.
 - Sorting on the returned data does not have secondary sort column, if 2 or more items have the same sort value, the order will be based on the order of items returned from dynamoDB.
 - Empty values for query parameters are ignored.
 - Unknown query parameters are ignored and not validated.
 - None of the query parameters are required, not passing any will return the entire set of data
 - Assumes the set of data is limited, as AWS SDK will only evaluate 1 MB of data at a time and return a *last evaluated key* if that is exceeded. This case is not handled

## Accessing the API

After running the executable or building and running `go run product_info.go`, the API can be accessed by going to the URL `http://localhost:8080/query` or by using an API tool such as Postman.

## API documentation

Query the dynamoDB for product name and price

- **URL** : `/query`

- **METHOD** : `GET`

- **Auth required** : `NO`

- **Permission required** : `None`

- **URL Params** :

  **Optional** :

  `search_term=[alphanumeric]` - Non-case-sensitive search by the name of the product

  `min_price=[integer]` - Minimum price of the product

  `max_price=[integer]` - Maximum price of the product

  `limit=[integer]` - Limit the maximum number of products returned

  `sort_by=["name"|"price"]` - Determines the column to sort by, value has to be either `name` or `price`. An empty value will default to sorting by price.

  `sort_order=["asc"|"desc"]` - Determines the order the sort should happen, `asc` for ascending, `desc` for descending. An empty value will default to sorting in descending order.

- **Success Response** :

 - **CODE** : 200 OK<br />
   **Content** : <br />
   ```
   {
     "Count" : <integer count of number of records returned>,
     "Data" : [
      <List of products returned in the following format>
      {
        "Name" : <Name of the product>,
        "Price" : <Price of the product>
      }
     ]
   }
   ```

- **Error Response** :

 - **CODE** : 400 BAD REQUEST<br />
   **Content** : Error messages showing which of the query parameters are invalid.

 - **CODE** : 405 METHOD NOT ALLOWED<br />
   **Content** : Error message showing the incorrect HTTP method used.

 - **CODE** : 500 INTERNAL SERVER ERROR<br />
   **Content** : Error message throw by the internal error.

## Sample Outputs

 - **200 OK** :

 ```
 Query : http://localhost:8080/query?search_term=cheese&limit=3&sort_order=asc

 Output :
 {
      "Count": 3,
      "Data": [
      {
          "Name": "Kraft Mac and Cheese",
          "Price": 2.99
      },
      {
          "Name": "Philadelphia Cream Cheese",
          "Price": 3.49
      },
      {
          "Name": "Doritos Nacho Cheese",
          "Price": 3.99
      }
      ]
}
 ```

 - **400 BAD REQUEST** :

 ```
 Query : http://localhost:8080/query?min_price=15.50&sort_order=ascending

 Output :
 15.50 is not valid for min_price
 sort_order has to be either "asc" or "desc"
 ```

### Future considerations

 - AWS Access keys - AWS Key and Secret ID are in plain sight in the code, this itself should be stored elsewhere.
 - Authentication - Add some form of authentication to make sure access to this API is permitted.
 - Security - Any possible SQL injection or attack code was not stripped.
 - Unit tests - The tests currently only test local data manipulation functions and does not access any database or mocks.
 - Large data sets - Handle data sets larger than 1 MB.
 - Logging - Logging currently prints to standard out, should be moved to log file
