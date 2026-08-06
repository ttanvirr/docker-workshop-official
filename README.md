# Table of Contents

1. [Overview](#overview-of-the-docker-workshop)
2. [Part 1: Containerize an application](#part-1-containarize-an-application)
3. [Part 2: Update the application](#part-2-update-the-application)
4. [Part 3: Share the application](#part-3-share-the-application)
5. [Part 4: Persist the DB](#part-4-persist-the-db)
   - [Persist the todo data](#persist-the-todo-data)

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

# Part 4: Persist the DB

In case you didn't notice, your todo list is empty every single time you launch the container. Why is this? In this part, you'll dive into how the container is working.

## The container's filesystem

When a container runs, it uses the various layers from an image for its filesystem. Each container also gets its own "scratch space" to create/update/remove files. Any changes WILL NOT BE SEEN in another container, even if they're using the same image.

### See this in practice

To see this in action, you're going to start two containers. In one container, you'll create a file. In the other container, you'll check whether that same file exists.

1. Start an Alpine container and create a new file in it.

   ```bash
   $ docker run --rm alpine touch greeting.txt
   ```

   > [!TIP]
   > Any commands you specify after the image name (alpine) are executed inside the container. In this case, the command `touch greeting.txt` puts a file named `greeting.txt` on the container's filesystem.
   - The `--rm` flag removes the container after running and creating the file.

2. Run a new Alpine container and use the `stat` command to check whether the file exists.

   ```bash
   $ docker run --rm alpine stat greeting.txt
   ```

   You should see output similar to the following that indicates the file does not exist in the new container.

   ```
   stat: can't stat 'greeting.txt': No such file or directory
   ```

The `greeting.txt` file created by the first container did not exist in the second container. That is because the writeable "top layer" of each container is isolated. Even though both containers shared the same underlying layers that make up the base image, the writable layer is unique to each container.

## Container volumes

With the previous experiment, you saw that each container starts from the image definition each time it starts. While containers can create, update, and delete files, those changes are lost when you remove the container. With volumes, you can change all of this.

**Volumes** provide the ability to connect specific filesystem paths of the container back to the host machine. If you mount a directory in the container, changes in that directory are also seen on the host machine. If you mount that same directory across container restarts, you'd see the same files.

There are two main types of volumes. You'll eventually use both, but you'll start with 'volume mounts'.

## Persist the todo data

By default, the todo app stores its data in a SQLite database at `/etc/todos/todo.db` in the container's filesystem. SQLite is simply a relational database that stores all the data in a single file. While this isn't the best for large-scale applications, it works for small demos. You'll learn how to switch this to a different database engine later.

With the database being a single file, if you can persist that file on the host and make it available to the next container, it should be able to pick up where the last one left off. By creating a volume and attaching (often called "mounting") it to the directory where you stored the data, you can persist the data.

As mentioned, you're going to use a **volume mount**. Think of a volume mount as an opaque bucket of data. Docker fully manages the volume, including the storage location on disk. You only need to remember the name of the volume.

### Create a volume and start the container

You can create the volume and start the container using the CLI or Docker Desktop's graphical interface.

#### Method 1: create volume using CLI

<hr style="height:1px; margin-top:0" />

1. Create a volume:

   ```bash
   $ docker volume create todo-db
   ```

2. Stop and remove the todo app container:

   ```bash
   docker rm -f <container_id>
   ```

3. Start the todo app container, but add the `--mount` option to specify a volume mount. Use the volume named `todo-db`, and mount it to `/etc/todos` in the container, which captures all files created at the path.

```bash
$ docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

#### Method 2: create volume using Docker Desktop

<hr style="height:1px; margin-top:0" />

To create a volume:

1. Select **Volumes** in Docker Desktop.
2. In Volumes, select **Create a volume**.
3. Specify `todo-db` as the volume name, and then select **Create**.

To stop and remove the app container:

1. Select **Containers** in Docker Desktop.
2. Select **Delete** in the Actions column for the container.

To start the todo app container with the volume mounted:

1. Select the search box at the top of Docker Desktop.
2. In the search box, search for the image, `getting-started`.

   > [!TIP]
   > Use the search filter to filter images and only show **Local images**.

3. Select your image and then select **Run**.
4. Select **Optional settings**.
5. In **Host port**, specify the port, for example, `3000`.
6. In **Host path**, specify the name of the volume, `todo-db`.
7. In Container path, specify `/etc/todos`.
8. Select **Run**.

### Verify that the data persists

1. Once the container starts up, open the app and add a few items to your todo list.
   ![Todo list](doc-images/todo-list-1.png)

2. Stop and remove the container for the todo app using previous steps
3. Start a new container using the previous steps.
4. Open the app in browser. You should see your items still in your list.
5. Go ahead and remove the container when you're done checking out your list.

You've now learned how to persist data.

## Dive into the volume

A lot of people frequently ask "Where is Docker storing my data when I use a volume?" If you want to know, you can use the `docker volume inspect` command.

```bash
$ docker volume inspect todo-db
```

You should see output like the following:

```
[
    {
        "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

The `Mountpoint` is the actual location of the data on the disk. Note that on most machines, you will need to have root access to access this directory from the host.

## Summary

In this section, you learned how to persist container data.
