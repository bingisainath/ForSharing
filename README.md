# For Sharing


// import React, { useState, useEffect } from 'react';
// import io from 'socket.io-client';

// let socket;

// const Chat = () => {
//     const [messages, setMessages] = useState([]);
//     const [message, setMessage] = useState('');
//     const [status, setStatus] = useState('Connecting...');
//     const [isConnected, setIsConnected] = useState(false);
//     const [socketId, setSocketId] = useState('');

//     const connectSocket = () => {
//         socket = io('http://localhost:5000', {
//             reconnection: true,
//             reconnectionAttempts: Infinity,
//             reconnectionDelay: 1000,
//             reconnectionDelayMax: 5000,
//             randomizationFactor: 0.5,
//             transports: ['websocket'] // Use only WebSocket transport
//         });

//         socket.on('connect', () => {
//             console.log('Connected to server');
//             setStatus('Connected');
//             setIsConnected(true);
//             setSocketId(socket.id);
//         });

//         socket.on('message', (messageData) => {
//             setMessages((prevMessages) => [...prevMessages, messageData]);
//         });

//         socket.on('disconnect', (reason) => {
//             console.log(`Disconnected from server: ${reason}`);
//             setStatus('Disconnected. Trying to reconnect...');
//             setIsConnected(false);
//         });

//         socket.on('reconnect_attempt', () => {
//             console.log('Reconnecting...');
//             setStatus('Reconnecting...');
//         });

//         socket.on('reconnect', () => {
//             console.log('Reconnected to server');
//             setStatus('Connected');
//             setIsConnected(true);
//         });

//         socket.on('reconnect_failed', () => {
//             console.log('Reconnection failed');
//             setStatus('Reconnection failed');
//         });

//         socket.on('connect_error', (err) => {
//             console.log(`Connection error: ${err.message}`);
//             setStatus(`Connection error: ${err.message}`);
//         });
//     };

//     useEffect(() => {
//         connectSocket();

//         return () => {
//             if (socket) {
//                 socket.disconnect();
//             }
//         };
//     }, []);

//     const sendMessage = (e) => {
//         e.preventDefault();
//         if (message) {
//             socket.emit('message', { message, id: socket.id });
//             setMessage('');
//         }
//     };

//     const handleReconnect = () => {
//         if (!isConnected) {
//             connectSocket();
//         }
//     };

//     return (
//         <div>
//             <h1>Chat Room</h1>
//             <p>Status: {status}</p>
//             <p>Connection ID: {socketId}</p>
//             {!isConnected && (
//                 <button onClick={handleReconnect}>Reconnect</button>
//             )}
//             <div>
//                 {messages
//                     .filter((msg) => msg.id !== socketId)
//                     .map((msg, index) => (
//                         <p key={index}>{msg.message}</p>
//                     ))}
//             </div>
//             <form onSubmit={sendMessage}>
//                 <input
//                     type="text"
//                     value={message}
//                     onChange={(e) => setMessage(e.target.value)}
//                 />
//                 <button type="submit">Send</button>
//             </form>
//         </div>
//     );
// };

// export default Chat;


import React, { useState, useEffect } from 'react';
import io from 'socket.io-client';

let socket;

const Chat = () => {
    const [messages, setMessages] = useState([]);
    const [message, setMessage] = useState('');
    const [status, setStatus] = useState('Connecting...');
    const [isConnected, setIsConnected] = useState(false);
    const [socketId, setSocketId] = useState('');
    const [users, setUsers] = useState({});
    const [selectedUser, setSelectedUser] = useState(null);
    const [privateMessages, setPrivateMessages] = useState({});

    const connectSocket = () => {
        socket = io('http://localhost:5000', {
            reconnection: true,
            reconnectionAttempts: Infinity,
            reconnectionDelay: 1000,
            reconnectionDelayMax: 5000,
            randomizationFactor: 0.5,
            transports: ['websocket']
        });

        socket.on('connect', () => {
            console.log('Connected to server');
            setStatus('Connected');
            setIsConnected(true);
            setSocketId(socket.id);
        });

        socket.on('users', (users) => {
            setUsers(users);
        });

        socket.on('private_message', ({ senderId, message }) => {
            setPrivateMessages((prevMessages) => ({
                ...prevMessages,
                [senderId]: [...(prevMessages[senderId] || []), { message, sender: senderId }]
            }));
        });

        socket.on('disconnect', (reason) => {
            console.log(`Disconnected from server: ${reason}`);
            setStatus('Disconnected. Trying to reconnect...');
            setIsConnected(false);
        });

        socket.on('reconnect_attempt', () => {
            console.log('Reconnecting...');
            setStatus('Reconnecting...');
        });

        socket.on('reconnect', () => {
            console.log('Reconnected to server');
            setStatus('Connected');
            setIsConnected(true);
        });

        socket.on('reconnect_failed', () => {
            console.log('Reconnection failed');
            setStatus('Reconnection failed');
        });

        socket.on('connect_error', (err) => {
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
            socket.emit('private_message', { recipientId: selectedUser, message });
            setPrivateMessages((prevMessages) => ({
                ...prevMessages,
                [selectedUser]: [...(prevMessages[selectedUser] || []), { message, sender: socket.id }]
            }));
            setMessage('');
        }
    };

    const handleReconnect = () => {
        if (!isConnected) {
            connectSocket();
        }
    };

    return (
        <div>
            <h1>Chat Room</h1>
            <p>Status: {status}</p>
            <p>Connection ID: {socketId}</p>
            {!isConnected && (
                <button onClick={handleReconnect}>Reconnect</button>
            )}
            <div>
                <h2>Users</h2>
                <ul>
                    {Object.keys(users).map((userId) => (
                        <li key={userId} onClick={() => setSelectedUser(userId)}>
                            {userId} {userId === socketId && '(You)'}
                        </li>
                    ))}
                </ul>
            </div>
            <div>
                <h2>Private Chat</h2>
                {selectedUser && (
                    <div>
                        <h3>Chatting with {selectedUser}</h3>
                        <div>
                            {privateMessages[selectedUser] && privateMessages[selectedUser].map((msg, index) => (
                                <p key={index}><strong>{msg.sender === socket.id ? 'You' : msg.sender}:</strong> {msg.message}</p>
                            ))}
                        </div>
                        <form onSubmit={sendMessage}>
                            <input
                                type="text"
                                value={message}
                                onChange={(e) => setMessage(e.target.value)}
                            />
                            <button type="submit">Send</button>
                        </form>
                    </div>
                )}
            </div>
        </div>
    );
};

export default Chat;
