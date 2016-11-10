This repo contains the HW3 project i.e. **Code Search Engine** based on Elastic Search running in Google Cloud.

Project Members are: Abhijay Patne , Kruti Sharma, Sandeep Singh


-------------------------------------------------------------------------------------------------------

Below is a description of the working of this project, followed by description of the project structure:

Project Working:

1. Elastic Search has already been set up in Google Cloud as a preprocessor for this project's working.

2. We download projects from Github through the Ohloh API and save them locally. 
We also obtain some project metadata from the XML response coming through API calls to Ohloh.

3. These downloaded projects are indexed into the Elastic Search. We have used two indexes, language and tags. These index types have different mapping identifier like project name, file name, project description, file content.
 
4. Now we create a web service which creates a client to our elastic search server. When the user makes rest calls involving search queries to our web service, the web server runs that search accross elastic search,
 and outputs the response to the user throguh the browser.
 
 
-------------------------------------------------------------------------------------------------------

**Project Structure:**

Scala main classes(/src/main/scala/):

1. **OhlohDownlStrMultiActor.scala**:

This file contains the code for streaming projects from OHLOH REST API and download each of the projects on the local directory. Once the entire projects are downloaded on the local folder, then the files from each of the downloaded folders are processed for indexing on the Elastic Search Engine. The files are indexed under **languages** - this is the main index for our search engine. Now each of the file extensions are used as the type of the index and the mapping contains the project name, project download url, project description, file name and the content from the file. Only code related files i.e. files with extensions as java, cs, js etc are sent for indexing.

The following actors are used for processing the streams, downloading the projects, extracting the bag of words from each of the code files
and finally call the create index to process the index for each of the files.

**StreamDownlESProject** - this is the main object which calls the other actor to start the processing.

**StreamDownlProjects** (main acotor) - this actor is responsible for following operations as defined by cases in its receive:

**1. StartStream(ohlohURL)** => this is the entry point for our program and passing message to **StreamDownlProjects actor** with this case class starts the streaming of the projects from OHLOH REST API. The URL used for streaming projects from OHLOH API are filtered to return projects contanining **github** in their
name or description or home page url which gives us the flexibility to download the repositories of github. This rest call returns an XML containing 10 projects information.

**2. StartParsing(allFiles,projectName,projectDesc,projectURL)** => this case class is used for parsing the files content into bag of words. The content from the file are read and all special characters except underscore are replaced, so now the content is bag of words and this content is associated with each indexed file which is stored in "content" mapping type field in the index created for the file type.

**ESClient** (2nd Actor) => this actor is responsible for making the Scala client for elastic search engine at : "104.197.155.244" and port : "9300". Once the client is instantiated, this client can be used for creating and quering indexes on the elastic search engine. The actor has the following case classes:

**1. StartDownloading(projectId,projectURL,projectName,projectDesc)** => this case class is responsible for downloading the github repository from the url as obtained by streaming each project present under homepage_url in the project directory with folder name as TestGitRepository_{project_id}

**2. StartFileIndex(projectName,projectDesc,fileName,fileExt,content,mappingType,projectURL) =>** this case class is responsible for indexing each file i.e. create index for each of the code file with index name = "languages", type = "java" (this is basically the extension of the file which denotes which code file that is being indexed), mapping = contains the project name, project url, project description, and content - which is the bag of words as extracted from the file (which has no special characters except _)

**3. createTagIndex(projectId, projectName, projectURL, projectDesc, projectTags)** => this case class is responsible for creating indexes for tags as obtained by processing the projects xml which may contain tags associated to each project. This helps in creating another search index for our search engine.

Overall message passing within actors for Streaming, Downloading, Parsing and Indexing the files on Elastic Search Engine => 

**StreamDownlProjects(Main Actor) ! StartStream** -------pass the download url of project to this actor ------> **ESClient (2nd Actor) ! StartDownloading** ----- extract the bag of words from the downloaded files ------> **StreamDownlProjects(Main Actor) ! StartParsing** -----start indexing each of the files-----> **ESClient(2nd Actor) ! StartFileIndex**


-------------------------------------------------------------------------------------------------------

**Unit Testing for OhlohDownlStrMultiActor.scala => StreamingDownlnitTest.scala**

This unit testing perform the following unit and integration testing:

1. **test("Test Streaming and Downloading")** =>  this test case tests whether the user can stream a single project url from OHLOH and do indexing of the project and files. The assert shows the project index created on the elastic search engine for the streamed project.

2. **test("Test Downloading projects")** => this unit test case tests whether the files as received from github from the streamed project are actually downloaded on the local or not. The assert checks the project directory is downloaded on the local folder or not.

---------------------------------------------------------------------------------

**2. SearchEngine.scala** 

