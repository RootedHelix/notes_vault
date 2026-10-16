

## System.out.printf()

This code snippet demonstrates formatted output using `System.out.printf()` in Java. Here's a detailed breakdown:

---

### 1. **`System.out.printf()`**
`printf` is a method used for formatted output, similar to `printf` in C. The format specifiers determine how the output will be formatted.

---

### 2. **Code Explanation**
#### **Code:**
```java
System.out.printf("%-15s", s.split(" ")[0]);
System.out.printf("%03d", Integer.parseInt(s.split(" ")[1]));
System.out.println();
```

#### **Step-by-step Breakdown:**

1. **`s.split(" ")`**
   - Splits the string `s` into parts based on spaces (`" "`).
   - `s.split(" ")[0]` retrieves the first word.
   - `s.split(" ")[1]` retrieves the second word.

2. **`%-15s`**
   - This is a format specifier for strings:
     - `%`: Indicates the beginning of a format specifier.
     - `-`: Left-align the output.
     - `15`: The output should occupy at least 15 characters. If the string is shorter, it's padded with spaces on the right.
     - `s`: Specifies the data type (string).
   - Example: If the first part of the split string is `"Hello"`, the output would be:
     ```
     Hello          (with 10 spaces on the right)
     ```

3. **`%03d`**
   - This is a format specifier for integers:
     - `%`: Indicates the beginning of a format specifier.
     - `0`: Pads the output with leading zeros if it's shorter than the specified width.
     - `3`: The output should occupy at least 3 characters.
     - `d`: Specifies the data type (decimal integer).
   - Example: If the second part of the split string is `"7"`, `Integer.parseInt()` converts it to an integer, and the output would be:
     ```
     007
     ```

4. **`System.out.println()`**
   - Moves the cursor to the next line after printing.

---

### 3. **Example**
Suppose the string `s` is:
```java
String s = "Hello 7";
```

**Execution:**
- `s.split(" ")[0]` is `"Hello"`.
- `s.split(" ")[1]` is `"7"` (converted to an integer).

**Formatted Output:**
```
Hello          007
```

---

### 4. **Key Points**
- **Left alignment (`-`)** and **padding (`0`)** are useful for creating neatly aligned tabular output.
- **Split and parse:** `split()` separates the string, and `Integer.parseInt()` ensures numeric formatting.
- This style of output is commonly used for creating formatted tables or logs in console applications.

`public static void main(String[] args) {

`Scanner sc=new Scanner(System.in);
`System.out.println("================================");

`for(int i=0;i<3;i++){

`String s1 = sc.nextLine();
`System.out.printf("%-15s", s1.split(" ")[0]);
`System.out.printf("%03d", Integer.parseInt(s1.split(" ")[1]));
`System.out.println();

`}
`System.out.println("================================");

  

}


## Buffered Reader

The line:

```java
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
```

is a way to set up a **BufferedReader** for reading input from the console (standard input) in Java. Here's a breakdown of what this means:

---

### **Components**
1. **`System.in`**
   - Represents the standard input stream (typically the keyboard).
   - Reads raw bytes from the console.

2. **`InputStreamReader`**
   - Converts the raw byte input from `System.in` into character data.
   - Ensures that the byte input is properly interpreted as text using the default character encoding or a specified one.

3. **`BufferedReader`**
   - Wraps the `InputStreamReader` to provide efficient reading of text data.
   - Buffers the input to reduce the number of reads directly from the console, making it faster for larger inputs.
   - Provides convenient methods like `readLine()` to read whole lines of input.

---

### **Usage**
The `BufferedReader` object created by this line is now ready to read input from the console efficiently.

#### **Reading Input**
Use the `readLine()` method to read a line of input:
```java
String input = bufferedReader.readLine();
```
- **Returns:** A string containing the input from the user (excluding the newline character).

---

### **Example: Reading and Processing Input**
```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.IOException;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));

        System.out.println("Enter your name:");
        String name = bufferedReader.readLine();

        System.out.println("Enter your age:");
        int age = Integer.parseInt(bufferedReader.readLine()); // Converting the input to an integer

        System.out.println("Name: " + name);
        System.out.println("Age: " + age);
    }
}
```

---

### **Key Features**
- **Efficiency:** Buffered reading minimizes the number of read operations, making it faster for large input.
- **Convenience:** `readLine()` simplifies input handling by returning strings directly.
- **Flexibility:** You can parse the input string into other data types (e.g., integers or doubles) as needed.

---

### **Important Notes**
- **Exception Handling:** Methods like `readLine()` can throw an `IOException`, so your code should handle this, either by using `throws` in the method signature or a `try-catch` block.
- **Parsing:** If the input needs to be converted (e.g., to integers or doubles), use methods like `Integer.parseInt()` or `Double.parseDouble()`.
- **Alternatives:** For simpler input handling, `Scanner` is often used instead of `BufferedReader`. However, `BufferedReader` is more efficient for large inputs and often preferred in competitive programming.


BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
int N = Integer.parseInt(bufferedReader.readLine().trim());
bufferedReader.close();