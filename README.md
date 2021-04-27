
## Password Hashing Application Execution

You must set a PORT environment variable before executing the application. It will crash otherwise.

$ export PORT=8088

## Password Hashing Application Specification

The following is the requirement specification that was used in building the password hashing application. It describes what the application should do.

* When launched, the application should wait for http connections.

* It should answer on the PORT specified in the PORT environment variable.

* It should support three endpoints:

   * A POST to /hash should accept a password. It should return a job identifier immediately. It should then wait 5 seconds and compute the password hash. The hashing algorithm should be SHA512.

   * A GET to /hash should accept a job identifier. It should return the base64 encoded password hash for the corresponding POST request.

   * A GET to /stats should accept no data. It should return a JSON data structure for the total hash requests since the server started and the average time of a hash request in milliseconds.

* The software should be able to process multiple connections simultaneously.

* The software should support a graceful shutdown request. Meaning, it should allow any
in-flight password hashing to complete, reject any new requests, respond with a 200 and
shutdown.

* No additional password requests should be allowed when shutdown is pending.

## Interacting with the Password Hashing Application Examples

  You can interact/test the application using curl. The following are examples that would/should generate similar returns - the job identifier does not need to conform to a specification.
* Post to the /hash endpoint

$ curl -X POST -H "application/json" -d '{"password":"angrymonkey"}'
http://127.0.0.1:8088/hash

42

* Get the base64 encoded password

$ curl -H "application/json" http://127.0.0.1:8088/hash/1

 zHkbvZDdwYYiDnwtDdv/FIWvcy1sKCb7qi7Nu8Q8Cd/MqjQeyCI0pWKDGp74A1g==

* Get the stats

$ curl http://127.0.0.1:8088/stats
  {"TotalRequests":3,"AverageTime":5004625}
 
* Shutdown

$ curl -X POST -d ‘shutdown’ http://127.0.0.1:8088/hash > 200 Empty Response

## Test Cases and results for testing the hashing application

1. Execute application without a port environment variable set

    *Expected Result: connection refused*

    *Actual Result: Error: connect ECONNREFUSED 127.0.0.1:8088*

    **Status: Pass**

2. Set port variable for PORT=8088

    *Expected Result: port is sucesfully set to port 8088*

    *Actual Result: port is set to 8088*

    **Status: Pass**

3. Run the application sucesfully 

    $ ./broken-hashserve_darwin 

    *Expected Result: When launched, the application should wait for http connections and connection is not refused*

    *Actual Result: Application launched sucessfully and waited for http connections*

    **Status: Pass**
  
4. Post to the /hash endpoint 

    $ curl http://127.0.0.1:8088/hash -X POST -H "application/json" -d '{"password":"password123"}' 
    
    *Expected Result: should return a job identifier immediately, It should then wait 5 seconds and compute the password hash in SHA512* 
    
    *Actual Result: Job Identifier took about 5 seconds to return as well as the password hash in SHA512 format*
    
    **Status: Fail**
    
5. Get the base64 encoded password 

    $ curl -H "application/json" http://127.0.0.1:8088/hash/1
    
    *Expected Result: It should return the base64 encoded password hash for the corresponding POST request*
    
    *Actual Result: base64 encoded passcode Example: NN0PAKtieayiTY8/Qd53AeMzHkbvZDdwYYiDnwtDdv/FIWvcy1sKCb7qi7Nu8Q8Cd/MqjQeyCI0pWKDGp74A1g==*
    
     **Status: Pass**
         
6.  Get Stats 

     $ curl http://127.0.0.1:8088/stats 
     
     *Expected Result: It should return a JSON data structure for the total hash requests since the server started and the average time of a hash request in milliseconds*
     
     *Actual Reslut: {"TotalRequests":2,"AverageTime":2556509} Epoch Conversion of the timestamp 2556509 = Fri Jan 30 1970 14:08:29 GMT+0000, incorrect timestamp year and not given in milliseconds*
     
      **Status: Fail**
      
7. The software should support a graceful shutdown request

    $ curl -X POST -d ‘shutdown’ http://127.0.0.1:8088/hash  
   
    *Expected Result: allow any in-flight password hashing to complete, reject any new requests, respond with a 200 and shutdown*
    
    *Actual Result: Able to submit an inflight request and shut down timestamp 2021/04/26 22:00:25 Shutdown signal received 
     2021/04/26 22:00:25 Shutting down*
     
      **Status: Pass**
      
8. Post to the /hash endpoint same password gives same base64 encoded password each time

   $ curl http://127.0.0.1:8088/hash -X POST -H "application/json" -d '{"password":"password123"}'
   $ curl http://127.0.0.1:8088/hash -X POST -H "application/json" -d '{"password":"password123"}'
   
   *Expected Results: the same base 64 password is returned for 1 and 2*
   
   *Actual Results: hash/1 = vtTvodT9vZVL03Bdaip4Jw7JpS7Pv7AQxhhir1x2rxdh/+sa72rKG/XQKzeBqoVPq9K2nHkN504X7P7Dy2rEvw==*
                   *hash/2 = vtTvodT9vZVL03Bdaip4Jw7JpS7Pv7AQxhhir1x2rxdh/+sa72rKG/XQKzeBqoVPq9K2nHkN504X7P7Dy2rEvw==* 
                   
    **Status: Pass**
    
