# 🎸 Django - The Web Framework for Perfectionists

![Django](https://img.shields.io/badge/Django-092E20?style=for-the-badge&logo=django&logoColor=white)

## 📋 What is Django?

**Django** is a high-level Python web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of web development, so you can focus on writing your app without needing to reinvent the wheel.

## 🔧 Key Features

- **"Batteries Included"** - Everything you need is included
- **ORM (Object-Relational Mapping)** - Database abstraction layer
- **Admin Interface** - Automatic admin interface generation
- **Security** - Built-in protection against common security threats
- **Scalability** - Handles high-traffic applications
- **MVT Architecture** - Model-View-Template pattern

## 💡 Why I Use Django

### 🏭 Industrial Web Applications
- **Mill Management Systems** - Complete web-based control systems
- **Sensor Data APIs** - RESTful APIs for IoT device communication
- **Real-time Dashboards** - WebSocket-enabled monitoring interfaces
- **User Management** - Role-based access control for industrial users

### 🎮 Gaming Platform Backend
- **Tournament Management** - Complex tournament bracket systems
- **Player Statistics** - Advanced gaming analytics and leaderboards
- **Community Features** - User profiles, messaging, and social features
- **Real-time Gaming** - WebSocket integration for live gaming features

## 🛠️ My Django Architecture

### Project Structure:
```
mill_management/
├── mill_management/
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── testing.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── apps/
│   ├── mills/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   └── admin.py
│   ├── sensors/
│   ├── alerts/
│   ├── users/
│   └── api/
├── static/
├── media/
├── templates/
├── requirements/
│   ├── base.txt
│   ├── development.txt
│   └── production.txt
├── config/
├── scripts/
└── manage.py
```

## 🌟 My Django Projects

### 1. 🏭 Mill Management Models
```python
# apps/mills/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.core.validators import MinValueValidator, MaxValueValidator
from django.utils.translation import gettext_lazy as _
import uuid

class Mill(models.Model):
    class StatusChoices(models.TextChoices):
        ACTIVE = 'active', _('Active')
        MAINTENANCE = 'maintenance', _('Maintenance')
        OFFLINE = 'offline', _('Offline')
        ERROR = 'error', _('Error')

    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(max_length=100, unique=True)
    location = models.CharField(max_length=200)
    latitude = models.DecimalField(max_digits=9, decimal_places=6, null=True, blank=True)
    longitude = models.DecimalField(max_digits=9, decimal_places=6, null=True, blank=True)
    capacity = models.PositiveIntegerField(help_text="Daily capacity in tons")
    status = models.CharField(max_length=20, choices=StatusChoices.choices, default=StatusChoices.ACTIVE)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['name']
        verbose_name = _('Mill')
        verbose_name_plural = _('Mills')

    def __str__(self):
        return self.name

    @property
    def active_sensors_count(self):
        return self.sensors.filter(is_active=True).count()

    @property
    def latest_production_data(self):
        return self.production_records.filter(
            date=models.functions.Now().date()
        ).first()

class Sensor(models.Model):
    class TypeChoices(models.TextChoices):
        TEMPERATURE = 'temperature', _('Temperature')
        HUMIDITY = 'humidity', _('Humidity')
        PRESSURE = 'pressure', _('Pressure')
        VIBRATION = 'vibration', _('Vibration')
        FLOW = 'flow', _('Flow')
        LEVEL = 'level', _('Level')

    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    mill = models.ForeignKey(Mill, on_delete=models.CASCADE, related_name='sensors')
    name = models.CharField(max_length=100)
    sensor_type = models.CharField(max_length=20, choices=TypeChoices.choices)
    device_eui = models.CharField(max_length=16, unique=True)
    location = models.CharField(max_length=100)
    is_active = models.BooleanField(default=True)
    sampling_interval = models.PositiveIntegerField(default=60, help_text="Seconds between readings")
    
    # Threshold settings
    critical_min = models.FloatField(null=True, blank=True)
    critical_max = models.FloatField(null=True, blank=True)
    warning_min = models.FloatField(null=True, blank=True)
    warning_max = models.FloatField(null=True, blank=True)
    
    # Calibration settings
    calibration_offset = models.FloatField(default=0.0)
    calibration_multiplier = models.FloatField(default=1.0)
    
    metadata = models.JSONField(default=dict, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['mill', 'name']
        unique_together = ['mill', 'name']

    def __str__(self):
        return f"{self.mill.name} - {self.name}"

    @property
    def latest_reading(self):
        return self.readings.order_by('-timestamp').first()

    def get_unit(self):
        units = {
            'temperature': '°C',
            'humidity': '%',
            'pressure': 'hPa',
            'vibration': 'Hz',
            'flow': 'L/min',
            'level': '%'
        }
        return units.get(self.sensor_type, '')

class SensorReading(models.Model):
    class QualityChoices(models.TextChoices):
        GOOD = 'good', _('Good')
        UNCERTAIN = 'uncertain', _('Uncertain')
        BAD = 'bad', _('Bad')

    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    sensor = models.ForeignKey(Sensor, on_delete=models.CASCADE, related_name='readings')
    value = models.DecimalField(max_digits=10, decimal_places=3)
    timestamp = models.DateTimeField(db_index=True)
    quality = models.CharField(max_length=10, choices=QualityChoices.choices, default=QualityChoices.GOOD)
    metadata = models.JSONField(default=dict, blank=True)

    class Meta:
        ordering = ['-timestamp']
        indexes = [
            models.Index(fields=['sensor', '-timestamp']),
            models.Index(fields=['timestamp']),
        ]

    def __str__(self):
        return f"{self.sensor.name}: {self.value} at {self.timestamp}"

class Alert(models.Model):
    class SeverityChoices(models.TextChoices):
        INFO = 'info', _('Info')
        WARNING = 'warning', _('Warning')
        CRITICAL = 'critical', _('Critical')
        EMERGENCY = 'emergency', _('Emergency')

    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    mill = models.ForeignKey(Mill, on_delete=models.CASCADE, related_name='alerts')
    sensor = models.ForeignKey(Sensor, on_delete=models.CASCADE, null=True, blank=True)
    severity = models.CharField(max_length=10, choices=SeverityChoices.choices)
    title = models.CharField(max_length=200)
    message = models.TextField()
    is_acknowledged = models.BooleanField(default=False)
    acknowledged_by = models.ForeignKey('users.User', null=True, blank=True, on_delete=models.SET_NULL)
    acknowledged_at = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['mill', '-created_at']),
            models.Index(fields=['severity', '-created_at']),
        ]

    def acknowledge(self, user):
        self.is_acknowledged = True
        self.acknowledged_by = user
        self.acknowledged_at = models.functions.Now()
        self.save(update_fields=['is_acknowledged', 'acknowledged_by', 'acknowledged_at'])
```

### 2. 🔧 Django REST Framework API
```python
# apps/api/views.py
from rest_framework import viewsets, status, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from django_filters.rest_framework import DjangoFilterBackend
from django.utils import timezone
from django.db.models import Avg, Count, Max, Q
from datetime import timedelta

from apps.mills.models import Mill, Sensor, SensorReading, Alert
from apps.api.serializers import (
    MillSerializer, SensorSerializer, 
    SensorReadingSerializer, AlertSerializer
)
from apps.api.permissions import MillOperatorPermission

class MillViewSet(viewsets.ModelViewSet):
    queryset = Mill.objects.all()
    serializer_class = MillSerializer
    permission_classes = [IsAuthenticated, MillOperatorPermission]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['status', 'capacity']
    search_fields = ['name', 'location']
    ordering_fields = ['name', 'created_at', 'capacity']

    @action(detail=True, methods=['get'])
    def sensors(self, request, pk=None):
        """Get all sensors for a specific mill"""
        mill = self.get_object()
        sensors = mill.sensors.filter(is_active=True)
        serializer = SensorSerializer(sensors, many=True)
        return Response(serializer.data)

    @action(detail=True, methods=['get'])
    def status(self, request, pk=None):
        """Get current mill status with real-time data"""
        mill = self.get_object()
        
        # Get latest sensor readings
        latest_readings = {}
        for sensor in mill.sensors.filter(is_active=True):
            latest = sensor.readings.order_by('-timestamp').first()
            if latest:
                latest_readings[sensor.id] = {
                    'value': float(latest.value),
                    'unit': sensor.get_unit(),
                    'timestamp': latest.timestamp,
                    'quality': latest.quality
                }

        # Get recent alerts
        recent_alerts = mill.alerts.filter(
            created_at__gte=timezone.now() - timedelta(hours=24),
            is_acknowledged=False
        ).count()

        # Calculate production efficiency
        today_production = mill.production_records.filter(
            date=timezone.now().date()
        ).first()

        efficiency = 0
        if today_production and mill.capacity:
            efficiency = (today_production.actual_production / mill.capacity) * 100

        return Response({
            'mill': MillSerializer(mill).data,
            'latest_readings': latest_readings,
            'recent_alerts': recent_alerts,
            'efficiency': round(efficiency, 2),
            'last_updated': timezone.now()
        })

    @action(detail=True, methods=['post'])
    def start_production(self, request, pk=None):
        """Start mill production"""
        mill = self.get_object()
        
        if mill.status != Mill.StatusChoices.ACTIVE:
            return Response(
                {'error': 'Mill must be in active status to start production'},
                status=status.HTTP_400_BAD_REQUEST
            )

        # Logic to start production
        # Send MQTT command to mill systems
        
        return Response({'message': 'Production started successfully'})

    @action(detail=True, methods=['post'])
    def emergency_stop(self, request, pk=None):
        """Emergency stop for mill operations"""
        mill = self.get_object()
        
        # Immediate stop logic
        mill.status = Mill.StatusChoices.OFFLINE
        mill.save()
        
        # Create emergency alert
        Alert.objects.create(
            mill=mill,
            severity=Alert.SeverityChoices.EMERGENCY,
            title="Emergency Stop Activated",
            message=f"Emergency stop activated for {mill.name} by {request.user.username}"
        )
        
        # Send MQTT emergency stop command
        
        return Response({'message': 'Emergency stop activated'})

class SensorViewSet(viewsets.ModelViewSet):
    queryset = Sensor.objects.all()
    serializer_class = SensorSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter]
    filterset_fields = ['mill', 'sensor_type', 'is_active']
    search_fields = ['name', 'location']

    @action(detail=True, methods=['get'])
    def readings(self, request, pk=None):
        """Get sensor readings with time range filtering"""
        sensor = self.get_object()
        
        # Get query parameters
        hours = int(request.query_params.get('hours', 24))
        limit = int(request.query_params.get('limit', 1000))
        
        start_time = timezone.now() - timedelta(hours=hours)
        
        readings = sensor.readings.filter(
            timestamp__gte=start_time
        ).order_by('-timestamp')[:limit]
        
        serializer = SensorReadingSerializer(readings, many=True)
        
        # Calculate statistics
        values = [float(r.value) for r in readings]
        stats = {}
        if values:
            stats = {
                'count': len(values),
                'average': sum(values) / len(values),
                'minimum': min(values),
                'maximum': max(values),
                'latest': values[0] if values else None
            }
        
        return Response({
            'readings': serializer.data,
            'statistics': stats,
            'time_range': {
                'start': start_time,
                'end': timezone.now(),
                'hours': hours
            }
        })

    @action(detail=True, methods=['post'])
    def calibrate(self, request, pk=None):
        """Calibrate sensor with reference values"""
        sensor = self.get_object()
        
        reference_value = request.data.get('reference_value')
        current_reading = sensor.latest_reading
        
        if not current_reading or not reference_value:
            return Response(
                {'error': 'Reference value and current reading required'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Calculate calibration offset
        offset = float(reference_value) - float(current_reading.value)
        sensor.calibration_offset = offset
        sensor.save()
        
        return Response({
            'message': 'Sensor calibrated successfully',
            'calibration_offset': offset,
            'reference_value': reference_value,
            'previous_reading': float(current_reading.value)
        })
```

### 3. 🔒 Custom Authentication & Permissions
```python
# apps/users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    class RoleChoices(models.TextChoices):
        ADMIN = 'admin', 'Administrator'
        OPERATOR = 'operator', 'Mill Operator'
        VIEWER = 'viewer', 'Read Only'
        TECHNICIAN = 'technician', 'Maintenance Technician'

    role = models.CharField(max_length=20, choices=RoleChoices.choices, default=RoleChoices.VIEWER)
    mills = models.ManyToManyField('mills.Mill', blank=True, related_name='operators')
    phone = models.CharField(max_length=20, blank=True)
    last_login_ip = models.GenericIPAddressField(null=True, blank=True)
    
    def has_mill_access(self, mill):
        if self.role == self.RoleChoices.ADMIN:
            return True
        return self.mills.filter(id=mill.id).exists()

# apps/api/permissions.py
from rest_framework import permissions

class MillOperatorPermission(permissions.BasePermission):
    """
    Custom permission to only allow operators to access their assigned mills.
    """
    
    def has_permission(self, request, view):
        return request.user and request.user.is_authenticated
    
    def has_object_permission(self, request, view, obj):
        # Admin users have access to everything
        if request.user.role == 'admin':
            return True
            
        # Check if user has access to this mill
        if hasattr(obj, 'mill'):
            return request.user.has_mill_access(obj.mill)
        elif hasattr(obj, 'mills'):
            return request.user.mills.filter(id__in=obj.mills.all()).exists()
        else:
            return request.user.has_mill_access(obj)
```

### 4. 🌐 WebSocket Integration
```python
# apps/realtime/consumers.py
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from django.contrib.auth.models import AnonymousUser
from apps.mills.models import Mill, Sensor

class MillDashboardConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.mill_id = self.scope['url_route']['kwargs']['mill_id']
        self.mill_group_name = f'mill_{self.mill_id}'
        
        # Check permissions
        if not await self.check_permissions():
            await self.close()
            return
        
        # Join mill group
        await self.channel_layer.group_add(
            self.mill_group_name,
            self.channel_name
        )
        
        await self.accept()
        
        # Send initial data
        await self.send_initial_data()

    async def disconnect(self, close_code):
        # Leave mill group
        await self.channel_layer.group_discard(
            self.mill_group_name,
            self.channel_name
        )

    async def receive(self, text_data):
        try:
            data = json.loads(text_data)
            message_type = data.get('type')
            
            if message_type == 'request_sensor_data':
                await self.send_sensor_data(data.get('sensor_id'))
            elif message_type == 'acknowledge_alert':
                await self.acknowledge_alert(data.get('alert_id'))
            elif message_type == 'update_sensor_config':
                await self.update_sensor_config(data.get('sensor_id'), data.get('config'))
                
        except json.JSONDecodeError:
            await self.send_error('Invalid JSON format')
        except Exception as e:
            await self.send_error(f'Error processing message: {str(e)}')

    async def sensor_update(self, event):
        """Handle sensor update messages"""
        await self.send(text_data=json.dumps({
            'type': 'sensor_update',
            'sensor_id': event['sensor_id'],
            'reading': event['reading'],
            'timestamp': event['timestamp']
        }))

    async def mill_alert(self, event):
        """Handle mill alert messages"""
        await self.send(text_data=json.dumps({
            'type': 'alert',
            'alert': event['alert'],
            'timestamp': event['timestamp']
        }))

    @database_sync_to_async
    def check_permissions(self):
        """Check if user has permission to access this mill"""
        user = self.scope['user']
        if isinstance(user, AnonymousUser):
            return False
            
        try:
            mill = Mill.objects.get(id=self.mill_id)
            return user.has_mill_access(mill)
        except Mill.DoesNotExist:
            return False

    @database_sync_to_async
    def get_mill_data(self):
        """Get initial mill data"""
        try:
            mill = Mill.objects.prefetch_related('sensors').get(id=self.mill_id)
            return {
                'mill': {
                    'id': str(mill.id),
                    'name': mill.name,
                    'status': mill.status,
                    'capacity': mill.capacity
                },
                'sensors': [
                    {
                        'id': str(sensor.id),
                        'name': sensor.name,
                        'type': sensor.sensor_type,
                        'location': sensor.location,
                        'is_active': sensor.is_active,
                        'latest_reading': self.get_latest_reading(sensor)
                    }
                    for sensor in mill.sensors.all()
                ]
            }
        except Mill.DoesNotExist:
            return None
```

## 📚 Learning Resources

- [Official Django Documentation](https://docs.djangoproject.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Django Channels](https://channels.readthedocs.io/)
- [Two Scoops of Django](https://www.feldroy.com/books/two-scoops-of-django-3-x)

## 🎯 Best Practices I Follow

1. **Settings Management** - Separate settings for different environments
2. **Security First** - Always use Django's built-in security features
3. **Database Optimization** - Proper indexing and query optimization
4. **Testing** - Comprehensive test coverage
5. **Documentation** - Clear API documentation with DRF
6. **Scalability** - Design for horizontal scaling from the start

---

*"Django's 'batteries included' philosophy makes it perfect for rapid development of complex web applications."* - Danny Verheyen

[← Back to Main Profile](../README.md)
