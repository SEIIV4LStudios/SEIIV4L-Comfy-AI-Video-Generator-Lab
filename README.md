# SEIIV4L-Comfy-AI-Video-Generator-Lab
  - node-based visual AI workflow engine and backend.

# building a local image-to-video inference pipeline.


# Node Pipline starts below:
                    ┌──────────────────────┐
                    │ Load Diffusion Model │
                    │      Wan 2.2         │
                    └──────────┬───────────┘
                               ↓
                    ┌──────────────────────┐
                    │   ModelSamplingSD3   │
                    └──────────┬───────────┘
                               ↓
                         ┌───────────┐
                         │ KSampler  │
                         └─────┬─────┘
                               │
       ┌───────────────────────┼────────────────────────┐
       │                       │                        │
       │                       │                        │
┌──────▼──────┐       ┌────────▼─────────┐     ┌────────▼────────┐
│ Positive    │       │ Negative Prompt  │     │ Image → Latent  │
│ Prompt      │       │ Conditioning     │     │ Conditioning    │
└──────▲──────┘       └────────▲─────────┘     └────────▲────────┘
       │                       │                        │
       └──────────┬────────────┘                        │
                  │                                     │
          ┌───────▼───────┐                    ┌────────▼─────┐
          │ Text Encoder  │                    │ Load Image   │
          │ UMT5 / CLIP   │                    └──────────────┘
          └───────────────┘

                               ↓

                         Generated Latent
                               ↓
                         ┌────────────┐
                         │ VAE Decode │
                         └─────┬──────┘
                               ↓
                         Video Frames
                               ↓
                        ┌─────────────┐
                        │Create Video │
                        └──────┬──────┘
                               ↓
                         ┌──────────┐
                         │Save Video│
                         └──────────┘
# Pipeline End at Save Video:

# this is essentially the minimum complete AI-video generation pipeline.
  Every generation needs several different jobs performed:
    
MODEL
   ↓
INSTRUCTIONS
   ↓
SOURCE DATA
   ↓
GENERATION
   ↓
DECODING
   ↓
MEDIA
   ↓
OUTPUT

Instead of putting all of that into one giant black box, ComfyUI exposes them in an local environment as separate components. 
When these components are integrated into a complete local workflow the system can generate AI video content from prompts and source images.
 
 
ComfyUI + nodes + model = generation system
output = AI-generated content

What Each node does:

# Load Diffusion Model  - (wan2.2_ti2v_5B_fp16.safetensors)
     (The Load Diffusion Model, is basically the AI brain responsible for generating the visual content. It contains the trained neural-network weights.)

ComfyUI = factory
Wan 2.2 = machine inside factory
  a. Without the model, ComfyUI doesn't know how to generate your video.

# ModelSamplingSD3 - 
      (configures the model for the sampling system the model expects. Acts like an adapter/configuration layer between the model and generation process.)
          *The sampler can't just randomly process every diffusion model exactly the same way. Different model architectures expect different sampling behavior.
          
# Load CLIP / text encoder
  - (generates pixels)
  - Turns the input prompt into numerical representations the AI can understand.

  - Essentially:
     
  Human language
     ↓
  Text Encoder
     ↓
  Vectors / embeddings
     ↓
  AI-understandable instructions
    *This is why just feeding English text directly into the diffusion model isn't enough.

  properties of the text encoder:
  
    - Positive Text Encoder -  This tells the generator What should exist.
Example:
  - cinematic Audi R8
  - nighttime city
  - police pursuit
  - realistic lighting
  - camera tracking vehicle
    
    - Negative Text Encoder - This tells the system what characteristics we want it to avoid.
Example:
  - blurry
  - deformed
  - duplicate vehicle
  - bad anatomy
  - distorted wheels
        *Thats 2 different Text Encoders*

the Text Encoder prompt becomes conditioning information.
Then the at conditioning goes into the sampler.

# Load Image - This is where you upload images. The starting image establishes things such as:

  - character appearance
  - vehicle appearance
  - - clothing
  - environment
  - composition
  - starting frame

# Video Latent - (Wan22ImageToVideoLatent)
    (compressed internal representation of the image/video data that the diffusion model works with.)
  - performs much of its generation in a mathematical representation called latent space
  - 
- Essentially:

  Starting image
  +
  video dimensions
  +
  frame information
  +
  Wan conditioning

  in one function turns into (Video latent) see below:

     IMAGE
  ↓
  Wan Image-to-Video Latent
  ↓
  AI-compatible video representation

# Ksampler - the generation engine of the workflow.
    - node where the actual generative process really comes together.
    - It gradually transforms noise toward something matching the instructions.
    

It receives components like:
   
  Diffusion MODEL (Brains / neural network)
  +
  POSITIVE PROMPT (Prompt text: whats should happen/what to expext)
  +
  NEGATIVE PROMPT (Whats not included / what we dont want)
  +
  STARTING LATENT (Video Representation/compressed data function works with)
  +
  SEED 
  +
  STEPS
  +
  SAMPLER SETTINGS (Ensures that Different model architectures expect different sampling behavior.)
  
*and this ksampler function performs the diffusion process.*

settings such as:

  Steps
  CFG
  Seed
  Sampler
  Scheduler

can significantly affect generation.

# Load VAE - a translator, handles translation between pixels, frames, and videos

  Think of it as a translator:
  Pixel space
  ↔
  Latent space

# Vae Decode - where the mathematical representation becomes visible imagery.
    - It's generated latent information.
    - what we have still isn't really the final video you watch.
    - VAE Decode performs:

  Generated latent:
       ↓
       VAE
       ↓
  Actual image frames

# Create Video - assembles Frames according to a frame rate into video.
  Example:`
    16 FPS
    24 FPS
    30 FPS
  into  
    video

# Save Video - handles the output So the workflow has a clear endpoint.
  Example:
    Generated video
        ↓
   filename / directory / format
        ↓
    disk

# Architecture of infrastructure:
    -  The architecture actually has five major subsystems:
  # INPUT
    - Load Image
    - Prompt
    - Negative Prompt
  # UNDERSTANDING
    - Text Encoder
    - Image/Video conditioning
  # GENERATION
    - Wan 2.2
    - ModelSampling
    - KSampler
  # TRANSLATION
    - VAE
    - VAE Decode
  #OUTPUT
    - Create Video
    - Save Video

# Component connection Chart
    *the diagram shows how the basic components should connect to each other! 

    Click the link below:

https://github.com/SEIIV4LStudios/SEIIV4L-Comfy-AI-Video-Generator-Lab/blob/0d9564a8cd6e6562cc6ca203b34ae902c07095b2/text%20to%20video%20basic%20pipeline.pdf
    
That's the system.

 *At this point you have the skills to create a basic AI text to video Generator utilizing those components*

# AI Generator Performance

