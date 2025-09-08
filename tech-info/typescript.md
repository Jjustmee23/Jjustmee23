# 🔷 TypeScript - JavaScript with Type Safety

![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)

## 📋 What is TypeScript?

**TypeScript** is a strongly typed programming language that builds on JavaScript, giving you better tooling at any scale. TypeScript adds static type definitions to JavaScript, enabling you to catch errors early in development and provide a better development experience with features like IntelliSense, refactoring, and navigation.

## 🔧 Key Features

- **Static Type Checking** - Catch errors at compile time
- **Enhanced IDE Support** - Better autocomplete and refactoring
- **Modern JavaScript Features** - Latest ECMAScript features
- **Gradual Adoption** - Can be added incrementally to existing projects
- **Interfaces & Generics** - Advanced type system capabilities
- **Compile-time Optimization** - Better performance through optimization

## 💡 Why I Use TypeScript

### 🏭 Industrial Applications
- **Mill Management Systems** - Type safety for critical industrial data
- **Sensor Data Processing** - Strict typing for IoT sensor readings
- **API Integrations** - Type-safe communication with backend services
- **Real-time Dashboards** - Reliable data flow in monitoring systems

### 🎮 Gaming Platform Development
- **Player Statistics** - Type-safe game data structures
- **Tournament Systems** - Complex tournament bracket logic
- **Real-time Features** - WebSocket message type definitions
- **Community Features** - User interaction type safety

## 🛠️ My TypeScript Setup

### TSConfig Configuration:
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["DOM", "DOM.Iterable", "ES2020"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "baseUrl": "src",
    "paths": {
      "@components/*": ["components/*"],
      "@hooks/*": ["hooks/*"],
      "@utils/*": ["utils/*"],
      "@types/*": ["types/*"],
      "@services/*": ["services/*"]
    },
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  },
  "include": [
    "src/**/*",
    "src/**/*.tsx",
    "src/**/*.ts"
  ],
  "exclude": [
    "node_modules",
    "build",
    "dist"
  ]
}
```

## 🌟 My TypeScript Projects

### 1. 🏭 Mill Management Type Definitions
```typescript
// types/mill.ts
export interface Mill {
  id: string;
  name: string;
  location: {
    latitude: number;
    longitude: number;
    address: string;
  };
  capacity: number;
  status: MillStatus;
  sensors: Sensor[];
  createdAt: Date;
  updatedAt: Date;
}

export enum MillStatus {
  ACTIVE = 'active',
  MAINTENANCE = 'maintenance',
  OFFLINE = 'offline',
  ERROR = 'error'
}

export interface Sensor {
  id: string;
  millId: string;
  name: string;
  type: SensorType;
  deviceEui: string;
  location: string;
  thresholds: SensorThresholds;
  isActive: boolean;
  lastReading?: SensorReading;
  metadata: Record<string, unknown>;
}

export enum SensorType {
  TEMPERATURE = 'temperature',
  HUMIDITY = 'humidity',
  PRESSURE = 'pressure',
  VIBRATION = 'vibration',
  FLOW = 'flow',
  LEVEL = 'level'
}

export interface SensorThresholds {
  critical: {
    min?: number;
    max?: number;
  };
  warning: {
    min?: number;
    max?: number;
  };
}

export interface SensorReading {
  id: string;
  sensorId: string;
  value: number;
  unit: string;
  timestamp: Date;
  quality: ReadingQuality;
  metadata?: Record<string, unknown>;
}

export enum ReadingQuality {
  GOOD = 'good',
  UNCERTAIN = 'uncertain',
  BAD = 'bad'
}

// API Response Types
export interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
  errors?: string[];
}

export interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  hasMore: boolean;
}

// WebSocket Message Types
export interface WebSocketMessage {
  type: MessageType;
  payload: unknown;
  timestamp: Date;
}

export enum MessageType {
  SENSOR_UPDATE = 'sensor_update',
  ALERT = 'alert',
  MILL_STATUS = 'mill_status',
  CONNECTION_STATUS = 'connection_status'
}

export interface SensorUpdateMessage extends WebSocketMessage {
  type: MessageType.SENSOR_UPDATE;
  payload: {
    sensorId: string;
    reading: SensorReading;
  };
}

export interface AlertMessage extends WebSocketMessage {
  type: MessageType.ALERT;
  payload: Alert;
}

