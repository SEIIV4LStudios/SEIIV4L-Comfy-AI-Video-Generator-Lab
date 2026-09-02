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
  *https://github.com/SEIIV4LStudios/SEIIV4L-Comfy-AI-Video-Generator-Lab/blob/0d9564a8cd6e6562cc6ca203b34ae902c07095b2/text%20to%20video%20basic%20pipeline.pdf*
    
  That's the system.
      *At this point you have the skills to create a basic AI text to video Generator utilizing those components*

# (Development currently @):
# AI Generator Performance
    *Now that you have your AI video engine generator, you must understand There are individual stages to performance/system productivity.*
  
  For example:
  Load model     → RAM / VRAM / disk
  Text encode    → CPU/GPU compute
  Image encode   → compute + memory
  KSampler       → HUGE compute load
  VAE Decode     → compute + memory
  Video encode   → CPU/GPU encoding
  Save           → disk

*we're trying to move toward:*

  generate
  ↓
  generate faster
  ↓
  short video
  ↓
  continuous videos
  ↓
  near-real-time generation
  ↓
  eventually streaming

  Trouble Shoot 1:
  *Spotted a flaw in in the infrastructure workflow, pattern bottleneck using consensus client/execution client explanation flow reveals
  Every additional blocking validation layer creates:*
  
    decision
     ↓
    wait
     ↓
    decision
     ↓
    wait
     ↓
    generation
     ↓
    wait
     ↓
    validation

  the design uses consensus client's for too many different responsibilities. so alternatively different stages need access to one shared consistency authority.
  So now the infrastructure should ensure there is one Coordinated State Of Authority:

ONE STATE AUTHORITY
        │
        ├───────────────┐
        ↓               ↓
    Generation     Observability
        ↓               ↑
    Quality Check ───────┘

  Much faster.

*Click link below* to view One State Authority Pipeline design that will essentially allow coordination between each state change simutaneously.

    https://github.com/SEIIV4LStudios/SEIIV4L-Comfy-AI-Video-Generator-Lab/blob/30ee59957abc753b9a737f981b43a85d18cf3888/SEIIV4L_AI_Video_Optimized_performance_Pipeline_Architecture.pdf
  
    # separation is way stronger than giving every stage its own “consensus client.”

# Clean Keyframe
    - That should be a major checkpoint in the architecture.Instead of sending questionable images farther down the expensive video pipeline:
    - If the generated scene has the wrong perspective, wrong car, wrong character, etc., it gets rejected before Wan burns GPU time generating an entire clip.

  Scene Request
     ↓
  Generate/Select Reference
     ↓
  CLEAN KEYFRAME GATE
     ↓
  only then
     ↓
  Wan I2V 

# Scene Inversion (function here)

# Amazon AWS (EC2) - on-demand, scalable computing capacity in the Amazon Web Services (AWS) Cloud.
   - use Amazon EC2 to launch as many or as few virtual servers as you need, configure security and networking, and manage storage.
   - You can add capacity (scale up) to handle compute-heavy tasks, such as monthly or yearly processes, or spikes in website traffic. When usage decreases, you can reduce capacity (scale down) again.

*Amazon AWS integrated with this workflow. To improve performance, productivity, and authorization in workflow automations. Amazon EC2 provides the following features:
   
    Instances:
      Virtual servers.
    Instance types:
      Various configurations of CPU, memory, storage, networking capacity , and graphics hardware for your instances.
    Amazon Machine Images (AMIs):
      Preconfigured templates for your instances that package the components you need for your server (including the operating system and additional software).
    Amazon EBS volumes:
      Persistent storage volumes for your data using Amazon Elastic Block Store (Amazon EBS).
    Instance store volumes:
      Storage volumes for temporary data that is deleted when you stop, hibernate, or terminate your instance.
    Key pairs:
      Secure login information for your instances. AWS stores the public key and you store the private key in a secure place.
    Security groups:
      A virtual firewall that allows you to specify the protocols, ports, and source IP ranges that can reach your instances, and the destination IP ranges to which your instances can         connect
  
*Amazon EC2 supports the processing, storage, and transmission of credit card data by a merchant or service provider, and has been validated as being compliant with Payment Card Industry (PCI) Data Security Standard (DSS). For more information about PCI DSS, including how to request a copy of the AWS PCI Compliance Package, see PCI DSS Level*

*Services to use with Amazon EC2:
    - AWS Systems Manager - Perform operations at scale on EC2 instances with this secure end-to-end management solution.
    - EC2 Image Builder - Automate the creation, management, and deployment of customized, secure, and up-to-date server images.
    - Amazon CloudWatch - Monitor your instances and Amazon EBS volumes.
    - Amazon GuardDuty - Detect potentially unauthorized or malicious use of your EC2 instances.

You can use other AWS services with the instances that you deploy using Amazon EC2.SEE
services to use with Amazon. Shttps://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html*
    - *See additional services and Pricing & Billing.*
    - *see Getting started with amazon EC2: (link)   *


  