We first create an actor system and create 3 actors: discardStopwords,ParseResponse and Fetch. These actors have mutiple cases amongst to handle different cases. Next, we create a web service binded to localhost on port 8080, with some handlers pre-defined. These handlers specify the different kinds of inputs that the web service can process. Depending on the rest call, the different parameters are retrieved and their values stored. These values are passed on to the actor Fetch for fetching response from Elastic Search..

**Actors:**

**discardStopwords** : In case a search keyword is also entered in the rest call, first discardStopwords, which removes common words in programming languages which can be discarded. Then filtered keywords are passes back to main class which passes all parameters to the actor Fetch, as mentioned previously.
			
**Fetch** : This actor uses the parameters specified by user, to make a rest call using Elastic Search's Search API. The response from Elastic Search is a json, which contains some meta data about the search, along with matching indexes for the search. This json response is passes onto the ParseResponse actor.
					  
**ParseResponse** : This actor parses the json response, and gets the value for fields like: total time taken, total number of hits, proejct names, file names etc. These values are properly formatted into a string and sent back to the main class, which displays the information in the browser to the user.
			
**Scala test classes** (/ src / test / scala /):
	
**SearchEngineTest.scala**: This test case is a unit test for the SearchEngine class. The test cases first creates the web service using SearchEngine.scala. The test cases waits for 10 secs, so that the web service is created and started using SearchEngine.scala. 

There are two test cases in this testcase file and both make rest calls to the web service and check its response.

-------------------------------------------------------------------------------------------------------

**How to run the project:**

1. Clone the repo and import the project into IntelliJ using SBT.

2. Both classes can be run individually, OhlohDownlStrMultiActor.scala and SearchEngine.scala, as they are independent. Right click on OhlohDownlStrMultiActor.scala and Run -> **StreamDownlESProject** - this is the pre-processing of index on the elastic search engine deployed on google cloud. Noew SearchEngine.scala -> Right click and run **SearchEngine** to run the code search engine and then use the search engine url in your browser.
  
3. After the web service is created, the URL to access it is http://localhost:8080
(This is specified in SearchEngine.scala ) 

   Instructions to use the web service created(These instructions can also be found when you browse to http://localhost:8080 using a browser:

    Note: If count is not specified, by default 5 search results are obtained 

       **Language based search**
   
     1. Searching all projects for a specified langugage(for example, java)
        Extension: ?language=java 
   
     So you enter the URL: http://localhost:8080/?language=java
   
     Some of the langugages used in the indexed projects are: Java, Go, PHP, JS, CSS, HTML, XML
   
     2. Searching all projects for all languages:
      Extension: language=all
    
     3. If you want to specify count(eg. 5) along with above parameters, just add the parameter(add below extension preceded by '&'):
        Extension: count=5
   
      So you enter the URL: http://localhost:8080/?language=java&count=5
   
     4. Other parameters possible are(only language is necessary, all others optional, all parameters take only single values and can be written in any order):
         Extension: projectname=eclipse
         Extension: keyword=search_term
   
   
      **Tag based search**
      You can also search all projects by specifying a tag. Tags are added to all projects. This search will return the project containing the specified tag in their tag list
   
      Use below query to give a tag to search in all project tags:
   
      http://localhost:8080/?tags=enter_tag_here
   
      This tag parameter cannot be combined with any other language based search parameters.
   
      Some example queries:
   
      http://localhost:8080/?language=java
      http://localhost:8080/?language=java&count=1
      http://localhost:8080/?language=java&projectname=eclipse
      http://localhost:8080/?language=java&projectname=eclipse&count=2
      http://localhost:8080/?language=java&projectname=eclipse&keyword=connector
      http://localhost:8080/?tags=security
  

------------------------------------------------------------------------------------------------------- 
**Limitations:**
1. Our search parameters can only contains single terms. Mutiple search values in the same parameter are not handled.
2. The web service we created is not hosted on the cloud, only runs locally.
3. The projects streamed for processing and creating index on elastic search are only Github projects, so the rest call made to Ohloh has the projects filtered for github and hence only works for projects which have a valid download url in their xml response.

-------------------------------------------------------------------------------------------------------
**Load Tests:**

The load test was performed using SOAPUI - LOADUI NG. The test case contained 4 queries : 

http://localhost:8080/ -- checking if the search engine is available

http://localhost:8080/?language=java&projectname=eclipse -- search based on language and project name

http://localhost:8080/?language=java -- search simply based on language

http://localhost:8080/?tags=security -- search simply based on tags

The number of VUs selected were : 2, 4, 6, 8 and 10. The average response time and the associated charts are present in the folder **SOAPUI Load Test Results**. The folder has analysis report for each of the VUs, chart for VUs = 10 and the consolidated chart for average response time vs number of VUs.

-----------------------------------------------------------------------------------------------

**Google Cloud details:**

**References:** present in "References.docx"