export interface Alert {
  id: string;
  millId: string;
  sensorId?: string;
  severity: AlertSeverity;
  title: string;
  message: string;
  acknowledged: boolean;
  createdAt: Date;
  acknowledgedAt?: Date;
  acknowledgedBy?: string;
}

export enum AlertSeverity {
  INFO = 'info',
  WARNING = 'warning',
  CRITICAL = 'critical',
  EMERGENCY = 'emergency'
}
```

### 2. 🎮 Gaming Platform Types
```typescript
// types/gaming.ts
export interface Player {
  id: string;
  username: string;
  email: string;
  avatar?: string;
  level: number;
  experience: number;
  stats: PlayerStats;
  achievements: Achievement[];
  isOnline: boolean;
  lastSeen: Date;
  createdAt: Date;
}

export interface PlayerStats {
  gamesPlayed: number;
  gamesWon: number;
  gamesLost: number;
  totalScore: number;
  averageScore: number;
  winRate: number;
  ranking: number;
  favoriteGame: string;
}

export interface Tournament {
  id: string;
  name: string;
  description: string;
  game: Game;
  format: TournamentFormat;
  status: TournamentStatus;
  maxPlayers: number;
  currentPlayers: number;
  prizePool: number;
  entryFee: number;
  startDate: Date;
  endDate?: Date;
  rules: string;
  brackets: TournamentBracket;
  participants: Player[];
  winner?: Player;
  createdBy: string;
  createdAt: Date;
}

export enum TournamentFormat {
  SINGLE_ELIMINATION = 'single_elimination',
  DOUBLE_ELIMINATION = 'double_elimination',
  ROUND_ROBIN = 'round_robin',
  SWISS = 'swiss'
}

export enum TournamentStatus {
  DRAFT = 'draft',
  OPEN = 'open',
  FULL = 'full',
  IN_PROGRESS = 'in_progress',
  COMPLETED = 'completed',
  CANCELLED = 'cancelled'
}

export interface Game {
  id: string;
  name: string;
  genre: GameGenre;
  platform: GamePlatform[];
  maxPlayers: number;
  averageGameTime: number; // in minutes
  description: string;
  image: string;
  isActive: boolean;
}

export enum GameGenre {
  FPS = 'fps',
  MOBA = 'moba',
  RTS = 'rts',
  FIGHTING = 'fighting',
  RACING = 'racing',
  SPORTS = 'sports'
}

export enum GamePlatform {
  PC = 'pc',
  XBOX = 'xbox',
  PLAYSTATION = 'playstation',
  NINTENDO = 'nintendo',
  MOBILE = 'mobile'
}

export interface Match {
  id: string;
  tournamentId: string;
  round: number;
  matchNumber: number;
  players: Player[];
  winner?: Player;
  score?: MatchScore;
  status: MatchStatus;
  scheduledAt?: Date;
  startedAt?: Date;
  completedAt?: Date;
  streamUrl?: string;
}

export enum MatchStatus {
  SCHEDULED = 'scheduled',
  IN_PROGRESS = 'in_progress',
  COMPLETED = 'completed',
  CANCELLED = 'cancelled',
  NO_SHOW = 'no_show'
}

export interface MatchScore {
  [playerId: string]: number;
}

export interface TournamentBracket {
  rounds: Round[];
  grandFinal?: Match;
}

export interface Round {
  number: number;
  matches: Match[];
  isCompleted: boolean;
}
```

### 3. 🔧 React Component with TypeScript
```typescript
// components/SensorDashboard.tsx
import React, { useState, useEffect, useCallback } from 'react';
import { Mill, Sensor, SensorReading, Alert, WebSocketMessage, MessageType } from '@types/mill';
import { useWebSocket } from '@hooks/useWebSocket';
import { useSensorData } from '@hooks/useSensorData';
import SensorCard from './SensorCard';
import AlertPanel from './AlertPanel';

interface SensorDashboardProps {
  mill: Mill;
  initialSensors: Sensor[];
  onMillStatusChange?: (status: Mill['status']) => void;
}

interface SensorDataMap {
  [sensorId: string]: SensorReading[];
}

