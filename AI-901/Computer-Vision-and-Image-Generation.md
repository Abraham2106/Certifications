# Computer Vision & Image Generation — Study Guide

**Certification Code:** AI-901 | **Phase:** 2 — Implementing AI Solutions with Microsoft Foundry

> [!NOTE]
> This guide covers Modules 10.1–10.3: interpreting visual input with a multimodal model, generating new images with generative models, and building a lightweight vision application.

---

# 10.1 Interpreting Visual Input — GPT-4o Multimodal

## What "Multimodal" Means on the Exam

A multimodal model accepts **more than one input type in the same prompt**. For the AI-901, GPT-4o is the primary multimodal model — it accepts **text + image** together and returns a text response.

> [!IMPORTANT]
> **The exam distinction you must know:**
> - **GPT-4o multimodal** → accepts image + text in one prompt → returns text (description, analysis, QA)
> - **DALL-E 3 / gpt-image-1** → accepts text prompt → returns a *new generated image*
> - **Azure AI Vision** → accepts image → returns *structured data* (labels, bounding boxes, OCR text)
>
> "Interpreting visual input" = GPT-4o multimodal. "Creating visual output" = DALL-E 3 / gpt-image-1.

---

## Installation

```bash
pip install azure-ai-projects azure-identity openai
```

---

## Passing an Image to GPT-4o (Two Methods)

### Method 1 — Base64 Encoded (Local File)

```python
import os
import base64
from pathlib import Path
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

project = AIProjectClient(
    endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
openai_client = project.get_openai_client(api_version="2024-12-01-preview")
DEPLOYMENT = os.environ["FOUNDRY_MODEL_DEPLOYMENT"]  # must be GPT-4o


def encode_image(image_path: str) -> str:
    """Convert a local image file to a base64 string for the API."""
    with open(image_path, "rb") as f:
        return base64.b64encode(f.read()).decode("utf-8")


def analyze_local_image(image_path: str, question: str) -> str:
    """
    Send a local image + a text question to GPT-4o and return the answer.

    The message content is a list — one item for the image, one for the text.
    GPT-4o processes both simultaneously to produce a grounded response.
    """
    encoded = encode_image(image_path)

    # Determine MIME type from file extension
    ext = Path(image_path).suffix.lower()
    mime_types = {".jpg": "image/jpeg", ".jpeg": "image/jpeg",
                  ".png": "image/png", ".gif": "image/gif", ".webp": "image/webp"}
    mime = mime_types.get(ext, "image/jpeg")

    response = openai_client.chat.completions.create(
        model=DEPLOYMENT,
        messages=[
            {
                "role": "system",
                "content": "You are a visual analysis assistant. Describe what you see accurately and concisely.",
            },
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": f"data:{mime};base64,{encoded}",
                            "detail": "high",  # "low" for faster/cheaper, "high" for detailed analysis
                        },
                    },
                    {
                        "type": "text",
                        "text": question,
                    },
                ],
            },
        ],
        max_tokens=500,
    )

    return response.choices[0].message.content


# Example usage
if __name__ == "__main__":
    result = analyze_local_image(
        image_path="product_photo.jpg",
        question="Does this product show any visible defects or damage? List them.",
    )
    print(result)
```

---

### Method 2 — Public URL

```python
def analyze_image_from_url(image_url: str, question: str) -> str:
    """
    Send a publicly accessible image URL + a text question to GPT-4o.
    No encoding needed — the model fetches the image directly from the URL.
    Use this for images already hosted online (CDN, blob storage with public access).
    """
    response = openai_client.chat.completions.create(
        model=DEPLOYMENT,
        messages=[
            {
                "role": "system",
                "content": "You are a helpful visual assistant.",
            },
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": image_url,  # Direct public URL — no encoding
                            "detail": "low",   # Use "low" when detail is not critical
                        },
                    },
                    {
                        "type": "text",
                        "text": question,
                    },
                ],
            },
        ],
        max_tokens=300,
    )
    return response.choices[0].message.content
```

