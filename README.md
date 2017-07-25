6.	Proposed System 
6.1	System Architecture
This chapter will explain the proposed system architecture in detail. It also explains the algorithms used to implement search engine.  
 
Fig 6.1. Proposed System Architecture
We have integrated web crawler, inverted indexing and page ranking for building our search engine. The diagram above shows the system architecture. The main components of the architecture are Web Crawler, Indexer, Page Ranker, Retriever, Searcher and GUI. Web crawler continuously renders the videos available on the internet and downloads the metadata which is required for process of indexing. Indexing retrieves the useful data required for searching relevant videos. Web Crawler also creates a directed graph of videos. This graph is given as input to the page ranking algorithm. This algorithm ranks the videos according to their importance. GUI takes the user query and is also used to display the search results. The query given by user is submitted to Searcher. This searcher uses the inverted index creates by indexer and also the ranking given by the page ranker. It combines both output and creates a result according to relevant and page ranks. This search result is submitted to GUI, which shows the search result to user. We have implemented, Web crawler module using breadth first search algorithm, Indexer module using inverted index algorithm and ranking module using page ranking algorithm. All these algorithm is implemented using Hadoop MapReduce framework.
We analyzed different graph traversing algorithm for using in web crawler module. Breadth first search, depth first search are two main algorithms which we analyzed in depth for implementing web crawler. As the websites are connected using hyperlinks, we can create a directed graph and can use the graph traversing algorithms for rendering websites over the net. The websites can be considered as the nodes of the graph. Web crawler is also responsible for downloading the web pages. We faced some issues while downloading the webpages. We tried to use different mechanism for downloading the webpages as text documents. We finally decided to go ahead with breadth first search algorithm and used JSON libraries for parsing the web pages and extracting the hyperlinks present in that web page. This hyperlink is then used to create a directed graph. This directed graph is given as input to the ranking algorithm. There are number of ranking algorithm present and we decided to analyze some of these algorithms which will suffice our need. TFIDF (Term Frequency Inverse Document Frequency), Word count, Page Ranking are the algorithms we analyzed for ranking search results. Out of those we found Page Ranking algorithm to be most matching for our requirements. So, we implemented ranking module with Page Ranking algorithm. We analyzed the forward and inverse index algorithm while working on indexer module. Inverted was the one which seems to fulfill our requirement of the module. So, we created inverted indexes over the document which we downloaded using web crawler. Thus, we integrated all three modules.   
Web Crawler: 
Prior to creating our own web crawler, we decided to use CommonCrawl. CommonCrawl is nonprofit organization which is responsible for creating, maintaining and making the data crawled by the CommonCrawl available publicly. CommonCrawl is used by lot of researcher for their research work in ranking and indexing the data. While doing so, they don’t need to bother about crawling the data. Gil Elbaz, former Google employee funded Common Crawl. CommonCrawl is implemented using Hadoop MapReduce Technology. Their strategy is to crawl broadly and frequently, to cover all the web pages. The crawling was based on the rank and date on which the web page was modified. After crawling is done, the crawled data is uploaded to S3 cluster on amazon cloud. This s3 is made available publicly for users. Libraries are built using Hadoop technology for supporting S3 data. Depending upon the customers, metadata and other services were started. This crawler doesn’t guarantee the comprehensive crawling of the web. Crawlers store the data into Hadoop Based File System (HDFS). Map-Reduce jobs parsed the web pages and fetched the metadata from the pages required to complete the crawling and ranking pages. This system is very useful for research purpose. Researchers who are not working on web crawling and don’t want to concentrate on that area can directly utilize the data provided by Common Crawl. 
I started with building Page Rank module before building my own web crawler. So, I had used CommonCrawl data for testing my module. After completing page rank module, I decided to implement inverted index algorithm. This algorithm also used data from CommonCrawl. 
Once I was done with developing rest of the modules, I decided to build my own web crawler. We start crawling with one URL, called as seed. Thus, we can consider web crawling problem as a tree traversal problem. I analyzed Breadth First Search (BFS) and Depth First Search (DFS) algorithms for implementing web crawler. Breadth First Search traverse all the nodes level wise, one level at a time. Then moves ahead with another level. While DFS works in different fashion. It goes on traversing till we reach to leaf node, starting with root node. Considering the unpredictability of number of levels, or the number of web pages available over web, I decided to implement web crawler using Breadth First Search algorithm. 
I started developing web crawler using core java. Firstly, I implemented the BFS algorithm to traverse limited number of nodes. I was doing it to verify correctness of my implementation. Once I was sure that my BFS is working properly, I added the logic to store the web pages into the local file system. I have used jsoup library for parsing html files. I retrieved hyperlinks from those html pages and added those links to the url lists which we maintain for traversing the web. This program was implemented using only core java and was a sequential program. To decrease the response time, we decided to implement the same module using multithreaded environment. We created a pool of threads which was used to traverse the web pages and download the files. Multithreading helped us to reduce the response time, but it was not meant for distributed systems. So, eventually we implemented web crawler using Hadoop MapReduce. I was having little knowledge of this technology. So, I tried to learn this technology using some simple problems such as word count problem. In this problem, we count the frequency of each word in each document. MapReduce works with text files. It takes text file as input and stores the output in a text files. All the data storage is done using Hadoop Distributed File System (HDFS). Once the word count solution was successfully implemented we started to convert core java implementation of web crawler to MapReduce implementation.         
Parallel breath first search:
Pseudo-code for parallel breath first search in MapReduce: the mappers emit distances to reachable nodes, while the reducers select the minimum of those distances for each destination node. Each iteration (one MapReduce job) of the algorithm expands the search frontier" by one hop.
1: class Mapper
2: method Map(nid n; node N)
3: 	d   N:Distance
4: 	Emit(nid n;N) . Pass along graph structure
5: 	        for all nodeid m 2 N:AdjacencyList do
6: 		Emit(nid m; d + 1) . Emit distances to reachable nodes
1: class Reducer
2: method Reduce(nid m; [d1; d2; : : :])
3: 	dmin   1
4: 	 M ;
5:          for all d 2 counts [d1; d2; : : :] do
6: 		if IsNode(d) then
7: 			M   d . Recover graph structure
8: 		else if d < dmin then . Look for shorter distance
9: 			dmin   d
10: 			M:Distance   dmin . Update shortest distance
11: 			Emit(nid m; node M)
Mapper source code
  
