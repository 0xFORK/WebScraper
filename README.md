# A Simple Web Scraping API
![GitHub repo size](https://img.shields.io/github/repo-size/tes-id/WebScraper)
![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/tes-id/WebScraper)
![GitHub top language](https://img.shields.io/github/languages/top/tes-id/WebScraper)
[![Build Status](https://travis-ci.org/tes-id/WebScraper.svg)](https://travis-ci.org/github/tes-id/WebScraper)
[![Known Vulnerabilities](https://snyk.io/test/github/tes-id/WebScraper/badge.svg)](https://snyk.io/test/github/tes-id/WebScraper)
![GitHub last commit](https://img.shields.io/github/last-commit/tes-id/WebScraper)
![GitHub](https://img.shields.io/github/license/tes-id/WebScraper)

## Quick Start
This section contains the pre-requisite to run the application and how to use the API.

### Deploy on Heroku
To deploy this project on Heroku, click the button below:

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/tes-id/WebScraper)

### Or Simply wake the Dyno
[`Wake the Dyno`](https://tes-id-webscraper.herokuapp.com/)
It takes between 15 to 20 seconds to wake the Dyno, you will need a little patience


### Getting the Project

*Required*
* [`Maven`](https://maven.apache.org/) 3.3+
* [`JDK`](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 8+

*Optional*
* [`Postman`](https://www.getpostman.com/) - for testing the api endpoint

Get the project from the source repository
>`git clone https://github.com/Prempeh-Gyan/WebScraper.git`

### Running the Project
To run the project, first navigate into the source directory `cd WebScraper` and execute the following command:

* `mvn spring-boot:run`:

that's all you need to get it started.

The application starts the server instance on port `8080`.
> [`http://localhost:8080`](http://localhost:8080)

Open the link in your browser and start using it.

### Application Features
The main functionality of this API is to take a given url, navigate to this url, `crawl` the page and extract all `<a>` tags on the page.
Using the `href` attribute of the tags, the `urls` defined in the tags are extracted and processed for presentation.
The `urls` are grouped using their `host names`. A list of `host name - frequency` is then returned in [JSON](http://json.org/) format.

#### The RESTful API endpoint

> [`http://localhost:8080/summarizeLinksOnPage`](http://localhost:8080/summarizeLinksOnPage)

This is the `API-endpoint` from which you send requests.
Note that when you do a get request from the browser you will have to follow the `API-endpoint` with a `?url=someActualURL`
The `url` is the parameter you are passing to the `Web-Service` for processing. Hence an example of a full request to the `API-endpoint` will be
> [`http://localhost:8080/summarizeLinksOnPage?url=https://github.com/Prempeh-Gyan/WebScraper`](http://localhost:8080/summarizeLinksOnPage?url=https://github.com/Prempeh-Gyan/WebScraper)

You can also make a post request to the same `API-endpoint` in which case you will have to provide the `url parameter` as a `form-data`

### The Web Service
Below is the code snippet for the web service of the `API-endpoint`

file: `src/main/java/com/prempeh/webscraper/service/WebScrapingService.java`

file: `src/main/java/com/prempeh/webscraper/serviceImpl/WebScrapingServiceImpl.java`

```java
package com.prempeh.webscraper.service;

import java.io.IOException;
import java.util.Map;
/**
 * This is the WebScrapingService interface defining the action required to retrieve a summary of the links on a web page
 *
 * @author Prince Prempeh Gyan
 * @version 1.0 <br/>
 *          Date: 19/10/2017
 *
 */
public interface WebScrapingService {

	Map<String, Long> getSummaryOfLinksOnPage(String url) throws IOException;
}



package com.prempeh.webscraper.serviceImpl;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;
import org.springframework.stereotype.Service;

import com.prempeh.webscraper.service.WebScrapingService;

import lombok.extern.slf4j.Slf4j;
/**
 * This WebScrapingService Implementation takes a url, uses Jsoup to connect to the page,
 * extract links from the page and return a list of all the links on the page
 *
 * @author Prince Prempeh Gyan
 * @version 1.1 <br/>
 *          Date: 19/10/2017
 *
 */
@Service
@Slf4j
public class WebScrapingServiceImpl implements WebScrapingService {

	@Override
	public Map<String, Long> getSummaryOfLinksOnPage(String url) throws IOException {
		List<String> linksExtractedOnPage = new ArrayList<>();

		log.info("Jsoup is connecting to : {}", url);

		/**
		 * When Jsoup connects to the url, it parses the resource as an HTML Document
		 * and saves it into the "webPage" variable for use
		 */
		Document webPage = Jsoup.connect(url).get();

		log.info("Extracting anchor tags from {}", url);

		/**
		 * The Elements map to actual elements in the HTML document saved in the
		 * "webPage" variable. By calling the "select" method on the "webPage" variable,
		 * the links on the page that appear in the anchor tag can be extracted by
		 * passing "a[href]" as an argument to method "select". The result is a list of
		 * anchor tag elements saved in the "linksOnPage" variable
		 */
		Elements linksOnPage = webPage.select("a[href]");

		/**
		 * Once the elements have been extracted, they need to be processed into actual
		 * URIs. Java 8s streams and lambdas are used here to enhance performance. For
		 * the purposes of this application only the host names of the URIs are of
		 * interest at this level, the "schemes" and the "paths" are not necessary. All
		 * extracted host names are added to the "linksExtractedOnPage" variable. At
		 * this point the list may contain duplicate host names.
		 */
		linksOnPage.parallelStream().forEach(linkOnPage -> {

			try {

				URI uri = new URI(linkOnPage.attr("abs:href"));

				String link = uri.getHost();

				log.info("Tag <a href= '{}'>", uri);
				log.info("HostName = {}", link);

				linksExtractedOnPage.add(link);

			} catch (URISyntaxException e) {

				System.err.println("URISyntaxException : " + "url = " + linkOnPage.attr("abs:href") + "\nMessage = "
						+ e.getMessage());
				System.out.println("URISyntaxException : " + "url = " + linkOnPage.attr("abs:href") + "\nMessage = "
						+ e.getMessage());

			}
		});

		log.info("Returning list of HostNames to caller");

		return getSummary(linksExtractedOnPage);
	}

	private Map<String, Long> getSummary(List<String> linksOnPage) {

		log.info("List of HostNames recieved");
		log.info("Removing empty HostNames from list");
		log.info("Grouping identical HostNames and counting");
		log.info("Creating a Map with unique HostNames as key and their frequencies as values");

		/**
		 * Since Maps have unique keys, while the stream is being processed the filtered
		 * host names which have no empty or null elements are grouped into a Map of Key
		 * Value pairs, where the host names are saved as Keys and their corresponding
		 * frequencies are saved as values. The result is then returned to the caller.
		 */

		Map<String, Long> SummaryOfLinksOnPage = linksOnPage.parallelStream()
				.filter(link -> (link != null && !link.isEmpty()))
				.collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));		
		return SummaryOfLinksOnPage;
	}

}

```

## Graphical Overview of using the API

*Homepage*
![Homepage](https://i.imgur.com/qif3U9b.png)

*Using the Browser for sending requests*
![Homepage](https://i.imgur.com/Wh5tu7z.png)

*Sending GET Request through Ajax by button click*
![Homepage](https://i.imgur.com/5xicOMt.png)

*Sending POST Request to MVC Controller*
![Homepage](https://i.imgur.com/ifIQDX6.png)

*Sending GET Request through Browser with url parameter*
![Homepage](https://i.imgur.com/srYiZHn.png)

*JSON Response*
![Homepage](https://i.imgur.com/fKVtNaF.png)

*Using Postman to send GET Request*
![Homepage](https://i.imgur.com/6YqsiKX.png)

*Using Postman to send POST Request*
![Homepage](https://i.imgur.com/i5mL4GX.png)
