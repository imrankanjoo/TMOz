# Perceptual Tone Mapping Model for High Dynamic Range Imaging

**Digital Object Identifier:** [https://doi.org/10.1109/ACCESS.2023.3320809](https://doi.org/10.1109/ACCESS.2023.3320809)

> **Note:** This code is not optimized for processing speed.

## Table of Contents

1. [Requirements](#requirements)
2. [Repository Structure](#repository-structure)
3. [How to Run the Code](#how-to-run-the-code)
4. [Expected Output](#expected-output)
5. [Parameters](#parameters)
6. [Proposed Model](#proposed-model)
7. [Results](#results)
8. [References](#references)
9. [License](#license)

## Requirements

Before running the code, make sure you have the following installed:

- MATLAB
- Image Processing Toolbox
- HDR Toolbox, or equivalent support for `hdrread` and `hdrwrite`
- The utility functions included in the `utils` folder

## Repository Structure

After downloading or cloning the repository, the folder should look like this:

```text
TMOz/
│
├── HDR/
│   └── input HDR images, for example Peppermill.hdr
│
├── output/
│   └── generated tone-mapped images
│
├── utils/
│   └── utility functions required by the tone mapping model
│
├── TMOz.m
└── readme.txt / README.md
```

`TMOz.m` is the main MATLAB script used to run the tone mapping model.

## How to Run the Code

### 1. Download the repository

Download or clone this repository to your computer.

If using Git, run:

```bash
git clone <repository-url>
cd TMOz
```

Alternatively, download the repository as a ZIP file and extract it.

### 2. Open the project in MATLAB

Open MATLAB and set the current folder to the repository folder that contains `TMOz.m`.

For example, the MATLAB current folder should be the folder containing:

```text
TMOz.m
HDR/
output/
utils/
```

### 3. Check the required folders

Make sure these folders exist in the same directory as `TMOz.m`:

```text
HDR
output
utils
```

The `HDR` folder should contain the input HDR image.

The `output` folder is used to save the final tone-mapped image.

The `utils` folder contains helper functions used by the tone mapping model.

### 4. Add or choose an HDR image

Place your HDR image inside the `HDR` folder.

For example:

```text
HDR/Peppermill.hdr
```

The example script uses:

```matlab
imgName = 'Peppermill';
```

Do not include the `.hdr` extension in `imgName`.

### 5. Open `TMOz.m`

Open the file `TMOz.m` in MATLAB.

Inside the script, check or edit the image name:

```matlab
imgName = 'Peppermill';
```

Change `Peppermill` to match the name of your HDR image.

For example, if your image is:

```text
HDR/MyHDRImage.hdr
```

then set:

```matlab
imgName = 'MyHDRImage';
```

### 6. Set the tone mapping conditions

The script uses the following default tone mapping conditions:

```matlab
cond.XYZw1 = [95.047, 100.00, 108.883];  % D65 white point for HDR image
cond.Lw1 = 10000;                        % Luminance for HDR image
cond.Yb = 20;                            % Background luminance
cond.XYZw2 = [95.047, 100.00, 108.883];  % White point for output display
cond.Lw2 = 100;                          % Luminance for output image
cond.sr = 'avg';                         % Tone mapping method
```

You can keep these default values for a basic run.

### 7. Run the script

Run `TMOz.m` from the MATLAB editor by clicking **Run**.

You can also run it from the MATLAB Command Window:

```matlab
TMOz
```

### 8. What the script does

The script performs the following steps:

1. Clears the MATLAB workspace.
2. Adds the `utils` and `HDR` folders to the MATLAB path.
3. Loads the selected HDR image using `hdrread`.
4. Converts the HDR image to double precision.
5. Resizes the HDR image for lower-resolution processing.
6. Replaces zero or near-zero HDR values to avoid numerical issues.
7. Optionally writes a low-resolution HDR file.
8. Applies the tone mapping operator using `TMOz2CAM16Q`.
9. Displays the tone-mapped image.
10. Saves the final SDR image to the `output` folder.

### 9. Example MATLAB code

```matlab
% Clear workspace variables
clearvars;

% Add utility paths
addpath(genpath('utils'));
addpath(genpath('HDR'));

%% Conditions for Tone Mapping
cond.XYZw1 = [95.047, 100.00, 108.883];  % D65 white point for HDR image
cond.Lw1 = 10000;                        % Luminance for HDR image
cond.Yb = 20;                            % Background luminance
cond.XYZw2 = [95.047, 100.00, 108.883];  % White point for sRGB image or display
cond.Lw2 = 100;                          % Luminance for output image
cond.sr = 'avg';                         % Tone mapping method

%% Load HDR Image and Apply Tone Mapping
imgName = 'Peppermill';                  % Base name of the HDR image file
hdr = double(hdrread([imgName, '.hdr'])); % Read input HDR radiance map

% Resize image for faster processing
hdr = imresize(hdr, 0.2);

% Avoid zero or near-zero values in the HDR image
hdr(hdr <= 0) = 0.0001;

% Write a low-resolution HDR image, optional
hdrwrite(hdr, 'PeppermillLow.hdr');

% Apply tone mapping operator
sdr = TMOz2CAM16Q(hdr, cond);

%% Display and Save the Result
figure, imshow(sdr, 'border', 'tight');

if ~exist('output', 'dir')
    mkdir('output');
end

imwrite(sdr, fullfile('output', [imgName '.png']));
```

## Expected Output

After running `TMOz.m`, MATLAB should display the tone-mapped image.

The result will also be saved as a PNG image in the `output` folder.

For the default example, the output file will be:

```text
output/Peppermill.png
```

If the script writes the low-resolution HDR image, it will also create:

```text
PeppermillLow.hdr
```

## Parameters

- `cond.XYZw1`: D65 white point for the HDR image. Default: `[95.047, 100.00, 108.883]`
- `cond.Lw1`: Luminance for the HDR image. Default: `10000`
- `cond.Yb`: Background luminance. Default: `20`
- `cond.XYZw2`: White point for the sRGB image or display. Default: `[95.047, 100.00, 108.883]`
- `cond.Lw2`: Luminance for the output image. Default: `100`
- `cond.sr`: Tone mapping method. Default: `'avg'`

## Proposed Model

The input to TMOz is HDR image data. The model uses CIECAM16 to compute perceptual attributes such as brightness, colorfulness, and hue.

### Workflow

![Workflow](Images/workflow.jpg)

### Brightness Calculation

The algorithm calculates standard CIECAM16 parameters, including adaptation luminance and background luminance, which are required for predicting appearance attributes.

### Brightness Decomposition

Brightness is decomposed into base and detail layers. The base layer is tone-compressed, while the detail layer helps preserve local contrast and reduce halo artifacts.

### Colorfulness and Hue Calculation

Hue is computed directly from the HDR image. Colorfulness is adapted using CIECAM16 under display conditions to improve the color reproduction of the final tone-mapped image.

## Results

### Visual Comparison

Examples comparing TMOz with other tone mapping operators are shown below.

#### Example 1

![Visual Comparison 1](Images/example1.jpg)

#### Example 2

![Visual Comparison 2](Images/example2.jpg)

#### Example 3

![Visual Comparison 3](Images/example3.jpg)

#### Example 4

![Visual Comparison 4](Images/example4.jpg)


## References

Mehmood, I., Shi, X., Khan, M. U., & Luo, M. R. (2023). Perceptual Tone Mapping Model for High Dynamic Range Imaging. *IEEE Access*, 11, 110272–110288.

DOI: [https://doi.org/10.1109/ACCESS.2023.3320809](https://doi.org/10.1109/ACCESS.2023.3320809)

## Cite

```bibtex
@article{mehmood2023perceptual,
  title={Perceptual Tone Mapping Model for High Dynamic Range Imaging},
  author={Mehmood, I. and Shi, X. and Khan, M. U. and Luo, M. R.},
  journal={IEEE Access},
  volume={11},
  pages={110272--110288},
  year={2023},
  doi={10.1109/ACCESS.2023.3320809}
}
```

## License

This work is licensed under the IEEE Access License Agreement. For more information, refer to the IEEE Access website and the specific licensing terms applicable to this publication.
