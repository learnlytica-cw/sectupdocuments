```bash
# setup.sh
#!/bin/bash

echo "Setting up Video Learning Platform..."

# Create directory structure
mkdir -p videos/module1 videos/module2 videos/thumbnails reviews/feedback config

# Initialize Git LFS
git lfs install
git lfs track "*.mp4"
git lfs track "*.webm"
git add .gitattributes

# Create initial config file
cat > config/config.js << EOL
const config = {
    videosPath: 'videos/',
    reviewsPath: 'reviews/',
    maxVideoSize: 100000000 // 100MB in bytes
};
EOL

# Create .gitignore
cat > .gitignore << EOL
config/secrets.js
node_modules/
.env
*.log
.DS_Store
EOL

# Optional: Create a GitHub Actions workflow for video processing
mkdir -p .github/workflows
cat > .github/workflows/video-process.yml << EOL
name: Video Processing
on:
  push:
    paths:
      - 'videos/**'
jobs:
  process-video:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          lfs: true
      - name: Process Videos
        run: |
          sudo apt-get update
          sudo apt-get install -y ffmpeg
          for video in videos/**/*.mp4; do
            ffmpeg -i "$video" -vf "scale=1280:720" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k "processed_${video##*/}"
            mv "processed_${video##*/}" "$video"
          done
EOL

echo "Setup completed successfully!"
```

2. Video Processing Script (process_videos.py):

```python
# process_videos.py
import os
import subprocess
from pathlib import Path

def process_video(input_path, output_path, target_size_mb=95):
    """Process video to meet GitHub file size limits."""
    
    # Get video info
    probe = subprocess.run([
        'ffprobe', '-v', 'quiet',
        '-print_format', 'json',
        '-show_format', '-show_streams',
        input_path
    ], capture_output=True, text=True)
    
    # Calculate bitrate for target size
    duration = float(probe.stdout.split('"duration":"')[1].split('"')[0])
    target_bitrate = int((target_size_mb * 8192) / duration)
    
    # Process video
    subprocess.run([
        'ffmpeg', '-i', input_path,
        '-c:v', 'libx264',
        '-b:v', f'{target_bitrate}k',
        '-pass', '1',
        '-f', 'null',
        '/dev/null'
    ])
    
    subprocess.run([
        'ffmpeg', '-i', input_path,
        '-c:v', 'libx264',
        '-b:v', f'{target_bitrate}k',
        '-pass', '2',
        output_path
    ])
    
    # Generate thumbnail
    thumbnail_path = output_path.replace('.mp4', '_thumb.jpg')
    subprocess.run([
        'ffmpeg', '-i', input_path,
        '-ss', '00:00:01',
        '-vframes', '1',
        thumbnail_path
    ])

def main():
    videos_dir = Path('videos')
    for video_path in videos_dir.rglob('*.mp4'):
        output_path = video_path.parent / f'processed_{video_path.name}'
        process_video(str(video_path), str(output_path))

if __name__ == '__main__':
    main()
```

3. Complete Deployment Guide:

```markdown
# Video Learning Platform Deployment Guide

## Prerequisites
1. Git and Git LFS installed
2. GitHub account with LFS enabled
3. FFmpeg installed for video processing
4. Python 3.7+ for video processing script

## Setup Steps

1. Create GitHub Repository:
   ```bash
   git init
   git lfs install
   git lfs track "*.mp4"
   git lfs track "*.webm"
   ```

2. Run Setup Script:
   ```bash
   chmod +x setup.sh
   ./setup.sh
   ```

3. Install Required Tools:
   ```bash
   # Ubuntu/Debian
   sudo apt-get update
   sudo apt-get install -y ffmpeg python3-pip
   
   # macOS
   brew install ffmpeg python3
   
   # Windows (using chocolatey)
   choco install ffmpeg python3
   ```

4. Process Videos:
   ```bash
   python3 process_videos.py
   ```

5. Configure GitHub Pages:
   - Go to repository settings
   - Navigate to Pages section
   - Select main branch as source
   - Choose root (/) as folder
   - Save settings

6. Deploy:
   ```bash
   git add .
   git commit -m "Initial deployment"
   git push origin main
   ```

## File Size Guidelines

1. Recommended video specifications:
   - Resolution: 1280x720 (720p)
   - Codec: H.264
   - Container: MP4
   - Bitrate: Adaptive (based on length)
   - Maximum size: 95MB (to stay under GitHub's 100MB limit)

2. Video processing options:
   a. Split longer videos:
      ```bash
      ffmpeg -i input.mp4 -c copy -segment_time 00:15:00 -f segment output%03d.mp4
      ```
   
   b. Compress videos:
      ```bash
      ffmpeg -i input.mp4 -vf "scale=1280:720" -c:v libx264 -crf 23 -c:a aac -b:a 128k output.mp4
      ```

## Directory Structure
```
your-repo/
├── index.html
├── videos/
│   ├── module1/
│   ├── module2/
│   └── thumbnails/
├── reviews/
│   └── feedback/
├── config/
│   ├── config.js
│   └── secrets.js (gitignored)
├── .gitattributes
├── .gitignore
└── .github/
    └── workflows/
        └── video-process.yml
```

## Configuration

1. Create secrets.js (add to .gitignore):
```javascript
// config/secrets.js
const secrets = {
    githubToken: 'your-token',
    apiKeys: {
        tinymce: 'your-tinymce-key'
    }
};
```

2. Update index.html:
   - Add secrets.js before main script
   - Update config object with secrets

## Maintenance

1. Adding new videos:
   ```bash
   # Process new video
   python3 process_videos.py
   
   # Push to repository
   git add videos/
   git commit -m "Add new video content"
   git push origin main
   ```

2. Updating content:
   - Edit index.html for UI changes
   - Update config files as needed
   - Use GitHub Actions for automated processing

## Troubleshooting

1. LFS Issues:
   ```bash
   # Reset LFS
   git lfs uninstall
   git lfs install
   git lfs pull
   ```

2. Video Processing:
   ```bash
   # Check video info
   ffprobe -v quiet -print_format json -show_format -show_streams video.mp4
   ```

3. GitHub Pages:
   - Check repository settings
   - Verify main branch is selected
   - Wait for deployment (can take a few minutes)
```

Would you like me to:
1. Add more automation scripts?
2. Include additional video processing options?
3. Add monitoring and analytics?
4. Create a troubleshooting guide?

Let me know what else you need!
