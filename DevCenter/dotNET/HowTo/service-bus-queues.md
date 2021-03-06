<properties linkid="dev-net-how-to-service-bus-queues" urldisplayname="Service Bus Queues" headerexpose="" pagetitle="Service Bus Queues - How To - .NET - Develop" metakeywords="Get Started Service Bus queues, Get started Azure Service Bus queues, Azure messaging, Azure brokered messaging, Azure messaging queue, Service Bus queue, Azure Service Bus queue, Azure messaging .NET, Azure messaging queue .NET, Azure Service Bus queue .NET, Service Bus queue .NET, Azure messaging C#, Azure messaging queue C#, Azure Service Bus queue C#, Service Bus queue C#" footerexpose="" metadescription="Get started with Windows Azure Service Bus queues, including how to create queues, how to send and receive messages, and how to delete queues." umbraconavihide="0" disquscomments="1"></properties>

# How to Use Service Bus Queues

<span>This guide will show you how to use Service Bus queues. The
samples are written in C\# and use the .NET API. The scenarios covered
include **creating queues, sending and receiving messages**, and
**deleting queues**. For more information on queues, see the [Next Steps] section. </span>

## Table of Contents

-   [What are Service Bus Queues][]
-   [Create a Service Namespace][]
-   [Obtain the Default Management Credentials for the Namespace][]
-   [Configure Your Application to Use Service Bus][]
-   [How to: Create a Security Token Provider][]
-   [How to: Create a Queue][]
-   [How to: Send Messages to a Queue][]
-   [How to: Receive Messages from a Queue][]
-   [How to: Handle Application Crashes and Unreadable Messages][]
-   [Next Steps][]

## <a name="what-queues"> </a>What are Service Bus Queues

<span>Service Bus Queues support a **brokered messaging communication**
model. When using queues, components of a distributed application do not
communicate directly with each other, they instead exchange messages via
a queue, which acts as an intermediary. A message producer (sender)
hands off a message to the queue and then continues its processing.
Asynchronously, a message consumer (receiver) pulls the message from the
queue and processes it. The producer does not have to wait for a reply
from the consumer in order to continue to process and send further
messages. Queues offer **First In, First Out (FIFO)** message delivery
to one or more competing consumers. That is, messages are typically
received and processed by the receivers in the order in which they were
added to the queue, and each message is received and processed by only
one message consumer.</span>

![Queue Concepts][]

Service Bus queues are a general-purpose technology that can be used for
a wide variety of scenarios:

-   Communication between web and worker roles in a multi-tier Windows
    Azure application
-   Communication between on-premises apps and Windows Azure hosted apps
    in a hybrid solution
-   Communication between components of a distributed application
    running on-premises in different organizations or departments of an
    organization

Using queues can enable you to scale out your applications better, and
enable more resiliency to your architecture.

## <a name="create-namespace"> </a>Create a Service Namespace

To begin using Service Bus queues in Windows Azure, you must first
create a service namespace. A service namespace provides a scoping
container for addressing Service Bus resources within your application.

To create a service namespace:

1.  Log on to the [Windows Azure Management Portal][].

2.  In the lower left navigation pane of the Management Portal, click
    **Service Bus, Access Control & Caching**.

3.  In the upper left pane of the Management Portal, click the **Service
    Bus** node, and then click the **New** button.   
    ![][0]

4.  In the **Create a new Service Namespace** dialog, enter a
    **Namespace**, and then to make sure that it is unique, click the
    **Check Availability** button.   
    ![][1]

5.  After making sure the namespace name is available, choose the
    country or region in which your namespace should be hosted (make
    sure you use the same country/region in which you are deploying your
    compute resources), and then click the **Create Namespace** button.

The namespace you created will then appear in the Management Portal and
takes a moment to activate. Wait until the status is **Active** before
moving on.

## <a name="obtain-creds"> </a>Obtain the Default Management Credentials for the Namespace

In order to perform management operations, such as creating a queue, on
the new namespace, you need to obtain the management credentials for the
namespace.

