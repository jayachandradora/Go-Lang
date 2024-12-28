# Crud Operations

CRUD operations (Create, Read, Update, Delete) are fundamental to most applications, especially web applications or APIs. Below are examples of how to perform CRUD operations in Go, including basic HTTP server functionality to interact with data.

We'll cover examples using:

1. **In-memory storage (map)** for simplicity.
2. **JSON-based APIs** to handle requests and responses.
3. **Basic routing and HTTP handling** with `net/http`.

### 1. **Create** - Add new data
This operation creates or adds new data to your storage.

### 2. **Read** - Retrieve data
This operation fetches data based on some identifier (like an ID).

### 3. **Update** - Modify existing data
This operation modifies data by ID or other identifier.

### 4. **Delete** - Remove data
This operation removes data from storage based on the provided identifier.

We'll implement CRUD operations for a simple "Book" entity in a REST API. Each book will have an `ID`, `Title`, and `Author`.

### 1. **Define the Book struct**

```go
package main

type Book struct {
	ID     string `json:"id"`
	Title  string `json:"title"`
	Author string `json:"author"`
}

var books = make(map[string]Book) // In-memory storage for books
```

Here, we define a `Book` struct with `ID`, `Title`, and `Author` fields. We also create an in-memory map `books` where the key is the book's `ID` and the value is the `Book` struct itself.

---

### 2. **Create** - Add new book

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"github.com/gorilla/mux"
)

func createBook(w http.ResponseWriter, r *http.Request) {
	var book Book
	err := json.NewDecoder(r.Body).Decode(&book)
	if err != nil {
		http.Error(w, "Invalid JSON", http.StatusBadRequest)
		return
	}

	// Check if book ID already exists
	if _, exists := books[book.ID]; exists {
		http.Error(w, "Book with this ID already exists", http.StatusConflict)
		return
	}

	books[book.ID] = book
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(book)
}
```

**Explanation**:
- The `createBook` function takes a JSON object, decodes it into a `Book`, and stores it in the `books` map.
- If a book with the same ID already exists, it returns a `409 Conflict` status.

---

### 3. **Read** - Get a book by ID

```go
func getBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	bookID := params["id"]

	// Check if book exists
	book, exists := books[bookID]
	if !exists {
		http.Error(w, "Book not found", http.StatusNotFound)
		return
	}

	// Return the book in JSON format
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(book)
}
```

**Explanation**:
- The `getBook` function retrieves a book by ID from the `books` map.
- If the book is not found, it returns a `404 Not Found` status.

---

### 4. **Update** - Update an existing book

```go
func updateBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	bookID := params["id"]

	// Retrieve the existing book
	book, exists := books[bookID]
	if !exists {
		http.Error(w, "Book not found", http.StatusNotFound)
		return
	}

	// Decode the new data for the book
	var updatedBook Book
	err := json.NewDecoder(r.Body).Decode(&updatedBook)
	if err != nil {
		http.Error(w, "Invalid JSON", http.StatusBadRequest)
		return
	}

	// Update the book's details
	book.Title = updatedBook.Title
	book.Author = updatedBook.Author
	books[bookID] = book

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(book)
}
```

**Explanation**:
- The `updateBook` function checks if the book exists, and then it decodes the new data from the request to update the book’s `Title` and `Author`.
- After updating, it responds with the updated book object.

---

### 5. **Delete** - Delete a book by ID

```go
func deleteBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	bookID := params["id"]

	// Check if book exists
	_, exists := books[bookID]
	if !exists {
		http.Error(w, "Book not found", http.StatusNotFound)
		return
	}

	// Delete the book from the map
	delete(books, bookID)

	w.WriteHeader(http.StatusNoContent) // No content to return
}
```

**Explanation**:
- The `deleteBook` function checks if the book exists in the map, and if so, deletes it.
- It responds with a `204 No Content` status, indicating the deletion was successful but there is no content to return.

---

### 6. **Set up Routes**

We'll use the `mux` router for handling HTTP routes. Install it using:

```bash
go get -u github.com/gorilla/mux
```

Now, let's create the routes for the CRUD operations.

```go
package main

import (
	"log"
	"net/http"
	"github.com/gorilla/mux"
)

