# http stress tester
HTTP Test case runner and Stress Test Utility

*  Ability to run from 1 to hundreds of concurrent requests across a wide variety complex use cases.
* Uses easily edited text format based on JSON to specify URI, METHOD, BODY and response matching.    [sample test script](data/sample-1.txt)
* Fully supports parallel execution with semantics to force all operations to finish before it starts the next stage of testing which can be required when some tests must complete before others can be started. 
* Will show the test ID and status for every test ran making it easy to integrate into a CICD pipeline.
* Provides RE matching of HTTP response and body to validate good responses.
* Provides RE no match to check proper filtering. 

###How it Compares

* Compares to [newman portion of postman](https://github.com/postmanlabs/newman) but supplies superior multi-threaded performance or stress testing.    
* Provides free and higher performance stress test than  [load runner](https://www.microfocus.com/en-us/products/loadrunner-load-testing/overview)

[httpTest](src/httpTest.go) provides a data driven, multi threaded test client able to support running at many threads for a while then waiting for all concurrent threads to finish before starting the next test.  This provides basic support for read after write tests.   It also provides easily parsed output that can be used to feed test results into downstream tools.  

The input to Generic Test client is a text file containing a series of JSON strings that describe each test.   It includes a few directives such as #WAIT to indicate a desire to wait for all prior requests to finish.   Comment strings are lines starting with #WAIT

## Local Build / Setup

Once you have the  [golang compiler](https://golang.org/dl/) installed.    

```
go get -u -t "github.com/joeatbayes/http-stress-test/httpTest"
```

It will create a executable in your GOPATH directory in bin/httpTest.  For windows it will be httpTest.exe.  If GOPATH is set the /tmp then the executable will be written to /tmp/bin/httpTest under Linux or /tmp/bin/httpTest.exe for windows. 

Once you build the executable you can copy it to other computers without the compiler. 

HINT: set GOTPATH= your current working directory.  Or set it to your desired target directory and the go get command will create the executable in bin inside of that directory which is good because you may not have write privileges to the default GOPATH.

##### To Download all pre-built test cases, scripts and sourcecode

```
git clone https://github.com/joeatbayes/http-stress-test httpTest
```

You could also just save the [sample script](https://raw.githubusercontent.com/joeatbayes/http-stress-test/master/data/sample-1.txt) using your browser or curl and edit it to use the executable built above. 

## Assumptions

- Unless specified otherwise assumes all requests can be completed in any order which may in fact happen since they are ran in a multi threaded fashion.     

## Command Line API

```
httpTest  -in=data/sample-1.txt -out=test1.log.txt  -MaxThread=100
```

​    

> > - Runs the test with [data/sample-1.txt](data/sample-1.txt) as the input file.    
> > - Writing basic results to test1.log.txt 
> > - Runs with 100 client threads.
> > - **Parameters**
> > - **-in** = the name of the file containing test specifications.
> > - **-out** = the name of the file to write test results and timing to.
> > - **-MaxThread** = maximum number of concurrent requests submitted from client to servers.

## File Input Format

```json
# Test simple index page contains expected content.
{  "id" : "0182772-airSolarWater-dem-index-contains-solomon",   
   "verb" : "GET", 
   "uri" : "https://airsolarwater.com/dem/", 
   "expected" : 200, 
   "rematch" : ".*Solomon.*Tuvalu.*Tesla", 
   "message" :"Air solar Water index must contain island names"
}
#END

# Test Google page contains expected text
{  "id" : "0182-google-home-contains-search",   
   "verb" : "GET", 
   "uri" : "https://google", 
   "expected" : 200, 
   "rematch" : ".*Search\<\/div\>.*Google", 
   "message" :"Air solar Water index must contain island names"
}
#END

```



- A series of lines containing JSON text which represents the specifications for the test 

- Each JSON string is terminated by a #END starting a otherwise blank line. 

- **[Sample-1](data/sample-1.txt):**

  Test ID = ID to print upon failure
  HTTP Verb = HTTP Verb to send to the server
  URI =  URI to open for this test 
  Headers = Array of Headers to send to the server
  rematch = RE pattern to match the response body against. Not match is failure.
  renomatch = RE pattern that must not be in the response data.

   expected = HTTP Response code expected from the server. 

  ```
          other response codes are treated as failure.
  ```

   body = Body string to send as Post Body the server 

- Blank rows are ignored

- Rows prefixed by # are treated as comment except when #WAIT or #END

- Rows Prefixed with "#WAIT" Cause the system to pause and wait for all previously queued requests to complete before continuing.  This can allow blocking to allow data setup calls to complete before their read equivalents to complete.

- Test Requests  can be read from file and executed in parallel threads unless blocked by #WAIT directive.

- HTTP VERBS SUPPORTED  GET,PUT,DELETE,POST

- HTTP Headers URI Encoded sent in order specified but this can not be guaranteed since it is treated internally as a map which does not guarantee ordering. 

- POST BODY IS URI Encoded in file but will be decoded prior to POSTING TO Test client.

> ## Example Output

```
GenericHTTPTestClient.go
Finished Queing
 took 0.000033 min
Finished all test records
 took 0.000067 min
FAIL: L128: elap=382.978ms       id=0182-DEM-CGI-CALL-GEOPOINT-EXPECT-FAIL      message=Checking CGI Geo Point  err Msg= L107:failed rematch= -19.71542,63.34569,1.00.*-19.71736,63.XXX34514
        verb=GET uri=http://airsolarwater.com/dem/drains.svc?buri=gdata/dnt-rodrigues-island-50&offset=1591792&geo=-19.71542,63.34569

SUCESS: L125: elap=383.976ms     id=0182-DEM-CGI-CALL-GEOPOINT message=Checking CGI Geo Point
SUCESS: L125: elap=482.708ms     id=0182-google-home-contains-search    message=Google home must contain 'search' followed by 'div' followed by 'Google'
SUCESS: L125: elap=520.611ms     id=0182772-airSolarWater-dem-index-contains-solomon    message=Air solar Water index must contain island names
numReq= 4 elapSec= 0.5245668 numSuc= 3 numFail= 1 failRate= 0 reqPerSec= 7.625339613563039
```



## Important Files

- [data/sample-1.txt](data/sample-1.txt) - Sample input data to drive some simple tests

- [actions.md](actions.md) - list of feature enhancements under consideration.  Roughly listed in order.

- [httpTest.go](src/httpTest.go) - GO source code for main driver supporting this test.

- [makego.bat](makego.bat) - windows batch file to build the httpTest executable

- [makego.sh](makego.sh) - linux shell script to build the httpTest executable

  [goutil github repository](https://github.com/joeatbayes/goutil) This code requires code that will be automatically downloaded when building this too.

## Local  Development Build

####  Download the Metadata server repository

```
git clone https://github.com/joeatbayes/http-stress-test httpTest
```

```
# Make your currenct directory the directory where you
# downloaded the repository
# cd  RepositoryBaseDir 
cd \jsoft\httpTest
# Use forward slash on 
# This is the directory where you downloaded the repository.   
```

​	

```
go get -u "github.com/joeatbayes/goutil/jutil"
go build src/httpTest.go
```

  or  

```
# Windows
makeGO.bat

# Linux
makeGO.sh

```

### Build to a specific location Direct from Repo

```
go get -u -t "github.com/joeatbayes/http-stress-test/httpTest"
go build -o /tmp/httpTest.exe "github.com/joeatbayes/http-stress-test/httpTest"
# Requires sucessful execution of the go get command above.
```

This will build a new executable and place it at the location specified in the -o. For windows the .exe extension is needed for Linux leave it off.  This will be a duplicate of the executable built during the go get command so it is probably better just move the one built by go get to a location the search path.

## Some of my other repositories:

- [file2consul](https://github.com/joeatbayes/file2consul)  Utility to load configuration parameters managed in GO into consul or HTTP server.  Supports inheritance,  parameter interpolation and other advanced techniques to minimize manual editing required to support multiple environments. 
- [GoPackaging](https://github.com/joeatbayes/GoPackaging) - Example of how to package a library for direct use from go command line.  Also shows an example program that uses that library.   
- [DevOps Automation with LXD and LXC containers](https://lxddevops.com/) - Scripts to create images that can be booted at will.  Demonstrates layered image creation.  Scripts to launch images,  map ports and to setup layer 5 routing for images ran across many hosts.  Consider an alternative to OpenStack or Kubernetes although it could run on OpenStack.    
- [Quantized Classifier](https://bitbucket.org/joexdobs/ml-classifier-gesture-recognition) -  Advanced machine learning classifier with full examples.  Very fast and competitive for both accuracy and and recall with tensor flow for many problems.  In some instances runs over 50 times faster than tensor flow with comparable precision. 
- [Metadata server MDS](https://bitbucket.org/joexdobs/meta-data-server) A server for storing arbitrary data by ID with very fast retrieval.   Ideal for use with very large data sets when very large scaling is required.   Compare to redis, memcache, memcacheb, riak, but designed to support very high scale on reasonable memory and can handle data sets of many T with good performance.   Pure HTTP based.  Written in GO.   Optimized along line of consuming a queue to keep multiple readers updated in near real-time.   Any single server can guarantee updates but servers as set are still eventual consistency. Measured at supporting over 16K requests at 4K per second  across a set of millions of records sustaining this load for months without degradation.
- [Computer Aided Call Response Engine](https://bitbucket.org/joexdobs/computer-aided-call-response-engine) Supports non technical user definition of call scripts that can be arbitrarily complex scripts. Based on JavaScript to integrate nicely with most RIA applications.  Features script tracking,   Local data storage,  spooling update events to server,   save and restore context for different users to allow call switching. etc.   Simple text based syntax to define the call tree allows rapid deployment and maintenance by non-programmers.
- [Healthcare provider Search](https://bitbucket.org/joexdobs/healthcare-provider-search)  Search UI that provides physician locator functionality.   Scripts to parse and load with over 1.2 million records from CMS.    Allows geo-proximity filtering,  zip to city resolution, etc.   Based on elastic search with node.js middle tier server.  Demonstrates very fast RIA JavaScript which allows hundreds of records faster that many sites render a one.   Take a close look if you want to understand high performance JavaScript.
- [CSV Tables in Browser](https://github.com/joeatbayes/CSVTablesInBrowser) Render CSV files in a browser automatically with little to know code.  Renders them fast and nice looking with automatically repeated headers.   Supports sorting,   Custom formatting by column and can even allow script callback to allow custom data generation for some fields.    Can dramatically reduce work to render large sets of columnar data. 

## License:

Copyright 2018 Joseph Ellsworth  [MIT License](https://opensource.org/licenses/MIT) 