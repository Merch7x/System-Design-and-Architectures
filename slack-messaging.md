How Slack sends Millions of Messages in Real Time
Slack is a chat tool that helps teams to communicate and work together easily. You can use it to send messages/files as well as do things like schedule meetings or have a video call.

Messages in Slack are sent inside of channels (think of this as a chat room or group chat you can set up within your company’s slack for a specific team/initiative). Every day, Slack has to send millions of messages across millions of channels in real time.

They need to accomplish this while handling highly variable traffic patterns. Most of Slack’s users are in North America, so they’re mostly online between 9 am and 5 pm with peaks at 11 am and 2 pm.

Sameera Thangudu is a Senior Software Engineer at Slack and she wrote a great blog post going through their architecture.

Slack uses a variety of different services to run their messaging system. Some of the important ones are

Channel Servers - These servers are responsible for holding the messages of channels (group chats within your team slack). Each channel has an ID which is hashed and mapped to a unique channel server. Slack uses consistent hashing to spread the load across the channel servers so that servers can easily be added/removed while minimizing the amount of resharding work needed for balanced partitions.

Gateway Servers - These servers sit in-between slack web/desktop/mobile clients and the Channel Servers. They hold information on which channels a user is subscribed to. Users will connect to Gateway Servers for subscribing to a channel’s messages so these servers are deployed across multiple geographical regions. The user can connect to whichever gateway server is closest to him/herself.

Presence Servers - These servers store user information; they keep track of which users are online (they’re responsible for powering the green presence dots). Slack clients can make queries to the Presence Servers through the Gateway Servers. This way, they can get presence status/changes.

Now, we’ll talk about how these different services interact with the slack mobile/desktop app.

Slack Client Boot Up
When you first open the Slack app, it will first send a request to Slack’s backend to get a user token and websocket connection setup information.

This information tells the Slack app which Gateway Server they should connect to (these servers are deployed on the edge to be close to clients).

For communicating with the Slack clients, Slack uses Envoy, an open source service proxy originally built at Lyft. Envoy is quite popular for powering communication between backend services; it has built in functionality to take care of things like protocol conversion, observability, service discovery, retries, load balancing and more. In a previous article, we talked about how Snapchat uses Envoy for their service mesh.

In addition, Envoy can be used for communication with clients, which is how Slack is using it here. The user sends a Websocket connection request to Envoy which forwards it to a Gateway server.

Once a user connects to the Gateway Server, it will fetch information on all of that user’s channel subscriptions from another service in Slack’s backend. After getting the data, the Gateway Server will subscribe to all the channel servers that hold those channels.

Now, the Slack client is ready to send and receive real time messages.

Sending a Message
Once you send a message from your app to your channel, Slack needs to make sure the message is broadcasted to all the clients who are online and in the channel.

The message is first sent to Slack’s backend, which will use another microservice to figure out which Channel Servers are responsible for holding the state around that channel.

The message gets delivered to the correct Channel Server, which will then send the message to every Gateway Server across the world that is subscribed to that channel.

Each Gateway Server receives that message and sends it to every connected client subscribed to that channel ID.

Scaling Channel Servers
As mentioned, Slack uses Consistent Hashing to map Channel IDs to Channel Servers. Here’s a great video on Consistent Hashing if you’re unfamiliar with it (it’s best explained visually).

A TL;DW; is that consistent hashing lets you easily add/remove channel servers to your cluster while minimizing the amount of resharding (shifting around Channel IDs between channel servers to balance the load across all of them).

Slack uses Consul to manage which Channel Servers are storing which Channels and they have Consistent Hash Ring Managers (CHARMs) to run the consistent hashing algorithms.

For more details, read the full blog post here.

How did you like this summary?
Your feedback really helps me improve curation for future emails.

I didn't like it
Mehh, it was okay
It was great

The Best Way to Handle Large Sets of Time-Stamped Data
Working with large sets of time-stamped data has its challenges.

Fortunately, InfluxDB is a time series database purpose-built to handle the unique workloads of time series data.

Using InfluxDB, developers can ingest billions of data points in real-time with unbounded cardinality, and store, analyze, and act on that data – all in a single database.

No matter what kind of time series data you’re working with – metrics, events, traces, or logs – InfluxDB Cloud provides a performant, elastic, serverless time series platform with the tools and features developers need. Native SQL compatibility makes it easy to get started with InfluxDB and to scale your solutions.

Companies like IBM, Cisco, and Robinhood all rely heavily on InfluxDB to build and manage responsive backend applications, to power predictive intelligence, and to monitor their systems for insights that they would otherwise miss.
