# AI Drive-Through Assistant

An AI-powered drive-through ordering system built with Streamlit and efficient small language models. Uses a **Model Context Protocol (MCP)** approach for reliable, deterministic state management. Designed for local deployment at restaurants with minimal computational requirements.

## Features

- 🤖 AI-powered order management using efficient models (default: Meta Llama 3.2 3B)
- 🔄 Model Context Protocol (MCP) for reliable state management
- 💬 Chat-style interface for natural ordering
- 📋 Real-time menu display
- 🛒 Live order summary with total calculation
- 📜 Order history tracking
- 🗑️ Clear order functionality
- 🎙️ Ready for voice integration (speech-to-text + text-to-speech)

## Setup

1. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

2. **Get an OpenRouter API key (for cloud models):**
   - Sign up at [OpenRouter](https://openrouter.ai/)
   - Get your API key from the dashboard
   - Alternatively, set up a local model server (see Local Deployment section)

3. **Run the Streamlit app:**
   ```bash
   streamlit run streamlit_app.py
   ```

## Usage

1. Open the app in your browser (usually `http://localhost:8501`)
2. Enter your OpenRouter API key in the sidebar (or configure local model endpoint)
3. Select a model (default: `meta-llama/llama-3.2-3b-instruct:free` - efficient and free)
4. Click "Initialize Assistant"
5. Start chatting to place your order!

## Example Interactions

- "I'll take a Big Mac and 2 Large Fries"
- "Actually, make that two Big Macs and remove one fry"
- "Change the remaining fry to a Coke"

## MCP (Model Context Protocol) Approach

This system implements a **Model Context Protocol (MCP)** architecture, which is a structured state management pattern for LLM interactions. Instead of treating the conversation as a free-form chat, this approach treats each interaction as a state update operation.

### How It Works:

1. **Context Injection**: Every API call includes the current state directly in the system prompt:
   - Complete menu with prices
   - Current order state (items and quantities)
   - User's latest request

2. **State Update Pattern**: The model receives:
   ```
   MENU: {all available items}
   CURRENT ORDER STATE: {current order}
   USER REQUEST: "add 2 Big Macs"
   ```

3. **Structured Output**: The model returns:
   - A conversational message for the user
   - The NEW complete order state as JSON

4. **State Persistence**: The system maintains a single source of truth (the order state) that gets updated with each interaction.

### Why MCP Approach?

**Advantages:**

1. **Deterministic State Management**: 
   - No ambiguity about what's in the order
   - The model always sees the complete current state
   - Eliminates conversational drift or forgetting

2. **Error Recovery**:
   - If the model makes an error, the next request sees the actual current state
   - Self-correcting: customer can say "remove that" and the model sees what "that" is

3. **Small Context Windows**:
   - Only the last few turns needed in conversation history
   - Most context is the explicit state (menu + order), not chat history
   - Perfect for smaller models like Llama 3.2 3B

4. **Reliability**:
   - Each response validates against the menu
   - Invalid items are automatically filtered
   - State is always valid

5. **Task-Specific Optimization**:
   - The model knows exactly what to do (update state)
   - Clear input format (state) → clear output format (new state)
   - No need for complex reasoning chains

### MCP vs Traditional Chat Approaches:

| Traditional Chat | MCP Approach |
|------------------|--------------|
| Conversation history only | Explicit state + minimal history |
| Model must remember state | State always provided |
| Can lose track of orders | State is always accurate |
| Requires large context | Small, focused context |
| General-purpose reasoning | Task-specific state updates |

### Implementation Details:

The system prompt dynamically injects the current order state:

```python
def get_system_prompt(self):
    return f"""
    MENU: {json.dumps(self.menu, indent=2)}
    CURRENT ORDER STATE: {json.dumps(self.current_order, indent=2)}
    
    INSTRUCTIONS:
    1. Update the 'CURRENT ORDER STATE' based on the user's new message.
    2. Output the NEW FINAL ORDER STATE as a JSON object.
    """
```

This ensures every interaction has complete context, making it perfect for small, efficient models that don't have large context windows or advanced reasoning capabilities.

## Why Small, Efficient Models?

This system uses lightweight models like **Meta Llama 3.2 3B** instead of large, expensive LLMs for several key reasons:

### Advantages Over Traditional LLMs:

1. **Cost-Effective**: Small models like Llama 3.2 3B are significantly cheaper to run (even free via OpenRouter), making them viable for small businesses and high-volume operations.

2. **Low Latency**: Faster response times crucial for drive-through scenarios where customers expect quick service.

3. **Local Deployment**: Can run entirely on-premises on modest hardware (even a single GPU or CPU-only setup), ensuring:
   - **Data Privacy**: Order data never leaves the restaurant
   - **No Internet Dependency**: Works even during internet outages
   - **Predictable Costs**: No per-request API fees

4. **Specialized Task**: Drive-through ordering is a constrained domain - you don't need the full reasoning capabilities of large models. Smaller models excel at:
   - Menu item recognition
   - Quantity parsing
   - Order state management
   - Simple substitutions

5. **Scalability**: Multiple terminals can run simultaneously without expensive API rate limits or costs.

## Local Deployment at Restaurants

### Setup Options:

**Option 1: Cloud (Current Implementation)**
- Uses OpenRouter API for model inference
- Requires internet connection
- Zero setup, works immediately
- Best for: Testing, small operations, temporary deployments

**Option 2: Local Model Server (Recommended for Production)**
- Deploy Llama 3.2 3B on local hardware (GPU recommended, but CPU works)
- Use tools like:
  - [Ollama](https://ollama.ai/) - Easy local LLM deployment
  - [llama.cpp](https://github.com/ggerganov/llama.cpp) - CPU-optimized inference
  - [vLLM](https://github.com/vllm-project/vllm) - Fast GPU inference
- Modify `drive_through_assistant.py` to point to local endpoint instead of OpenRouter
- Benefits:
  - No ongoing API costs
  - Complete data privacy
  - Offline operation
  - Multiple terminals, single hardware investment

**Hardware Requirements:**
- **Minimum**: CPU-only setup (slower but functional)
- **Recommended**: Single NVIDIA GPU (GTX 1660+ or RTX 3060+)
- **Ideal**: GPU server supporting multiple concurrent terminals

### Voice Integration Potential:

The system is designed to easily integrate with voice input/output:

1. **Speech-to-Text**: Add real-time transcription using:
   - Google Speech-to-Text API
   - Azure Speech Services
   - Local solutions: Whisper (OpenAI) or Whisper.cpp

2. **Text-to-Speech**: Convert assistant responses to voice:
   - Google Text-to-Speech
   - Azure Neural Voices
   - Local TTS engines: Coqui TTS, Festival

3. **Integration Points**:
   - Replace chat input with microphone input
   - Stream audio responses instead of text
   - Add push-to-talk buttons for drive-through windows

**Example Integration:**
```python
# Add to streamlit_app.py
import speech_recognition as sr

# Voice input option
if st.button("🎙️ Speak Order"):
    # Record audio, transcribe, then process
    pass
```

## Configuration

You can modify the menu by editing the `DEFAULT_MENU` dictionary in `streamlit_app.py`.

## Requirements

- Python 3.8+
- OpenRouter API key (for cloud deployment) OR local model server (for on-premises)
- Internet connection (only if using cloud API)

