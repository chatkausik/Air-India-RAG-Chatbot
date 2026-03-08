# Air India RAG Chatbot

A Retrieval-Augmented Generation (RAG) chatbot designed to provide intelligent assistance for Air India customer queries using advanced AI technologies.

## 🚀 Overview

This project implements a conversational AI chatbot that combines information retrieval with large language models to answer questions about Air India services, policies, flight information, and customer support.

## ✨ Features

- **Intelligent Query Understanding**: Natural language processing for customer queries
- **Context-Aware Responses**: RAG architecture for accurate, contextual answers
- **Real-time Information Retrieval**: Access to Air India's knowledge base
- **Conversational Interface**: Smooth, human-like interactions
- **Scalable Architecture**: Built with modern Python frameworks

## 🛠️ Technology Stack

- **Language**: Python 3.x
- **AI/ML**: RAG (Retrieval-Augmented Generation)
- **Development Tools**: 
  - Ruff (linting)
  - Marimo (interactive notebooks)
  - Cursor (AI-powered code editor)
  - Abstra (process automation)

## 📋 Prerequisites

- Python 3.8 or higher
- pip or uv package manager
- Virtual environment manager (venv/poetry/pdm)

## 🔧 Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/chatkausik/Air-India-RAG-Chatbot.git
   cd Air-India-RAG-Chatbot
   ```

2. **Create a virtual environment**
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your API keys and configuration
   ```

## 🚦 Usage

### Running the Chatbot

```bash
python main.py
```

### Interactive Development

If using Marimo notebooks:
```bash
marimo run notebook.py
```

## 📁 Project Structure

```
Air-India-RAG-Chatbot/
├── .env                 # Environment variables (not in git)
├── .gitignore          # Git ignore rules
├── requirements.txt    # Python dependencies
├── main.py            # Main application entry point
├── src/               # Source code
│   ├── chatbot/       # Chatbot logic
│   ├── retrieval/     # RAG retrieval system
│   └── utils/         # Utility functions
└── data/              # Knowledge base and datasets
```

## 🔑 Environment Variables

Create a `.env` file with the following variables:

```env
# API Keys
OPENAI_API_KEY=your_api_key_here
# Add other required environment variables
```

## 🧪 Testing

```bash
pytest tests/
```

## 📝 Configuration

Configuration settings can be modified in the `config.py` file or through environment variables.

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👥 Authors

- Your Name - *Initial work*

## 🙏 Acknowledgments

- Air India for domain knowledge
- OpenAI/Anthropic for LLM capabilities
- The open-source community

## 📧 Contact

For questions or support, please open an issue on GitHub or contact [your-email@example.com]

## 🗺️ Roadmap

- [ ] Multi-language support
- [ ] Voice interface integration
- [ ] Real-time flight status integration
- [ ] Enhanced context window management
- [ ] Performance optimization

---

**Note**: This is an AI-powered assistant. Always verify critical information through official Air India channels.
