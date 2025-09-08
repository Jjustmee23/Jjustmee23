# ⚛️ React - JavaScript Library for Building User Interfaces

![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)

## 📋 What is React?

**React** is a free and open-source front-end JavaScript library for building user interfaces based on components. Developed by Meta (formerly Facebook), React is maintained by Meta and a community of individual developers and companies. React can be used to develop single-page, mobile, or server-rendered applications.

## 🔧 Key Features

- **Component-Based** - Build encapsulated components that manage their own state
- **Virtual DOM** - Efficient rendering and updates
- **JSX Syntax** - Write HTML-like syntax in JavaScript
- **Unidirectional Data Flow** - Predictable state management
- **Rich Ecosystem** - Vast library of third-party packages
- **Developer Tools** - Excellent debugging and development experience

## 💡 Why I Use React

### 🏭 Industrial Dashboard Applications
- **Real-time Mill Monitoring** - Live sensor data visualization
- **Interactive Control Panels** - Equipment control interfaces
- **Data Analytics Dashboards** - Complex data presentation
- **Mobile-Responsive Design** - Works on all devices

### 🎮 Gaming Platform Development
- **Community Interfaces** - User interaction and social features
- **Real-time Gaming Stats** - Live tournament and player data
- **Interactive Game Elements** - Dynamic gaming components
- **Responsive Gaming UI** - Optimized for different screen sizes

## 🛠️ My React Architecture

### Modern React Setup:
```javascript
// Project structure for mill management dashboard
src/
├── components/
│   ├── common/
│   │   ├── Header.jsx
│   │   ├── Sidebar.jsx
│   │   └── LoadingSpinner.jsx
│   ├── dashboard/
│   │   ├── MillOverview.jsx
│   │   ├── SensorGrid.jsx
│   │   └── AlertPanel.jsx
│   └── sensors/
│       ├── SensorCard.jsx
│       ├── SensorChart.jsx
│       └── SensorConfig.jsx
├── hooks/
│   ├── useWebSocket.js
│   ├── useSensorData.js
│   └── useMillStatus.js
├── services/
│   ├── api.js
│   ├── websocket.js
│   └── mqtt.js
├── store/
│   ├── index.js
│   ├── millSlice.js
│   └── sensorSlice.js
├── utils/
│   ├── formatters.js
│   ├── constants.js
│   └── helpers.js
└── App.jsx
```

### Package.json Configuration:
```json
{
  "name": "mill-management-dashboard",
  "version": "2.1.0",
  "private": true,
  "dependencies": {
    "@reduxjs/toolkit": "^1.9.7",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.8.1",
    "react-redux": "^8.0.5",
    "axios": "^1.3.4",
    "socket.io-client": "^4.6.1",
    "recharts": "^2.5.0",
    "antd": "^5.3.0",
    "@ant-design/icons": "^5.0.1",
    "moment": "^2.29.4",
    "mqtt": "^4.3.7"
  },
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^5.16.5",
    "@testing-library/user-event": "^14.4.3",
    "eslint": "^8.36.0",
    "prettier": "^2.8.4",
    "husky": "^8.0.3",
    "lint-staged": "^13.2.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  }
}
```

## 🌟 My React Projects

