# functions and methods

In Go, functions and methods are central to the way code is structured and organized. Here's an overview of different types of functions and methods in Go with code examples.

### 1. **Basic Function**

A function is a block of code that can be called to perform a task. It can have parameters and return values.

#### Example of a Basic Function:
```go
package main

import "fmt"

// Basic function that takes two integers and returns their sum
func add(a int, b int) int {
    return a + b
}

func main() {
    result := add(3, 4)
    fmt.Println("Sum:", result)
}
```

**Output:**
```
Sum: 7
```

### 2. **Function with Multiple Return Values**

Go allows functions to return multiple values, which is often used for error handling and other purposes.

#### Example of a Function with Multiple Return Values:
```go
package main

import "fmt"

// Function that returns multiple values
func divide(a int, b int) (int, string) {
    if b == 0 {
        return 0, "Error: Division by zero"
    }
    return a / b, "Success"
}

func main() {
    result, message := divide(10, 2)
    fmt.Println("Result:", result, ", Message:", message)
    
    result, message = divide(10, 0)
    fmt.Println("Result:", result, ", Message:", message)
}
```

**Output:**
```
Result: 5 , Message: Success
Result: 0 , Message: Error: Division by zero
```

### 3. **Named Return Values**

You can give return values a name and return them implicitly by using the `return` statement without specifying the return values.

#### Example of Named Return Values:
```go
package main

import "fmt"

// Function with named return values
func multiply(a int, b int) (result int) {
    result = a * b
    return // Implicit return of result
}

func main() {
    res := multiply(4, 5)
    fmt.Println("Multiplication Result:", res)
}
```

**Output:**
```
Multiplication Result: 20
```

### 4. **Variadic Functions**

Go supports variadic functions, which allow you to pass a variable number of arguments to a function.

#### Example of a Variadic Function:
```go
package main

import "fmt"

// Variadic function that takes multiple integers
func sum(numbers ...int) int {
    total := 0
    for _, number := range numbers {
        total += number
    }
    return total
}

func main() {
    result := sum(1, 2, 3, 4, 5)
    fmt.Println("Sum of numbers:", result)
}
```

**Output:**
```
Sum of numbers: 15
```

### 5. **Functions as First-Class Citizens (Function Types)**

In Go, functions are first-class citizens, meaning you can assign them to variables and pass them around as arguments.

#### Example of Functions as First-Class Citizens:
```go
package main

import "fmt"

// Function type definition
type operation func(int, int) int

// Function that uses a function as an argument
func applyOperation(a, b int, op operation) int {
    return op(a, b)
}

// Function to add two numbers
func add(a, b int) int {
    return a + b
}

func main() {
    result := applyOperation(10, 20, add)
    fmt.Println("Addition Result:", result)
}
```

**Output:**
```
Addition Result: 30
```

### 6. **Methods in Go**

Methods are functions with a receiver, which can be associated with a specific type (struct). Methods allow you to define behavior on types.

#### Example of a Method:
```go
package main

import "fmt"

// Define a struct
type Rectangle struct {
    width, height int
}

// Method with a receiver of type Rectangle
func (r Rectangle) area() int {
    return r.width * r.height
}

func main() {
    rect := Rectangle{width: 10, height: 5}
    fmt.Println("Area of rectangle:", rect.area())
}
```

**Output:**
```
Area of rectangle: 50
```

### 7. **Pointer Receiver vs Value Receiver**

In Go, you can define methods with either a value receiver or a pointer receiver.

#### Example of a Method with Pointer Receiver:
```go
package main

import "fmt"

// Define a struct
type Counter struct {
    count int
}

// Method with a pointer receiver (so it can modify the original object)
func (c *Counter) increment() {
    c.count++
}

func main() {
    c := Counter{count: 0}
    c.increment() // Modifies the original Counter
    fmt.Println("Count after increment:", c.count)
}
```

**Output:**
```
Count after increment: 1
```

#### Example of Method with Value Receiver:
```go
package main

import "fmt"

// Define a struct
type Rectangle struct {
    width, height int
}

// Method with a value receiver (does not modify the original object)
func (r Rectangle) area() int {
    return r.width * r.height
}

func main() {
    rect := Rectangle{width: 10, height: 5}
    fmt.Println("Area of rectangle:", rect.area())
}
```

**Output:**
```
Area of rectangle: 50
```

### 8. **Method Overloading in Go**

Go does not support method overloading (i.e., multiple methods with the same name but different parameter types). However, you can achieve similar behavior using different function names or by passing parameters of different types using interfaces.

#### Example of Method Overloading Alternative (Using Interfaces):
```go
package main

import "fmt"

// Define a struct
type Printer struct{}

// Method to print integers
func (p Printer) print(value int) {
    fmt.Println("Printing integer:", value)
}

// Method to print strings
func (p Printer) print(value string) {
    fmt.Println("Printing string:", value)
}

func main() {
    p := Printer{}
    p.print(42)           // Prints an integer
    p.print("Hello Go!")  // Prints a string
}
```

Since Go does not support method overloading, you would typically use different method names (e.g., `printInt` and `printString`) or interfaces to handle different types.

### 9. **Anonymous Functions (Lambdas)**

Go also supports anonymous functions (lambdas), which can be defined inline without naming them.

#### Example of an Anonymous Function:
```go
package main

import "fmt"

func main() {
    // Define and invoke an anonymous function
    result := func(a, b int) int {
        return a + b
    }(5, 10)
    
    fmt.Println("Anonymous function result:", result)
}
```

**Output:**
```
Anonymous function result: 15
```

### 10. **Function Closures**

Go supports closures, where a function can "close over" variables from its surrounding context.

#### Example of a Closure:
```go
package main

import "fmt"

func outer() func() int {
    x := 10
    return func() int {
        x++
        return x
    }
}

func main() {
    closure := outer()
    fmt.Println(closure()) // 11
    fmt.Println(closure()) // 12
    fmt.Println(closure()) // 13
}
```

**Output:**
```
11
12
13
```

### Summary of Function and Method Types in Go:

- **Basic function**: A function with parameters and a return value.
- **Multiple return values**: Functions can return more than one value.
- **Named return values**: Return values can be named and returned implicitly.
- **Variadic functions**: Functions that accept a variable number of arguments.
- **Functions as first-class citizens**: Functions can be assigned to variables, passed as arguments, and returned from other functions.
- **Methods**: Functions associated with a type (typically structs).
- **Pointer receivers**: Methods that modify the object they are called on.
- **Value receivers**: Methods that do not modify the object.
- **Anonymous functions**: Functions that are defined inline without a name.
- **Closures**: Functions that can capture and use variables from their surrounding scope.

These are the common ways of defining and using functions and methods in Go!