Reducer source code 
Reducer continued 
    Retriever: 
Retriever module is integrated with web crawler module. Retriever is responsible for fetching the links from the web pages using parsing mechanism. Every time a new url is found it was checked if we have already visited it or not. If that link is unvisited, then that url is added to the url lists. It also provides the urls from the url list, one url at one time.  
Page Ranker: 
One iteration of PageRank requires two MapReduce jobs. First is used to distribute PageRank mass along graph edges, and the second to take care of dangling nodes and the random jump factor. At end of each iteration, we end up with exactly the same data structure as the beginning, which is a requirement for the iterative algorithm to work. Also, the PageRank values of all nodes sum up to one, which ensures a valid probability distribution. Typically, PageRank is iterated until convergence, i.e., when the PageRank values of nodes no longer change (within some tolerance, to take into account, for example, floating point precision errors). Therefore, at the end of each iteration, the PageRank driver program must check to see if convergence has been reached. Alternative stopping criteria include running a fixed number of iterations (useful if one wishes to bound algorithm running time) or stopping when the ranks of PageRank values no longer change. The latter is useful for some applications that only care about comparing the PageRank of two arbitrary pages and do not need the actual PageRank values. Rank stability is obtained faster than the actual convergence of values.
Pseudo-code for PageRank: 
Pseudo-code for PageRank in MapReduce (leaving aside dangling nodes and the random jump factor). In the map phase, we evenly divide up each node's PageRank mass and pass each piece along outgoing edges to neighbors. In the reduce phase PageRank contributions are summed up at each destination node. Each MapReduce job corresponds to one iteration of the algorithm. Pseudo-code for Dijkstra's algorithm, which is based on maintaining a global priority queue of nodes with priorities equal to their distances from the source node. At each iteration, the algorithm expands the node with the shortest distance and updates distances to all reachable nodes.
1: class Mapper
2: 	method Map(nid n; node N)
3: 		p   N:PageRank=jN:AdjacencyListj
4: 		Emit(nid n;N) . Pass along graph structure
5: 		 for all nodeid m 2 N:AdjacencyList do
6: 			    Emit(nid m; p) . Pass PageRank mass to neighbors
1: class Reducer
2: 	method Reduce(nid m; [p1; p2; : : :])
3: 	M ;
4: 	for all p 2 counts [p1; p2; : : :] do
5: 		if IsNode(p) then
6: 			      M   p . Recover graph structure
7: 		else
8: 			      s =   s + p . Sum incoming PageRank contributions
9: 			      M:PageRank   s
10: 			     Emit(nid m; node M)
	Phase 1 MapReduce
 
