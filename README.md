# 🎓 Examiner AI - Intelligent PDF Document Examiner

An AI-powered chatbot that acts as an examiner for your PDF documents. Upload a project proposal, research paper, or any document, and the AI will ask intelligent questions about it, evaluate your answers with marks out of 10, and provide constructive feedback with a comprehensive report.

## ✨ Features

- **📄 PDF Analysis**: Upload and analyze PDF documents automatically with dual extraction strategy
- **🎯 Customizable Questions**: Select 1-10 questions per examination session
- **🤖 Multi-Model AI**: Automatic fallback system with 5 different Gemini models for reliability
- **📊 Scoring System**: Each answer is marked out of 10 with detailed feedback
- **🎓 Pass/Fail Grading**: Automatic evaluation with 50% passing threshold
- **🔄 Retry Mechanism**: Automatic retry button for rate limit and API errors
- **💡 Lifeline System**: 20% of questions can be rephrased or replaced
- **📑 PDF Export**: Generate comprehensive examination reports with meaningful filenames
- **🎨 Beautiful Interface**: Clean Gradio-based chat interface with real-time model display
- **⚡ Powered by Google Gemini**: Multi-model setup with intelligent failover

## 🚀 Quick Start

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/yousuf-git/examiner-ai.git
   cd examiner-ai
   ```

2. **Set up environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. **Configure API Key**
   ```bash
   cp .env.example .env
   # Edit .env and add your Gemini API key
   ```

4. **Run the application**
   ```bash
   python app.py
   ```

5. **Open your browser** at `http://localhost:7860`

### Docker Deployment

```bash
docker build -t examiner-ai .
docker run -p 7860:7860 --env-file .env examiner-ai
```

## 🌐 Deploy to Hugging Face Spaces

1. Create a new Space on [Hugging Face](https://huggingface.co/spaces)
2. Select "Docker" as the SDK
3. Clone your Space repository
4. Copy all files to the Space repository
5. Add your `GEMINI_API_KEY` as a Space secret
6. Push to the repository:
   ```bash
   git add .
   git commit -m "Initial commit"
   git push
   ```

## 📋 Requirements

- Python 3.9+
- Google Gemini API Key ([Get it here](https://makersuite.google.com/app/apikey))

## 🏗️ Project Structure

```
examiner-ai/
├── app.py                  # Main Gradio application with UI
├── examiner_logic.py       # AI logic: Q&A, evaluation, multi-model fallback
├── pdf_handler.py          # PDF extraction with dual-library strategy
├── requirements.txt        # Python dependencies (includes reportlab)
├── Dockerfile             # Docker configuration for deployment
├── .env.example           # Environment template
├── README.md             # Main documentation
├── ARCHITECTURE.md       # Detailed architecture guide
└── IMPROVEMENTS.md       # Feature changelog
```

## 🎨 UI Features

- **Real-time Model Display**: Shows which Gemini model is currently active
- **Lifelines Counter**: Live tracking of remaining/total lifelines
- **Error Notifications**: Clear, non-intrusive error messages with retry button
- **Question Progress**: Track current question out of total
- **Export Button**: One-click PDF report generation
- **Responsive Design**: Clean, professional interface

## 🔄 Error Handling

The system includes robust error handling:

- **Rate Limit Errors**: Automatic model fallback + retry button
- **API Failures**: Clear error messages with retry option
- **Network Issues**: Graceful degradation with user feedback
- **PDF Processing Errors**: Detailed error messages for troubleshooting

When errors occur, a retry button appears automatically, allowing you to retry the last action without losing your progress.

## 🎯 How It Works

1. **Upload PDF**: User uploads a document (e.g., project proposal)
2. **Select Questions**: Choose 1-10 questions for the examination
3. **Analysis**: AI analyzes the document content using Gemini 2.5 Flash
4. **Interactive Q&A**: AI asks questions about:
   - Problem statement and motivation
   - Project scope and boundaries
   - Objectives and expected outcomes
   - Methodology and approach
   - Innovation, feasibility, and impact
5. **Lifelines Available**: Use 20% of questions as lifelines to:
   - 🔄 Rephrase unclear questions
   - 🆕 Get completely different questions
6. **Scoring**: Each answer receives:
   - Marks out of 10
   - Detailed feedback with scoring criteria
   - Constructive suggestions for improvement
7. **Retry on Errors**: If rate limits or errors occur, use the retry button
8. **Final Results**: 
   - Overall evaluation using premium Gemini 2.5 Pro model
   - Total marks and percentage
   - Pass/Fail status (50% threshold)
9. **Export Report**: Download comprehensive PDF report with:
   - Document title from filename
   - Meaningful filename with timestamp
   - Overall evaluation section
   - Question-wise performance breakdown
   - Lifelines usage statistics

## 🏆 Grading System

- **Each Question**: Marked out of 10 based on:
  - Relevance to question (3 points)
  - Depth of understanding (3 points)
  - Use of document content (2 points)
  - Clarity of expression (2 points)

- **Final Grade**:
  - **50%+ = PASS ✅**
  - **Below 50% = FAIL ❌**

- **Lifelines**: 20% of total questions (minimum 1)
  - Rephrase difficult questions
  - Request completely new questions

## 🤖 AI Models Used

The system uses multiple Gemini models for optimal performance:

| Model | Usage | Rate Limit |
|-------|-------|------------|
| **gemini-2.5-flash** | Primary Q&A | 10 RPM |
| **gemini-2.0-flash-lite** | Fallback 1 | 30 RPM |
| **gemini-2.5-flash-lite** | Fallback 2 | 15 RPM |
| **gemini-2.0-flash** | Fallback 3 | 15 RPM |
| **gemini-2.5-pro** | Final Evaluation | 3 RPM |

*Automatic fallback ensures high availability even during peak usage*

## 📝 License

MIT License - Feel free to use and modify!

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
