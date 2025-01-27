# chatsisst
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple ChatGPT Interface</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <div class="chat-container">
            <div class="chat-header">
                <h1>ChatGPT Interface</h1>
            </div>
            <div class="chat-messages" id="chat-messages">
                <!-- Messages will appear here -->
            </div>
            <div class="chat-input">
                <input type="text" id="api-key" placeholder="Enter your OpenAI API key" class="api-input">
                <div class="message-input-container">
                    <input type="text" id="user-input" placeholder="Type your message..." class="message-input">
                    <button onclick="sendMessage()" id="send-button">Send</button>
                </div>
            </div>
        </div>
    </div>
    <script src="script.js"></script>
</body>
</html>

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: Arial, sans-serif;
}

.container {
    width: 100%;
    min-height: 100vh;
    background: #f5f5f5;
    padding: 20px;
}

.chat-container {
    max-width: 800px;
    margin: 0 auto;
    background: white;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0,0,0,0.1);
    overflow: hidden;
}

.chat-header {
    background: #075e54;
    color: white;
    padding: 20px;
    text-align: center;
}

.chat-messages {
    padding: 20px;
    height: 60vh;
    overflow-y: auto;
}

.message {
    margin-bottom: 20px;
    padding: 10px 15px;
    border-radius: 5px;
    max-width: 70%;
    word-wrap: break-word;
}

.user-message {
    background: #dcf8c6;
    margin-left: auto;
}

.bot-message {
    background: #e8e8e8;
}

.chat-input {
    padding: 20px;
    border-top: 1px solid #eee;
}

.api-input {
    width: 100%;
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
}

.message-input-container {
    display: flex;
    gap: 10px;
}

.message-input {
    flex-grow: 1;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
}

button {
    padding: 10px 20px;
    background: #075e54;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

button:hover {
    background: #064942;
}

.error {
    color: red;
    margin-top: 5px;
    font-size: 14px;
}

.loading {
    color: #666;
    font-style: italic;
}

let isProcessing = false;

function addMessage(message, isUser) {
    const chatMessages = document.getElementById('chat-messages');
    const messageDiv = document.createElement('div');
    messageDiv.className = `message ${isUser ? 'user-message' : 'bot-message'}`;
    messageDiv.textContent = message;
    chatMessages.appendChild(messageDiv);
    chatMessages.scrollTop = chatMessages.scrollHeight;
}

async function sendMessage() {
    if (isProcessing) return;

    const apiKey = document.getElementById('api-key').value.trim();
    const userInput = document.getElementById('user-input').value.trim();
    const sendButton = document.getElementById('send-button');

    if (!apiKey) {
        alert('Please enter your OpenAI API key');
        return;
    }

    if (!userInput) return;

    isProcessing = true;
    sendButton.disabled = true;
    addMessage(userInput, true);
    document.getElementById('user-input').value = '';

    try {
        const response = await fetch('https://api.openai.com/v1/chat/completions', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${apiKey}`
            },
            body: JSON.stringify({
                model: 'gpt-3.5-turbo',
                messages: [{
                    role: 'user',
                    content: userInput
                }],
                temperature: 0.7
            })
        });

        const data = await response.json();

        if (response.ok) {
            const botResponse = data.choices[0].message.content;
            addMessage(botResponse, false);
        } else {
            addMessage(`Error: ${data.error.message}`, false);
        }
    } catch (error) {
        addMessage(`Error: ${error.message}`, false);
    } finally {
        isProcessing = false;
        sendButton.disabled = false;
    }
}

// Allow sending message with Enter key
document.getElementById('user-input').addEventListener('keypress', function(e) {
    if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault();
        sendMessage();
    }
});
