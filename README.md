# AI Drive-Through Assistant

An AI-powered drive-through ordering system using efficient small language models (default: Meta Llama 3.2 3B). Implements **Model Context Protocol (MCP)** for reliable state management. Designed for local deployment at restaurants with minimal hardware requirements.

## Features

- 🤖 AI-powered order management using efficient models
- 🔄 Model Context Protocol (MCP) for deterministic state management
- 💬 Chat-style interface for natural ordering
- 📋 Real-time menu display with order summary
- 🎙️ Ready for voice integration (speech-to-text + text-to-speech)

## Setup

1. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

2. **Get OpenRouter API key** (or set up local model server):
   - Sign up at [OpenRouter](https://openrouter.ai/) for cloud deployment
   - Or use local tools like [Ollama](https://ollama.ai/), [llama.cpp](https://github.com/ggerganov/llama.cpp), or [vLLM](https://github.com/vllm-project/vllm)

3. **Run the Streamlit app:**
   ```bash
   streamlit run streamlit_app.py
   ```

## Usage

1. Open `http://localhost:8501` in your browser
2. Enter your OpenRouter API key (or configure local endpoint)
3. Select a model (default: `meta-llama/llama-3.2-3b-instruct:free`)
4. Click "Initialize Assistant" and start ordering!

**Example Interactions:**
- "I'll take a Big Mac and 2 Large Fries"
- "Actually, make that two Big Macs and remove one fry"
- "Change the remaining fry to a Coke"

## MCP (Model Context Protocol) Approach

This system uses a **Model Context Protocol** where each interaction is treated as a state update operation rather than free-form chat.

**How it works:**
- The system injects complete context (menu + current order state) into every API call
- The model receives explicit state and returns the updated state as JSON
- This eliminates conversational drift and ensures state accuracy

**Why MCP?**
- **Deterministic**: Model always sees complete current state, no ambiguity
- **Error Recovery**: Next request sees actual state, self-correcting
- **Small Context Windows**: Perfect for efficient models like Llama 3.2 3B
- **Reliability**: State validation against menu, always valid
- **Task-Specific**: Optimized for state updates, not general reasoning

**MCP vs Traditional Chat:**
| Traditional Chat | MCP Approach |
|------------------|--------------|
| Conversation history only | Explicit state + minimal history |
| Model must remember state | State always provided |
| Large context required | Small, focused context |

## Why Small, Efficient Models?

**Advantages:**
1. **Cost-Effective**: Free via OpenRouter, viable for small businesses
2. **Low Latency**: Fast responses crucial for drive-through service
3. **Local Deployment**: Can run on-premises (CPU or GPU) ensuring:
   - Data privacy (order data never leaves restaurant)
   - Offline operation (no internet dependency)
   - Predictable costs (no per-request fees)
4. **Perfect Fit**: Drive-through ordering is constrained - small models excel at menu recognition, quantity parsing, and state management
5. **Scalable**: Multiple terminals without expensive API limits

## Local Deployment

**Cloud (Current)**: Uses OpenRouter API - zero setup, internet required  
**Local (Recommended for Production)**: Deploy Llama 3.2 3B on local hardware

**Hardware Requirements:**
- **Minimum**: CPU-only (functional but slower)
- **Recommended**: Single NVIDIA GPU (GTX 1660+ or RTX 3060+)
- **Ideal**: GPU server for multiple concurrent terminals

**Benefits**: No API costs, complete data privacy, offline operation

## Voice Integration

Designed for easy voice integration:
- **Speech-to-Text**: Google/Azure APIs or local Whisper
- **Text-to-Speech**: Google/Azure TTS or local Coqui TTS
- Replace chat input with microphone, stream audio responses

## Configuration

Modify the menu by editing `DEFAULT_MENU` in `streamlit_app.py`.

## Requirements

- Python 3.8+
- OpenRouter API key (cloud) OR local model server (on-premises)
- Internet connection (only if using cloud API)
