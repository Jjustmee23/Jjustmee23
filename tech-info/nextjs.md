# ⚫ Next.js - The React Framework for Production

![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)

## 📋 What is Next.js?

**Next.js** is a React framework that gives you building blocks to create web applications. By framework, we mean Next.js handles the tooling and configuration needed for React, and provides additional structure, features, and optimizations for your application.

## 🔧 Key Features

- **Server-Side Rendering (SSR)** - Better SEO and performance
- **Static Site Generation (SSG)** - Pre-built pages for speed
- **API Routes** - Full-stack capabilities
- **File-based Routing** - Automatic route generation
- **Image Optimization** - Built-in image optimization
- **TypeScript Support** - First-class TypeScript integration

## 💡 Why I Use Next.js

### 🏭 Industrial Web Applications
- **Mill Management Dashboards** - Server-side rendered for better performance
- **Real-time Monitoring Interfaces** - Optimized for industrial environments
- **Client Portals** - Secure customer-facing applications
- **API Integration** - Seamless backend connectivity

### 🎮 Gaming Platform Frontend
- **Tournament Platforms** - High-performance gaming interfaces
- **Player Statistics** - SEO-optimized profile pages
- **Community Features** - Social gaming experiences
- **Real-time Leaderboards** - Live updating game data

## 🛠️ My Next.js Architecture

### Project Structure:
```
mill-dashboard-next/
├── pages/
│   ├── api/
│   │   ├── mills/
│   │   │   └── [id].js
│   │   ├── sensors/
│   │   │   └── data.js
│   │   └── auth/
│   │       └── login.js
│   ├── dashboard/
│   │   ├── index.js
│   │   ├── mills/
│   │   │   └── [id].js
│   │   └── sensors.js
│   ├── _app.js
│   ├── _document.js
│   └── index.js
├── components/
│   ├── Layout/
│   │   ├── Header.jsx
│   │   ├── Sidebar.jsx
│   │   └── Footer.jsx
│   ├── Dashboard/
│   │   ├── MillOverview.jsx
│   │   ├── SensorGrid.jsx
│   │   └── AlertPanel.jsx
│   └── UI/
│       ├── Button.jsx
│       ├── Card.jsx
│       └── Modal.jsx
├── hooks/
│   ├── useAuth.js
│   ├── useSWR.js
│   └── useWebSocket.js
├── lib/
│   ├── auth.js
│   ├── db.js
│   └── api.js
├── styles/
│   ├── globals.css
│   └── components/
└── public/
    ├── images/
    └── icons/
```

## 🌟 My Next.js Projects

### 1. 🏭 Mill Management Dashboard
```javascript
// pages/dashboard/mills/[id].js
import { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import Head from 'next/head';
import Image from 'next/image';
import { GetServerSideProps } from 'next';
import Layout from '../../../components/Layout';
import SensorGrid from '../../../components/Dashboard/SensorGrid';
import AlertPanel from '../../../components/Dashboard/AlertPanel';
import { getMillById, getSensorData } from '../../../lib/api';

export default function MillDashboard({ mill, initialSensorData }) {
  const router = useRouter();
  const [sensorData, setSensorData] = useState(initialSensorData);
  const [alerts, setAlerts] = useState([]);

  useEffect(() => {
    // Set up real-time updates
    const ws = new WebSocket(`${process.env.NEXT_PUBLIC_WS_URL}/mills/${mill.id}`);
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setSensorData(prev => ({ ...prev, ...data }));
      
      // Check for alerts
      if (data.alert) {
        setAlerts(prev => [data.alert, ...prev.slice(0, 9)]);
      }
    };

    return () => ws.close();
  }, [mill.id]);

  return (
    <Layout>
      <Head>
        <title>{mill.name} - Mill Dashboard</title>
        <meta name="description" content={`Real-time monitoring for ${mill.name}`} />
      </Head>

      <div className="mill-dashboard">
        <div className="mill-header">
          <div className="mill-info">
            <Image
              src={mill.image || '/images/default-mill.jpg'}
              alt={mill.name}
              width={100}
              height={100}
              className="mill-image"
            />
            <div>
              <h1>{mill.name}</h1>
              <p>Location: {mill.location}</p>
              <p>Capacity: {mill.capacity} tons/day</p>
              <div className={`status ${mill.status}`}>
                {mill.status.toUpperCase()}
              </div>
            </div>
          </div>
        </div>

        <AlertPanel alerts={alerts} />
        
        <SensorGrid 
          sensors={sensorData} 
          millId={mill.id}
          onSensorUpdate={setSensorData}
        />

        <div className="mill-stats">
          <div className="stat-card">
            <h3>Today's Production</h3>
            <p className="stat-value">{mill.todayProduction} tons</p>
          </div>
          <div className="stat-card">
            <h3>Efficiency</h3>
            <p className="stat-value">{mill.efficiency}%</p>
          </div>
          <div className="stat-card">
            <h3>Active Sensors</h3>
            <p className="stat-value">{Object.keys(sensorData).length}</p>
          </div>
        </div>
      </div>
    </Layout>
  );
}

export const getServerSideProps = async (context) => {
  const { id } = context.params;
  
  try {
    const mill = await getMillById(id);
    const initialSensorData = await getSensorData(id);
    
    if (!mill) {
      return {
        notFound: true,
      };
    }

    return {
      props: {
        mill,
        initialSensorData,
      },
    };
  } catch (error) {
    console.error('Error fetching mill data:', error);
    return {
      props: {
        mill: null,
        initialSensorData: {},
      },
    };
  }
};
```

