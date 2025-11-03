# Lab 4 - Noise and Filtering

Short, section-by-section notes for the MATLAB script. Replace the two image placeholders with your generated figures. 

## Setup and input image

Closes figures, clears variables, and reads the `peppers.png` image. Converts to grayscale and to double precision so arithmetic and filtering behave predictably. 

```matlab
close all; clear; clc;
I = im2double(rgb2gray(imread('peppers.png')));
```

## 1) Add different noise types

Creates two noisy versions of the image: Gaussian noise with zero mean and variance 0.01, and salt & pepper noise with density 0.05. Displays a montage (rejilla de imágenes) with original, Gaussian, and salt & pepper for quick visual comparison. 

```matlab
I_gauss = imnoise(I,'gaussian',0,0.01);
I_sp = imnoise(I,'salt & pepper',0.05);
figure; montage({I, I_gauss, I_sp},'Size',[1 3]);
title('Original | Gaussian noise | Salt & pepper noise');
```

<img width="1045" height="290" alt="Figure_1" src="https://github.com/user-attachments/assets/79ab8a03-80c4-4139-8b5b-3258159f3ba9" />


## 2) Compute simple quality metrics

Computes mean squared error, MSE (error cuadrático medio), between each noisy image and the original, then prints the values for a quantitative baseline. 

```matlab
MSE_gauss = immse(I_gauss, I);
MSE_sp = immse(I_sp, I);
fprintf('MSE Gaussian: %.4f | MSE S&P: %.4f\n', MSE_gauss, MSE_sp);
```

## 3) Linear filtering (mean, Gaussian)

Applies two linear filters. First, a 3×3 average filter made with `fspecial('average',3)` is applied to both noise types using `imfilter` with `'replicate'` borders (duplicar bordes). Second, a 3×3 Gaussian kernel with σ = 0.7 is applied to the Gaussian-noisy image. 

```matlab
h_avg = fspecial('average',3);
I_avg_gauss = imfilter(I_gauss,h_avg,'replicate');
I_avg_sp = imfilter(I_sp,h_avg,'replicate');
h_gauss = fspecial('gaussian',[3 3],0.7);
I_gauss_gauss = imfilter(I_gauss,h_gauss,'replicate');
```

## 4) Non-linear filtering (median)

Uses a 3×3 median filter, `medfilt2`, on both noisy images. Shows a 2×2 montage comparing average vs median on salt & pepper (top row) and on Gaussian noise (bottom row). Median filtering is non-linear (no lineal) and often robust to outliers. 

```matlab
I_med_gauss = medfilt2(I_gauss,[3 3]);
I_med_sp = medfilt2(I_sp,[3 3]);
figure; montage({I_avg_sp, I_med_sp, I_avg_gauss, I_med_gauss},'Size',[2 2]);
title('Top: Avg vs Median (S&P) | Bottom: Avg vs Median (Gaussian)');
```

<img width="883" height="687" alt="Figure_2" src="https://github.com/user-attachments/assets/3a676063-ca6c-401c-bc4e-4fd05c982edc" />


## 5) Compare metrics after filtering

Prints post-filter MSE for the salt & pepper case, comparing average vs median to quantify which filter preserved the original image better under that noise. 

```matlab
fprintf('After filtering, MSE S&P avg=%.4f, med=%.4f\n', ...
    immse(I_avg_sp,I), immse(I_med_sp,I));
```

## 6) Reflections

- Median filtering is well suited for salt and pepper noise because it rejects outliers while keeping edge positions. Linear averaging tends to smear impulses into their neighbors.

- For zero-mean Gaussian noise, small Gaussian or mean filters reduce variance effectively, but there is a trade-off (compromiso) between noise removal and edge blur as kernel size grows.

- Edge preservation matters more than a single metric. MSE can prefer overly smooth results; adding PSNR and SSIM (medidas de similitud estructural) would give a fuller picture of perceived quality.

- Border handling with replication avoids artificial dark or bright rims, but reflective padding could preserve gradients better.

- Practical tuning: pick the smallest kernel or sigma that visibly stabilizes flat regions without rounding corners.

- Possible improvements: adaptive median for high impulse densities, bilateral or non-local means for edge-preserving smoothing (conservador de bordes), and Wiener filtering when noise statistics are known.
