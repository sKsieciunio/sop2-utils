# Comprehensive Guide to String Manipulation in C Standard Library

## Table of Contents
1. [Introduction to C Strings](#introduction-to-c-strings)
2. [String.h Header](#stringh-header)
3. [Basic String Operations](#basic-string-operations)
4. [String Examination Functions](#string-examination-functions)
5. [String Comparison Functions](#string-comparison-functions)
6. [String Manipulation Functions](#string-manipulation-functions)
7. [Memory Management Functions](#memory-management-functions)
8. [Character Classification and Conversion](#character-classification-and-conversion)
9. [String to Number Conversions](#string-to-number-conversions)
10. [Buffer Safety and Best Practices](#buffer-safety-and-best-practices)
11. [Common String Algorithms](#common-string-algorithms)
12. [Advanced Topics](#advanced-topics)

## Introduction to C Strings

In C, strings are represented as arrays of characters terminated by a null character (`'\0'`). This termination character marks the end of the string and is crucial for string manipulation functions to work correctly.

```c
// String declaration and initialization methods
char str1[] = "Hello";          // Array initialized with string literal
char str2[6] = {'H','e','l','l','o','\0'}; // Explicit null termination
char *str3 = "World";           // Pointer to string literal (constant)
char str4[20];                  // Uninitialized char array
```

Key characteristics of C strings:

- No built-in string type; strings are character arrays
- Null-terminated (`'\0'` marks the end)
- String literals (e.g., "hello") are constants stored in read-only memory
- No automatic bounds checking (responsibility falls on the programmer)
- Length is determined at runtime by scanning for the null terminator

## String.h Header

The `string.h` header provides most string manipulation functions in C. It defines various functions for:

- String copying and concatenation
- String comparison
- String searching
- Memory manipulation
- String length determination

```c
#include <string.h>  // Required for using string manipulation functions
```

## Basic String Operations

### String Length

```c
size_t strlen(const char *str);
```

Returns the length of the string (excluding the null terminator).

```c
char message[] = "Hello, World!";
size_t length = strlen(message);  // length = 13
```

### String Copying

```c
char *strcpy(char *dest, const char *src);
char *strncpy(char *dest, const char *src, size_t n);
```

- `strcpy()` copies the entire source string (including null terminator) to destination
- `strncpy()` copies up to n characters (does not guarantee null termination if src length >= n)

```c
char source[] = "Hello";
char destination[10];

strcpy(destination, source);  // destination = "Hello"

// Safer: ensures null termination
strncpy(destination, source, sizeof(destination) - 1);
destination[sizeof(destination) - 1] = '\0';
```

### String Concatenation

```c
char *strcat(char *dest, const char *src);
char *strncat(char *dest, const char *src, size_t n);
```

- `strcat()` appends source to destination (must have enough space)
- `strncat()` appends up to n characters from source and always null-terminates

```c
char str1[20] = "Hello, ";
char str2[] = "World!";

strcat(str1, str2);  // str1 = "Hello, World!"

// Safer version with size limit
strncat(str1, str2, sizeof(str1) - strlen(str1) - 1);
```

## String Examination Functions

### String Searching

```c
char *strchr(const char *str, int c);
char *strrchr(const char *str, int c);
char *strstr(const char *haystack, const char *needle);
char *strpbrk(const char *str, const char *accept);
```

- `strchr()`: Find first occurrence of character c
- `strrchr()`: Find last occurrence of character c
- `strstr()`: Find first occurrence of substring
- `strpbrk()`: Find first occurrence of any character from accept

```c
char text[] = "This is a simple text example";

char *p1 = strchr(text, 'i');      // Points to first 'i' (in "This")
char *p2 = strrchr(text, 'i');     // Points to last 'i' (in "simple")
char *p3 = strstr(text, "simple"); // Points to "simple text example"
char *p4 = strpbrk(text, "aeiou"); // Points to 'i' (first vowel found)
```

### Substring Functions

```c
size_t strspn(const char *str, const char *accept);
size_t strcspn(const char *str, const char *reject);
```

- `strspn()`: Length of initial segment containing only characters from accept
- `strcspn()`: Length of initial segment containing no characters from reject

```c
char str[] = "123abc456";

size_t n1 = strspn(str, "0123456789");  // n1 = 3 (length of numeric prefix)
size_t n2 = strcspn(str, "abc");        // n2 = 3 (position of first 'a')
```

### String Tokenization

```c
char *strtok(char *str, const char *delim);
```

Breaks string into tokens based on delimiter characters.

```c
char input[] = "name,age,email,phone";
char *token;

// Get first token
token = strtok(input, ",");  // token = "name"

// Continue getting tokens
while (token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, ",");  // Get next token
}

// Note: strtok modifies the original string!
```

## String Comparison Functions

```c
int strcmp(const char *s1, const char *s2);
int strncmp(const char *s1, const char *s2, size_t n);
```

- `strcmp()`: Compare two strings lexicographically
- `strncmp()`: Compare up to n characters

Return values:
- < 0: s1 is less than s2
- = 0: s1 is equal to s2
- > 0: s1 is greater than s2

```c
char str1[] = "apple";
char str2[] = "banana";
char str3[] = "apple";

int result1 = strcmp(str1, str2);  // result1 < 0 (a comes before b)
int result2 = strcmp(str1, str3);  // result2 = 0 (strings are identical)
int result3 = strncmp("applejuice", "apple", 5);  // result3 = 0 (first 5 chars match)
```

Case-insensitive comparison (not part of C standard, but common extension):

```c
int strcasecmp(const char *s1, const char *s2);      // POSIX function
int strncasecmp(const char *s1, const char *s2, size_t n);  // POSIX function
```

On Windows, equivalents are:
```c
int _stricmp(const char *s1, const char *s2);        // Microsoft specific
int _strnicmp(const char *s1, const char *s2, size_t n);  // Microsoft specific
```

## String Manipulation Functions

### String Duplication

Not in C89/C90 standard, but common:

```c
char *strdup(const char *s);  // POSIX function (C23 standard)
```

Creates a new copy of a string, allocating memory as needed.

```c
char *original = "Hello World";
char *copy = strdup(original);  // Allocates new memory and copies string

// Don't forget to free when done
free(copy);
```

### String Formatting

```c
int sprintf(char *str, const char *format, ...);
int snprintf(char *str, size_t size, const char *format, ...);
```

- `sprintf()`: Format and store string in buffer
- `snprintf()`: Format with size limit (safer)

```c
char buffer[50];
int age = 30;
float height = 1.75;

sprintf(buffer, "Age: %d, Height: %.2f m", age, height);
// buffer now contains "Age: 30, Height: 1.75 m"

// Safer version with size limit
snprintf(buffer, sizeof(buffer), "Age: %d, Height: %.2f m", age, height);
```

## Memory Management Functions

These functions operate on memory blocks rather than null-terminated strings:

```c
void *memcpy(void *dest, const void *src, size_t n);
void *memmove(void *dest, const void *src, size_t n);
void *memset(void *s, int c, size_t n);
int memcmp(const void *s1, const void *s2, size_t n);
void *memchr(const void *s, int c, size_t n);
```

- `memcpy()`: Copy n bytes from src to dest (regions cannot overlap)
- `memmove()`: Copy n bytes from src to dest (handles overlapping regions)
- `memset()`: Fill memory with a constant byte
- `memcmp()`: Compare two memory blocks
- `memchr()`: Find a byte in a memory block

```c
char str[20] = "Hello World";

// Copy "World" to beginning, replacing "Hello"
memmove(str, str + 6, 5);
str[5] = '\0';  // Manually terminate the string
// str now contains "World"

// Set all bytes to 'A'
memset(str, 'A', 5);
// str now contains "AAAAA"
```

## Character Classification and Conversion

Found in `<ctype.h>` header:

### Character Classification

```c
int isalpha(int c);  // Is c an alphabetic character?
int isdigit(int c);  // Is c a decimal digit?
int isalnum(int c);  // Is c alphanumeric?
int isspace(int c);  // Is c a whitespace character?
int islower(int c);  // Is c a lowercase letter?
int isupper(int c);  // Is c an uppercase letter?
int ispunct(int c);  // Is c a punctuation character?
int isprint(int c);  // Is c a printable character?
int iscntrl(int c);  // Is c a control character?
```

### Character Conversion

```c
int tolower(int c);  // Convert c to lowercase
int toupper(int c);  // Convert c to uppercase
```

Example of character-by-character processing:

```c
void capitalize(char *str) {
    for (int i = 0; str[i] != '\0'; i++) {
        if (i == 0 || isspace(str[i-1])) {
            str[i] = toupper(str[i]);
        } else {
            str[i] = tolower(str[i]);
        }
    }
}

char sentence[] = "hello world";
capitalize(sentence);  // sentence becomes "Hello World"
```

## String to Number Conversions

Functions from `<stdlib.h>`:

```c
int atoi(const char *str);          // String to integer
long atol(const char *str);         // String to long
long long atoll(const char *str);   // String to long long
double atof(const char *str);       // String to double

// More robust versions that detect errors:
long strtol(const char *str, char **endptr, int base);
long long strtoll(const char *str, char **endptr, int base);
unsigned long strtoul(const char *str, char **endptr, int base);
unsigned long long strtoull(const char *str, char **endptr, int base);
double strtod(const char *str, char **endptr);
float strtof(const char *str, char **endptr);
```

Examples:

```c
char num_str[] = "123";
int num = atoi(num_str);  // num = 123

char float_str[] = "3.14159";
double pi = atof(float_str);  // pi = 3.14159

// Using the more robust versions:
char hex_str[] = "0x1A";
char *end;
long value = strtol(hex_str, &end, 0);  // value = 26 (0x1A in decimal)

char mixed[] = "100 apples";
int count = strtol(mixed, &end, 10);  // count = 100, end points to " apples"
```

## Buffer Safety and Best Practices

C string functions have earned notoriety for security vulnerabilities due to buffer overflows. Modern defensive C programming employs these practices:

### 1. Use Bounded Functions

Prefer functions with size limits:
- `strncpy()` over `strcpy()`
- `strncat()` over `strcat()`
- `snprintf()` over `sprintf()`

```c
// Unsafe
char buffer[10];
strcpy(buffer, "This string is too long");  // Buffer overflow!

// Safer
char buffer[10];
strncpy(buffer, "This string is too long", sizeof(buffer) - 1);
buffer[sizeof(buffer) - 1] = '\0';  // Ensure null termination
```

### 2. Validate String Inputs

Always check string lengths before operations:

```c
if (strlen(source) < sizeof(destination)) {
    strcpy(destination, source);  // Safe if condition is true
} else {
    // Handle the error
}
```

### 3. Consider Safer Alternatives

Several libraries offer safer string handling:

- POSIX `strlcpy()` and `strlcat()`
- Microsoft's Secure CRT functions (`strcpy_s()`, `strcat_s()`, etc.)
- Third-party string libraries with bounds checking

### 4. Check Return Values

Many string functions return important status information:

```c
if (snprintf(buffer, sizeof(buffer), "%s", input) >= sizeof(buffer)) {
    // Output was truncated
    // Handle appropriately
}
```

## Common String Algorithms

### String Reversal

```c
void reverse_string(char *str) {
    size_t len = strlen(str);
    for (size_t i = 0; i < len / 2; i++) {
        char temp = str[i];
        str[i] = str[len - i - 1];
        str[len - i - 1] = temp;
    }
}
```

### Case Conversion

```c
void to_uppercase(char *str) {
    for (int i = 0; str[i] != '\0'; i++) {
        str[i] = toupper(str[i]);
    }
}

void to_lowercase(char *str) {
    for (int i = 0; str[i] != '\0'; i++) {
        str[i] = tolower(str[i]);
    }
}
```

### Trim Whitespace

```c
char *trim(char *str) {
    char *end;
    
    // Trim leading space
    while (isspace((unsigned char)*str)) str++;
    
    if (*str == 0)  // All spaces?
        return str;
    
    // Trim trailing space
    end = str + strlen(str) - 1;
    while (end > str && isspace((unsigned char)*end)) end--;
    
    // Write new null terminator
    *(end + 1) = '\0';
    
    return str;
}
```

### String Splitting

```c
char **split_string(char *str, const char *delim, int *count) {
    char *copy = strdup(str);  // Create a copy to avoid modifying original
    char *token;
    int num_tokens = 0;
    
    // Count tokens
    char *temp = strdup(copy);
    token = strtok(temp, delim);
    while (token != NULL) {
        num_tokens++;
        token = strtok(NULL, delim);
    }
    free(temp);
    
    // Allocate array for pointers
    char **result = (char**)malloc((num_tokens + 1) * sizeof(char*));
    
    // Get tokens and store in array
    token = strtok(copy, delim);
    int i = 0;
    while (token != NULL) {
        result[i] = strdup(token);
        i++;
        token = strtok(NULL, delim);
    }
    result[i] = NULL;  // Null-terminate the array
    
    free(copy);
    *count = num_tokens;
    return result;
}

// Don't forget to free the memory when done:
void free_string_array(char **arr) {
    for (int i = 0; arr[i] != NULL; i++) {
        free(arr[i]);
    }
    free(arr);
}
```

## Advanced Topics

### String Interning

String interning is a technique to store only one copy of each distinct string value:

```c
#include <string.h>
#include <stdlib.h>

typedef struct {
    char **strings;
    size_t count;
    size_t capacity;
} StringPool;

StringPool *create_string_pool(size_t initial_capacity) {
    StringPool *pool = malloc(sizeof(StringPool));
    pool->strings = malloc(initial_capacity * sizeof(char*));
    pool->count = 0;
    pool->capacity = initial_capacity;
    return pool;
}

const char *intern_string(StringPool *pool, const char *str) {
    // Check if string already exists in pool
    for (size_t i = 0; i < pool->count; i++) {
        if (strcmp(pool->strings[i], str) == 0) {
            return pool->strings[i];
        }
    }
    
    // If not found, add to pool
    if (pool->count >= pool->capacity) {
        pool->capacity *= 2;
        pool->strings = realloc(pool->strings, pool->capacity * sizeof(char*));
    }
    
    pool->strings[pool->count] = strdup(str);
    return pool->strings[pool->count++];
}

void destroy_string_pool(StringPool *pool) {
    for (size_t i = 0; i < pool->count; i++) {
        free(pool->strings[i]);
    }
    free(pool->strings);
    free(pool);
}
```

### Working with Unicode/Wide Strings

For Unicode support, C provides wide character functions in `<wchar.h>`:

```c
#include <wchar.h>

wchar_t *wide_str = L"Wide character string";
size_t len = wcslen(wide_str);           // Length of wide string
wchar_t *copy = (wchar_t*)malloc((len + 1) * sizeof(wchar_t));
wcscpy(copy, wide_str);                  // Copy wide string
```

Key wide character functions:
- `wcslen()`, `wcscpy()`, `wcsncpy()`, `wcscat()`, `wcsncat()`
- `wcscmp()`, `wcsncmp()`, `wcscoll()`
- `wcschr()`, `wcsrchr()`, `wcsstr()`
- `wcstok()`

### String Views (C23)

The upcoming C23 standard introduces string views for non-owning references to strings:

```c
#include <string_view.h>

void process_string_view(string_view sv) {
    // Work with string view without copying the string
}

// Usage
char *str = "Hello World";
string_view sv = string_view_from_cstr(str);
process_string_view(sv);
```

### Custom String Libraries

For performance-critical applications, consider specialized string libraries:

- Small String Optimization (SSO): Store short strings directly in the string object
- Ropes: Represent strings as binary trees for efficient concatenation/splitting
- Immutable strings: Improve safety by preventing modification

```c
// Example of a simple SSO string implementation
typedef struct {
    union {
        struct {
            char *ptr;
            size_t len;
            size_t capacity;
        } large;
        struct {
            char data[24];  // Inline buffer for small strings
            uint8_t len;
        } small;
    } u;
    uint8_t is_small;
} SSOString;
```