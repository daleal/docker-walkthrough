# Example using Docker and Compose

This example is a _weird_ Rails application using `rails 6` and the [_asset pipeline_](https://guides.rubyonrails.org/asset_pipeline.html). Given that the important part of this example is the Docker related stuff, pay no mind to the outdated Rails application more than to test the Docker configuration. The relevant files are the following:

- `Dockerfile`
- `entrypoint.sh`
- `docker-compose.yml`
- `.env.example`

## Requirements

As you probably imagined, there are **only 2 requirements to run this application**:

- `Docker`
- `Compose`

And that's it! _Neat_, right? 💖

## Startup Workflow

I'll assume that your terminal is inside the `example` folder, on the root of the Rails app. To be able to run the application, the workflow will work something like this:

1. Create a `.env` file. This step is not always necessary, but, because we declared an `env_file` in our `docker-compose.yml`, we need to have a `.env` file in the root of the repository. Compose _automagically_ injects the variables inside the `.env` as environmental variables into the Container ✨. To copy the example `.env` file, run the following command:

    ```bash
    cp .env.example .env
    ```

2. Now that you have a `.env` in your root, you can _build_ the services. This step looks for every `Dockerfile` used within the `docker-compose.yml` and builds it into a Docker Image. To build the services, run the following command (**warning**: this might take a **long** time):

    ```bash
    docker-compose build
    ```

3. Now, you need to create the database. Notice that our `docker-compose.yml` has 2 services: `web` and `db`. As you might have guessed, the `db` service corresponds to a containerized PostgreSQL engine, and the `web` service corresponds to the Rails application. Also, notice that they are connected by a Network named `backend`, and that the `db` service has a volume that maps `data` (an _automagically_ generated folder inside your local machine, courtesy of Docker, created inside the `volumes` section) to `/var/lib/postgresql/data` inside the Container. That means that everything written to `/var/lib/postgresql/data` will, actually, get written in your local machine in a folder stored by Docker. To create the database, run the following command:

    ```bash
    docker-compose run web rails db:create
    ```

4. Now, all that is left is to start the application. To do that, all you need to do is run the following command:

    ```bash
    docker-compose up
    ```

    _Voilà_! You can access your application at `http://localhost:3000` 🐳

From now on, every time that a new gem gets installed, you should build the Docker Image again, to add that gem to the Image.

## Tips and tricks

Docker uses _cache_ to speed up builds! The way that it works is that every Docker command gets built into a layer on top of the previous layer. Then, every time that a layer changes, that layer until the end of the `Dockerfile` gets built again. Because of that, you can notice that, in the sample `Dockerfile`, the first things that are run are the ones that are least likely to change (like installing OS dependencies, which tend to be more stable than the application code, for example). Then, the `Node` dependencies are installed (you probably won't touch that any time soon). Then, the gems get installed. You will sometimes add a new gem to your `Gemfile`, and you should build right after that. Lastly, you copy all the files into the Container Image. This step is the most likely to change **all the time**. If you run this step as the first step, you would have to re-run **the whole build process every time you built the Container Images**.

Note that the compose contains various declarations with the form `{x}:{y}`. This is used commonly for Volumes (for example, `.:/usr/src/app`) and for port publication (for example, `"3000:3000"`). It is important to understand that on `{x}:{y}` declarations, `{x}` is the local machine information and `{y}` is the Container information. For example, when declaring the Volume `.:/usr/src/app`, that means that the folder `.` of the local machine will get mapped to the folder `/usr/src/app` of the Container. When declaring the ports, if the declaration was `"80:3000"`, for example, that would mean that all the traffic going through the port `80` of the local machine should get mapped to the port `3000` of the Container.

You are probably wondering, what the 🐳 is an entrypoint? Simply put, it is a script that runs every time that the Container starts, just before the main process. It is useful to run things like removing a dangling PID, for example. Most of the time, it will not be used for much more that that, though.