---

## The `detail` Parameter — Exam Awareness

| Value | Behavior | Token Cost | Use When |
|:---|:---|:---|:---|
| `"low"` | Resizes to 512×512, uses 85 tokens | Low | General scene description, thumbnails |
| `"high"` | Tiles the image for detail analysis | High (up to 1,105 tokens) | Medical images, charts, fine text |
| `"auto"` | Model decides based on image size | Variable | Default if unsure |

---

## 🎯 Exam Practice: Visual Input

<details>
<summary><strong>Q1.</strong> A quality control team wants an AI system where a technician photographs a circuit board and asks "Are there any solder bridges or missing components?" The system must reason about the image AND the question together.</summary>

- A) Azure AI Vision object detection — it can identify components
- B) Azure Content Understanding — it extracts fields from images
- C) GPT-4o in multimodal mode — text question + image in the same prompt
- D) DALL-E 3 — it generates the circuit board layout for comparison

**Answer: C** — The scenario requires combining a natural language question with an image in a single reasoning step. GPT-4o multimodal is the correct model. Azure AI Vision (A) would classify or detect objects but cannot answer a natural language question about what it sees. Content Understanding (B) is for extracting predefined fields from documents.

</details>

---

<details>
<summary><strong>Q2.</strong> A developer is deciding between <code>"detail": "low"</code> and <code>"detail": "high"</code> for a GPT-4o vision call. The application analyses satellite images to detect deforestation patterns in high resolution. Which setting is appropriate and why?</summary>

- A) `"low"` — lower cost is always preferred
- B) `"high"` — detailed spatial analysis requires the full image resolution
- C) `"auto"` — the model always makes the optimal choice
- D) Neither — satellite images are not supported by GPT-4o

**Answer: B** — Deforestation detection requires analyzing fine spatial patterns across a large image. `"high"` tiles the image for detailed processing. `"low"` compresses to 512×512 and would lose the fine-grain spatial detail needed for this task.

</details>

---

# 10.2 Image Generation — DALL-E 3 and gpt-image-1

## What Each Model Does

| Model | Input | Output | Use Case |
|:---|:---|:---|:---|
| **DALL-E 3** | Text prompt | New image | Marketing art, illustrations, concept visualization |
| **gpt-image-1** | Text prompt (+ optional image) | New / edited image | Product mockups, image editing, inpainting |

> [!IMPORTANT]
> **Key exam distinction:** GPT-4o can *understand* images (multimodal input). DALL-E 3 and gpt-image-1 *generate* new images (generative output). They are different model families serving opposite directions.

---

## Generating an Image with DALL-E 3

```python
import os
import requests
from pathlib import Path
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

project = AIProjectClient(
    endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
openai_client = project.get_openai_client(api_version="2024-12-01-preview")


def generate_image(
    prompt: str,
    output_path: str = "generated_image.png",
    size: str = "1024x1024",
    quality: str = "hd",
    style: str = "vivid",
) -> str:
    """
    Generate a new image from a text prompt using DALL-E 3.

    Parameters:
    - size: "1024x1024" (square), "1792x1024" (landscape), "1024x1792" (portrait)
    - quality: "standard" (faster/cheaper) or "hd" (more detail and consistency)
    - style: "vivid" (hyper-real, dramatic) or "natural" (realistic, less saturated)

    Returns the local path of the saved image.
    """
    response = openai_client.images.generate(
        model="dall-e-3",         # Must be the DALL-E 3 deployment name
        prompt=prompt,
        n=1,                      # DALL-E 3 supports n=1 only per call
        size=size,
        quality=quality,
        style=style,
    )

    image_url = response.data[0].url
    revised_prompt = response.data[0].revised_prompt
    print(f"Revised prompt: {revised_prompt}")

    # Download and save the generated image
    image_data = requests.get(image_url).content
    Path(output_path).write_bytes(image_data)
    print(f"Image saved to: {output_path}")

    return output_path


# Example: generate product marketing image
if __name__ == "__main__":
    generate_image(
        prompt=(
            "A sleek silver smartwatch resting on a polished obsidian surface, "
            "with soft studio lighting and a blurred city skyline in the background. "
            "Product photography style, ultra high resolution."
        ),
        output_path="watch_ad.png",
        quality="hd",
        style="vivid",
    )
```