### 2. 🎮 Gaming Tournament Platform
```javascript
// pages/tournaments/[id].js
import { useState, useEffect } from 'react';
import Head from 'next/head';
import { useRouter } from 'next/router';
import { io } from 'socket.io-client';
import Layout from '../../components/Layout';
import PlayerList from '../../components/Tournament/PlayerList';
import MatchBracket from '../../components/Tournament/MatchBracket';
import LiveChat from '../../components/Tournament/LiveChat';

export default function TournamentPage({ tournament, matches, players }) {
  const router = useRouter();
  const [socket, setSocket] = useState(null);
  const [liveMatches, setLiveMatches] = useState(matches);
  const [tournamentPlayers, setTournamentPlayers] = useState(players);

  useEffect(() => {
    const newSocket = io(process.env.NEXT_PUBLIC_SOCKET_URL);
    setSocket(newSocket);

    // Join tournament room
    newSocket.emit('join-tournament', tournament.id);

    // Listen for tournament updates
    newSocket.on('match-update', (matchData) => {
      setLiveMatches(prev => 
        prev.map(match => 
          match.id === matchData.id ? { ...match, ...matchData } : match
        )
      );
    });

    newSocket.on('player-joined', (player) => {
      setTournamentPlayers(prev => [...prev, player]);
    });

    newSocket.on('tournament-status', (status) => {
      // Handle tournament status changes
      if (status === 'started') {
        router.reload();
      }
    });

    return () => newSocket.close();
  }, [tournament.id, router]);

  const handleJoinTournament = async () => {
    try {
      const response = await fetch(`/api/tournaments/${tournament.id}/join`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      });

      if (response.ok) {
        const updatedTournament = await response.json();
        router.reload();
      }
    } catch (error) {
      console.error('Failed to join tournament:', error);
    }
  };

  return (
    <Layout>
      <Head>
        <title>{tournament.name} - Gaming Tournament</title>
        <meta name="description" content={tournament.description} />
        <meta property="og:title" content={tournament.name} />
        <meta property="og:description" content={tournament.description} />
        <meta property="og:image" content={tournament.banner} />
      </Head>

      <div className="tournament-page">
        <div className="tournament-header">
          <div className="tournament-banner">
            <img src={tournament.banner} alt={tournament.name} />
          </div>
          <div className="tournament-info">
            <h1>{tournament.name}</h1>
            <p>{tournament.description}</p>
            <div className="tournament-details">
              <span>Game: {tournament.game}</span>
              <span>Prize Pool: ${tournament.prizePool}</span>
              <span>Players: {tournamentPlayers.length}/{tournament.maxPlayers}</span>
              <span>Status: {tournament.status}</span>
            </div>
            {tournament.status === 'open' && (
              <button 
                onClick={handleJoinTournament}
                className="join-button"
              >
                Join Tournament
              </button>
            )}
          </div>
        </div>

        <div className="tournament-content">
          <div className="main-content">
            <MatchBracket matches={liveMatches} />
          </div>
          
          <div className="sidebar">
            <PlayerList players={tournamentPlayers} />
            <LiveChat tournamentId={tournament.id} socket={socket} />
          </div>
        </div>
      </div>
    </Layout>
  );
}

export const getStaticProps = async (context) => {
  const { id } = context.params;
  
  try {
    const [tournament, matches, players] = await Promise.all([
      getTournamentById(id),
      getMatchesByTournament(id),
      getPlayersByTournament(id)
    ]);

    return {
      props: {
        tournament,
        matches,
        players,
      },
      revalidate: 60, // Revalidate every minute
    };
  } catch (error) {
    return {
      notFound: true,
    };
  }
};

export const getStaticPaths = async () => {
  const tournaments = await getAllTournaments();
  
  const paths = tournaments.map((tournament) => ({
    params: { id: tournament.id.toString() },
  }));

  return {
    paths,
    fallback: 'blocking',
  };
};
```

