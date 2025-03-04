# Docling Actor on Apify

[![Docling Actor](https://apify.com/actor-badge?actor=vancura/docling?fpr=docling)](https://apify.com/vancura/docling)

This Actor (specification v1) wraps the [Docling project](https://ds4sd.github.io/docling/) to provide serverless document processing in the cloud. It can process complex documents (PDF, DOCX, images) and convert them into structured formats (Markdown, JSON, HTML, Text, or DocTags) with optional OCR support.

## What are Actors?

[Actors](https://docs.apify.com/platform/actors?fpr=docling) are serverless microservices running on the [Apify Platform](https://apify.com/?fpr=docling). They are based on the [Actor SDK](https://docs.apify.com/sdk/js?fpr=docling) and can be found in the [Apify Store](https://apify.com/store?fpr=docling). Learn more about Actors in the [Apify Whitepaper](https://whitepaper.actor?fpr=docling).

## Table of Contents

1. [Features](#features)
2. [Usage](#usage)
3. [Input Parameters](#input-parameters)
4. [Output](#output)
5. [Performance & Resources](#performance--resources)
6. [Troubleshooting](#troubleshooting)
7. [Local Development](#local-development)
8. [Requirements & Installation](#requirements--installation)
9. [License](#license)
10. [Acknowledgments](#acknowledgments)
11. [Security Considerations](#security-considerations)

## Features

- Runs Docling v2.17.0 in a fully managed environment on Apify
- Processes multiple document formats:
  - PDF documents (scanned or digital)
  - Microsoft Office files (DOCX, XLSX, PPTX)
  - Images (PNG, JPG, TIFF)
  - Other text-based formats
- Provides OCR capabilities for scanned documents
- Exports to multiple formats:
  - Markdown
  - JSON
  - HTML
  - Plain Text
  - DocTags (structured format)
- No local setup needed—just provide input via a simple JSON config

## Usage

### Using Apify Console

1. Go to the Apify Actor page.
2. Click "Run".
3. In the input form, fill in:
   - The URL of the document.
   - Output format (`md`, `json`, `html`, `text`, or `doctags`).
   - OCR boolean toggle.
4. The Actor will run and produce its outputs in the default key-value store under the key `OUTPUT_RESULT`.

### Using Apify API

```bash
curl --request POST \
  --url "https://api.apify.com/v2/acts/username~actorname/run" \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer YOUR_API_TOKEN' \
  --data '{
    "documentUrl": "https://arxiv.org/pdf/2408.09869.pdf",
    "outputFormat": "md",
    "ocr": true
  }'
```

### Using Apify CLI

```bash
apify call username/actorname --input='{
    "documentUrl": "https://arxiv.org/pdf/2408.09869.pdf",
    "outputFormat": "md",
    "ocr": true
}'
```

## Input Parameters

The Actor accepts a JSON schema matching the file `.actor/input_schema.json`. Below is a summary of the fields:

| Field          | Type    | Required | Default  | Description                                                                                               |
|----------------|---------|----------|----------|-----------------------------------------------------------------------------------------------------------|
| `documentUrl`  | string  | Yes      | None     | URL of the document (PDF, image, DOCX, etc.) to be processed. Must be directly accessible via public URL. |
| `outputFormat` | string  | No       | `md`     | Desired output format. One of `md`, `json`, `html`, `text`, or `doctags`.                                 |
| `ocr`          | boolean | No       | `true`   | If set to true, OCR will be applied to scanned PDFs or images for text recognition.                       |

### Example Input

```json
{
    "documentUrl": "https://arxiv.org/pdf/2408.09869.pdf",
    "outputFormat": "md",
    "ocr": false
}
```

## Output

The Actor provides three types of outputs:

1. **Processed Document** - The Actor will provide the direct URL to your result in the run log, looking like:

   ```text
   You can find your results at: 'https://api.apify.com/v2/key-value-stores/[YOUR_STORE_ID]/records/OUTPUT_RESULT'
   ```

2. **Processing Log** - Available in the key-value store as `DOCLING_LOG`

3. **Dataset Record** - Contains processing metadata with:
   - Input document URL
   - Direct link to the processed output
   - Processing status

You can access the results in several ways:

1. **Direct URL** (shown in Actor run logs):

```text
https://api.apify.com/v2/key-value-stores/[STORE_ID]/records/OUTPUT_RESULT
```

2. **Programmatically** via Apify CLI:

```bash
apify key-value-stores get-value OUTPUT_RESULT
```

3. **Dataset** - Check the "Dataset" tab in the Actor run details to see processing metadata

### Example Outputs

#### Markdown (md)

```markdown
# Document Title

## Section 1
Content of section 1...

## Section 2
Content of section 2...
```

#### JSON

```json
{
    "title": "Document Title",
    "sections": [
        {
            "level": 1,
            "title": "Section 1",
            "content": "Content of section 1..."
        }
    ]
}
```

#### HTML

```html
<h1>Document Title</h1>
<h2>Section 1</h2>
<p>Content of section 1...</p>
```

### Processing Logs (`DOCLING_LOG`)

The Actor maintains detailed processing logs including:

- Memory usage statistics
- Processing steps and timing
- Error messages and stack traces
- Input validation results
- OCR processing details (when enabled)

Access logs via:

```bash
apify key-value-stores get-record DOCLING_LOG
```

## Performance & Resources

- **Docker Image Size**: ~6 GB (includes OCR libraries and ML models)
- **Memory Requirements**:
  - Minimum: 4 GB RAM
  - Recommended: 8 GB RAM for large documents
- **Memory Monitoring**:
  - Real-time memory usage tracking during processing
  - Detailed memory statistics in `DOCLING_LOG`
  - Automatic failure detection for out-of-memory situations
- **Processing Time**:
  - Simple documents: 30-60 seconds
  - Complex PDFs with OCR: 2-5 minutes
  - Large documents (100+ pages): 5-15 minutes

## Troubleshooting

Common issues and solutions:

1. **Document URL Not Accessible**
   - Ensure the URL is publicly accessible
   - Check if the document requires authentication
   - Verify the URL leads directly to the document

2. **OCR Processing Fails**
   - Verify the document is not password-protected
   - Check if the image quality is sufficient
   - Try processing with OCR disabled

3. **Memory Issues**
   - For large documents, try splitting them into smaller chunks
   - Consider using a higher-memory compute unit
   - Disable OCR if not strictly necessary

4. **Output Format Issues**
   - Verify the output format is supported
   - Check if the document structure is compatible
   - Review the `DOCLING_LOG` for specific errors

### Error Handling

The Actor implements comprehensive error handling:

- Input validation for document URLs and parameters
- Detailed error messages in `DOCLING_LOG`
- Proper exit codes for different failure scenarios
- Memory monitoring and out-of-memory detection
- Automatic cleanup on failure
- Dataset records with processing status

## Local Development

If you wish to develop or modify this Actor locally:

1. Clone the repository.
2. Ensure Docker is installed.
3. The Actor files are located in the `.actor` directory:
   - `Dockerfile` - Defines the container environment
   - `actor.json` - Actor configuration and metadata
   - `actor.sh` - Main execution script
   - `input_schema.json` - Input parameter definitions
   - `.dockerignore` - Build optimization rules
4. Run the Actor locally using:

   ```bash
   apify run
   ```

### Actor Structure

```text
.actor/
├── Dockerfile          # Container definition
├── actor.json          # Actor metadata
├── actor.sh            # Execution script
├── input_schema.json   # Input parameters
├── .dockerignore       # Build exclusions
└── README.md           # This documentation
```

## Requirements & Installation

- An [Apify account](https://console.apify.com/?fpr=docling) (free tier available)
- For local development:
  - Docker installed
  - Apify CLI (`npm install -g apify-cli`)
  - Git for version control
- The Actor's Docker image (~6 GB) includes:
  - Python 3.11 with optimized caching (.pyc, .pyo excluded)
  - Node.js 20.x
  - Docling v2.17.0 and its dependencies
  - OCR libraries and ML models

### Build Optimizations

The Actor uses several optimizations to maintain efficiency:

- Python cache files (`pycache`, `.pyc`, `.pyo`, `.pyd`) are excluded
- Development artifacts (`.git`, `.env`, `.venv`) are ignored
- Log and test files (`*.log`, `.pytest_cache`, `.coverage`) are excluded from builds

## License

This wrapper project is under the MIT License, matching the original Docling license. See [LICENSE](../LICENSE) for details.

## Acknowledgments

- [Docling](https://ds4sd.github.io/docling/) codebase by IBM
- [Apify](https://apify.com/?fpr=docling) for the serverless actor environment

## Security Considerations

- Actor runs under a non-root user (appuser) for enhanced security
- Input URLs are validated before processing
- Temporary files are securely managed and cleaned up
- Process isolation through Docker containerization
- Secure handling of processing artifacts