const SensorDashboard: React.FC<SensorDashboardProps> = ({
  mill,
  initialSensors,
  onMillStatusChange
}) => {
  const [sensors, setSensors] = useState<Sensor[]>(initialSensors);
  const [alerts, setAlerts] = useState<Alert[]>([]);
  const [sensorData, setSensorData] = useState<SensorDataMap>({});
  
  const { 
    connectionStatus, 
    lastMessage, 
    sendMessage 
  } = useWebSocket<WebSocketMessage>(`ws://localhost:8001/ws/mill/${mill.id}`);
  
  const { 
    data: historicalData, 
    isLoading, 
    error 
  } = useSensorData(mill.id, {
    refreshInterval: 30000,
    onError: (error) => console.error('Sensor data error:', error)
  });

  const handleWebSocketMessage = useCallback((message: WebSocketMessage) => {
    switch (message.type) {
      case MessageType.SENSOR_UPDATE: {
        const { sensorId, reading } = message.payload as {
          sensorId: string;
          reading: SensorReading;
        };
        
        setSensorData(prev => ({
          ...prev,
          [sensorId]: [...(prev[sensorId] || []), reading].slice(-100)
        }));
        
        setSensors(prev => 
          prev.map(sensor => 
            sensor.id === sensorId 
              ? { ...sensor, lastReading: reading }
              : sensor
          )
        );
        break;
      }
      
      case MessageType.ALERT: {
        const alert = message.payload as Alert;
        setAlerts(prev => [alert, ...prev.slice(0, 9)]);
        break;
      }
      
      case MessageType.MILL_STATUS: {
        const status = message.payload as Mill['status'];
        onMillStatusChange?.(status);
        break;
      }
      
      default:
        console.warn('Unknown message type:', message.type);
    }
  }, [onMillStatusChange]);

  useEffect(() => {
    if (lastMessage) {
      handleWebSocketMessage(lastMessage);
    }
  }, [lastMessage, handleWebSocketMessage]);

  const handleSensorConfigUpdate = useCallback(async (
    sensorId: string, 
    config: Partial<Sensor>
  ) => {
    try {
      const response = await fetch(`/api/sensors/${sensorId}`, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(config),
      });

      if (!response.ok) {
        throw new Error('Failed to update sensor configuration');
      }

      const updatedSensor: Sensor = await response.json();
      
      setSensors(prev =>
        prev.map(sensor =>
          sensor.id === sensorId ? updatedSensor : sensor
        )
      );
    } catch (error) {
      console.error('Error updating sensor:', error);
    }
  }, []);

  const handleAlertAcknowledge = useCallback(async (alertId: string) => {
    try {
      await fetch(`/api/alerts/${alertId}/acknowledge`, {
        method: 'POST',
      });

      setAlerts(prev =>
        prev.map(alert =>
          alert.id === alertId
            ? { ...alert, acknowledged: true, acknowledgedAt: new Date() }
            : alert
        )
      );
    } catch (error) {
      console.error('Error acknowledging alert:', error);
    }
  }, []);

  if (isLoading) {
    return <div className="loading-spinner">Loading sensor data...</div>;
  }

  if (error) {
    return <div className="error-message">Error loading sensor data: {error}</div>;
  }

  return (
    <div className="sensor-dashboard">
      <div className="dashboard-header">
        <h1>{mill.name} - Sensor Dashboard</h1>
        <div className="connection-status">
          <span className={`status-indicator ${connectionStatus.toLowerCase()}`}>
            {connectionStatus}
          </span>
        </div>
      </div>

      <AlertPanel 
        alerts={alerts}
        onAcknowledge={handleAlertAcknowledge}
      />

      <div className="sensors-grid">
        {sensors.map(sensor => (
          <SensorCard
            key={sensor.id}
            sensor={sensor}
            data={sensorData[sensor.id] || []}
            onConfigUpdate={(config) => handleSensorConfigUpdate(sensor.id, config)}
          />
        ))}
      </div>

      <div className="dashboard-stats">
        <div className="stat-item">
          <span className="stat-label">Active Sensors:</span>
          <span className="stat-value">
            {sensors.filter(s => s.isActive).length}/{sensors.length}
          </span>
        </div>
        <div className="stat-item">
          <span className="stat-label">Critical Alerts:</span>
          <span className="stat-value critical">
            {alerts.filter(a => a.severity === 'critical' && !a.acknowledged).length}
          </span>
        </div>
        <div className="stat-item">
          <span className="stat-label">Mill Status:</span>
          <span className={`stat-value ${mill.status}`}>
            {mill.status.toUpperCase()}
          </span>
        </div>
      </div>
    </div>
  );
};

export default SensorDashboard;
```

### 4. 🔗 Custom Hooks with TypeScript
```typescript
// hooks/useWebSocket.ts
import { useState, useEffect, useRef, useCallback } from 'react';

