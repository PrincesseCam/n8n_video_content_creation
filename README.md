# n8n Video Content Creation Automation (More info comming soon)

A complete, locally-hosted system for automating content creation workflows using n8n. This system leverages local AI models, FFmpeg for video processing, and coordinates the entire content creation pipeline from idea generation to publishing.

![n8n workflow system diagram](https://raw.githubusercontent.com/PrincesseCam/n8n_video_content_creation/main/docs/workflow_diagram.png)

## System Overview

This project automates the entire content creation process:

1. **Content Planning**: Generate content topics and ideas
2. **Script Generation**: Create optimized scripts for videos
3. **Audio Generation**: Convert scripts to voice narration using ElevenLabs
4. **Visual Generation**: Create images based on the script using local Stable Diffusion
5. **Video Assembly**: Combine audio and visuals with FFmpeg
6. **Multi-platform Publishing**: Prepare content for various social media platforms

## System Requirements

- **OS**: Windows 10/11 (with WSL2)
- **GPU**: NVIDIA 3050 with 8GB VRAM or better
- **Docker**: Version 27.5+ with GPU support
- **Storage**: At least 50GB free space
- **Memory**: 16GB RAM minimum (32GB recommended)

## Technologies Used

- **n8n**: Workflow automation and orchestration (v1.81.4+)
- **FFmpeg**: Audio/video processing (via Docker)
- **Stable Diffusion**: Image generation via AUTOMATIC1111 WebUI
- **ElevenLabs**: Text-to-speech API
- **Ollama**: Local large language models
- **PostgreSQL**: Metadata storage
- **Qdrant**: Vector database for content search
- **Docker**: Containerization for all components

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/PrincesseCam/n8n_video_content_creation.git
cd n8n_video_content_creation
```

### 2. Configure Environment
Refer to this repo for n8n, postgres and qdrant environnement settings:  
https://github.com/coleam00/ai-agents-masterclass/tree/main/local-ai-packaged

Update the `.env` file in the project root with the following contents:

```
POSTGRES_USER=n8n
POSTGRES_PASSWORD=yourpassword
POSTGRES_DB=n8n
N8N_ENCRYPTION_KEY=your-random-encryption-key
N8N_USER_MANAGEMENT_JWT_SECRET=your-random-jwt-secret
```

### 3. Start the Core Services

```bash
# Start n8n, PostgreSQL, FFmpeg, Qdrant
docker compose --profile gpu-nvidia up -d
```

### 4.Stable Diffusion WebUI
You can find the repo here:   
https://github.com/AbdBarho/stable-diffusion-webui-docker?tab=readme-ov-file  
Start Stable Diffusion WebUI in a separate terminal :

```bash
cd stable-diffusion-webui-docker
docker-compose --profile auto up -d
```

### 5. Set Up Directory Structure

The system expects a specific directory structure. Use the content planning workflow to automatically create this structure, or create it manually:

```
shared/
├── BG_music/                # Background music directory
├── configs/                 # Configuration files
├── templates/               # Template files
└── video_TIMESTAMP/         # Content directories (created automatically)
    ├── main/                # Scripts and metadata
    ├── audio/               # Voice narration files
    ├── images/              # Generated images
    ├── video/               # Processing directory
    └── final/               # Final video outputs
```

### 6. Import Workflows to n8n

1. Access n8n at http://localhost:5678
2. Create an account or log in
3. Go to Settings → Import From File
4. Import each workflow JSON file from the `workflows` directory

### 7. Configure Credentials

Set up the following credentials in n8n:

- **ElevenLabs API**: For voice generation
- **Google API** (optional): For YouTube publishing

## Workflow System Architecture

The system consists of multiple interconnected workflows that work together to create content:

### 1. Content Planning Workflow

- **Trigger**: Manual or scheduled
- **Purpose**: Generate content ideas based on specified niche and style
- **Key Features**:
  - Creates directory structure for each content project
  - Uses AI to generate trending topics
  - Initializes metadata for the content project

### 2. Script Generation Workflow

- **Trigger**: Webhook from Content Planning
- **Purpose**: Create comprehensive scripts for the selected topic
- **Key Features**:
  - Creates engaging script with proper sections
  - Generates visual descriptions for each section
  - Creates both JSON and TXT versions of the script

### 3. Audio Generation Workflow

- **Trigger**: Webhook from Script Generation
- **Purpose**: Convert script to high-quality voice narration
- **Key Features**:
  - Uses ElevenLabs API for lifelike voice narration
  - Normalizes audio volume
  - Generates subtitles in SRT and VTT formats
  - Measures accurate audio duration

### 4. Visual Generation Workflow

- **Trigger**: Webhook from Script Generation
- **Purpose**: Create images based on script sections
- **Key Features**:
  - Enhances visual prompts using AI
  - Uses local Stable Diffusion API
  - Manages GPU resources with cooldown periods
  - Saves images with sequential naming

### 5. Assembly Readiness Workflow

- **Trigger**: Webhooks from Audio and Visual workflows
- **Purpose**: Coordinate when both audio and visuals are ready
- **Key Features**:
  - Checks completion signals from both workflows
  - Triggers video assembly when all components are ready

### 6. Video Assembly Workflow

- **Trigger**: Webhook from Assembly Readiness
- **Purpose**: Combine audio and images into video
- **Key Features**:
  - Uses FFmpeg for professional video assembly
  - Adds background music with proper mixing
  - Creates versions for different platforms (YouTube, Instagram, TikTok)
  - Generates versions with burnt-in subtitles
  - Prepares publishing metadata

### 7. Publishing Workflow (Optional)

- **Trigger**: Webhook from Video Assembly
- **Purpose**: Publish videos to social media platforms
- **Key Features**:
  - Integrates with platform APIs
  - Customizes content for each platform
  - Tracks publishing status in database

## Usage

### Creating a New Video

1. Navigate to n8n and open the Content Planning Workflow
2. Click "Execute Workflow" and enter parameters:
   - **niche**: Topic area (e.g., "tech", "health", "finance")
   - **style**: Content style (e.g., "informative", "tutorial", "entertaining")
   - **targetLength**: Desired video length (e.g., "60s", "2m")
   - **publishTo[Platform]**: Boolean flags for publishing destinations

3. The system will automatically:
   - Create a new content directory
   - Generate topic ideas
   - Trigger the full content creation pipeline
   - Notify when each stage is complete

4. Final videos will be available in the `/shared/video_[TIMESTAMP]/final/` directory

### Monitoring Workflow Progress

Each workflow creates completion signals at `[content_dir]/main/`:
- `content_plan.json`: Initial content plan
- `script.json` and `script.txt`: Generated script
- `audio_complete.json`: Audio generation completed
- `visual_complete.json`: Visual generation completed
- `assembly_complete.json`: Video assembly completed
- `platform_metadata.json`: Publishing metadata

## Directory Structure Explained

Each content project is stored in a directory with this structure:

```
video_TIMESTAMP/
├── main/                   # Core content files
│   ├── content_plan.json   # Initial content plan
│   ├── script.json         # Script with metadata
│   ├── script.txt          # Plain text script
│   ├── subtitles.srt       # Subtitles in SRT format
│   └── subtitles.vtt       # Subtitles in WebVTT format
├── audio/                  # Audio files
│   └── narration.mp3       # Voice narration
├── images/                 # Generated images
│   ├── image00.png         # Section image 1
│   ├── image01.png         # Section image 2
│   └── ...
├── video/                  # Processing directory
│   └── temp/               # Temporary files
└── final/                  # Final video outputs
    ├── video_TIMESTAMP_final.mp4       # Original version
    ├── youtube_fb_version.mp4          # YouTube/Facebook (16:9)
    ├── insta_tiktok_version.mp4        # Instagram/TikTok (9:16)
    ├── insta_square_version.mp4        # Instagram Square (1:1)
    ├── youtube_subtitled.mp4           # With burnt-in subtitles
    ├── insta_subtitled.mp4             # With burnt-in subtitles
    └── square_subtitled.mp4            # With burnt-in subtitles
```

## Customization

### Background Music

Store background music files in the `shared/BG_music/` directory. The system will randomly select a track for each video.

### Voice Settings

Modify voice settings in the Audio Generation workflow:

1. Open the "Prepare Voice Settings" node
2. Update the `elevenlabsVoiceId` with your preferred voice
3. Adjust `stability` and `similarity_boost` parameters

### Image Generation Settings

Optimize image generation in the Visual Generation workflow:

1. Open the "Prepare Image Prompts" node
2. Adjust the `imageSettings` parameters:
   - `width`, `height`: Image dimensions
   - `steps`: Generation steps (higher = better quality, but slower)
   - `cfgScale`: Prompt adherence (higher = closer to prompt)
   - `sampler`: Stable Diffusion sampling method

## Troubleshooting

### Webhook Connectivity Issues

If workflows are not triggering each other:

1. Use `host.docker.internal` instead of `localhost` in webhook URLs
2. Ensure all workflows are activated
3. Check webhook paths match between sender and receiver

### Image Generation Issues

If images fail to generate:

1. Check Stable Diffusion WebUI is running (`http://localhost:7861`)
2. Ensure API is enabled (with `--api` flag)
3. Monitor GPU memory usage
4. Increase cooldown time between generations

### Video Assembly Errors

If video assembly fails:

1. Check FFmpeg logs for specific errors
2. Verify all images and audio files exist
3. Try the alternative assembly methods in the workflow
4. Ensure subtitles file is valid

## Performance Optimization

For NVIDIA 3050 8GB GPUs:

1. **Reduce Image Size**: Use 768x768 instead of 1024x1024
2. **Optimize Steps**: Use 20-30 steps for image generation
3. **Sequential Processing**: Generate images one at a time
4. **GPU Cooldown**: Include 20-30 second pauses between image generations

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- n8n team for the amazing workflow automation tool
- AUTOMATIC1111 for the Stable Diffusion WebUI
- ElevenLabs for the voice synthesis API
- FFmpeg team for the powerful media processing tools
