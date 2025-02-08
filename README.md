# Image Compression

## General Schema Sketch
- novel vanilla generative-based approaches
- low bitrate focus (optimizing for vs breaking from RD metric)
- the jpeg-inspired (frequency components, dct/dwt-related)
- to categorize: end goal (perceptual quality, machine tasks quality, reconstruction quality (idempotence))

## Notable Papers
- image compression survey
	- [high fidelity generative image compression](https://proceedings.neurips.cc/paper/2020/file/8a50bae297807da9e97722a0b3fd8f27-Paper.pdf): (high-res gan perceptual compr)
		- angle: gan-based compression for high-res: better than past approaches (even when they have higher bitrates)
		- metrics: nothing encapsulates well yet (written in 2020), but FID and KID are not bad
	- [unified image compression method for human perception and multiple vision tasks](https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/09009.pdf): (simult perceptual/machine compression)
		- angle: compressing images so the reconstructed is both visually similar, and works about the same on cv tasks
		- multi-task collaborative embedding (mtce): many expert ae learn latent rep for task, cross-task loss to balance with human perceptual
		- diffusion-based invariant knowledge learning (dikl): extract knowledge from observed tasks to generalize to the unseen
			- memory bank with L entries (representative features from latent space) -> aligned embedding
			- used as task condition for diffusion model (self-supervised denoising)
		- other mentioned work: scalable coding (multiple bitstreams, decode different resolutions)
	- [idempotence and perceptual image compression](https://openreview.net/pdf?id=Cy5v64DqEF): (perceptual idempotence)
		- angle: wrangling existing unconditional generative models as image compressors by imposing idempotence (consistent compress/reconstruct) constraint
		- unconditional generative model (GAN, VAE): data -> latent -> new random data
			- invert this: map real data sample (image) back to model's latent space (which exact point in latent space) to reconstruct (similar) sample
		- implement by having idempotence loss (compare difference between multiple re-constructions); this is way faster in training but longer in testing
		- issues: time taken to reconstruct compressed data (inversion is computationally expensive), difficulty with high res (can split to patches, but then patchy)
	- [toward image compression with perfect realism at ultra-low bitrates](https://openreview.net/pdf?id=ktdETU9JBg): (perceptual compression)
		- angle: distortion-resistant image reconstruction independent of bitrate via diffusion models (and augmented with text encoding guideline)
			- perfect realism codec (no f-divergence/FID): original different from reconstructed, but can't tell which is the original
		- flow: image -> caption + ldm encoder -> hyper encoder (compress latent repr) -> quantized into finite set of learned discrete codes (like vq-vae)
		- decode: encoded image caption + compressed image -> conditional diffusion model (xT) -> ldm decoder -> reconstructed image
	- [frequency-aware transformer for learned image compression](https://openreview.net/pdf?id=HKGQDDTuvZ): (traditional compression)
		- angle: instead of dct, learn to decompose image into frequency components via FAT and feed into the following pipeline
		- frequency-decomposition window attention (fdwm): choose good window shapes for dividing into frequency components
		- frequency-modulation feed forward network (fmffn) determines scaling factor for each frequency compoment
		- transformer-based channel-wise autoregressive model (t-ca): learn how to connect freq components, especially since they aren't independent (ex. strong edges/high freq correlate with brightness/low-freq)
	- [learning-driven lossy image compression; A Comprehensive Survey](https://arxiv.org/pdf/2201.09240): (2022 survey paper)
		- angle: survey of image compression, first one compiling dedicated section on ML-based compression
		- model types surveyed: AE, VAE, CNN, RNN, GAN, PCA, fuzzy means clustering (up until 2022)