<properties linkid="dev-net-how-to-queue-service" urldisplayname="Queue Service" headerexpose="" pagetitle="How to Use the Queue Storage Service from .NET" metakeywords="Get started Azure queue, Azure asynchronous processing, Azure queue, Azure queue storage, Azure queue .NET, Azure queue storage .NET, Azure queue C#, Azure queue storage C#" footerexpose="" metadescription="Learn how to use the Windows Azure queue storage service to create and delete queues and insert, peek, get, and delete queue messages." umbraconavihide="0" disquscomments="1"></properties>

# How to Use the Queue Storage Service

This guide will show you how to perform common scenarios using the
Windows Azure Queue storage service. The samples are written in C\# code
and use the .NET API. The scenarios covered include **inserting**,
**peeking**, **getting**, and **deleting**queue messages, as well as
**creating and deleting queues**. For more information on queues, refer
to the [Next Steps][] section.

## Table of Contents

-   [What is Queue Storage][]
-   [Concepts][]
-   [Create a Windows Azure Storage Account][]
-   [Create a Windows Azure Project in Visual Studio][]
-   [Configure your Application to Access Storage][]
-   [Setup a Windows Azure Storage Connection String][]
-   [How To: Create a Queue][]
-   [How To: Insert a Message into a Queue][]
-   [How To: Peek at the Next Message][]
-   [How To: Change the Contents of a Queued Message][]
-   [How To: Dequeue the Next Message][]
-   [How To: Additional Options for Dequeuing Messages][]
-   [How To: Get the Queue Length][]
-   [How To: Delete a Queue][]
-   [Next Steps][]

<div chunk="../../Shared/Chunks/howto-queue-storage.md" />

## <a name="create-account"> </a>Create a Windows Azure Storage Account
<div chunk="../../Shared/Chunks/create-storage-account.md" />

## <a name="create-project"> </a>Create a Windows Azure Project in Visual Studio

In this guide, you will use storage features within a Windows Azure
cloud project in Visual Studio. To learn how to create Windows Azure
cloud projects:

1.  [Download and install the Windows Azure SDK for .NET][] if you have
    not already done so.

2.  Read [Creating a Windows Azure Project in Visual Studio][] on MSDN,
    and follow the steps for creating a Windows Azure project with at
    least one web or worker role.

## <a name="configure-access"> </a>Configure Your Application to Access Storage

The web and worker roles in your cloud project are already configured to
use Windows Azure storage. Add the following to the top of any C\# file
where you want to use Windows Azure Storage:

    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.StorageClient;

## <a name="setup-connection-string"> </a>Setup a Windows Azure Storage Connection String

The Windows Azure .NET storage API supports using a storage connection
string to configure endpoints and credentials for accessing storage
services. You can put your storage connection string in a configuration
file, rather than hard-coding it in code:

- When using Windows Azure Cloud Services, it is recommended you store your connection string using the Windows Azure service configuration system (`*.csdef` and `*.cscfg` files).
- When using Windows Azure Web Sites or Windows Azure Virtual Machines, it is recommended you store your connection string using the .NET configuration system (e.g. `web.config` file).

In both cases, you can retrieve your connection string using the `CloudConfigurationManager.GetSetting` method as shown later in this guide.

### Configuring your connection string when using Cloud Services

The service configuration mechanism is unique to Windows Azure Cloud Services
projects and enables you to dynamically change configuration settings
from the Windows Azure Management Portal without redeploying your
application.

To configure your connection string in the Windows Azure service
configuration:

1.  Within the Solution Explorer of Visual Studio, in the **Roles**
    folder of your Windows Azure Deployment Project, right-click your
    web role or worker role and click **Properties**.  
    ![Blob5][]

2.  Click the **Settings** tab and press the **Add Setting** button.  
    ![Blob6][]

    A new **Setting1** entry will then show up in the settings grid.

3.  In the **Type** drop-down of the new **Setting1** entry, choose
    **Connection String**.  
    ![Blob7][]

4.  Click the **...** button at the right end of the **Setting1** entry.
    The **Storage Account Connection String** dialog will open.