1.  In the left navigation pane, click the **Service Bus** node, to
    display the list of available namespaces:   
    ![][0]

2.  Select the namespace you just created from the list shown:   
    ![][2]

3.  The right-hand **Properties** pane will list the properties for the
    new namespace:   
    ![][3]

4.  The **Default Key** is hidden. Click the **View** button to display
    the security credentials:   
    ![][4]

5.  Make a note of the **Default Issuer** and the **Default Key** as you
    will use this information below to perform operations with the
    namespace.

## <a name="configure-app"> </a>Configure Your Application to Use Service Bus

When you create an application that uses Service Bus, you will need to
add a reference to the Service Bus assembly and include the
corresponding namespaces.

### Add a Reference to the Service Bus Assembly

1.  In Visual Studio's **Solution Explorer**, right-click
    **References**, and then click **Add Reference**.

2.  In the **Browse** tab, go to C:\\Program Files\\Microsoft SDKs\\Windows Azure\\.NET SDK\\2012-06\\ref and add a **Microsoft.ServiceBus.dll** reference.

### Import the Service Bus Namespaces

Add the following to the top of any C\# file where you want to use
Service Bus queues:

    using Microsoft.ServiceBus;
    using Microsoft.ServiceBus.Messaging;

You are now ready to write code against the Service Bus.

## <a name="create-provider"> </a>How to Set Up a Service Bus Connection String

The Service Bus uses a connection string to store endpoints and credentials. You can put your connection string in a configuration file, rather than hard-coding it in code:

- When using Windows Azure Cloud Services, it is recommended you store your connection string using the Windows Azure service configuration system (`*.csdef` and `*.cscfg` files).
- When using Windows Azure Web Sites or Windows Azure Virtual Machines, it is recommended you store your connection string using the .NET configuration system (e.g. `web.config` file).

In both cases, you can retrieve your connection string using the `CloudConfigurationManager.GetSetting` method as shown later in this guide.

### Configuring your connection string when using Cloud Services

The service configuration mechanism is unique to Windows Azure Cloud Services
projects and enables you to dynamically change configuration settings
from the Windows Azure Management Portal without redeploying your
application.  For example, add a Setting to your service definition (`*.csdef`) file, as shown below:

	<ServiceDefinition name="WindowsAzure1">
	...
		<WebRole name="MyRole" vmsize="Small">
	    	<ConfigurationSettings>
	      		<Setting name="Microsoft.ServiceBus.ConnectionString" />
    		</ConfigurationSettings>
  		</WebRole>
	...
	</ServiceDefinition>

You then specify values in the service configuration (`*.cscfg`) file:

	<ServiceConfiguration serviceName="WindowsAzure1">
	...
		<Role name="MyRole">
			<ConfigurationSettings>
				<Setting name="Microsoft.ServiceBus.ConnectionString" 
						 value="Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]" />
			</ConfigurationSettings>
		</Role>
	...
	</ServiceConfiguration>

Use the issuer and key values retrieved from the Management Portal as
described in the previous section.

### Configuring your connection string when using Web Sites or Virtual Machines

When using Web Sites or Virtual Machines, it is recommended you use the .NET configuration system (e.g. `web.config`).  You store the connection string using the `<appSettings>` element:

	<configuration>
	    <appSettings>
		    <add key="Microsoft.ServiceBus.ConnectionString"
			     value="Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]" />
		</appSettings>
	</configuration>

Use the issuer and key values retrieved from the Management Portal as
described in the previous section.

## <a name="create-queue"> </a>How to Create a Queue

Management operations for Service Bus queues can be performed via the
**NamespaceManager** class. The **NamespaceManager** class provides methods to create, enumerate, and delete queues. 

