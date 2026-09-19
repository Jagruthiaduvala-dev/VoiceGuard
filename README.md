#  VoiceGuard — Real-Time AI Voice Clone Detection for Scam Call Prevention

**Detect AI voice clones in real time — before you send money.**

## The Problem

India has seen a sharp rise in AI voice-cloning scams: fraudsters use a few seconds of a
person's voice (scraped from social media, a call, or a video) to clone it, then call
family members impersonating them in fake emergencies — accidents, kidnappings, bail
money, blocked bank accounts. Because the voice *sounds* like a loved one, these calls
bypass the skepticism people have learned for text/email scams. Elderly people, who are
less familiar with AI capabilities and more likely to act under panic, are
disproportionately targeted.

There is currently no easy, consumer-facing way for an ordinary person to check, in the
moment, whether a call they just received is a real human voice or an AI-generated clone.

**VoiceGuard closes that gap:** upload or record a suspicious clip, get an instant
verdict — Real or AI-Generated — with a confidence score and a plain-language
explanation, so you can pause before transferring money.

## How It Works

```
Audio Input → Preprocessing → Classifier → Explainable Output
```

1. **Preprocessing** — any uploaded audio (mp3, wav, m4a) is resampled to 16kHz mono,
   silence-trimmed, and normalized so the model receives clean, consistent input
   regardless of source quality.
2. **Classification** — a Wav2Vec2 XLS-R + AASIST hybrid model (ONNX) trained on
   multilingual real and AI-generated speech, with noise and telephony-artifact
   augmentation, predicts real vs. spoofed.
3. **Explainability layer** — the raw probability score is translated into a
   confidence tier (High / Moderate / Uncertain) and a plain-language explanation of
   what these models typically detect (pitch consistency, spectral texture,
   micro-pauses), so the result is useful to a non-technical user, not a black box.

## Tech Stack

- **Model:** [`ayush2635/Dhwani-Multilingual-Deepfake-Audio-Detection-Model`](https://huggingface.co/ayush2635/Dhwani-Multilingual-Deepfake-Audio-Detection-Model)
  (Wav2Vec2 XLS-R front-end + AASIST back-end, ONNX format)
- **Inference:** `onnxruntime`
- **Audio processing:** `librosa`, `soundfile`
- **Interface:** `gradio`
- **Environment:** Python 3, developed and run on Google Colab (T4 GPU)

## Setup / Running It

1. Open the notebook in Google Colab.
2. Run all cells top to bottom (`Runtime → Run all`). This installs dependencies,
   downloads the model from Hugging Face, and launches the Gradio app.
3. The final cell prints a public `.gradio.live` link — open it, upload or record a
   clip, click **Analyze**.

## Model Source & Citation

- Model: Dhwani Multilingual Deepfake Audio Detection Model, originally developed
  during the HCL Guvi Hackathon by ayush2635, hosted on Hugging Face.
- Training data: Mozilla Common Voice (v24.0, Indic languages) and
  `garystafford/deepfake-audio-detection` for real audio; synthetic TTS via
  `vdivyasharma/IndicSynth` and `garystafford/deepfake-audio-detection` for fake audio.
- License: MIT.

## Evaluation

We tested against a self-curated dataset of 10 short scam-style clips (5 real
recordings, 5 AI-generated clones) spanning both family-emergency scenarios
(accident, kidnapping, bail, lost phone, blocked card) and institutional-impersonation
scenarios (bank KYC, credit card fraud alert, EMI default, insurance renewal, tax
refund).

**Result: 9/10 correct (90% accuracy)** on this dataset. The one miss was a
cloned clip classified as real at only 75.6% confidence — a moderate-confidence
result, not a confident failure, meaning the system correctly signaled uncertainty
rather than being wrong with false conviction.

## Known Limitations (Honest Scope)

- **Domain sensitivity:** deepfake detectors trained on curated datasets can behave
  unpredictably on out-of-distribution audio (unusual background noise, very short
  clips, heavy compression). We selected this model specifically because it was
  trained with telephony/transmission-artifact augmentation. In initial testing, one
  real recording was misclassified with high confidence; re-recording it under
  cleaner conditions resolved the issue, suggesting audio quality (not the model
  itself) was the root cause — worth noting since it shows the tool is sensitive to
  recording conditions, a limitation worth flagging to end users.
- **Unseen TTS engines:** generative voice cloning evolves quickly; detection accuracy
  against entirely novel, state-of-the-art cloning systems not represented in training
  data may be lower than against known TTS engines.
- **Not a guarantee:** this is a decision-support tool, not proof. We explicitly
  recommend users verify unusual requests for money through a separate, trusted
  channel regardless of the tool's verdict.

## Roadmap

- **Phase 2:** Browser/phone-app extension that screens calls in real time, during
  the call itself, rather than only post-hoc file upload.
- **Phase 3:** Partnership integration — telecom carrier call-screening
  infrastructure, or bank fraud-alert systems (flagging high-risk transactions
  triggered immediately after a suspected clone call).
- **Phase 4:** Expanded multilingual support for regional Indian languages, since
  scam calls frequently target non-English speakers. (Our current model already
  supports English, Hindi, Tamil, Telugu, and Malayalam.)

## Privacy Note

Audio uploaded to the demo app is processed for classification only and is not
stored beyond the session.
