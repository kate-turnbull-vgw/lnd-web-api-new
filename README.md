# L&D Web API exercise

This repository hosts a Learning & Development exercise to build a Web API.

The goal of this exercise is for you to build a simple HTTP Server that passes a set of tests included in this repo.
The automated test script is provided as [PostMan scripts](https://www.getpostman.com/) and can be run from the command line with [newman](https://www.npmjs.com/package/newman).

## Requirements

The minimum requirement you will have is to be able to build a web server that supports HTTP 1.1.
You can choose your programming language, frameworks and libraries.

To complete the exercise entirely you will also need Docker installed so that you can produce and run docker images.

## Problem space

In this exercise you will build a simple wallet to store and retrieve a _coin_ balance for a player.
The rules are pretty simple:

* Requests for an unknown wallet's coin balance return 404 HTTP response code.
* Requests for an known wallet's coin balance return the balance with a 200 HTTP response code.
* Credits to a wallet will return the updated balance and a 201/Created HTTP response code.
* Debits to a wallet will return the updated balance and a 201/Created HTTP response code.
* Debit for more than the balance will be rejected with a 400/BadRequest HTTP response code.
* If the same Credit or Debit is issued immediately after each other, the request is acknowledged as successful, but is not actually processed. The balance is returned with a 202/Accepted HTTP response code.
* A wallet is created simply by crediting it.

## Exercise Objectives

You may complete the objectives in any order, but the following is the recommended approach.

### 1. Create a basic web server

Create a web server that responds with a either a 404 or the following balance payload when are request is issued to http://localhost:8080/wallets/{someid}

```JSON
{
    "transactionId": "tx101",
    "version" : 1,
    "coins" : 100
}
```

Next implement a Credit endpoint on `http://localhost:8080/wallets/{some-id}/credit` that takes an HTTP Post request with the following payload and returns a balance.

```JSON
{
    "transactionId" : "tx102",
    "coins" : 1000
}
```

### 2. Implement the domain model

Create an independent module that will host your domain model.
This module should have no dependencies.
Create a suite of unit tests for this module.

Ideally in a test first approach, create a wallet domain model that will incrementally satisfy the requirements of crediting, debiting, rejecting an excessive Debit, and ensuring an idempotent system.

Part of this work should ideally include setting up your project with a code coverage report.

### 3. Pass all automated test scripts

Now integrate your web server to your domain model.
Keeping your domain model free of any dependencies.
It should not know about serialization formats, web protocols, or anything but the language it is built with.

The web server now needs to take requests and map them to commands issued to your domain model.
Once this is done you should be able to run the web server locally and execute the postman scripts to validate your implementation.
You may have to implement an in-memory _repository_.

You can install the Postman GUI from here - https://www.getpostman.com/.
If you want to run from the command line, then _newman_ can be installed via NPM

```bash
npm install -g newman
```

The scripts can be executed with newman as such:

```bash
newman run ./postman/LND-Web-API.postman-collection.json
```

### 4. Integrate with a data store

If you have created a Repository interface and provided an in memory implementation in the last step, here you can replace it with an implementation that will actually persist to a data store.

The data store provisioning should be scripted.

Use the Postman scripts to validate your implementation.

### 5. Build and run in Docker

Now that you have a fully working implementation, lets host it in Docker.
You will also want to have any build steps executed in docker.
This is to avoid your reviewer needing to installing a compatible tool set to which you have built it in.

Finally use Docker Compose to run your web application and your data store.

Again, use the Postman Scripts to validate your implementation.

### Example of successful output

After running the newman cli tool
```bash
$ newman run ./postman/LND-Web-API.postman-collection.json
```

You should expect to see the following output:

```bash
newman
VGW LND-Web-API
→ Get Empty Wallet Balance
  GET http://localhost:8080/wallets/1d4e7d81-ce9d-457b-b056-0f883baa783d [404 Not Found, 568B, 556ms]
  ✓  Status code is 404-Not Found
→ Credit Wallet with initial balance
  POST http://localhost:8080/wallets/1d4e7d81-ce9d-457b-b056-0f883baa783d/credit [201 Created, 178B, 103ms]
  ✓  Status code is 201-Created
→ Get Wallet Balance after initial credit
  GET http://localhost:8080/wallets/1d4e7d81-ce9d-457b-b056-0f883baa783d [200 OK, 189B, 20ms]
  ✓  Status code is 200-Ok
  ✓  Payload as expected
→ Duplicate Credit
  POST http://localhost:8080/wallets/1d4e7d81-ce9d-457b-b056-0f883baa783d/credit [202 Accepted, 179B, 23ms]
  ✓  Status code is 202-Accepted
→ Debit Wallet with more than balance
  POST http://localhost:8080/wallets/1d4e7d81-ce9d-457b-b056-0f883baa783d/debit [400 Bad Request, 580B, 23ms]
  ✓  Status code is 400-Client Error
→ Debit Wallet with less than balance
  POST http://localhost:8080/wallets/1d4e7d81-ce9d-457b-b056-0f883baa783d/debit [201 Created, 176B, 29ms]
  ✓  Status code is 201-Created
→ Duplicate Debit
  POST http://localhost:8080/wallets/1d4e7d81-ce9d-457b-b056-0f883baa783d/debit [202 Accepted, 177B, 25ms]
  ✓  Status code is 202-Accepted
→ Get Wallet Balance
  GET http://localhost:8080/wallets/1d4e7d81-ce9d-457b-b056-0f883baa783d [200 OK, 188B, 19ms]
  ✓  Status code is 200-Ok
  ✓  Payload as expected
┌─────────────────────────┬──────────┬──────────┐
│                         │ executed │   failed │
├─────────────────────────┼──────────┼──────────┤
│              iterations │        1 │        0 │
├─────────────────────────┼──────────┼──────────┤
│                requests │        8 │        0 │
├─────────────────────────┼──────────┼──────────┤
│            test-scripts │        8 │        0 │
├─────────────────────────┼──────────┼──────────┤
│      prerequest-scripts │        0 │        0 │
├─────────────────────────┼──────────┼──────────┤
│              assertions │       10 │        0 │
├─────────────────────────┴──────────┴──────────┤
│ total run duration: 1004ms                    │
├───────────────────────────────────────────────┤
│ total data received: 1.06KB (approx)          │
├───────────────────────────────────────────────┤
│ average response time: 99ms                   │
└───────────────────────────────────────────────┘
```

## Further reading and learning content

### Web APIs and ReST

#### June 2014 HTTP RFC (RFC 7230 family)

The formal specification of the HTTP 1.1 specification (obsoleting both the RFC 2068 of 1997 and the RFC 2616 of 1999)

* [RFC 7230](https://en.wikipedia.org/wiki/Request_for_Comments_(identifier)), HTTP/1.1: Message Syntax and Routing
* [RFC 7231](https://tools.ietf.org/html/rfc7231), HTTP/1.1: Semantics and Content
* [RFC 7232](https://tools.ietf.org/html/rfc7232), HTTP/1.1: Conditional Requests
* [RFC 7233](https://tools.ietf.org/html/rfc7233), HTTP/1.1: Range Requests
* [RFC 7234](https://tools.ietf.org/html/rfc7234), HTTP/1.1: Caching
* [RFC 7235](https://tools.ietf.org/html/rfc7235), HTTP/1.1: Authentication

Note that while at the time of writting in 2019, that HTTP 2.0 and HTTP 3.0 do have formal RFCs, they are not the target of this exercise.

#### Architectural Styles and the Design of Network-based Software Architectures

Roy Fielding's seminal dissertation that introduced the concept of Representational State Transfer (ReST) (specifically in Chapter 5) as an architectural style suitable for systems built upon HTTP network protocol.

<https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf>

#### REsT in Practice : Hypermedia and Systems Architecture

A practical guide that introduces the concept of the 3 maturity levels of a Web API.

* <http://restinpractice.com>
* <https://www.amazon.com/REST-Practice-Hypermedia-Systems-Architecture/dp/0596805829>

### Isolated domain models

#### Ports and Adapters - Alistair Cockburn

a.k.a _Hexagonal Architecture_, a.k.a _Onion architecture_

An application design pattern that drives better encapsulation and testability through isolation of the domain from the host and the infrastructure.
The Domain defines the interfaces (Ports) that it requires and the host application is to compose and provide implementations of these interfaces (Adapters).

* <http://alistair.cockburn.us/Hexagonal+architecture> (under reconstruction at time of writting)
  * [web archive](https://web.archive.org/web/20180822100852/http://alistair.cockburn.us/Hexagonal+architecture)
* <https://blog.octo.com/en/hexagonal-architecture-three-principles-and-an-implementation-example/>
* <http://wiki.c2.com/?HexagonalArchitecture>