# 🐍 Python - Versatile Programming Language

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)

## 📋 What is Python?

**Python** is a high-level, interpreted programming language with dynamic semantics. Its high-level built-in data structures, combined with dynamic typing and dynamic binding, make it very attractive for Rapid Application Development, as well as for use as a scripting or glue language to connect existing components.

## 🔧 Key Features

- **Simple Syntax** - Easy to read and write
- **Interpreted Language** - No compilation step needed
- **Dynamic Typing** - Flexible variable types
- **Extensive Libraries** - Rich ecosystem of packages
- **Cross-Platform** - Runs on all major operating systems
- **Object-Oriented** - Supports OOP paradigms

## 💡 Why I Use Python

### 🏭 Industrial IoT Applications
- **Sensor Data Processing** - Real-time data analysis and filtering
- **MQTT Communication** - IoT device communication protocols
- **Mill Automation Scripts** - Automated control systems
- **Data Analytics** - Processing large datasets from industrial sensors

### 🤖 Automation & Scripting
- **System Administration** - Server management and monitoring
- **Data Migration** - Database migration and synchronization
- **Report Generation** - Automated reporting systems
- **DevOps Automation** - Infrastructure management scripts

## 🌟 My Python Projects

### 1. 🏭 Mill Sensor Data Collector
```python
#!/usr/bin/env python3
"""
Mill Sensor Data Collector
Collects data from various sensors and publishes to MQTT broker
"""

import asyncio
import json
import logging
from datetime import datetime, timezone
from typing import Dict, List, Optional, Any
from dataclasses import dataclass, asdict
import paho.mqtt.client as mqtt
import psycopg2
from psycopg2.extras import RealDictCursor
import serial
import struct

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

@dataclass
class SensorReading:
    sensor_id: str
    value: float
    unit: str
    timestamp: datetime
    quality: str = 'good'
    metadata: Optional[Dict[str, Any]] = None

    def to_dict(self) -> Dict[str, Any]:
        data = asdict(self)
        data['timestamp'] = self.timestamp.isoformat()
        return data

@dataclass
class SensorConfig:
    id: str
    name: str
    sensor_type: str
    device_path: str
    baud_rate: int = 9600
    sampling_interval: int = 60  # seconds
    thresholds: Optional[Dict[str, float]] = None
    calibration: Optional[Dict[str, float]] = None

class SensorDataCollector:
    def __init__(self, config_file: str):
        self.config = self._load_config(config_file)
        self.mqtt_client = None
        self.db_connection = None
        self.sensors: Dict[str, SensorConfig] = {}
        self.serial_connections: Dict[str, serial.Serial] = {}
        
    def _load_config(self, config_file: str) -> Dict[str, Any]:
        """Load configuration from JSON file"""
        with open(config_file, 'r') as f:
            return json.load(f)
    
    async def initialize(self):
        """Initialize all connections and sensors"""
        try:
            # Initialize MQTT client
            await self._init_mqtt()
            
            # Initialize database connection
            await self._init_database()
            
            # Initialize sensors
            await self._init_sensors()
            
            logger.info("Sensor data collector initialized successfully")
            
        except Exception as e:
            logger.error(f"Failed to initialize: {e}")
            raise

    async def _init_mqtt(self):
        """Initialize MQTT client"""
        self.mqtt_client = mqtt.Client(client_id=self.config['mqtt']['client_id'])
        
        # Set up callbacks
        self.mqtt_client.on_connect = self._on_mqtt_connect
        self.mqtt_client.on_disconnect = self._on_mqtt_disconnect
        self.mqtt_client.on_publish = self._on_mqtt_publish
        
        # Connect to broker
        mqtt_config = self.config['mqtt']
        await asyncio.get_event_loop().run_in_executor(
            None,
            self.mqtt_client.connect,
            mqtt_config['host'],
            mqtt_config['port'],
            mqtt_config['keepalive']
        )
        
        self.mqtt_client.loop_start()
        
    def _on_mqtt_connect(self, client, userdata, flags, rc):
        if rc == 0:
            logger.info("Connected to MQTT broker")
        else:
            logger.error(f"Failed to connect to MQTT broker: {rc}")
    
    def _on_mqtt_disconnect(self, client, userdata, rc):
        logger.warning("Disconnected from MQTT broker")
    
    def _on_mqtt_publish(self, client, userdata, mid):
        logger.debug(f"Message {mid} published to MQTT")

    async def _init_database(self):
        """Initialize PostgreSQL connection"""
        db_config = self.config['database']
        self.db_connection = psycopg2.connect(
            host=db_config['host'],
            port=db_config['port'],
            database=db_config['name'],
            user=db_config['user'],
            password=db_config['password'],
            cursor_factory=RealDictCursor
        )
        logger.info("Connected to PostgreSQL database")

    async def _init_sensors(self):
        """Initialize sensor connections"""
        for sensor_config in self.config['sensors']:
            sensor = SensorConfig(**sensor_config)
            self.sensors[sensor.id] = sensor
            
            # Initialize serial connection for sensor
            try:
                serial_conn = serial.Serial(
                    sensor.device_path,
                    sensor.baud_rate,
                    timeout=1
                )
                self.serial_connections[sensor.id] = serial_conn
                logger.info(f"Initialized sensor: {sensor.name}")
                
            except Exception as e:
                logger.error(f"Failed to initialize sensor {sensor.name}: {e}")

    async def read_sensor_data(self, sensor_id: str) -> Optional[SensorReading]:
        """Read data from a specific sensor"""
        if sensor_id not in self.sensors:
            logger.error(f"Unknown sensor: {sensor_id}")
            return None
            
        sensor = self.sensors[sensor_id]
        serial_conn = self.serial_connections.get(sensor_id)
        
        if not serial_conn or not serial_conn.is_open:
            logger.error(f"No connection to sensor: {sensor.name}")
            return None
            
        try:
            # Read raw data from sensor
            raw_data = serial_conn.readline().decode('utf-8').strip()
            
            if not raw_data:
                return None
                
            # Parse sensor data based on type
            value = self._parse_sensor_data(sensor, raw_data)
            
            # Apply calibration if configured
            if sensor.calibration:
                value = self._apply_calibration(value, sensor.calibration)
            
            # Create reading object
            reading = SensorReading(
                sensor_id=sensor_id,
                value=value,
                unit=self._get_sensor_unit(sensor.sensor_type),
                timestamp=datetime.now(timezone.utc),
                quality=self._assess_data_quality(sensor, value),
                metadata={
                    'sensor_name': sensor.name,
                    'sensor_type': sensor.sensor_type,
                    'raw_data': raw_data
                }
            )
            
            return reading
            
        except Exception as e:
            logger.error(f"Error reading from sensor {sensor.name}: {e}")
            return None

    def _parse_sensor_data(self, sensor: SensorConfig, raw_data: str) -> float:
        """Parse raw sensor data based on sensor type"""
        if sensor.sensor_type == 'temperature_dht22':
            # DHT22 format: "TEMP:25.6,HUM:60.2"
            parts = raw_data.split(',')
            temp_part = next(p for p in parts if p.startswith('TEMP:'))
            return float(temp_part.split(':')[1])
            
        elif sensor.sensor_type == 'pressure_bmp280':
            # BMP280 format: "PRESS:1013.25"
            return float(raw_data.split(':')[1])
            
        elif sensor.sensor_type == 'level_ultrasonic':
            # Ultrasonic format: "DIST:150.5"
            distance = float(raw_data.split(':')[1])
            # Convert distance to level percentage
            max_distance = sensor.metadata.get('max_distance', 200)
            level = ((max_distance - distance) / max_distance) * 100
            return max(0, min(100, level))
            
        else:
            # Generic numeric format
            return float(raw_data)

    def _apply_calibration(self, value: float, calibration: Dict[str, float]) -> float:
        """Apply calibration offset and multiplier"""
        offset = calibration.get('offset', 0)
        multiplier = calibration.get('multiplier', 1)
        return (value + offset) * multiplier

    def _get_sensor_unit(self, sensor_type: str) -> str:
        """Get unit for sensor type"""
        units = {
            'temperature_dht22': '°C',
            'humidity_dht22': '%',
            'pressure_bmp280': 'hPa',
            'level_ultrasonic': '%',
            'vibration': 'Hz',
            'flow': 'L/min'
        }
        return units.get(sensor_type, '')

    def _assess_data_quality(self, sensor: SensorConfig, value: float) -> str:
        """Assess quality of sensor reading"""
        if sensor.thresholds:
            critical = sensor.thresholds.get('critical', {})
            if 'min' in critical and value < critical['min']:
                return 'bad'
            if 'max' in critical and value > critical['max']:
                return 'bad'
                
        # Add more quality checks here
        return 'good'

    async def store_reading(self, reading: SensorReading):
        """Store sensor reading in database"""
        try:
            with self.db_connection.cursor() as cursor:
                cursor.execute("""
                    INSERT INTO sensor_readings 
                    (sensor_id, value, unit, timestamp, quality, metadata)
                    VALUES (%s, %s, %s, %s, %s, %s)
                """, (
                    reading.sensor_id,
                    reading.value,
                    reading.unit,
                    reading.timestamp,
                    reading.quality,
                    json.dumps(reading.metadata)
                ))
                self.db_connection.commit()
                
        except Exception as e:
            logger.error(f"Error storing reading: {e}")
            self.db_connection.rollback()

    async def publish_reading(self, reading: SensorReading):
        """Publish sensor reading to MQTT"""
        try:
            topic = f"mill/{reading.metadata['mill_id']}/sensors/{reading.sensor_id}"
            payload = json.dumps(reading.to_dict())
            
            result = self.mqtt_client.publish(topic, payload, qos=1)
            
            if result.rc != mqtt.MQTT_ERR_SUCCESS:
                logger.error(f"Failed to publish to MQTT: {result.rc}")
            else:
                logger.debug(f"Published reading for sensor {reading.sensor_id}")
                
        except Exception as e:
            logger.error(f"Error publishing to MQTT: {e}")

    async def run_collection_loop(self):
        """Main data collection loop"""
        logger.info("Starting sensor data collection loop")
        
        while True:
            try:
                tasks = []
                
                for sensor_id in self.sensors:
                    task = asyncio.create_task(
                        self._collect_sensor_data(sensor_id)
                    )
                    tasks.append(task)
                
                # Wait for all sensors to be read
                await asyncio.gather(*tasks)
                
                # Wait for next collection interval
                min_interval = min(s.sampling_interval for s in self.sensors.values())
                await asyncio.sleep(min_interval)
                
            except Exception as e:
                logger.error(f"Error in collection loop: {e}")
                await asyncio.sleep(10)  # Wait before retrying

    async def _collect_sensor_data(self, sensor_id: str):
        """Collect data from a single sensor"""
        try:
            reading = await self.read_sensor_data(sensor_id)
            
            if reading:
                # Store in database
                await self.store_reading(reading)
                
                # Publish to MQTT
                await self.publish_reading(reading)
                
                # Check for alerts
                await self._check_alerts(reading)
                
        except Exception as e:
            logger.error(f"Error collecting data from sensor {sensor_id}: {e}")

    async def _check_alerts(self, reading: SensorReading):
        """Check if reading triggers any alerts"""
        sensor = self.sensors[reading.sensor_id]
        
        if sensor.thresholds:
            critical = sensor.thresholds.get('critical', {})
            warning = sensor.thresholds.get('warning', {})
            
            alert_type = None
            
            if 'max' in critical and reading.value > critical['max']:
                alert_type = 'critical'
            elif 'min' in critical and reading.value < critical['min']:
                alert_type = 'critical'
            elif 'max' in warning and reading.value > warning['max']:
                alert_type = 'warning'
            elif 'min' in warning and reading.value < warning['min']:
                alert_type = 'warning'
            
            if alert_type:
                alert = {
                    'sensor_id': reading.sensor_id,
                    'sensor_name': sensor.name,
                    'alert_type': alert_type,
                    'value': reading.value,
                    'unit': reading.unit,
                    'timestamp': reading.timestamp.isoformat(),
                    'message': f"{sensor.name} reading {reading.value}{reading.unit} exceeds {alert_type} threshold"
                }
                
                # Publish alert
                self.mqtt_client.publish(
                    f"mill/alerts/{alert_type}",
                    json.dumps(alert),
                    qos=2
                )
                
                logger.warning(f"Alert triggered: {alert['message']}")

async def main():
    """Main application entry point"""
    collector = SensorDataCollector('config/sensors.json')
    
    try:
        await collector.initialize()
        await collector.run_collection_loop()
        
    except KeyboardInterrupt:
        logger.info("Shutting down sensor data collector")
        
    except Exception as e:
        logger.error(f"Unexpected error: {e}")
        
    finally:
        # Cleanup
        if collector.mqtt_client:
            collector.mqtt_client.loop_stop()
            collector.mqtt_client.disconnect()
            
        if collector.db_connection:
            collector.db_connection.close()
            
        for conn in collector.serial_connections.values():
            if conn.is_open:
                conn.close()

if __name__ == "__main__":
    asyncio.run(main())
```