In this example, a **NamespaceManager** object is constructed by using the Windows Azure **CloudConfigurationManager** class
with a connection string consisting of the base address of a Service Bus namespace and the appropriate
credentials with permissions to manage it. This connection string is of the form
"Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]"". For example, given the configuration settings in the previous section:

	// Create the queue if it does not exist already
	string connectionString = 
	    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");
	var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);
    if (!namespaceManager.QueueExists("TestQueue"))
    {
        namespaceManager.CreateQueue("TestQueue");
    }

There are overloads of the **CreateQueue** method that allow properties
of the queue to be tuned (for example, to set the default
"time-to-live" value to be applied to messages sent to the queue). These
settings are applied by using the **QueueDescription** class. The
following example shows how to create a queue named "TestQueue" with a
maximum size of 5GB and a default message time-to-live of 1 minute:

	// Configure Queue Settings
    QueueDescription qd = new QueueDescription("TestQueue");
    qd.MaxSizeInMegabytes = 5120;
    qd.DefaultMessageTimeToLive = new TimeSpan(0, 1, 0);

	// Create a new Queue with custom settings
	string connectionString = 
	    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");
	var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);
    if (!namespaceManager.QueueExists("TestQueue"))
    {
        namespaceManager.CreateQueue(QueueDescription);
    }

**Note:** You can use the **QueueExists** method on **NamespaceManager**
objects to check if a queue with a specified name already exists within
a service namespace.

## <a name="send-messages"> </a>How to Send Messages to a Queue

To send a message to a Service Bus queue, your application will create a
**MessageSender** object. Similar to **NamespaceManager** objects, this object
is created from the base URI of the service namespace and the
appropriate credentials (the connection string).

The code below demonstrates how to create a **MessageSender** object
for the "TestQueue" queue created above:

	string connectionString = 
	    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

	namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

    Client = QueueClient.CreateFromConnectionString(connectionString, "TestQueue");
	Client.Send(new BrokeredMessage());

Messages sent to (and received from) Service Bus queues are instances of
the **BrokeredMessage** class. **BrokeredMessage** objects have a set of
standard properties (such as **Label** and **TimeToLive**), a dictionary
that is used to hold custom application specific properties, and a body
of arbitrary application data. An application can set the body of the
message by passing any serializable object into the constructor of the
**BrokeredMessage**, and the appropriate **DataContractSerializer** will
then be used to serialize the object. Alternatively, a
**System.IO.Stream** can be provided.

The following example demonstrates how to send five test messages to the
"TestQueue" **MessageSender** obtained in the code snippet above:

     for (int i=0; i<5; i++)
     {
       // Create message, passing a string message for the body
       BrokeredMessage message = new BrokeredMessage("Test message " + i);

       // Set some additional custom app-specific properties
       message.Properties["TestProperty"] = "TestValue";
       message.Properties["Message number"] = i;   

       // Send message to the queue
       testQueue.Send(message);
     }

Service Bus queues support a maximum message size of 256 KB (the header,
which includes the standard and custom application properties, can have
a maximum size of 64 KB). There is no limit on the number of messages
held in a queue but there is a cap on the total size of the messages
held by a queue. This queue size is defined at creation time, with an
upper limit of 5 GB.

## <a name="receive-messages"> </a>How to Receive Messages from a Queue

The easiest way to receive messages from a queue is to use a
**MessageReceiver** object. **MessageReceiver** objects can work in two
different modes: **ReceiveAndDelete** and **PeekLock**.

When using the **ReceiveAndDelete** mode, the receive is a single-shot
operation - that is, when the Service Bus receives a read request for a
message in a queue, it marks the message as consumed, and returns
it to the application. **ReceiveAndDelete** mode is the simplest model
and works best for scenarios in which an application can tolerate not
processing a message in the event of a failure. To understand this,
consider a scenario in which the consumer issues the receive request and
then crashes before processing it. Because the Service Bus will have marked
the message as being consumed, when the application restarts and
begins consuming messages again, it will have missed the message that
was consumed prior to the crash.