interface UseWebSocketOptions {
  reconnectAttempts?: number;
  reconnectInterval?: number;
  onConnect?: () => void;
  onDisconnect?: () => void;
  onError?: (error: Event) => void;
}

interface UseWebSocketReturn<T> {
  connectionStatus: 'Connecting' | 'Connected' | 'Disconnected' | 'Error';
  lastMessage: T | null;
  sendMessage: (message: T | string) => void;
  reconnect: () => void;
}

export const useWebSocket = <T = unknown>(
  url: string,
  options: UseWebSocketOptions = {}
): UseWebSocketReturn<T> => {
  const [connectionStatus, setConnectionStatus] = useState<UseWebSocketReturn<T>['connectionStatus']>('Connecting');
  const [lastMessage, setLastMessage] = useState<T | null>(null);
  
  const socketRef = useRef<WebSocket | null>(null);
  const reconnectTimeoutRef = useRef<NodeJS.Timeout | null>(null);
  const reconnectAttemptsRef = useRef(0);

  const {
    reconnectAttempts = 5,
    reconnectInterval = 3000,
    onConnect,
    onDisconnect,
    onError
  } = options;

  const connect = useCallback(() => {
    try {
      const socket = new WebSocket(url);
      
      socket.onopen = () => {
        setConnectionStatus('Connected');
        reconnectAttemptsRef.current = 0;
        onConnect?.();
      };

      socket.onmessage = (event) => {
        try {
          const message = JSON.parse(event.data) as T;
          setLastMessage(message);
        } catch (error) {
          console.error('Failed to parse WebSocket message:', error);
          setLastMessage(event.data as T);
        }
      };

      socket.onclose = () => {
        setConnectionStatus('Disconnected');
        socketRef.current = null;
        onDisconnect?.();

        // Attempt to reconnect
        if (reconnectAttemptsRef.current < reconnectAttempts) {
          reconnectAttemptsRef.current++;
          reconnectTimeoutRef.current = setTimeout(() => {
            setConnectionStatus('Connecting');
            connect();
          }, reconnectInterval);
        }
      };

      socket.onerror = (error) => {
        setConnectionStatus('Error');
        onError?.(error);
      };

      socketRef.current = socket;
    } catch (error) {
      setConnectionStatus('Error');
      console.error('WebSocket connection error:', error);
    }
  }, [url, reconnectAttempts, reconnectInterval, onConnect, onDisconnect, onError]);

  const sendMessage = useCallback((message: T | string) => {
    if (socketRef.current?.readyState === WebSocket.OPEN) {
      const messageString = typeof message === 'string' 
        ? message 
        : JSON.stringify(message);
      socketRef.current.send(messageString);
    } else {
      console.warn('WebSocket is not connected');
    }
  }, []);

  const reconnect = useCallback(() => {
    if (reconnectTimeoutRef.current) {
      clearTimeout(reconnectTimeoutRef.current);
    }
    
    if (socketRef.current) {
      socketRef.current.close();
    }
    
    reconnectAttemptsRef.current = 0;
    setConnectionStatus('Connecting');
    connect();
  }, [connect]);

  useEffect(() => {
    connect();

    return () => {
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
      }
      if (socketRef.current) {
        socketRef.current.close();
      }
    };
  }, [connect]);

  return {
    connectionStatus,
    lastMessage,
    sendMessage,
    reconnect
  };
};

// hooks/useSensorData.ts
import { useState, useEffect } from 'react';
import { SensorReading } from '@types/mill';

interface UseSensorDataOptions {
  refreshInterval?: number;
  onError?: (error: string) => void;
}

interface UseSensorDataReturn {
  data: SensorReading[];
  isLoading: boolean;
  error: string | null;
  refetch: () => Promise<void>;
}

