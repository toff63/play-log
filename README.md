#Spring-xd Hackhaton

Download spring xd http://repo.spring.io/simple/libs-milestone-local/org/springframework/xd/spring-xd/1.0.0.M4/spring-xd-1.0.0.M4-dist.zip

Unzip it and then create an environment variable: XD_HOME pointing to the directory where you unzipped it.

Run:

    $ xd/bin/xd-singlenode
    
In another console run
    $ shell/bin/xd-shell


Our application is generating logs like 

    {"action": "controllers.navigation.RemoteAssets.at" ,"path": "/assets/nav/1401921699948/images/red_header.jpg" ,"time": 4 , "status": 304}
    
We creating a stream from the application log file only keeping lines that are jsons:

    stream create --name playlogs --definition "tail --name=/tmp/xd/input/application.log | filter --expression=payload.startsWith('{') |log" --deploy
    
Then we created a stream to count the number of time each http status code the application returned:

    stream create --name httpStatusCode --definition "tap:stream:playlogs.filter > field-value-counter --fieldName=status" --deploy
    
We have done the same to count the frequency each url is being used

    stream create --name mostUsedUrl --definition "tap:stream:playlogs.filter > field-value-counter --fieldName=path" --deploy
    
    
We played a bit with the application and could find the results:

At url http://localhost:9393/metrics/field-value-counters/mostUsedUrl

    {
    	"name": "mostUsedUrl",
    	"links": [
    		{
    		"rel": "self",
    		"href": "http://localhost:9393/metrics/field-value-counters/mostUsedUrl"
    		}
    	],
    	"counts": {
    		"/fruit/1": 1,
    		"/reports/supplier": 1,
    		"/home": 4,
    		"/fruits": 1,
    		"/fruits/list": 3,
    		"/suplier/list": 1,
    		"/supliers": 1,
    		"/supliers/autocomplete": 2
    	}
    }

At url http://localhost:9393/metrics/field-value-counters/httpStatusCode


    {
    	"name": "httpStatusCode",
    	"links": [
    		{
    		"rel": "self",
    		"href": "http://localhost:9393/metrics/field-value-counters/httpStatusCode"
    		}
    	],
    	"counts": {
    		"200": 12,
    		"303": 2
    	}
    }



The problem spring-xd currently have is that it simply kill the streams when any bad event occur. Any json parsing error will kill the stream trying to parse and the stream being parsed. No logs tell it, so we had to discover it the hard way.
