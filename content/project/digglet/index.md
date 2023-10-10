---
title: "Digglet"
date: 2022-06-01
showReadingTime: false
# Description will be shown in the link.
description: "Real-time audio streaming server that support speaker verification and sound level detection."
summary: "Real-time audio streaming server that support speaker verification and sound level detection."
showHero: false
---

<!-- What is the project about? -->

[Link to project](https://github.com/8igMac/diglett)

Digglet is a backend project that streams
data (in this case audio signal) from
the frontend to the backend and do some kind of processing (verifying
who is speaking) and send the processed information back to the frontend.

## Tech stack
The following is the tech stack I used:

- Websocket to handle the data streaming.
- FastAPI as a Python webframework to build the backend application.
- SpeechBrain for audio embedding to get the speaker audio features using
  machine learning.
- Pyaudio for audio recording
- pytorch to reshape tensor and measure consine similarity
- numpy to calculate mean energy of audio segment.
- dotenv for handling sensitive information.
- Python asyncio for transmitting information to the backend asyncrnously.

Ops:

- docker for application containerization
- Github action for continuous integration

Dev dependencies:

- pytest for python test framwork
- black for python formatting


## System Architecture
- Limitation: only 2 speakers are allowed.
- Configuration stage:
  - open websocket for configuration
  - 1 first speaker register audio feature by speaking for 5 secs
  - 2 first speaker register audio feature by speaking for 5 secs
  - Server finished audio feature extraction for both speaker stored
    the features in the backend.
- Running stage:
  - open websocket for audio streaming
  - the audio are semgneted and send through websocket to the backend
    server.
  - web server encode the audio segment using speechbrain classifier
    as an embedding.
  - the server compare the embedding of the audio received against
    the 2 previous stored embedding. If the consine similarity of the
    2 embedding are below a certaint threshold (i.e., they are similary)
    we reconize it as one of the people speaking the send back the 
    information includeing the meaning energy level of the segment. 
    If there were no similar embedding found, report "noise".