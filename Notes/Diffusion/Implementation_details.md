

**Pipeline/Model configs**
- config/experiments/temporal/diffusion
    - Config params
        - Should be updated in the file synth2track/configs/base_config.py
            - Network params: NetworkConfigBase

**Node**
- define a node in ```_evaluation.py```
    - e.g make_diffusion_noiser, DiffusionNoiser

**Model**
- synth2track/models/Denoising_Diffusor.py

**Dataset/Loader**
- synth2track/data/datasets/synthetic_temporal_3d.py

**Path definitions**
- LocalPaths.json
    - Can be seen after in configs

**Evaluation**
- Diffusion steps
    - Hacked start_evaluation() (synth2track/evaluation/evaluator.py)
        - Added inner loop for diffusion
- Visualisation
    - synth2track/visualization/visualizer_presets.py

**Loss function**
- synth2track/training/loss_function_diffusion.py

**Normalisation**
- Steps
    - Define in the shared config
    - run ```python run_preprocessing.py``` (in launch.json already)
    - generates new normalisation.txt
- NOTE
    - Need to rerun these steps if more data is added to ```"TrainingDataset"``` in the shared config
    - Normalisation happens implicitly in training pipeline but has to be explicit in eval pipeline (for various reasons...)

**Note on trying new models**
- Have a new config if you change parameters
    - The config isn't copied over so eval won't work anymore if you train something then change the config
- 

**Visualise Blender**
- Run Evaluation
- Run Retargeting
    - Set launch.json retargeting paths
    - Run Retargeting Predictions/Ground Truth
- Import in blender
    - Open st2_to_blender.py in Vscode or blender scripting
    - Set ```input_path = ...```
    - Press arrow to run script

- Blender tips
    - ```.``` on keypad focuses on selected item
    - Can delete stuff in top right box

**Tensorboard**

```bash
cd /Documents/masters_thesis/bodytracking/summary/runs
tensorboard --logdir .
```
- model name is shown in the tensorboard so have nice model names