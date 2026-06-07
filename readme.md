# Indian TTS Dataset – Build Report

This repository contains the dataset curation pipeline, quality filtration criteria, and design decisions for a **balanced, bilingual Text-to-Speech (TTS) training dataset** comprising approximately 60 minutes of clean, single-speaker audio split evenly between Marathi and Indian English.

The dataset contains accurate, manually verified transcripts matched with semantic emotion tags and is published as a public HuggingFace dataset.

🔗 **HuggingFace Dataset Link:** [Kalpeshkhare7777/indian-tts-dataset-marathi-english](https://huggingface.co/datasets/Kalpeshkhare7777/indian-tts-dataset-marathi-english)

---

## 📊 Dataset Overview

*   **Total Duration:** ~60 minutes
*   **Audio Format:** WAV, 22050 Hz, mono, 16-bit PCM
*   **Speaker Profiles:** Clean, single-speaker studio quality for both languages
*   **Language Split:**
    *   **Marathi:** ~30 minutes – Audiobook (*Shyamchi Aai* by Sane Guruji) | [Source Video](https://www.youtube.com/watch?v=SXfzVU2OoSE)
    *   **Indian English:** ~30 minutes – Audiobook (*The Palace of Illusions* by Chitra Divakaruni) | [Source Video](https://www.youtube.com/watch?v=TlLyX_tHTrc)

---

## 🛠️ How the Pipeline Works

The core extraction, text verification, and style tagging pipeline follows a robust, symmetrical workflow for both language tracks.

```
[Audiobook Source] ➔ [Manual Paragraph Splitting] ➔ [Text Verification against Book]
                                                               ↓
[Clean TTS Dataset] 💾 [HuggingFace Upload] ⬅ [sarvam-30b-llm Emotion Tagging]
```

### 1. Audio Processing & Segmentation
*   **Download:** Audio tracks were downloaded via `yt-dlp` and normalized to single-channel (mono) 22050 Hz WAV using `FFmpeg`.
*   **Manual Boundaries:** Instead of relying on automated Voice Activity Detection (VAD) which frequently slices text mid-sentence, natural paragraph boundaries were identified manually by listening.
*   **Target Length:** Audio was split precisely at paragraph breaks, targeting **30–90 seconds per segment** to maintain semantic continuity.

### 2. Transcript Verification
*   Transcripts were completely cross-checked and manually verified word-for-word against the printed texts of *Shyamchi Aai* and *The Palace of Illusions*.
*   Any spoken words that deviated from the official book text were explicitly corrected in the metadata to ensure near-perfect phonetic alignment.
*   Segments shorter than 20 seconds were omitted during quality filtering.

### 3. Emotion Tagging Prompt Design
Each verified segment transcript was processed using the `sarvam-30b-llm` API with a structured prompt to assign one of the following emotion labels:
*   `emotional`
*   `narrative calm` *(introduced specifically for literary storytelling)*
*   `conversational`
*   `sad`
*   `neutral`
*   `tense`

> ⚠️ **Limitation Note:** This approach utilizes text-based classification, meaning it labels the *semantic content* of speech rather than its *acoustic delivery*. A segment labeled as "neutral" based on text could still contain expressive acoustic prosody.

---

## 🔄 Iterative Improvements

*   **Iteration 1 (Automatic to Manual Splits):** Initial passes used automated VAD-based splitting. However, short pauses between sub-clauses resulted in grammatically incomplete utterances. The pipeline was transitioned to manual paragraph-level splitting, which dramatically increased overall TTS training suitability.
*   **Iteration 2 (Granular Style Mapping):** The initial scheme used only `positive`, `negative`, and `neutral`. This was too coarse for expressive audiobooks. The taxonomy was expanded to 5+ descriptive labels, including `narrative calm` and `tense`, followed by a full dataset re-tagging pass.
*   **Iteration 3 (Phonetic Rigor):** Automated ASR outputs were entirely abandoned in favor of a full manual alignment pass against the official book texts, eliminating spelling, grammatical, and formatting discrepancies.

---

## 📈 Quality Observations

*   **What Worked Well:** Leveraging professional audiobooks provided excellent single-speaker continuity and clean recording environments. Manual paragraph segmentation successfully eliminated unnatural clipping.
*   **TTS Readiness:** The resulting segments are grammatically complete, preserve natural phrasing, and maintain a consistent speaker identity, satisfying the key criteria needed for single-speaker TTS fine-tuning.

---

## 🚀 Future Roadmap & Improvements
Given more time, the following enhancements would be integrated into the pipeline:
1.  **Acoustic-based Tagging:** Combine the text-based LLM labels with acoustic feature extraction (pitch variance, energy variance, and speech tempo) to capture true vocal delivery.
2.  **Metadata Enrichment:** Include comprehensive speaker metadata (demographics, approximate age, and specific dialect/regional accent profiles).

---

## 💻 Tech Stack & Tooling

*   **Sarvam LLM (`sarvam-30b-llm`)** – Emotion & style classification
*   **yt-dlp** – High-fidelity source audio retrieval
*   **FFmpeg** – Audio downsampling (22050 Hz), channel mixing (mono), and precise segmentation
*   **Python** – Pipeline orchestration via `datasets`, `requests`, and `pathlib`
*   **HuggingFace Datasets** – Packaging, serialization, and public distribution
*   **Claude (Anthropic)** – Architecture design, pipeline review, and documentation assistance