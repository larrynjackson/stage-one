# stage-one

The three projects, stage-one, stage-two and stage-final, repesent a real-time message processing simulation system capable of processing
500 incoming messages per second. Each stage runs as a stand-alone go application on the same machine. To run the simulation as currently
configured deploy the three stages and start each from the main.go directory with the 'go run .' command. Deploy the unix-sender project
which currently sends 5000 messages to stage-one with a 2 millisecond delay between each sent message and run it.

The stages communicate via unix sockets. 

unix-sender -> stage-one -> stage-two -> stage-final

1. unix-sender message format:       
                                              
type InputMessage struct {                      
	Id          int64  `json:"id"`                    
	ShortCode   string `json:"shortcode"`             
	Destination string `json:"destination"`           
	Tag         string `json:"tag"`                   
}                                                 
The sender injects a 2 millisecond delay between each message to control the message transmission rate.
   
2. stage-one injects a 25 millisecond delay representing some amount of work being done on each message
3. stage-two and stage-final both inject a 100 millisecond delay representing some amount of work being done on each message.

   
Running at the current default settings on a dell intel COREi7 laptop each message was captured, processed and passed along
with no data loss. However, depending on the speed of the test machine, the message rate may need adjustment to avoid data
loss. The messages enter stage-one unix socket buffer and are immediately pushed into a link-list queue. If the input data rate exceeds
the default unix socket buffer size during the time each message is received and pushed into the queue, data will be lost. Each message
is taken from the queue, processed (injected delay) and passed along to the next stage. The link-list queue acts as a flexable
buffer.

The simulated data feed is a constant message rate of 500 per second intended to stress the system during testing. In a real-time
system the input data rate is expected to vary with bursts up to 500 messages per second. The internal queue allows each stage to
expand and contract as the message rate varies. The stage-two code is identical to stage-one with only the simulated delay being
different. Therefore, as many intermediate stages as needed can be inserted into the system as the message processing time increases.

Once the system is up and running, any down-stream stage after stage-one can be stopped and restarted with no loss of data so
long as the previous stage queue can grow without exceeding available system memory. This characteristic does seem to have its 
advantages. Each individual stage's processing capability can be managed as independent code base. The code base can be modified
and deployed with no negative impact to the working system.

stage-one
const (
	protocol           = "unix"
	sockAddrIn  string = "/temp/stage-one-socket"
	sockAddrOut string = "/temp/stage-two-socket"
)


stage-two
const (
	protocol           = "unix"
	sockAddrIn  string = "/temp/stage-two-socket"
	sockAddrOut string = "/temp/stage-final-socket"
)

Notice the pattern here naming in and out sockets. The socket path must be a valid existing location on your machine.

No great contibution to computer science here but an interesting example project helping to learn go.
