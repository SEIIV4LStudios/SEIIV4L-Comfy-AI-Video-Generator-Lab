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

# Latent Preparation Node - (Wan22ImageToVideoLatent)
    (compressed internal representation of the image/video data that the diffusion model works with.)
  - performs much of its generation in a mathematical representation called latent space
  - 
- Essentially:                                                                                                                                    

  Input Image                                                                                                                                            
        ↓
  Wan22ImageToVideoLatent
        ↓      
  Creates / prepares latent video representation                                                                    
        ↓                                                                                                                                  
  KSampler / Wan 2.2 diffusion process
        ↓                                                                                                                                      
  Generated video frames

 or

 IMAGE
↓
Wan Image-to-Video Latent
↓
AI-compatible video representation

or 




  
