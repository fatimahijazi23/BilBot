# BilBot
Source code and scripts for BilBot, the Lebanese Arabic voice-controlled robot.
#!/usr/bin/python3
# coding=utf8

# === Import Required Libraries ===
import times
import signal
import math
import base64
import cv2
import numpy as np
import pandas as pd
import sounddevice as sd
import scipy.io.wavfile as wav
from gtts import gTTS
import os
from picamera2 import Picamera2
import common.sonar as Sonar
import common.mecanum as mecanum
from kinematics.transform import *
from kinematics.arm_move_ik import *
from common.ros_robot_controller_sdk import Board
import openai
from aiogram import Bot
from aiogram.types.input_file import FSInputFile
import asyncio
# === Initialize OpenAI API ===

client = openai.OpenAI(api_key="")
# === Global Variables ===
global target_object, object_found
target_object = ""
object_found = False
API_TOKEN = ""
CHAT_ID = -4794478323
# === Initialize Hardware Components ===
board = Board()
board.pwm_servo_set_position(0.5, [[6, 1500]])
car = mecanum.MecanumChassis()
HWSONAR = Sonar.Sonar()
AK = ArmIK()
AK.board = board



def get_dynamic_synonyms(object_name):
    prompt = f"""
You are an assistant that returns a list of synonyms or closely related words
for the object name '{object_name}' in both Arabic and English.
Please provide only the list of words separated by commas.
"""
    response = client.chat.completions.create(
        model="gpt-4-turbo",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=50,
        temperature=0.5,
    )
    text = response.choices[0].message.content.strip()
    # Split by commas and clean spaces
    synonyms = [w.strip() for w in text.split(",")]
    return synonyms

# Cache for synonyms to avoid repeated API calls
synonym_cache = {}

def get_cached_synonyms(object_name):
    if object_name not in synonym_cache:
        synonym_cache[object_name.lower()] = get_dynamic_synonyms(object_name)
    return synonym_cache[object_name]


def textToSpeach(response):
    text = response
    tts = gTTS(text=text, lang='ar')
    
    # Save to Desktop
    desktop_path = os.path.expanduser("~/Desktop")
    audio_path = os.path.join(desktop_path, "bilbot_speech.mp3")
    
    # Save and play
    tts.save(audio_path)
    os.system(f"mpg123 '{audio_path}'")  # Use single quotes in case of spaces
def is_yes(text):
    """Detects if the Arabic text means 'yes'"""
    yes_words = ["نعم", "اي", "ايه", "أجل", "أكيد", "طبعا", "صح", "صحيح", "مزبوط", "اه"]
    return any(word in text.lower() for word in yes_words)

def is_no(text):
    """Detects if the Arabic text means 'no'"""
    no_words = ["لا", "كلا", "مش", "ما", "منا", "لأ", "لأه", "غلط", "مو", "مش صحيح"]
    return any(word in text.lower() for word in no_words)


# === Capture Image ===
def capture_image(output_path="snapshot.jpg"):
    """Capture image from camera feed"""
    cap = cv2.VideoCapture('http://127.0.0.1:8080?action=stream')
    if not cap.isOpened():
        print("❌ Cannot open camera")
        return None
    ret, frame = cap.read()
    cap.release()
    if ret:
        cv2.imwrite(output_path, frame)
        return output_path
    else:
        print("❌ Failed to capture image")
        return None


# === Record Audio ===
def record_audio(output_path="audio.wav", duration=3, samplerate=44100):
    """Record audio input"""
    print("🎤 Recording... Speak now!")
    textToSpeach("سأبدأ التسجيل، تفضل بالكلام الآن")
    recording = sd.rec(int(duration * samplerate), samplerate=samplerate, channels=1, dtype='int16')
    sd.wait()
    wav.write(output_path, samplerate, recording)
    textToSpeach("انتهى التسجيل")
    print(f"✅ Audio saved as: {output_path}")
    return output_path


# === Transcribe Audio ===
def transcribe_audio(file_path):
    """Convert speech to text using Whisper (new OpenAI API client)"""
    with open(file_path, "rb") as audio_file:
        response = client.audio.transcriptions.create(
            model="whisper-1",
            file=audio_file,
            response_format="json"
        )
    return response.text


# === Translate to English ===
def translate_to_english(text):
    """Translate Arabic sentence to English"""
    prompt = f"""You are a smart interpreter of Lebanese Arabic.
Translate the sentence below into English.
Sentence: \"{text}\"
"""
    response = client.chat.completions.create(model="gpt-4-turbo", messages=[{"role": "user", "content": prompt}])
    return response.choices[0].message.content.strip()


# === Extract Object Name ===
def extract_object(text):
    """Extract object name from sentence"""
    prompt = f"""
You are an AI robot assistant. Extract only the object the user wants the robot to retrieve.
- If therSe's no clear object, say: None
- Do NOT return full sentences.
Sentence: "{text}"
Return only the object name, or None."""
    response = client.chat.completions.create(model="gpt-4-turbo", messages=[{"role": "user", "content": prompt}])
    return response.choices[0].message.content.strip()