### 3. 🔧 API Routes for Backend Integration
```javascript
// pages/api/mills/[id]/sensors.js
import { getSession } from 'next-auth/react';
import { connectToDatabase } from '../../../../lib/db';

export default async function handler(req, res) {
  const session = await getSession({ req });
  
  if (!session) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  const { id } = req.query;

  if (req.method === 'GET') {
    try {
      const { db } = await connectToDatabase();
      
      const sensors = await db
        .collection('sensors')
        .find({ millId: id })
        .sort({ createdAt: -1 })
        .toArray();

      // Get latest readings for each sensor
      const sensorsWithData = await Promise.all(
        sensors.map(async (sensor) => {
          const latestReading = await db
            .collection('sensor_readings')
            .findOne(
              { sensorId: sensor._id },
              { sort: { timestamp: -1 } }
            );

          return {
            ...sensor,
            latestReading: latestReading || null,
            status: getStatus(latestReading),
          };
        })
      );

      res.status(200).json(sensorsWithData);
    } catch (error) {
      console.error('Error fetching sensors:', error);
      res.status(500).json({ error: 'Internal server error' });
    }
  } else if (req.method === 'POST') {
    // Add new sensor reading
    try {
      const { sensorId, value, unit, metadata } = req.body;
      
      const { db } = await connectToDatabase();
      
      const reading = {
        sensorId,
        value: parseFloat(value),
        unit,
        metadata: metadata || {},
        timestamp: new Date(),
        millId: id,
      };

      await db.collection('sensor_readings').insertOne(reading);

      // Check for alerts
      const sensor = await db.collection('sensors').findOne({ _id: sensorId });
      const alert = checkForAlerts(sensor, reading);
      
      if (alert) {
        await db.collection('alerts').insertOne({
          ...alert,
          millId: id,
          createdAt: new Date(),
        });
      }

      res.status(201).json({ success: true, alert });
    } catch (error) {
      console.error('Error adding sensor reading:', error);
      res.status(500).json({ error: 'Internal server error' });
    }
  } else {
    res.setHeader('Allow', ['GET', 'POST']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}

function getStatus(reading) {
  if (!reading) return 'offline';
  
  const timeSinceReading = Date.now() - new Date(reading.timestamp).getTime();
  const fiveMinutes = 5 * 60 * 1000;
  
  if (timeSinceReading > fiveMinutes) return 'offline';
  
  // Check thresholds (simplified)
  if (reading.value > 50) return 'critical';
  if (reading.value > 40) return 'warning';
  return 'normal';
}

function checkForAlerts(sensor, reading) {
  const thresholds = sensor.thresholds || {};
  
  if (reading.value > thresholds.critical) {
    return {
      type: 'critical',
      message: `${sensor.name} reading ${reading.value}${reading.unit} exceeds critical threshold`,
      sensorId: sensor._id,
      value: reading.value,
      threshold: thresholds.critical,
    };
  }
  
  if (reading.value > thresholds.warning) {
    return {
      type: 'warning',
      message: `${sensor.name} reading ${reading.value}${reading.unit} exceeds warning threshold`,
      sensorId: sensor._id,
      value: reading.value,
      threshold: thresholds.warning,
    };
  }
  
  return null;
}
```

## 🚀 Performance Optimization

