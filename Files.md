# Files 

In Go, handling files involves several key operations, such as opening files, reading from them, writing to them, and closing them. Here's an overview of different types of file operations in Go with examples.

### 1. **Opening a File**

To open a file in Go, you can use the `os.Open` or `os.OpenFile` functions. `os.Open` opens a file for reading, and `os.OpenFile` provides more control, allowing you to specify how the file should be opened (read, write, append, etc.).

#### Example: Opening a File for Reading
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Open a file for reading
	file, err := os.Open("example.txt")
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close() // Ensure the file is closed after we are done

	// File successfully opened
	fmt.Println("File opened successfully:", file.Name())
}
```

### 2. **Reading from a File**

Go provides multiple ways to read data from files: using `Read`, `ReadLine`, `Scanner`, or `ioutil.ReadFile` (deprecated in Go 1.16, replaced by `os.ReadFile`).

#### Example: Reading File Contents Using `Read`
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Open file for reading
	file, err := os.Open("example.txt")
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Read file content
	buffer := make([]byte, 100)
	n, err := file.Read(buffer)
	if err != nil {
		fmt.Println("Error reading file:", err)
		return
	}

	fmt.Printf("Read %d bytes: %s\n", n, string(buffer[:n]))
}
```

#### Example: Reading File Contents Using `bufio.Scanner`
```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	// Open file for reading
	file, err := os.Open("example.txt")
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Create a scanner for reading the file line by line
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text()) // Print each line
	}

	if err := scanner.Err(); err != nil {
		fmt.Println("Error reading file:", err)
	}
}
```

#### Example: Reading Entire File Using `os.ReadFile`
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Read the entire file
	content, err := os.ReadFile("example.txt")
	if err != nil {
		fmt.Println("Error reading file:", err)
		return
	}

	fmt.Println("File content:")
	fmt.Println(string(content))
}
```

### 3. **Writing to a File**

You can use `os.Create`, `os.OpenFile`, and `ioutil.WriteFile` (deprecated in Go 1.16) to write to files. Use `os.Create` to create a new file, and `os.OpenFile` to open an existing file with write permissions.

#### Example: Writing to a File Using `os.Create`
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Create or overwrite a file
	file, err := os.Create("output.txt")
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	// Write to the file
	content := []byte("Hello, Go!\nThis is a new file.")
	_, err = file.Write(content)
	if err != nil {
		fmt.Println("Error writing to file:", err)
		return
	}

	fmt.Println("File written successfully.")
}
```

#### Example: Writing to a File Using `os.OpenFile`
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Open the file in append mode
	file, err := os.OpenFile("output.txt", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Write to the file
	content := []byte("\nAppended line to the file.")
	_, err = file.Write(content)
	if err != nil {
		fmt.Println("Error writing to file:", err)
		return
	}

	fmt.Println("Content appended to the file.")
}
```

#### Example: Writing to a File Using `os.WriteFile` (Go 1.16+)
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Write to the file (this will create the file if it doesn't exist)
	content := []byte("This is written using os.WriteFile in Go 1.16+.\n")
	err := os.WriteFile("output.txt", content, 0644)
	if err != nil {
		fmt.Println("Error writing to file:", err)
		return
	}

	fmt.Println("File written successfully.")
}
```

### 4. **Appending to a File**

You can append data to an existing file using `os.OpenFile` with the `os.O_APPEND` flag.

#### Example: Appending to a File
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Open the file in append mode
	file, err := os.OpenFile("output.txt", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Append to the file
	content := []byte("\nAppended some more text to the file.")
	_, err = file.Write(content)
	if err != nil {
		fmt.Println("Error writing to file:", err)
		return
	}

	fmt.Println("Content appended successfully.")
}
```

### 5. **File Info (Getting Metadata)**

You can get the file's metadata (such as size, permissions, and modification time) using `os.Stat`.

#### Example: Getting File Info
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Get file info
	fileInfo, err := os.Stat("example.txt")
	if err != nil {
		fmt.Println("Error getting file info:", err)
		return
	}

	// Display file information
	fmt.Println("File name:", fileInfo.Name())
	fmt.Println("File size:", fileInfo.Size())
	fmt.Println("File mode:", fileInfo.Mode())
	fmt.Println("Last modified:", fileInfo.ModTime())
	fmt.Println("Is directory?", fileInfo.IsDir())
}
```

