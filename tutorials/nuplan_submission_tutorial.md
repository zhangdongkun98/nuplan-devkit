![](https://www.nuplan.org/static/media/nuPlan_final.3fde7586.png)

### Contents

1. [Introduction](#introduction)
2. [Creating a valid submission](#creating_submission)
3. [Testing the submission locally](#testing_submission)
3. [Submitting a solution](#submission)
4. [Visualizing metrics and scenarios](#dashboard)

# nuPlan submissions <a name="introduction"></a>

This tutorial will walk you through how to generate a valid submission, and send it to the evaluation server.

## Prerequisites

In this guide, it is assumed you followed the `nuplan_framework` tutorial and are familiar with the nuPlan environment. 
It is suggested, but not required, to first go through this tutorial with the example Planner (`SimplePlanner`) 
and then make the necessary modifications to use your custom planner.

Another requirement is to have a local Docker installation. You can follow the official documentation
[here](https://docs.docker.com/get-docker/).

## Submission infrastructure

In order to evaluate submitted planners, our team implemented a Docker based framework. 
Your submission will act as a server, ran within a Docker container, providing the computation of trajectories as a 
service, which will be called from the simulation run by us (the client). 
A visualization of the architecture is available below.

**TODO**: add figure

### Protocol specifications

The client/server communication is made using [gRPC](https://grpc.io/), the configuration files and code related to 
the communication can be found at `~/nuplan_devkit/nuplan/submission`. 
In order for submissions to work as expected, the protocol files (`protos/`) and the autogenerated 
(`challenge_pb2.py` and `challenge_pb2_grpc.py`) files **MUST NOT** be modified, as we will use our version of them.

Similarly, you should not modify `submission_container.py` and only modify the specified section of 
`submission_planner.py`, to avoid invalid submissions.

# Creating a valid submission <a name="creating_submission"></a>

The first step is to create a Docker image using your solution, you can follow these steps:

1. Add your pip requirements to `requirements_submission.txt`

2. Add you apt requirements to `requirements_submission_apt.txt`

3. Edit `Dockerfile.submission` (not necessary in general)

4. From the root of the repository, run:

```console
$ docker build -f Dockerfile.submission . -t CONTESTANT_ID/SUBMISSION_NAME
```

This should generate a Docker image, you can check if it is present with

```console
$ docker image ls
```

And to create a shell inside the image, to make sure all the wanted files are present, run

```console
$ docker run -it --rm CONTESTANT_ID/SUBMISSION_NAME /bin/bash
```

### Creating an image using your planner
In order to use your planner you will have to modify the `submission_planner.py` file, 
specifically the `if __name__ == "main"` part. This script will be executed when the container is run during the 
evaluations.
You need to instantiate your planner and pass it as argument to the SubmissionPlanner which is created there. 
This way, when your image is run during evaluation, the server is created using your planner.

# Testing your solution <a name="testing_submission"></a>

Before heading to the actual submission server, you want to make sure that your submission is valid. 
To do so you can run it locally using the `RemotePlanner`. This will create a container with the given image, 
and run the simulation as usual, though delegating the trajectory computation to the service running in the container.

To run the simulation this way, you need to overwrite some hydra parameters. In particular the `planner` parameter
must be set to use the `RemotePlanner`, and the `planner.contestant_id` and `planner.submission_id` must reflect
the name of the Docker image you generated. From the root of the repo run:

```console
$ python nuplan/planning/script/run_simulation.py +simulation=open_loop_boxes planner="[remote_planner]" planner.contestant_id="test-contestant" planner.submission_id="nuplan-test-submission' scenario_builder.nuplan.scenario_filter.limit_scenarios_per_type=3
```
**TODO** Add configs for contestant id and submission

This will run a very brief simulation, but will let you check if the planner is running correctly.

# Submitting a solution <a name="submission"></a>

**TODO**

# Seeing the outcome of your submission <a name="dashboard"></a>

**TODO**