In **PeekLock** mode (which is the default mode), the receive becomes a two-stage operation, which makes it possible to support applications that
cannot tolerate missing messages. When the Service Bus receives a request,
it finds the next message to be consumed, locks it to prevent other
consumers receiving it, and then returns it to the application. After
the application finishes processing the message (or stores it reliably
for future processing), it completes the second stage of the receive
process by calling **Complete** on the received message. When the Service
Bus sees the **Complete** call, it marks the message as being
consumed and removes it from the queue.

The example below demonstrates how messages can be received and
processed using the default **PeekLock** mode. The example creates an infinite loop and processes messages as they arrive into the "TestQueue":

	string connectionString = 
	    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

    queueClient.Receive();
     
    // Continuously process messages sent to the "TestQueue" 
    while (true) 
    {  
       BrokeredMessage message = QueueClient.Receive();

       if (message != null)
       {
          try 
          {
             Console.WriteLine("Body: " + message.GetBody<string>());
             Console.WriteLine("MessageID: " + message.MessageId);
             Console.WriteLine("Custom Property: " + message.Properties["TestProperty"]);

             // Remove message from queue
             message.Complete();
          }
          catch (Exception)
          {
             // Indicate a problem, unlock message in queue
             message.Abandon();
          }
       }
    } 

## <a name="handle-crashes"> </a>How to Handle Application Crashes and Unreadable Messages

The Service Bus provides functionality to help you gracefully recover from
errors in your application or difficulties processing a message. If a
receiver application is unable to process the message for some reason,
then it can call the **Abandon** method on the received message (instead
of the **Complete** method). This will cause the Service Bus to unlock the
message within the queue and make it available to be received again,
either by the same consuming application or by another consuming
application.

There is also a timeout associated with a message locked within the
queue, and if the application fails to process the message before the
lock timeout expires (for example, if the application crashes), then the Service
Bus unlocks the message automatically and makes it available to be
received again.

In the event that the application crashes after processing the message
but before the **Complete** request is issued, then the message will be
redelivered to the application when it restarts. This is often called
**At Least Once Processing**, that is, each message will be processed at
least once but in certain situations the same message may be
redelivered. If the scenario cannot tolerate duplicate processing, then
application developers should add additional logic to their application
to handle duplicate message delivery. This is often achieved using the
**MessageId** property of the message, which will remain constant across
delivery attempts.

## <a name="next-steps"> </a>Next Steps

Now that you've learned the basics of Service Bus queues, follow these
links to learn more.

-   See the MSDN Reference: [Queues, Topics, and Subscriptions.][]
-   Build a working application that sends and receives messages to and
    from a Service Bus queue: [Service Bus Brokered Messaging .NET
    Tutorial].

  [Next Steps]: #next-steps
  [What are Service Bus Queues]: #what-queues
  [Create a Service Namespace]: #create-namespace
  [Obtain the Default Management Credentials for the Namespace]: #obtain-creds
  [Configure Your Application to Use Service Bus]: #configure-app
  [How to: Create a Security Token Provider]: #create-provider
  [How to: Create a Queue]: #create-queue
  [How to: Send Messages to a Queue]: #send-messages
  [How to: Receive Messages from a Queue]: #receive-messages
  [How to: Handle Application Crashes and Unreadable Messages]: #handle-crashes
  [Queue Concepts]: ../../../DevCenter/dotNet/Media/sb-queues-08.png
  [Windows Azure Management Portal]: http://windows.azure.com
  [0]: ../../../DevCenter/dotNet/Media/sb-queues-03.png
  [1]: ../../../DevCenter/dotNet/Media/sb-queues-04.png
  [2]: ../../../DevCenter/dotNet/Media/sb-queues-05.png
  [3]: ../../../DevCenter/dotNet/Media/sb-queues-06.png
  [4]: ../../../DevCenter/dotNet/Media/sb-queues-07.png
  [Queues, Topics, and Subscriptions.]: http://msdn.microsoft.com/en-us/library/windowsazure/hh367516.aspx
  [Service Bus Brokered Messaging .NET Tutorial]: http://msdn.microsoft.com/en-us/library/windowsazure/hh367512.aspx