---

## DALL-E 3 Parameters — Exam Reference

| Parameter | Options | Notes |
|:---|:---|:---|
| `n` | 1 only | DALL-E 3 generates one image per call |
| `size` | `1024x1024`, `1792x1024`, `1024x1792` | Landscape and portrait are widescreen formats |
| `quality` | `standard`, `hd` | `hd` improves fine details and consistency |
| `style` | `vivid`, `natural` | `vivid` is dramatic; `natural` is realistic |
| `revised_prompt` | Returned in response | DALL-E 3 may rewrite your prompt for safety/quality |

> [!NOTE]
> DALL-E 3 automatically rewrites ("revises") prompts to improve quality and enforce content policies. The `revised_prompt` field in the response shows what was actually used. This is expected behavior — not a bug.

---

## 🎯 Exam Practice: Image Generation

<details>
<summary><strong>Q1.</strong> A marketing team wants to generate a promotional banner image from a text description: "A tropical beach at sunset with the product logo floating above the waves." Which Azure service should they use?</summary>

- A) GPT-4o multimodal — it understands image descriptions
- B) Azure AI Vision — it generates images from object descriptions
- C) DALL-E 3 — it generates new images from text prompts
- D) Azure AI Language — it interprets the visual description

**Answer: C** — DALL-E 3 is the purpose-built image *generation* model. GPT-4o (A) can understand and describe images but does not generate new ones. Azure AI Vision (B) analyzes existing images; it is not generative. Language (D) is for text NLP.

</details>

---

<details>
<summary><strong>Q2.</strong> A developer generates an image with DALL-E 3 and notices the output does not exactly match their prompt. They check the API response and see a <code>revised_prompt</code> field with different text. What is the correct interpretation?</summary>

- A) An API error occurred and the wrong model was called
- B) DALL-E 3 rewrote the prompt to improve quality or comply with content policies — this is expected behavior
- C) The model misunderstood the prompt due to language model hallucination
- D) The `revised_prompt` field only appears when generation fails

**Answer: B** — DALL-E 3 automatically revises user prompts to enhance image quality and enforce content safety guidelines. The `revised_prompt` shows exactly what was sent to the image generation model. This is documented behavior, not an error.

</details>

---

# 10.3 Lightweight Vision Application

## Complete Multi-Feature Vision Application

This combines GPT-4o multimodal analysis AND Azure AI Vision OCR in a single application — demonstrating how to choose between them based on the task.