def ask_about_object_in_image(image_path, object_name_list):
    """
    object_name_list: list of synonyms including the main object name
    Ask GPT if any of these names appear in the image.
    """
    names_str = ", ".join(object_name_list)
    with open(image_path, "rb") as image_file:
        base64_img = base64.b64encode(image_file.read()).decode('utf-8')
    prompt_text = f"""
You are a strict image inspector. Look carefully at the image. 
Do you see any of these objects: {names_str} — clearly and unmistakably visible?
- Do NOT guess.
- If none of these objects are clearly present, say only: No.
- Answer only one word: Yes or No.
"""
    response = client.chat.completions.create(
        model="gpt-4-turbo",
        messages=[
            {"role": "user",
             "content": [
                   {"type": "text", "text": prompt_text},
                   {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{base64_img}"}}
               ]
             }
        ],
        max_tokens=100,
    )
    return response.choices[0].message.content.strip()

async def send_message(message):
    bot = Bot(token=API_TOKEN)
    try:
        await bot.send_message(chat_id=CHAT_ID, text=message)
        photo = FSInputFile("snapshot.jpg")
        await bot.send_photo(chat_id=CHAT_ID, photo=photo, caption="")
        print("Message and photo sent successfully!")
    except Exception as e:
        print(f"An error occurred: {e}")
    finally:
        await bot.session.close()


# === Determine Object Location ===
def ask_where_the_object_is_in_the_image(image_path, object_name):
    """Ask GPT the relative location of the object in image"""
    with open(image_path, "rb") as image_file:
        base64_img = base64.b64encode(image_file.read()).decode('utf-8')
    response = client.chat.completions.create(
        model="gpt-4-turbo",
        messages=[
            {"role": "user",
             "content": [
                 {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{base64_img}"}},
                 {"type": "text",
                  "text": f"check if the object: {object_name} is: far left, left, slightly left, middle, slightly right, right, far right, or not found in this image. Return only one of the options and nothing else."}
             ]}
        ],
        max_tokens=100,
    )
    return response.choices[0].message.content.strip()



# === Object Detection Workflow ===
def object_to_find():
    global target_object,object_found
    print(f"🔎 Current target object: {target_object}")
    car.set_velocity(0, 0, 0)
    while target_object == "":
        board.pwm_servo_set_position(0.5, [[1, 2000]])
        audio_path = record_audio()
        arabic_text = transcribe_audio(audio_path)
        print("📝 Transcription:", arabic_text)
        english_text = translate_to_english(arabic_text)
        print("🌍 Translation:", english_text)
        target_object = extract_object(english_text)
        print("🎯 Object to retrieve:", target_object)
        if target_object.lower() == "none" or "no object" in target_object.lower() or len(target_object.split()) > 4:
            print("❌ No valid object detected. Please try again.")
            textToSpeach("لم أفهم الغرض المطلوب، حاول مرة أخرى من فضلك")
            target_object = ""
            continue  # 🔁 Go back and ask again

        synonyms = get_cached_synonyms(target_object)
        print("🔍 Synonyms for the object:", synonyms)

        textToSpeach(" هل الغرض الذي سأبحث عنه هو " + target_object + " هذا صحيح؟")
        time.sleep(5)
        audio_path = record_audio(duration=6)
        arabic_text = transcribe_audio(audio_path)
        english_text = translate_to_english(arabic_text)
        if is_no(arabic_text):
            target_object = ""
            continue
    time.sleep(15)

    image_path = capture_image("snapshot.jpg")
    result = ask_about_object_in_image(image_path, [target_object] + synonyms)
    print("🖼 Object presence:", result)

    if "yes" in result.lower():
        print("➡ Rotating to align with object...")
        object_found=True
        asyncio.run(send_message(target_object))
    else:
        print("object not im this image")
        
        
# === Circle Scan Routine ===
def circle_detect():
    for _ in range(7):  
        if object_found:
            break
        object_to_find()
        car.set_velocity(0, 90, -0.2385)  # Reduced angular velocity
        time.sleep(0.5)  # Reduced duration
        car.set_velocity(0, 0, 0)
        time.sleep(1)  # Short pause for stability

def go_front():
    counter = 8
    dist = HWSONAR.getDistance() / 10
    while counter != 0:
        if dist >= 30:
            car.set_velocity(40, 90, -0.03)
            time.sleep(0.5)
        else:
            car.set_velocity(40, 90, -0.3)
            time.sleep(0.5)
        counter -= 1
        dist = HWSONAR.getDistance() / 10

    car.set_velocity(0, 0, 0)

# === Main Loop ===
if __name__ == "__main__":
    print("🔄 Starting robot object detection routine...")
    
    while True:
        circle_detect()
        if object_found:
            print("✅ Object found and reached. Stopping robot.")
            car.set_velocity(0, 0, 0)
            object_found = False
            target_object = ""
            textToSpeach("هل ترغب في البحث عن غرض آخر؟")
            audio_path = record_audio()
            arabic_text = transcribe_audio(audio_path)
            english_text = translate_to_english(arabic_text)
            
            if is_no(arabic_text):
                textToSpeach("حسنًا، سأتوقف الآن. إلى اللقاء!")
                break
            elif is_yes(arabic_text):
                print("🔄 Starting new object search.")
                textToSpeach("حسنًا، سأبدأ البحث عن غرض جديد.")
                object_found = False
