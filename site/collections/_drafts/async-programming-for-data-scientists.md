---
date: 2023-05-02 00:00:00
title: Async Programming for Data Scientists
description: >-
  Asynchronous programming is a technique that enables your program to start a
  potentially long-running task and still be able to be responsive to other
  events while that task runs, rather than having to wait until that task has
  finished.
tags:
  - python
  - data-science
  - performance-coding
image: /uploads/async-programming.jpeg
---
Asynchronous programming is a powerful technique that allows developers to write more efficient and responsive applications. By executing multiple tasks concurrently, asynchronous programming can help improve the performance of your code, especially when dealing with I/O-bound tasks such as network requests or database queries.

While asynchronous programming has been around for a long time, it has become more popular in recent years as more developers have embraced its benefits. Asynchronous programming is supported in many programming languages, including Python, JavaScript, and Java, among others.

In this post, we will explore the basics of asynchronous programming, including:

* Differences between synchronous and asynchronous programming&nbsp;
* Benefits and drawbacks of asynchronous programming.
* Using asynchronous programming as a data scientist
* Asynchronous Programming outside of data science

## Synchronous vs. Asynchronous Programming

Synchronous and asynchronous programming are two distinct approaches to managing the execution of tasks in software development. Both paradigms have their advantages and trade-offs, and understanding their differences is crucial for developers when deciding which approach is the best for the task at hand.

Synchronous programming follows a linear execution model, where tasks are executed sequentially in the order they are called. This means that a task must be completed before the next one can start. While this approach is often simpler to understand and implement, it can lead to inefficiencies in situations where a task takes a long time to complete, such as waiting for data from a slow network or disk. In these cases, synchronous programming can cause the application to become unresponsive or "blocked," as it has to wait for the slow task to finish before moving on to the next one.

In contrast, asynchronous programming allows tasks to be executed concurrently, without having to wait for the completion of previous tasks. This is achieved by using callbacks, promises, or other mechanisms to manage the flow of execution, so that when a long-running task is encountered, the program can continue processing other tasks in parallel. As a result, asynchronous programming is better suited for handling tasks with unpredictable or variable completion times, such as those involving network requests or user input. However, the non-linear nature of asynchronous programming can make it more complex to understand, design, and debug, as it often involves managing multiple execution paths and handling potential race conditions.

## Benefits and drawbacks of asynchronous programming

Asynchronous programming offers several advantages and disadvantages, which should be considered when designing the system. Here are some key benefits and drawbacks:

### Benefits

1. **Improved performance:** Asynchronous programming enables better utilization of system resources by allowing tasks to be executed concurrently, without waiting for the completion of other tasks. This can lead to improved performance, especially when dealing with tasks that involve network requests, user input, or other time-consuming operations.
2. **Scalability:** Asynchronous code is often more scalable, as it can handle a larger number of simultaneous requests or tasks without being blocked by slow operations. This is particularly important for server-side applications, where the ability to handle many concurrent connections is crucial.
3. **Responsiveness:** Asynchronous programming can improve the responsiveness of applications, as it allows them to continue processing other tasks while waiting for long-running operations to complete. This can be especially beneficial for user interfaces, as it prevents the application from freezing or becoming unresponsive during time-consuming tasks.
4. **Better resource utilization:** Asynchronous code can lead to better resource utilization, as it allows for the efficient use of CPU and memory resources by executing tasks concurrently and sharing resources among multiple tasks.

### Drawbacks

1. **Increased complexity:** Asynchronous programming can be more complex to understand, design, and debug, as it often involves managing multiple execution paths and handling potential race conditions or deadlocks.
2. **Difficulty in error handling:** Error handling can be more challenging in asynchronous programming, as errors may occur at different stages of the execution process and may need to be propagated through multiple callbacks or promises.
3. **Harder to read and maintain:** Asynchronous code can be more difficult to read and maintain, as it often involves nested callbacks, promises, or other constructs that can make the code harder to follow.
4. **Potential performance overhead:** While asynchronous programming can improve performance in certain situations, it can also introduce overhead due to the need to manage callbacks, promises, or other concurrency mechanisms. This can sometimes result in slower execution times compared to synchronous code, particularly when dealing with small tasks or simple operations.
5. **Learning curve:** Developers may need to learn new programming concepts and techniques when transitioning from synchronous to asynchronous programming, which can require additional time and effort.

## Using asynchronous programming as a data scientist

Data scientists can leverage asynchronous programming in Python to improve the performance and responsiveness of their data processing tasks, particularly when dealing with I/O-bound operations or tasks that can run concurrently. Asynchronous programming in Python is primarily supported by the `asyncio` library, which provides an event loop, coroutines, and other tools for managing concurrency.

Here are some ways data scientists can use asynchronous programming in Python:

### **Web scraping**

When scraping data from multiple web pages, asynchronous programming can speed up the process by allowing multiple HTTP requests to be executed concurrently, rather than waiting for each request to complete sequentially.

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup

# Asynchronous function to fetch the content of a given URL
async def fetch_content(url, session):
    async with session.get(url) as response:
        return await response.text()

# Asynchronous function to parse the fetched content and extract article titles
async def extract_article_titles(url):
    async with aiohttp.ClientSession() as session:
        content = await fetch_content(url, session)
        soup = BeautifulSoup(content, 'html.parser')
        article_titles = [title.text for title in soup.find_all('h2', class_='post-title')]
        return article_titles

# Main asynchronous function to run the web scraping tasks
async def main():
    url = 'https://example-blog.com'
    article_titles = await extract_article_titles(url)
    print(f'Article titles from {url}:')
    for title in article_titles:
        print(f'- {title}')

# Run the main function using asyncio.run
if __name__ == '__main__':
    asyncio.run(main())
