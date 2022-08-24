# Backend technical test: price map in Paris

## Objective

The objective is to build a price map of the average selling prices (€/m²) per 
_arrondissement_ (district) in Paris, with statistics, from an API of listings from 
Meilleurs Agents.

Here is an example of what we want to achieve:

![image 2](resources/img/image2.png)
![image 1](resources/img/image1.png)

## Current application

### Dependencies

At Meilleurs Agents, we need to collect listings around different websites and sources
(like Se Loger, notaries, government, etc.) to be able to follow the market price 
evolution and have the most accurate estimations.

To simplify this exercise, we simulate this gathering with an API: the listingAPI.
**You must NOT edit the listingAPI.**

As a database, we use PostgreSQL within a dockerized instance.

### Two languages

This repository contains two implementations of the `pricemap` API:
- `pricemap-python/`: provides a Python implementation
- `pricemap-typescript/`: provides a TypeScript implementation

You are expected to edit one of the two folders depending on your personal preferences.
In this README, the API is called `pricemap-*` regardless of the implementation you
choose to work with.

### The application

A first version of the project is already completed: called `pricemap-*`.

It gives you:
- a web application for visualization
- an API which contains a first implementation for the part 1 ("Collecting data") and 
the part 2 ("Displaying prices")

**The entirety of your code must run in this `pricemap-*` container.**

You are completely free to rewrite every part of the `pricemap-*` application the way
you want and/or need.

### Requirements and usage

To use it, the following tools are required:
- `docker`
- `docker-compose`
- `make`

For the rest, you can use the tools you prefer.

A documentation detailing how the project is designed can be found [here](./usages.md) 
to understand how to start the project, interact and enter into containers if needed, 
etc.

### What we will look at

- If you found all mistakes in current source code
- How you answered to the following exercises
- How you justify your choices
- How maintainable is your code
- Your way of working
- And globally how expert you are about all subjects

## 1 - Collecting data

We want to collect the entirety of real estate properties in Paris. These properties 
will be retrieved from the real estate properties API (`listingapi`), that you will 
need to browse entirely.

You can access this API, after starting the project:
- from your local environment, at: http://localhost:8181/listings/32682
- from the container of your `pricemap-*` application at:
http://listingapi:5000/listings/32682

To get started, a first implementation already exists in the `app.py` or `app.ts` file.
This code adds a `/update_data` endpoint to the `pricemap-*` app, which starts the data
retrieval process. If you feel it is necessary to change the way the data retrieval
process is started, feel free to change it.

**This code does work, but requires improvements.**

The aim of this part is to rework and complete existing code, to make it cleaner, easier
to maintain and to match the state of the art.
You can work the way you want, you can move code, add libraries, change the way code is 
called, etc.

### 1.1 - Location filter

In Paris, we want to display listings per _arrondissement_.

When calling the `listingapi`, the parameter of the request named `place_id` will 
successively take the value of each Paris _arrondissement_ identifier as presented in 
the following array.

Those identifiers are also available in database, in the public schema in a table called
`geo_place`, containing Paris _arrondissements_ and their COG (Code Officiel 
Géographique - official geographical code).

| Arrondissement | Id    |
|----------------|-------|
| Paris 1        | 32682 |
| Paris 2        | 32683 |
| Paris 3        | 32684 |
| Paris 4        | 32685 |
| Paris 5        | 32686 |
| Paris 6        | 32687 |
| Paris 7        | 32688 |
| Paris 8        | 32689 |
| Paris 9        | 32690 |
| Paris 10       | 32691 |
| Paris 11       | 32692 |
| Paris 12       | 32693 |
| Paris 13       | 32694 |
| Paris 14       | 32695 |
| Paris 15       | 32696 |
| Paris 16       | 32697 |
| Paris 17       | 32698 |
| Paris 18       | 32699 |
| Paris 19       | 32700 |
| Paris 20       | 32701 |

#### 1.2 - Pagination

The `listingapi` returns 20 listings per page. You will have to browse all pages for 
all _arrondissements_, using the parameter `?page=<page_number>`.

