# LyricForge

LyricForge takes a song and a theme, rewrites the lyrics around that theme, sings them with an AI voice, and mixes the new vocals back over the original instrumental.

Built in a team at HackNotts 2025 (University of Nottingham).

## What it does

- Splits a song into a vocal track and an instrumental track.
- Transcribes the vocals with word-level timestamps.
- Rewrites the lyrics around any theme you give it, such as "space exploration", while trying to keep the same syllable count and rhythm per line.
- Picks an AI voice that sounds like a chosen artist and sings the new lyrics.
- Mixes the new vocals over the original instrumental and saves a WAV file.

## How it's built

It is a Python command-line tool. `pipeline.py` runs five agents in order, each a small class with one job:

1. Pre-process: Spleeter's 2-stem model separates vocals from the accompaniment, then Whisper (`base` model) transcribes the vocals with `word_timestamps=True`.
2. Lyric generation: Google Gemini rewrites the transcript to the theme. The prompt asks it to keep the structure, rhythm and syllables per line.
3. Voice synthesis: Gemini writes a short description of the target artist's voice, then picks the closest match from the ElevenLabs voice library. ElevenLabs (`eleven_multilingual_v2`) generates the vocal track.
4. Alignment: a placeholder for now. The plan was to time the new vocals to the original using dynamic time warping (`fastdtw`).
5. Mixing: pydub trims both tracks to the same length and overlays them.

Each stage falls back instead of crashing where it can. With no Gemini key the original lyrics are kept, and with no ElevenLabs key the original vocals are used.

## My role

I built this with Joshua Bakata. I set up the repository, the five-stage pipeline structure and the stem separation stage. Joshua built the working ElevenLabs voice synthesis agent and its test scripts.

## Run it locally

You need Python 3.8+, FFmpeg on your `PATH`, a Google Gemini API key and an ElevenLabs API key.

```bash
git clone https://github.com/Maseeek/LyricForge.git
cd LyricForge
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt elevenlabs
cp .env.example .env   # then add GEMINI_API_KEY and ELEVENLABS_API_KEY
```

Run it on a song:

```bash
python pipeline.py --song input.mp3 --theme "space exploration" --artist "Drake"
```

`--artist` is optional. You can also pass `--gemini-api-key` and `--elevenlabs-api-key` instead of using `.env`.

Output goes to `output/`: the separated stems in `output/separated/<song>/`, the generated vocals in `output/new_vocals.mp3`, and the final mix in `output/<song>_themed_<theme>.wav`.

## Status

Hackathon prototype. The alignment stage is not implemented yet, so the new vocals are not timed to the beat. MIT License.
