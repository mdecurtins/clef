Max DeCurtins  
Harvard University
## Clef: A MIR Application
Clef is a web-based music information retrieval (MIR) application framework designed to provide a common interface for diverse algorithms performing searches of symbolic music data. Clef is an **experimental** framework and will likely change significantly.

Current Version: 1.0.0
## Usage
Instructions for installing and using Clef.
### Requirements
In order to run Clef, you must have installed:
- Docker
- A UNIX `sh` or `bash` shell

### Installation
Simply `git clone` this repository into a directory, preferably one within your home directory. To make sure you have the latest version of the Clef core services, run `git submodule update --init --recursive`.
#### Directory Structure
The directory structure of Clef is as follows, showing how to add your own algorithms:
```
.
|__ core/
|   |__ clef/
|       |__ ... Clef source code is located here
|
|__ algorithms/
|   |__ yourAlgorithm/
|       |__ clefinfo.json
|       |__ Dockerfile
|       |__ ... algorithm files   
|   
|__ webapp/
|__ db/
|__ symbolic_data/
    |__ yourDataset/
        |__ clefdataset.json
        |__ ... dataset files
```
The root of the tree is the directory containing the `docker-compose.yml` file. As currently written, `docker-compose.yml` will look to mount the `symbolic_data` directory as the source for datasets, though this can be changed easily enough to accommodate a standalone named Docker volume containing symbolic music datasets. Remember that each dataset must have an accompanying `clefdataset.json` file.
### Running Clef
Run `docker-compose up -d` in the Clef root directory. Once the services defined in the `docker-compose.yml` file have started, you will need to start the containers that actually serve the algorithms present in the `algorithms` directory.

Each algorithm _must_ have a `clefinfo.json` and a `Dockerfile` in the algorithm root directory, which itself must be a subdirectory of `algorithms`.

If you have not already built images for the algorithms, `cd` into each algorithm subdirectory and run the command `docker build -t your_image_name .` to build the image and add it to your local Docker repository.

To start an algorithm container, use `docker run` as shown below:
```
docker run -d -p hostPort:containerPort --rm --name clef-your-container-name --network "clef_default" --network-alias "clef-your-container-name" your_image_name
```
This tells Docker to run your algorithm container as a detached process. If desired, you can use a command-line JSON parser like `jq` to pull the option values from the `clefinfo.json` file. The `--rm` flag tells Docker to remove the container when it shuts down.

To verify that your container is running alongside the main Clef containers, run the `docker ps` command. It is _highly_ recommended that you prefix your container name with `clef-`&mdash;this will allow the `clef-stop` script to stop your algorithm container.

To shut down the Clef system, execute the `clef-stop.sh` script. It will stop any containers whose names contain `clef`.
## Components
- `clef-webapp`: A ReactJS-powered interface with an embedded Flat music notation editor, backed by a Redux store
- `clef-core`: A Java-based framework for managing requests from the web application and to containers that run music information retrieval algorithms
- `clef-db`: A MySQL database for storing metadata gathered from source files of symbolic music data
## Algorithms
Algorithms run in Docker containers and communicate with the `clef-core` service over HTTP. 
### Adding an Algorithm
 Adding an algorithm to Clef requires a straightforward, but non-trivial, amount of work. The Clef system treats algorithms as services accessible via an API over HTTP, thus you must configure your algorithm container to respond to HTTP requests made to a URL such as `/runMyAlgorithm?foo=bar&lorem=ipsum`.
 
 In the root folder of the algorithm, which itself is a subdirectory under `algorithms/`, you must place two files:
 1. A `Dockerfile` describing the container that will run your algorithm, including all libraries, dependencies, environment variables, etc. For reference, see the Dockerfile syntax documentation.
 2. A `clefinfo.json` file. This file describes attributes of your algorithm, as well as a few container attributes that should match what you define in your `Dockerfile`. This does duplicate some information, but it is necessary to provide reliable configuration information to `clef-core` and to separate Clef somewhat from the specific implementation in Docker.
 
 Note that&mdash;if it is already running&mdash;you must restart the `core` service in order for the Clef system to detect new algorithms and datasets (i.e., there is no "hot" reloading).
 
How you choose to serve the algorithm is up to you; frameworks such as Spring for Java and Flask for Python make setting up a simple API relatively easy. The API route defined as `queryRoute` in your `clefinfo.json` file should map to an endpoint that executes your algorithm. The method that handles this must be able to accept HTTP `POST` requests with a `Content-type: application/xml` header and a body consisting of a MusicXML document.

### Adding a Dataset
Adding a dataset is rather less complicated than adding an algorithm. As with algorithms, you must restart the `core` service in order for Clef to detect the change.

Your dataset should be placed in a subdirectory of `symbolic_data` and must contain, within the same directory, a `clefdataset.json` file.

**n.b.** The format defined in the `clefdataset.json` file must match _exactly_ with one of the enumerated values in [MusicFormat](https://github.com/mdecurtins/clef-core/blob/master/clef/src/main/java/clef/mir/MusicFormat.java).

#### Metadata
At this time metadata gathering is enabled only for files in Humdrum format.