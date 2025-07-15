# ERP POC (Start here...):

## Introduction
This project is intended to showcase an real world application as part of my personal coding portfolio. 

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

| Month    | Savings |
| -------- | ------- |
| erp-core-node  | $250    |
| erp-api-entity-mgmnt-node | $80     |
| March    | $420    |


### Java

| Month    | Savings |
| -------- | ------- |
| January  | $250    |
| February | $80     |
| March    | $420    |

## Prequisites

## Node.js

Node v24.3.0 or greater is required.

## Java

JDK 20 si required

## MySQL, RabbitMQ

The application suite requires both MySQL and RabbitMQ instances to be running. For the purposes of streamining the environment, docker images have been created under the erp-docker-deps project. 

## Building the project

As of the time this document is published, the modules use local references between shared projects rely on file references. The facilitates rapid local development, however is not ideal for a long term production approach, namely because all projects need to be checked out an built in a certain order. 

Ideally we would be using published node modules which would either reside in the main node repo (for public distributions) or a private repo (which would be the case for private corporate scenarios). The reliance on local references is a strategic move, until a future point is reached where I will refactor the application to use npm published packages.

Keep in mind, the front end is stack agnostic, and only one API stak implementation may be ran at a time (they all share the same project specific port numbers, and each stack is independant of the others).

### Bulding the front end applications

1. Configure a proxy using the configuration below as an example
2. Compile the erp-core-node project via the command `yarn run build`
3. Compile the shared UI library (erp-core-react)
4. start the base UI module: `cd ./erp-ui-base/app`, `yarn run dev`
5. start the entity management module: `cd ./erp-ui-entity-mgmnt/app`, `yarn run dev`
6. start the work order management module: `cd ./erp-ui-work-order-mgmnt/app`, `yarn run dev`
7. start the inventory management module: `cd ./erp-ui-inv-mgmnt/app`, `yarn run dev`

### Starting the required application servers
A docker image is located under the 'erp-docker-deps' project. This image will need to be started prior to initializing any of the API projects.


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
        add_header X-Which-App "API base module (node)";
        proxy_pass http://localhost:5176/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_bypass $http_upgrade;
    }
    
    # Base API
    location /wo-mgmnt-api/ {
        add_header X-Which-App "API base module (node)";
        proxy_pass http://localhost:5177/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_cache_bypass $http_upgrade;
    }

    # Base API
    location /inv-mgmnt-api/ {
        add_header X-Which-App "API base module (node)";
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

## Local project setup

## Project dependencies

### Persistent store

### Proxy

### NodeJS

### Java 