### 6. **Renaming or Moving Files**

You can rename or move a file using `os.Rename`.

#### Example: Renaming a File
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Rename the file
	err := os.Rename("output.txt", "new_output.txt")
	if err != nil {
		fmt.Println("Error renaming file:", err)
		return
	}

	fmt.Println("File renamed successfully.")
}
```

### 7. **Deleting a File**

To delete a file, you can use `os.Remove`.

#### Example: Deleting a File
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Delete the file
	err := os.Remove("new_output.txt")
	if err != nil {
		fmt.Println("Error deleting file:", err)
		return
	}

	fmt.Println("File deleted successfully.")
}
```

### 8. **Creating and Using Directories**

You can create directories using `os.Mkdir` or `os.MkdirAll`. The `MkdirAll` function will create any necessary parent directories as well.

#### Example: Creating a Directory
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Create a new directory
	err := os.Mkdir("mydir", 0755)
	if err != nil {
		fmt.Println("Error creating directory:", err)
		return
	}

	fmt.Println("Directory created successfully.")
}
```

#### Example: Creating Parent Directories
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Create directories along the path
	err := os.MkdirAll("parent/child/grandchild", 0755)
	if err != nil {
		fmt.Println("Error creating directories:", err)
		return
	}

	fmt.Println("Parent and child directories created successfully.")
}
```

### 9. **Removing a Directory**

To remove a directory, you can use `os.Remove` (for empty directories) or `os.RemoveAll` (for non-empty directories).

#### Example: Removing a Directory
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Remove an empty directory
	err := os.Remove("mydir")
	if err != nil {
		fmt.Println("Error removing directory:", err)
		return
	}

	fmt.Println("Directory removed successfully.")
}
```

#### Example: Removing a Non-Empty Directory
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Remove a non-empty directory
	err := os.RemoveAll("parent")
	if err != nil {
		fmt.Println("Error removing directory:", err)
		return
	}

	fmt.Println("Non-empty directory removed successfully.")
}
```

### Summary of File Operations in Go:
- **Opening Files**: `

os.Open`, `os.OpenFile`
- **Reading Files**: `file.Read`, `bufio.Scanner`, `os.ReadFile`
- **Writing to Files**: `os.Create`, `os.OpenFile`, `os.WriteFile`
- **Appending to Files**: `os.OpenFile` with `os.O_APPEND`
- **File Metadata**: `os.Stat`
- **Renaming Files**: `os.Rename`
- **Deleting Files**: `os.Remove`
- **Directories**: `os.Mkdir`, `os.MkdirAll`, `os.RemoveAll`

These examples cover most common file operations in Go. Depending on your use case, you can combine these operations to work with files effectively.

# Editing and Merging two files and Reading line by line

Sure! Let's break down the task into two parts:

1. **Editing a File**: We will modify a file by reading its content, modifying it, and then writing the changes back to the file.
2. **Merging Two Files**: We will read two files and print their content line by line, merging them together.

### Example 1: Editing a File

We can open the file, read its contents, modify it, and then overwrite the file with the updated content.

#### Code to Edit a File:
```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strings"
)

func main() {
	// Open the file for reading
	fileName := "example.txt"
	file, err := os.OpenFile(fileName, os.O_RDWR, 0644)
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Read the contents of the file
	var content []string
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		content = append(content, scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		fmt.Println("Error reading file:", err)
		return
	}

	// Modify the content: For example, add a new line or update existing ones
	content = append(content, "This is the new line added to the file.")

	// Write back the modified content to the file
	file.Seek(0, 0) // Move the pointer to the beginning of the file
	file.Truncate(0) // Remove the old content
	writer := bufio.NewWriter(file)
	for _, line := range content {
		writer.WriteString(line + "\n")
	}

	// Ensure the content is written to the file
	writer.Flush()

	fmt.Println("File edited successfully.")
}
```

### Explanation of the Editing Example:
- **Opening the file**: We open the file with `os.OpenFile` using `os.O_RDWR` (read and write) permission.
- **Reading content**: We read the file line by line using a `bufio.Scanner` and store the lines in a slice.
- **Modifying content**: We append a new line to the slice.
- **Writing back the content**: We use `file.Seek(0, 0)` to move the pointer to the beginning of the file and `file.Truncate(0)` to clear its content before writing the modified content back using `bufio.NewWriter`.

---

### Example 2: Merging Two Files and Printing Line by Line

