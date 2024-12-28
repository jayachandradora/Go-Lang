# Web Request

In Go, handling web requests is a core part of building web servers and APIs. Go provides powerful packages such as `net/http` to work with HTTP requests and responses. Below are various types of web request handling scenarios in Go, with code examples for each.

### 1. **Basic Web Server**

A simple web server that listens to HTTP requests and responds with a message.

#### Example: Basic Web Server
```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

func main() {
	http.HandleFunc("/", handler) // Associate handler with URL path "/"
	err := http.ListenAndServe(":8080", nil) // Start the web server on port 8080
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

- **Explanation**: `http.HandleFunc("/", handler)` associates the `handler` function with the root URL path `/`. When a request is made to `/`, the `handler` function is invoked, and it writes the message `"Hello, World!"` to the response.
  
### 2. **Handling Different HTTP Methods (GET, POST, PUT, DELETE)**

Web servers often need to handle different HTTP methods like GET, POST, PUT, DELETE, etc. You can differentiate these using `http.HandleFunc` or `http.MethodXXX`.

#### Example: Handling Different HTTP Methods
```go
package main

import (
	"fmt"
	"net/http"
)

func getHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "GET request received")
}

func postHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "POST request received")
}

func putHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "PUT request received")
}

func deleteHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "DELETE request received")
}

