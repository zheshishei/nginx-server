# Queue Server

##Introduction

This is a container-based server that exposes two endpoints:

  1. */write* - accepts a POST request in the form of a json object {"text" : <YOUR TEXT HERE> }.  
  2. */read_* - accepts a GET request and returns a piece of text.

*/write* and */read* together create a queue. You can post text to */write* and then 
retrieve the oldest text through */read*.

##Architecture

The server is comprised of four containers, which are:
  
  1. __redis -__ Redis is used as the datastore that holds the posted text. It also provides atomic reading/writing
  2. __writer-server -__ The writer-server is the server application that handles requests to /write
  3. __reader-server -__ The reader-server is the server application that handles requests to /read
  4. __nginx-server -__ The nginx-server acts as a reverse proxy to forward the requests to the 
                        correct server, depending on the path.


##Setting up

To set up a Queue Server, follow these steps:

 1. Make sure your machine has Docker installed. To see if you have Docker installed, type `docker`. 
If you don't have Docker, check the [Docker Installation Docs](https://docs.docker.com/installation/)
for your specific OS.

 2. Start the Docker Daemon

 3. Pull and run a Redis instance:

        sudo docker run -d --name redis redis

 4. Run the Reader-server and Writer-server:
    
    a. Pull and run the Writer-server:

        sudo docker run -d --name writer-server --link redis:redis zheshishei/writer-server

    b. Pull and run the Reader-server:

        sudo docker run -d --name reader-server --link redis:redis zheshishei/reader-server

 5. Pull and run the Nginx-server:

        sudo docker run -p 80:80 -d --name nginx-server --link reader-server:reader-server --link writer-server:writer-server zheshishei/nginx-server


##Notes on Errors/Edge Cases and Other things

 * The /write endpoint only accepts well-formed json
 * The /write endpoint only accepts requests that have a content-type of "application/json"
 * Maintaining the applications may get slightly hairy since every new version would require building a new docker image.
   This could be solved by creating a container with a start script that clones a repository and then runs the code
 * There's no logging being done
 * The above steps could be replaced with a single script
 * If a container goes down, then it has to be manually restarted
