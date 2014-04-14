---
layout: post
title: Tracking and Understanding TCP States
comments: true
---

#### Irrelevant Background
During my Computer Networks course, I used to hate the person who illustrated the TCP State Transition Diagram.


<img src="/public/images/tcpdiag.gif" height="400" width="400"/>

It was just too complicated. I skimmed over the section just enough to recognize keywords like SYN,ACK,CLOSED and the likes.

I moved on and joined Cisco. One day, a senior colleague of mine told me that he was planning to put up some material that would help others in understanding the TCP State Transistions and needed an extra hand. He had already written some sample code. His method was pretty cool. He used iptables to freeze the states at different stages during a tcp connection. But we never got around to finish it. So after three years of consistent procrastination, I thought I could give it another try.Bottom line, my colleague Ratheesh conceived this article. I just put it up :)

I realize that the TCP state diagram is a really basic thing to write about, and probably anyone reading this would have a better idea about it than the writer(me). But I believe that this is a different and hands-on approach, and might *just* help that one person studying computer networks who is getting frustrated at the picture above, in understanding it better. In addition, it might provide some knowledge about a few networking tools and commands in Linux.

####Before We Start
I did the entire exercise on my Linux vm ( elementaryOS ). Any Linux machine should do fine.  We need to run ss,watch,gcc,ifconfig and iptables commands. The iptables and ifconfig commands will require root permission. It might help if you know basic Linux socket programming. Even though I have added some comments in the code, if you feel lost, there is always [Beej's Guide to Network Programming](http://beej.us/guide/bgnet/output/html/singlepage/bgnet.html).Its a really good tutorial. Also, you can find the complete code of this tutorial at [Understanding TCP](https://github.com/woodstok/woodstok-code/tree/master/understanding_tcp")

Note: *shameless tmux promotion* : Since we will be running code and scripts on multiple windows, using tmux or screen would be awesome. I use tmux since it has better support for split windows. 

### Setting up Interfaces
We need a server and a client , and we need them to talk to each other. Lets assign IP and ports to both of them. I just chose sensible random values here.

- Server - 192.168.100.100 Port - 32000
- Client -  192.168.100.51   Port - 32089

Of course, you could set up two machines (physical/virtual) with the above mentioned IPs, have your server code in one, and the client code in the other. Or, you could just create two alias interfaces on a single system. Like so:
{% highlight console %}
sudo ifconfig eth0:0 192.168.100.100 up
sudo ifconfig eth0:1 192.168.100.51  up
{% endhighlight %}

###Server Code
Lets look at [server.c](https://github.com/woodstok/woodstok-code/blob/master/understanding_tcp/server.c). 
You don't have to pay detailed attention to the code. In short the server code does the following

* Create a tcp Linux socket
* Bind it to 192.168.100.100:32000
* Wait for any clients to connect
* If any client connects, accept and set up a new connection.
* Wait for user input to exit
* Close the socket and connection 


{% highlight cpp %}
   int  listen_fd, conn_fd;  
   struct sockaddr_in serv_addr ,cli_addr;
   int cli_len , n = 0 ;
   char mesg[256]; 

   // create a TCP socket
   listen_fd=socket(AF_INET,SOCK_STREAM,0);

   bzero(&serv_addr,sizeof(serv_addr));
   serv_addr.sin_family = AF_INET;
   // convert dotted ipaddress to unsigned int
   serv_addr.sin_addr.s_addr=inet_addr(SERVER_IP_ADDRESS);   
   serv_addr.sin_port=htons(SERVER_PORT);


   //bind the socket to the server:port
   bind(listen_fd,(struct sockaddr *)&serv_addr,sizeof(serv_addr))

   //make the socket ready of incoming connections
   listen(listen_fd,10)
 
   //we get a new socket_fd for use with send and recv
   //while listen_fd can be used to accept() more new connections(max 10)
   conn_fd = accept(listen_fd,(struct sockaddr *)&cli_addr, &cli_len);


   printf ( "\n************ Press any key to exit  \n ");
   getchar();
   close(listen_fd);

{% endhighlight %}   

Compile and run the code

{% highlight console %}
gcc -o server server.c
./server
{% endhighlight %}

Now we have a simple server waiting for clients to connect to it.

## Monitoring the tcp states

`ss` is a utility to monitor sockets. Here is a small bash script  `server-state.sh` that will display the states for our client and server
{% highlight console %}
ss -a | awk '{ print "ip:port = " $4  "  State = " $1 }' | grep "192.168.100.100"
ss -a | awk '{ print "ip:port = " $4  "  State = " $1 }' | grep "192.168.100.51"
{% endhighlight %}
We can repeat this command every second by running 
{% highlight console %}
watch -n 1 ./server-state.sh
{% endhighlight %}
Lets keep this running in a separate shell throughout the tutorial.Since we have only the server running, you should see 
{% highlight console %}
ip:port = 192.168.100.100:32000  State = LISTEN
{%endhighlight %}

## Client and Server
[client.c]( https://github.com/woodstok/woodstok-code/blob/master/understanding_tcp/client.c ) is not much different from `server.c` . It connects to the server that we have started previously. Lets compile and run `client.c` in a separate shell.

{% highlight console %}
gcc -o client.c client
./client
{% endhighlight %}

Right now, the TCP states should be (from your server-state.sh shell)

> server - ESTAB </br>
> client  - ESTAB

Which means that both the server and client code have established their tcp connection and are ready to talk to each other. If you look at the diagram, ESTABLISHED is the central state . At this point, a proper working program would have code to `send()` and `recv()` functions to transfer messages using TCP. 



## Looking at the States step by step
<img src="/public/images/tcpdiag.gif" height="400" width="400"/>
As soon as we ran the `client` we saw both the server and client going to ESTABLISHED state. Both the server and client went through the intermediate states in the diagram pretty quickly. 

*Note: For each transition arrow in the diagram, the part before the `/` was what triggered that transition, and the part after the `/` is what that is sent out during that transition.
	eg: For the arrow from ESTABLISHED to CLOSE_WAIT, the label is FIN/ACK which means, if the socket which is in ESTABLISHED state receives a FIN, it moves to CLOSE_WAIT and sends out an ACK as part of this transition*

From the initial CLOSED state, here is what could have happened. 
		
1. With the listen() function call, the server moved to LISTEN state.
2. connect() function call in client.c moved the client from CLOSED to SYN_SENT state. Client sends a SYN as part of this transition.
3. server gets this SYN, changes from LISTEN to SYN_RCVD and sends out a SYN+ACK.</br>
*Note: For these diagrams, server elements are colored blue, and client side is red. Some elements are also numbered corresponding to their step number in this sequence list* 
<img src="/public/images/step1.png" height="250" width="480"/>
4. client gets this SYN+ACK, moves from SYN_SENT to ESTABLISHED and sends out an ACK. 
5. server gets this ACK. Moves from SYN_RCVD to ESTABLISHED.

<img src="/public/images/step2.png" height="250" width="480"/>
   

## SYN_SENT state

We already saw the server at LISTEN state before the client was started.  Lets try to catch the client at the SYN\_SENT state. In order to do that, we are going to block the initial SYN packet from the client. If we block the SYN, we break the chain and the client remains at SYN\_SENT state. 
<img src="/public/images/synblocked.png" height="250" width="480"/>
To block packets with specific TCP flags, we will use a command called `iptables`. `iptables` helps a user (with root permission) to configure the Linux Kernel firewall.

*Since we might be doing this a lot, login to a separate shell as root instead of prepending `sudo` to every iptables command that we are going to execute.*

{% highlight console %}
iptables -A INPUT -s 192.168.100.51  -p tcp --tcp-flags SYN SYN -j DROP
{% endhighlight %}

The command takes in all INPUT( incoming ) packets from source 192.168.100.51 (our client) , with protocol TCP, and TCP flag SYN and DROPS them. Once this rule is set, lets start the server and the client. As you will see, 

    server - LISTEN 
    client - SYN_SENT

Remove the SYN rule that you had added in `iptables`. 
{% highlight console %}
iptables -D INPUT 1
{% endhighlight %}
The command assumes that you did not have any other rules preconfigured in ur iptables. If you did, change the number to match the rule number when you display it using `iptables -L`. You can stop the server and client and wait for the states to get closed. Similarly we can vary the iptables command to block syn+ack from the server, making it to stop at the SYN_RCVD state, and so on. 


## Teardown States
Here is what usually happens during a tcp teardown.  Lets assume that the client closed first.
{% highlight ruby %}
Trigger         | Prev State        > Next State | Action
-----------------------------------------------------------   										
client close()  | client ESTAB      > FIN_WAIT1  | sends FIN
server gets FIN | server ESTAB      > CLOSE_WAIT | sends ACK
client gets ACK | client FIN_WAIT1  > FIN_WAIT2  |
server close()  | server CLOSE_WAIT > LAST_ACK   | sends FIN
client gets FIN | client FIN_WAIT2  > TIME_WAIT  | sends ACK
server gets ACK | server LAST_ACK   > CLOSED     |
{% endhighlight %}	

We can repeat the previous steps to observe the teardown states also, but there is catch. The iptables rule would work well with FIN packet. But if you are planning to block the ACK packets, make sure you are adding the iptables rule after both the server and client are in the ESTAB state. Both the server and client are made to wait for user keypress before the close() function is called. So first run server and client, confirm that their states are ESTAB, add the iptables rule, and give a keypress on the client.I leave these states as an exercise. I have also added steps to log the packets filtered through iptables in the readme at [Understanding tcp](https://github.com/woodstok/woodstok-code/tree/master/understanding_tcp).

   
This is my first article. I hope i can do more. Please leave your comments and feedback below. You can also reach me at woodstok@outlook.com

