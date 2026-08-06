# Table of Contents

1. [Overview](#overview-of-the-docker-workshop)
2. [Part 1: Containerize an application](#part-1-containarize-an-application)
3. [Part 2: Update the application](#part-2-update-the-application)
4. [Part 3: Share the application](#part-3-share-the-application)

# Overview of the Docker workshop

This workshop contains step-by-step instructions on how to get started with Docker. We'll be working with a simple 'todo list manager' that runs on `Node.js`. This workshop shows you how to:

- Build and run an image as a container.
- Share images using Docker Hub.
- Deploy Docker applications using multiple containers with a database.
- Run applications using Docker Compose.

# Part 1: Containarize an application

## Get the app

1. Clone the starter repo from: https://github.com/docker/getting-started-app.git.
2. remove the existing `.git` folder if you want.
3. You should see the following files and sub-directories:

```
├── getting-started-app/
│ ├── .dockerignore
│ ├── package.json
│ ├── package-lock.json
│ ├── README.md
│ ├── spec/
│ ├── src/
```

## Build the app's image

To build the image, you'll need to use a `Dockerfile`. A **Dockerfile** is simply a text-based file with no file extension that contains a script of instructions. Docker uses this script to build a container image.

1. Create Dockerfile

In the `getting-started-app` directory, the same location as the `package.json` file, create a file named `Dockerfile` with the following contents:

```dockerfile
# syntax=docker/dockerfile:1

FROM node:24-alpine
WORKDIR /app
COPY . .
RUN npm install --omit=dev
CMD ["node", "src/index.js"]
EXPOSE 3000
```

This Dockerfile does the following:

- Uses `node:24-alpine` as the base image, a lightweight Linux image with `Node.js` pre-installed
- Sets `/app` as the working directory (in image)
- Copies source code into the image (into `/app`)
- Installs the necessary dependencies
- Specifies the command to start the application
- Documents that the app listens on port 3000

2. Build the image

In the terminal, make sure you're in the `getting-started-app` directory:

```bash
$ cd /path/to/getting-started-app
```

Build the image:

```bash
$ docker build -t getting-started .
```

The `docker build` command uses the `Dockerfile` to build a new image. You might have noticed that Docker downloaded a lot of "layers". This is because you instructed the builder that you wanted to start from the `node:24-alpine` image. But, since you didn't have that on your machine, Docker needed to download the image.

After Docker downloaded the image, the instructions from the Dockerfile copied in your application and used npm to install your application's dependencies.

Finally, the `-t` flag tags your image. Think of this as a human-readable name for the final image. You can refer to that image name when you run a container.

The `.` at the end of the docker build command tells Docker that it should look for the `Dockerfile` in the current directory.

## Start an app container

1. Run your container using the `docker run` command and specify the name of the image you just created:

```bash
$ docker run -d -p 127.0.0.1:3000:3000 getting-started
```

- The `-d` flag (short for `--detach`) runs the container in the background. This means that Docker starts your container and returns you to the terminal prompt. Also, it does not display logs in the terminal.

- The `-p` flag (short for `--publish`) creates a port mapping between the host and the container. The `-p` flag takes a string value in the format of `HOST:CONTAINER`, where `HOST` is the address on the host, and `CONTAINER` is the port on the container. The command publishes the container's port `3000` to `127.0.0.1:3000` (`localhost:3000`) on the host. Without the port mapping, you wouldn't be able to access the application from the host.

2. After a few seconds, open your web browser to http://localhost:3000. You should see your app.

3. Add an item or two and see that it works as you expect. You can mark items as complete and remove them. Your frontend is successfully storing items in the backend.

At this point, you have a running `todo list manager` with a few items.

If you take a quick look at your containers on Docker Desktop (or using `docker ps` command), you should see at least one container running that's using the `getting-started` image and on port `3000`.

## Summary

In this section, you learned the basics about creating a `Dockerfile` to build an image. Then you started a container and saw the running app.

# Part 2: Update the application

In this part, you'll update the application and image. You'll also learn how to stop and remove a container.

## Update the source code

In the following steps, you'll change the "empty text" when you don't have any todo list items to "You have no todo items yet! Add one above!"

1. In the `src/static/js/app.js` file, update line 56 (may differ) to use the new empty text.

```
- <p className="text-center">No items yet! Add one above!</p>
+ <p className="text-center">You have no todo items yet! Add one above!</p>
```

2. Build your updated version of the image, using the `docker build` command.

```bash
$ docker build -t getting-started .
```

3. Start a new container using the updated code.

```bash
$ docker run -dp 127.0.0.1:3000:3000 getting-started
```

You probably saw an error like this:

```
docker: Error response from daemon:......: Bind for 127.0.0.1:3000 failed: port is already allocated.
```

The reason is that the old container is already using the host's port `3000` and only one process on the machine (containers included) can listen to a specific port. To fix this, you need to remove the old container.

## Remove the old container

To remove a container, you first need to stop it. Then, you can remove it.

You can remove the old container using the CLI or Docker Desktop's graphical interface.

### Method-1: Remove a container using the CLI

1. Get the ID of the container by using the `docker ps -a` command.

2. Use the `docker stop` command to stop the container.

```bash
$ docker stop <the-container-id>
```

3. Once the container has stopped, you can remove it by using the `docker rm` command.

```bash
$ docker rm <the-container-id>
```

> [!NOTE]
> You can stop and remove a container in a single command by adding the force flag to the `docker rm` command. For example: `docker rm -f <the-container-id>`

### Method-2: Remove a container using Docker Desktop

1. Open Docker Desktop to the `Containers` view.
2. Select the trash can icon under the **Actions** column for the container that you want to delete.
3. In the confirmation dialog, select **Delete forever**.

## Start the updated app container

1. Now, start your updated app using the `docker run` command.

```bash
$ docker run -dp 127.0.0.1:3000:3000 getting-started
```

2. Refresh your browser on http://localhost:3000 and you should see your updated help text.

## Summary

In this section, you learned how to update and rebuild an image, as well as how to stop and remove a container.

# Part 3: Share the application

Now that you've built an image, you can share it. To share Docker images, you have to use a Docker registry. The default registry is Docker Hub and is where all of the images you've used have come from.

## Create a repository

To push an image, you first need to create a repository on Docker Hub.

1. Sign up or Sign in to [Docker Hub](#https://hub.docker.com/).
2. Select the `Create Repository` button.
3. For the repository name, use `getting-started`. Make sure the `Visibility` is `Public`.
4. Select `Create`.

## Push the image

Let's try to push the image to Docker Hub.

1. In the command line, you should run (match the repository name in Docker Hub):

- Replace the username with your Docker hub username.

```bash
$ docker push YOUR-USER-NAME/getting-started
```

But you'll see an error like this:

```
Using default tag: latest
The push refers to repository [docker.io/ttanvirr/getting-started]
An image does not exist locally with the tag: ttanvirr/getting-started
```

This failure is expected because Docker is looking for an image name `ttanvirr/getting-started`, but your local image is still named `getting-started`.

You can confirm this by running: `docker image ls`

2. To fix this, use the `docker tag` command to give the `getting-started` image a new name. Replace `YOUR-USER-NAME` with your Docker ID.

```bash
$ docker tag getting-started YOUR-USER-NAME/getting-started
```

3. Now run the `docker push` command again. If you don't specify a `:tag`, Docker uses a tag called `latest`.

```bash
$ docker push YOUR-USER-NAME/getting-started
```

## Run the image on a new instance

Now that your image has been built and pushed into a registry, you can run your app on any machine that has Docker installed. Try pulling and running your image on another computer or a cloud instance.

## Summary

In this section, you learned how to share your images by pushing them to a registry.
