# Flutter_doc_CokBK_Net_Communicate_with_WebSockets
 https://docs.flutter.dev/cookbook/networking/web-sockets
Communicate with WebSockets
===========================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Networking](https://docs.flutter.dev/cookbook/networking)
3.  [Communicate with WebSockets](https://docs.flutter.dev/cookbook/networking/web-sockets)

In addition to normal HTTP requests, you can connect to servers using `WebSockets`. `WebSockets` allow for two-way communication with a server without polling.

In this example, connect to a [test WebSocket server sponsored by Lob.com](https://www.lob.com/blog/websocket-org-is-down-here-is-an-alternative). The server sends back the same message you send to it. This recipe uses the following steps:

1.  Connect to a WebSocket server.
2.  Listen for messages from the server.
3.  Send data to the server.
4.  Close the WebSocket connection.

[](https://docs.flutter.dev/cookbook/networking/web-sockets#1-connect-to-a-websocket-server)1\. Connect to a WebSocket server
-----------------------------------------------------------------------------------------------------------------------------

The [`web_socket_channel`](https://pub.dev/packages/web_socket_channel) package provides the tools you need to connect to a WebSocket server.

The package provides a `WebSocketChannel` that allows you to both listen for messages from the server and push messages to the server.

In Flutter, use the following line to create a `WebSocketChannel` that connects to a server:

content_copy

```
final channel = WebSocketChannel.connect(
  Uri.parse('wss://echo.websocket.events'),
);
```

[](https://docs.flutter.dev/cookbook/networking/web-sockets#2-listen-for-messages-from-the-server)2\. Listen for messages from the server
-----------------------------------------------------------------------------------------------------------------------------------------

Now that you've established a connection, listen to messages from the server.

After sending a message to the test server, it sends the same message back.

In this example, use a [`StreamBuilder`](https://api.flutter.dev/flutter/widgets/StreamBuilder-class.html) widget to listen for new messages, and a [`Text`](https://api.flutter.dev/flutter/widgets/Text-class.html) widget to display them.

content_copy

```
StreamBuilder(
  stream: channel.stream,
  builder: (context, snapshot) {
    return Text(snapshot.hasData ? '${snapshot.data}' : '');
  },
)
```

### [](https://docs.flutter.dev/cookbook/networking/web-sockets#how-this-works)How this works

The `WebSocketChannel` provides a [`Stream`](https://api.flutter.dev/flutter/dart-async/Stream-class.html) of messages from the server.

The `Stream` class is a fundamental part of the `dart:async` package. It provides a way to listen to async events from a data source. Unlike `Future`, which returns a single async response, the `Stream` class can deliver many events over time.

The [`StreamBuilder`](https://api.flutter.dev/flutter/widgets/StreamBuilder-class.html) widget connects to a `Stream` and asks Flutter to rebuild every time it receives an event using the given `builder()` function.

[](https://docs.flutter.dev/cookbook/networking/web-sockets#3-send-data-to-the-server)3\. Send data to the server
-----------------------------------------------------------------------------------------------------------------

To send data to the server, `add()` messages to the `sink` provided by the `WebSocketChannel`.

content_copy

```
channel.sink.add('Hello!');
```

### [](https://docs.flutter.dev/cookbook/networking/web-sockets#how-this-works-1)How this works

The `WebSocketChannel` provides a [`StreamSink`](https://api.flutter.dev/flutter/dart-async/StreamSink-class.html) to push messages to the server.

The `StreamSink` class provides a general way to add sync or async events to a data source.

[](https://docs.flutter.dev/cookbook/networking/web-sockets#4-close-the-websocket-connection)4\. Close the WebSocket connection
-------------------------------------------------------------------------------------------------------------------------------

After you're done using the WebSocket, close the connection:

content_copy

```
channel.sink.close();
```

[](https://docs.flutter.dev/cookbook/networking/web-sockets#complete-example)Complete example
---------------------------------------------------------------------------------------------

content_copy

```
import 'package:flutter/material.dart';
import 'package:web_socket_channel/web_socket_channel.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    const title = 'WebSocket Demo';
    return const MaterialApp(
      title: title,
      home: MyHomePage(
        title: title,
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({
    super.key,
    required this.title,
  });

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final TextEditingController _controller = TextEditingController();
  final _channel = WebSocketChannel.connect(
    Uri.parse('wss://echo.websocket.events'),
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Form(
              child: TextFormField(
                controller: _controller,
                decoration: const InputDecoration(labelText: 'Send a message'),
              ),
            ),
            const SizedBox(height: 24),
            StreamBuilder(
              stream: _channel.stream,
              builder: (context, snapshot) {
                return Text(snapshot.hasData ? '${snapshot.data}' : '');
              },
            )
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _sendMessage,
        tooltip: 'Send message',
        child: const Icon(Icons.send),
      ), // This trailing comma makes auto-formatting nicer for build methods.
    );
  }

  void _sendMessage() {
    if (_controller.text.isNotEmpty) {
      _channel.sink.add(_controller.text);
    }
  }

  @override
  void dispose() {
    _channel.sink.close();
    _controller.dispose();
    super.dispose();
  }
}
```