We will read two files and print their contents line by line, merging them one after the other.

#### Code to Merge Two Files:
```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func printFileContent(fileName string) {
	// Open the file for reading
	file, err := os.Open(fileName)
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Read the content of the file line by line and print it
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		fmt.Println("Error reading file:", err)
	}
}

func main() {
	// File names to merge
	file1 := "file1.txt"
	file2 := "file2.txt"

	// Print the contents of the first file
	fmt.Println("Contents of", file1+":")
	printFileContent(file1)

	// Print the contents of the second file
	fmt.Println("Contents of", file2+":")
	printFileContent(file2)
}
```

### Explanation of the Merging Example:
- **printFileContent function**: This function opens a file, reads it line by line using a `bufio.Scanner`, and prints each line.
- **Merging**: In the `main` function, we call `printFileContent` for both files, printing their contents one after another.

---

### Example Input and Output

Let's assume the content of the files:

#### `file1.txt`:
```
Line 1 in file1
Line 2 in file1
Line 3 in file1
```

#### `file2.txt`:
```
Line 1 in file2
Line 2 in file2
Line 3 in file2
```

#### Output (when you run the merging code):
```
Contents of file1.txt:
Line 1 in file1
Line 2 in file1
Line 3 in file1

Contents of file2.txt:
Line 1 in file2
Line 2 in file2
Line 3 in file2
```

### Modifying and Merging Two Files in One Go

If you want to modify the content of one file, append it to another, and then print the merged content, you can combine both operations into a single Go program.

Here’s an example:

#### Code for Merging and Modifying Files:
```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func modifyFileContent(fileName string) ([]string, error) {
	// Open the file for reading
	file, err := os.Open(fileName)
	if err != nil {
		return nil, err
	}
	defer file.Close()

	// Read the content of the file into a slice of strings
	var content []string
	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		content = append(content, scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		return nil, err
	}

	// Modify content (e.g., add a new line)
	content = append(content, "This is a new line added to the file.")

	return content, nil
}

func writeContentToFile(fileName string, content []string) error {
	// Open the file for writing (this will overwrite the file)
	file, err := os.OpenFile(fileName, os.O_RDWR|os.O_CREATE, 0644)
	if err != nil {
		return err
	}
	defer file.Close()

	// Write modified content to the file
	writer := bufio.NewWriter(file)
	for _, line := range content {
		writer.WriteString(line + "\n")
	}

	// Flush content to the file
	return writer.Flush()
}

func main() {
	// Files to modify and merge
	file1 := "file1.txt"
	file2 := "file2.txt"

	// Modify content of both files
	content1, err := modifyFileContent(file1)
	if err != nil {
		fmt.Println("Error reading file1:", err)
		return
	}

	content2, err := modifyFileContent(file2)
	if err != nil {
		fmt.Println("Error reading file2:", err)
		return
	}

	// Merge the content
	mergedContent := append(content1, content2...)

	// Write the merged content back to the first file (or any other file)
	err = writeContentToFile("merged_output.txt", mergedContent)
	if err != nil {
		fmt.Println("Error writing to merged_output.txt:", err)
		return
	}

	// Print the merged content line by line
	fmt.Println("Merged content:")
	for _, line := range mergedContent {
		fmt.Println(line)
	}
}
```

### Explanation:
1. **`modifyFileContent` function**: Reads a file, adds a new line to the content, and returns the modified content as a slice of strings.
2. **`writeContentToFile` function**: Writes the modified content back to a file.
3. **In `main`**: We modify both `file1.txt` and `file2.txt`, merge their contents, and write the merged content to `merged_output.txt`.

### Example Output for Merged Content:

After running this program, `merged_output.txt` will have the following content:

```
Line 1 in file1
Line 2 in file1
Line 3 in file1
This is a new line added to the file.
Line 1 in file2
Line 2 in file2
Line 3 in file2
This is a new line added to the file.
```

And it will be printed line by line:

```
Merged content:
Line 1 in file1
Line 2 in file1
Line 3 in file1
This is a new line added to the file.
Line 1 in file2
Line 2 in file2
Line 3 in file2
This is a new line added to the file.
```

---

### Summary:

- **Editing a File**: You can modify file content by reading it, making changes, and then writing the changes back.
- **Merging Two Files**: You can read and print the content of two files one after the other, or merge their contents into a new file.

