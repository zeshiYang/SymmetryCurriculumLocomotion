# Learning Symmetric and Low-energy Locomotion

This is code for our paper: https://arxiv.org/abs/1801.08093

## Install from script

The script below has been tested on clean Ubuntu 16.04/18.04 computers.

```bash
sudo -s
source prereq_install.sh
exit
```

## Manual Install

The code consists of two parts: dart-env, which is an extension of OpenAI Gym that uses Dart for rigid-body simulation, and baselines, which is adapted from OpenAI Baselines. Both libraries are included in the repository so you will not need to install the original library.

To install dart-env, you need to install Dart and Pydart, follow the instructions at: https://github.com/DartEnv/dart-env/wiki for installing these two packages. After installing Dart and Pydart, install dart-env by:

```bash
cd dart-env
pip install -e .
```


To install baselines, execute the following:

```bash
cd baselines
pip install -e .
```



## How to use

Example scripts for training and testing the policies are provided in [siggraph_script](siggraph_script/). The specific python code corresponding to each script can be found [here](baselines/baselines/siggraph).

The training results will be saved to data/. The final policy is saved as policy_params.pkl. You can also find the intermediate policies in the folders organized by the corresponding curriculums. To test a policy, run:

```bash
python baselines/test_policy.py ENV_NAME PATH_TO_POLICY
```

To visualize the learning curve, run:

```bash
python baselines/plot_benchmark.py PATH_TO_FOLDER
```



### Setup environment

4 example environments are included: [DartWalker3d-v1](dart-env/gym/envs/dart/walker3d.py), [DartHumanWalker-v1](dart-env/gym/envs/dart/human_walker.py), [DartDogRobot-v1](dart-env/gym/envs/dart/dog_robot.py) and [DartHexapod-v1](dart-env/gym/envs/dart/hexapod.py).

The desired velocity is controlled by three variables in the initialization of each environment: **init_tv** sets the target velocity at the beginning of the rollout, **final_tv** sets the target velocity we want the character to reach eventually, and **tv_endtime** sets the amount of time (in seconds) it takes to accelerate from **init_tv** to **final_tv**.

### Setup training script

Refer to [run_walker3d_staged_learning.py](baselines/baselines/ppo1/run_walker3d_staged_learning.py) for an example on how to setup the training script for the biped walking robot.

#### Mirror-symmetry 

The mirror-symmetry loss for a new environment is configured with the argument **observation_permutation** and **action_permutation** when initializing MlpMirrorPolicy in the training script.

For **observation_permutation** and **action_permutation**, they are two vectors used for mirror symmetric loss. Each entry in these two denotes the index of the corresponding entry in observation/action AFTER it is mirrored w.r.t the sagittal plane of the character, and the sign of the element means whether the entry should multiply -1 after mirroring. For example, if a character has its left and right elbow joint angle at index 4 and 7 of the obsevation vector, then observation_permutation[4] should be 7 and observation_permutation[7] should be 4. Further, if the behavior of the two dofs are opposite, e.g. larger value of left elbow angle means flexion while larger value of right elbow angle means extension, then a -1 should be multiplied to both entries in observation_permutation. Note that for dofs at the center of the character (like pelvis rotation), their corresponding mirrored entry are simply themselves, with -1 multiplied to some of them. Also, if the entry at index 0 need to be negated, you need to use a small negative value like -0.0001, as multiplying -1 wouldn't change 0.

## Additional notes

For a newly created dart-env environment, you can use [examine_skel.py](baselines/examine_skel.py) to test the model configurations, which I found to be helpful in debugging joint limits.

Refer to [run_walker3d_staged_learning.py](baselines/baselines/ppo1/run_walker3d_staged_learning.py) for an example on how to setup the training script for the biped walking robot.


## ODE Internal Error
If you see errors like: ODE INTERNAL ERROR 1: assertion "d[i] != dReal(0.0)" failed in _dLDLTRemove(), try downloading [lcp.cpp](https://drive.google.com/file/d/1MCho3QBtyPhSoKNV77VFOvCqsMJPk3NF/view) and replace the one in dart/external/odelcpsolver/ with it. Recompile Dart and Pydart2 afterward and the issue should be gone.

