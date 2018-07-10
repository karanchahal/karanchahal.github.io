This post talks about how the DHT is implemented for hydra. We are assuming that the reader is already aware of the Kademlia protocol. If not read [this](https://github.com/hydra-hoard/hydra/wiki/Distributed-Hash-Table-Algorithm)  to get upto date.

## Goals
We set out to construct the following components:

1. An _in memory_ data structure that is _fault tolerant_. It supports failure by maintaining a __transaction log__ and periodic __snapshotting__. We shall go into the exact meaning of these two later in this post.

2. A concurrent data structure that supports multiple requests.

3. An in memory cache to keep track of dead nodes. Tjis cache helps reduce unnessesary network requests.

Let's go into these 3 aspects of a DHT in detail now.

## In memory Data Structure with Transaction Logging and Snapshotting

_Kademlia_ requires one to keep a 256 key size key value map. Each value is a list of nodes that denote the nodes closest to the index's address. Hence, we set out to create the very same.

We had to keep track of 3 key things however.

1. The data structure should be saved to disk i.e it should be persistant. We do not want to loose the data that the node has collected in the event of a crash.

2. The data structure should be concurrent. There could be multiple nodes trying to connect to the DHT. They should be served with minimum latency. 

### Transaction Logging

A transaction log is a append only text file that records all the operation that have been done to the data structure. The reason for keeping this file , is when you play back the instructions in the log file, you can arrive at the last seen state of the data structure. 
That's why we need a transaction log. The log is an __append only__ text file. It is a paradigmn used by major databaes like MongoDb , Redis and many more. It is a very standard thing done to maintain persistent data bases.

The exact schema for the log for Hydra has not been decided as of now. It would be of following type probably

```
push index1,node_value
add index1,index2,node_value
```

### Periodic Snapshotting

We have one worrying scenario with this above approah. What happens when the transaction log grows uncontrollably. This could lead to an buffer overflow issue. To solve this, we need to keep clearing the transaction log, by saving the actual DHT to disk.  Two pointers are maintained __front__ and __back__. 

1. start - denotes log entry that is not covered by the most recent snapshot 
2. end - denotes end of transaction log.

During each snapshot operation, the start is brought to the end index position.

The saving to disk for DHT has not been implemented as yet. The page will be updated with technical details once it is.

## Concurrency in the DHT

We approached this by doing two things.

1. Each DHT 256 key index has a go routine listening for it. This decouples the DHT into 256 different parts. Now, the DHT can be modified if two different indexes are being operated upon independently.

2. These go routines are fired when we pass the new node data using go channels. Once, in the go routine, the function checks for __two__ things . If the size of the _list_ which the go routine is responsible for is _full_ or not. If _full_, we check our dead nodes cache to figure out if we have dead nodes.  If _yes_, we simply replace the dead node with the new node value. If there are _no_ dead nodes found, we ping _all_ the nodes in the list to find out if dead nodes exist. If _yes_, as before we _replace_ , if _no_ ,  we discard this new node value.

3. The cache is a simple in memory map data structure that is non persistant.

4. The pings for all the nodes are done concurrently using go routines which are merged back after all the pings are completed. This is one using the ```go``` and ```select``` conurrency paradigms of golang. 

This is why golang is a perfect langauge for building a system like this. It provides very helpful and easy to use concurrency primitives that make server / networking code really efficient.

## Sample Code

The sample code for a concurrenct DHT described above is given below. Keep in mind that this code would probably not be used in the production build. It is just for teaching purposes.

```go
package dht

import (
	"context"
	"fmt"
	"log"
	pb "protobuf/node"
	"strconv"
	"time"

	"google.golang.org/grpc"
)

var (
	serverKey             = "110010"
	timeDuration          = 5 * time.Second
	timeDurationInMinutes = 5.0
	maxNodesInList        = 2
	keySize               = 5
	dht                   = DHT{
		tableInputs: make([]chan nodePacket, keySize),
		table:       make([][]node, keySize),
	}
	cache = Cache{
		table: make([][]cacheObject, keySize),
	}
)

// AddNodeResponse is the reponse sent from AddNodes call
// it returns the index in which the node was added
// ping returns true if pings were called
// input return true if node is added
type AddNodeResponse struct {
	ListIndex int
	Ping      bool
	Input     bool
}

type node struct {
	domain string
	port   int32
	nodeID string
}

type nodePacket struct {
	node         node
	nodeResponse chan AddNodeResponse
}

type nodeChannel struct {
	channel chan node
}

// DHT is the main Hash Table
type DHT struct {
	table       [][]node
	tableInputs []chan nodePacket
}

// Cache Map for checking dead nodes
type cacheObject struct {
	lastTime time.Time
	dead     bool
}

// Cache stores the cache Object
type Cache struct {
	table [][]cacheObject
}

// get node Client sets up connection
func getNodeClient(serverAddress *string) (pb.NodeDiscoveryClient, *grpc.ClientConn) {
	var opts []grpc.DialOption
	opts = append(opts, grpc.WithInsecure())

	conn, err := grpc.Dial(*serverAddress, opts...)

	if err != nil {
		log.Fatalf("fail to dial: %v", err)
	}

	// Testing for Client
	client := pb.NewNodeDiscoveryClient(conn)
	return client, conn
}

// Ping node to check livliness, if no response till 5 seconds, return dead node
func Ping(dNode node, cacheList *[]cacheObject, i int, pings chan int) {
	c := make(chan int, 1)

	go func() {
		hostname := dNode.domain + ":" + strconv.Itoa(int(dNode.port))
		client, conn := getNodeClient(&hostname)
		defer conn.Close()

		ctx, cancel := context.WithTimeout(context.Background(), timeDuration)
		defer cancel()
		livliness, err := client.Ping(ctx, &pb.Node{
			NodeId: dNode.nodeID,
			Domain: dNode.domain,
			Port:   dNode.port,
		})
		ob := cacheObject{lastTime: time.Now(), dead: false}

		if err != nil {
			// log.Fatalf("%v.Ping(_) = _, %v: ", client, err)
			ob.dead = true
			(*cacheList)[i] = ob
		} else if livliness.Alive {
			// dead node false
			(*cacheList)[i] = ob
		} else {
			// dead node true
			ob.dead = true
			(*cacheList)[i] = ob
		}
		c <- 1
	}()
        // This sends in the ping response for merging of all pings
	select {
	case <-c:
		pings <- 1

	case <-time.After(timeDuration):
		ob := cacheObject{lastTime: time.Now(), dead: true}
		ob.dead = false
		(*cacheList)[i] = ob
		pings <- 1
	}
}

// checks response for all nodes and returns after all nodes have responded
func mergeAllPings(final chan int, pings chan int) {
	i := 0
	for {
		i += <-pings
		if i == maxNodesInList {
			final <- 1
			return
		}
	}
}

// checkForDeadNodes checks for dead nodes in cache
func checkForDeadNodes(cacheList *[]cacheObject) (bool, int) {

	for j, dNode := range *cacheList {
		if dNode.dead == true {
			// indicate to all nodes to finsh their go functions
			return true, j
		}
	}
	return false, -1
}

/*
   checkAndUpdateCache checks cache for dead nodes, if
   not found update pings of all nodes. Then check for
   dead nodes, return index if any. Else return -1
*/
func checkAndUpdateCache(list *[]node, cacheList *[]cacheObject) (int, bool) {
	ping := false
	dead, i := checkForDeadNodes(cacheList)

	if dead {
		return i, ping
	}
	ping = true

	final := make(chan int)
	pings := make(chan int)

	go mergeAllPings(final, pings)

	for j := range *cacheList {
                // ping request for all nodes
		go Ping((*list)[j], cacheList, j, pings)
	}

	<-final
	// return index of dead node
	dead, i = checkForDeadNodes(cacheList)
	if dead {
		return i, ping
	}

	return -1, ping
}

// IndexListener adds nodes into index i of DHT and updates cache
func IndexListener(list *chan nodePacket, i int) {

	for {
		val := <-*list
		response := AddNodeResponse{Ping: false, Input: false, ListIndex: i}

		// check size
		size := len(dht.table[i])
		// adds if size is good
		if size == maxNodesInList {
			j, ping := checkAndUpdateCache(&dht.table[i], &cache.table[i])

			response.Ping = ping

			if j != -1 {
				add(val.node, i, j)
				response.Input = true
			}
		} else if size < maxNodesInList {
			// just push into list
			push(val.node, i)
			response.Input = true

		} else {
			log.Fatal("Size Is Greater Than Max Number of Nodes !!")
		}

		val.nodeResponse <- response
	}

}
// Appends a node value to a list
func push(val node, i int) {
	cacheVal := cacheObject{lastTime: time.Now(), dead: false}
	dht.table[i] = append(dht.table[i], val)
	cache.table[i] = append(cache.table[i], cacheVal)
}

// Adds a node value to a particular index in a list
func add(val node, i int, j int) {
	cacheVal := cacheObject{lastTime: time.Now(), dead: false}
	dht.table[i][j] = val
	cache.table[i][j] = cacheVal
}

// getIndex gets index of list of nodes of DHT to get for given key
func getIndex(nodeID string) int {
	for i := 0; i < keySize; i++ {
		if nodeID[i] != serverKey[i] {
			return i
		}
	}
	return -1
}

// InitDHT Initialises the DHT and setups listeners at each index
func InitDHT(bitSpace int) {

	fmt.Println("Setting up listeners ")
	keySize = bitSpace
	// setting up listeners
	for i := 0; i < keySize; i++ {
		dht.tableInputs[i] = make(chan nodePacket)
		go IndexListener(&dht.tableInputs[i], i)
	}
}

// AddNode adds a new node into DHT
func AddNode(domain string, port int32, nodeID string) chan AddNodeResponse {

	nodeResponse := make(chan AddNodeResponse)
	value := nodePacket{node: node{
		domain: domain,
		port:   port,
		nodeID: nodeID,
	},
		nodeResponse: nodeResponse,
	}
	index := getIndex(value.node.nodeID)
	dht.tableInputs[index] <- value

	return nodeResponse
}

```


## Conclusion

1. Keep data structure persistant by using transaction logs and snapshotting.

2. An implementation on how a concurrent data structure could work.



