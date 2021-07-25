## Production-Grade Workflow with Docker

1. Create a react app.
2. Create a file named `Dockerfile.dev` in the root directory.
3. Remove the `node_modules` folder from the project directory. (node_modules folder has a lot of libraries included which gets copied to the docker container. So we don't need it in out local machine. It will also build the container faster.)
4. Create a `docker-compose.yml` file in the root directory.
    ```yml
    version: "3"
    services:
        web: # Creating the container for hosting the web application
            build:
                context: . # Current directory where Dockerfile exists
                dockerfile: Dockerfile.dev # Dockerfile name. Used to override the dockerfile selection.
            ports:
                - "3000:3000"
            volumes:
                - /app/node_modules # Bookmarking the node_modules folder so that it does not get synced
                - .:/app # Sync all the files in current directory in local machine to files in app directory in the container.
            environment:
                - CHOKIDAR_USEPOLLING=true # For windows only, to make the files sync.
        tests: # Creating a container for running the tests
            build:
                context: .
                dockerfile: Dockerfile.dev
            volumes:
                - /app/node_modules
                - .:/app
            command: ["npm", "run", "test"] # This will execute the 'npm run test' command before starting the app
    ```
5. For production, we create a multi-step docker builds.
6. Create a `Dockerfile` in the root directory.

    ```Dockerfile
    FROM node:alpine as builder
    WORKDIR '/app'
    COPY package.json .
    RUN npm install
    COPY . .
    RUN npm run build

    FROM nginx
    COPY --from=builder /app/build /usr/share/nginx/html
    ```

7. In the Dockerfile, we are defining a Build phase and a Run phase.
8. In the Build phase, use the node:alpine base image and name it as `builder`. Inside this, build the app using `npm run build` command.
9. In the Run phase, use the nginx base image to run our app. Copy the build folder from the `builder` to the `/usr/share/nginx/html` directory inside the nginx container.
10. Now, build the image using `Dockerfile` and run it using the port mapping.
    ```
    docker build .
    docker run -p 8080:80 <Image_ID>
    ```
