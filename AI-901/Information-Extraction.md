# Information Extraction — Study Guide

**Certification Code:** AI-901 | **Phase:** 2 — Implementing AI Solutions with Microsoft Foundry

> [!NOTE]
> This guide covers Module 11: extracting structured information from documents, images, audio, and video using Azure Content Understanding in Foundry Tools.

---

# 11.1 What Is Information Extraction?

## Exam-Ready Definition

Information extraction workloads process **structured and semi-structured sources** (forms, invoices, contracts, PDFs, audio recordings, videos) to extract **specific named fields** as machine-readable output (JSON).

> [!IMPORTANT]
> **The critical distinction for the exam:**
>
> | Workload | Input | Goal | Azure Service |
> |:---|:---|:---|:---|
> | **Text Analysis** | Unstructured prose (reviews, emails) | Understand and label free text | Azure AI Language |
> | **OCR** | Image with text | Extract all readable text verbatim | Azure AI Vision |
> | **Information Extraction** | Documents/forms with a known structure | Extract specific named fields | Azure Content Understanding |
>
> A shipping label with 3 known fields (sender, recipient, tracking number) → **Content Understanding**.
> A customer email about a bad experience → **Azure AI Language**.
> A photo of a whiteboard → **Azure AI Vision (OCR)**.

---

## Azure Content Understanding

Azure Content Understanding is the unified service in Foundry Tools for extracting information from **any modality**: documents, images, audio files, and video. It replaced the older Azure Form Recognizer (Document Intelligence) as the primary exam-tested service for this workload.

**Supported input modalities:**

| Modality | Example Sources | What It Extracts |
|:---|:---|:---|
| **Documents / Forms** | PDFs, Word docs, scanned forms, invoices | Named key-value pairs, tables, line items |
| **Images** | Photos of receipts, labels, IDs | Text fields, structured data from visual layout |
| **Audio** | Call recordings, voicemails | Speaker, timestamps, field values from speech |
| **Video** | Meeting recordings, training videos | Segments, transcripts, key moments, field values |

---

# 11.2 Extracting from Documents and Forms

## Installation

```bash
pip install azure-ai-projects azure-identity azure-ai-documentintelligence
```

---

## Using Content Understanding for Document Field Extraction

```python
import os
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.documentintelligence import DocumentIntelligenceClient
from azure.ai.documentintelligence.models import AnalyzeDocumentRequest
from azure.core.credentials import AzureKeyCredential


# --- Client setup ---
# Content Understanding is accessed via the Document Intelligence SDK.
# The endpoint and key come from your Azure Content Understanding resource.
doc_client = DocumentIntelligenceClient(
    endpoint=os.environ["AZURE_CONTENT_UNDERSTANDING_ENDPOINT"],
    credential=AzureKeyCredential(os.environ["AZURE_CONTENT_UNDERSTANDING_KEY"]),
)


def extract_invoice_fields(pdf_path: str) -> dict:
    """
    Extract structured fields from an invoice PDF using the prebuilt invoice model.

    Prebuilt models understand common document layouts without training:
    - prebuilt-invoice: invoices (vendor, date, total, line items)
    - prebuilt-receipt: receipts (merchant, items, total, tax)
    - prebuilt-idDocument: passports and national IDs
    - prebuilt-contract: contracts (parties, dates, obligations)
    - prebuilt-read: general OCR for any document

    For custom document types (e.g., a proprietary internal form),
    you train a custom model in the Content Understanding portal.
    """
    with open(pdf_path, "rb") as f:
        pdf_bytes = f.read()

    # begin_analyze_document starts a long-running operation.
    # The SDK returns a poller — call .result() to wait for completion.
    poller = doc_client.begin_analyze_document(
        model_id="prebuilt-invoice",
        body=AnalyzeDocumentRequest(bytes_source=pdf_bytes),
    )
    result = poller.result()

    extracted = {}

    for document in result.documents:
        fields = document.fields

        # Each field has a .value_string / .value_number / .value_date
        # and a .confidence score (0.0 to 1.0).
        extracted["vendor_name"] = _get_field(fields, "VendorName")
        extracted["invoice_date"] = _get_field(fields, "InvoiceDate")
        extracted["invoice_total"] = _get_field(fields, "InvoiceTotal")
        extracted["invoice_id"] = _get_field(fields, "InvoiceId")
        extracted["customer_name"] = _get_field(fields, "CustomerName")

        # Line items are nested — extract as a list of dicts
        line_items = []
        if "Items" in fields and fields["Items"].value_array:
            for item in fields["Items"].value_array:
                if item.value_object:
                    line_items.append({
                        "description": _get_field(item.value_object, "Description"),
                        "quantity": _get_field(item.value_object, "Quantity"),
                        "unit_price": _get_field(item.value_object, "UnitPrice"),
                        "amount": _get_field(item.value_object, "Amount"),
                    })
        extracted["line_items"] = line_items

    return extracted


def _get_field(fields: dict, field_name: str) -> str | None:
    """Safely extract a field value with its confidence score."""
    if field_name in fields and fields[field_name]:
        field = fields[field_name]
        value = field.content or str(field.value) if field.value else None
        confidence = round(field.confidence, 2) if field.confidence else None
        return {"value": value, "confidence": confidence}
    return None


def process_invoice_batch(pdf_paths: list[str]) -> list[dict]:
    """Process multiple invoices and return structured results."""
    results = []
    for path in pdf_paths:
        print(f"Processing: {path}")
        extracted = extract_invoice_fields(path)
        extracted["source_file"] = path
        results.append(extracted)
        print(f"  Vendor: {extracted.get('vendor_name')}")
        print(f"  Total: {extracted.get('invoice_total')}")
        print(f"  Line items: {len(extracted.get('line_items', []))}")
    return results


if __name__ == "__main__":
    invoices = ["invoice_001.pdf", "invoice_002.pdf", "invoice_003.pdf"]
    all_results = process_invoice_batch(invoices)

    print(f"\nProcessed {len(all_results)} invoices.")
    total_sum = sum(
        float(r["invoice_total"]["value"].replace("$", "").replace(",", ""))
        for r in all_results
        if r.get("invoice_total") and r["invoice_total"].get("value")
    )
    print(f"Total invoiced amount: ${total_sum:,.2f}")
```

