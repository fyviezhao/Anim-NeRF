# Animatable Neural Radiance Fields from Monocular RGB Videos
### [Github](https://github.com/JanaldoChen/Anim-NeRF) | [Paper](https://arxiv.org/abs/2106.13629)

![overview](assets/Overview.png)

> [Animatable Neural Radiance Fields from Monocular RGB Videos](https://arxiv.org/abs/2106.13629)    
> Jianchuan Chen, Ying Zhang, Di Kang, Xuefei Zhe, Linchao Bao, Xu Jia, Huchuan Lu

## Demos

* male-3-casual

    <img width="30%" src='assets/demos/male-3-casual/input.gif'></img><img width="60%" src='assets/demos/male-3-casual/male-3-casual.gif'></img>

More demos please see [Demos](./DEMOS.md).

## Requirements
- Python3.7 or later
- [PyTorch](https://pytorch.org/) 1.6 or later
- Pytorch-lightning
- [KNN_CUDA](https://github.com/unlimblue/KNN_CUDA)

### For visualization
- pyrender
- Trimesh
- PyMCubes

First clone this repo:
```sh
git clone --recursive https://github.com/fyviezhao/Anim-NeRF.git
```

Then run the following code to install all pip packages:
```sh
pip install -r requirements.txt
```

To install [KNN_CUDA](https://github.com/unlimblue/KNN_CUDA), we provide two ways:
* from source
  ```sh
  git clone https://github.com/unlimblue/KNN_CUDA.git
  cd KNN_CUDA
  make && make install
  ```
* from wheel
  ```sh
  pip install --upgrade https://github.com/unlimblue/KNN_CUDA/releases/download/0.2/KNN_CUDA-0.2-py3-none-any.whl
  ```

### SMPL models
To download the *SMPL* model go to [this](http://smpl.is.tue.mpg.de) (male, female and neutral models).

**Place them as following:**
```bash
smplx
└── models
    └── smpl
        ├── SMPL_FEMALE.pkl
        ├── SMPL_MALE.pkl
        └── SMPL_NEUTRAL.pkl
```

## Data Preparation
### People-Snapshot datasets
* Prepare images and smpls
  ```sh
  python -m tools.people_snapshot --data_root ${path_to_people_snapshot_datasets} --people_ID male-3-casual --gender male --output_dir data/people_snapshot
  ```
* Prepare template
  ```sh
  python -m tools.prepare_template --data_root data/people_snapshot --people_ID male-3-casual --model_type smpl --gender male --model_path ${path_to_smpl_models}
  ```

### iPER datasets or Custom datasets
* Prepare images

  First convert video to images, and crop the images to make the person as centered as possible.
  ```sh
  python -m tools.video_to_images --vid_file ${path_to_iper_datasets}/023_1_1.mp4 --output_folder data/iper/iper_023_1_1/cam000/images --img_wh 800 800 --offsets -30 20
  ```

* Images segmentation

  Here, we use [RVM](https://github.com/PeterL1n/RobustVideoMatting) to extact the foreground mask of the person. You need download RVM's pretrained model [rvm_resnet50.pth](https://github.com/PeterL1n/RobustVideoMatting/releases/download/v1.0.0/rvm_resnet50.pth) and put it under `third_party/RobustVideoMatting/checkpoints`.
  ```sh
  python -m tools.rvm --images_folder data/iper/iper_023_1_1/cam000/images --output_folder data/iper/iper_023_1_1/cam000/images
  ```

* SMPL estimation

  In our experiment, we use [VIBE](https://github.com/mkocabas/VIBE) to estimate the smpl parameters. You need follow [this instruction](https://github.com/fyviezhao/VIBE#getting-started) to set up the VIBE environment and the necessary `vibe_data` directory.
  ```sh
  python -m tools.vibe --images_folder data/iper/iper_023_1_1/cam000/images --output_folder data/iper/iper_023_1_1
  ```
  Then convert **vibe_output.pkl** to the format in our experiment setup.
  ```sh
  python -m tools.convert_vibe --data_root data/iper --people_ID iper_023_1_1 --gender neutral
  ```

* Prepare template
  ```sh
  python -m tools.prepare_template --data_root data/iper --people_ID iper_023_1_1 --model_type smpl --gender neutral --model_path ${path_to_smpl_models}
  ```

## Training
- Training on the training frames
  ```sh
  python train.py --cfg_file configs/people_snapshot/male-3-casual.yaml
  ```
- Finetuning the smpl params on the testing frames
  ```sh
  python train.py --cfg_file configs/people_snapshot/male-3-casual_refine.yaml train.ckpt_path checkpoints/male-3-casual/last.ckpt
  ```
We provide the preprocessed data and pretrained models at [Here](https://drive.google.com/drive/folders/1iXD2CShfcjk8fxUAC0VmTdiKeDz-DOc8?usp=sharing)
## Visualization
### Novel view synthesis
```sh
python novel_view.py --ckpt_path checkpoints/male-3-casual/last.ckpt
```
### 3D reconstruction
```sh
python extract_mesh.py --ckpt_path checkpoints/male-3-casual/last.ckpt
```
### Shape Editing
- To make the person look fatter, we can properly reduce the *betas_2th* parameter.
  ```sh
  python novel_view.py --ckpt_path checkpoints/male-3-casual/last.ckpt --betas_2th -2.0
  ```
- In contrast, we can increase the *betas_2th* parameter to make the person look slimmer.
  ```sh
  python novel_view.py --ckpt_path checkpoints/male-3-casual/last.ckpt --betas_2th 3.0
  ```
### Novel pose synthesis
```sh
python novel_pose.py --ckpt_path checkpoints/male-3-casual/last.ckpt
```
The [mixamo](https://www.mixamo.com) motion capture smpl parameters can be downloaded from [here](https://drive.google.com/drive/folders/1iXD2CShfcjk8fxUAC0VmTdiKeDz-DOc8?usp=sharing).
## Testing
```sh
python test.py --ckpt_path checkpoints/male-3-casual_refine/last.ckpt --vis
```


## Citation

If you find the code useful, please cite: 

```
@misc{chen2021animatable,
      title={Animatable Neural Radiance Fields from Monocular RGB Videos}, 
      author={Jianchuan Chen and Ying Zhang and Di Kang and Xuefei Zhe and Linchao Bao and Xu Jia and Huchuan Lu},
      year={2021},
      eprint={2106.13629},
      archivePrefix={arXiv},
      primaryClass={cs.CV}
}
```

## Acknowledgements
Parts of the code were based on from kwea123's NeRF implementation: https://github.com/kwea123/nerf_pl.
Some functions are borrowed from PixelNeRF https://github.com/sxyu/pixel-nerf