```

### API Requests

Data scientists often need to fetch data from various APIs, which can involve numerous network calls. Using asynchronous programming allows multiple API requests to be executed simultaneously, reducing the overall waiting time.

```python
import asyncio
import httpx

# Asynchronous function to fetch data from a given API endpoint
async def fetch_data(url, session):
    async with session.get(url) as response:
        return await response.json()

# Asynchronous function to fetch multiple user data using the API
async def fetch_users_data(user_ids):
    async with httpx.AsyncClient() as session:
        tasks = [fetch_data(f'https://jsonplaceholder.typicode.com/users/{user_id}', session) for user_id in user_ids]
        users_data = await asyncio.gather(*tasks)
        return users_data

# Main asynchronous function to run the API request tasks
async def main():
    user_ids = [1, 2, 3, 4, 5]
    users_data = await fetch_users_data(user_ids)
    print("User data:")
    for user_data in users_data:
        print(f"{user_data['id']}: {user_data['name']} ({user_data['email']})")

# Run the main function using asyncio.run
if __name__ == '__main__':
    asyncio.run(main())
```

### Data Preprocessing

Asynchronous programming can be used to parallelize data preprocessing tasks, such as reading and processing large files or cleaning and transforming datasets. This can significantly improve the performance of data preparation workflows.

Asynchronous programming may not always provide significant benefits for CPU-bound tasks, such as data preprocessing. This is because Python's Global Interpreter Lock (GIL) allows only one thread to execute Python bytecode at a time, even on multi-core systems. However, you can still use asynchronous programming for I/O-bound preprocessing tasks, such as reading data from multiple files.

```python

import asyncio
import aiofiles
import pandas as pd
from io import StringIO

# Asynchronous function to read the content of a given file
async def read_file(file_path):
    async with aiofiles.open(file_path, mode='r') as file:
        content = await file.read()
        return content

# Asynchronous function to read and parse CSV data from multiple files
async def load_csv_data(file_paths):
    tasks = [read_file(file_path) for file_path in file_paths]
    csv_contents = await asyncio.gather(*tasks)

    dataframes = []
    for content in csv_contents:
        buffer = StringIO(content)
        df = pd.read_csv(buffer)
        dataframes.append(df)

    return dataframes

# Main asynchronous function to run the data preprocessing tasks
async def main():
    file_paths = ['data1.csv', 'data2.csv', 'data3.csv']
    dataframes = await load_csv_data(file_paths)

    # Perform further preprocessing and analysis tasks with the loaded DataFrames
    for idx, df in enumerate(dataframes):
        print(f"Data from file {file_paths[idx]}:")
        print(df.head())

# Run the main function using asyncio.run
if __name__ == '__main__':
    asyncio.run(main())
```

### Parallelizing Model Training

Asynchronous programming can help data scientists train multiple machine learning models concurrently, particularly when using distributed computing or cloud-based resources.

```python
import numpy as np
import multiprocessing
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import SVC

# Function to train a model and return the accuracy score
def train_model(model, X_train, X_test, y_train, y_test):
    model.fit(X_train, y_train)
    accuracy = model.score(X_test, y_test)
    return accuracy

# Main function to run the parallel model training tasks
def main():
    iris = load_iris()
    X_train, X_test, y_train, y_test = train_test_split(iris.data, iris.target, test_size=0.2, random_state=42)

    models = [
        LogisticRegression(solver='lbfgs', multi_class='auto', max_iter=1000),
        RandomForestClassifier(n_estimators=100),
        SVC(kernel='rbf', C=1.0, gamma='auto')
    ]

    with multiprocessing.Pool() as pool:
        results = pool.starmap(train_model, [(model, X_train, X_test, y_train, y_test) for model in models])

    for idx, accuracy in enumerate(results):
        print(f"Model {idx + 1} accuracy: {accuracy:.2f}")

if __name__ == '__main__':
    main()
```

## Asynchronous Programming outside of data science

Asynchronous programming is widely used outside of data science to improve the performance and responsiveness of various types of applications and systems. We won't provide examples in code here, but some common use cases include:

1. **Web development:** In web applications, asynchronous programming is commonly used to handle user input, perform background tasks, or fetch data from APIs without blocking the main thread, improving the responsiveness of the user interface. Modern JavaScript frameworks and libraries, such as Node.js, React, and Angular, rely heavily on asynchronous programming.
2. **Web servers:** Asynchronous web servers, like Node.js or Python's Tornado or FastAPI, can handle multiple incoming requests concurrently without blocking, which helps maintain responsiveness and scalability under heavy load.
3. **Networking and communication:** Asynchronous programming is often employed in network programming to handle multiple concurrent connections or to perform tasks like sending and receiving data without waiting for each operation to complete, allowing better utilization of network resources and improving overall performance.
4. **Real-time applications:** Asynchronous programming is useful for developing real-time applications, such as online gaming, chat applications, or live data streaming, where the system needs to react quickly to events and maintain a high level of responsiveness.
5. **File I/O operations:** Asynchronous programming can be used to perform file reading and writing operations concurrently without blocking the main execution flow, which can significantly speed up tasks that involve handling multiple files or large amounts of data.
6. **GUI applications:** In graphical user interface (GUI) applications, asynchronous programming is used to perform background tasks or update the UI without freezing the application, ensuring a smooth and responsive user experience.
7. **Microservices and distributed systems:** Asynchronous programming is a crucial component in the development of microservices and distributed systems, where multiple services communicate with each other through APIs or message queues. By using asynchronous communication, these systems can handle requests and process data concurrently, improving performance and fault tolerance.

Feel free to reach out with any questions using the&nbsp;[contact](https://zakraicik.xyz/contact/)&nbsp;page or hitting me up on any of my social links!