func main() {
	// Create a new router
	r := mux.NewRouter()

	// Define the CRUD routes
	r.HandleFunc("/books", createBook).Methods("POST")       // Create a book
	r.HandleFunc("/books/{id}", getBook).Methods("GET")      // Read a book by ID
	r.HandleFunc("/books/{id}", updateBook).Methods("PUT")   // Update a book by ID
	r.HandleFunc("/books/{id}", deleteBook).Methods("DELETE")// Delete a book by ID

	// Start the server
	log.Println("Server starting at http://localhost:8080")
	log.Fatal(http.ListenAndServe(":8080", r))
}
```

**Explanation**:
- The `mux.NewRouter()` creates a new router.
- The routes are defined using `r.HandleFunc`, associating each HTTP method with its handler function.
  - `POST /books` -> `createBook`
  - `GET /books/{id}` -> `getBook`
  - `PUT /books/{id}` -> `updateBook`
  - `DELETE /books/{id}` -> `deleteBook`
- Finally, `http.ListenAndServe` starts the server on port `8080`.

---

### 7. **Complete Example**

Here's the complete Go program with all CRUD operations:

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"github.com/gorilla/mux"
)

type Book struct {
	ID     string `json:"id"`
	Title  string `json:"title"`
	Author string `json:"author"`
}

var books = make(map[string]Book)

func createBook(w http.ResponseWriter, r *http.Request) {
	var book Book
	err := json.NewDecoder(r.Body).Decode(&book)
	if err != nil {
		http.Error(w, "Invalid JSON", http.StatusBadRequest)
		return
	}

	// Check if book ID already exists
	if _, exists := books[book.ID]; exists {
		http.Error(w, "Book with this ID already exists", http.StatusConflict)
		return
	}

	books[book.ID] = book
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(book)
}

func getBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	bookID := params["id"]

	book, exists := books[bookID]
	if !exists {
		http.Error(w, "Book not found", http.StatusNotFound)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(book)
}

func updateBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	bookID := params["id"]

	book, exists := books[bookID]
	if !exists {
		http.Error(w, "Book not found", http.StatusNotFound)
		return
	}

	var updatedBook Book
	err := json.NewDecoder(r.Body).Decode(&updatedBook)
	if err != nil {
		http.Error(w, "Invalid JSON", http.StatusBadRequest)
		return
	}

	book.Title = updatedBook.Title
	book.Author = updatedBook.Author
	books[bookID] = book

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(book)
}

func deleteBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	bookID := params["id"]

	_, exists := books[bookID]
	if !exists {
		http.Error(w, "Book not found", http.StatusNotFound)
		return
	}

	delete(books, bookID)
	w.WriteHeader(http.StatusNoContent)
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/books", createBook).Methods("POST")
	r.HandleFunc("/books/{id}", getBook).Methods("GET")
	r.HandleFunc("/books/{id}", updateBook).Methods("PUT")
	r.HandleFunc("/books/{id}", deleteBook).Methods("DELETE")

	log.Println("Server starting at http

://localhost:8080")
	log.Fatal(http.ListenAndServe(":8080", r))
}
```

### 8. **Test the CRUD API**

#### **Create a new book (POST)**
```bash
curl -X POST http://localhost:8080/books -d '{"id": "1", "title": "Go Programming", "author": "John Doe"}' -H "Content-Type: application/json"
```

#### **Read a book by ID (GET)**
```bash
curl http://localhost:8080/books/1
```

#### **Update a book by ID (PUT)**
```bash
curl -X PUT http://localhost:8080/books/1 -d '{"title": "Go Programming 101", "author": "John Doe"}' -H "Content-Type: application/json"
```

#### **Delete a book by ID (DELETE)**
```bash
curl -X DELETE http://localhost:8080/books/1
```

---

### Summary

- **Create**: You can create new data (e.g., books) by sending a `POST` request with the data in the request body.
- **Read**: You can retrieve data (e.g., a book) using a `GET` request with the ID in the URL path.
- **Update**: You can modify existing data with a `PUT` request, providing the updated data in the body.
- **Delete**: You can remove data by sending a `DELETE` request with the ID in the URL path.

This example demonstrates basic CRUD operations in Go, working with a simple in-memory storage (a map), and can be extended to work with databases like MySQL, PostgreSQL, or MongoDB as required.
