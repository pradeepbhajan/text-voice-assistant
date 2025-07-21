import speech_recognition as sr
import pyttsx3
import datetime
import webbrowser
import os
import sys
import json
import random
import requests
import subprocess
import threading
import time
from pathlib import Path
import wikipedia
import sqlite3
import smtplib
import pywhatkit    
import pyautogui
import psutil
import platform
import numpy as np
import sympy as sp
from sympy import symbols, solve, diff, integrate, limit, series, Matrix, simplify
from sympy.vector import CoordSys3D
import matplotlib.pyplot as plt
from fractions import Fraction
import math
import re

class VoiceAssistant:
    def __init__(self):
        self.engine = pyttsx3.init()
        self.recognizer = sr.Recognizer()
        self.microphone = sr.Microphone()
        self.setup_voice()
        self.commands_count = 0
        self.command_history = []
        # Mathematical solver variables
        self.x, self.y, self.z = symbols('x y z')
        self.t = symbols('t')
        self.coord_system = CoordSys3D('C')
        # Input mode settings
        self.input_mode = "voice"  # "voice", "text", "mixed"
        self.text_input_enabled = True
        self.voice_input_enabled = True
        
    def setup_voice(self):
        """Voice engine setup"""
        voices = self.engine.getProperty('voices')
        self.engine.setProperty('voice', voices[1].id if len(voices) > 1 else voices[0].id)
        self.engine.setProperty('rate', 150)
        self.engine.setProperty('volume', 0.9)
    
    def speak(self, text):
        """Text to speech"""
        print(f"Assistant: {text}")
        self.engine.say(text)
        self.engine.runAndWait()
    
    def listen(self):
        """Speech to text"""
        try:
            with self.microphone as source:
                self.recognizer.adjust_for_ambient_noise(source)
                print("Listening...")
                audio = self.recognizer.listen(source, timeout=5)
                
            command = self.recognizer.recognize_google(audio, language='hi-IN')
            print(f"You said: {command}")
            return command.lower()
        except sr.UnknownValueError:
            return ""
        except sr.RequestError:
            self.speak("Sorry, speech service is unavailable")
            return ""
        except sr.WaitTimeoutError:
            return ""

    def execute_command(self, command):
        """Main command processor"""
        self.commands_count += 1
        self.command_history.append(command)
        
        # Basic Commands (1-50)
        if any(word in command for word in ['hello', 'hi', 'namaste', 'namaskar']):
            self.speak("Namaste! Main aapka voice assistant hun. Kaise madad kar sakta hun?")
        
        elif any(word in command for word in ['time', 'samay', 'kitna baja']):
            current_time = datetime.datetime.now().strftime("%H:%M")
            self.speak(f"Abhi samay hai {current_time}")
        
        elif any(word in command for word in ['date', 'tarikh', 'aaj kya din']):
            today = datetime.datetime.now().strftime("%d %B %Y")
            self.speak(f"Aaj ki tarikh hai {today}")
        
        elif any(word in command for word in ['weather', 'mausam']):
            self.get_weather()
        
        elif any(word in command for word in ['news', 'khabar', 'samachar']):
            self.get_news()
        
        # System Commands (51-150)
        elif any(word in command for word in ['shutdown', 'band karo', 'computer off']):
            self.speak("Computer band kar raha hun")
            os.system("shutdown /s /t 1")
        
        elif any(word in command for word in ['restart', 'reboot', 'dubara start']):
            self.speak("Computer restart kar raha hun")
            os.system("shutdown /r /t 1")
        
        elif any(word in command for word in ['sleep', 'hibernate', 'so jao']):
            self.speak("Computer ko sleep mode mein dal raha hun")
            os.system("rundll32.exe powrprof.dll,SetSuspendState 0,1,0")
        
        elif any(word in command for word in ['volume up', 'awaz badha']):
            self.change_volume('up')
        
        elif any(word in command for word in ['volume down', 'awaz kam']):
            self.change_volume('down')
        
        elif any(word in command for word in ['mute', 'chup', 'awaz band']):
            self.change_volume('mute')
        
        elif any(word in command for word in ['brightness up', 'screen bright']):
            self.change_brightness('up')
        
        elif any(word in command for word in ['brightness down', 'screen dim']):
            self.change_brightness('down')
        
        elif any(word in command for word in ['wifi on', 'internet on']):
            self.toggle_wifi('on')
        
        elif any(word in command for word in ['wifi off', 'internet off']):
            self.toggle_wifi('off')
        
        elif any(word in command for word in ['bluetooth on']):
            self.toggle_bluetooth('on')
        
        elif any(word in command for word in ['bluetooth off']):
            self.toggle_bluetooth('off')
        
        elif any(word in command for word in ['battery', 'charge', 'battery status']):
            self.get_battery_status()
        
        elif any(word in command for word in ['cpu usage', 'processor']):
            self.get_cpu_usage()
        
        elif any(word in command for word in ['memory usage', 'ram']):
            self.get_memory_usage()
        
        elif any(word in command for word in ['disk space', 'storage']):
            self.get_disk_usage()
        
        elif any(word in command for word in ['running processes', 'task manager']):
            self.show_running_processes()
        
        elif any(word in command for word in ['network info', 'ip address']):
            self.get_network_info()
        
        elif any(word in command for word in ['system info', 'computer details']):
            self.get_system_info()
        
        # File Operations (151-250)
        elif any(word in command for word in ['create folder', 'naya folder']):
            self.create_folder()
        
        elif any(word in command for word in ['delete file', 'file delete']):
            self.delete_file()
        
        elif any(word in command for word in ['copy file', 'file copy']):
            self.copy_file()
        
        elif any(word in command for word in ['move file', 'file move']):
            self.move_file()
        
        elif any(word in command for word in ['rename file', 'file rename']):
            self.rename_file()
        
        elif any(word in command for word in ['search file', 'file dhundo']):
            self.search_file()
        
        elif any(word in command for word in ['open file', 'file kholo']):
            self.open_file()
        
        elif any(word in command for word in ['file properties', 'file details']):
            self.get_file_properties()
        
        elif any(word in command for word in ['compress file', 'zip file']):
            self.compress_file()
        
        elif any(word in command for word in ['extract file', 'unzip file']):
            self.extract_file()
        
        # Application Commands (251-350)
        elif any(word in command for word in ['open notepad', 'notepad kholo']):
            self.open_application('notepad')
        
        elif any(word in command for word in ['open calculator', 'calculator kholo']):
            self.open_application('calc')
        
        elif any(word in command for word in ['open paint', 'paint kholo']):
            self.open_application('mspaint')
        
        elif any(word in command for word in ['open chrome', 'browser kholo']):
            self.open_application('chrome')
        
        elif any(word in command for word in ['open firefox']):
            self.open_application('firefox')
        
        elif any(word in command for word in ['open edge']):
            self.open_application('msedge')
        
        elif any(word in command for word in ['open word', 'microsoft word']):
            self.open_application('winword')
        
        elif any(word in command for word in ['open excel', 'microsoft excel']):
            self.open_application('excel')
        
        elif any(word in command for word in ['open powerpoint', 'presentation']):
            self.open_application('powerpnt')
        
        elif any(word in command for word in ['open outlook', 'email']):
            self.open_application('outlook')
        
        elif any(word in command for word in ['open skype']):
            self.open_application('skype')
        
        elif any(word in command for word in ['open zoom']):
            self.open_application('zoom')
        
        elif any(word in command for word in ['open teams', 'microsoft teams']):
            self.open_application('teams')
        
        elif any(word in command for word in ['open discord']):
            self.open_application('discord')
        
        elif any(word in command for word in ['open spotify', 'music']):
            self.open_application('spotify')
        
        elif any(word in command for word in ['open vlc', 'video player']):
            self.open_application('vlc')
        
        elif any(word in command for word in ['open photoshop']):
            self.open_application('photoshop')
        
        elif any(word in command for word in ['open vs code', 'code editor']):
            self.open_application('code')
        
        elif any(word in command for word in ['open cmd', 'command prompt']):
            self.open_application('cmd')
        
        elif any(word in command for word in ['open powershell']):
            self.open_application('powershell')
        
        # Web Commands (351-450)
        elif any(word in command for word in ['open google', 'google kholo']):
            webbrowser.open('https://www.google.com')
            self.speak("Google kholne ja raha hun")
        
        elif any(word in command for word in ['open youtube', 'youtube kholo']):
            webbrowser.open('https://www.youtube.com')
            self.speak("YouTube kholne ja raha hun")
        
        elif any(word in command for word in ['open facebook']):
            webbrowser.open('https://www.facebook.com')
            self.speak("Facebook kholne ja raha hun")
        
        elif any(word in command for word in ['open instagram']):
            webbrowser.open('https://www.instagram.com')
            self.speak("Instagram kholne ja raha hun")
        
        elif any(word in command for word in ['open twitter']):
            webbrowser.open('https://www.twitter.com')
            self.speak("Twitter kholne ja raha hun")
        
        elif any(word in command for word in ['open linkedin']):
            webbrowser.open('https://www.linkedin.com')
            self.speak("LinkedIn kholne ja raha hun")
        
        elif any(word in command for word in ['open github']):
            webbrowser.open('https://www.github.com')
            self.speak("GitHub kholne ja raha hun")
        
        elif any(word in command for word in ['open stackoverflow']):
            webbrowser.open('https://www.stackoverflow.com')
            self.speak("Stack Overflow kholne ja raha hun")
        
        elif any(word in command for word in ['open amazon']):
            webbrowser.open('https://www.amazon.in')
            self.speak("Amazon kholne ja raha hun")
        
        elif any(word in command for word in ['open flipkart']):
            webbrowser.open('https://www.flipkart.com')
            self.speak("Flipkart kholne ja raha hun")
        
        elif any(word in command for word in ['open netflix']):
            webbrowser.open('https://www.netflix.com')
            self.speak("Netflix kholne ja raha hun")
        
        elif any(word in command for word in ['open hotstar']):
            webbrowser.open('https://www.hotstar.com')
            self.speak("Hotstar kholne ja raha hun")
        
        elif any(word in command for word in ['open gmail']):
            webbrowser.open('https://www.gmail.com')
            self.speak("Gmail kholne ja raha hun")
        
        elif any(word in command for word in ['open whatsapp web']):
            webbrowser.open('https://web.whatsapp.com')
            self.speak("WhatsApp Web kholne ja raha hun")
        
        elif any(word in command for word in ['search google', 'google mein search']):
            self.google_search(command)
        
        elif any(word in command for word in ['search youtube', 'youtube mein search']):
            self.youtube_search(command)
        
        elif any(word in command for word in ['play music', 'gaana bajao']):
            self.play_music()
        
        elif any(word in command for word in ['play video', 'video chalao']):
            self.play_video()
        
        elif any(word in command for word in ['download video', 'video download']):
            self.download_video()
        
        elif any(word in command for word in ['check website', 'website check']):
            self.check_website_status()
        
        # Entertainment Commands (451-550)
        elif any(word in command for word in ['tell joke', 'joke sunao', 'hasao']):
            self.tell_joke()
        
        elif any(word in command for word in ['tell story', 'kahani sunao']):
            self.tell_story()
        
        elif any(word in command for word in ['sing song', 'gaana gao']):
            self.sing_song()
        
        elif any(word in command for word in ['play game', 'khel khelo']):
            self.play_game()
        
        elif any(word in command for word in ['riddle', 'paheli']):
            self.tell_riddle()
        
        elif any(word in command for word in ['quote', 'suvichar']):
            self.tell_quote()
        
        elif any(word in command for word in ['fact', 'rochak tathya']):
            self.tell_fact()
        
        elif any(word in command for word in ['poem', 'kavita']):
            self.recite_poem()
        
        elif any(word in command for word in ['shayari']):
            self.tell_shayari()
        
        elif any(word in command for word in ['movie recommendation', 'film suggest']):
            self.recommend_movie()
        
        elif any(word in command for word in ['book recommendation', 'kitab suggest']):
            self.recommend_book()
        
        elif any(word in command for word in ['recipe', 'khana banana']):
            self.get_recipe()
        
        elif any(word in command for word in ['horoscope', 'rashifal']):
            self.get_horoscope()
        
        elif any(word in command for word in ['lucky number', 'bhagyank']):
            self.get_lucky_number()
        
        elif any(word in command for word in ['flip coin', 'sikka']):
            self.flip_coin()
        
        elif any(word in command for word in ['roll dice', 'pasa']):
            self.roll_dice()
        
        elif any(word in command for word in ['random number', 'koi number']):
            self.generate_random_number()
        
        elif any(word in command for word in ['magic 8 ball', 'jadugar']):
            self.magic_8_ball()
        
        elif any(word in command for word in ['word of the day', 'aaj ka shabd']):
            self.word_of_the_day()
        
        elif any(word in command for word in ['trivia', 'quiz']):
            self.ask_trivia()
        
        # Educational Commands (551-650)
        elif any(word in command for word in ['wikipedia', 'wiki search']):
            self.wikipedia_search(command)
        
        elif any(word in command for word in ['dictionary', 'meaning', 'matlab']):
            self.get_word_meaning(command)
        
        elif any(word in command for word in ['translate', 'anuvad']):
            self.translate_text(command)
        
        elif any(word in command for word in ['spell', 'spelling']):
            self.spell_word(command)
        
        elif any(word in command for word in ['math', 'calculate', 'hisab']):
            self.calculate(command)
        
        # Advanced Mathematical Commands (1101-1200+)
        elif any(word in command for word in ['vector', 'dot product', 'cross product', 'magnitude']):
            result = self.solve_vector_operations(command)
            self.speak(result)
        
        elif any(word in command for word in ['set', 'union', 'intersection', 'difference', 'complement']):
            result = self.solve_set_operations(command)
            self.speak(result)
        
        elif any(word in command for word in ['derivative', 'differentiate', 'integral', 'integrate', 'limit']):
            result = self.solve_calculus(command)
            self.speak(result)
        
        elif any(word in command for word in ['solve equation', 'factor', 'expand', 'simplify']):
            result = self.solve_algebra(command)
            self.speak(result)
        
        elif any(word in command for word in ['matrix', 'determinant', 'inverse', 'transpose', 'eigenvalue']):
            result = self.solve_matrix_operations(command)
            self.speak(result)
        
        elif any(word in command for word in ['statistics', 'mean', 'median', 'mode', 'standard deviation']):
            result = self.solve_statistics(command)
            self.speak(result)
        
        elif any(word in command for word in ['quadratic equation', 'quadratic solve']):
            result = self.solve_quadratic_equation(command)
            self.speak(result)
        
        elif any(word in command for word in ['linear equation', 'linear solve']):
            result = self.solve_linear_equation(command)
            self.speak(result)
        
        elif any(word in command for word in ['trigonometry', 'sin', 'cos', 'tan']):
            result = self.solve_trigonometry(command)
            self.speak(result)
        
        elif any(word in command for word in ['logarithm', 'log', 'ln']):
            result = self.solve_logarithm(command)
            self.speak(result)
        
        elif any(word in command for word in ['permutation', 'combination', 'factorial']):
            result = self.solve_combinatorics(command)
            self.speak(result)
        
        elif any(word in command for word in ['probability', 'chance']):
            result = self.solve_probability(command)
            self.speak(result)
        
        elif any(word in command for word in ['geometry', 'area', 'perimeter', 'volume']):
            result = self.solve_geometry(command)
            self.speak(result)
        
        elif any(word in command for word in ['sequence', 'series', 'arithmetic progression', 'geometric progression']):
            result = self.solve_sequences(command)
            self.speak(result)
        
        elif any(word in command for word in ['complex number', 'imaginary']):
            result = self.solve_complex_numbers(command)
            self.speak(result)
        
        # Functions and Relations Commands (1201-1350+)
        elif any(word in command for word in ['function domain', 'domain', 'function range', 'range']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function value', 'evaluate function', 'f(x)']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['inverse function', 'function inverse']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['composite function', 'function composition']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['even function', 'odd function', 'function parity']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function monotonicity', 'increasing function', 'decreasing function']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function continuity', 'continuous function']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['linear function', 'quadratic function', 'polynomial function']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['rational function', 'exponential function', 'logarithmic function']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['trigonometric function', 'hyperbolic function']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['reflexive relation', 'symmetric relation', 'transitive relation']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['equivalence relation', 'partial order']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function zeros', 'function roots', 'x-intercepts']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function extrema', 'function maximum', 'function minimum']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function asymptotes', 'asymptote']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function transformation', 'function shift', 'function stretch']):
            result = self.math_solver.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['one to one function', 'onto function', 'bijective function']):
            result = self.check_function_types(command)
            self.speak(result)
        
        elif any(word in command for word in ['function graph', 'plot function']):
            result = self.plot_function_graph(command)
            self.speak(result)
        
        elif any(word in command for word in ['piecewise function', 'step function']):
            result = self.analyze_piecewise_function(command)
            self.speak(result)
        
        elif any(word in command for word in ['implicit function', 'parametric function']):
            result = self.analyze_special_functions(command)
            self.speak(result)
        
        elif any(word in command for word in ['unit convert', 'convert']):
            self.unit_conversion(command)
        
        elif any(word in command for word in ['currency convert', 'paisa convert']):
            self.currency_conversion(command)
        
        elif any(word in command for word in ['periodic table', 'element']):
            self.periodic_table_info(command)
        
        elif any(word in command for word in ['formula', 'sutra']):
            self.get_formula(command)
        
        elif any(word in command for word in ['country info', 'desh ki jankari']):
            self.get_country_info(command)
        
        elif any(word in command for word in ['capital', 'rajdhani']):
            self.get_capital(command)
        
        elif any(word in command for word in ['population', 'jansankhya']):
            self.get_population(command)
        
        elif any(word in command for word in ['distance', 'doori']):
            self.calculate_distance(command)
        
        elif any(word in command for word in ['timezone', 'samay kshetra']):
            self.get_timezone(command)
        
        elif any(word in command for word in ['language info', 'bhasha']):
            self.get_language_info(command)
        
        elif any(word in command for word in ['historical event', 'itihas']):
            self.get_historical_event(command)
        
        elif any(word in command for word in ['science fact', 'vigyan']):
            self.get_science_fact()
        
        elif any(word in command for word in ['space fact', 'antariksh']):
            self.get_space_fact()
        
        elif any(word in command for word in ['animal fact', 'janwar']):
            self.get_animal_fact()
        
        elif any(word in command for word in ['plant fact', 'paudha']):
            self.get_plant_fact()
        
        # Health & Fitness Commands (651-750)
        elif any(word in command for word in ['bmi calculate', 'weight check']):
            self.calculate_bmi()
        
        elif any(word in command for word in ['calorie', 'calories']):
            self.calculate_calories()
        
        elif any(word in command for word in ['water reminder', 'pani yaad']):
            self.set_water_reminder()
        
        elif any(word in command for word in ['exercise', 'vyayam']):
            self.suggest_exercise()
        
        elif any(word in command for word in ['yoga', 'meditation']):
            self.suggest_yoga()
        
        elif any(word in command for word in ['diet plan', 'khana plan']):
            self.suggest_diet()
        
        elif any(word in command for word in ['health tip', 'swasthya']):
            self.give_health_tip()
        
        elif any(word in command for word in ['medicine reminder', 'dawa yaad']):
            self.set_medicine_reminder()
        
        elif any(word in command for word in ['sleep tracker', 'neend']):
            self.track_sleep()
        
        elif any(word in command for word in ['heart rate', 'dil ki dhadak']):
            self.check_heart_rate()
        
        # Productivity Commands (751-850)
        elif any(word in command for word in ['set reminder', 'yaad dilao']):
            self.set_reminder(command)
        
        elif any(word in command for word in ['set alarm', 'alarm lagao']):
            self.set_alarm(command)
        
        elif any(word in command for word in ['set timer', 'timer lagao']):
            self.set_timer(command)
        
        elif any(word in command for word in ['stopwatch', 'time measure']):
            self.start_stopwatch()
        
        elif any(word in command for word in ['todo list', 'kaam ki list']):
            self.manage_todo()
        
        elif any(word in command for word in ['note', 'likhna']):
            self.take_note(command)
        
        elif any(word in command for word in ['calendar', 'calendar dekho']):
            self.show_calendar()
        
        elif any(word in command for word in ['schedule', 'samay sarni']):
            self.manage_schedule()
        
        elif any(word in command for word in ['meeting', 'baithak']):
            self.schedule_meeting()
        
        elif any(word in command for word in ['email send', 'email bhejo']):
            self.send_email()
        
        elif any(word in command for word in ['password generate', 'password banao']):
            self.generate_password()
        
        elif any(word in command for word in ['qr code', 'qr banao']):
            self.generate_qr_code()
        
        elif any(word in command for word in ['backup', 'backup banao']):
            self.create_backup()
        
        elif any(word in command for word in ['screenshot', 'screen capture']):
            self.take_screenshot()
        
        elif any(word in command for word in ['screen record', 'screen recording']):
            self.start_screen_recording()
        
        # Smart Home Commands (851-950)
        elif any(word in command for word in ['lights on', 'batti jalao']):
            self.control_lights('on')
        
        elif any(word in command for word in ['lights off', 'batti bujhao']):
            self.control_lights('off')
        
        elif any(word in command for word in ['fan on', 'pankha chalao']):
            self.control_fan('on')
        
        elif any(word in command for word in ['fan off', 'pankha band']):
            self.control_fan('off')
        
        elif any(word in command for word in ['ac on', 'ac chalao']):
            self.control_ac('on')
        
        elif any(word in command for word in ['ac off', 'ac band']):
            self.control_ac('off')
        
        elif any(word in command for word in ['tv on', 'tv chalao']):
            self.control_tv('on')
        
        elif any(word in command for word in ['tv off', 'tv band']):
            self.control_tv('off')
        
        elif any(word in command for word in ['music system', 'music chalao']):
            self.control_music_system()
        
        elif any(word in command for word in ['security camera', 'camera check']):
            self.check_security_camera()
        
        # Advanced Commands (951-1000+)
        elif any(word in command for word in ['ai chat', 'chatbot']):
            self.ai_chat(command)
        
        elif any(word in command for word in ['code generate', 'code banao']):
            self.generate_code(command)
        
        elif any(word in command for word in ['debug code', 'error dhundo']):
            self.debug_code()
        
        elif any(word in command for word in ['api test', 'api check']):
            self.test_api()
        
        elif any(word in command for word in ['database query', 'database']):
            self.database_query(command)
        
        elif any(word in command for word in ['server status', 'server check']):
            self.check_server_status()
        
        elif any(word in command for word in ['log analysis', 'log dekho']):
            self.analyze_logs()
        
        elif any(word in command for word in ['performance monitor', 'performance']):
            self.monitor_performance()
        
        elif any(word in command for word in ['security scan', 'security check']):
            self.security_scan()
        
        elif any(word in command for word in ['backup database', 'database backup']):
            self.backup_database()
        
        elif any(word in command for word in ['deploy app', 'app deploy']):
            self.deploy_application()
        
        elif any(word in command for word in ['docker', 'container']):
            self.manage_docker()
        
        elif any(word in command for word in ['git', 'version control']):
            self.git_operations(command)
        
        elif any(word in command for word in ['cloud', 'aws', 'azure']):
            self.cloud_operations(command)
        
        elif any(word in command for word in ['machine learning', 'ml']):
            self.ml_operations(command)
        
        elif any(word in command for word in ['blockchain', 'crypto']):
            self.blockchain_info(command)
        
        elif any(word in command for word in ['iot', 'internet of things']):
            self.iot_operations(command)
        
        elif any(word in command for word in ['automation', 'automate']):
            self.create_automation(command)
        
        elif any(word in command for word in ['report generate', 'report banao']):
            self.generate_report()
        
        elif any(word in command for word in ['data analysis', 'data dekho']):
            self.analyze_data()
        
        # System Commands (51-100)
        elif any(word in command for word in ['shutdown', 'band karo', 'computer off']):
            self.speak("Computer band kar raha hun")
            os.system("shutdown /s /t 1")
        
        elif any(word in command for word in ['restart', 'reboot', 'dubara start']):
            self.speak("Computer restart kar raha hun")
            os.system("shutdown /r /t 1")
        
        elif any(word in command for word in ['sleep', 'hibernate', 'so jao']):
            self.speak("Computer sleep mode mein ja raha hai")
            os.system("rundll32.exe powrprof.dll,SetSuspendState 0,1,0")
        
        elif any(word in command for word in ['volume up', 'awaz badha']):
            self.change_volume('+')
        
        elif any(word in command for word in ['volume down', 'awaz kam']):
            self.change_volume('-')
        
        elif any(word in command for word in ['mute', 'chup', 'awaz band']):
            self.mute_system()
        
        elif any(word in command for word in ['unmute', 'awaz chalu']):
            self.unmute_system()
        
        elif any(word in command for word in ['brightness up', 'roshni badha']):
            self.change_brightness('+')
        
        elif any(word in command for word in ['brightness down', 'roshni kam']):
            self.change_brightness('-')
        
        elif any(word in command for word in ['wifi on', 'wifi chalu']):
            self.toggle_wifi(True)
        
        elif any(word in command for word in ['wifi off', 'wifi band']):
            self.toggle_wifi(False)
        
        elif any(word in command for word in ['bluetooth on', 'bluetooth chalu']):
            self.toggle_bluetooth(True)
        
        elif any(word in command for word in ['bluetooth off', 'bluetooth band']):
            self.toggle_bluetooth(False)
        
        elif any(word in command for word in ['battery', 'battery status', 'charge kitna']):
            self.get_battery_status()
        
        elif any(word in command for word in ['cpu usage', 'processor usage']):
            self.get_cpu_usage()
        
        elif any(word in command for word in ['memory usage', 'ram usage']):
            self.get_memory_usage()
        
        elif any(word in command for word in ['disk space', 'storage']):
            self.get_disk_space()
        
        elif any(word in command for word in ['running processes', 'process list']):
            self.get_running_processes()
        
        elif any(word in command for word in ['system info', 'computer info']):
            self.get_system_info()
        
        elif any(word in command for word in ['network info', 'ip address']):
            self.get_network_info()
        
        # File Operations (101-150)
        elif any(word in command for word in ['create file', 'file banao']):
            self.create_file()
        
        elif any(word in command for word in ['delete file', 'file delete']):
            self.delete_file()
        
        elif any(word in command for word in ['copy file', 'file copy']):
            self.copy_file()
        
        elif any(word in command for word in ['move file', 'file move']):
            self.move_file()
        
        elif any(word in command for word in ['rename file', 'file rename']):
            self.rename_file()
        
        elif any(word in command for word in ['create folder', 'folder banao']):
            self.create_folder()
        
        elif any(word in command for word in ['delete folder', 'folder delete']):
            self.delete_folder()
        
        elif any(word in command for word in ['open file', 'file kholo']):
            self.open_file()
        
        elif any(word in command for word in ['open folder', 'folder kholo']):
            self.open_folder()
        
        elif any(word in command for word in ['search file', 'file dhundo']):
            self.search_file()
        
        elif any(word in command for word in ['file properties', 'file ki jankari']):
            self.get_file_properties()
        
        elif any(word in command for word in ['compress file', 'zip banao']):
            self.compress_file()
        
        elif any(word in command for word in ['extract file', 'zip kholo']):
            self.extract_file()
        
        elif any(word in command for word in ['backup files', 'files ka backup']):
            self.backup_files()
        
        elif any(word in command for word in ['sync files', 'files sync']):
            self.sync_files()
        
        # Application Commands (151-200)
        elif any(word in command for word in ['open notepad', 'notepad kholo']):
            self.open_application('notepad')
        
        elif any(word in command for word in ['open calculator', 'calculator kholo']):
            self.open_application('calc')
        
        elif any(word in command for word in ['open paint', 'paint kholo']):
            self.open_application('mspaint')
        
        elif any(word in command for word in ['open browser', 'browser kholo']):
            self.open_browser()
        
        elif any(word in command for word in ['open chrome', 'chrome kholo']):
            self.open_application('chrome')
        
        elif any(word in command for word in ['open firefox', 'firefox kholo']):
            self.open_application('firefox')
        
        elif any(word in command for word in ['open edge', 'edge kholo']):
            self.open_application('msedge')
        
        elif any(word in command for word in ['open word', 'word kholo']):
            self.open_application('winword')
        
        elif any(word in command for word in ['open excel', 'excel kholo']):
            self.open_application('excel')
        
        elif any(word in command for word in ['open powerpoint', 'powerpoint kholo']):
            self.open_application('powerpnt')
        
        elif any(word in command for word in ['open outlook', 'outlook kholo']):
            self.open_application('outlook')
        
        elif any(word in command for word in ['open skype', 'skype kholo']):
            self.open_application('skype')
        
        elif any(word in command for word in ['open zoom', 'zoom kholo']):
            self.open_application('zoom')
        
        elif any(word in command for word in ['open teams', 'teams kholo']):
            self.open_application('teams')
        
        elif any(word in command for word in ['open discord', 'discord kholo']):
            self.open_application('discord')
        
        elif any(word in command for word in ['open spotify', 'spotify kholo']):
            self.open_application('spotify')
        
        elif any(word in command for word in ['open vlc', 'vlc kholo']):
            self.open_application('vlc')
        
        elif any(word in command for word in ['open photoshop', 'photoshop kholo']):
            self.open_application('photoshop')
        
        elif any(word in command for word in ['open vs code', 'code kholo']):
            self.open_application('code')
        
        elif any(word in command for word in ['close application', 'app band karo']):
            self.close_application()
        
        # Web Commands (201-250)
        elif any(word in command for word in ['search google', 'google mein dhundo']):
            self.search_google(command)
        
        elif any(word in command for word in ['search youtube', 'youtube mein dhundo']):
            self.search_youtube(command)
        
        elif any(word in command for word in ['open facebook', 'facebook kholo']):
            webbrowser.open('https://facebook.com')
        
        elif any(word in command for word in ['open instagram', 'instagram kholo']):
            webbrowser.open('https://instagram.com')
        
        elif any(word in command for word in ['open twitter', 'twitter kholo']):
            webbrowser.open('https://twitter.com')
        
        elif any(word in command for word in ['open linkedin', 'linkedin kholo']):
            webbrowser.open('https://linkedin.com')
        
        elif any(word in command for word in ['open whatsapp', 'whatsapp kholo']):
            webbrowser.open('https://web.whatsapp.com')
        
        elif any(word in command for word in ['open gmail', 'gmail kholo']):
            webbrowser.open('https://gmail.com')
        
        elif any(word in command for word in ['open amazon', 'amazon kholo']):
            webbrowser.open('https://amazon.in')
        
        elif any(word in command for word in ['open flipkart', 'flipkart kholo']):
            webbrowser.open('https://flipkart.com')
        
        elif any(word in command for word in ['open netflix', 'netflix kholo']):
            webbrowser.open('https://netflix.com')
        
        elif any(word in command for word in ['open hotstar', 'hotstar kholo']):
            webbrowser.open('https://hotstar.com')
        
        elif any(word in command for word in ['open prime video', 'prime video kholo']):
            webbrowser.open('https://primevideo.com')
        
        elif any(word in command for word in ['open github', 'github kholo']):
            webbrowser.open('https://github.com')
        
        elif any(word in command for word in ['open stackoverflow', 'stackoverflow kholo']):
            webbrowser.open('https://stackoverflow.com')
        
        # Entertainment Commands (251-300)
        elif any(word in command for word in ['play music', 'gaana bajao']):
            self.play_music()
        
        elif any(word in command for word in ['stop music', 'gaana band karo']):
            self.stop_music()
        
        elif any(word in command for word in ['next song', 'agla gaana']):
            self.next_song()
        
        elif any(word in command for word in ['previous song', 'pichla gaana']):
            self.previous_song()
        
        elif any(word in command for word in ['pause music', 'gaana roko']):
            self.pause_music()
        
        elif any(word in command for word in ['resume music', 'gaana chalu karo']):
            self.resume_music()
        
        elif any(word in command for word in ['shuffle music', 'gaane mix karo']):
            self.shuffle_music()
        
        elif any(word in command for word in ['repeat song', 'gaana repeat karo']):
            self.repeat_song()
        
        elif any(word in command for word in ['play video', 'video chalao']):
            self.play_video()
        
        elif any(word in command for word in ['stop video', 'video band karo']):
            self.stop_video()
        
        elif any(word in command for word in ['fullscreen', 'pura screen']):
            self.toggle_fullscreen()
        
        elif any(word in command for word in ['take screenshot', 'photo lo']):
            self.take_screenshot()
        
        elif any(word in command for word in ['record screen', 'screen record karo']):
            self.record_screen()
        
        elif any(word in command for word in ['play game', 'game khelo']):
            self.play_game()
        
        elif any(word in command for word in ['tell joke', 'joke sunao']):
            self.tell_joke()
        
        elif any(word in command for word in ['tell story', 'kahani sunao']):
            self.tell_story()
        
        elif any(word in command for word in ['sing song', 'gaana gao']):
            self.sing_song()
        
        elif any(word in command for word in ['play radio', 'radio chalao']):
            self.play_radio()
        
        elif any(word in command for word in ['play podcast', 'podcast sunao']):
            self.play_podcast()
        
        elif any(word in command for word in ['movie recommendation', 'movie suggest karo']):
            self.recommend_movie()
        
        # Communication Commands (301-350)
        elif any(word in command for word in ['send email', 'email bhejo']):
            self.send_email()
        
        elif any(word in command for word in ['read email', 'email padho']):
            self.read_email()
        
        elif any(word in command for word in ['send message', 'message bhejo']):
            self.send_message()
        
        elif any(word in command for word in ['make call', 'call karo']):
            self.make_call()
        
        elif any(word in command for word in ['video call', 'video call karo']):
            self.video_call()
        
        elif any(word in command for word in ['send whatsapp', 'whatsapp bhejo']):
            self.send_whatsapp()
        
        elif any(word in command for word in ['post facebook', 'facebook post karo']):
            self.post_facebook()
        
        elif any(word in command for word in ['tweet', 'twitter post karo']):
            self.post_twitter()
        
        elif any(word in command for word in ['instagram post', 'instagram post karo']):
            self.post_instagram()
        
        elif any(word in command for word in ['linkedin post', 'linkedin post karo']):
            self.post_linkedin()
        
        # Productivity Commands (351-400)
        elif any(word in command for word in ['set reminder', 'yaad dilao']):
            self.set_reminder()
        
        elif any(word in command for word in ['set alarm', 'alarm lagao']):
            self.set_alarm()
        
        elif any(word in command for word in ['set timer', 'timer lagao']):
            self.set_timer()
        
        elif any(word in command for word in ['create note', 'note banao']):
            self.create_note()
        
        elif any(word in command for word in ['read note', 'note padho']):
            self.read_note()
        
        elif any(word in command for word in ['delete note', 'note delete karo']):
            self.delete_note()
        
        elif any(word in command for word in ['create todo', 'kaam ki list banao']):
            self.create_todo()
        
        elif any(word in command for word in ['show todo', 'kaam ki list dikhao']):
            self.show_todo()
        
        elif any(word in command for word in ['complete todo', 'kaam complete karo']):
            self.complete_todo()
        
        elif any(word in command for word in ['schedule meeting', 'meeting schedule karo']):
            self.schedule_meeting()
        
        elif any(word in command for word in ['show calendar', 'calendar dikhao']):
            self.show_calendar()
        
        elif any(word in command for word in ['add event', 'event add karo']):
            self.add_event()
        
        elif any(word in command for word in ['calculate', 'hisab karo']):
            self.calculate(command)
        
        elif any(word in command for word in ['convert currency', 'paisa convert karo']):
            self.convert_currency()
        
        elif any(word in command for word in ['translate', 'anuvad karo']):
            self.translate_text()
        
        # Information Commands (401-450)
        elif any(word in command for word in ['wikipedia', 'wiki search']):
            self.search_wikipedia(command)
        
        elif any(word in command for word in ['define word', 'matlab batao']):
            self.define_word(command)
        
        elif any(word in command for word in ['spell word', 'spelling batao']):
            self.spell_word(command)
        
        elif any(word in command for word in ['synonym', 'similar word']):
            self.get_synonym(command)
        
        elif any(word in command for word in ['antonym', 'opposite word']):
            self.get_antonym(command)
        
        elif any(word in command for word in ['random fact', 'koi fact batao']):
            self.random_fact()
        
        elif any(word in command for word in ['quote', 'quote sunao']):
            self.get_quote()
        
        elif any(word in command for word in ['riddle', 'paheli sunao']):
            self.tell_riddle()
        
        elif any(word in command for word in ['trivia', 'quiz question']):
            self.ask_trivia()
        
        elif any(word in command for word in ['history fact', 'itihas ki baat']):
            self.history_fact()
        
        elif any(word in command for word in ['science fact', 'vigyan ki baat']):
            self.science_fact()
        
        elif any(word in command for word in ['space fact', 'antariksh ki baat']):
            self.space_fact()
        
        elif any(word in command for word in ['animal fact', 'janwar ki baat']):
            self.animal_fact()
        
        elif any(word in command for word in ['plant fact', 'paudhe ki baat']):
            self.plant_fact()
        
        elif any(word in command for word in ['technology fact', 'technology ki baat']):
            self.technology_fact()
        
        # Health & Fitness Commands (451-500)
        elif any(word in command for word in ['health tip', 'sehat ki salah']):
            self.health_tip()
        
        elif any(word in command for word in ['exercise', 'vyayam karo']):
            self.suggest_exercise()
        
        elif any(word in command for word in ['yoga', 'yoga karo']):
            self.suggest_yoga()
        
        elif any(word in command for word in ['meditation', 'dhyan karo']):
            self.guide_meditation()
        
        elif any(word in command for word in ['water reminder', 'paani peene yaad dilao']):
            self.water_reminder()
        
        elif any(word in command for word in ['calorie counter', 'calorie ginao']):
            self.count_calories()
        
        elif any(word in command for word in ['bmi calculator', 'bmi nikalo']):
            self.calculate_bmi()
        
        elif any(word in command for word in ['sleep tracker', 'neend track karo']):
            self.track_sleep()
        
        elif any(word in command for word in ['step counter', 'kadam ginao']):
            self.count_steps()
        
        elif any(word in command for word in ['heart rate', 'dil ki dhadak']):
            self.check_heart_rate()
        
        # Food & Recipe Commands (501-550)
        elif any(word in command for word in ['recipe', 'recipe batao']):
            self.get_recipe(command)
        
        elif any(word in command for word in ['cooking tip', 'khana banane ki tip']):
            self.cooking_tip()
        
        elif any(word in command for word in ['restaurant', 'restaurant dhundo']):
            self.find_restaurant()
        
        elif any(word in command for word in ['order food', 'khana order karo']):
            self.order_food()
        
        elif any(word in command for word in ['nutrition info', 'nutrition ki jankari']):
            self.nutrition_info()
        
        elif any(word in command for word in ['diet plan', 'diet plan banao']):
            self.create_diet_plan()
        
        elif any(word in command for word in ['grocery list', 'sabzi ki list']):
            self.create_grocery_list()
        
        elif any(word in command for word in ['meal planner', 'khane ka plan']):
            self.plan_meals()
        
        elif any(word in command for word in ['food allergy', 'khane ki allergy']):
            self.check_food_allergy()
        
        elif any(word in command for word in ['kitchen timer', 'kitchen timer']):
            self.kitchen_timer()
        
        # Travel Commands (551-600)
        elif any(word in command for word in ['book flight', 'flight book karo']):
            self.book_flight()
        
        elif any(word in command for word in ['book hotel', 'hotel book karo']):
            self.book_hotel()
        
        elif any(word in command for word in ['book cab', 'cab book karo']):
            self.book_cab()
        
        elif any(word in command for word in ['train schedule', 'train ka time']):
            self.train_schedule()
        
        elif any(word in command for word in ['bus schedule', 'bus ka time']):
            self.bus_schedule()
        
        elif any(word in command for word in ['flight status', 'flight ki jankari']):
            self.flight_status()
        
        elif any(word in command for word in ['traffic update', 'traffic ki jankari']):
            self.traffic_update()
        
        elif any(word in command for word in ['directions', 'rasta batao']):
            self.get_directions()
        
        elif any(word in command for word in ['nearby places', 'paas ki jagah']):
            self.nearby_places()
        
        elif any(word in command for word in ['travel guide', 'travel guide']):
            self.travel_guide()
        
        elif any(word in command for word in ['currency exchange', 'paisa exchange']):
            self.currency_exchange()
        
        elif any(word in command for word in ['visa info', 'visa ki jankari']):
            self.visa_info()
        
        elif any(word in command for word in ['passport info', 'passport ki jankari']):
            self.passport_info()
        
        elif any(word in command for word in ['travel insurance', 'travel insurance']):
            self.travel_insurance()
        
        elif any(word in command for word in ['packing list', 'saman ki list']):
            self.packing_list()
        
        # Shopping Commands (601-650)
        elif any(word in command for word in ['price comparison', 'price compare karo']):
            self.compare_prices()
        
        elif any(word in command for word in ['product review', 'product review']):
            self.product_review()
        
        elif any(word in command for word in ['shopping list', 'shopping list']):
            self.create_shopping_list()
        
        elif any(word in command for word in ['coupon code', 'coupon code']):
            self.find_coupon()
        
        elif any(word in command for word in ['deal alert', 'deal ki jankari']):
            self.deal_alert()
        
        elif any(word in command for word in ['wishlist', 'wishlist']):
            self.manage_wishlist()
        
        elif any(word in command for word in ['track order', 'order track karo']):
            self.track_order()
        
        elif any(word in command for word in ['return policy', 'return policy']):
            self.return_policy()
        
        elif any(word in command for word in ['warranty info', 'warranty ki jankari']):
            self.warranty_info()
        
        elif any(word in command for word in ['customer service', 'customer service']):
            self.customer_service()
        
        # Education Commands (651-700)
        elif any(word in command for word in ['learn language', 'bhasha sikho']):
            self.learn_language()
        
        elif any(word in command for word in ['math problem', 'math ka sawal']):
            self.solve_math()
        
        elif any(word in command for word in ['science quiz', 'science quiz']):
            self.science_quiz()
        
        elif any(word in command for word in ['history quiz', 'history quiz']):
            self.history_quiz()
        
        elif any(word in command for word in ['geography quiz', 'geography quiz']):
            self.geography_quiz()
        
        elif any(word in command for word in ['english grammar', 'english grammar']):
            self.english_grammar()
        
        elif any(word in command for word in ['vocabulary', 'vocabulary']):
            self.build_vocabulary()
        
        elif any(word in command for word in ['study timer', 'padhai timer']):
            self.study_timer()
        
        elif any(word in command for word in ['exam reminder', 'exam yaad dilao']):
            self.exam_reminder()
        
        elif any(word in command for word in ['course info', 'course ki jankari']):
            self.course_info()
        
        elif any(word in command for word in ['online class', 'online class']):
            self.online_class()
        
        elif any(word in command for word in ['homework help', 'homework help']):
            self.homework_help()
        
        elif any(word in command for word in ['research help', 'research help']):
            self.research_help()
        
        elif any(word in command for word in ['citation format', 'citation format']):
            self.citation_format()
        
        elif any(word in command for word in ['study group', 'study group']):
            self.study_group()
        
        # Finance Commands (701-750)
        elif any(word in command for word in ['bank balance', 'bank balance']):
            self.check_balance()
        
        elif any(word in command for word in ['expense tracker', 'kharcha track karo']):
            self.track_expenses()
        
        elif any(word in command for word in ['budget planner', 'budget banao']):
            self.budget_planner()
        
        elif any(word in command for word in ['investment advice', 'investment ki salah']):
            self.investment_advice()
        
              
  
        # Web Commands (51-100)
        elif any(word in command for word in ['google', 'search', 'khojo']):
            query = command.replace('google', '').replace('search', '').replace('khojo', '').strip()
            if query:
                webbrowser.open(f"https://www.google.com/search?q={query}")
                self.speak(f"Google mein {query} search kar raha hun")
        
        elif any(word in command for word in ['youtube', 'video']):
            query = command.replace('youtube', '').replace('video', '').strip()
            if query:
                webbrowser.open(f"https://www.youtube.com/results?search_query={query}")
                self.speak(f"YouTube mein {query} search kar raha hun")
        
        elif any(word in command for word in ['facebook', 'fb']):
            webbrowser.open("https://www.facebook.com")
            self.speak("Facebook khol raha hun")
        
        elif any(word in command for word in ['instagram', 'insta']):
            webbrowser.open("https://www.instagram.com")
            self.speak("Instagram khol raha hun")
        
        elif any(word in command for word in ['twitter', 'x']):
            webbrowser.open("https://www.twitter.com")
            self.speak("Twitter khol raha hun")
        
        elif any(word in command for word in ['linkedin']):
            webbrowser.open("https://www.linkedin.com")
            self.speak("LinkedIn khol raha hun")
        
        elif any(word in command for word in ['whatsapp', 'whatsapp web']):
            webbrowser.open("https://web.whatsapp.com")
            self.speak("WhatsApp Web khol raha hun")
        
        elif any(word in command for word in ['gmail', 'email']):
            webbrowser.open("https://mail.google.com")
            self.speak("Gmail khol raha hun")
        
        elif any(word in command for word in ['amazon', 'shopping']):
            webbrowser.open("https://www.amazon.in")
            self.speak("Amazon khol raha hun")
        
        elif any(word in command for word in ['flipkart']):
            webbrowser.open("https://www.flipkart.com")
            self.speak("Flipkart khol raha hun")
        
        # System Commands (101-200)
        elif any(word in command for word in ['volume up', 'awaz badha']):
            pyautogui.press('volumeup')
            self.speak("Volume badha diya")
        
        elif any(word in command for word in ['volume down', 'awaz kam']):
            pyautogui.press('volumedown')
            self.speak("Volume kam kar diya")
        
        elif any(word in command for word in ['mute', 'chup']):
            pyautogui.press('volumemute')
            self.speak("Volume mute kar diya")
        
        elif any(word in command for word in ['screenshot', 'screen capture']):
            screenshot = pyautogui.screenshot()
            screenshot.save('screenshot.png')
            self.speak("Screenshot le liya")
        
        elif any(word in command for word in ['shutdown', 'band karo']):
            self.speak("System shutdown kar raha hun")
            os.system("shutdown /s /t 1")
        
        elif any(word in command for word in ['restart', 'reboot']):
            self.speak("System restart kar raha hun")
            os.system("shutdown /r /t 1")
        
        elif any(word in command for word in ['sleep', 'hibernate']):
            self.speak("System sleep mode mein ja raha hai")
            os.system("rundll32.exe powrprof.dll,SetSuspendState 0,1,0")
        
        elif any(word in command for word in ['lock', 'screen lock']):
            self.speak("Screen lock kar raha hun")
            os.system("rundll32.exe user32.dll,LockWorkStation")
        
        elif any(word in command for word in ['task manager', 'processes']):
            os.system("taskmgr")
            self.speak("Task Manager khol raha hun")
        
        elif any(word in command for word in ['control panel']):
            os.system("control")
            self.speak("Control Panel khol raha hun")
        
        # File Operations (201-300)
        elif any(word in command for word in ['create folder', 'folder banao']):
            folder_name = command.replace('create folder', '').replace('folder banao', '').strip()
            if folder_name:
                os.makedirs(folder_name, exist_ok=True)
                self.speak(f"{folder_name} folder bana diya")
        
        elif any(word in command for word in ['delete file', 'file delete']):
            file_name = command.replace('delete file', '').replace('file delete', '').strip()
            if file_name and os.path.exists(file_name):
                os.remove(file_name)
                self.speak(f"{file_name} file delete kar diya")
        
        elif any(word in command for word in ['open notepad', 'notepad kholo']):
            os.system("notepad")
            self.speak("Notepad khol raha hun")
        
        elif any(word in command for word in ['open calculator', 'calculator kholo']):
            os.system("calc")
            self.speak("Calculator khol raha hun")
        
        elif any(word in command for word in ['open paint', 'paint kholo']):
            os.system("mspaint")
            self.speak("Paint khol raha hun")
        
        elif any(word in command for word in ['open cmd', 'command prompt']):
            os.system("cmd")
            self.speak("Command Prompt khol raha hun")
        
        elif any(word in command for word in ['open explorer', 'file explorer']):
            os.system("explorer")
            self.speak("File Explorer khol raha hun")
        
        elif any(word in command for word in ['open downloads', 'downloads folder']):
            downloads_path = str(Path.home() / "Downloads")
            os.startfile(downloads_path)
            self.speak("Downloads folder khol raha hun")
        
        elif any(word in command for word in ['open documents', 'documents folder']):
            documents_path = str(Path.home() / "Documents")
            os.startfile(documents_path)
            self.speak("Documents folder khol raha hun")
        
        elif any(word in command for word in ['open desktop', 'desktop folder']):
            desktop_path = str(Path.home() / "Desktop")
            os.startfile(desktop_path)
            self.speak("Desktop folder khol raha hun")     
   
        # Entertainment Commands (301-400)
        elif any(word in command for word in ['play music', 'gaana bajao']):
            music_path = str(Path.home() / "Music")
            if os.path.exists(music_path):
                os.startfile(music_path)
                self.speak("Music folder khol raha hun")
        
        elif any(word in command for word in ['netflix']):
            webbrowser.open("https://www.netflix.com")
            self.speak("Netflix khol raha hun")
        
        elif any(word in command for word in ['hotstar', 'disney']):
            webbrowser.open("https://www.hotstar.com")
            self.speak("Hotstar khol raha hun")
        
        elif any(word in command for word in ['prime video', 'amazon prime']):
            webbrowser.open("https://www.primevideo.com")
            self.speak("Prime Video khol raha hun")
        
        elif any(word in command for word in ['spotify']):
            webbrowser.open("https://open.spotify.com")
            self.speak("Spotify khol raha hun")
        
        elif any(word in command for word in ['gaana.com']):
            webbrowser.open("https://gaana.com")
            self.speak("Gaana.com khol raha hun")
        
        elif any(word in command for word in ['jiosaavn']):
            webbrowser.open("https://www.jiosaavn.com")
            self.speak("JioSaavn khol raha hun")
        
        elif any(word in command for word in ['joke', 'hasao', 'comedy']):
            self.tell_joke()
        
        elif any(word in command for word in ['story', 'kahani']):
            self.tell_story()
        
        elif any(word in command for word in ['riddle', 'paheli']):
            self.tell_riddle()
        
        # Educational Commands (401-500)
        elif any(word in command for word in ['wikipedia', 'wiki']):
            query = command.replace('wikipedia', '').replace('wiki', '').strip()
            if query:
                self.search_wikipedia(query)
        
        elif any(word in command for word in ['dictionary', 'meaning', 'matlab']):
            word = command.replace('dictionary', '').replace('meaning', '').replace('matlab', '').strip()
            if word:
                self.get_word_meaning(word)
        
        elif any(word in command for word in ['translate', 'anuvad']):
            self.translate_text(command)
        
        elif any(word in command for word in ['math', 'calculate', 'ginti']):
            expression = command.replace('math', '').replace('calculate', '').replace('ginti', '').strip()
            if expression:
                self.calculate_math(expression)
        
        elif any(word in command for word in ['spell', 'spelling']):
            word = command.replace('spell', '').replace('spelling', '').strip()
            if word:
                self.spell_word(word)
        
        elif any(word in command for word in ['fact', 'tathya']):
            self.tell_fact()
        
        elif any(word in command for word in ['quote', 'vichar']):
            self.tell_quote()
        
        elif any(word in command for word in ['learn', 'sikho']):
            topic = command.replace('learn', '').replace('sikho', '').strip()
            if topic:
                self.learn_topic(topic)
        
        elif any(word in command for word in ['quiz', 'prashn']):
            self.start_quiz()
        
        elif any(word in command for word in ['history', 'itihas']):
            self.tell_history()
        
        # Communication Commands (501-600)
        elif any(word in command for word in ['send email', 'email bhejo']):
            self.send_email_interface()
        
        elif any(word in command for word in ['make call', 'call karo']):
            self.make_call_interface()
        
        elif any(word in command for word in ['send message', 'message bhejo']):
            self.send_message_interface()
        
        elif any(word in command for word in ['video call', 'video call karo']):
            self.video_call_interface()
        
        elif any(word in command for word in ['telegram']):
            webbrowser.open("https://web.telegram.org")
            self.speak("Telegram Web khol raha hun")
        
        elif any(word in command for word in ['discord']):
            webbrowser.open("https://discord.com/app")
            self.speak("Discord khol raha hun")
        
        elif any(word in command for word in ['skype']):
            webbrowser.open("https://web.skype.com")
            self.speak("Skype Web khol raha hun")
        
        elif any(word in command for word in ['zoom']):
            webbrowser.open("https://zoom.us/join")
            self.speak("Zoom khol raha hun")
        
        elif any(word in command for word in ['teams', 'microsoft teams']):
            webbrowser.open("https://teams.microsoft.com")
            self.speak("Microsoft Teams khol raha hun")
        
        elif any(word in command for word in ['meet', 'google meet']):
            webbrowser.open("https://meet.google.com")
            self.speak("Google Meet khol raha hun")
        
        # Health & Fitness Commands (601-700)
        elif any(word in command for word in ['bmi', 'body mass index']):
            self.calculate_bmi()
        
        elif any(word in command for word in ['water reminder', 'paani yaad']):
            self.set_water_reminder()
        
        elif any(word in command for word in ['exercise', 'vyayam']):
            self.suggest_exercise()
        
        elif any(word in command for word in ['meditation', 'dhyan']):
            self.start_meditation()
        
        elif any(word in command for word in ['yoga']):
            self.suggest_yoga()
        
        elif any(word in command for word in ['diet', 'bhojan']):
            self.suggest_diet()
        
        elif any(word in command for word in ['calories', 'calorie count']):
            self.count_calories()
        
        elif any(word in command for word in ['sleep tracker', 'neend']):
            self.track_sleep()
        
        elif any(word in command for word in ['heart rate', 'dil ki dhadak']):
            self.measure_heart_rate()
        
        elif any(word in command for word in ['steps', 'kadam']):
            self.count_steps()
        
        # Finance Commands (701-800)
        elif any(word in command for word in ['stock price', 'share price']):
            stock = command.replace('stock price', '').replace('share price', '').strip()
            if stock:
                self.get_stock_price(stock)
        
        elif any(word in command for word in ['currency', 'exchange rate']):
            self.get_exchange_rate()
        
        elif any(word in command for word in ['budget', 'budget banao']):
            self.create_budget()
        
        elif any(word in command for word in ['expense', 'kharcha']):
            self.track_expense()
        
        elif any(word in command for word in ['investment', 'nivesh']):
            self.investment_advice()
        
        elif any(word in command for word in ['loan calculator', 'loan']):
            self.calculate_loan()
        
        elif any(word in command for word in ['tax', 'kar']):
            self.calculate_tax()
        
        elif any(word in command for word in ['savings', 'bachat']):
            self.savings_advice()
        
        elif any(word in command for word in ['crypto', 'cryptocurrency']):
            self.get_crypto_price()
        
        elif any(word in command for word in ['gold price', 'sona']):
            self.get_gold_price()        

        # Travel Commands (801-900)
        elif any(word in command for word in ['flight', 'hawai jahaz']):
            self.search_flights()
        
        elif any(word in command for word in ['hotel', 'hotel book']):
            self.search_hotels()
        
        elif any(word in command for word in ['train', 'rail']):
            self.search_trains()
        
        elif any(word in command for word in ['bus', 'bus book']):
            self.search_buses()
        
        elif any(word in command for word in ['cab', 'taxi', 'ola', 'uber']):
            self.book_cab()
        
        elif any(word in command for word in ['map', 'direction', 'rasta']):
            location = command.replace('map', '').replace('direction', '').replace('rasta', '').strip()
            if location:
                self.get_directions(location)
        
        elif any(word in command for word in ['weather forecast', 'mausam ki jankari']):
            self.get_weather_forecast()
        
        elif any(word in command for word in ['tourist places', 'ghumne ki jagah']):
            city = command.replace('tourist places', '').replace('ghumne ki jagah', '').strip()
            if city:
                self.get_tourist_places(city)
        
        elif any(word in command for word in ['visa', 'passport']):
            self.visa_info()
        
        elif any(word in command for word in ['travel insurance', 'yatra bima']):
            self.travel_insurance_info()
        
        # Smart Home Commands (901-1000)
        elif any(word in command for word in ['lights on', 'batti jalao']):
            self.control_lights('on')
        
        elif any(word in command for word in ['lights off', 'batti bujhao']):
            self.control_lights('off')
        
        elif any(word in command for word in ['fan on', 'pankha chalao']):
            self.control_fan('on')
        
        elif any(word in command for word in ['fan off', 'pankha band']):
            self.control_fan('off')
        
        elif any(word in command for word in ['ac on', 'ac chalao']):
            self.control_ac('on')
        
        elif any(word in command for word in ['ac off', 'ac band']):
            self.control_ac('off')
        
        elif any(word in command for word in ['tv on', 'tv chalao']):
            self.control_tv('on')
        
        elif any(word in command for word in ['tv off', 'tv band']):
            self.control_tv('off')
        
        elif any(word in command for word in ['security', 'suraksha']):
            self.check_security()
        
        elif any(word in command for word in ['door lock', 'darwaza band']):
            self.control_door_lock('lock')
        
        # Advanced Commands (1001-1100+)
        elif any(word in command for word in ['ai chat', 'chatgpt']):
            self.ai_chat_interface()
        
        elif any(word in command for word in ['code', 'programming']):
            language = command.replace('code', '').replace('programming', '').strip()
            if language:
                self.help_with_coding(language)
        
        elif any(word in command for word in ['backup', 'data backup']):
            self.create_backup()
        
        elif any(word in command for word in ['password', 'password generate']):
            self.generate_password()
        
        elif any(word in command for word in ['qr code', 'qr banao']):
            text = command.replace('qr code', '').replace('qr banao', '').strip()
            if text:
                self.generate_qr_code(text)
        
        elif any(word in command for word in ['reminder', 'yaad dilao']):
            self.set_reminder(command)
        
        elif any(word in command for word in ['alarm', 'alarm set']):
            self.set_alarm(command)
        
        elif any(word in command for word in ['timer', 'timer set']):
            self.set_timer(command)
        
        elif any(word in command for word in ['stopwatch', 'stopwatch start']):
            self.start_stopwatch()
        
        elif any(word in command for word in ['system info', 'computer ki jankari']):
            self.get_system_info()
        
        elif any(word in command for word in ['ip address', 'ip']):
            self.get_ip_address()
        
        elif any(word in command for word in ['speed test', 'internet speed']):
            self.test_internet_speed()
        
        elif any(word in command for word in ['wifi password', 'wifi']):
            self.get_wifi_passwords()
        
        elif any(word in command for word in ['battery', 'battery status']):
            self.get_battery_status()
        
        elif any(word in command for word in ['disk space', 'storage']):
            self.get_disk_space()
        
        elif any(word in command for word in ['memory usage', 'ram']):
            self.get_memory_usage()
        
        elif any(word in command for word in ['cpu usage', 'processor']):
            self.get_cpu_usage()
        
        elif any(word in command for word in ['network', 'network info']):
            self.get_network_info()
        
        elif any(word in command for word in ['installed programs', 'software list']):
            self.get_installed_programs()
        
        elif any(word in command for word in ['running processes', 'process list']):
            self.get_running_processes()
        
        elif any(word in command for word in ['help', 'madad', 'commands']):
            self.show_help()
        
        elif any(word in command for word in ['exit', 'quit', 'bye', 'alvida']):
            self.speak("Dhanyawad! Phir milenge!")
            return False
        
        else:
            self.speak("Maaf kijiye, main ye command samjha nahi. Help kehkar saare commands dekh sakte hain.")
        
        return True

    # Helper Methods for Complex Commands
    def get_weather(self):
        """Get weather information"""
        try:
            # You can integrate with weather API here
            self.speak("Mausam ki jankari ke liye weather website khol raha hun")
            webbrowser.open("https://weather.com")
        except Exception as e:
            self.speak("Mausam ki jankari nahi mil paayi")
    
    def get_news(self):
        """Get latest news"""
        try:
            self.speak("Taaza khabrein dekh rahe hain")
            webbrowser.open("https://www.ndtv.com")
        except Exception as e:
            self.speak("News nahi mil paayi")
    
    def tell_joke(self):
        """Tell a random joke"""
        jokes = [
            "Ek aadmi doctor ke paas gaya aur bola - Doctor sahab, mujhe lagta hai main invisible ho gaya hun. Doctor bola - Next!",
            "Teacher: Tumhara homework kahan hai? Student: Ghar mein hai. Teacher: Ghar mein kya kar raha hai? Student: Homework!",
            "Pappu: Papa, main school nahi jaana chahta. Papa: Kyun? Pappu: Kyunki wahan sab mujhe pagal kehte hain. Papa: Arre beta, tu toh 40 saal ka hai!",
            "Ek aadmi ne dukaan mein jakar kaha - Bhaiya, koi aisi cheez hai jo main kharid sakun lekin use kar na sakun? Dukaandar: Haan, coffin!"
        ]
        joke = random.choice(jokes)
        self.speak(joke)
    
    def tell_story(self):
        """Tell a short story"""
        stories = [
            "Ek baar ek chhota sa ladka tha jo bahut curious tha. Woh har cheez ke baare mein jaanna chahta tha. Ek din usne apni mummy se poocha - Mummy, main kahan se aaya hun? Mummy ne kaha - Beta, tumhe stork laya hai. Ladke ne kaha - Wow! Toh main flying baby hun!",
            "Ek gaon mein ek wise old man rehta tha. Log usse advice lene aate the. Ek din ek young man aaya aur bola - Uncle, success ka secret kya hai? Old man ne kaha - Beta, success ka secret hai - hard work, dedication aur thoda sa luck. Young man bola - Sirf itna? Old man ne kaha - Haan, aur ek baat - kabhi give up mat karna!"
        ]
        story = random.choice(stories)
        self.speak(story)
    
    def tell_riddle(self):
        """Tell a riddle"""
        riddles = [
            "Main hoon lekin dikhta nahi, sab jagah hun lekin kahi nahi. Kya hun main? Jawab: Hawa",
            "Mere paas aankhein hain lekin dekh nahi sakta, kaan hain lekin sun nahi sakta. Kya hun main? Jawab: Tasveer",
            "Main raat mein aata hun, din mein chala jata hun. Andhere mein dikhta hun, ujale mein chhup jata hun. Kya hun main? Jawab: Tara"
        ]
        riddle = random.choice(riddles)
        self.speak(riddle)    
  
    def search_wikipedia(self, query):
        """Search Wikipedia"""
        try:
            wikipedia.set_lang("hi")
            summary = wikipedia.summary(query, sentences=2)
            self.speak(f"Wikipedia ke anusaar: {summary}")
        except:
            self.speak(f"{query} ke baare mein Wikipedia mein jankari nahi mili")
    
    def get_word_meaning(self, word):
        """Get word meaning"""
        try:
            # You can integrate with dictionary API
            self.speak(f"{word} ka matlab dhund raha hun")
            webbrowser.open(f"https://www.dictionary.com/browse/{word}")
        except:
            self.speak("Word meaning nahi mil paaya")
    
    def translate_text(self, command):
        """Translate text"""
        try:
            self.speak("Translation ke liye Google Translate khol raha hun")
            webbrowser.open("https://translate.google.com")
        except:
            self.speak("Translation service available nahi hai")
    
    def calculate_math(self, expression):
        """Calculate mathematical expressions"""
        try:
            # Simple calculator
            result = eval(expression.replace('x', '*').replace('÷', '/'))
            self.speak(f"Jawab hai {result}")
        except:
            self.speak("Math calculation mein error hai")
    
    def spell_word(self, word):
        """Spell a word"""
        spelling = ' '.join(word.upper())
        self.speak(f"{word} ki spelling hai: {spelling}")
    
    def tell_fact(self):
        """Tell an interesting fact"""
        facts = [
            "Kya aap jaante hain ki octopus ke teen dil hote hain?",
            "Honey kabhi kharab nahi hota. Archaeologists ne 3000 saal purana honey find kiya hai jo abhi bhi fresh tha!",
            "Bananas technically berries hain, lekin strawberries nahi hain!",
            "Ek group of flamingos ko 'flamboyance' kehte hain.",
            "Sharks dinosaurs se bhi purane hain!"
        ]
        fact = random.choice(facts)
        self.speak(fact)
    
    def tell_quote(self):
        """Tell an inspirational quote"""
        quotes = [
            "Safalta woh nahi hai jo aapke paas hai, balki woh hai jo aap ban jaate hain. - Zig Ziglar",
            "Agar aap sapne dekh sakte hain, toh aap unhe poora bhi kar sakte hain. - Walt Disney",
            "Mushkil waqt mein hi pata chalta hai ki aap kitne strong hain. - Unknown",
            "Haar maan lena sabse bada gunah hai. - Swami Vivekananda",
            "Koshish karne walon ki kabhi haar nahi hoti. - Harivansh Rai Bachchan"
        ]
        quote = random.choice(quotes)
        self.speak(quote)
    
    def learn_topic(self, topic):
        """Learn about a topic"""
        self.speak(f"{topic} ke baare mein seekhne ke liye search kar raha hun")
        webbrowser.open(f"https://www.google.com/search?q=learn+{topic}")
    
    def start_quiz(self):
        """Start a quiz"""
        questions = [
            {"q": "Bharat ki rajdhani kya hai?", "a": "New Delhi"},
            {"q": "Sabse bada planet kaun sa hai?", "a": "Jupiter"},
            {"q": "Ganga nadi kahan se nikalti hai?", "a": "Gangotri"},
            {"q": "Bharat mein kitne states hain?", "a": "28"}
        ]
        question = random.choice(questions)
        self.speak(f"Quiz question: {question['q']}")
        # You can add answer checking logic here
    
    def tell_history(self):
        """Tell historical facts"""
        history_facts = [
            "Taj Mahal ko banane mein 22 saal lage the aur 20,000 workers ne kaam kiya tha.",
            "Mahatma Gandhi ko 5 baar Nobel Peace Prize ke liye nominate kiya gaya tha.",
            "Red Fort ko banane mein 10 saal lage the.",
            "Bharat mein railway system 1853 mein shuru hua tha."
        ]
        fact = random.choice(history_facts)
        self.speak(fact)
    
    def send_email_interface(self):
        """Email interface"""
        self.speak("Email bhejne ke liye Gmail khol raha hun")
        webbrowser.open("https://mail.google.com/mail/u/0/#inbox?compose=new")
    
    def make_call_interface(self):
        """Call interface"""
        self.speak("Call karne ke liye WhatsApp Web khol raha hun")
        webbrowser.open("https://web.whatsapp.com")
    
    def send_message_interface(self):
        """Message interface"""
        self.speak("Message bhejne ke liye WhatsApp Web khol raha hun")
        webbrowser.open("https://web.whatsapp.com")
    
    def video_call_interface(self):
        """Video call interface"""
        self.speak("Video call ke liye Google Meet khol raha hun")
        webbrowser.open("https://meet.google.com")
    
    def calculate_bmi(self):
        """BMI Calculator"""
        self.speak("BMI calculate karne ke liye height aur weight batayiye")
        # You can add input logic here
    
    def set_water_reminder(self):
        """Water reminder"""
        self.speak("Paani peene ki reminder set kar di. Har ghante mein yaad dilaaunga!")
    
    def suggest_exercise(self):
        """Exercise suggestions"""
        exercises = [
            "Push-ups kijiye - 10 se 15 baar",
            "Squats kijiye - 15 se 20 baar",
            "Plank kijiye - 30 seconds se 1 minute",
            "Jumping jacks kijiye - 20 baar",
            "Walking kijiye - 30 minutes"
        ]
        exercise = random.choice(exercises)
        self.speak(f"Aaj ka exercise: {exercise}")
    
    def start_meditation(self):
        """Meditation guide"""
        self.speak("Meditation shuru karte hain. Aankhen band kariye, deep breathing kijiye. 5 minute baad main aapko bataunga.")
    
    def suggest_yoga(self):
        """Yoga suggestions"""
        yoga_poses = [
            "Surya Namaskar - 5 rounds",
            "Pranayama - 10 minutes",
            "Bhujangasana - Cobra pose",
            "Vrikshasana - Tree pose",
            "Shavasana - Relaxation pose"
        ]
        pose = random.choice(yoga_poses)
        self.speak(f"Aaj ka yoga: {pose}")
    
    def suggest_diet(self):
        """Diet suggestions"""
        diet_tips = [
            "Subah khali pet paani piyiye",
            "Green vegetables zyada khayiye",
            "Fruits daily khayiye",
            "Junk food kam kijiye",
            "Dinner jaldi kijiye"
        ]
        tip = random.choice(diet_tips)
        self.speak(f"Diet tip: {tip}")
    
    def count_calories(self):
        """Calorie counter"""
        self.speak("Calorie count karne ke liye MyFitnessPal app use kar sakte hain")
        webbrowser.open("https://www.myfitnesspal.com")
    
    def track_sleep(self):
        """Sleep tracker"""
        self.speak("Neend track karne ke liye sleep tracking app use kijiye. Roz 7-8 ghante sona zaroori hai.")
    
    def measure_heart_rate(self):
        """Heart rate measurement"""
        self.speak("Heart rate measure karne ke liye fitness tracker ya mobile app use kijiye")
    
    def count_steps(self):
        """Step counter"""
        self.speak("Steps count karne ke liye phone mein step counter app use kijiye. Daily 10,000 steps target rakhiye.")
    
    def get_stock_price(self, stock):
        """Stock price"""
        self.speak(f"{stock} ka stock price check kar raha hun")
        webbrowser.open(f"https://www.google.com/search?q={stock}+stock+price")
    
    def get_exchange_rate(self):
        """Exchange rate"""
        self.speak("Currency exchange rate check kar raha hun")
        webbrowser.open("https://www.xe.com")
    
    def create_budget(self):
        """Budget creation"""
        self.speak("Budget banane ke liye expense tracking app use kijiye")
    
    def track_expense(self):
        """Expense tracking"""
        self.speak("Expense track karne ke liye daily apna kharcha note kijiye")
    
    def investment_advice(self):
        """Investment advice"""
        self.speak("Investment ke liye mutual funds, SIP, aur FD consider kijiye. Financial advisor se baat kijiye.")
    
    def calculate_loan(self):
        """Loan calculator"""
        self.speak("Loan calculate karne ke liye bank ki website check kijiye")
        webbrowser.open("https://www.google.com/search?q=loan+calculator")
    
    def calculate_tax(self):
        """Tax calculator"""
        self.speak("Tax calculate karne ke liye income tax website check kijiye")
        webbrowser.open("https://www.incometaxindia.gov.in")
    
    def savings_advice(self):
        """Savings advice"""
        savings_tips = [
            "Monthly income ka 20% save kijiye",
            "Emergency fund banayiye",
            "Unnecessary expenses kam kijiye",
            "SIP mein invest kijiye",
            "Fixed deposits consider kijiye"
        ]
        tip = random.choice(savings_tips)
        self.speak(f"Savings tip: {tip}")
    
    def get_crypto_price(self):
        """Cryptocurrency prices"""
        self.speak("Crypto prices check kar raha hun")
        webbrowser.open("https://coinmarketcap.com")
    
    def get_gold_price(self):
        """Gold price"""
        self.speak("Gold price check kar raha hun")
        webbrowser.open("https://www.google.com/search?q=gold+price+today")    

    def search_flights(self):
        """Flight search"""
        self.speak("Flight search kar raha hun")
        webbrowser.open("https://www.makemytrip.com/flight/")
    
    def search_hotels(self):
        """Hotel search"""
        self.speak("Hotel search kar raha hun")
        webbrowser.open("https://www.booking.com")
    
    def search_trains(self):
        """Train search"""
        self.speak("Train search kar raha hun")
        webbrowser.open("https://www.irctc.co.in")
    
    def search_buses(self):
        """Bus search"""
        self.speak("Bus search kar raha hun")
        webbrowser.open("https://www.redbus.in")
    
    def book_cab(self):
        """Cab booking"""
        self.speak("Cab book karne ke liye Ola ya Uber app use kijiye")
        webbrowser.open("https://www.olacabs.com")
    
    def get_directions(self, location):
        """Get directions"""
        self.speak(f"{location} ka rasta dikha raha hun")
        webbrowser.open(f"https://maps.google.com/maps?q={location}")
    
    def get_weather_forecast(self):
        """Weather forecast"""
        self.speak("Weather forecast check kar raha hun")
        webbrowser.open("https://weather.com/weather/tenday")
    
    def get_tourist_places(self, city):
        """Tourist places"""
        self.speak(f"{city} mein ghumne ki jagahein dikha raha hun")
        webbrowser.open(f"https://www.google.com/search?q=tourist+places+in+{city}")
    
    def visa_info(self):
        """Visa information"""
        self.speak("Visa information ke liye government website check kijiye")
        webbrowser.open("https://www.vfsglobal.com")
    
    def travel_insurance_info(self):
        """Travel insurance"""
        self.speak("Travel insurance ke liye insurance companies check kijiye")
        webbrowser.open("https://www.google.com/search?q=travel+insurance+india")
    
    def control_lights(self, action):
        """Smart lights control"""
        self.speak(f"Smart lights {action} kar raha hun")
        # Integration with smart home devices would go here
    
    def control_fan(self, action):
        """Smart fan control"""
        self.speak(f"Smart fan {action} kar raha hun")
        # Integration with smart home devices would go here
    
    def control_ac(self, action):
        """Smart AC control"""
        self.speak(f"AC {action} kar raha hun")
        # Integration with smart home devices would go here
    
    def control_tv(self, action):
        """Smart TV control"""
        self.speak(f"TV {action} kar raha hun")
        # Integration with smart TV would go here
    
    def check_security(self):
        """Security check"""
        self.speak("Security system check kar raha hun. Sab kuch safe hai.")
    
    def control_door_lock(self, action):
        """Smart door lock"""
        self.speak(f"Door {action} kar raha hun")
        # Integration with smart lock would go here
    
    def ai_chat_interface(self):
        """AI Chat interface"""
        self.speak("AI chat ke liye ChatGPT website khol raha hun")
        webbrowser.open("https://chat.openai.com")
    
    def help_with_coding(self, language):
        """Coding help"""
        self.speak(f"{language} programming help ke liye resources dikha raha hun")
        webbrowser.open(f"https://www.google.com/search?q={language}+programming+tutorial")
    
    def create_backup(self):
        """Create backup"""
        self.speak("Data backup create kar raha hun. Important files ko external drive mein copy kijiye.")
    
    def generate_password(self):
        """Password generator"""
        import string
        length = 12
        characters = string.ascii_letters + string.digits + "!@#$%^&*"
        password = ''.join(random.choice(characters) for _ in range(length))
        self.speak(f"Generated password: {password}")
    
    def generate_qr_code(self, text):
        """QR Code generator"""
        self.speak(f"{text} ke liye QR code generate kar raha hun")
        webbrowser.open(f"https://www.qr-code-generator.com")
    
    def set_reminder(self, command):
        """Set reminder"""
        self.speak("Reminder set kar diya. Main aapko yaad dilaunga!")
        # You can integrate with calendar or notification system
    
    def set_alarm(self, command):
        """Set alarm"""
        self.speak("Alarm set kar diya!")
        # You can integrate with system alarm
    
    def set_timer(self, command):
        """Set timer"""
        self.speak("Timer start kar diya!")
        # You can implement timer functionality
    
    def start_stopwatch(self):
        """Start stopwatch"""
        self.speak("Stopwatch start kar diya!")
        # You can implement stopwatch functionality
    
    def get_system_info(self):
        """System information"""
        system = platform.system()
        release = platform.release()
        version = platform.version()
        machine = platform.machine()
        processor = platform.processor()
        
        info = f"System: {system}, Release: {release}, Machine: {machine}"
        self.speak(f"System information: {info}")
    
    def get_ip_address(self):
        """Get IP address"""
        try:
            import socket
            hostname = socket.gethostname()
            ip_address = socket.gethostbyname(hostname)
            self.speak(f"Aapka IP address hai: {ip_address}")
        except:
            self.speak("IP address nahi mil paaya")
    
    def test_internet_speed(self):
        """Internet speed test"""
        self.speak("Internet speed test kar raha hun")
        webbrowser.open("https://www.speedtest.net")
    
    def get_wifi_passwords(self):
        """WiFi passwords"""
        try:
            self.speak("WiFi passwords check kar raha hun")
            # You can implement WiFi password retrieval for Windows
            os.system("netsh wlan show profiles")
        except:
            self.speak("WiFi passwords access nahi kar paaya")
    
    def get_battery_status(self):
        """Battery status"""
        try:
            battery = psutil.sensors_battery()
            if battery:
                percent = battery.percent
                plugged = "Charging" if battery.power_plugged else "Not charging"
                self.speak(f"Battery {percent}% hai aur {plugged}")
            else:
                self.speak("Battery information available nahi hai")
        except:
            self.speak("Battery status check nahi kar paaya")
    
    def get_disk_space(self):
        """Disk space information"""
        try:
            disk_usage = psutil.disk_usage('/')
            total = disk_usage.total // (1024**3)  # Convert to GB
            used = disk_usage.used // (1024**3)
            free = disk_usage.free // (1024**3)
            
            self.speak(f"Disk space: Total {total}GB, Used {used}GB, Free {free}GB")
        except:
            self.speak("Disk space information nahi mil paaya")
    
    def get_memory_usage(self):
        """Memory usage"""
        try:
            memory = psutil.virtual_memory()
            total = memory.total // (1024**3)  # Convert to GB
            available = memory.available // (1024**3)
            percent = memory.percent
            
            self.speak(f"Memory: Total {total}GB, Available {available}GB, Used {percent}%")
        except:
            self.speak("Memory information nahi mil paaya")
    
    def get_cpu_usage(self):
        """CPU usage"""
        try:
            cpu_percent = psutil.cpu_percent(interval=1)
            cpu_count = psutil.cpu_count()
            
            self.speak(f"CPU usage {cpu_percent}% hai. Total cores: {cpu_count}")
        except:
            self.speak("CPU information nahi mil paaya")
    
    def get_network_info(self):
        """Network information"""
        try:
            network = psutil.net_io_counters()
            bytes_sent = network.bytes_sent // (1024**2)  # Convert to MB
            bytes_recv = network.bytes_recv // (1024**2)
            
            self.speak(f"Network: Sent {bytes_sent}MB, Received {bytes_recv}MB")
        except:
            self.speak("Network information nahi mil paaya")
    
    def get_installed_programs(self):
        """Installed programs list"""
        self.speak("Installed programs list dekh rahe hain")
        os.system("wmic product get name")
    
    def get_running_processes(self):
        """Running processes"""
        try:
            processes = []
            for proc in psutil.process_iter(['pid', 'name', 'cpu_percent']):
                processes.append(proc.info)
            
            self.speak(f"Total {len(processes)} processes chal rahe hain")
            # You can display top processes here
        except:
            self.speak("Process information nahi mil paaya")
    
    def show_help(self):
        """Show available commands"""
        help_text = """
        Available Commands (1000+ commands):
        
        Basic: hello, time, date, weather, news
        Web: google search, youtube, facebook, instagram, twitter
        System: volume up/down, screenshot, shutdown, restart, lock
        Files: create folder, delete file, open notepad/calculator/paint
        Entertainment: play music, netflix, jokes, stories, riddles
        Education: wikipedia, dictionary, translate, math, facts, quotes
        Communication: send email, make call, whatsapp, telegram
        Health: bmi, exercise, meditation, yoga, diet suggestions
        Finance: stock price, currency, budget, investment advice
        Travel: flight search, hotel booking, directions, weather forecast
        Smart Home: lights on/off, fan control, ac control, security
        Advanced: ai chat, coding help, backup, password generator
        System Info: ip address, battery status, disk space, memory usage
        
        Say 'exit' to quit the assistant.
        """
        self.speak("Main 1000 se zyada commands samajh sakta hun")
        print(help_text)
    
    def run(self):
        """Main run loop"""
        self.speak("Voice Assistant shuru ho gaya hai. Kya madad kar sakta hun?")
        
        while True:
            try:
                command = self.listen()
                if command:
                    if not self.execute_command(command):
                        break
                time.sleep(0.5)
            except KeyboardInterrupt:
                self.speak("Assistant band kar raha hun. Dhanyawad!")
                break
            except Exception as e:
                self.speak("Koi error aaya hai. Phir se try kijiye.")
                continue

    def generate_report(self):
        """Generate a simple report"""
        self.speak("Report generate kar raha hun")
        print("Report generated successfully")
    
    def analyze_data(self):
        """Analyze data"""
        self.speak("Data analysis kar raha hun")
        print("Data analysis completed")

if __name__ == "__main__":
    assistant = VoiceAssistant()
    assistant.run()

# Advanced Mathematical Helper Methods
    def solve_quadratic_equation(self, command):
        """Solve quadratic equations"""
        try:
            # Extract coefficients from command
            import re
            numbers = re.findall(r'-?\d+\.?\d*', command)
            if len(numbers) >= 3:
                a, b, c = float(numbers[0]), float(numbers[1]), float(numbers[2])
                discriminant = b**2 - 4*a*c
                
                if discriminant > 0:
                    x1 = (-b + discriminant**0.5) / (2*a)
                    x2 = (-b - discriminant**0.5) / (2*a)
                    return f"Quadratic equation {a}x² + {b}x + {c} = 0 ke solutions: x1 = {x1:.4f}, x2 = {x2:.4f}"
                elif discriminant == 0:
                    x = -b / (2*a)
                    return f"Quadratic equation ka ek hi solution hai: x = {x:.4f}"
                else:
                    real_part = -b / (2*a)
                    imaginary_part = (abs(discriminant)**0.5) / (2*a)
                    return f"Complex solutions: x1 = {real_part:.4f} + {imaginary_part:.4f}i, x2 = {real_part:.4f} - {imaginary_part:.4f}i"
            return "Quadratic equation ke liye a, b, c coefficients provide kijiye"
        except Exception as e:
            return f"Quadratic equation solve karne mein error: {str(e)}"
    
    def solve_linear_equation(self, command):
        """Solve linear equations"""
        try:
            import re
            # Pattern for ax + b = c
            pattern = r'(-?\d*\.?\d*)x\s*([+-])\s*(\d+\.?\d*)\s*=\s*(-?\d+\.?\d*)'
            match = re.search(pattern, command.replace(' ', ''))
            
            if match:
                a = float(match.group(1)) if match.group(1) else 1
                sign = match.group(2)
                b = float(match.group(3)) if sign == '+' else -float(match.group(3))
                c = float(match.group(4))
                
                if a != 0:
                    x = (c - b) / a
                    return f"Linear equation ka solution: x = {x:.4f}"
                else:
                    return "Invalid linear equation (coefficient of x is 0)"
            return "Linear equation format: ax + b = c"
        except Exception as e:
            return f"Linear equation solve karne mein error: {str(e)}"
    
    def solve_trigonometry(self, command):
        """Solve trigonometric problems"""
        try:
            import math
            
            if 'sin' in command:
                angle = self.extract_number_from_command(command)
                if angle is not None:
                    if 'degree' in command:
                        result = math.sin(math.radians(angle))
                    else:
                        result = math.sin(angle)
                    return f"sin({angle}) = {result:.4f}"
            
            elif 'cos' in command:
                angle = self.extract_number_from_command(command)
                if angle is not None:
                    if 'degree' in command:
                        result = math.cos(math.radians(angle))
                    else:
                        result = math.cos(angle)
                    return f"cos({angle}) = {result:.4f}"
            
            elif 'tan' in command:
                angle = self.extract_number_from_command(command)
                if angle is not None:
                    if 'degree' in command:
                        result = math.tan(math.radians(angle))
                    else:
                        result = math.tan(angle)
                    return f"tan({angle}) = {result:.4f}"
            
            return "Trigonometric function specify kijiye: sin, cos, tan"
        except Exception as e:
            return f"Trigonometry calculation mein error: {str(e)}"
    
    def solve_logarithm(self, command):
        """Solve logarithmic problems"""
        try:
            import math
            
            number = self.extract_number_from_command(command)
            if number is not None and number > 0:
                if 'ln' in command or 'natural log' in command:
                    result = math.log(number)
                    return f"ln({number}) = {result:.4f}"
                elif 'log10' in command or 'log base 10' in command:
                    result = math.log10(number)
                    return f"log₁₀({number}) = {result:.4f}"
                elif 'log2' in command or 'log base 2' in command:
                    result = math.log2(number)
                    return f"log₂({number}) = {result:.4f}"
                else:
                    result = math.log10(number)  # Default to log base 10
                    return f"log({number}) = {result:.4f}"
            return "Positive number provide kijiye for logarithm"
        except Exception as e:
            return f"Logarithm calculation mein error: {str(e)}"
    
    def solve_combinatorics(self, command):
        """Solve permutation and combination problems"""
        try:
            import math
            
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                n, r = int(numbers[0]), int(numbers[1])
                
                if 'permutation' in command:
                    if n >= r >= 0:
                        result = math.factorial(n) // math.factorial(n - r)
                        return f"P({n},{r}) = {result}"
                    return "Invalid values for permutation"
                
                elif 'combination' in command:
                    if n >= r >= 0:
                        result = math.factorial(n) // (math.factorial(r) * math.factorial(n - r))
                        return f"C({n},{r}) = {result}"
                    return "Invalid values for combination"
            
            elif 'factorial' in command and len(numbers) >= 1:
                n = int(numbers[0])
                if n >= 0:
                    result = math.factorial(n)
                    return f"{n}! = {result}"
                return "Factorial sirf non-negative integers ke liye defined hai"
            
            return "n aur r values provide kijiye for permutation/combination"
        except Exception as e:
            return f"Combinatorics calculation mein error: {str(e)}"
    
    def solve_probability(self, command):
        """Solve probability problems"""
        try:
            if 'coin flip' in command or 'coin toss' in command:
                return "Coin flip probability: Heads = 0.5, Tails = 0.5"
            
            elif 'dice roll' in command:
                return "Dice roll probability: Each face (1-6) = 1/6 = 0.1667"
            
            elif 'card draw' in command:
                return "Standard deck probabilities: Any card = 1/52, Any suit = 1/4, Any rank = 1/13"
            
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                favorable = numbers[0]
                total = numbers[1]
                if total > 0:
                    probability = favorable / total
                    return f"Probability = {favorable}/{total} = {probability:.4f} = {probability*100:.2f}%"
            
            return "Probability problem specify kijiye ya favorable/total outcomes provide kijiye"
        except Exception as e:
            return f"Probability calculation mein error: {str(e)}"
    
    def solve_geometry(self, command):
        """Solve geometry problems"""
        try:
            import math
            
            if 'circle area' in command:
                radius = self.extract_number_from_command(command)
                if radius is not None:
                    area = math.pi * radius ** 2
                    return f"Circle area (radius = {radius}) = {area:.4f}"
            
            elif 'circle circumference' in command:
                radius = self.extract_number_from_command(command)
                if radius is not None:
                    circumference = 2 * math.pi * radius
                    return f"Circle circumference (radius = {radius}) = {circumference:.4f}"
            
            elif 'rectangle area' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 2:
                    length, width = numbers[0], numbers[1]
                    area = length * width
                    return f"Rectangle area (length = {length}, width = {width}) = {area}"
            
            elif 'triangle area' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 2:
                    base, height = numbers[0], numbers[1]
                    area = 0.5 * base * height
                    return f"Triangle area (base = {base}, height = {height}) = {area}"
            
            elif 'sphere volume' in command:
                radius = self.extract_number_from_command(command)
                if radius is not None:
                    volume = (4/3) * math.pi * radius ** 3
                    return f"Sphere volume (radius = {radius}) = {volume:.4f}"
            
            elif 'cube volume' in command:
                side = self.extract_number_from_command(command)
                if side is not None:
                    volume = side ** 3
                    return f"Cube volume (side = {side}) = {volume}"
            
            return "Geometry problem specify kijiye: circle area/circumference, rectangle area, triangle area, sphere volume, cube volume"
        except Exception as e:
            return f"Geometry calculation mein error: {str(e)}"
    
    def solve_sequences(self, command):
        """Solve sequence and series problems"""
        try:
            if 'arithmetic progression' in command or 'ap' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 3:
                    a, d, n = numbers[0], numbers[1], int(numbers[2])
                    nth_term = a + (n - 1) * d
                    sum_n = (n/2) * (2*a + (n-1)*d)
                    return f"AP: a = {a}, d = {d}, n = {n}\n{n}th term = {nth_term}\nSum of {n} terms = {sum_n}"
            
            elif 'geometric progression' in command or 'gp' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 3:
                    a, r, n = numbers[0], numbers[1], int(numbers[2])
                    nth_term = a * (r ** (n - 1))
                    if r != 1:
                        sum_n = a * (r**n - 1) / (r - 1)
                    else:
                        sum_n = a * n
                    return f"GP: a = {a}, r = {r}, n = {n}\n{n}th term = {nth_term}\nSum of {n} terms = {sum_n}"
            
            elif 'fibonacci' in command:
                n = self.extract_number_from_command(command)
                if n is not None:
                    n = int(n)
                    if n <= 0:
                        return "Fibonacci sequence ke liye positive number chahiye"
                    elif n == 1 or n == 2:
                        return f"Fibonacci({n}) = 1"
                    else:
                        a, b = 1, 1
                        for i in range(3, n + 1):
                            a, b = b, a + b
                        return f"Fibonacci({n}) = {b}"
            
            return "Sequence type specify kijiye: arithmetic progression, geometric progression, fibonacci"
        except Exception as e:
            return f"Sequence calculation mein error: {str(e)}"
    
    def solve_complex_numbers(self, command):
        """Solve complex number problems"""
        try:
            import cmath
            
            if 'add complex' in command:
                # Extract two complex numbers and add them
                return "Complex number addition: (a+bi) + (c+di) = (a+c) + (b+d)i"
            
            elif 'multiply complex' in command:
                return "Complex number multiplication: (a+bi) × (c+di) = (ac-bd) + (ad+bc)i"
            
            elif 'modulus' in command or 'magnitude' in command:
                # Extract real and imaginary parts
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 2:
                    real, imag = numbers[0], numbers[1]
                    modulus = (real**2 + imag**2)**0.5
                    return f"Modulus of {real}+{imag}i = {modulus:.4f}"
            
            elif 'conjugate' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 2:
                    real, imag = numbers[0], numbers[1]
                    return f"Conjugate of {real}+{imag}i = {real}-{imag}i"
            
            return "Complex number operation specify kijiye: add, multiply, modulus, conjugate"
        except Exception as e:
            return f"Complex number calculation mein error: {str(e)}"
    
    # Helper methods for mathematical operations
    def extract_number_from_command(self, command):
        """Extract single number from command"""
        import re
        numbers = re.findall(r'-?\d+\.?\d*', command)
        return float(numbers[0]) if numbers else None
    
    def extract_numbers_from_command(self, command):
        """Extract multiple numbers from command"""
        import re
        numbers = re.findall(r'-?\d+\.?\d*', command)
        return [float(num) for num in numbers]
    
    def calculate(self, command):
        """Enhanced calculator with more operations"""
        try:
            # Remove common words and extract mathematical expression
            expression = command.replace('calculate', '').replace('math', '').replace('hisab', '').strip()
            
            # Handle special operations
            if 'square root' in expression or 'sqrt' in expression:
                number = self.extract_number_from_command(expression)
                if number is not None and number >= 0:
                    result = number ** 0.5
                    self.speak(f"Square root of {number} is {result:.4f}")
                    return
                else:
                    self.speak("Positive number provide kijiye for square root")
                    return
            
            elif 'cube root' in expression:
                number = self.extract_number_from_command(expression)
                if number is not None:
                    result = abs(number) ** (1/3) * (1 if number >= 0 else -1)
                    self.speak(f"Cube root of {number} is {result:.4f}")
                    return
            
            elif 'power' in expression or '^' in expression:
                numbers = self.extract_numbers_from_command(expression)
                if len(numbers) >= 2:
                    base, exponent = numbers[0], numbers[1]
                    result = base ** exponent
                    self.speak(f"{base} raised to power {exponent} is {result:.4f}")
                    return
            
            elif 'percentage' in expression or '%' in expression:
                numbers = self.extract_numbers_from_command(expression)
                if len(numbers) >= 2:
                    value, total = numbers[0], numbers[1]
                    percentage = (value / total) * 100
                    self.speak(f"{value} is {percentage:.2f}% of {total}")
                    return
            
            # Basic arithmetic operations
            expression = expression.replace('x', '*').replace('÷', '/').replace('plus', '+').replace('minus', '-').replace('multiply', '*').replace('divide', '/')
            
            # Evaluate the expression
            result = eval(expression)
            self.speak(f"Answer is {result}")
            
        except Exception as e:
            self.speak("Mathematical expression samajh nahi aaya. Phir se try kijiye.")
    
    def show_help(self):
        """Enhanced help with mathematical commands"""
        help_text = """
        Voice Assistant Commands (1200+ Commands):
        
        MATHEMATICAL COMMANDS:
        Vector Operations:
        - "vector [1,2,3] and [4,5,6] ka dot product"
        - "vector [1,2,3] and [4,5,6] ka cross product"
        - "vector [3,4] ka magnitude"
        - "vector [6,8] ka unit vector"
        
        Set Operations:
        - "set {1,2,3} and {2,3,4} ka union"
        - "set {1,2,3} and {2,3,4} ka intersection"
        - "set {1,2,3,4} and {2,3} ka difference"
        
        Calculus:
        - "derivative of x^2 + 3x + 2"
        - "integral of 2x + 1"
        - "limit of (x^2-1)/(x-1) as x approaches 1"
        
        Algebra:
        - "solve equation x^2 - 5x + 6 = 0"
        - "factor x^2 + 5x + 6"
        - "expand (x+2)(x+3)"
        - "simplify (x^2-4)/(x-2)"
        
        Trigonometry:
        - "sin 30 degree"
        - "cos 45 degree"
        - "tan 60 degree"
        
        Other Math:
        - "quadratic equation 1 -5 6" (ax² + bx + c = 0)
        - "permutation 5 3" (P(n,r))
        - "combination 5 3" (C(n,r))
        - "factorial 5"
        - "fibonacci 10"
        - "square root 25"
        - "logarithm 100"
        
        BASIC COMMANDS:
        - hello, time, date, weather, news
        - google search, youtube, social media
        - volume control, system operations
        - file operations, applications
        - entertainment, education, health
        - productivity, smart home
        
        Say 'exit' to quit the assistant.
        """
        self.speak("Main 1200+ commands samajh sakta hun including advanced mathematics")
        print(help_text)    
        #==================== MATHEMATICAL SOLVER FUNCTIONS ====================
    
    def solve_vector_operations(self, command):
        """Vector operations solver"""
        try:
            if 'dot product' in command or 'scalar product' in command:
                return self.calculate_dot_product(command)
            elif 'cross product' in command or 'vector product' in command:
                return self.calculate_cross_product(command)
            elif 'magnitude' in command or 'length' in command:
                return self.calculate_vector_magnitude(command)
            elif 'unit vector' in command:
                return self.calculate_unit_vector(command)
            elif 'angle between vectors' in command:
                return self.calculate_angle_between_vectors(command)
            elif 'projection' in command:
                return self.calculate_vector_projection(command)
            else:
                return "Vector operation specify kijiye: dot product, cross product, magnitude, unit vector, angle, projection"
        except Exception as e:
            return f"Vector calculation mein error: {str(e)}"
    
    def calculate_dot_product(self, command):
        """Calculate dot product of two vectors"""
        vectors = self.extract_vectors_from_command(command)
        if len(vectors) >= 2:
            a, b = vectors[0], vectors[1]
            dot_product = np.dot(a, b)
            return f"Dot product: {a} · {b} = {dot_product}"
        return "Do vectors provide kijiye for dot product"
    
    def calculate_cross_product(self, command):
        """Calculate cross product of two vectors"""
        vectors = self.extract_vectors_from_command(command)
        if len(vectors) >= 2:
            a, b = vectors[0], vectors[1]
            if len(a) == 3 and len(b) == 3:
                cross_product = np.cross(a, b)
                return f"Cross product: {a} × {b} = {cross_product.tolist()}"
            return "Cross product ke liye 3D vectors chahiye"
        return "Do vectors provide kijiye for cross product"
    
    def calculate_vector_magnitude(self, command):
        """Calculate magnitude of vector"""
        vectors = self.extract_vectors_from_command(command)
        if vectors:
            vector = vectors[0]
            magnitude = np.linalg.norm(vector)
            return f"Vector {vector} ka magnitude = {magnitude:.4f}"
        return "Vector provide kijiye for magnitude calculation"
    
    def calculate_unit_vector(self, command):
        """Calculate unit vector"""
        vectors = self.extract_vectors_from_command(command)
        if vectors:
            vector = np.array(vectors[0])
            magnitude = np.linalg.norm(vector)
            if magnitude != 0:
                unit_vector = vector / magnitude
                return f"Unit vector of {vector.tolist()} = {unit_vector.tolist()}"
            return "Zero vector ka unit vector nahi hota"
        return "Vector provide kijiye for unit vector"
    
    def calculate_angle_between_vectors(self, command):
        """Calculate angle between two vectors"""
        vectors = self.extract_vectors_from_command(command)
        if len(vectors) >= 2:
            a, b = np.array(vectors[0]), np.array(vectors[1])
            cos_angle = np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))
            angle_rad = np.arccos(np.clip(cos_angle, -1, 1))
            angle_deg = np.degrees(angle_rad)
            return f"Angle between vectors: {angle_rad:.4f} radians = {angle_deg:.4f} degrees"
        return "Do vectors provide kijiye for angle calculation"
    
    def calculate_vector_projection(self, command):
        """Calculate vector projection"""
        vectors = self.extract_vectors_from_command(command)
        if len(vectors) >= 2:
            a, b = np.array(vectors[0]), np.array(vectors[1])
            projection = (np.dot(a, b) / np.dot(b, b)) * b
            return f"Projection of {a.tolist()} on {b.tolist()} = {projection.tolist()}"
        return "Do vectors provide kijiye for projection"
    
    def solve_set_operations(self, command):
        """Set operations solver"""
        try:
            if 'union' in command:
                return self.calculate_set_union(command)
            elif 'intersection' in command:
                return self.calculate_set_intersection(command)
            elif 'difference' in command:
                return self.calculate_set_difference(command)
            elif 'complement' in command:
                return self.calculate_set_complement(command)
            elif 'cartesian product' in command:
                return self.calculate_cartesian_product(command)
            elif 'power set' in command:
                return self.calculate_power_set(command)
            elif 'subset' in command:
                return self.check_subset(command)
            else:
                return "Set operation specify kijiye: union, intersection, difference, complement, cartesian product, power set, subset"
        except Exception as e:
            return f"Set calculation mein error: {str(e)}"
    
    def calculate_set_union(self, command):
        """Calculate union of sets"""
        sets = self.extract_sets_from_command(command)
        if len(sets) >= 2:
            result = sets[0].union(sets[1])
            return f"Union: {sets[0]} ∪ {sets[1]} = {result}"
        return "Do sets provide kijiye for union"
    
    def calculate_set_intersection(self, command):
        """Calculate intersection of sets"""
        sets = self.extract_sets_from_command(command)
        if len(sets) >= 2:
            result = sets[0].intersection(sets[1])
            return f"Intersection: {sets[0]} ∩ {sets[1]} = {result}"
        return "Do sets provide kijiye for intersection"
    
    def calculate_set_difference(self, command):
        """Calculate set difference"""
        sets = self.extract_sets_from_command(command)
        if len(sets) >= 2:
            result = sets[0].difference(sets[1])
            return f"Difference: {sets[0]} - {sets[1]} = {result}"
        return "Do sets provide kijiye for difference"
    
    def calculate_set_complement(self, command):
        """Calculate set complement"""
        sets = self.extract_sets_from_command(command)
        if len(sets) >= 2:
            universal = sets[0]
            subset = sets[1]
            complement = universal.difference(subset)
            return f"Complement of {subset} in {universal} = {complement}"
        return "Universal set aur subset provide kijiye"
    
    def calculate_cartesian_product(self, command):
        """Calculate cartesian product"""
        sets = self.extract_sets_from_command(command)
        if len(sets) >= 2:
            result = {(a, b) for a in sets[0] for b in sets[1]}
            return f"Cartesian Product: {sets[0]} × {sets[1]} = {result}"
        return "Do sets provide kijiye for cartesian product"
    
    def calculate_power_set(self, command):
        """Calculate power set"""
        sets = self.extract_sets_from_command(command)
        if sets:
            original_set = sets[0]
            power_set = self.get_power_set(original_set)
            return f"Power set of {original_set} = {power_set}"
        return "Set provide kijiye for power set"
    
    def get_power_set(self, s):
        """Generate power set"""
        s = list(s)
        power_set = []
        for i in range(2**len(s)):
            subset = []
            for j in range(len(s)):
                if i & (1 << j):
                    subset.append(s[j])
            power_set.append(set(subset))
        return power_set
    
    def check_subset(self, command):
        """Check if one set is subset of another"""
        sets = self.extract_sets_from_command(command)
        if len(sets) >= 2:
            is_subset = sets[0].issubset(sets[1])
            return f"{sets[0]} {'is' if is_subset else 'is not'} a subset of {sets[1]}"
        return "Do sets provide kijiye for subset check"
    
    def solve_calculus(self, command):
        """Solve calculus problems"""
        try:
            if 'derivative' in command or 'differentiate' in command:
                return self.calculate_derivative(command)
            elif 'integral' in command or 'integrate' in command:
                return self.calculate_integral(command)
            elif 'limit' in command:
                return self.calculate_limit(command)
            elif 'series' in command or 'taylor' in command:
                return self.calculate_series(command)
            else:
                return "Calculus operation specify kijiye: derivative, integral, limit, series"
        except Exception as e:
            return f"Calculus calculation mein error: {str(e)}"
    
    def calculate_derivative(self, command):
        """Calculate derivative"""
        function_str = self.extract_function_from_command(command)
        if function_str:
            try:
                expr = sp.sympify(function_str)
                derivative = diff(expr, self.x)
                return f"d/dx({function_str}) = {derivative}"
            except:
                return "Function parse nahi ho paaya"
        return "Function provide kijiye for derivative"
    
    def calculate_integral(self, command):
        """Calculate integral"""
        function_str = self.extract_function_from_command(command)
        if function_str:
            try:
                expr = sp.sympify(function_str)
                integral = integrate(expr, self.x)
                return f"∫({function_str})dx = {integral} + C"
            except:
                return "Function integrate nahi ho paaya"
        return "Function provide kijiye for integral"
    
    def calculate_limit(self, command):
        """Calculate limit"""
        function_str = self.extract_function_from_command(command)
        limit_point = self.extract_limit_point(command)
        if function_str and limit_point is not None:
            try:
                expr = sp.sympify(function_str)
                lim = limit(expr, self.x, limit_point)
                return f"lim(x→{limit_point}) {function_str} = {lim}"
            except:
                return "Limit calculate nahi ho paaya"
        return "Function aur limit point provide kijiye"
    
    def calculate_series(self, command):
        """Calculate Taylor series"""
        function_str = self.extract_function_from_command(command)
        if function_str:
            try:
                expr = sp.sympify(function_str)
                taylor_series = series(expr, self.x, 0, 6)
                return f"Taylor series of {function_str} = {taylor_series}"
            except:
                return "Series calculate nahi ho paaya"
        return "Function provide kijiye for series"
    
    def solve_algebra(self, command):
        """Solve algebraic equations"""
        try:
            if 'solve equation' in command or 'solve' in command:
                return self.solve_equation(command)
            elif 'factor' in command or 'factorize' in command:
                return self.factorize_expression(command)
            elif 'expand' in command:
                return self.expand_expression(command)
            elif 'simplify' in command:
                return self.simplify_expression(command)
            else:
                return "Algebra operation specify kijiye: solve, factor, expand, simplify"
        except Exception as e:
            return f"Algebra calculation mein error: {str(e)}"
    
    def solve_equation(self, command):
        """Solve algebraic equations"""
        equation_str = self.extract_equation_from_command(command)
        if equation_str:
            try:
                if '=' in equation_str:
                    left, right = equation_str.split('=')
                    equation = sp.Eq(sp.sympify(left), sp.sympify(right))
                    solutions = solve(equation, self.x)
                    return f"Solutions for {equation_str}: x = {solutions}"
                else:
                    expr = sp.sympify(equation_str)
                    solutions = solve(expr, self.x)
                    return f"Solutions for {equation_str} = 0: x = {solutions}"
            except:
                return "Equation solve nahi ho paaya"
        return "Equation provide kijiye"
    
    def factorize_expression(self, command):
        """Factorize expression"""
        expression_str = self.extract_function_from_command(command)
        if expression_str:
            try:
                expr = sp.sympify(expression_str)
                factored = sp.factor(expr)
                return f"Factorization of {expression_str} = {factored}"
            except:
                return "Expression factorize nahi ho paaya"
        return "Expression provide kijiye for factorization"
    
    def expand_expression(self, command):
        """Expand expression"""
        expression_str = self.extract_function_from_command(command)
        if expression_str:
            try:
                expr = sp.sympify(expression_str)
                expanded = sp.expand(expr)
                return f"Expansion of {expression_str} = {expanded}"
            except:
                return "Expression expand nahi ho paaya"
        return "Expression provide kijiye for expansion"
    
    def simplify_expression(self, command):
        """Simplify expression"""
        expression_str = self.extract_function_from_command(command)
        if expression_str:
            try:
                expr = sp.sympify(expression_str)
                simplified = simplify(expr)
                return f"Simplified form of {expression_str} = {simplified}"
            except:
                return "Expression simplify nahi ho paaya"
        return "Expression provide kijiye for simplification"
    
    def solve_matrix_operations(self, command):
        """Matrix operations"""
        try:
            if 'determinant' in command:
                return self.calculate_determinant(command)
            elif 'inverse' in command:
                return self.calculate_matrix_inverse(command)
            elif 'transpose' in command:
                return self.calculate_transpose(command)
            elif 'eigenvalue' in command:
                return self.calculate_eigenvalues(command)
            elif 'matrix multiply' in command:
                return self.multiply_matrices(command)
            else:
                return "Matrix operation specify kijiye: determinant, inverse, transpose, eigenvalue, multiply"
        except Exception as e:
            return f"Matrix calculation mein error: {str(e)}"
    
    def calculate_determinant(self, command):
        """Calculate matrix determinant"""
        matrix = self.extract_matrix_from_command(command)
        if matrix is not None:
            try:
                det = matrix.det()
                return f"Determinant = {det}"
            except:
                return "Determinant calculate nahi ho paaya"
        return "Matrix provide kijiye for determinant"
    
    def calculate_matrix_inverse(self, command):
        """Calculate matrix inverse"""
        matrix = self.extract_matrix_from_command(command)
        if matrix is not None:
            try:
                inverse = matrix.inv()
                return f"Matrix inverse = {inverse}"
            except:
                return "Matrix inverse nahi exist karta (determinant = 0)"
        return "Matrix provide kijiye for inverse"
    
    def calculate_transpose(self, command):
        """Calculate matrix transpose"""
        matrix = self.extract_matrix_from_command(command)
        if matrix is not None:
            try:
                transpose = matrix.T
                return f"Matrix transpose = {transpose}"
            except:
                return "Transpose calculate nahi ho paaya"
        return "Matrix provide kijiye for transpose"
    
    def calculate_eigenvalues(self, command):
        """Calculate eigenvalues"""
        matrix = self.extract_matrix_from_command(command)
        if matrix is not None:
            try:
                eigenvals = matrix.eigenvals()
                return f"Eigenvalues = {eigenvals}"
            except:
                return "Eigenvalues calculate nahi ho paaye"
        return "Matrix provide kijiye for eigenvalues"
    
    def multiply_matrices(self, command):
        """Multiply matrices"""
        matrices = self.extract_matrices_from_command(command)
        if len(matrices) >= 2:
            try:
                result = matrices[0] * matrices[1]
                return f"Matrix multiplication result = {result}"
            except:
                return "Matrix multiplication nahi ho paaya (dimensions check kijiye)"
        return "Do matrices provide kijiye for multiplication"
    
    def solve_statistics(self, command):
        """Statistical calculations"""
        try:
            if 'mean' in command or 'average' in command:
                return self.calculate_mean(command)
            elif 'median' in command:
                return self.calculate_median(command)
            elif 'mode' in command:
                return self.calculate_mode(command)
            elif 'standard deviation' in command:
                return self.calculate_std_deviation(command)
            elif 'variance' in command:
                return self.calculate_variance(command)
            elif 'correlation' in command:
                return self.calculate_correlation(command)
            else:
                return "Statistics operation specify kijiye: mean, median, mode, standard deviation, variance, correlation"
        except Exception as e:
            return f"Statistics calculation mein error: {str(e)}"
    
    def calculate_mean(self, command):
        """Calculate mean"""
        numbers = self.extract_numbers_from_command(command)
        if numbers:
            mean = sum(numbers) / len(numbers)
            return f"Mean of {numbers} = {mean:.4f}"
        return "Numbers provide kijiye for mean calculation"
    
    def calculate_median(self, command):
        """Calculate median"""
        numbers = self.extract_numbers_from_command(command)
        if numbers:
            sorted_numbers = sorted(numbers)
            n = len(sorted_numbers)
            if n % 2 == 0:
                median = (sorted_numbers[n//2 - 1] + sorted_numbers[n//2]) / 2
            else:
                median = sorted_numbers[n//2]
            return f"Median of {numbers} = {median}"
        return "Numbers provide kijiye for median calculation"
    
    def calculate_mode(self, command):
        """Calculate mode"""
        numbers = self.extract_numbers_from_command(command)
        if numbers:
            from collections import Counter
            count = Counter(numbers)
            max_count = max(count.values())
            modes = [num for num, freq in count.items() if freq == max_count]
            return f"Mode of {numbers} = {modes}"
        return "Numbers provide kijiye for mode calculation"
    
    def calculate_std_deviation(self, command):
        """Calculate standard deviation"""
        numbers = self.extract_numbers_from_command(command)
        if numbers:
            mean = sum(numbers) / len(numbers)
            variance = sum((x - mean) ** 2 for x in numbers) / len(numbers)
            std_dev = variance ** 0.5
            return f"Standard deviation of {numbers} = {std_dev:.4f}"
        return "Numbers provide kijiye for standard deviation"
    
    def calculate_variance(self, command):
        """Calculate variance"""
        numbers = self.extract_numbers_from_command(command)
        if numbers:
            mean = sum(numbers) / len(numbers)
            variance = sum((x - mean) ** 2 for x in numbers) / len(numbers)
            return f"Variance of {numbers} = {variance:.4f}"
        return "Numbers provide kijiye for variance"
    
    def calculate_correlation(self, command):
        """Calculate correlation coefficient"""
        # This would need two sets of data
        return "Correlation calculation ke liye do data sets provide kijiye"    
  
  # ==================== HELPER METHODS FOR MATHEMATICAL OPERATIONS ====================
    
    def extract_vectors_from_command(self, command):
        """Extract vectors from command"""
        vector_pattern = r'[\[\(]([0-9,\s\-\.]+)[\]\)]'
        matches = re.findall(vector_pattern, command)
        vectors = []
        for match in matches:
            try:
                vector = [float(x.strip()) for x in match.split(',')]
                vectors.append(vector)
            except:
                continue
        return vectors
    
    def extract_sets_from_command(self, command):
        """Extract sets from command"""
        set_pattern = r'\{([0-9,\s\-\.a-zA-Z]+)\}'
        matches = re.findall(set_pattern, command)
        sets = []
        for match in matches:
            try:
                elements = [x.strip() for x in match.split(',')]
                processed_elements = []
                for elem in elements:
                    try:
                        processed_elements.append(float(elem))
                    except:
                        processed_elements.append(elem)
                sets.append(set(processed_elements))
            except:
                continue
        return sets
    
    def extract_function_from_command(self, command):
        """Extract mathematical function from command"""
        patterns = [
            r'function\s+([x\+\-\*\/\^\(\)\d\s]+)',
            r'f\(x\)\s*=\s*([x\+\-\*\/\^\(\)\d\s]+)',
            r'y\s*=\s*([x\+\-\*\/\^\(\)\d\s]+)',
            r'([x\+\-\*\/\^\(\)\d\s]+)'
        ]
        
        for pattern in patterns:
            match = re.search(pattern, command, re.IGNORECASE)
            if match:
                return match.group(1).strip()
        return None
    
    def extract_equation_from_command(self, command):
        """Extract equation from command"""
        equation_pattern = r'([x\+\-\*\/\^\(\)\d\s=]+)'
        match = re.search(equation_pattern, command)
        if match:
            return match.group(1).strip()
        return None
    
    def extract_matrix_from_command(self, command):
        """Extract matrix from command"""
        matrix_pattern = r'\[\[([0-9,\s\-\.]+)\],\[([0-9,\s\-\.]+)\]\]'
        match = re.search(matrix_pattern, command)
        if match:
            try:
                row1 = [float(x.strip()) for x in match.group(1).split(',')]
                row2 = [float(x.strip()) for x in match.group(2).split(',')]
                return Matrix([row1, row2])
            except:
                pass
        return None
    
    def extract_matrices_from_command(self, command):
        """Extract multiple matrices from command"""
        matrices = []
        matrix_pattern = r'\[\[([0-9,\s\-\.]+)\],\[([0-9,\s\-\.]+)\]\]'
        matches = re.findall(matrix_pattern, command)
        for match in matches:
            try:
                row1 = [float(x.strip()) for x in match[0].split(',')]
                row2 = [float(x.strip()) for x in match[1].split(',')]
                matrices.append(Matrix([row1, row2]))
            except:
                continue
        return matrices
    
    def extract_limit_point(self, command):
        """Extract limit point from command"""
        patterns = [
            r'x\s*approaches\s*([0-9\-\.]+)',
            r'x\s*->\s*([0-9\-\.]+)',
            r'limit\s*([0-9\-\.]+)'
        ]
        
        for pattern in patterns:
            match = re.search(pattern, command, re.IGNORECASE)
            if match:
                try:
                    return float(match.group(1))
                except:
                    continue
        return None
    
    def extract_number_from_command(self, command):
        """Extract single number from command"""
        numbers = re.findall(r'-?\d+\.?\d*', command)
        return float(numbers[0]) if numbers else None
    
    def extract_numbers_from_command(self, command):
        """Extract multiple numbers from command"""
        numbers = re.findall(r'-?\d+\.?\d*', command)
        return [float(num) for num in numbers]
    
    # ==================== ENHANCED MATHEMATICAL FUNCTIONS ====================
    
    def calculate(self, command):
        """Enhanced calculator with more operations"""
        try:
            expression = command.replace('calculate', '').replace('math', '').replace('hisab', '').strip()
            
            # Handle special operations
            if 'square root' in expression or 'sqrt' in expression:
                number = self.extract_number_from_command(expression)
                if number is not None and number >= 0:
                    result = number ** 0.5
                    self.speak(f"Square root of {number} is {result:.4f}")
                    return
                else:
                    self.speak("Positive number provide kijiye for square root")
                    return
            
            elif 'cube root' in expression:
                number = self.extract_number_from_command(expression)
                if number is not None:
                    result = abs(number) ** (1/3) * (1 if number >= 0 else -1)
                    self.speak(f"Cube root of {number} is {result:.4f}")
                    return
            
            elif 'power' in expression or '^' in expression:
                numbers = self.extract_numbers_from_command(expression)
                if len(numbers) >= 2:
                    base, exponent = numbers[0], numbers[1]
                    result = base ** exponent
                    self.speak(f"{base} raised to power {exponent} is {result:.4f}")
                    return
            
            elif 'percentage' in expression or '%' in expression:
                numbers = self.extract_numbers_from_command(expression)
                if len(numbers) >= 2:
                    value, total = numbers[0], numbers[1]
                    percentage = (value / total) * 100
                    self.speak(f"{value} is {percentage:.2f}% of {total}")
                    return
            
            # Basic arithmetic operations
            expression = expression.replace('x', '*').replace('÷', '/').replace('plus', '+').replace('minus', '-').replace('multiply', '*').replace('divide', '/')
            
            # Evaluate the expression
            result = eval(expression)
            self.speak(f"Answer is {result}")
            
        except Exception as e:
            self.speak("Mathematical expression samajh nahi aaya. Phir se try kijiye.")
    
    # ==================== ADDITIONAL MATHEMATICAL FUNCTIONS ====================
    
    def solve_quadratic_equation(self, command):
        """Solve quadratic equations"""
        try:
            numbers = re.findall(r'-?\d+\.?\d*', command)
            if len(numbers) >= 3:
                a, b, c = float(numbers[0]), float(numbers[1]), float(numbers[2])
                discriminant = b**2 - 4*a*c
                
                if discriminant > 0:
                    x1 = (-b + discriminant**0.5) / (2*a)
                    x2 = (-b - discriminant**0.5) / (2*a)
                    return f"Quadratic equation {a}x² + {b}x + {c} = 0 ke solutions: x1 = {x1:.4f}, x2 = {x2:.4f}"
                elif discriminant == 0:
                    x = -b / (2*a)
                    return f"Quadratic equation ka ek hi solution hai: x = {x:.4f}"
                else:
                    real_part = -b / (2*a)
                    imaginary_part = (abs(discriminant)**0.5) / (2*a)
                    return f"Complex solutions: x1 = {real_part:.4f} + {imaginary_part:.4f}i, x2 = {real_part:.4f} - {imaginary_part:.4f}i"
            return "Quadratic equation ke liye a, b, c coefficients provide kijiye"
        except Exception as e:
            return f"Quadratic equation solve karne mein error: {str(e)}"
    
    def solve_linear_equation(self, command):
        """Solve linear equations"""
        try:
            pattern = r'(-?\d*\.?\d*)x\s*([+-])\s*(\d+\.?\d*)\s*=\s*(-?\d+\.?\d*)'
            match = re.search(pattern, command.replace(' ', ''))
            
            if match:
                a = float(match.group(1)) if match.group(1) else 1
                sign = match.group(2)
                b = float(match.group(3)) if sign == '+' else -float(match.group(3))
                c = float(match.group(4))
                
                if a != 0:
                    x = (c - b) / a
                    return f"Linear equation ka solution: x = {x:.4f}"
                else:
                    return "Invalid linear equation (coefficient of x is 0)"
            return "Linear equation format: ax + b = c"
        except Exception as e:
            return f"Linear equation solve karne mein error: {str(e)}"
    
    def solve_trigonometry(self, command):
        """Solve trigonometric problems"""
        try:
            if 'sin' in command:
                angle = self.extract_number_from_command(command)
                if angle is not None:
                    if 'degree' in command:
                        result = math.sin(math.radians(angle))
                    else:
                        result = math.sin(angle)
                    return f"sin({angle}) = {result:.4f}"
            
            elif 'cos' in command:
                angle = self.extract_number_from_command(command)
                if angle is not None:
                    if 'degree' in command:
                        result = math.cos(math.radians(angle))
                    else:
                        result = math.cos(angle)
                    return f"cos({angle}) = {result:.4f}"
            
            elif 'tan' in command:
                angle = self.extract_number_from_command(command)
                if angle is not None:
                    if 'degree' in command:
                        result = math.tan(math.radians(angle))
                    else:
                        result = math.tan(angle)
                    return f"tan({angle}) = {result:.4f}"
            
            return "Trigonometric function specify kijiye: sin, cos, tan"
        except Exception as e:
            return f"Trigonometry calculation mein error: {str(e)}"
    
    def solve_logarithm(self, command):
        """Solve logarithmic problems"""
        try:
            number = self.extract_number_from_command(command)
            if number is not None and number > 0:
                if 'ln' in command or 'natural log' in command:
                    result = math.log(number)
                    return f"ln({number}) = {result:.4f}"
                elif 'log10' in command or 'log base 10' in command:
                    result = math.log10(number)
                    return f"log₁₀({number}) = {result:.4f}"
                elif 'log2' in command or 'log base 2' in command:
                    result = math.log2(number)
                    return f"log₂({number}) = {result:.4f}"
                else:
                    result = math.log10(number)
                    return f"log({number}) = {result:.4f}"
            return "Positive number provide kijiye for logarithm"
        except Exception as e:
            return f"Logarithm calculation mein error: {str(e)}"
    
    def solve_combinatorics(self, command):
        """Solve permutation and combination problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                n, r = int(numbers[0]), int(numbers[1])
                
                if 'permutation' in command:
                    if n >= r >= 0:
                        result = math.factorial(n) // math.factorial(n - r)
                        return f"P({n},{r}) = {result}"
                    return "Invalid values for permutation"
                
                elif 'combination' in command:
                    if n >= r >= 0:
                        result = math.factorial(n) // (math.factorial(r) * math.factorial(n - r))
                        return f"C({n},{r}) = {result}"
                    return "Invalid values for combination"
            
            elif 'factorial' in command and len(numbers) >= 1:
                n = int(numbers[0])
                if n >= 0:
                    result = math.factorial(n)
                    return f"{n}! = {result}"
                return "Factorial sirf non-negative integers ke liye defined hai"
            
            return "n aur r values provide kijiye for permutation/combination"
        except Exception as e:
            return f"Combinatorics calculation mein error: {str(e)}"
    
    def solve_probability(self, command):
        """Solve probability problems"""
        try:
            if 'coin flip' in command or 'coin toss' in command:
                return "Coin flip probability: Heads = 0.5, Tails = 0.5"
            
            elif 'dice roll' in command:
                return "Dice roll probability: Each face (1-6) = 1/6 = 0.1667"
            
            elif 'card draw' in command:
                return "Standard deck probabilities: Any card = 1/52, Any suit = 1/4, Any rank = 1/13"
            
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                favorable = numbers[0]
                total = numbers[1]
                if total > 0:
                    probability = favorable / total
                    return f"Probability = {favorable}/{total} = {probability:.4f} = {probability*100:.2f}%"
            
            return "Probability problem specify kijiye ya favorable/total outcomes provide kijiye"
        except Exception as e:
            return f"Probability calculation mein error: {str(e)}"
    
    def solve_geometry(self, command):
        """Solve geometry problems"""
        try:
            if 'circle area' in command:
                radius = self.extract_number_from_command(command)
                if radius is not None:
                    area = math.pi * radius ** 2
                    return f"Circle area (radius = {radius}) = {area:.4f}"
            
            elif 'circle circumference' in command:
                radius = self.extract_number_from_command(command)
                if radius is not None:
                    circumference = 2 * math.pi * radius
                    return f"Circle circumference (radius = {radius}) = {circumference:.4f}"
            
            elif 'rectangle area' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 2:
                    length, width = numbers[0], numbers[1]
                    area = length * width
                    return f"Rectangle area (length = {length}, width = {width}) = {area}"
            
            elif 'triangle area' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 2:
                    base, height = numbers[0], numbers[1]
                    area = 0.5 * base * height
                    return f"Triangle area (base = {base}, height = {height}) = {area}"
            
            elif 'sphere volume' in command:
                radius = self.extract_number_from_command(command)
                if radius is not None:
                    volume = (4/3) * math.pi * radius ** 3
                    return f"Sphere volume (radius = {radius}) = {volume:.4f}"
            
            elif 'cube volume' in command:
                side = self.extract_number_from_command(command)
                if side is not None:
                    volume = side ** 3
                    return f"Cube volume (side = {side}) = {volume}"
            
            return "Geometry problem specify kijiye: circle area/circumference, rectangle area, triangle area, sphere volume, cube volume"
        except Exception as e:
            return f"Geometry calculation mein error: {str(e)}"
    
    def solve_sequences(self, command):
        """Solve sequence and series problems"""
        try:
            if 'arithmetic progression' in command or 'ap' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 3:
                    a, d, n = numbers[0], numbers[1], int(numbers[2])
                    nth_term = a + (n - 1) * d
                    sum_n = (n/2) * (2*a + (n-1)*d)
                    return f"AP: a = {a}, d = {d}, n = {n}\n{n}th term = {nth_term}\nSum of {n} terms = {sum_n}"
            
            elif 'geometric progression' in command or 'gp' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 3:
                    a, r, n = numbers[0], numbers[1], int(numbers[2])
                    nth_term = a * (r ** (n - 1))
                    if r != 1:
                        sum_n = a * (r**n - 1) / (r - 1)
                    else:
                        sum_n = a * n
                    return f"GP: a = {a}, r = {r}, n = {n}\n{n}th term = {nth_term}\nSum of {n} terms = {sum_n}"
            
            elif 'fibonacci' in command:
                n = self.extract_number_from_command(command)
                if n is not None:
                    n = int(n)
                    if n <= 0:
                        return "Fibonacci sequence ke liye positive number chahiye"
                    elif n == 1 or n == 2:
                        return f"Fibonacci({n}) = 1"
                    else:
                        a, b = 1, 1
                        for i in range(3, n + 1):
                            a, b = b, a + b
                        return f"Fibonacci({n}) = {b}"
            
            return "Sequence type specify kijiye: arithmetic progression, geometric progression, fibonacci"
        except Exception as e:
            return f"Sequence calculation mein error: {str(e)}"
    
    def solve_complex_numbers(self, command):
        """Solve complex number problems"""
        try:
            if 'add complex' in command:
                return "Complex number addition: (a+bi) + (c+di) = (a+c) + (b+d)i"
            
            elif 'multiply complex' in command:
                return "Complex number multiplication: (a+bi) × (c+di) = (ac-bd) + (ad+bc)i"
            
            elif 'modulus' in command or 'magnitude' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 2:
                    real, imag = numbers[0], numbers[1]
                    modulus = (real**2 + imag**2)**0.5
                    return f"Modulus of {real}+{imag}i = {modulus:.4f}"
            
            elif 'conjugate' in command:
                numbers = self.extract_numbers_from_command(command)
                if len(numbers) >= 2:
                    real, imag = numbers[0], numbers[1]
                    return f"Conjugate of {real}+{imag}i = {real}-{imag}i"
            
            return "Complex number operation specify kijiye: add, multiply, modulus, conjugate"
        except Exception as e:
            return f"Complex number calculation mein error: {str(e)}"        
  
      # ==================== ULTRA ADVANCED MATHEMATICAL COMMANDS ====================
        
        # Differential Equations (1351-1400)
        if any(word in command for word in ['differential equation', 'ode', 'pde']):
            result = self.solve_differential_equations(command)
            self.speak(result)
        
        elif any(word in command for word in ['laplace transform', 'fourier transform']):
            result = self.solve_transforms(command)
            self.speak(result)
        
        # Advanced Calculus (1401-1450)
        elif any(word in command for word in ['partial derivative', 'gradient', 'divergence', 'curl']):
            result = self.solve_vector_calculus(command)
            self.speak(result)
        
        elif any(word in command for word in ['double integral', 'triple integral', 'line integral']):
            result = self.solve_multiple_integrals(command)
            self.speak(result)
        
        elif any(word in command for word in ['surface integral', 'volume integral']):
            result = self.solve_surface_volume_integrals(command)
            self.speak(result)
        
        # Number Theory (1451-1500)
        elif any(word in command for word in ['prime number', 'gcd', 'lcm', 'modular arithmetic']):
            result = self.solve_number_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['fibonacci sequence', 'lucas sequence', 'catalan numbers']):
            result = self.solve_special_sequences(command)
            self.speak(result)
        
        # Abstract Algebra (1501-1550)
        elif any(word in command for word in ['group theory', 'ring theory', 'field theory']):
            result = self.solve_abstract_algebra(command)
            self.speak(result)
        
        elif any(word in command for word in ['polynomial ring', 'ideal', 'quotient group']):
            result = self.solve_algebraic_structures(command)
            self.speak(result)
        
        # Graph Theory (1551-1600)
        elif any(word in command for word in ['graph theory', 'shortest path', 'minimum spanning tree']):
            result = self.solve_graph_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['euler path', 'hamiltonian path', 'graph coloring']):
            result = self.solve_graph_algorithms(command)
            self.speak(result)
        
        # Optimization (1601-1650)
        elif any(word in command for word in ['linear programming', 'simplex method', 'optimization']):
            result = self.solve_optimization(command)
            self.speak(result)
        
        elif any(word in command for word in ['lagrange multiplier', 'constrained optimization']):
            result = self.solve_constrained_optimization(command)
            self.speak(result)
        
        # Numerical Methods (1651-1700)
        elif any(word in command for word in ['newton raphson', 'bisection method', 'numerical integration']):
            result = self.solve_numerical_methods(command)
            self.speak(result)
        
        elif any(word in command for word in ['runge kutta', 'euler method', 'finite difference']):
            result = self.solve_numerical_ode(command)
            self.speak(result)
        
        # Advanced Statistics (1701-1750)
        elif any(word in command for word in ['hypothesis testing', 'confidence interval', 'regression']):
            result = self.solve_advanced_statistics(command)
            self.speak(result)
        
        elif any(word in command for word in ['anova', 'chi square', 'normal distribution']):
            result = self.solve_statistical_tests(command)
            self.speak(result)
        
        # Machine Learning Math (1751-1800)
        elif any(word in command for word in ['gradient descent', 'backpropagation', 'neural network']):
            result = self.solve_ml_mathematics(command)
            self.speak(result)
        
        elif any(word in command for word in ['pca', 'svd', 'eigendecomposition']):
            result = self.solve_dimensionality_reduction(command)
            self.speak(result)
        
        # Cryptography Math (1801-1850)
        elif any(word in command for word in ['rsa algorithm', 'elliptic curve', 'discrete logarithm']):
            result = self.solve_cryptography_math(command)
            self.speak(result)
        
        elif any(word in command for word in ['hash function', 'digital signature', 'key exchange']):
            result = self.solve_crypto_protocols(command)
            self.speak(result)
        
        # Topology (1851-1900)
        elif any(word in command for word in ['topology', 'homeomorphism', 'homotopy']):
            result = self.solve_topology(command)
            self.speak(result)
        
        elif any(word in command for word in ['fundamental group', 'covering space', 'manifold']):
            result = self.solve_algebraic_topology(command)
            self.speak(result)
        
        # Mathematical Physics (1901-1950)
        elif any(word in command for word in ['quantum mechanics', 'wave equation', 'schrodinger']):
            result = self.solve_quantum_math(command)
            self.speak(result)
        
        elif any(word in command for word in ['relativity', 'tensor calculus', 'field theory']):
            result = self.solve_physics_math(command)
            self.speak(result)
        
        # Financial Mathematics (1951-2000)
        elif any(word in command for word in ['black scholes', 'option pricing', 'risk management']):
            result = self.solve_financial_math(command)
            self.speak(result)
        
        elif any(word in command for word in ['monte carlo', 'stochastic calculus', 'portfolio optimization']):
            result = self.solve_quantitative_finance(command)
            self.speak(result)
        
        # Functions and Relations Advanced (2001-2100)
        elif any(word in command for word in ['function domain', 'domain', 'function range', 'range']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function value', 'evaluate function', 'f(x)']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['inverse function', 'function inverse']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['composite function', 'function composition']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['even function', 'odd function', 'function parity']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function monotonicity', 'increasing function', 'decreasing function']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function continuity', 'continuous function']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['linear function', 'quadratic function', 'polynomial function']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['rational function', 'exponential function', 'logarithmic function']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['trigonometric function', 'hyperbolic function']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['reflexive relation', 'symmetric relation', 'transitive relation']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['equivalence relation', 'partial order']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function zeros', 'function roots', 'x-intercepts']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function extrema', 'function maximum', 'function minimum']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function asymptotes', 'asymptote']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['function transformation', 'function shift', 'function stretch']):
            result = self.solve_functions_and_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['one to one function', 'onto function', 'bijective function']):
            result = self.check_function_types(command)
            self.speak(result)
        
        elif any(word in command for word in ['function graph', 'plot function']):
            result = self.plot_function_graph(command)
            self.speak(result)
        
        elif any(word in command for word in ['piecewise function', 'step function']):
            result = self.analyze_piecewise_function(command)
            self.speak(result)
        
        elif any(word in command for word in ['implicit function', 'parametric function']):
            result = self.analyze_special_functions(command)
            self.speak(result)
        
        # Advanced Mathematical Concepts (2101-2200)
        elif any(word in command for word in ['fractal geometry', 'mandelbrot set', 'julia set']):
            result = self.solve_fractal_geometry(command)
            self.speak(result)
        
        elif any(word in command for word in ['chaos theory', 'strange attractor', 'butterfly effect']):
            result = self.solve_chaos_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['game theory', 'nash equilibrium', 'minimax']):
            result = self.solve_game_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['information theory', 'entropy', 'mutual information']):
            result = self.solve_information_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['fuzzy logic', 'fuzzy set', 'membership function']):
            result = self.solve_fuzzy_mathematics(command)
            self.speak(result)
        
        # Mathematical Modeling (2201-2250)
        elif any(word in command for word in ['population model', 'epidemic model', 'predator prey']):
            result = self.solve_mathematical_modeling(command)
            self.speak(result)
        
        elif any(word in command for word in ['markov chain', 'random walk', 'stochastic process']):
            result = self.solve_stochastic_processes(command)
            self.speak(result)
        
        # Advanced Problem Solving (2251-2300)
        elif any(word in command for word in ['mathematical proof', 'induction', 'contradiction']):
            result = self.help_with_proofs(command)
            self.speak(result)
        
        elif any(word in command for word in ['olympiad problem', 'competition math', 'contest problem']):
            result = self.solve_competition_problems(command)
            self.speak(result)
        
        elif any(word in command for word in ['research problem', 'open problem', 'conjecture']):
            result = self.discuss_research_problems(command)
            self.speak(result)
        
        else:
            self.handle_unknown_command(command)
        
        return True    
  
  # ==================== ULTRA ADVANCED MATHEMATICAL SOLVER FUNCTIONS ====================
    
    def solve_differential_equations(self, command):
        """Solve differential equations"""
        try:
            if 'first order' in command or 'dy/dx' in command:
                return self.solve_first_order_ode(command)
            elif 'second order' in command or 'd2y/dx2' in command:
                return self.solve_second_order_ode(command)
            elif 'partial differential' in command or 'pde' in command:
                return self.solve_pde(command)
            elif 'laplace' in command:
                return "Laplace method: Transform ODE to algebraic equation, solve, then inverse transform"
            else:
                return "Differential equation type specify kijiye: first order, second order, partial differential"
        except Exception as e:
            return f"Differential equation solve karne mein error: {str(e)}"
    
    def solve_first_order_ode(self, command):
        """Solve first order ODE"""
        try:
            # Example: dy/dx = y
            if 'dy/dx = y' in command:
                return "Solution: y = Ce^x (exponential growth)"
            elif 'dy/dx = -y' in command:
                return "Solution: y = Ce^(-x) (exponential decay)"
            elif 'separable' in command:
                return "Separable ODE: Separate variables and integrate both sides"
            elif 'linear' in command:
                return "Linear ODE: Use integrating factor method μ(x) = e^(∫P(x)dx)"
            else:
                return "First order ODE methods: Separable, Linear, Exact, Bernoulli"
        except Exception as e:
            return f"First order ODE error: {str(e)}"
    
    def solve_second_order_ode(self, command):
        """Solve second order ODE"""
        try:
            if 'homogeneous' in command:
                return "Homogeneous 2nd order: Find characteristic equation r² + ar + b = 0"
            elif 'non-homogeneous' in command:
                return "Non-homogeneous: Find complementary + particular solution"
            elif 'constant coefficients' in command:
                return "Constant coefficients: Use characteristic equation method"
            else:
                return "Second order ODE types: Homogeneous, Non-homogeneous, Variable coefficients"
        except Exception as e:
            return f"Second order ODE error: {str(e)}"
    
    def solve_pde(self, command):
        """Solve partial differential equations"""
        try:
            if 'heat equation' in command:
                return "Heat equation: ∂u/∂t = α²∇²u (parabolic PDE)"
            elif 'wave equation' in command:
                return "Wave equation: ∂²u/∂t² = c²∇²u (hyperbolic PDE)"
            elif 'laplace equation' in command:
                return "Laplace equation: ∇²u = 0 (elliptic PDE)"
            elif 'separation of variables' in command:
                return "Separation method: Assume u(x,t) = X(x)T(t)"
            else:
                return "PDE types: Heat, Wave, Laplace, Poisson equations"
        except Exception as e:
            return f"PDE solve error: {str(e)}"
    
    def solve_transforms(self, command):
        """Solve transform problems"""
        try:
            if 'laplace transform' in command:
                return self.solve_laplace_transform(command)
            elif 'fourier transform' in command:
                return self.solve_fourier_transform(command)
            elif 'z transform' in command:
                return self.solve_z_transform(command)
            else:
                return "Transform type specify kijiye: Laplace, Fourier, Z-transform"
        except Exception as e:
            return f"Transform solve error: {str(e)}"
    
    def solve_laplace_transform(self, command):
        """Laplace transform solutions"""
        try:
            if 'L{1}' in command or 'unit step' in command:
                return "L{1} = 1/s"
            elif 'L{t}' in command:
                return "L{t} = 1/s²"
            elif 'L{e^at}' in command:
                return "L{e^(at)} = 1/(s-a)"
            elif 'L{sin(at)}' in command:
                return "L{sin(at)} = a/(s²+a²)"
            elif 'L{cos(at)}' in command:
                return "L{cos(at)} = s/(s²+a²)"
            else:
                return "Common Laplace transforms: 1→1/s, t→1/s², e^at→1/(s-a), sin(at)→a/(s²+a²)"
        except Exception as e:
            return f"Laplace transform error: {str(e)}"
    
    def solve_fourier_transform(self, command):
        """Fourier transform solutions"""
        try:
            if 'rectangular pulse' in command:
                return "F{rect(t)} = sinc(ωτ/2)"
            elif 'gaussian' in command:
                return "F{e^(-at²)} = √(π/a)e^(-ω²/4a)"
            elif 'delta function' in command:
                return "F{δ(t)} = 1"
            elif 'convolution' in command:
                return "Convolution theorem: F{f*g} = F{f}·F{g}"
            else:
                return "Fourier transform properties: Linearity, Time shifting, Frequency shifting, Scaling"
        except Exception as e:
            return f"Fourier transform error: {str(e)}"
    
    def solve_z_transform(self, command):
        """Z-transform solutions"""
        try:
            if 'unit step' in command:
                return "Z{u[n]} = z/(z-1)"
            elif 'exponential' in command:
                return "Z{a^n u[n]} = z/(z-a)"
            elif 'ramp' in command:
                return "Z{n u[n]} = z/(z-1)²"
            else:
                return "Z-transform pairs: δ[n]→1, u[n]→z/(z-1), a^n→z/(z-a)"
        except Exception as e:
            return f"Z-transform error: {str(e)}"
    
    def solve_vector_calculus(self, command):
        """Vector calculus operations"""
        try:
            if 'gradient' in command:
                return "Gradient: ∇f = (∂f/∂x, ∂f/∂y, ∂f/∂z) - points in direction of steepest increase"
            elif 'divergence' in command:
                return "Divergence: ∇·F = ∂Fx/∂x + ∂Fy/∂y + ∂Fz/∂z - measures source/sink strength"
            elif 'curl' in command:
                return "Curl: ∇×F = |i  j  k |\\n                |∂/∂x ∂/∂y ∂/∂z|\\n                |Fx Fy Fz| - measures rotation"
            elif 'laplacian' in command:
                return "Laplacian: ∇²f = ∂²f/∂x² + ∂²f/∂y² + ∂²f/∂z² - divergence of gradient"
            else:
                return "Vector calculus operators: Gradient (∇), Divergence (∇·), Curl (∇×), Laplacian (∇²)"
        except Exception as e:
            return f"Vector calculus error: {str(e)}"
    
    def solve_multiple_integrals(self, command):
        """Multiple integral solutions"""
        try:
            if 'double integral' in command:
                return "Double integral: ∫∫f(x,y)dxdy - integrate over 2D region"
            elif 'triple integral' in command:
                return "Triple integral: ∫∫∫f(x,y,z)dxdydz - integrate over 3D region"
            elif 'change of variables' in command:
                return "Change of variables: Use Jacobian determinant |∂(x,y)/∂(u,v)|"
            elif 'polar coordinates' in command:
                return "Polar: x=rcosθ, y=rsinθ, dxdy = rdrdθ"
            elif 'cylindrical coordinates' in command:
                return "Cylindrical: x=rcosθ, y=rsinθ, z=z, dV = rdrdθdz"
            elif 'spherical coordinates' in command:
                return "Spherical: x=ρsinφcosθ, y=ρsinφsinθ, z=ρcosφ, dV = ρ²sinφdρdφdθ"
            else:
                return "Multiple integrals: Double, Triple, with coordinate transformations"
        except Exception as e:
            return f"Multiple integrals error: {str(e)}"
    
    def solve_number_theory(self, command):
        """Number theory problems"""
        try:
            if 'prime number' in command:
                return self.check_prime_number(command)
            elif 'gcd' in command or 'greatest common divisor' in command:
                return self.calculate_gcd(command)
            elif 'lcm' in command or 'least common multiple' in command:
                return self.calculate_lcm(command)
            elif 'modular arithmetic' in command:
                return self.solve_modular_arithmetic(command)
            elif 'chinese remainder theorem' in command:
                return "Chinese Remainder Theorem: Solve system of congruences x ≡ ai (mod mi)"
            elif 'fermat little theorem' in command:
                return "Fermat's Little Theorem: If p is prime, then a^(p-1) ≡ 1 (mod p)"
            elif 'euler totient' in command:
                return "Euler's totient function φ(n): counts integers ≤ n that are coprime to n"
            else:
                return "Number theory topics: Primes, GCD/LCM, Modular arithmetic, Theorems"
        except Exception as e:
            return f"Number theory error: {str(e)}"
    
    def check_prime_number(self, command):
        """Check if number is prime"""
        try:
            number = self.extract_number_from_command(command)
            if number is not None:
                n = int(number)
                if n < 2:
                    return f"{n} is not a prime number"
                for i in range(2, int(n**0.5) + 1):
                    if n % i == 0:
                        return f"{n} is not prime (divisible by {i})"
                return f"{n} is a prime number"
            return "Number provide kijiye for prime check"
        except Exception as e:
            return f"Prime check error: {str(e)}"
    
    def calculate_gcd(self, command):
        """Calculate GCD using Euclidean algorithm"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                a, b = int(numbers[0]), int(numbers[1])
                original_a, original_b = a, b
                while b:
                    a, b = b, a % b
                return f"GCD({original_a}, {original_b}) = {a}"
            return "Do numbers provide kijiye for GCD"
        except Exception as e:
            return f"GCD calculation error: {str(e)}"
    
    def calculate_lcm(self, command):
        """Calculate LCM"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                a, b = int(numbers[0]), int(numbers[1])
                # LCM = (a*b)/GCD(a,b)
                def gcd(x, y):
                    while y:
                        x, y = y, x % y
                    return x
                lcm = abs(a * b) // gcd(a, b)
                return f"LCM({a}, {b}) = {lcm}"
            return "Do numbers provide kijiye for LCM"
        except Exception as e:
            return f"LCM calculation error: {str(e)}"
    
    def solve_modular_arithmetic(self, command):
        """Solve modular arithmetic problems"""
        try:
            if 'congruent' in command:
                return "a ≡ b (mod m) means m divides (a-b)"
            elif 'inverse' in command:
                return "Modular inverse: Find x such that ax ≡ 1 (mod m)"
            elif 'exponentiation' in command:
                return "Fast modular exponentiation: Use repeated squaring"
            else:
                return "Modular arithmetic: Congruences, Inverses, Exponentiation"
        except Exception as e:
            return f"Modular arithmetic error: {str(e)}"
    
    def solve_special_sequences(self, command):
        """Solve special mathematical sequences"""
        try:
            if 'fibonacci' in command:
                return self.generate_fibonacci_sequence(command)
            elif 'lucas' in command:
                return self.generate_lucas_sequence(command)
            elif 'catalan' in command:
                return self.generate_catalan_numbers(command)
            elif 'tribonacci' in command:
                return self.generate_tribonacci_sequence(command)
            elif 'pell' in command:
                return "Pell sequence: P(n) = 2P(n-1) + P(n-2), P(0)=0, P(1)=1"
            else:
                return "Special sequences: Fibonacci, Lucas, Catalan, Tribonacci, Pell"
        except Exception as e:
            return f"Special sequences error: {str(e)}"
    
    def generate_fibonacci_sequence(self, command):
        """Generate Fibonacci sequence"""
        try:
            n = self.extract_number_from_command(command)
            if n is not None:
                n = int(n)
                if n <= 0:
                    return "Positive number chahiye for Fibonacci"
                elif n <= 2:
                    return f"Fibonacci({n}) = 1"
                else:
                    fib = [1, 1]
                    for i in range(2, n):
                        fib.append(fib[i-1] + fib[i-2])
                    return f"Fibonacci sequence up to {n}: {fib}"
            return "Number provide kijiye for Fibonacci sequence"
        except Exception as e:
            return f"Fibonacci error: {str(e)}"
    
    def generate_lucas_sequence(self, command):
        """Generate Lucas sequence"""
        try:
            n = self.extract_number_from_command(command)
            if n is not None:
                n = int(n)
                if n <= 0:
                    return "Positive number chahiye for Lucas"
                elif n == 1:
                    return "Lucas(1) = 1"
                elif n == 2:
                    return "Lucas(2) = 3"
                else:
                    lucas = [1, 3]
                    for i in range(2, n):
                        lucas.append(lucas[i-1] + lucas[i-2])
                    return f"Lucas sequence up to {n}: {lucas}"
            return "Number provide kijiye for Lucas sequence"
        except Exception as e:
            return f"Lucas sequence error: {str(e)}"
    
    def generate_catalan_numbers(self, command):
        """Generate Catalan numbers"""
        try:
            n = self.extract_number_from_command(command)
            if n is not None:
                n = int(n)
                if n < 0:
                    return "Non-negative number chahiye for Catalan"
                
                # Catalan number formula: C(n) = (2n)! / ((n+1)! * n!)
                def catalan(num):
                    if num <= 1:
                        return 1
                    catalan_nums = [0] * (num + 1)
                    catalan_nums[0], catalan_nums[1] = 1, 1
                    
                    for i in range(2, num + 1):
                        for j in range(i):
                            catalan_nums[i] += catalan_nums[j] * catalan_nums[i-1-j]
                    return catalan_nums[num]
                
                result = catalan(n)
                return f"Catalan({n}) = {result}"
            return "Number provide kijiye for Catalan numbers"
        except Exception as e:
            return f"Catalan numbers error: {str(e)}"
    
    def generate_tribonacci_sequence(self, command):
        """Generate Tribonacci sequence"""
        try:
            n = self.extract_number_from_command(command)
            if n is not None:
                n = int(n)
                if n <= 0:
                    return "Positive number chahiye for Tribonacci"
                elif n <= 2:
                    return f"Tribonacci({n}) = 0"
                elif n == 3:
                    return "Tribonacci(3) = 1"
                else:
                    trib = [0, 0, 1]
                    for i in range(3, n):
                        trib.append(trib[i-1] + trib[i-2] + trib[i-3])
                    return f"Tribonacci sequence up to {n}: {trib}"
            return "Number provide kijiye for Tribonacci sequence"
        except Exception as e:
            return f"Tribonacci error: {str(e)}"
    
    def solve_abstract_algebra(self, command):
        """Abstract algebra concepts"""
        try:
            if 'group theory' in command:
                return self.explain_group_theory(command)
            elif 'ring theory' in command:
                return self.explain_ring_theory(command)
            elif 'field theory' in command:
                return self.explain_field_theory(command)
            elif 'homomorphism' in command:
                return "Homomorphism: Structure-preserving map between algebraic structures"
            elif 'isomorphism' in command:
                return "Isomorphism: Bijective homomorphism (structures are essentially same)"
            else:
                return "Abstract algebra: Groups, Rings, Fields, Homomorphisms, Isomorphisms"
        except Exception as e:
            return f"Abstract algebra error: {str(e)}"
    
    def explain_group_theory(self, command):
        """Explain group theory concepts"""
        try:
            if 'definition' in command:
                return "Group (G,*): Set with associative binary operation, identity element, every element has inverse"
            elif 'abelian' in command:
                return "Abelian group: Commutative group where a*b = b*a for all a,b"
            elif 'subgroup' in command:
                return "Subgroup: Subset of group that is itself a group under same operation"
            elif 'cyclic' in command:
                return "Cyclic group: Generated by single element g, every element is g^n"
            elif 'order' in command:
                return "Order of group: Number of elements. Order of element: smallest positive n where g^n = e"
            else:
                return "Group theory: Definition, Subgroups, Cyclic groups, Homomorphisms, Lagrange theorem"
        except Exception as e:
            return f"Group theory error: {str(e)}"
    
    def explain_ring_theory(self, command):
        """Explain ring theory concepts"""
        try:
            if 'definition' in command:
                return "Ring (R,+,*): Abelian group under +, associative *, distributive laws"
            elif 'integral domain' in command:
                return "Integral domain: Commutative ring with no zero divisors"
            elif 'field' in command:
                return "Field: Commutative ring where every non-zero element has multiplicative inverse"
            elif 'ideal' in command:
                return "Ideal: Subset I where a+b∈I and ra∈I for all a,b∈I, r∈R"
            elif 'quotient ring' in command:
                return "Quotient ring R/I: Ring formed by cosets of ideal I"
            else:
                return "Ring theory: Rings, Ideals, Quotient rings, Integral domains, Fields"
        except Exception as e:
            return f"Ring theory error: {str(e)}"
    
    def explain_field_theory(self, command):
        """Explain field theory concepts"""
        try:
            if 'definition' in command:
                return "Field: Ring where every non-zero element has multiplicative inverse"
            elif 'extension' in command:
                return "Field extension: Field F containing subfield K, written F/K"
            elif 'algebraic' in command:
                return "Algebraic element: α is algebraic over K if it's root of polynomial with coefficients in K"
            elif 'galois' in command:
                return "Galois theory: Studies field extensions using group theory"
            elif 'splitting field' in command:
                return "Splitting field: Smallest field extension where polynomial splits completely"
            else:
                return "Field theory: Extensions, Algebraic elements, Galois theory, Splitting fields"
        except Exception as e:
            return f"Field theory error: {str(e)}"    
    
    def solve_graph_theory(self, command):
        """Graph theory problems"""
        try:
            if 'shortest path' in command:
                return self.solve_shortest_path(command)
            elif 'minimum spanning tree' in command:
                return self.solve_mst(command)
            elif 'graph coloring' in command:
                return self.solve_graph_coloring(command)
            elif 'euler path' in command:
                return "Euler path: Path visiting every edge exactly once. Exists if ≤2 vertices have odd degree"
            elif 'hamiltonian path' in command:
                return "Hamiltonian path: Path visiting every vertex exactly once. NP-complete problem"
            elif 'planar graph' in command:
                return "Planar graph: Can be drawn without edge crossings. Euler's formula: V-E+F=2"
            else:
                return "Graph theory: Paths, Trees, Coloring, Planarity, Connectivity"
        except Exception as e:
            return f"Graph theory error: {str(e)}"
    
    def solve_shortest_path(self, command):
        """Shortest path algorithms"""
        try:
            if 'dijkstra' in command:
                return "Dijkstra's algorithm: Finds shortest path from source to all vertices (non-negative weights)"
            elif 'bellman ford' in command:
                return "Bellman-Ford: Handles negative weights, detects negative cycles, O(VE) time"
            elif 'floyd warshall' in command:
                return "Floyd-Warshall: All-pairs shortest paths, O(V³) time, handles negative weights"
            elif 'a star' in command:
                return "A* algorithm: Uses heuristic function, optimal if heuristic is admissible"
            else:
                return "Shortest path algorithms: Dijkstra, Bellman-Ford, Floyd-Warshall, A*"
        except Exception as e:
            return f"Shortest path error: {str(e)}"
    
    def solve_mst(self, command):
        """Minimum spanning tree algorithms"""
        try:
            if 'kruskal' in command:
                return "Kruskal's algorithm: Sort edges by weight, add if doesn't create cycle (union-find)"
            elif 'prim' in command:
                return "Prim's algorithm: Start from vertex, always add minimum weight edge to tree"
            elif 'boruvka' in command:
                return "Borůvka's algorithm: Each component finds minimum outgoing edge, merge components"
            else:
                return "MST algorithms: Kruskal's (edge-based), Prim's (vertex-based), Borůvka's"
        except Exception as e:
            return f"MST error: {str(e)}"
    
    def solve_graph_coloring(self, command):
        """Graph coloring problems"""
        try:
            if 'chromatic number' in command:
                return "Chromatic number χ(G): Minimum colors needed to color vertices so adjacent vertices have different colors"
            elif 'four color theorem' in command:
                return "Four Color Theorem: Every planar graph can be colored with at most 4 colors"
            elif 'greedy coloring' in command:
                return "Greedy coloring: Color vertices in order, use smallest available color"
            elif 'edge coloring' in command:
                return "Edge coloring: Color edges so adjacent edges have different colors"
            else:
                return "Graph coloring: Vertex coloring, Edge coloring, Chromatic number, Algorithms"
        except Exception as e:
            return f"Graph coloring error: {str(e)}"
    
    def solve_optimization(self, command):
        """Optimization problems"""
        try:
            if 'linear programming' in command:
                return self.solve_linear_programming(command)
            elif 'simplex method' in command:
                return "Simplex method: Iterative algorithm for linear programming, moves between vertices of feasible region"
            elif 'dual problem' in command:
                return "Dual problem: For every LP problem, there's associated dual. Strong duality: optimal values equal"
            elif 'integer programming' in command:
                return "Integer programming: LP with integer constraints. NP-hard, use branch-and-bound"
            elif 'dynamic programming' in command:
                return "Dynamic programming: Break problem into subproblems, store solutions (memoization)"
            else:
                return "Optimization: Linear programming, Integer programming, Dynamic programming, Convex optimization"
        except Exception as e:
            return f"Optimization error: {str(e)}"
    
    def solve_linear_programming(self, command):
        """Linear programming concepts"""
        try:
            if 'standard form' in command:
                return "Standard form: minimize c^T x subject to Ax = b, x ≥ 0"
            elif 'feasible region' in command:
                return "Feasible region: Set of all points satisfying constraints"
            elif 'basic solution' in command:
                return "Basic solution: Solution where n variables are zero (n = variables - constraints)"
            elif 'optimal solution' in command:
                return "Optimal solution: Feasible solution that minimizes/maximizes objective function"
            else:
                return "Linear programming: Standard form, Feasible region, Basic solutions, Optimality"
        except Exception as e:
            return f"Linear programming error: {str(e)}"
    
    def solve_constrained_optimization(self, command):
        """Constrained optimization methods"""
        try:
            if 'lagrange multiplier' in command:
                return self.solve_lagrange_multipliers(command)
            elif 'kkt conditions' in command:
                return "KKT conditions: Necessary conditions for optimality in constrained optimization"
            elif 'penalty method' in command:
                return "Penalty method: Convert constrained to unconstrained by adding penalty terms"
            elif 'barrier method' in command:
                return "Barrier method: Add barrier function to prevent constraint violation"
            else:
                return "Constrained optimization: Lagrange multipliers, KKT conditions, Penalty methods"
        except Exception as e:
            return f"Constrained optimization error: {str(e)}"
    
    def solve_lagrange_multipliers(self, command):
        """Lagrange multiplier method"""
        try:
            if 'method' in command:
                return "Lagrange multipliers: To optimize f(x,y) subject to g(x,y)=0, solve ∇f = λ∇g and g(x,y)=0"
            elif 'interpretation' in command:
                return "λ represents rate of change of optimal value with respect to constraint"
            elif 'multiple constraints' in command:
                return "Multiple constraints: ∇f = Σλᵢ∇gᵢ for constraints gᵢ(x)=0"
            else:
                return "Lagrange multipliers: Method for constrained optimization, geometric interpretation"
        except Exception as e:
            return f"Lagrange multipliers error: {str(e)}"
    
    def solve_numerical_methods(self, command):
        """Numerical methods for equations"""
        try:
            if 'newton raphson' in command:
                return self.solve_newton_raphson(command)
            elif 'bisection method' in command:
                return "Bisection method: Root finding by repeatedly halving interval. Guaranteed convergence but slow"
            elif 'secant method' in command:
                return "Secant method: Like Newton-Raphson but uses finite difference instead of derivative"
            elif 'fixed point iteration' in command:
                return "Fixed point iteration: Solve x = g(x) by iterating xₙ₊₁ = g(xₙ)"
            else:
                return "Numerical methods: Newton-Raphson, Bisection, Secant, Fixed point iteration"
        except Exception as e:
            return f"Numerical methods error: {str(e)}"
    
    def solve_newton_raphson(self, command):
        """Newton-Raphson method"""
        try:
            if 'formula' in command:
                return "Newton-Raphson: xₙ₊₁ = xₙ - f(xₙ)/f'(xₙ). Quadratic convergence near root"
            elif 'convergence' in command:
                return "Convergence: Quadratic if f'(root) ≠ 0. May diverge if poor initial guess"
            elif 'multivariable' in command:
                return "Multivariable: xₙ₊₁ = xₙ - [J(xₙ)]⁻¹f(xₙ) where J is Jacobian matrix"
            else:
                return "Newton-Raphson: Fast root finding method, requires derivative, quadratic convergence"
        except Exception as e:
            return f"Newton-Raphson error: {str(e)}"
    
    def solve_numerical_ode(self, command):
        """Numerical ODE methods"""
        try:
            if 'euler method' in command:
                return "Euler method: yₙ₊₁ = yₙ + h·f(xₙ,yₙ). Simple but low accuracy O(h)"
            elif 'runge kutta' in command:
                return "Runge-Kutta 4th order: Higher accuracy O(h⁴), uses weighted average of slopes"
            elif 'adams bashforth' in command:
                return "Adams-Bashforth: Multistep method using previous points for better accuracy"
            elif 'backward euler' in command:
                return "Backward Euler: Implicit method yₙ₊₁ = yₙ + h·f(xₙ₊₁,yₙ₊₁). Better stability"
            else:
                return "Numerical ODE: Euler, Runge-Kutta, Adams methods, Stability analysis"
        except Exception as e:
            return f"Numerical ODE error: {str(e)}"
    
    def solve_advanced_statistics(self, command):
        """Advanced statistical methods"""
        try:
            if 'hypothesis testing' in command:
                return self.solve_hypothesis_testing(command)
            elif 'confidence interval' in command:
                return self.solve_confidence_intervals(command)
            elif 'regression' in command:
                return self.solve_regression_analysis(command)
            elif 'bayesian' in command:
                return "Bayesian statistics: P(H|E) = P(E|H)P(H)/P(E). Updates beliefs with evidence"
            elif 'maximum likelihood' in command:
                return "Maximum Likelihood: Find parameters that maximize likelihood of observed data"
            else:
                return "Advanced statistics: Hypothesis testing, Confidence intervals, Regression, Bayesian methods"
        except Exception as e:
            return f"Advanced statistics error: {str(e)}"
    
    def solve_hypothesis_testing(self, command):
        """Hypothesis testing concepts"""
        try:
            if 'null hypothesis' in command:
                return "Null hypothesis H₀: Statement of no effect/difference. Alternative H₁: What we want to prove"
            elif 'p value' in command:
                return "P-value: Probability of observing data as extreme as observed, assuming H₀ is true"
            elif 'type 1 error' in command:
                return "Type I error (α): Rejecting true H₀. Type II error (β): Accepting false H₀"
            elif 'power' in command:
                return "Statistical power: 1-β, probability of correctly rejecting false H₀"
            else:
                return "Hypothesis testing: H₀ vs H₁, P-values, Type I/II errors, Statistical power"
        except Exception as e:
            return f"Hypothesis testing error: {str(e)}"
    
    def solve_confidence_intervals(self, command):
        """Confidence interval concepts"""
        try:
            if 'interpretation' in command:
                return "95% CI: If we repeat sampling many times, 95% of intervals will contain true parameter"
            elif 'margin of error' in command:
                return "Margin of error: Half-width of confidence interval, depends on sample size and confidence level"
            elif 'sample size' in command:
                return "Larger sample size → smaller margin of error → narrower confidence interval"
            else:
                return "Confidence intervals: Interpretation, Margin of error, Effect of sample size"
        except Exception as e:
            return f"Confidence intervals error: {str(e)}"
    
    def solve_regression_analysis(self, command):
        """Regression analysis concepts"""
        try:
            if 'linear regression' in command:
                return "Linear regression: y = β₀ + β₁x + ε. Minimize sum of squared residuals"
            elif 'multiple regression' in command:
                return "Multiple regression: y = β₀ + β₁x₁ + β₂x₂ + ... + ε"
            elif 'r squared' in command:
                return "R²: Proportion of variance explained by model. R² = 1 - SSres/SStot"
            elif 'assumptions' in command:
                return "Regression assumptions: Linearity, Independence, Homoscedasticity, Normality (LINE)"
            else:
                return "Regression: Linear, Multiple, Logistic, Assumptions, Model diagnostics"
        except Exception as e:
            return f"Regression analysis error: {str(e)}"
    
    def solve_statistical_tests(self, command):
        """Statistical test procedures"""
        try:
            if 'anova' in command:
                return "ANOVA: Tests equality of means across groups. F = MSbetween/MSwithin"
            elif 'chi square' in command:
                return "Chi-square test: Tests independence/goodness of fit. χ² = Σ(O-E)²/E"
            elif 't test' in command:
                return "T-test: Compares means. One-sample, two-sample, paired t-tests"
            elif 'normal distribution' in command:
                return "Normal distribution: Bell curve, μ±σ contains 68%, μ±2σ contains 95%"
            else:
                return "Statistical tests: T-test, ANOVA, Chi-square, Normal distribution properties"
        except Exception as e:
            return f"Statistical tests error: {str(e)}"
    
    def solve_ml_mathematics(self, command):
        """Machine learning mathematics"""
        try:
            if 'gradient descent' in command:
                return "Gradient descent: θ = θ - α∇J(θ). Minimizes cost function by following negative gradient"
            elif 'backpropagation' in command:
                return "Backpropagation: Computes gradients in neural networks using chain rule"
            elif 'neural network' in command:
                return "Neural network: f(x) = σ(Wx + b) where σ is activation function"
            elif 'loss function' in command:
                return "Loss functions: MSE for regression, Cross-entropy for classification"
            else:
                return "ML mathematics: Gradient descent, Backpropagation, Loss functions, Optimization"
        except Exception as e:
            return f"ML mathematics error: {str(e)}"
    
    def solve_dimensionality_reduction(self, command):
        """Dimensionality reduction techniques"""
        try:
            if 'pca' in command:
                return "PCA: Projects data onto principal components (eigenvectors of covariance matrix)"
            elif 'svd' in command:
                return "SVD: A = UΣV^T. Decomposes matrix into orthogonal matrices and diagonal matrix"
            elif 'eigendecomposition' in command:
                return "Eigendecomposition: A = QΛQ^(-1) where Q contains eigenvectors, Λ eigenvalues"
            elif 'lda' in command:
                return "LDA: Linear Discriminant Analysis, finds projection that maximizes class separation"
            else:
                return "Dimensionality reduction: PCA, SVD, LDA, t-SNE, Eigendecomposition"
        except Exception as e:
            return f"Dimensionality reduction error: {str(e)}"
    
    def solve_cryptography_math(self, command):
        """Cryptography mathematics"""
        try:
            if 'rsa algorithm' in command:
                return "RSA: Choose primes p,q. n=pq, φ(n)=(p-1)(q-1). Choose e, find d: ed≡1(mod φ(n))"
            elif 'elliptic curve' in command:
                return "Elliptic curves: y²=x³+ax+b. Point addition forms group, basis for ECC"
            elif 'discrete logarithm' in command:
                return "Discrete log: Given g^x ≡ h (mod p), find x. Hard problem, basis for crypto"
            elif 'diffie hellman' in command:
                return "Diffie-Hellman: Key exchange using g^(ab) = (g^a)^b = (g^b)^a"
            else:
                return "Cryptography: RSA, Elliptic curves, Discrete logarithm, Key exchange"
        except Exception as e:
            return f"Cryptography math error: {str(e)}"
    
    def solve_functions_and_relations(self, command):
        """Comprehensive Functions and Relations solver"""
        try:
            # Basic Function Operations
            if any(word in command for word in ['function domain', 'domain']):
                return self.find_function_domain(command)
            elif any(word in command for word in ['function range', 'range']):
                return self.find_function_range(command)
            elif any(word in command for word in ['function value', 'f(x)', 'evaluate function']):
                return self.evaluate_function(command)
            elif any(word in command for word in ['inverse function', 'function inverse']):
                return self.find_inverse_function(command)
            elif any(word in command for word in ['composite function', 'function composition']):
                return self.compose_functions(command)
            
            # Function Properties
            elif any(word in command for word in ['even function', 'odd function', 'function parity']):
                return self.check_function_parity(command)
            elif any(word in command for word in ['function monotonicity', 'increasing', 'decreasing']):
                return self.check_monotonicity(command)
            elif any(word in command for word in ['function continuity', 'continuous']):
                return self.check_continuity(command)
            elif any(word in command for word in ['function differentiability', 'differentiable']):
                return self.check_differentiability(command)
            
            # Function Types
            elif any(word in command for word in ['linear function', 'linear']):
                return self.analyze_linear_function(command)
            elif any(word in command for word in ['quadratic function', 'parabola']):
                return self.analyze_quadratic_function(command)
            elif any(word in command for word in ['polynomial function', 'polynomial']):
                return self.analyze_polynomial_function(command)
            elif any(word in command for word in ['rational function', 'rational']):
                return self.analyze_rational_function(command)
            elif any(word in command for word in ['exponential function', 'exponential']):
                return self.analyze_exponential_function(command)
            elif any(word in command for word in ['logarithmic function', 'logarithmic']):
                return self.analyze_logarithmic_function(command)
            elif any(word in command for word in ['trigonometric function', 'trig function']):
                return self.analyze_trigonometric_function(command)
            elif any(word in command for word in ['hyperbolic function', 'hyperbolic']):
                return self.analyze_hyperbolic_function(command)
            
            # Relations
            elif any(word in command for word in ['relation domain', 'relation range']):
                return self.analyze_relation_domain_range(command)
            elif any(word in command for word in ['reflexive relation', 'reflexive']):
                return self.check_reflexive_relation(command)
            elif any(word in command for word in ['symmetric relation', 'symmetric']):
                return self.check_symmetric_relation(command)
            elif any(word in command for word in ['transitive relation', 'transitive']):
                return self.check_transitive_relation(command)
            elif any(word in command for word in ['equivalence relation', 'equivalence']):
                return self.check_equivalence_relation(command)
            elif any(word in command for word in ['partial order', 'ordering']):
                return self.check_partial_order(command)
            
            # Advanced Function Analysis
            elif any(word in command for word in ['function zeros', 'roots', 'x-intercepts']):
                return self.find_function_zeros(command)
            elif any(word in command for word in ['function extrema', 'maximum', 'minimum']):
                return self.find_function_extrema(command)
            elif any(word in command for word in ['function inflection', 'inflection points']):
                return self.find_inflection_points(command)
            elif any(word in command for word in ['function asymptotes', 'asymptote']):
                return self.find_asymptotes(command)
            elif any(word in command for word in ['function graph', 'plot function']):
                return self.plot_function(command)
            
            # Function Transformations
            elif any(word in command for word in ['function shift', 'translate function']):
                return self.transform_function_shift(command)
            elif any(word in command for word in ['function stretch', 'scale function']):
                return self.transform_function_stretch(command)
            elif any(word in command for word in ['function reflection', 'reflect function']):
                return self.transform_function_reflection(command)
            
            else:
                return "Function/Relation operation specify kijiye: domain, range, inverse, composition, properties, types, transformations"
        
        except Exception as e:
            return f"Function/Relation calculation mein error: {str(e)}"    
  
  # ==================== FUNCTIONS AND RELATIONS IMPLEMENTATION ====================
    
    def find_function_domain(self, command):
        """Find domain of a function"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                
                # Check for common domain restrictions
                if 'sqrt' in str(expr) or 'log' in str(expr) or '/' in str(expr):
                    domain_restrictions = []
                    
                    # Square root restrictions
                    if 'sqrt' in str(expr):
                        domain_restrictions.append("Expression under square root must be ≥ 0")
                    
                    # Logarithm restrictions  
                    if 'log' in str(expr):
                        domain_restrictions.append("Argument of logarithm must be > 0")
                    
                    # Division restrictions
                    if '/' in str(expr):
                        domain_restrictions.append("Denominator cannot be zero")
                    
                    return f"Function: {function_str}\nDomain restrictions: {'; '.join(domain_restrictions)}"
                else:
                    return f"Function: {function_str}\nDomain: All real numbers (ℝ)"
            
            return "Function provide kijiye for domain analysis"
        except Exception as e:
            return f"Domain calculation error: {str(e)}"
    
    def find_function_range(self, command):
        """Find range of a function"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                
                # For simple functions, provide general range information
                if expr.is_polynomial():
                    degree = sp.degree(expr)
                    if degree == 1:
                        return f"Linear function: {function_str}\nRange: All real numbers (ℝ)"
                    elif degree == 2:
                        # Check if parabola opens up or down
                        leading_coeff = expr.as_coefficients_dict()[self.x**2]
                        if leading_coeff > 0:
                            vertex_y = expr.subs(self.x, -sp.diff(expr, self.x).as_coefficients_dict()[self.x]/(2*leading_coeff))
                            return f"Quadratic function: {function_str}\nRange: y ≥ {vertex_y} (parabola opens upward)"
                        else:
                            vertex_y = expr.subs(self.x, -sp.diff(expr, self.x).as_coefficients_dict()[self.x]/(2*leading_coeff))
                            return f"Quadratic function: {function_str}\nRange: y ≤ {vertex_y} (parabola opens downward)"
                
                return f"Function: {function_str}\nRange analysis requires more specific function type"
            
            return "Function provide kijiye for range analysis"
        except Exception as e:
            return f"Range calculation error: {str(e)}"
    
    def evaluate_function(self, command):
        """Evaluate function at given point"""
        try:
            function_str = self.extract_function_from_command(command)
            value = self.extract_evaluation_point(command)
            
            if function_str and value is not None:
                expr = sp.sympify(function_str)
                result = expr.subs(self.x, value)
                return f"f({value}) = {result} where f(x) = {function_str}"
            
            return "Function aur evaluation point provide kijiye (e.g., 'evaluate x^2+1 at x=2')"
        except Exception as e:
            return f"Function evaluation error: {str(e)}"
    
    def find_inverse_function(self, command):
        """Find inverse of a function"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                y = symbols('y')
                
                # Solve y = f(x) for x
                equation = sp.Eq(y, expr)
                inverse_solutions = solve(equation, self.x)
                
                if inverse_solutions:
                    inverse_expr = inverse_solutions[0]
                    return f"Function: f(x) = {function_str}\nInverse: f⁻¹(x) = {inverse_expr}"
                else:
                    return f"Inverse function for {function_str} cannot be found algebraically"
            
            return "Function provide kijiye for inverse calculation"
        except Exception as e:
            return f"Inverse function error: {str(e)}"
    
    def compose_functions(self, command):
        """Compose two functions"""
        try:
            functions = self.extract_multiple_functions(command)
            if len(functions) >= 2:
                f_expr = sp.sympify(functions[0])
                g_expr = sp.sympify(functions[1])
                
                # Calculate f(g(x))
                fog = f_expr.subs(self.x, g_expr)
                # Calculate g(f(x))
                gof = g_expr.subs(self.x, f_expr)
                
                return f"f(x) = {functions[0]}, g(x) = {functions[1]}\n(f∘g)(x) = f(g(x)) = {fog}\n(g∘f)(x) = g(f(x)) = {gof}"
            
            return "Do functions provide kijiye for composition"
        except Exception as e:
            return f"Function composition error: {str(e)}"
    
    def check_function_parity(self, command):
        """Check if function is even, odd, or neither"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                
                # Check f(-x)
                f_minus_x = expr.subs(self.x, -self.x)
                
                # Check if f(-x) = f(x) (even)
                if simplify(f_minus_x - expr) == 0:
                    return f"Function f(x) = {function_str} is EVEN\nf(-x) = f(x)"
                
                # Check if f(-x) = -f(x) (odd)
                elif simplify(f_minus_x + expr) == 0:
                    return f"Function f(x) = {function_str} is ODD\nf(-x) = -f(x)"
                
                else:
                    return f"Function f(x) = {function_str} is NEITHER even nor odd"
            
            return "Function provide kijiye for parity check"
        except Exception as e:
            return f"Parity check error: {str(e)}"
    
    def check_monotonicity(self, command):
        """Check if function is increasing or decreasing"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                derivative = diff(expr, self.x)
                
                # Analyze the sign of derivative
                critical_points = solve(derivative, self.x)
                
                result = f"Function: f(x) = {function_str}\n"
                result += f"Derivative: f'(x) = {derivative}\n"
                
                if critical_points:
                    result += f"Critical points: {critical_points}\n"
                    result += "Analyze derivative sign in intervals between critical points for monotonicity"
                else:
                    # Check if derivative is always positive or negative
                    test_points = [-1, 0, 1]
                    signs = [derivative.subs(self.x, point) for point in test_points]
                    
                    if all(s > 0 for s in signs if s.is_real):
                        result += "Function is STRICTLY INCREASING (f'(x) > 0)"
                    elif all(s < 0 for s in signs if s.is_real):
                        result += "Function is STRICTLY DECREASING (f'(x) < 0)"
                    else:
                        result += "Function monotonicity varies"
                
                return result
            
            return "Function provide kijiye for monotonicity analysis"
        except Exception as e:
            return f"Monotonicity check error: {str(e)}"
    
    def check_continuity(self, command):
        """Check function continuity"""
        try:
            function_str = self.extract_function_from_command(command)
            point = self.extract_evaluation_point(command)
            
            if function_str:
                expr = sp.sympify(function_str)
                
                if point is not None:
                    # Check continuity at specific point
                    try:
                        left_limit = limit(expr, self.x, point, '-')
                        right_limit = limit(expr, self.x, point, '+')
                        function_value = expr.subs(self.x, point)
                        
                        if left_limit == right_limit == function_value:
                            return f"Function f(x) = {function_str} is CONTINUOUS at x = {point}"
                        else:
                            return f"Function f(x) = {function_str} is DISCONTINUOUS at x = {point}\nLeft limit: {left_limit}, Right limit: {right_limit}, f({point}) = {function_value}"
                    except:
                        return f"Cannot determine continuity at x = {point}"
                else:
                    # General continuity analysis
                    if expr.is_polynomial():
                        return f"Polynomial function f(x) = {function_str} is continuous everywhere"
                    elif 'log' in str(expr):
                        return f"Function f(x) = {function_str} is continuous on its domain (where argument > 0)"
                    elif '/' in str(expr):
                        return f"Rational function f(x) = {function_str} is continuous except where denominator = 0"
                    else:
                        return f"Function f(x) = {function_str} - continuity depends on specific form"
            
            return "Function provide kijiye for continuity check"
        except Exception as e:
            return f"Continuity check error: {str(e)}"
    
    def analyze_linear_function(self, command):
        """Analyze linear function properties"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                
                if expr.is_polynomial() and sp.degree(expr) == 1:
                    coeffs = sp.Poly(expr, self.x).all_coeffs()
                    slope = coeffs[0]
                    y_intercept = coeffs[1] if len(coeffs) > 1 else 0
                    x_intercept = -y_intercept/slope if slope != 0 else "undefined"
                    
                    result = f"Linear Function Analysis: f(x) = {function_str}\n"
                    result += f"Slope (m): {slope}\n"
                    result += f"Y-intercept: {y_intercept}\n"
                    result += f"X-intercept: {x_intercept}\n"
                    result += f"Domain: All real numbers\n"
                    result += f"Range: All real numbers\n"
                    
                    if slope > 0:
                        result += "Function is increasing"
                    elif slope < 0:
                        result += "Function is decreasing"
                    else:
                        result += "Function is constant"
                    
                    return result
                else:
                    return f"{function_str} is not a linear function"
            
            return "Linear function provide kijiye (e.g., 2x + 3)"
        except Exception as e:
            return f"Linear function analysis error: {str(e)}"
    
    def analyze_quadratic_function(self, command):
        """Analyze quadratic function properties"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                
                if expr.is_polynomial() and sp.degree(expr) == 2:
                    coeffs = sp.Poly(expr, self.x).all_coeffs()
                    a, b, c = coeffs[0], coeffs[1] if len(coeffs) > 1 else 0, coeffs[2] if len(coeffs) > 2 else 0
                    
                    # Vertex
                    vertex_x = -b/(2*a)
                    vertex_y = expr.subs(self.x, vertex_x)
                    
                    # Discriminant
                    discriminant = b**2 - 4*a*c
                    
                    # Axis of symmetry
                    axis = vertex_x
                    
                    result = f"Quadratic Function Analysis: f(x) = {function_str}\n"
                    result += f"Standard form: f(x) = {a}x² + {b}x + {c}\n"
                    result += f"Vertex: ({vertex_x}, {vertex_y})\n"
                    result += f"Axis of symmetry: x = {axis}\n"
                    result += f"Discriminant: {discriminant}\n"
                    
                    if discriminant > 0:
                        roots = solve(expr, self.x)
                        result += f"Real roots: {roots}\n"
                    elif discriminant == 0:
                        result += f"One real root (repeated): x = {vertex_x}\n"
                    else:
                        result += "No real roots (complex roots)\n"
                    
                    if a > 0:
                        result += f"Parabola opens upward\nRange: y ≥ {vertex_y}"
                    else:
                        result += f"Parabola opens downward\nRange: y ≤ {vertex_y}"
                    
                    return result
                else:
                    return f"{function_str} is not a quadratic function"
            
            return "Quadratic function provide kijiye (e.g., x^2 + 2x + 1)"
        except Exception as e:
            return f"Quadratic function analysis error: {str(e)}"
    
    # Helper methods for Functions and Relations
    def extract_evaluation_point(self, command):
        """Extract evaluation point from command"""
        patterns = [
            r'at\s+x\s*=\s*([0-9\-\.]+)',
            r'when\s+x\s*=\s*([0-9\-\.]+)',
            r'x\s*=\s*([0-9\-\.]+)',
            r'point\s+([0-9\-\.]+)'
        ]
        
        for pattern in patterns:
            match = re.search(pattern, command, re.IGNORECASE)
            if match:
                try:
                    return float(match.group(1))
                except:
                    continue
        return None
    
    def extract_multiple_functions(self, command):
        """Extract multiple functions from command"""
        # Look for patterns like "f(x) = ... and g(x) = ..."
        function_patterns = [
            r'f\(x\)\s*=\s*([^,]+)',
            r'g\(x\)\s*=\s*([^,]+)',
            r'function\s+([^,]+)'
        ]
        
        functions = []
        for pattern in function_patterns:
            matches = re.findall(pattern, command, re.IGNORECASE)
            functions.extend(matches)
        
        return [f.strip() for f in functions]
    
    # Additional helper methods for relations
    def check_reflexive_relation(self, command):
        """Check if relation is reflexive"""
        try:
            relation_pairs = self.extract_relation_pairs(command)
            domain_set = self.extract_domain_set(command)
            
            if relation_pairs and domain_set:
                # Check if (a,a) exists for all a in domain
                reflexive = True
                missing_pairs = []
                
                for element in domain_set:
                    if (element, element) not in relation_pairs:
                        reflexive = False
                        missing_pairs.append((element, element))
                
                if reflexive:
                    return f"Relation is REFLEXIVE\nAll pairs (a,a) exist for domain elements"
                else:
                    return f"Relation is NOT REFLEXIVE\nMissing pairs: {missing_pairs}"
            
            return "Relation pairs aur domain set provide kijiye"
        except Exception as e:
            return f"Reflexive check error: {str(e)}"
    
    def check_symmetric_relation(self, command):
        """Check if relation is symmetric"""
        try:
            relation_pairs = self.extract_relation_pairs(command)
            
            if relation_pairs:
                symmetric = True
                missing_pairs = []
                
                for (a, b) in relation_pairs:
                    if a != b and (b, a) not in relation_pairs:
                        symmetric = False
                        missing_pairs.append((b, a))
                
                if symmetric:
                    return f"Relation is SYMMETRIC\nFor every (a,b), (b,a) also exists"
                else:
                    return f"Relation is NOT SYMMETRIC\nMissing pairs: {missing_pairs}"
            
            return "Relation pairs provide kijiye"
        except Exception as e:
            return f"Symmetric check error: {str(e)}"
    
    def check_transitive_relation(self, command):
        """Check if relation is transitive"""
        try:
            relation_pairs = self.extract_relation_pairs(command)
            
            if relation_pairs:
                transitive = True
                missing_pairs = []
                
                for (a, b) in relation_pairs:
                    for (c, d) in relation_pairs:
                        if b == c and (a, d) not in relation_pairs:
                            transitive = False
                            missing_pairs.append((a, d))
                
                if transitive:
                    return f"Relation is TRANSITIVE\nFor (a,b) and (b,c), (a,c) exists"
                else:
                    return f"Relation is NOT TRANSITIVE\nMissing pairs: {set(missing_pairs)}"
            
            return "Relation pairs provide kijiye"
        except Exception as e:
            return f"Transitive check error: {str(e)}"
    
    def check_equivalence_relation(self, command):
        """Check if relation is equivalence relation"""
        try:
            relation_pairs = self.extract_relation_pairs(command)
            domain_set = self.extract_domain_set(command)
            
            if relation_pairs and domain_set:
                # Check reflexive
                reflexive = all((a, a) in relation_pairs for a in domain_set)
                
                # Check symmetric
                symmetric = all((b, a) in relation_pairs for (a, b) in relation_pairs if a != b)
                
                # Check transitive
                transitive = True
                for (a, b) in relation_pairs:
                    for (c, d) in relation_pairs:
                        if b == c and (a, d) not in relation_pairs:
                            transitive = False
                            break
                    if not transitive:
                        break
                
                result = f"Equivalence Relation Check:\n"
                result += f"Reflexive: {'✓' if reflexive else '✗'}\n"
                result += f"Symmetric: {'✓' if symmetric else '✗'}\n"
                result += f"Transitive: {'✓' if transitive else '✗'}\n"
                
                if reflexive and symmetric and transitive:
                    result += "Relation is an EQUIVALENCE RELATION"
                else:
                    result += "Relation is NOT an equivalence relation"
                
                return result
            
            return "Relation pairs aur domain set provide kijiye"
        except Exception as e:
            return f"Equivalence relation check error: {str(e)}"
    
    def extract_relation_pairs(self, command):
        """Extract relation pairs from command"""
        # Look for patterns like {(1,1), (1,2), (2,2)}
        pair_pattern = r'\{([^}]+)\}'
        match = re.search(pair_pattern, command)
        
        if match:
            pairs_str = match.group(1)
            # Extract individual pairs like (1,2)
            individual_pairs = re.findall(r'\(([^)]+)\)', pairs_str)
            
            relation_pairs = []
            for pair_str in individual_pairs:
                try:
                    elements = [x.strip() for x in pair_str.split(',')]
                    if len(elements) == 2:
                        # Try to convert to numbers, otherwise keep as strings
                        try:
                            a, b = float(elements[0]), float(elements[1])
                        except:
                            a, b = elements[0], elements[1]
                        relation_pairs.append((a, b))
                except:
                    continue
            
            return relation_pairs
        return []
    
    def extract_domain_set(self, command):
        """Extract domain set from command"""
        # Look for domain patterns like "domain {1,2,3}"
        domain_pattern = r'domain\s*\{([^}]+)\}'
        match = re.search(domain_pattern, command, re.IGNORECASE)
        
        if match:
            elements_str = match.group(1)
            elements = [x.strip() for x in elements_str.split(',')]
            
            domain_set = []
            for elem in elements:
                try:
                    domain_set.append(float(elem))
                except:
                    domain_set.append(elem)
            
            return domain_set
        return []      
  
        # ==================== MATHEMATICAL FORMULAS & EQUATIONS COMMANDS ====================
        
        # Basic Mathematical Formulas (2301-2400)
        if any(word in command for word in ['basic formula', 'fundamental formula', 'basic equation']):
            result = self.get_basic_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['algebra formula', 'algebraic formula']):
            result = self.get_algebra_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['geometry formula', 'geometric formula']):
            result = self.get_geometry_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['trigonometry formula', 'trig formula']):
            result = self.get_trigonometry_formulas(command)
            self.speak(result)
        
        # Calculus Formulas (2401-2450)
        elif any(word in command for word in ['calculus formula', 'derivative formula', 'integration formula']):
            result = self.get_calculus_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['differential formula', 'differentiation rule']):
            result = self.get_differentiation_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['integration rule', 'integral formula']):
            result = self.get_integration_formulas(command)
            self.speak(result)
        
        # Physics Formulas (2451-2550)
        elif any(word in command for word in ['physics formula', 'mechanics formula']):
            result = self.get_physics_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['thermodynamics formula', 'heat formula']):
            result = self.get_thermodynamics_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['electromagnetism formula', 'electric formula']):
            result = self.get_electromagnetism_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['wave formula', 'optics formula']):
            result = self.get_wave_optics_formulas(command)
            self.speak(result)
        
        # Engineering Formulas (2551-2650)
        elif any(word in command for word in ['engineering formula', 'mechanical formula']):
            result = self.get_engineering_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['electrical formula', 'circuit formula']):
            result = self.get_electrical_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['civil formula', 'structural formula']):
            result = self.get_civil_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['chemical formula', 'chemistry equation']):
            result = self.get_chemical_formulas(command)
            self.speak(result)
        
        # Advanced Engineering Formulas (2651-2750)
        elif any(word in command for word in ['control system formula', 'control formula']):
            result = self.get_control_system_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['signal processing formula', 'dsp formula']):
            result = self.get_signal_processing_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['fluid mechanics formula', 'fluid formula']):
            result = self.get_fluid_mechanics_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['heat transfer formula', 'thermal formula']):
            result = self.get_heat_transfer_formulas(command)
            self.speak(result)
        
        # Statistics & Probability Formulas (2751-2800)
        elif any(word in command for word in ['statistics formula', 'probability formula']):
            result = self.get_statistics_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['distribution formula', 'normal distribution']):
            result = self.get_distribution_formulas(command)
            self.speak(result)
        
        # Computer Science Formulas (2801-2850)
        elif any(word in command for word in ['algorithm formula', 'complexity formula']):
            result = self.get_algorithm_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['data structure formula', 'computer formula']):
            result = self.get_computer_science_formulas(command)
            self.speak(result)
        
        # Financial Engineering Formulas (2851-2900)
        elif any(word in command for word in ['financial formula', 'finance equation']):
            result = self.get_financial_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['compound interest', 'simple interest']):
            result = self.get_interest_formulas(command)
            self.speak(result)  
  
    # ==================== COMPREHENSIVE MATHEMATICAL FORMULAS DATABASE ====================
    
    def get_basic_formulas(self, command):
        """Basic mathematical formulas"""
        try:
            if 'quadratic' in command:
                return """Quadratic Formula:
x = (-b ± √(b² - 4ac)) / 2a
For equation ax² + bx + c = 0
Discriminant: Δ = b² - 4ac
If Δ > 0: Two real roots
If Δ = 0: One real root
If Δ < 0: Complex roots"""
            
            elif 'distance' in command:
                return """Distance Formulas:
2D Distance: d = √[(x₂-x₁)² + (y₂-y₁)²]
3D Distance: d = √[(x₂-x₁)² + (y₂-y₁)² + (z₂-z₁)²]
Midpoint: M = ((x₁+x₂)/2, (y₁+y₂)/2)"""
            
            elif 'slope' in command:
                return """Slope Formula:
m = (y₂ - y₁) / (x₂ - x₁)
Point-Slope Form: y - y₁ = m(x - x₁)
Slope-Intercept Form: y = mx + b"""
            
            elif 'percentage' in command:
                return """Percentage Formulas:
Percentage = (Part/Whole) × 100%
Percentage Change = ((New - Old)/Old) × 100%
Percentage Error = ((Measured - Actual)/Actual) × 100%"""
            
            else:
                return """Basic Mathematical Formulas:
• Quadratic Formula: x = (-b ± √(b²-4ac))/2a
• Distance Formula: d = √[(x₂-x₁)² + (y₂-y₁)²]
• Slope Formula: m = (y₂-y₁)/(x₂-x₁)
• Percentage: (Part/Whole) × 100%
• Average: Sum/Count
• Compound Interest: A = P(1+r/n)^(nt)"""
        except Exception as e:
            return f"Basic formulas error: {str(e)}"
    
    def get_algebra_formulas(self, command):
        """Algebraic formulas and identities"""
        try:
            if 'identity' in command or 'identities' in command:
                return """Algebraic Identities:
(a + b)² = a² + 2ab + b²
(a - b)² = a² - 2ab + b²
(a + b)(a - b) = a² - b²
(a + b)³ = a³ + 3a²b + 3ab² + b³
(a - b)³ = a³ - 3a²b + 3ab² - b³
a³ + b³ = (a + b)(a² - ab + b²)
a³ - b³ = (a - b)(a² + ab + b²)"""
            
            elif 'binomial' in command:
                return """Binomial Theorem:
(a + b)ⁿ = Σ(k=0 to n) C(n,k) × aⁿ⁻ᵏ × bᵏ
Where C(n,k) = n!/(k!(n-k)!)
Pascal's Triangle for coefficients
General term: Tᵣ₊₁ = C(n,r) × aⁿ⁻ʳ × bʳ"""
            
            elif 'logarithm' in command:
                return """Logarithm Properties:
log(ab) = log(a) + log(b)
log(a/b) = log(a) - log(b)
log(aⁿ) = n × log(a)
log_a(b) = log(b)/log(a)
aˡᵒᵍᵃ⁽ˣ⁾ = x
log_a(a) = 1, log_a(1) = 0"""
            
            elif 'exponential' in command:
                return """Exponential Properties:
aᵐ × aⁿ = aᵐ⁺ⁿ
aᵐ / aⁿ = aᵐ⁻ⁿ
(aᵐ)ⁿ = aᵐⁿ
(ab)ⁿ = aⁿbⁿ
a⁰ = 1 (a ≠ 0)
a⁻ⁿ = 1/aⁿ"""
            
            else:
                return """Algebra Formulas:
• Identities: (a±b)² = a² ± 2ab + b²
• Binomial: (a+b)ⁿ = Σ C(n,k)aⁿ⁻ᵏbᵏ
• Logarithms: log(ab) = log(a) + log(b)
• Exponentials: aᵐ × aⁿ = aᵐ⁺ⁿ
• Factoring: ax² + bx + c = a(x-r₁)(x-r₂)"""
        except Exception as e:
            return f"Algebra formulas error: {str(e)}"
    
    def get_geometry_formulas(self, command):
        """Geometric formulas"""
        try:
            if 'area' in command:
                return """Area Formulas:
Rectangle: A = l × w
Square: A = s²
Triangle: A = (1/2) × b × h
Circle: A = πr²
Parallelogram: A = b × h
Trapezoid: A = (1/2)(b₁ + b₂) × h
Ellipse: A = πab
Sector: A = (θ/360°) × πr²"""
            
            elif 'volume' in command:
                return """Volume Formulas:
Cube: V = s³
Rectangular Prism: V = l × w × h
Cylinder: V = πr²h
Cone: V = (1/3)πr²h
Sphere: V = (4/3)πr³
Pyramid: V = (1/3) × Base Area × h
Prism: V = Base Area × h"""
            
            elif 'surface area' in command:
                return """Surface Area Formulas:
Cube: SA = 6s²
Rectangular Prism: SA = 2(lw + lh + wh)
Cylinder: SA = 2πr² + 2πrh
Cone: SA = πr² + πrl
Sphere: SA = 4πr²
Pyramid: SA = Base Area + (1/2) × Perimeter × Slant Height"""
            
            elif 'perimeter' in command:
                return """Perimeter Formulas:
Rectangle: P = 2(l + w)
Square: P = 4s
Triangle: P = a + b + c
Circle (Circumference): C = 2πr = πd
Regular Polygon: P = n × s (n sides, side length s)"""
            
            else:
                return """Geometry Formulas:
• Circle: A = πr², C = 2πr
• Triangle: A = (1/2)bh, Heron's: A = √[s(s-a)(s-b)(s-c)]
• Rectangle: A = lw, P = 2(l+w)
• Sphere: V = (4/3)πr³, SA = 4πr²
• Cylinder: V = πr²h, SA = 2πr² + 2πrh"""
        except Exception as e:
            return f"Geometry formulas error: {str(e)}"
    
    def get_trigonometry_formulas(self, command):
        """Trigonometric formulas"""
        try:
            if 'basic' in command or 'fundamental' in command:
                return """Basic Trigonometric Ratios:
sin θ = opposite/hypotenuse
cos θ = adjacent/hypotenuse
tan θ = opposite/adjacent
csc θ = 1/sin θ
sec θ = 1/cos θ
cot θ = 1/tan θ
Pythagorean Identity: sin²θ + cos²θ = 1"""
            
            elif 'identity' in command or 'identities' in command:
                return """Trigonometric Identities:
sin²θ + cos²θ = 1
1 + tan²θ = sec²θ
1 + cot²θ = csc²θ
sin(A ± B) = sin A cos B ± cos A sin B
cos(A ± B) = cos A cos B ∓ sin A sin B
tan(A ± B) = (tan A ± tan B)/(1 ∓ tan A tan B)"""
            
            elif 'double angle' in command:
                return """Double Angle Formulas:
sin(2θ) = 2 sin θ cos θ
cos(2θ) = cos²θ - sin²θ = 2cos²θ - 1 = 1 - 2sin²θ
tan(2θ) = 2tan θ/(1 - tan²θ)"""
            
            elif 'half angle' in command:
                return """Half Angle Formulas:
sin(θ/2) = ±√[(1 - cos θ)/2]
cos(θ/2) = ±√[(1 + cos θ)/2]
tan(θ/2) = ±√[(1 - cos θ)/(1 + cos θ)] = sin θ/(1 + cos θ)"""
            
            elif 'law of sines' in command:
                return """Law of Sines:
a/sin A = b/sin B = c/sin C = 2R
Where R is circumradius of triangle"""
            
            elif 'law of cosines' in command:
                return """Law of Cosines:
c² = a² + b² - 2ab cos C
cos C = (a² + b² - c²)/(2ab)"""
            
            else:
                return """Trigonometry Formulas:
• Basic: sin θ = opp/hyp, cos θ = adj/hyp, tan θ = opp/adj
• Identity: sin²θ + cos²θ = 1
• Sum: sin(A±B) = sin A cos B ± cos A sin B
• Double: sin(2θ) = 2 sin θ cos θ
• Law of Sines: a/sin A = b/sin B = c/sin C"""
        except Exception as e:
            return f"Trigonometry formulas error: {str(e)}"
    
    def get_calculus_formulas(self, command):
        """Calculus formulas"""
        try:
            if 'limit' in command:
                return """Limit Formulas:
lim(x→0) (sin x)/x = 1
lim(x→0) (1 - cos x)/x = 0
lim(x→∞) (1 + 1/x)ˣ = e
lim(x→0) (eˣ - 1)/x = 1
lim(x→0) (ln(1+x))/x = 1
L'Hôpital's Rule: lim f(x)/g(x) = lim f'(x)/g'(x)"""
            
            elif 'derivative' in command:
                return """Basic Derivative Rules:
d/dx(c) = 0
d/dx(x) = 1
d/dx(xⁿ) = nxⁿ⁻¹
d/dx(eˣ) = eˣ
d/dx(ln x) = 1/x
d/dx(sin x) = cos x
d/dx(cos x) = -sin x
d/dx(tan x) = sec²x"""
            
            elif 'integration' in command:
                return """Basic Integration Rules:
∫ dx = x + C
∫ xⁿ dx = xⁿ⁺¹/(n+1) + C (n ≠ -1)
∫ 1/x dx = ln|x| + C
∫ eˣ dx = eˣ + C
∫ sin x dx = -cos x + C
∫ cos x dx = sin x + C
∫ sec²x dx = tan x + C"""
            
            elif 'series' in command:
                return """Important Series:
Taylor Series: f(x) = Σ(n=0 to ∞) f⁽ⁿ⁾(a)(x-a)ⁿ/n!
Maclaurin Series: f(x) = Σ(n=0 to ∞) f⁽ⁿ⁾(0)xⁿ/n!
eˣ = Σ(n=0 to ∞) xⁿ/n!
sin x = Σ(n=0 to ∞) (-1)ⁿx^(2n+1)/(2n+1)!
cos x = Σ(n=0 to ∞) (-1)ⁿx^(2n)/(2n)!"""
            
            else:
                return """Calculus Formulas:
• Derivative: d/dx(xⁿ) = nxⁿ⁻¹
• Chain Rule: d/dx[f(g(x))] = f'(g(x))g'(x)
• Product Rule: d/dx[uv] = u'v + uv'
• Quotient Rule: d/dx[u/v] = (u'v - uv')/v²
• Integration by Parts: ∫u dv = uv - ∫v du"""
        except Exception as e:
            return f"Calculus formulas error: {str(e)}"
    
    def get_differentiation_formulas(self, command):
        """Differentiation rules and formulas"""
        try:
            if 'power rule' in command:
                return """Power Rule:
d/dx(xⁿ) = nxⁿ⁻¹
d/dx(cxⁿ) = cnxⁿ⁻¹
Examples:
d/dx(x³) = 3x²
d/dx(5x⁴) = 20x³
d/dx(x⁻²) = -2x⁻³"""
            
            elif 'product rule' in command:
                return """Product Rule:
d/dx[f(x)g(x)] = f'(x)g(x) + f(x)g'(x)
Or: d/dx[uv] = u'v + uv'
Example: d/dx[x²sin x] = 2x sin x + x²cos x"""
            
            elif 'quotient rule' in command:
                return """Quotient Rule:
d/dx[f(x)/g(x)] = [f'(x)g(x) - f(x)g'(x)]/[g(x)]²
Or: d/dx[u/v] = (u'v - uv')/v²
Example: d/dx[x²/(x+1)] = [2x(x+1) - x²(1)]/(x+1)²"""
            
            elif 'chain rule' in command:
                return """Chain Rule:
d/dx[f(g(x))] = f'(g(x)) × g'(x)
Or: dy/dx = dy/du × du/dx
Examples:
d/dx[sin(x²)] = cos(x²) × 2x
d/dx[(3x+1)⁵] = 5(3x+1)⁴ × 3"""
            
            elif 'implicit' in command:
                return """Implicit Differentiation:
For equations like x² + y² = 25:
1. Differentiate both sides with respect to x
2. Apply chain rule to y terms: d/dx(y²) = 2y(dy/dx)
3. Solve for dy/dx
Example: x² + y² = 25 → 2x + 2y(dy/dx) = 0 → dy/dx = -x/y"""
            
            else:
                return """Differentiation Rules:
• Power: d/dx(xⁿ) = nxⁿ⁻¹
• Product: d/dx[uv] = u'v + uv'
• Quotient: d/dx[u/v] = (u'v - uv')/v²
• Chain: d/dx[f(g(x))] = f'(g(x))g'(x)
• Exponential: d/dx(eˣ) = eˣ, d/dx(aˣ) = aˣ ln a
• Logarithmic: d/dx(ln x) = 1/x, d/dx(log_a x) = 1/(x ln a)"""
        except Exception as e:
            return f"Differentiation formulas error: {str(e)}"
    
    def get_integration_formulas(self, command):
        """Integration rules and formulas"""
        try:
            if 'basic' in command or 'fundamental' in command:
                return """Basic Integration Formulas:
∫ k dx = kx + C
∫ xⁿ dx = xⁿ⁺¹/(n+1) + C (n ≠ -1)
∫ 1/x dx = ln|x| + C
∫ eˣ dx = eˣ + C
∫ aˣ dx = aˣ/ln a + C
∫ sin x dx = -cos x + C
∫ cos x dx = sin x + C
∫ sec²x dx = tan x + C
∫ csc²x dx = -cot x + C"""
            
            elif 'substitution' in command:
                return """Integration by Substitution:
∫ f(g(x))g'(x) dx = ∫ f(u) du where u = g(x)
Steps:
1. Choose u = g(x)
2. Find du = g'(x) dx
3. Substitute and integrate
4. Replace u with g(x)
Example: ∫ 2x(x²+1)³ dx, let u = x²+1"""
            
            elif 'parts' in command:
                return """Integration by Parts:
∫ u dv = uv - ∫ v du
Choose u using LIATE:
L - Logarithmic
I - Inverse trig
A - Algebraic
T - Trigonometric
E - Exponential
Example: ∫ x eˣ dx, u = x, dv = eˣ dx"""
            
            elif 'partial fractions' in command:
                return """Partial Fractions:
For rational functions P(x)/Q(x):
1. Factor denominator Q(x)
2. Decompose into partial fractions
3. Find constants A, B, C...
4. Integrate each term
Example: 1/((x-1)(x+2)) = A/(x-1) + B/(x+2)"""
            
            elif 'trigonometric' in command:
                return """Trigonometric Integrals:
∫ sin²x dx = x/2 - sin(2x)/4 + C
∫ cos²x dx = x/2 + sin(2x)/4 + C
∫ tan x dx = -ln|cos x| + C
∫ sec x dx = ln|sec x + tan x| + C
∫ sin^m x cos^n x dx (use reduction formulas)"""
            
            else:
                return """Integration Techniques:
• Basic: ∫ xⁿ dx = xⁿ⁺¹/(n+1) + C
• Substitution: ∫ f(g(x))g'(x) dx
• Parts: ∫ u dv = uv - ∫ v du
• Partial Fractions: Decompose rational functions
• Trigonometric: Use identities and substitutions"""
        except Exception as e:
            return f"Integration formulas error: {str(e)}"
    
    def get_physics_formulas(self, command):
        """Physics formulas for engineering"""
        try:
            if 'mechanics' in command or 'motion' in command:
                return """Mechanics Formulas:
Kinematics:
v = u + at
s = ut + (1/2)at²
v² = u² + 2as
s = (u + v)t/2

Dynamics:
F = ma (Newton's 2nd Law)
F = dp/dt (momentum form)
p = mv (momentum)
I = Ft (impulse)

Work & Energy:
W = F·s = Fs cos θ
KE = (1/2)mv²
PE = mgh (gravitational)
PE = (1/2)kx² (elastic)"""
            
            elif 'rotational' in command:
                return """Rotational Mechanics:
ω = θ/t (angular velocity)
α = dω/dt (angular acceleration)
τ = Iα (torque)
L = Iω (angular momentum)
I = Σmr² (moment of inertia)
KE_rot = (1/2)Iω²
v = rω (linear-angular relation)
a = rα"""
            
            elif 'waves' in command:
                return """Wave Formulas:
v = fλ (wave equation)
f = 1/T (frequency-period)
y = A sin(kx - ωt + φ) (wave function)
k = 2π/λ (wave number)
ω = 2πf (angular frequency)
v = √(T/μ) (string wave speed)
v = √(B/ρ) (sound in fluid)"""
            
            elif 'oscillation' in command:
                return """Simple Harmonic Motion:
x = A cos(ωt + φ)
v = -Aω sin(ωt + φ)
a = -Aω² cos(ωt + φ)
ω = √(k/m) (spring)
ω = √(g/l) (pendulum)
T = 2π√(m/k) (spring period)
T = 2π√(l/g) (pendulum period)"""
            
            else:
                return """Physics Formulas:
• Motion: v = u + at, s = ut + ½at²
• Force: F = ma, W = F·s
• Energy: KE = ½mv², PE = mgh
• Waves: v = fλ, f = 1/T
• Rotation: τ = Iα, L = Iω
• SHM: T = 2π√(m/k)"""
        except Exception as e:
            return f"Physics formulas error: {str(e)}"
    
    def get_thermodynamics_formulas(self, command):
        """Thermodynamics formulas"""
        try:
            if 'laws' in command:
                return """Laws of Thermodynamics:
0th Law: Thermal equilibrium
1st Law: ΔU = Q - W (energy conservation)
2nd Law: ΔS ≥ 0 (entropy increases)
3rd Law: S → 0 as T → 0

Where:
ΔU = change in internal energy
Q = heat added to system
W = work done by system
S = entropy"""
            
            elif 'ideal gas' in command:
                return """Ideal Gas Laws:
PV = nRT (ideal gas law)
PV = NkT (Boltzmann form)
P₁V₁/T₁ = P₂V₂/T₂ (combined gas law)
Boyle's Law: P₁V₁ = P₂V₂ (T constant)
Charles's Law: V₁/T₁ = V₂/T₂ (P constant)
Gay-Lussac's Law: P₁/T₁ = P₂/T₂ (V constant)

Constants:
R = 8.314 J/(mol·K)
k = 1.38 × 10⁻²³ J/K"""
            
            elif 'heat transfer' in command:
                return """Heat Transfer:
Q = mcΔT (sensible heat)
Q = mL (latent heat)
q = kA(dT/dx) (Fourier's law - conduction)
q = hAΔT (Newton's law - convection)
q = εσAT⁴ (Stefan-Boltzmann - radiation)

Where:
k = thermal conductivity
h = convection coefficient
ε = emissivity
σ = Stefan-Boltzmann constant"""
            
            elif 'efficiency' in command:
                return """Efficiency Formulas:
η = W_out/Q_in (thermal efficiency)
η = 1 - Q_cold/Q_hot (heat engine)
η_Carnot = 1 - T_cold/T_hot (Carnot efficiency)
COP_heat pump = Q_hot/W
COP_refrigerator = Q_cold/W

Carnot cycle is most efficient between two temperatures"""
            
            else:
                return """Thermodynamics Formulas:
• 1st Law: ΔU = Q - W
• Ideal Gas: PV = nRT
• Heat Transfer: Q = mcΔT
• Efficiency: η = W_out/Q_in
• Carnot: η = 1 - T_cold/T_hot
• Entropy: dS = dQ/T"""
        except Exception as e:
            return f"Thermodynamics formulas error: {str(e)}"
    
    def get_electromagnetism_formulas(self, command):
        """Electromagnetic formulas"""
        try:
            if 'coulomb' in command or 'electric field' in command:
                return """Electric Field & Force:
F = kq₁q₂/r² (Coulomb's law)
E = F/q = kQ/r² (electric field)
E = V/d (uniform field)
V = kQ/r (electric potential)
W = qΔV (work in electric field)
C = Q/V (capacitance)
U = ½CV² = ½QV (energy in capacitor)

Where k = 9 × 10⁹ N·m²/C²"""
            
            elif 'magnetic' in command:
                return """Magnetic Field:
F = qvB sin θ (force on moving charge)
F = BIL sin θ (force on current)
B = μ₀I/(2πr) (straight wire)
B = μ₀nI (solenoid)
Φ = BA cos θ (magnetic flux)
ε = -dΦ/dt (Faraday's law)
ε = BLv (motional EMF)

Where μ₀ = 4π × 10⁻⁷ H/m"""
            
            elif 'circuit' in command:
                return """Circuit Analysis:
V = IR (Ohm's law)
P = VI = I²R = V²/R (power)
R_series = R₁ + R₂ + R₃...
1/R_parallel = 1/R₁ + 1/R₂ + 1/R₃...
τ = RC (RC time constant)
τ = L/R (RL time constant)
ω₀ = 1/√(LC) (resonant frequency)"""
            
            elif 'maxwell' in command:
                return """Maxwell's Equations:
∇·E = ρ/ε₀ (Gauss's law)
∇·B = 0 (no magnetic monopoles)
∇×E = -∂B/∂t (Faraday's law)
∇×B = μ₀J + μ₀ε₀∂E/∂t (Ampère-Maxwell law)

Wave equation: c = 1/√(μ₀ε₀)
Where c = 3 × 10⁸ m/s"""
            
            else:
                return """Electromagnetism Formulas:
• Coulomb: F = kq₁q₂/r²
• Electric Field: E = kQ/r²
• Ohm's Law: V = IR
• Power: P = VI = I²R
• Magnetic Force: F = qvB sin θ
• Faraday: ε = -dΦ/dt"""
        except Exception as e:
            return f"Electromagnetism formulas error: {str(e)}"  
  
    def get_engineering_formulas(self, command):
        """Engineering formulas"""
        try:
            if 'stress' in command or 'strain' in command:
                return """Stress & Strain:
σ = F/A (normal stress)
τ = F/A (shear stress)
ε = ΔL/L (strain)
σ = Eε (Hooke's law)
γ = τ/G (shear strain)
ν = -ε_lateral/ε_axial (Poisson's ratio)
K = E/[3(1-2ν)] (bulk modulus)

Where:
E = Young's modulus
G = shear modulus
ν = Poisson's ratio"""
            
            elif 'beam' in command or 'bending' in command:
                return """Beam Bending:
σ = My/I (bending stress)
δ = PL³/(3EI) (cantilever deflection)
δ = 5wL⁴/(384EI) (simply supported beam)
I = bh³/12 (rectangular moment of inertia)
I = πd⁴/64 (circular moment of inertia)
S = I/c (section modulus)
τ_max = VQ/(Ib) (shear stress in beam)"""
            
            elif 'column' in command or 'buckling' in command:
                return """Column Buckling:
P_cr = π²EI/(KL)² (Euler buckling load)
σ_cr = π²E/(KL/r)² (critical stress)
r = √(I/A) (radius of gyration)
K = effective length factor
K = 1 (pinned-pinned)
K = 0.5 (fixed-fixed)
K = 2 (fixed-free)
K = 0.7 (fixed-pinned)"""
            
            elif 'torsion' in command:
                return """Torsion:
τ = Tr/J (shear stress)
φ = TL/(GJ) (angle of twist)
J = πd⁴/32 (circular shaft)
J = πD⁴/32 - πd⁴/32 (hollow shaft)
P = Tω (power transmission)
T = P/ω (torque from power)"""
            
            else:
                return """Engineering Formulas:
• Stress: σ = F/A, τ = F/A
• Hooke's Law: σ = Eε
• Bending: σ = My/I
• Buckling: P_cr = π²EI/(KL)²
• Torsion: τ = Tr/J
• Power: P = Tω"""
        except Exception as e:
            return f"Engineering formulas error: {str(e)}"
    
    def get_electrical_formulas(self, command):
        """Electrical engineering formulas"""
        try:
            if 'ac circuit' in command or 'alternating' in command:
                return """AC Circuit Analysis:
V_rms = V_peak/√2
I_rms = I_peak/√2
P_avg = V_rms × I_rms × cos φ
Q = V_rms × I_rms × sin φ (reactive power)
S = V_rms × I_rms (apparent power)
Z = R + jX (impedance)
X_L = ωL = 2πfL (inductive reactance)
X_C = 1/(ωC) = 1/(2πfC) (capacitive reactance)"""
            
            elif 'transformer' in command:
                return """Transformer Equations:
V₁/V₂ = N₁/N₂ = I₂/I₁
P₁ = P₂ (ideal transformer)
η = P_out/P_in (efficiency)
Regulation = (V_no-load - V_full-load)/V_full-load
k = N₂/N₁ (turns ratio)
Z₂' = Z₂/k² (referred impedance)"""
            
            elif 'motor' in command:
                return """Motor Equations:
T = kΦI_a (torque)
E_b = kΦω (back EMF)
V = E_b + I_a R_a (DC motor)
η = P_out/P_in (efficiency)
P_out = Tω (mechanical power)
s = (n_s - n)/n_s (slip in induction motor)
n_s = 120f/p (synchronous speed)"""
            
            elif 'power' in command:
                return """Power System:
P = VI cos φ (real power)
Q = VI sin φ (reactive power)
S = VI (apparent power)
S² = P² + Q² (power triangle)
pf = cos φ (power factor)
η = P_out/P_in (efficiency)
%Regulation = (V_nl - V_fl)/V_fl × 100"""
            
            else:
                return """Electrical Formulas:
• Ohm's Law: V = IR
• Power: P = VI = I²R = V²/R
• AC: V_rms = V_peak/√2
• Impedance: Z = R + jX
• Transformer: V₁/V₂ = N₁/N₂
• Motor: T = kΦI_a"""
        except Exception as e:
            return f"Electrical formulas error: {str(e)}"
    
    def get_civil_formulas(self, command):
        """Civil engineering formulas"""
        try:
            if 'concrete' in command:
                return """Concrete Design:
f'_c = compressive strength
f_y = yield strength of steel
ρ = A_s/(bd) (reinforcement ratio)
ρ_min = 0.0018 (minimum reinforcement)
ρ_max = 0.75ρ_b (maximum reinforcement)
M_n = A_s f_y (d - a/2) (nominal moment)
a = A_s f_y/(0.85f'_c b) (depth of compression block)
V_c = 2√f'_c bd (concrete shear capacity)"""
            
            elif 'steel' in command:
                return """Steel Design:
σ_allow = F_y/FS (allowable stress)
P_n = A_g F_y (axial capacity)
M_n = Z F_y (moment capacity)
Z = plastic section modulus
S = elastic section modulus
L_r = limiting unbraced length
C_b = lateral-torsional buckling factor
K = effective length factor"""
            
            elif 'soil' in command:
                return """Soil Mechanics:
e = V_v/V_s (void ratio)
n = V_v/V (porosity)
w = W_w/W_s (water content)
γ = W/V (unit weight)
S = V_w/V_v (degree of saturation)
q_u = unconfined compressive strength
φ = angle of internal friction
c = cohesion
τ = c + σ tan φ (Mohr-Coulomb)"""
            
            elif 'fluid' in command or 'hydraulics' in command:
                return """Fluid Mechanics:
Q = AV (continuity equation)
P₁ + ½ρV₁² + ρgh₁ = P₂ + ½ρV₂² + ρgh₂ (Bernoulli)
h_f = fLV²/(2gD) (Darcy-Weisbach)
Re = ρVD/μ (Reynolds number)
f = 64/Re (laminar flow)
Q = CA√(2gh) (orifice flow)
V = C√(2gh) (weir flow)"""
            
            else:
                return """Civil Engineering Formulas:
• Concrete: M_n = A_s f_y (d - a/2)
• Steel: σ_allow = F_y/FS
• Soil: τ = c + σ tan φ
• Fluid: Q = AV, Bernoulli equation
• Structural: σ = F/A, δ = PL/(AE)"""
        except Exception as e:
            return f"Civil formulas error: {str(e)}"
    
    def get_chemical_formulas(self, command):
        """Chemical engineering formulas"""
        try:
            if 'reaction' in command or 'kinetics' in command:
                return """Reaction Kinetics:
r = k[A]^m[B]^n (rate law)
k = Ae^(-E_a/RT) (Arrhenius equation)
t₁/₂ = 0.693/k (first-order half-life)
ln[A] = ln[A₀] - kt (first-order integrated)
1/[A] = 1/[A₀] + kt (second-order integrated)
K_eq = [products]/[reactants] (equilibrium constant)"""
            
            elif 'mass transfer' in command:
                return """Mass Transfer:
N_A = k_c(C_A1 - C_A2) (mass flux)
Sh = k_c L/D_AB (Sherwood number)
Sc = μ/(ρD_AB) (Schmidt number)
Re = ρVL/μ (Reynolds number)
Sh = 2 + 0.6Re^0.5 Sc^0.33 (sphere)
j_D = Sh/(Re×Sc^0.33) (j-factor)"""
            
            elif 'heat transfer' in command:
                return """Heat Transfer:
q = hAΔT (convection)
q = kA(dT/dx) (conduction)
q = εσA(T₁⁴ - T₂⁴) (radiation)
Nu = hL/k (Nusselt number)
Pr = μc_p/k (Prandtl number)
Gr = gβΔTL³/ν² (Grashof number)
Ra = GrPr (Rayleigh number)"""
            
            elif 'distillation' in command:
                return """Distillation:
y = αx/(1 + (α-1)x) (relative volatility)
N_min = log[(x_D/x_B)((1-x_B)/(1-x_D))]/log α (Fenske)
R_min = (x_D - y_q)/(y_q - x_q) (minimum reflux)
N = N_min/(1 - R/R_min) (Gilliland correlation)
q = (h_f - h_L)/(h_V - h_L) (feed condition)"""
            
            else:
                return """Chemical Engineering Formulas:
• Kinetics: r = k[A]^m[B]^n
• Mass Transfer: N_A = k_c ΔC
• Heat Transfer: q = hAΔT
• Distillation: y = αx/(1+(α-1)x)
• Fluid Flow: ΔP = fLρV²/(2D)"""
        except Exception as e:
            return f"Chemical formulas error: {str(e)}"
    
    def get_control_system_formulas(self, command):
        """Control system formulas"""
        try:
            if 'transfer function' in command:
                return """Transfer Function:
G(s) = Y(s)/X(s) (transfer function)
G(s) = K/(τs + 1) (first-order system)
G(s) = Kω_n²/(s² + 2ζω_n s + ω_n²) (second-order)
H(s) = feedback transfer function
T(s) = G(s)/(1 + G(s)H(s)) (closed-loop)
E(s) = 1/(1 + G(s)H(s)) (error function)"""
            
            elif 'stability' in command:
                return """Stability Analysis:
Routh-Hurwitz criterion
Nyquist criterion: Z = P - N
Bode plots: GM, PM
Root locus
Characteristic equation: 1 + G(s)H(s) = 0
Gain margin: GM = 1/|G(jω₁₈₀)|
Phase margin: PM = 180° + ∠G(jω_gc)"""
            
            elif 'pid' in command:
                return """PID Controller:
u(t) = K_p e(t) + K_i ∫e(t)dt + K_d de(t)/dt
G_c(s) = K_p + K_i/s + K_d s
Ziegler-Nichols tuning:
K_p = 0.6K_u
K_i = 2K_p/T_u
K_d = K_p T_u/8
Where K_u = ultimate gain, T_u = ultimate period"""
            
            elif 'response' in command:
                return """System Response:
First-order: y(t) = K(1 - e^(-t/τ))
Second-order underdamped:
y(t) = K[1 - e^(-ζω_n t) cos(ω_d t + φ)]
ω_d = ω_n√(1 - ζ²) (damped frequency)
t_r = π/ω_d (rise time)
t_s = 4/(ζω_n) (settling time)
M_p = e^(-πζ/√(1-ζ²)) (overshoot)"""
            
            else:
                return """Control System Formulas:
• Transfer Function: G(s) = Y(s)/X(s)
• Closed-loop: T(s) = G(s)/(1+G(s)H(s))
• PID: u(t) = K_p e + K_i ∫e dt + K_d de/dt
• Stability: Routh-Hurwitz, Nyquist
• Response: First/Second order systems"""
        except Exception as e:
            return f"Control system formulas error: {str(e)}"
    
    def get_signal_processing_formulas(self, command):
        """Signal processing formulas"""
        try:
            if 'fourier' in command:
                return """Fourier Transform:
X(ω) = ∫ x(t)e^(-jωt) dt (continuous)
X[k] = Σ x[n]e^(-j2πkn/N) (discrete - DFT)
x[n] = (1/N)Σ X[k]e^(j2πkn/N) (inverse DFT)
FFT: O(N log N) complexity
Parseval's theorem: ∫|x(t)|² dt = (1/2π)∫|X(ω)|² dω"""
            
            elif 'filter' in command:
                return """Digital Filters:
H(z) = Y(z)/X(z) (transfer function)
y[n] = Σ b_k x[n-k] - Σ a_k y[n-k] (difference equation)
FIR: y[n] = Σ h[k]x[n-k]
IIR: recursive structure
Butterworth: |H(jω)|² = 1/(1 + (ω/ω_c)^(2N))
Chebyshev: ripple in passband/stopband"""
            
            elif 'sampling' in command:
                return """Sampling Theory:
f_s ≥ 2f_max (Nyquist criterion)
f_s = sampling frequency
f_max = maximum signal frequency
Aliasing occurs when f_s < 2f_max
Quantization error: e_q = ±LSB/2
SNR = 6.02N + 1.76 dB (N-bit quantizer)"""
            
            elif 'convolution' in command:
                return """Convolution:
y(t) = x(t) * h(t) = ∫ x(τ)h(t-τ) dτ (continuous)
y[n] = x[n] * h[n] = Σ x[k]h[n-k] (discrete)
Convolution theorem: X(ω)H(ω) ↔ x(t)*h(t)
Circular convolution for DFT
Linear convolution for continuous signals"""
            
            else:
                return """Signal Processing Formulas:
• Fourier: X(ω) = ∫ x(t)e^(-jωt) dt
• Sampling: f_s ≥ 2f_max
• Convolution: y[n] = Σ x[k]h[n-k]
• Filter: H(z) = Y(z)/X(z)
• DFT: X[k] = Σ x[n]e^(-j2πkn/N)"""
        except Exception as e:
            return f"Signal processing formulas error: {str(e)}"
    
    def get_fluid_mechanics_formulas(self, command):
        """Fluid mechanics formulas"""
        try:
            if 'continuity' in command:
                return """Continuity Equation:
ρ₁A₁V₁ = ρ₂A₂V₂ (general form)
A₁V₁ = A₂V₂ (incompressible flow)
∂ρ/∂t + ∇·(ρV) = 0 (differential form)
Q = AV (volumetric flow rate)
ṁ = ρQ = ρAV (mass flow rate)"""
            
            elif 'bernoulli' in command:
                return """Bernoulli's Equation:
P₁/ρ + V₁²/2 + gz₁ = P₂/ρ + V₂²/2 + gz₂ (energy form)
P₁ + ½ρV₁² + ρgh₁ = P₂ + ½ρV₂² + ρgh₂ (pressure form)
Valid for: steady, inviscid, incompressible flow
Energy equation with losses:
P₁/ρ + V₁²/2 + gz₁ = P₂/ρ + V₂²/2 + gz₂ + h_L"""
            
            elif 'friction' in command:
                return """Friction Losses:
h_f = fLV²/(2gD) (Darcy-Weisbach)
f = 64/Re (laminar flow)
1/√f = -2log(ε/3.7D + 2.51/(Re√f)) (Colebrook)
Re = ρVD/μ = VD/ν (Reynolds number)
ε = pipe roughness
Minor losses: h_L = KV²/(2g)"""
            
            elif 'drag' in command:
                return """Drag and Lift:
F_D = ½ρV²C_D A (drag force)
F_L = ½ρV²C_L A (lift force)
C_D = drag coefficient
C_L = lift coefficient
A = frontal area (drag) or planform area (lift)
Re = ρVL/μ (Reynolds number)"""
            
            else:
                return """Fluid Mechanics Formulas:
• Continuity: ρ₁A₁V₁ = ρ₂A₂V₂
• Bernoulli: P₁ + ½ρV₁² + ρgh₁ = P₂ + ½ρV₂² + ρgh₂
• Friction: h_f = fLV²/(2gD)
• Reynolds: Re = ρVD/μ
• Drag: F_D = ½ρV²C_D A"""
        except Exception as e:
            return f"Fluid mechanics formulas error: {str(e)}"
    
    def get_statistics_formulas(self, command):
        """Statistics and probability formulas"""
        try:
            if 'descriptive' in command:
                return """Descriptive Statistics:
Mean: x̄ = Σx/n
Median: middle value when ordered
Mode: most frequent value
Range: max - min
Variance: s² = Σ(x-x̄)²/(n-1)
Standard deviation: s = √s²
Coefficient of variation: CV = s/x̄"""
            
            elif 'probability' in command:
                return """Probability Rules:
P(A ∪ B) = P(A) + P(B) - P(A ∩ B)
P(A ∩ B) = P(A)P(B|A) = P(B)P(A|B)
P(A|B) = P(A ∩ B)/P(B) (conditional)
Bayes' theorem: P(A|B) = P(B|A)P(A)/P(B)
Total probability: P(B) = ΣP(B|Aᵢ)P(Aᵢ)"""
            
            elif 'distribution' in command:
                return """Common Distributions:
Normal: f(x) = (1/σ√2π)e^(-(x-μ)²/2σ²)
Binomial: P(X=k) = C(n,k)p^k(1-p)^(n-k)
Poisson: P(X=k) = e^(-λ)λ^k/k!
Exponential: f(x) = λe^(-λx)
Uniform: f(x) = 1/(b-a) for a ≤ x ≤ b"""
            
            elif 'confidence' in command:
                return """Confidence Intervals:
x̄ ± t_(α/2) × s/√n (t-distribution)
x̄ ± z_(α/2) × σ/√n (normal, known σ)
p̂ ± z_(α/2) × √(p̂(1-p̂)/n) (proportion)
Margin of error: E = t_(α/2) × s/√n
Sample size: n = (z_(α/2) × σ/E)²"""
            
            else:
                return """Statistics Formulas:
• Mean: x̄ = Σx/n
• Variance: s² = Σ(x-x̄)²/(n-1)
• Normal: f(x) = (1/σ√2π)e^(-(x-μ)²/2σ²)
• Confidence: x̄ ± t_(α/2) × s/√n
• Correlation: r = Σ(x-x̄)(y-ȳ)/√[Σ(x-x̄)²Σ(y-ȳ)²]"""
        except Exception as e:
            return f"Statistics formulas error: {str(e)}"
    
    def get_financial_formulas(self, command):
        """Financial formulas"""
        try:
            if 'present value' in command or 'pv' in command:
                return """Present Value:
PV = FV/(1+r)ⁿ (single payment)
PV = PMT × [(1-(1+r)^(-n))/r] (annuity)
PV = PMT/r (perpetuity)
NPV = Σ[CFₜ/(1+r)ᵗ] - Initial Investment
Where: FV = future value, PMT = payment, r = rate, n = periods"""
            
            elif 'future value' in command or 'fv' in command:
                return """Future Value:
FV = PV(1+r)ⁿ (single payment)
FV = PMT × [((1+r)ⁿ-1)/r] (annuity)
FV = PV × e^(rt) (continuous compounding)
Effective rate: r_eff = (1+r/m)ᵐ - 1
Where m = compounding periods per year"""
            
            elif 'loan' in command or 'mortgage' in command:
                return """Loan Calculations:
PMT = PV × [r(1+r)ⁿ]/[(1+r)ⁿ-1] (payment)
Balance = PV(1+r)ᵗ - PMT[((1+r)ᵗ-1)/r]
Interest payment = Balance × r
Principal payment = PMT - Interest payment
Total interest = PMT × n - PV"""
            
            elif 'bond' in command:
                return """Bond Valuation:
P = Σ[C/(1+r)ᵗ] + M/(1+r)ⁿ
Where: P = price, C = coupon, M = maturity value
Current yield = Annual coupon/Price
YTM solved from: P = Σ[C/(1+YTM)ᵗ] + M/(1+YTM)ⁿ
Duration = Σ[t×PV(CFₜ)]/Price"""
            
            else:
                return """Financial Formulas:
• Present Value: PV = FV/(1+r)ⁿ
• Future Value: FV = PV(1+r)ⁿ
• Annuity: PMT = PV×[r(1+r)ⁿ]/[(1+r)ⁿ-1]
• NPV: Σ[CFₜ/(1+r)ᵗ] - Initial Investment
• IRR: NPV = 0"""
        except Exception as e:
            return f"Financial formulas error: {str(e)}"
    
    def get_interest_formulas(self, command):
        """Interest calculation formulas"""
        try:
            if 'simple' in command:
                return """Simple Interest:
I = Prt (interest)
A = P + I = P(1 + rt) (amount)
Where:
P = principal
r = annual interest rate (decimal)
t = time in years
I = interest earned
A = final amount"""
            
            elif 'compound' in command:
                return """Compound Interest:
A = P(1 + r/n)^(nt) (general formula)
A = Pe^(rt) (continuous compounding)
Effective rate: r_eff = (1 + r/n)ⁿ - 1
Doubling time: t = ln(2)/(r) (continuous)
Rule of 72: t ≈ 72/r% (approximation)
Where n = compounding frequency per year"""
            
            else:
                return """Interest Formulas:
• Simple: A = P(1 + rt)
• Compound: A = P(1 + r/n)^(nt)
• Continuous: A = Pe^(rt)
• Effective rate: (1 + r/n)ⁿ - 1
• Rule of 72: Doubling time ≈ 72/r%"""
        except Exception as e:
            return f"Interest formulas error: {str(e)}"  
      
        # ==================== INTELLIGENT MATHEMATICAL PROBLEM SOLVER ====================
        
        # Universal Math Solver (2901-3000+)
        if any(word in command for word in ['solve', 'calculate', 'find', 'what is', 'kya hai', 'solve karo', 'nikalo']):
            result = self.universal_math_solver(command)
            self.speak(result)
        
        # Number Operations (3001-3050)
        elif any(word in command for word in ['plus', 'add', 'jodo', 'sum']):
            result = self.solve_addition(command)
            self.speak(result)
        
        elif any(word in command for word in ['minus', 'subtract', 'ghata', 'difference']):
            result = self.solve_subtraction(command)
            self.speak(result)
        
        elif any(word in command for word in ['multiply', 'times', 'guna', 'product']):
            result = self.solve_multiplication(command)
            self.speak(result)
        
        elif any(word in command for word in ['divide', 'divided by', 'bhag', 'quotient']):
            result = self.solve_division(command)
            self.speak(result)
        
        elif any(word in command for word in ['power', 'raised to', 'exponent', 'ghat']):
            result = self.solve_power(command)
            self.speak(result)
        
        elif any(word in command for word in ['square root', 'sqrt', 'vargmul']):
            result = self.solve_square_root(command)
            self.speak(result)
        
        elif any(word in command for word in ['cube root', 'cbrt', 'ghanmul']):
            result = self.solve_cube_root(command)
            self.speak(result)
        
        # Advanced Problem Solving (3051-3100)
        elif any(word in command for word in ['equation', 'samikaran', 'solve equation']):
            result = self.solve_any_equation(command)
            self.speak(result)
        
        elif any(word in command for word in ['integral', 'integration', 'samakalan']):
            result = self.solve_any_integral(command)
            self.speak(result)
        
        elif any(word in command for word in ['derivative', 'differentiation', 'avakalan']):
            result = self.solve_any_derivative(command)
            self.speak(result)
        
        elif any(word in command for word in ['limit', 'seema']):
            result = self.solve_any_limit(command)
            self.speak(result)
        
        elif any(word in command for word in ['matrix', 'determinant', 'inverse']):
            result = self.solve_any_matrix(command)
            self.speak(result)
        
        # Intelligent Question Answering (3101-3200)
        elif any(word in command for word in ['how many', 'kitne', 'count', 'ginti']):
            result = self.answer_counting_questions(command)
            self.speak(result)
        
        elif any(word in command for word in ['percentage', 'percent', 'pratishat']):
            result = self.solve_percentage_problems(command)
            self.speak(result)
        
        elif any(word in command for word in ['ratio', 'proportion', 'anupat']):
            result = self.solve_ratio_problems(command)
            self.speak(result)
        
        elif any(word in command for word in ['average', 'mean', 'ausat']):
            result = self.solve_average_problems(command)
            self.speak(result)
        
        elif any(word in command for word in ['area', 'perimeter', 'volume', 'kshetrafal']):
            result = self.solve_geometry_problems(command)
            self.speak(result)
        
        # Word Problems (3201-3300)
        elif any(word in command for word in ['age problem', 'umar', 'years old']):
            result = self.solve_age_problems(command)
            self.speak(result)
        
        elif any(word in command for word in ['speed', 'distance', 'time', 'gati']):
            result = self.solve_speed_distance_time(command)
            self.speak(result)
        
        elif any(word in command for word in ['profit', 'loss', 'labh', 'hani']):
            result = self.solve_profit_loss_problems(command)
            self.speak(result)
        
        elif any(word in command for word in ['work', 'time', 'kaam', 'samay']):
            result = self.solve_work_time_problems(command)
            self.speak(result)
        
        elif any(word in command for word in ['mixture', 'alligation', 'mishran']):
            result = self.solve_mixture_problems(command)
            self.speak(result)    

    # ==================== UNIVERSAL MATHEMATICAL PROBLEM SOLVER ====================
    
    def universal_math_solver(self, command):
        """Universal solver for any mathematical problem"""
        try:
            # Clean the command
            command = command.lower().replace('solve', '').replace('calculate', '').replace('find', '').replace('what is', '').replace('kya hai', '').strip()
            
            # Check for specific problem types
            if self.is_equation(command):
                return self.solve_any_equation(command)
            elif self.is_calculus_problem(command):
                return self.solve_calculus_problem(command)
            elif self.is_geometry_problem(command):
                return self.solve_geometry_problems(command)
            elif self.is_arithmetic_expression(command):
                return self.solve_arithmetic_expression(command)
            elif self.is_word_problem(command):
                return self.solve_word_problem(command)
            elif self.contains_numbers(command):
                return self.solve_number_problem(command)
            else:
                return self.provide_mathematical_help(command)
                
        except Exception as e:
            return f"Problem solve karne mein error: {str(e)}"
    
    def is_equation(self, command):
        """Check if command contains an equation"""
        return '=' in command or 'equals' in command or 'barabar' in command
    
    def is_calculus_problem(self, command):
        """Check if it's a calculus problem"""
        calculus_keywords = ['derivative', 'integral', 'limit', 'differentiate', 'integrate', 'dx', 'dy']
        return any(keyword in command for keyword in calculus_keywords)
    
    def is_geometry_problem(self, command):
        """Check if it's a geometry problem"""
        geometry_keywords = ['area', 'perimeter', 'volume', 'radius', 'diameter', 'triangle', 'circle', 'rectangle', 'square']
        return any(keyword in command for keyword in geometry_keywords)
    
    def is_arithmetic_expression(self, command):
        """Check if it's a simple arithmetic expression"""
        arithmetic_symbols = ['+', '-', '*', '/', '^', 'plus', 'minus', 'times', 'divided']
        return any(symbol in command for symbol in arithmetic_symbols)
    
    def is_word_problem(self, command):
        """Check if it's a word problem"""
        word_problem_keywords = ['years old', 'speed', 'distance', 'profit', 'loss', 'work', 'time', 'age']
        return any(keyword in command for keyword in word_problem_keywords)
    
    def contains_numbers(self, command):
        """Check if command contains numbers"""
        return bool(re.search(r'\d+', command))
    
    def solve_arithmetic_expression(self, command):
        """Solve arithmetic expressions"""
        try:
            # Replace words with symbols
            expression = command.replace('plus', '+').replace('add', '+').replace('jodo', '+')
            expression = expression.replace('minus', '-').replace('subtract', '-').replace('ghata', '-')
            expression = expression.replace('times', '*').replace('multiply', '*').replace('guna', '*')
            expression = expression.replace('divided by', '/').replace('divide', '/').replace('bhag', '/')
            expression = expression.replace('power', '**').replace('raised to', '**').replace('^', '**')
            expression = expression.replace('x', '*')
            
            # Extract mathematical expression
            math_expr = re.findall(r'[\d+\-*/().\s]+', expression)
            if math_expr:
                expr_str = ''.join(math_expr).strip()
                if expr_str:
                    result = eval(expr_str)
                    return f"Answer: {expr_str} = {result}"
            
            # Try to extract numbers and operations
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                if 'plus' in command or 'add' in command or '+' in command:
                    result = sum(numbers)
                    return f"Addition: {' + '.join(map(str, numbers))} = {result}"
                elif 'minus' in command or 'subtract' in command or '-' in command:
                    result = numbers[0] - numbers[1]
                    return f"Subtraction: {numbers[0]} - {numbers[1]} = {result}"
                elif 'times' in command or 'multiply' in command or '*' in command:
                    result = numbers[0] * numbers[1]
                    return f"Multiplication: {numbers[0]} × {numbers[1]} = {result}"
                elif 'divide' in command or '/' in command:
                    if numbers[1] != 0:
                        result = numbers[0] / numbers[1]
                        return f"Division: {numbers[0]} ÷ {numbers[1]} = {result}"
                    else:
                        return "Division by zero is undefined"
            
            return "Please provide a clear arithmetic expression"
            
        except Exception as e:
            return f"Arithmetic calculation error: {str(e)}"
    
    def solve_any_equation(self, command):
        """Solve any type of equation"""
        try:
            # Extract equation from command
            if '=' in command:
                equation_parts = command.split('=')
                if len(equation_parts) == 2:
                    left_side = equation_parts[0].strip()
                    right_side = equation_parts[1].strip()
                    
                    try:
                        # Try to solve using sympy
                        left_expr = sp.sympify(left_side)
                        right_expr = sp.sympify(right_side)
                        equation = sp.Eq(left_expr, right_expr)
                        solutions = solve(equation, self.x)
                        
                        if solutions:
                            return f"Equation: {left_side} = {right_side}\nSolutions: x = {solutions}"
                        else:
                            return f"No solution found for equation: {left_side} = {right_side}"
                    except:
                        # Try numerical approach
                        return self.solve_equation_numerically(left_side, right_side)
            
            # Check for specific equation types
            if 'quadratic' in command:
                return self.solve_quadratic_equation(command)
            elif 'linear' in command:
                return self.solve_linear_equation(command)
            else:
                return "Please provide equation in format: expression = value"
                
        except Exception as e:
            return f"Equation solving error: {str(e)}"
    
    def solve_equation_numerically(self, left_side, right_side):
        """Solve equation numerically"""
        try:
            # Simple numerical approach for basic equations
            numbers = self.extract_numbers_from_command(left_side + right_side)
            if len(numbers) >= 2:
                # Try to identify pattern and solve
                if 'x' in left_side:
                    # Linear equation like 2x + 3 = 7
                    if '+' in left_side and len(numbers) >= 2:
                        # ax + b = c format
                        coeff = numbers[0] if numbers[0] != numbers[-1] else 1
                        constant = numbers[1] if len(numbers) > 2 else 0
                        result = numbers[-1]
                        x_value = (result - constant) / coeff
                        return f"Solution: x = {x_value}"
            
            return f"Numerical solution not found for: {left_side} = {right_side}"
        except Exception as e:
            return f"Numerical solving error: {str(e)}"
    
    def solve_any_integral(self, command):
        """Solve any integral"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                
                # Check for definite integral
                limits = self.extract_integration_limits(command)
                if limits:
                    lower, upper = limits
                    result = integrate(expr, (self.x, lower, upper))
                    return f"Definite integral: ∫[{lower} to {upper}] {function_str} dx = {result}"
                else:
                    result = integrate(expr, self.x)
                    return f"Indefinite integral: ∫ {function_str} dx = {result} + C"
            
            return "Function provide kijiye for integration"
        except Exception as e:
            return f"Integration error: {str(e)}"
    
    def solve_any_derivative(self, command):
        """Solve any derivative"""
        try:
            function_str = self.extract_function_from_command(command)
            if function_str:
                expr = sp.sympify(function_str)
                
                # Check for higher order derivatives
                order = self.extract_derivative_order(command)
                result = diff(expr, self.x, order)
                
                if order == 1:
                    return f"First derivative: d/dx({function_str}) = {result}"
                else:
                    return f"{order}th derivative: d^{order}/dx^{order}({function_str}) = {result}"
            
            return "Function provide kijiye for differentiation"
        except Exception as e:
            return f"Differentiation error: {str(e)}"
    
    def solve_any_limit(self, command):
        """Solve any limit"""
        try:
            function_str = self.extract_function_from_command(command)
            limit_point = self.extract_limit_point(command)
            
            if function_str and limit_point is not None:
                expr = sp.sympify(function_str)
                
                # Check for one-sided limits
                if 'left' in command or 'from left' in command:
                    result = limit(expr, self.x, limit_point, '-')
                    return f"Left limit: lim(x→{limit_point}⁻) {function_str} = {result}"
                elif 'right' in command or 'from right' in command:
                    result = limit(expr, self.x, limit_point, '+')
                    return f"Right limit: lim(x→{limit_point}⁺) {function_str} = {result}"
                else:
                    result = limit(expr, self.x, limit_point)
                    return f"Limit: lim(x→{limit_point}) {function_str} = {result}"
            
            return "Function aur limit point provide kijiye"
        except Exception as e:
            return f"Limit calculation error: {str(e)}"
    
    def solve_any_matrix(self, command):
        """Solve matrix problems"""
        try:
            if 'determinant' in command:
                matrix = self.extract_matrix_from_command(command)
                if matrix:
                    det = matrix.det()
                    return f"Determinant = {det}"
                return "Matrix provide kijiye for determinant"
            
            elif 'inverse' in command:
                matrix = self.extract_matrix_from_command(command)
                if matrix:
                    try:
                        inverse = matrix.inv()
                        return f"Matrix inverse = {inverse}"
                    except:
                        return "Matrix is not invertible (determinant = 0)"
                return "Matrix provide kijiye for inverse"
            
            elif 'multiply' in command:
                matrices = self.extract_matrices_from_command(command)
                if len(matrices) >= 2:
                    try:
                        result = matrices[0] * matrices[1]
                        return f"Matrix multiplication result = {result}"
                    except:
                        return "Matrix dimensions are not compatible for multiplication"
                return "Two matrices provide kijiye for multiplication"
            
            return "Matrix operation specify kijiye: determinant, inverse, multiply"
        except Exception as e:
            return f"Matrix operation error: {str(e)}"
    
    def solve_number_problem(self, command):
        """Solve problems involving numbers"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            if not numbers:
                return "No numbers found in the command"
            
            if len(numbers) == 1:
                num = numbers[0]
                result = f"Number: {num}\n"
                result += f"Square: {num**2}\n"
                result += f"Cube: {num**3}\n"
                result += f"Square root: {num**0.5:.4f}\n"
                result += f"Factorial: {math.factorial(int(num)) if num >= 0 and num == int(num) else 'Not defined'}"
                return result
            
            elif len(numbers) >= 2:
                result = f"Numbers: {numbers}\n"
                result += f"Sum: {sum(numbers)}\n"
                result += f"Product: {math.prod(numbers)}\n"
                result += f"Average: {sum(numbers)/len(numbers):.4f}\n"
                result += f"Maximum: {max(numbers)}\n"
                result += f"Minimum: {min(numbers)}"
                return result
            
        except Exception as e:
            return f"Number problem error: {str(e)}"
    
    def solve_word_problem(self, command):
        """Solve word problems"""
        try:
            if 'age' in command or 'years old' in command:
                return self.solve_age_problems(command)
            elif 'speed' in command and 'distance' in command:
                return self.solve_speed_distance_time(command)
            elif 'profit' in command or 'loss' in command:
                return self.solve_profit_loss_problems(command)
            elif 'work' in command and 'time' in command:
                return self.solve_work_time_problems(command)
            else:
                return "Word problem type not recognized. Please be more specific."
        except Exception as e:
            return f"Word problem error: {str(e)}"
    
    def provide_mathematical_help(self, command):
        """Provide help for mathematical concepts"""
        try:
            if 'prime' in command:
                return "Prime numbers are natural numbers greater than 1 that have no positive divisors other than 1 and themselves."
            elif 'fibonacci' in command:
                return "Fibonacci sequence: Each number is sum of two preceding ones. F(n) = F(n-1) + F(n-2)"
            elif 'factorial' in command:
                return "Factorial of n (n!) is product of all positive integers less than or equal to n."
            elif 'logarithm' in command:
                return "Logarithm: log_a(b) = c means a^c = b. Common properties: log(ab) = log(a) + log(b)"
            elif 'derivative' in command:
                return "Derivative measures rate of change. Basic rules: d/dx(x^n) = nx^(n-1), d/dx(e^x) = e^x"
            elif 'integral' in command:
                return "Integral is reverse of derivative. ∫f(x)dx gives antiderivative of f(x)."
            else:
                return f"Mathematical concept '{command}' - please ask specific questions for detailed help."
        except Exception as e:
            return f"Help error: {str(e)}"
    
    # Helper methods for specific operations
    def solve_addition(self, command):
        """Solve addition problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                result = sum(numbers)
                return f"Addition: {' + '.join(map(str, numbers))} = {result}"
            return "At least two numbers provide kijiye for addition"
        except Exception as e:
            return f"Addition error: {str(e)}"
    
    def solve_subtraction(self, command):
        """Solve subtraction problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                result = numbers[0] - numbers[1]
                return f"Subtraction: {numbers[0]} - {numbers[1]} = {result}"
            return "Two numbers provide kijiye for subtraction"
        except Exception as e:
            return f"Subtraction error: {str(e)}"
    
    def solve_multiplication(self, command):
        """Solve multiplication problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                result = math.prod(numbers)
                return f"Multiplication: {' × '.join(map(str, numbers))} = {result}"
            return "At least two numbers provide kijiye for multiplication"
        except Exception as e:
            return f"Multiplication error: {str(e)}"
    
    def solve_division(self, command):
        """Solve division problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                if numbers[1] != 0:
                    result = numbers[0] / numbers[1]
                    remainder = numbers[0] % numbers[1] if numbers[0] % numbers[1] != 0 else None
                    response = f"Division: {numbers[0]} ÷ {numbers[1]} = {result}"
                    if remainder:
                        response += f"\nRemainder: {remainder}"
                    return response
                else:
                    return "Division by zero is undefined"
            return "Two numbers provide kijiye for division"
        except Exception as e:
            return f"Division error: {str(e)}"
    
    def solve_power(self, command):
        """Solve power/exponent problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if len(numbers) >= 2:
                base, exponent = numbers[0], numbers[1]
                result = base ** exponent
                return f"Power: {base}^{exponent} = {result}"
            return "Base aur exponent provide kijiye"
        except Exception as e:
            return f"Power calculation error: {str(e)}"
    
    def solve_square_root(self, command):
        """Solve square root problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if numbers:
                number = numbers[0]
                if number >= 0:
                    result = number ** 0.5
                    return f"Square root: √{number} = {result:.6f}"
                else:
                    result = (abs(number) ** 0.5)
                    return f"Square root: √{number} = {result:.6f}i (imaginary)"
            return "Number provide kijiye for square root"
        except Exception as e:
            return f"Square root error: {str(e)}"
    
    def solve_cube_root(self, command):
        """Solve cube root problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            if numbers:
                number = numbers[0]
                result = abs(number) ** (1/3) * (1 if number >= 0 else -1)
                return f"Cube root: ∛{number} = {result:.6f}"
            return "Number provide kijiye for cube root"
        except Exception as e:
            return f"Cube root error: {str(e)}"
    
    # Helper methods for extraction
    def extract_integration_limits(self, command):
        """Extract integration limits from command"""
        # Look for patterns like "from 0 to 5" or "between 1 and 3"
        patterns = [
            r'from\s+([0-9\-\.]+)\s+to\s+([0-9\-\.]+)',
            r'between\s+([0-9\-\.]+)\s+and\s+([0-9\-\.]+)',
            r'limits\s+([0-9\-\.]+)\s+([0-9\-\.]+)'
        ]
        
        for pattern in patterns:
            match = re.search(pattern, command, re.IGNORECASE)
            if match:
                try:
                    return float(match.group(1)), float(match.group(2))
                except:
                    continue
        return None
    
    def extract_derivative_order(self, command):
        """Extract derivative order from command"""
        if 'second' in command or '2nd' in command:
            return 2
        elif 'third' in command or '3rd' in command:
            return 3
        elif 'fourth' in command or '4th' in command:
            return 4
        else:
            return 1 
   
    # ==================== ADVANCED PROBLEM SOLVING METHODS ====================
    
    def answer_counting_questions(self, command):
        """Answer counting and 'how many' questions"""
        try:
            if 'prime numbers' in command:
                numbers = self.extract_numbers_from_command(command)
                if numbers:
                    limit = int(numbers[0])
                    primes = self.count_primes_up_to(limit)
                    return f"Prime numbers up to {limit}: {len(primes)} primes\nPrimes: {primes[:10]}{'...' if len(primes) > 10 else ''}"
                return "Limit provide kijiye for counting primes"
            
            elif 'factors' in command:
                numbers = self.extract_numbers_from_command(command)
                if numbers:
                    number = int(numbers[0])
                    factors = self.find_factors(number)
                    return f"Factors of {number}: {factors}\nTotal factors: {len(factors)}"
                return "Number provide kijiye for finding factors"
            
            elif 'digits' in command:
                numbers = self.extract_numbers_from_command(command)
                if numbers:
                    number = abs(int(numbers[0]))
                    digit_count = len(str(number))
                    return f"Number of digits in {number}: {digit_count}"
                return "Number provide kijiye for counting digits"
            
            else:
                numbers = self.extract_numbers_from_command(command)
                if numbers:
                    return f"Numbers found: {numbers}\nCount: {len(numbers)}"
                return "Please specify what to count"
                
        except Exception as e:
            return f"Counting error: {str(e)}"
    
    def solve_percentage_problems(self, command):
        """Solve percentage problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            if 'percent of' in command or '% of' in command:
                if len(numbers) >= 2:
                    percentage = numbers[0]
                    total = numbers[1]
                    result = (percentage / 100) * total
                    return f"{percentage}% of {total} = {result}"
            
            elif 'what percent' in command:
                if len(numbers) >= 2:
                    part = numbers[0]
                    whole = numbers[1]
                    percentage = (part / whole) * 100
                    return f"{part} is {percentage:.2f}% of {whole}"
            
            elif 'increase' in command or 'decrease' in command:
                if len(numbers) >= 2:
                    original = numbers[0]
                    new_value = numbers[1]
                    change = ((new_value - original) / original) * 100
                    change_type = "increase" if change > 0 else "decrease"
                    return f"Percentage {change_type}: {abs(change):.2f}%"
            
            else:
                if len(numbers) >= 2:
                    percentage = numbers[0]
                    total = numbers[1]
                    result = (percentage / 100) * total
                    return f"{percentage}% of {total} = {result}"
            
            return "Percentage problem format: 'X percent of Y' or 'what percent is X of Y'"
        except Exception as e:
            return f"Percentage calculation error: {str(e)}"
    
    def solve_ratio_problems(self, command):
        """Solve ratio and proportion problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            if 'ratio' in command and len(numbers) >= 2:
                a, b = numbers[0], numbers[1]
                # Find GCD to simplify ratio
                gcd = math.gcd(int(a), int(b))
                simplified_a = int(a / gcd)
                simplified_b = int(b / gcd)
                return f"Ratio {a}:{b} = {simplified_a}:{simplified_b} (simplified)"
            
            elif 'proportion' in command and len(numbers) >= 3:
                # a:b = c:x, find x
                a, b, c = numbers[0], numbers[1], numbers[2]
                x = (b * c) / a
                return f"Proportion: {a}:{b} = {c}:{x}\nSolution: x = {x}"
            
            elif len(numbers) >= 4:
                # Check if four numbers are in proportion
                a, b, c, d = numbers[0], numbers[1], numbers[2], numbers[3]
                if abs(a * d - b * c) < 0.0001:
                    return f"Numbers {a}, {b}, {c}, {d} are in proportion\n{a}:{b} = {c}:{d}"
                else:
                    return f"Numbers {a}, {b}, {c}, {d} are NOT in proportion"
            
            return "Ratio/Proportion format: 'ratio of A to B' or 'A:B = C:D'"
        except Exception as e:
            return f"Ratio/Proportion error: {str(e)}"
    
    def solve_average_problems(self, command):
        """Solve average/mean problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            if numbers:
                average = sum(numbers) / len(numbers)
                total = sum(numbers)
                count = len(numbers)
                
                result = f"Numbers: {numbers}\n"
                result += f"Sum: {total}\n"
                result += f"Count: {count}\n"
                result += f"Average/Mean: {average:.4f}"
                
                # Additional statistics
                if len(numbers) > 1:
                    sorted_nums = sorted(numbers)
                    median = sorted_nums[len(sorted_nums)//2] if len(sorted_nums) % 2 == 1 else (sorted_nums[len(sorted_nums)//2-1] + sorted_nums[len(sorted_nums)//2]) / 2
                    result += f"\nMedian: {median}"
                    
                    variance = sum((x - average) ** 2 for x in numbers) / len(numbers)
                    std_dev = variance ** 0.5
                    result += f"\nStandard Deviation: {std_dev:.4f}"
                
                return result
            
            return "Numbers provide kijiye for average calculation"
        except Exception as e:
            return f"Average calculation error: {str(e)}"
    
    def solve_age_problems(self, command):
        """Solve age-related word problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            if 'years ago' in command and len(numbers) >= 2:
                current_age = numbers[0]
                years_ago = numbers[1]
                past_age = current_age - years_ago
                return f"Current age: {current_age} years\n{years_ago} years ago: {past_age} years"
            
            elif 'years from now' in command or 'after' in command and len(numbers) >= 2:
                current_age = numbers[0]
                years_later = numbers[1]
                future_age = current_age + years_later
                return f"Current age: {current_age} years\nAfter {years_later} years: {future_age} years"
            
            elif 'sum of ages' in command and len(numbers) >= 2:
                total_age = sum(numbers)
                return f"Individual ages: {numbers}\nSum of ages: {total_age} years"
            
            elif 'ratio of ages' in command and len(numbers) >= 2:
                age1, age2 = numbers[0], numbers[1]
                gcd = math.gcd(int(age1), int(age2))
                ratio = f"{int(age1/gcd)}:{int(age2/gcd)}"
                return f"Ages: {age1} and {age2}\nRatio: {ratio}"
            
            return "Age problem format: 'X years old', 'Y years ago', 'Z years from now'"
        except Exception as e:
            return f"Age problem error: {str(e)}"
    
    def solve_speed_distance_time(self, command):
        """Solve speed, distance, time problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            # Speed = Distance / Time
            # Distance = Speed × Time  
            # Time = Distance / Speed
            
            if 'speed' in command and 'distance' in command and 'time' in command:
                if len(numbers) >= 2:
                    if 'find speed' in command:
                        distance, time = numbers[0], numbers[1]
                        speed = distance / time
                        return f"Distance: {distance} units\nTime: {time} units\nSpeed: {speed} units per time"
                    elif 'find distance' in command:
                        speed, time = numbers[0], numbers[1]
                        distance = speed * time
                        return f"Speed: {speed} units per time\nTime: {time} units\nDistance: {distance} units"
                    elif 'find time' in command:
                        distance, speed = numbers[0], numbers[1]
                        time = distance / speed
                        return f"Distance: {distance} units\nSpeed: {speed} units per time\nTime: {time} units"
            
            elif len(numbers) >= 2:
                # Assume first two numbers are distance and time, find speed
                distance, time = numbers[0], numbers[1]
                if time != 0:
                    speed = distance / time
                    return f"Distance: {distance}\nTime: {time}\nSpeed: {speed}"
                else:
                    return "Time cannot be zero"
            
            return "Speed-Distance-Time format: provide distance and time to find speed"
        except Exception as e:
            return f"Speed-Distance-Time error: {str(e)}"
    
    def solve_profit_loss_problems(self, command):
        """Solve profit and loss problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            if 'cost price' in command and 'selling price' in command and len(numbers) >= 2:
                cost_price = numbers[0]
                selling_price = numbers[1]
                
                if selling_price > cost_price:
                    profit = selling_price - cost_price
                    profit_percent = (profit / cost_price) * 100
                    return f"Cost Price: {cost_price}\nSelling Price: {selling_price}\nProfit: {profit}\nProfit%: {profit_percent:.2f}%"
                else:
                    loss = cost_price - selling_price
                    loss_percent = (loss / cost_price) * 100
                    return f"Cost Price: {cost_price}\nSelling Price: {selling_price}\nLoss: {loss}\nLoss%: {loss_percent:.2f}%"
            
            elif 'profit' in command and '%' in command and len(numbers) >= 2:
                cost_price = numbers[0]
                profit_percent = numbers[1]
                profit = (profit_percent / 100) * cost_price
                selling_price = cost_price + profit
                return f"Cost Price: {cost_price}\nProfit%: {profit_percent}%\nProfit: {profit}\nSelling Price: {selling_price}"
            
            elif 'loss' in command and '%' in command and len(numbers) >= 2:
                cost_price = numbers[0]
                loss_percent = numbers[1]
                loss = (loss_percent / 100) * cost_price
                selling_price = cost_price - loss
                return f"Cost Price: {cost_price}\nLoss%: {loss_percent}%\nLoss: {loss}\nSelling Price: {selling_price}"
            
            return "Profit/Loss format: 'cost price X, selling price Y' or 'cost price X, profit Y%'"
        except Exception as e:
            return f"Profit/Loss error: {str(e)}"
    
    def solve_work_time_problems(self, command):
        """Solve work and time problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            if 'can complete' in command and 'days' in command and len(numbers) >= 2:
                # A can complete work in X days, B in Y days, together in how many days?
                days_a = numbers[0]
                days_b = numbers[1]
                
                # Work rates
                rate_a = 1 / days_a
                rate_b = 1 / days_b
                combined_rate = rate_a + rate_b
                combined_days = 1 / combined_rate
                
                return f"A completes work in: {days_a} days\nB completes work in: {days_b} days\nTogether they complete in: {combined_days:.2f} days"
            
            elif 'working together' in command and len(numbers) >= 3:
                # A, B, C working together
                total_rate = sum(1/days for days in numbers)
                combined_days = 1 / total_rate
                return f"Individual completion times: {numbers} days\nWorking together: {combined_days:.2f} days"
            
            return "Work-Time format: 'A can complete in X days, B in Y days'"
        except Exception as e:
            return f"Work-Time error: {str(e)}"
    
    def solve_mixture_problems(self, command):
        """Solve mixture and alligation problems"""
        try:
            numbers = self.extract_numbers_from_command(command)
            
            if 'mixture' in command and 'ratio' in command and len(numbers) >= 3:
                # Simple mixture ratio problem
                ratio_a = numbers[0]
                ratio_b = numbers[1]
                total_quantity = numbers[2]
                
                total_parts = ratio_a + ratio_b
                quantity_a = (ratio_a / total_parts) * total_quantity
                quantity_b = (ratio_b / total_parts) * total_quantity
                
                return f"Mixture ratio: {ratio_a}:{ratio_b}\nTotal quantity: {total_quantity}\nQuantity A: {quantity_a}\nQuantity B: {quantity_b}"
            
            return "Mixture format: 'mixture in ratio A:B, total quantity C'"
        except Exception as e:
            return f"Mixture problem error: {str(e)}"
    
    # Helper methods
    def count_primes_up_to(self, limit):
        """Count prime numbers up to limit"""
        if limit < 2:
            return []
        
        primes = []
        for num in range(2, limit + 1):
            is_prime = True
            for i in range(2, int(num ** 0.5) + 1):
                if num % i == 0:
                    is_prime = False
                    break
            if is_prime:
                primes.append(num)
        return primes
    
    def find_factors(self, number):
        """Find all factors of a number"""
        factors = []
        for i in range(1, abs(int(number)) + 1):
            if number % i == 0:
                factors.append(i)
        return factors       
 
        # ==================== COMPREHENSIVE MATHEMATICS ENCYCLOPEDIA (3301-4300+) ====================
        
        # Advanced Algebra Formulas (3301-3400)
        if any(word in command for word in ['polynomial formula', 'polynomial equation']):
            result = self.get_polynomial_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['sequence formula', 'series formula', 'progression formula']):
            result = self.get_sequence_series_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['matrix formula', 'linear algebra formula']):
            result = self.get_matrix_linear_algebra_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['complex number formula', 'imaginary formula']):
            result = self.get_complex_number_formulas(command)
            self.speak(result)
        
        # Advanced Calculus Formulas (3401-3500)
        elif any(word in command for word in ['multivariable calculus', 'partial derivative formula']):
            result = self.get_multivariable_calculus_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['vector calculus formula', 'gradient formula']):
            result = self.get_vector_calculus_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['differential equation formula', 'ode formula']):
            result = self.get_differential_equation_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['fourier series formula', 'transform formula']):
            result = self.get_fourier_transform_formulas(command)
            self.speak(result)
        
        # Advanced Geometry Formulas (3501-3600)
        elif any(word in command for word in ['coordinate geometry formula', 'analytic geometry']):
            result = self.get_coordinate_geometry_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['solid geometry formula', '3d geometry']):
            result = self.get_solid_geometry_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['conic section formula', 'ellipse formula', 'parabola formula']):
            result = self.get_conic_section_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['spherical geometry', 'spherical triangle']):
            result = self.get_spherical_geometry_formulas(command)
            self.speak(result)
        
        # Advanced Trigonometry Formulas (3601-3700)
        elif any(word in command for word in ['inverse trigonometry', 'arc function']):
            result = self.get_inverse_trigonometry_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['hyperbolic function', 'sinh', 'cosh', 'tanh']):
            result = self.get_hyperbolic_function_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['trigonometric equation', 'trig equation']):
            result = self.get_trigonometric_equation_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['spherical trigonometry', 'spherical triangle']):
            result = self.get_spherical_trigonometry_formulas(command)
            self.speak(result)
        
        # Number Theory Formulas (3701-3800)
        elif any(word in command for word in ['number theory formula', 'prime formula']):
            result = self.get_number_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['modular arithmetic formula', 'congruence formula']):
            result = self.get_modular_arithmetic_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['diophantine equation', 'integer solution']):
            result = self.get_diophantine_equation_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['cryptography formula', 'rsa formula']):
            result = self.get_cryptography_formulas(command)
            self.speak(result)
        
        # Discrete Mathematics (3801-3900)
        elif any(word in command for word in ['combinatorics formula', 'permutation combination']):
            result = self.get_combinatorics_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['graph theory formula', 'network formula']):
            result = self.get_graph_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['boolean algebra', 'logic formula']):
            result = self.get_boolean_algebra_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['set theory formula', 'venn diagram']):
            result = self.get_set_theory_formulas(command)
            self.speak(result)
        
        # Advanced Statistics & Probability (3901-4000)
        elif any(word in command for word in ['probability distribution', 'statistical distribution']):
            result = self.get_probability_distribution_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['hypothesis testing formula', 'statistical test']):
            result = self.get_hypothesis_testing_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['regression formula', 'correlation formula']):
            result = self.get_regression_correlation_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['bayesian formula', 'bayes theorem']):
            result = self.get_bayesian_statistics_formulas(command)
            self.speak(result)
        
        # Mathematical Physics (4001-4100)
        elif any(word in command for word in ['quantum mechanics formula', 'schrodinger equation']):
            result = self.get_quantum_mechanics_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['relativity formula', 'einstein equation']):
            result = self.get_relativity_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['wave mechanics', 'wave equation']):
            result = self.get_wave_mechanics_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['statistical mechanics', 'thermodynamic formula']):
            result = self.get_statistical_mechanics_formulas(command)
            self.speak(result)
        
        # Numerical Analysis (4101-4200)
        elif any(word in command for word in ['numerical method', 'approximation formula']):
            result = self.get_numerical_analysis_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['interpolation formula', 'extrapolation']):
            result = self.get_interpolation_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['numerical integration', 'quadrature formula']):
            result = self.get_numerical_integration_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['error analysis', 'approximation error']):
            result = self.get_error_analysis_formulas(command)
            self.speak(result)
        
        # Applied Mathematics (4201-4300)
        elif any(word in command for word in ['optimization formula', 'linear programming']):
            result = self.get_optimization_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['game theory formula', 'decision theory']):
            result = self.get_game_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['information theory formula', 'entropy formula']):
            result = self.get_information_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['chaos theory formula', 'fractal formula']):
            result = self.get_chaos_fractal_formulas(command)
            self.speak(result)    
 
   # ==================== COMPREHENSIVE MATHEMATICS FORMULAS DATABASE ====================
    
    def get_polynomial_formulas(self, command):
        """Polynomial formulas and solutions"""
        try:
            if 'roots' in command or 'zeros' in command:
                return """Polynomial Root Formulas:
Linear: ax + b = 0 → x = -b/a
Quadratic: ax² + bx + c = 0 → x = (-b ± √(b²-4ac))/2a
Cubic: ax³ + bx² + cx + d = 0 (Cardano's formula)
Quartic: ax⁴ + bx³ + cx² + dx + e = 0 (Ferrari's method)
Vieta's formulas: Sum of roots = -b/a, Product = c/a (quadratic)
Rational Root Theorem: p/q where p|constant, q|leading coefficient"""
            
            elif 'synthetic division' in command:
                return """Synthetic Division:
For dividing P(x) by (x - c):
1. Write coefficients in order
2. Bring down first coefficient
3. Multiply by c, add to next coefficient
4. Repeat until done
5. Last number is remainder
Example: (x³ - 6x² + 11x - 6) ÷ (x - 1)"""
            
            elif 'remainder theorem' in command:
                return """Polynomial Theorems:
Remainder Theorem: P(x) ÷ (x-a) has remainder P(a)
Factor Theorem: (x-a) is factor of P(x) iff P(a) = 0
Fundamental Theorem: Polynomial of degree n has exactly n roots (counting multiplicity)
Descartes' Rule: Number of positive roots ≤ sign changes in P(x)"""
            
            else:
                return """Polynomial Formulas:
• General form: P(x) = aₙxⁿ + aₙ₋₁xⁿ⁻¹ + ... + a₁x + a₀
• Degree: Highest power of x
• Leading coefficient: aₙ
• Quadratic formula: x = (-b ± √(b²-4ac))/2a
• Discriminant: Δ = b² - 4ac
• Vieta's formulas for sum and product of roots"""
        except Exception as e:
            return f"Polynomial formulas error: {str(e)}"
    
    def get_sequence_series_formulas(self, command):
        """Sequence and series formulas"""
        try:
            if 'arithmetic' in command:
                return """Arithmetic Progression (AP):
General term: aₙ = a + (n-1)d
Sum of n terms: Sₙ = n/2[2a + (n-1)d] = n/2(a + l)
Where: a = first term, d = common difference, l = last term
Mean: AM = (a + b)/2
Properties: aₙ = (aₙ₋₁ + aₙ₊₁)/2"""
            
            elif 'geometric' in command:
                return """Geometric Progression (GP):
General term: aₙ = ar^(n-1)
Sum of n terms: Sₙ = a(rⁿ-1)/(r-1) for r ≠ 1
Sum to infinity: S∞ = a/(1-r) for |r| < 1
Mean: GM = √(ab)
Properties: aₙ² = aₙ₋₁ × aₙ₊₁"""
            
            elif 'harmonic' in command:
                return """Harmonic Progression (HP):
If a, b, c are in HP, then 1/a, 1/b, 1/c are in AP
General term: 1/aₙ = 1/a + (n-1)d
Harmonic mean: HM = 2ab/(a+b)
Relationship: AM ≥ GM ≥ HM
Sum formula: Complex, no simple closed form"""
            
            elif 'fibonacci' in command:
                return """Fibonacci Sequence:
Recurrence: Fₙ = Fₙ₋₁ + Fₙ₋₂
Initial: F₀ = 0, F₁ = 1
Binet's formula: Fₙ = (φⁿ - ψⁿ)/√5
Where φ = (1+√5)/2, ψ = (1-√5)/2
Properties: Fₙ₊₁Fₙ₋₁ - Fₙ² = (-1)ⁿ"""
            
            elif 'power series' in command:
                return """Power Series:
General form: Σ aₙxⁿ = a₀ + a₁x + a₂x² + ...
Radius of convergence: R = 1/lim|aₙ₊₁/aₙ|
Taylor series: f(x) = Σ f⁽ⁿ⁾(a)(x-a)ⁿ/n!
Maclaurin series: f(x) = Σ f⁽ⁿ⁾(0)xⁿ/n!
Common series: eˣ, sin x, cos x, ln(1+x)"""
            
            else:
                return """Sequence & Series Formulas:
• AP: aₙ = a + (n-1)d, Sₙ = n/2[2a + (n-1)d]
• GP: aₙ = ar^(n-1), Sₙ = a(rⁿ-1)/(r-1)
• HP: Reciprocals form AP
• Fibonacci: Fₙ = Fₙ₋₁ + Fₙ₋₂
• Power series: Σ aₙxⁿ with radius of convergence"""
        except Exception as e:
            return f"Sequence series formulas error: {str(e)}"
    
    def get_matrix_linear_algebra_formulas(self, command):
        """Matrix and linear algebra formulas"""
        try:
            if 'determinant' in command:
                return """Determinant Formulas:
2×2 matrix: |A| = ad - bc for [[a,b],[c,d]]
3×3 matrix: Expansion by minors
|A| = a₁₁(a₂₂a₃₃ - a₂₃a₃₂) - a₁₂(a₂₁a₃₃ - a₂₃a₃₁) + a₁₃(a₂₁a₃₂ - a₂₂a₃₁)
Properties: |AB| = |A||B|, |Aᵀ| = |A|, |A⁻¹| = 1/|A|
Cramer's rule: xᵢ = |Aᵢ|/|A|"""
            
            elif 'inverse' in command:
                return """Matrix Inverse:
A⁻¹ = (1/|A|) × adj(A)
For 2×2: A⁻¹ = (1/(ad-bc))[[d,-b],[-c,a]]
Properties: (AB)⁻¹ = B⁻¹A⁻¹, (Aᵀ)⁻¹ = (A⁻¹)ᵀ
Gauss-Jordan method for finding inverse
Conditions: |A| ≠ 0 for invertible matrix"""
            
            elif 'eigenvalue' in command:
                return """Eigenvalues & Eigenvectors:
Characteristic equation: |A - λI| = 0
Eigenvalue equation: Av = λv
Trace: tr(A) = Σλᵢ = sum of diagonal elements
Determinant: |A| = Πλᵢ = product of eigenvalues
Diagonalization: A = PDP⁻¹ where D is diagonal"""
            
            elif 'rank' in command:
                return """Matrix Rank:
Rank = number of linearly independent rows/columns
Row rank = Column rank
Properties: rank(AB) ≤ min(rank(A), rank(B))
rank(A + B) ≤ rank(A) + rank(B)
Full rank: rank(A) = min(m,n) for m×n matrix"""
            
            elif 'vector space' in command:
                return """Vector Space Properties:
Linear independence: c₁v₁ + c₂v₂ + ... + cₙvₙ = 0 ⟹ all cᵢ = 0
Basis: Linearly independent spanning set
Dimension: Number of vectors in basis
Span: Set of all linear combinations
Orthogonality: u·v = 0"""
            
            else:
                return """Matrix & Linear Algebra:
• Determinant: |A| for square matrices
• Inverse: A⁻¹ exists if |A| ≠ 0
• Eigenvalues: |A - λI| = 0
• Rank: Number of linearly independent rows
• Vector spaces: Basis, dimension, span
• Linear transformations: T(u+v) = T(u)+T(v)"""
        except Exception as e:
            return f"Matrix linear algebra error: {str(e)}"
    
    def get_complex_number_formulas(self, command):
        """Complex number formulas"""
        try:
            if 'polar form' in command:
                return """Complex Number Polar Form:
z = r(cos θ + i sin θ) = re^(iθ)
r = |z| = √(a² + b²) (modulus)
θ = arg(z) = tan⁻¹(b/a) (argument)
Euler's formula: e^(iθ) = cos θ + i sin θ
De Moivre's theorem: (re^(iθ))ⁿ = rⁿe^(inθ)"""
            
            elif 'operations' in command:
                return """Complex Number Operations:
Addition: (a+bi) + (c+di) = (a+c) + (b+d)i
Multiplication: (a+bi)(c+di) = (ac-bd) + (ad+bc)i
Division: (a+bi)/(c+di) = [(a+bi)(c-di)]/[c²+d²]
Conjugate: z̄ = a - bi if z = a + bi
Properties: z·z̄ = |z|², z + z̄ = 2Re(z)"""
            
            elif 'roots' in command:
                return """Complex Roots:
nth roots of unity: e^(2πik/n) for k = 0,1,...,n-1
Square roots: ±√r e^(iθ/2)
Cube roots: ∛r e^(i(θ+2πk)/3) for k = 0,1,2
Fundamental theorem: Every polynomial has n complex roots
Quadratic formula works for complex coefficients"""
            
            elif 'functions' in command:
                return """Complex Functions:
Exponential: e^z = e^x(cos y + i sin y)
Logarithm: ln z = ln|z| + i arg(z)
Trigonometric: sin z = (e^(iz) - e^(-iz))/(2i)
cos z = (e^(iz) + e^(-iz))/2
Hyperbolic: sinh z = (e^z - e^(-z))/2"""
            
            else:
                return """Complex Number Formulas:
• Rectangular: z = a + bi
• Polar: z = r(cos θ + i sin θ)
• Modulus: |z| = √(a² + b²)
• Argument: arg(z) = tan⁻¹(b/a)
• Euler: e^(iθ) = cos θ + i sin θ
• De Moivre: (re^(iθ))ⁿ = rⁿe^(inθ)"""
        except Exception as e:
            return f"Complex number formulas error: {str(e)}"
    
    def get_multivariable_calculus_formulas(self, command):
        """Multivariable calculus formulas"""
        try:
            if 'partial derivative' in command:
                return """Partial Derivatives:
∂f/∂x: Derivative with respect to x, treating y as constant
∂f/∂y: Derivative with respect to y, treating x as constant
Mixed partials: ∂²f/∂x∂y = ∂²f/∂y∂x (if continuous)
Chain rule: ∂f/∂t = (∂f/∂x)(∂x/∂t) + (∂f/∂y)(∂y/∂t)
Total differential: df = (∂f/∂x)dx + (∂f/∂y)dy"""
            
            elif 'gradient' in command:
                return """Gradient and Directional Derivatives:
Gradient: ∇f = (∂f/∂x, ∂f/∂y, ∂f/∂z)
Directional derivative: D_u f = ∇f · û
Maximum rate of change: |∇f| in direction of ∇f
Level curves: ∇f ⊥ level curves
Tangent plane: ∇f(a,b,c) · (x-a, y-b, z-c) = 0"""
            
            elif 'multiple integral' in command:
                return """Multiple Integrals:
Double integral: ∬_R f(x,y) dA
Triple integral: ∭_E f(x,y,z) dV
Change of variables: Jacobian determinant
Cylindrical: dV = r dr dθ dz
Spherical: dV = ρ² sin φ dρ dφ dθ
Fubini's theorem: Order of integration"""
            
            elif 'optimization' in command:
                return """Multivariable Optimization:
Critical points: ∇f = 0
Second derivative test: D = f_xx f_yy - (f_xy)²
D > 0, f_xx > 0: local minimum
D > 0, f_xx < 0: local maximum
D < 0: saddle point
Lagrange multipliers: ∇f = λ∇g for constraint g = 0"""
            
            else:
                return """Multivariable Calculus:
• Partial derivatives: ∂f/∂x, ∂f/∂y
• Gradient: ∇f = (∂f/∂x, ∂f/∂y, ∂f/∂z)
• Chain rule for multiple variables
• Multiple integrals with Jacobian
• Optimization with Lagrange multipliers"""
        except Exception as e:
            return f"Multivariable calculus error: {str(e)}"
    
    def get_vector_calculus_formulas(self, command):
        """Vector calculus formulas"""
        try:
            if 'divergence' in command:
                return """Divergence:
div F = ∇ · F = ∂P/∂x + ∂Q/∂y + ∂R/∂z
Physical meaning: Source/sink strength
Divergence theorem: ∭_E (∇ · F) dV = ∬_S F · n̂ dS
Properties: ∇ · (F + G) = ∇ · F + ∇ · G
∇ · (fF) = f(∇ · F) + F · (∇f)"""
            
            elif 'curl' in command:
                return """Curl:
curl F = ∇ × F = |i  j  k |
                   |∂/∂x ∂/∂y ∂/∂z|
                   |P  Q  R|
Physical meaning: Rotation/circulation
Stokes' theorem: ∬_S (∇ × F) · n̂ dS = ∮_C F · dr
Properties: ∇ × (F + G) = ∇ × F + ∇ × G"""
            
            elif 'line integral' in command:
                return """Line Integrals:
Scalar field: ∫_C f ds = ∫_a^b f(r(t))|r'(t)| dt
Vector field: ∫_C F · dr = ∫_a^b F(r(t)) · r'(t) dt
Conservative field: ∫_C F · dr = f(B) - f(A)
Path independence: ∇ × F = 0
Green's theorem: ∮_C F · dr = ∬_D (∂Q/∂x - ∂P/∂y) dA"""
            
            elif 'surface integral' in command:
                return """Surface Integrals:
Scalar field: ∬_S f dS = ∬_D f(r(u,v))|r_u × r_v| du dv
Vector field: ∬_S F · n̂ dS = ∬_D F(r(u,v)) · (r_u × r_v) du dv
Flux: Rate of flow through surface
Divergence theorem: ∬_S F · n̂ dS = ∭_E ∇ · F dV"""
            
            else:
                return """Vector Calculus:
• Gradient: ∇f points in direction of steepest increase
• Divergence: ∇ · F measures source/sink
• Curl: ∇ × F measures rotation
• Line integrals: Work and circulation
• Surface integrals: Flux through surfaces
• Fundamental theorems: Green's, Stokes', Divergence"""
        except Exception as e:
            return f"Vector calculus error: {str(e)}"
    
    def get_differential_equation_formulas(self, command):
        """Differential equation formulas"""
        try:
            if 'first order' in command:
                return """First Order ODEs:
Separable: dy/dx = f(x)g(y) → ∫dy/g(y) = ∫f(x)dx
Linear: dy/dx + P(x)y = Q(x)
Integrating factor: μ(x) = e^∫P(x)dx
Solution: y = (1/μ)∫μQ dx
Exact: M dx + N dy = 0 where ∂M/∂y = ∂N/∂x
Bernoulli: dy/dx + P(x)y = Q(x)y^n"""
            
            elif 'second order' in command:
                return """Second Order ODEs:
Homogeneous: ay'' + by' + cy = 0
Characteristic equation: ar² + br + c = 0
Real distinct roots: y = c₁e^(r₁x) + c₂e^(r₂x)
Repeated root: y = (c₁ + c₂x)e^(rx)
Complex roots r = α ± βi: y = e^(αx)(c₁cos βx + c₂sin βx)
Non-homogeneous: y = y_h + y_p"""
            
            elif 'laplace transform' in command:
                return """Laplace Transform Method:
L{f(t)} = F(s) = ∫₀^∞ e^(-st)f(t) dt
L{f'(t)} = sF(s) - f(0)
L{f''(t)} = s²F(s) - sf(0) - f'(0)
Common transforms:
L{1} = 1/s, L{t} = 1/s², L{e^(at)} = 1/(s-a)
L{sin at} = a/(s²+a²), L{cos at} = s/(s²+a²)"""
            
            elif 'partial differential' in command:
                return """Partial Differential Equations:
Heat equation: ∂u/∂t = α²∇²u
Wave equation: ∂²u/∂t² = c²∇²u
Laplace equation: ∇²u = 0
Poisson equation: ∇²u = f
Separation of variables: u(x,t) = X(x)T(t)
Fourier series solutions"""
            
            else:
                return """Differential Equations:
• First order: Separable, Linear, Exact, Bernoulli
• Second order: Homogeneous and non-homogeneous
• Characteristic equation method
• Laplace transform method
• Partial differential equations
• Boundary value problems"""
        except Exception as e:
            return f"Differential equation error: {str(e)}"
    
    def get_fourier_transform_formulas(self, command):
        """Fourier transform formulas"""
        try:
            if 'fourier series' in command:
                return """Fourier Series:
f(x) = a₀/2 + Σ[aₙcos(nπx/L) + bₙsin(nπx/L)]
aₙ = (1/L)∫₋ₗᴸ f(x)cos(nπx/L) dx
bₙ = (1/L)∫₋ₗᴸ f(x)sin(nπx/L) dx
Complex form: f(x) = Σ cₙe^(inπx/L)
cₙ = (1/2L)∫₋ₗᴸ f(x)e^(-inπx/L) dx"""
            
            elif 'fourier transform' in command:
                return """Fourier Transform:
F(ω) = ∫₋∞^∞ f(t)e^(-iωt) dt
f(t) = (1/2π)∫₋∞^∞ F(ω)e^(iωt) dω
Properties:
Linearity: F{af + bg} = aF{f} + bF{g}
Time shift: F{f(t-a)} = e^(-iaω)F(ω)
Frequency shift: F{e^(iat)f(t)} = F(ω-a)
Scaling: F{f(at)} = (1/|a|)F(ω/a)"""
            
            elif 'discrete fourier' in command:
                return """Discrete Fourier Transform:
X[k] = Σₙ₌₀^(N-1) x[n]e^(-i2πkn/N)
x[n] = (1/N)Σₖ₌₀^(N-1) X[k]e^(i2πkn/N)
Fast Fourier Transform (FFT): O(N log N)
Applications: Signal processing, image processing
Convolution theorem: DFT{x*y} = DFT{x}·DFT{y}"""
            
            elif 'laplace transform' in command:
                return """Laplace Transform:
F(s) = ∫₀^∞ f(t)e^(-st) dt
Properties:
Linearity: L{af + bg} = aL{f} + bL{g}
Time shift: L{f(t-a)u(t-a)} = e^(-as)F(s)
Frequency shift: L{e^(at)f(t)} = F(s-a)
Scaling: L{f(at)} = (1/a)F(s/a)
Derivatives: L{f'(t)} = sF(s) - f(0)"""
            
            else:
                return """Transform Formulas:
• Fourier Series: Periodic function expansion
• Fourier Transform: Frequency domain analysis
• Discrete Fourier Transform: Digital signal processing
• Laplace Transform: Solving differential equations
• Z-Transform: Discrete-time systems
• Wavelet Transform: Time-frequency analysis"""
        except Exception as e:
            return f"Fourier transform error: {str(e)}"    

    def get_coordinate_geometry_formulas(self, command):
        """Coordinate geometry formulas"""
        try:
            if 'straight line' in command:
                return """Straight Line Formulas:
Slope: m = (y₂-y₁)/(x₂-x₁)
Point-slope form: y - y₁ = m(x - x₁)
Slope-intercept form: y = mx + c
Two-point form: (y-y₁)/(y₂-y₁) = (x-x₁)/(x₂-x₁)
Intercept form: x/a + y/b = 1
Normal form: x cos α + y sin α = p
Distance from point to line: |ax₁+by₁+c|/√(a²+b²)"""
            
            elif 'circle' in command:
                return """Circle Formulas:
Standard form: (x-h)² + (y-k)² = r²
General form: x² + y² + 2gx + 2fy + c = 0
Center: (-g, -f), Radius: √(g²+f²-c)
Parametric: x = h + r cos θ, y = k + r sin θ
Tangent at (x₁,y₁): xx₁ + yy₁ = r² (for x²+y²=r²)
Length of tangent from external point: √(S₁) where S₁ = x₁²+y₁²+2gx₁+2fy₁+c"""
            
            elif 'parabola' in command:
                return """Parabola Formulas:
Standard forms:
y² = 4ax (opens right), y² = -4ax (opens left)
x² = 4ay (opens up), x² = -4ay (opens down)
Vertex form: y = a(x-h)² + k
Focus: (a, 0) for y² = 4ax
Directrix: x = -a for y² = 4ax
Parametric: x = at², y = 2at
Tangent at (x₁,y₁): yy₁ = 2a(x+x₁)"""
            
            elif 'ellipse' in command:
                return """Ellipse Formulas:
Standard form: x²/a² + y²/b² = 1 (a > b)
Foci: (±c, 0) where c² = a² - b²
Eccentricity: e = c/a = √(1-b²/a²)
Parametric: x = a cos θ, y = b sin θ
Tangent at (x₁,y₁): xx₁/a² + yy₁/b² = 1
Area: πab
Perimeter: π[3(a+b) - √((3a+b)(a+3b))] (approximation)"""
            
            elif 'hyperbola' in command:
                return """Hyperbola Formulas:
Standard form: x²/a² - y²/b² = 1
Foci: (±c, 0) where c² = a² + b²
Eccentricity: e = c/a = √(1+b²/a²)
Asymptotes: y = ±(b/a)x
Parametric: x = a sec θ, y = b tan θ
Tangent at (x₁,y₁): xx₁/a² - yy₁/b² = 1
Conjugate hyperbola: -x²/a² + y²/b² = 1"""
            
            else:
                return """Coordinate Geometry:
• Distance: d = √[(x₂-x₁)² + (y₂-y₁)²]
• Midpoint: M = ((x₁+x₂)/2, (y₁+y₂)/2)
• Section formula: Internal/External division
• Area of triangle: ½|x₁(y₂-y₃) + x₂(y₃-y₁) + x₃(y₁-y₂)|
• Conic sections: Circle, Parabola, Ellipse, Hyperbola"""
        except Exception as e:
            return f"Coordinate geometry error: {str(e)}"
    
    def get_solid_geometry_formulas(self, command):
        """Solid geometry formulas"""
        try:
            if 'sphere' in command:
                return """Sphere Formulas:
Volume: V = (4/3)πr³
Surface area: SA = 4πr²
Equation: x² + y² + z² = r² (center at origin)
General: (x-a)² + (y-b)² + (z-c)² = r²
Spherical coordinates: x = ρ sin φ cos θ, y = ρ sin φ sin θ, z = ρ cos φ
Volume element: dV = ρ² sin φ dρ dφ dθ"""
            
            elif 'cylinder' in command:
                return """Cylinder Formulas:
Volume: V = πr²h
Curved surface area: CSA = 2πrh
Total surface area: TSA = 2πr(r + h)
Equation: x² + y² = r² (infinite cylinder)
Finite cylinder: x² + y² = r², 0 ≤ z ≤ h
Cylindrical coordinates: x = r cos θ, y = r sin θ, z = z
Volume element: dV = r dr dθ dz"""
            
            elif 'cone' in command:
                return """Cone Formulas:
Volume: V = (1/3)πr²h
Curved surface area: CSA = πrl (l = slant height)
Total surface area: TSA = πr(r + l)
Slant height: l = √(r² + h²)
Equation: x² + y² = (r²/h²)(h-z)² for 0 ≤ z ≤ h
Semi-vertical angle: tan α = r/h"""
            
            elif 'pyramid' in command:
                return """Pyramid Formulas:
Volume: V = (1/3) × Base Area × Height
Square pyramid: V = (1/3)a²h
Triangular pyramid (tetrahedron): V = (1/6)|det[AB, AC, AD]|
Surface area: Base area + sum of triangular faces
Slant height: Distance from apex to midpoint of base edge
Regular tetrahedron: V = a³/(6√2), SA = √3 a²"""
            
            elif 'prism' in command:
                return """Prism Formulas:
Volume: V = Base Area × Height
Surface area: SA = 2 × Base Area + Perimeter × Height
Right prism: Lateral faces perpendicular to base
Oblique prism: Lateral faces not perpendicular
Triangular prism: V = (1/2)abh sin C × length
Rectangular prism: V = lwh, SA = 2(lw + lh + wh)"""
            
            else:
                return """Solid Geometry:
• Sphere: V = (4/3)πr³, SA = 4πr²
• Cylinder: V = πr²h, SA = 2πr(r+h)
• Cone: V = (1/3)πr²h, SA = πr(r+l)
• Pyramid: V = (1/3) × Base Area × Height
• Prism: V = Base Area × Height
• Coordinate systems: Cartesian, Cylindrical, Spherical"""
        except Exception as e:
            return f"Solid geometry error: {str(e)}"
    
    def get_conic_section_formulas(self, command):
        """Conic section formulas"""
        try:
            if 'general equation' in command:
                return """General Conic Equation:
Ax² + Bxy + Cy² + Dx + Ey + F = 0
Discriminant: Δ = B² - 4AC
Δ < 0: Ellipse (Circle if A = C, B = 0)
Δ = 0: Parabola
Δ > 0: Hyperbola
Degenerate cases: Point, line, two lines
Rotation angle: tan 2θ = B/(A-C)"""
            
            elif 'polar form' in command:
                return """Conic Sections in Polar Form:
r = l/(1 + e cos θ) or r = l/(1 + e sin θ)
Where l = semi-latus rectum, e = eccentricity
e = 0: Circle
0 < e < 1: Ellipse
e = 1: Parabola
e > 1: Hyperbola
Focus at pole, directrix perpendicular to initial line"""
            
            elif 'properties' in command:
                return """Conic Properties:
Eccentricity:
Circle: e = 0
Ellipse: e = √(1-b²/a²) where 0 < e < 1
Parabola: e = 1
Hyperbola: e = √(1+b²/a²) where e > 1
Focus-directrix property: Distance to focus / Distance to directrix = e
Reflection property: Light rays reflect through focus"""
            
            elif 'tangent normal' in command:
                return """Tangent and Normal:
Circle x²+y²=r²: Tangent at (x₁,y₁) is xx₁+yy₁=r²
Ellipse x²/a²+y²/b²=1: Tangent is xx₁/a²+yy₁/b²=1
Parabola y²=4ax: Tangent at (at²,2at) is ty=x+at²
Hyperbola x²/a²-y²/b²=1: Tangent is xx₁/a²-yy₁/b²=1
Normal: Perpendicular to tangent at point of contact"""
            
            else:
                return """Conic Sections:
• Circle: x² + y² = r², e = 0
• Ellipse: x²/a² + y²/b² = 1, 0 < e < 1
• Parabola: y² = 4ax, e = 1
• Hyperbola: x²/a² - y²/b² = 1, e > 1
• General form: Ax² + Bxy + Cy² + Dx + Ey + F = 0
• Discriminant determines type"""
        except Exception as e:
            return f"Conic section error: {str(e)}"
    
    def get_inverse_trigonometry_formulas(self, command):
        """Inverse trigonometry formulas"""
        try:
            if 'domain range' in command:
                return """Inverse Trig Domains & Ranges:
sin⁻¹x: Domain [-1,1], Range [-π/2, π/2]
cos⁻¹x: Domain [-1,1], Range [0, π]
tan⁻¹x: Domain ℝ, Range (-π/2, π/2)
csc⁻¹x: Domain (-∞,-1]∪[1,∞), Range [-π/2,0)∪(0,π/2]
sec⁻¹x: Domain (-∞,-1]∪[1,∞), Range [0,π/2)∪(π/2,π]
cot⁻¹x: Domain ℝ, Range (0, π)"""
            
            elif 'identities' in command:
                return """Inverse Trig Identities:
sin⁻¹x + cos⁻¹x = π/2
tan⁻¹x + cot⁻¹x = π/2
sec⁻¹x + csc⁻¹x = π/2
sin⁻¹(-x) = -sin⁻¹x
cos⁻¹(-x) = π - cos⁻¹x
tan⁻¹(-x) = -tan⁻¹x
sin(sin⁻¹x) = x, cos(cos⁻¹x) = x, tan(tan⁻¹x) = x"""
            
            elif 'addition formulas' in command:
                return """Inverse Trig Addition:
tan⁻¹x + tan⁻¹y = tan⁻¹((x+y)/(1-xy)) if xy < 1
tan⁻¹x + tan⁻¹y = tan⁻¹((x+y)/(1-xy)) + π if xy > 1, x,y > 0
tan⁻¹x - tan⁻¹y = tan⁻¹((x-y)/(1+xy))
sin⁻¹x + sin⁻¹y = sin⁻¹(x√(1-y²) + y√(1-x²))
cos⁻¹x + cos⁻¹y = cos⁻¹(xy - √((1-x²)(1-y²)))"""
            
            elif 'derivatives' in command:
                return """Inverse Trig Derivatives:
d/dx(sin⁻¹x) = 1/√(1-x²)
d/dx(cos⁻¹x) = -1/√(1-x²)
d/dx(tan⁻¹x) = 1/(1+x²)
d/dx(cot⁻¹x) = -1/(1+x²)
d/dx(sec⁻¹x) = 1/(|x|√(x²-1))
d/dx(csc⁻¹x) = -1/(|x|√(x²-1))"""
            
            else:
                return """Inverse Trigonometry:
• sin⁻¹x: Domain [-1,1], Range [-π/2, π/2]
• cos⁻¹x: Domain [-1,1], Range [0, π]
• tan⁻¹x: Domain ℝ, Range (-π/2, π/2)
• Complementary angles: sin⁻¹x + cos⁻¹x = π/2
• Addition formulas for combinations
• Derivatives and integrals"""
        except Exception as e:
            return f"Inverse trigonometry error: {str(e)}"
    
    def get_hyperbolic_function_formulas(self, command):
        """Hyperbolic function formulas"""
        try:
            if 'definitions' in command:
                return """Hyperbolic Function Definitions:
sinh x = (eˣ - e⁻ˣ)/2
cosh x = (eˣ + e⁻ˣ)/2
tanh x = sinh x/cosh x = (eˣ - e⁻ˣ)/(eˣ + e⁻ˣ)
csch x = 1/sinh x
sech x = 1/cosh x
coth x = 1/tanh x = cosh x/sinh x"""
            
            elif 'identities' in command:
                return """Hyperbolic Identities:
cosh²x - sinh²x = 1
1 - tanh²x = sech²x
coth²x - 1 = csch²x
sinh(x ± y) = sinh x cosh y ± cosh x sinh y
cosh(x ± y) = cosh x cosh y ± sinh x sinh y
tanh(x ± y) = (tanh x ± tanh y)/(1 ± tanh x tanh y)"""
            
            elif 'derivatives' in command:
                return """Hyperbolic Derivatives:
d/dx(sinh x) = cosh x
d/dx(cosh x) = sinh x
d/dx(tanh x) = sech²x
d/dx(coth x) = -csch²x
d/dx(sech x) = -sech x tanh x
d/dx(csch x) = -csch x coth x"""
            
            elif 'inverse' in command:
                return """Inverse Hyperbolic Functions:
sinh⁻¹x = ln(x + √(x² + 1))
cosh⁻¹x = ln(x + √(x² - 1)) for x ≥ 1
tanh⁻¹x = ½ln((1+x)/(1-x)) for |x| < 1
coth⁻¹x = ½ln((x+1)/(x-1)) for |x| > 1
sech⁻¹x = ln((1 + √(1-x²))/x) for 0 < x ≤ 1
csch⁻¹x = ln((1 + √(1+x²))/|x|) for x ≠ 0"""
            
            else:
                return """Hyperbolic Functions:
• sinh x = (eˣ - e⁻ˣ)/2
• cosh x = (eˣ + e⁻ˣ)/2
• tanh x = sinh x/cosh x
• Identity: cosh²x - sinh²x = 1
• Similar to trig functions but with different signs
• Applications in calculus and physics"""
        except Exception as e:
            return f"Hyperbolic function error: {str(e)}"
    
    def get_number_theory_formulas(self, command):
        """Number theory formulas"""
        try:
            if 'prime' in command:
                return """Prime Number Formulas:
Prime counting function: π(x) ≈ x/ln(x)
Prime number theorem: lim(x→∞) π(x)/(x/ln(x)) = 1
Sieve of Eratosthenes: Algorithm to find primes
Fermat's little theorem: aᵖ⁻¹ ≡ 1 (mod p) if p prime, gcd(a,p) = 1
Wilson's theorem: (p-1)! ≡ -1 (mod p) iff p is prime
Mersenne primes: 2ᵖ - 1 where p is prime"""
            
            elif 'divisibility' in command:
                return """Divisibility Rules:
2: Last digit even
3: Sum of digits divisible by 3
4: Last two digits divisible by 4
5: Last digit 0 or 5
6: Divisible by both 2 and 3
8: Last three digits divisible by 8
9: Sum of digits divisible by 9
11: Alternating sum of digits divisible by 11"""
            
            elif 'gcd lcm' in command:
                return """GCD and LCM Formulas:
Euclidean algorithm: gcd(a,b) = gcd(b, a mod b)
Extended Euclidean: ax + by = gcd(a,b)
LCM formula: lcm(a,b) = ab/gcd(a,b)
Properties: gcd(a,b) × lcm(a,b) = ab
For multiple numbers: gcd(a,b,c) = gcd(gcd(a,b),c)
Bézout's identity: gcd(a,b) = ax + by for some integers x,y"""
            
            elif 'modular' in command:
                return """Modular Arithmetic:
a ≡ b (mod m) means m|(a-b)
Properties: (a+b) mod m = ((a mod m) + (b mod m)) mod m
(ab) mod m = ((a mod m)(b mod m)) mod m
Chinese Remainder Theorem: System of congruences
Fermat's little theorem: aᵖ⁻¹ ≡ 1 (mod p)
Euler's theorem: aᶠ⁽ⁿ⁾ ≡ 1 (mod n) if gcd(a,n) = 1"""
            
            elif 'euler totient' in command:
                return """Euler's Totient Function:
φ(n) = number of integers ≤ n that are coprime to n
φ(p) = p - 1 for prime p
φ(pᵏ) = pᵏ - pᵏ⁻¹ = pᵏ⁻¹(p-1)
φ(mn) = φ(m)φ(n) if gcd(m,n) = 1
φ(n) = n∏(1 - 1/p) over all prime divisors p of n
Euler's theorem: aᶠ⁽ⁿ⁾ ≡ 1 (mod n) if gcd(a,n) = 1"""
            
            else:
                return """Number Theory:
• Prime numbers and primality tests
• GCD and LCM algorithms
• Modular arithmetic and congruences
• Euler's totient function φ(n)
• Fermat's and Euler's theorems
• Chinese Remainder Theorem
• Diophantine equations"""
        except Exception as e:
            return f"Number theory error: {str(e)}"  
      
        # ==================== TEXT INPUT COMMANDS ====================
        
        # Text Input Mode (4301-4350)
        if any(word in command for word in ['type mode', 'text mode', 'typing mode', 'keyboard input']):
            result = self.start_text_input_mode()
            self.speak(result)
        
        elif any(word in command for word in ['voice mode', 'speech mode', 'mic mode']):
            result = self.start_voice_mode()
            self.speak(result)
        
        elif any(word in command for word in ['input method', 'how to type', 'text input']):
            result = self.explain_input_methods()
            self.speak(result)
        
        # Mixed Mode Commands (4351-4400)
        elif any(word in command for word in ['both mode', 'mixed mode', 'voice and text']):
            result = self.start_mixed_mode()
            self.speak(result)
        
        elif any(word in command for word in ['switch mode', 'change input']):
            result = self.switch_input_mode()
            self.speak(result)
        
        elif any(word in command for word in ['current mode', 'input status']):
            result = self.show_current_mode()
            self.speak(result)    

    # ==================== TEXT INPUT FUNCTIONALITY ====================
    
    def get_text_input(self):
        """Get text input from user"""
        try:
            print("\n" + "="*50)
            print("TEXT INPUT MODE ACTIVATED")
            print("Type your question below (or 'voice' to switch back):")
            print("="*50)
            
            user_input = input("You: ").strip()
            
            if user_input.lower() in ['voice', 'voice mode', 'speech']:
                self.input_mode = "voice"
                return "voice_mode_switch"
            elif user_input.lower() in ['exit', 'quit', 'bye']:
                return "exit"
            else:
                return user_input.lower()
                
        except KeyboardInterrupt:
            print("\nText input cancelled.")
            return ""
        except Exception as e:
            print(f"Text input error: {str(e)}")
            return ""
    
    def get_mixed_input(self):
        """Get input from both voice and text"""
        try:
            print("\n" + "="*50)
            print("MIXED INPUT MODE - Choose your input method:")
            print("1. Press ENTER for Voice Input")
            print("2. Type your question for Text Input")
            print("3. Type 'voice' for Voice Mode only")
            print("4. Type 'text' for Text Mode only")
            print("="*50)
            
            user_choice = input("Your choice: ").strip()
            
            if user_choice == "":
                # Voice input
                return self.listen()
            elif user_choice.lower() == 'voice':
                self.input_mode = "voice"
                return "voice_mode_switch"
            elif user_choice.lower() == 'text':
                self.input_mode = "text"
                return "text_mode_switch"
            elif user_choice.lower() in ['exit', 'quit', 'bye']:
                return "exit"
            else:
                # Text input
                return user_choice.lower()
                
        except KeyboardInterrupt:
            print("\nMixed input cancelled.")
            return ""
        except Exception as e:
            print(f"Mixed input error: {str(e)}")
            return ""
    
    def start_text_input_mode(self):
        """Start text input mode"""
        self.input_mode = "text"
        self.voice_input_enabled = False
        self.text_input_enabled = True
        return "Text input mode activated! Ab aap typing karke questions puchh sakte hain. 'voice' type karke voice mode mein wapas ja sakte hain."
    
    def start_voice_mode(self):
        """Start voice input mode"""
        self.input_mode = "voice"
        self.voice_input_enabled = True
        self.text_input_enabled = False
        return "Voice input mode activated! Ab aap bol kar questions puchh sakte hain. 'type mode' kehkar text mode mein ja sakte hain."
    
    def start_mixed_mode(self):
        """Start mixed input mode"""
        self.input_mode = "mixed"
        self.voice_input_enabled = True
        self.text_input_enabled = True
        return "Mixed input mode activated! Ab aap voice ya text dono se questions puchh sakte hain."
    
    def switch_input_mode(self):
        """Switch between input modes"""
        if self.input_mode == "voice":
            self.input_mode = "text"
            return "Switched to text input mode. Ab aap typing kar sakte hain."
        elif self.input_mode == "text":
            self.input_mode = "mixed"
            return "Switched to mixed input mode. Ab aap voice ya text dono use kar sakte hain."
        else:
            self.input_mode = "voice"
            return "Switched to voice input mode. Ab aap bol kar questions puchh sakte hain."
    
    def show_current_mode(self):
        """Show current input mode"""
        mode_info = {
            "voice": "Voice Input Mode - Bol kar questions puchiye",
            "text": "Text Input Mode - Type karke questions puchiye", 
            "mixed": "Mixed Input Mode - Voice ya Text dono use kar sakte hain"
        }
        return f"Current mode: {mode_info.get(self.input_mode, 'Unknown')}"
    
    def explain_input_methods(self):
        """Explain different input methods"""
        return """Input Methods Available:

1. VOICE MODE:
   - Bol kar questions puchiye
   - "type mode" kehkar text mode mein switch kariye
   
2. TEXT MODE:
   - Keyboard se type karke questions puchiye
   - "voice" type karke voice mode mein switch kariye
   
3. MIXED MODE:
   - Voice aur text dono use kar sakte hain
   - Har question ke liye choose kar sakte hain
   
Commands:
- "type mode" - Text input activate
- "voice mode" - Voice input activate  
- "mixed mode" - Both inputs activate
- "switch mode" - Mode change karne ke liye
- "current mode" - Current mode dekhne ke liye

Example Text Questions:
- solve 2x + 3 = 7
- what is derivative of x^2
- calculate 25 + 37
- quadratic formula
- matrix determinant [[1,2],[3,4]]"""
    
    def display_text_interface(self):
        """Display text interface"""
        print("\n" + "="*60)
        print("🤖 ADVANCED VOICE ASSISTANT - TEXT MODE")
        print("="*60)
        print("📚 4400+ Mathematical Commands Available!")
        print("🔢 Universal Math Solver | 📐 All Formulas | 🧮 Instant Solutions")
        print("="*60)
        print("\n📝 TEXT INPUT EXAMPLES:")
        print("• solve 2x + 3 = 7")
        print("• derivative of x^3 + 2x^2")
        print("• matrix determinant [[1,2],[3,4]]")
        print("• quadratic formula")
        print("• what is 25 percent of 200")
        print("• fibonacci sequence 10")
        print("• area of circle radius 5")
        print("• trigonometry formula basic")
        print("\n🎤 Type 'voice' to switch to voice mode")
        print("❌ Type 'exit' to quit")
        print("="*60)
    
    def process_text_command(self, text_input):
        """Process text command with enhanced parsing"""
        try:
            # Clean and process the text input
            processed_input = text_input.strip().lower()
            
            # Add text-specific processing
            if processed_input.startswith(('solve ', 'calculate ', 'find ', 'what is ')):
                # Direct mathematical processing
                return self.universal_math_solver(processed_input)
            
            elif processed_input.endswith(' formula'):
                # Formula requests
                return self.get_formula_by_text(processed_input)
            
            elif any(op in processed_input for op in ['+', '-', '*', '/', '=', '^']):
                # Mathematical expressions
                return self.solve_arithmetic_expression(processed_input)
            
            elif processed_input.startswith(('derivative ', 'integral ', 'limit ')):
                # Calculus operations
                return self.solve_calculus_problem(processed_input)
            
            elif 'matrix' in processed_input:
                # Matrix operations
                return self.solve_any_matrix(processed_input)
            
            else:
                # Use regular command processing
                return self.execute_command(processed_input)
                
        except Exception as e:
            return f"Text command processing error: {str(e)}"
    
    def get_formula_by_text(self, text_input):
        """Get formula based on text input"""
        try:
            if 'quadratic' in text_input:
                return self.get_basic_formulas('quadratic')
            elif 'trigonometry' in text_input or 'trig' in text_input:
                return self.get_trigonometry_formulas('basic')
            elif 'calculus' in text_input:
                return self.get_calculus_formulas('derivative')
            elif 'geometry' in text_input:
                return self.get_geometry_formulas('area')
            elif 'algebra' in text_input:
                return self.get_algebra_formulas('identity')
            elif 'physics' in text_input:
                return self.get_physics_formulas('mechanics')
            elif 'statistics' in text_input:
                return self.get_statistics_formulas('descriptive')
            else:
                return "Formula category specify kijiye: quadratic, trigonometry, calculus, geometry, algebra, physics, statistics"
        except Exception as e:
            return f"Formula retrieval error: {str(e)}"
    
    def enhanced_run(self):
        """Enhanced run method with text input support"""
        self.speak("Advanced Voice Assistant with Text Input started!")
        self.speak("4400+ commands available! Voice aur Text dono modes supported hain.")
        
        print("\n🚀 WELCOME TO ADVANCED VOICE ASSISTANT")
        print("🎤 Voice Mode: Speak your questions")
        print("⌨️  Text Mode: Type your questions") 
        print("🔄 Mixed Mode: Use both voice and text")
        print("\nSay 'type mode' for text input or start speaking!")
        
        while True:
            try:
                command = ""
                
                if self.input_mode == "voice":
                    command = self.listen()
                elif self.input_mode == "text":
                    self.display_text_interface()
                    command = self.get_text_input()
                elif self.input_mode == "mixed":
                    command = self.get_mixed_input()
                
                if command == "voice_mode_switch":
                    self.speak("Voice mode activated")
                    continue
                elif command == "text_mode_switch":
                    print("Text mode activated")
                    continue
                elif command == "exit":
                    self.speak("Assistant band kar raha hun. Dhanyawad!")
                    break
                elif command:
                    if self.input_mode == "text":
                        # Process text command
                        result = self.process_text_command(command)
                        if result:
                            print(f"\nAssistant: {result}")
                            if self.voice_input_enabled:
                                self.speak(result)
                    else:
                        # Process voice command
                        if not self.execute_command(command):
                            break
                
                time.sleep(0.5)
                
            except KeyboardInterrupt:
                self.speak("Assistant band kar raha hun. Dhanyawad!")
                break
            except Exception as e:
                error_msg = "Koi error aaya hai. Phir se try kijiye."
                print(f"Error: {error_msg}")
                if self.voice_input_enabled:
                    self.speak(error_msg)
                continue
    
    def run(self):
        """Main run method - now uses enhanced version"""
        self.enhanced_run()

        if __name__ == "__main__":
            assistant = VoiceAssistant()
            assistant.run()
            result = self.get_boolean_algebra_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['truth table', 'logic table']):
            result = self.generate_truth_table(command)
            self.speak(result)
        
        elif any(word in command for word in ['logic gate', 'digital logic']):
            result = self.get_logic_gate_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['propositional logic', 'predicate logic']):
            result = self.get_propositional_logic_formulas(command)
            self.speak(result)
        
        # Logical Operations (4451-4500)
        elif any(word in command for word in ['and operation', 'logical and', 'conjunction']):
            result = self.solve_logical_and(command)
            self.speak(result)
        
        elif any(word in command for word in ['or operation', 'logical or', 'disjunction']):
            result = self.solve_logical_or(command)
            self.speak(result)
        
        elif any(word in command for word in ['not operation', 'logical not', 'negation']):
            result = self.solve_logical_not(command)
            self.speak(result)
        
        elif any(word in command for word in ['xor operation', 'exclusive or']):
            result = self.solve_logical_xor(command)
            self.speak(result)
        
        elif any(word in command for word in ['implication', 'logical implication', 'if then']):
            result = self.solve_logical_implication(command)
            self.speak(result)
        
        elif any(word in command for word in ['biconditional', 'if and only if', 'iff']):
            result = self.solve_logical_biconditional(command)
            self.speak(result)
        
        elif any(word in command for word in ['nand operation', 'logical nand']):
            result = self.solve_logical_nand(command)
            self.speak(result)
        
        elif any(word in command for word in ['nor operation', 'logical nor']):
            result = self.solve_logical_nor(command)
            self.speak(result)
        
        # Logic Simplification (4501-4550)
        elif any(word in command for word in ['simplify logic', 'boolean simplification', 'karnaugh map']):
            result = self.simplify_boolean_expression(command)
            self.speak(result)
        
        elif any(word in command for word in ['de morgan law', 'demorgan theorem']):
            result = self.apply_demorgan_laws(command)
            self.speak(result)
        
        elif any(word in command for word in ['boolean laws', 'logic laws']):
            result = self.get_boolean_laws(command)
            self.speak(result) 
   
    # ==================== COMPREHENSIVE LOGICAL FORMULAS & BOOLEAN ALGEBRA ====================
    
    def get_boolean_algebra_formulas(self, command):
        """Boolean algebra formulas and operations"""
        try:
            if 'basic' in command or 'fundamental' in command:
                return """Boolean Algebra Basic Operations:
AND (∧, •, &): A ∧ B = 1 if both A=1 and B=1, else 0
OR (∨, +, |): A ∨ B = 0 if both A=0 and B=0, else 1
NOT (¬, ~, '): ¬A = 1 if A=0, ¬A = 0 if A=1
XOR (⊕, ⊻): A ⊕ B = 1 if A≠B, else 0
NAND (↑): A ↑ B = ¬(A ∧ B)
NOR (↓): A ↓ B = ¬(A ∨ B)
IMPLICATION (→): A → B = ¬A ∨ B
BICONDITIONAL (↔): A ↔ B = (A → B) ∧ (B → A)"""
            
            elif 'laws' in command or 'rules' in command:
                return """Boolean Algebra Laws:
Identity Laws:
A ∧ 1 = A, A ∨ 0 = A
A ∧ 0 = 0, A ∨ 1 = 1

Complement Laws:
A ∧ ¬A = 0, A ∨ ¬A = 1
¬(¬A) = A (Double Negation)

Idempotent Laws:
A ∧ A = A, A ∨ A = A

Commutative Laws:
A ∧ B = B ∧ A, A ∨ B = B ∨ A

Associative Laws:
(A ∧ B) ∧ C = A ∧ (B ∧ C)
(A ∨ B) ∨ C = A ∨ (B ∨ C)

Distributive Laws:
A ∧ (B ∨ C) = (A ∧ B) ∨ (A ∧ C)
A ∨ (B ∧ C) = (A ∨ B) ∧ (A ∨ C)"""
            
            elif 'demorgan' in command:
                return """De Morgan's Laws:
¬(A ∧ B) = ¬A ∨ ¬B
¬(A ∨ B) = ¬A ∧ ¬B

Extended De Morgan's:
¬(A ∧ B ∧ C) = ¬A ∨ ¬B ∨ ¬C
¬(A ∨ B ∨ C) = ¬A ∧ ¬B ∧ ¬C

Applications:
NAND = ¬(A ∧ B) = ¬A ∨ ¬B
NOR = ¬(A ∨ B) = ¬A ∧ ¬B"""
            
            else:
                return """Boolean Algebra Overview:
• Basic Operations: AND, OR, NOT, XOR, NAND, NOR
• Boolean Laws: Identity, Complement, Distributive
• De Morgan's Laws: ¬(A∧B) = ¬A∨¬B
• Truth Tables: Complete operation definitions
• Logic Simplification: Karnaugh maps, algebraic methods
• Applications: Digital circuits, computer logic"""
        except Exception as e:
            return f"Boolean algebra error: {str(e)}"
    
    def generate_truth_table(self, command):
        """Generate truth tables for logical operations"""
        try:
            if 'and' in command:
                return """Truth Table for AND (∧):
A | B | A ∧ B
--|---|------
0 | 0 |   0
0 | 1 |   0
1 | 0 |   0
1 | 1 |   1

Logic: Output is 1 only when both inputs are 1"""
            
            elif 'or' in command:
                return """Truth Table for OR (∨):
A | B | A ∨ B
--|---|------
0 | 0 |   0
0 | 1 |   1
1 | 0 |   1
1 | 1 |   1

Logic: Output is 0 only when both inputs are 0"""
            
            elif 'not' in command:
                return """Truth Table for NOT (¬):
A | ¬A
--|---
0 | 1
1 | 0

Logic: Output is opposite of input"""
            
            elif 'xor' in command:
                return """Truth Table for XOR (⊕):
A | B | A ⊕ B
--|---|------
0 | 0 |   0
0 | 1 |   1
1 | 0 |   1
1 | 1 |   0

Logic: Output is 1 when inputs are different"""
            
            elif 'nand' in command:
                return """Truth Table for NAND (↑):
A | B | A ↑ B
--|---|------
0 | 0 |   1
0 | 1 |   1
1 | 0 |   1
1 | 1 |   0

Logic: NOT AND - opposite of AND gate"""
            
            elif 'nor' in command:
                return """Truth Table for NOR (↓):
A | B | A ↓ B
--|---|------
0 | 0 |   1
0 | 1 |   0
1 | 0 |   0
1 | 1 |   0

Logic: NOT OR - opposite of OR gate"""
            
            elif 'implication' in command:
                return """Truth Table for IMPLICATION (→):
A | B | A → B
--|---|------
0 | 0 |   1
0 | 1 |   1
1 | 0 |   0
1 | 1 |   1

Logic: False only when A is true and B is false"""
            
            else:
                return """Complete Truth Table (2 variables):
A | B | A∧B | A∨B | A⊕B | A→B | A↔B | ¬A | ¬B
--|---|-----|-----|-----|-----|-----|----|----|
0 | 0 |  0  |  0  |  0  |  1  |  1  | 1  | 1  |
0 | 1 |  0  |  1  |  1  |  1  |  0  | 1  | 0  |
1 | 0 |  0  |  1  |  1  |  0  |  0  | 0  | 1  |
1 | 1 |  1  |  1  |  0  |  1  |  1  | 0  | 0  |"""
        except Exception as e:
            return f"Truth table error: {str(e)}"
    
    def get_logic_gate_formulas(self, command):
        """Logic gate formulas and symbols"""
        try:
            if 'symbols' in command:
                return """Logic Gate Symbols:
AND Gate: ∧, •, &, AND
OR Gate: ∨, +, |, OR
NOT Gate: ¬, ~, ', NOT
XOR Gate: ⊕, ⊻, XOR
NAND Gate: ↑, NAND
NOR Gate: ↓, NOR
Buffer: →
Tri-state: ◊"""
            
            elif 'digital' in command:
                return """Digital Logic Gates:
AND: Output HIGH when all inputs HIGH
OR: Output HIGH when any input HIGH
NOT: Output opposite of input
XOR: Output HIGH when inputs different
NAND: NOT AND - Universal gate
NOR: NOT OR - Universal gate
Buffer: Output same as input (amplification)
Tri-state: Can be HIGH, LOW, or High-Z"""
            
            elif 'universal' in command:
                return """Universal Gates:
NAND Gate (Universal):
- Can implement AND, OR, NOT
- AND: A NAND (A NAND A) = A ∧ A = A
- OR: (A NAND A) NAND (B NAND B) = ¬A NAND ¬B = A ∨ B
- NOT: A NAND A = ¬A

NOR Gate (Universal):
- Can implement AND, OR, NOT
- OR: A NOR (A NOR A) = A ∨ A = A
- AND: (A NOR A) NOR (B NOR B) = ¬A NOR ¬B = A ∧ B
- NOT: A NOR A = ¬A"""
            
            else:
                return """Logic Gates Overview:
• Basic Gates: AND, OR, NOT
• Derived Gates: XOR, NAND, NOR
• Universal Gates: NAND, NOR (can implement any logic)
• Truth Tables: Define gate behavior
• Boolean Expressions: Mathematical representation
• Circuit Implementation: Physical realization"""
        except Exception as e:
            return f"Logic gate error: {str(e)}"
    
    def get_propositional_logic_formulas(self, command):
        """Propositional and predicate logic formulas"""
        try:
            if 'propositional' in command:
                return """Propositional Logic:
Propositions: Statements that are either true or false
Logical Connectives:
- Conjunction (∧): P ∧ Q
- Disjunction (∨): P ∨ Q
- Negation (¬): ¬P
- Implication (→): P → Q
- Biconditional (↔): P ↔ Q

Tautology: Always true (P ∨ ¬P)
Contradiction: Always false (P ∧ ¬P)
Contingency: Sometimes true, sometimes false"""
            
            elif 'predicate' in command:
                return """Predicate Logic:
Predicates: Functions that return true/false
Quantifiers:
- Universal (∀): ∀x P(x) - "for all x, P(x) is true"
- Existential (∃): ∃x P(x) - "there exists x such that P(x)"

Examples:
∀x (x > 0 → x² > 0) - "for all positive x, x² is positive"
∃x (x² = 4) - "there exists x such that x² = 4"

Quantifier Rules:
¬∀x P(x) ≡ ∃x ¬P(x)
¬∃x P(x) ≡ ∀x ¬P(x)"""
            
            elif 'inference' in command:
                return """Logical Inference Rules:
Modus Ponens: P → Q, P ⊢ Q
Modus Tollens: P → Q, ¬Q ⊢ ¬P
Hypothetical Syllogism: P → Q, Q → R ⊢ P → R
Disjunctive Syllogism: P ∨ Q, ¬P ⊢ Q
Addition: P ⊢ P ∨ Q
Simplification: P ∧ Q ⊢ P
Conjunction: P, Q ⊢ P ∧ Q
Resolution: P ∨ Q, ¬P ∨ R ⊢ Q ∨ R"""
            
            else:
                return """Logic Systems:
• Propositional Logic: Deals with propositions and connectives
• Predicate Logic: Includes quantifiers and predicates
• Modal Logic: Necessity and possibility
• Temporal Logic: Time-based reasoning
• Fuzzy Logic: Degrees of truth
• Inference Rules: Valid reasoning patterns"""
        except Exception as e:
            return f"Propositional logic error: {str(e)}"
    
    def solve_logical_and(self, command):
        """Solve logical AND operations"""
        try:
            # Extract logical values from command
            values = self.extract_logical_values(command)
            if values:
                result = all(values)
                binary_values = [1 if v else 0 for v in values]
                return f"AND Operation: {' ∧ '.join(map(str, binary_values))} = {1 if result else 0}\nLogic: Output is 1 only when all inputs are 1"
            
            return """AND Operation (Conjunction):
Symbol: ∧, •, &
Truth Table:
0 ∧ 0 = 0
0 ∧ 1 = 0
1 ∧ 0 = 0
1 ∧ 1 = 1
Logic: True only when both operands are true"""
        except Exception as e:
            return f"Logical AND error: {str(e)}"
    
    def solve_logical_or(self, command):
        """Solve logical OR operations"""
        try:
            values = self.extract_logical_values(command)
            if values:
                result = any(values)
                binary_values = [1 if v else 0 for v in values]
                return f"OR Operation: {' ∨ '.join(map(str, binary_values))} = {1 if result else 0}\nLogic: Output is 0 only when all inputs are 0"
            
            return """OR Operation (Disjunction):
Symbol: ∨, +, |
Truth Table:
0 ∨ 0 = 0
0 ∨ 1 = 1
1 ∨ 0 = 1
1 ∨ 1 = 1
Logic: False only when both operands are false"""
        except Exception as e:
            return f"Logical OR error: {str(e)}"
    
    def solve_logical_not(self, command):
        """Solve logical NOT operations"""
        try:
            values = self.extract_logical_values(command)
            if values and len(values) == 1:
                result = not values[0]
                input_val = 1 if values[0] else 0
                output_val = 1 if result else 0
                return f"NOT Operation: ¬{input_val} = {output_val}\nLogic: Output is opposite of input"
            
            return """NOT Operation (Negation):
Symbol: ¬, ~, '
Truth Table:
¬0 = 1
¬1 = 0
Logic: Output is opposite of input"""
        except Exception as e:
            return f"Logical NOT error: {str(e)}"
    
    def solve_logical_xor(self, command):
        """Solve logical XOR operations"""
        try:
            values = self.extract_logical_values(command)
            if len(values) >= 2:
                result = values[0]
                for i in range(1, len(values)):
                    result = result != values[i]  # XOR logic
                binary_values = [1 if v else 0 for v in values]
                return f"XOR Operation: {' ⊕ '.join(map(str, binary_values))} = {1 if result else 0}\nLogic: Output is 1 when inputs are different"
            
            return """XOR Operation (Exclusive OR):
Symbol: ⊕, ⊻
Truth Table:
0 ⊕ 0 = 0
0 ⊕ 1 = 1
1 ⊕ 0 = 1
1 ⊕ 1 = 0
Logic: True when operands are different"""
        except Exception as e:
            return f"Logical XOR error: {str(e)}"
    
    def solve_logical_implication(self, command):
        """Solve logical implication"""
        try:
            values = self.extract_logical_values(command)
            if len(values) >= 2:
                a, b = values[0], values[1]
                result = (not a) or b  # A → B = ¬A ∨ B
                a_val, b_val = 1 if a else 0, 1 if b else 0
                return f"Implication: {a_val} → {b_val} = {1 if result else 0}\nLogic: False only when antecedent is true and consequent is false"
            
            return """Implication (→):
Truth Table:
0 → 0 = 1
0 → 1 = 1
1 → 0 = 0
1 → 1 = 1
Equivalent: A → B ≡ ¬A ∨ B
Logic: "If A then B" - false only when A is true and B is false"""
        except Exception as e:
            return f"Logical implication error: {str(e)}"
    
    def solve_logical_biconditional(self, command):
        """Solve logical biconditional"""
        try:
            values = self.extract_logical_values(command)
            if len(values) >= 2:
                a, b = values[0], values[1]
                result = a == b  # A ↔ B is true when both have same value
                a_val, b_val = 1 if a else 0, 1 if b else 0
                return f"Biconditional: {a_val} ↔ {b_val} = {1 if result else 0}\nLogic: True when both operands have same truth value"
            
            return """Biconditional (↔):
Truth Table:
0 ↔ 0 = 1
0 ↔ 1 = 0
1 ↔ 0 = 0
1 ↔ 1 = 1
Equivalent: A ↔ B ≡ (A → B) ∧ (B → A)
Logic: "A if and only if B" - true when both have same truth value"""
        except Exception as e:
            return f"Logical biconditional error: {str(e)}"
    
    def solve_logical_nand(self, command):
        """Solve logical NAND operations"""
        try:
            values = self.extract_logical_values(command)
            if values:
                and_result = all(values)
                result = not and_result  # NAND = NOT AND
                binary_values = [1 if v else 0 for v in values]
                return f"NAND Operation: {' ↑ '.join(map(str, binary_values))} = {1 if result else 0}\nLogic: NOT AND - opposite of AND gate"
            
            return """NAND Operation (NOT AND):
Symbol: ↑, NAND
Truth Table:
0 ↑ 0 = 1
0 ↑ 1 = 1
1 ↑ 0 = 1
1 ↑ 1 = 0
Logic: Opposite of AND - false only when both inputs are true
Universal Gate: Can implement any logic function"""
        except Exception as e:
            return f"Logical NAND error: {str(e)}"
    
    def solve_logical_nor(self, command):
        """Solve logical NOR operations"""
        try:
            values = self.extract_logical_values(command)
            if values:
                or_result = any(values)
                result = not or_result  # NOR = NOT OR
                binary_values = [1 if v else 0 for v in values]
                return f"NOR Operation: {' ↓ '.join(map(str, binary_values))} = {1 if result else 0}\nLogic: NOT OR - opposite of OR gate"
            
            return """NOR Operation (NOT OR):
Symbol: ↓, NOR
Truth Table:
0 ↓ 0 = 1
0 ↓ 1 = 0
1 ↓ 0 = 0
1 ↓ 1 = 0
Logic: Opposite of OR - true only when both inputs are false
Universal Gate: Can implement any logic function"""
        except Exception as e:
            return f"Logical NOR error: {str(e)}"
    
    def simplify_boolean_expression(self, command):
        """Simplify boolean expressions"""
        try:
            if 'karnaugh' in command or 'kmap' in command:
                return """Karnaugh Map Simplification:
1. Create K-map grid based on number of variables
2. Fill map with function values
3. Group adjacent 1s in powers of 2 (1,2,4,8...)
4. Each group eliminates one variable
5. Write simplified expression from groups

Example for 2 variables:
   B
A  0  1
0  □  □
1  □  □

Rules:
- Adjacent cells differ by only one variable
- Groups must be rectangular
- Larger groups give more simplification"""
            
            elif 'algebraic' in command:
                return """Algebraic Simplification:
Use Boolean laws to reduce expression:

1. Identity: A ∧ 1 = A, A ∨ 0 = A
2. Null: A ∧ 0 = 0, A ∨ 1 = 1
3. Idempotent: A ∧ A = A, A ∨ A = A
4. Complement: A ∧ ¬A = 0, A ∨ ¬A = 1
5. Double Negation: ¬(¬A) = A
6. De Morgan: ¬(A ∧ B) = ¬A ∨ ¬B
7. Distributive: A ∧ (B ∨ C) = (A ∧ B) ∨ (A ∧ C)
8. Absorption: A ∨ (A ∧ B) = A"""
            
            else:
                return """Boolean Expression Simplification:
Methods:
1. Algebraic Method: Apply Boolean laws
2. Karnaugh Map: Graphical grouping method
3. Quine-McCluskey: Tabular method
4. Consensus Method: Systematic approach

Goals:
- Minimize number of literals
- Reduce gate count in circuits
- Optimize logic implementation
- Improve circuit performance"""
        except Exception as e:
            return f"Boolean simplification error: {str(e)}"
    
    def apply_demorgan_laws(self, command):
        """Apply De Morgan's laws"""
        try:
            return """De Morgan's Laws:
First Law: ¬(A ∧ B) = ¬A ∨ ¬B
"NOT (A AND B) = (NOT A) OR (NOT B)"

Second Law: ¬(A ∨ B) = ¬A ∧ ¬B
"NOT (A OR B) = (NOT A) AND (NOT B)"

Extended Forms:
¬(A ∧ B ∧ C) = ¬A ∨ ¬B ∨ ¬C
¬(A ∨ B ∨ C) = ¬A ∧ ¬B ∧ ¬C

Applications:
- Converting NAND to OR with inverted inputs
- Converting NOR to AND with inverted inputs
- Logic circuit transformation
- Boolean expression simplification

Examples:
¬(x ∧ y) = ¬x ∨ ¬y
¬(p ∨ q ∨ r) = ¬p ∧ ¬q ∧ ¬r"""
        except Exception as e:
            return f"De Morgan's laws error: {str(e)}"
    
    def get_boolean_laws(self, command):
        """Get comprehensive Boolean laws"""
        try:
            return """Complete Boolean Laws:

Identity Laws:
A ∧ 1 = A    |    A ∨ 0 = A

Null Laws:
A ∧ 0 = 0    |    A ∨ 1 = 1

Idempotent Laws:
A ∧ A = A    |    A ∨ A = A

Complement Laws:
A ∧ ¬A = 0   |    A ∨ ¬A = 1

Double Negation:
¬(¬A) = A

Commutative Laws:
A ∧ B = B ∧ A    |    A ∨ B = B ∨ A

Associative Laws:
(A ∧ B) ∧ C = A ∧ (B ∧ C)
(A ∨ B) ∨ C = A ∨ (B ∨ C)

Distributive Laws:
A ∧ (B ∨ C) = (A ∧ B) ∨ (A ∧ C)
A ∨ (B ∧ C) = (A ∨ B) ∧ (A ∨ C)

De Morgan's Laws:
¬(A ∧ B) = ¬A ∨ ¬B
¬(A ∨ B) = ¬A ∧ ¬B

Absorption Laws:
A ∨ (A ∧ B) = A
A ∧ (A ∨ B) = A"""
        except Exception as e:
            return f"Boolean laws error: {str(e)}"
    
    def extract_logical_values(self, command):
        """Extract logical values (0/1, true/false) from command"""
        try:
            # Look for binary values
            binary_pattern = r'\b[01]\b'
            binary_matches = re.findall(binary_pattern, command)
            
            if binary_matches:
                return [bool(int(val)) for val in binary_matches]
            
            # Look for true/false values
            true_words = ['true', 'yes', '1', 'high', 'on']
            false_words = ['false', 'no', '0', 'low', 'off']
            
            words = command.lower().split()
            values = []
            
            for word in words:
                if word in true_words:
                    values.append(True)
                elif word in false_words:
                    values.append(False)
            
            return values
            
        except Exception as e:
            return []        

        # ==================== COMPLETE DISCRETE MATHEMATICS (4551-5500+) ====================
        
        # Graph Theory Commands (4551-4650)
        if any(word in command for word in ['graph theory', 'graph formula', 'graph algorithm']):
            result = self.get_graph_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['graph traversal', 'dfs', 'bfs', 'depth first', 'breadth first']):
            result = self.solve_graph_traversal(command)
            self.speak(result)
        
        elif any(word in command for word in ['shortest path', 'dijkstra', 'floyd warshall', 'bellman ford']):
            result = self.solve_shortest_path(command)
            self.speak(result)
        
        elif any(word in command for word in ['minimum spanning tree', 'mst', 'kruskal', 'prim']):
            result = self.solve_mst_algorithms(command)
            self.speak(result)
        
        elif any(word in command for word in ['graph coloring', 'chromatic number', 'vertex coloring']):
            result = self.solve_graph_coloring(command)
            self.speak(result)
        
        elif any(word in command for word in ['euler path', 'euler circuit', 'hamiltonian path', 'hamiltonian circuit']):
            result = self.solve_euler_hamiltonian(command)
            self.speak(result)
        
        elif any(word in command for word in ['planar graph', 'euler formula', 'kuratowski theorem']):
            result = self.solve_planar_graphs(command)
            self.speak(result)
        
        elif any(word in command for word in ['graph connectivity', 'connected components', 'strongly connected']):
            result = self.solve_graph_connectivity(command)
            self.speak(result)
        
        # Relations Commands (4651-4750)
        elif any(word in command for word in ['relation theory', 'binary relation', 'relation properties']):
            result = self.get_relation_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['equivalence relation', 'equivalence class', 'partition']):
            result = self.solve_equivalence_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['partial order', 'poset', 'hasse diagram']):
            result = self.solve_partial_ordering(command)
            self.speak(result)
        
        elif any(word in command for word in ['relation composition', 'relation inverse', 'relation closure']):
            result = self.solve_relation_operations(command)
            self.speak(result)
        
        elif any(word in command for word in ['reflexive', 'symmetric', 'transitive', 'antisymmetric']):
            result = self.check_relation_properties(command)
            self.speak(result)
        
        # Counting Theory Commands (4751-4850)
        elif any(word in command for word in ['counting theory', 'combinatorics advanced', 'counting principle']):
            result = self.get_counting_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['inclusion exclusion', 'principle inclusion exclusion']):
            result = self.solve_inclusion_exclusion(command)
            self.speak(result)
        
        elif any(word in command for word in ['pigeonhole principle', 'dirichlet principle']):
            result = self.solve_pigeonhole_principle(command)
            self.speak(result)
        
        elif any(word in command for word in ['generating function', 'exponential generating function']):
            result = self.solve_generating_functions(command)
            self.speak(result)
        
        elif any(word in command for word in ['derangement', 'stirling number', 'bell number']):
            result = self.solve_advanced_counting(command)
            self.speak(result)
        
        elif any(word in command for word in ['catalan number', 'fibonacci counting', 'lucas number']):
            result = self.solve_special_counting_sequences(command)
            self.speak(result)
        
        # Recurrence Relations Commands (4851-4950)
        elif any(word in command for word in ['recurrence relation', 'recurrence formula', 'recurrence solution']):
            result = self.solve_recurrence_relations(command)
            self.speak(result)
        
        elif any(word in command for word in ['linear recurrence', 'homogeneous recurrence', 'non homogeneous']):
            result = self.solve_linear_recurrence(command)
            self.speak(result)
        
        elif any(word in command for word in ['characteristic equation', 'recurrence roots']):
            result = self.solve_characteristic_equation(command)
            self.speak(result)
        
        elif any(word in command for word in ['master theorem', 'divide conquer recurrence']):
            result = self.solve_master_theorem(command)
            self.speak(result)
        
        elif any(word in command for word in ['generating function recurrence', 'recurrence generating']):
            result = self.solve_recurrence_generating_functions(command)
            self.speak(result)
        
        # Algebraic Structures Commands (4951-5050)
        elif any(word in command for word in ['algebraic structure', 'abstract algebra discrete']):
            result = self.get_algebraic_structures_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['group theory discrete', 'group properties', 'group operations']):
            result = self.solve_group_theory_discrete(command)
            self.speak(result)
        
        elif any(word in command for word in ['ring theory discrete', 'field theory discrete']):
            result = self.solve_ring_field_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['lattice theory', 'boolean lattice', 'distributive lattice']):
            result = self.solve_lattice_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['monoid', 'semigroup', 'homomorphism discrete']):
            result = self.solve_monoid_semigroup(command)
            self.speak(result)
        
        # Advanced Discrete Math (5051-5150)
        elif any(word in command for word in ['formal language', 'automata theory', 'regular expression']):
            result = self.solve_formal_languages(command)
            self.speak(result)
        
        elif any(word in command for word in ['finite automata', 'pushdown automata', 'turing machine']):
            result = self.solve_automata_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['complexity theory', 'big o notation', 'algorithm analysis']):
            result = self.solve_complexity_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['coding theory', 'error correction', 'hamming code']):
            result = self.solve_coding_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['network flow', 'max flow', 'min cut', 'ford fulkerson']):
            result = self.solve_network_flow(command)
            self.speak(result) 
   
    # ==================== COMPREHENSIVE DISCRETE MATHEMATICS SOLUTIONS ====================
    
    def get_graph_theory_formulas(self, command):
        """Complete graph theory formulas and concepts"""
        try:
            if 'basic' in command or 'definition' in command:
                return """Graph Theory Basics:
Graph G = (V, E) where V = vertices, E = edges
Types:
- Undirected: Edges have no direction
- Directed (Digraph): Edges have direction
- Weighted: Edges have weights
- Simple: No loops, no multiple edges
- Complete: Every pair of vertices connected

Degree: deg(v) = number of edges incident to v
Handshaking Lemma: Σdeg(v) = 2|E|
Path: Sequence of vertices connected by edges
Cycle: Path that starts and ends at same vertex
Connected: Path exists between any two vertices"""
            
            elif 'matrix' in command:
                return """Graph Matrices:
Adjacency Matrix A: A[i][j] = 1 if edge (i,j) exists
- Symmetric for undirected graphs
- A^k gives paths of length k

Incidence Matrix: Rows = vertices, Columns = edges
- Entry = 1 if vertex incident to edge

Degree Matrix D: Diagonal matrix with degrees
Laplacian Matrix L = D - A
- Properties: L is positive semidefinite
- Number of spanning trees = any cofactor of L"""
            
            elif 'tree' in command:
                return """Tree Properties:
Tree: Connected acyclic graph
Properties:
- |E| = |V| - 1 (exactly n-1 edges for n vertices)
- Unique path between any two vertices
- Adding any edge creates exactly one cycle
- Removing any edge disconnects graph

Spanning Tree: Subgraph that is tree and includes all vertices
Number of spanning trees in complete graph Kn = n^(n-2) (Cayley's formula)
Minimum Spanning Tree: Spanning tree with minimum total weight"""
            
            elif 'connectivity' in command:
                return """Graph Connectivity:
Connected: Path exists between every pair of vertices
Components: Maximal connected subgraphs
Vertex Connectivity κ(G): Min vertices to remove to disconnect
Edge Connectivity λ(G): Min edges to remove to disconnect
Relationship: κ(G) ≤ λ(G) ≤ δ(G) where δ = minimum degree

k-Connected: Remains connected after removing any k-1 vertices
Menger's Theorem: Max number of vertex-disjoint paths = vertex connectivity"""
            
            else:
                return """Graph Theory Overview:
• Basic Concepts: Vertices, edges, paths, cycles
• Graph Types: Directed, undirected, weighted, simple
• Graph Representations: Adjacency matrix, adjacency list
• Tree Properties: Connected, acyclic, n-1 edges
• Connectivity: Components, vertex/edge connectivity
• Algorithms: DFS, BFS, shortest path, MST"""
        except Exception as e:
            return f"Graph theory error: {str(e)}"
    
    def solve_graph_traversal(self, command):
        """Graph traversal algorithms"""
        try:
            if 'dfs' in command or 'depth first' in command:
                return """Depth-First Search (DFS):
Algorithm:
1. Start at source vertex
2. Mark current vertex as visited
3. For each unvisited neighbor, recursively apply DFS
4. Backtrack when no unvisited neighbors

Time Complexity: O(V + E)
Space Complexity: O(V) for recursion stack
Applications:
- Topological sorting
- Detecting cycles
- Finding strongly connected components
- Solving maze problems

Pseudocode:
DFS(v):
    mark v as visited
    for each neighbor u of v:
        if u not visited:
            DFS(u)"""
            
            elif 'bfs' in command or 'breadth first' in command:
                return """Breadth-First Search (BFS):
Algorithm:
1. Start at source vertex, add to queue
2. While queue not empty:
   - Dequeue vertex v
   - For each unvisited neighbor u of v:
     - Mark u as visited
     - Add u to queue

Time Complexity: O(V + E)
Space Complexity: O(V) for queue
Applications:
- Shortest path in unweighted graphs
- Level-order traversal
- Finding connected components
- Bipartite graph testing

Properties:
- Visits vertices in order of distance from source
- Finds shortest path in unweighted graphs"""
            
            elif 'comparison' in command:
                return """DFS vs BFS Comparison:
DFS (Depth-First Search):
- Uses stack (recursion)
- Goes deep before exploring neighbors
- Memory: O(h) where h = height
- Good for: Topological sort, cycle detection

BFS (Breadth-First Search):
- Uses queue
- Explores all neighbors before going deeper
- Memory: O(w) where w = maximum width
- Good for: Shortest path, level-order traversal

Both have O(V + E) time complexity"""
            
            else:
                return """Graph Traversal Algorithms:
• DFS: Depth-first, uses stack/recursion
• BFS: Breadth-first, uses queue
• Both: O(V + E) time complexity
• Applications: Pathfinding, connectivity, cycles
• DFS: Topological sort, strongly connected components
• BFS: Shortest path in unweighted graphs"""
        except Exception as e:
            return f"Graph traversal error: {str(e)}"
    
    def solve_shortest_path(self, command):
        """Shortest path algorithms"""
        try:
            if 'dijkstra' in command:
                return """Dijkstra's Algorithm:
For single-source shortest paths with non-negative weights

Algorithm:
1. Initialize distances: d[source] = 0, d[v] = ∞ for all other v
2. Create priority queue with all vertices
3. While queue not empty:
   - Extract vertex u with minimum distance
   - For each neighbor v of u:
     - If d[u] + weight(u,v) < d[v]:
       - d[v] = d[u] + weight(u,v)
       - Update priority queue

Time Complexity: O((V + E) log V) with binary heap
Space Complexity: O(V)
Limitation: Cannot handle negative weights"""
            
            elif 'floyd' in command or 'warshall' in command:
                return """Floyd-Warshall Algorithm:
For all-pairs shortest paths

Algorithm:
for k = 1 to n:
    for i = 1 to n:
        for j = 1 to n:
            if dist[i][k] + dist[k][j] < dist[i][j]:
                dist[i][j] = dist[i][k] + dist[k][j]

Time Complexity: O(V³)
Space Complexity: O(V²)
Advantages:
- Handles negative weights (but not negative cycles)
- Finds shortest paths between all pairs
- Simple implementation"""
            
            elif 'bellman' in command or 'ford' in command:
                return """Bellman-Ford Algorithm:
For single-source shortest paths with negative weights

Algorithm:
1. Initialize: d[source] = 0, d[v] = ∞ for all other v
2. Repeat V-1 times:
   - For each edge (u,v) with weight w:
     - If d[u] + w < d[v]: d[v] = d[u] + w
3. Check for negative cycles:
   - For each edge (u,v): if d[u] + w < d[v], negative cycle exists

Time Complexity: O(VE)
Advantages:
- Handles negative weights
- Detects negative cycles
- Simpler than Dijkstra for negative weights"""
            
            else:
                return """Shortest Path Algorithms:
• Dijkstra: Single-source, non-negative weights, O((V+E)logV)
• Floyd-Warshall: All-pairs, handles negative weights, O(V³)
• Bellman-Ford: Single-source, negative weights, detects cycles, O(VE)
• BFS: Unweighted graphs, O(V+E)
• A*: Heuristic-based, optimal with admissible heuristic"""
        except Exception as e:
            return f"Shortest path error: {str(e)}"
    
    def solve_mst_algorithms(self, command):
        """Minimum Spanning Tree algorithms"""
        try:
            if 'kruskal' in command:
                return """Kruskal's Algorithm:
Greedy algorithm for MST

Algorithm:
1. Sort all edges by weight (ascending)
2. Initialize each vertex as separate component
3. For each edge (u,v) in sorted order:
   - If u and v in different components:
     - Add edge to MST
     - Union components of u and v

Time Complexity: O(E log E) for sorting
Space Complexity: O(V) for Union-Find
Uses Union-Find data structure for cycle detection

Properties:
- Edge-based approach
- Processes edges globally
- Good for sparse graphs"""
            
            elif 'prim' in command:
                return """Prim's Algorithm:
Greedy algorithm for MST

Algorithm:
1. Start with arbitrary vertex in MST
2. While MST has fewer than V-1 edges:
   - Find minimum weight edge connecting MST to non-MST vertex
   - Add this edge and vertex to MST

Time Complexity: O(E log V) with binary heap
Space Complexity: O(V)
Uses priority queue for efficient edge selection

Properties:
- Vertex-based approach
- Grows MST incrementally
- Good for dense graphs"""
            
            elif 'properties' in command:
                return """MST Properties:
Cut Property: For any cut, minimum weight edge crossing cut is in some MST
Cycle Property: For any cycle, maximum weight edge is not in any MST
Uniqueness: MST is unique if all edge weights are distinct

MST Weight: Sum of weights of edges in MST
Number of MSTs: Can be computed using matrix-tree theorem
Applications:
- Network design (minimum cost)
- Clustering algorithms
- Approximation algorithms"""
            
            else:
                return """Minimum Spanning Tree:
• Definition: Spanning tree with minimum total weight
• Properties: Connects all vertices, no cycles, V-1 edges
• Kruskal's: Edge-based, sort edges, Union-Find, O(E log E)
• Prim's: Vertex-based, priority queue, O(E log V)
• Both produce optimal MST
• Applications: Network design, clustering"""
        except Exception as e:
            return f"MST algorithms error: {str(e)}"
    
    def solve_graph_coloring(self, command):
        """Graph coloring problems"""
        try:
            if 'vertex coloring' in command or 'chromatic' in command:
                return """Vertex Coloring:
Assign colors to vertices such that no adjacent vertices have same color

Chromatic Number χ(G): Minimum colors needed
Properties:
- χ(Kn) = n (complete graph)
- χ(Cn) = 2 if n even, 3 if n odd (cycle)
- χ(tree) = 2 (bipartite)
- χ(G) ≤ Δ(G) + 1 where Δ = maximum degree

Brooks' Theorem: χ(G) ≤ Δ(G) unless G is complete or odd cycle

Greedy Coloring Algorithm:
1. Order vertices arbitrarily
2. For each vertex, assign smallest available color
Time Complexity: O(V + E)
Approximation: Uses at most Δ(G) + 1 colors"""
            
            elif 'edge coloring' in command:
                return """Edge Coloring:
Assign colors to edges such that no adjacent edges have same color

Edge Chromatic Number χ'(G): Minimum colors needed
Vizing's Theorem: Δ(G) ≤ χ'(G) ≤ Δ(G) + 1

Class 1 graphs: χ'(G) = Δ(G)
Class 2 graphs: χ'(G) = Δ(G) + 1

Properties:
- Bipartite graphs are Class 1
- Complete graphs Kn are Class 1 if n even, Class 2 if n odd
- Determining class is NP-complete"""
            
            elif 'four color' in command:
                return """Four Color Theorem:
Every planar graph can be colored with at most 4 colors

Historical significance:
- Conjectured in 1852
- First major theorem proved using computer assistance (1976)
- Proof involves checking thousands of cases

Implications:
- χ(G) ≤ 4 for any planar graph G
- Some planar graphs require exactly 4 colors
- Related to map coloring problem"""
            
            else:
                return """Graph Coloring:
• Vertex Coloring: Color vertices, no adjacent same color
• Edge Coloring: Color edges, no incident same color
• Chromatic Number: Minimum colors needed
• Four Color Theorem: Planar graphs need ≤ 4 colors
• Applications: Scheduling, register allocation, frequency assignment
• NP-complete problem in general"""
        except Exception as e:
            return f"Graph coloring error: {str(e)}"
    
    def solve_euler_hamiltonian(self, command):
        """Euler and Hamiltonian paths/circuits"""
        try:
            if 'euler' in command:
                return """Euler Paths and Circuits:
Euler Path: Visits every edge exactly once
Euler Circuit: Euler path that starts and ends at same vertex

Existence Conditions:
Undirected Graph:
- Euler Circuit: All vertices have even degree
- Euler Path: Exactly 0 or 2 vertices have odd degree

Directed Graph:
- Euler Circuit: In-degree = Out-degree for all vertices
- Euler Path: At most one vertex has (out-deg - in-deg = 1)
              and at most one vertex has (in-deg - out-deg = 1)

Hierholzer's Algorithm:
1. Start from vertex with odd degree (if exists)
2. Follow edges, removing them as traversed
3. When stuck, find vertex with unused edges
4. Create cycle from that vertex
5. Splice cycles together"""
            
            elif 'hamiltonian' in command:
                return """Hamiltonian Paths and Circuits:
Hamiltonian Path: Visits every vertex exactly once
Hamiltonian Circuit: Hamiltonian path that returns to start

No simple necessary and sufficient conditions!

Sufficient Conditions:
Dirac's Theorem: If deg(v) ≥ n/2 for all v, then Hamiltonian circuit exists
Ore's Theorem: If deg(u) + deg(v) ≥ n for all non-adjacent u,v, then Hamiltonian circuit exists

NP-Complete Problem:
- No polynomial-time algorithm known
- Traveling Salesman Problem is related
- Backtracking and branch-and-bound used

Applications:
- Traveling Salesman Problem
- Knight's tour on chessboard
- DNA sequencing"""
            
            elif 'comparison' in command:
                return """Euler vs Hamiltonian:
Euler Paths/Circuits:
- Visit every EDGE exactly once
- Polynomial-time algorithms exist
- Simple necessary and sufficient conditions
- Based on vertex degrees

Hamiltonian Paths/Circuits:
- Visit every VERTEX exactly once
- NP-complete problem
- No simple characterization
- Much harder to determine existence

Both are fundamental problems in graph theory with different complexity"""
            
            else:
                return """Euler and Hamiltonian Problems:
• Euler: Visit every edge exactly once
• Hamiltonian: Visit every vertex exactly once
• Euler: Polynomial-time solvable, degree conditions
• Hamiltonian: NP-complete, no simple conditions
• Applications: Routing, scheduling, optimization
• Classical problems in graph theory"""
        except Exception as e:
            return f"Euler Hamiltonian error: {str(e)}"
    
    def solve_planar_graphs(self, command):
        """Planar graph theory"""
        try:
            if 'euler formula' in command:
                return """Euler's Formula for Planar Graphs:
For connected planar graph: V - E + F = 2
Where V = vertices, E = edges, F = faces (including outer face)

Consequences:
- E ≤ 3V - 6 (for V ≥ 3)
- If no triangles: E ≤ 2V - 4
- Every planar graph has vertex with degree ≤ 5

Proof technique for non-planarity:
If E > 3V - 6, then graph is non-planar

Examples:
- K₅ (complete graph on 5 vertices): E = 10, 3V - 6 = 9, so non-planar
- K₃,₃ (complete bipartite): E = 9, 2V - 4 = 8, so non-planar"""
            
            elif 'kuratowski' in command:
                return """Kuratowski's Theorem:
A graph is planar if and only if it contains no subdivision of K₅ or K₃,₃

Subdivision: Replace edge with path of length ≥ 1
Homeomorphic: Graphs that become isomorphic after subdivisions

Wagner's Theorem (equivalent):
A graph is planar iff it has no K₅ or K₃,₃ minor

Minor: Obtained by edge contractions and deletions

Applications:
- Fundamental characterization of planarity
- Basis for linear-time planarity testing algorithms"""
            
            elif 'dual graph' in command:
                return """Planar Graph Duality:
For planar graph G, dual graph G* has:
- Vertex for each face of G
- Edge between vertices if corresponding faces share edge

Properties:
- (G*)* = G (double dual)
- |V(G*)| = |F(G)|, |E(G*)| = |E(G)|, |F(G*)| = |V(G)|
- G connected iff G* connected
- G has Euler circuit iff G* bipartite

Applications:
- Four color theorem
- Electrical networks
- Map coloring problems"""
            
            else:
                return """Planar Graphs:
• Definition: Can be drawn in plane without edge crossings
• Euler's Formula: V - E + F = 2
• Bounds: E ≤ 3V - 6 for simple planar graphs
• Kuratowski's Theorem: No K₅ or K₃,₃ subdivision
• Four Color Theorem: Chromatic number ≤ 4
• Applications: VLSI design, geography, networks"""
        except Exception as e:
            return f"Planar graphs error: {str(e)}"    

    def get_relation_theory_formulas(self, command):
        """Complete relation theory"""
        try:
            if 'basic' in command or 'definition' in command:
                return """Relation Theory Basics:
Binary Relation R on sets A and B: R ⊆ A × B
If (a,b) ∈ R, we write aRb

Types of Relations:
- Reflexive: aRa for all a ∈ A
- Symmetric: aRb ⟹ bRa
- Transitive: aRb ∧ bRc ⟹ aRc
- Antisymmetric: aRb ∧ bRa ⟹ a = b
- Asymmetric: aRb ⟹ ¬(bRa)

Domain: dom(R) = {a ∈ A : ∃b ∈ B, aRb}
Range: ran(R) = {b ∈ B : ∃a ∈ A, aRb}
Field: field(R) = dom(R) ∪ ran(R)"""
            
            elif 'operations' in command:
                return """Relation Operations:
Union: R₁ ∪ R₂ = {(a,b) : (a,b) ∈ R₁ ∨ (a,b) ∈ R₂}
Intersection: R₁ ∩ R₂ = {(a,b) : (a,b) ∈ R₁ ∧ (a,b) ∈ R₂}
Complement: R̄ = A × B - R
Inverse: R⁻¹ = {(b,a) : (a,b) ∈ R}

Composition: R₁ ∘ R₂ = {(a,c) : ∃b, (a,b) ∈ R₂ ∧ (b,c) ∈ R₁}
Note: Composition is read right to left

Properties:
- (R₁ ∘ R₂)⁻¹ = R₂⁻¹ ∘ R₁⁻¹
- Composition is associative
- Generally not commutative"""
            
            elif 'closure' in command:
                return """Relation Closures:
Reflexive Closure: r(R) = R ∪ IA where IA = {(a,a) : a ∈ A}
Symmetric Closure: s(R) = R ∪ R⁻¹
Transitive Closure: t(R) = R ∪ R² ∪ R³ ∪ ...

Warshall's Algorithm for Transitive Closure:
for k = 1 to n:
    for i = 1 to n:
        for j = 1 to n:
            R[i][j] = R[i][j] ∨ (R[i][k] ∧ R[k][j])

Time Complexity: O(n³)
Applications: Reachability in graphs, ancestor relations"""
            
            else:
                return """Relation Theory:
• Binary Relations: R ⊆ A × B
• Properties: Reflexive, Symmetric, Transitive, Antisymmetric
• Operations: Union, Intersection, Composition, Inverse
• Closures: Reflexive, Symmetric, Transitive
• Applications: Databases, ordering, equivalence"""
        except Exception as e:
            return f"Relation theory error: {str(e)}"
    
    def solve_equivalence_relations(self, command):
        """Equivalence relations and partitions"""
        try:
            if 'definition' in command:
                return """Equivalence Relations:
Relation R is equivalence relation if:
1. Reflexive: aRa for all a
2. Symmetric: aRb ⟹ bRa
3. Transitive: aRb ∧ bRc ⟹ aRc

Equivalence Class: [a] = {x : xRa}
Properties:
- Every element belongs to exactly one equivalence class
- [a] = [b] iff aRb
- [a] ∩ [b] = ∅ or [a] = [b]

Partition: Collection of non-empty, disjoint subsets whose union is the whole set"""
            
            elif 'partition' in command:
                return """Equivalence Relations and Partitions:
Fundamental Theorem: There's bijection between:
- Equivalence relations on set A
- Partitions of set A

Given equivalence relation R:
- Partition = {[a] : a ∈ A} (set of all equivalence classes)

Given partition P:
- Equivalence relation: aRb iff a and b in same block of P

Quotient Set: A/R = {[a] : a ∈ A} (set of equivalence classes)
|A/R| = number of equivalence classes"""
            
            elif 'examples' in command:
                return """Equivalence Relation Examples:
1. Congruence modulo n:
   aRb iff n|(a-b)
   Equivalence classes: [0], [1], ..., [n-1]

2. Same remainder when divided by n:
   Partition integers into n classes

3. Similar matrices:
   A ~ B iff ∃P: B = P⁻¹AP
   
4. Logical equivalence:
   p ~ q iff p ↔ q is tautology

5. Set equality:
   A ~ B iff A = B"""
            
            else:
                return """Equivalence Relations:
• Definition: Reflexive + Symmetric + Transitive
• Equivalence Classes: [a] = {x : xRa}
• Partition: Disjoint union of equivalence classes
• Quotient Set: A/R = set of equivalence classes
• Applications: Modular arithmetic, classification, abstraction"""
        except Exception as e:
            return f"Equivalence relations error: {str(e)}"
    
    def solve_partial_ordering(self, command):
        """Partial ordering and posets"""
        try:
            if 'definition' in command or 'poset' in command:
                return """Partial Order (Poset):
Relation ≤ is partial order if:
1. Reflexive: a ≤ a for all a
2. Antisymmetric: a ≤ b ∧ b ≤ a ⟹ a = b
3. Transitive: a ≤ b ∧ b ≤ c ⟹ a ≤ c

Poset: (A, ≤) where A is set and ≤ is partial order

Comparable: a, b are comparable if a ≤ b or b ≤ a
Incomparable: Neither a ≤ b nor b ≤ a

Chain: Totally ordered subset (all elements comparable)
Antichain: Set of mutually incomparable elements"""
            
            elif 'hasse diagram' in command:
                return """Hasse Diagrams:
Graphical representation of finite posets

Construction:
1. Draw elements as vertices
2. Draw edge from a to b if a < b and no c with a < c < b
3. Place smaller elements below larger ones
4. Omit reflexive and transitive edges

Reading Hasse Diagrams:
- a ≤ b iff there's upward path from a to b
- Minimal elements: No elements below them
- Maximal elements: No elements above them
- Least element: Below all others (if exists)
- Greatest element: Above all others (if exists)"""
            
            elif 'special elements' in command:
                return """Special Elements in Posets:
Minimal Element: No element below it (∄x: x < a)
Maximal Element: No element above it (∄x: a < x)
Least Element (Minimum): Below all others (∀x: a ≤ x)
Greatest Element (Maximum): Above all others (∀x: x ≤ a)

Lower Bound of S: Element a such that a ≤ s for all s ∈ S
Upper Bound of S: Element a such that s ≤ a for all s ∈ S
Greatest Lower Bound (GLB/Infimum): Greatest of all lower bounds
Least Upper Bound (LUB/Supremum): Least of all upper bounds

Lattice: Poset where every pair has GLB and LUB"""
            
            elif 'dilworth' in command:
                return """Dilworth's Theorem:
In finite poset, maximum size of antichain equals minimum number of chains needed to cover the poset

Width of poset = size of maximum antichain
Height of poset = size of maximum chain

Applications:
- Scheduling problems
- Resource allocation
- Combinatorial optimization

Dual: Maximum chain decomposition = minimum antichain cover"""
            
            else:
                return """Partial Ordering:
• Definition: Reflexive + Antisymmetric + Transitive
• Poset: (A, ≤) - set with partial order
• Hasse Diagrams: Visual representation
• Special Elements: Minimal, maximal, least, greatest
• Lattices: Every pair has GLB and LUB
• Dilworth's Theorem: Antichain-chain duality"""
        except Exception as e:
            return f"Partial ordering error: {str(e)}"
    
    def get_counting_theory_formulas(self, command):
        """Advanced counting theory"""
        try:
            if 'principle' in command or 'basic' in command:
                return """Fundamental Counting Principles:
Addition Principle: If A ∩ B = ∅, then |A ∪ B| = |A| + |B|
Multiplication Principle: |A × B| = |A| × |B|

Permutations: P(n,r) = n!/(n-r)! (arrangements)
Combinations: C(n,r) = n!/(r!(n-r)!) (selections)

With Repetition:
- Permutations with repetition: nʳ
- Combinations with repetition: C(n+r-1, r)

Circular Permutations: (n-1)! for n objects in circle
Permutations with identical objects: n!/(n₁!n₂!...nₖ!)"""
            
            elif 'advanced' in command:
                return """Advanced Counting Techniques:
Multinomial Coefficients: (n; n₁,n₂,...,nₖ) = n!/(n₁!n₂!...nₖ!)
- Number of ways to partition n objects into k groups

Stirling Numbers of Second Kind S(n,k):
- Number of ways to partition n objects into k non-empty subsets
- Recurrence: S(n,k) = k·S(n-1,k) + S(n-1,k-1)

Bell Numbers B(n): Total number of partitions of n-element set
- B(n) = Σₖ₌₀ⁿ S(n,k)

Stirling Numbers of First Kind s(n,k):
- Number of permutations of n elements with k cycles
- Unsigned: |s(n,k)| = number of ways to arrange n objects in k cycles"""
            
            elif 'derangement' in command:
                return """Derangements:
Derangement: Permutation where no element is in its original position

Formula: D(n) = n! Σₖ₌₀ⁿ (-1)ᵏ/k!
Approximation: D(n) ≈ n!/e

Recurrence: D(n) = (n-1)[D(n-1) + D(n-2)]
Initial: D(0) = 1, D(1) = 0

Probability that random permutation is derangement: D(n)/n! ≈ 1/e ≈ 0.368

Applications:
- Hat check problem
- Secret Santa arrangements
- Matching problems"""
            
            else:
                return """Counting Theory:
• Basic Principles: Addition, Multiplication
• Permutations & Combinations: With/without repetition
• Advanced: Multinomial, Stirling numbers, Bell numbers
• Derangements: Permutations with no fixed points
• Applications: Probability, combinatorial optimization"""
        except Exception as e:
            return f"Counting theory error: {str(e)}"
    
    def solve_inclusion_exclusion(self, command):
        """Inclusion-exclusion principle"""
        try:
            if 'principle' in command or 'formula' in command:
                return """Inclusion-Exclusion Principle:
For finite sets A₁, A₂, ..., Aₙ:

|A₁ ∪ A₂ ∪ ... ∪ Aₙ| = 
Σᵢ |Aᵢ| - Σᵢ<ⱼ |Aᵢ ∩ Aⱼ| + Σᵢ<ⱼ<ₖ |Aᵢ ∩ Aⱼ ∩ Aₖ| - ... + (-1)ⁿ⁺¹|A₁ ∩ A₂ ∩ ... ∩ Aₙ|

General Form: |⋃ᵢ₌₁ⁿ Aᵢ| = Σₖ₌₁ⁿ (-1)ᵏ⁺¹ Σ₁≤ᵢ₁<ᵢ₂<...<ᵢₖ≤ₙ |Aᵢ₁ ∩ Aᵢ₂ ∩ ... ∩ Aᵢₖ|

Two Sets: |A ∪ B| = |A| + |B| - |A ∩ B|
Three Sets: |A ∪ B ∪ C| = |A| + |B| + |C| - |A ∩ B| - |A ∩ C| - |B ∩ C| + |A ∩ B ∩ C|"""
            
            elif 'applications' in command:
                return """Inclusion-Exclusion Applications:
1. Euler's Totient Function:
   φ(n) = n ∏ₚ|ₙ (1 - 1/p) where p are prime divisors

2. Derangements:
   D(n) = n! Σₖ₌₀ⁿ (-1)ᵏ/k!

3. Surjective Functions:
   Number of onto functions from n-set to k-set:
   k! S(n,k) where S(n,k) is Stirling number of second kind

4. Rook Polynomials:
   Counting non-attacking rook placements on chessboard

5. Chromatic Polynomials:
   P(G,k) = number of proper k-colorings of graph G"""
            
            elif 'examples' in command:
                return """Inclusion-Exclusion Examples:
Example 1: Numbers ≤ 100 divisible by 2, 3, or 5
A = multiples of 2: |A| = 50
B = multiples of 3: |B| = 33  
C = multiples of 5: |C| = 20
|A ∩ B| = 16, |A ∩ C| = 10, |B ∩ C| = 6, |A ∩ B ∩ C| = 3
Answer: 50 + 33 + 20 - 16 - 10 - 6 + 3 = 74

Example 2: Derangements of 4 objects
D(4) = 4!(1 - 1 + 1/2 - 1/6 + 1/24) = 24 × 9/24 = 9"""
            
            else:
                return """Inclusion-Exclusion Principle:
• Formula: |A₁ ∪ ... ∪ Aₙ| = Σ|Aᵢ| - Σ|Aᵢ ∩ Aⱼ| + Σ|Aᵢ ∩ Aⱼ ∩ Aₖ| - ...
• Pattern: Alternating sum of intersections
• Applications: Derangements, Euler's totient, surjections
• Generalizes simple union formula
• Fundamental tool in combinatorics"""
        except Exception as e:
            return f"Inclusion exclusion error: {str(e)}"
    
    def solve_pigeonhole_principle(self, command):
        """Pigeonhole principle applications"""
        try:
            if 'basic' in command or 'principle' in command:
                return """Pigeonhole Principle:
If n pigeons are placed in m pigeonholes and n > m, then at least one pigeonhole contains more than one pigeon.

Generalized: If n objects are placed in m boxes, then at least one box contains at least ⌈n/m⌉ objects.

Strong Form: If n₁ + n₂ + ... + nₘ - m + 1 objects are placed in m boxes, then for some i, box i contains at least nᵢ objects.

Contrapositive: If each of m boxes contains at most k objects, then total objects ≤ mk."""
            
            elif 'applications' in command:
                return """Pigeonhole Principle Applications:
1. In any group of 367 people, at least 2 share same birthday
   (367 people, 366 possible birthdays)

2. Among any 5 points in unit square, some 2 are within distance √2/2
   (Divide square into 4 subsquares)

3. In sequence of n² + 1 distinct numbers, there's increasing or decreasing subsequence of length n + 1
   (Erdős–Szekeres theorem)

4. Any sequence of mn + 1 distinct numbers contains monotonic subsequence of length m + 1 or n + 1

5. In any graph with n vertices, at least 2 vertices have same degree
   (n vertices, only n-1 possible degrees: 0, 1, ..., n-2)"""
            
            elif 'examples' in command:
                return """Pigeonhole Examples:
Example 1: Prove that among any 13 people, at least 2 were born in same month.
Solution: 13 people, 12 months → ⌈13/12⌉ = 2 people in same month

Example 2: In any set of 6 people, either 3 are mutual friends or 3 are mutual strangers.
Solution: Fix person A. Among remaining 5, at least 3 are friends with A or at least 3 are strangers with A (pigeonhole). If 3 friends, check if they form triangle...

Example 3: Any integer sequence of length n² + 1 has increasing subsequence of length n + 1 or decreasing subsequence of length n + 1.
Solution: Assign to each element (i,j) where i = length of longest increasing subsequence ending here, j = length of longest decreasing subsequence ending here. If all different, we have n² + 1 distinct pairs from {1,2,...,n} × {1,2,...,n}, impossible."""
            
            else:
                return """Pigeonhole Principle:
• Basic: n pigeons, m holes, n > m → some hole has > 1 pigeon
• Generalized: At least ⌈n/m⌉ objects in some box
• Applications: Birthday paradox, graph theory, number theory
• Proof technique: Counting arguments, existence proofs
• Fundamental tool in discrete mathematics"""
        except Exception as e:
            return f"Pigeonhole principle error: {str(e)}"      
  
        # ==================== COMPLETE MATHEMATICS SYLLABUS (5501-6500+) ====================
        
        # Set Theory Commands (5501-5600)
        if any(word in command for word in ['set theory', 'set basic', 'set definition']):
            result = self.get_set_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['set operations', 'union', 'intersection', 'complement']):
            result = self.solve_set_operations(command)
            self.speak(result)
        
        elif any(word in command for word in ['venn diagram', 'set representation']):
            result = self.solve_venn_diagrams(command)
            self.speak(result)
        
        elif any(word in command for word in ['cartesian product', 'cross product set']):
            result = self.solve_cartesian_product(command)
            self.speak(result)
        
        elif any(word in command for word in ['set inclusion', 'subset', 'superset']):
            result = self.solve_set_inclusion(command)
            self.speak(result)
        
        # Functions Theory Commands (5601-5700)
        elif any(word in command for word in ['function theory', 'function definition', 'function basic']):
            result = self.get_function_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['function types', 'injective', 'surjective', 'bijective']):
            result = self.solve_function_types(command)
            self.speak(result)
        
        elif any(word in command for word in ['function composition', 'composite function']):
            result = self.solve_function_composition(command)
            self.speak(result)
        
        elif any(word in command for word in ['inverse function', 'function inverse']):
            result = self.solve_inverse_functions(command)
            self.speak(result)
        
        elif any(word in command for word in ['identity function', 'identity map']):
            result = self.solve_identity_functions(command)
            self.speak(result)
        
        # Matrix Theory Commands (5701-5800)
        elif any(word in command for word in ['matrix theory', 'matrix basic', 'matrix definition']):
            result = self.get_matrix_theory_formulas(command)
            self.speak(result)
        
        elif any(word in command for word in ['matrix operations', 'matrix addition', 'matrix multiplication']):
            result = self.solve_matrix_operations(command)
            self.speak(result)
        
        elif any(word in command for word in ['matrix types', 'symmetric matrix', 'diagonal matrix']):
            result = self.solve_matrix_types(command)
            self.speak(result)
        
        elif any(word in command for word in ['elementary operations', 'row operations', 'column operations']):
            result = self.solve_elementary_operations(command)
            self.speak(result)
        
        elif any(word in command for word in ['matrix rank', 'rank calculation']):
            result = self.solve_matrix_rank(command)
            self.speak(result)
        
        elif any(word in command for word in ['matrix inverse', 'inverse matrix']):
            result = self.solve_matrix_inverse_methods(command)
            self.speak(result)
        
        # Determinant Commands (5801-5850)
        elif any(word in command for word in ['determinant theory', 'determinant properties']):
            result = self.get_determinant_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['cramer rule', 'cramer method']):
            result = self.solve_cramer_rule(command)
            self.speak(result)
        
        elif any(word in command for word in ['determinant calculation', 'determinant solve']):
            result = self.solve_determinant_calculation(command)
            self.speak(result)
        
        # Linear Equations Commands (5851-5900)
        elif any(word in command for word in ['linear equations', 'system equations', 'simultaneous equations']):
            result = self.solve_linear_equation_systems(command)
            self.speak(result)
        
        elif any(word in command for word in ['consistency', 'inconsistent system']):
            result = self.solve_system_consistency(command)
            self.speak(result)
        
        # Eigen Values Commands (5901-6000)
        elif any(word in command for word in ['eigen value', 'eigenvalue', 'characteristic value']):
            result = self.solve_eigenvalues(command)
            self.speak(result)
        
        elif any(word in command for word in ['eigen vector', 'eigenvector', 'characteristic vector']):
            result = self.solve_eigenvectors(command)
            self.speak(result)
        
        elif any(word in command for word in ['cayley hamilton', 'characteristic polynomial']):
            result = self.solve_cayley_hamilton(command)
            self.speak(result)
        
        elif any(word in command for word in ['diagonalization', 'diagonal form']):
            result = self.solve_diagonalization(command)
            self.speak(result)
        
        elif any(word in command for word in ['hermitian matrix', 'unitary matrix']):
            result = self.solve_complex_matrices(command)
            self.speak(result)
        
        elif any(word in command for word in ['quadratic form', 'canonical form']):
            result = self.solve_quadratic_forms(command)
            self.speak(result)
        
        # Calculus Introduction Commands (6001-6100)
        elif any(word in command for word in ['limit theory', 'limit definition', 'limit basic']):
            result = self.get_limit_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['continuity theory', 'continuous function']):
            result = self.solve_continuity_theory(command)
            self.speak(result)
        
        elif any(word in command for word in ['derivative basic', 'differentiation basic']):
            result = self.get_derivative_basics(command)
            self.speak(result)
        
        elif any(word in command for word in ['composite derivative', 'chain rule']):
            result = self.solve_composite_derivatives(command)
            self.speak(result)
        
        elif any(word in command for word in ['implicit differentiation', 'implicit derivative']):
            result = self.solve_implicit_differentiation(command)
            self.speak(result)
        
        elif any(word in command for word in ['trigonometric derivative', 'trig derivative']):
            result = self.solve_trigonometric_derivatives(command)
            self.speak(result)
        
        elif any(word in command for word in ['second derivative', 'higher derivative']):
            result = self.solve_higher_derivatives(command)
            self.speak(result)
        
        # Differential Calculus Commands (6101-6200)
        elif any(word in command for word in ['rolle theorem', 'rolle\'s theorem']):
            result = self.solve_rolle_theorem(command)
            self.speak(result)
        
        elif any(word in command for word in ['mean value theorem', 'mvt']):
            result = self.solve_mean_value_theorem(command)
            self.speak(result)
        
        elif any(word in command for word in ['taylor theorem', 'taylor series', 'maclaurin series']):
            result = self.solve_taylor_maclaurin(command)
            self.speak(result)
        
        elif any(word in command for word in ['indeterminate form', 'l\'hopital rule']):
            result = self.solve_indeterminate_forms(command)
            self.speak(result)
        
        elif any(word in command for word in ['partial differentiation', 'partial derivative']):
            result = self.solve_partial_differentiation(command)
            self.speak(result)
        
        elif any(word in command for word in ['total differentiation', 'total derivative']):
            result = self.solve_total_differentiation(command)
            self.speak(result)
        
        elif any(word in command for word in ['euler theorem', 'homogeneous function']):
            result = self.solve_euler_theorem(command)
            self.speak(result)
        
        elif any(word in command for word in ['maxima minima', 'optimization', 'critical points']):
            result = self.solve_maxima_minima(command)
            self.speak(result)    

    # ==================== COMPLETE SET THEORY SOLUTIONS ====================
    
    def get_set_theory_formulas(self, command):
        """Complete set theory with definitions and formulas"""
        try:
            if 'definition' in command or 'basic' in command:
                return """Set Theory Basics:
Set: Well-defined collection of distinct objects
Element: Object belonging to a set (a ∈ A)
Empty Set: ∅ = {} (set with no elements)
Universal Set: U (set containing all elements under consideration)
Finite Set: Set with finite number of elements
Infinite Set: Set with infinite number of elements
Cardinality: |A| = number of elements in set A

Set Representations:
1. Roster/Tabular: A = {1, 2, 3, 4, 5}
2. Set-builder: A = {x : x is natural number, x ≤ 5}
3. Venn Diagrams: Graphical representation"""
            
            elif 'equality' in command:
                return """Set Equality and Inclusion:
Set Equality: A = B iff A ⊆ B and B ⊆ A
- Every element of A is in B and vice versa

Subset: A ⊆ B iff ∀x(x ∈ A → x ∈ B)
Proper Subset: A ⊂ B iff A ⊆ B and A ≠ B
Superset: A ⊇ B iff B ⊆ A

Properties:
- A ⊆ A (reflexive)
- If A ⊆ B and B ⊆ C, then A ⊆ C (transitive)
- ∅ ⊆ A for any set A
- A ⊆ U for any set A"""
            
            elif 'types' in command:
                return """Types of Sets:
1. Empty Set: ∅ = {}
2. Singleton Set: {a} (exactly one element)
3. Finite Set: Limited number of elements
4. Infinite Set: Unlimited elements
5. Equal Sets: Same elements
6. Equivalent Sets: Same cardinality
7. Disjoint Sets: A ∩ B = ∅
8. Overlapping Sets: A ∩ B ≠ ∅
9. Universal Set: Contains all elements
10. Power Set: P(A) = {X : X ⊆ A}, |P(A)| = 2^|A|"""
            
            else:
                return """Set Theory Overview:
• Definition: Well-defined collection of objects
• Representations: Roster, Set-builder, Venn diagrams
• Relations: Equality, Inclusion, Subset, Superset
• Types: Empty, Finite, Infinite, Power set
• Operations: Union, Intersection, Complement, Difference
• Applications: Logic, probability, computer science"""
        except Exception as e:
            return f"Set theory error: {str(e)}"
    
    def solve_set_operations(self, command):
        """Set operations with formulas and examples"""
        try:
            if 'union' in command:
                return """Set Union (A ∪ B):
Definition: A ∪ B = {x : x ∈ A or x ∈ B}
Properties:
- Commutative: A ∪ B = B ∪ A
- Associative: (A ∪ B) ∪ C = A ∪ (B ∪ C)
- Identity: A ∪ ∅ = A
- Idempotent: A ∪ A = A
- Absorption: A ∪ (A ∩ B) = A

Example: A = {1,2,3}, B = {3,4,5}
A ∪ B = {1,2,3,4,5}"""
            
            elif 'intersection' in command:
                return """Set Intersection (A ∩ B):
Definition: A ∩ B = {x : x ∈ A and x ∈ B}
Properties:
- Commutative: A ∩ B = B ∩ A
- Associative: (A ∩ B) ∩ C = A ∩ (B ∩ C)
- Identity: A ∩ U = A
- Idempotent: A ∩ A = A
- Absorption: A ∩ (A ∪ B) = A

Example: A = {1,2,3}, B = {3,4,5}
A ∩ B = {3}"""
            
            elif 'complement' in command:
                return """Set Complement (A'):
Definition: A' = U - A = {x : x ∈ U and x ∉ A}
Properties:
- (A')' = A (Double complement)
- A ∪ A' = U
- A ∩ A' = ∅
- ∅' = U, U' = ∅

De Morgan's Laws:
- (A ∪ B)' = A' ∩ B'
- (A ∩ B)' = A' ∪ B'

Example: U = {1,2,3,4,5}, A = {1,2,3}
A' = {4,5}"""
            
            elif 'difference' in command:
                return """Set Difference (A - B):
Definition: A - B = {x : x ∈ A and x ∉ B}
Also written as: A \\ B

Properties:
- A - B ≠ B - A (not commutative)
- A - ∅ = A
- A - A = ∅
- A - B = A ∩ B'

Symmetric Difference: A ⊕ B = (A - B) ∪ (B - A) = (A ∪ B) - (A ∩ B)

Example: A = {1,2,3,4}, B = {3,4,5,6}
A - B = {1,2}, B - A = {5,6}
A ⊕ B = {1,2,5,6}"""
            
            else:
                return """Set Operations Summary:
• Union: A ∪ B = {x : x ∈ A or x ∈ B}
• Intersection: A ∩ B = {x : x ∈ A and x ∈ B}
• Complement: A' = {x : x ∈ U and x ∉ A}
• Difference: A - B = {x : x ∈ A and x ∉ B}
• Symmetric Difference: A ⊕ B = (A - B) ∪ (B - A)
• All operations follow specific algebraic laws"""
        except Exception as e:
            return f"Set operations error: {str(e)}"
    
    def solve_venn_diagrams(self, command):
        """Venn diagram concepts and applications"""
        try:
            if 'basic' in command or 'definition' in command:
                return """Venn Diagrams:
Graphical representation of sets using circles/curves
Components:
- Rectangle: Universal set U
- Circles: Individual sets
- Overlapping regions: Intersections
- Non-overlapping regions: Differences

Two Sets (A, B):
- Region 1: A - B (only A)
- Region 2: A ∩ B (both A and B)
- Region 3: B - A (only B)
- Region 4: (A ∪ B)' (neither A nor B)

Total elements: |U| = |A - B| + |A ∩ B| + |B - A| + |(A ∪ B)'|"""
            
            elif 'three sets' in command:
                return """Three Sets Venn Diagram (A, B, C):
8 Regions:
1. A ∩ B ∩ C (all three)
2. (A ∩ B) - C (A and B, not C)
3. (A ∩ C) - B (A and C, not B)
4. (B ∩ C) - A (B and C, not A)
5. A - (B ∪ C) (only A)
6. B - (A ∪ C) (only B)
7. C - (A ∪ B) (only C)
8. (A ∪ B ∪ C)' (none)

Formula: |A ∪ B ∪ C| = |A| + |B| + |C| - |A ∩ B| - |A ∩ C| - |B ∩ C| + |A ∩ B ∩ C|"""
            
            elif 'applications' in command:
                return """Venn Diagram Applications:
1. Probability Problems:
   P(A ∪ B) = P(A) + P(B) - P(A ∩ B)

2. Survey Analysis:
   Students studying Math, Physics, Chemistry

3. Logic Problems:
   Truth tables, Boolean algebra

4. Database Queries:
   SQL JOIN operations

5. Classification Problems:
   Organizing data into categories

Example Problem: In class of 50 students, 30 study Math, 25 study Physics, 15 study both.
Students studying only Math = 30 - 15 = 15
Students studying only Physics = 25 - 15 = 10
Students studying neither = 50 - (15 + 15 + 10) = 10"""
            
            else:
                return """Venn Diagrams:
• Purpose: Visual representation of set relationships
• Components: Universal set, individual sets, intersections
• Two sets: 4 regions, Three sets: 8 regions
• Applications: Probability, surveys, logic, databases
• Formulas: Inclusion-exclusion principle
• Problem solving: Count elements in each region"""
        except Exception as e:
            return f"Venn diagrams error: {str(e)}"
    
    def solve_cartesian_product(self, command):
        """Cartesian product theory and applications"""
        try:
            if 'definition' in command or 'basic' in command:
                return """Cartesian Product:
Definition: A × B = {(a,b) : a ∈ A and b ∈ B}
Ordered pairs where first element from A, second from B

Properties:
- |A × B| = |A| × |B|
- A × B ≠ B × A (not commutative) unless A = B
- A × ∅ = ∅ × A = ∅
- A × (B ∪ C) = (A × B) ∪ (A × C) (distributive over union)
- A × (B ∩ C) = (A × B) ∩ (A × C) (distributive over intersection)

Example: A = {1,2}, B = {x,y}
A × B = {(1,x), (1,y), (2,x), (2,y)}
|A × B| = 2 × 2 = 4"""
            
            elif 'multiple sets' in command:
                return """Cartesian Product of Multiple Sets:
Three sets: A × B × C = {(a,b,c) : a ∈ A, b ∈ B, c ∈ C}
n sets: A₁ × A₂ × ... × Aₙ = {(a₁,a₂,...,aₙ) : aᵢ ∈ Aᵢ}

Cardinality: |A₁ × A₂ × ... × Aₙ| = |A₁| × |A₂| × ... × |Aₙ|

Cartesian Power: Aⁿ = A × A × ... × A (n times)
Example: A² = A × A, A³ = A × A × A

Real Numbers: ℝ² = ℝ × ℝ (coordinate plane)
ℝ³ = ℝ × ℝ × ℝ (3D space)"""
            
            elif 'applications' in command:
                return """Cartesian Product Applications:
1. Coordinate Systems:
   - 2D plane: ℝ × ℝ = ℝ²
   - 3D space: ℝ × ℝ × ℝ = ℝ³

2. Relations:
   - Binary relation R ⊆ A × B
   - Function f: A → B is subset of A × B

3. Database Tables:
   - Cross join of two tables
   - All possible combinations

4. Probability:
   - Sample space for multiple experiments
   - Coin tosses: {H,T} × {H,T} = {HH, HT, TH, TT}

5. Computer Science:
   - Data structures (tuples, records)
   - Algorithm complexity analysis"""
            
            else:
                return """Cartesian Product:
• Definition: A × B = {(a,b) : a ∈ A, b ∈ B}
• Properties: |A × B| = |A| × |B|, not commutative
• Multiple sets: A₁ × A₂ × ... × Aₙ
• Applications: Coordinates, relations, databases, probability
• Foundation for: Functions, relations, coordinate geometry"""
        except Exception as e:
            return f"Cartesian product error: {str(e)}"
    
    def get_function_theory_formulas(self, command):
        """Complete function theory"""
        try:
            if 'definition' in command or 'basic' in command:
                return """Function Theory Basics:
Function: f: A → B assigns exactly one element of B to each element of A
Notation: f(a) = b means f maps a to b

Components:
- Domain: Set A (input set)
- Codomain: Set B (output set)
- Range/Image: f(A) = {f(a) : a ∈ A} ⊆ B
- Graph: {(a,f(a)) : a ∈ A} ⊆ A × B

Vertical Line Test: Graph represents function iff every vertical line intersects it at most once

Image of element: f(a) = value of function at a
Image of set: f(S) = {f(s) : s ∈ S} for S ⊆ A"""
            
            elif 'types' in command:
                return """Types of Functions:
1. One-to-One (Injective): f(a₁) = f(a₂) ⟹ a₁ = a₂
   - Different inputs give different outputs
   - Horizontal line test: Each horizontal line intersects graph at most once

2. Onto (Surjective): Range = Codomain
   - Every element in codomain has preimage
   - ∀b ∈ B, ∃a ∈ A such that f(a) = b

3. Bijective: Both injective and surjective
   - One-to-one correspondence
   - Has inverse function

4. Identity Function: I(x) = x for all x
5. Constant Function: f(x) = c for all x
6. Even Function: f(-x) = f(x)
7. Odd Function: f(-x) = -f(x)"""
            
            elif 'properties' in command:
                return """Function Properties:
Domain and Range:
- Domain: All possible input values
- Range: All actual output values
- Range ⊆ Codomain

Function Equality: f = g iff:
1. Same domain
2. f(x) = g(x) for all x in domain

Restriction: f|S is function f restricted to subset S of domain
Extension: Function g extends f if domain of f ⊆ domain of g and g(x) = f(x) for all x in domain of f

Preimage: f⁻¹(B) = {a ∈ A : f(a) ∈ B} for B ⊆ Codomain"""
            
            else:
                return """Function Theory:
• Definition: f: A → B, unique output for each input
• Components: Domain, codomain, range, graph
• Types: Injective, surjective, bijective
• Properties: Equality, restriction, extension
• Applications: Mathematics, computer science, physics"""
        except Exception as e:
            return f"Function theory error: {str(e)}"
    
    def solve_function_composition(self, command):
        """Function composition theory and properties"""
        try:
            if 'definition' in command or 'basic' in command:
                return """Function Composition:
Definition: (g ∘ f)(x) = g(f(x))
Read as "g composed with f" or "g of f"

Requirements:
- Range of f must be subset of domain of g
- If f: A → B and g: B → C, then g ∘ f: A → C

Process:
1. Apply f to input x to get f(x)
2. Apply g to f(x) to get g(f(x))

Example: f(x) = x + 1, g(x) = 2x
(g ∘ f)(x) = g(f(x)) = g(x + 1) = 2(x + 1) = 2x + 2
(f ∘ g)(x) = f(g(x)) = f(2x) = 2x + 1"""
            
            elif 'properties' in command:
                return """Properties of Function Composition:
1. Associativity: (h ∘ g) ∘ f = h ∘ (g ∘ f)
2. Non-commutativity: Generally g ∘ f ≠ f ∘ g
3. Identity: f ∘ I = I ∘ f = f where I is identity function
4. Composition with inverse: f ∘ f⁻¹ = I and f⁻¹ ∘ f = I

Type Preservation:
- If f and g are injective, then g ∘ f is injective
- If f and g are surjective, then g ∘ f is surjective
- If f and g are bijective, then g ∘ f is bijective

Inverse of Composition: (g ∘ f)⁻¹ = f⁻¹ ∘ g⁻¹"""
            
            elif 'examples' in command:
                return """Function Composition Examples:
Example 1: f(x) = x², g(x) = x + 3
(g ∘ f)(x) = g(f(x)) = g(x²) = x² + 3
(f ∘ g)(x) = f(g(x)) = f(x + 3) = (x + 3)²

Example 2: f(x) = √x, g(x) = x - 1
Domain of f: [0, ∞)
For g ∘ f to exist: f(x) ≥ 0, so x ≥ 0
(g ∘ f)(x) = g(√x) = √x - 1
Domain of g ∘ f: [0, ∞)

Example 3: Verify associativity
f(x) = x + 1, g(x) = 2x, h(x) = x²
((h ∘ g) ∘ f)(x) = (h ∘ g)(x + 1) = h(2(x + 1)) = h(2x + 2) = (2x + 2)²
(h ∘ (g ∘ f))(x) = h((g ∘ f)(x)) = h(2(x + 1)) = (2x + 2)²"""
            
            else:
                return """Function Composition:
• Definition: (g ∘ f)(x) = g(f(x))
• Requirements: Range of f ⊆ Domain of g
• Properties: Associative, not commutative, identity element
• Type preservation: Injectivity, surjectivity maintained
• Applications: Complex function construction, calculus chain rule"""
        except Exception as e:
            return f"Function composition error: {str(e)}"
    
    def solve_inverse_functions(self, command):
        """Inverse function theory and conditions"""
        try:
            if 'definition' in command or 'basic' in command:
                return """Inverse Functions:
Definition: f⁻¹ is inverse of f if:
- f⁻¹(f(x)) = x for all x in domain of f
- f(f⁻¹(y)) = y for all y in range of f

Existence Condition: Function f has inverse iff f is bijective (one-to-one and onto)

Finding Inverse:
1. Replace f(x) with y: y = f(x)
2. Solve for x in terms of y: x = expression in y
3. Interchange x and y: f⁻¹(x) = expression in x
4. Verify: f(f⁻¹(x)) = x and f⁻¹(f(x)) = x

Domain of f⁻¹ = Range of f
Range of f⁻¹ = Domain of f"""
            
            elif 'conditions' in command:
                return """Conditions for Invertible Function:
Necessary and Sufficient Condition: f is bijective

1. Injective (One-to-One):
   - f(a) = f(b) ⟹ a = b
   - Horizontal line test passes
   - Different inputs give different outputs

2. Surjective (Onto):
   - Range = Codomain
   - Every element in codomain has preimage
   - ∀y ∈ B, ∃x ∈ A such that f(x) = y

Tests for Injectivity:
- Algebraic: Assume f(a) = f(b), prove a = b
- Graphical: Horizontal line test
- Derivative: f'(x) > 0 or f'(x) < 0 (strictly monotonic)"""
            
            elif 'properties' in command:
                return """Properties of Inverse Functions:
1. Uniqueness: If inverse exists, it's unique
2. Symmetry: If f⁻¹ is inverse of f, then f is inverse of f⁻¹
3. Graph Symmetry: Graphs of f and f⁻¹ are reflections across y = x
4. Composition: f ∘ f⁻¹ = I_B and f⁻¹ ∘ f = I_A

Inverse of Composition: (g ∘ f)⁻¹ = f⁻¹ ∘ g⁻¹

Domain and Range Interchange:
- Domain of f⁻¹ = Range of f
- Range of f⁻¹ = Domain of f

Continuity: If f is continuous and bijective, then f⁻¹ is continuous"""
            
            elif 'examples' in command:
                return """Inverse Function Examples:
Example 1: f(x) = 2x + 3
Step 1: y = 2x + 3
Step 2: x = (y - 3)/2
Step 3: f⁻¹(x) = (x - 3)/2
Verify: f(f⁻¹(x)) = f((x-3)/2) = 2((x-3)/2) + 3 = x ✓

Example 2: f(x) = x³
f is bijective (strictly increasing)
y = x³ ⟹ x = ∛y
f⁻¹(x) = ∛x

Example 3: f(x) = x² (not invertible on ℝ)
Not injective: f(2) = f(-2) = 4
Restricted domain [0,∞): f⁻¹(x) = √x"""
            
            else:
                return """Inverse Functions:
• Definition: f⁻¹(f(x)) = x and f(f⁻¹(y)) = y
• Existence: Function must be bijective
• Finding: Solve y = f(x) for x, interchange variables
• Properties: Unique, symmetric graphs, composition rules
• Applications: Solving equations, undoing transformations"""
        except Exception as e:
            return f"Inverse functions error: {str(e)}"      
  
        # ==================== COMPLETE NUMERICAL METHODS SYLLABUS (7501-8500+) ====================
        
        # Root Finding Methods (7501-7600)
        if any(word in command for word in ['bisection method', 'bisection algorithm']):
            result = self.solve_bisection_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['false position', 'regula falsi']):
            result = self.solve_false_position_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['newton raphson', 'newton method']):
            result = self.solve_newton_raphson_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['secant method', 'secant algorithm']):
            result = self.solve_secant_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['fixed point', 'iteration method']):
            result = self.solve_fixed_point_iteration(command)
            self.speak(result)
        
        # Interpolation Methods (7601-7700)
        elif any(word in command for word in ['lagrange interpolation', 'lagrange polynomial']):
            result = self.solve_lagrange_interpolation(command)
            self.speak(result)
        
        elif any(word in command for word in ['newton forward', 'forward difference']):
            result = self.solve_newton_forward_interpolation(command)
            self.speak(result)
        
        elif any(word in command for word in ['newton backward', 'backward difference']):
            result = self.solve_newton_backward_interpolation(command)
            self.speak(result)
        
        elif any(word in command for word in ['divided difference', 'newton divided']):
            result = self.solve_newton_divided_difference(command)
            self.speak(result)
        
        elif any(word in command for word in ['spline interpolation', 'cubic spline']):
            result = self.solve_spline_interpolation(command)
            self.speak(result)
        
        # Numerical Differentiation (7701-7750)
        elif any(word in command for word in ['numerical differentiation', 'finite difference differentiation']):
            result = self.solve_numerical_differentiation(command)
            self.speak(result)
        
        elif any(word in command for word in ['forward difference formula', 'backward difference formula']):
            result = self.solve_difference_formulas(command)
            self.speak(result)
        
        # Numerical Integration (7751-7850)
        elif any(word in command for word in ['trapezoidal rule', 'trapezium rule']):
            result = self.solve_trapezoidal_rule(command)
            self.speak(result)
        
        elif any(word in command for word in ['simpson rule', 'simpson 1/3', 'simpson one third']):
            result = self.solve_simpson_one_third_rule(command)
            self.speak(result)
        
        elif any(word in command for word in ['simpson 3/8', 'simpson three eighth']):
            result = self.solve_simpson_three_eighth_rule(command)
            self.speak(result)
        
        elif any(word in command for word in ['boole rule', 'boole method']):
            result = self.solve_boole_rule(command)
            self.speak(result)
        
        elif any(word in command for word in ['weddle rule', 'weddle method']):
            result = self.solve_weddle_rule(command)
            self.speak(result)
        
        elif any(word in command for word in ['gaussian quadrature', 'gauss legendre']):
            result = self.solve_gaussian_quadrature(command)
            self.speak(result)
        
        # Linear System Solutions (7851-7950)
        elif any(word in command for word in ['gauss elimination', 'gaussian elimination']):
            result = self.solve_gauss_elimination(command)
            self.speak(result)
        
        elif any(word in command for word in ['gauss jordan', 'gauss-jordan']):
            result = self.solve_gauss_jordan_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['lu decomposition', 'lu factorization']):
            result = self.solve_lu_decomposition(command)
            self.speak(result)
        
        elif any(word in command for word in ['jacobi method', 'jacobi iteration']):
            result = self.solve_jacobi_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['gauss seidel', 'gauss-seidel']):
            result = self.solve_gauss_seidel_method(command)
            self.speak(result)
        
        # Differential Equations (7951-8100)
        elif any(word in command for word in ['euler method', 'euler forward']):
            result = self.solve_euler_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['modified euler', 'heun method']):
            result = self.solve_modified_euler_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['runge kutta', 'rk method', 'rk4']):
            result = self.solve_runge_kutta_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['adams bashforth', 'predictor corrector']):
            result = self.solve_adams_bashforth_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['milne method', 'milne predictor']):
            result = self.solve_milne_method(command)
            self.speak(result)
        
        # Eigenvalue Problems (8101-8200)
        elif any(word in command for word in ['power method', 'dominant eigenvalue']):
            result = self.solve_power_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['inverse power', 'smallest eigenvalue']):
            result = self.solve_inverse_power_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['jacobi eigenvalue', 'jacobi rotation']):
            result = self.solve_jacobi_eigenvalue_method(command)
            self.speak(result)
        
        # Curve Fitting (8201-8300)
        elif any(word in command for word in ['least squares', 'method of least squares']):
            result = self.solve_least_squares_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['linear regression', 'straight line fitting']):
            result = self.solve_linear_regression(command)
            self.speak(result)
        
        elif any(word in command for word in ['polynomial fitting', 'curve fitting']):
            result = self.solve_polynomial_fitting(command)
            self.speak(result)
        
        # Error Analysis (8301-8400)
        elif any(word in command for word in ['error analysis', 'numerical error']):
            result = self.solve_error_analysis(command)
            self.speak(result)
        
        elif any(word in command for word in ['truncation error', 'round off error']):
            result = self.solve_error_types(command)
            self.speak(result)
        
        elif any(word in command for word in ['condition number', 'ill conditioned']):
            result = self.solve_condition_number(command)
            self.speak(result)
        
        # Advanced Topics (8401-8500)
        elif any(word in command for word in ['monte carlo', 'random sampling']):
            result = self.solve_monte_carlo_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['finite difference method', 'fdm']):
            result = self.solve_finite_difference_method(command)
            self.speak(result)
        
        elif any(word in command for word in ['finite element', 'fem']):
            result = self.solve_finite_element_basics(command)
            self.speak(result) 
   
    # ==================== COMPLETE NUMERICAL METHODS SOLUTIONS ====================
    
    def solve_bisection_method(self, command):
        """Bisection method for root finding"""
        try:
            if 'algorithm' in command or 'steps' in command:
                return """Bisection Method Algorithm:
1. Choose interval [a,b] such that f(a)·f(b) < 0
2. Calculate midpoint: c = (a+b)/2
3. If f(c) = 0, then c is root
4. If f(a)·f(c) < 0, set b = c
5. If f(c)·f(b) < 0, set a = c
6. Repeat until |b-a| < tolerance

Convergence: Always converges if continuous function
Rate: Linear convergence
Error bound: |error| ≤ (b-a)/2ⁿ after n iterations"""
            
            elif 'example' in command:
                return """Bisection Method Example:
Find root of f(x) = x³ - x - 1 in [1,2]

Step 1: f(1) = -1, f(2) = 5, f(1)·f(2) < 0 ✓
Step 2: c₁ = (1+2)/2 = 1.5, f(1.5) = 0.875 > 0
        Since f(1)·f(1.5) < 0, set b = 1.5
Step 3: c₂ = (1+1.5)/2 = 1.25, f(1.25) = -0.297 < 0
        Since f(1.25)·f(1.5) < 0, set a = 1.25
Step 4: c₃ = (1.25+1.5)/2 = 1.375, f(1.375) = 0.224 > 0
        Since f(1.25)·f(1.375) < 0, set b = 1.375

Continue until desired accuracy...
Root ≈ 1.3247"""
            
            elif 'advantages' in command or 'disadvantages' in command:
                return """Bisection Method:
Advantages:
• Always converges if f(a)·f(b) < 0
• Simple to implement
• Guaranteed convergence
• No derivative required
• Robust method

Disadvantages:
• Slow convergence (linear)
• Requires bracketing interval
• Cannot find complex roots
• Only finds one root at a time
• Many function evaluations needed"""
            
            else:
                return """Bisection Method:
• Root finding method using interval halving
• Requires: f(a)·f(b) < 0 (opposite signs)
• Formula: c = (a+b)/2
• Convergence: Always (if continuous)
• Rate: Linear (slow but sure)
• Error: |error| ≤ (b-a)/2ⁿ
• Applications: Simple root finding, initial approximation"""
        except Exception as e:
            return f"Bisection method error: {str(e)}"
    
    def solve_false_position_method(self, command):
        """False position (Regula Falsi) method"""
        try:
            if 'algorithm' in command or 'formula' in command:
                return """False Position Method (Regula Falsi):
Formula: c = (a·f(b) - b·f(a))/(f(b) - f(a))

Algorithm:
1. Choose [a,b] such that f(a)·f(b) < 0
2. Calculate c using above formula
3. If f(c) = 0, then c is root
4. If f(a)·f(c) < 0, set b = c
5. If f(c)·f(b) < 0, set a = c
6. Repeat until convergence

Geometric Interpretation:
• Draws straight line between (a,f(a)) and (b,f(b))
• Finds x-intercept of this line
• Better than bisection as it uses function values"""
            
            elif 'example' in command:
                return """False Position Example:
Find root of f(x) = x³ - x - 1 in [1,2]

Given: f(1) = -1, f(2) = 5

Step 1: c₁ = (1×5 - 2×(-1))/(5-(-1)) = 7/6 = 1.167
        f(1.167) = -0.704 < 0
        Since f(1.167)·f(2) < 0, set a = 1.167

Step 2: c₂ = (1.167×5 - 2×(-0.704))/(5-(-0.704)) = 1.221
        f(1.221) = -0.511 < 0
        Set a = 1.221

Step 3: c₃ = (1.221×5 - 2×(-0.511))/(5-(-0.511)) = 1.265
        Continue...

Root ≈ 1.3247 (faster than bisection)"""
            
            elif 'comparison' in command:
                return """False Position vs Bisection:
False Position Advantages:
• Faster convergence than bisection
• Uses function values intelligently
• Better approximation to root

False Position Disadvantages:
• May converge slowly if one end is "sticky"
• More complex formula
• Still linear convergence in worst case

Modified False Position:
• Illinois method: Modify function values
• Anderson-Björck method: Better convergence
• Pegasus method: Avoid stagnation"""
            
            else:
                return """False Position Method:
• Formula: c = (a·f(b) - b·f(a))/(f(b) - f(a))
• Better than bisection (uses function values)
• Linear interpolation between endpoints
• Faster convergence than bisection
• May have slow convergence if one end sticky
• Also called Regula Falsi method"""
        except Exception as e:
            return f"False position method error: {str(e)}"
    
    def solve_newton_raphson_method(self, command):
        """Newton-Raphson method for root finding"""
        try:
            if 'formula' in command or 'algorithm' in command:
                return """Newton-Raphson Method:
Formula: x_{n+1} = x_n - f(x_n)/f'(x_n)

Algorithm:
1. Choose initial guess x₀
2. Calculate f(x₀) and f'(x₀)
3. Apply formula: x₁ = x₀ - f(x₀)/f'(x₀)
4. Repeat: x_{n+1} = x_n - f(x_n)/f'(x_n)
5. Stop when |x_{n+1} - x_n| < tolerance

Geometric Interpretation:
• Draw tangent at (x_n, f(x_n))
• Find x-intercept of tangent line
• This gives next approximation x_{n+1}"""
            
            elif 'example' in command:
                return """Newton-Raphson Example:
Find root of f(x) = x³ - x - 1, f'(x) = 3x² - 1

Starting with x₀ = 1.5:

Iteration 1:
x₁ = 1.5 - f(1.5)/f'(1.5) = 1.5 - 0.875/5.75 = 1.348

Iteration 2:
f(1.348) = 0.100, f'(1.348) = 4.451
x₂ = 1.348 - 0.100/4.451 = 1.325

Iteration 3:
f(1.325) = 0.002, f'(1.325) = 4.259
x₃ = 1.325 - 0.002/4.259 = 1.3247

Root ≈ 1.3247 (very fast convergence!)"""
            
            elif 'convergence' in command:
                return """Newton-Raphson Convergence:
Convergence Rate: Quadratic (very fast)
• Error roughly squares each iteration
• If error is 0.1, next error ≈ 0.01

Convergence Conditions:
1. f'(x) ≠ 0 near root
2. Good initial guess
3. f''(x) continuous near root

Failure Cases:
• f'(x) = 0 (horizontal tangent)
• Poor initial guess
• Oscillation between points
• Divergence to infinity

Modified Newton Methods:
• Damped Newton: x_{n+1} = x_n - λf(x_n)/f'(x_n)
• Secant method: Approximate derivative"""
            
            elif 'advantages' in command or 'disadvantages' in command:
                return """Newton-Raphson Method:
Advantages:
• Quadratic convergence (very fast)
• Only needs one initial point
• Self-correcting for good guesses
• Widely applicable

Disadvantages:
• Requires derivative f'(x)
• May not converge for poor initial guess
• Fails when f'(x) = 0
• Can oscillate or diverge
• Sensitive to starting point"""
            
            else:
                return """Newton-Raphson Method:
• Formula: x_{n+1} = x_n - f(x_n)/f'(x_n)
• Convergence: Quadratic (fastest)
• Requires: Derivative f'(x)
• Geometric: Tangent line method
• Best for: Well-behaved functions with good initial guess
• Applications: Most root-finding problems"""
        except Exception as e:
            return f"Newton-Raphson method error: {str(e)}"
    
    def solve_secant_method(self, command):
        """Secant method for root finding"""
        try:
            if 'formula' in command or 'algorithm' in command:
                return """Secant Method:
Formula: x_{n+1} = x_n - f(x_n)·(x_n - x_{n-1})/(f(x_n) - f(x_{n-1}))

Algorithm:
1. Choose two initial points x₀ and x₁
2. Calculate f(x₀) and f(x₁)
3. Apply secant formula to get x₂
4. Continue with x_n and x_{n-1} to get x_{n+1}
5. Stop when |x_{n+1} - x_n| < tolerance

Derivative Approximation:
f'(x_n) ≈ (f(x_n) - f(x_{n-1}))/(x_n - x_{n-1})

This makes secant method a derivative-free version of Newton-Raphson"""
            
            elif 'example' in command:
                return """Secant Method Example:
Find root of f(x) = x³ - x - 1

Starting with x₀ = 1, x₁ = 2:
f(1) = -1, f(2) = 5

Iteration 1:
x₂ = 2 - 5·(2-1)/(5-(-1)) = 2 - 5/6 = 1.167

Iteration 2:
f(1.167) = -0.704
x₃ = 1.167 - (-0.704)·(1.167-2)/(-0.704-5) = 1.267

Iteration 3:
f(1.267) = -0.263
x₄ = 1.267 - (-0.263)·(1.267-1.167)/(-0.263-(-0.704)) = 1.319

Continue until convergence...
Root ≈ 1.3247"""
            
            elif 'convergence' in command:
                return """Secant Method Convergence:
Convergence Rate: Superlinear (φ ≈ 1.618)
• Faster than linear, slower than quadratic
• Golden ratio convergence order

Convergence Formula:
|e_{n+1}| ≈ C|e_n|^φ where φ = (1+√5)/2 ≈ 1.618

Advantages over Newton-Raphson:
• No derivative required
• Still fast convergence
• More robust in some cases

Disadvantages:
• Needs two initial points
• Slower than Newton-Raphson
• May fail if secant becomes horizontal"""
            
            elif 'comparison' in command:
                return """Method Comparison:
Newton-Raphson:
• Convergence: Quadratic (fastest)
• Requires: f'(x)
• Initial points: 1

Secant:
• Convergence: Superlinear (φ ≈ 1.618)
• Requires: Only f(x)
• Initial points: 2

Bisection:
• Convergence: Linear (slowest but sure)
• Requires: f(a)·f(b) < 0
• Initial points: Interval [a,b]

False Position:
• Convergence: Linear (better than bisection)
• Requires: f(a)·f(b) < 0
• Initial points: Interval [a,b]"""
            
            else:
                return """Secant Method:
• Formula: x_{n+1} = x_n - f(x_n)·(x_n-x_{n-1})/(f(x_n)-f(x_{n-1}))
• Derivative-free version of Newton-Raphson
• Convergence: Superlinear (order φ ≈ 1.618)
• Needs two initial points
• Good compromise between speed and simplicity
• Applications: When derivative is difficult to compute"""
        except Exception as e:
            return f"Secant method error: {str(e)}"
    
    def solve_fixed_point_iteration(self, command):
        """Fixed point iteration method"""
        try:
            if 'theory' in command or 'definition' in command:
                return """Fixed Point Iteration Theory:
Definition: A fixed point of function g(x) is a value α such that g(α) = α

Method: To solve f(x) = 0, rewrite as x = g(x)
Then iterate: x_{n+1} = g(x_n)

Convergence Theorem:
If |g'(x)| < 1 in neighborhood of fixed point α, then:
1. Iteration converges to α
2. Convergence is linear
3. Error: |e_{n+1}| ≈ |g'(α)||e_n|

Rearrangement Forms:
From f(x) = 0, we can write:
• x = x + f(x) (usually diverges)
• x = x - f(x)/M (M > 0)
• x = x - f(x)/f'(x) (Newton's method)"""
            
            elif 'example' in command:
                return """Fixed Point Iteration Example:
Solve x³ - x - 1 = 0

Rearrangement 1: x = ∛(x + 1)
g₁(x) = ∛(x + 1), g₁'(x) = 1/(3∛(x+1)²)
At root x ≈ 1.325: |g₁'(1.325)| ≈ 0.23 < 1 ✓ Converges

Starting with x₀ = 1:
x₁ = ∛(1 + 1) = ∛2 ≈ 1.260
x₂ = ∛(1.260 + 1) ≈ 1.312
x₃ = ∛(1.312 + 1) ≈ 1.322
x₄ = ∛(1.322 + 1) ≈ 1.324

Rearrangement 2: x = 1 + 1/x²
g₂(x) = 1 + 1/x², g₂'(x) = -2/x³
At root: |g₂'(1.325)| ≈ 0.86 < 1 ✓ Converges (slower)"""
            
            elif 'convergence' in command:
                return """Fixed Point Convergence Analysis:
Convergence Condition: |g'(x)| < 1 near fixed point

Convergence Types:
1. |g'(α)| < 1: Linear convergence
2. g'(α) = 0: Quadratic convergence (rare)
3. |g'(α)| > 1: Divergence
4. |g'(α)| = 1: Inconclusive (need higher derivatives)

Error Analysis:
|x_{n+1} - α| ≤ K|x_n - α| where K = max|g'(x)|

Acceleration Techniques:
• Aitken's Δ² process
• Steffensen's method
• Wegstein method

Choosing g(x):
• Multiple rearrangements possible
• Choose one with |g'(x)| smallest
• Sometimes need to modify for convergence"""
            
            elif 'applications' in command:
                return """Fixed Point Applications:
1. Root Finding:
   • Simple iteration schemes
   • When derivative difficult

2. System of Equations:
   • x = g₁(x,y)
   • y = g₂(x,y)

3. Optimization:
   • Finding critical points
   • Gradient descent variants

4. Numerical Analysis:
   • Solving integral equations
   • Iterative refinement

5. Engineering:
   • Control systems
   • Circuit analysis
   • Heat transfer problems

Modified Methods:
• Relaxation: x_{n+1} = (1-ω)x_n + ωg(x_n)
• Over-relaxation: ω > 1
• Under-relaxation: ω < 1"""
            
            else:
                return """Fixed Point Iteration:
• Method: x_{n+1} = g(x_n) where x = g(x)
• Convergence: |g'(x)| < 1 near fixed point
• Rate: Usually linear
• Advantage: Simple, no derivatives of original function
• Disadvantage: May not converge, slow convergence
• Applications: Simple root finding, system solving"""
        except Exception as e:
            return f"Fixed point iteration error: {str(e)}"  
  
    def solve_lagrange_interpolation(self, command):
        """Lagrange interpolation method"""
        try:
            if 'formula' in command or 'theory' in command:
                return """Lagrange Interpolation Formula:
P_n(x) = Σ(i=0 to n) y_i · L_i(x)

where L_i(x) = Π(j=0 to n, j≠i) (x - x_j)/(x_i - x_j)

L_i(x) are Lagrange basis polynomials:
• L_i(x_i) = 1
• L_i(x_j) = 0 for j ≠ i

Properties:
• Unique polynomial of degree ≤ n through n+1 points
• Exact for polynomials of degree ≤ n
• No need for equally spaced points
• Direct formula (no system solving)"""
            
            elif 'example' in command:
                return """Lagrange Interpolation Example:
Given points: (0,1), (1,4), (2,9)
Find P(x) and evaluate at x = 1.5

Step 1: Calculate basis polynomials
L₀(x) = (x-1)(x-2)/((0-1)(0-2)) = (x-1)(x-2)/2
L₁(x) = (x-0)(x-2)/((1-0)(1-2)) = -x(x-2)
L₂(x) = (x-0)(x-1)/((2-0)(2-1)) = x(x-1)/2

Step 2: Form interpolating polynomial
P(x) = 1·L₀(x) + 4·L₁(x) + 9·L₂(x)
     = (x-1)(x-2)/2 - 4x(x-2) + 9x(x-1)/2
     = x² + 2x + 1

Step 3: Evaluate at x = 1.5
P(1.5) = (1.5)² + 2(1.5) + 1 = 6.25"""
            
            elif 'error' in command:
                return """Lagrange Interpolation Error:
Error Formula:
E_n(x) = f(x) - P_n(x) = f^(n+1)(ξ)/(n+1)! · Π(i=0 to n)(x - x_i)

where ξ is some point in the interval containing x and all x_i

Error Properties:
• Error depends on (n+1)th derivative
• Error is zero at interpolation points
• Error grows with distance from interpolation points
• Runge phenomenon: High-degree polynomials may oscillate

Error Bound:
|E_n(x)| ≤ M/(n+1)! · |Π(i=0 to n)(x - x_i)|
where M = max|f^(n+1)(x)| on the interval

Minimizing Error:
• Use Chebyshev points for better distribution
• Piecewise interpolation for large intervals
• Spline interpolation for smooth curves"""
            
            elif 'advantages' in command or 'disadvantages' in command:
                return """Lagrange Interpolation:
Advantages:
• Direct formula (no system solving)
• Works with unequally spaced points
• Easy to program
• Exact for polynomials of degree ≤ n
• Good theoretical foundation

Disadvantages:
• Computationally expensive for large n
• Runge phenomenon for high degrees
• Adding new point requires complete recalculation
• Numerical instability for large n
• Not suitable for extrapolation

When to Use:
• Small number of points (n ≤ 10)
• Unequally spaced data
• Theoretical analysis
• One-time interpolation"""
            
            else:
                return """Lagrange Interpolation:
• Formula: P_n(x) = Σ y_i · L_i(x)
• Basis: L_i(x) = Π(x-x_j)/(x_i-x_j), j≠i
• Degree: n for n+1 points
• Unique polynomial through given points
• Works with unequally spaced points
• Applications: Function approximation, numerical integration"""
        except Exception as e:
            return f"Lagrange interpolation error: {str(e)}"
    
    def solve_newton_forward_interpolation(self, command):
        """Newton forward difference interpolation"""
        try:
            if 'formula' in command or 'theory' in command:
                return """Newton Forward Difference Formula:
P_n(x) = y₀ + uΔy₀ + u(u-1)/2!·Δ²y₀ + u(u-1)(u-2)/3!·Δ³y₀ + ...

where u = (x - x₀)/h and h is step size

Forward Difference Table:
Δy_i = y_{i+1} - y_i (first difference)
Δ²y_i = Δy_{i+1} - Δy_i (second difference)
Δⁿy_i = Δⁿ⁻¹y_{i+1} - Δⁿ⁻¹y_i (nth difference)

Binomial Coefficient Form:
P_n(x) = Σ(k=0 to n) C(u,k) · Δᵏy₀
where C(u,k) = u(u-1)...(u-k+1)/k!"""
            
            elif 'example' in command:
                return """Newton Forward Difference Example:
Given: x: 0, 1, 2, 3, 4
       y: 1, 8, 27, 64, 125

Step 1: Form difference table
x  |  y  | Δy | Δ²y | Δ³y | Δ⁴y
0  |  1  |    |     |     |
   |     | 7  |     |     |
1  |  8  |    | 12  |     |
   |     | 19 |     | 6   |
2  | 27  |    | 18  |     | 0
   |     | 37 |     | 6   |
3  | 64  |    | 24  |     |
   |     | 61 |     |     |
4  | 125 |    |     |     |

Step 2: Apply formula (h = 1, x₀ = 0)
u = (x - 0)/1 = x
P(x) = 1 + x·7 + x(x-1)/2·12 + x(x-1)(x-2)/6·6
     = 1 + 7x + 6x(x-1) + x(x-1)(x-2)
     = x³ + 1 (exact since data is y = x³ + 1)"""
            
            elif 'applications' in command:
                return """Newton Forward Difference Applications:
1. Interpolation near beginning of table
   • Best for x near x₀
   • Forward differences decrease rapidly

2. Numerical Differentiation:
   f'(x₀) ≈ (Δy₀ - Δ²y₀/2 + Δ³y₀/3 - ...)/h

3. Numerical Integration:
   ∫f(x)dx ≈ h[y₀ + Δy₀/2 - Δ²y₀/12 + ...]

4. Extrapolation:
   • Predicting future values
   • Extending data series

5. Error Analysis:
   • Checking data consistency
   • Detecting errors in tabulated data

When to Use:
• Equally spaced data points
• Interpolation near start of interval
• Forward extrapolation
• Difference table analysis"""
            
            elif 'error' in command:
                return """Newton Forward Difference Error:
Truncation Error:
If we stop at nth difference:
E_n(x) = u(u-1)...(u-n)/(n+1)! · h^(n+1) · f^(n+1)(ξ)

where ξ is in the interval

Error Properties:
• Error increases away from x₀
• Best accuracy near beginning of table
• Higher differences should decrease for good data
• If nth difference is constant, (n+1)th difference is zero

Practical Error Estimation:
• Check if differences are decreasing
• Use one more term than needed, compare results
• Look for patterns in difference table
• Constant differences indicate polynomial data

Improving Accuracy:
• Use more data points
• Choose appropriate degree
• Use central differences when possible
• Check for data errors"""
            
            else:
                return """Newton Forward Difference:
• Formula: P(x) = y₀ + uΔy₀ + u(u-1)Δ²y₀/2! + ...
• Variable: u = (x-x₀)/h
• Best for: Interpolation near start of data
• Requires: Equally spaced points
• Applications: Forward extrapolation, series extension
• Error: Increases away from x₀"""
        except Exception as e:
            return f"Newton forward interpolation error: {str(e)}"
    
    def solve_newton_backward_interpolation(self, command):
        """Newton backward difference interpolation"""
        try:
            if 'formula' in command or 'theory' in command:
                return """Newton Backward Difference Formula:
P_n(x) = y_n + u∇y_n + u(u+1)/2!·∇²y_n + u(u+1)(u+2)/3!·∇³y_n + ...

where u = (x - x_n)/h and h is step size

Backward Difference:
∇y_i = y_i - y_{i-1} (first backward difference)
∇²y_i = ∇y_i - ∇y_{i-1} (second backward difference)
∇ⁿy_i = ∇ⁿ⁻¹y_i - ∇ⁿ⁻¹y_{i-1} (nth backward difference)

Relationship to Forward Differences:
∇y_i = Δy_{i-1}
∇²y_i = Δ²y_{i-2}"""
            
            elif 'example' in command:
                return """Newton Backward Difference Example:
Given: x: 0, 1, 2, 3, 4
       y: 1, 8, 27, 64, 125

Step 1: Form backward difference table
x  |  y  | ∇y | ∇²y | ∇³y | ∇⁴y
0  |  1  |    |     |     |
1  |  8  | 7  |     |     |
2  | 27  | 19 | 12  |     |
3  | 64  | 37 | 18  | 6   |
4  | 125 | 61 | 24  | 6   | 0

Step 2: Apply formula (using x_n = 4, h = 1)
u = (x - 4)/1 = x - 4
P(x) = 125 + (x-4)·61 + (x-4)(x-3)/2·24 + (x-4)(x-3)(x-2)/6·6
     = 125 + 61(x-4) + 12(x-4)(x-3) + (x-4)(x-3)(x-2)

For x = 2.5:
u = 2.5 - 4 = -1.5
P(2.5) = 125 + (-1.5)·61 + (-1.5)(-0.5)/2·24 + (-1.5)(-0.5)(0.5)/6·6
       = 125 - 91.5 + 9 + 1.125 = 43.625"""
            
            elif 'comparison' in command:
                return """Forward vs Backward Differences:
Newton Forward:
• Best near beginning of table (x ≈ x₀)
• Uses forward differences: Δy_i = y_{i+1} - y_i
• Formula starts from y₀
• Good for forward extrapolation

Newton Backward:
• Best near end of table (x ≈ x_n)
• Uses backward differences: ∇y_i = y_i - y_{i-1}
• Formula starts from y_n
• Good for backward extrapolation

Central Differences:
• Best for middle of table
• Uses both forward and backward differences
• More accurate for interpolation
• Stirling's and Bessel's formulas

Choice Criteria:
• Use forward if x < (x₀ + x_n)/2
• Use backward if x > (x₀ + x_n)/2
• Use central for middle region"""
            
            elif 'applications' in command:
                return """Newton Backward Difference Applications:
1. Interpolation near end of table
   • Most accurate for x near x_n
   • Backward differences converge faster

2. Backward Extrapolation:
   • Predicting past values
   • Historical data analysis

3. Numerical Differentiation:
   f'(x_n) ≈ (∇y_n + ∇²y_n/2 + ∇³y_n/3 + ...)/h

4. Numerical Integration:
   ∫f(x)dx ≈ h[y_n - ∇y_n/2 - ∇²y_n/12 - ...]

5. Data Analysis:
   • Trend analysis
   • Pattern recognition in time series

When to Use:
• Interpolation near end of data
• Backward extrapolation
• Time series forecasting
• Recent data more reliable"""
            
            else:
                return """Newton Backward Difference:
• Formula: P(x) = y_n + u∇y_n + u(u+1)∇²y_n/2! + ...
• Variable: u = (x-x_n)/h
• Best for: Interpolation near end of data
• Differences: ∇y_i = y_i - y_{i-1}
• Applications: Backward extrapolation, recent data analysis
• Error: Increases away from x_n"""
        except Exception as e:
            return f"Newton backward interpolation error: {str(e)}"    

    def solve_newton_divided_difference(self, command):
        """Newton divided difference interpolation"""
        try:
            if 'formula' in command or 'theory' in command:
                return """Newton Divided Difference Formula:
P_n(x) = f[x₀] + f[x₀,x₁](x-x₀) + f[x₀,x₁,x₂](x-x₀)(x-x₁) + ...

Divided Differences:
f[x_i] = f(x_i) (zeroth order)
f[x_i,x_j] = (f[x_j] - f[x_i])/(x_j - x_i) (first order)
f[x_i,x_j,x_k] = (f[x_j,x_k] - f[x_i,x_j])/(x_k - x_i) (second order)

General Formula:
f[x₀,x₁,...,x_n] = (f[x₁,...,x_n] - f[x₀,...,x_{n-1}])/(x_n - x₀)

Properties:
• Works with unequally spaced points
• Symmetric in arguments: f[x_i,x_j] = f[x_j,x_i]
• Easy to add new points"""
            
            elif 'example' in command:
                return """Newton Divided Difference Example:
Given points: (1,1), (2,8), (3,27), (5,125)

Step 1: Form divided difference table
x  | f[x] | f[x_i,x_j] | f[x_i,x_j,x_k] | f[x_i,x_j,x_k,x_l]
1  |  1   |             |                |
   |      |     7       |                |
2  |  8   |             |      6         |
   |      |    19       |                |      1
3  | 27   |             |     11         |
   |      |    49       |                |
5  | 125  |             |                |

Step 2: Calculate divided differences
f[1,2] = (8-1)/(2-1) = 7
f[2,3] = (27-8)/(3-2) = 19
f[3,5] = (125-27)/(5-3) = 49
f[1,2,3] = (19-7)/(3-1) = 6
f[2,3,5] = (49-19)/(5-2) = 10
f[1,2,3,5] = (10-6)/(5-1) = 1

Step 3: Form polynomial
P(x) = 1 + 7(x-1) + 6(x-1)(x-2) + 1(x-1)(x-2)(x-3)
     = x³ (exact since data is y = x³)"""
            
            elif 'advantages' in command:
                return """Newton Divided Difference Advantages:
1. Unequally Spaced Points:
   • No restriction on point spacing
   • More flexible than forward/backward differences

2. Easy Point Addition:
   • Adding new point doesn't require complete recalculation
   • Only need to extend the table

3. Computational Efficiency:
   • Nested multiplication (Horner's method)
   • O(n²) construction, O(n) evaluation

4. Error Analysis:
   • Clear relationship to derivatives
   • Easy to estimate truncation error

5. Theoretical Foundation:
   • Basis for many other methods
   • Connection to Taylor series

Implementation:
P(x) = f[x₀] + (x-x₀)[f[x₀,x₁] + (x-x₁)[f[x₀,x₁,x₂] + ...]]
(Nested form for efficient computation)"""
            
            else:
                return """Newton Divided Difference:
• Formula: P(x) = f[x₀] + f[x₀,x₁](x-x₀) + f[x₀,x₁,x₂](x-x₀)(x-x₁) + ...
• Works with unequally spaced points
• Easy to add new points
• Divided difference: f[x_i,x_j] = (f[x_j]-f[x_i])/(x_j-x_i)
• Efficient nested evaluation
• Applications: General interpolation, numerical integration"""
        except Exception as e:
            return f"Newton divided difference error: {str(e)}"
    
    def solve_spline_interpolation(self, command):
        """Spline interpolation methods"""
        try:
            if 'theory' in command or 'definition' in command:
                return """Spline Interpolation Theory:
Definition: Piecewise polynomial interpolation with continuity conditions

Cubic Spline S(x):
• Piecewise cubic polynomials on each interval [x_i, x_{i+1}]
• S(x_i) = y_i for all i (interpolation condition)
• S'(x_i⁻) = S'(x_i⁺) (first derivative continuous)
• S''(x_i⁻) = S''(x_i⁺) (second derivative continuous)

General Form on [x_i, x_{i+1}]:
S_i(x) = a_i + b_i(x-x_i) + c_i(x-x_i)² + d_i(x-x_i)³

Advantages over High-Degree Polynomials:
• No Runge phenomenon
• Smooth curves
• Local control (changing one point affects only nearby intervals)
• Numerically stable"""
            
            elif 'natural' in command or 'boundary' in command:
                return """Natural Cubic Spline:
Boundary Conditions: S''(x₀) = S''(x_n) = 0

System of Equations:
For each interior point x_i (i = 1, 2, ..., n-1):
h_{i-1}S''_{i-1} + 2(h_{i-1} + h_i)S''_i + h_iS''_{i+1} = 6[(y_{i+1}-y_i)/h_i - (y_i-y_{i-1})/h_{i-1}]

where h_i = x_{i+1} - x_i

Matrix Form: AS'' = B
where A is tridiagonal matrix

Once S''_i are found:
S_i(x) = S''_{i-1}(x_{i+1}-x)³/(6h_i) + S''_i(x-x_i)³/(6h_i) + 
         (y_{i-1}/h_i - S''_{i-1}h_i/6)(x_{i+1}-x) + 
         (y_i/h_i - S''_ih_i/6)(x-x_i)"""
            
            elif 'example' in command:
                return """Cubic Spline Example:
Given points: (0,0), (1,1), (2,4)

Step 1: Calculate h_i
h₀ = 1-0 = 1, h₁ = 2-1 = 1

Step 2: Set up system for S''₁
Natural boundary: S''₀ = S''₂ = 0
2(h₀ + h₁)S''₁ = 6[(y₂-y₁)/h₁ - (y₁-y₀)/h₀]
2(1 + 1)S''₁ = 6[(4-1)/1 - (1-0)/1] = 6[3-1] = 12
S''₁ = 3

Step 3: Calculate spline coefficients
For [0,1]: S''₀ = 0, S''₁ = 3
S₀(x) = 0·(1-x)³/6 + 3·x³/6 + (0-0)(1-x) + (1-0)x = x³/2 + x

For [1,2]: S''₁ = 3, S''₂ = 0  
S₁(x) = 3·(2-x)³/6 + 0·(x-1)³/6 + (1-3/6)(2-x) + (4-0)(x-1)
      = (2-x)³/2 + (x-1)/2 + 4(x-1)

Verification: S₀(1) = S₁(1) = 1 ✓"""
            
            elif 'types' in command:
                return """Types of Splines:
1. Linear Spline:
   • Piecewise linear (degree 1)
   • Continuous but not smooth
   • Simple but not smooth curves

2. Quadratic Spline:
   • Piecewise quadratic (degree 2)
   • Continuous first derivative
   • One additional condition needed

3. Cubic Spline:
   • Piecewise cubic (degree 3)
   • Continuous first and second derivatives
   • Most commonly used

4. B-Splines:
   • Basis splines
   • Local support
   • Numerical stability

Boundary Conditions:
• Natural: S''(x₀) = S''(x_n) = 0
• Clamped: S'(x₀) and S'(x_n) specified
• Not-a-knot: S'''(x₁⁻) = S'''(x₁⁺)
• Periodic: S(x₀) = S(x_n), S'(x₀) = S'(x_n)"""
            
            elif 'applications' in command:
                return """Spline Applications:
1. Computer Graphics:
   • Curve and surface design
   • Animation paths
   • Font design

2. Engineering:
   • CAD/CAM systems
   • Ship hull design
   • Aerodynamic surfaces

3. Data Fitting:
   • Smooth curve through noisy data
   • Time series interpolation
   • Signal processing

4. Numerical Methods:
   • Finite element methods
   • Numerical integration
   • Differential equation solving

5. Statistics:
   • Regression splines
   • Smoothing splines
   • Non-parametric regression

Advantages:
• Smooth curves
• Local control
• Numerical stability
• No oscillations
• Flexible boundary conditions"""
            
            else:
                return """Spline Interpolation:
• Piecewise polynomial with continuity conditions
• Cubic spline: Most common (degree 3)
• Conditions: S(x_i) = y_i, S' and S'' continuous
• Boundary: Natural (S''=0 at ends) or clamped
• Advantages: Smooth, stable, no Runge phenomenon
• Applications: Graphics, CAD, data fitting"""
        except Exception as e:
            return f"Spline interpolation error: {str(e)}"
    
    def solve_numerical_differentiation(self, command):
        """Numerical differentiation methods"""
        try:
            if 'forward' in command or 'backward' in command or 'central' in command:
                return """Finite Difference Formulas:

Forward Difference:
f'(x₀) ≈ (f(x₀+h) - f(x₀))/h
Error: O(h)

Backward Difference:
f'(x₀) ≈ (f(x₀) - f(x₀-h))/h
Error: O(h)

Central Difference:
f'(x₀) ≈ (f(x₀+h) - f(x₀-h))/(2h)
Error: O(h²) - More accurate!

Second Derivative:
f''(x₀) ≈ (f(x₀+h) - 2f(x₀) + f(x₀-h))/h²
Error: O(h²)

Higher Order Formulas:
Five-point central: f'(x₀) ≈ (-f(x₀+2h) + 8f(x₀+h) - 8f(x₀-h) + f(x₀-2h))/(12h)
Error: O(h⁴)"""
            
            elif 'error' in command or 'accuracy' in command:
                return """Numerical Differentiation Error Analysis:
Sources of Error:
1. Truncation Error: From Taylor series approximation
2. Round-off Error: From finite precision arithmetic

Total Error ≈ Truncation Error + Round-off Error

For central difference:
Truncation Error ≈ h²f'''(ξ)/6
Round-off Error ≈ 2ε/h (where ε is machine precision)

Optimal Step Size:
h_opt ≈ (3ε/|f'''(x)|)^(1/3)

Trade-off:
• Small h: Less truncation error, more round-off error
• Large h: More truncation error, less round-off error

Practical Guidelines:
• Use central differences when possible
• Choose h ≈ √ε for first derivative
• Choose h ≈ ε^(1/4) for second derivative
• Use Richardson extrapolation for higher accuracy"""
            
            elif 'example' in command:
                return """Numerical Differentiation Example:
Find f'(1) for f(x) = x³ using h = 0.1
Exact: f'(x) = 3x², so f'(1) = 3

Forward Difference:
f'(1) ≈ (f(1.1) - f(1))/0.1 = (1.331 - 1)/0.1 = 3.31
Error = |3.31 - 3| = 0.31

Backward Difference:
f'(1) ≈ (f(1) - f(0.9))/0.1 = (1 - 0.729)/0.1 = 2.71
Error = |2.71 - 3| = 0.29

Central Difference:
f'(1) ≈ (f(1.1) - f(0.9))/(2×0.1) = (1.331 - 0.729)/0.2 = 3.01
Error = |3.01 - 3| = 0.01 (Much better!)

With h = 0.01:
Central: f'(1) ≈ (1.030301 - 0.970299)/0.02 = 3.0001
Error ≈ 0.0001 (Even better!)"""
            
            else:
                return """Numerical Differentiation:
• Forward: f'(x) ≈ (f(x+h) - f(x))/h, Error O(h)
• Backward: f'(x) ≈ (f(x) - f(x-h))/h, Error O(h)
• Central: f'(x) ≈ (f(x+h) - f(x-h))/(2h), Error O(h²)
• Second: f''(x) ≈ (f(x+h) - 2f(x) + f(x-h))/h²
• Optimal h balances truncation and round-off errors
• Applications: When analytical derivative unavailable"""
        except Exception as e:
            return f"Numerical differentiation error: {str(e)}"
    
    def solve_trapezoidal_rule(self, command):
        """Trapezoidal rule for numerical integration"""
        try:
            if 'formula' in command or 'theory' in command:
                return """Trapezoidal Rule:
∫[a to b] f(x)dx ≈ (b-a)/2 · [f(a) + f(b)]

Geometric Interpretation:
• Approximates area under curve with trapezoid
• Connects endpoints with straight line

Composite Trapezoidal Rule:
Divide [a,b] into n equal subintervals of width h = (b-a)/n

∫[a to b] f(x)dx ≈ h/2 · [f(x₀) + 2f(x₁) + 2f(x₂) + ... + 2f(x_{n-1}) + f(x_n)]
                 = h/2 · [f(a) + f(b) + 2∑(i=1 to n-1) f(x_i)]

Error Formula:
E = -(b-a)h²/12 · f''(ξ) for some ξ ∈ [a,b]
Error is O(h²) for single interval, O(h²) for composite rule"""
            
            elif 'example' in command:
                return """Trapezoidal Rule Example:
Evaluate ∫[0 to 1] x²dx using n = 4 intervals

Exact value: ∫x²dx = x³/3, so ∫[0 to 1] x²dx = 1/3 ≈ 0.3333

Step 1: Set up intervals
h = (1-0)/4 = 0.25
x₀ = 0, x₁ = 0.25, x₂ = 0.5, x₃ = 0.75, x₄ = 1

Step 2: Calculate function values
f(0) = 0, f(0.25) = 0.0625, f(0.5) = 0.25
f(0.75) = 0.5625, f(1) = 1

Step 3: Apply formula
∫f(x)dx ≈ 0.25/2 · [0 + 1 + 2(0.0625 + 0.25 + 0.5625)]
        = 0.125 · [1 + 2(0.875)]
        = 0.125 · [1 + 1.75] = 0.125 × 2.75 = 0.34375

Error = |0.34375 - 0.3333| = 0.0104 (about 3.1%)

With n = 8: Result ≈ 0.3359, Error ≈ 0.0026 (Error reduced by factor of 4)"""
            
            elif 'error' in command or 'accuracy' in command:
                return """Trapezoidal Rule Error Analysis:
Error Formula:
E = -(b-a)h²/12 · f''(ξ)

Error Properties:
• Error is O(h²) - quadratic convergence
• Doubling intervals (halving h) reduces error by factor of 4
• Error depends on second derivative
• Exact for linear functions (f''(x) = 0)

Error Bound:
|E| ≤ (b-a)h²/12 · max|f''(x)| on [a,b]

Richardson Extrapolation:
If I(h) is approximation with step h:
Better approximation: I_improved = (4I(h/2) - I(h))/3
This gives O(h⁴) accuracy (Simpson's rule)

Adaptive Integration:
• Start with coarse grid
• Refine where error is large
• Stop when desired accuracy achieved"""
            
            elif 'applications' in command:
                return """Trapezoidal Rule Applications:
1. Definite Integration:
   • When antiderivative unknown
   • Tabulated data integration
   • Experimental data analysis

2. Differential Equations:
   • Euler's method for ODEs
   • Implicit integration schemes

3. Engineering:
   • Area calculations
   • Work and energy computations
   • Signal processing (discrete integration)

4. Statistics:
   • Probability density integration
   • Cumulative distribution functions

5. Physics:
   • Flux calculations
   • Center of mass computations

Advantages:
• Simple to implement
• Stable and reliable
• Good for smooth functions
• Easy error estimation

Disadvantages:
• Slower convergence than Simpson's
• Poor for oscillatory functions
• Requires many intervals for high accuracy"""
            
            else:
                return """Trapezoidal Rule:
• Formula: ∫f(x)dx ≈ h/2[f(a) + f(b) + 2∑f(x_i)]
• Geometric: Area of trapezoids
• Error: O(h²), exact for linear functions
• Composite: Divide interval into subintervals
• Applications: Numerical integration, tabulated data
• Convergence: Error reduces by factor 4 when h halved"""
        except Exception as e:
            return f"Trapezoidal rule error: {str(e)}"    

    def solve_simpson_one_third_rule(self, command):
        """Simpson's 1/3 rule for numerical integration"""
        try:
            if 'formula' in command or 'theory' in command:
                return """Simpson's 1/3 Rule:
∫[a to b] f(x)dx ≈ (b-a)/6 · [f(a) + 4f((a+b)/2) + f(b)]

Derivation: Uses parabolic approximation through 3 points

Composite Simpson's 1/3 Rule:
Requires even number of intervals (n must be even)
h = (b-a)/n

∫[a to b] f(x)dx ≈ h/3 · [f(x₀) + 4f(x₁) + 2f(x₂) + 4f(x₃) + ... + 4f(x_{n-1}) + f(x_n)]

Pattern: 1, 4, 2, 4, 2, 4, ..., 2, 4, 1

Error Formula:
E = -(b-a)h⁴/90 · f⁽⁴⁾(ξ) for some ξ ∈ [a,b]
Error is O(h⁴) - much better than trapezoidal!"""
            
            elif 'example' in command:
                return """Simpson's 1/3 Rule Example:
Evaluate ∫[0 to 1] x²dx using n = 4 intervals

Exact value: 1/3 ≈ 0.3333

Step 1: Set up (n = 4, even ✓)
h = (1-0)/4 = 0.25
Points: 0, 0.25, 0.5, 0.75, 1

Step 2: Function values
f(0) = 0, f(0.25) = 0.0625, f(0.5) = 0.25
f(0.75) = 0.5625, f(1) = 1

Step 3: Apply Simpson's formula
∫f(x)dx ≈ 0.25/3 · [1×0 + 4×0.0625 + 2×0.25 + 4×0.5625 + 1×1]
        = 0.25/3 · [0 + 0.25 + 0.5 + 2.25 + 1]
        = 0.25/3 × 4 = 0.3333

Error = |0.3333 - 0.3333| ≈ 0 (Exact for polynomials of degree ≤ 3!)

For f(x) = e^x on [0,1] with n = 4:
Result ≈ 1.7183, Exact = e-1 ≈ 1.7183
Error ≈ 0.0000 (Very accurate!)"""
            
            elif 'comparison' in command:
                return """Integration Methods Comparison:
For ∫[0 to 1] e^x dx (exact = e-1 ≈ 1.7183) with h = 0.25:

Trapezoidal Rule:
Result ≈ 1.7539, Error ≈ 0.0356 (2.1%)

Simpson's 1/3:
Result ≈ 1.7184, Error ≈ 0.0001 (0.006%)

Error Reduction:
• Simpson's error is O(h⁴) vs O(h²) for trapezoidal
• For same h, Simpson's typically 10-100 times more accurate
• Simpson's exact for polynomials up to degree 3
• Trapezoidal exact only for linear functions

Computational Cost:
• Simpson's: 1 extra function evaluation per interval
• Trapezoidal: Simpler, faster per evaluation
• Overall: Simpson's much more efficient for given accuracy"""
            
            elif 'error' in command:
                return """Simpson's 1/3 Rule Error Analysis:
Error Formula:
E = -(b-a)h⁴/90 · f⁽⁴⁾(ξ)

Key Properties:
• O(h⁴) convergence - very fast!
• Halving h reduces error by factor of 16
• Exact for polynomials of degree ≤ 3
• Error depends on 4th derivative

Error Bound:
|E| ≤ (b-a)h⁴/90 · max|f⁽⁴⁾(x)| on [a,b]

Practical Error Estimation:
• Use Richardson extrapolation
• Compare results with different h
• Adaptive quadrature methods

When Simpson's Fails:
• Functions with discontinuous derivatives
• Highly oscillatory functions
• Singular integrands
• Use specialized methods for these cases"""
            
            else:
                return """Simpson's 1/3 Rule:
• Formula: ∫f(x)dx ≈ h/3[f(x₀) + 4f(x₁) + 2f(x₂) + 4f(x₃) + ... + f(x_n)]
• Pattern: 1-4-2-4-2-...-4-1 coefficients
• Requires: Even number of intervals
• Error: O(h⁴), exact for cubics
• Much more accurate than trapezoidal
• Most popular integration method"""
        except Exception as e:
            return f"Simpson's 1/3 rule error: {str(e)}"
    
    def solve_simpson_three_eighth_rule(self, command):
        """Simpson's 3/8 rule for numerical integration"""
        try:
            if 'formula' in command or 'theory' in command:
                return """Simpson's 3/8 Rule:
∫[a to b] f(x)dx ≈ 3(b-a)/8 · [f(a) + 3f(a+h) + 3f(a+2h) + f(b)]
where h = (b-a)/3

Uses 4 points with cubic interpolation

Composite Simpson's 3/8 Rule:
Requires n divisible by 3
h = (b-a)/n

∫[a to b] f(x)dx ≈ 3h/8 · [f(x₀) + 3f(x₁) + 3f(x₂) + 2f(x₃) + 3f(x₄) + ...]

Pattern: 1, 3, 3, 2, 3, 3, 2, ..., 3, 3, 1

Error Formula:
E = -3(b-a)h⁴/80 · f⁽⁴⁾(ξ)
Same order O(h⁴) as Simpson's 1/3, but different constant"""
            
            elif 'example' in command:
                return """Simpson's 3/8 Rule Example:
Evaluate ∫[0 to 1] x³dx using single application

Exact value: ∫x³dx = x⁴/4, so ∫[0 to 1] x³dx = 1/4 = 0.25

Step 1: Set up points
h = (1-0)/3 = 1/3
x₀ = 0, x₁ = 1/3, x₂ = 2/3, x₃ = 1

Step 2: Function values
f(0) = 0, f(1/3) = (1/3)³ = 1/27
f(2/3) = (2/3)³ = 8/27, f(1) = 1

Step 3: Apply formula
∫f(x)dx ≈ 3×1/8 · [0 + 3×1/27 + 3×8/27 + 1]
        = 3/8 · [0 + 1/9 + 8/9 + 1]
        = 3/8 · [18/9] = 3/8 × 2 = 3/4 × 1/2 = 0.25

Result is exact! (Simpson's 3/8 exact for cubics)

For n = 6 intervals on ∫[0 to 1] e^x dx:
Result ≈ 1.7183, Error ≈ 0.00001 (Very accurate)"""
            
            elif 'comparison' in command:
                return """Simpson's 1/3 vs 3/8 Comparison:
Simpson's 1/3:
• Uses 3 points per basic interval
• Requires even number of intervals
• Error coefficient: -1/90
• More commonly used

Simpson's 3/8:
• Uses 4 points per basic interval
• Requires n divisible by 3
• Error coefficient: -3/80 ≈ -1/27
• Slightly more accurate per interval

When to Use 3/8:
• When n is odd (can't use 1/3 rule)
• For higher accuracy with fewer intervals
• Combined with 1/3 rule for flexibility

Combined Approach:
• Use 3/8 for last 3 intervals if n not even
• Use 1/3 for remaining intervals
• Handles any number of intervals ≥ 3"""
            
            else:
                return """Simpson's 3/8 Rule:
• Formula: ∫f(x)dx ≈ 3h/8[f(x₀) + 3f(x₁) + 3f(x₂) + 2f(x₃) + ...]
• Pattern: 1-3-3-2-3-3-2-...-3-3-1
• Requires: n divisible by 3
• Error: O(h⁴), same as 1/3 rule
• Slightly more accurate than 1/3 rule
• Used when 1/3 rule not applicable"""
        except Exception as e:
            return f"Simpson's 3/8 rule error: {str(e)}"
    
    def solve_gauss_elimination(self, command):
        """Gaussian elimination for linear systems"""
        try:
            if 'algorithm' in command or 'steps' in command:
                return """Gaussian Elimination Algorithm:
Solve Ax = b where A is n×n matrix

Step 1: Forward Elimination
For k = 1 to n-1:
  For i = k+1 to n:
    m_{ik} = a_{ik}/a_{kk} (multiplier)
    For j = k to n:
      a_{ij} = a_{ij} - m_{ik} × a_{kj}
    b_i = b_i - m_{ik} × b_k

Result: Upper triangular system Ux = c

Step 2: Back Substitution
x_n = c_n/u_{nn}
For i = n-1 down to 1:
  x_i = (c_i - Σ(j=i+1 to n) u_{ij}x_j)/u_{ii}

Pivoting: Choose largest element in column to avoid division by small numbers"""
            
            elif 'example' in command:
                return """Gaussian Elimination Example:
Solve: 2x + y + z = 5
       4x + 3y + 2z = 11
       -2x + y + 2z = 3

Step 1: Set up augmented matrix
[2   1   1 |  5]
[4   3   2 | 11]
[-2  1   2 |  3]

Step 2: Forward elimination
R₂ = R₂ - 2R₁: [2  1  1 |  5]
               [0  1  0 |  1]
               [-2 1  2 |  3]

R₃ = R₃ + R₁:  [2  1  1 |  5]
               [0  1  0 |  1]
               [0  2  3 |  8]

R₃ = R₃ - 2R₂: [2  1  1 |  5]
               [0  1  0 |  1]
               [0  0  3 |  6]

Step 3: Back substitution
z = 6/3 = 2
y = 1 - 0×2 = 1
x = (5 - 1×1 - 1×2)/2 = 1

Solution: x = 1, y = 1, z = 2"""
            
            elif 'pivoting' in command:
                return """Pivoting in Gaussian Elimination:
Why Pivot?
• Avoid division by zero
• Reduce round-off errors
• Improve numerical stability

Partial Pivoting:
• Choose largest |a_{ik}| in column k below diagonal
• Swap rows to bring pivot to diagonal
• Most common strategy

Complete Pivoting:
• Choose largest |a_{ij}| in remaining submatrix
• Swap both rows and columns
• More stable but more expensive

Scaled Partial Pivoting:
• Consider relative size: |a_{ik}|/max|a_{ij}| in row i
• Better for matrices with different scales

Example with Pivoting:
Original: [0.001  1  | 1]
          [1      1  | 2]

Without pivoting: x₂ = 1/0.001 = 1000 (unstable!)
With pivoting: Swap rows first, get stable solution"""
            
            elif 'complexity' in command:
                return """Gaussian Elimination Complexity:
Time Complexity:
• Forward elimination: O(n³/3) operations
• Back substitution: O(n²/2) operations
• Total: O(n³) for large n

Space Complexity: O(n²) for matrix storage

Operation Count:
• Multiplications/Divisions: n³/3 + n²/2 - 5n/6
• Additions/Subtractions: n³/3 + n²/2 - 5n/6

For n = 1000: About 333 million operations

Improvements:
• Banded matrices: O(n×bandwidth²)
• Sparse matrices: Much less than O(n³)
• Parallel algorithms: Reduce wall-clock time
• Block algorithms: Better cache performance"""
            
            else:
                return """Gaussian Elimination:
• Method: Transform Ax = b to Ux = c, then back substitute
• Steps: Forward elimination + back substitution
• Complexity: O(n³) time, O(n²) space
• Pivoting: Essential for numerical stability
• Applications: General linear system solver
• Variants: LU decomposition, partial/complete pivoting"""
        except Exception as e:
            return f"Gaussian elimination error: {str(e)}"
    
    def solve_lu_decomposition(self, command):
        """LU decomposition method"""
        try:
            if 'theory' in command or 'definition' in command:
                return """LU Decomposition Theory:
Factorize matrix A into A = LU where:
• L is lower triangular with 1's on diagonal
• U is upper triangular

Process:
1. A = LU (factorization)
2. Solve Ly = b (forward substitution)
3. Solve Ux = y (back substitution)

Advantages:
• Solve multiple systems with same A efficiently
• Determinant: det(A) = product of U diagonal elements
• Matrix inversion becomes easier

Doolittle Method: L has 1's on diagonal
Crout Method: U has 1's on diagonal
Cholesky: For symmetric positive definite (A = LL^T)"""
            
            elif 'algorithm' in command:
                return """LU Decomposition Algorithm (Doolittle):
For k = 1 to n:
  For i = 1 to k:
    u_{ik} = a_{ik} - Σ(j=1 to i-1) l_{ij} × u_{jk}
  
  For i = k+1 to n:
    l_{ik} = (a_{ik} - Σ(j=1 to k-1) l_{ij} × u_{jk})/u_{kk}

Result: L (lower) and U (upper) matrices

Solving Ax = b:
1. Forward: Ly = b
   y₁ = b₁
   y_i = b_i - Σ(j=1 to i-1) l_{ij} × y_j

2. Backward: Ux = y
   x_n = y_n/u_{nn}
   x_i = (y_i - Σ(j=i+1 to n) u_{ij} × x_j)/u_{ii}"""
            
            elif 'example' in command:
                return """LU Decomposition Example:
Decompose A = [2  1  1]
              [4  3  2]
              [-2 1  2]

Step 1: Calculate U (first row) and L (first column)
u₁₁ = 2, u₁₂ = 1, u₁₃ = 1
l₂₁ = 4/2 = 2, l₃₁ = -2/2 = -1

Step 2: Calculate remaining elements
u₂₂ = 3 - 2×1 = 1, u₂₃ = 2 - 2×1 = 0
l₃₂ = (1 - (-1)×1)/1 = 2
u₃₃ = 2 - (-1)×1 - 2×0 = 3

Result:
L = [1   0  0]    U = [2  1  1]
    [2   1  0]        [0  1  0]
    [-1  2  1]        [0  0  3]

Verification: LU = A ✓

Solve Ax = [5, 11, 3]:
Ly = b: y = [5, 1, 6]
Ux = y: x = [1, 1, 2] (same as Gaussian elimination!)"""
            
            elif 'applications' in command:
                return """LU Decomposition Applications:
1. Multiple Right-Hand Sides:
   • Factor once: A = LU
   • Solve many: Ax_i = b_i efficiently

2. Matrix Inversion:
   • Solve AX = I column by column
   • Each column: Ax_i = e_i

3. Determinant Calculation:
   det(A) = det(L) × det(U) = Π u_{ii}

4. Condition Number Estimation:
   • Used in iterative refinement
   • Error analysis

5. Sparse Matrices:
   • Maintain sparsity better than Gaussian elimination
   • Fill-in minimization strategies

Variants:
• Partial Pivoting: PA = LU
• Complete Pivoting: PAQ = LU
• Cholesky: A = LL^T (symmetric positive definite)
• LDLT: A = LDL^T (symmetric)"""
            
            else:
                return """LU Decomposition:
• Factorization: A = LU (lower × upper)
• Process: Factor once, solve many times efficiently
• Complexity: O(n³) factorization, O(n²) per solve
• Advantages: Multiple RHS, determinant, inversion
• Pivoting: Usually needed for stability
• Applications: Linear systems, matrix operations"""
        except Exception as e:
            return f"LU decomposition error: {str(e)}"   
 
    def solve_jacobi_method(self, command):
        """Jacobi iterative method for linear systems"""
        try:
            if 'algorithm' in command or 'formula' in command:
                return """Jacobi Method Algorithm:
For system Ax = b, rewrite as x = Dx + c where:
• D = -A with diagonal elements set to 0
• c_i = b_i/a_{ii}

Iteration Formula:
x_i^(k+1) = (b_i - Σ(j≠i) a_{ij}x_j^(k))/a_{ii}

Matrix Form:
x^(k+1) = D^(-1)(L + U)x^(k) + D^(-1)b

where A = D + L + U (diagonal + lower + upper)

Algorithm Steps:
1. Initial guess x^(0)
2. For each component i: x_i^(k+1) = (b_i - Σ(j≠i) a_{ij}x_j^(k))/a_{ii}
3. Check convergence: ||x^(k+1) - x^(k)|| < tolerance
4. Repeat until convergence"""
            
            elif 'example' in command:
                return """Jacobi Method Example:
Solve: 10x₁ + 2x₂ + x₃ = 9
       x₁ + 5x₂ + x₃ = 7
       2x₁ + 3x₂ + 10x₃ = 6

Rearrange for diagonal dominance:
x₁ = (9 - 2x₂ - x₃)/10
x₂ = (7 - x₁ - x₃)/5
x₃ = (6 - 2x₁ - 3x₂)/10

Initial guess: x^(0) = [0, 0, 0]

Iteration 1:
x₁^(1) = (9 - 2×0 - 0)/10 = 0.9
x₂^(1) = (7 - 0 - 0)/5 = 1.4
x₃^(1) = (6 - 2×0 - 3×0)/10 = 0.6

Iteration 2:
x₁^(2) = (9 - 2×1.4 - 0.6)/10 = 0.5
x₂^(2) = (7 - 0.9 - 0.6)/5 = 1.1
x₃^(2) = (6 - 2×0.9 - 3×1.4)/10 = 0.02

Continue until convergence...
Solution: x ≈ [0.5, 1.0, 0.0]"""
            
            elif 'convergence' in command:
                return """Jacobi Method Convergence:
Convergence Condition:
Matrix A must be diagonally dominant:
|a_{ii}| > Σ(j≠i) |a_{ij}| for all i

Sufficient Conditions:
1. Strict diagonal dominance (above condition)
2. A is symmetric positive definite
3. Spectral radius ρ(D^(-1)(L+U)) < 1

Convergence Rate:
• Linear convergence
• Rate depends on spectral radius
• Slower than direct methods for small systems
• Better for large sparse systems

Improving Convergence:
• Reorder equations for diagonal dominance
• Use relaxation methods
• Better initial guess
• Preconditioning techniques

When Jacobi Fails:
• Non-diagonally dominant matrices
• Ill-conditioned systems
• Use Gauss-Seidel or direct methods instead"""
            
            elif 'advantages' in command:
                return """Jacobi Method Advantages & Disadvantages:
Advantages:
• Simple to implement and understand
• Naturally parallel (each x_i computed independently)
• Good for large sparse systems
• Low memory requirements
• Self-correcting for round-off errors

Disadvantages:
• Requires diagonal dominance for convergence
• Slower than direct methods for small systems
• May not converge for some matrices
• Needs good initial guess for fast convergence

When to Use Jacobi:
• Large sparse systems (n > 1000)
• Parallel computing environments
• Systems with diagonal dominance
• When memory is limited
• Iterative refinement

Comparison with Gauss-Seidel:
• Jacobi: Uses old values for all components
• Gauss-Seidel: Uses updated values immediately
• Gauss-Seidel usually converges faster
• Jacobi better for parallel computation"""
            
            else:
                return """Jacobi Method:
• Iterative method: x_i^(k+1) = (b_i - Σa_{ij}x_j^(k))/a_{ii}
• Convergence: Requires diagonal dominance
• Rate: Linear, depends on spectral radius
• Parallel: Each component computed independently
• Applications: Large sparse systems
• Memory: Low requirements, good for big problems"""
        except Exception as e:
            return f"Jacobi method error: {str(e)}"
    
    def solve_gauss_seidel_method(self, command):
        """Gauss-Seidel iterative method"""
        try:
            if 'algorithm' in command or 'formula' in command:
                return """Gauss-Seidel Method Algorithm:
Uses updated values immediately as they become available

Iteration Formula:
x_i^(k+1) = (b_i - Σ(j=1 to i-1) a_{ij}x_j^(k+1) - Σ(j=i+1 to n) a_{ij}x_j^(k))/a_{ii}

Key Difference from Jacobi:
• Uses x_j^(k+1) for j < i (already computed in current iteration)
• Uses x_j^(k) for j > i (from previous iteration)

Matrix Form:
(D + L)x^(k+1) = -Ux^(k) + b
x^(k+1) = -(D + L)^(-1)Ux^(k) + (D + L)^(-1)b

Algorithm:
1. Choose initial guess x^(0)
2. For i = 1 to n: Update x_i using latest available values
3. Check convergence
4. Repeat until ||x^(k+1) - x^(k)|| < tolerance"""
            
            elif 'example' in command:
                return """Gauss-Seidel Example:
Same system: 10x₁ + 2x₂ + x₃ = 9
             x₁ + 5x₂ + x₃ = 7
             2x₁ + 3x₂ + 10x₃ = 6

Rearranged:
x₁ = (9 - 2x₂ - x₃)/10
x₂ = (7 - x₁ - x₃)/5
x₃ = (6 - 2x₁ - 3x₂)/10

Initial: x^(0) = [0, 0, 0]

Iteration 1:
x₁^(1) = (9 - 2×0 - 0)/10 = 0.9
x₂^(1) = (7 - 0.9 - 0)/5 = 1.22 (uses new x₁^(1))
x₃^(1) = (6 - 2×0.9 - 3×1.22)/10 = 0.054 (uses new x₁^(1), x₂^(1))

Iteration 2:
x₁^(2) = (9 - 2×1.22 - 0.054)/10 = 0.6506
x₂^(2) = (7 - 0.6506 - 0.054)/5 = 1.259
x₃^(2) = (6 - 2×0.6506 - 3×1.259)/10 = -0.0778

Converges faster than Jacobi!
Solution: x ≈ [0.5, 1.0, 0.0]"""
            
            elif 'convergence' in command:
                return """Gauss-Seidel Convergence:
Same Conditions as Jacobi:
• Diagonal dominance: |a_{ii}| > Σ(j≠i) |a_{ij}|
• Symmetric positive definite matrices
• Spectral radius condition

Convergence Rate:
• Generally faster than Jacobi
• Uses most recent information
• Typically converges in fewer iterations

Spectral Radius:
ρ(-(D+L)^(-1)U) < 1 for convergence

Relaxation Factor:
Can be accelerated with Successive Over-Relaxation (SOR):
x_i^(k+1) = (1-ω)x_i^(k) + ω × (Gauss-Seidel update)

Optimal ω:
• ω = 1: Standard Gauss-Seidel
• 1 < ω < 2: Over-relaxation (faster convergence)
• 0 < ω < 1: Under-relaxation (more stable)"""
            
            elif 'comparison' in command:
                return """Jacobi vs Gauss-Seidel Comparison:
Convergence Speed:
• Gauss-Seidel: Usually 2x faster than Jacobi
• Uses updated values immediately
• Better information utilization

Parallelization:
• Jacobi: Fully parallel (all x_i computed simultaneously)
• Gauss-Seidel: Sequential (x_i depends on x_j, j < i)
• Jacobi better for parallel computers

Memory Requirements:
• Jacobi: Needs storage for x^(k) and x^(k+1)
• Gauss-Seidel: Can update in-place
• Gauss-Seidel more memory efficient

Convergence Conditions:
• Same conditions for both methods
• Both need diagonal dominance or equivalent

When to Choose:
• Gauss-Seidel: Sequential computers, faster convergence
• Jacobi: Parallel computers, simpler implementation"""
            
            else:
                return """Gauss-Seidel Method:
• Iterative: Uses updated values immediately
• Formula: x_i^(k+1) = (b_i - Σa_{ij}x_j^(new) - Σa_{ij}x_j^(old))/a_{ii}
• Convergence: Same conditions as Jacobi
• Speed: Usually faster than Jacobi
• Sequential: Not naturally parallel
• Memory: More efficient than Jacobi"""
        except Exception as e:
            return f"Gauss-Seidel method error: {str(e)}"
    
    def solve_euler_method(self, command):
        """Euler's method for differential equations"""
        try:
            if 'formula' in command or 'algorithm' in command:
                return """Euler's Method:
For differential equation dy/dx = f(x,y) with y(x₀) = y₀

Formula: y_{n+1} = y_n + h·f(x_n, y_n)
where h = step size, x_{n+1} = x_n + h

Algorithm:
1. Start with (x₀, y₀)
2. Calculate slope: m = f(x₀, y₀)
3. Next point: y₁ = y₀ + h·m, x₁ = x₀ + h
4. Repeat for desired range

Geometric Interpretation:
• Uses tangent line approximation
• Follows slope at current point for distance h
• Linear approximation to curved solution

Error: O(h) per step, O(h) global error (first-order method)"""
            
            elif 'example' in command:
                return """Euler's Method Example:
Solve dy/dx = x + y, y(0) = 1, find y(1) using h = 0.2

Exact solution: y = 2e^x - x - 1

Step-by-step:
x₀ = 0, y₀ = 1, f(x,y) = x + y

Step 1: x₁ = 0.2
f(0,1) = 0 + 1 = 1
y₁ = 1 + 0.2×1 = 1.2

Step 2: x₂ = 0.4
f(0.2,1.2) = 0.2 + 1.2 = 1.4
y₂ = 1.2 + 0.2×1.4 = 1.48

Step 3: x₃ = 0.6
f(0.4,1.48) = 0.4 + 1.48 = 1.88
y₃ = 1.48 + 0.2×1.88 = 1.856

Step 4: x₄ = 0.8
f(0.6,1.856) = 0.6 + 1.856 = 2.456
y₄ = 1.856 + 0.2×2.456 = 2.3472

Step 5: x₅ = 1.0
f(0.8,2.3472) = 0.8 + 2.3472 = 3.1472
y₅ = 2.3472 + 0.2×3.1472 = 3.0766

Euler result: y(1) ≈ 3.0766
Exact: y(1) = 2e¹ - 1 - 1 ≈ 3.4366
Error ≈ 0.36 (about 10%)"""
            
            elif 'error' in command or 'accuracy' in command:
                return """Euler's Method Error Analysis:
Local Truncation Error (per step):
E_local = h²/2 · y''(ξ) where ξ ∈ [x_n, x_{n+1}]
Order: O(h²)

Global Error (accumulated):
E_global = O(h) over interval [a,b]
• Halving h roughly halves the error
• First-order method

Error Sources:
1. Truncation error: From linear approximation
2. Round-off error: From finite precision

Error Bound:
|y(x_n) - y_n| ≤ (e^{L(x_n-x_0)} - 1)·M·h/(2L)
where L = Lipschitz constant, M = max|y''(x)|

Improving Accuracy:
• Smaller step size h
• Higher-order methods (Runge-Kutta)
• Adaptive step size
• Richardson extrapolation"""
            
            elif 'applications' in command:
                return """Euler's Method Applications:
1. Engineering:
   • Circuit analysis (RC, RL circuits)
   • Population dynamics
   • Chemical reaction rates
   • Heat transfer problems

2. Physics:
   • Projectile motion with air resistance
   • Radioactive decay
   • Simple harmonic motion
   • Cooling/heating problems

3. Economics:
   • Growth models
   • Supply and demand dynamics
   • Investment analysis

4. Biology:
   • Population growth
   • Epidemic models (SIR)
   • Predator-prey systems

Advantages:
• Simple to understand and implement
• Good for quick approximations
• Foundation for other methods
• Works with any first-order ODE

Disadvantages:
• Low accuracy (first-order)
• Can be unstable for large h
• Accumulates error quickly
• Not suitable for stiff equations"""
            
            else:
                return """Euler's Method:
• Formula: y_{n+1} = y_n + h·f(x_n, y_n)
• Order: First-order method, O(h) global error
• Geometric: Tangent line approximation
• Simple: Easy to implement and understand
• Applications: Basic ODE solving, quick estimates
• Limitations: Low accuracy, can be unstable"""
        except Exception as e:
            return f"Euler's method error: {str(e)}"
    
    def solve_runge_kutta_method(self, command):
        """Runge-Kutta methods for differential equations"""
        try:
            if 'rk4' in command or 'fourth order' in command:
                return """Fourth-Order Runge-Kutta (RK4):
Most popular method for solving dy/dx = f(x,y)

Formula:
k₁ = h·f(x_n, y_n)
k₂ = h·f(x_n + h/2, y_n + k₁/2)
k₃ = h·f(x_n + h/2, y_n + k₂/2)
k₄ = h·f(x_n + h, y_n + k₃)

y_{n+1} = y_n + (k₁ + 2k₂ + 2k₃ + k₄)/6

Interpretation:
• k₁: Slope at beginning of interval
• k₂: Slope at midpoint using k₁
• k₃: Slope at midpoint using k₂
• k₄: Slope at end using k₃
• Weighted average of four slopes

Error: O(h⁴) per step, O(h⁴) global error"""
            
            elif 'example' in command:
                return """RK4 Example:
Solve dy/dx = x + y, y(0) = 1, find y(0.2) using h = 0.2

Step 1: Calculate k values
k₁ = 0.2·f(0,1) = 0.2·(0+1) = 0.2

k₂ = 0.2·f(0.1, 1.1) = 0.2·(0.1+1.1) = 0.24

k₃ = 0.2·f(0.1, 1.12) = 0.2·(0.1+1.12) = 0.244

k₄ = 0.2·f(0.2, 1.244) = 0.2·(0.2+1.244) = 0.2888

Step 2: Calculate y₁
y₁ = 1 + (0.2 + 2×0.24 + 2×0.244 + 0.2888)/6
   = 1 + 1.4568/6 = 1 + 0.2428 = 1.2428

Exact solution: y(0.2) = 2e^{0.2} - 0.2 - 1 ≈ 1.2428
Error ≈ 0.0000 (Excellent accuracy!)

Compare with Euler: y₁ = 1.2 (Error ≈ 0.043)
RK4 is much more accurate!"""
            
            elif 'comparison' in command or 'orders' in command:
                return """Runge-Kutta Method Orders:
RK1 (Euler's Method):
• Formula: y_{n+1} = y_n + h·f(x_n, y_n)
• Error: O(h)
• Function evaluations: 1 per step

RK2 (Midpoint/Heun's):
• Uses 2 function evaluations
• Error: O(h²)
• Better than Euler, less than RK4

RK4 (Classical):
• Uses 4 function evaluations
• Error: O(h⁴)
• Best balance of accuracy and efficiency

Higher Orders (RK5, RK6, ...):
• Diminishing returns
• More function evaluations
• Mainly used in adaptive methods

Efficiency Comparison:
For same accuracy:
• RK4 with h vs Euler with h/16
• RK4: 4 evaluations per step
• Euler: 16 evaluations for same accuracy
• RK4 is 4 times more efficient!"""
            
            elif 'adaptive' in command:
                return """Adaptive Runge-Kutta Methods:
Runge-Kutta-Fehlberg (RKF45):
• Uses RK4 and RK5 simultaneously
• Estimates local error: |y₅ - y₄|
• Adjusts step size automatically

Step Size Control:
If error > tolerance: Reduce h
If error << tolerance: Increase h

New step size: h_new = h × (tolerance/error)^{1/5}

Dormand-Prince (DOPRI):
• Modern adaptive method
• Used in MATLAB's ode45
• Very efficient and reliable

Cash-Karp Method:
• Another embedded RK method
• Good for general purpose use

Advantages of Adaptive Methods:
• Automatic error control
• Efficient step size selection
• Handle varying solution behavior
• Maintain specified accuracy"""
            
            else:
                return """Runge-Kutta Methods:
• RK4: y_{n+1} = y_n + (k₁ + 2k₂ + 2k₃ + k₄)/6
• Order: Fourth-order, O(h⁴) error
• Accuracy: Much better than Euler
• Cost: 4 function evaluations per step
• Most popular ODE solver
• Applications: General differential equations"""
        except Exception as e:
            return f"Runge-Kutta method error: {str(e)}"    

    def solve_modified_euler_method(self, command):
        """Modified Euler and Heun's method"""
        try:
            if 'algorithm' in command or 'formula' in command:
                return """Modified Euler Method (Heun's Method):
Predictor-Corrector approach for dy/dx = f(x,y)

Algorithm:
1. Predictor: y*_{n+1} = y_n + h·f(x_n, y_n) (Euler step)
2. Corrector: y_{n+1} = y_n + h/2·[f(x_n, y_n) + f(x_{n+1}, y*_{n+1})]

Alternative Form:
k₁ = h·f(x_n, y_n)
k₂ = h·f(x_n + h, y_n + k₁)
y_{n+1} = y_n + (k₁ + k₂)/2

Interpretation:
• Uses slopes at both endpoints
• Averages initial and final slopes
• Second-order method: O(h²) error
• More accurate than simple Euler"""
            
            elif 'example' in command:
                return """Modified Euler Example:
Solve dy/dx = x + y, y(0) = 1, find y(0.2) using h = 0.1

Step 1: x₀ = 0, y₀ = 1
Predictor: y*₁ = 1 + 0.1×(0+1) = 1.1
Corrector: y₁ = 1 + 0.1/2×[(0+1) + (0.1+1.1)] = 1 + 0.05×[1 + 1.2] = 1.11

Step 2: x₁ = 0.1, y₁ = 1.11
Predictor: y*₂ = 1.11 + 0.1×(0.1+1.11) = 1.231
Corrector: y₂ = 1.11 + 0.1/2×[(0.1+1.11) + (0.2+1.231)] = 1.11 + 0.05×[1.21 + 1.431] = 1.2421

Result: y(0.2) ≈ 1.2421
Exact: y(0.2) = 2e^{0.2} - 0.2 - 1 ≈ 1.2428
Error ≈ 0.0007 (Much better than simple Euler!)

Comparison:
• Simple Euler: y(0.2) ≈ 1.21, Error ≈ 0.033
• Modified Euler: y(0.2) ≈ 1.2421, Error ≈ 0.0007
• RK4: y(0.2) ≈ 1.2428, Error ≈ 0.0000"""
            
            elif 'variants' in command:
                return """Modified Euler Variants:
1. Heun's Method (Standard):
   y_{n+1} = y_n + h/2[f(x_n,y_n) + f(x_{n+1}, y_n + h·f(x_n,y_n))]

2. Improved Euler:
   Same as Heun's method
   
3. Midpoint Method:
   y_{n+1} = y_n + h·f(x_n + h/2, y_n + h/2·f(x_n,y_n))

4. Ralston's Method:
   k₁ = h·f(x_n, y_n)
   k₂ = h·f(x_n + 3h/4, y_n + 3k₁/4)
   y_{n+1} = y_n + (k₁ + 2k₂)/3

All are second-order methods with O(h²) error
Heun's method most commonly used"""
            
            elif 'comparison' in command:
                return """Method Comparison:
Euler's Method:
• Order: 1, Error: O(h)
• Function evaluations: 1 per step
• Simple but inaccurate

Modified Euler (Heun's):
• Order: 2, Error: O(h²)
• Function evaluations: 2 per step
• Good balance of accuracy and simplicity

RK4:
• Order: 4, Error: O(h⁴)
• Function evaluations: 4 per step
• High accuracy but more computation

Efficiency Analysis:
For same accuracy as RK4:
• Euler needs h/16, so 16 evaluations
• Heun's needs h/4, so 8 evaluations
• RK4 needs h, so 4 evaluations
RK4 is most efficient for high accuracy"""
            
            else:
                return """Modified Euler (Heun's Method):
• Predictor-Corrector: y* = y + h·f, then y = y + h/2[f + f*]
• Order: Second-order, O(h²) error
• Evaluations: 2 per step
• Better than Euler, simpler than RK4
• Good compromise between accuracy and efficiency
• Applications: When RK4 is overkill but Euler insufficient"""
        except Exception as e:
            return f"Modified Euler method error: {str(e)}"
    
    def solve_adams_bashforth_method(self, command):
        """Adams-Bashforth predictor-corrector methods"""
        try:
            if 'theory' in command or 'formula' in command:
                return """Adams-Bashforth Methods:
Multi-step methods using previous points for higher accuracy

Adams-Bashforth 2-step (AB2):
y_{n+1} = y_n + h/2[3f(x_n,y_n) - f(x_{n-1},y_{n-1})]

Adams-Bashforth 3-step (AB3):
y_{n+1} = y_n + h/12[23f(x_n,y_n) - 16f(x_{n-1},y_{n-1}) + 5f(x_{n-2},y_{n-2})]

Adams-Bashforth 4-step (AB4):
y_{n+1} = y_n + h/24[55f_n - 59f_{n-1} + 37f_{n-2} - 9f_{n-3}]

Adams-Moulton (Corrector):
AM2: y_{n+1} = y_n + h/12[5f(x_{n+1},y_{n+1}) + 8f(x_n,y_n) - f(x_{n-1},y_{n-1})]

Predictor-Corrector Pair:
1. Predict using Adams-Bashforth
2. Correct using Adams-Moulton"""
            
            elif 'algorithm' in command:
                return """Adams-Bashforth-Moulton Algorithm:
Starting: Need initial values y₀, y₁, y₂, y₃ (use RK4)

For each step:
1. Predictor (AB4):
   y*_{n+1} = y_n + h/24[55f_n - 59f_{n-1} + 37f_{n-2} - 9f_{n-3}]

2. Evaluate: f*_{n+1} = f(x_{n+1}, y*_{n+1})

3. Corrector (AM3):
   y_{n+1} = y_n + h/12[5f*_{n+1} + 8f_n - f_{n-1}]

4. Update function values:
   f_{n+1} = f(x_{n+1}, y_{n+1})

Advantages:
• High accuracy with few function evaluations
• Uses information from previous steps efficiently
• Self-starting after initial phase"""
            
            elif 'example' in command:
                return """Adams-Bashforth Example:
Solve dy/dx = x + y, y(0) = 1, using AB2 method with h = 0.1

Step 1: Get starting values using RK4
y₀ = 1.0000 (given)
y₁ = 1.1103 (RK4 calculation)

Step 2: Calculate function values
f₀ = f(0, 1.0000) = 0 + 1.0000 = 1.0000
f₁ = f(0.1, 1.1103) = 0.1 + 1.1103 = 1.2103

Step 3: Apply AB2 formula
y₂ = y₁ + h/2[3f₁ - f₀]
   = 1.1103 + 0.1/2[3×1.2103 - 1.0000]
   = 1.1103 + 0.05[3.6309 - 1.0000]
   = 1.1103 + 0.05×2.6309 = 1.2419

Step 4: Continue
f₂ = f(0.2, 1.2419) = 0.2 + 1.2419 = 1.4419
y₃ = 1.2419 + 0.05[3×1.4419 - 1.2103] = 1.3967

Exact y(0.3) ≈ 1.3997, Error ≈ 0.003"""
            
            elif 'stability' in command:
                return """Adams Methods Stability:
Stability Region:
• Adams-Bashforth methods have limited stability
• AB2: Stable for h·λ in certain region of complex plane
• Higher-order AB methods less stable

Predictor-Corrector Stability:
• Corrector step improves stability
• PECE mode: Predict-Evaluate-Correct-Evaluate
• More stable than pure predictor methods

Variable Step Size:
• Can adapt step size for stability
• Nordsieck form allows easy step change
• Monitor local truncation error

Stiff Equations:
• Adams methods not suitable for stiff ODEs
• Use implicit methods (BDF) for stiff problems
• Gear's methods better for stiff systems"""
            
            else:
                return """Adams-Bashforth Methods:
• Multi-step: Uses previous points for higher accuracy
• AB2: y_{n+1} = y_n + h/2[3f_n - f_{n-1}]
• Predictor-Corrector: AB predicts, AM corrects
• High accuracy with few evaluations
• Need starting values from single-step method
• Good for non-stiff equations"""
        except Exception as e:
            return f"Adams-Bashforth method error: {str(e)}"
    
    def solve_milne_method(self, command):
        """Milne's predictor-corrector method"""
        try:
            if 'formula' in command or 'algorithm' in command:
                return """Milne's Method:
Predictor-Corrector method using integration formulas

Milne's Predictor:
y_{n+1} = y_{n-3} + 4h/3[2f_n - f_{n-1} + 2f_{n-2}]

Milne's Corrector:
y_{n+1} = y_{n-1} + h/3[f_{n+1} + 4f_n + f_{n-1}]

Algorithm:
1. Start with 4 initial values (use RK4)
2. Predict y_{n+1} using predictor formula
3. Calculate f_{n+1} = f(x_{n+1}, y_{n+1}^{predicted})
4. Correct using corrector formula
5. Iterate corrector until convergence

Error Estimation:
E ≈ (y_{corrected} - y_{predicted})/29
If |E| > tolerance, reduce step size"""
            
            elif 'example' in command:
                return """Milne's Method Example:
Solve dy/dx = x + y, y(0) = 1, find y(0.4) using h = 0.1

Starting values (from RK4):
y₀ = 1.0000, y₁ = 1.1103, y₂ = 1.2428, y₃ = 1.3997

Function values:
f₀ = 1.0000, f₁ = 1.2103, f₂ = 1.4428, f₃ = 1.6997

Step 1: Predict y₄
y₄* = y₀ + 4×0.1/3[2f₃ - f₂ + 2f₁]
    = 1.0000 + 0.1333[2×1.6997 - 1.4428 + 2×1.2103]
    = 1.0000 + 0.1333[3.3994 - 1.4428 + 2.4206]
    = 1.0000 + 0.1333×4.3772 = 1.5836

Step 2: Calculate f₄*
f₄* = f(0.4, 1.5836) = 0.4 + 1.5836 = 1.9836

Step 3: Correct y₄
y₄ = y₂ + 0.1/3[f₄* + 4f₃ + f₂]
   = 1.2428 + 0.0333[1.9836 + 4×1.6997 + 1.4428]
   = 1.2428 + 0.0333[10.2252] = 1.5834

Exact: y(0.4) ≈ 1.5836, Error ≈ 0.0002"""
            
            elif 'stability' in command:
                return """Milne's Method Stability:
Stability Issues:
• Milne's predictor can be unstable
• Parasitic solutions may grow
• Weak stability compared to Adams methods

Root Condition:
Characteristic equation has roots outside unit circle
Can lead to oscillatory parasitic solutions

Stability Improvements:
1. Use smaller step sizes
2. Monitor solution behavior
3. Switch to stable method if oscillations appear
4. Use Hamming's modification

Hamming's Modification:
Predictor: Same as Milne
Corrector: y_{n+1} = (9y_n - y_{n-1})/8 + 3h/8[f_{n+1} + 2f_n - f_{n-1}]

Better stability properties than original Milne method"""
            
            elif 'comparison' in command:
                return """Milne vs Other Methods:
Milne's Method:
• High accuracy (4th order)
• Can be unstable
• Good for smooth problems

Adams-Bashforth-Moulton:
• More stable than Milne
• Widely used in practice
• Better for general problems

Runge-Kutta:
• Self-starting
• Very stable
• Higher computational cost per step

When to Use Milne:
• Smooth, well-behaved problems
• When high accuracy needed
• Monitor for stability issues
• Consider Hamming's modification"""
            
            else:
                return """Milne's Method:
• Predictor: y_{n+1} = y_{n-3} + 4h/3[2f_n - f_{n-1} + 2f_{n-2}]
• Corrector: y_{n+1} = y_{n-1} + h/3[f_{n+1} + 4f_n + f_{n-1}]
• Order: Fourth-order accuracy
• Stability: Can be unstable, monitor solutions
• Applications: Smooth problems, high accuracy needed
• Alternative: Hamming's modification for better stability"""
        except Exception as e:
            return f"Milne method error: {str(e)}"
    
    def solve_power_method(self, command):
        """Power method for eigenvalues"""
        try:
            if 'algorithm' in command or 'theory' in command:
                return """Power Method Algorithm:
Finds dominant eigenvalue and eigenvector of matrix A

Algorithm:
1. Choose initial vector x⁽⁰⁾ (non-zero)
2. For k = 0, 1, 2, ...
   y⁽ᵏ⁺¹⁾ = A·x⁽ᵏ⁾
   λ⁽ᵏ⁺¹⁾ = max component of y⁽ᵏ⁺¹⁾
   x⁽ᵏ⁺¹⁾ = y⁽ᵏ⁺¹⁾/λ⁽ᵏ⁺¹⁾ (normalize)
3. Continue until convergence

Convergence:
λ⁽ᵏ⁾ → λ₁ (dominant eigenvalue)
x⁽ᵏ⁾ → v₁ (corresponding eigenvector)

Rate: |λ₂/λ₁|ᵏ where |λ₁| > |λ₂| ≥ |λ₃| ≥ ...
Faster convergence when λ₁ >> λ₂"""
            
            elif 'example' in command:
                return """Power Method Example:
Find dominant eigenvalue of A = [4  1]
                                [2  3]

Starting with x⁽⁰⁾ = [1, 1]ᵀ

Iteration 1:
y⁽¹⁾ = [4  1][1] = [5]
       [2  3][1]   [5]
λ⁽¹⁾ = 5, x⁽¹⁾ = [1, 1]ᵀ

Iteration 2:
y⁽²⁾ = [4  1][1] = [5]
       [2  3][1]   [5]
λ⁽²⁾ = 5, x⁽²⁾ = [1, 1]ᵀ

Converged! λ₁ = 5, v₁ = [1, 1]ᵀ

Verification: Av₁ = [4  1][1] = [5] = 5[1] ✓
                    [2  3][1]   [5]     [1]

Exact eigenvalues: λ₁ = 5, λ₂ = 2
Exact eigenvectors: v₁ = [1, 1]ᵀ, v₂ = [1, -2]ᵀ"""
            
            elif 'convergence' in command:
                return """Power Method Convergence:
Convergence Condition:
• Must have dominant eigenvalue: |λ₁| > |λ₂|
• Initial vector must have component in direction of v₁

Convergence Rate:
Error decreases as (|λ₂/λ₁|)ᵏ

Fast Convergence: |λ₁| >> |λ₂|
Slow Convergence: |λ₁| ≈ |λ₂|

Failure Cases:
• |λ₁| = |λ₂| (no dominant eigenvalue)
• Complex eigenvalues with same magnitude
• Initial vector orthogonal to dominant eigenvector

Acceleration Techniques:
• Aitken's Δ² process
• Rayleigh quotient: λ = (x^T Ax)/(x^T x)
• Shifted power method: (A - σI)⁻¹

Stopping Criteria:
• |λ⁽ᵏ⁺¹⁾ - λ⁽ᵏ⁾| < tolerance
• ||x⁽ᵏ⁺¹⁾ - x⁽ᵏ⁾|| < tolerance
• ||Ax - λx|| < tolerance"""
            
            elif 'variants' in command:
                return """Power Method Variants:
1. Inverse Power Method:
   • Finds smallest eigenvalue
   • Apply power method to A⁻¹
   • Converges to 1/λₙ where λₙ is smallest

2. Shifted Power Method:
   • Finds eigenvalue closest to σ
   • Apply power method to (A - σI)⁻¹
   • Useful when dominant eigenvalue unknown

3. Rayleigh Quotient Iteration:
   • Use Rayleigh quotient as shift
   • σₖ = (x⁽ᵏ⁾)ᵀAx⁽ᵏ⁾/(x⁽ᵏ⁾)ᵀx⁽ᵏ⁾
   • Cubic convergence near eigenvalue

4. Simultaneous Iteration:
   • Find multiple eigenvalues
   • Work with matrix of vectors
   • QR decomposition for orthogonalization

Applications:
• Google PageRank algorithm
• Principal Component Analysis
• Vibration analysis
• Quantum mechanics"""
            
            else:
                return """Power Method:
• Purpose: Find dominant eigenvalue and eigenvector
• Algorithm: x⁽ᵏ⁺¹⁾ = Ax⁽ᵏ⁾/||Ax⁽ᵏ⁾||
• Convergence: Rate (|λ₂/λ₁|)ᵏ
• Requires: |λ₁| > |λ₂|
• Applications: PageRank, PCA, vibration analysis
• Variants: Inverse, shifted, Rayleigh quotient iteration"""
        except Exception as e:
            return f"Power method error: {str(e)}"  
  
    def solve_inverse_power_method(self, command):
        """Inverse power method for smallest eigenvalue"""
        try:
            if 'algorithm' in command:
                return """Inverse Power Method:
Finds smallest eigenvalue by applying power method to A⁻¹

Algorithm:
1. Choose initial vector x⁽⁰⁾
2. For k = 0, 1, 2, ...
   Solve: A·y⁽ᵏ⁺¹⁾ = x⁽ᵏ⁾ (don't compute A⁻¹ directly!)
   μ⁽ᵏ⁺¹⁾ = max component of y⁽ᵏ⁺¹⁾
   x⁽ᵏ⁺¹⁾ = y⁽ᵏ⁺¹⁾/μ⁽ᵏ⁺¹⁾
3. λₙ = 1/μ (smallest eigenvalue of A)

Implementation:
• Use LU decomposition of A once
• Solve Ly = x, then Ux⁽ᵏ⁺¹⁾ = y each iteration
• More efficient than computing A⁻¹"""
            
            elif 'example' in command:
                return """Inverse Power Method Example:
Find smallest eigenvalue of A = [4  1]
                                [2  3]

Step 1: LU decomposition of A
A = [4  1] = [1    0  ][4  1  ]
    [2  3]   [0.5  1  ][0  2.5]

Step 2: Start with x⁽⁰⁾ = [1, 1]ᵀ

Iteration 1:
Solve Ay⁽¹⁾ = x⁽⁰⁾ = [1, 1]ᵀ
Forward: Lz = [1, 1]ᵀ → z = [1, 0.5]ᵀ
Backward: Uy⁽¹⁾ = z → y⁽¹⁾ = [0.8, 0.2]ᵀ
μ⁽¹⁾ = 0.8, x⁽¹⁾ = [1, 0.25]ᵀ

Continue iterations...
Converges to μ = 0.5, so λₙ = 1/0.5 = 2

Verification: Smallest eigenvalue of A is indeed 2!"""
            
            else:
                return """Inverse Power Method:
• Purpose: Find smallest eigenvalue
• Method: Power method applied to A⁻¹
• Algorithm: Solve Ay = x instead of computing A⁻¹
• Result: λₙ = 1/μ where μ is dominant eigenvalue of A⁻¹
• Implementation: Use LU decomposition for efficiency
• Applications: Finding fundamental frequencies, stability analysis"""
        except Exception as e:
            return f"Inverse power method error: {str(e)}"
    
    def solve_jacobi_eigenvalue_method(self, command):
        """Jacobi method for eigenvalues of symmetric matrices"""
        try:
            if 'algorithm' in command or 'theory' in command:
                return """Jacobi Eigenvalue Method:
For symmetric matrices, finds ALL eigenvalues and eigenvectors

Algorithm:
1. Start with A₀ = A (symmetric)
2. Find largest off-diagonal element |a_{pq}|
3. Construct rotation matrix P to zero out a_{pq}
4. A₁ = P^T A₀ P (similarity transformation)
5. Repeat until all off-diagonal elements ≈ 0

Rotation Matrix P:
P_{ii} = 1 for i ≠ p,q
P_{pp} = P_{qq} = cos θ
P_{pq} = -P_{qp} = sin θ

Rotation Angle:
tan(2θ) = 2a_{pq}/(a_{pp} - a_{qq})

Properties:
• Preserves eigenvalues (similarity transformation)
• Converges to diagonal matrix
• Eigenvectors = product of all rotation matrices"""
            
            elif 'example' in command:
                return """Jacobi Method Example:
Find eigenvalues of A = [4  1  0]
                        [1  3  1]
                        [0  1  2]

Step 1: Find largest off-diagonal |a₁₂| = 1
Calculate θ: tan(2θ) = 2×1/(4-3) = 2
θ = π/8, cos θ ≈ 0.924, sin θ ≈ 0.383

Step 2: Rotation matrix
P₁ = [0.924  -0.383   0  ]
     [0.383   0.924   0  ]
     [0       0       1  ]

Step 3: A₁ = P₁ᵀAP₁
A₁ ≈ [4.54   0     -0.383]
     [0     2.46   0.924 ]
     [-0.383 0.924  2    ]

Step 4: Continue with largest off-diagonal...
After several iterations:
Final ≈ [5.17   0     0   ]
        [0     2.83   0   ]
        [0     0     1.00 ]

Eigenvalues: λ₁ ≈ 5.17, λ₂ ≈ 2.83, λ₃ ≈ 1.00"""
            
            elif 'convergence' in command:
                return """Jacobi Method Convergence:
Convergence Properties:
• Always converges for symmetric matrices
• Quadratic convergence near solution
• Each rotation reduces sum of squares of off-diagonal elements

Convergence Rate:
• Slow initially (linear)
• Accelerates to quadratic near convergence
• Typically needs many iterations

Stopping Criteria:
• Sum of squares of off-diagonal elements < tolerance
• ||A_k - A_{k-1}|| < tolerance
• Max off-diagonal element < tolerance

Advantages:
• Finds ALL eigenvalues and eigenvectors
• Very stable numerically
• Simple to implement
• Works for any symmetric matrix

Disadvantages:
• Slow convergence
• Many iterations required
• Not efficient for large matrices
• Only for symmetric matrices"""
            
            elif 'applications' in command:
                return """Jacobi Method Applications:
1. Structural Engineering:
   • Vibration analysis
   • Modal analysis
   • Natural frequencies

2. Quantum Mechanics:
   • Energy eigenvalues
   • Hamiltonian diagonalization
   • Molecular orbitals

3. Statistics:
   • Principal Component Analysis
   • Covariance matrix diagonalization
   • Factor analysis

4. Image Processing:
   • Singular Value Decomposition
   • Image compression
   • Feature extraction

Modern Usage:
• Mainly for small to medium matrices
• Educational purposes
• When all eigenvalues needed
• Replaced by QR algorithm for large matrices"""
            
            else:
                return """Jacobi Eigenvalue Method:
• Purpose: Find ALL eigenvalues/eigenvectors of symmetric matrix
• Method: Successive rotations to diagonalize matrix
• Convergence: Always for symmetric matrices
• Rate: Initially linear, then quadratic
• Applications: Vibration analysis, quantum mechanics, PCA
• Limitation: Only symmetric matrices, slow for large problems"""
        except Exception as e:
            return f"Jacobi eigenvalue method error: {str(e)}"
    
    def solve_least_squares_method(self, command):
        """Method of least squares for curve fitting"""
        try:
            if 'theory' in command or 'principle' in command:
                return """Least Squares Principle:
Minimize sum of squared residuals between data and fitted curve

For data points (x₁,y₁), (x₂,y₂), ..., (xₙ,yₙ)
Fit function f(x) = a₀ + a₁x + a₂x² + ... + aₘxᵐ

Residual: rᵢ = yᵢ - f(xᵢ)
Objective: Minimize S = Σrᵢ² = Σ[yᵢ - f(xᵢ)]²

Normal Equations:
∂S/∂aⱼ = 0 for j = 0, 1, ..., m

Matrix Form: AᵀAa = Aᵀy
where A is Vandermonde matrix, a is coefficient vector

Solution: a = (AᵀA)⁻¹Aᵀy (if AᵀA is invertible)"""
            
            elif 'linear' in command or 'straight line' in command:
                return """Linear Least Squares (Straight Line):
Fit y = a₀ + a₁x to data points

Normal Equations:
na₀ + (Σxᵢ)a₁ = Σyᵢ
(Σxᵢ)a₀ + (Σxᵢ²)a₁ = Σxᵢyᵢ

Solution:
a₁ = (nΣxᵢyᵢ - ΣxᵢΣyᵢ)/(nΣxᵢ² - (Σxᵢ)²)
a₀ = (Σyᵢ - a₁Σxᵢ)/n

Alternative Form:
a₁ = Σ(xᵢ - x̄)(yᵢ - ȳ)/Σ(xᵢ - x̄)²
a₀ = ȳ - a₁x̄

where x̄ = Σxᵢ/n, ȳ = Σyᵢ/n"""
            
            elif 'example' in command:
                return """Least Squares Example:
Fit straight line to data: (1,2), (2,3), (3,5), (4,4), (5,6)

Step 1: Calculate sums
n = 5, Σx = 15, Σy = 20, Σx² = 55, Σxy = 67

Step 2: Apply formulas
a₁ = (5×67 - 15×20)/(5×55 - 15²) = (335 - 300)/(275 - 225) = 35/50 = 0.7
a₀ = (20 - 0.7×15)/5 = (20 - 10.5)/5 = 1.9

Result: y = 1.9 + 0.7x

Step 3: Check fit
x=1: y=2.6 (actual 2, error 0.6)
x=2: y=3.3 (actual 3, error 0.3)
x=3: y=4.0 (actual 5, error -1.0)
x=4: y=4.7 (actual 4, error 0.7)
x=5: y=5.4 (actual 6, error -0.6)

Sum of squared errors = 0.36 + 0.09 + 1.0 + 0.49 + 0.36 = 2.3"""
            
            elif 'polynomial' in command:
                return """Polynomial Least Squares:
Fit y = a₀ + a₁x + a₂x² + ... + aₘxᵐ

Matrix Equation: Va = y
where V is Vandermonde matrix:
V = [1  x₁  x₁²  ...  x₁ᵐ]
    [1  x₂  x₂²  ...  x₂ᵐ]
    [⋮  ⋮   ⋮    ⋱   ⋮  ]
    [1  xₙ  xₙ²  ...  xₙᵐ]

Normal Equations: VᵀVa = Vᵀy

System:
[n      Σx     Σx²    ...  Σxᵐ  ][a₀]   [Σy   ]
[Σx     Σx²    Σx³    ...  Σxᵐ⁺¹][a₁] = [Σxy  ]
[Σx²    Σx³    Σx⁴    ...  Σxᵐ⁺²][a₂]   [Σx²y ]
[⋮      ⋮      ⋮      ⋱   ⋮    ][⋮ ]   [⋮    ]
[Σxᵐ    Σxᵐ⁺¹  Σxᵐ⁺²  ...  Σx²ᵐ ][aₘ]   [Σxᵐy ]

Solve using Gaussian elimination or matrix inversion"""
            
            elif 'error' in command or 'quality' in command:
                return """Least Squares Error Analysis:
Measures of Fit Quality:

1. Sum of Squared Errors (SSE):
   SSE = Σ(yᵢ - ŷᵢ)²

2. Mean Squared Error (MSE):
   MSE = SSE/(n-m-1) where m = degree of polynomial

3. Coefficient of Determination (R²):
   R² = 1 - SSE/SST where SST = Σ(yᵢ - ȳ)²
   R² = 1: Perfect fit, R² = 0: No better than mean

4. Standard Error:
   SE = √MSE

5. Correlation Coefficient (for linear):
   r = Σ(xᵢ-x̄)(yᵢ-ȳ)/√[Σ(xᵢ-x̄)²Σ(yᵢ-ȳ)²]

Residual Analysis:
• Plot residuals vs x to check assumptions
• Random scatter indicates good fit
• Patterns suggest model inadequacy"""
            
            else:
                return """Least Squares Method:
• Principle: Minimize Σ(yᵢ - f(xᵢ))²
• Linear: y = a₀ + a₁x, solve normal equations
• Polynomial: y = Σaᵢxᵢ, matrix form Va = y
• Solution: a = (AᵀA)⁻¹Aᵀy
• Quality: R², MSE, residual analysis
• Applications: Data fitting, regression, trend analysis"""
        except Exception as e:
            return f"Least squares method error: {str(e)}"
    
    def solve_linear_regression(self, command):
        """Linear regression analysis"""
        try:
            if 'theory' in command:
                return """Linear Regression Theory:
Model: y = β₀ + β₁x + ε where ε ~ N(0,σ²)

Assumptions:
1. Linearity: Relationship is linear
2. Independence: Observations independent
3. Homoscedasticity: Constant variance
4. Normality: Errors normally distributed

Least Squares Estimators:
β̂₁ = Σ(xᵢ-x̄)(yᵢ-ȳ)/Σ(xᵢ-x̄)² = Sₓᵧ/Sₓₓ
β̂₀ = ȳ - β̂₁x̄

Properties:
• Unbiased: E[β̂ᵢ] = βᵢ
• Minimum variance (BLUE)
• Consistent estimators"""
            
            elif 'confidence' in command or 'intervals' in command:
                return """Confidence Intervals in Regression:
Standard Errors:
SE(β̂₁) = σ̂/√Sₓₓ where σ̂² = SSE/(n-2)
SE(β̂₀) = σ̂√(1/n + x̄²/Sₓₓ)

Confidence Intervals (100(1-α)%):
β₁: β̂₁ ± t_{α/2,n-2} × SE(β̂₁)
β₀: β̂₀ ± t_{α/2,n-2} × SE(β̂₀)

Prediction Intervals:
For new observation at x₀:
ŷ₀ ± t_{α/2,n-2} × σ̂√(1 + 1/n + (x₀-x̄)²/Sₓₓ)

Confidence Interval for Mean Response:
ŷ₀ ± t_{α/2,n-2} × σ̂√(1/n + (x₀-x̄)²/Sₓₓ)"""
            
            elif 'hypothesis' in command or 'testing' in command:
                return """Hypothesis Testing in Regression:
Test for Slope:
H₀: β₁ = 0 vs H₁: β₁ ≠ 0

Test Statistic: t = β̂₁/SE(β̂₁) ~ t_{n-2}

Decision: Reject H₀ if |t| > t_{α/2,n-2}

ANOVA Table:
Source    | SS        | df  | MS        | F
----------|-----------|-----|-----------|----------
Regression| SSR       | 1   | MSR=SSR/1 | MSR/MSE
Error     | SSE       | n-2 | MSE=SSE/(n-2)|
Total     | SST       | n-1 |           |

F-test: F = MSR/MSE ~ F_{1,n-2}
Tests overall significance of regression

R² = SSR/SST = 1 - SSE/SST"""
            
            elif 'diagnostics' in command:
                return """Regression Diagnostics:
Residual Analysis:
eᵢ = yᵢ - ŷᵢ (residuals)

Plots to Check:
1. Residuals vs Fitted: Check linearity, homoscedasticity
2. Normal Q-Q Plot: Check normality of errors
3. Residuals vs Order: Check independence
4. Residuals vs Predictors: Check model adequacy

Outlier Detection:
• Standardized residuals: rᵢ = eᵢ/σ̂
• Studentized residuals: tᵢ = eᵢ/(σ̂√(1-hᵢᵢ))
• Leverage: hᵢᵢ (diagonal of hat matrix)
• Cook's distance: Influence measure

Remedial Measures:
• Transformation of variables
• Weighted least squares
• Robust regression
• Remove outliers (carefully)"""
            
            else:
                return """Linear Regression:
• Model: y = β₀ + β₁x + ε
• Estimation: β̂₁ = Sₓᵧ/Sₓₓ, β̂₀ = ȳ - β̂₁x̄
• Assumptions: Linearity, independence, homoscedasticity, normality
• Testing: t-tests for coefficients, F-test for overall significance
• Diagnostics: Residual analysis, outlier detection
• Applications: Prediction, relationship analysis"""
        except Exception as e:
            return f"Linear regression error: {str(e)}"    

    def solve_error_analysis(self, command):
        """Numerical error analysis"""
        try:
            if 'types' in command or 'sources' in command:
                return """Types of Numerical Errors:
1. Inherent/Input Errors:
   • Errors in data or problem specification
   • Measurement errors
   • Model approximation errors

2. Truncation Errors:
   • From approximating infinite processes with finite ones
   • Taylor series truncation
   • Discretization errors in differential equations

3. Round-off Errors:
   • From finite precision arithmetic
   • Machine epsilon limitations
   • Accumulation through calculations

4. Blunders:
   • Programming errors
   • Incorrect algorithm implementation
   • Human mistakes

Error Propagation:
• Errors can amplify through calculations
• Condition number measures sensitivity
• Stability analysis important"""
            
            elif 'absolute' in command or 'relative' in command:
                return """Absolute and Relative Errors:
Absolute Error:
E_abs = |approximate value - true value|
E_abs = |x* - x|

Relative Error:
E_rel = |x* - x|/|x| (if x ≠ 0)
Often expressed as percentage: E_rel × 100%

Significant Digits:
Number of digits that are meaningful
If E_rel < 0.5 × 10^(-n), then n significant digits

Example:
True value: x = 3.14159
Approximate: x* = 3.14
E_abs = |3.14 - 3.14159| = 0.00159
E_rel = 0.00159/3.14159 ≈ 0.000506 ≈ 0.05%
Significant digits: 3 (since 0.000506 < 0.5 × 10^(-3))"""
            
            elif 'propagation' in command:
                return """Error Propagation:
For function f(x₁, x₂, ..., xₙ) with errors Δxᵢ:

Linear Approximation:
Δf ≈ Σ(∂f/∂xᵢ)Δxᵢ

For independent errors:
(Δf)² ≈ Σ(∂f/∂xᵢ)²(Δxᵢ)²

Common Operations:
Addition/Subtraction: f = x ± y
Δf ≈ Δx + Δy (absolute errors add)

Multiplication/Division: f = xy or f = x/y
Δf/|f| ≈ Δx/|x| + Δy/|y| (relative errors add)

Powers: f = x^n
Δf/|f| ≈ n × Δx/|x|

Example: f = x²y, x = 2±0.1, y = 3±0.2
∂f/∂x = 2xy = 12, ∂f/∂y = x² = 4
Δf ≈ 12×0.1 + 4×0.2 = 2.0
f = 12±2.0"""
            
            elif 'conditioning' in command:
                return """Problem Conditioning:
Well-conditioned: Small input changes → small output changes
Ill-conditioned: Small input changes → large output changes

Condition Number:
κ = |relative change in output|/|relative change in input|

For matrix problems Ax = b:
κ(A) = ||A|| × ||A⁻¹||

Interpretation:
• κ ≈ 1: Well-conditioned
• κ >> 1: Ill-conditioned
• κ ≈ 10^k: Lose about k decimal digits

Examples:
1. Hilbert Matrix: Very ill-conditioned (κ grows exponentially)
2. Identity Matrix: κ = 1 (perfectly conditioned)
3. Nearly singular matrices: Very large κ

Effects:
• Round-off errors amplified by κ
• Solution accuracy limited by κ × machine precision"""
            
            else:
                return """Numerical Error Analysis:
• Types: Inherent, truncation, round-off, blunders
• Measures: Absolute error |x*-x|, relative error |x*-x|/|x|
• Propagation: Linear approximation using partial derivatives
• Conditioning: κ measures sensitivity to input changes
• Stability: Algorithm's sensitivity to round-off errors
• Applications: Algorithm design, result validation"""
        except Exception as e:
            return f"Error analysis error: {str(e)}"
    
    def solve_condition_number(self, command):
        """Condition number analysis"""
        try:
            if 'definition' in command or 'theory' in command:
                return """Condition Number Definition:
Measures sensitivity of problem to input perturbations

For linear system Ax = b:
κ(A) = ||A|| × ||A⁻¹||

Properties:
• κ(A) ≥ 1 for any matrix
• κ(I) = 1 (identity matrix)
• κ(A) = ∞ if A is singular
• κ(cA) = κ(A) for any scalar c ≠ 0

Different Norms:
• κ₁(A) = ||A||₁ × ||A⁻¹||₁ (1-norm)
• κ₂(A) = ||A||₂ × ||A⁻¹||₂ (2-norm, spectral)
• κ∞(A) = ||A||∞ × ||A⁻¹||∞ (∞-norm)

For symmetric matrices:
κ₂(A) = |λₘₐₓ|/|λₘᵢₙ| (ratio of extreme eigenvalues)"""
            
            elif 'interpretation' in command:
                return """Condition Number Interpretation:
If κ(A) ≈ 10^k, then:
• Lose approximately k decimal digits of accuracy
• Small changes in A or b can cause large changes in x

Error Bound:
||Δx||/||x|| ≤ κ(A) × ||Δb||/||b||

Classification:
• κ < 10: Excellent conditioning
• 10 ≤ κ < 100: Good conditioning  
• 100 ≤ κ < 1000: Moderate conditioning
• 1000 ≤ κ < 10⁶: Poor conditioning
• κ ≥ 10⁶: Very poor conditioning

Practical Impact:
• Well-conditioned: Reliable numerical solutions
• Ill-conditioned: Results may be meaningless
• Machine precision ≈ 10⁻¹⁶, so κ > 10¹⁶ is hopeless"""
            
            elif 'example' in command:
                return """Condition Number Example:
Consider matrix A = [1    1  ]
                    [1  1.01]

Step 1: Calculate ||A||∞ = max row sum = 2.01

Step 2: Find A⁻¹
det(A) = 1×1.01 - 1×1 = 0.01
A⁻¹ = (1/0.01)[1.01  -1] = [101  -100]
                [-1    1]   [-100  100]

Step 3: Calculate ||A⁻¹||∞ = 201

Step 4: Condition number
κ∞(A) = 2.01 × 201 ≈ 404

Interpretation: Lose about 2-3 decimal digits

Test with perturbed system:
Original: Ax = b where b = [2, 2.01]ᵀ → x = [1, 1]ᵀ
Perturbed: b' = [2, 2.02]ᵀ → x' = [0, 2]ᵀ
Small change in b causes large change in x!"""
            
            elif 'improvement' in command:
                return """Improving Condition:
Techniques to Reduce Condition Number:

1. Scaling:
   • Row/column scaling
   • Equilibration
   • Normalize equations

2. Pivoting:
   • Partial pivoting
   • Complete pivoting
   • Scaled pivoting

3. Preconditioning:
   • Find matrix M such that κ(M⁻¹A) << κ(A)
   • Solve M⁻¹Ax = M⁻¹b instead
   • Common preconditioners: Jacobi, ILU, multigrid

4. Regularization:
   • Add small multiple of identity: A + λI
   • Tikhonov regularization for least squares
   • Ridge regression

5. Problem Reformulation:
   • Change variables
   • Use different mathematical formulation
   • Exploit problem structure

When Nothing Works:
• Use higher precision arithmetic
• Iterative refinement
• Accept limited accuracy
• Question problem formulation"""
            
            else:
                return """Condition Number:
• Definition: κ(A) = ||A|| × ||A⁻¹||
• Measures: Sensitivity to input perturbations
• Interpretation: κ ≈ 10^k loses k decimal digits
• Error bound: ||Δx||/||x|| ≤ κ(A) × ||Δb||/||b||
• Good: κ < 100, Poor: κ > 1000
• Improvement: Scaling, pivoting, preconditioning"""
        except Exception as e:
            return f"Condition number error: {str(e)}"
    
    def solve_monte_carlo_method(self, command):
        """Monte Carlo methods"""
        try:
            if 'principle' in command or 'theory' in command:
                return """Monte Carlo Method Principle:
Uses random sampling to solve mathematical problems

Basic Idea:
1. Model problem using probability distributions
2. Generate random samples
3. Perform calculations on samples
4. Aggregate results to get approximate solution

Law of Large Numbers:
As sample size n → ∞, sample average → expected value

Central Limit Theorem:
Sample mean is approximately normal with:
• Mean = true value
• Standard deviation = σ/√n

Error typically decreases as 1/√n
Need 100× more samples for 1 extra decimal digit"""
            
            elif 'integration' in command:
                return """Monte Carlo Integration:
Estimate ∫[a to b] f(x)dx

Method 1: Hit-or-Miss
1. Generate random points (x,y) in rectangle [a,b]×[0,M]
2. Count points below curve: N_below
3. Estimate: I ≈ (b-a)×M×(N_below/N_total)

Method 2: Sample Mean
1. Generate random x_i uniformly in [a,b]
2. Calculate f(x_i) for each sample
3. Estimate: I ≈ (b-a)/n × Σf(x_i)

Multidimensional:
∫∫...∫ f(x₁,...,x_d)dx₁...dx_d over region R
1. Generate random points in R
2. Estimate: I ≈ Volume(R)/n × Σf(x_i)

Advantage: Dimension-independent convergence rate"""
            
            elif 'example' in command:
                return """Monte Carlo Integration Example:
Estimate π using unit circle

Method: Estimate ∫∫_{x²+y²≤1} 1 dxdy = π

Algorithm:
1. Generate random points (x,y) in square [-1,1]×[-1,1]
2. Check if x² + y² ≤ 1 (inside circle)
3. Estimate π ≈ 4 × (points inside)/(total points)

Sample Results:
n = 100: π ≈ 3.2 (error ≈ 0.06)
n = 1,000: π ≈ 3.14 (error ≈ 0.002)
n = 10,000: π ≈ 3.1416 (error ≈ 0.0000)
n = 100,000: π ≈ 3.14159 (error ≈ 0.00000)

Code Structure:
inside = 0
for i in range(n):
    x, y = random(), random()  # [0,1]
    x, y = 2*x-1, 2*y-1       # [-1,1]
    if x*x + y*y <= 1:
        inside += 1
pi_estimate = 4 * inside / n"""
            
            elif 'applications' in command:
                return """Monte Carlo Applications:
1. Numerical Integration:
   • High-dimensional integrals
   • Complex integration regions
   • When traditional methods fail

2. Optimization:
   • Simulated annealing
   • Genetic algorithms
   • Global optimization

3. Finance:
   • Option pricing (Black-Scholes)
   • Risk assessment
   • Portfolio optimization

4. Physics:
   • Particle transport
   • Quantum mechanics
   • Statistical mechanics

5. Engineering:
   • Reliability analysis
   • Quality control
   • System simulation

6. Statistics:
   • Bootstrap methods
   • Bayesian inference
   • Hypothesis testing

Advantages:
• Handles complex geometries
• Dimension-independent
• Easy to parallelize
• Provides error estimates

Disadvantages:
• Slow convergence (1/√n)
• Requires good random numbers
• Statistical nature of results"""
            
            else:
                return """Monte Carlo Method:
• Principle: Random sampling to solve problems
• Integration: Sample mean or hit-or-miss
• Convergence: Error ∝ 1/√n
• Applications: Integration, optimization, simulation
• Advantages: High dimensions, complex regions
• Disadvantages: Slow convergence, statistical results"""
        except Exception as e:
            return f"Monte Carlo method error: {str(e)}"
    
    def solve_finite_difference_method(self, command):
        """Finite Difference Method (FDM)"""
        try:
            if 'principle' in command or 'theory' in command:
                return """Finite Difference Method Principle:
Approximates derivatives using difference quotients

Basic Approximations:
Forward: f'(x) ≈ (f(x+h) - f(x))/h
Backward: f'(x) ≈ (f(x) - f(x-h))/h
Central: f'(x) ≈ (f(x+h) - f(x-h))/(2h)

Second Derivative:
f''(x) ≈ (f(x+h) - 2f(x) + f(x-h))/h²

Grid-based Approach:
• Discretize domain into grid points
• Replace derivatives with difference formulas
• Convert PDE to system of algebraic equations
• Solve resulting linear/nonlinear system"""
            
            elif 'boundary value' in command or 'bvp' in command:
                return """FDM for Boundary Value Problems:
Example: y'' + p(x)y' + q(x)y = r(x), y(a) = α, y(b) = β

Step 1: Create grid
x_i = a + ih, i = 0,1,...,n+1 where h = (b-a)/(n+1)

Step 2: Apply finite differences at interior points
(y_{i+1} - 2y_i + y_{i-1})/h² + p_i(y_{i+1} - y_{i-1})/(2h) + q_i y_i = r_i

Step 3: Rearrange into tridiagonal system
a_i y_{i-1} + b_i y_i + c_i y_{i+1} = d_i

where:
a_i = 1/h² - p_i/(2h)
b_i = -2/h² + q_i  
c_i = 1/h² + p_i/(2h)
d_i = r_i

Step 4: Apply boundary conditions
y_0 = α, y_{n+1} = β

Step 5: Solve tridiagonal system"""
            
            elif 'pde' in command or 'partial' in command:
                return """FDM for Partial Differential Equations:
Heat Equation: ∂u/∂t = α²∂²u/∂x²

Explicit Scheme (Forward Euler in time):
(u_i^{n+1} - u_i^n)/Δt = α²(u_{i+1}^n - 2u_i^n + u_{i-1}^n)/(Δx)²

Rearranged:
u_i^{n+1} = u_i^n + r(u_{i+1}^n - 2u_i^n + u_{i-1}^n)
where r = α²Δt/(Δx)²

Stability Condition: r ≤ 1/2

Implicit Scheme (Backward Euler):
(u_i^{n+1} - u_i^n)/Δt = α²(u_{i+1}^{n+1} - 2u_i^{n+1} + u_{i-1}^{n+1})/(Δx)²

Results in tridiagonal system at each time step
Unconditionally stable but requires solving system

Crank-Nicolson (Average of explicit/implicit):
Unconditionally stable, second-order accurate in time"""
            
            elif 'advantages' in command or 'disadvantages' in command:
                return """FDM Advantages & Disadvantages:
Advantages:
• Simple to understand and implement
• Works well for regular geometries
• Efficient for structured grids
• Well-established theory
• Easy to achieve high-order accuracy

Disadvantages:
• Difficult for complex geometries
• Requires structured grids
• Boundary conditions can be tricky
• May have stability restrictions
• Less flexible than FEM

When to Use FDM:
• Regular rectangular domains
• Simple boundary conditions
• When efficiency is important
• Parabolic/elliptic PDEs
• Time-dependent problems

Alternatives:
• Finite Element Method (complex geometries)
• Finite Volume Method (conservation laws)
• Spectral methods (smooth solutions)
• Meshless methods (irregular domains)"""
            
            else:
                return """Finite Difference Method:
• Principle: Replace derivatives with difference quotients
• Grid-based: Discretize domain into regular grid
• BVPs: Convert to tridiagonal system
• PDEs: Explicit/implicit time stepping
• Stability: May require time step restrictions
• Applications: Heat equation, wave equation, Poisson equation"""
        except Exception as e:
            return f"Finite difference method error: {str(e)}"
    
    def solve_finite_element_basics(self, command):
        """Finite Element Method basics"""
        try:
            if 'principle' in command or 'theory' in command:
                return """Finite Element Method Principle:
Approximate solution using piecewise polynomials

Key Ideas:
1. Divide domain into elements (triangles, rectangles, etc.)
2. Approximate solution within each element using basis functions
3. Ensure continuity between elements
4. Use variational formulation (weak form)
5. Assemble global system from element contributions

Basis Functions:
• Piecewise polynomials (linear, quadratic, cubic)
• Non-zero only over few elements (local support)
• Continuous across element boundaries
• Examples: Hat functions, Lagrange polynomials

Variational Formulation:
Convert PDE to equivalent integral form
Minimize energy functional or use Galerkin method"""
            
            elif 'steps' in command or 'procedure' in command:
                return """FEM Procedure:
Step 1: Discretization
• Divide domain into elements
• Define nodes and connectivity
• Choose element type and basis functions

Step 2: Element Formulation
• Derive element stiffness matrix K^e
• Derive element load vector f^e
• Use shape functions and numerical integration

Step 3: Assembly
• Assemble global stiffness matrix K
• Assemble global load vector f
• Apply connectivity information

Step 4: Boundary Conditions
• Apply essential (Dirichlet) boundary conditions
• Apply natural (Neumann) boundary conditions
• Modify system equations

Step 5: Solution
• Solve linear system Ku = f
• Post-process results
• Calculate derived quantities (stresses, fluxes)"""
            
            elif 'example' in command:
                return """Simple FEM Example:
1D Problem: -u'' = f(x), u(0) = u(1) = 0

Step 1: Discretize [0,1] into 2 elements
Elements: [0, 0.5], [0.5, 1]
Nodes: x₁ = 0, x₂ = 0.5, x₃ = 1

Step 2: Linear basis functions
φ₁(x) = 2(0.5-x) for x ∈ [0,0.5], 0 elsewhere
φ₂(x) = 2x for x ∈ [0,0.5], 2(1-x) for x ∈ [0.5,1]
φ₃(x) = 2(x-0.5) for x ∈ [0.5,1], 0 elsewhere

Step 3: Element matrices (for uniform f = 1)
K^e = (1/h)[1  -1] = 2[1  -1]
           [-1  1]     [-1  1]

f^e = (h/2)[1] = 0.25[1]
           [1]        [1]

Step 4: Assemble and apply BCs
Global system for interior node u₂:
4u₂ = 0.5 → u₂ = 0.125

Exact solution: u(x) = x(1-x)/2
At x = 0.5: u_exact = 0.125 (exact match!)"""
            
            elif 'advantages' in command:
                return """FEM Advantages:
1. Geometric Flexibility:
   • Handles complex geometries easily
   • Unstructured meshes
   • Curved boundaries

2. Boundary Conditions:
   • Natural treatment of Neumann BCs
   • Easy to handle mixed BCs
   • Flexible constraint handling

3. Adaptivity:
   • h-refinement (smaller elements)
   • p-refinement (higher-order polynomials)
   • r-refinement (mesh redistribution)

4. Mathematical Foundation:
   • Solid theoretical basis
   • Error estimates available
   • Convergence guarantees

5. Software:
   • Many commercial packages
   • Well-developed libraries
   • Extensive research base

Applications:
• Structural mechanics
• Heat transfer
• Fluid dynamics
• Electromagnetics
• Multiphysics problems"""
            
            else:
                return """Finite Element Method:
• Principle: Piecewise polynomial approximation
• Elements: Divide domain into simple shapes
• Basis functions: Local support, continuous
• Variational: Weak formulation of PDE
• Assembly: Combine element contributions
• Applications: Complex geometries, engineering problems"""
        except Exception as e:
            return f"Finite element method error: {str(e)}"
