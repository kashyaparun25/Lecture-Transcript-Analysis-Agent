# Lecture Transcript Analysis Agent

This Streamlit application leverages the CrewAI framework and Google's Gemini model to process lecture transcripts and generate comprehensive, structured educational materials.  It takes raw lecture transcripts (in various formats) and outputs a well-organized document suitable for study, review, and assessment.

## Features

*   **Multiple File Format Support:** Accepts transcripts in `.txt`, `.pdf`, `.docx`, and `.vtt` formats.
*   **Automated Content Generation:**  Produces a structured document containing the following sections, in the specified order:
    *   Title and Speaker Information
    *   Key Quotes (20-25 impactful quotes)
    *   Closing Statements
    *   Briefing Document (Summary)
    *   Key Themes and Ideas
    *   Notable Quotes with Context
    *   FAQ Section (10 questions)
    *   Quiz Questions (10 questions)
    *   Quiz Answer Key
    *   Essay Questions (5-7 questions)
    *   Essay Answers
    *   Speaker Bio
*   **Pharmaceutical Company Reference Removal:**  Automatically detects and redacts mentions of pharmaceutical companies to maintain generality and avoid bias.  This is a crucial feature for creating broadly applicable educational content.
*   **Download Options:** Allows users to download the processed output in Markdown, Word (.docx), and PDF formats.
*   **Progress Tracking:**  Provides visual feedback on the processing status with a progress bar and status messages.
* **Session Reset:** Includes a reset button in the sidebar to clear the session state and start fresh.
* **Dynamic Filename Generation**: Incorporates the date, extracted from the input filename if available, into the output filenames. If no date is found, it uses the current date.
* **Preview and Raw Markdown:** Displays results in two tabs: one for a formatted preview and another showing the raw Markdown code.

## Installation and Setup

1.  **Clone the repository:**

    ```bash
    git clone <repository_url>
    cd <repository_directory>
    ```

2.  **Install dependencies:**

    ```bash
    pip install -r requirements.txt
    ```
    The `requirements.txt` file should contain (at least) the following:

    ```
    streamlit
    crewai
    python-docx
    pypdf2
    xhtml2pdf
    markdown2
    ```

3.  **Obtain a Google Gemini API Key:** You'll need an API key for Google's Gemini model.  You can get one from the Google Cloud Console.

4.  **Run the application:**

    ```bash
    streamlit run transcript_processor.py  # Replace transcript_processor.py with the actual filename
    ```

## Usage

1.  **Enter API Key:** Once the Streamlit application is running, enter your Gemini API key in the sidebar input field.
2.  **Enter Speaker Name:** Provide the name of the lecturer or speaker.
3.  **Upload Transcripts:** Upload one or more transcript files using the file uploader.  Supported formats are `.txt`, `.pdf`, `.docx`, and `.vtt`.
4.  **Process Transcripts:** Click the "Process Transcripts" button.  The application will display a progress bar and status messages while processing.
5.  **Download Results:** Once processing is complete, the processed document will be displayed.  You can download it in Markdown, Word, or PDF format using the provided buttons.
6. **Reset Session (Optional):** If you want to process new transcripts, click the "Reset Session" button in the sidebar to clear the previous results.

## Code Structure and Explanation

The code is organized into several key components:

*   **File Reading Functions:**  `read_pdf`, `read_docx`, `read_vtt`, `read_txt`, and `read_file` handle reading and extracting text from different file types.  The `read_file` function acts as a dispatcher, selecting the appropriate reading function based on the file extension.  These functions robustly handle both file paths (strings) and file-like objects (BytesIO), and seek to the beginning of file-like objects before processing.

*   **`TranscriptProcessor` Class:** This class encapsulates the core logic for processing transcripts using CrewAI.

    *   `__init__(self, api_key)`: Initializes the class with the Gemini API key and sets up the CrewAI agents.
    *   `setup_agents(self)`: Defines the three CrewAI agents:
        *   `content_analyzer`: Analyzes the transcript and extracts initial information (title, speaker, key quotes, closing statements).
        *   `quote_extractor`: Extracts key themes, creates a briefing document, and identifies notable quotes with context.
        *   `content_writer`:  Creates the final structured document, assembling all the sections in the required order. This agent has detailed instructions to handle the formatting and organization of the output.
    *   `process_transcript(self, transcript_text, speaker_name, progress_bar, status_text)`: This is the main method that orchestrates the transcript processing. It defines the CrewAI tasks, creates the `Crew`, and runs the process. It also includes:
        *   **Progress Updates:**  Updates the Streamlit progress bar and status text to provide feedback to the user.
        *   **Error Handling:**  Uses `try...except` blocks to catch and report errors during processing.
        *   **Pharmaceutical Reference Removal:** Post-processes the output to remove any remaining mentions of "Pfizer".
        *   **Detailed Task Descriptions:** Each task has a very specific `description` and `expected_output` to guide the LLM. This is crucial for achieving the desired results. The descriptions explicitly instruct the agents *not* to include any pharmaceutical company references.

*   **Markdown to PDF/DOCX Conversion:** The `markdown_to_pdf` and `markdown_to_docx` functions convert the generated Markdown output to PDF and Word documents, respectively, handling basic styling. They utilize `xhtml2pdf` (for PDF) and `python-docx` (for Word). The DOCX conversion includes specific styling rules for different text elements (titles, headers, quotes, lists, etc.) to ensure a well-formatted output document.

*   **Streamlit UI:** The Streamlit code provides the user interface, including input fields for the API key and speaker name, a file uploader, a process button, progress indicators, and download buttons for the output. It also includes error handling to display messages to the user if any issues occur. The use of `st.session_state` ensures that the processed result persists across reruns.

## Mermaid Diagrams

### Process Flow

```mermaid
graph TD
    A[Start] --> B{User Uploads Transcript(s)};
    B -- Yes --> C[Read File(s)];
    B -- No --> J[End];
    C --> D{Determine File Type};
    D -- .pdf --> E[Read PDF];
    D -- .docx --> F[Read DOCX];
    D -- .txt --> G[Read TXT];
    D -- .vtt --> H[Read VTT];
    E --> I[Combine Text];
    F --> I;
    G --> I;
    H --> I;
    I --> K[Initialize TranscriptProcessor];
    K --> L[Create CrewAI Agents];
    L --> M[Define CrewAI Tasks];
    M --> N[Create Crew];
    N --> O[Run Crew (Sequential Process)];
    O --> P[Post-process (Remove Pharmaceutical References)];
    P --> Q[Display Results];
    Q --> R[Download Options (MD, DOCX, PDF)];
     R --> J;
