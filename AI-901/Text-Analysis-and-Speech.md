# Text Analysis & Speech — Study Guide

**Certification Code:** AI-901 | **Phase:** 2 — Implementing AI Solutions with Microsoft Foundry

> [!NOTE]
> This guide covers Modules 9.1–9.3: building lightweight applications with Azure AI Language (text analysis) and Azure AI Speech, and wiring the spoken-prompt pattern with a deployed multimodal model.

---

# 9.1 Text Analysis — Azure AI Language

## What It Is (Exam-Ready Definition)

Azure AI Language is a cloud-based service in Foundry Tools that provides natural language processing features for **analyzing existing text** — it does not generate new text. It returns structured insights: labels, scores, entities, and summaries.

**The four capabilities you must know:**

| Capability | What It Does | Output |
|:---|:---|:---|
| **Key Phrase Extraction** | Identifies the main concepts in unstructured text | List of key phrases |
| **Named Entity Recognition (NER)** | Identifies and categorizes people, orgs, locations, dates, etc. | Entity list with type + confidence |
| **Sentiment Analysis** | Evaluates whether text is positive, negative, or mixed. Can link sentiment to specific aspects. | Score (0–1) per sentence + overall |
| **Summarization** | Extractive: selects key sentences. Abstractive: generates concise new sentences. | Summary string |

> [!IMPORTANT]
> **Exam trap:** Text analysis ≠ information extraction. Text analysis works on **unstructured prose** (reviews, emails, articles). Information extraction works on **structured documents** (forms, invoices, PDFs with defined fields). If the scenario involves "reading fields from a form" → Content Understanding.

---

## Installation

```bash
pip install azure-ai-textanalytics azure-ai-projects azure-identity
```

---

## The Complete Lightweight Text Analysis Application

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

# --- Option A: Use the Language client directly with an endpoint + key ---
# This is the standalone pattern used when your Language resource
# is provisioned separately from a Foundry project.
language_endpoint = os.environ["AZURE_LANGUAGE_ENDPOINT"]
language_key = os.environ["AZURE_LANGUAGE_KEY"]

text_client = TextAnalyticsClient(
    endpoint=language_endpoint,
    credential=AzureKeyCredential(language_key),
)

