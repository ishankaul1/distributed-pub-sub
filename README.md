# Distributed Pub-Sub

In this project, we built a Topic-based Publisher-Subscriber system using the Zookeeper and ZMQ libraries in Python with the following properties:

  1. Publisher-Subscriber anonymity. This is a basic property of the Publisher-Subscriber pattern, and is achieved by using the 'Publisher' and 'Subscriber' class middlewares, which would run on different hosts and allow client application connections to join our system. Publisher and Subscriber applications and middlewares can join or leave the system at any time without breaking functionality of the whole system. Client applications would connect directly to the middlewares to join the system.
	
  2. "Topic" subscriptions. Each Publisher or Subscriber can register to one or more 'Topic's. Subscribers will only receive data from topics they are registered to, while Publishers will only send messages to the topics they are registered to. Both Publishers and Subscribers must register to at least one topic before they join the system.
  
  3. Globally configurable dissemination. Starting the Broker middleware with option 1 will route all messages published in the system through the broker before they get to subscribers, whereas option 2 only requires registrations with the Broker before publishers send messages directly to each subscriber on the topic. In both cases, regsitration messages from middlewares are sent to the Broker(s).

  4. Broker fault tolerance. If multiple broker instances are spun up, they will have a 'leader election', where one does all the work, while the others wait for their turn. If the leader broker is killed off, one of the backup brokers will takes its place.

  5. "History Length" and "Ownership Strength" quality of services. Any publishers connecting to a topic will be ranked by 'ownership strength' (currently just the order in which they tried to join the system). Each publisher and subscriber must register with a 'history length' - aka the length of each message they will send/receive. If a publisher registered with history length n, it will always send the last n messages posted to it. Subscribers will only receive messages from the publisher with highest ownership strength that registerd with a history length greater than its own. For example, if Pub1 is publishing to topic1 with history length 3 and ownership strength 1, Pub2 is publishing with history length 5 and ownership strength 2, and Sub1 is listening with history length 4, Sub1 will only receive messages from Pub2. However, if Pub1 had registered with a history length >4, then Sub1 would only receive messages from Pub1.

Opportunities for enhancements and optimizations:
  1. We currently send messages between entities in our system as pure strings for simplicity, which we parse by using very specific string.split() command. Ideally, data should be stored in Objects or json-format to enforce consistency, and then serialized for peformance.
  2. We were only able to test this setup on Mininet, due to not having an actual network of machines available. At some point, we would like to see how this system peforms in a real network.
  3. We would like to implement a load-balancing system for the brokers. We currently have fault tolerance, but an additional layer of backup Brokers with multiple actives at the same time would greatly increase the scalability of this system. 
  4. It is part of our intended design that brokers should not be started with different dissemination options in the same network. However, since we configure dissemination option at the individual runtime of the broker instances, this rule is not actually enforced. A global configuration of brokers for both dissemination option and number of copies would make this system far more robust. In addition to this and load balancing, we would like to create an 'auto-scaling' feature, where brokers automatically scale up or down depending on network traffic.
  5. The different options for broker dissemination should be separated by class inheritance, instead of if-statements sprawled throughout our code. We implemented it like this for lack of time during our Distributed Systems semseter, but this is a huge detriment to readability, is plain bad design, and takes away from the ability to continue incremental work on this project. This is likely the first feature we will implement. We plan on refactoring the plain Broker, Publisher, and Subscriber class to be abstract, then having one subclass of each for each dissemination option. Publisher, Subscriber, and Broker will each have a factory class as well, and should be separated into different packages to enhance readability of the directory structure.
  6. Some of the sockets on subscriber side will remain open even if not being used. We should create a system for closing these out.
  7. Publishers currently only send their own history. It would intuitively make more sense if they published the last n messages of the entire topic. Since Zookeeper is technically optimized for coordination and not shared memory, having a centralized node for storing the history of each topic would definitely slow things down, and require locking since all pubs would attempt to write to it whenever they publish a message. We would also have to set a global limit on how much history can be stored at a time.

We are always open to feedback and ideas on how to improve this project. Please feel free to email ishankaul2000@gmail.com or ishan.kaul@Vanderbilt.edu for any questions or comments. Thanks for taking the time to read!
