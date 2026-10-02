# Part 25: Networking
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 25.1 HTTP Client (Java 11+)

```java
import java.net.http.*;
import java.net.URI;
import java.util.*;
import java.util.concurrent.*;

public class HttpClientDemo {
    
    static final HttpClient client = HttpClient.newBuilder()
        .version(HttpClient.Version.HTTP_2)
        .followRedirects(HttpClient.Redirect.NORMAL)
        .connectTimeout(java.time.Duration.ofSeconds(10))
        .build();
    
    // Synchronous GET
    static String get(String url) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .header("Accept", "application/json")
            .GET()
            .build();
        
        HttpResponse<String> response = client.send(request,
            HttpResponse.BodyHandlers.ofString());
        
        System.out.printf("GET %s -> Status: %d%n", url, response.statusCode());
        return response.body();
    }
    
    // Asynchronous GET
    static CompletableFuture<String> getAsync(String url) {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .GET()
            .build();
        
        return client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
            .thenApply(response -> {
                System.out.printf("Async GET %s -> %d%n", url, response.statusCode());
                return response.body();
            });
    }
    
    // POST with JSON
    static String post(String url, String jsonBody) throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(url))
            .header("Content-Type", "application/json")
            .header("Accept", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
            .build();
        
        HttpResponse<String> response = client.send(request,
            HttpResponse.BodyHandlers.ofString());
        
        System.out.printf("POST %s -> Status: %d%n", url, response.statusCode());
        return response.body();
    }
    
    public static void main(String[] args) throws Exception {
        // GET example (using a public API)
        try {
            String body = get("https://httpbin.org/get");
            System.out.println("Response: " + body.substring(0, Math.min(200, body.length())) + "...");
        } catch (Exception e) {
            System.out.println("GET failed (expected if no internet): " + e.getMessage());
        }
        
        // Async multiple requests
        List<String> urls = List.of(
            "https://httpbin.org/delay/1",
            "https://httpbin.org/delay/2",
            "https://httpbin.org/delay/1"
        );
        
        System.out.println("\nAsync requests started...");
        long start = System.currentTimeMillis();
        
        List<CompletableFuture<String>> futures = urls.stream()
            .map(HttpClientDemo::getAsync)
            .toList();
        
        CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
        
        System.out.printf("All done in %dms%n", System.currentTimeMillis() - start);
    }
}
```

---

## 25.2 TCP Socket

```java
import java.net.*;
import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class TCPSocket {
    
    // Simple Echo Server
    static class EchoServer {
        private final int port;
        private ServerSocket serverSocket;
        private ExecutorService pool = Executors.newFixedThreadPool(10);
        private volatile boolean running = false;
        
        EchoServer(int port) { this.port = port; }
        
        void start() throws IOException {
            serverSocket = new ServerSocket(port);
            running = true;
            System.out.println("Echo server started on port " + port);
            
            while (running) {
                try {
                    Socket client = serverSocket.accept();
                    pool.submit(() -> handleClient(client));
                } catch (IOException e) {
                    if (running) System.out.println("Accept error: " + e.getMessage());
                }
            }
        }
        
        private void handleClient(Socket client) {
            String clientAddr = client.getInetAddress().getHostAddress() + ":" + client.getPort();
            System.out.println("Client connected: " + clientAddr);
            
            try (BufferedReader in = new BufferedReader(
                     new InputStreamReader(client.getInputStream()));
                 PrintWriter out = new PrintWriter(client.getOutputStream(), true)) {
                
                String line;
                while ((line = in.readLine()) != null) {
                    System.out.println("[" + clientAddr + "] " + line);
                    out.println("Echo: " + line);
                    if ("BYE".equalsIgnoreCase(line)) break;
                }
            } catch (IOException e) {
                System.out.println("Client error: " + e.getMessage());
            } finally {
                System.out.println("Client disconnected: " + clientAddr);
                try { client.close(); } catch (IOException e) {}
            }
        }
        
        void stop() throws IOException {
            running = false;
            serverSocket.close();
            pool.shutdown();
        }
    }
    
    // Client
    static class EchoClient {
        private final String host;
        private final int port;
        
        EchoClient(String host, int port) {
            this.host = host;
            this.port = port;
        }
        
        void sendMessages(String... messages) throws IOException {
            try (Socket socket = new Socket(host, port);
                 PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
                 BufferedReader in = new BufferedReader(
                     new InputStreamReader(socket.getInputStream()))) {
                
                for (String msg : messages) {
                    out.println(msg);
                    String response = in.readLine();
                    System.out.println("Sent: " + msg + " | Got: " + response);
                }
            }
        }
    }
    
    public static void main(String[] args) throws Exception {
        int port = 9090;
        EchoServer server = new EchoServer(port);
        
        // Start server in background
        Thread serverThread = new Thread(() -> {
            try { server.start(); }
            catch (IOException e) { System.out.println("Server error: " + e.getMessage()); }
        });
        serverThread.setDaemon(true);
        serverThread.start();
        Thread.sleep(500);  // wait for server to start
        
        // Connect client
        EchoClient client = new EchoClient("localhost", port);
        client.sendMessages("Hello", "World", "Java", "BYE");
        
        server.stop();
        System.out.println("Done");
    }
}
```

---

## 25.3 UDP Socket