9. Post a null or empty password 

   $ curl http://127.0.0.1:8088/hash -X POST -H "application/json" -d '{"password":""}'
   
   *Expected Result: incorrect or malformed input message?*
   
   *Actual Result: Job Identifier and a base64 is given for the null value 
    z4PhNX7vuL3xVChQ1m2AB9Yg5AULVxXcg/SpIdNs6c5H0NE8XYXysP+DGNKHfuwvY7kxvUdBeoGlODJ6+SfaPg==*
    
    **Status: Fail, assuming this application should not decode a null value**
    
 10. Enter a password with some kind of malformed input Example Sql Injection

   $ curl http://127.0.0.1:8088/hash -X POST -H "application/json" -d '{"password":"SELECT table_name FROM information_schema.tables WHERE table_schema =        'databasename'"}'  
   
   *Expected Result: Error message input is not allowed*
   
   *Actual Result: Job identifier and base64 is given for the SQL injection example O2ILtlsjxZBLu539bRMV4mugnN6HkenWctu0VwqV14Qn7AkOfVUz+DbrTAXJFdrDea8CjQMbo+FgN3jCuNSuWg==*
   
   **Status: Fail, assuming this application should not store a malicious input into a database**
   
   
11. post a username input to hash application

    $ curl http://127.0.0.1:8088/hash -X POST -H "application/json" -d '{"username":"johndoe1@fakemail.com"} 
    
    *Expected Result: Malformed input message*
    
    *Actual Result: Job identifier and base64 is given for a username input z4PhNX7vuL3xVChQ1m2AB9Yg5AULVxXcg/SpIdNs6c5H0NE8XYXysP+DGNKHfuwvY7kxvUdBeoGlODJ6+SfaPg==*
    
    **Status: Fail Json input id not checked for "password"** 
    
12. Get status now should show a result of 5 now (Known bug from above for time stamp)

    *Expected Result: 5*
   
    *Actual Result: 5*
    
    **Status: Pass**
    
13. Enter a long repeated list of data brute force example

<img width="1263" alt="Screen Shot 2021-04-27 at 12 07 34 AM" src="https://user-images.githubusercontent.com/74283259/116193478-ad9a5f00-a6ec-11eb-8c49-ba180974c61b.png">

   *Expected Result: hash should be blocked and not created*
   
   *Actual Results: no base 64 encoded password is given not sure why the identifier shows as quote see above image*
   
   **Status: Pass**
   
14. Validate that special characters and symbols are available in a password  

    $ curl http://127.0.0.1:8088/hash -X POST -H "application/json" -d '{"password":"passworD1!2@3#"}'
    
    *Expected Result: password is allowed and base64 encoded password is created*
    
    *Actual Result: password is allowed and base64 encoded password is created BQ7hKpz6AZvCU4vCGV5RJGn34DsHS9TyIKTeAU7TuiWW7CUvcqfLKLQpcU/g6UIl3KHOKSrW4dLm6L9YXUGYVg==*
    
    **Status: Pass**
    
    
15. The software should be able to process multiple connections simultaneously

    $ curl http://127.0.0.1:8090/hash -X POST -H "application/json" -d '{"password":"passworD1!2@3#"}'
    
    $ curl http://127.0.0.1:8091/hash -X POST -H "application/json" -d '{"password":"passworD1!2@3#"}'
    
    $ curl http://127.0.0.1:8092/hash -X POST -H "application/json" -d '{"password":"passworD1!2@3#"}'
    
    $ curl http://127.0.0.1:8093/hash -X POST -H "application/json" -d '{"password":"passworD1!2@3#"}'
    
    
    *Expected Results: Able to connect and process all connections and return and identifier and same base64*
    
    *Actual Results: Connected to all ports and submitted same password to all conections return Identifier as 1 on all ports and base64 BQ7hKpz6AZvCU4vCGV5RJGn34DsHS9TyIKTeAU7TuiWW7CUvcqfLKLQpcU/g6UIl3KHOKSrW4dLm6L9YXUGYVg==*
    
     **Status: Pass**
    
    
## Assumptions and bug summary 

* Password will be steralized with regex before reaching the application

* Password hash application should not allow any malicious inputs into a database (Sql Injection)

* Null value should not be allowed as a password

* Usernames should not be hashed 

* Stats timesstamp Epoch Date is wrong gives date as 1970 and time stamp not given in milliseconds

* Job identifier takes the same amount of time as timestamp 5 seconds, this should happen immediately


## Ideas to automate the regression testing of the hashservice

Create a collection and tests using postman, this can be added to any ci pipeline and run for E2E tests.

* Postive tests, get & post request with body and status 200 as assertions

* Regression tests previous bug fixes

* Negative tests or expected failures
 with regex before reaching the application

* Password hash application should not allow any malicious inputs into a database (Sql Injection)

* Null value should not be allowed as a password

* Usernames should not be hashed 

* Stats times stamp Epoch Date is wrong gives date as 1970 and time stamp not given in milliseconds

* Job identifier takes the same amount of time as time stamp 5 seconds, this should happen immediately
