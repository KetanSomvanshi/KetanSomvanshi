---
layout: post
title: "Structuring a FastAPI App: An In-Depth Guide"
date: 2023-06-06
author: Ketan Somvanshi
categories: [FastAPI]
---
---
## Table of Contents
- [Table of Contents](#table-of-contents)
- [Introduction to FastAPI](#introduction-to-fastapi)
- [Structure Overview](#structure-overview)
  - [Server](#server)
  - [Controller](#controller)
  - [Data Adapter](#data-adapter)
  - [Models](#models)
  - [Service](#service)
  - [Logger](#logger)
  - [Config](#config)
  - [Docker](#docker)
  - [Docker Test](#docker-test)
  - [Scripts](#scripts)
  - [Tests](#tests)
  - [Utils](#utils)
- [Dependency Injection](#dependency-injection)
- [Error Handling](#error-handling)
- [Additional Libraries And Concepts](#additional-libraries-and-concepts)
  - [Pydantic and Typing](#pydantic-and-typing)
  - [Context Managers](#context-managers)
  - [Working with DB Sessions](#working-with-db-sessions)
  - [Gunicorn Server](#gunicorn-server)
  - [Linting and Formatting](#linting-and-formatting)
- [Conclusion](#conclusion)


## Introduction to FastAPI

FastAPI is a powerful and high-performance Python web framework designed for building APIs quickly and efficiently. It leverages modern Python features such as type annotations and async/await syntax to provide a seamless development experience. FastAPI offers incredible speed and scalability, making it a preferred choice for building robust web applications.

With its performance, asynchronous support, automatic API documentation generation, and built-in request validation, FastAPI has gained popularity among developers.<span style="background-color: green;"> However, there aren't always enough resources available to guide developers on building well-structured, maintainable code with better coding practices. In this blog post, we will provide an in-depth guide on how to structure a FastAPI application for optimal maintainability and scalability.</span>

## Structure Overview

When creating a FastAPI application, it's important to follow a well-structured project organization to ensure readability, maintainability, and scalability. In the scope of this blog, we would go through an example of [Cart Service](https://github.com/KetanSomvanshi/cart-service) in FastAPI. Let's take a closer look at the structure overview provided in the Cart Service repository:

### Server

The [`server`](https://github.com/KetanSomvanshi/cart-service/tree/master/server) directory includes the main application files responsible for handling HTTP requests and routing. It typically includes files for initializing the application, configuring routes, and implementing authentication and authorization functionality.

### Controller

The [`controller`](https://github.com/KetanSomvanshi/cart-service/tree/master/controller) directory houses the API controllers responsible for handling requests and responses. Each controller file corresponds to a specific resource or module in the application. It contains files for handling different API endpoints and managing context and status codes.

### Data Adapter

The [`data_adapter`](https://github.com/KetanSomvanshi/cart-service/tree/master/data_adapter) directory contains modules responsible for interacting with the data layer, such as the database, cache, Elasticsearch, and more. It includes files for data manipulation, database connection handling, and all data layer-specific queries abstracted from the service layer.

### Models

The [`models`](https://github.com/KetanSomvanshi/cart-service/tree/master/models) directory holds the data models or schemas used by the application. It includes files for defining the application's models and their relationships. This also helps us to implement DTO(Data Transfer through Objects) pattern where we are exchanging data in between different layers through these model instances.

### Service

The [`service`](https://github.com/KetanSomvanshi/cart-service/tree/master/service) directory contains the business logic or services used by the application. Each service corresponds to a specific resource or module and includes files for implementing the business logic related to that resource.

### Logger

The [`logger`](https://github.com/KetanSomvanshi/cart-service/tree/master/logger) directory consists of modules related to logging and log configuration. It typically includes an initialization file for the logger module.

### Config

The [`config`](https://github.com/KetanSomvanshi/cart-service/tree/master/config) directory contains various configuration files required for the application's settings and environment variables. It includes files for initialization, constants, settings, and utility functions to inject those config parameters.

### Docker

The [`docker`](https://github.com/KetanSomvanshi/cart-service/tree/master/docker) directory includes Docker-related files for containerizing the FastAPI application. It typically contains a Dockerfile and a docker-compose.yml file for defining the application's Docker image and services.

### Docker Test

The [`docker_test`](https://github.com/KetanSomvanshi/cart-service/tree/master/docker_test) directory contains Docker-related files specifically for running tests within a containerized environment.

### Scripts

The [`scripts`](https://github.com/KetanSomvanshi/cart-service/tree/master/scripts) directory contains utility scripts for various purposes, such as setting up the database or generating data. It typically includes scripts like init_db.sql for initializing the database.

### Tests

The [`tests`](https://github.com/KetanSomvanshi/cart-service/tree/master/tests) directory contains unit tests to ensure the correctness of the application. It typically includes subdirectories for different components, such as service tests, and each test file corresponds to a specific component or module.

### Utils

The [`utils`](https://github.com/KetanSomvanshi/cart-service/tree/master/utils) directory houses utility modules and files required for the application's functionality. It typically includes files for handling exceptions, providing helper functions, and implementing common functionality such as JWT token handling and password hashing.

Other important files in the project structure include `.gitignore`, `README.md`, `entrypoint.sh`, `entrypoint_test.sh`, and `requirements.txt`. These files provide necessary information, manage dependencies, and assist in executing various tasks related to the project.

## Dependency Injection

FastAPI encourages the use of dependency injection to manage dependencies and promote testability and modular code. There are several dependency injection frameworks available for FastAPI, such as `Dependency` and `Inject`. These frameworks help organize and manage the dependencies required by different components of your application, making it easier to write testable and maintainable code.

## Error Handling

Proper error handling is crucial in any web application, and FastAPI provides mechanisms to handle exceptions and return appropriate error responses to clients. By using FastAPI's exception handling capabilities, you can gracefully handle errors and provide meaningful feedback to users when something goes wrong. like below -
```python
@app.exception_handler(ValidationError)
async def pydantic_validation_exception_handler(request: Request, exc):
    context_set_db_session_rollback.set(True)
    logger.error(extra=context_log_meta.get(), msg=f"data validation failed {exc.errors()}")
    return build_api_response(GenericResponseModel(status_code=http.HTTPStatus.BAD_REQUEST,
                                                   error="Data Validation Failed"))
```

## Additional Libraries And Concepts

FastAPI integrates well with several additional libraries and concepts that enhance its functionality and development experience. Let's explore a few of them:

### Pydantic and Typing

FastAPI recommends using the Pydantic library to create models and handle data validations, format conversions, encoding, and more. Pydantic simplifies the process of defining data models and ensures the incoming and outgoing data is properly validated and formatted. Additionally, FastAPI leverages Python's `typing` module to provide type annotations and improve code readability and maintainability.

### Context Managers

FastAPI allows you to set and access context within your application using context variables or Starlette's request and application state. Context managers help manage resources and variables that should be available within the context of a request or application.[In this scope we have used context variables](https://github.com/KetanSomvanshi/cart-service/blob/master/controller/context_manager.py).
We build a request context and inject it as a dependency for every request so that the context is set with every request.

### Working with DB Sessions

In every request we can initiate the DB transactions by injecting the dependecy that yields db session. And at the end of the request we can commit or rollback the transaction.
Example of db dependency injector -
```python
def get_db():
    """this function is used to inject db_session dependency in every rest api requests"""
    from controller.context_manager import context_set_db_session_rollback
    db: Session = SessionLocal()
    try:
        yield db
        #  commit the db session if no exception occurs
        #  if context_set_db_session_rollback is set to True then rollback the db session
        if context_set_db_session_rollback.get():
            logging.info('rollback db session')
            db.rollback()
        else:
            db.commit()
    except Exception as e:
        #  rollback the db session if any exception occurs
        logging.error(e)
        db.rollback()
    finally:
        #  close the db session
        db.close()
```

### Gunicorn Server

Gunicorn (Green Unicorn) is a widely used ASGI server for running Python web applications, including FastAPI. It provides high performance and scalability, making it suitable for production deployments. Gunicorn can handle multiple worker processes, allowing your FastAPI application to handle a large number of concurrent requests efficiently.

### Linting and Formatting 

we can use tox for linting and formatting checks - [Sample tox file](https://github.com/KetanSomvanshi/cart-service/blob/master/tox.ini)


## Conclusion

Structuring your FastAPI application properly is essential for maintaining code quality, scalability, and readability. By following the suggested structure and incorporating best practices like dependency injection, error handling, and utilizing additional libraries, you can build robust and maintainable FastAPI applications. Remember to adapt the structure to your specific project requirements and keep the codebase organized as it grows over time.
