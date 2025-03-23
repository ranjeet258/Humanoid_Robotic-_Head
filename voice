import google.generativeai as genai
import speech_recognition as sr
from gtts import gTTS
import pygame
import os
import tempfile
import threading
import RPi.GPIO as GPIO
import time

# ? Gemini API Key (Keep it safe in production!)
API_KEY = "AIzaSyC7offlchzed7wvuwt5DESFLzSywA71P-Y"

# ? Configure Gemini
genai.configure(api_key=API_KEY)

# ? Initialize Gemini 1.5 Pro model
model = genai.GenerativeModel('gemini-1.5-pro-latest')

# ? Initialize pygame mixer
pygame.init()
pygame.mixer.init()

# ? Servo setup
GPIO.setmode(GPIO.BCM)
GPIO.setup(17, GPIO.OUT)

pwm = GPIO.PWM(17, 50)
pwm.start(0)

# ? Function to move the servo between 0Â° and 20Â° smoothly
def servo_motion(stop_event):
    try:
        while not stop_event.is_set():
            # Sweep from 0Â° to 20Â°
            for angle in range(0, 21, 1):
                if stop_event.is_set():
                    break
                duty = 2 + (angle / 18)
                pwm.ChangeDutyCycle(duty)
                time.sleep(0.02)

            # Sweep from 20Â° to 0Â°
            for angle in range(20, -1, -1):
                if stop_event.is_set():
                    break
                duty = 2 + (angle / 18)
                pwm.ChangeDutyCycle(duty)
                time.sleep(0.02)

        pwm.ChangeDutyCycle(0)  # Release the servo when stopped
    except Exception as e:
        print(f"âš ï¸ Servo motion error: {e}")

# ? Function to recognize voice and convert to text
def listen_to_microphone():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        print("\nðŸŽ¤ Listening...")
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)

    try:
        print("ðŸ§  Recognizing...")
        text = recognizer.recognize_google(audio)
        print(f"ðŸ™‹ You said: {text}")
        return text.lower()

    except sr.UnknownValueError:
        print("â“ Sorry, I could not understand.")
        return None

    except sr.RequestError:
        print("âš ï¸ Speech Recognition Service not available.")
        return None

# ? Function to get Gemini AI response with error handling
def ask_gemini(prompt):
    print("ðŸ’¡ Thinking...")
    try:
        concise_prompt = (
            f"{prompt}\n\n"
            "Please give the answer in one or two lines only. Be precise and concise."
        )

        response = model.generate_content(concise_prompt)
        return response.text.strip()

    except Exception as e:
        print(f"âš ï¸ Gemini API Error: {e}")
        return "I'm sorry, I couldn't process that request."

# ? Function to convert text to speech and play it with pygame
def speak_text(text):
    print(f"ðŸ—£ï¸ Gemini says: {text}")

    try:
        tts = gTTS(text=text, lang='en')

        with tempfile.NamedTemporaryFile(delete=False, suffix='.mp3') as fp:
            temp_file_path = fp.name
            tts.save(temp_file_path)

        # Play the file using pygame
        pygame.mixer.music.load(temp_file_path)
        pygame.mixer.music.play()

        # Start servo motion in parallel
        stop_event = threading.Event()
        servo_thread = threading.Thread(target=servo_motion, args=(stop_event,))
        servo_thread.start()

        # Wait until the sound finishes playing
        while pygame.mixer.music.get_busy():
            pygame.time.Clock().tick(10)

        # Stop the servo motion after audio playback
        stop_event.set()
        servo_thread.join()

        # Clean up temp file
        os.remove(temp_file_path)

    except Exception as e:
        print(f"âš ï¸ Error in speaking text: {e}")

# ? Main loop
def ai_voice_assistant():
    print("ðŸ¤– Hello! I am your AI Assistant. Say something... (say 'exit' to quit)")

    try:
        while True:
            user_input = listen_to_microphone()

            if user_input is None:
                continue

            if "exit" in user_input or "quit" in user_input:
                speak_text("Goodbye! Have a nice day!")
                while pygame.mixer.music.get_busy():
                    pygame.time.Clock().tick(10)
                break

            response = ask_gemini(user_input)

            speak_text(response)

    except KeyboardInterrupt:
        print("\nðŸ›‘ Interrupted by user!")

    finally:
        pwm.stop()
        GPIO.cleanup()
        pygame.quit()

# ? Run the assistant
if __name__ == "__main__":
    ai_voice_assistant()