### 2. 🤖 MQTT Message Processor
```python
# mqtt_processor.py
"""
MQTT Message Processor for Mill Management System
Processes incoming MQTT messages and updates database
"""

import json
import logging
from typing import Dict, Any, Callable
from datetime import datetime
import paho.mqtt.client as mqtt
import redis
from sqlalchemy import create_engine, Column, Integer, String, Float, DateTime, JSON
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session

logger = logging.getLogger(__name__)

Base = declarative_base()

class SensorReading(Base):
    __tablename__ = 'sensor_readings'
    
    id = Column(Integer, primary_key=True)
    sensor_id = Column(String(50), nullable=False)
    mill_id = Column(String(50), nullable=False)
    value = Column(Float, nullable=False)
    unit = Column(String(10), nullable=False)
    timestamp = Column(DateTime, nullable=False)
    quality = Column(String(20), default='good')
    metadata = Column(JSON)

class MQTTProcessor:
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.mqtt_client = mqtt.Client()
        self.redis_client = redis.Redis(
            host=config['redis']['host'],
            port=config['redis']['port'],
            db=config['redis']['db']
        )
        
        # Database setup
        self.engine = create_engine(config['database']['url'])
        self.SessionLocal = sessionmaker(bind=self.engine)
        
        # Message handlers
        self.handlers: Dict[str, Callable] = {
            'mill/+/sensors/+': self._handle_sensor_data,
            'mill/+/status': self._handle_mill_status,
            'mill/+/commands/+': self._handle_commands,
            'mill/alerts/+': self._handle_alerts
        }
        
        self._setup_mqtt_callbacks()

    def _setup_mqtt_callbacks(self):
        """Setup MQTT client callbacks"""
        self.mqtt_client.on_connect = self._on_connect
        self.mqtt_client.on_message = self._on_message
        self.mqtt_client.on_disconnect = self._on_disconnect

    def _on_connect(self, client, userdata, flags, rc):
        """Callback for MQTT connection"""
        if rc == 0:
            logger.info("Connected to MQTT broker")
            
            # Subscribe to all topics
            for topic in self.handlers.keys():
                client.subscribe(topic)
                logger.info(f"Subscribed to topic: {topic}")
        else:
            logger.error(f"Failed to connect to MQTT broker: {rc}")

    def _on_message(self, client, userdata, msg):
        """Process incoming MQTT messages"""
        try:
            topic = msg.topic
            payload = json.loads(msg.payload.decode('utf-8'))
            
            logger.debug(f"Received message on topic {topic}: {payload}")
            
            # Find matching handler
            handler = self._find_handler(topic)
            if handler:
                handler(topic, payload)
            else:
                logger.warning(f"No handler found for topic: {topic}")
                
        except Exception as e:
            logger.error(f"Error processing message: {e}")

    def _find_handler(self, topic: str) -> Callable:
        """Find appropriate handler for topic"""
        for pattern, handler in self.handlers.items():
            if self._topic_matches(topic, pattern):
                return handler
        return None

    def _topic_matches(self, topic: str, pattern: str) -> bool:
        """Check if topic matches pattern with wildcards"""
        topic_parts = topic.split('/')
        pattern_parts = pattern.split('/')
        
        if len(topic_parts) != len(pattern_parts):
            return False
            
        for topic_part, pattern_part in zip(topic_parts, pattern_parts):
            if pattern_part != '+' and pattern_part != topic_part:
                return False
                
        return True

    def _handle_sensor_data(self, topic: str, payload: Dict[str, Any]):
        """Handle sensor data messages"""
        try:
            # Extract mill_id and sensor_id from topic
            parts = topic.split('/')
            mill_id = parts[1]
            sensor_id = parts[3]
            
            # Create sensor reading
            reading = SensorReading(
                sensor_id=sensor_id,
                mill_id=mill_id,
                value=payload['value'],
                unit=payload['unit'],
                timestamp=datetime.fromisoformat(payload['timestamp']),
                quality=payload.get('quality', 'good'),
                metadata=payload.get('metadata')
            )
            
            # Store in database
            with self.SessionLocal() as session:
                session.add(reading)
                session.commit()
                
            # Cache latest reading in Redis
            cache_key = f"sensor:{sensor_id}:latest"
            self.redis_client.setex(
                cache_key,
                300,  # 5 minutes TTL
                json.dumps(payload)
            )
            
            # Publish to real-time topic for dashboards
            realtime_topic = f"realtime/mill/{mill_id}/sensor/{sensor_id}"
            self.mqtt_client.publish(realtime_topic, json.dumps(payload))
            
            logger.debug(f"Processed sensor data for {sensor_id}")
            
        except Exception as e:
            logger.error(f"Error handling sensor data: {e}")

    def _handle_mill_status(self, topic: str, payload: Dict[str, Any]):
        """Handle mill status updates"""
        try:
            mill_id = topic.split('/')[1]
            status = payload['status']
            
            # Update mill status in cache
            cache_key = f"mill:{mill_id}:status"
            self.redis_client.setex(cache_key, 3600, status)  # 1 hour TTL
            
            # Publish to real-time topic
            realtime_topic = f"realtime/mill/{mill_id}/status"
            self.mqtt_client.publish(realtime_topic, json.dumps(payload))
            
            logger.info(f"Mill {mill_id} status updated to: {status}")
            
        except Exception as e:
            logger.error(f"Error handling mill status: {e}")

    def _handle_commands(self, topic: str, payload: Dict[str, Any]):
        """Handle command messages"""
        try:
            parts = topic.split('/')
            mill_id = parts[1]
            command = parts[3]
            
            logger.info(f"Received command {command} for mill {mill_id}")
            
            # Process command based on type
            if command == 'start_production':
                self._start_mill_production(mill_id, payload)
            elif command == 'stop_production':
                self._stop_mill_production(mill_id, payload)
            elif command == 'update_config':
                self._update_mill_config(mill_id, payload)
            else:
                logger.warning(f"Unknown command: {command}")
                
        except Exception as e:
            logger.error(f"Error handling command: {e}")

    def _handle_alerts(self, topic: str, payload: Dict[str, Any]):
        """Handle alert messages"""
        try:
            alert_type = topic.split('/')[-1]
            
            # Store alert in database
            # Send notifications
            # Update dashboard
            
            logger.warning(f"Alert received: {alert_type} - {payload['message']}")
            
        except Exception as e:
            logger.error(f"Error handling alert: {e}")

    def start(self):
        """Start the MQTT processor"""
        try:
            mqtt_config = self.config['mqtt']
            self.mqtt_client.connect(
                mqtt_config['host'],
                mqtt_config['port'],
                mqtt_config['keepalive']
            )
            
            logger.info("Starting MQTT message processor")
            self.mqtt_client.loop_forever()
            
        except Exception as e:
            logger.error(f"Error starting MQTT processor: {e}")
            raise

    def stop(self):
        """Stop the MQTT processor"""
        logger.info("Stopping MQTT message processor")
        self.mqtt_client.loop_stop()
        self.mqtt_client.disconnect()

# Configuration example
if __name__ == "__main__":
    config = {
        'mqtt': {
            'host': 'localhost',
            'port': 1883,
            'keepalive': 60
        },
        'database': {
            'url': 'postgresql://user:password@localhost:5432/mill_db'
        },
        'redis': {
            'host': 'localhost',
            'port': 6379,
            'db': 0
        }
    }
    
    processor = MQTTProcessor(config)
    
    try:
        processor.start()
    except KeyboardInterrupt:
        processor.stop()
```

## 📚 Learning Resources

- [Official Python Documentation](https://docs.python.org/3/)
- [Python Package Index (PyPI)](https://pypi.org/)
- [Real Python Tutorials](https://realpython.com/)
- [Python.org Tutorial](https://docs.python.org/3/tutorial/)

## 🎯 Best Practices I Follow

1. **PEP 8** - Follow Python style guide
2. **Type Hints** - Use type annotations for better code clarity
3. **Virtual Environments** - Isolate project dependencies
4. **Error Handling** - Proper exception handling and logging
5. **Testing** - Comprehensive unit and integration tests
6. **Documentation** - Clear docstrings and comments

---

*"Python's simplicity and power make it perfect for everything from IoT data processing to web development."* - Danny Verheyen

[← Back to Main Profile](../README.md)
