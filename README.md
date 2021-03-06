Habitat-API
==============================

Habitat-API is a modular high-level library for end-to-end development in embodied AI --
defining embodied AI tasks (e.g. navigation, instruction following, question answering), configuring embodied agents (physical form, sensors, capabilities), training these agents (via imitation or reinforcement learning, or no learning at all as in classical SLAM), and benchmarking their performance on the defined tasks using standard metrics.

Habitat-API currently uses [`Habitat-Sim`](https://github.com/facebookresearch/habitat-sim) as the core simulator, but is designed with a modular abstraction for the simulator backend to maintain compatibility over multiple simulators.

<p align="center">
  <img src='docs/img/habitat_compressed.gif' />
</p>

---

## Table of contents
   1. [Motivation](#motivation)
   1. [Citing Habitat](#citing-habitat)
   1. [Details](#details)
   1. [Installation](#installation)
   1. [Example](#example)
   1. [Data](#data)
   1. [Baselines](#baselines)

## Motivation
A key long-term goal of artificial intelligence (AI) is to build intelligent _agents_ that can
* _see_ or more generally perceive their environment through vision, audition, or other senses,
* _talk_, i.e. hold a natural language dialog grounded in the environment,
* _act_, i.e. navigate their environment and interact with it to accomplish goals, and
* _reason_, i.e. consider the long-term consequences of their actions

#### From ‘Internet AI’ to Embodied AI.
While there has been significant progress in the vision and language communities thanks to recent advances in deep representations, we believe there is a growing disconnect between ‘internet AI’ and embodied AI. The focus of the former is pattern recognition in images, videos, and text on datasets typically curated from the internet. The focus of the latter is to enable action by an embodied agent in an environment (e.g. a robot). This brings to the forefront issues of active perception, long-term planning, learning from interaction, and holding a dialog grounded in an environment.

To this end, we aim to standardize the entire ‘software stack’ for training embodied agents – scanning the world and creating highly photorealistic 3D assets, developing the next generation of highly efficient and parallelizable simulators, specifying embodied AI tasks that enable us to benchmark scientific progress, and releasing modular high-level libraries to train and deploy embodied agents.

#### Why the name _Habitat_?
Because that's where AI agents live :slightly_smiling_face:

#### Example
<!--- Please, update `examples/example.py` if you update example. -->

Example code-snippet which uses [`tasks/pointnav.yaml`](configs/tasks/pointnav.yaml) for configuration of task and agent.

```python
import habitat

env = habitat.Env(
    config=habitat.get_config(config_file="tasks/pointnav.yaml")
)

observations = env.reset()

while not env.episode_over:
    observations = env.step(env.action_space.sample())

```

## Citing Habitat
If you use the Habitat platform in your research, please cite the following technical report:
```
@article{habitat19arxiv,
  title =   {Habitat: A Platform for Embodied AI Research},
  author =  {{Manolis Savva*}, {Abhishek Kadian*}, {Oleksandr Maksymets*}, Yili Zhao, Erik Wijmans, Bhavana Jain, Julian Straub, Jia Liu, Vladlen Koltun, Jitendra Malik, Devi Parikh and Dhruv Batra},
  journal = {arXiv preprint arXiv:},
  year =    {2019}
}
```

## Details
An important objective of Habitat-API is to make it easy for users to set up a variety of embodied agent tasks in 3D environments. The process of setting up a task involves using environment information provided by the simulator, connecting the information with a dataset (e.g. PointGoal targets, or question and answer pairs for Embodied QA) and providing observations which can be used by the agents. Keeping this primary objective in mind the core API defines the following key concepts as abstractions that can be extended:

* `Env`: the fundamental environment concept for Habitat. All the information needed for working on embodied tasks with a simulator is abstracted inside an Env. This class acts as a base for other derived environment classes. Env consists of three major components: a Simulator, a Dataset (containing Episodes), and a Task, and it serves to connects all these three components together.

* `Dataset`: contains a list of task-specific episodes from a particular data split and additional dataset-wide information. Handles loading and saving of a dataset to disk, getting a list of scenes, and getting a list of episodes for a particular scene.

* `Episode`: a class for episode specification that includes the initial position and orientation of an Agent, a scene id, a goal position and optionally shortest paths to the goal. An episode is a description of one task instance for the agent.

<p align="center">
  <img src='docs/img/habitat-api_structure.png' alt="teaser results" width="100%"/>
  <p align="center"><i>Architecture of Habitat-API</i></p>
</p>

* `Task`: this class builds on top of the simulator and dataset. The criteria of episode termination and measures of success are provided by the Task.

* `Sensor`: a generalization of the physical Sensor concept provided by a Simulator, with the capability to provide Task-specific Observation data in a specified format.

* `Observation`: data representing an observation from a Sensor. This can correspond to physical sensors on an Agent (e.g. RGB, depth, semantic segmentation masks, collision sensors) or more abstract sensors such as the current agent state.

Note that the core functionality defines fundamental building blocks such as the API for interacting with the simulator backend, and receiving observations through Sensors. Concrete simulation backends, 3D datasets, and embodied agent baselines are implemented on top of the core API.

## Installation

1. Clone the github repository and install using the commands below. Note that we only support [python3](https://pythonclock.org/), all the development and testing was done using python3.6. Please use 3.6 to avoid possible issues.
```bash
cd habitat-api
pip install -e .
```

2. Install `habitat-sim` from [github repo](https://github.com/facebookresearch/habitat-sim) and as stated there, remember to add `habitat-sim` to `PYTHONPATH`:
```bash
export PYTHONPATH="${PYTHONPATH}:/path/to/habitat-sim/:/path/to/habitat-sim/build/esp/bindings"
```
3. Download the [test scenes data](http://dl.fbaipublicfiles.com/habitat/habitat-test-scenes.zip) and extract `data` folder in zip to `{HABITAT_API_MAIN_DIR}/data/`.

4. Run the example script `python examples/example.py ` which in the end should print out number of steps agent took inside an environment (eg: `Episode finished after 2 steps.`). To verify that tests pass run `python setup.py test` which should print out a log about passed, skipped and failed tests.

## Data
To make things easier we expect `data` folder of particular structure or symlink presented in habitat-api working directory.

### Scenes datasets
| Scenes models | Extract path | Size |
| --- | --- | --- |
| [Gibson](#Gibson) | `data/scene_datasets/gibson/{scene}.glb` | 15 GB |
| [MatterPort3D](#Matterport3D) | `data/scene_datasets/mp3d/{scene}/{scene}.glb` | 21 GB |

#### Matterport3D
The full Matterport3D (MP3D) dataset for use with Habitat can be downloaded using the official [Matterport3D](https://niessner.github.io/Matterport/) download script as follows: `python download_mp.py --task habitat -o data/scene_datasets/mp3d/`. You only need the habitat zip archive and not the entire Matterport3D dataset. Note that this download script requires python 2.7 to run.

#### Gibson
Download Habitat edition [link will be provided] of [Gibson scenes](http://gibsonenv.stanford.edu/database/) and extract it to `data/scene_datasets/gibson/`.


### Task datasets
| Task | Scenes | Link | Extract path | Config to use | Size |
| --- | --- | --- | --- | --- | --- |
| Point goal navigtaion | [Gibson](#Gibson) | [pointnav_gibson_v1.zip](https://dl.fbaipublicfiles.com/habitat/data/datasets/pointnav/gibson/v1/pointnav_gibson_v1.zip) | `data/datasets/pointnav/gibson/v1/` |  [`datasets/pointnav/gibson.yaml`](configs/datasets/pointnav/gibson.yaml) | 385 MB |
| Point goal navigtaion | [MatterPort3D](#Matterport3D) | [pointnav_mp3d_v1.zip](https://dl.fbaipublicfiles.com/habitat/data/datasets/pointnav/mp3d/v1/pointnav_mp3d_v1.zip) | `data/datasets/pointnav/mp3d/v1/` | [`datasets/pointnav/mp3d.yaml`](configs/datasets/pointnav/mp3d.yaml) | 400 MB |

To use an episode dataset provide related config to the Env in [the example](#example) or use the config for [RL agent training](baselines/README.md#reinforcement-learning-rl).

## Baselines
Habitat-API includes reinforcement learning (via PPO) and classical SLAM based baselines. For running PPO training on sample data and more details refer [baselines/README.md](baselines/README.md).

## Acknowledgments
The Habitat project would not have been possible without the support and contributions of many individuals. We would like to thank Xinlei Chen, Georgia Gkioxari, Daniel Gordon, Leonidas Guibas, Saurabh Gupta, Or Litany, Marcus Rohrbach, Amanpreet Singh, Devendra Singh Chaplot, Yuandong Tian, Yuxin Wu and Dmytro Mishkin for many helpful conversations and guidance on the design and development of the Habitat platform.

## License
Habitat-API is MIT licensed. See the LICENSE file for details.
