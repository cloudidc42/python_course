# Part 84: LLM Integration - Anthropic API & OpenAI

## สารบัญ
1. [LLM Concepts](#llm-concepts)
2. [Anthropic Claude API](#anthropic-api)
3. [anthropic Python SDK](#anthropic-sdk)
4. [Messages API](#messages-api)
5. [Streaming Responses](#streaming)
6. [System Prompts](#system-prompts)
7. [Prompt Engineering](#prompt-engineering)
8. [Tool Use - Function Calling](#tool-use)
9. [Vision Capabilities](#vision)
10. [OpenAI API Comparison](#openai)
11. [LangChain Framework](#langchain)
12. [Vector Databases](#vector-db)
13. [RAG - Retrieval Augmented Generation](#rag)
14. [Building AI Applications](#ai-apps)
15. [ตัวอย่างโปรแกรมจริง](#real-examples)
16. [แบบฝึกหัด](#exercises)

---

## 1. LLM Concepts {#llm-concepts}

### LLM คืออะไร

Large Language Models (LLMs) เป็น neural network ขนาดใหญ่ที่ train บนข้อมูล text จำนวนมาก เพื่อเรียนรู้การทำนาย token ถัดไป

### Key Concepts

```python
# ตัวอย่างที่ 1: LLM fundamental concepts

# 1. Tokens
# Token = smallest unit of text an LLM processes
# "Hello world" ≈ 2 tokens
# "antidisestablishmentarianism" ≈ 6 tokens
# Thai text is more tokens per character

token_examples = {
    "Hello": 1,
    "Hello world": 2,
    "Hello, world!": 3,
    "Python programming language": 4,
    "สวัสดีครับ": 5,  # Thai uses more tokens
}

print("Token approximations:")
for text, approx in token_examples.items():
    chars = len(text)
    print(f"  '{text}' ~ {approx} tokens ({chars} chars)")

# Pricing is usually per token
cost_per_million_tokens = {
    "claude-3-5-sonnet-20241022": {"input": 3.0, "output": 15.0},
    "claude-3-haiku-20240307": {"input": 0.25, "output": 1.25},
    "gpt-4o": {"input": 2.5, "output": 10.0},
    "gpt-4o-mini": {"input": 0.15, "output": 0.60},
}

print("\nCost per million tokens (USD):")
for model, costs in cost_per_million_tokens.items():
    print(f"  {model}: input=${costs['input']}, output=${costs['output']}")

# 2. Context Window
# Maximum tokens the model can process at once
context_windows = {
    "claude-3-5-sonnet-20241022": 200_000,
    "claude-3-haiku-20240307": 200_000,
    "gpt-4o": 128_000,
    "gpt-3.5-turbo": 16_000,
}

print("\nContext windows (tokens):")
for model, ctx in context_windows.items():
    pages = ctx // 500  # ~500 tokens per page
    print(f"  {model}: {ctx:,} tokens (~{pages} pages)")
```

```python
# ตัวอย่างที่ 2: Temperature and sampling parameters

import math
import random

def temperature_demo(logits, temperatures):
    """Demonstrate effect of temperature on token selection"""
    
    # Softmax with temperature
    def softmax_temp(x, T=1.0):
        x_scaled = [v / T for v in x]
        max_x = max(x_scaled)
        exp_x = [math.exp(v - max_x) for v in x_scaled]
        sum_exp = sum(exp_x)
        return [v / sum_exp for v in exp_x]
    
    words = ['cat', 'dog', 'bird', 'fish', 'snake']
    
    print("Token probabilities at different temperatures:")
    print(f"{'Word':10}", end="")
    for T in temperatures:
        print(f"  T={T}", end="")
    print()
    print("-" * 50)
    
    for T in temperatures:
        probs = softmax_temp(logits, T)
        for word, prob in zip(words, probs):
            pass  # Just storing
    
    for i, word in enumerate(words):
        print(f"{word:10}", end="")
        for T in temperatures:
            probs = softmax_temp(logits, T)
            print(f"  {probs[i]:.3f}", end="")
        print()

# Raw logits
logits = [3.0, 2.0, 1.5, 1.0, 0.5]
temperatures = [0.1, 0.5, 1.0, 2.0]
temperature_demo(logits, temperatures)

print("\nTemperature guidelines:")
guidelines = {
    "0.0": "Deterministic (always pick top token)",
    "0.2": "Very focused, less creative",
    "0.5": "Balanced (good for most tasks)",
    "0.7-1.0": "Creative writing, brainstorming",
    "1.5+": "Very random, diverse",
}
for temp, desc in guidelines.items():
    print(f"  T={temp}: {desc}")
```

```python
# ตัวอย่างที่ 3: Top-p and Top-k sampling

def top_p_sampling_demo(probs, p_values):
    """Demonstrate Top-p (nucleus) sampling"""
    words = ['hello', 'hi', 'hey', 'greetings', 'howdy', 'yo', 'sup']
    
    print("Top-p sampling (nucleus sampling):")
    print("At each p, only use tokens within top p cumulative probability")
    print()
    
    # Sort by probability
    sorted_items = sorted(zip(words, probs), key=lambda x: -x[1])
    sorted_words = [x[0] for x in sorted_items]
    sorted_probs = [x[1] for x in sorted_items]
    
    cumulative = [sum(sorted_probs[:i+1]) for i in range(len(sorted_probs))]
    
    print("Token distribution:")
    for word, prob, cum in zip(sorted_words, sorted_probs, cumulative):
        print(f"  '{word}': prob={prob:.3f}, cumulative={cum:.3f}")
    
    print()
    for p in p_values:
        # Find cutoff
        selected = []
        cum = 0
        for word, prob in zip(sorted_words, sorted_probs):
            selected.append(word)
            cum += prob
            if cum >= p:
                break
        
        print(f"  top_p={p}: tokens considered = {selected}")

# Example probabilities
probs = [0.4, 0.25, 0.15, 0.1, 0.05, 0.03, 0.02]
top_p_sampling_demo(probs, [0.5, 0.7, 0.9, 0.95])
```

---

## 2. Anthropic Claude API {#anthropic-api}

```python
# ตัวอย่างที่ 4: Anthropic API overview

# Documentation: https://docs.anthropic.com/
# Available models (as of knowledge cutoff):
anthropic_models = {
    # Claude 3.5 series
    "claude-3-5-sonnet-20241022": {
        "context": 200_000,
        "description": "Best combination of intelligence and speed",
        "best_for": "Complex reasoning, coding, analysis"
    },
    "claude-3-5-haiku-20241022": {
        "context": 200_000,
        "description": "Fastest and most compact",
        "best_for": "Simple tasks, high throughput"
    },
    # Claude 3 series
    "claude-3-opus-20240229": {
        "context": 200_000,
        "description": "Most powerful",
        "best_for": "Highly complex tasks"
    },
    "claude-3-sonnet-20240229": {
        "context": 200_000,
        "description": "Balance of speed and intelligence",
        "best_for": "General tasks"
    },
    "claude-3-haiku-20240307": {
        "context": 200_000,
        "description": "Fast and affordable",
        "best_for": "Customer service, content moderation"
    },
}

print("Anthropic Claude Models:")
print("-" * 80)
for model, info in anthropic_models.items():
    print(f"\n{model}:")
    print(f"  Context: {info['context']:,} tokens")
    print(f"  Description: {info['description']}")
    print(f"  Best for: {info['best_for']}")

# Rate limits
print("\n\nTypical Rate Limits (vary by tier):")
rate_limits = {
    "Free tier": "5 RPM, 25K TPM",
    "Build tier": "50 RPM, 50K TPM",
    "Scale tier": "Custom based on usage",
}
for tier, limits in rate_limits.items():
    print(f"  {tier}: {limits}")
```

---

## 3. anthropic Python SDK {#anthropic-sdk}

```python
# ตัวอย่างที่ 5: SDK installation and basic setup
# pip install anthropic

import os
from typing import Optional

# Basic client setup
# API key from environment variable (best practice)
# export ANTHROPIC_API_KEY="your-api-key-here"

# DO NOT hardcode API keys in code!
# api_key = os.environ.get("ANTHROPIC_API_KEY")

print("SDK Installation:")
print("  pip install anthropic")
print()
print("Environment setup:")
print("  export ANTHROPIC_API_KEY='your-api-key'")
print()
print("Python setup:")
print("""
import anthropic
import os

client = anthropic.Anthropic(
    api_key=os.environ.get("ANTHROPIC_API_KEY")
)

# Or use .env file with python-dotenv:
# from dotenv import load_dotenv
# load_dotenv()
# client = anthropic.Anthropic()  # Reads ANTHROPIC_API_KEY automatically
""")

# Demonstrate the client structure (without making actual API calls)
class MockAnthropicClient:
    """Mock client for demonstration"""
    
    def __init__(self, api_key=None):
        self.api_key = api_key or "mock-key"
        self.messages = MockMessagesAPI()
        print(f"Client initialized with key: {self.api_key[:4]}...")
    
    def count_tokens(self, messages, model="claude-3-5-sonnet-20241022", system=None):
        """Estimate token count"""
        total = 0
        for msg in messages:
            content = msg.get("content", "")
            if isinstance(content, str):
                total += len(content.split()) * 1.3  # Rough estimate
        if system:
            total += len(system.split()) * 1.3
        return int(total)

class MockMessagesAPI:
    def create(self, **kwargs):
        return MockResponse(kwargs)

class MockResponse:
    def __init__(self, params):
        self.id = "msg_mock123"
        self.type = "message"
        self.role = "assistant"
        self.content = [MockContent("This is a mock response for demonstration.")]
        self.model = params.get("model", "claude-3-5-sonnet-20241022")
        self.stop_reason = "end_turn"
        self.usage = MockUsage(50, 20)

class MockContent:
    def __init__(self, text):
        self.type = "text"
        self.text = text

class MockUsage:
    def __init__(self, input_tokens, output_tokens):
        self.input_tokens = input_tokens
        self.output_tokens = output_tokens

# Demo usage
client = MockAnthropicClient()
print("Mock client ready!")
```

---

## 4. Messages API {#messages-api}

```python
# ตัวอย่างที่ 6: Messages API - basic usage

# Actual API call pattern:
"""
import anthropic
client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "What is machine learning?"}
    ]
)
print(message.content[0].text)
"""

# Demonstration with mock client
def basic_message_demo():
    """Demonstrate basic Messages API usage"""
    
    # Message structure
    messages = [
        {
            "role": "user",
            "content": "What is machine learning?"
        }
    ]
    
    print("Message API call structure:")
    print("=" * 60)
    print("POST https://api.anthropic.com/v1/messages")
    print("Headers:")
    print("  x-api-key: YOUR_API_KEY")
    print("  anthropic-version: 2023-06-01")
    print("  content-type: application/json")
    print()
    print("Body:")
    
    body = {
        "model": "claude-3-5-sonnet-20241022",
        "max_tokens": 1024,
        "messages": messages
    }
    
    import json
    print(json.dumps(body, indent=2))
    
    print()
    print("Response structure:")
    response = {
        "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
        "type": "message",
        "role": "assistant",
        "content": [
            {
                "type": "text",
                "text": "Machine learning is a subset of AI..."
            }
        ],
        "model": "claude-3-5-sonnet-20241022",
        "stop_reason": "end_turn",
        "stop_sequence": None,
        "usage": {
            "input_tokens": 25,
            "output_tokens": 150
        }
    }
    print(json.dumps(response, indent=2))

basic_message_demo()
```

```python
# ตัวอย่างที่ 7: Multi-turn conversations

def multi_turn_demo():
    """Demonstrate multi-turn conversation"""
    
    # Build conversation history
    conversation = []
    
    def add_user_message(text):
        conversation.append({"role": "user", "content": text})
    
    def add_assistant_message(text):
        conversation.append({"role": "assistant", "content": text})
    
    def get_response(user_input):
        """Simulate getting response (in real code: call API)"""
        add_user_message(user_input)
        
        # Mock responses
        responses = {
            "What is Python?": "Python is a high-level, interpreted programming language known for its simplicity.",
            "What are its main uses?": "Python is mainly used for: data science, web development, automation, AI/ML, and scripting.",
            "What about performance?": "Python is slower than compiled languages like C++, but for many applications, this doesn't matter.",
        }
        
        response = responses.get(user_input, "I understand your question.")
        add_assistant_message(response)
        return response
    
    # Conversation turns
    questions = [
        "What is Python?",
        "What are its main uses?",
        "What about performance?",
    ]
    
    print("Multi-turn Conversation Demo:")
    print("=" * 60)
    
    for q in questions:
        print(f"\nUser: {q}")
        response = get_response(q)
        print(f"Assistant: {response}")
    
    print(f"\nTotal turns: {len(conversation)} messages")
    print(f"Conversation tokens: ~{sum(len(m['content'].split()) for m in conversation) * 1.3:.0f}")
    
    # Show memory consideration
    print("\nMemory tip: Each message adds to context window!")
    print("For long conversations, consider:")
    print("  - Summarize old turns")
    print("  - Keep only recent turns")
    print("  - Use retrieval for important info")

multi_turn_demo()
```

```python
# ตัวอย่างที่ 8: Content types (text, images, tool results)

def content_types_demo():
    """Demonstrate different content types in messages"""
    
    # Text content (simplest)
    text_message = {
        "role": "user",
        "content": "What is the capital of France?"
    }
    
    # Or as array (more explicit)
    text_message_explicit = {
        "role": "user",
        "content": [
            {"type": "text", "text": "What is the capital of France?"}
        ]
    }
    
    # Image content (requires base64 or URL)
    image_message = {
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": "...base64_encoded_image..."
                }
            },
            {
                "type": "text",
                "text": "What do you see in this image?"
            }
        ]
    }
    
    # Image URL (when using url type)
    image_url_message = {
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "url",
                    "url": "https://example.com/image.jpg"
                }
            },
            {
                "type": "text",
                "text": "Describe this image."
            }
        ]
    }
    
    # Tool result (after tool use)
    tool_result_message = {
        "role": "user",
        "content": [
            {
                "type": "tool_result",
                "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
                "content": "The weather in Paris is 22°C, sunny."
            }
        ]
    }
    
    print("Content Types in Messages API:")
    print()
    
    content_types = [
        ("Text (string shorthand)", text_message),
        ("Text (explicit)", text_message_explicit),
        ("Image (base64)", image_message),
        ("Image (URL)", image_url_message),
        ("Tool Result", tool_result_message),
    ]
    
    for name, message in content_types:
        print(f"  {name}:")
        content = message.get("content", "")
        if isinstance(content, str):
            print(f"    Simple string: '{content[:50]}'")
        elif isinstance(content, list):
            for item in content:
                print(f"    [{item.get('type', 'unknown')}] ", end="")
                if item.get('type') == 'text':
                    print(f"'{item.get('text', '')[:40]}'")
                elif item.get('type') == 'image':
                    source = item.get('source', {})
                    print(f"({source.get('type', '')} {source.get('media_type', source.get('url', '')[:30])})")
                elif item.get('type') == 'tool_result':
                    print(f"tool_id={item.get('tool_use_id', '')[:20]}...")
        print()

content_types_demo()
```

---

## 5. Streaming Responses {#streaming}

```python
# ตัวอย่างที่ 9: Streaming implementation

import time
import sys

def streaming_demo():
    """Demonstrate streaming response pattern"""
    
    # Real streaming code:
    """
    import anthropic
    client = anthropic.Anthropic()
    
    with client.messages.stream(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Write a haiku about Python"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
    
    # Access final message
    final_message = stream.get_final_message()
    print(f"\\nUsage: {final_message.usage}")
    """
    
    # Simulated streaming
    def simulate_stream(text, delay=0.03):
        """Simulate streaming output word by word"""
        words = text.split()
        for i, word in enumerate(words):
            yield word + (" " if i < len(words) - 1 else "")
            time.sleep(delay)
    
    print("Streaming Demo:")
    print("=" * 60)
    print("(Simulating Claude response word by word)")
    print()
    
    response_text = """
Machine learning is a fascinating field at the intersection of 
computer science and statistics. It enables computers to learn 
from data without being explicitly programmed for each task. 
The key idea is that algorithms can identify patterns in large 
datasets and make predictions or decisions based on those patterns.
""".strip()
    
    print("Claude: ", end="")
    sys.stdout.flush()
    
    for chunk in simulate_stream(response_text, delay=0.02):
        print(chunk, end="")
        sys.stdout.flush()
    
    print("\n\nStreaming complete!")

streaming_demo()
```

```python
# ตัวอย่างที่ 10: Streaming with event types

def streaming_events_demo():
    """Demonstrate different streaming event types"""
    
    # In real API, these are Server-Sent Events (SSE)
    # Event types:
    stream_events = [
        {
            "type": "message_start",
            "message": {
                "id": "msg_01234",
                "type": "message",
                "role": "assistant",
                "content": [],
                "model": "claude-3-5-sonnet-20241022",
                "usage": {"input_tokens": 25, "output_tokens": 0}
            }
        },
        {
            "type": "content_block_start",
            "index": 0,
            "content_block": {"type": "text", "text": ""}
        },
        {
            "type": "content_block_delta",
            "index": 0,
            "delta": {"type": "text_delta", "text": "Machine"}
        },
        {
            "type": "content_block_delta",
            "index": 0,
            "delta": {"type": "text_delta", "text": " learning"}
        },
        {
            "type": "content_block_stop",
            "index": 0
        },
        {
            "type": "message_delta",
            "delta": {"stop_reason": "end_turn", "stop_sequence": None},
            "usage": {"output_tokens": 150}
        },
        {
            "type": "message_stop"
        }
    ]
    
    print("Streaming Event Types:")
    print("=" * 60)
    
    accumulated_text = ""
    
    for event in stream_events:
        event_type = event.get("type", "unknown")
        
        if event_type == "message_start":
            print(f"  Event: message_start | id={event['message']['id']}")
        
        elif event_type == "content_block_start":
            print(f"  Event: content_block_start | index={event['index']}")
        
        elif event_type == "content_block_delta":
            text = event["delta"].get("text", "")
            accumulated_text += text
            print(f"  Event: content_block_delta | text='{text}'")
        
        elif event_type == "content_block_stop":
            print(f"  Event: content_block_stop")
        
        elif event_type == "message_delta":
            usage = event.get("usage", {})
            print(f"  Event: message_delta | stop_reason={event['delta']['stop_reason']} | output_tokens={usage.get('output_tokens', 0)}")
        
        elif event_type == "message_stop":
            print(f"  Event: message_stop")
    
    print(f"\nAccumulated text: '{accumulated_text}'")

streaming_events_demo()
```

---

## 6. System Prompts {#system-prompts}

```python
# ตัวอย่างที่ 11: System prompts
import json

def system_prompt_examples():
    """Demonstrate effective system prompts"""
    
    # Basic system prompt
    basic_system = """You are a helpful assistant."""
    
    # Role-based system prompt
    role_system = """You are an expert Python developer with 15 years of experience.
You help users write clean, efficient Python code.
Always follow PEP 8 guidelines and include type hints."""
    
    # Persona system prompt
    persona_system = """You are Ada, a friendly AI coding tutor.
Your communication style:
- Explain concepts simply, without jargon
- Use examples and analogies
- Encourage and be patient
- Ask clarifying questions when needed
- Give step-by-step guidance

You specialize in teaching Python to beginners."""
    
    # Structured output system prompt
    structured_system = """You are a data extraction assistant.
Always respond with valid JSON in this format:
{
  "entities": [
    {"name": "string", "type": "PERSON|ORG|LOC|DATE", "context": "string"}
  ],
  "summary": "string",
  "sentiment": "positive|negative|neutral"
}
Do not include any text outside the JSON."""
    
    # Constrained system prompt
    constrained_system = """You are a customer service bot for TechShop.
Rules:
1. Only discuss topics related to our products
2. For refunds, always say "Please contact support@techshop.com"
3. Never promise specific delivery dates
4. If you don't know something, admit it honestly
5. Keep responses under 3 paragraphs

Available topics: laptops, phones, accessories, warranties"""
    
    # Chain-of-thought system prompt
    cot_system = """You are a problem-solving assistant.
For every question:
1. Think step by step
2. Show your reasoning process
3. Consider edge cases
4. Provide a clear final answer

Format your response as:
[THINKING]: Your reasoning process
[ANSWER]: The final answer"""
    
    print("System Prompt Patterns:")
    print("=" * 60)
    
    patterns = {
        "Basic": basic_system,
        "Role-based": role_system,
        "Persona": persona_system,
        "Structured Output": structured_system,
        "Constrained": constrained_system,
        "Chain of Thought": cot_system,
    }
    
    for name, system in patterns.items():
        lines = system.strip().split('\n')
        first_line = lines[0][:70]
        print(f"\n{name}:")
        print(f"  Preview: {first_line}...")
        print(f"  Length: {len(system)} chars")

system_prompt_examples()
```

---

## 7. Prompt Engineering {#prompt-engineering}

```python
# ตัวอย่างที่ 12: Prompt engineering techniques

class PromptEngineer:
    """Collection of prompt engineering techniques"""
    
    @staticmethod
    def zero_shot(task, input_text):
        """Direct instruction without examples"""
        return f"{task}\n\nInput: {input_text}"
    
    @staticmethod
    def few_shot(task, examples, input_text):
        """Provide examples to guide response"""
        prompt = task + "\n\n"
        for i, (inp, out) in enumerate(examples):
            prompt += f"Example {i+1}:\n"
            prompt += f"Input: {inp}\n"
            prompt += f"Output: {out}\n\n"
        prompt += f"Now apply to:\nInput: {input_text}\nOutput:"
        return prompt
    
    @staticmethod
    def chain_of_thought(question, cot_instructions=None):
        """Encourage step-by-step reasoning"""
        if cot_instructions:
            return f"{question}\n\n{cot_instructions}"
        return f"{question}\n\nThink through this step by step:"
    
    @staticmethod
    def role_play(role, task, input_text):
        """Assign a role for better responses"""
        return f"You are {role}.\n\n{task}\n\nInput: {input_text}"
    
    @staticmethod
    def xml_structured(task, input_text, output_format):
        """Use XML tags for structure"""
        return f"""<task>{task}</task>

<input>
{input_text}
</input>

<output_format>
{output_format}
</output_format>

Please complete the task and format your response accordingly."""
    
    @staticmethod
    def react_prompt(task, tools_available):
        """ReAct: Reasoning + Acting pattern"""
        tools_str = "\n".join(f"- {t}" for t in tools_available)
        return f"""Task: {task}

Available tools:
{tools_str}

Follow this pattern:
Thought: [Your reasoning about what to do]
Action: [Tool name to use]
Action Input: [Input to the tool]
Observation: [Result from tool]
... (repeat as needed)
Thought: [Final reasoning]
Final Answer: [Your conclusion]

Begin:"""
    
    @staticmethod
    def self_consistency(question, num_paths=3):
        """Generate multiple reasoning paths"""
        return f"""{question}

Please solve this problem {num_paths} different ways and check 
if they reach the same answer. If they disagree, identify which
is correct and why.

Solution 1:
...(first approach)

Solution 2:
...(second approach)

Solution 3:
...(third approach)

Consensus answer:"""

# Demo
pe = PromptEngineer()

# Zero-shot
zs = pe.zero_shot(
    "Classify the sentiment (positive/negative/neutral)",
    "This product is amazing! I love it."
)
print("=== Zero-shot ===")
print(zs)

# Few-shot
examples = [
    ("I hate this!", "negative"),
    ("Great product!", "positive"),
    ("It's okay.", "neutral"),
]
fs = pe.few_shot(
    "Classify sentiment",
    examples,
    "Absolutely wonderful experience!"
)
print("\n=== Few-shot ===")
print(fs)

# Chain of thought
cot = pe.chain_of_thought(
    "If a train travels 120km in 2 hours, how long to travel 300km?",
    "Identify what we know, calculate the speed, then find the time."
)
print("\n=== Chain of Thought ===")
print(cot)
```

```python
# ตัวอย่างที่ 13: Advanced prompt techniques

class AdvancedPromptTechniques:
    """Advanced prompting strategies"""
    
    @staticmethod
    def meta_prompt(original_task):
        """Have model improve its own prompt"""
        return f"""I need to create an effective prompt for this task:
"{original_task}"

Please create an improved, more detailed prompt that:
1. Clearly defines the task
2. Specifies output format
3. Includes relevant constraints
4. Provides any helpful context

Output ONLY the improved prompt, nothing else."""
    
    @staticmethod
    def scratchpad_prompt(problem):
        """Use scratchpad for complex reasoning"""
        return f"""{problem}

<scratchpad>
Let me work through this carefully:
1. What information do I have?
2. What do I need to find?
3. What approach should I use?
4. Let me calculate/reason step by step...
</scratchpad>

Based on my analysis above, the answer is:"""
    
    @staticmethod
    def devil_advocate(claim):
        """Challenge assumptions and find weaknesses"""
        return f"""Claim to evaluate: "{claim}"

Please:
1. Identify the strongest arguments FOR this claim
2. Identify the strongest arguments AGAINST this claim
3. Consider edge cases and exceptions
4. Point out any unstated assumptions
5. Give a balanced final verdict

Be intellectually rigorous and devil's advocate."""
    
    @staticmethod
    def template_filling(template, variables):
        """Fill template with variables"""
        result = template
        for key, value in variables.items():
            result = result.replace(f"{{{key}}}", str(value))
        return result

# Demo
adv = AdvancedPromptTechniques()

# Meta prompt
print("=== Meta Prompt ===")
meta = adv.meta_prompt("Write Python code to parse CSV files")
print(meta[:200] + "...")

# Scratchpad
print("\n=== Scratchpad ===")
scratchpad = adv.scratchpad_prompt("A store sells items at 20% discount. After discount, an item costs $48. What was the original price?")
print(scratchpad[:300] + "...")

# Template filling
template = """Dear {name},

Thank you for your {product} purchase on {date}.
Your order number is {order_id}.

Best regards,
{company}"""

filled = adv.template_filling(template, {
    "name": "John Smith",
    "product": "Python Course",
    "date": "2024-01-15",
    "order_id": "ORD-12345",
    "company": "LearnPython Co."
})
print("\n=== Template Filling ===")
print(filled)
```

---

## 8. Tool Use - Function Calling {#tool-use}

```python
# ตัวอย่างที่ 14: Tool use (function calling)

import json
from datetime import datetime
import math

# Define tools that Claude can use
TOOLS = [
    {
        "name": "get_weather",
        "description": "Get current weather for a location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City and country, e.g. 'Bangkok, Thailand'"
                },
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "Temperature unit"
                }
            },
            "required": ["location"]
        }
    },
    {
        "name": "calculate",
        "description": "Perform mathematical calculations",
        "input_schema": {
            "type": "object",
            "properties": {
                "expression": {
                    "type": "string",
                    "description": "Mathematical expression to evaluate, e.g. '2 + 2 * 3'"
                }
            },
            "required": ["expression"]
        }
    },
    {
        "name": "search_database",
        "description": "Search company product database",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Search query"
                },
                "category": {
                    "type": "string",
                    "enum": ["laptop", "phone", "tablet", "accessory"],
                    "description": "Product category filter"
                },
                "max_price": {
                    "type": "number",
                    "description": "Maximum price filter"
                }
            },
            "required": ["query"]
        }
    }
]

# Tool implementations
def get_weather(location: str, unit: str = "celsius") -> dict:
    """Mock weather API"""
    # In real code: call actual weather API
    mock_data = {
        "Bangkok, Thailand": {"temp": 35, "conditions": "Hot and humid", "humidity": 80},
        "London, UK": {"temp": 15, "conditions": "Cloudy", "humidity": 65},
        "New York, US": {"temp": 22, "conditions": "Partly cloudy", "humidity": 55},
    }
    
    data = mock_data.get(location, {"temp": 20, "conditions": "Unknown", "humidity": 50})
    temp = data["temp"]
    
    if unit == "fahrenheit":
        temp = temp * 9/5 + 32
    
    return {
        "location": location,
        "temperature": f"{temp}°{'F' if unit == 'fahrenheit' else 'C'}",
        "conditions": data["conditions"],
        "humidity": f"{data['humidity']}%"
    }

def calculate(expression: str) -> dict:
    """Safe math evaluation"""
    import re
    # Only allow safe characters
    if not re.match(r'^[0-9+\-*/()., \^%]+$', expression):
        return {"error": "Invalid expression"}
    
    try:
        # Replace ^ with ** for power
        expression = expression.replace('^', '**')
        result = eval(expression, {"__builtins__": {}}, {})
        return {"expression": expression, "result": result}
    except Exception as e:
        return {"error": str(e)}

def search_database(query: str, category: str = None, max_price: float = None) -> dict:
    """Mock product search"""
    products = [
        {"name": "MacBook Pro", "category": "laptop", "price": 1999, "stock": 5},
        {"name": "iPhone 15", "category": "phone", "price": 999, "stock": 20},
        {"name": "iPad Pro", "category": "tablet", "price": 799, "stock": 8},
        {"name": "AirPods Pro", "category": "accessory", "price": 249, "stock": 50},
        {"name": "Dell XPS 15", "category": "laptop", "price": 1499, "stock": 3},
        {"name": "Samsung S24", "category": "phone", "price": 899, "stock": 15},
    ]
    
    results = []
    for product in products:
        if query.lower() in product["name"].lower():
            if category and product["category"] != category:
                continue
            if max_price and product["price"] > max_price:
                continue
            results.append(product)
    
    return {"query": query, "count": len(results), "products": results}

# Tool dispatcher
TOOL_FUNCTIONS = {
    "get_weather": get_weather,
    "calculate": calculate,
    "search_database": search_database,
}

def execute_tool(tool_name: str, tool_input: dict) -> str:
    """Execute tool and return result as string"""
    fn = TOOL_FUNCTIONS.get(tool_name)
    if not fn:
        return json.dumps({"error": f"Unknown tool: {tool_name}"})
    
    try:
        result = fn(**tool_input)
        return json.dumps(result, indent=2)
    except Exception as e:
        return json.dumps({"error": str(e)})

# Demonstrate tool use flow
print("=== Tool Use Flow ===")
print()
print("1. Define tools with JSON schema")
print("2. Send request with tools to API")
print("3. Model responds with tool_use block when it wants to call a tool")
print("4. Execute the tool locally")
print("5. Send tool_result back to model")
print("6. Model continues with tool results")
print()

# Example tool use conversation flow
print("Example conversation:")
print()

# User message
user_msg = "What's the weather in Bangkok and what's 25 * 48?"
print(f"User: {user_msg}")
print()

# Simulate model response with tool calls
mock_tool_calls = [
    {
        "type": "tool_use",
        "id": "toolu_weather123",
        "name": "get_weather",
        "input": {"location": "Bangkok, Thailand"}
    },
    {
        "type": "tool_use",
        "id": "toolu_calc456",
        "name": "calculate",
        "input": {"expression": "25 * 48"}
    }
]

print("Model requests tools:")
for tc in mock_tool_calls:
    print(f"  Tool: {tc['name']}")
    print(f"  Input: {json.dumps(tc['input'])}")
    result = execute_tool(tc['name'], tc['input'])
    print(f"  Result: {result}")
    print()

# Final response
print("Model final response:")
print("  The weather in Bangkok is 35°C, hot and humid. And 25 × 48 = 1,200.")
```

---

## 9. Vision Capabilities {#vision}

```python
# ตัวอย่างที่ 15: Vision - analyzing images

import base64
import io
from pathlib import Path

def encode_image_to_base64(image_path: str) -> tuple:
    """Encode image to base64 for API"""
    path = Path(image_path)
    ext = path.suffix.lower()
    
    media_types = {
        '.jpg': 'image/jpeg',
        '.jpeg': 'image/jpeg',
        '.png': 'image/png',
        '.gif': 'image/gif',
        '.webp': 'image/webp'
    }
    
    media_type = media_types.get(ext, 'image/jpeg')
    
    with open(image_path, 'rb') as f:
        data = base64.standard_b64encode(f.read()).decode('utf-8')
    
    return media_type, data

def create_vision_message(image_path: str, question: str) -> dict:
    """Create message with image for API"""
    # media_type, data = encode_image_to_base64(image_path)
    
    # For demo, use placeholder
    message = {
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": "...base64_encoded_image..."
                }
            },
            {
                "type": "text",
                "text": question
            }
        ]
    }
    
    return message

# Vision use cases
vision_use_cases = {
    "Image Description": "What do you see in this image? Describe in detail.",
    "Chart Analysis": "Analyze the data shown in this chart. What are the key trends?",
    "Code Screenshot": "Read the code in this screenshot and explain what it does.",
    "Document Parsing": "Extract all text and structure from this document image.",
    "Product Analysis": "What product is shown? List its visible features.",
    "Medical Image": "Describe what you observe in this medical scan (educational only).",
    "UI Review": "Review this UI screenshot. What improvements would you suggest?",
    "Math Problem": "Solve the math problem shown in this image.",
}

print("Vision API Use Cases:")
print("=" * 60)
for use_case, prompt in vision_use_cases.items():
    print(f"\n{use_case}:")
    print(f"  Prompt: '{prompt[:80]}'")

# Image size considerations
print("\n\nImage Guidelines:")
image_guidelines = {
    "Max size": "5MB per image",
    "Max resolution": "8192 × 8192 pixels",
    "Supported formats": "JPEG, PNG, GIF, WebP",
    "Max images per request": "20 images",
    "Cost": "Calculated based on image tokens",
    "Tip": "Resize large images for cost efficiency",
}

for key, value in image_guidelines.items():
    print(f"  {key}: {value}")

# Token calculation for images
print("\n\nImage Token Calculation:")
print("  Image tokens ≈ (width/32) × (height/32) × tiles")
print("  A 1000×1000 image ≈ 1334 tokens")
print("  Smaller images = fewer tokens = lower cost")

def estimate_image_tokens(width: int, height: int) -> int:
    """Rough estimate of image tokens"""
    # Based on Anthropic's tiling approach
    tiles_w = (width + 511) // 512
    tiles_h = (height + 511) // 512
    return tiles_w * tiles_h * 170 + 85  # Approximate formula

sizes = [(256, 256), (512, 512), (1024, 768), (1920, 1080), (4096, 4096)]
for w, h in sizes:
    tokens = estimate_image_tokens(w, h)
    print(f"  {w}×{h}: ~{tokens} tokens")
```

---

## 10. OpenAI API Comparison {#openai}

```python
# ตัวอย่างที่ 16: Anthropic vs OpenAI comparison

def api_comparison():
    """Compare Anthropic and OpenAI APIs"""
    
    print("=== API Comparison: Anthropic vs OpenAI ===\n")
    
    # Basic message structure comparison
    anthropic_message = {
        "model": "claude-3-5-sonnet-20241022",
        "max_tokens": 1024,
        "system": "You are a helpful assistant.",  # Top-level
        "messages": [
            {"role": "user", "content": "Hello!"}
        ]
    }
    
    openai_message = {
        "model": "gpt-4o",
        "max_tokens": 1024,
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},  # In messages
            {"role": "user", "content": "Hello!"}
        ]
    }
    
    import json
    print("Anthropic request:")
    print(json.dumps(anthropic_message, indent=2))
    
    print("\nOpenAI request:")
    print(json.dumps(openai_message, indent=2))
    
    # Key differences
    differences = {
        "System prompt": {
            "Anthropic": "Top-level 'system' parameter",
            "OpenAI": "Message with role='system'"
        },
        "Response access": {
            "Anthropic": "response.content[0].text",
            "OpenAI": "response.choices[0].message.content"
        },
        "Token counting": {
            "Anthropic": "response.usage.input_tokens",
            "OpenAI": "response.usage.prompt_tokens"
        },
        "Streaming": {
            "Anthropic": "client.messages.stream() context manager",
            "OpenAI": "stream=True parameter"
        },
        "Tools/Functions": {
            "Anthropic": "tools parameter with input_schema",
            "OpenAI": "tools/functions parameter"
        },
        "Vision": {
            "Anthropic": "image source with type/media_type/data",
            "OpenAI": "image_url with url"
        },
    }
    
    print("\n\nKey Differences:")
    for feature, apis in differences.items():
        print(f"\n{feature}:")
        for api, desc in apis.items():
            print(f"  {api}: {desc}")

api_comparison()
```

```python
# ตัวอย่างที่ 17: OpenAI Python SDK usage

def openai_sdk_demo():
    """OpenAI SDK usage patterns"""
    
    print("=== OpenAI SDK ===")
    print("pip install openai")
    print()
    
    # Setup
    print("""
import openai
import os

client = openai.OpenAI(api_key=os.environ.get("OPENAI_API_KEY"))

# Basic completion
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Python?"}
    ]
)
print(response.choices[0].message.content)
print(f"Tokens used: {response.usage.total_tokens}")

# Function calling
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto"
)

# Check for tool calls
if response.choices[0].finish_reason == "tool_calls":
    tool_calls = response.choices[0].message.tool_calls
    for tc in tool_calls:
        print(f"Tool: {tc.function.name}")
        print(f"Args: {tc.function.arguments}")

# Streaming
stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Tell me a story"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
""")

openai_sdk_demo()
```

---

## 11. LangChain Framework {#langchain}

```python
# ตัวอย่างที่ 18: LangChain basics
# pip install langchain langchain-anthropic

print("=== LangChain Overview ===")
print("LangChain provides abstractions for building LLM applications:")
print()

# LangChain components
components = {
    "LLMs/ChatModels": "Interface to LLM providers (Claude, GPT, etc.)",
    "Prompts": "PromptTemplate, ChatPromptTemplate",
    "Output Parsers": "Parse LLM output (JSON, lists, etc.)",
    "Chains": "Sequence of operations (LLMChain, SequentialChain)",
    "Agents": "LLMs that can use tools dynamically",
    "Memory": "Conversation memory management",
    "Document Loaders": "Load from PDF, web, DB, etc.",
    "Text Splitters": "Split long documents",
    "Embeddings": "Vector representations of text",
    "Vector Stores": "Store and retrieve embeddings",
    "Retrievers": "Retrieve relevant documents",
    "Callbacks": "Monitor and log operations",
}

for component, description in components.items():
    print(f"  {component:20}: {description}")

# LangChain code examples
print("\n=== LangChain Code Patterns ===")
print()

langchain_examples = """
# 1. Basic LLM Chain
from langchain_anthropic import ChatAnthropic
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser

llm = ChatAnthropic(model="claude-3-5-sonnet-20241022")

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{input}")
])

chain = prompt | llm | StrOutputParser()
result = chain.invoke({"input": "What is Python?"})

# 2. Memory in conversations
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain

memory = ConversationBufferMemory()
conversation = ConversationChain(llm=llm, memory=memory)

response1 = conversation.predict(input="My name is Alice")
response2 = conversation.predict(input="What's my name?")  # Should remember

# 3. Document loading and splitting
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

loader = PyPDFLoader("document.pdf")
pages = loader.load()

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
chunks = splitter.split_documents(pages)

# 4. Embeddings and Vector Store
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import Chroma

embeddings = HuggingFaceEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# 5. RAG Chain
from langchain.chains import RetrievalQA

qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    return_source_documents=True
)

result = qa_chain({"query": "What does the document say about X?"})
print(result["result"])
print(result["source_documents"])
"""

print(langchain_examples)
```

---

## 12. Vector Databases {#vector-db}

```python
# ตัวอย่างที่ 19: Vector database concepts

import numpy as np
from typing import List, Dict, Tuple

class SimpleVectorDB:
    """
    Simple in-memory vector database for demonstration.
    In production use: Chroma, Pinecone, Weaviate, Qdrant, etc.
    """
    
    def __init__(self, embedding_dim=384):
        self.embedding_dim = embedding_dim
        self.vectors = []
        self.metadata = []
        self.ids = []
    
    def add(self, id: str, vector: np.ndarray, metadata: dict = None):
        """Add vector to database"""
        if len(vector) != self.embedding_dim:
            raise ValueError(f"Expected dim {self.embedding_dim}, got {len(vector)}")
        
        self.ids.append(id)
        self.vectors.append(vector / (np.linalg.norm(vector) + 1e-10))  # Normalize
        self.metadata.append(metadata or {})
    
    def add_batch(self, items: List[Tuple[str, np.ndarray, dict]]):
        """Add multiple vectors"""
        for id, vector, metadata in items:
            self.add(id, vector, metadata)
    
    def search(self, query_vector: np.ndarray, top_k: int = 5,
               filter_fn=None) -> List[Dict]:
        """Search for similar vectors using cosine similarity"""
        if len(self.vectors) == 0:
            return []
        
        # Normalize query
        query_norm = query_vector / (np.linalg.norm(query_vector) + 1e-10)
        
        # Compute similarities
        vectors_matrix = np.array(self.vectors)
        similarities = vectors_matrix @ query_norm
        
        # Get top k indices
        indices = np.argsort(similarities)[::-1]
        
        results = []
        for idx in indices:
            # Apply filter
            if filter_fn and not filter_fn(self.metadata[idx]):
                continue
            
            results.append({
                'id': self.ids[idx],
                'score': float(similarities[idx]),
                'metadata': self.metadata[idx]
            })
            
            if len(results) >= top_k:
                break
        
        return results
    
    def delete(self, id: str) -> bool:
        """Delete vector by ID"""
        if id in self.ids:
            idx = self.ids.index(id)
            self.ids.pop(idx)
            self.vectors.pop(idx)
            self.metadata.pop(idx)
            return True
        return False
    
    def update_metadata(self, id: str, metadata: dict):
        """Update metadata for a vector"""
        if id in self.ids:
            idx = self.ids.index(id)
            self.metadata[idx].update(metadata)
            return True
        return False
    
    def stats(self) -> dict:
        """Get database statistics"""
        return {
            'total_vectors': len(self.vectors),
            'embedding_dim': self.embedding_dim,
            'memory_mb': (len(self.vectors) * self.embedding_dim * 4) / (1024*1024)
        }

def mock_embed(text: str, dim: int = 384) -> np.ndarray:
    """Mock embedding function - in real code use actual embeddings"""
    np.random.seed(hash(text) % 10000)
    return np.random.randn(dim)

# Demo
db = SimpleVectorDB(embedding_dim=384)

# Add documents
documents = [
    ("doc1", "Python is a programming language", {"source": "textbook", "page": 1}),
    ("doc2", "Machine learning uses statistics", {"source": "paper", "page": 5}),
    ("doc3", "Deep learning is a subset of ML", {"source": "paper", "page": 10}),
    ("doc4", "OpenCV processes images", {"source": "docs", "page": 1}),
    ("doc5", "NLP handles text data", {"source": "textbook", "page": 20}),
    ("doc6", "Python libraries include numpy", {"source": "textbook", "page": 15}),
    ("doc7", "Neural networks have layers", {"source": "paper", "page": 3}),
    ("doc8", "Transformers use attention", {"source": "paper", "page": 8}),
]

for doc_id, text, metadata in documents:
    vector = mock_embed(text)
    db.add(doc_id, vector, {**metadata, "text": text})

print("=== Vector Database Demo ===")
print(f"\nDatabase stats: {db.stats()}")

# Search
query = "deep learning neural network"
query_vec = mock_embed(query)

print(f"\nQuery: '{query}'")
results = db.search(query_vec, top_k=3)
print("Top 3 results:")
for r in results:
    print(f"  [{r['id']}] score={r['score']:.4f} | {r['metadata']['text']}")

# Filtered search (only from papers)
paper_results = db.search(
    query_vec, 
    top_k=3,
    filter_fn=lambda meta: meta.get("source") == "paper"
)
print(f"\nFiltered (papers only):")
for r in paper_results:
    print(f"  [{r['id']}] score={r['score']:.4f} | {r['metadata']['text']}")
```

---

## 13. RAG - Retrieval Augmented Generation {#rag}

```python
# ตัวอย่างที่ 20: Complete RAG Pipeline
import re
from typing import List, Dict, Optional

class DocumentProcessor:
    """Process documents for RAG pipeline"""
    
    def __init__(self, chunk_size=500, chunk_overlap=50):
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap
    
    def load_text(self, text: str, source: str = "unknown") -> List[Dict]:
        """Load and create document"""
        return [{"content": text, "source": source, "type": "text"}]
    
    def split_into_chunks(self, documents: List[Dict]) -> List[Dict]:
        """Split documents into overlapping chunks"""
        chunks = []
        
        for doc in documents:
            text = doc["content"]
            
            # Split by sentences first
            sentences = re.split(r'(?<=[.!?])\s+', text)
            
            current_chunk = []
            current_len = 0
            
            for sentence in sentences:
                sent_len = len(sentence.split())
                
                if current_len + sent_len > self.chunk_size and current_chunk:
                    # Save current chunk
                    chunk_text = " ".join(current_chunk)
                    chunks.append({
                        "content": chunk_text,
                        "source": doc["source"],
                        "chunk_id": len(chunks),
                        "word_count": len(chunk_text.split())
                    })
                    
                    # Keep overlap
                    overlap_sentences = current_chunk[-2:] if len(current_chunk) > 2 else current_chunk
                    current_chunk = overlap_sentences.copy()
                    current_len = sum(len(s.split()) for s in current_chunk)
                
                current_chunk.append(sentence)
                current_len += sent_len
            
            # Last chunk
            if current_chunk:
                chunk_text = " ".join(current_chunk)
                chunks.append({
                    "content": chunk_text,
                    "source": doc["source"],
                    "chunk_id": len(chunks),
                    "word_count": len(chunk_text.split())
                })
        
        return chunks

class RAGPipeline:
    """
    Complete RAG (Retrieval Augmented Generation) pipeline
    
    Architecture:
    1. Document Processing & Indexing
    2. Query Processing
    3. Retrieval
    4. Context Augmentation
    5. LLM Generation
    """
    
    def __init__(self, vector_db: SimpleVectorDB, embed_fn, llm_fn):
        self.vector_db = vector_db
        self.embed_fn = embed_fn
        self.llm_fn = llm_fn
        self.processor = DocumentProcessor()
        self.indexed_docs = {}
    
    def index_document(self, doc_id: str, text: str, metadata: dict = None):
        """Index a document for retrieval"""
        # Load and chunk
        documents = self.processor.load_text(text, source=doc_id)
        chunks = self.processor.split_into_chunks(documents)
        
        print(f"Indexing '{doc_id}': {len(chunks)} chunks")
        
        # Embed and store
        for chunk in chunks:
            chunk_id = f"{doc_id}_chunk_{chunk['chunk_id']}"
            embedding = self.embed_fn(chunk["content"])
            
            self.vector_db.add(
                id=chunk_id,
                vector=embedding,
                metadata={
                    **chunk,
                    **(metadata or {}),
                    "doc_id": doc_id
                }
            )
        
        self.indexed_docs[doc_id] = len(chunks)
        return len(chunks)
    
    def retrieve(self, query: str, top_k: int = 5, 
                 source_filter: Optional[str] = None) -> List[Dict]:
        """Retrieve relevant chunks for query"""
        query_embedding = self.embed_fn(query)
        
        filter_fn = None
        if source_filter:
            filter_fn = lambda meta: meta.get("doc_id") == source_filter
        
        results = self.vector_db.search(query_embedding, top_k=top_k, filter_fn=filter_fn)
        return results
    
    def build_context(self, retrieved_chunks: List[Dict]) -> str:
        """Build context string from retrieved chunks"""
        context_parts = []
        
        for chunk in retrieved_chunks:
            meta = chunk["metadata"]
            source = meta.get("doc_id", "unknown")
            content = meta.get("content", "")
            score = chunk["score"]
            
            context_parts.append(f"[Source: {source}, Relevance: {score:.3f}]\n{content}")
        
        return "\n\n---\n\n".join(context_parts)
    
    def generate_answer(self, query: str, context: str) -> str:
        """Generate answer using LLM with context"""
        
        system_prompt = """You are a helpful assistant that answers questions based on provided context.

Rules:
1. Only use information from the provided context
2. If the answer isn't in the context, say so clearly
3. Cite your sources (e.g., "According to [source]...")
4. Be concise and accurate"""
        
        user_message = f"""Context:
{context}

Question: {query}

Answer based on the context above:"""
        
        return self.llm_fn(system_prompt, user_message)
    
    def query(self, question: str, top_k: int = 5, 
              source_filter: Optional[str] = None) -> Dict:
        """Full RAG query pipeline"""
        print(f"\nQuery: {question}")
        
        # Retrieve
        retrieved = self.retrieve(question, top_k=top_k, source_filter=source_filter)
        print(f"Retrieved {len(retrieved)} chunks")
        
        if not retrieved:
            return {
                "answer": "No relevant information found in the knowledge base.",
                "sources": [],
                "chunks": []
            }
        
        # Build context
        context = self.build_context(retrieved)
        
        # Generate
        answer = self.generate_answer(question, context)
        
        # Collect sources
        sources = list(set(r["metadata"].get("doc_id", "unknown") for r in retrieved))
        
        return {
            "answer": answer,
            "sources": sources,
            "chunks": retrieved,
            "context_length": len(context.split())
        }

# Mock LLM function (replace with real API call)
def mock_llm(system: str, user: str) -> str:
    """Mock LLM for demonstration"""
    # Extract first retrieved source info
    if "Source:" in user:
        import re
        sources = re.findall(r'\[Source: ([^\]]+)', user)
        unique_sources = list(set(sources))
        return f"Based on the provided context from {', '.join(unique_sources)}, the answer is: [Mock response based on retrieved context]"
    return "Based on the provided context: [Mock answer]"

# Build RAG system
db = SimpleVectorDB(embedding_dim=384)
rag = RAGPipeline(db, mock_embed, mock_llm)

# Index documents
documents = {
    "python_guide": """
Python is a high-level programming language created by Guido van Rossum in 1991.
It emphasizes code readability and simplicity. Python supports multiple programming 
paradigms including procedural, object-oriented, and functional programming.
Python is widely used in data science, machine learning, web development, and automation.
The Python Package Index (PyPI) hosts thousands of libraries.
    """,
    
    "ml_fundamentals": """
Machine learning is a subset of artificial intelligence that enables systems to learn 
from data without being explicitly programmed. The main types are supervised learning,
unsupervised learning, and reinforcement learning. Supervised learning uses labeled 
training data. Neural networks are inspired by biological brain structures and consist 
of layers of interconnected nodes.
    """,
    
    "deep_learning": """
Deep learning uses neural networks with many layers (hence 'deep'). Convolutional 
Neural Networks (CNNs) excel at image recognition tasks. Recurrent Neural Networks 
(RNNs) process sequential data like text. Transformers use attention mechanisms and 
have revolutionized NLP. Training deep networks requires large amounts of data and 
computational resources, often GPUs.
    """,
}

for doc_id, content in documents.items():
    rag.index_document(doc_id, content.strip())

print(f"\nTotal vectors in DB: {db.stats()['total_vectors']}")

# Query examples
questions = [
    "What is Python and when was it created?",
    "How does machine learning work?",
    "What are transformers in deep learning?",
    "Who uses Python for data science?",
]

for question in questions[:2]:
    result = rag.query(question, top_k=3)
    print(f"\n{'='*50}")
    print(f"Q: {question}")
    print(f"A: {result['answer']}")
    print(f"Sources: {result['sources']}")
```

---

## 14. Building AI Applications {#ai-apps}

```python
# ตัวอย่างที่ 21: AI-powered code assistant

class CodeAssistant:
    """AI-powered coding assistant"""
    
    def __init__(self, llm_fn):
        self.llm_fn = llm_fn
        self.conversation_history = []
        
        self.system_prompt = """You are an expert Python developer and code assistant.

Capabilities:
1. Write, review, and debug Python code
2. Explain programming concepts
3. Suggest best practices and optimizations
4. Generate documentation and tests

Response format:
- For code: use ```python code blocks
- For explanations: clear, structured prose
- For reviews: list specific issues and improvements
- Always include error handling in code examples"""
    
    def _add_message(self, role: str, content: str):
        self.conversation_history.append({"role": role, "content": content})
    
    def _get_response(self, user_input: str) -> str:
        """Get response from LLM"""
        self._add_message("user", user_input)
        
        # In real code: include conversation_history in API call
        response = self.llm_fn(self.system_prompt, user_input)
        self._add_message("assistant", response)
        return response
    
    def generate_code(self, description: str, language: str = "python") -> str:
        """Generate code from description"""
        prompt = f"""Generate {language} code for the following requirement:

{description}

Requirements:
- Production-ready quality
- Include error handling
- Add type hints (for Python)
- Include docstring
- Add usage example in comments"""
        
        return self._get_response(prompt)
    
    def review_code(self, code: str) -> str:
        """Review code for issues"""
        prompt = f"""Review the following code and provide detailed feedback:

```python
{code}
```

Analyze:
1. Correctness (bugs, logic errors)
2. Performance (bottlenecks, complexity)
3. Security (vulnerabilities, input validation)
4. Readability (naming, structure, comments)
5. Best practices (PEP 8, design patterns)

Format: List specific issues with line numbers and suggested fixes."""
        
        return self._get_response(prompt)
    
    def explain_code(self, code: str, detail_level: str = "normal") -> str:
        """Explain what code does"""
        detail_prompts = {
            "simple": "Explain in simple terms for a beginner",
            "normal": "Explain the logic and purpose of each section",
            "detailed": "Explain every line in detail including algorithms used"
        }
        
        prompt = f"""Explain this code:

```python
{code}
```

{detail_prompts.get(detail_level, detail_prompts['normal'])}

Include:
- Overall purpose
- Key functions/classes
- Data flow
- Any algorithms or patterns used"""
        
        return self._get_response(prompt)
    
    def generate_tests(self, code: str) -> str:
        """Generate unit tests for code"""
        prompt = f"""Generate comprehensive unit tests for this code:

```python
{code}
```

Create tests using pytest that cover:
1. Happy path (normal inputs)
2. Edge cases (empty, None, boundary values)
3. Error cases (invalid inputs, exceptions)
4. Integration aspects if applicable

Use descriptive test names and add docstrings."""
        
        return self._get_response(prompt)
    
    def debug_code(self, code: str, error_message: str) -> str:
        """Help debug code with error"""
        prompt = f"""Debug this code that produces an error:

Code:
```python
{code}
```

Error message:
```
{error_message}
```

Provide:
1. Root cause analysis
2. Fixed code
3. Explanation of the fix
4. Prevention tips"""
        
        return self._get_response(prompt)

# Test the assistant
def mock_llm_code(system: str, user: str) -> str:
    """Mock LLM for demonstration"""
    if "Generate" in user:
        return '''Here's the Python code:

```python
def fibonacci(n: int) -> list[int]:
    """Generate Fibonacci sequence up to n terms."""
    if n <= 0:
        raise ValueError("n must be positive")
    sequence = [0, 1]
    while len(sequence) < n:
        sequence.append(sequence[-2] + sequence[-1])
    return sequence[:n]

# Usage: fibonacci(10) -> [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```'''
    
    elif "Review" in user:
        return '''Code Review:

**Issues Found:**
1. Line 3: Missing type hints - add `def process(data: list[str]) -> list[str]:`
2. Line 7: No input validation - add `if not data: return []`
3. Line 12: Variable name `x` not descriptive - rename to `item`

**Suggestions:**
- Add docstring explaining purpose
- Consider using list comprehension for efficiency
- Add logging for debugging'''
    
    return "I understand the request and here's my response: [Mock response]"

assistant = CodeAssistant(mock_llm_code)

print("=== AI Code Assistant Demo ===\n")

# Generate code
desc = "Generate a Fibonacci sequence function"
print(f"Request: {desc}")
result = assistant.generate_code(desc)
print(f"Response:\n{result[:300]}...")

# Review code
sample_code = """
def process(data):
    result = []
    for x in data:
        if x > 0:
            result.append(x * 2)
    return result
"""
print(f"\n\nCode Review:")
review = assistant.review_code(sample_code)
print(review[:300])
```

---

## 15. ตัวอย่างโปรแกรมจริง {#real-examples}

### Example 1: AI Assistant with Memory

```python
# ตัวอย่างที่ 22: Persistent AI assistant
import json
from datetime import datetime
from typing import List, Dict, Optional

class ConversationMemory:
    """Manage conversation history with summarization"""
    
    def __init__(self, max_messages=20, max_tokens_estimate=4000):
        self.messages = []
        self.max_messages = max_messages
        self.max_tokens = max_tokens_estimate
        self.summary = None
    
    def add(self, role: str, content: str):
        """Add message to history"""
        self.messages.append({
            "role": role,
            "content": content,
            "timestamp": datetime.now().isoformat(),
            "tokens": len(content.split()) * 1.3  # Estimate
        })
        
        # Trim if needed
        if len(self.messages) > self.max_messages:
            self._summarize_old()
    
    def get_messages(self) -> List[Dict]:
        """Get messages for API call"""
        msgs = []
        
        if self.summary:
            msgs.append({
                "role": "user",
                "content": f"[Previous conversation summary: {self.summary}]"
            })
            msgs.append({
                "role": "assistant",
                "content": "Understood, I have the context from our previous conversation."
            })
        
        for msg in self.messages:
            msgs.append({"role": msg["role"], "content": msg["content"]})
        
        return msgs
    
    def _summarize_old(self):
        """Summarize older messages"""
        # Keep last N messages, summarize the rest
        keep_count = self.max_messages // 2
        old_messages = self.messages[:-keep_count]
        
        # Create summary (in real code: use LLM to summarize)
        topics = set()
        for msg in old_messages:
            words = msg["content"].lower().split()
            topics.update([w for w in words if len(w) > 4][:3])
        
        self.summary = f"Discussion about: {', '.join(list(topics)[:5])}"
        self.messages = self.messages[-keep_count:]
    
    def clear(self):
        """Clear conversation history"""
        self.messages = []
        self.summary = None
    
    def get_stats(self) -> Dict:
        total_tokens = sum(msg["tokens"] for msg in self.messages)
        return {
            "messages": len(self.messages),
            "estimated_tokens": int(total_tokens),
            "has_summary": self.summary is not None
        }

class PersistentAIAssistant:
    """
    Persistent AI assistant with:
    - Conversation memory
    - Context retention
    - Topic switching
    - Personality settings
    """
    
    def __init__(self, name="Claude", personality="helpful"):
        self.name = name
        self.memory = ConversationMemory()
        self.personality = personality
        self.context = {}
        
        personalities = {
            "helpful": "You are a helpful, accurate, and thorough assistant.",
            "concise": "You are a concise assistant. Keep responses under 3 sentences.",
            "expert": "You are a world-class expert who provides deep insights.",
            "casual": "You're a friendly, casual assistant who uses simple language.",
            "socratic": "You guide users to answers through questions."
        }
        
        self.base_system = personalities.get(personality, personalities["helpful"])
    
    def set_context(self, key: str, value):
        """Set context information"""
        self.context[key] = value
    
    def _build_system_prompt(self) -> str:
        """Build system prompt with context"""
        system = self.base_system
        
        if self.context:
            context_str = "\n".join(f"- {k}: {v}" for k, v in self.context.items())
            system += f"\n\nContext about the user:\n{context_str}"
        
        return system
    
    def chat(self, user_input: str, llm_fn) -> str:
        """Send message and get response"""
        # Add user message
        self.memory.add("user", user_input)
        
        # Build prompt
        system = self._build_system_prompt()
        messages = self.memory.get_messages()
        
        # Get response (using mock)
        response = llm_fn(system, messages[-1]["content"] if messages else user_input)
        
        # Add to memory
        self.memory.add("assistant", response)
        
        return response
    
    def save_session(self, filepath: str):
        """Save conversation to file"""
        session = {
            "name": self.name,
            "personality": self.personality,
            "context": self.context,
            "messages": self.memory.messages,
            "summary": self.memory.summary,
            "saved_at": datetime.now().isoformat()
        }
        with open(filepath, 'w') as f:
            json.dump(session, f, indent=2)
        print(f"Session saved to {filepath}")
    
    def load_session(self, filepath: str):
        """Load conversation from file"""
        with open(filepath) as f:
            session = json.load(f)
        
        self.memory.messages = session["messages"]
        self.memory.summary = session["summary"]
        self.context = session["context"]
        print(f"Session loaded: {len(self.memory.messages)} messages")

# Demo
def mock_llm_assistant(system: str, user: str) -> str:
    responses = {
        "hello": "Hello! How can I help you today?",
        "python": "Python is a great language! It's versatile and easy to learn.",
        "help": "Of course! What do you need help with?",
    }
    for key, resp in responses.items():
        if key in user.lower():
            return resp
    return f"I understand you're asking about: {user[:50]}. Let me help with that!"

assistant = PersistentAIAssistant(name="PyAssist", personality="helpful")
assistant.set_context("language", "Python")
assistant.set_context("level", "intermediate")

conversations = [
    "Hello!",
    "I need help with Python",
    "Can you explain list comprehensions?",
]

print("=== AI Assistant Demo ===\n")
for msg in conversations:
    print(f"You: {msg}")
    response = assistant.chat(msg, mock_llm_assistant)
    print(f"Assistant: {response}\n")

print(f"Memory stats: {assistant.memory.get_stats()}")
```

### Example 2: Document Q&A System

```python
# ตัวอย่างที่ 23: Document Q&A with RAG
from dataclasses import dataclass, field
from typing import List, Optional, Dict
import re

@dataclass
class Document:
    """Represents a document in the system"""
    id: str
    title: str
    content: str
    metadata: Dict = field(default_factory=dict)
    chunks: List[Dict] = field(default_factory=list)

@dataclass
class QAResult:
    """Result from Q&A query"""
    question: str
    answer: str
    sources: List[str]
    confidence: float
    chunks_used: int
    processing_time: float

class DocumentQASystem:
    """
    Complete Document Q&A System with:
    - Multi-document support
    - Source citation
    - Confidence scoring
    - Query rewriting
    """
    
    def __init__(self, vector_db: SimpleVectorDB, embed_fn, llm_fn):
        self.db = vector_db
        self.embed_fn = embed_fn
        self.llm_fn = llm_fn
        self.documents = {}
    
    def add_document(self, title: str, content: str, 
                    metadata: dict = None) -> Document:
        """Add document to system"""
        doc_id = f"doc_{len(self.documents):03d}"
        doc = Document(
            id=doc_id,
            title=title,
            content=content,
            metadata=metadata or {}
        )
        
        # Chunk and index
        chunks = self._chunk_document(content)
        
        for i, chunk in enumerate(chunks):
            chunk_id = f"{doc_id}_c{i:03d}"
            embedding = self.embed_fn(chunk)
            
            self.db.add(chunk_id, embedding, {
                "doc_id": doc_id,
                "doc_title": title,
                "content": chunk,
                "chunk_index": i,
                **doc.metadata
            })
        
        doc.chunks = chunks
        self.documents[doc_id] = doc
        print(f"Added '{title}': {len(chunks)} chunks")
        return doc
    
    def _chunk_document(self, text: str, chunk_size=300, overlap=30) -> List[str]:
        """Split document into overlapping chunks"""
        # Split by paragraphs first
        paragraphs = [p.strip() for p in text.split('\n\n') if p.strip()]
        
        chunks = []
        current_words = []
        
        for para in paragraphs:
            words = para.split()
            current_words.extend(words)
            
            while len(current_words) >= chunk_size:
                chunk = ' '.join(current_words[:chunk_size])
                chunks.append(chunk)
                current_words = current_words[chunk_size - overlap:]
        
        if current_words:
            chunks.append(' '.join(current_words))
        
        return chunks
    
    def rewrite_query(self, query: str) -> List[str]:
        """Generate multiple query variants for better retrieval"""
        # Simple query expansion
        variants = [query]
        
        # Add question prefix variations
        if not query.endswith('?'):
            query_with_q = query + '?'
        else:
            query_with_q = query
        
        # Variations
        variants.extend([
            query_with_q,
            f"What is {query}",
            f"How does {query} work",
            f"Explain {query}",
        ])
        
        return list(set(variants))[:3]  # Return top 3 unique variants
    
    def _retrieve_with_variants(self, query: str, top_k: int = 5) -> List[Dict]:
        """Retrieve using multiple query variants"""
        all_results = {}
        variants = self.rewrite_query(query)
        
        for variant in variants:
            embedding = self.embed_fn(variant)
            results = self.db.search(embedding, top_k=top_k)
            
            for r in results:
                chunk_id = r['id']
                if chunk_id not in all_results:
                    all_results[chunk_id] = r
                else:
                    # Average scores from multiple queries
                    all_results[chunk_id]['score'] = max(
                        all_results[chunk_id]['score'], r['score']
                    )
        
        # Sort by score
        sorted_results = sorted(all_results.values(), key=lambda x: -x['score'])
        return sorted_results[:top_k]
    
    def query(self, question: str, top_k: int = 5) -> QAResult:
        """Answer a question"""
        import time
        start = time.time()
        
        # Retrieve
        retrieved = self._retrieve_with_variants(question, top_k=top_k)
        
        if not retrieved:
            return QAResult(
                question=question,
                answer="I don't have information about that in my knowledge base.",
                sources=[],
                confidence=0.0,
                chunks_used=0,
                processing_time=time.time() - start
            )
        
        # Build context
        context = "\n\n".join([
            f"[From: {r['metadata']['doc_title']}]\n{r['metadata']['content']}"
            for r in retrieved
        ])
        
        # Generate answer
        system = """You are a helpful assistant that answers questions based on provided documents.

Guidelines:
- Answer based ONLY on the provided context
- If uncertain, express your confidence level
- Always cite sources like: "According to [Document Title]..."
- For questions not in context, say: "This information is not in my knowledge base."
- Be concise but complete"""
        
        user_prompt = f"""Context:
{context}

Question: {question}

Please answer the question based on the context provided. Include source citations."""
        
        answer = self.llm_fn(system, user_prompt)
        
        # Calculate confidence
        max_score = max(r['score'] for r in retrieved) if retrieved else 0
        confidence = min(max_score, 1.0)
        
        # Get sources
        sources = list(set(r['metadata']['doc_title'] for r in retrieved))
        
        return QAResult(
            question=question,
            answer=answer,
            sources=sources,
            confidence=confidence,
            chunks_used=len(retrieved),
            processing_time=time.time() - start
        )

# Build Q&A system
db = SimpleVectorDB()
qa = DocumentQASystem(db, mock_embed, mock_llm)

# Add documents
python_content = """
Python was created by Guido van Rossum and released in 1991.
It is interpreted, dynamically typed, and garbage collected.
Python supports multiple programming paradigms.
Python's philosophy emphasizes code readability using significant indentation.
Common Python libraries include NumPy for numerical computing, Pandas for data analysis,
and Scikit-learn for machine learning.
Python is widely used in web development with frameworks like Django and Flask.
"""

ml_content = """
Machine learning is a subset of artificial intelligence.
Supervised learning uses labeled training data to learn a mapping function.
Unsupervised learning finds patterns in unlabeled data.
Common algorithms include linear regression, decision trees, and neural networks.
Deep learning uses multi-layer neural networks for complex pattern recognition.
Gradient descent is the key optimization algorithm used in training neural networks.
"""

qa.add_document("Python Guide", python_content)
qa.add_document("Machine Learning Basics", ml_content)

# Query
print("\n=== Document Q&A Demo ===\n")
questions = [
    "Who created Python?",
    "What is supervised learning?",
    "What libraries does Python have?",
]

for q in questions:
    result = qa.query(q, top_k=3)
    print(f"Q: {q}")
    print(f"A: {result.answer}")
    print(f"Sources: {result.sources}")
    print(f"Confidence: {result.confidence:.2f}")
    print()
```

---

## 16. แบบฝึกหัด {#exercises}

### แบบฝึกหัดที่ 1: Prompt Template Engine

```python
# เฉลย
from string import Template
from typing import Dict, List, Optional
import json, re

class PromptTemplateEngine:
    """Advanced prompt template system"""
    
    def __init__(self):
        self.templates = {}
        self.partials = {}
    
    def register(self, name: str, template: str):
        """Register a template"""
        self.templates[name] = template
    
    def register_partial(self, name: str, partial: str):
        """Register a partial (reusable snippet)"""
        self.partials[name] = partial
    
    def render(self, template_name: str, variables: Dict = None) -> str:
        """Render template with variables"""
        if template_name not in self.templates:
            raise ValueError(f"Template '{template_name}' not found")
        
        template = self.templates[template_name]
        variables = variables or {}
        
        # Replace partials
        for name, partial in self.partials.items():
            template = template.replace(f"{{{{>{name}}}}}", partial)
        
        # Replace conditional blocks {{#if var}}...{{/if}}
        def replace_conditional(match):
            var_name = match.group(1)
            content = match.group(2)
            return content if variables.get(var_name) else ""
        
        template = re.sub(r'\{\{#if (\w+)\}\}(.*?)\{\{/if\}\}', 
                         replace_conditional, template, flags=re.DOTALL)
        
        # Replace loop blocks {{#each list}}...{{/each}}
        def replace_loop(match):
            var_name = match.group(1)
            content = match.group(2)
            items = variables.get(var_name, [])
            results = []
            for i, item in enumerate(items):
                item_content = content
                if isinstance(item, dict):
                    for k, v in item.items():
                        item_content = item_content.replace(f"{{{{this.{k}}}}}", str(v))
                else:
                    item_content = item_content.replace("{{this}}", str(item))
                item_content = item_content.replace("{{@index}}", str(i))
                results.append(item_content)
            return ''.join(results)
        
        template = re.sub(r'\{\{#each (\w+)\}\}(.*?)\{\{/each\}\}',
                         replace_loop, template, flags=re.DOTALL)
        
        # Replace simple variables
        for key, value in variables.items():
            template = template.replace(f"{{{{{key}}}}}", str(value))
        
        return template.strip()
    
    def list_templates(self) -> List[str]:
        return list(self.templates.keys())

# Test
engine = PromptTemplateEngine()

# Register partials
engine.register_partial("json_reminder", "Always respond with valid JSON only.")
engine.register_partial("safety_note", "Follow ethical guidelines and be safe.")

# Register templates
engine.register("classify_text", """You are a text classifier.
{{>json_reminder}}

Classify the following text into categories: {{categories}}

Text: {{text}}

Output format:
{{
  "category": "string",
  "confidence": 0.0-1.0,
  "reasoning": "string"
}}""")

engine.register("code_review", """You are a senior {{language}} developer.
Review the following code:

```{{language}}
{{code}}
```

{{#if focus_area}}Focus specifically on: {{focus_area}}{{/if}}

{{#if checklist}}
Check these aspects:
{{#each checklist}}
- {{@index}}. {{this}}
{{/each}}
{{/if}}

Provide actionable feedback.""")

# Render templates
text_prompt = engine.render("classify_text", {
    "categories": "tech, sports, politics, entertainment",
    "text": "Python 3.12 released with improved performance"
})
print("=== Classify Template ===")
print(text_prompt[:300])

code_prompt = engine.render("code_review", {
    "language": "python",
    "code": "def add(a, b): return a + b",
    "focus_area": "type safety",
    "checklist": ["Type hints", "Error handling", "Tests"]
})
print("\n=== Code Review Template ===")
print(code_prompt[:400])
```

### แบบฝึกหัดที่ 2: LLM Rate Limiter

```python
# เฉลย
import time
import threading
from collections import deque
from typing import Callable, Any

class TokenBucket:
    """Token bucket rate limiter"""
    
    def __init__(self, rate: float, capacity: int):
        self.rate = rate        # tokens per second
        self.capacity = capacity
        self.tokens = capacity
        self.last_update = time.time()
        self.lock = threading.Lock()
    
    def consume(self, tokens: int = 1) -> bool:
        """Try to consume tokens. Returns True if successful."""
        with self.lock:
            now = time.time()
            elapsed = now - self.last_update
            
            # Add tokens based on elapsed time
            self.tokens = min(
                self.capacity,
                self.tokens + elapsed * self.rate
            )
            self.last_update = now
            
            if self.tokens >= tokens:
                self.tokens -= tokens
                return True
            return False
    
    def wait_and_consume(self, tokens: int = 1, max_wait: float = 60):
        """Wait until tokens available and consume"""
        start = time.time()
        while True:
            if self.consume(tokens):
                return True
            
            if time.time() - start > max_wait:
                raise TimeoutError(f"Couldn't acquire {tokens} tokens within {max_wait}s")
            
            time.sleep(0.1)

class LLMRateLimiter:
    """Rate limiter for LLM API calls"""
    
    def __init__(self, rpm: int = 50, tpm: int = 50_000, 
                 daily_limit: int = None):
        self.rpm_limiter = TokenBucket(rpm / 60, rpm)
        self.tpm_limiter = TokenBucket(tpm / 60, tpm)
        self.daily_limit = daily_limit
        
        self.stats = {
            "requests": 0,
            "tokens_in": 0,
            "tokens_out": 0,
            "errors": 0,
            "start_time": time.time()
        }
        self.request_times = deque(maxlen=1000)
    
    def estimate_tokens(self, text: str) -> int:
        """Rough token estimate"""
        return int(len(text.split()) * 1.3)
    
    def check_daily_limit(self):
        """Check daily token limit"""
        if self.daily_limit:
            total = self.stats["tokens_in"] + self.stats["tokens_out"]
            if total >= self.daily_limit:
                raise Exception(f"Daily limit of {self.daily_limit:,} tokens reached")
    
    def call(self, fn: Callable, prompt: str, *args, **kwargs) -> Any:
        """Make rate-limited LLM call"""
        self.check_daily_limit()
        
        # Estimate tokens
        estimated_tokens = self.estimate_tokens(prompt)
        
        # Acquire rate limits
        self.rpm_limiter.wait_and_consume(1)
        self.tpm_limiter.wait_and_consume(estimated_tokens)
        
        # Make call
        start = time.time()
        try:
            result = fn(prompt, *args, **kwargs)
            elapsed = time.time() - start
            
            # Update stats
            self.stats["requests"] += 1
            self.stats["tokens_in"] += estimated_tokens
            output_tokens = self.estimate_tokens(str(result))
            self.stats["tokens_out"] += output_tokens
            self.request_times.append(elapsed)
            
            return result
        
        except Exception as e:
            self.stats["errors"] += 1
            raise
    
    def get_stats(self) -> dict:
        elapsed = time.time() - self.stats["start_time"]
        times = list(self.request_times)
        
        return {
            **self.stats,
            "avg_time": sum(times) / len(times) if times else 0,
            "current_rpm": self.stats["requests"] / (elapsed / 60) if elapsed > 0 else 0,
            "total_tokens": self.stats["tokens_in"] + self.stats["tokens_out"]
        }

# Test
limiter = LLMRateLimiter(rpm=60, tpm=100_000)

def mock_api_call(prompt: str) -> str:
    time.sleep(0.01)  # Simulate API latency
    return f"Response to: {prompt[:50]}"

print("=== Rate Limiter Test ===")
for i in range(5):
    result = limiter.call(mock_api_call, f"Query number {i+1}: What is Python?")
    print(f"  Call {i+1}: {result}")

stats = limiter.get_stats()
print(f"\nStats:")
for k, v in stats.items():
    if isinstance(v, float):
        print(f"  {k}: {v:.2f}")
    else:
        print(f"  {k}: {v}")
```

---

## สรุป

ใน Part 84 นี้ เราได้เรียนรู้:

1. **LLM Concepts** - tokens, context window, temperature, sampling
2. **Anthropic Claude API** - models, features, rate limits
3. **Python SDK** - client setup, authentication
4. **Messages API** - request/response format, content types
5. **Streaming** - event types, real-time output
6. **System Prompts** - patterns and best practices
7. **Prompt Engineering** - zero/few-shot, CoT, structured output
8. **Tool Use** - function calling with JSON schemas
9. **Vision** - image analysis capabilities
10. **OpenAI Comparison** - API differences
11. **LangChain** - high-level abstractions
12. **Vector Databases** - embeddings and similarity search
13. **RAG** - retrieval augmented generation pipeline
14. **AI Applications** - code assistant, document Q&A

### ขั้นต่อไป
- Part 85: Project - AI-Powered Document Assistant
- ศึกษาเพิ่มเติม: https://docs.anthropic.com/

---
*Part 84 - LLM Integration: Anthropic API & OpenAI | Python Course*