### 1. 🏭 Mill Management Dashboard
```jsx
// Real-time mill monitoring dashboard
import React, { useState, useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { Card, Row, Col, Alert, Spin } from 'antd';
import { 
  TemperatureOutlined, 
  DashboardOutlined, 
  AlertOutlined 
} from '@ant-design/icons';
import useWebSocket from '../hooks/useWebSocket';
import SensorChart from '../components/SensorChart';
import { updateSensorData } from '../store/sensorSlice';

const MillDashboard = () => {
  const dispatch = useDispatch();
  const { sensors, alerts, isLoading } = useSelector(state => state.mill);
  
  // WebSocket connection for real-time data
  const { lastMessage, connectionStatus } = useWebSocket('ws://localhost:8001/ws/mill/');
  
  useEffect(() => {
    if (lastMessage) {
      const sensorData = JSON.parse(lastMessage.data);
      dispatch(updateSensorData(sensorData));
    }
  }, [lastMessage, dispatch]);
  
  const criticalAlerts = alerts.filter(alert => alert.severity === 'critical');
  
  if (isLoading) {
    return (
      <div style={{ textAlign: 'center', padding: '50px' }}>
        <Spin size="large" />
      </div>
    );
  }
  
  return (
    <div className="mill-dashboard">
      <Row gutter={[16, 16]}>
        {/* Connection Status */}
        <Col span={24}>
          <Alert
            message={`Connection Status: ${connectionStatus}`}
            type={connectionStatus === 'Connected' ? 'success' : 'error'}
            showIcon
          />
        </Col>
        
        {/* Critical Alerts */}
        {criticalAlerts.length > 0 && (
          <Col span={24}>
            <Card title={<><AlertOutlined /> Critical Alerts</>} type="inner">
              {criticalAlerts.map(alert => (
                <Alert
                  key={alert.id}
                  message={alert.message}
                  type="error"
                  showIcon
                  style={{ marginBottom: 8 }}
                />
              ))}
            </Card>
          </Col>
        )}
        
        {/* Sensor Overview */}
        <Col span={24}>
          <Card title={<><DashboardOutlined /> Mill Overview</>}>
            <Row gutter={[16, 16]}>
              {sensors.map(sensor => (
                <Col xs={24} sm={12} md={8} lg={6} key={sensor.id}>
                  <Card
                    size="small"
                    title={sensor.name}
                    extra={<TemperatureOutlined />}
                  >
                    <div className="sensor-value">
                      <span className="value">{sensor.currentValue}</span>
                      <span className="unit">{sensor.unit}</span>
                    </div>
                    <div className="sensor-status">
                      Status: <span className={`status ${sensor.status}`}>
                        {sensor.status}
                      </span>
                    </div>
                  </Card>
                </Col>
              ))}
            </Row>
          </Card>
        </Col>
        
        {/* Real-time Charts */}
        <Col span={24}>
          <Card title="Real-time Sensor Data">
            <Row gutter={[16, 16]}>
              {sensors.slice(0, 4).map(sensor => (
                <Col xs={24} lg={12} key={sensor.id}>
                  <SensorChart 
                    sensorId={sensor.id}
                    title={sensor.name}
                    data={sensor.historicalData}
                  />
                </Col>
              ))}
            </Row>
          </Card>
        </Col>
      </Row>
    </div>
  );
};

export default MillDashboard;
```

### 2. 🎮 Gaming Community Platform
```jsx
// Gaming community interface with real-time features
import React, { useState, useEffect } from 'react';
import { Layout, Menu, Avatar, Badge, Tooltip } from 'antd';
import { 
  UserOutlined, 
  TrophyOutlined, 
  MessageOutlined,
  GamepadOutlined 
} from '@ant-design/icons';
import { useSocket } from '../hooks/useSocket';
import GameRoom from './GameRoom';
import ChatPanel from './ChatPanel';
import PlayerStats from './PlayerStats';

const { Header, Sider, Content } = Layout;

const GamingPlatform = () => {
  const [activeUsers, setActiveUsers] = useState([]);
  const [currentRoom, setCurrentRoom] = useState(null);
  const [messages, setMessages] = useState([]);
  
  const socket = useSocket('http://localhost:3001');
  
  useEffect(() => {
    if (socket) {
      socket.on('user-joined', (user) => {
        setActiveUsers(prev => [...prev, user]);
      });
      
      socket.on('user-left', (userId) => {
        setActiveUsers(prev => prev.filter(u => u.id !== userId));
      });
      
      socket.on('new-message', (message) => {
        setMessages(prev => [...prev, message]);
      });
      
      socket.on('game-invitation', (invitation) => {
        // Handle game invitations
        showGameInvitation(invitation);
      });
    }
    
    return () => {
      if (socket) {
        socket.off('user-joined');
        socket.off('user-left');
        socket.off('new-message');
        socket.off('game-invitation');
      }
    };
  }, [socket]);
  
  return (
    <Layout style={{ minHeight: '100vh' }}>
      <Header className="gaming-header">
        <div className="logo">
          <GamepadOutlined /> Gaming Hub
        </div>
        <div className="user-info">
          <Badge count={messages.filter(m => !m.read).length}>
            <Avatar icon={<UserOutlined />} />
          </Badge>
        </div>
      </Header>
      
      <Layout>
        <Sider width={250} className="gaming-sider">
          <div className="active-users">
            <h4>Online Players ({activeUsers.length})</h4>
            {activeUsers.map(user => (
              <div key={user.id} className="user-item">
                <Badge status="success" />
                <Avatar size="small" src={user.avatar} />
                <span className="username">{user.username}</span>
                <Tooltip title={`Level ${user.level}`}>
                  <TrophyOutlined className="level-icon" />
                </Tooltip>
              </div>
            ))}
          </div>
        </Sider>
        
        <Layout>
          <Content className="gaming-content">
            {currentRoom ? (
              <GameRoom 
                room={currentRoom} 
                onLeaveRoom={() => setCurrentRoom(null)}
              />
            ) : (
              <div className="game-lobby">
                <PlayerStats />
                <GameRoomList onJoinRoom={setCurrentRoom} />
              </div>
            )}
          </Content>
          
          <Sider width={300} className="chat-sider">
            <ChatPanel 
              messages={messages}
              onSendMessage={(message) => {
                socket.emit('send-message', message);
              }}
            />
          </Sider>
        </Layout>
      </Layout>
    </Layout>
  );
};

export default GamingPlatform;
```

