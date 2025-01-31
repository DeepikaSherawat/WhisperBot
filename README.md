# WhisperBot
WhisperBot is a cutting-edge voice-based assistant designed to revolutionize the way you interact with technology. Equipped with advanced speech recognition and text-to-speech capabilities, WhisperBot offers seamless and natural voice interactions.


Speech-to-Speech Assistant
This project is a speech-to-speech assistant that uses speech recognition and text-to-speech technologies to have voice-based interactions with users. It also integrates with the Groq API for generating responses and stores conversation history in an SQLite database.

Features
Speech Recognition: Converts speech to text using the speech_recognition library.

Text-to-Speech: Converts text to speech using the pyttsx3 library.

Groq API Integration: Uses the Groq API to generate responses based on user input.

Conversation History: Stores user inputs and bot responses in an SQLite database.

Error Handling: Handles various errors, including API errors and audio recognition errors.

Requirements
Python 3.x

speech_recognition

pyttsx3

requests

pyaudio

sqlite3

Installation
Clone this repository or download the code.

Install the required packages:

sh
pip install SpeechRecognition pyttsx3 requests pyaudio
Usage
Create the SQLite database by running the script. This will create a table to store conversation history.

Run the speech_to_speech_bot() function to start the assistant.

python
import speech_recognition as sr
import pyttsx3
import requests
import sqlite3
import json
import pyaudio

# Initialize recognizer and text-to-speech engine
recognizer = sr.Recognizer()
engine = pyttsx3.init()

# Groq API configuration
GROQ_API_KEY = "YOUR_GROQ_API_KEY"
GROQ_API_URL = "YOUR_GROQ_API_URL"
MODEL_NAME = "YOUR_MODEL_NAME"

# SQLite database setup
DATABASE_NAME = "conversation_history.db"

def create_database():
    conn = sqlite3.connect(DATABASE_NAME)
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS conversations (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_input TEXT,
            bot_response TEXT
        )
    ''')
    conn.commit()
    conn.close()

def save_conversation(user_input, bot_response):
    conn = sqlite3.connect(DATABASE_NAME)
    cursor = conn.cursor()
    cursor.execute('''
        INSERT INTO conversations (user_input, bot_response)
        VALUES (?, ?)
    ''', (user_input, bot_response))
    conn.commit()
    conn.close()

def get_groq_response(user_input):
    headers = {
        "Authorization": f"Bearer {GROQ_API_KEY}",
        "Content-Type": "application/json"
    }
    data = {
        "model": MODEL_NAME,
        "messages": [{"role": "user", "content": user_input}],
        "temperature": 0.7,
        "max_tokens": 500
    }
    
    try:
        response = requests.post(GROQ_API_URL, headers=headers, json=data, timeout=15)
        response.raise_for_status()
        return response.json()["choices"][0]["message"]["content"]
    except requests.exceptions.HTTPError as err:
        print(f"HTTP Error: {err}")
        print(f"Response content: {response.text}")
        return "Sorry, there was an error processing your request."
    except KeyError as e:
        print(f"Key Error in response parsing: {e}")
        return "Sorry, there was an issue processing the response."
    except Exception as e:
        print(f"General Error: {str(e)}")
        return "Sorry, I'm having trouble processing this request."
   

def speak(text):
    engine.say(text)
    engine.runAndWait()

def listen():
    with sr.Microphone() as source:
        recognizer.energy_threshold = 3000  # Increased sensitivity
        recognizer.dynamic_energy_threshold = True
        recognizer.pause_threshold = 1.5  # Longer pause before considering speech ended

        try:
            print("\nAdjusting for ambient noise... (2 seconds)")
            recognizer.adjust_for_ambient_noise(source, duration=2)
            print("Listening... (5 second timeout)")
            audio = recognizer.listen(source, timeout=5, phrase_time_limit=8)
            print("Recognizing...")
            text = recognizer.recognize_google(audio, language="en-IN", show_all=False)
            print(f"You said: {text}")
            return text.lower()
        except sr.WaitTimeoutError:
            print("No speech detected within timeout")
            return None
        except sr.UnknownValueError:
            print("Could not understand audio")
            return None
        except sr.RequestError as e:
            print(f"API Error: {e}")
            return None

def speech_to_speech_bot():
    create_database()
    engine = pyttsx3.init()

    try:
        speak("Hello! I'm your assistant. How can I help you today?")
        while True:
            print("\n" + "="*40)
            user_input = listen()
            
            if not user_input:
                speak("I didn't catch that. Could you please repeat?")
                continue
                
            if any(word in user_input for word in ["exit", "quit", "goodbye"]):
                speak("Goodbye! Have a great day!")
                break

            print(f"Processing: {user_input}")
            speak("Let me think about that...")
            
            bot_response = get_groq_response(user_input)
            print(f"\nBot Response: {bot_response}")
            save_conversation(user_input, bot_response)
            speak(bot_response)

    except KeyboardInterrupt:
        print("\nExiting gracefully...")
        speak("Goodbye! Thanks for chatting!")
    finally:
        engine.stop()

# Run the bot
if __name__ == "__main__":
    speech_to_speech_bot()
Configuration
Replace the placeholder values in the code with your actual Groq API key and the correct API URL.

Error Handling
The bot handles various errors, including:

API errors

Audio recognition errors

Timeout errors