---

## Prebuilt Models — Exam Reference

| Model ID | Document Type | Key Fields Extracted |
|:---|:---|:---|
| `prebuilt-invoice` | Invoices | Vendor, date, total, line items, tax |
| `prebuilt-receipt` | Receipts | Merchant, items, subtotal, total, tax |
| `prebuilt-idDocument` | Passports, IDs | Name, DOB, ID number, expiry, nationality |
| `prebuilt-contract` | Contracts | Parties, dates, payment terms, clauses |
| `prebuilt-read` | Any document | Raw text with layout (OCR + layout) |
| `prebuilt-layout` | Any document | Text, tables, selection marks with positions |

---

# 11.3 Extracting from Images

## Extracting Information from a Photographed Form

Content Understanding handles photos of physical documents just as well as digital PDFs. The model accounts for perspective, lighting variation, and handwriting.

```python
def extract_from_image(image_path: str, model_id: str = "prebuilt-idDocument") -> dict:
    """
    Extract structured fields from a photographed document (JPEG, PNG, TIFF, BMP).

    Same API as PDF extraction — Content Understanding handles the modality automatically.
    The model_id determines what fields are expected.

    Common use cases:
    - Photo of a driver's license → prebuilt-idDocument
    - Photo of a handwritten expense form → custom trained model
    - Photo of a receipt → prebuilt-receipt
    """
    with open(image_path, "rb") as f:
        image_bytes = f.read()

    poller = doc_client.begin_analyze_document(
        model_id=model_id,
        body=AnalyzeDocumentRequest(bytes_source=image_bytes),
    )
    result = poller.result()

    extracted = {}
    for document in result.documents:
        for field_name, field in document.fields.items():
            if field and field.content:
                extracted[field_name] = {
                    "value": field.content,
                    "confidence": round(field.confidence, 2) if field.confidence else None,
                }

    return extracted


if __name__ == "__main__":
    # Extracting a photographed government ID
    id_fields = extract_from_image("passport_photo.jpg", model_id="prebuilt-idDocument")
    print("Extracted ID fields:")
    for field, data in id_fields.items():
        print(f"  {field}: {data['value']} (confidence: {data['confidence']})")
```

---

# 11.4 Extracting from Audio and Video

## Audio Extraction — Call Recordings and Voicemails

For audio, Content Understanding transcribes the audio and then extracts the specified fields from the transcript in a single pipeline. You do not need to manage a separate STT step.

