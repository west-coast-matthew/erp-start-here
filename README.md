# ERP POC (Start here...):

## Introduction
I wanted to share my personal experience with building a 'real world' application. Dual purpose here, show off my abilities, have a little fun.

The application consists of a series of front end and back end projects. The front end is an Typescript React implementation, and the backend consists of a series of primarily REST base APIs, grouped into projects based on functional area.

The overall to showcase an real world application as part of my personal coding portfolio. 

To make things interesting, the API implementations have been created in both Java/Spring and NodeJS (Express,Typescript,TypeORM) stacks. This helps to illustrate a lot of the common design concepts that may be applied to both tech snacks.

## Architecture
The overall application has been decomposed into a series of individual projects. An relational database is used to persist the data, and an messaging bus is utilized in order to facilitate asnych operations where appropriate. 

## Project modules

The 'entire' application consists of a series of projects. Basically projects are grouped into front end, backend, and shared modules. The UI projects follow a naming convention where a 'erp-ui' prefix is used. Shared modules utilize an 'erp-core' prefix, and all API projects follow the 'erp-api' prefix. 

The UI components are implemented in React. They agnostic of the tech stack as they basically interact with REST APIs. As for the API implementations, at this point in time, API implementations are provided in both Java and Node.js stacks. Running the application requires all UI projects to be started, and then depending upon the desired API stack, all related projects need to be started for the desired implementation. 

