import React, { useState, useEffect } from "react";
import io from "socket.io-client";
import "./Chat.css";

let socket;

const Chat = () => {
  const [messages, setMessages] = useState([]);
  const [message, setMessage] = useState("");
  const [status, setStatus] = useState("Connecting...");
  const [isConnected, setIsConnected] = useState(false);
  const [socketId, setSocketId] = useState("");
  const [users, setUsers] = useState({});
  const [selectedUser, setSelectedUser] = useState(null);
  const [privateMessages, setPrivateMessages] = useState({});

  const connectSocket = () => {
    socket = io("http://localhost:5000", {
      reconnection: true,
      reconnectionAttempts: Infinity,
      reconnectionDelay: 1000,
      reconnectionDelayMax: 5000,
      randomizationFactor: 0.5,
      transports: ["websocket"],
    });

    socket.on("connect", () => {
      console.log("Connected to server");
      setStatus("Connected");
      setIsConnected(true);
      setSocketId(socket.id);
    });

    socket.on("users", (users) => {
      setUsers(users);
    });

    socket.on("private_message", ({ senderId, message }) => {
      setPrivateMessages((prevMessages) => ({
        ...prevMessages,
        [senderId]: [
          ...(prevMessages[senderId] || []),
          { message, sender: senderId },
        ],
      }));
    });

    socket.on("disconnect", (reason) => {
      console.log(`Disconnected from server: ${reason}`);
      setStatus("Disconnected. Trying to reconnect...");
      setIsConnected(false);
    });

    socket.on("reconnect_attempt", () => {
      console.log("Reconnecting...");
      setStatus("Reconnecting...");
    });

    socket.on("reconnect", () => {
      console.log("Reconnected to server");
      setStatus("Connected");
      setIsConnected(true);
    });

    socket.on("reconnect_failed", () => {
      console.log("Reconnection failed");
      setStatus("Reconnection failed");
    });

    socket.on("connect_error", (err) => {
      console.log(`Connection error: ${err.message}`);
      setStatus(`Connection error: ${err.message}`);
    });
  };

  useEffect(() => {
    connectSocket();

    return () => {
      if (socket) {
        socket.disconnect();
      }
    };
  }, []);

  const sendMessage = (e) => {
    e.preventDefault();
    if (message && selectedUser) {
      socket.emit("private_message", { recipientId: selectedUser, message });
      setPrivateMessages((prevMessages) => ({
        ...prevMessages,
        [selectedUser]: [
          ...(prevMessages[selectedUser] || []),
          { message, sender: socket.id },
        ],
      }));
      setMessage("");
    }
  };

  const handleReconnect = () => {
    if (!isConnected) {
      connectSocket();
    }
  };

  return (
    <div className="chat-container">
      <h1>Chat Room</h1>
      <p>Status: {status}</p>
      <p>Connection ID: {socketId}</p>
      {!isConnected && <button onClick={handleReconnect}>Reconnect</button>}
      <div className="chat-content">
        <div className="users-list">
          <h2>Users</h2>
          {/* <ul>
                        {Object.keys(users).map((userId) => (
                            <li key={userId} onClick={() => setSelectedUser(userId)} className={selectedUser === userId ? 'selected' : ''}>
                                {userId} {userId === socketId && '(You)'}
                            </li>
                        ))}
                    </ul> */}
          <ul>
            {Object.keys(users).map((userId) => (
              <li
                key={userId}
                onClick={() => setSelectedUser(userId)}
                className={selectedUser === userId ? "selected" : ""}
              >
                {/* {userId} {userId === socketId && "(You)"} */}
                {userId === socketId ? "" : userId}
              </li>
            ))}
          </ul>
        </div>
        <div className="chat-box">
          <h2>Private Chat</h2>
          {selectedUser ? (
            <div className="chat-section">
              <div className="messages">
                {privateMessages[selectedUser] &&
                  privateMessages[selectedUser].map((msg, index) => (
                    <p key={index}>
                      <strong>
                        {msg.sender === socket.id ? "You" : msg.sender}:
                      </strong>{" "}
                      {msg.message}
                    </p>
                  ))}
              </div>
              <form onSubmit={sendMessage} className="message-form">
                <input
                  type="text"
                  value={message}
                  onChange={(e) => setMessage(e.target.value)}
                  placeholder="Type a message"
                />
                <button type="submit">Send</button>
              </form>
            </div>
          ) : (
            <p>You haven't started a conversation yet.</p>
          )}
        </div>
      </div>
    </div>
  );
};

export default Chat;



#css

body {
    font-family: Arial, sans-serif;
    background-color: #f5f5f5;
    margin: 0;
    padding: 0;
}

.chat-container {
    width: 100%;
    height: 100vh;
    display: flex;
    flex-direction: column;
    padding: 20px;
    background-color: #f3e6ff;
}

h1 {
    text-align: center;
    color: #6b21a8;
}

p {
    text-align: center;
    color: #6b21a8;
}

button {
    display: block;
    margin: 10px auto;
    padding: 10px 20px;
    background-color: #6b21a8;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.chat-content {
    display: flex;
    height: 90%;
}

.users-list {
    width: 30%;
    padding: 10px;
    background-color: #ffffff;
    border-radius: 10px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
    overflow-y: auto;
}

.users-list ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.users-list li {
    padding: 10px;
    cursor: pointer;
    border-bottom: 1px solid #ececec;
}

.users-list li:hover, .users-list .selected {
    background-color: #e0bbff;
}

.chat-box {
    width: 70%;
    padding: 10px;
    background-color: #ffffff;
    border-radius: 10px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: column;
}

.chat-box h2, .chat-box p {
    text-align: left;
    color: #6b21a8;
}

.chat-section {
    display: flex;
    flex-direction: column;
    flex-grow: 1;
    justify-content: space-between;
}

.messages {
    flex-grow: 1;
    overflow-y: auto;
    margin-bottom: 10px;
}

.messages p {
    margin: 5px 0;
}

.message-form {
    display: flex;
    justify-content: space-between;
    padding-top: 10px;
    border-top: 1px solid #d1c4e9;
}

.message-form input {
    flex: 1;
    padding: 10px;
    border: 1px solid #d1c4e9;
    border-radius: 5px;
    margin-right: 10px;
}

.message-form button {
    padding: 10px 20px;
    background-color: #6b21a8;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}
