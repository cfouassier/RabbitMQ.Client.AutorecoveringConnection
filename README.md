# RabbitMQ.Client.AutorecoveringConnection
Example how to get new replyQueueName after network recovery

There is no sample how to use AutorecoveringConnection in RabbitMq documentation (c#).
Here is mine if it can help.


1. Create a connection with *AutomaticRecoveryEnabled*
```c#
  factory = new ConnectionFactory() { Uri = new Uri("amqp://guest:guest@localhost:5672/") };
  factory.AutomaticRecoveryEnabled = true;
  factory.RequestedHeartbeat = 10;
  connection = factory.CreateConnection();
```
   
2. Convert it to *AutorecoveringConnection* and connect to event *QueueNameChangeAfterRecovery*
```c#     
   using RabbitMQ.Client.Framing.Impl;
   
   ...

    AutorecoveringConnection autorecoveringConnection = connection as AutorecoveringConnection;
    if (autorecoveringConnection != null)
    {
        autorecoveringConnection.QueueNameChangeAfterRecovery += AutorecoveringConnection_QueueNameChangeAfterRecovery;
    }
    ...
    
```
3. The most important thing !
When RabbitMq recover network connection the QueueName is modified! YOU HAVE TO manage it.
If you do not, you consumer will reply on the wrong queue !
```c#   
    ...
    
    private void AutorecoveringConnection_QueueNameChangeAfterRecovery(object sender, QueueNameChangedAfterRecoveryEventArgs e)
    {
        replyQueueName = e.NameAfter;
    }
    
    ...
    
    public void SendMessage(string routingKey,  byte[] messageBytes)
    {
        IBasicProperties props = CreateDefaultProperties();
        channel.BasicPublish("", routingKey, props, messageBytes);

    }

    ...
    
    public IBasicProperties CreateDefaultProperties()
    {

        var props = channel.CreateBasicProperties();
        props.ReplyTo = replyQueueName;
        props.Priority = 10;

        return props;
    }
    
    ...
```