### 3. 📊 Custom Hooks for Data Management
```javascript
// Custom hook for WebSocket connections
import { useState, useEffect, useRef } from 'react';

export const useWebSocket = (url, options = {}) => {
  const [socket, setSocket] = useState(null);
  const [lastMessage, setLastMessage] = useState(null);
  const [connectionStatus, setConnectionStatus] = useState('Connecting');
  const [messageHistory, setMessageHistory] = useState([]);
  
  const reconnectTimeouts = useRef([]);
  const maxReconnectAttempts = options.maxReconnectAttempts || 5;
  const reconnectInterval = options.reconnectInterval || 3000;
  
  useEffect(() => {
    let ws;
    let reconnectAttempts = 0;
    
    const connect = () => {
      try {
        ws = new WebSocket(url);
        
        ws.onopen = () => {
          setConnectionStatus('Connected');
          setSocket(ws);
          reconnectAttempts = 0;
        };
        
        ws.onmessage = (event) => {
          const message = {
            data: event.data,
            timestamp: Date.now()
          };
          setLastMessage(message);
          setMessageHistory(prev => [...prev.slice(-99), message]);
        };
        
        ws.onclose = () => {
          setConnectionStatus('Disconnected');
          setSocket(null);
          
          // Attempt to reconnect
          if (reconnectAttempts < maxReconnectAttempts) {
            const timeout = setTimeout(() => {
              reconnectAttempts++;
              setConnectionStatus(`Reconnecting... (${reconnectAttempts}/${maxReconnectAttempts})`);
              connect();
            }, reconnectInterval);
            
            reconnectTimeouts.current.push(timeout);
          } else {
            setConnectionStatus('Failed to connect');
          }
        };
        
        ws.onerror = (error) => {
          console.error('WebSocket error:', error);
          setConnectionStatus('Error');
        };
        
      } catch (error) {
        console.error('Failed to create WebSocket:', error);
        setConnectionStatus('Error');
      }
    };
    
    connect();
    
    return () => {
      reconnectTimeouts.current.forEach(clearTimeout);
      if (ws) {
        ws.close();
      }
    };
  }, [url, maxReconnectAttempts, reconnectInterval]);
  
  const sendMessage = (message) => {
    if (socket && socket.readyState === WebSocket.OPEN) {
      socket.send(typeof message === 'string' ? message : JSON.stringify(message));
    }
  };
  
  return {
    socket,
    lastMessage,
    connectionStatus,
    messageHistory,
    sendMessage
  };
};

// Custom hook for sensor data management
export const useSensorData = (sensorId, interval = 5000) => {
  const [data, setData] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let intervalId;
    
    const fetchSensorData = async () => {
      try {
        const response = await fetch(`/api/sensors/${sensorId}/data`);
        if (!response.ok) throw new Error('Failed to fetch sensor data');
        
        const newData = await response.json();
        setData(prevData => {
          const combined = [...prevData, ...newData];
          // Keep only last 100 data points
          return combined.slice(-100);
        });
        setError(null);
      } catch (err) {
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    };
    
    // Initial fetch
    fetchSensorData();
    
    // Set up interval for regular updates
    intervalId = setInterval(fetchSensorData, interval);
    
    return () => {
      if (intervalId) clearInterval(intervalId);
    };
  }, [sensorId, interval]);
  
  return { data, isLoading, error };
};
```

## 📊 State Management with Redux Toolkit

