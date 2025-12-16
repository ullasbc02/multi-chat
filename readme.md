
##  HIGH-LEVEL FLOW OVERVIEW:

###  1. Server is started

```java
ServerSocket serverSocket = new ServerSocket(1234);
Server server = new Server(serverSocket);
server.startServer();  // starts accepting clients
```

###  2. Client connects to server

```java
Socket socket = new Socket("localhost", 1234);
Client client = new Client(socket, username);
client.listenForMessage();  // thread: listen for broadcasts
client.sendMessage();       // main thread: send input to server
```

###  3. Server accepts client and creates ClientHandler

```java
Socket socket = serverSocket.accept();
ClientHandler clientHandler = new ClientHandler(socket);
new Thread(clientHandler).start();
```

###  4. ClientHandler:

* Reads username from client
* Adds itself to `clientHandlers` list
* Broadcasts: **"{username} has entered the chat!"**
* Enters loop: reads from client → broadcasts to others

---

##  DRY RUN: Step-by-Step Example

Let’s say:

* **User1 joins**
* **User1 sends "Hi"**
* **User2 joins**
* **User2 sends "Hello"**
* **User1 leaves**

---

###  Step 1: Start server

```java
startServer() → serverSocket.accept()
```

* Waiting for client...

---

###  Step 2: User1 starts client

```java
Client("User1") → connects to server
→ sends "User1" (username)
→ starts `sendMessage()` loop
→ starts `listenForMessage()` thread
```

####  Server accepts User1:

```java
accept()
→ ClientHandler(socket)
→ reads username = "User1"
→ adds self to clientHandlers
→ broadcasts "Server: User1 has entered the chat"
```

 Only User1 is in the list, so no one receives this message except console log.

---

###  Step 3: User1 sends "Hi"

Client:

```java
bufferedWriter.write("User1: Hi")
```

`ClientHandler.run()` reads:

```java
message = "User1: Hi"
→ broadcastMessage("User1: Hi")
→ loops over clientHandlers
→ skips itself
→ (no other users yet → nothing happens visibly)
```

---

###  Step 4: User2 joins

```java
Client("User2") → connects to server
→ sends "User2"
→ starts listening and sending threads
```

Server:

```java
→ accept()
→ new ClientHandler(socket)
→ reads username = "User2"
→ adds to clientHandlers
→ broadcasts "Server: User2 has entered the chat"
```

This time `clientHandlers` has:

* User1's handler
* User2's handler

So:

* "Server: User2 has entered the chat" is sent to **User1**
* User2 won't receive this because of the `if (!equals(clientUsername))` check

---

###  Step 5: User2 sends "Hello"

```java
User2 sends "User2: Hello"
→ ClientHandler reads it
→ broadcast to others
→ User1 receives "User2: Hello"
```

---

###  Step 6: User1 disconnects (e.g., Ctrl+C or window closes)

This triggers `IOException` in:

```java
while (socket.isConnected()) → throws
→ closeEverything()
→ removeClientHandler()
→ broadcast "Server: User1 has left the chat"
```

`broadcastMessage()`:

* Sends "Server: User1 has left the chat" to **User2**

---

##  Summary of Key Function Calls

| Event                   | Function Calls                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------ |
| Client connects         | `Server.accept()` → `new ClientHandler()` → `start thread`                           |
| Client sends message    | `ClientHandler.run()` → `bufferedReader.readLine()` → `broadcastMessage()`           |
| Client receives message | `Client.listenForMessage()` thread → `bufferedReader.readLine()`                     |
| Client disconnects      | `IOException` → `closeEverything()` → `removeClientHandler()` → `broadcastMessage()` |

---

##  Important Notes

* Each client has **its own thread** via `ClientHandler`
* `clientHandlers` is shared (static) to enable broadcasting
* Messages never bounce back to sender (unless you remove the `if (!equals)` condition)


