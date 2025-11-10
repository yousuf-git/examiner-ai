# 🏗️ Examiner AI - Architecture & Deployment Guide

## 📐 System Architecture

### Overview
Examiner AI is a modular, AI-powered document examination system with intelligent scoring, lifeline support, and comprehensive reporting. The architecture follows clean separation of concerns with three main components and robust error handling.

```
┌─────────────────────────────────────────────────────────────┐
│                        User Interface                        │
│                  (Gradio Web App + PDF Export)               │
│  • Question Selector (1-10)                                  │
│  • Lifeline Buttons (Rephrase/New)                          │
│  • Retry Button (Auto-shown on errors)                      │
│  • Model Display (Real-time)                                │
│  • PDF Report Export (ReportLab)                            │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     Application Layer                        │
│                         (app.py)                             │
│  • Session Management                                        │
│  • UI Event Handling                                         │
│  • State Coordination                                        │
│  • Error Display & Retry Logic                              │
│  • PDF Report Generation (ReportLab)                        │
│  • Lifeline Management                                       │
└──────────────┬────────────────────────┬─────────────────────┘
               │                        │
               ▼                        ▼
┌──────────────────────────┐  ┌──────────────────────────────┐
│    PDF Handler Module    │  │   Examiner Logic Module      │
│   (pdf_handler.py)       │  │  (examiner_logic.py)         │
│                          │  │                              │
│  • Text Extraction       │  │  • Document Analysis         │
│  • Metadata Parsing      │  │  • Question Generation       │
│  • Content Validation    │  │  • Answer Evaluation (Marks) │
│  • Filename Extraction   │  │  • Multi-Model Fallback      │
│                          │  │  • Lifeline Handling         │
│  Libraries:              │  │  • Final Evaluation (50%)    │
│  - PyMuPDF (primary)     │  │                              │
│  - pdfplumber (fallback) │  │  AI Models (5 models):       │
│                          │  │  - gemini-2.5-flash (10 RPM) │
│                          │  │  - gemini-2.0-flash-lite (30)│
│                          │  │  - gemini-2.5-flash-lite (15)│
│                          │  │  - gemini-2.0-flash (15 RPM) │
│                          │  │  - gemini-2.5-pro (3 RPM)    │
└──────────────────────────┘  └──────────────────────────────┘
```

## 🔧 Component Breakdown

### 1. **PDF Handler (`pdf_handler.py`)**
**Purpose:** Handles all PDF-related operations

**Key Features:**
- Dual extraction strategy (PyMuPDF + pdfplumber fallback)
- Metadata extraction (page count, title, author)
- Content validation
- Text quality assessment

**Class: `PDFHandler`**
```python
Methods:
- extract_text(pdf_path) → str
- get_summary() → Dict
- validate_content() → bool
```

**How it works:**
1. Receives PDF file path from UI
2. Attempts extraction with PyMuPDF (fast)
3. Falls back to pdfplumber if needed (robust)
4. Returns structured text with page markers
5. Validates content length and quality

### 2. **Examiner Logic (`examiner_logic.py`)**
**Purpose:** Core AI examination functionality

**Key Features:**
- Document analysis using Gemini AI
- Dynamic question generation
- Intelligent answer evaluation
- Conversation state management
- Final summary generation

**Class: `ExaminerAI`**
```python
Methods:
- analyze_document(text, title) → (str, error)
- generate_next_question() → (str, error)
- evaluate_answer(answer) → (str, error)
- generate_final_summary() → (str, error)
- set_total_questions(total: int)
- use_lifeline(type: str) → bool
- get_lifelines_status() → (remaining, total)
- get_current_model() → str
- is_examination_complete() → bool
- reset_state()
```

**Class: `ConversationState`**
```python
Attributes:
- document_text
- document_title
- questions_asked
- answers_given
- evaluations
- marks (out of 10)
- current_question_index
- total_questions (customizable: 1-10)
- lifelines_total (20% of questions)
- lifelines_remaining
- lifelines_used
- awaiting_lifeline_response
- final_evaluation
```