```python
import os
import time
import requests

# Content Understanding audio/video extraction uses a REST API
# (the Document Intelligence SDK covers documents/images).
# The endpoint format is the Content Understanding REST endpoint.

CONTENT_UNDERSTANDING_ENDPOINT = os.environ["AZURE_CONTENT_UNDERSTANDING_ENDPOINT"]
CONTENT_UNDERSTANDING_KEY = os.environ["AZURE_CONTENT_UNDERSTANDING_KEY"]

HEADERS = {
    "Ocp-Apim-Subscription-Key": CONTENT_UNDERSTANDING_KEY,
    "Content-Type": "application/json",
}


def extract_from_audio(audio_url: str, fields_to_extract: list[str]) -> dict:
    """
    Extract specific named fields from an audio recording via Content Understanding.

    The service:
    1. Transcribes the audio internally
    2. Runs field extraction on the transcript
    3. Returns structured JSON with field values and confidence scores

    audio_url: A SAS URL or publicly accessible URL to the audio file.
    fields_to_extract: List of field names to extract (e.g., ["CallerName", "PolicyNumber", "IncidentDate"])
    """
    # Build the field schema dynamically
    field_schema = {
        field: {"type": "string", "description": f"The {field} mentioned in the audio."}
        for field in fields_to_extract
    }

    # Submit the extraction job
    submit_url = f"{CONTENT_UNDERSTANDING_ENDPOINT}/contentunderstanding/analyzers/prebuilt-audioAnalyzer:analyze?api-version=2024-12-01-preview"

    payload = {
        "url": audio_url,
        "fields": field_schema,
    }

    response = requests.post(submit_url, headers=HEADERS, json=payload)
    response.raise_for_status()

    # The response contains an operation-location header for polling
    operation_url = response.headers.get("operation-location")
    print(f"Job submitted. Polling: {operation_url}")

    # Poll until complete
    while True:
        poll_response = requests.get(operation_url, headers=HEADERS)
        poll_response.raise_for_status()
        status_data = poll_response.json()

        status = status_data.get("status")
        print(f"Status: {status}")

        if status == "succeeded":
            return status_data.get("result", {})
        elif status in ("failed", "canceled"):
            raise RuntimeError(f"Extraction failed: {status_data.get('error')}")

        time.sleep(3)


if __name__ == "__main__":
    # Example: extract claim fields from an insurance voicemail
    audio_sas_url = "https://mystorage.blob.core.windows.net/audio/claim_voicemail.mp3?sv=..."

    result = extract_from_audio(
        audio_url=audio_sas_url,
        fields_to_extract=["CallerName", "PolicyNumber", "IncidentDate", "IncidentDescription"],
    )

    print("\nExtracted fields:")
    for field, data in result.get("fields", {}).items():
        print(f"  {field}: {data.get('valueString')} (confidence: {data.get('confidence', 'N/A')})")
```

---

## Video Extraction

```python
def extract_from_video(video_url: str) -> dict:
    """
    Extract structured information from a video file using Content Understanding.

    The service analyzes the video for:
    - Transcript segments (speech → text, per speaker, with timestamps)
    - Key moments and chapter boundaries
    - Visual scenes and objects
    - Custom field values extracted from spoken or visual content

    video_url: Accessible URL to the video file (MP4, MOV, AVI, etc.)
    Returns the full result object with segments, transcript, and extracted fields.
    """
    submit_url = f"{CONTENT_UNDERSTANDING_ENDPOINT}/contentunderstanding/analyzers/prebuilt-videoAnalyzer:analyze?api-version=2024-12-01-preview"

    payload = {"url": video_url}
    response = requests.post(submit_url, headers=HEADERS, json=payload)
    response.raise_for_status()

    operation_url = response.headers["operation-location"]

    while True:
        poll = requests.get(operation_url, headers=HEADERS)
        poll.raise_for_status()
        data = poll.json()

        if data.get("status") == "succeeded":
            return data.get("result", {})
        elif data.get("status") in ("failed", "canceled"):
            raise RuntimeError(f"Video extraction failed: {data.get('error')}")

        time.sleep(5)
```

---

# 11.5 Lightweight Information Extraction Application

## Complete Production-Ready Extraction Client

