# In-Hand Object Rotation Codebase

This codebase is initially built for code release of the following paper:

<b>In-Hand Object Rotation via Rapid Motor Adaptation</b> <br>
[Haozhi Qi*](https://haozhi.io/),
[Ashish Kumar*](https://ashish-kmr.github.io/),
[Roberto Calandra](https://www.robertocalandra.com/about/),
[Yi Ma](http://people.eecs.berkeley.edu/~yima/),
[Jitendra Malik](https://people.eecs.berkeley.edu/~malik/) <br>
Conference on Robot Learning (CoRL), 2022 <br>
[[Website](https://haozhi.io/hora)],
[[Paper](https://arxiv.org/abs/2210.04887)],
[[Video](https://www.youtube.com/watch?v=yH0e0l-H7-8)]

<p align="center">
  <img src="https://user-images.githubusercontent.com/10141467/204687717-bb649cb5-ab0f-4450-a98b-2d40788029f6.gif" width="1000"/>
</p>

After the initial release, we are still actively building this project by adding new features and resolving previous bugs. Therefore, some of the experiment number may be inconsistent from what was reported in the above paper. Please check out version [0.0.1](https://github.com/HaozhiQi/hora/tree/v0.0.1) if you want to reproduce the numbers reported in the paper.

We also maintain a [changelog and bugs](docs/changelog.md) we found during this development process.

Join our [discord](https://discord.gg/Trxzk78TQh) channel for interactive communications.

## Disclaimer

It is worth noticing that:
1. Simulation: The repo is mainly developed and debugged using IsaacGym Preview 4.0 ([Download](https://drive.google.com/file/d/1StaRl_hzYFYbJegQcyT7-yjgutc6C7F9)). Please note the results will be inconsistent if you train with IsaacGym Preview 3.0.
2. Hardware: The method is developed using an internal version of AllegroHand. We also provide a reference implementation (but please refer to version [0.0.1](https://github.com/HaozhiQi/hora/tree/v0.0.1) readme) and [video results](https://haozhi.io/hora/allegro_v4) using the public AllegroHand-v4.
3. Results: The reward number in this repository are higher than what is reported in the paper. This is because we change the `reset` function order following [LeggedGym](https://github.com/leggedrobotics/legged_gym) instead of the one in [IsaacGymEnvs](https://github.com/NVIDIA-Omniverse/IsaacGymEnvs/blob/e8f1c66b24/isaacgymenvs/tasks/base/vec_task.py).

## Installation

See [installation instructions](docs/install.md).

## Getting Started

This repository contains the following functionalities:
1. We start from loading, visualizing, and evaluating a pretrained policy in simulation.
2. We show how to train your own policy for in-hand object rotation.
3. We provide code for deploying a trained policy on real-world hardware.

### Prerequisite

In this paper, we consider in-hand object rotation starting from a stable initial grasp. Download the pre-computed grasping pose files (see [development instructions](docs/dev.md) for how to customize grasp generation) for both the public allegro hand and our internal allegro hand.

```
# file size 109M
gdown 1xqmCDCiZjl2N7ndGsS_ZvnpViU7PH7a3 -O cache/data.zip
unzip cache/data.zip -d cache/
```

The data structure should look like the following:

```
# s07 means the size of the object is 0.7 times canonical size.
# 50k means there are 50k randomly sampled grasping pose for each scale.

cache/
  internal_allegro_grasp_50k_s07.npy
  ...
  internal_allegro_grasp_50k_s088.npy
  public_allegro_grasp_50k_s088.npy
  ...
  public_allegro_grasp_50k_s088.npy
```

## Visualize and Evaluate a Trained Policy

This section can verify whether you install the repository and dependencies correctly. It also gives you a rough idea of how the policy looks like in the simulation.

Download a pretrained policy:
```
cd outputs/AllegroHandHora/
gdown 17fr40KQcUyFXz4W1ejuLTzRqP-Qu9EPS -O hora_v0.0.2.zip
unzip hora_v0.0.2.zip -d ./hora_v0.0.2
cd ../../
```

The data structure should look like:
```
outputs/
  AllegroHandHora/
    hora_v0.0.2/
      stage1_nn/  # stage 1 checkpoints
      stage2_nn/  # stage 2 checkpoints
```

Visualize it by running the following command. Note that stage 1 policy refers to the one trained with privileged object information while stage 2 policy refers to the one trained with proprioceptive history. The stage 2 policy is also what we deployed in the real-world.

```
# s1 and s2 stands for stage 1 and 2, respectively
scripts/vis_s1.sh hora_v0.0.2
scripts/vis_s2.sh hora_v0.0.2

# NOTE: if an error appears: "libtorch_cpu.so: undefined symbol: iJIT_NotifyEvent", Re-install PyTorch as follows, and try again:
pip install --force-reinstall --no-cache-dir torch -f https://download.pytorch.org/whl/cu118
<run above scripts again>

# NOTE: if an numpy.float error apperas, rollback numpy to an older version (likely needs <1.20, see what error message says), and try again:
pip install "numpy<1.20"
```

Evaluate this two policies by running:

```
# change {GPU_ID} to a valid number
scripts/eval_s1.sh ${GPU_ID} hora_v0.0.2
scripts/eval_s2.sh ${GPU_ID} hora_v0.0.2
```

## Training the Policy

To train the policy, we need to first train a stage 1 base policy. This policy has the privileged information as the input, as specified by `train.ppo.priv_info=True`. You need to specify `GPU_ID`, a random seed `SEED_ID`, and an output name for the command below:
```
# e.g. scripts/train_s1.sh 0 0 debug
scripts/train_s1.sh ${GPU_ID} ${SEED_ID} ${OUTPUT_NAME}
```

After training this policy, you can run `train_s2.sh` to train the proprioceptive adaptation module.
```
# e.g. scripts/train_s2.sh 0 0 debug
scripts/train_s2.sh ${GPU_ID} ${SEED_ID} ${OUTPUT_NAME}
```

If you want to train with the public allegro, use:
```
scripts/train_s1.sh ${GPU_ID} ${SEED_ID} ${OUTPUT_NAME} task=PublicAllegroHandHora
scripts/train_s2.sh ${GPU_ID} ${SEED_ID} ${OUTPUT_NAME} task=PublicAllegroHandHora
```

## Hardware Deployment

See [deployment instructions](docs/deploy.md).

## Further Development

We provide a few notes for development, see [development instructions](docs/dev.md). If you run into any issues, or want to provide any feedback, or want to contribute, please do it through GitHub issue / pull request. You can also join the [discord](https://discord.gg/Trxzk78TQh) channel for interactive communications.

## Citing

If you find **Hora** or this codebase helpful in your research, please consider citing:

```
@InProceedings{qi2022hand,
  author={Qi, Haozhi and Kumar, Ashish and Calandra, Roberto and Ma, Yi and Malik, Jitendra},
  title={{In-Hand Object Rotation via Rapid Motor Adaptation}},
  booktitle={Conference on Robot Learning (CoRL)},
  year={2022}
}
```
