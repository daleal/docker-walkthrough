# Docker for Living Beings 🐳

This repository aims to be a _good enough_ place to start for using [Docker](https://docs.docker.com/) and [Compose](https://docs.docker.com/compose/). Bear in mind that this technology is **huge**, and, in my experience, impossible to learn without using it and failing several times. Without further ado, let's get started!

## Why Docker? 🤔

Docker is a technology that allows you, among other things, to run an application within a **controlled environment**. This means that you can run an application that needs `Node`, for example, **without having to install it on your local machine**. Essentially, you declare the environment in which the application will run inside a `Dockerfile` (we will talk about that a bit later) and forget about the current environment in your local machine. Then, you can use **the exact same environment** in **any** machine capable of running the Docker Engine. Cool, right? ✨

## Docker Concepts 🤓

First of all, we need to understand the _concepts_ involved in the usage of Docker. We will be focusing mainly on 5:

- **Dockerfile**: A special file that declares how to build a Container Image.
- **Container Image**: You can think of a Container Image as the _snapshot_ of an environment. The Container Image should contain everything that the application will need. For example, if the application needs to run using `PHP 7.2`, the Container Image should have that version installed. A Container Image can also contain other OS tools, such as `wget`, for instance.
- **Container**: A Container is **an instance** of a Container Image. This means that, while the Container Image is a _snapshot_ of the environment, the Container takes that _snapshot_ and _instanciates_ it into an isolated running environment. **Your application runs inside a Container**. You can instanciate multiple Containers from the same Container Image, if you want to do it, and starting a Container is usually really fast. **A Container does not have persistent storage capabilities**. This means that the environment (or application files) **cannot be changed**. Changes must be done to the Container Image and then the Container should be _re-instanciated_.
- **Volume**: Containers not having persistent storage capabilities generates a problem when trying to run applications that **need** to persist data, such as a database engine, for example. That's where Volumes come in. A Volume maps a folder inside the Container to a folder within the local machine. That way, changes made to that folder get stored in the local machine instead of trying to change the Container itself. This is useful for applications such as database engines, but is also useful for using Docker for developing, where you don't want to re-build your image every time you change a piece of code. Instead, you can mount a Volume mapping your local machine folder containing the code with the folder inside the Container Image where the code is located. Now, every code change will get reflacted inside the Container.
- **Network**: Containers run isolated inside the Docker Engine. But sometimes, you want one container to communicate with another one. For example, you may have a containerized Rails application that needs to communicate with a containerized database application. For that, you attach both containers into the same network and allow communication through that channel.

## Why Compose? 🤔

Maybe an idea of the technology and its usages is starting to appear in your head. The next natural question is, if Docker is so great, why do I need to use Compose with Docker? First of all, let's _pseudo-define_ what Compose is. Compose is an adjacent technology to Docker, that lets you, among other things, declare multiple services (like Containers, Volumes and Networks) inside a file and start them all at once. With that in mind, you can see why using Docker without Compose can be such a [_PITA_](https://www.urbandictionary.com/define.php?term=pita). Imagine having **very simple** application with only a containerized Django app and a containerized database engine. You should have to **manually** start the database Container, then a Volume to persist the data, then the Django Container and finally a Network for them to communicate. You should also have to remember the names given to each Docker Image (I haven't talked about that until now, just because we are not using it) and make sure to avoid all typos. That's why we use Compose 💖.

## Compose Concepts and Commands 🤓

Compose builds starting from the Docker base. Every concept explained within the Docker section will be used in this section. There is just one more concept and a few commands:

- **docker-compose.yml**: The Compose file. Here you declare all your services and its environmental varaibles and that kind of stuff.
- **docker-compose build**: This command builds every service's Container Image from the specified `Dockerfile`. Note that you can also pull Container Images from the web, and those Images won't get built (they will get pulled when using them for the first time, and then they will stay in the caché).
- **docker-compose up**: This command start every service declared within the `docker-compose.yml` file. You can pass the `--detach` flag to run in _daemon_ mode (start the Containers and return the control of the terminal to the user). You can also pass the `--build` flag to first build the Container Images and then start the Containers.
- **docker-compose down**: This command stops every service declared within the `docker-compose.yml`.
- **docker-compose run \<service> \<command>**: This command runs `<command>` inside the `<service>` Container. This is useful when you want, for example, to migrate the database using the containerized application. If you had Rails application named `application` in your `docker-compose.yml`, you could run `rails db:migrate` in that environment using the command `docker-compose run application rails db:migrate`.

## Some considerations 🔎

While declaring Networks using Compose is quite easy (as you will see in a bit), Compose is smart and, if you don't declare a Network, it creates a default Network for you services and attaches every Container to it. From within a container, you can reach other containers using a Network by refering to them with the following format:

```
<scheme>://<service-name>:<port>
```

That means that, for example, if you have a container with the service name `db` listening to the port `5432`, you can access that database from another container attached to the same Network as `db` by querying `http://db:5432/`.

Another thing to keep in mind is that the Compose command line takes in consideration **only** what's inside the `docker-compose.yml`. This seem obvious, but bare with me a little. Imagine that you've a `docker-compose.yml` file containing 2 services: `web` and `db`. You then run `docker-compose up --detach` and start your Containers. So far, so good. Now imagine that you erase the `web` service from the `docker-compose.yml`. Now, when running `docker-compose down`, **only the `db` Container will be stopped**. Compose has no way to remember that it started `web` alongside `db`, so you should keep that in mind.

## Examples?

Yes, I have rambled on about some concepts and commands, but I haven't shown **any** examples whatsoever. Don't worry! It's **finally** time. Go to the `example` and read the `README.md` file to guide you through the example.