### React/Frontend
| Project    | Description |
| -------- | ------- |
| erp-ui-entity-mgmnt  | Dictionary object management    |
| erp-ui-base | Dashboard and preference related screens     |
| erp-core-react    | Common elements shared by UI projects (i.e. Menu    |


### Node.js

| Project    | Description |
| -------- | ------- |
| erp-core-node  | common shared library    |
| erp-api-entity-mgmnt-node | basic entity management functions     |
| erp-api-wo-mgmnt-node | work order centric operations    |
| erp-api-inv-mgmnt-node | inventory related operations, supports messaging bus in addition to exposing REST APIs     |


### Java

| Month    | Savings |
| -------- | ------- |
| erp-core-node  | common shared library    |
| erp-api-entity-mgmnt-node | basic entity management functions     |
| erp-api-wo-mgmnt-node | work order centric operations    |
| erp-api-inv-mgmnt-node | inventory related operations, supports messaging bus in addition to exposing REST APIs     |


## Prequisites

## Node.js

Node v24.3.0 or greater is required.

## Java

JDK 20 is required

## MySQL, RabbitMQ

The application suite requires both MySQL and RabbitMQ instances to be running. For the purposes of streamining the environment, docker images have been created under the erp-docker-deps project. 

## Building the project

As of the time this document is published, the modules use 'yarn link' to establish dependendcies. The facilitates rapid local development, however is not ideal for a long term production approach, namely because all projects need to be checked out an built in a certain order. 

Ideally we would be using published node modules which would either reside in the main node repo (for public distributions) or a private repo (which would be the case for private corporate scenarios). The reliance on local references is a strategic move, until a future point is reached where I will refactor the application to use npm published packages.

Keep in mind, the front end is stack agnostic, and only one API stak implementation may be ran at a time (they all share the same project specific port numbers, and each stack is independant of the others).

Yarn (https://yarnpkg.com/) is the preferred package manager for building UI and node based modules. Apache Maven is used for all Java cetric source.

It's worth noting that a monrepo approach was not used as the idea was to potentialy allow modules to have dependencies on different versions of other projects. The goal is to support a deployment model where promotions are performed more of an isolated manner, allowing flexibility in deployments, and minimizing QA/testing efforts.

### Pre-flight check
The following ports will need to be available
80 - For Http proxy
5173, 5174, 5179, 5180- For UI implementtions
5175, 5176, 5177, 5178 - For API implementations
3306 - For database instance
5672 - For messaging bus

These are all 'default' ports, everything is configurable should the requirement arise.

### Starting the required application servers
A docker image is located under the 'erp-docker-deps' project. This image will need to be started prior to initializing any of the API projects. This will expose an Mariadb and Rabbit MQ instance.

### Bulding the front end applications

1. Configure a proxy using the configuration below as an example
2. Compile the erp-core-node project via the command `yarn run build`
3. Compile the shared UI library (erp-core-react)
4. Execute the command 'yarn link' command within the shared module.
5. From within each of the UI applications, link to the shared module via the command 'yarn link @west-coast-matthew/erp-core-node'. Keep in mind, each UI project lives under the 'app' sub folder of the parent app. 
5. start the base UI module: `cd ./erp-ui-base/app`, `yarn run dev`
6. start the entity management module: `cd ./erp-ui-entity-mgmnt/app`, `yarn run dev`
7. start the work order management module: `cd ./erp-ui-work-order-mgmnt/app`, `yarn run dev`
8. start the inventory management module: `cd ./erp-ui-inv-mgmnt/app`, `yarn run dev`



### Building Node.js API implementations
1. copy the '.env.example' file to new instance named `.env` in the projects [TODO: insert project names here]. The projects use the dotenv project to store authentication information for the relational store and messaging engine.
2. build the shared core library if this was not performed in the previous step. `cd erp-core-node`, `yarn run dev`
3. build the shared ui library under the 'erp-core-react' project if not previously performed.
4. build the entity management project: `cd erp-api-entity-mgmnt-node`, `yarn run dev`
5. build the inventory management project: `cd erp-api-inv-mgmnt-node`, `yarn run dev`
6. build the work order management project: `cd erp-api-wo-mgmnt-node`, `yarn run dev`
7. buld the base/dashboard application project: `cd erp-api-base-node`, `yarn run dev`

### Building Java API implementations
[todo: document]

## Proxy configuration (Nginx)

The applications require a proxy in front of the UI and API modules. Port numbers are assigned to each of the individual projects. For local development purpposes, I use Nginx for this role, and the configuration is as follows, which may be used as a model for other proxy implementations.

This approach meets the needs for local development purposes, however for true production deployments, it might be appropriate to spawn up multiple instances of a given API instance for scalability purposes. Be aware that when scaling, a single instance of the API used to perform composition calculations should be used, and calculation requests need to be processed in a FIFO manner.

```

server {
    listen 80;
    server_name localhost;

    ################################################################
    # UI modules
    #
    # proxy_pass directive here does not include the trailing '/'
    # char, indicating that the prefix will be passed to the 
    # application instance.
    #
    # The corresponding vite.config.ts needs to reflect the corresponding
    # value for the prefix, which routes all behind the scenes fetching
    # performed via vite to use the same url suffix, ensuring the proxy
    # will route all of these reqests to the corresponding 
    #
    ################################################################

    # Entity management UI
    location /entity-mgmnt/ {
        # Do NOT rewrite here — let Vite handle full base path
        add_header X-Which-App "Entity management module";
        proxy_pass http://localhost:5174;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_bypass $http_upgrade;
    }

    # Default module UI (ui base)
    location /main/ {
        add_header X-Which-App "Main application module";
        proxy_pass http://localhost:5173;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_bypass $http_upgrade;
    }

    ################################################################
    # API modules
    #
    # Note: Naming convention here uses a trailing '/' character in 
    # the proxy_pass directive, essentially dropping the location
    # prefix when the API request is recieved.
    #
    ################################################################

    # Base API
    location /api-base/ {
        add_header X-Which-App "API base module (node)";
        proxy_pass http://localhost:5175/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_bypass $http_upgrade;
    }

    # Base API
    location /entity-mgmnt-api/ {
        add_header X-Which-App "API entity mgmnt module (node)";
        proxy_pass http://localhost:5176/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_bypass $http_upgrade;
    }

    # Base API
    location /wo-mgmnt-api/ {
        add_header X-Which-App "API wo mgmnt module (node)";
        proxy_pass http://localhost:5177/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_bypass $http_upgrade;
    }

    # Base API
    location /inv-mgmnt-api/ {
        add_header X-Which-App "API in mgmnt module (node)";
        proxy_pass http://localhost:5178/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_bypass $http_upgrade;
    }

}


```

## The domain problem

In order to present an inteteresting problem space (the world does not need another shopping cart or twitter clone) the application is based off of the beverage manufacturing space, and area where I have personal experience. This allows me to introduce business logic, as I wanted to reach beyond a simple CRUD application. In order to protect IP from previous employers, I am focusing on a business that is focused on making orange juice. 

The concept of a large set of complex business logic is something I wanted to illustrate here. This is often overlooked in projects and tutorials. I am including 'just enough' to illustrate design decisions, but definately fall short of a full production level set. In past experiences, I have implemented *hundreds* of pages of conditional sets of logic, which presents a few unique challenges. 

In addition to data entry operations, an ansychrous aspect has been introduced for calculating composition related operations.

In an attempt to go beyond basic data entry operations, a creative approach to visualizing inventory levels has been introduced.

The following is a high level explanation of the overall domain problem, however a detailed version of the domain space can be located [here](./problem-domain.md).

### An overview
Imagine a business responsible for manufacturing orange juice. Bascially oranges are purchased from various sources, the juice is extracted via a series of operations (extraction/pressing) and initially stored into a series of bulk storage tanks. An initial set of operations is performed to pasturize and filter unwanted debree from the initial juice yield. From that point, a series of additional steps are performed during the manufacturing process, until the product reaches it's final state (consumer packaged).

A key take away is that there are a few steps in the initial process to get orange juice, things are stored in bulk (like really large quantities), numerous operations are performed against the intial extracted juice based on planned end consumer packaging, and the entire process is executed in an 'on demand' manner.

High level requirements/concepts to be famillair with:

* Track expected juice yields from raw materials vs actual results.
* Create a details record of operation performed from the product base up until the final consumer ready packaging. 
* Calculate detailed metrics on the state of product base after every stage in the manufacturing process (referred to as 'composition'
* Support the ability to historically 'trace' activity in the manufacturing process, so in the event of a product recall we can identify all impacted consumer product runs and act accordingly.
* Expose high level business events to potential external systems for integration purposes.

Notes on integration with other systems... In a lot of cases, data needs to be 'shared' with other systems. ETL extraction processes are a common occurance. Personally, I am not a big fan of this approach as this exposes topics such as requirements around data latency, avoiding redundant data transfers, data transformations, etc. Additionally, consuming systems are tightly coupled and exposed to data store specifics and structure. Upgrading to other versions or potentially other RDBMs systems are required to be performed. My personal preference is to broadcase business events onto a messaging bus, abstracting the core application from consumers, and supporting the ability for consuming systems to digest 'delta' level data updates. 





 