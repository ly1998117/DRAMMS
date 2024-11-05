# DRAMMS with Standard Deforamtion Field
+ I modify the source code of DRAMMS-1.5 to output the standard deformation field (SDF) in the NIfTI format. 
+ The SDF is also useful for comparing deformation fields across different algorithms. 

## Modifications

I modified the source code of DRAMMS-1.5 in the following files:
  + `src/common/utilities.cxx:ApplyTransform`
  + `src/tools/ApplyTransform.cxx:dir_name`

The x,y,z components of the SDF are saved in the 1st, 2nd, and 3rd channels of the NIfTI file, respectively. When using DRAMMS for registration, the SDF will be saved as :
  + `DFx.nii.gz  DFy.nii.gz  DFz.nii.gz`

For example, using the following command:
```bash
dramms --source a.nii.gz --target MNI152.nii.gz --outimg src2trg.nii.gz --outdef def_src2trg.nii.gz
```
The SDF will be saved as `DFx.nii.gz  DFy.nii.gz  DFz.nii.g` in same directory as `src2trg.nii.gz`.

## Installation
Follow the instructions in the original DRAMMS-1.5 Manual to install the software. 
+ [DRAMMS-1.5 Manual](https://www.cbica.upenn.edu/sbia/software/dramms/_downloads/DRAMMS_Software_Manual.pdf)

## Usage
After saving x,y,z components of the SDF, you can use the following command to combine them into a single NIfTI file:
```python
import os
import nibabel as nib
import numpy as np
df = []
for direction in ['z', 'x', 'y']:
        df.append(nib.load(f'DF{direction}.nii.gz').get_fdata())
df = np.stack(df, axis=-1)
nib.save(nib.Nifti1Image(df, np.eye(4)), 'DF.nii.gz')
```
The combined SDF will be saved as `DF.nii.gz`. For using the SDF, you can follow this file: `EXAMPLES/RegistrationTorch.ipynb`.
Part of the code is shown below:
```python
def generate_grid(imgshape):
    grid = np.mgrid[0:imgshape[0], 0:imgshape[1], 0:imgshape[2]].transpose(1, 2, 3, 0)[..., [2, 1, 0]]
    grid = torch.from_numpy(grid).unsqueeze(0).float()
    return grid
    
def transform(x, flow):
    grid = generate_grid(flow.shape[1:])
    grid = grid + flow
    grid[0, :, :, :, 0] = (grid[0, :, :, :, 0] - ((grid.size()[3] - 1) / 2)) / (grid.size()[3] - 1) * 2
    grid[0, :, :, :, 1] = (grid[0, :, :, :, 1] - ((grid.size()[2] - 1) / 2)) / (grid.size()[2] - 1) * 2
    grid[0, :, :, :, 2] = (grid[0, :, :, :, 2] - ((grid.size()[1] - 1) / 2)) / (grid.size()[1] - 1) * 2
    x = torch.nn.functional.grid_sample(x, grid, mode='bilinear', align_corners=True)
    return x
```