**Multi-Model System:**
```python
Primary Model: gemini-2.5-flash (10 RPM)
Fallbacks:
  1. gemini-2.0-flash-lite (30 RPM - fastest)
  2. gemini-2.5-flash-lite (15 RPM)
  3. gemini-2.0-flash (15 RPM)
Premium Model: gemini-2.5-pro (3 RPM - for final evaluation)
```

**How it works:**
1. Analyzes uploaded document to understand context
2. Generates questions focusing on:
   - Problem statement and motivation
   - Scope and boundaries
   - Objectives and expected outcomes
   - Methodology and approach
   - Innovation, feasibility, and impact
3. Evaluates each answer with marks out of 10 based on:
   - Relevance (3 points)
   - Depth (3 points)
   - Document usage (2 points)
   - Clarity (2 points)
4. Handles lifeline requests:
   - Rephrase: Simplifies current question
   - New: Generates different question on different topic
5. Automatic model fallback on rate limits
6. Maintains conversation context
7. Produces final evaluation with 50% passing threshold using premium model

### 3. **Application Layer (`app.py`)**
**Purpose:** User interface and orchestration

**Key Features:**
- Gradio-based web interface
- Session management
- Event handling
- Chat history management
- Beautiful, responsive UI
- Real-time model display
- Lifelines counter display
- Error notification with retry button
- PDF report generation (ReportLab)

**Main Functions:**
```python
- initialize_app() → str
- process_pdf(file, num_questions) → (status, chat, error, model, lifelines)
- chat_with_examiner(msg, history) → (history, input, error, model, lifelines, show_retry)
- use_lifeline(type, history) → (history, error, model, lifelines)
- retry_last_action(msg, history) → (history, input, error, model, lifelines, show_retry)
- reset_session() → (status, history, input, error, model, lifelines, show_retry)
- export_report() → (file_path, error)
```

**UI Components:**
- PDF upload area
- Number of questions slider (1-10)
- Status display with document info
- Model indicator (real-time)
- Lifelines status display
- Error notification banner
- Retry button (auto-shown on errors)
- Interactive chatbot
- Lifeline buttons (Rephrase/New)
- Progress tracking
- Export report button
- Reset functionality

**PDF Report Features (ReportLab):**
- Meaningful filename: `Examination_Report_{DocName}_{Timestamp}.pdf`
- Document information table
- Results summary with Pass/Fail status
- Overall evaluation section
- Question-wise performance breakdown
- Professional formatting with colors and tables

**How it works:**
1. Loads environment variables (.env)
2. Initializes Gemini API with multiple models
3. Sets up Gradio interface with custom theme and CSS
4. Handles file uploads and processing
5. Extracts filename for document title
6. Manages chat interactions with error handling
7. Tracks examination progress and lifelines
8. Displays current model being used
9. Shows retry button on errors
10. Generates PDF reports with ReportLab
11. Provides session reset capability

## 🔄 Data Flow

### Examination Flow:

```
1. User uploads PDF + selects questions (1-10)
        ↓
2. PDF Handler extracts text and filename
        ↓
3. Examiner analyzes document (gemini-2.5-flash)
        ↓
4. Lifelines calculated (20% of questions, min 1)
        ↓
5. First question generated
        ↓
6. User answers question
        ↓
7. Answer evaluated with marks out of 10
        ↓
8. Check if examination complete
        ↓
9. If not complete: Next question generated
        ↓
10. If rate limit: Auto-fallback to next model
        ↓
11. If error: Show retry button
        ↓
12. Repeat steps 6-11 until all questions answered
        ↓
13. Final summary with gemini-2.5-pro (premium)
        ↓
14. Calculate total marks and Pass/Fail (50%)
        ↓
15. User exports PDF report
        ↓
16. Session complete
```

### Lifeline Flow:

```
User clicks Rephrase/New button
        ↓
Check lifelines_remaining > 0
        ↓
If yes: Decrement lifelines_remaining
        ↓
Set awaiting_lifeline_response = True
        ↓
If "rephrase": Simplify last question
        ↓
If "new": Generate different question
        ↓
Update lifelines counter display
        ↓
Continue examination
```

### Error & Retry Flow:

