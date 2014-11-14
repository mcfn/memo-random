# Hight Performance Browser Networking

## Chapter 2. Building blocks of TCP

* IP  
internet protocal, is what provides the host-to-host routing and addressing.
* TCP  
Transmission Control Protocol, is what provides the abstraction of a reliable networking running over an unreliable channel.

When you work with a TCP stream, you are guaranteed that all bytes send will be identical with bytes received and that they will arrive in the sam order to the client. As such, TCP is optimized for accurate delivery, rather than a timely one.

UDP: User Datagram Protocol or UDP


## Three-Way Handshake

* SYN  
Client picks a random sequence number x and sends a SYN packet, which may also include additional TCP flags and options.
* SYN ACK
Server increments x by one, picks own random sequence number y, appends its own set of flags and options, and dispatches the response.
* ACK  
Client increments both x and y by one and comoletes the handshake by dispatching the last ACK packet in the handshake


(TODO)TFO: TCP Fast open


## Congestion Avoidance and Control
拥塞避免和控制

包含有三个方法
* flow control
* congestion control
* congestion avoidance

### Flow Control
Flow control is a mechanism to prevent the sender from overwhelming the receiver with data it may not be able to process.  

To address this, each side of the TCP connection advertises its own recieve window (rwnd), which communicates the size of the available buffer space to hold the imcomming data


when the connection is first established, both sides initiate their rwnd values by using there system default settings.

If, for any reason, one of the sides is not able to keep up, then it can advertise a small window to the sender.

This workflow continues throughout the lifetime of every TCP connection: each ACK packet carries the latest rwnd value for each side, allowing both sides to dynamically adjust the data flow rate to the capacity and processing speed of the sender and receiver.


### Slow-Start


Congestion window side - cwnd
Sender-side limit on the amount of data the sender can have in flight before receiving an acknowledment(ACK) from client

cwnd is not advertised or exchanged between the sender and receiver, it will be a private value in one side.

set to 10 segments in RFC6928 in 2013

The maximum amount of data in flight for a new TCP connection is the minimum of the rwnd and cwnd values;

Then for every recieved ACK, the slow-start algorithm indicates that the server can increment its cwnd window side by one segment - for every ACKed packet, two new pakcets can be sent.
This phase of the TCP connection is commonly know as the **exponential growth** algorithm  


Because of the Slow-start algorithm, no matter the available bandwidth, every TCP connection must go through the slow-start phase - we cannot use the full capacity of the link immediately.

如果可以重用tcp连接，则会减去three-way shake和slow-start的时间。

### Congestion Avoidance