func main() {
	http.HandleFunc("/get", getHandler)    // Handle GET request
	http.HandleFunc("/post", postHandler)  // Handle POST request
	http.HandleFunc("/put", putHandler)    // Handle PUT request
	http.HandleFunc("/delete", deleteHandler) // Handle DELETE request
	
	err := http.ListenAndServe(":8080", nil) // Start the web server
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

- **Explanation**: Each handler corresponds to a specific HTTP method:
  - **GET**: Returns a response for a GET request.
  - **POST**: Handles POST requests.
  - **PUT**: Handles PUT requests.
  - **DELETE**: Handles DELETE requests.

### 3. **Handling Query Parameters**

Query parameters are typically passed in the URL. You can access them through the `r.URL.Query()` method.

#### Example: Handling Query Parameters
```go
package main

import (
	"fmt"
	"net/http"
)

func queryHandler(w http.ResponseWriter, r *http.Request) {
	// Extract query parameters
	name := r.URL.Query().Get("name")
	age := r.URL.Query().Get("age")
	fmt.Fprintf(w, "Name: %s, Age: %s", name, age)
}

func main() {
	http.HandleFunc("/query", queryHandler)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

- **Explanation**: You can access query parameters using `r.URL.Query().Get("param_name")`. In this case, the query parameters `name` and `age` are extracted from the URL and printed in the response.

#### Example Request:
```
http://localhost:8080/query?name=John&age=30
```

#### Example Response:
```
Name: John, Age: 30
```

### 4. **Handling URL Path Parameters**

URL path parameters (also known as route parameters) can be extracted from the URL using a router package or custom logic.

#### Example: Handling URL Path Parameters
```go
package main

import (
	"fmt"
	"net/http"
	"strings"
)

func pathParamHandler(w http.ResponseWriter, r *http.Request) {
	// Extract the "id" path parameter
	segments := strings.Split(r.URL.Path, "/")
	if len(segments) > 1 {
		id := segments[1] // Path parameter is the second segment
		fmt.Fprintf(w, "Received ID: %s", id)
	} else {
		http.Error(w, "ID not provided", http.StatusBadRequest)
	}
}

func main() {
	http.HandleFunc("/item/", pathParamHandler) // Item URL with path parameter
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

- **Explanation**: This example demonstrates how to extract a parameter from the URL path. The URL path is split into segments using `strings.Split`, and the second segment is extracted as the `id`.

#### Example Request:
```
http://localhost:8080/item/123
```

#### Example Response:
```
Received ID: 123
```

### 5. **Handling JSON Data (POST Request)**

To handle JSON data in POST requests, you can use the `json` package to parse and respond with JSON.

#### Example: Handling JSON in a POST Request
```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type Person struct {
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func postJSONHandler(w http.ResponseWriter, r *http.Request) {
	// Decode JSON from the request body
	var person Person
	err := json.NewDecoder(r.Body).Decode(&person)
	if err != nil {
		http.Error(w, "Invalid JSON", http.StatusBadRequest)
		return
	}

	// Respond with the same data
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(person)
}

func main() {
	http.HandleFunc("/postjson", postJSONHandler)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

- **Explanation**: 
  - The `postJSONHandler` function decodes JSON data from the request body into a `Person` struct.
  - The response is encoded as JSON using `json.NewEncoder(w).Encode(person)`.

#### Example Request (using `curl`):
```bash
curl -X POST http://localhost:8080/postjson -d '{"name": "John", "age": 30}' -H "Content-Type: application/json"
```

#### Example Response:
```json
{
  "name": "John",
  "age": 30
}
```

### 6. **File Uploads**

To handle file uploads in Go, you can use the `http.Request`'s `FormFile` method to extract the uploaded file.

#### Example: Handling File Uploads
```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
)

func fileUploadHandler(w http.ResponseWriter, r *http.Request) {
	// Parse the form to retrieve the file
	err := r.ParseMultipartForm(10 << 20) // Limit file size to 10MB
	if err != nil {
		http.Error(w, "Error parsing form", http.StatusBadRequest)
		return
	}

	// Get the file from the form
	file, _, err := r.FormFile("file")
	if err != nil {
		http.Error(w, "Error retrieving file", http.StatusBadRequest)
		return
	}
	defer file.Close()

	// Save the file
	out, err := os.Create("uploaded_file.txt")
	if err != nil {
		http.Error(w, "Error saving file", http.StatusInternalServerError)
		return
	}
	defer out.Close()

	_, err = io.Copy(out, file)
	if err != nil {
		http.Error(w, "Error copying file", http.StatusInternalServerError)
		return
	}

	fmt.Fprintf(w, "File uploaded successfully.")
}

func main() {
	http.HandleFunc("/upload", fileUploadHandler)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

- **Explanation**: 
  - The handler uses `r.FormFile("file")` to get the uploaded file.
  - It saves the file to the server using `io.Copy`.

#### Example Request (using `curl`):
```bash
curl -X POST -F "file=@yourfile.txt" http://localhost:8080/upload
```

### 7. **Serving Static Files**

Go has built-in support for serving static files using the `http.ServeFile` or `http.FileServer` functions.

#### Example: Serving Static Files
```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	// Serve static files from the current directory
	http.Handle("/", http.FileServer(http.Dir("./static")))

	// Start the server
	fmt.Println("Serving static files at http://localhost:8080")
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

- **Explanation**: `http.FileServer` serves files from a directory. In this case, it serves files from the `./static` folder.

---

### 8. **Middleware (Logging, Authentication)**

Middleware is used to add functionality to your request handling pipeline

 (e.g., logging, authentication).

#### Example: Middleware (Logging)
```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func logRequestMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		next.ServeHTTP(w, r)
		fmt.Printf("Request %s %s took %s\n", r.Method, r.URL.Path, time.Since(start))
	})
}

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, World!")
}

func main() {
	http.Handle("/", logRequestMiddleware(http.HandlerFunc(handler)))

	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println("Error starting server:", err)
	}
}
```

- **Explanation**: `logRequestMiddleware` wraps the handler to log the request details (method, path, and duration).

### Summary of Common Go Web Handling Techniques:

- **Basic Server**: Using `http.HandleFunc` to create basic handlers.
- **HTTP Methods**: Handle different HTTP methods like GET, POST, PUT, DELETE.
- **Query Parameters**: Using `r.URL.Query()` to access URL query parameters.
- **URL Path Parameters**: Extract path parameters by splitting the URL path or using a router.
- **JSON Handling**: Using `encoding/json` to parse and respond with JSON data.
- **File Uploads**: Handling file uploads with `r.FormFile`.
- **Static Files**: Serving static files using `http.FileServer`.
- **Middleware**: Adding middleware for logging or authentication.

These examples cover most common patterns you’ll encounter while handling web requests in Go. Depending on your specific use case, you can mix and match these techniques to build robust web servers.
