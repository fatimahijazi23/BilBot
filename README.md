# 🤖 BilBot - Lebanese Arabic Voice-Controlled Object Retrieval Robot

BilBot is an AI-powered autonomous robot designed to interpret Lebanese Arabic voice commands, identify requested objects in its environment, and move toward them. It integrates advanced technologies like GPT-4 (via OpenAI), Whisper, computer vision, inverse kinematics, and Telegram notifications to deliver a fully interactive experience.

---

## Features

- **Arabic Voice Recognition**: Captures and transcribes voice commands in Lebanese Arabic using OpenAI Whisper.
- **Arabic to English Translation**: Translates Lebanese Arabic into English using GPT-4.
- **Object Extraction & Synonym Detection**: Extracts the object name from speech and generates a dynamic list of synonyms via GPT-4.
- **Image Inspection via GPT-4 Vision**: Takes snapshots and asks GPT-4 if the object appears in the image.
- **Object Location Detection**: Determines relative position of objects (e.g. "left", "right", etc.) using GPT-4 Vision.
- **Autonomous Movement**: Uses mecanum wheels and sonar sensor to navigate toward the object.
- **Arabic Text-to-Speech (TTS)**: Converts status messages and prompts to spoken Arabic using gTTS.
- **Telegram Bot Integration**: Sends messages and photos to a Telegram chat for real-time updates.

---

## Requirements

### Hardware

- Raspberry Pi with Linux
- Hiwonder Arm & Mecanum Wheel Chassis
- Sonar Sensor
- USB Microphone
- Camera (e.g., PiCamera or IP Cam stream)

### Software & Libraries

Install the following Python libraries:

```bash
pip install opencv-python numpy pandas sounddevice scipy gTTS openai aiogram

