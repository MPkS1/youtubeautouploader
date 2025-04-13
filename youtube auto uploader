# 📁 youtube_shorts_uploader.py (FULL DESKTOP APP)

import os
import pickle
import time
import smtplib
from email.mime.text import MIMEText
from datetime import datetime
from tkinter import filedialog
import customtkinter as ctk
from tkcalendar import DateEntry
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options

# ---- Configuration ----
SCOPES = ["https://www.googleapis.com/auth/youtube.upload"]
EMAIL = "yougmail account"# Replace with your email
EMAIL_PASSWORD = "your gmail account password"  # Enable App Password in Gmail

# ---- Functions ----
def send_email_alert(video_name):
    msg = MIMEText(f"Copyright content detected in video: {video_name}")
    msg['Subject'] = "⚠️ Copyright Alert"
    msg['From'] = EMAIL
    msg['To'] = EMAIL
    server = smtplib.SMTP('smtp.gmail.com', 587)
    server.starttls()
    server.login(EMAIL, EMAIL_PASSWORD)
    server.send_message(msg)
    server.quit()

def detect_copyright(video_path):
    chrome_options = Options()
    chrome_options.add_argument("--headless=new")
    driver = webdriver.Chrome(options=chrome_options)
    driver.get("https://studio.youtube.com/")
    time.sleep(15)  # Allow manual login if needed

    # You can add detection code here later
    driver.quit()
    return False  # Mock: No copyright for now

def upload_video(video_file, title, description, audience, schedule_datetime):
    if os.path.exists("token.pickle"):
        with open("token.pickle", "rb") as token:
            credentials = pickle.load(token)
    else:
        flow = InstalledAppFlow.from_client_secrets_file("client_secret.json", SCOPES)
        credentials = flow.run_local_server(port=0)
        with open("token.pickle", "wb") as token:
            pickle.dump(credentials, token)

    youtube = build("youtube", "v3", credentials=credentials)
    body = {
        "snippet": {
            "title": title,
            "description": description,
            "tags": ["shorts", "fun", "tech"],
            "categoryId": "22"
        },
        "status": {
            "privacyStatus": "private",
            "selfDeclaredMadeForKids": True if audience == "For Kids" else False,
            "publishAt": schedule_datetime.isoformat() + "Z"
        }
    }

    request = youtube.videos().insert(
        part="snippet,status",
        body=body,
        media_body=video_file
    )
    response = request.execute()
    return response["id"]

# ---- GUI ----
ctk.set_appearance_mode("dark")
ctk.set_default_color_theme("blue")
app = ctk.CTk()
app.geometry("600x700")
app.title("🔮 YouTube Shorts Uploader")

video_path = ""
def select_video():
    global video_path
    video_path = filedialog.askopenfilename(filetypes=[("Video files", "*.mp4")])
    video_label.configure(text=f"Selected: {os.path.basename(video_path)}")

title_entry = ctk.CTkEntry(app, placeholder_text="Enter video title")
desc_entry = ctk.CTkEntry(app, placeholder_text="Enter description")
audience_menu = ctk.CTkOptionMenu(app, values=["For Kids", "Not for Kids"])
audience_menu.set("For Kids")
date_picker = DateEntry(app)
time_entry = ctk.CTkEntry(app, placeholder_text="HH:MM")
video_label = ctk.CTkLabel(app, text="No video selected")
status_label = ctk.CTkLabel(app, text="")


def schedule_upload():
    if not video_path:
        status_label.configure(text="❗ Please select a video", text_color="red")
        return
    title = title_entry.get()
    desc = desc_entry.get()
    audience = audience_menu.get()
    date = date_picker.get_date()
    time_text = time_entry.get()
    try:
        hour, minute = map(int, time_text.split(":"))
        schedule_datetime = datetime(date.year, date.month, date.day, hour, minute)
    except:
        status_label.configure(text="❗ Invalid time format", text_color="red")
        return

    status_label.configure(text="⏳ Checking copyright...", text_color="yellow")
    app.update()

    if detect_copyright(video_path):
        send_email_alert(os.path.basename(video_path))
        status_label.configure(text="⚠️ Copyright detected. Upload cancelled.", text_color="red")
    else:
        status_label.configure(text="🚀 Uploading...", text_color="yellow")
        app.update()
        try:
            video_id = upload_video(video_path, title, desc, audience, schedule_datetime)
            status_label.configure(text=f"✅ Uploaded! Video ID: {video_id}", text_color="green")
        except Exception as e:
            status_label.configure(text=f"❌ Upload failed: {e}", text_color="red")

# ---- Layout ----
ctk.CTkLabel(app, text="🎥 YouTube Shorts Scheduler", font=("Arial", 22)).pack(pady=10)
ctk.CTkButton(app, text="Select Video", command=select_video).pack(pady=10)
video_label.pack()
title_entry.pack(pady=10)
desc_entry.pack(pady=10)
audience_menu.pack(pady=10)
date_picker.pack(pady=10)
time_entry.pack(pady=10)
ctk.CTkButton(app, text="Schedule Upload", command=schedule_upload).pack(pady=20)
status_label.pack(pady=10)

app.mainloop()
