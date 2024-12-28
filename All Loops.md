# Loops

In Go, there is essentially one `for` loop construct, but it can be used in several ways to emulate different types of loops you might be familiar with in other programming languages. Here are the different forms of the `for` loop in Go:

### 1. **Basic For Loop (Traditional C-style loop)**
This is the most common type of `for` loop, where you initialize the loop variable, check a condition, and modify the variable after each iteration.

```go
package main

import "fmt"

func main() {
    // Traditional for loop
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }
}
```

**Output:**
```
0
1
2
3
4
```

### 2. **For Loop with Only Condition (Equivalent to while loop in other languages)**
You can omit the initialization and post statement, leaving only the condition, similar to a `while` loop in languages like C or Java.

```go
package main

import "fmt"

func main() {
    i := 0
    // Loop with condition only
    for i < 5 {
        fmt.Println(i)
        i++
    }
}
```

**Output:**
```
0
1
2
3
4
```

### 3. **For Loop with Infinite Loop (Equivalent to a `while(true)` loop)**
If you omit both the initialization and condition, it creates an infinite loop, which you can exit using a `break` statement.

```go
package main

import "fmt"

func main() {
    i := 0
    // Infinite for loop
    for {
        fmt.Println(i)
        i++
        if i >= 5 {
            break
        }
    }
}
```

**Output:**
```
0
1
2
3
4
```

### 4. **For-Range Loop (Used for Iterating Over Arrays, Slices, Maps, Strings, etc.)**
Go provides a `for`-`range` loop, which is typically used to iterate over collections such as arrays, slices, maps, and strings.

#### 4.1 Iterating Over an Array/Slice
```go
package main

import "fmt"

func main() {
    nums := []int{10, 20, 30, 40, 50}
    
    // Using for-range to iterate over slice
    for index, value := range nums {
        fmt.Printf("Index: %d, Value: %d\n", index, value)
    }
}
```

**Output:**
```
Index: 0, Value: 10
Index: 1, Value: 20
Index: 2, Value: 30
Index: 3, Value: 40
Index: 4, Value: 50
```

#### 4.2 Iterating Over a Map
```go
package main

import "fmt"

func main() {
    cities := map[string]string{
        "USA":    "Washington",
        "Japan":  "Tokyo",
        "India":  "New Delhi",
    }

    // Using for-range to iterate over map
    for country, city := range cities {
        fmt.Printf("Country: %s, Capital: %s\n", country, city)
    }
}
```

**Output:**
```
Country: USA, Capital: Washington
Country: Japan, Capital: Tokyo
Country: India, Capital: New Delhi
```

#### 4.3 Iterating Over a String (Character by Character)
```go
package main

import "fmt"

func main() {
    str := "Hello, Go!"
    
    // Using for-range to iterate over a string
    for index, runeValue := range str {
        fmt.Printf("Index: %d, Rune: %c\n", index, runeValue)
    }
}
```

**Output:**
```
Index: 0, Rune: H
Index: 1, Rune: e
Index: 2, Rune: l
Index: 3, Rune: l
Index: 4, Rune: o
Index: 5, Rune: ,
Index: 6, Rune:  
Index: 7, Rune: G
Index: 8, Rune: o
Index: 9, Rune: !
```

In the above examples:
- The first value in the `range` is the index (or key, for maps).
- The second value is the actual value in the collection.

You can also ignore either the index or the value by using the blank identifier `_`:
```go
for _, value := range nums { // Ignore the index
    fmt.Println(value)
}
```

### Summary:
- **Basic for loop**: `for i := 0; i < 5; i++ { ... }`
- **Condition-only loop** (similar to `while`): `for condition { ... }`
- **Infinite loop**: `for { ... }`
- **For-range loop** (used for iterating over arrays, slices, maps, strings): `for key, value := range collection { ... }`

These examples cover most of the ways you will use the `for` loop in Go.