5.  Choose whether you want to target the storage emulator (the Windows
    Azure storage simulated on your local machine) or an actual storage
    account in the cloud. The code in this guide works with either
    option. Enter the **Primary Access Key** value copied from the
    earlier step in this tutorial if you wish to store blob data in the
    storage account we created earlier on Windows Azure.   
    ![Blob8][]

6.  Change the entry **Name** from **Setting1** to a "friendlier" name
    like **StorageConnectionString**. You will reference this
    connectionstring later in the code in this guide.  
    ![Blob9][]
	
### Configuring your connection string when using Web Sites or Virtual Machines

When using Web Sites or Virtual Machines, it is recommended you use the .NET configuration system (e.g. `web.config`).  You store the connection string using the `<appSettings>` element:

	<configuration>
	    <appSettings>
		    <add key="StorageConnectionString"
			     value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey]" />
		</appSettings>
	</configuration>

Read [Configuring Connection Strings][] for more information on storage connection strings.
	
You are now ready to perform the How To's in this guide.

## <a name="create-queue"> </a>How To: Create a Queue

A **CloudQueueClient** object lets you get reference objects for queues.
The following code creates a **CloudQueueClient**object. All code in
this guide uses a storage connection string stored in the Windows Azure
application's service configuration. There are also other ways to create
a **CloudStorageAccount** object. See [CloudStorageAccount][]
documentation for details.

    // Retrieve storage account from connection-string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

Use the **queueClient** object to get a reference to the queue you want
to use. You can create the queue if it doesn't exist.

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Create the queue if it doesn't already exist
    queue.CreateIfNotExist();

## <a name="insert-message"> </a>How To: Insert a Message into a Queue

