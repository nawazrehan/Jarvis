import pyttsx3
import speech_recognition as sr
import datetime
import webbrowser
import os
from youtube_search import YoutubeSearch
import pywhatkit as kit
import google.generativeai as genai

engine = pyttsx3.init('sapi5')
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[1].id)


def speak(audio):
    engine.say(audio)
    engine.runAndWait()


def wishMe():
    hour = int(datetime.datetime.now().hour)
    if hour >= 0 and hour < 12:
        speak("Good Morning!")
    elif hour >= 12 and hour < 18:
        speak("Good Afternoon!")
    else:
        speak("Good Evening!")
    speak("I am Jarvis Sir. Please tell me how may I help you")
    print("I am Jarvis Sir. Please tell me how may I help you")


def takeCommand():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        r.pause_threshold = 0.8
        audio = r.listen(source)

    try:
        print("Recognizing...")
        query = r.recognize_google(audio, language='en-in')
        print(f"User said: {query}\n")

    except Exception as e:
        print("Say that again please...")
        return "None"
    return query


def openapp(query):
    dict_app = {'vscode': r'C:\Users\SSHASHANK R\Desktop\Visual Studio Code.lnk',
                'spotify': r'C:\Users\SHASHANK R\Desktop\Spotify.lnk',
                }
    a = query.replace('open', '').strip()
    for name, path in dict_app.items():
        if a in name:
            os.startfile(path)


def chatgpt(query):
    # Configure Gemini API
    genai.configure(api_key="AIzaSyBg29wb7MWBkE0_08k-Zd8rnBdYGUb2LDQ")

    question = query.replace('ask gpt', '').strip()
    if question == "":
        question = query

    # Updated model names that actually work
    model_names = [
        'gemini-1.5-flash',  # Fast and reliable
        'gemini-1.5-pro',    # More capable
        'gemini-1.0-pro'     # Fallback
    ]

    for model_name in model_names:
        try:
            model = genai.GenerativeModel(model_name)
            prompt = "Answer the following in only 2 to 3 lines:\n\n" + question
            response = model.generate_content(prompt)
            
            if response and response.text:
                print("Using model:", model_name)
                print("\nGPT:", response.text, "\n")
                speak(response.text)
                return
        except Exception as e:
            print(f"Error with {model_name}: {e}")
            continue
    
    # If all models fail
    error_msg = "I'm sorry, I couldn't process that request at the moment."
    print("\nGPT:", error_msg, "\n")
    speak(error_msg)


if _name_ == "_main_":
    wishMe()

    while True:
        query = takeCommand().lower()

        if 'what is the time' in query or "what's the time" in query:
            strTime = datetime.datetime.now().strftime("%H:%M:%S")
            speak(f"Sir, the time is {strTime}")
            print(f"Sir, the time is {strTime}")

        elif 'find' in query:
            query = query.replace("find", "").strip()
            speak(f"Searching Google for {query}")
            url = f"https://www.google.com/search?q={query}"
            webbrowser.open(url)

        elif 'play' in query:
            video = query.replace('play', '').strip()
            speak(f'Playing {video} on YouTube')
            results = YoutubeSearch(video, max_results=1).to_dict()
            video_id = results[0]['id']
            webbrowser.open(f'https://www.youtube.com/watch?v={video_id}')

        elif 'stop' in query or 'exit' in query or 'quit' in query or 'good night' in query:
            speak("Goodbye Sir. Have a nice day.")
            break

        elif 'open' in query:
            openapp(query)

        else:
            chatgpt(query)
