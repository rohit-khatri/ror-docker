
# ROR Containerization

In this, you will learn how to dockerize a Ruby on Rails application from the ground up.






## Prerequisites

  Install Docker and Docker Compose. It's just download and open the installer, register an account on dockerhub, and you're good to go. Check out Docker installation docs.

- [Docker](https://docs.docker.com/get-docker)
- [Docker Compose](https://docs.docker.com/compose/)
- [Git](https://git-scm.com/downloads)


## Setup

  1. Create a directory
  2. Make a .dockerignore file
      > Dockerignore file just tells Docker which files to ignore in it's container. An example is when you have a minified assets, js, css files, that gets changed from time to time whenever you change the original code.
  3. Make a .gitignore file
      > A gitignore file specifies intentionally untracked files that Git should ignore. Files already tracked by Git are not affected.
  4. Dockerfile and docker-compose file
      > This is where most of the operation happens. Think of these two files as set of instructions Docker follows on how to setup your virtual container. Dockerfile and docker-compose file works hand in hand. You can have multiple Dockerfiles for different services, and one docker-compose file to tie them up together.

      - Make a file named Dockerfile
          > A Dockerfile is a file with a set of rules you'll set that Docker will follow. There is a pre-built set of rules found on the Docker hub. After making your Dockerfile, put the below code on your Dockerfile.

          ```
          FROM ruby

          WORKDIR /home/app

          ENV PORT 3000

          EXPOSE $PORT

          RUN gem install rails bundler
          RUN gem install rails
          RUN apt-get update -qq && apt-get install -y nodejs

          ENTRYPOINT [ "/bin/bash" ]
          ```

          > FROM ruby - this means docker will pull a pre-built setup by ruby. You don't need to think about updating or installing on your machine the latest ruby version. You'll see the list of Docker's pre-built images on their Dockerhub. Think of it as like npm.

          > WORKDIR /home/app - Work directory. Work directory means this is be your default folder location when you start your development environment. You can name it whatever you want.

          > ENV PORT 3000 - Environment variable. This will set a variable named $PORT on your bash terminal to be '3000'.

          > EXPOSE $PORT - expose port 3000 (that we've set earlier) of the virtual container to your local machine.

          > RUN - Run commands are some setup instructions that you want the terminal to run before you use it. In our case, we installed ruby on rails, bundler, and node.js before we even use the development environment so it's all ready when we use it.

          > ENTRYPOINT ["/bin/bash"]- this command tells docker what command to execute when we run the container. In our case, we need to run bash terminal so we can have access to rails.

      - Make a file named docker-compose.yml
          > Docker compose file is a file that ties up different services together. A good example is when you're wiring up your rails app to different servers like Postgres database server, or redis caching server. You can easily make them work with this file.

          Put this in your docker-compose.yml file.

          ```
          version: "3.7"

          services:
            ruby_dev:
              build: .
              container_name: ruby_container
              ports:
                - "3000:3000"
              volumes:
                - ./:/home/app
          ```
          As you can see, it kinda looks like the Dockerfile, but with a little bit of indentation. Let's go through the lines.

          > version - Through time, docker-compose file went through changes. That's why in docker-compose files, they need to specify which version they are using. Update version according to your doccker-compose installation.

          > services - Specify list of services. You can have many services like a rails server, and a Postgres server on your project. You can name your services any name you want. Here, I named it ruby_dev.

          > build: . - The dot here means a file path where to find the Dockerfile, which is the build instructions.

          > container_name - The name of the container.

          > ports: - these are the ports to expose from the docker container to our host local machine. The pattern here is HOST:CONTAINER. In our case it's "3000:3000". Which means we are letting the default Rails server port (3000) be available in our local machine's "localhost:3000".

          > volumes: - volume means even if we quit or delete Docker, we can specify which files we can keep in our local machine. We put ./:/home/app there because we named in our Dockerfile earlier the workdir to be /home/app.

## Building and running the container
With all config files setup, let's build and run the container.

- Open terminal and go to the directory in wihich *Dockerfile* and *docker-compose.yml* files placed

- Execute `docker-compose build .`
  > This command will get Dockerfile and install all the necessary things to make a rails development environment. It may take some time so wait untill finish.

- Once above command execution complete the execute `docker-compose run --rm --service-ports ruby_dev`.
  > This command will start a bash terminal that will be rails development environment where the rails commands are available. Notice that our command has some flags, --rm means remove the container after using it, and --service-ports means use port 3000 in our container so we can see our rails server in action. The name ruby_dev also came from services found at our docker-compose.yml.


## Test-run a rails app
Now that we've successfully made our rails development environment, we'll test a sample rails app.

- Execute in bash `rails new myapp && cd myapp`
  > This command will create a new rails app in a folder named myapp. After that the terminal will go the folder. You can name it whatever you want.

- Update and install gems. Execute `bundle update && bundle install`
  > Just make sure you're in the right folder, in myapp, which contains the rails app files. This command will update and install your dependencies.

- Test the server by running `rails server -p $PORT -b 0.0.0.0`
  > Remember the port we specified in our Dockerfile before? This is where we can use it. In our case, rails will use port 3000 to start the server. Don't forget to put -b 0.0.0.0 because you won't see the app on your local machine without this.
- Open browser and hit the http://localhost:3000
  > Congratulation! your ROR container is ready.