# --- Option B: Get the Language client via AIProjectClient (Foundry-native) ---
# If your Language resource is connected to a Foundry project,
# this retrieves an authenticated client without managing keys.
project = AIProjectClient(
    endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
# text_client = project.inference.get_text_analysis_client()


# Sample documents — the SDK accepts a list, enabling batch processing.
documents = [
    "The delivery was incredibly fast and the packaging was excellent. "
    "However, the product itself stopped working after two days.",

    "Contoso's customer service team resolved my issue within an hour. "
    "Maria from the support team was particularly helpful.",

    "I ordered a laptop stand and received a phone case instead. "
    "Very disappointed with the fulfillment process.",
]


def analyze_text(docs: list[str]) -> None:
    """
    Run sentiment analysis, key phrase extraction, and NER on a list of documents.
    The SDK batches these efficiently — one API call per operation type.
    """

    print("=" * 60)
    print("SENTIMENT ANALYSIS")
    print("=" * 60)

    # sentiment_analysis_batch returns one result per document.
    # Each result contains: sentiment (positive/negative/mixed/neutral),
    # confidence scores, and per-sentence breakdowns.
    sentiment_results = text_client.analyze_sentiment(
        documents=docs,
        show_opinion_mining=True,  # Links sentiment to specific aspects (e.g., "packaging" = positive)
    )

    for i, result in enumerate(sentiment_results):
        if result.is_error:
            print(f"Doc {i}: ERROR — {result.error.code}: {result.error.message}")
            continue
        print(f"\nDoc {i}: Overall sentiment = {result.sentiment.upper()}")
        print(f"  Scores → Positive: {result.confidence_scores.positive:.2f} | "
              f"Negative: {result.confidence_scores.negative:.2f} | "
              f"Neutral: {result.confidence_scores.neutral:.2f}")
        for sentence in result.sentences:
            print(f"  Sentence ({sentence.sentiment}): \"{sentence.text[:60]}...\"")
            for opinion in sentence.mined_opinions:
                target = opinion.target
                print(f"    Aspect: '{target.text}' → {target.sentiment}")

    print("\n" + "=" * 60)
    print("KEY PHRASE EXTRACTION")
    print("=" * 60)

    # extract_key_phrases_batch returns the main concepts from each document.
    # Use this for automatic tagging, search indexing, or topic detection.
    key_phrase_results = text_client.extract_key_phrases(documents=docs)

    for i, result in enumerate(key_phrase_results):
        if result.is_error:
            print(f"Doc {i}: ERROR — {result.error.code}: {result.error.message}")
            continue
        print(f"\nDoc {i} key phrases: {', '.join(result.key_phrases)}")

    print("\n" + "=" * 60)
    print("NAMED ENTITY RECOGNITION (NER)")
    print("=" * 60)

    # recognize_entities_batch identifies structured entities:
    # Person, Organization, Location, DateTime, Quantity, URL, Email, etc.
    ner_results = text_client.recognize_entities(documents=docs)

    for i, result in enumerate(ner_results):
        if result.is_error:
            print(f"Doc {i}: ERROR — {result.error.code}: {result.error.message}")
            continue
        print(f"\nDoc {i} entities:")
        for entity in result.entities:
            print(f"  '{entity.text}' → Category: {entity.category} "
                  f"(Confidence: {entity.confidence_score:.2f})")

    print("\n" + "=" * 60)
    print("ABSTRACTIVE SUMMARIZATION")
    print("=" * 60)

    # begin_abstract_summary is a long-running operation — it returns a poller.
    # Abstractive summarization generates new sentences; it does NOT copy from the source.
    poller = text_client.begin_abstract_summary(documents=docs)
    summary_results = poller.result()

    for i, result in enumerate(summary_results):
        if result.is_error:
            print(f"Doc {i}: ERROR — {result.error.code}: {result.error.message}")
            continue
        print(f"\nDoc {i} summary:")
        for summary in result.summaries:
            print(f"  {summary.text}")


if __name__ == "__main__":
    analyze_text(documents)
```

---

## 🎯 Exam Practice: Text Analysis

<details>
<summary><strong>Q1.</strong> A company wants to automatically categorize support emails by topic (billing, technical, general) and identify which customers are mentioned by name. Which Azure AI Language features apply?</summary>

- A) Sentiment analysis and summarization
- B) Key phrase extraction and abstractive summarization
- C) Custom text classification and Named Entity Recognition (NER)
- D) Sentiment analysis and key phrase extraction

**Answer: C** — Categorizing emails into defined classes is text classification (custom model). Identifying customer names is NER (Person category). Sentiment tells you *how* the customer feels, not *what* the email is about.

</details>

---

<details>
<summary><strong>Q2.</strong> A developer uses Azure AI Language sentiment analysis on product reviews. A review reads: "The screen is beautiful but the battery lasts only 2 hours." The result shows overall sentiment = mixed. What additional feature would reveal *which specific aspects* of the product are positive vs. negative?</summary>

- A) Key phrase extraction
- B) Named entity recognition
- C) Opinion mining (aspect-based sentiment analysis)
- D) Abstractive summarization

**Answer: C** — Opinion mining (`show_opinion_mining=True`) links sentiment to specific targets within the text — in this case, "screen" = positive, "battery" = negative. Standard sentiment only returns the overall document-level score.

</details>

---

# 9.2 Responding to Spoken Prompts with a Multimodal Model

## The Architecture (Exam-Critical Pattern)

This is the pattern the exam calls **"respond to spoken prompts by using a deployed multimodal model."**

GPT-4o (in standard Azure deployment) accepts **text and images** as input — not raw audio. The spoken-prompt pattern wraps it:

```
[User speaks] → Azure AI Speech STT → [Text] → GPT-4o → [Text response] → Azure AI Speech TTS → [Spoken audio]
```

The "multimodal model" in this context refers to GPT-4o's ability to accept image + text together — the speech wrapping is handled by Azure AI Speech as a pre/post-processing step.

---

## The Complete Spoken Prompt Application

```python
import os
import azure.cognitiveservices.speech as speechsdk
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

# --- Azure AI Speech configuration ---
speech_config = speechsdk.SpeechConfig(
    subscription=os.environ["AZURE_SPEECH_KEY"],
    region=os.environ["AZURE_SPEECH_REGION"],
)
speech_config.speech_recognition_language = "en-US"
speech_config.speech_synthesis_voice_name = "en-US-AriaNeural"  # Neural TTS voice

# --- Foundry / OpenAI client ---
project = AIProjectClient(
    endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
openai_client = project.get_openai_client(api_version="2024-12-01-preview")
DEPLOYMENT_NAME = os.environ["FOUNDRY_MODEL_DEPLOYMENT"]

SYSTEM_PROMPT = (
    "You are a helpful voice assistant. "
    "Keep responses concise — under 3 sentences — because they will be read aloud."
)


def speech_to_text() -> str | None:
    """
    Capture one utterance from the microphone and return it as text.
    Uses the default microphone. Returns None on recognition failure.
    """
    audio_config = speechsdk.AudioConfig(use_default_microphone=True)
    recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config,
        audio_config=audio_config,
    )

    print("Listening... (speak now)")
    result = recognizer.recognize_once_async().get()

    if result.reason == speechsdk.ResultReason.RecognizedSpeech:
        print(f"Recognized: {result.text}")
        return result.text
    elif result.reason == speechsdk.ResultReason.NoMatch:
        print("No speech detected.")
    elif result.reason == speechsdk.ResultReason.Canceled:
        details = speechsdk.CancellationDetails.from_result(result)
        print(f"Recognition canceled: {details.reason} — {details.error_details}")
    return None


def text_to_speech(text: str) -> None:
    """
    Synthesize text to speech using the configured neural voice.
    Output goes to the default speaker.
    """
    synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config)
    result = synthesizer.speak_text_async(text).get()

    if result.reason == speechsdk.ResultReason.Canceled:
        details = speechsdk.CancellationDetails.from_result(result)
        print(f"TTS canceled: {details.reason} — {details.error_details}")


