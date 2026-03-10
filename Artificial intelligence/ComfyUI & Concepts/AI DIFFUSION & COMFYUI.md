- Comfy UI consists of many nodes that are programmed functions that execute tasks

- First node to encounter is Checkpoint , this node loads the checkpoint ( AI MODEL )

- A diffusion model checkpoint typically includes:
	- **Text encoder** → understands prompts
	- **U-Net** → generates the image
	- **VAE** → converts between latent space and pixels
	- **Tokenizer** → processes text
	- **Scheduler config** → controls the denoising process

- Diffusion is the process of generating images where the model changes the static noise step by step into a more accurate image , each step redefines the structure more and makes a better image
	![[Pasted image 20260309194448.png|661]]

- The `ksampler` is the node which tunes the diffusion model so we can generate the images we want , it has the number of steps , CFG , seed , denoise strength , sampler algorithm

-  What the `KSampler` Actually Does
	During generation:
	
	- **Start with random noise** (identified by the seed)
	- The **`KSampler`** begins the diffusion loop.
	- At each step:
	    - It sends the current noisy latent to the **U-Net inside the checkpoint**
	    - The model predicts **what noise should be removed**
	- The **`KSampler` applies the sampler algorithm** (Euler, DPM++, etc.)
	- Repeat for **N steps** (Remove more noise and redefine each step)
	- The final latent is sent to the **VAE decoder** to produce the image.
	
	The **`KSampler` controls this loop**.

- Seed in the `Ksampler` is what identifies the starting point of the noise , fixed seed will generate the same image since its the same seed (some changes may change the seed output even if fixed)

- **How does a diffusion model know how to identify static noise that seem impossible to identify ?** 
	1. the model is shown multiple images paired with description
	2. Then noise is gradually added to the images (Forward diffusion) until the image become pure noise
	3. The model then learns how different objects form different noises and it can identify them by their noise (goal isnt to memorize, goal is to detect patterns)
	4. Then we can use that knowledge into generating images (Reverse diffusion)
	
	![[Pasted image 20260309200020.png]]

- **How does the model generate images?** we explained previously forward diffusion then we use the knowledge that the model has to reverse the process (Reverse diffusion):
	 1. we start with pure noise (Random seed)
	 2. the model starts to denoise it step by step
	 3. at the end it removes the noise revealing the content

- Forward diffusion vs Reverse diffusion:
	![[Pasted image 20260309200356.png]]

- The prompt activates different learned patterns in the model , that why images looks similar but different images 

- Diffusion models work in latent space instead of pixel space ( pixel space is images in normal pixels we see ) , latent space is a compressed representation of the image it keeps the basic structure of the image but removes unnecessary details. It uses less memory , diffusion faster , less processing

- Diffusion models changes pixel space into latent space using a VAE (Variational auto encoder) that encodes images and decodes images , 

- Diffusion and VAE:
	![[CAE.png]]

- During diffusion the prompt is used in every denoising step , each step the model checks if the image is moving closer to what the prompt describe, the prompt influence is called CFG (classified free guidance ) , low CFG makes the model follows the prompt loosely and more randomness , high CFG is the other way , TOO high CFG can produce images that are overly sharpened and not natural

- We talked about seeds enough , seed defines what the initial noise looks like:
	![[Pasted image 20260309201938.png]]

- Prompt tweaks can be used to influence the priority of the description needed:
	 - order matters, earlier words influence more
	 - some words have more weight , words have more weight depending on the training of that model
	 - brackets puts more weight to words (word1) ((word2)) (((word3))) word1 < word2 < word3

- Steps control the number of denoise and more details added:
	![[Pasted image 20260309202358.png]]

- CLIP or text encoder just changes text into numbers that the model can understand, usually diffusion models take positive prompt to follow and negative prompt to avoid

- While the noise is being processed there are 2 systems that control how they are being processed ( Sampler , Scheduler ). The sampler decides how a noise is removed (algorithm used for denoising). Scheduler splits denoising tasks among steps based on the algorithm used for example liner Scheduler splits work evenly among all the steps  
	![[Pasted image 20260309202830.png]]

- LORAS (Low rank adaptations) its a small addon that modifies how a base model behaves , instead of training a new model from scratch we use LORAS to change the behavior of the model by teaching it something new
	![[Pasted image 20260309203411.png]]

- Control Net , they let you guide the model generation by another image not just prompt allowing more precise images for complex patterns its like providing a sample to an engineer that he can use to replicate
	![[Pasted image 20260309203453.png]]
	Control Nets have a preprocessor that changes the provided image to a image that the control net can understand and apply

- Checkpoints, these as we said are the models we use , there are multiple types of models out there ( AIO, FP16 , FP8 , GGUF , others ) they are large collections of numbers representing what the models learned same knowledge can be the same across multiple models but how these numbers are stored are different making the model act differently this exists to balance between speed , resources usage , compatibility ( some formats are larger and precise and lower are less precise but faster ), FP32 is the highest but require very high VRAM , FP8 is the lowest providing lower resolution and less precise images but low demand and high speed generations . AIO models consists of everything (Base model , VAE , CLIP ) all in single file. GGUF is a GPT generated unified format which is a very low resource demand AI model that can even run on CPUS 

- Batch size define how many images to be generated at a time (it will consume more resources)

```c
COOL NODES

Load CheckPoint - load checkpoint of AIO model
Load Diffusion Model - Load diffusion model
Load CLIP - load text encoder
Load VAE - load VAE
Load Tokenizer - load tokenizer for text prompts
Load Scheduler - manage diffusion step scheduling

Model sampling aura flow - control strength of diffusion model
Ksampler - sampler
Noise Injector - add noise to latent for creative variation
Latent Mixer - blend multiple latent representations

Empty sd3 latent image - set image resolution and size before creation

VAE encode - encode to latent
VAE decode - decode to pixel
Pixel Inspector - inspect pixel values of generated image
Latent Inspector - view latent space representation

Prompt Enhancer - refine text prompt for better results
Seed Manager - set or randomize seed for reproducibility
Output Saver - save generated images with metadata
Preview Node - preview intermediate results

Conditioning zero out - zero out a prompt

SDXL Empty Latent image - change image resolution (rgthree)
Flux Resolution Calc - change image resolution and megapixels (rgthree)
```

```C
NOTES

CFG 1 disables negative prompt

```