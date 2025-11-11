Multilingual AI Voicepod Creator Studio
Create professionally enhanced, emotion-rich voice episodes using state-of-the-art AI for denoising, transcription, emotion analysis, and music/SFX mixing.

Table of Contents
Project Overview
Features
AI Models & Libraries Used
Installation
Configuration
Usage
Folder Structure
Code Structure

Project Overview
This project processes uploaded or recorded voice/audio, automatically cleans noise, transcribes speech (multilingual), detects the emotion/tone, and overlays matching background music and SFX. Final audio is downloadable, ready for sharing or podcast publication.

Features
Upload audio/video files or record live
Denoising and audio normalization (Cleanvoice AI)
Multilingual speech recognition & transcription (OpenAI Whisper)
Emotion & tone detection (OpenAI GPT-4o)
Automatic background music and SFX overlays based on emotion/keywords
Optional intro/outro music
Download ready-for-publishing enhanced audio with transcript

AI Models & Libraries Used

Model/Library
Role
Cleanvoice AI
Professional audio cleaning & enhancement
OpenAI Whisper
Multilingual speech-to-text transcription
OpenAI GPT-4o
Emotion/tone classification from transcript
pydub, numpy
Audio mixing, SFX/music overlay
streamlit
Interactive UI platform




Installation
Installation
Clone this repository:

git clone <repo-url>
cd <repo-folder>

Create and activate a virtual environment:

python -m venv venv
source venv/bin/activate    # On Windows: venv\Scripts\activate

Install required packages:

pip install -r requirements.txt

Configuration
Add your API keys to .env file in the project root:

CLEANVOICE_API_KEY=your_cleanvoice_api_key
OPENAI_API_KEY=your_openai_api_key

Place music and SFX files in the /music_beds/ folders as referenced in app.py.

Usage
Start the Streamlit app:

streamlit run app.py

Use the web UI to upload, process, and download enhanced audio.

Folder Structure
text
project-root/
├── app.py
├── requirements.txt
├── .env
├── Input_audio/
├── music_beds/
│   ├── background_music/
│   └── SFX/
└── utils/
    ├── music_mixer.py
    ├── cue_detection.py
    └── intro_outro.py


Code Structure
app.py: Main application logic and UI flow
utils/: Modular mixer and sound logic
music_beds/: Audio assets used for overlay and SFX





—-----------------------

Here’s a clear explanation about how keywords are detected in the transcript and how SFX (sound effects) are placed at the correct timestamps:

Keyword Detection and SFX Placement
How keyword detection works
After the audio is transcribed using OpenAI Whisper, the transcript (plain text) is scanned for certain predefined keywords.
The code uses a keyword_sfx_map dictionary, where each keyword (like "rain", "thunder", "train", etc.) maps to a specific SFX (MP3/WAV) file.
For every keyword found in the transcript, the code records its position (the character index of the keyword within the lowercased transcript text).
How SFX placement timestamp is calculated
The position of the keyword in the transcript (as a character index) is mapped proportionally into the duration of the audio.
This is done with the formula:

timestamp (ms) = (index of keyword in transcript / length of transcript) * total audio length (ms)

This means if a keyword occurs halfway through the transcript, its SFX will be placed at about half the duration of the audio.
Placing SFX in the audio
For each detected keyword, the assigned SFX file is mixed (overlaid) onto the processed audio at its calculated timestamp.
The overlay uses pydub's overlay method and adjusts volume so it sits well under the spoken voice.
In code, this workflow is handled by the keyword_positions function and a loop that overlays the SFX at each computed position.
Example
If the transcript is "Today it started to rain, and thunder was heard.", and the total audio length is 30 seconds:
"rain" is detected at about position 21/44 characters ≈ 47% of the way into the audio = about 14 seconds.
"thunder" is at about 39/44 ≈ 89% of the way in = about 26.7 seconds.
The "rain" SFX is placed at 14s, and "thunder" at 26.7s.

In your code
This snippet illustrates how SFX placement is calculated:
python
def keyword_positions(text, keywords, audio_len_ms):
    positions = {}
    text_lower = text.lower()
    for keyword in keywords:
        idx = text_lower.find(keyword)
        if idx != -1:
            positions[keyword] = int((idx / len(text_lower)) * audio_len_ms)
    return positions

And then:
python
positions = keyword_positions(transcript, keyword_sfx_map.keys(), len(audio))
for k, pos in positions.items():
    sfx = AudioSegment.from_file(keyword_sfx_map[k]) - 12  # volume down
    audio = audio.overlay(sfx, position=pos)


Summary:
Keyword detection is based on matching predefined words in the transcript. SFX are placed in the audio at the corresponding relative position, simulating the effect as if the sound happened alongside the spoken word.