### Image Optimization:
```javascript
// components/OptimizedImage.jsx
import Image from 'next/image';
import { useState } from 'react';

const OptimizedImage = ({ src, alt, width, height, priority = false }) => {
  const [isLoading, setIsLoading] = useState(true);

  return (
    <div className="image-container">
      <Image
        src={src}
        alt={alt}
        width={width}
        height={height}
        priority={priority}
        quality={85}
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAYEBQYFBAYGBQYHBwYIChAKCgkJChQODwwQFxQYGBcUFhYaHSUfGhsjHBYWICwgIyYnKSopGR8tMC0oMCUoKSj/2wBDAQcHBwoIChMKChMoGhYaKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCgoKCj/wAARCAABAAEDASIAAhEBAxEB/8QAFQABAQAAAAAAAAAAAAAAAAAAAAv/xAAUEAEAAAAAAAAAAAAAAAAAAAAA/8QAFQEBAQAAAAAAAAAAAAAAAAAAAAX/xAAUEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwCdABmX/9k="
        onLoadingComplete={() => setIsLoading(false)}
        className={`image ${isLoading ? 'loading' : 'loaded'}`}
      />
      {isLoading && <div className="image-skeleton" />}
    </div>
  );
};

export default OptimizedImage;
```

### Static Generation with ISR:
```javascript
// pages/mills/index.js
export const getStaticProps = async () => {
  try {
    const mills = await getAllMills();
    
    return {
      props: {
        mills,
      },
      revalidate: 300, // Revalidate every 5 minutes
    };
  } catch (error) {
    return {
      props: {
        mills: [],
      },
      revalidate: 60,
    };
  }
};
```

## 📊 SEO & Meta Optimization

### Custom Head Component:
```javascript
// components/SEOHead.jsx
import Head from 'next/head';

const SEOHead = ({ 
  title, 
  description, 
  keywords, 
  image, 
  url,
  type = 'website' 
}) => {
  const siteTitle = 'Mill Management System';
  const fullTitle = title ? `${title} | ${siteTitle}` : siteTitle;

  return (
    <Head>
      <title>{fullTitle}</title>
      <meta name="description" content={description} />
      <meta name="keywords" content={keywords} />
      <meta name="viewport" content="width=device-width, initial-scale=1" />
      
      {/* Open Graph */}
      <meta property="og:type" content={type} />
      <meta property="og:title" content={fullTitle} />
      <meta property="og:description" content={description} />
      <meta property="og:image" content={image} />
      <meta property="og:url" content={url} />
      
      {/* Twitter */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={fullTitle} />
      <meta name="twitter:description" content={description} />
      <meta name="twitter:image" content={image} />
      
      {/* Canonical URL */}
      <link rel="canonical" href={url} />
      
      {/* Favicon */}
      <link rel="icon" href="/favicon.ico" />
      <link rel="apple-touch-icon" href="/apple-touch-icon.png" />
    </Head>
  );
};

export default SEOHead;
```

## 🔒 Authentication & Security

### NextAuth.js Configuration:
```javascript
// pages/api/auth/[...nextauth].js
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { verifyPassword } from '../../../lib/auth';

export default NextAuth({
  providers: [
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' }
      },
      async authorize(credentials) {
        const user = await verifyUser(credentials.email, credentials.password);
        
        if (user) {
          return {
            id: user.id,
            email: user.email,
            name: user.name,
            role: user.role,
          };
        }
        
        return null;
      }
    })
  ],
  pages: {
    signIn: '/auth/signin',
    error: '/auth/error',
  },
  session: {
    strategy: 'jwt',
    maxAge: 24 * 60 * 60, // 24 hours
  },
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.role = user.role;
      }
      return token;
    },
    async session({ session, token }) {
      session.user.role = token.role;
      return session;
    },
  },
});
```

## 📚 Learning Resources

- [Official Next.js Documentation](https://nextjs.org/docs)
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [Learn Next.js](https://nextjs.org/learn)
- [Next.js Conf](https://nextjs.org/conf)

## 🎯 Best Practices I Follow

1. **File-based Routing** - Organized page structure
2. **API Routes** - Full-stack capabilities in one framework
3. **Image Optimization** - Automatic image optimization
4. **SEO Optimization** - Server-side rendering for better SEO
5. **Performance** - Code splitting and lazy loading
6. **TypeScript** - Type safety throughout the application

---

*"Next.js combines the power of React with production-ready features, making it perfect for scalable web applications."* - Danny Verheyen

[← Back to Main Profile](../README.md)