```
API call fails (rate limit/error)
        ↓
Return error message to UI
        ↓
Show retry button (auto-visible)
        ↓
Display error in notification banner
        ↓
User clicks Retry
        ↓
Re-attempt last action with same input
        ↓
If success: Hide retry button
        ↓
If failure: Keep retry button visible
```

### State Management:

```python
Session State:
├── PDF Content (extracted text)
├── Document Title (from filename)
├── Document Analysis (AI summary)
├── Question History (list)
├── Answer History (list)
├── Evaluation History (list with marks)
├── Marks List (list of integers 0-10)
├── Current Question Index (int)
├── Total Questions (int: 1-10)
├── Lifelines Total (int: 20% of questions)
├── Lifelines Remaining (int)
├── Lifelines Used (list of tuples)
├── Current Model Name (string)
├── Final Evaluation (string)
├── Awaiting Lifeline Response (bool)
└── Session Active (bool)
```

## 🚀 Deployment Guide

### **Option 1: Local Development**

#### Prerequisites:
- Python 3.9 or higher
- Gemini API key

#### Steps:

1. **Clone/Download the project:**
```bash
cd examiner-ai
```

2. **Create virtual environment:**
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
# or
venv\Scripts\activate  # Windows
```

3. **Install dependencies:**
```bash
pip install -r requirements.txt
```

4. **Configure API key:**
```bash
cp .env.example .env
# Edit .env and add your Gemini API key:
# GEMINI_API_KEY=your_actual_api_key_here
```

5. **Run the application:**
```bash
python app.py
```

6. **Access the interface:**
Open your browser at `http://localhost:7860`

### **Option 2: Docker Deployment**

#### Prerequisites:
- Docker installed
- Gemini API key

#### Steps:

1. **Build the Docker image:**
```bash
docker build -t examiner-ai .
```

2. **Run the container:**
```bash
docker run -p 7860:7860 \
  -e GEMINI_API_KEY=your_api_key_here \
  examiner-ai
```

Or use an `.env` file:
```bash
docker run -p 7860:7860 --env-file .env examiner-ai
```

3. **Access the interface:**
Open `http://localhost:7860`

### **Option 3: Hugging Face Spaces (Recommended for Production)**

#### Prerequisites:
- Hugging Face account
- Git installed
- Gemini API key

#### Steps:

1. **Create a new Space:**
   - Go to https://huggingface.co/spaces
   - Click "Create new Space"
   - Name it (e.g., "examiner-ai")
   - Select "Docker" as the SDK
   - Choose "Public" or "Private"
   - Click "Create Space"

2. **Clone your Space repository:**
```bash
git clone https://huggingface.co/spaces/YOUR_USERNAME/examiner-ai
cd examiner-ai
```

3. **Copy all project files to the Space:**
```bash
cp /path/to/examiner-ai/* .
```

Files to include:
- `app.py`
- `pdf_handler.py`
- `examiner_logic.py`
- `requirements.txt`
- `Dockerfile`
- `README.md`

4. **Add your API key as a Space Secret:**
   - Go to your Space settings on Hugging Face
   - Navigate to "Settings" → "Variables and secrets"
   - Add a new secret:
     - Name: `GEMINI_API_KEY`
     - Value: Your actual Gemini API key
   - This is more secure than hardcoding in .env

5. **Commit and push:**
```bash
git add .
git commit -m "Initial commit: Examiner AI application"
git push
```

6. **Wait for build:**
   - Hugging Face will automatically build your Docker image
   - Monitor the build logs in the Space interface
   - Build typically takes 2-5 minutes

7. **Access your deployed app:**
   - Your app will be available at:
   - `https://huggingface.co/spaces/YOUR_USERNAME/examiner-ai`

#### **Hugging Face Spaces Configuration:**

The app is pre-configured for Spaces with:
- Port 7860 (Spaces default)
- Server name 0.0.0.0 (required for Spaces)
- Proper health checks
- Optimized Docker layers

#### **Environment Variables in Spaces:**

Set these in Space Settings → Variables and secrets:
- `GEMINI_API_KEY` (required) - Your Google Gemini API key

## 🔐 Security Best Practices

1. **Never commit API keys** to version control
2. **Use environment variables** for sensitive data
3. **Use Spaces Secrets** for production deployment
4. **Run container as non-root user** (already configured)
5. **Keep dependencies updated** regularly

