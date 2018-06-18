Max DeCurtins  
Harvard University
## Clef: A MIR Application
Clef is a web-based music information retrieval (MIR) application framework designed to provide a common interface for diverse algorithms performing searches of symbolic music data.

Current Version: 1.0.0
## Usage
Instructions for installing and using Clef.
### Requirements
In order to run Clef, you must have installed:
- Docker
- A UNIX `sh` or `bash` shell
- `jq`, a command line JSON parser
### Installation
Simply `git clone` this repository into a directory, preferably one within your home directory.
#### Directory Structure
The directory structure of Clef is as follows, showing how to add your own algorithms:
```
.
|__ core/
|   |__ algorithms/
|   |   |__ yourAlgorithm/
|   |       |__ clefinfo.json
|   |       |__ Dockerfile
|   |       |__ ... algorithm files
|   |__ clef/
|   |__ datasets/
|__ webapp/
```
### Running Clef
Run `clef-start.sh`. This will do all the necessary preprocessing and then execute `docker-compose up -d` in the Clef root directory.

Each algorithm _must_ have a `clefinfo.json` and a `Dockerfile` in the algorithm root directory, which itself must be a subdirectory of `algorithms`.
## Components
- `clef-webapp`: A ReactJS-powered interface with an embedded Flat music notation editor, backed by a Redux store
- `clef-core`: A Java-based framework for managing requests from the web application and to containers that run music information retrieval algorithms
## Algorithms
Algorithms run in Docker containers. Launching Clef via `clef-start.sh` will locate any algorithms in the `algorithms` directory, and the script will attempt to start these containers and add them to the `clef_default` network created by `docker-compose`.

Algorithms communicate with `clef-core` over HTTP. 
### Adding an Algorithm
 Adding an algorithm to Clef requires a straightforward, but non-trivial, amount of work.
 
 In the root folder of the algorithm, which itself is a subdirectory under `algorithms/`, you must place two files:
 1. A `Dockerfile` describing the container that will run your algorithm, including all libraries, dependencies, environment variables, etc. For reference, see the Dockerfile syntax documentation.
 2. A `clefinfo.json` file. This file describes attributes of your algorithm, as well as a few container attributes that should match what you define in your `Dockerfile`. This does duplicate some information, but it is necessary to provide reliable configuration information to `clef-core` and to separate Clef somewhat from the specific implementation in Docker.