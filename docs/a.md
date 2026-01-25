
# Admin Log Socket Token (optional - any token works in development)
ADMIN_LOG_TOKEN=your-secure-secret-token-here
Note: In development mode (NODE_ENV=development), any token is accepted. In production, the token must match.

2. Admin Panel Setup (Frontend)
Install Socket.IO Client

npm install socket.io-client
Connect to Logs

import { io } from 'socket.io-client';

// Connect to your server
const socket = io('https://your-server-url.com', {
  transports: ['websocket', 'polling']
});

// Join the logs room with token
socket.emit('join-admin-logs', { 
  adminToken: 'your-secure-secret-token-here'  // Must match ADMIN_LOG_TOKEN
});

// Confirmation
socket.on('admin-logs-joined', (response) => {
  if (response.success) {
    console.log('Connected to server logs!');
  } else {
    console.error('Failed:', response.message);
  }
});

// Receive all server logs in real-time
socket.on('server-log', (log) => {
  // log structure:
  // {
  //   id: "1706123456789-abc123def",
  //   timestamp: "2024-01-24T10:30:45.123Z",
  //   level: "log" | "info" | "warn" | "error" | "debug",
  //   message: "The log message content",
  //   source: {
  //     file: "orderController.js",
  //     line: 142,
  //     function: "createOrder"
  //   },
  //   stack: "Error stack trace (only for error/warn)"
  // }
  
  console.log(`[${log.level.toUpperCase()}] ${log.message}`);
});

// Disconnect when leaving
socket.emit('leave-admin-logs');
3. React Component Example

import { useEffect, useState } from 'react';
import { io } from 'socket.io-client';

const SERVER_URL = process.env.REACT_APP_SERVER_URL;
const ADMIN_LOG_TOKEN = process.env.REACT_APP_ADMIN_LOG_TOKEN;

export default function LogViewer() {
  const [logs, setLogs] = useState([]);
  const [connected, setConnected] = useState(false);

  useEffect(() => {
    const socket = io(SERVER_URL);

    socket.emit('join-admin-logs', { adminToken: ADMIN_LOG_TOKEN });

    socket.on('admin-logs-joined', (res) => {
      setConnected(res.success);
    });

    socket.on('server-log', (log) => {
      setLogs(prev => [log, ...prev].slice(0, 500)); // Keep last 500
    });

    return () => {
      socket.emit('leave-admin-logs');
      socket.disconnect();
    };
  }, []);

  const getLevelColor = (level) => {
    const colors = {
      error: '#ff4444',
      warn: '#ffaa00', 
      info: '#4488ff',
      debug: '#888888',
      log: '#ffffff'
    };
    return colors[level] || '#ffffff';
  };

  return (
    <div style={{ background: '#1e1e1e', color: '#fff', padding: 20, fontFamily: 'monospace' }}>
      <h2>Server Logs {connected ? '🟢' : '🔴'}</h2>
      <div style={{ maxHeight: '80vh', overflow: 'auto' }}>
        {logs.map(log => (
          <div key={log.id} style={{ 
            borderLeft: `3px solid ${getLevelColor(log.level)}`,
            padding: '4px 8px',
            marginBottom: 4
          }}>
            <span style={{ color: '#888' }}>{new Date(log.timestamp).toLocaleTimeString()}</span>
            <span style={{ color: getLevelColor(log.level), marginLeft: 8 }}>[{log.level.toUpperCase()}]</span>
            <span style={{ color: '#aaa', marginLeft: 8 }}>{log.source?.file}:{log.source?.line}</span>
            <div style={{ marginLeft: 16 }}>{log.message}</div>
          </div>
        ))}
      </div>
    </div>
  );
}
4. Check Status via API

GET /api/management/logs/status
Authorization: Bearer <management-jwt-token>
Requires canSeeLogs permission in user's role.

Summary
Item	Value
Env Variable	ADMIN_LOG_TOKEN=your-secret
Socket Event (join)	join-admin-logs
Socket Event (receive)	server-log
API Endpoint	GET /api/management/logs/status
Permission Required	canSeeLogs