Example: http://listingapi:5000/listings/32682?page=7

The number of pages is different for each _arrondissement_: from zero to dozens of 
pages. You will have to imagine a mechanism that can be adapted to the number of pages 
to browse. There are multiple implementations possible.

### 1.3 - Extracting listings properties

For each listing, we are interested in the following properties:

- `listing_id`: listing identifier for MeilleursAgents;
- `place_id`: _arrondissement_ identifier (as passed in the query parameters);
- `price`: selling price, integer, euros;
- `area`: area, integer, square meters;
- `room_count`: number of rooms, integer.

It is possible that some properties might not be exposed as expected by the `listingapi`
API, and that some of those might require to be extracted. Beware of the 1-room 
apartments called « Studio ».

### 1.4 - Data structure in database

Once listings are extracted, following above specification, they must be stored in a 
database, in one or multiple tables. The existing code implementation stores listings 
in a `listings` table.
As told before, you can adjust data modeling if needed.

In addition to their properties, we also want to model the evolution of listings across 
time. Concretely, we want to know:
- when the listing was put online (or at least when it was seen for the first time)
- when the listing was removed (or when we saw it for the last time)
- the full price history

Here are the required credentials to connect to the database:

- type: PostgreSQL
- host: `db`
- port: `5432`
- user: `pricemap`
- password: `pricemap`
- database: `pricemap`

## 2 - Displaying prices

The map and the histogram presented in the introduction of this document are served by 
the `pricemap-*`.

As for part 1, the web application is already working, but **can be improved**.

### 2.1 - Displaying prices for each _arrondissement_

When the page loads, the JavaScript code that generates the map calls the `pricemap-*`
web application to get the list of geographical entities to display. The web application 
responds with GeoJSON formatted data that contains a list of _arrondissements_ to 
display, their geometrical shape as well as an average price. The color of the shape 
depends on the price of the _arrondissement_ that is represented, following the same 
color scale as the one currently used on MeilleursAgents website for Paris map.

For each _arrondissement_, the average price per square meter is computed, to be 
displayed on the website.

Code is already implemented in the `/geoms` endpoint. As for the first part of this
exercise, code does work but can be improved: by checking the calculation, improving the
SQL request, improving data modeling, etc.

### 2.2 - Displaying stats for each _arrondissement_

When we click on an _arrondissement_, a histogram is displayed. It represents the 
listings' distribution per price bin for this _arrondissement_. As previously, 
JavaScript code in charge of generating this histogram asks the web application 
before each display, by passing the _arrondissement_ identifier as a query parameter. 
The web application responds with JSON formatted data that contains, among others, the 
values for each bar of the histogram. The y-axis is then automatically scaled based on 
the returned values.

For each targeted _arrondissement_, the listings price distribution is calculated on 
the API side, to be integrated to the response of the web application. The JavaScript 
code will use this response to generate the histogram.

Code is already implemented in the `/get_price/<path:cog>` endpoint in 
`pricemap-python/pricemap/blueprints/api.py` or `pricemap-typescript/src/api.ts`.
As for the other endpoint, code can be improved. 
You can change distribution, labels, calculation method of the histogram, etc.

### 2.3 - Displaying the average price of the _arrondissement_ (bonus)

Between the map and the histogram, we never display the average price of the 
_arrondissement_ in as a number. What could we do to display it on the web page when we 
click on an _arrondissement_ on the map?

## 3 - Industrialization

### 3.1 - Scaling

We now want to have price history at France scale to offer a price map containing values
as recent as possible to the users of our public website.

To do so, you need to think about an architecture that will fulfill the following 
constraints:

1. Inserting listings prices for France in less than 5 minutes
2. Guarantee a <500ms response time when consulting the map, independently of the number
of users currently connected

You have all the money you want, can use any tool, language, framework, infrastructure.

The expected output of your study is a schema (you can for instance use 
https://app.diagrams.net/) that will be used as a base for discussion during the 
debrief.
