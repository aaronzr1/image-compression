# Image Compression

## Interesting Directions
- multimodal compression (compress the image but incorporate text)
    - text: maybe more useful for perceptual quality rather than info storing
- machine-readable over basic perceptual quality (image compression for ML)
    - what about reconstruction specifically for machine tasks, instead of perceptual (feels like distillation)
- compression that enables realtime and video performance
- beyond mere perceptual quality: enforcing idempotence could help? (less diversity from diffusion)

## Terms
- RD: rate-distoration tradeoff, compression rate (how small) vs distortion (how accurate post-reconstruction)
- FID: realism metric, appears to be at a tradeoff with MSE/PSNR (unproven)
- LPIPS: perceptual similarity to original metric
- MSE/PSNR: pixel-wise fidelity metrics

## Notable Papers

### Same Task on Paper, Different Way of Viewing Problem
- [towards real-time neural image compression with mask decay](https://openreview.net/pdf?id=XUxad2Gj40n): (realtime image compression)
    - angle: simplify encoder/decoder complexity, dynamically adjust this complexity based on latency constraints
    - evc: single model that can adjust bitrate, instead of requiring different models for different RD; enabled by what's describd below, can stop at different levels of refinement now
    - mask decay: large models can be reduced to simpler/smaller ones without retraining
        - extracts simplified version by disabling certain neurons/connections, model pruning
        - enables same model to be used across different hardware (vs. training different models for each)
    - progressive residual representation learning: gradually compress image in steps, using bits more efficiently
        - at each step, first encode most important features, then residuals (difference between this and original) at each step
        - better RD since more bits allocated to complex regions, uses less bits on redundant information
        - enables scalable encoding: dynamically adjust bitrate/quality
- [unified image compression method for human perception and multiple vision tasks](https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/09009.pdf): (compression for perceptual quality AND ML tasks)
    - angle: compressing images so the reconstructed is both visually similar, and works about the same on cv tasks
    - multi-task collaborative embedding (mtce): many expert ae learn latent rep for task, cross-task loss to balance with human perceptual
    - diffusion-based invariant knowledge learning (dikl): extract knowledge from observed tasks to generalize to the unseen
        - memory bank with L entries (representative features from latent space) -> aligned embedding
        - used as task condition for diffusion model (self-supervised denoising)
    - other mentioned work: scalable coding (multiple bitstreams, decode different resolutions)
- [toward image compression with perfect realism at ultra-low bitrates](https://openreview.net/pdf?id=ktdETU9JBg): (reconstruct perfectly normal photo but slightly different from original, like human memory)
    - angle: distortion-resistant image reconstruction independent of bitrate via diffusion models (and augmented with text encoding guideline)
        - perfect realism codec (no f-divergence/FID): original different from reconstructed, but can't tell which is the original
    - flow: image -> caption + ldm encoder -> hyper encoder (compress latent repr) -> quantized into finite set of learned discrete codes (like vq-vae)
    - decode: encoded image caption + compressed image -> conditional diffusion model (xT) -> ldm decoder -> reconstructed image

### Interesting Methods/Approaches
- [neural image compression with text-guided encoding for both pixel-level and perceptual fidelity](https://openreview.net/pdf?id=u8TZ9gm4im): (text-guided image compression for perceptual and pixel-accuracy)
    - angle: not just perceptually accurate, but also pixel-wise accurate (historically, this comes at the tradeoff of realism)
    - observation: text-based diffusion has too much diversity (good realism!), which is bad for pixel accuracy
    - text for encoding -> inform how humans perceive image, then standard decoders for accurate + low-diversity generation
- [idempotence and perceptual image compression](https://openreview.net/pdf?id=Cy5v64DqEF): (idempotence: adapting models for faster training, slower testing)
    - angle: wrangling existing unconditional generative models as image compressors by imposing idempotence (consistent compress/reconstruct) constraint
    - unconditional generative model (GAN, VAE): data -> latent -> new random data
        - invert this: map real data sample (image) back to model's latent space (which exact point in latent space) to reconstruct (similar) sample
    - implement by having idempotence loss (compare difference between multiple re-constructions); this is way faster in training but longer in testing
    - issues: time taken to reconstruct compressed data (inversion is computationally expensive), difficulty with high res (can split to patches, but then patchy)
- [frequency-aware transformer for learned image compression](https://openreview.net/pdf?id=HKGQDDTuvZ): (traditional compression ideas)
    - angle: instead of dct, learn to decompose image into frequency components via FAT and feed into the following pipeline
    - frequency-decomposition window attention (fdwm): choose good window shapes for dividing into frequency components
    - frequency-modulation feed forward network (fmffn) determines scaling factor for each frequency compoment
    - transformer-based channel-wise autoregressive model (t-ca): learn how to connect freq components, especially since they aren't independent (ex. strong edges/high freq correlate with brightness/low-freq)
- [high fidelity generative image compression](https://proceedings.neurips.cc/paper/2020/file/8a50bae297807da9e97722a0b3fd8f27-Paper.pdf): (gan-based perceptual compression, works well with high res too)
    - angle: gan-based compression for high-res: better than past approaches (even when they have higher bitrates)
    - metrics: nothing encapsulates well yet (written in 2020), but FID and KID are not bad
    - Note: should this be moved into background section?

### Background
- [learning-driven lossy image compression; A Comprehensive Survey](https://arxiv.org/pdf/2201.09240): (2022 survey paper)
    - angle: survey of image compression, first one compiling dedicated section on ML-based compression
    - model types surveyed: AE, VAE, CNN, RNN, GAN, PCA, fuzzy means clustering (up until 2022)

## General Schema Sketch (Constantly Updating)
- novel vanilla generative-based approaches
- low bitrate focus (optimizing for vs breaking from RD metric)
- the jpeg-inspired (frequency components, dct/dwt-related)
- to categorize: end goal (perceptual quality, machine tasks quality, reconstruction quality (idempotence))