Phase2 Mapper source code
 
Phase2 Reducer source code
 
Indexer: 
Input to the mapper consists of document ids (keys) paired with the actual content (values). Individual documents are processed in parallel by the mappers. First, each document is analyzed and broken down into its component terms. The processing pipeline differs depending on the application and type of document, but for web pages typically involves stripping out HTML tags and other elements such as JavaScript code, tokenizing, case folding, removing stop words (common words such as `the', `a', `of', etc.), and stemming (removing affixes from words so that `dogs' becomes `dog'). Once the document has been analyzed, term frequencies are computed by iterating over all the terms and keeping track of counts. Lines 4 and 5 in the pseudo-code reflect the process of computing term frequencies, but hides the details of document processing. After this histogram has been built, the mapper then iterates over all terms. For each term, a pair consisting of the document id and the term frequency is created. Each pair, denoted by < n, H {t}>  in the pseudo-code, represents an individual posting. The mapper then emits an intermediate key-value pair with the term as the key and the posting as the value, in line 7 of the mapper pseudo-code. Although as presented here only the term frequency is stored in the posting, this algorithm can be easily augmented to store additional information (e.g., term positions) in the payload.
Algorithm Inverted Index
            1:                   Inverted_Index (int docID[n], string doc[n])
            2:	               M -> new HashMap
3: 		   Count = 0
4:		   For all document with docID m from 0 to n-1
          	5: 			For all term tm and position pos in doc
6: 				With docID m do
7: 				M {tm, previous pos, previous m} -> M {tm, pos, m} +1
8: 				Count (tm, m) ++
9: 				For each tm in M with docID m
10: 					Sort (count (tm, m))
Map function pseudo-code
            1:	          Algorithm Map (int docID[x], string doc[x])
2: 		     M -> new HashMap
3: 		     Count ->0
4: 		      For all document of docID m from 0 to x-1
5: 			 For all term tm and position pos in doc with docID m do
6: 			 M {tm, previous pos, previous m} -> M{ tm, pos, m }+1
7: 			 Count (tm, m) ++
8: 			 emit (M, count (tm, m))
Reduce function pseudo-code
1: Algorithm Reduce (term tm, List of hash maps of each mapper[], count{tm, docID})
2: 	G -> new HashMap //G is common HashMap for all reducers
3: 	for each hash map H from all mappers
4: 		for each term tm in document with docID m and position pos in H
5: 			//n is the total number of documents
6: 			G{ tm , previous pos, previous m} ->
7: 			H{ tm, pos , m }+1
8: 			Sort( count (tm , m ))
9: 	//values in list is sorted based on the count value of each term in a document
10: 			emit(G)
        9.1.2 Inverted Index
   Mapper source code
 
Reducer
 
