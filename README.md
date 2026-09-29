const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server);

// Backend Real-Time Logic
io.on('connection', (socket) => {
  socket.on('join_room', ({ username, room }) => {
    socket.join(room);
  });

  socket.on('send_message', (data) => {
    io.to(data.room).emit('receive_message', data);
  });
});

// Frontend HTML + React Code
app.get('/', (req, res) => {
  res.send(`
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Hyy Buddy - Anu & Paru</title>
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script src="/socket.io/socket.io.js"></script>
  
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: sans-serif; }
    body { background-color: #d1d7db; display: flex; justify-content: center; align-items: center; height: 100vh; }
    .chat-box { width: 380px; height: 550px; background: #efeae2; border-radius: 8px; display: flex; flex-direction: column; overflow: hidden; box-shadow: 0 4px 15px rgba(0,0,0,0.2); }
    .chat-header { background: #075e54; color: white; padding: 15px; }
    .chat-body { flex: 1; padding: 15px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; }
    .msg { padding: 8px 12px; border-radius: 8px; max-width: 75%; font-size: 14px; }
    .msg.you { background: #d9fdd3; align-self: flex-end; }
    .msg.other { background: #ffffff; align-self: flex-start; }
    .meta { font-size: 10px; color: #777; margin-top: 4px; display: flex; justify-content: space-between; gap: 10px; }
    .chat-footer { display: flex; padding: 10px; background: #f0f2f5; }
    .chat-footer input { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 4px; outline: none; }
    .chat-footer button { padding: 10px 15px; background: #00a884; color: white; border: none; border-radius: 4px; margin-left: 5px; cursor: pointer; }
  </style>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    const socket = io();

    function App() {
      const [username, setUsername] = React.useState('Anu');
      const [room] = React.useState('Anu-Paru-Room');
      const [message, setMessage] = React.useState('');
      
      // Default conversation state pre-populated with Anu and Paru messages
      const [messages, setMessages] = React.useState([
        { author: 'Anu', message: 'Hello how are u', time: '10:00 AM' },
        { author: 'Paru', message: 'hi fine', time: '10:01 AM' }
      ]);

      const sendMessage = () => {
        if (message) {
          const data = {
            room,
            author: username,
            message,
            time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
          };
          socket.emit('send_message', data);
          setMessage('');
        }
      };

      React.useEffect(() => {
        socket.emit('join_room', { username, room });
        socket.on('receive_message', (data) => {
          setMessages((prev) => [...prev, data]);
        });
        return () => socket.off('receive_message');
      }, []);

      return (
        <div className="chat-box">
          <div className="chat-header">
            <h3>Hyy Buddy: Anu & Paru</h3>
            <span style={{fontSize: "12px", opacity: 0.8}}>Active User: {username}</span>
          </div>
          <div className="chat-body">
            {messages.map((m, idx) => (
              <div key={idx} className={`msg ${m.author === username ? 'you' : 'other'}`}>
                <div>{m.message}</div>
                <div className="meta">
                  <b>{m.author === username ? 'Anu (You)' : m.author}</b>
                  <span>{m.time}</span>
                </div>
              </div>
            ))}
          </div>
          <div className="chat-footer">
            <input 
              value={message} 
              placeholder="Type a message..." 
              onChange={(e) => setMessage(e.target.value)}
              onKeyDown={(e) => e.key === 'Enter' && sendMessage()}
            />
            <button onClick={sendMessage}>Send</button>
          </div>
        </div>
      );
    }

    ReactDOM.createRoot(document.getElementById('root')).render(<App />);
  </script>
</body>
</html>
  `);
});

const PORT = 3000;
server.listen(PORT, () => console.log(`Running on http://localhost:${PORT}`));
