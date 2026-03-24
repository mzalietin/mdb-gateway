# MovieDB backend

## Component description

MovieDB - Gateway Service

## Task description

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

## Build

Prerequisites: Java 17

`gradlew clean build`

## Run

`java -jar build/libs/mdb-gateway-0.0.1.jar`

## Debug

`java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005 -jar build/libs/mdb-gateway-0.0.1.jar`