```python
import os
import base64
from pathlib import Path
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential


# --- Clients ---
project = AIProjectClient(
    endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
    credential=DefaultAzureCredential(),
)
openai_client = project.get_openai_client(api_version="2024-12-01-preview")
DEPLOYMENT = os.environ["FOUNDRY_MODEL_DEPLOYMENT"]

# Azure AI Vision client — for structured vision tasks (OCR, object detection, etc.)
vision_client = ImageAnalysisClient(
    endpoint=os.environ["AZURE_VISION_ENDPOINT"],
    credential=AzureKeyCredential(os.environ["AZURE_VISION_KEY"]),
)


def extract_text_from_image(image_path: str) -> str:
    """
    Use Azure AI Vision OCR to extract all readable text from an image.
    Returns the extracted text as a single string.

    Use this when: you need raw text extracted from signage, documents,
    whiteboards, printed forms — without natural language reasoning about it.
    """
    with open(image_path, "rb") as f:
        image_data = f.read()

    result = vision_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ],  # READ = OCR
    )

    extracted_lines = []
    if result.read and result.read.blocks:
        for block in result.read.blocks:
            for line in block.lines:
                extracted_lines.append(line.text)

    return "\n".join(extracted_lines)


def detect_objects_in_image(image_path: str) -> list[dict]:
    """
    Use Azure AI Vision object detection to find and locate objects in an image.
    Returns a list of detected objects with bounding boxes and confidence scores.

    Use this when: you need structured data about WHAT is in the image and WHERE.
    """
    with open(image_path, "rb") as f:
        image_data = f.read()

    result = vision_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.OBJECTS],
    )

    detected = []
    if result.objects and result.objects.list:
        for obj in result.objects.list:
            detected.append({
                "name": obj.tags[0].name if obj.tags else "unknown",
                "confidence": round(obj.tags[0].confidence, 2) if obj.tags else 0,
                "bounding_box": {
                    "x": obj.bounding_box.x,
                    "y": obj.bounding_box.y,
                    "width": obj.bounding_box.width,
                    "height": obj.bounding_box.height,
                },
            })
    return detected


def reason_about_image(image_path: str, question: str) -> str:
    """
    Use GPT-4o multimodal to reason about an image with a natural language question.
    Returns a natural language answer grounded in both the image and the question.

    Use this when: you need the model to *understand and reason* about what it sees —
    not just extract structured labels or text.
    """
    with open(image_path, "rb") as f:
        encoded = base64.b64encode(f.read()).decode("utf-8")

    ext = Path(image_path).suffix.lower()
    mime = "image/jpeg" if ext in (".jpg", ".jpeg") else "image/png"

    response = openai_client.chat.completions.create(
        model=DEPLOYMENT,
        messages=[
            {"role": "system", "content": "You are a precise visual analysis assistant."},
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": f"data:{mime};base64,{encoded}", "detail": "high"}},
                    {"type": "text", "text": question},
                ],
            },
        ],
        max_tokens=400,
    )
    return response.choices[0].message.content


if __name__ == "__main__":
    image = "store_shelf.jpg"

    print("=== OCR (Azure AI Vision) ===")
    ocr_text = extract_text_from_image(image)
    print(ocr_text)

    print("\n=== Object Detection (Azure AI Vision) ===")
    objects = detect_objects_in_image(image)
    for obj in objects:
        print(f"  {obj['name']} (confidence: {obj['confidence']}) at {obj['bounding_box']}")

    print("\n=== Natural Language Reasoning (GPT-4o Multimodal) ===")
    answer = reason_about_image(image, "Which products appear out of stock? How can you tell?")
    print(answer)
```

---

## When to Use Which Vision Tool

| Scenario | Tool | Why |
|:---|:---|:---|
| Count objects in an image | Azure AI Vision (Objects) | Returns structured count + bounding boxes |
| Read text from a sign or photo | Azure AI Vision (OCR / READ) | Fast, accurate text extraction |
| "What is wrong with this machine?" | GPT-4o multimodal | Requires reasoning, not just detection |
| Generate a new product image | DALL-E 3 / gpt-image-1 | Image generation, not analysis |
| "Is this invoice format correct?" | GPT-4o multimodal OR Content Understanding | Reasoning about structure |
| Extract amount from an invoice | Azure Content Understanding | Structured field extraction |

---

## Sources

- [GPT-4o Vision Documentation — Azure OpenAI — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/gpt-with-vision)
- [Image Generation with DALL-E — Azure OpenAI — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/dall-e)
- [Azure AI Vision Overview — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview)
- [Image Analysis — Azure AI Vision — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/concept-object-detection)
- [OCR — Azure AI Vision — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/concept-ocr)
- [Azure AI Vision Python SDK — Microsoft Learn](https://learn.microsoft.com/en-us/python/api/overview/azure/ai-vision-imageanalysis-readme)
- [AI-901 Official Study Guide — Microsoft Learn](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-901)
