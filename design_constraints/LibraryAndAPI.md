
# Application - ToDO list application

## SPA application

It will contain UI and BFF for ToDO List application with the next design constraints with the latest React best practice.

* Latest React as application framework
* React Testing Library as testing library
* Playwright testing should be added to critical user case where it make sense
* Suggest Component Library to be used in UI

## Rest API for API in React application as BFF

It will contain Java SpringBoot Application exposting Rest API for API in React application as BFF

### Rest API Controller

* Separate API to be consumed by SPA application
* API using Latest Java SpringBoot application
* Spring Web should expose Rest API endpoints

### Service Layer

* Service layer should be called by Rest API endpoints and it should encapsulate all the necessary logic to call Repository
* Spring Data JPA to act as data repository
* H2 DataBase for data store

### Data Layer to host Data to store TODO items

Data layer should act as repository with Spring Data JPA to expose

* It should contain Spring Data JPA as repository
* It should host JPA entity for data table, table attributes designed with Spring Boot Data JPA best practice.
* It should contain Mapper between DTO to Business entity.