### Store Configuration:
```javascript
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import { persistStore, persistReducer } from 'redux-persist';
import storage from 'redux-persist/lib/storage';
import millReducer from './millSlice';
import sensorReducer from './sensorSlice';
import authReducer from './authSlice';

const persistConfig = {
  key: 'root',
  storage,
  whitelist: ['auth'] // Only persist auth state
};

const persistedAuthReducer = persistReducer(persistConfig, authReducer);

export const store = configureStore({
  reducer: {
    mill: millReducer,
    sensors: sensorReducer,
    auth: persistedAuthReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE']
      }
    })
});

export const persistor = persistStore(store);
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### Mill Slice with RTK Query:
```javascript
// store/millSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

// API slice for mill data
export const millApi = createApi({
  reducerPath: 'millApi',
  baseQuery: fetchBaseQuery({
    baseUrl: '/api/mills/',
    prepareHeaders: (headers, { getState }) => {
      const token = getState().auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    }
  }),
  tagTypes: ['Mill', 'Sensor'],
  endpoints: (builder) => ({
    getMills: builder.query({
      query: () => '',
      providesTags: ['Mill']
    }),
    getMillById: builder.query({
      query: (id) => `${id}/`,
      providesTags: (result, error, id) => [{ type: 'Mill', id }]
    }),
    updateMillStatus: builder.mutation({
      query: ({ id, status }) => ({
        url: `${id}/status/`,
        method: 'PATCH',
        body: { status }
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Mill', id }]
    })
  })
});

// Mill slice for local state
const millSlice = createSlice({
  name: 'mill',
  initialState: {
    selectedMill: null,
    alerts: [],
    realTimeData: {},
    isConnected: false
  },
  reducers: {
    setSelectedMill: (state, action) => {
      state.selectedMill = action.payload;
    },
    addAlert: (state, action) => {
      state.alerts.unshift(action.payload);
      // Keep only last 50 alerts
      if (state.alerts.length > 50) {
        state.alerts = state.alerts.slice(0, 50);
      }
    },
    updateRealTimeData: (state, action) => {
      const { sensorId, data } = action.payload;
      state.realTimeData[sensorId] = {
        ...state.realTimeData[sensorId],
        ...data,
        lastUpdated: Date.now()
      };
    },
    setConnectionStatus: (state, action) => {
      state.isConnected = action.payload;
    }
  }
});

export const { setSelectedMill, addAlert, updateRealTimeData, setConnectionStatus } = millSlice.actions;
export const { useGetMillsQuery, useGetMillByIdQuery, useUpdateMillStatusMutation } = millApi;
export default millSlice.reducer;
```

## 🎨 Styling and UI Components

### Styled Components with Emotion:
```javascript
// styles/components.js
import styled from '@emotion/styled';
import { Card, Button } from 'antd';

export const DashboardContainer = styled.div`
  padding: 24px;
  background: #f0f2f5;
  min-height: 100vh;
`;

export const SensorCard = styled(Card)`
  .sensor-value {
    display: flex;
    align-items: baseline;
    gap: 8px;
    margin-bottom: 8px;
    
    .value {
      font-size: 2rem;
      font-weight: bold;
      color: ${props => {
        switch (props.status) {
          case 'normal': return '#52c41a';
          case 'warning': return '#faad14';
          case 'critical': return '#ff4d4f';
          default: return '#8c8c8c';
        }
      }};
    }
    
    .unit {
      font-size: 1rem;
      color: #8c8c8c;
    }
  }
  
  .sensor-status {
    font-size: 0.875rem;
    
    .status {
      font-weight: bold;
      
      &.normal { color: #52c41a; }
      &.warning { color: #faad14; }
      &.critical { color: #ff4d4f; }
      &.offline { color: #8c8c8c; }
    }
  }
`;

export const AlertButton = styled(Button)`
  &.critical {
    background: #ff4d4f;
    border-color: #ff4d4f;
    animation: pulse 2s infinite;
    
    &:hover {
      background: #ff7875;
      border-color: #ff7875;
    }
  }
  
  @keyframes pulse {
    0% { box-shadow: 0 0 0 0 rgba(255, 77, 79, 0.7); }
    70% { box-shadow: 0 0 0 10px rgba(255, 77, 79, 0); }
    100% { box-shadow: 0 0 0 0 rgba(255, 77, 79, 0); }
  }
`;
```

## 🧪 Testing Strategy

### Component Testing with React Testing Library:
```javascript
// __tests__/MillDashboard.test.jsx
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Provider } from 'react-redux';
import { configureStore } from '@reduxjs/toolkit';
import MillDashboard from '../components/MillDashboard';
import millReducer from '../store/millSlice';

// Mock WebSocket
const mockWebSocket = {
  send: jest.fn(),
  close: jest.fn(),
  readyState: WebSocket.OPEN
};

global.WebSocket = jest.fn(() => mockWebSocket);

const createTestStore = (initialState = {}) => {
  return configureStore({
    reducer: {
      mill: millReducer
    },
    preloadedState: initialState
  });
};

describe('MillDashboard', () => {
  let store;
  
  beforeEach(() => {
    store = createTestStore({
      mill: {
        sensors: [
          {
            id: 1,
            name: 'Temperature Sensor 1',
            currentValue: 25.5,
            unit: '°C',
            status: 'normal'
          }
        ],
        alerts: [],
        isLoading: false
      }
    });
  });
  
  test('renders sensor data correctly', async () => {
    render(
      <Provider store={store}>
        <MillDashboard />
      </Provider>
    );
    
    expect(screen.getByText('Temperature Sensor 1')).toBeInTheDocument();
    expect(screen.getByText('25.5')).toBeInTheDocument();
    expect(screen.getByText('°C')).toBeInTheDocument();
  });
  
  test('displays critical alerts', async () => {
    const storeWithAlerts = createTestStore({
      mill: {
        sensors: [],
        alerts: [
          {
            id: 1,
            message: 'Temperature too high!',
            severity: 'critical'
          }
        ],
        isLoading: false
      }
    });
    
    render(
      <Provider store={storeWithAlerts}>
        <MillDashboard />
      </Provider>
    );
    
    expect(screen.getByText('Critical Alerts')).toBeInTheDocument();
    expect(screen.getByText('Temperature too high!')).toBeInTheDocument();
  });
  
  test('handles WebSocket connection', async () => {
    render(
      <Provider store={store}>
        <MillDashboard />
      </Provider>
    );
    
    // Simulate WebSocket message
    const messageEvent = new MessageEvent('message', {
      data: JSON.stringify({
        sensorId: 1,
        value: 26.0,
        timestamp: Date.now()
      })
    });
    
    mockWebSocket.onmessage(messageEvent);
    
    await waitFor(() => {
      expect(screen.getByText('Connected')).toBeInTheDocument();
    });
  });
});
```

## 📚 Learning Resources

- [Official React Documentation](https://reactjs.org/docs/)
- [React Patterns](https://reactpatterns.com/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)

## 🚀 Performance Optimization

### Code Splitting and Lazy Loading:
```javascript
// Lazy loading for better performance
import { lazy, Suspense } from 'react';
import { Spin } from 'antd';

const MillDashboard = lazy(() => import('./components/MillDashboard'));
const GamingPlatform = lazy(() => import('./components/GamingPlatform'));
const SensorConfig = lazy(() => import('./components/SensorConfig'));

const App = () => {
  return (
    <Suspense fallback={<Spin size="large" />}>
      <Routes>
        <Route path="/dashboard" element={<MillDashboard />} />
        <Route path="/gaming" element={<GamingPlatform />} />
        <Route path="/config" element={<SensorConfig />} />
      </Routes>
    </Suspense>
  );
};
```

### Memoization and Optimization:
```javascript
// Optimized component with React.memo and useMemo
import React, { memo, useMemo, useCallback } from 'react';

const SensorChart = memo(({ sensorId, data, title }) => {
  const chartData = useMemo(() => {
    return data.map(point => ({
      timestamp: new Date(point.timestamp).toLocaleTimeString(),
      value: parseFloat(point.value.toFixed(2))
    }));
  }, [data]);
  
  const handleDataPointClick = useCallback((dataPoint) => {
    console.log('Data point clicked:', dataPoint);
  }, []);
  
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={chartData} onClick={handleDataPointClick}>
        <XAxis dataKey="timestamp" />
        <YAxis />
        <CartesianGrid strokeDasharray="3 3" />
        <Line 
          type="monotone" 
          dataKey="value" 
          stroke="#1890ff" 
          strokeWidth={2}
          dot={{ r: 4 }}
        />
      </LineChart>
    </ResponsiveContainer>
  );
});

export default SensorChart;
```

---

*"React transforms complex UIs into manageable, reusable components - making frontend development scalable and maintainable."* - Danny Verheyen

[← Back to Main Profile](../README.md)
