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
