<<<<<<< HEAD
# Deepfake-Detection
=======
# MRI-GAN: A Generalized Approach to Detect DeepFakes using Perceptual Image Assessment


This readme file gives basic overview of the scrope of this project, sample results, and steps needed to replicate the work, either from scratch or using pre-trained models. Reproducing the results from-scratch is very involved process and includes training of all the models. In either case data processing needs to be done. Details are described below.

The full research paper is available at: https://arxiv.org/abs/2203.00108

**TLDR.**
## Abstract
DeepFakes are synthetic videos generated by swapping a face of an original image with the face of somebody else. In this paper, we describe our work to develop general, deep learning-based models to classify DeepFake content. We propose a novel framework for using Generative Adversarial Network (GAN)-based models, we call MRI-GAN, that utilizes perceptual differences in images to detect synthesized videos. We test our MRI-GAN approach, and a plain-frames-based model using the DeepFake Detection Challenge Dataset. Our plain frames-based-model achieves 91% test accuracy, and a model which uses our MRI-GAN framework with Structural Similarity Index Measurement (SSIM) for the perceptual differences achieves 74% test accuracy. The results of MRI-GAN are preliminary and maybe improved further by modifying the choice of loss function, tuning hyper-parameters, or by using a more advanced perceptual similarity metric.


## MRI-GAN

MRI-GAN generates MRI of the input image. The MRI is DeepFake image contains artifacts which highlights regions of synthesisezed pixels. The MRI of non-DeepFake image is just black image.

<img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/mri_model_arch.png" width=35% height=35%>

# MRI-GAN adversarial training

<img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/dis_model.png" width=35% height=35%>
<img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/gen_model.png" width=55% height=55%>

# MRI-GAN training data formulation

<img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/mri_df_dataset_gen.png" width=55% height=55%>

# MRI-GAN sample output on validation set

<img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/MRI_demo.png" width=70% height=70%>

# MRI-GAN training progress

<img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/L2_loss_plot.png" width=40% height=40% /> <img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/loss_G_plot.png" width=40% height=40%/> <img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/ssim_loss_plot.png" width=40% height=40%/> <img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/ssim_score_plot.png" width=40% height=40%/> <img src="https://github.com/pratikpv/mri_gan_deepfake/blob/main/images/loss_D_plot.png" width=40% height=40%/>


## Steps to replicate the overall work

Note: This is very involved process.

1. Set development environment.
   We have used conda for our python distribution and related libraries on Ubuntu 20.04 OS. Create a new environment using below command and activate it. We have provided our environment.yml in the codebase.
   
    `conda env create -f environment.yml`
1. Download datasets and extract.
    1. DFDC dataset from https://ai.facebook.com/datasets/dfdc/
    1. Celeb-DF-v2 dataset from https://github.com/yuezunli/celeb-deepfakeforensics
    1. FFHQ dataset from https://github.com/NVlabs/ffhq-dataset
    1. FDF dataset from https://github.com/hukkelas/FDF
1. Configure the paths and other params.
   
    Note: Configuration of these paths may have been optimized by setting relative paths, but due huge size of dataset and limitation of available storage space, we have set absolute paths for each entity to have flexibility to choose where to save individual outcomes. Downside of this choice is that, we have to set all paths individually which can be tedious.
   
    1. `config.yml` is the key configuration to control the whole flow. Update paths of the dataset paths as needed. You would need to update all the paths which starts from /home/directory, other filenames does not need be changed.  
    1. DFDC dataset configuration
        1. update `['data_path']['dfdc']['train']` : path of the training set
        1. update `['data_path']['dfdc']['valid']` : path of the validation set
        1. update `['data_path']['dfdc']['test']` : path of the test set
        1. Update all key-value pairs under `['features']['dfdc']['landmarks_paths']` to point to where you want to save generated landmarks for DFDC
        1. Update all key-value pairs under `['features']['dfdc']['crop_faces']` to point to where you want to save extracted images of faces for DFDC
        1. update `['features']['dfdc']['mri_path']` : path where all MRIs will be saved. These MRIs are used for MRI-GAN training 
        1. update `['features']['dfdc']['train_mrip2p_faces']` : After MRI-GAN is trained, it is used to predict MRIs of DFDC. All predicted MRIs are saved here. Same for `valid_mrip2p_faces` and `test_mrip2p_faces`: update the paths.
    1. Celeb-DF-v2  dataset configuration.
        1. `['data_path']['celeb_df_v2']['real']` : path of real samples (Celeb-real)
        1. `['data_path']['celeb_df_v2']['fake']` : path of fake samples (Celeb-synthesis)
        1. `['features']['celeb_df_v2']['landmarks_path']['train']` : path where landmarks will be saved
        1. `['features']['celeb_df_v2']['crop_faces']['train']` : path where extracted faces will be saved
    1. FDF dataset configuration.
        1. `['data_path']['fdf']['data_path']` : path of samples (cc-by-nc-sa-2/128)
        1. `['data_path']['fdf']['landmarks_path']['train']` : path where landmarks will be saved
        1. `['features']['fdf']['json_filename']` : path to a json file where landmarks will be saved
        1. `['features']['fdf']['crops_path']` : path where extracted faces will be saved
    1. FFHQ dataset configuration.
        1. `['data_path']['ffhq']['data_path']` : path of samples (images1024x1024)
        1. `['features']['ffhq']['json_filename']` : path to a json file where landmarks will be saved
        1. `['features']['ffhq']['crops_path']` : path where extracted faces will be saved