## 📊 Performance Optimization

### Current Optimizations:
- **Multi-model fallback** system for high availability (5 models)
- **Gemini 2.5 Flash** model for faster Q&A responses (10 RPM)
- **Gemini 2.0 Flash Lite** for high-volume fallback (30 RPM)
- **Gemini 2.5 Pro** for premium final evaluation (3 RPM)
- **Text limiting** in prompts (first 2-3k chars for analysis)
- **Dual PDF extraction** with fallback strategy
- **Slim Docker image** for faster deployment
- **Layer caching** in Dockerfile
- **ReportLab PDF generation** for efficient report creation
- **Automatic retry mechanism** for transient failures

### Recommendations:
- For very large PDFs, consider chunking strategy
- Implement response caching for repeated questions
- Add rate limiting awareness for API calls
- Consider GPU instances for heavy usage
- Monitor model performance and switch priorities if needed

## 🐛 Troubleshooting

### Common Issues:

**1. API Key Error:**
```
Solution: Ensure GEMINI_API_KEY is set correctly in .env or Spaces Secrets
```

**2. Rate Limit Errors:**
```
Solution: System automatically falls back to alternative models
- Try clicking the Retry button that appears
- Wait a moment for rate limits to reset
- System uses 30 RPM fallback model automatically
```

**3. PDF Extraction Fails:**
```
Solution: Check if PDF has selectable text (not scanned images)
Consider adding OCR support for scanned documents
Ensure PDF is not password-protected
```

**4. Marks Not Showing:**
```
Solution: Evaluation uses specific format "**Marks: X/10**"
- Check examiner_logic.py evaluate_answer() method
- Regex extracts marks from AI response
- Default to 5/10 if format not found
```

**5. Lifelines Not Working:**
```
Solution: Check lifelines_remaining > 0
- Lifelines calculated as 20% of total questions
- Minimum 1 lifeline always available
- UI shows remaining/total count
```

**6. PDF Report Export Fails:**
```
Solution: Ensure reportlab is installed
- Check requirements.txt includes reportlab==4.0.7
- Verify temp directory is writable
- Check session has completed examination
```

**7. Docker Build Fails:**
```
Solution: Ensure all files are present
Check Docker daemon is running
Verify internet connection for dependency downloads
```

**8. Gradio Port Already in Use:**
```
Solution: Change port in app.py or kill process using port 7860
pkill -f "python.*app.py"
```

## 📈 Future Enhancements

Potential improvements:
- [ ] OCR support for scanned PDFs
- [ ] Multi-document comparison
- [✅] Export evaluation reports to PDF (COMPLETED)
- [✅] Custom question count selector (COMPLETED)
- [✅] Marks out of 10 system (COMPLETED)
- [✅] Lifeline system (COMPLETED)
- [✅] Retry mechanism (COMPLETED)
- [✅] Multi-model fallback (COMPLETED)
- [ ] Custom question templates
- [ ] Multi-language support
- [ ] Voice input/output
- [ ] Integration with learning management systems
- [ ] Historical performance tracking
- [ ] Answer comparison with reference answers
- [ ] Plagiarism detection
- [ ] Collaborative examination mode

## 📚 Dependencies Explained

- **gradio==4.19.2**: Web UI framework (intentionally older version for HuggingFace Hub compatibility)
- **google-generativeai==0.8.2**: Gemini AI SDK for multiple model support
- **PyMuPDF==1.24.10**: Fast PDF text extraction (primary)
- **pdfplumber==0.11.4**: Robust PDF parsing (fallback)
- **python-dotenv==1.0.1**: Environment variable management
- **reportlab==4.0.7**: PDF report generation for examination results

## 📞 Support

For issues or questions:
- Check the README.md for quick start guide
- Review IMPROVEMENTS.md for feature changelog
- Review error messages in logs and UI
- Verify API key is valid
- Ensure Python version is 3.9+
- Check that all dependencies are installed
- Use the retry button for transient errors
- Monitor model display for fallback information

---

**Built with ❤️ using Gradio and Google Gemini AI**
**Latest Update: Multi-model system with 5 Gemini models, scoring system, lifelines, retry mechanism, and PDF export**
