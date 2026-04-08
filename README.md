# MovieDB backend

# Component

## MovieDB - Gateway Service

### Related components

+ https://github.com/mzalietin/mdb-aggregator
+ https://github.com/mzalietin/mdb-demo

# Task

Application that stores info about movies, users and their reviews.

<details>
  <summary>Click to expand</summary>

Movie entity:
* Name (unique), issueDate, rating (double).

User entity:
* FirstName, lastName, age, username (unique).

MovieReview entity:
* User, movie, rating (integer from 1 to 10), comment.

You have 2 main services: we can call them gateway-service and aggregator-service.
Gateway-service is the main entrance of your application. Gateway-service does not have access to your main storage (where all data is stored), however it can call some caching storage for some GET operations. It can call aggregator-service to retrieve data, it can call any service to put data. It supports the following operations:
* GET – top 10 movies with the highest rating
* GET – user info by username
* GET – top 10 favorite user movies by username
* GET – movie rating by movie name
* POST – create new movie
* POST – create new user
* POST – add new movie review
* PUT – update movie review
* DELETE – delete movie review
* DELETE – delete user (all user reviews should be also deleted in 10 minutes)

Customer requirements:
- Each new/updated/deleted movie review should affect movie rating in 5 minutes.
- You can receive up 200 movie reviews per second.
- Load balancing should be supported.
- All movie rating calculation logic should be placed inside aggregator-service.

</details>

# Design

<details>
  <summary>Click to expand</summary>

This application is a modular monolith that implements **Saga pattern with Event-Sourcing**.

The design addresses following concerns:
+ **Command-Query Responsibility Segregation (CQRS)** - aggregates which handle commands (`movie`, `movie-review`, `user`) are separated from read projection (`query-service`)
+ **Replay capability** - projection (`query-service`) can be rebuilt from event store (Kafka)
+ **Idempotent consumers** - consumers (`movie`, `movie-review`, `query-service`) are configured to handle duplicate messages in a fail-safe manner, but aim to leverage Kafka's Exactly-Once Semantics

Acknowledged but unaddressed concerns:
+ **Transactional Outbox Pattern** - would avoid dual write problem (synchronous DB + Kafka write) with potential lost events
+ **Snapshots** - avoid replaying millions of events
+ **Security**
+ **Observability**
+ **CI/CD**
+ **Unit/Integration test coverage**
+ etc. due to timebox constraints

</details>

## Architectural diagram

<details>
  <summary>Click to expand</summary>

![Alt text of the image](https://github.com/mzalietin/mdb-demo/blob/2bc910ca8fd9d656ca6ce7d569f63c5c76642277/diagram/mdb-backend.jpg)

</details>

## REST API

Environment: `mdb-aggregator` running on `localhost`

| Desc                        | Method | URL                                                              | Body                                                                              |
|-----------------------------|:------:|:-----------------------------------------------------------------|:----------------------------------------------------------------------------------|
| create movie                |  POST  | `http://localhost:8080/api/movies`                               | {"name":"string","releaseDate":"yyyy-mm-dd"}                                      |
| create review               |  POST  | `http://localhost:8080/api/movie-reviews`                        | {"movieId":"string","username":"string","rating":"int [1-10]","comment":"string"} |
| update review               |  PUT   | `http://localhost:8080/api/movie-reviews`                        | {"movieId":"string","username":"string","rating":"int [1-10]","comment":"string"} |
| delete review               | DELETE | `http://localhost:8080/api/movie-reviews`                        | {"movieId":"string","username":"string"}                                          |
| create user                 |  POST  | `http://localhost:8080/api/users`                                | {"username":"string","firstName":"string","lastName":"string","age":"int"}        |
| delete user                 | DELETE | `http://localhost:8080/api/users/{username}`                     |                                                                                   |
| movie rating by name        |  GET   | `http://localhost:8080/api/movies/rating?name={movieName}`       |                                                                                   |
| top movies by avg rating    |  GET   | `http://localhost:8080/api/movies/top/{limit}`                   |                                                                                   |
| top movies by user's rating |  GET   | `http://localhost:8080/api/movie-reviews/{username}/top/{limit}` |                                                                                   |
| user info by username       |  GET   | `http://localhost:8080/api/users/{username}`                     |                                                                                   |

## Tech stack

+ Java 17
+ Spring Cloud Gateway WebFlux

# Build

## Default build & test

Environment: Java 17

`./gradlew clean build`

### Build Docker image

Environment: Docker

`./gradlew clean build jibDockerBuild`

## Run

`java -jar build/libs/mdb-gateway-0.0.1.jar`

## Debug

`java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 -jar build/libs/mdb-gateway-0.0.1.jar`