```python
import os
from enum import Enum
from azure.ai.documentintelligence import DocumentIntelligenceClient
from azure.ai.documentintelligence.models import AnalyzeDocumentRequest
from azure.core.credentials import AzureKeyCredential


class DocumentType(Enum):
    INVOICE = "prebuilt-invoice"
    RECEIPT = "prebuilt-receipt"
    ID_DOCUMENT = "prebuilt-idDocument"
    CONTRACT = "prebuilt-contract"
    GENERAL = "prebuilt-read"


class ContentUnderstandingClient:
    """
    Lightweight wrapper for Azure Content Understanding document extraction.
    Handles both local files and remote URLs.
    """

    def __init__(self):
        self.client = DocumentIntelligenceClient(
            endpoint=os.environ["AZURE_CONTENT_UNDERSTANDING_ENDPOINT"],
            credential=AzureKeyCredential(os.environ["AZURE_CONTENT_UNDERSTANDING_KEY"]),
        )

    def extract(
        self,
        source: str,
        document_type: DocumentType = DocumentType.INVOICE,
        confidence_threshold: float = 0.7,
    ) -> dict:
        """
        Extract structured fields from a document.

        source: Local file path OR a URL (SAS URL for Azure Blob Storage).
        document_type: Which prebuilt model to use.
        confidence_threshold: Fields below this confidence are flagged for review.
        """
        if source.startswith("http"):
            # URL source — send URL directly
            body = AnalyzeDocumentRequest(url_source=source)
        else:
            # Local file source — read bytes
            with open(source, "rb") as f:
                body = AnalyzeDocumentRequest(bytes_source=f.read())

        poller = self.client.begin_analyze_document(
            model_id=document_type.value,
            body=body,
        )
        result = poller.result()

        extracted_fields = {}
        low_confidence_fields = []

        for document in result.documents:
            for field_name, field in document.fields.items():
                if not field or not field.content:
                    continue

                confidence = field.confidence or 0.0
                entry = {
                    "value": field.content,
                    "confidence": round(confidence, 2),
                    "needs_review": confidence < confidence_threshold,
                }
                extracted_fields[field_name] = entry

                if confidence < confidence_threshold:
                    low_confidence_fields.append(field_name)

        return {
            "fields": extracted_fields,
            "model_used": document_type.value,
            "low_confidence_fields": low_confidence_fields,
            "requires_human_review": len(low_confidence_fields) > 0,
        }


def main():
    client = ContentUnderstandingClient()

    # Process a batch of invoices
    sources = [
        "invoice_q1.pdf",
        "https://myblobstorage.blob.core.windows.net/invoices/invoice_q2.pdf?sv=...",
        "receipt_scan.jpg",
    ]

    document_types = [
        DocumentType.INVOICE,
        DocumentType.INVOICE,
        DocumentType.RECEIPT,
    ]

    for source, doc_type in zip(sources, document_types):
        print(f"\nProcessing: {source}")
        result = client.extract(source, doc_type, confidence_threshold=0.75)

        print(f"  Model: {result['model_used']}")
        print(f"  Fields extracted: {len(result['fields'])}")
        print(f"  Requires human review: {result['requires_human_review']}")
        if result["low_confidence_fields"]:
            print(f"  Low confidence fields: {', '.join(result['low_confidence_fields'])}")

        for field, data in result["fields"].items():
            flag = " ⚠️ REVIEW" if data["needs_review"] else ""
            print(f"  {field}: {data['value']} ({data['confidence']}){flag}")


if __name__ == "__main__":
    main()
```

---

## 🎯 Exam Practice: Information Extraction

<details>
<summary><strong>Q1.</strong> A government agency receives thousands of scanned tax forms. They need to extract taxpayer ID, declared income, and filing date from each form as structured data. Which service and why?</summary>

- A) Azure AI Language — key phrase extraction identifies the important fields
- B) Azure AI Vision OCR — extracts all text from the scanned image
- C) Azure Content Understanding with a prebuilt or custom model
- D) GPT-4o multimodal — it can read the form and extract the fields

**Answer: C** — OCR (B) would return all text indiscriminately, not mapped to field names. Language (A) is for unstructured prose analysis. GPT-4o (D) would work but is expensive, slow for batch, and not the purpose-built solution. Content Understanding is designed for exactly this: extracting predefined named fields from structured forms at scale.

</details>

---

<details>
<summary><strong>Q2.</strong> An insurance company needs to extract claimant name, policy number, and incident date from recorded voicemail messages. Which service handles this?</summary>

- A) Azure AI Speech STT to transcribe, then Azure AI Language NER to extract entities
- B) Azure Content Understanding audio extraction mode
- C) Azure AI Vision OCR applied to audio waveforms
- D) GPT-4o multimodal with audio input

**Answer: B** — Content Understanding handles the full pipeline for audio: transcription + field extraction in one service call. Option A would also work technically (STT + NER) but requires two services and is not the purpose-built answer. The exam signals "extract specific fields from audio/video" → Content Understanding.

</details>

---

<details>
<summary><strong>Q3.</strong> A developer uses Content Understanding on a scanned invoice. One field (InvoiceTotal) comes back with a confidence score of 0.42. What should the application do?</summary>

- A) Trust the result — confidence above 0.0 is sufficient
- B) Discard the entire invoice and request a rescan
- C) Flag the field for human review before entering it into the ERP system
- D) Switch to Azure AI Language to re-extract the total

**Answer: C** — Confidence scores indicate the model's certainty. A low-confidence field should be flagged for human review — not trusted blindly and not cause the entire document to be discarded. This is the human-in-the-loop pattern applied to information extraction.

</details>

---

## Sources

- [Azure Content Understanding Overview — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/overview)
- [Document Intelligence (Prebuilt Models) — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/overview)
- [Prebuilt Invoice Model — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/prebuilt/invoice)
- [Content Understanding Audio Extraction — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/audio/overview)
- [Content Understanding Video Extraction — Microsoft Learn](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/video/overview)
- [Azure AI Document Intelligence Python SDK — Microsoft Learn](https://learn.microsoft.com/en-us/python/api/overview/azure/ai-documentintelligence-readme)
- [AI-901 Official Study Guide — Microsoft Learn](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/ai-901)
