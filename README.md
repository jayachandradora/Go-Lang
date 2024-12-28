# Crud Operations with Postgres DB

When building production-ready CRUD operations in Go with PostgreSQL, there are several things you need to consider: handling database connections efficiently, managing errors, and structuring your code for scalability and maintainability. In this example, I'll guide you through the entire process of implementing production-ready CRUD operations in Go, using PostgreSQL as the backend database.

### Prerequisites

- Go installed on your machine (https://golang.org/dl/)
- PostgreSQL installed and running
- `github.com/lib/pq` PostgreSQL driver for Go
- `github.com/gorilla/mux` for routing

You can install the necessary Go packages using:

```bash
go get github.com/lib/pq
go get github.com/gorilla/mux
go get github.com/jmoiron/sqlx
```

### 1. **Setup PostgreSQL Database**

Let's assume we have a `books` table in PostgreSQL with the following schema:

```sql
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    author VARCHAR(100) NOT NULL,
    published_date DATE
);
```

This will create a simple `books` table where each book has an `id`, `title`, `author`, and `published_date`.

### 2. **Go Code for Production-Ready CRUD Operations**

We'll break the Go code into the following sections:
1. **Database Connection Setup**
2. **Model Definitions**
3. **CRUD Handlers (Create, Read, Update, Delete)**
4. **Router Setup and Server Start**

#### 1. **Database Connection Setup**

We’ll use `github.com/jmoiron/sqlx`, which is a higher-level wrapper around `database/sql`. It provides convenient methods for working with databases, especially with structs and named queries.

```go
package main

import (
	"database/sql"
	"fmt"
	"log"
	"os"

	_ "github.com/lib/pq"
	"github.com/jmoiron/sqlx"
)

var db *sqlx.DB

// Initialize the PostgreSQL database connection
func initDB() {
	var err error
	connStr := "user=postgres dbname=booksdb sslmode=disable" // Change this to your DB credentials
	db, err = sqlx.Open("postgres", connStr)
	if err != nil {
		log.Fatalf("Failed to connect to the database: %v", err)
	}

	// Check if the database connection is alive
	err = db.Ping()
	if err != nil {
		log.Fatalf("Failed to ping the database: %v", err)
	}

	log.Println("Database connected successfully")
}

// Close the database connection gracefully
func closeDB() {
	err := db.Close()
	if err != nil {
		log.Printf("Error closing DB connection: %v", err)
	}
}
```

- **Explanation**:
  - `sqlx.Open("postgres", connStr)`: Opens a connection to the PostgreSQL database.
  - `db.Ping()`: Checks if the connection is alive.
  - The `connStr` string should be updated to match your PostgreSQL connection credentials.

---

#### 2. **Model Definitions**

Here, we define a `Book` struct that represents the book entity in our database.

```go
package main

import (
	"time"
)

type Book struct {
	ID            int       `json:"id" db:"id"`
	Title         string    `json:"title" db:"title"`
	Author        string    `json:"author" db:"author"`
	PublishedDate time.Time `json:"published_date" db:"published_date"`
}
```

- **Explanation**:
  - `db:"column_name"`: The `db` struct tags map the struct fields to database columns.
  - `time.Time` is used for the `PublishedDate`, as it corresponds to the `DATE` type in PostgreSQL.

---

#### 3. **CRUD Handlers (Create, Read, Update, Delete)**

##### **Create a New Book**

```go
package main

import (
	"encoding/json"
	"net/http"
)

func createBook(w http.ResponseWriter, r *http.Request) {
	var book Book

	// Decode the JSON request body into the book struct
	err := json.NewDecoder(r.Body).Decode(&book)
	if err != nil {
		http.Error(w, "Invalid request payload", http.StatusBadRequest)
		return
	}

	// Insert the new book into the database
	query := `INSERT INTO books (title, author, published_date) VALUES ($1, $2, $3) RETURNING id`
	err = db.QueryRow(query, book.Title, book.Author, book.PublishedDate).Scan(&book.ID)
	if err != nil {
		http.Error(w, "Error inserting book into database", http.StatusInternalServerError)
		return
	}

	// Respond with the created book as JSON
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(book)
}
```

- **Explanation**:
  - We decode the incoming JSON request body into the `book` struct.
  - The query inserts the book data into the database and returns the new `id` using `RETURNING id`.
  - We send a response with HTTP status `201 Created` and the created book.

##### **Read a Book by ID**

```go
func getBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	id := params["id"]

	// Retrieve the book from the database by ID
	var book Book
	query := `SELECT id, title, author, published_date FROM books WHERE id = $1`
	err := db.Get(&book, query, id)
	if err != nil {
		if err == sql.ErrNoRows {
			http.Error(w, "Book not found", http.StatusNotFound)
			return
		}
		http.Error(w, "Error retrieving book", http.StatusInternalServerError)
		return
	}

	// Return the book as JSON
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(book)
}
```

- **Explanation**:
  - The `db.Get` method retrieves a single record from the database and maps it into the `book` struct.
  - If the book is not found (`sql.ErrNoRows`), we return `404 Not Found`.
  - Otherwise, we return the book as a JSON response.

##### **Update an Existing Book**

```go
func updateBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	id := params["id"]

	// Decode the JSON request body into the updated book struct
	var updatedBook Book
	err := json.NewDecoder(r.Body).Decode(&updatedBook)
	if err != nil {
		http.Error(w, "Invalid request payload", http.StatusBadRequest)
		return
	}

	// Update the book in the database
	query := `UPDATE books SET title = $1, author = $2, published_date = $3 WHERE id = $4`
	result, err := db.Exec(query, updatedBook.Title, updatedBook.Author, updatedBook.PublishedDate, id)
	if err != nil {
		http.Error(w, "Error updating book", http.StatusInternalServerError)
		return
	}

	// Check if any row was updated
	rowsAffected, _ := result.RowsAffected()
	if rowsAffected == 0 {
		http.Error(w, "Book not found", http.StatusNotFound)
		return
	}

	// Return the updated book as JSON
	updatedBook.ID = id
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(updatedBook)
}
```

- **Explanation**:
  - The `db.Exec` method executes the `UPDATE` query.
  - We check if any rows were affected (if `rowsAffected == 0`, the book was not found).
  - The updated book is returned as a JSON response.

##### **Delete a Book**

```go
func deleteBook(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	id := params["id"]

	// Delete the book from the database
	query := `DELETE FROM books WHERE id = $1`
	result, err := db.Exec(query, id)
	if err != nil {
		http.Error(w, "Error deleting book", http.StatusInternalServerError)
		return
	}

	// Check if any row was deleted
	rowsAffected, _ := result.RowsAffected()
	if rowsAffected == 0 {
		http.Error(w, "Book not found", http.StatusNotFound)
		return
	}

	w.WriteHeader(http.StatusNoContent) // No content to return
}
```

- **Explanation**:
  - We delete the book using the `DELETE` SQL query.
  - If no rows are affected (i.e., the book is not found), we return a `404 Not Found` response.
  - A successful deletion returns `204 No Content`.

---

#### 4. **Router Setup and Server Start**

Finally, let's set up the router and start the server.

```go
package main

import (
	"log"
	"net/http"
	"github.com/gorilla/mux"
)

func main() {
	// Initialize the database connection
	initDB()
	defer closeDB()

	// Set up the router
	r := mux.NewRouter()

	// Define the routes
	r.HandleFunc("/books", createBook).Methods("POST")        // Create book
	r.HandleFunc("/books/{id}", getBook).Methods("GET")       // Get book by ID
	r.HandleFunc("/books/{id}", updateBook).Methods("PUT")    // Update book by ID
	r.HandleFunc("/books/{id}", deleteBook).Methods("DELETE") // Delete book by ID

	// Start the server
	log.Println("Server is running on http://localhost:8080")
	log

.Fatal(http.ListenAndServe(":8080", r))
}
```

- **Explanation**:
  - We define routes for each CRUD operation (POST, GET, PUT, DELETE).
  - The server is started on `http://localhost:8080`.

---

### 5. **Testing the API**

Use `curl` or Postman to test the API.

1. **Create a book**:
   ```bash
   curl -X POST http://localhost:8080/books -d '{"title": "Go Programming", "author": "John Doe", "published_date": "2024-01-01"}' -H "Content-Type: application/json"
   ```

2. **Get a book by ID**:
   ```bash
   curl http://localhost:8080/books/1
   ```

3. **Update a book**:
   ```bash
   curl -X PUT http://localhost:8080/books/1 -d '{"title": "Go Programming 2", "author": "John Doe", "published_date": "2025-01-01"}' -H "Content-Type: application/json"
   ```

4. **Delete a book**:
   ```bash
   curl -X DELETE http://localhost:8080/books/1
   ```

### Conclusion

This setup provides a production-ready implementation of CRUD operations with PostgreSQL in Go. Key points include:

- **Database Connection**: Managed efficiently using `sqlx`.
- **Error Handling**: Proper error messages and HTTP status codes for each operation.
- **SQL Queries**: Safe SQL queries with parameterized values to prevent SQL injection.
- **Router**: `gorilla/mux` handles routing.
- **API Design**: Simple and effective RESTful API design for a book entity.

This is a scalable foundation that you can extend for more complex applications, including adding features like pagination, authentication, and advanced query handling.
