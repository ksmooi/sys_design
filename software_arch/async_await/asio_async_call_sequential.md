# Sequential Asynchronous Function Calls with Asio's `io_context`: Chaining, Coroutines, and Futures

Asynchronous programming is a fundamental paradigm in modern software development, enabling applications to perform multiple operations concurrently without blocking the main execution thread. In C++, [Asio](https://think-async.com/Asio/) (available as a standalone library or as part of Boost) is a widely-adopted library that facilitates asynchronous I/O operations. This article explores three primary methods for calling multiple asynchronous functions sequentially using Asio's `io_context`: **Chaining**, **Coroutines**, and **Futures**. Through detailed explanations and illustrative code examples, we delve into how each method facilitates efficient and maintainable asynchronous workflows.

## Table of Contents

1. [Introduction](#introduction)
2. [Asynchronous Programming with Asio](#asynchronous-programming-with-asio)
3. [Method 1: Chaining Asynchronous Functions](#method-1-chaining-asynchronous-functions)
    - [Understanding Chaining](#understanding-chaining)
    - [Chaining with Asio Handlers](#chaining-with-asio-handlers)
    - [Code Example](#code-example)
4. [Method 2: Using Coroutines for Sequential Async Calls](#method-2-using-coroutines-for-sequential-async-calls)
    - [Introduction to Coroutines in Asio](#introduction-to-coroutines-in-asio)
    - [Implementing Sequential Async Calls with Coroutines](#implementing-sequential-async-calls-with-coroutines)
    - [Code Example](#code-example-1)
5. [Method 3: Utilizing `std::future` for Sequential Operations](#method-3-utilizing-stdfuture-for-sequential-operations)
    - [Understanding `std::future` with Asio](#understanding-stdfuture-with-asio)
    - [Sequential Execution with `std::future`](#sequential-execution-with-stdfuture)
    - [Code Example](#code-example-2)
6. [Comparative Analysis](#comparative-analysis)
7. [Best Practices](#best-practices)
8. [Conclusion](#conclusion)
9. [References](#references)

---

## Introduction

In high-performance and responsive applications, the ability to execute multiple tasks asynchronously is paramount. Whether it's handling file I/O operations, managing network requests, or performing compute-intensive tasks, asynchronous programming allows programs to manage these operations efficiently without stalling the main thread.

C++ offers several mechanisms for asynchronous programming, with Asio being a prominent library that provides a consistent asynchronous model using `io_context`. With the introduction of C++20 coroutines, Asio has further enhanced its capabilities, allowing developers to write asynchronous code that is both efficient and readable.

This article explores three methods to call multiple asynchronous functions sequentially using Asio's `io_context`:

1. **Chaining**: Leveraging handlers to initiate subsequent asynchronous operations upon completion.
2. **Coroutines**: Utilizing C++20 coroutines to write sequential-looking asynchronous code.
3. **Futures**: Employing `std::future` to manage and synchronize multiple asynchronous tasks.

Each method is dissected with comprehensive explanations and accompanied by practical code examples to demonstrate their implementation and use cases.

---

## Asynchronous Programming with Asio

[Asio](https://think-async.com/Asio/) is a cross-platform C++ library for network and low-level I/O programming that provides a consistent asynchronous model using [`io_context`](https://think-async.com/Asio/asio-1.18.2/doc/asio/reference/io_context.html). It supports various asynchronous operations, allowing developers to handle tasks like reading from or writing to sockets, files, and more without blocking the main execution thread.

**Key Components:**

- **`io_context`**: The core I/O event dispatcher that coordinates asynchronous operations.
- **Asynchronous Operations**: Functions prefixed with `async_` (e.g., `async_read`, `async_write`) that initiate asynchronous tasks and require handlers to process completion events.
- **Handlers**: Callback functions invoked upon the completion of asynchronous operations.

With the advent of C++20, Asio has incorporated coroutine support, enabling developers to write asynchronous code using the coroutine syntax, which simplifies complex asynchronous operations.

---

## Method 1: Chaining Asynchronous Functions

### Understanding Chaining

**Chaining** refers to the pattern where multiple asynchronous functions are executed in a sequence, with each function initiating the next upon its completion. This approach ensures that tasks are executed in a specific order without blocking the main thread. Chaining is particularly useful when subsequent operations depend on the results of preceding ones.

In Asio, chaining can be implemented by having each asynchronous function's handler initiate the next asynchronous operation. This method maintains a clear and predictable execution flow, albeit with potential complexity as the number of chained functions increases.

### Chaining with Asio Handlers

Using Asio handlers, developers can chain asynchronous functions by defining a sequence of callbacks where each callback initiates the next asynchronous operation. This approach ensures that each task begins only after the previous one has completed, maintaining the desired order of execution.

**Chaining Pattern:**

```cpp
void initialize(asio::io_context& io) {
    // Perform initialization actions
    // ...
    // Upon completion, post the next handler
    io.post([&io]() { process(io); });
}

void process(asio::io_context& io) {
    // Perform processing actions
    // ...
    // Upon completion, post the next handler
    io.post([&io]() { finalize(io); });
}

void finalize(asio::io_context& io) {
    // Perform finalization actions
    // ...
    // Optionally, notify the caller or perform cleanup
    std::cout << "All asynchronous operations completed successfully.\n";
}
```

**Explanation:**

1. **`initialize` Function:**
    - Performs initialization tasks.
    - Upon completion, it posts the `process` function to the `io_context`, initiating the next step.

2. **`process` Function:**
    - Performs processing tasks.
    - Upon completion, it posts the `finalize` function to the `io_context`, initiating the final step.

3. **`finalize` Function:**
    - Performs finalization tasks.
    - Optionally, it can notify the caller or perform any necessary cleanup.

This pattern ensures that each asynchronous operation is executed only after the preceding one has completed, maintaining a clear and ordered workflow.

### Code Example

Below is a comprehensive C++ program that demonstrates chaining asynchronous functions using Asio's `io_context` and handlers. The program simulates a sequence of operations: initializing data, processing it, and finalizing the process.

```cpp
#include <iostream>
#include <asio.hpp>
#include <thread>
#include <chrono>
#include <string>

// Function declarations
void initialize(asio::io_context& io);
void process(asio::io_context& io);
void finalize(asio::io_context& io);

// Simulated asynchronous initialization function
void initialize(asio::io_context& io) {
    // Simulate asynchronous work using a separate thread
    std::thread([&io]() {
        std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
        std::cout << "Initialization complete.\n";
        // Upon completion, post the next handler
        io.post([&io]() { process(io); });
    }).detach();
}

// Simulated asynchronous processing function
void process(asio::io_context& io) {
    // Simulate asynchronous work using a separate thread
    std::thread([&io]() {
        std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
        std::cout << "Processing complete.\n";
        // Upon completion, post the next handler
        io.post([&io]() { finalize(io); });
    }).detach();
}

// Simulated asynchronous finalization function
void finalize(asio::io_context& io) {
    // Simulate asynchronous work using a separate thread
    std::thread([&io]() {
        std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
        std::cout << "Finalization complete. All operations done.\n";
        // Optionally, stop the io_context if no more tasks are pending
        io.stop();
    }).detach();
}

int main() {
    asio::io_context io;

    // Start the asynchronous chain with initialization
    initialize(io);

    // Run the io_context to process asynchronous events
    io.run();

    std::cout << "Program terminated gracefully.\n";

    return 0;
}
```

**Explanation:**

1. **Function Definitions:**
    - **`initialize`**: Simulates an initialization task by sleeping for 500ms. After completion, it posts the `process` function to the `io_context`.
    - **`process`**: Simulates a processing task by sleeping for 500ms. After completion, it posts the `finalize` function to the `io_context`.
    - **`finalize`**: Simulates a finalization task by sleeping for 500ms. After completion, it prints a confirmation message and stops the `io_context`.

2. **Main Function:**
    - **`asio::io_context io;`**: Creates an instance of `io_context` which manages asynchronous operations.
    - **`initialize(io);`**: Initiates the asynchronous chain by calling the `initialize` function.
    - **`io.run();`**: Runs the `io_context` to process all posted asynchronous events. The program will continue running until `io.stop()` is called.
    - **Program Termination**: After all asynchronous tasks complete and `io_context` is stopped, the program prints a termination message and exits gracefully.

**Output:**

```
Initialization complete.
Processing complete.
Finalization complete. All operations done.
Program terminated gracefully.
```

**Advantages:**

- **Simplicity**: The chaining pattern is straightforward for simple sequences of dependent tasks.
- **Deterministic Execution**: Ensures that tasks execute in a specific order, maintaining the desired workflow.

**Disadvantages:**

- **Callback Nesting**: As the number of chained tasks increases, the code can become deeply nested and harder to maintain, leading to what's often referred to as "callback hell."
- **Thread Overhead**: Launching a new thread for each asynchronous operation can lead to performance overhead, especially with a large number of tasks.

---

## Method 2: Using Coroutines for Sequential Async Calls

### Introduction to Coroutines in Asio

Coroutines, introduced in C++20, offer a powerful abstraction for writing asynchronous code that is both efficient and readable. They allow functions to suspend (`co_await`) and resume their execution, enabling developers to write asynchronous workflows that resemble synchronous code. This approach enhances clarity and maintainability by eliminating the need for explicit chaining and callback structures.

Asio has integrated coroutine support, allowing developers to write asynchronous code using the coroutine syntax, which simplifies complex asynchronous operations.

**Key Components:**

- **`asio::awaitable`**: Represents an awaitable coroutine task.
- **`asio::co_spawn`**: Launches a coroutine within the `io_context`.
- **`co_await`**: Suspends the coroutine until the awaited operation completes.

### Implementing Sequential Async Calls with Coroutines

To utilize coroutines in Asio, developers can leverage the `asio::awaitable` type along with the `co_await` keyword. This integration allows for writing asynchronous code that executes sequentially without the nested callbacks associated with chaining.

**Coroutine Pattern:**

```cpp
// Coroutine function performing sequential async operations
asio::awaitable<void> run_async_tasks(asio::io_context& io) {
    // Initialize
    co_await asio::post(io, []() { initialize(io); });
    
    // Process
    co_await asio::post(io, []() { process(io); });
    
    // Finalize
    co_await asio::post(io, []() { finalize(io); });
    
    co_return;
}
```

**Explanation:**

1. **Initialization**: The coroutine awaits the completion of the `initialize` function.
2. **Processing**: After initialization, it awaits the `process` function.
3. **Finalization**: Finally, it awaits the `finalize` function.
4. **Completion**: Once all operations are complete, the coroutine returns, signaling the end of the asynchronous workflow.

This pattern ensures that each asynchronous operation completes before the next one begins, maintaining a clear and ordered execution flow without deep callback nesting.

### Code Example

The following C++ program demonstrates how to use coroutines with Asio to call multiple asynchronous functions sequentially. The example simulates a sequence of operations: initializing data, processing it, and finalizing the process.

```cpp
#include <iostream>
#include <asio.hpp>
#include <thread>
#include <chrono>
#include <string>
#include <functional>

// Alias for convenience
using asio::awaitable;
using asio::co_spawn;
using asio::detached;
using asio::use_awaitable;

// Simulated asynchronous initialization function
awaitable<void> initialize_async(asio::io_context& io) {
    // Simulate asynchronous work
    co_await asio::post(io, use_awaitable); // Yield to allow other operations
    std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
    std::cout << "Initialization complete.\n";
    co_return;
}

// Simulated asynchronous processing function
awaitable<void> process_async(asio::io_context& io) {
    // Simulate asynchronous work
    co_await asio::post(io, use_awaitable); // Yield to allow other operations
    std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
    std::cout << "Processing complete.\n";
    co_return;
}

// Simulated asynchronous finalization function
awaitable<void> finalize_async(asio::io_context& io) {
    // Simulate asynchronous work
    co_await asio::post(io, use_awaitable); // Yield to allow other operations
    std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
    std::cout << "Finalization complete. All operations done.\n";
    co_return;
}

// Coroutine function using co_await to perform sequential async calls
awaitable<void> run_async_tasks(asio::io_context& io) {
    // Initialize
    co_await initialize_async(io);
    
    // Process
    co_await process_async(io);
    
    // Finalize
    co_await finalize_async(io);
    
    co_return;
}

int main() {
    asio::io_context io;

    // Launch the coroutine using co_spawn
    co_spawn(io, run_async_tasks(io), detached);

    // Run the io_context to process asynchronous events
    io.run();

    std::cout << "Program terminated gracefully.\n";

    return 0;
}
```

**Explanation:**

1. **Asynchronous Functions:**
    - **`initialize_async`**: Simulates an initialization task by sleeping for 500ms. It uses `co_await asio::post(io, use_awaitable);` to yield control back to the `io_context`, allowing other operations to run.
    - **`process_async`**: Simulates a processing task by sleeping for 500ms. Similarly, it yields control before performing its task.
    - **`finalize_async`**: Simulates a finalization task by sleeping for 500ms. After completion, it prints a confirmation message.

2. **Coroutine Function (`run_async_tasks`):**
    - Sequentially awaits each asynchronous operation using `co_await`, ensuring that each task completes before the next one begins.
    - This linear flow enhances readability and maintainability, eliminating the need for nested callbacks.

3. **Launching the Coroutine:**
    - **`co_spawn`**: Launches the `run_async_tasks` coroutine within the `io_context`, using the `detached` completion token to indicate that the coroutine should run independently.
    - The `io_context` manages the execution and resumption of the coroutine as asynchronous tasks complete.

4. **Asio's `io_context`:**
    - **`io.run();`**: Runs the `io_context` to process all asynchronous events. The program continues running until all posted tasks and coroutines have completed.

5. **Program Termination:**
    - After all asynchronous tasks complete and the `io_context` is stopped, the program prints a termination message and exits gracefully.

**Output:**

```
Initialization complete.
Processing complete.
Finalization complete. All operations done.
Program terminated gracefully.
```

**Advantages:**

- **Readability**: Coroutines allow writing asynchronous code that resembles synchronous code, enhancing clarity.
- **Maintainability**: Eliminates the need for explicit chaining and callback structures, reducing code complexity.
- **Flexibility**: Supports more complex asynchronous workflows with easy suspension and resumption.

**Disadvantages:**

- **Complexity**: Understanding coroutine promise types and handles can be intricate, especially for developers new to coroutines.
- **C++20 Requirement**: Requires a C++20-compliant compiler, limiting compatibility with older systems.

---

## Method 3: Utilizing `std::future` for Sequential Operations

### Understanding `std::future` with Asio

[`std::future`](https://en.cppreference.com/w/cpp/thread/future) is a powerful feature in C++ that represents a value that will be available at some point in the future. It allows developers to retrieve results from asynchronous operations and synchronize threads efficiently. By combining `std::future` with Asio's asynchronous mechanisms, developers can manage the execution and synchronization of multiple asynchronous tasks seamlessly.

Asio provides the `use_future` completion token, which allows asynchronous operations to return a `std::future`, facilitating easier integration with `std::future` and `std::async`.

### Sequential Execution with `std::future`

Sequential execution using `std::future` involves launching asynchronous tasks and waiting for their completion before proceeding to the next task. This ensures that tasks are executed in a specific order, with each task potentially depending on the results of the previous ones.

Unlike chaining, which relies on function calls to initiate subsequent tasks, using `std::future` allows for more explicit synchronization and control over task execution order.

### Code Example

The following C++ program illustrates how to use `std::future` with Asio to call multiple asynchronous functions sequentially. The example emphasizes managing dependencies between tasks using futures.

```cpp
#include <iostream>
#include <asio.hpp>
#include <thread>
#include <chrono>
#include <string>
#include <future>

// Simulated asynchronous fetch function using Asio
std::future<int> fetch_data_async(asio::io_context& io) {
    std::promise<int> prom;
    auto fut = prom.get_future();

    // Post the asynchronous task to io_context
    asio::post(io, [prom = std::move(prom)]() mutable {
        std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
        std::cout << "Data fetched.\n";
        prom.set_value(42); // Example data
    });

    return fut;
}

// Simulated asynchronous compute function using Asio
std::future<int> compute_async(asio::io_context& io, int data) {
    std::promise<int> prom;
    auto fut = prom.get_future();

    // Post the asynchronous task to io_context
    asio::post(io, [prom = std::move(prom), data]() mutable {
        std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
        std::cout << "Computation complete.\n";
        prom.set_value(data * 2); // Example computation
    });

    return fut;
}

// Simulated asynchronous save function using Asio
std::future<void> save_async(asio::io_context& io, int result) {
    std::promise<void> prom;
    auto fut = prom.get_future();

    // Post the asynchronous task to io_context
    asio::post(io, [prom = std::move(prom), result]() mutable {
        std::this_thread::sleep_for(std::chrono::milliseconds(500)); // Simulate delay
        std::cout << "Result saved: " << result << "\n";
        prom.set_value(); // Indicate completion
    });

    return fut;
}

int main() {
    asio::io_context io;

    // Run the io_context in a separate thread to handle asynchronous tasks
    std::thread io_thread([&io]() {
        io.run();
    });

    // Launch fetch_data asynchronously
    std::future<int> fetch_future = fetch_data_async(io);

    // Wait for fetch_data to complete and get the result
    int fetched_data = fetch_future.get();

    // Launch compute asynchronously, dependent on fetched_data
    std::future<int> compute_future = compute_async(io, fetched_data);

    // Wait for compute to complete and get the result
    int computed_result = compute_future.get();

    // Launch save asynchronously, dependent on computed_result
    std::future<void> save_future = save_async(io, computed_result);

    // Wait for save to complete
    save_future.get();

    std::cout << "All asynchronous operations completed successfully.\n";

    // Stop the io_context and join the thread
    io.stop();
    io_thread.join();

    return 0;
}
```

**Explanation:**

1. **Asynchronous Functions:**
    - **`fetch_data_async`**: Returns a `std::future<int>`. It creates a `std::promise<int>`, posts an asynchronous task to the `io_context` that sleeps for 500ms, prints a message, and sets the promise's value to `42`.
    - **`compute_async`**: Takes the fetched data (`int`), returns a `std::future<int>`, and performs a similar asynchronous operation, multiplying the data by `2`.
    - **`save_async`**: Takes the computed result (`int`), returns a `std::future<void>`, sleeps for 500ms, prints the result, and sets the promise to indicate completion.

2. **Launching and Synchronizing Tasks:**
    - **Fetch Data**: The main thread calls `fetch_data_async`, obtaining a `std::future<int>`. It then calls `.get()` to wait for the fetch operation to complete and retrieve the fetched data.
    - **Compute**: Using the fetched data, the main thread calls `compute_async`, obtaining a `std::future<int>`. It calls `.get()` to wait for the computation to complete and retrieve the computed result.
    - **Save**: Using the computed result, the main thread calls `save_async`, obtaining a `std::future<void>`. It calls `.get()` to wait for the save operation to complete.

3. **Asio's `io_context`:**
    - The `io_context` runs in a separate thread (`io_thread`), processing the posted asynchronous tasks.
    - Each asynchronous function posts its task to the `io_context` using `asio::post`, which schedules the task to be executed by the `io_context`'s event loop.

4. **Thread Management:**
    - The `io_context` runs in a separate thread to handle asynchronous operations without blocking the main thread.
    - After all asynchronous operations complete, the main thread stops the `io_context` and joins the `io_thread` to ensure a clean shutdown.

**Output:**

```
Data fetched.
Computation complete.
Result saved: 84
All asynchronous operations completed successfully.
```

**Advantages:**

- **Explicit Synchronization**: Using `std::future` provides clear and explicit synchronization points, making it easier to manage dependencies between tasks.
- **Thread Safety**: `std::future` and `std::promise` are thread-safe, ensuring reliable communication between asynchronous tasks and the main thread.

**Disadvantages:**

- **Blocking Calls**: Using `.get()` blocks the main thread until the future is ready, which can negate some benefits of asynchronous execution if overused.
- **Manual Promise Management**: Developers must manually manage `std::promise` and `std::future`, which can introduce complexity and potential for errors.

---

## Comparative Analysis

| Feature                     | Chaining (Asio Handlers)               | Coroutines                | Futures (`std::future`)  |
|-----------------------------|----------------------------------------|---------------------------|--------------------------|
| **Syntax Complexity**       | Moderate                               | High                      | Moderate                 |
| **Readability**             | Moderate                               | Excellent                 | Good                     |
| **Control Flow**            | Explicit chaining via handlers         | Implicit via `co_await`    | Explicit synchronization |
| **Performance**             | Depends on thread management           | Potentially better        | Depends on thread management |
| **Error Handling**          | Requires manual management             | Simplified with try-catch  | Requires manual management |
| **C++ Version Required**    | C++11                                   | C++20                     | C++11                     |
| **Use Case Suitability**    | Simple sequential tasks                 | Complex asynchronous workflows | Simple to moderately complex tasks |
| **Maintenance**             | Can become cumbersome with deep nesting | Easier to maintain         | Easier than chaining     |

**Key Insights:**

- **Chaining** is suitable for straightforward sequences of dependent tasks but can become unwieldy with increasing complexity.
- **Coroutines** offer superior readability and maintainability, especially for complex asynchronous workflows, but come with increased syntax complexity and require C++20.
- **Futures** provide explicit control over synchronization and are easier to manage compared to chaining, but still require careful handling to avoid blocking the main thread excessively.

---

## Best Practices

1. **Choose the Right Method for the Task:**
    - **Chaining** is appropriate for simple sequences with minimal dependencies.
    - **Coroutines** are ideal for complex asynchronous workflows requiring multiple dependencies and enhanced readability.
    - **Futures** are suitable when explicit synchronization and control over task dependencies are needed.

2. **Avoid Excessive Blocking:**
    - While using `.get()` with futures provides synchronization, excessive blocking can lead to performance bottlenecks. Consider integrating coroutines or other non-blocking mechanisms to maintain responsiveness.

3. **Manage Thread Lifecycles Carefully:**
    - Ensure that threads launched for asynchronous operations are properly managed and joined or detached as appropriate to prevent resource leaks and undefined behavior.

4. **Handle Exceptions Gracefully:**
    - Implement robust error handling within asynchronous operations to catch and manage exceptions without crashing the application.

5. **Leverage Asio's Integration with Modern C++ Features:**
    - Utilize Asio's support for coroutines and other modern C++ features to write more efficient and maintainable asynchronous code.

6. **Optimize Resource Usage:**
    - Be mindful of the number of threads and asynchronous tasks launched to prevent overwhelming system resources. Utilize thread pools or limit concurrency where necessary.

7. **Document Asynchronous Workflows Clearly:**
    - Maintain clear documentation and code comments to elucidate the flow of asynchronous operations, especially when chaining or using coroutines.

---

## Conclusion

Asynchronous programming in C++ has evolved significantly, offering developers multiple paradigms to handle concurrent operations effectively. Asio's `io_context`, combined with C++ features like `std::future` and coroutines, provides a robust framework for managing asynchronous tasks.

- **Chaining Asynchronous Functions** using Asio handlers offers a straightforward approach for simple sequential tasks but can lead to complex and deeply nested code structures as the number of tasks increases.
  
- **Coroutines** in C++20 present a more elegant and maintainable solution for complex asynchronous workflows, allowing developers to write code that closely resembles synchronous code. This method enhances readability and reduces the complexity associated with callback chains.

- **Futures** (`std::future` and `std::promise`) provide explicit synchronization mechanisms, enabling developers to manage dependencies between tasks effectively. However, they require careful handling to avoid blocking the main thread and managing resources efficiently.

Understanding the strengths and limitations of each method empowers developers to make informed choices based on the specific requirements of their applications. By adhering to best practices and leveraging Asio's capabilities alongside modern C++ features, developers can build responsive, efficient, and maintainable asynchronous applications.

---

## References

1. **Asio C++ Library**  
   [Asio Official Documentation](https://think-async.com/Asio/asio-1.18.2/doc/asio/index.html)

2. **C++ Reference - `std::future`**  
   [cppreference.com - std::future](https://en.cppreference.com/w/cpp/thread/future)

3. **C++ Reference - Coroutines**  
   [cppreference.com - Coroutines](https://en.cppreference.com/w/cpp/language/coroutines)

4. **Boost.Asio Coroutines**  
   [Boost.Asio Coroutines Documentation](https://www.boost.org/doc/libs/1_75_0/doc/html/boost_asio/reference/co_spawn.html)

5. **Effective Modern C++ by Scott Meyers**  
   [Amazon Link](https://www.amazon.com/Effective-Modern-Specific-Ways-Improve/dp/1491903996)

6. **Asynchronous Programming with C++ Coroutines**  
   [Fluent C++ - Coroutines](https://www.fluentcpp.com/2019/09/12/cpp-coroutines-tutorial/)

7. **C++ Concurrency in Action by Anthony Williams**  
   [Manning Publications](https://www.manning.com/books/c-plus-plus-concurrency-in-action-second-edition)

8. **Asio Coroutines and C++20**  
   [Asio Coroutines Guide](https://think-async.com/Asio/asio-1.18.2/doc/asio/reference/co_spawn.html)
