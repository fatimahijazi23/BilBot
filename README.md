🤖 BilBot - Lebanese Arabic Voice-Controlled Object Retrieval Robot
BilBot is an AI-powered autonomous robot designed to interpret Lebanese Arabic voice commands, identify requested objects in its environment, and move toward them. It integrates advanced technologies like GPT-4 (via OpenAI), Whisper, computer vision, inverse kinematics, and Telegram notifications to deliver a fully interactive experience.

🚀 Features
Arabic Voice Recognition: Captures and transcribes voice commands in Lebanese Arabic using OpenAI Whisper.

Arabic to English Translation: Translates Lebanese Arabic into English using GPT-4.

Object Extraction & Synonym Detection: Extracts the object name from speech and generates a dynamic list of synonyms via GPT-4.

Image Inspection via GPT-4 Vision: Takes snapshots and asks GPT-4 if the object appears in the image.

Object Location Detection: Determines relative position of objects (e.g., "left", "right", etc.) using GPT-4 Vision.

Autonomous Movement: Uses mecanum wheels and sonar sensor to navigate toward the object.

Arabic Text-to-Speech (TTS): Converts status messages and prompts to spoken Arabic using gTTS.

Telegram Bot Integration: Sends messages and photos to a Telegram chat for real-time updates.

🧰 Requirements
Hardware
Raspberry Pi with Linux

Hiwonder Arm & Mecanum Wheel Chassis

Sonar Sensor

USB Microphone

Camera (e.g., PiCamera or IP Cam stream)

Software & Libraries
Install the required Python libraries:

bash
Copy
Edit
pip install opencv-python numpy pandas sounddevice scipy gTTS openai aiogram
Also make sure you have:

mpg123 (for audio playback — install via sudo apt install mpg123)

OpenAI API Key (with access to GPT-4 and Whisper)

Telegram Bot API Token (generated via @BotFather on Telegram)

Your Telegram Chat ID (can be retrieved using bots like @userinfobot)

⚙️ Configuration
Edit the Python script and set your API keys and credentials:

python
Copy
Edit
API_TOKEN = "YOUR_TELEGRAM_BOT_TOKEN"
CHAT_ID = YOUR_CHAT_ID
client = openai.OpenAI(api_key="YOUR_OPENAI_API_KEY")
🧠 How It Works
Voice Input
The robot records your command in Lebanese Arabic using a microphone.

Speech Processing

Transcribes audio using Whisper

Translates Arabic to English using GPT-4

Extracts the object name using GPT-4

Confirms the object via Arabic TTS

Image Capture & Detection

Captures a snapshot of the current view

GPT-4 Vision checks if the object or any synonym is visible

Identifies the object's position (left, middle, right, etc.)

Navigation

If object is detected, robot rotates and moves toward it

Sends snapshot and update via Telegram for real-time feedback

Loop or Exit

Once the object is reached, the robot asks if you want to search for another object

Repeats or ends based on the user's voice input

🧪 Example Use Case
User says:
"بدي جزدان" (I want a purse)

Robot:

Records audio, transcribes, and translates

Extracts "purse" as the object

Captures image and checks for presence

Moves toward the object if found, sends photo via Telegram

Asks if user wants to search for something else

📸 Telegram Integration
When an object is detected, the robot:

Sends a message with the object's name

Sends the captured photo

Confirms movement toward the object

📋 File Structure
bash
Copy
Edit
bilbot/
├── main.py               # Main robot logic
├── snapshot.jpg          # Latest captured image
├── audio.wav             # Recorded audio file
├── bilbot_speech.mp3     # Arabic voice prompts
├── common/               # Chassis and sonar control modules
├── kinematics/           # Arm inverse kinematics and transforms
🔮 Future Enhancements
Real-time object detection with YOLOv8 or OpenCV DNN

Voice conversation loop (voice-GPT interactive dialogue)

Battery and obstacle avoidance monitoring

Web dashboard with live video and controls

🤝 Credits
Developed by Fatima Hijazi

Powered by OpenAI (GPT-4, Whisper), gTTS, Aiogram, and Hiwonder Robotics

📄 License
This project is for educational and research use only.
Contact the author for commercial use or redistribution.



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
