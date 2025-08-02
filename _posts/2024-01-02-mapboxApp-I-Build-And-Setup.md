---
title: Mapbox App I [Log] - Build And Setup
date: 2024-01-02 00:00:00 +0800
categories: [Mapbox, MapboxApp]
tags: [mapbox, react, docker, app]
# TAG names should always be lowercase
---
Update Date: 2024-01-03
## 1. Introduction
In this series, I will try to write developing log about using mapbox to cooperate the location and photo information together into an application. This is an experimental application and I have totoally no experience about developing an app, so just try and error.

Also, I read the document from mapbox and takes some notes [here](https://neko2048.github.io/posts/mapboxApp-II/)

In this first blog, I will need to set up the developing environment. The developing environment is MacOS.
The following packages are required:
* NPM & NVM (what is the purpose?)
* Docker

## 2. Name your project
According to the mapbox documents[^fn1], let's name your project name like `use-mapbox-gl-js-with-react`. 

And the file structure should be
```
use-mapbox-gl-js-with-react/
    package.json
    public/
        index.html
    src/
        App.js
        index.css
        index.js
```
and follow the instructions to the step before `npm run`.
We will study the code after.

## 3. Installing NPM and NVM
Since I have already install the NPM earlier due to the jekyll blog, so I just skip this section.
You can install nvm according to the tutorial [^fn2].

By the way, after my test and other reports, the NPM version 18 would create error like the following post [^fn3]:
> [Error message "error:0308010C:digital envelope routines::unsupported"](https://stackoverflow.com/questions/69692842/error-message-error0308010cdigital-envelope-routinesunsupported)

There are two ways to solve it according to what I tested and searched:
1. Enable legacy OpenSSL Provider:
 ```bash
 export NODE_OPTIONS=--openssl-legacy-provider
```
or in the `package.json` of react app:
```json
"scripts": {
        "start": "react-scripts --openssl-legacy-provider start",
        "build": "react-scripts build"
    }
```

2. Downgrade to Node.js version 16:
```bash
nvm ls 
# to show the installed versions locally
nvm install 16.20.2
# install the version you specify
nvm use 16.20.2
# use the version you specify
```
or specfiy it in Dockerfile[^fn4]:
```Dockerfile
FROM node:16
```
This will be done in [section 4](#1-write-the-dockerfile)

## 4. Docker
It's might be good to develop using docker, the reference [^fn3] is taken and the file structure I have:
```
mapBoxReact/
    Dockerfile
    docker-compose.yml
    app/
        use-mapbox-gl-js-with-react/
        ... (see #name-your-project)
```
The building steps are as follow:
#### 1. Write the `Dockerfile`
```Dockerfile
FROM node:16 
ADD ./app /app
WORKDIR /app
```
Note that `FROM node:16` specify the version of Node.js [^fn4].
#### 2. Write the `docker-compose-yml`
```yml
version: '3.8'
services:
  node:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./app:/app
    command: sh -c "cd use-mapbox-gl-js-with-react && npm start"
    ports:
      - "3000:3000"
    stdin_open: true
```
> `cd use-mapbox-gl-js-with-react` is changing directory to where `package.json` is

Be sure that at the folder having both `Dockerfile` and `docker-compose.yml`

#### 3. Build Docker image
```bash
docker-compose build
```
The image should be generated and you can checkt it by
```bash
docker images
```

#### 4. Install Node Packages
The following command line would install the packages you need according to the `package.json` and generate `package-lock.json` at the same folder of `package.json`
```bash
docker-compose run --rm node sh -c "cd use-mapbox-gl-js-with-react && npm install -g"
```
> it's now at `app/` due to `Dockerfile` and `cd use-mapbox-gl-js-with-react` is to execute `npm install ` at the folder of `package.json`.<br />
`-g` is installing globally.

#### 5. Activate Docker Image
```bash
docker-compose up
```

#### 6. Preview the results
You can just see the results by browser with [http://localhost:3000](http://localhost:3000)

## Reference
[^fn1]: https://docs.mapbox.com/help/tutorials/use-mapbox-gl-js-with-react/#set-up-the-react-app-structure

[^fn2]: https://www.casper.tw/development/2022/01/10/install-nvm/

[^fn3]: https://stackoverflow.com/questions/69692842/error-message-error0308010cdigital-envelope-routinesunsupported

[^fn4]: https://github.com/nodejs/docker-node#create-a-dockerfile-in-your-nodejs-app-project

[^fn5]: https://learningsky.io/use-docker-to-create-react-development-environment/
