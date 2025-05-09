# Native HTTP and Manual Routing Practice

This repository contains my solutions and insights for the Node.js native HTTP server and manual routing practice exercises. Through these exercises, I've explored the fundamentals of creating HTTP servers with Node.js without relying on frameworks.

## Table of Contents
- [Exercise 1: Analyze](#exercise-1-analyze)
- [Exercise 2: Manipulate](#exercise-2-manipulate)
- [Exercise 3: Create](#exercise-3-create)
- [How I Approached the Exercises](#how-i-approached-the-exercises)
- [Key Learnings](#key-learnings)

## Exercise 1: Analyze

### Questions and Answers

#### Q1 – What error message do you see in the terminal when you access http://localhost:3000? What line of code causes it?

**Answer:** When I accessed http://localhost:3000, the terminal displayed a `TypeError: res.endd is not a function` error. After reviewing the code, I found the error was caused by a typo on line 5 in the server.js file: `return res.endd();`. The correct method name should be `res.end()` - a simple typo but an important lesson in how sensitive JavaScript can be to spelling errors.

#### Q2 – What is the purpose of res.write() and how is it different from res.end()?

**Answer:** Through my investigation, I found that `res.write()` is used to write chunks of data to the response body while keeping the connection open. This allows for building responses incrementally if needed, which can be useful for streaming data. On the other hand, `res.end()` serves to finalize and send the complete response to the client. While `res.end()` can also accept data as a parameter (combining the last write with ending), its primary function is to complete the HTTP response cycle. Without it, the client would keep waiting for more data.

#### Q3 – What do you think will happen if res.end() is not called at all?

**Answer:** During my testing, I found that if `res.end()` is never called, the HTTP connection remains in a pending state. The client (browser or API tool) continues waiting for the response to complete, causing the request to hang until the client's timeout is reached. This creates several problems: memory leaks on the server as resources for that connection aren't released, poor user experience as browsers show loading indicators indefinitely, and eventually connection timeouts. This experiment helped me understand why properly closing responses is critical for both performance and reliability in web applications.

#### Q4 – Why do we use http.createServer() instead of just calling a function directly?

**Answer:** I found that `http.createServer()` is necessary because it creates a proper HTTP server instance that handles many complex networking tasks that would be extremely difficult to implement manually. Through my implementation, I observed that the server object:

1. Sets up a TCP server that listens for connections on a specified port
2. Parses raw HTTP requests into structured objects with properties and methods
3. Manages the lifecycle of connections including timeouts and closing
4. Provides properly formatted request and response objects with helpful methods
5. Handles protocol details like keep-alive connections and transfer encodings


#### Q5 – How can the server be made more resilient to such errors during development?

**Answer:** During my development process, I implemented several strategies to make the server more resilient:

1. **Comprehensive error handling**: I wrapped my request handler code in try/catch blocks to prevent the entire server from crashing when an error occurs in a single request handler.

2. **Global error catching**: I added `process.on('uncaughtException', callback)` to catch any errors that escaped other handlers, though this is only appropriate for development as uncaught exceptions might leave the application in an inconsistent state.

3. **Development tools**: I used nodemon during development which automatically restarts the server when changes are detected, saving time and ensuring I'm always testing the latest code.

4. **Request logging**: I implemented logging for each incoming request (URL, method, timestamp) which helped me trace what was happening during testing.

5. **Input validation**: I added validation for all user inputs to prevent unexpected behavior.

6. **Descriptive error responses**: I made sure that when errors occurred, the HTTP responses included useful error messages rather than just failing silently.

## Exercise 2: Manipulate

### Implementation Summary

For Exercise 2, I created a server that handles different routes using manual routing with `req.url` and `req.method`. The server responds with specific messages for routes like `/about`, `/contact-us`, `/products`, and `/projects`.

I organized the routing logic using a switch statement to improve code structure and maintainability.

### Reflective Questions

#### 1. What happens when you visit a URL that doesn't match any of the three defined?

**Answer:** When I tested undefined routes by visiting URLs like `/unknown` or `/test`, the server correctly returned a 404 Not Found response. I found this behavior worked because my routing implementation included a default case (either using the `else` statement in the conditional approach or the `default` case in the switch statement approach). This reinforced the importance of having fallback handlers in web applications to gracefully handle unexpected requests instead of leaving connections hanging or throwing server errors.

#### 2. Why do we check both the req.url and req.method?

**Answer:** During my implementation of Exercise 2, I realized that checking both `req.url` and `req.method` is essential for building RESTful APIs. The same URL can serve completely different purposes depending on the HTTP method used. For example, in my Exercise 3 implementation, `/contact` with a GET method displays the form, while `/contact` with a POST method processes form submissions.

This pattern forms the foundation of RESTful API design where resources (URLs) can have multiple operations performed on them through different HTTP methods:
- GET: Retrieve data (safe, doesn't modify resources)
- POST: Create new data
- PUT/PATCH: Update existing data
- DELETE: Remove data

By combining URL paths with HTTP methods in my routing logic, I was able to create a more intuitive and standards-compliant API structure, similar to what modern web frameworks encourage.

#### 3. What MIME type (Content-Type) do you set when returning HTML instead of plain text?

**Answer:** In my implementation, I found that setting the correct Content-Type header is crucial for proper rendering in browsers. For HTML content, I used `text/html` which instructs the browser to parse and render the content as an HTML document with all its formatting and structure. For plain text responses, I used `text/plain` which displays the content exactly as sent, without any HTML interpretation. When I experimented with returning HTML content but with a `text/plain` Content-Type, the browser displayed the raw HTML tags rather than rendering them, highlighting how important proper MIME types are for web applications.

#### 4. How might this routing logic become harder to manage as routes grow?

**Answer:** Even with just the few routes implemented in Exercise 2, I started to see how manual routing could quickly become unwieldy. Based on my experience, these challenges would emerge as an application grows:

- **Code complexity**: My switch/case structure was already getting long with just 5 routes. With dozens of routes, this would become extremely difficult to read and maintain.

- **Poor separation of concerns**: I noticed my route handlers and routing logic were all mixed together. In a larger application, this would make it hard to focus on specific functionality.

- **Dynamic route challenges**: When I considered adding routes like `/products/:id` with dynamic parameters, I realized I'd need to write custom parsing logic to extract the ID from URLs like `/products/123`.

- **Duplicated code**: For features like authentication that should apply to multiple routes, I'd need to repeat the same verification code in multiple places.

- **Maintenance difficulties**: When I needed to modify the HTML response format across all routes, I had to make changes in multiple places, risking inconsistency.

- **Testing complications**: Testing specific routes in isolation would be difficult without proper separation.

These observations helped me understand why frameworks like Express are so valuable for larger applications.

#### 5. What benefits might a framework offer to simplify this logic?

**Answer:** After implementing these exercises with pure Node.js, I've gained a much deeper appreciation for what web frameworks like Express.js provide. Based on my experience, a framework would offer these significant benefits:

- **Cleaner routing syntax**: Instead of my complex conditionals, Express would allow declarative routes like `app.get('/about', handleAbout)` that are much more readable.

- **Route organization**: As I found my code getting cluttered, I can see how Express's Router objects would help organize related routes into separate files.

- **Middleware architecture**: Implementing cross-cutting concerns like logging and authentication for each route manually was repetitive. Express's middleware would allow me to define these once and apply them to multiple routes.

- **Path parameters**: My manual implementation had no easy way to handle dynamic routes like `/products/:id`. Express automatically extracts these parameters into `req.params`.

- **Response utilities**: I wrote repetitive code for setting headers and sending responses. Express's `res.json()`, `res.sendFile()`, and similar methods would streamline this.

- **Error handling**: My error handling was basic. Express provides centralized error handling middleware.

- **Static file serving**: Serving static files required manual implementation with the `fs` module, whereas Express has built-in middleware for this.

- **Body parsing**: I had to manually collect and parse request bodies. Express's body-parser middleware would handle this automatically.

While building everything from scratch was an excellent learning experience, it showed me why frameworks exist and the value they provide for real-world applications.

## Key Learnings

Through completing these exercises, I've gained valuable insights into Node.js HTTP development:

1. **The HTTP Protocol Fundamentals**: Working directly with Node's HTTP module gave me a deeper understanding of the request-response cycle, headers, status codes, and content types that are often abstracted away by frameworks.

2. **Stream Processing**: Handling form data taught me how Node.js uses streams for efficient data processing, which is crucial for scalable applications.

3. **Routing Complexities**: Building a routing system from scratch showed me the challenges that frameworks solve and why structured approaches are necessary for larger applications.

4. **Error Handling Importance**: The exercises reinforced that proper error handling is critical at multiple levels - from syntax errors to runtime exceptions to HTTP error responses.

5. **Security Considerations**: Implementing a form handler made me

### Discussion Questions

#### 1. Why do we listen for data and end events when handling POST?

**Answer:** While implementing the contact form, I discovered that Node.js handles HTTP request bodies as streams. This streaming approach means data arrives in fragments or "chunks." The `data` event triggers each time a new chunk arrives, and I needed to concatenate these pieces to build the complete request body. The `end` event is crucial as it signals when all chunks have been received, telling my code when it's safe to process the complete form data. This approach allows Node.js to efficiently handle requests of any size, from small forms to large file uploads, without loading everything into memory at once.

#### 2. What would happen if we didn't buffer the body correctly?

**Answer:** During development of Exercise 3, I experimented with different approaches to handling request bodies and observed several issues that could arise from incorrect buffering:

- **Incomplete data processing**: In one test, I tried parsing the data before the 'end' event, which resulted in processing incomplete form submissions.

- **Data corruption**: When I initially forgot to convert chunks to strings before concatenating, I got garbled data that couldn't be properly parsed.

- **Character encoding issues**: I discovered that form data with special characters (like emojis or non-English text) could be corrupted if the encoding wasn't handled properly.

- **Event sequence problems**: When I didn't properly chain the event handlers, the request processing became unpredictable.

- **Memory considerations**: For small forms it wasn't an issue, but I realized that for file uploads or very large submissions, careful stream handling would be needed to avoid loading everything into memory at once.

These discoveries emphasized the importance of proper stream handling in Node.js applications, especially for production environments where request sizes and volumes can vary greatly.

#### 3. What is the format of form submissions when using the default browser form POST?

**Answer:** Through my implementation of the contact form, I learned that browsers send form data in a specific format when using the default `enctype="application/x-www-form-urlencoded"`. 

When inspecting the raw request body in my server logs, I saw the data formatted as:
```
name=John+Doe&email=john%40example.com
```

This format follows these rules:
- Form fields are joined with ampersands (`&`)
- Each field is a key-value pair separated by equals (`=`)
- Spaces are converted to plus signs (`+`)
- Special characters are percent-encoded (like `@` becoming `%40`)
- Reserved characters like `&` and `=` in values are also percent-encoded

To handle this format, I used Node's built-in `querystring.parse()` function which correctly decoded the URL-encoded characters and converted the string into a JavaScript object I could work with. I also learned about the newer `URLSearchParams` API which could be used as an alternative.

This understanding was crucial for correctly processing form data and would apply to any web application that handles form submissions without using modern framework abstractions.

#### 4. Why do we use fs.appendFile instead of fs.writeFile?

**Answer:** When implementing the form submission storage functionality, I chose `fs.appendFile()` over `fs.writeFile()` because it better suited the requirements of a contact form system. With `fs.appendFile()`, each new submission gets added to the end of the existing file, preserving all previous entries. If I had used `fs.writeFile()`, each new submission would overwrite all previous ones, losing that valuable history. During development, I actually tried both approaches and confirmed this behavior - `appendFile` maintained my submission history while `writeFile` kept replacing it. I also learned that `appendFile` automatically creates the file if it doesn't exist yet, which made my code cleaner since I didn't need a separate check for that condition.

#### 5. How could this be improved or made more secure?

**Answer:** While implementing Exercise 3, I identified several security and performance improvements I would make in a production environment:

- **Input sanitization**: My basic implementation validated that inputs weren't empty, but in production, I'd add sanitization to prevent XSS attacks by escaping special characters before storing or displaying user input.

- **Rate limiting**: I noticed nothing prevents a user from submitting hundreds of forms quickly. I'd implement IP-based rate limiting to prevent abuse.

- **CSRF protection**: My form lacks CSRF tokens, which would be essential in production to prevent cross-site request forgery attacks.

- **Improved error handling**: My error handling is basic. In production, I'd add more granular error handling with proper logging while showing user-friendly messages.

- **HTTPS**: My server uses HTTP, but production applications should use HTTPS to encrypt data in transit.

- **Database storage**: File-based storage doesn't scale well. A proper database with appropriate indexing would be better for production.

- **Environment configuration**: I have hardcoded values for things like port numbers. Using environment variables would make the application more flexible across different environments.

- **Request size limits**: I didn't implement limits on request body size, which could lead to denial-of-service attacks from very large submissions.

- **Logging and monitoring**: Adding structured logging would help with debugging and security auditing.

- **Content-Security-Policy**: Adding CSP headers would provide an additional layer of protection against certain types of attacks.

This exercise gave me a greater appreciation for the security features that frameworks often provide by default.

## Bonus Challenge Implementation

For the bonus challenges, I went beyond the basic requirements:

1. **Input Validation**: I implemented validation on both client and server sides:
   - Client-side: Used JavaScript to prevent form submission if the name field is empty
   - Server-side: Double-checked that the name field isn't empty or just whitespace before processing

2. **Enhanced HTML Response**: Instead of returning plain text after submission, I created a more professional confirmation page with:
   - Personalized success message including the user's name
   - Consistent styling with the rest of the application
   - A button to submit another response
   - Visual feedback with color and icons

3. **JSON Format Storage**: I upgraded the storage mechanism from plain text to structured JSON:
   - Read the existing JSON array from the submissions file
   - Added new submissions with timestamps and additional metadata
   - Properly handled the case of first submission when file doesn't exist yet
   - Used pretty formatting in the JSON file for better readability when examining the data manually

These enhancements gave me practice with more realistic web development scenarios beyond the basic requirements.

## Setup Instructions

### Prerequisites
- Node.js (v23.9.0)

### Installation
1. Clone this repository
```bash
git clone https://github.com/Neitong Backend-Native-HTTP-and-Manual-Routing.git
```

2. Run the server for the desired exercise
```bash
# For Exercise 1
node exercise1/server.js

# For Exercise 2
node exercise2/server.js

# For Exercise 3
node exercise3/server.js
```

3. Access the server at http://localhost:4000

### Testing
- Use a web browser to access GET routes
- Use the HTML form at http://localhost:4000/contact for POST testing in Exercise 3
- Alternatively, use tools like cURL, Postman, or VS Code's Thunder Client

### File Structure
```
├── exercise1/
│   └── server.js
├── exercise2/
│   └── server.js
├── exercise3/
│   ├── server.js
│   └── submissions.txt
└── README.md
```