def ask_model(user_text: str, history: list) -> str:
    """
    Send the user's text to GPT-4o and return the assistant's response.
    History is maintained for multi-turn spoken conversation.
    """
    history.append({"role": "user", "content": user_text})

    response = openai_client.chat.completions.create(
        model=DEPLOYMENT_NAME,
        messages=[{"role": "system", "content": SYSTEM_PROMPT}] + history,
        temperature=0.7,
        max_tokens=150,  # Short responses for spoken output
    )

    assistant_text = response.choices[0].message.content
    history.append({"role": "assistant", "content": assistant_text})
    return assistant_text


def run_voice_assistant():
    """Multi-turn spoken conversation loop."""
    conversation_history = []
    print("Voice assistant ready. Say 'stop' or 'exit' to quit.\n")

    while True:
        user_text = speech_to_text()
        if not user_text:
            continue

        if user_text.lower().strip(".!?") in ("stop", "exit", "quit"):
            text_to_speech("Goodbye!")
            break

        response_text = ask_model(user_text, conversation_history)
        print(f"Assistant: {response_text}\n")
        text_to_speech(response_text)


if __name__ == "__main__":
    run_voice_assistant()
```

---

## 🎯 Exam Practice: Spoken Prompts

<details>
<summary><strong>Q1.</strong> A developer wants to build a voice assistant backed by GPT-4o. A user speaks a question → the assistant responds in audio. Which architecture is correct?</summary>

- A) Pass the audio file directly to GPT-4o — it accepts audio natively in all deployments
- B) Azure AI Speech STT → text to GPT-4o → response text to Azure AI Speech TTS → spoken audio
- C) Azure Content Understanding transcribes audio → GPT-4o generates response → DALL-E 3 speaks it
- D) Azure AI Language converts speech → GPT-4o responds → Azure AI Vision reads it aloud

**Answer: B** — GPT-4o in standard Azure Foundry deployment accepts text and images, not raw audio. The speech wrapping (STT before, TTS after) is done by Azure AI Speech. Content Understanding (C) is for document field extraction. Language (D) is for text analysis, not STT. Vision (D) is for image analysis, not TTS.

</details>

---

# 9.3 Lightweight Speech Application — Azure AI Speech

## Installation

```bash
pip install azure-cognitiveservices-speech azure-ai-projects azure-identity
```

## Core Capabilities Quick Reference

| Capability | SDK Class | Use Case |
|:---|:---|:---|
| **STT (microphone)** | `SpeechRecognizer` + `AudioConfig(use_default_microphone=True)` | Live voice input |
| **STT (audio file)** | `SpeechRecognizer` + `AudioConfig(filename=...)` | Pre-recorded audio |
| **TTS** | `SpeechSynthesizer` | Read responses aloud |
| **Speech Translation** | `TranslationRecognizer` | Real-time voice translation |
| **Batch Transcription** | REST API | Large volumes of audio files async |

## Transcribing an Audio File

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_config = speechsdk.SpeechConfig(
    subscription=os.environ["AZURE_SPEECH_KEY"],
    region=os.environ["AZURE_SPEECH_REGION"],
)
speech_config.speech_recognition_language = "en-US"

# Use an audio file instead of a microphone
audio_config = speechsdk.AudioConfig(filename="meeting_recording.wav")

recognizer = speechsdk.SpeechRecognizer(
    speech_config=speech_config,
    audio_config=audio_config,
)

# recognize_once_async processes one continuous utterance.
# For long audio files, use continuous recognition with event handlers.
result = recognizer.recognize_once_async().get()

if result.reason == speechsdk.ResultReason.RecognizedSpeech:
    print(f"Transcript: {result.text}")
elif result.reason == speechsdk.ResultReason.NoMatch:
    print("Speech could not be recognized.")
```

