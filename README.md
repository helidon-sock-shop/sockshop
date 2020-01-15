# Helidon Sock Shop 

This project is an implementation of the microservices based application
using [Helidon Microservices Framework](https://helidon.io/).

The application is an online store that sells, well, socks, and is based
on the canonical [SockShop Microservices Demo](https://microservices-demo.github.io)
originally written and published under Apache 2.0 license by [Weaveworks](https://go.weave.works/socks).

You can see a working demo of the original application [here](http://socks.weave.works/).

This demo still uses the original front end implementation provided by Weaveworks,
but all back end services have been re-implemented from scratch to use Helidon MP 
and showcase its features and best practices.

## Architecture

The application consists of a Node.js-based front end and 6 back end services:

- **[Product Catalog](https://github.com/helidon-sock-shop/catalog)**, which provides 
REST API that allows you to search product catalog and retrieve individual product details;

- **[Shopping Cart](https://github.com/helidon-sock-shop/carts)**, which provides 
REST API that allows you to manage customers' shopping carts;

- **[Orders](https://github.com/helidon-sock-shop/orders)**, which provides REST API 
that allows customers to place orders;

- **[Payment](https://github.com/helidon-sock-shop/payment)**, which provides REST API 
that allows you to process payments;

- **[Shipping](https://github.com/helidon-sock-shop/shipping)**, which provides REST API 
that allows you to ship orders and track shipments;

- **[Users](https://github.com/helidon-sock-shop/users)**, which provides REST API 
that allows you to manage customer information and provides registration and 
authentication functionality for the customers.

Each service has a _core_ module, which is where most of the code for the service is
(domain model, JAX-RS resources, etc.), and which also provides a basic in-memory
implementation of the data store that is useful for testing.

Most services also have one or more data store-specific modules, which build on the
_core_ module and demonstrate integrations of various popular data stores with Helidon 
(MySQL, Mongo, Redis, etc.)
 
You can find more details for each service within documentation pages for individual
services, which can be accessed using the links above.

## Project Structure

Each back end service described above has its own Github repo, so it can be versioned
and released independently from other services. 

In addition to that, there is also a main 
[Sock Shop](https://github.com/helidon-sock-shop/sockshop) repository (the one you are 
currently in), which contains Kubernetes deployment files for the whole application, 
top-level POM file which allows you to easily build the whole project and import it 
into your favorite IDE, and a _bash_ script that makes it easy to checkout and update 
all project repositories at once.

## Quick Start

The easiest way to try the demo is to use [provided Docker images](https://hub.docker.com/orgs/helidonsockshop/repositories)
and Kubernetes deployment scripts from this repo. 

Kubernetes scripts depend on Kustomize, so make sure that you have a newer version of `kubectl`
that supports it. If you do, you can simply run

```bash
$ kubectl apply -k sockshop/core 
```

from the top-level directory, and this will merge all the files under the specified 
directory and create all Kubernetes resources defined by them, such as deployment
and service for each microservice.

Assuming you deployed the application to a local Kubernetes cluster, you should be able
to access the home page for the application by pointing your browser to http://localhost:30001/.

Once you are finished, you can clean up the environment by executing

```bash
$ kubectl delete -k sockshop/core 
```

## Development
 
If you want to modify the demo, you will need to check out the code for the project, build it 
locally, and (optionally) push new Docker images to the repository of your choice.
 
### Checking Out the Project

The easiest way to checkout the source code for all the services is to run the provided 
`update.sh` script:

```bash
$ mkdir helidon-sock-shop
$ cd helidon-sock-shop
$ bash <(curl -s https://raw.githubusercontent.com/helidon-sock-shop/sockshop/master/update.sh)
```

Once you have the code locally, you can update it to the latest version by running the 
same script locally:

```bash
$ cd helidon-sock-shop
$ . sockshop/update.sh
```

**Note:** Make sure that you run the update script from the same top-level directory 
that you used to checkout all the services into or it will fail.

### Building the Code

You can build individual services by running `mvn package` within top-level directory
for each service, but if you want to build the whole project at once, there is an easier
way: simply run `mvn package` within the `sockshop` directory.

This will compile the code for each service and run the tests. 

### Creating Docker Images

Pre-built Docker images are already available on [DockerHub](https://hub.docker.com/orgs/helidonsockshop/repositories),
so you can simply run `docker pull` to download them and use locally.

However, if you are making changes to various service implementations and need to re-create
Docker images, you can easily do that in several different ways.

If you only want to build Docker images locally, make sure that you have Docker Daemon
running and execute the following command: 

```bash
$ mvn package -Pdocker
```

You can then tag images any way you want and push them to a Docker repo of your choice 
using standard Docker commands.

On the other hand, if you cannot or do not want to run Docker locally, you can push images
directly to the remote repository:

```bash
$ mvn package -Pdocker -Ddocker.repo={your_docker_repo} -Djib.goal=build
```

You should replace `{your_docker_repo}` in the command above with the name of the 
Docker repository that you can push Docker images to.

We use [Jib Maven Plugin](https://github.com/GoogleContainerTools/jib) to create and publish
Docker images, with a default Jib goal set to `dockerBuild`, in order to create local image.
Changing the goal to `build` via `jib.goal` system property allows you to push images to a 
remote repository directly.

### Running Modified Application

Kubernetes deployment scripts in this repository reference Docker images from the default
Docker Hub repository, `helidonsockshop`. You will not be able to push the images to that repo,
which is why you had to specify `-Ddocker.repo` argument in the command above.

In order to deploy the application that uses your custom Docker images, you will also have to modify
relevant `deployment.yaml` files in within `sockshop` repository to use correct names for your
Docker images.

## License

Apache License 2.0