export const useSensorData = (
  millId: string,
  options: UseSensorDataOptions = {}
): UseSensorDataReturn => {
  const [data, setData] = useState<SensorReading[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const { refreshInterval = 30000, onError } = options;

  const fetchSensorData = async (): Promise<void> => {
    try {
      setError(null);
      const response = await fetch(`/api/mills/${millId}/sensors/data`);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const sensorData: SensorReading[] = await response.json();
      setData(sensorData);
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Unknown error';
      setError(errorMessage);
      onError?.(errorMessage);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    fetchSensorData();

    const interval = setInterval(fetchSensorData, refreshInterval);
    
    return () => clearInterval(interval);
  }, [millId, refreshInterval]);

  return {
    data,
    isLoading,
    error,
    refetch: fetchSensorData
  };
};
```

## 🧪 Testing with TypeScript

### Jest Configuration:
```typescript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  moduleNameMapping: {
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@hooks/(.*)$': '<rootDir>/src/hooks/$1',
    '^@types/(.*)$': '<rootDir>/src/types/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
  ],
  coverageReporters: ['text', 'lcov', 'html'],
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{ts,tsx}',
  ],
};
```

### Type-safe Testing:
```typescript
// __tests__/SensorDashboard.test.tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import '@testing-library/jest-dom';
import { Mill, Sensor, MillStatus, SensorType } from '@types/mill';
import SensorDashboard from '../SensorDashboard';

const mockMill: Mill = {
  id: '1',
  name: 'Test Mill',
  location: {
    latitude: 50.8503,
    longitude: 4.3517,
    address: 'Brussels, Belgium'
  },
  capacity: 1000,
  status: MillStatus.ACTIVE,
  sensors: [],
  createdAt: new Date('2023-01-01'),
  updatedAt: new Date('2023-01-01')
};

const mockSensors: Sensor[] = [
  {
    id: 'sensor-1',
    millId: '1',
    name: 'Temperature Sensor 1',
    type: SensorType.TEMPERATURE,
    deviceEui: 'ABC123',
    location: 'Mill Floor 1',
    thresholds: {
      critical: { max: 50 },
      warning: { max: 40 }
    },
    isActive: true,
    metadata: {}
  }
];

// Mock WebSocket
const mockWebSocket = {
  send: jest.fn(),
  close: jest.fn(),
  readyState: WebSocket.OPEN
};

global.WebSocket = jest.fn(() => mockWebSocket) as any;

describe('SensorDashboard', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('renders mill name correctly', () => {
    render(
      <SensorDashboard 
        mill={mockMill} 
        initialSensors={mockSensors} 
      />
    );

    expect(screen.getByText('Test Mill - Sensor Dashboard')).toBeInTheDocument();
  });

  test('displays sensor information', () => {
    render(
      <SensorDashboard 
        mill={mockMill} 
        initialSensors={mockSensors} 
      />
    );

    expect(screen.getByText('Temperature Sensor 1')).toBeInTheDocument();
    expect(screen.getByText('Mill Floor 1')).toBeInTheDocument();
  });

  test('calls onMillStatusChange when mill status changes', async () => {
    const mockOnStatusChange = jest.fn();
    
    render(
      <SensorDashboard 
        mill={mockMill} 
        initialSensors={mockSensors}
        onMillStatusChange={mockOnStatusChange}
      />
    );

    // Simulate WebSocket message
    const statusMessage = {
      type: 'mill_status',
      payload: MillStatus.MAINTENANCE,
      timestamp: new Date()
    };

    mockWebSocket.onmessage({ data: JSON.stringify(statusMessage) });

    await waitFor(() => {
      expect(mockOnStatusChange).toHaveBeenCalledWith(MillStatus.MAINTENANCE);
    });
  });

  test('handles sensor configuration updates', async () => {
    global.fetch = jest.fn().mockResolvedValue({
      ok: true,
      json: async () => ({
        ...mockSensors[0],
        thresholds: { critical: { max: 60 }, warning: { max: 50 } }
      })
    });

    render(
      <SensorDashboard 
        mill={mockMill} 
        initialSensors={mockSensors} 
      />
    );

    // Simulate sensor config update
    const configButton = screen.getByText('Configure');
    fireEvent.click(configButton);

    await waitFor(() => {
      expect(global.fetch).toHaveBeenCalledWith(
        '/api/sensors/sensor-1',
        expect.objectContaining({
          method: 'PATCH',
          headers: { 'Content-Type': 'application/json' }
        })
      );
    });
  });
});
```

## 📚 Learning Resources

- [Official TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

## 🎯 Best Practices I Follow

1. **Strict Type Checking** - Enable all strict compiler options
2. **Interface over Type** - Use interfaces for object shapes
3. **Generic Constraints** - Proper generic type constraints
4. **Utility Types** - Leverage built-in utility types
5. **Type Guards** - Runtime type checking with type predicates
6. **Incremental Adoption** - Gradually migrate JavaScript to TypeScript

---

*"TypeScript brings the confidence of static typing to JavaScript, making large-scale applications more maintainable and reliable."* - Danny Verheyen

[← Back to Main Profile](../README.md)