## Real-Time Speech Translation

```python
import azure.cognitiveservices.speech as speechsdk

speech_translation_config = speechsdk.translation.SpeechTranslationConfig(
    subscription=os.environ["AZURE_SPEECH_KEY"],
    region=os.environ["AZURE_SPEECH_REGION"],
)
speech_translation_config.speech_recognition_language = "es-ES"   # source: Spanish
speech_translation_config.add_target_language("en")               # target: English

audio_config = speechsdk.AudioConfig(use_default_microphone=True)

translator = speechsdk.translation.TranslationRecognizer(
    translation_config=speech_translation_config,
    audio_config=audio_config,
)

print("Speak in Spanish...")
result = translator.recognize_once_async().get()

if result.reason == speechsdk.ResultReason.TranslatedSpeech:
    print(f"Original (ES): {result.text}")
    print(f"Translation (EN): {result.translations['en']}")
```

---

## Sources

- [Azure AI Language Overview — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/language-service/overview)
- [Sentiment Analysis and Opinion Mining — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/language-service/sentiment-opinion-mining/overview)
- [Named Entity Recognition — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/language-service/named-entity-recognition/overview)
- [Azure AI Speech Overview — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/overview)
- [Speech-to-Text Quickstart — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/get-started-speech-to-text)
- [Text-to-Speech Quickstart — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/get-started-text-to-speech)
- [Speech Translation — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-translation)
- [AI-901 Official Study Guide — Microsoft Learn](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-901)