To insert a message into an existing queue, first create a new
**CloudQueueMessage**. Next, call the **AddMessage** method. A
**CloudQueueMessage** can be created from either a string (in UTF-8
format) or a **byte** array. Here is code which creates a queue (if it
doesn't exist) and inserts the message 'Hello, World'.

    // Retrieve storage account from connection-string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Create the queue if it doesn't already exist
    queue.CreateIfNotExist();

    // Create a message and add it to the queue
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    queue.AddMessage(message);

## <a name="peek-message"> </a>How To: Peek at the Next Message

You can peek at the message in the front of a queue without removing it
from the queue by calling the **PeekMessage** method.

    // Retrieve storage account from connection-string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Peek at the next message
    CloudQueueMessage peekedMessage = queue.PeekMessage();

## <a name="change-contents"> </a>How To: Change the Contents of a Queued Message

You can change the contents of a message in-place in the queue. If the
message represents a work task, you could use this feature to update the
status of the work task. The following code updates the queue message
with new contents, and sets the visibility timeout to extend another 60
seconds. This saves the state of work associated with the message, and
gives the client another minute to continue working on the message. You
could use this technique to track multi-step workflows on queue
messages, without having to start over from the beginning if a
processing step fails due to hardware or software failure. Typically,
you would keep a retry count as well, and if the message is retried more
than *n* times, you would delete it. This protects against a message
that triggers an application error each time it is processed.

    // Retrieve storage account from connection-string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    CloudQueueMessage message = queue.GetMessage();
    message.SetMessageContent("Updated contents.") ;
    queue.UpdateMessage(message, 
        TimeSpan.FromSeconds(0.0),  // visible immediately
        MessageUpdateFields.Content | MessageUpdateFields.Visibility);

## <a name="get-message"> </a>How To: Dequeue the Next Message

Your code dequeues a message from a queue in two steps. When you call
**GetMessage**, you get the next message in a queue. A message returned
from **GetMessage** becomes invisible to any other code reading messages
from this queue. By default, this message stays invisible for 30
seconds. To finish removing the message from the queue, you must also
call **DeleteMessage**. This two-step process of removing a message
assures that if your code fails to process a message due to hardware or
software failure, another instance of your code can get the same message
and try again. Your code calls **DeleteMessage** right after the message
has been processed.

    // Retrieve storage account from connection-string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Get the next message
    CloudQueueMessage retrievedMessage = queue.GetMessage();

    //Process the message in less than 30 seconds, and then delete the message
    queue.DeleteMessage(retrievedMessage);

## <a name="advanced-get"> </a>Additional Options for Dequeuing Messages

There are two ways you can customize message retrieval from a queue.
First, you can get a batch of messages (up to 32). Second, you can set a
longer or shorter invisibility timeout, allowing your code more or less
time to fully process each message. The following code example uses the
**GetMessages** method to get 20 messages in one call. Then it processes
each message using a **foreach** loop. It also sets the invisibility
timeout to five minutes for each message. Note that the 5 minutes starts
for all messages at the same time, so after 5 minutes have passed since
the call to **GetMessages**, any messages which have not been deleted
will become visible again.

    // Retrieve storage account from connection-string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    foreach (CloudQueueMessage message in queue.GetMessages(20, TimeSpan.FromMinutes(5)))
    {
        // Do processing for all messages in less than 5 minutes, deleting each message after processing
        queue.DeleteMessage(message);
    }

## <a name="get-queue-length"> </a>How To: Get the Queue Length

You can get an estimate of the number of messages in a queue. The
**RetrieveApproximateMessageCount** method asks the Queue service to
count how many messages are in a queue. The count is only an
approximation because messages can be added or removed after the Queue
service responds to your request. The **ApproximateMethodCount**
property returns the last value retrieved by the
**RetrieveApproximateMessageCount**, without calling the Queue service.

    // Retrieve storage account from connection-string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Retrieve the approximate message count
    int freshMessageCount = queue.RetrieveApproximateMessageCount();

    // Retrieve the cached approximate message count
    int? cachedMessageCount = queue.ApproximateMessageCount;

## <a name="delete-queue"> </a>How To: Delete a Queue

To delete a queue and all the messages contained in it, call the
**Delete** method on the queue object.

    // Retrieve storage account from connection-string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Delete the queue
    queue.Delete();

## <a name="next-steps"> </a>Next Steps

Now that you've learned the basics of queue storage, follow these links
to learn how to do more complex storage tasks.

-   See the MSDN Reference: [Storing and Accessing Data in Windows
    Azure][]
-   Visit the [Windows Azure Storage Team Blog][]

  [Next Steps]: #next-steps
  [What is Queue Storage]: #what-is
  [Concepts]: #concepts
  [Create a Windows Azure Storage Account]: #create-account
  [Create a Windows Azure Project in Visual Studio]: #create-project
  [Configure your Application to Access Storage]: #configure-access
  [Setup a Windows Azure Storage Connection String]: #setup-connection-string
  [How To: Create a Queue]: #create-queue
  [How To: Insert a Message into a Queue]: #insert-message
  [How To: Peek at the Next Message]: #peek-message
  [How To: Change the Contents of a Queued Message]: #change-contents
  [How To: Dequeue the Next Message]: #get-message
  [How To: Additional Options for Dequeuing Messages]: #advanced-get
  [How To: Get the Queue Length]: #get-queue-length
  [How To: Delete a Queue]: #delete-queue
  [Download and install the Windows Azure SDK for .NET]: /en-us/develop/net/
  [Creating a Windows Azure Project in Visual Studio]: http://msdn.microsoft.com/en-us/library/windowsazure/ee405487.aspx
  [Blob5]: ../../../DevCenter/dotNet/Media/blob5.png
  [Blob6]: ../../../DevCenter/dotNet/Media/blob6.png
  [Blob7]: ../../../DevCenter/dotNet/Media/blob7.png
  [Blob8]: ../../../DevCenter/dotNet/Media/blob8.png
  [Blob9]: ../../../DevCenter/dotNet/Media/blob9.png
  [CloudStorageAccount]: http://msdn.microsoft.com/en-us/library/microsoft.windowsazure.cloudstorageaccount_methods.aspx
  [Storing and Accessing Data in Windows Azure]: http://msdn.microsoft.com/en-us/library/windowsazure/gg433040.aspx
  [Windows Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
  [Configuring Connection Strings]: http://msdn.microsoft.com/en-us/library/windowsazure/ee758697.aspx