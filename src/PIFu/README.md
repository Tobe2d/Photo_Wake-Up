# PIFu: Pixel-Aligned Implicit Function for High-Resolution Clothed Human Digitization


This repository contains a pytorch implementation of "[PIFu: Pixel-Aligned Implicit Function for High-Resolution Clothed Human Digitization](https://arxiv.org/abs/1905.05172)".

This codebase provides: 
- test code
- training code
- data generation code

## Requirements
- Python 3
- [PyTorch](https://pytorch.org/) tested on 1.4.0
- json
- PIL
- skimage
- tqdm
- numpy
- cv2

for training and data generation
- [trimesh](https://trimsh.org/) with [pyembree](https://github.com/scopatz/pyembree)
- [pyexr](https://github.com/tvogels/pyexr)
- PyOpenGL
- freeglut (use `sudo apt-get install freeglut3-dev` for ubuntu users)

## Windows demo installation instuction

- Install [miniconda](https://docs.conda.io/en/latest/miniconda.html)
- Add `conda` to PATH
- Install [git bash](https://git-scm.com/downloads)
- Launch `Git\bin\bash.exe`
- `eval "$(conda shell.bash hook)"` then `conda activate my_env` because of [this](https://github.com/conda/conda-build/issues/3371)
- Automatic `env create -f environment.yml` (look [this](https://github.com/conda/conda/issues/3417))
- OR manually setup [environment](https://towardsdatascience.com/a-guide-to-conda-environments-bc6180fc533)
    - `conda create —name pifu python` where `pifu` is name of your environment
    - `conda activate`
    - `conda install pytorch torchvision cudatoolkit=10.1 -c pytorch`
    - `conda install pillow`
    - `conda install scikit-image`
    - `conda install tqdm`
    - `conda install -c menpo opencv`
- Download [wget.exe](https://eternallybored.org/misc/wget/)
- Place it into `Git\mingw64\bin`
- `sh ./scripts/download_trained_model.sh`
- Remove background from your image ([this](https://www.remove.bg/), for example)
- Create black-white mask .png
- Replace original from sample_images/
- Try it out - `sh ./scripts/test.sh`
- Download [Meshlab](http://www.meshlab.net/)
- Open .obj file in Meshlab


## Demo
Warning: The released model is trained with mostly upright standing scans with weak perspectie projection and the pitch angle of 0 degree. Reconstruction quality may degrade for images highly deviated from trainining data.
1. run the following script to download the pretrained models from the following link and copy them under `./PIFu/checkpoints/`.
```
sh ./scripts/download_trained_model.sh
```

2. run the following script. the script creates a textured `.obj` file under `./PIFu/eval_results/`. You may need to use `./apps/crop_img.py` to roughly align an input image and the corresponding mask to the training data for better performance. For background removal, you can use any off-the-shelf tools such as [removebg](https://www.remove.bg/).
```
sh ./scripts/test.sh
```

## Data Generation (Linux Only)
While we are unable to release the full training data due to the restriction of commertial scans, we provide rendering code using free models in [RenderPeople](https://renderpeople.com/free-3d-people/).
This tutorial uses `rp_dennis_posed_004` model. Please download the model from [this link](https://renderpeople.com/sample/free/rp_dennis_posed_004_OBJ.zip) and unzip the content under a folder named `rp_dennis_posed_004_OBJ`. The same process can be applied to other RenderPeople data.

Warning: the following code becomes extremely slow without [pyembree](https://github.com/scopatz/pyembree). Please make sure you install pyembree.

1. run the following script to compute spherical harmonics coefficients for [precomputed radiance transfer (PRT)](https://sites.fas.harvard.edu/~cs278/papers/prt.pdf). In a nutshell, PRT is used to account for accurate light transport including ambient occlusion without compromising online rendering time, which significantly improves the photorealism compared with [a common sperical harmonics rendering using surface normals](https://cseweb.ucsd.edu/~ravir/papers/envmap/envmap.pdf). This process has to be done once for each obj file.
```
python -m apps.prt_util -i {path_to_rp_dennis_posed_004_OBJ}
```

2. run the following script. Under the specified data path, the code creates folders named `GEO`, `RENDER`, `MASK`, `PARAM`, `UV_RENDER`, `UV_MASK`, `UV_NORMAL`, and `UV_POS`. Note that you may need to list validation subjects to exclude from training in `{path_to_training_data}/val.txt` (this tutorial has only one subject and leave it empty).
```
python -m apps.render_data -i {path_to_rp_dennis_posed_004_OBJ} -o {path_to_training_data}
```

## Training (Linux Only)

Warning: the following code becomes extremely slow without [pyembree](https://github.com/scopatz/pyembree). Please make sure you install pyembree.

1. run the following script to train the shape module. The intermediate results and checkpoints are saved under `./results` and `./checkpoints` respectively. You can add `--batch_size` and `--num_sample_input` flags to adjust the batch size and the number of sampled points based on available GPU memory.
```
python -m apps.train_shape --dataroot {path_to_training_data} --random_flip --random_scale --random_trans
```

2. run the following script to train the color module. 
```
python -m apps.train_color --dataroot {path_to_training_data} --num_sample_inout 0 --num_sample_color 5000 --sigma 0.1 --random_flip --random_scale --random_trans
```
