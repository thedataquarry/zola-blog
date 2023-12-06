+++
title =  "Neo4j for Pythonistas (2)"
description = "Build a REST API on top of a Neo4j graph"
date = 2023-06-04
draft = false

[taxonomies]
tags = ["neo4j", "async", "graph-db"]
categories = ["Tools"]

[extra]
toc = true
comment = true
+++

## Build a RESTful API on top of a Neo4j graph

This is the second part of a series on Neo4j for Pythonistas, in which we will go through an end-to-end workflow to build and analyze graph data in Neo4j using Python. [Part 1 of this series](https://prrao87.github.io/posts/neo4j-python-1/) covered what the data was, how it was validated in Pydantic, and how it was ingested into Neo4j. If all of that is familiar to you, read on!

{% note() %}
If you like reading code and want to directly see the FastAPI codebase for this post, go to `src/api` [in the source repo](https://github.com/prrao87/neo4j-python-fastapi/tree/main).
{% end %}

### Why build a REST API?

Data engineers typically build data-handling and ETL pipelines to ingest large amounts of data into a database. However, for the true value of the data to be realized, it's also important that the data is made available to end users (i.e., "consumers") in the most convenient way possible. In most cases, the consumer would be a front-end or full stack developer responsible for building a client-facing application for a business case. An API layer that sits between the database (server) and the front end (client) application is ideally suited for this purpose -- it allows a database/backend engineer to ensure that the data being stored is being queried, and most importantly, _served_ to the client as necessary to deliver the most value to the business unit that builds the application.

{{ figure(src="api_design.jpeg") }}

## A quick recap on the data

As described in [part 1 of this series](https://thedataquarry.com/posts/neo4j-python-1/), we're working with a dataset of 130k wine reviews ingested as a graph into Neo4j. A sample wine review is shown below.

```json
{
    "points": "90",
    "title": "Castello San Donato in Perano 2009 Riserva  (Chianti Classico)",
    "description": "Made from a blend of 85% Sangiovese and 15% Merlot, this ripe wine delivers soft plum, black currants, clove and cracked pepper sensations accented with coffee and espresso notes. A backbone of firm tannins give structure. Drink now through 2019.",
    "taster_name": "Kerin O'Keefe",
    "taster_twitter_handle": "@kerinokeefe",
    "price": 30,
    "designation": "Riserva",
    "variety": "Red Blend",
    "region_1": "Chianti Classico",
    "region_2": null,
    "province": "Tuscany",
    "country": "Italy",
    "winery": "Castello San Donato in Perano",
    "id": 40825
}
```

The graph data model used is a simple one, for the purposes of this blog post.

{{ figure(src="schema.svg") }}

A `:Wine` is from a province and a country, with the province itself belonging to a country. In addition, a `:Person` (who is a wine reviewer) tastes each wine and rates it for the review.

## Building the REST API

Once the data is ingested into Neo4j (see [part 1](https://thedataquarry.com/posts/neo4j-python-1/) regarding sync and async methods to ingest data via Python), we can proceed to build an API layer on top of the database. As is most common these days, [FastAPI](https://fastapi.tiangolo.com) is the framework used to build performance, async-friendly APIs in Python. Note that all code in the following sections is available [here](https://github.com/prrao87/neo4j-python-fastapi).

### Run the API and database via Docker

The `docker-compose.yml` [provided](https://github.com/prrao87/neo4j-python-fastapi) initiates two separate services that run in their own containers: one that runs Neo4j, and another one that serves a REST API (written in FastAPI) on top of the database.

```sh
docker compose up -d
```

This compose file starts a persistent-volume Neo4j database with credentials specified in `.env`. Both containers can communicate with one another with the common network that they share, on the exact port numbers specified in the environment file.

The services can be stopped at any time using the following command.

```sh
docker compose down
```

Opening `http://localhost:8000` should display a message as shown below on the browser.

```json
"message": "REST API for querying Neo4j database of 130k wine reviews from the Wine Enthusiast magazine"
```

We're all set to build out our Neo4j database connection!

### Create an async lifespan

FastAPI `0.95.0` introduced a cleaner, simpler interface to set up async interfaces to databases. This is done by using the `asynccontextmanager` decorator available as part of the `contextlib` standard library in Python.

{% codeblock(name="python") %}
```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None, None]:
    """Async context manager for Neo4j connection."""
    URI = "bolt://neo4j:7687"
    AUTH = ("neo4j", "password")
    async with AsyncGraphDatabase.driver(URI, auth=AUTH) as driver:
        async with driver.session(database="neo4j") as session:
            app.session = session
            print("Successfully connected to wine reviews Neo4j DB")
            yield
            print("Successfully closed wine reviews Neo4j connection")


app = FastAPI(lifespan=lifespan)

# Define routes below
# ...
```
{% end %}

Note that the URI for the Neo4j browser isn't `localhost` as is normally the case we specify `bolt://neo4j:7687` in this case. This basically says "Connect to the `neo4j` container using the Bolt protocol via port 7687". This is because, as per [part 1](https://thedataquarry.com/posts/neo4j-python-1/) of this series, we initiated the database via Docker. The [`docker-compose.yml`](https://github.com/prrao87/neo4j-python-fastapi/blob/main/docker-compose.yml) file shows how we composed two separate services: one for the Neo4j database, and another for the FastAPI server, both running in their own containers within a shared network `wine`. The container running the FastAPI service is named `neo4j`, which is why the URI is specified the way it is.


{% note(header="Note") %}
In earlier versions of FastAPI (prior to `v0.95.0`), it was common to define an event-based logic for managing database connections, using the `@app.on_event("startup")` and `@app.on_event("shutdown")` decorators [as described in the docs](https://fastapi.tiangolo.com/advanced/events/#startup-event). However, this has since been deprecated, and the `lifespan` object is the recommended method to manage DB connections in FastAPI, especially for asynchronous connections.
{% end %}

### Create routers

[As mentioned in the FastAPI docs](https://fastapi.tiangolo.com/tutorial/bigger-applications/), there are multiple ways to structure your application directories. There's no one "right" way to structure a large application with multiple end points, but, after some experimenting, I find that the structure shown below works really well for a variety of use cases, allowing for easy extensibility by myself or other developers.

```sh
└── src
    └── api
        ├── routers
        |    └── rest.py
        ├── schemas
        |    └── wine.py
        ├── __init__.py
        └── main.py
```

In FastAPI terminology, a "route" is a pathway to a set of endpoints that answer specific kinds of queries -- these are grouped together and stored under the `routers` directory. FastAPI relies on Pydantic for model validation, so, just like in the case with data ingestion, we specify a REST schema in the `schemas` directory.

The `rest.py` router file contains the following general layout.

```python
from fastapi import APIRouter, HTTPException, Query, Request
from neo4j import AsyncManagedTransaction
from src.schemas.response import ResponseModel

router = APIRouter()

# --- Routes ---

@router.get("/search", response_model=ResponseModel)
async def search_by_keywords(
    request: Request,
    terms: str,
    max_price: float = 100.0
) -> list[ResponseModel] | None:
    session = request.app.session
    result = await session.execute_read(_search_by_keywords, terms, max_price)
    if not result:
        raise HTTPException(
            status_code=404,
            detail=f"No wine with the provided terms '{terms}' found in database - please try again",
        )
    return result


# --- Neo4j query funcs ---

async def _search_by_keywords(
    tx: AsyncManagedTransaction,
    terms: str,
    price: float,
) -> list[FullTextSearch] | None:
    query = """
        CALL db.index.fulltext.queryNodes("searchText", $terms) YIELD node AS wine, score
        WITH DISTINCT wine, score
            MATCH (wine)-[:IS_FROM_COUNTRY]->(c:Country)
            WHERE wine.price <= $price
        RETURN
            c.countryName AS country,
            wine.wineID AS wineID,
            wine.points AS points,
            wine.title AS title,
            wine.description AS description,
            coalesce(wine.price, "Not available") AS price,
            wine.variety AS variety,
            wine.winery AS winery
        ORDER BY score DESC, points DESC LIMIT 5
    """
    response = await tx.run(query, terms=terms, price=price)
    result = await response.data()
    if result:
        return [FullTextSearch(**r) for r in result]
    return None
```

Note the clear separation between the _endpoint_ logic and the _query_ logic. The initial portion of the file `rest.py` contains the definition of the endpoint query parameters, and the read query is executed via the FastAPI request's database session object. Because FastAPI itself relies on Pydantic, a response model must be specified. For this example, the same directory that hosts the Pydantic schema for data ingestion is reused, but in practice, the schema directory from which the response model is called can reside at the same level as `main.py`.

The latter portion of `rest.py` contains the an example full text search query that queries the Neo4j database's full text index with the user's keyword terms.

More endpoints can be added this way, depending on the kinds of questions the end user may want to ask of the database.

### Attach endpoints in the router to `main.py`

We can then attach the endpoints in the router defined in `rest.py` to the FastAPI server in `main.py` below the lifespan object specification as follows:

```py

app = FastAPI(lifespan=lifespan)

# Attach routes
app.include_router(rest.router, prefix="/v1/rest", tags=["rest"])
```

As per the statement above, we serve the endpoints for the REST API with the route prefix `/v1/rest`, so the endpoint can be reached at the following URL:

```
https://localhost:8000/v1/rest/search
```

{% tip(header="Tip") %}
It's good practice to set a prefix to the router, for e.g., `/v1/rest` to indicate the version of the API and the fact that it's a REST API. The version number specifies to users and developers which version of the API they are calling (in case there are breaking changes in the future).
{% end %}

### Test endpoint

Pass a simple search query with the terms `tuscany red` with a max price of 50 to a cURL request as follows.

{% codeblock(name="bash") %}
```sh
curl -X 'GET' \
  'http://localhost:8000/v1/rest/search?terms=tuscany%20red&max_price=50'
```
{% end %}

The search terms and filter specified in the request are converted to a working Cypher query in the FastAPI router file. The query runs and retrieves results from a full text search index (that looks for these keywords in the title and variety of the sample).

```json
[
    {
        "wineID": 66393,
        "country": "Italy",
        "title": "Capezzana 1999 Ghiaie Della Furba Red (Tuscany)",
        "description": "Very much a baby, this is one big, bold, burly Cab-Merlot-Syrah blend that's filled to the brim with extracted plum fruit, bitter chocolate and earth. It takes a long time in the glass for it to lose its youthful, funky aromatics, and on the palate things are still a bit scattered. But in due time things will settle and integrate",
        "points": 90,
        "price": 49,
        "variety": "Red Blend",
        "winery": "Capezzana"
    },
    {
        "wineID": 40960,
        "country": "Italy",
        "title": "Fattoria di Grignano 2011 Pietramaggio Red (Toscana)",
        "description": "Here's a simple but well made red from Tuscany that has floral aromas of violet and rose with berry notes. The palate offers bright cherry, red currant and a touch of spice. Pair this with pasta dishes or grilled vegetables.",
        "points": 86,
        "price": 11,
        "variety": "Red Blend",
        "winery": "Fattoria di Grignano"
    },
    {
        "wineID": 73595,
        "country": "Italy",
        "title": "I Giusti e Zanza 2011 Belcore Red (Toscana)",
        "description": "With aromas of violet, tilled soil and red berries, this blend of Sangiovese and Merlot recalls sunny Tuscany. It's loaded with wild cherry flavors accented by white pepper, cinnamon and vanilla. The palate is uplifted by vibrant acidity and fine tannins.",
        "points": 89,
        "price": 27,
        "variety": "Red Blend",
        "winery": "I Giusti e Zanza"
    }
]
```

Not bad! This example correctly returns some highly rated Tuscan red wines along with their price and country of origin, Italy.

### Extend the API

With this design, the REST API can be easily extended to add more functionality and endpoints as needed. It's generally good practice to organize endpoints that are related to a particular functionality in a single file, and then reference the router by its name in `main.py`.

* The `schemas` directory houses the Pydantic schemas, both for the data input as well as for the endpoint outputs
  * As the data model gets more complex, we can add more files and separate the ingestion logic from the API logic here
* The `api/routers` directory contains the endpoint routes so that we can provide additional endpoint that answer more business questions
  * For e.g.: "What are the top rated wines from Argentina?"
  * In general, it makes sense to organize specific business use cases into their own router files
* The `api/main.py` file collects all the routes and schemas to run the API


### API docs

The great thing about FastAPI is that you get API docs for free, via the OpenAPI spec. With the Docker containers up and running, navigate to `localhost:8000/docs` to see the existing endpoints defined in this example.

{{ figure(src="api_docs.png") }}

## Conclusions

Building a FastAPI app on top of a Neo4j database can be a lot of fun, once you know the basics! Although a lot of the content in this post requires a bit of prior knowledge of how APIs work, the REST spec, etc., it's easy to appreciate why exposing the data in your graph database via a REST API makes a lot of sense. Developing using Docker also makes the entire process much more reproducible and easier to debug.

### Benefits of FastAPI and APIs in general

- **Ease of transmission**: The API acts as a bridge between the query layer (Cypher) and the user (front-end), so your end users don't need to know any Cypher to get the data they want!
- **Scalability**: As the number of users sending queries to the graph increases, frameworks like FastAPI efficiently handle async connections, best utilizing your CPU resources in a non-blocking manner
- **Security**: Rather than exposing the database to anybody out there, FastAPI provides [middleware options](https://fastapi.tiangolo.com/tutorial/middleware/) and [authentication methods](https://fastapi.tiangolo.com/tutorial/security/) to control who gets access to the data
- **Control**: The consumer of the data only sees what you, the backend engineer expose to them! For example, if we do not want to expose the name of a person to the consumer, we can set up the endpoint that way in the API layer upfront.

### A common downside to REST APIs

A common problem with RESTful APIs is that, in case a user wants a subset or a superset of the data exposed by an endpoint, they cannot change the request in a way that lets them fetch only what they want. As a result, it's common for RESTful endpoints to over-fetch data that the user doesn't need, or under-fetch data that they might need, and the only way to address this need is for the backend developer to define a new endpoint.

To address this issue, a different kind of API called [GraphQL](https://stablekernel.com/article/advantages-and-disadvantages-of-graphql/) was introduced. I'll plan on covering this in a future blog post.

## Code

All code for this post is available [on GitHub](https://github.com/prrao87/neo4j-python-fastapi).

## Additional learning

Here are some nice tutorials that helped me get up and running with Docker and FastAPI.

1. "Learn how to set up a great dev environment in Docker", Patrick Loeber, [YouTube](https://www.youtube.com/watch?v=0H2miBK_gAk&t=1s)
2. Intro tutorial by Sebastian Ramirez, [FastAPI docs](https://fastapi.tiangolo.com/lo/tutorial/)