1.  Data pre-processing. Enter following commands in sequence
    1. `python data_preprocess.py --gen_aug_plan` (select random video files in the DFDC training set and make a plan to apply various 
       random combinations of augmentation and distractions. This command generates the plan and saves in a .pkl file.) 
    1. `python data_preprocess.py --apply_aug_to_all` (Execute the plan generated in step #1. This command reads the .pkl file
       generated in step #1 and executes the plan one-by-one for each video file selected in DFDC training set)
    1. `python data_preprocess.py --extract_landmarks` (Use pre-trained MTCNN to extract landmarks of each face detected in the video frames.
       Every 10th frame is used by default in each video. Landmarks are extracted for each video in train, validation and test set. All landmarks
       are saved in separate .json files for each video)
    1. `python data_preprocess.py --crop_faces` (Save faces from landmarks json files for each video)
    1. `python data_preprocess.py --gen_mri_dataset` (Generate MRI-DF dataset. This generates the images of perceptual dissimilarity for DFDC train 
       set -(50% of DFDC train set as mentioned in the paper))

1.  MRI-GAN training
    1. Configure `config.yml`. Parameters under ['MRI_GAN']['model_params'] section can be tweaked. 'tau' is adjusted for different results.
       'batch_size' can be changed depending upon GPU memory available for your machine.
    1. `python train_MRI_GAN.py --train_from_scratch` (Train the MRI-GAN model. Check help for option on --train_resume to resume training 
       if it was stopped earlier. Logs will be generated and saved under logs/<date_time_stamp> directory, model weights will also be saved in the same directory)
    1. `cp logs/<date_time_stamp>/MRI_GAN/checkpoint_best_G.chkpt assets/weights/MRI_GAN_weights.chkpt` (Copy trained MRI-GAN weights)
    1. `python data_preprocess.py --gen_dfdc_mri` (Use trained MRI-GAN to predict MRIs for DFDC dataset) 

1. Train and test the DeepFake Detection model
    1. `python data_preprocess.py --gen_deepfake_metadata` (Generate metadata csv files used by DataLoaders of PyTorch classes)
    1.  Using plain-frames method
        1. Configure `config.yml`. Parameters under ['deep_fake']['model_params'] section can be tweaked. For plain-frames method set following params. 
           'train_transform' : 'complex'
           'dataset' : 'plain'
           'batch_size' can be changed depending upon GPU memory available for your machine.
        1. `python deep_fake_detect.py --train_from_scratch` (Start training from scratch. Also check `--train_resume` command line option if you want to resume previously started training. After all epochs are done, testing of the model will start)
        1. `python deep_fake_detect.py --test_saved_model <path>` (Test the model which was saved on disk. e.g. if the training was killed before all epochs were completed, this option can be used to test the model which was saved during training process)
    1.  Using MRI-based method
        1. Configure `config.yml`. Parameters under ['deep_fake']['model_params'] section can be tweaked. For plain-frames method set following params. 
           'train_transform' : 'simple'
           'dataset' : 'mri'
           'batch_size' can be changed depending upon GPU memory available for your machine.
        1. `python deep_fake_detect.py --train_from_scratch` (Start training from scratch. Also check `--train_resume` command line option if you want to resume previously started training. After all epochs are done, testing of the model will start)
        1. `python deep_fake_detect.py --test_saved_model <path>` (Test the model which was saved on disk. e.g. if the training was killed before all epochs were completed, this option can be used to test the model which was saved during training process)




### Other Notes
* check --help of all scripts mentioned above to see more utility methods, e.g. to resume training of models if the trained was stopped in between.

## Pre-trained models

Download all pre-trained model weights to reproduce the results.

1. MRI-GAN. Model with tau = 0.3 and Generator with the lowest loss:  https://drive.google.com/uc?id=1qEfI96SYOWCumzPdQlcZJZvtAW_OXUcH
1. DeepFake detection models
   1. Plain-frames based: https://drive.google.com/uc?id=1_Pxv6ptxqXKtDJNkodkDmMTD_KRo08za
   1. MRI based: https://drive.google.com/uc?id=1xKzehNuq1B1th-_-U6OG9v2Q2Odws6VG 

## DeepFake Detection App

Use the model to test a given video file.

1. Download all pre-trained model weights.
1. Run the command-line App 
`python detect_deepfake_app.py --input_videofile <path to video file> --method <detection method>`. Detection method can be `plain_frames` or `MRI`

## How to cite our research!
```
Pratikkumar Prajapati and Chris Pollett, MRI-GAN: A Generalized Approach to Detect DeepFakes using Perceptual Image Assessment. arXiv preprint arXiv:2203.00108 (2022)
```
or
```
@misc{2203.00108,
Author = {Pratikkumar Prajapati and Chris Pollett},
Title = {MRI-GAN: A Generalized Approach to Detect DeepFakes using Perceptual Image Assessment},
Year = {2022},
Eprint = {arXiv:2203.00108},
}
>>>>>>> 53c5734 (First commit)