```java
import java.net.*;

public class UDPSocket {
    
    static class UDPServer {
        private final int port;
        private volatile boolean running;
        
        UDPServer(int port) { this.port = port; }
        
        void start() throws Exception {
            try (DatagramSocket socket = new DatagramSocket(port)) {
                socket.setSoTimeout(5000);
                running = true;
                System.out.println("UDP server on port " + port);
                
                byte[] buffer = new byte[1024];
                
                while (running) {
                    try {
                        DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                        socket.receive(packet);
                        
                        String message = new String(packet.getData(), 0, packet.getLength());
                        System.out.println("[UDP] Received: " + message);
                        
                        // Echo back
                        String response = "Echo: " + message;
                        byte[] responseBytes = response.getBytes();
                        DatagramPacket reply = new DatagramPacket(
                            responseBytes, responseBytes.length,
                            packet.getAddress(), packet.getPort());
                        socket.send(reply);
                    } catch (SocketTimeoutException e) {
                        // timeout - check if still running
                    }
                }
            }
        }
        
        void stop() { running = false; }
    }
    
    static void sendUDP(String message, String host, int port) throws Exception {
        try (DatagramSocket socket = new DatagramSocket()) {
            byte[] data = message.getBytes();
            InetAddress addr = InetAddress.getByName(host);
            DatagramPacket packet = new DatagramPacket(data, data.length, addr, port);
            socket.send(packet);
            System.out.println("UDP sent: " + message);
            
            // Wait for echo
            socket.setSoTimeout(2000);
            byte[] buffer = new byte[1024];
            DatagramPacket response = new DatagramPacket(buffer, buffer.length);
            socket.receive(response);
            System.out.println("UDP echo: " + new String(response.getData(), 0, response.getLength()));
        }
    }
    
    public static void main(String[] args) throws Exception {
        UDPServer server = new UDPServer(9091);
        Thread serverThread = new Thread(() -> {
            try { server.start(); } catch (Exception e) {}
        });
        serverThread.setDaemon(true);
        serverThread.start();
        Thread.sleep(200);
        
        sendUDP("Hello UDP", "localhost", 9091);
        sendUDP("Java Network", "localhost", 9091);
        
        server.stop();
    }
}
```

---

## 25.4 Simple REST API Client

```java
import java.net.http.*;
import java.net.URI;
import java.util.*;

public class RestApiClient {
    
    // Generic REST client
    static class ApiClient {
        private final String baseUrl;
        private final HttpClient http;
        private final Map<String, String> defaultHeaders = new HashMap<>();
        
        ApiClient(String baseUrl) {
            this.baseUrl = baseUrl;
            this.http = HttpClient.newHttpClient();
            defaultHeaders.put("Content-Type", "application/json");
            defaultHeaders.put("Accept", "application/json");
        }
        
        void setAuth(String token) {
            defaultHeaders.put("Authorization", "Bearer " + token);
        }
        
        private HttpRequest.Builder buildRequest(String path) {
            HttpRequest.Builder builder = HttpRequest.newBuilder()
                .uri(URI.create(baseUrl + path));
            defaultHeaders.forEach(builder::header);
            return builder;
        }
        
        String get(String path) throws Exception {
            HttpRequest req = buildRequest(path).GET().build();
            return http.send(req, HttpResponse.BodyHandlers.ofString()).body();
        }
        
        String post(String path, String body) throws Exception {
            HttpRequest req = buildRequest(path)
                .POST(HttpRequest.BodyPublishers.ofString(body)).build();
            return http.send(req, HttpResponse.BodyHandlers.ofString()).body();
        }
        
        String put(String path, String body) throws Exception {
            HttpRequest req = buildRequest(path)
                .PUT(HttpRequest.BodyPublishers.ofString(body)).build();
            return http.send(req, HttpResponse.BodyHandlers.ofString()).body();
        }
        
        int delete(String path) throws Exception {
            HttpRequest req = buildRequest(path).DELETE().build();
            return http.send(req, HttpResponse.BodyHandlers.discarding()).statusCode();
        }
    }
    
    // Usage example (JSONPlaceholder API)
    public static void main(String[] args) {
        ApiClient api = new ApiClient("https://jsonplaceholder.typicode.com");
        
        try {
            // Get all todos
            String todos = api.get("/todos?_limit=3");
            System.out.println("Todos: " + todos.substring(0, Math.min(200, todos.length())));
            
            // Get specific user
            String user = api.get("/users/1");
            System.out.println("\nUser 1: " + user.substring(0, Math.min(200, user.length())));
            
            // Create post
            String newPost = api.post("/posts",
                "{\"title\":\"Test Post\",\"body\":\"Hello World\",\"userId\":1}");
            System.out.println("\nCreated: " + newPost);
            
        } catch (Exception e) {
            System.out.println("API call failed (need internet): " + e.getMessage());
        }
    }
}
```

---

## 25.5 สรุป Part 25

ในบทนี้คุณได้เรียนรู้:

✅ Java 11+ HttpClient  
✅ Synchronous และ Asynchronous requests  
✅ TCP Socket server/client  
✅ UDP Socket  
✅ REST API client  

---

*[← Part 24: JDBC](./part-24-jdbc.md) | [Part 26: Reflection & Annotations →](./part-26-reflection.md)*
