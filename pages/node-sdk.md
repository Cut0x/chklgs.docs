# SDK Node.js Official

Le SDK Node.js officiel de checklogs.dev simplifie l'intégration du monitoring de logs dans vos applications Node.js. Il supporte à la fois CommonJS et ES Modules avec TypeScript complet.

## Installation

```bash
npm install @checklogs/node-sdk
```

## Configuration rapide

Pour une configuration guidée automatique :

```bash
npx @checklogs/node-sdk quick-start
```

Cette commande va :
- Détecter votre type de projet (CommonJS ou ES Modules)
- Créer des fichiers de test appropriés
- Configurer une configuration de base
- Fournir les étapes suivantes

## Démarrage rapide

### CommonJS (require)

```javascript
const { createLogger } = require('@checklogs/node-sdk');

// Créer une instance de logger
const logger = createLogger('your-api-key-here');

// Envoyer des logs
await logger.info('Application started');
await logger.error('Something went wrong', { error_code: 500 });
```

### ES Modules (import)

```javascript
import { createLogger } from '@checklogs/node-sdk';

// Créer une instance de logger
const logger = createLogger('your-api-key-here');

// Envoyer des logs
await logger.info('Application started');
await logger.error('Something went wrong', { error_code: 500 });
```

## Support des modules

Ce package supporte à la fois CommonJS et ES modules :

- **CommonJS** : Utilisez la syntaxe `require()` - fonctionne avec les projets Node.js traditionnels
- **ES Modules** : Utilisez la syntaxe `import/export` - fonctionne avec les projets Node.js modernes avec `"type": "module"` dans package.json ou les fichiers `.mjs`

Le package fournit automatiquement le bon format selon la façon dont vous l'importez.

## Classes principales

### CheckLogsClient

Client de base pour l'API REST.

#### CommonJS

```javascript
const { CheckLogsClient } = require('@checklogs/node-sdk');

const client = new CheckLogsClient('your-api-key');
```

#### ES Modules

```javascript
import { CheckLogsClient } from '@checklogs/node-sdk';

const client = new CheckLogsClient('your-api-key');
```

#### Utilisation

```javascript
// Envoyer un log
await client.log({
  message: 'User logged in',
  level: 'info',
  context: { user_id: 123, ip: '192.168.1.1' }
});

// Récupérer les logs
const logs = await client.getLogs({
  limit: 100,
  level: 'error',
  since: '2024-01-01'
});

// Obtenir les statistiques
const stats = await client.getStats();
```

### CheckLogsLogger

Logger amélioré avec fonctionnalités avancées.

#### CommonJS

```javascript
const { CheckLogsLogger } = require('@checklogs/node-sdk');
```

#### ES Modules

```javascript
import { CheckLogsLogger } from '@checklogs/node-sdk';
```

#### Utilisation

```javascript
const logger = new CheckLogsLogger('your-api-key', {
  source: 'my-app',
  defaultContext: { version: '1.0.0' },
  consoleOutput: true
});

// Méthodes de convenance
await logger.info('Info message');
await logger.warning('Warning message');
await logger.error('Error message');
await logger.critical('Critical message');
await logger.debug('Debug message');
```

## Options de configuration

### Options du Client

```javascript
const client = new CheckLogsClient('api-key', {
  timeout: 5000,           // Timeout de requête en ms
  validatePayload: true,   // Valider les données avant envoi
  baseUrl: 'https://checklogs.dev/api/logs'  // URL personnalisée
});
```

### Options du Logger

```javascript
const logger = new CheckLogsLogger('api-key', {
  // Options du client
  timeout: 5000,
  validatePayload: true,
  
  // Options spécifiques au logger
  source: 'my-app',                    // Source par défaut
  user_id: 123,                        // ID utilisateur par défaut
  defaultContext: { env: 'prod' },     // Contexte par défaut
  silent: false,                       // Supprimer toute sortie
  consoleOutput: true,                 // Logger aussi sur la console
  enabledLevels: ['info', 'error'],    // Seulement ces niveaux
  includeTimestamp: true,              // Ajouter timestamp au contexte
  includeHostname: true,               // Ajouter hostname au contexte
  includeProcessInfo: true             // Ajouter infos du processus
});
```

## Fonctionnalités avancées

### Child Loggers

Créez des loggers enfants avec contexte hérité :

```javascript
const mainLogger = new CheckLogsLogger('api-key', {
  defaultContext: { service: 'api' }
});

const userLogger = mainLogger.child({ module: 'user' });
const orderLogger = mainLogger.child({ module: 'orders' });

// Chaque enfant hérite du contexte parent
await userLogger.info('User created');  // Contexte: { service: 'api', module: 'user' }
await orderLogger.error('Order failed'); // Contexte: { service: 'api', module: 'orders' }
```

### Mesure de temps

Mesurez le temps d'exécution :

```javascript
const endTimer = logger.time('db-query', 'Executing database query');

// ... votre code ici ...

const duration = endTimer(); // Log automatique de fin avec durée
console.log(`Operation took ${duration}ms`);
```

### Statistiques et Analytics

```javascript
// Obtenir les statistiques de base
const stats = await client.stats.getStats();
console.log('Total logs:', stats.data.total_logs);

// Obtenir un résumé des analytics
const summary = await client.stats.getSummary();
console.log('Error rate:', summary.data.analytics.error_rate);

// Obtenir des métriques spécifiques
const errorRate = await client.stats.getErrorRate();
const trend = await client.stats.getTrend();
const peakDay = await client.stats.getPeakDay();
```

### Gestion des erreurs

Le SDK fournit des types d'erreur spécifiques :

```javascript
const { ApiError, NetworkError, ValidationError } = require('@checklogs/node-sdk');

try {
  await logger.log({ /* données invalides */ });
} catch (error) {
  if (error instanceof ValidationError) {
    console.log('Validation failed:', error.message);
  } else if (error instanceof ApiError) {
    console.log('API error:', error.statusCode, error.message);
    
    if (error.isAuthError()) {
      console.log('Authentication problem');
    } else if (error.isRateLimitError()) {
      console.log('Rate limit exceeded');
    }
  } else if (error instanceof NetworkError) {
    console.log('Network problem:', error.message);
    
    if (error.isTimeoutError()) {
      console.log('Request timed out');
    }
  }
}
```

### Mécanisme de retry

Le logger retente automatiquement les requêtes échouées :

```javascript
// Vérifier le statut de la queue de retry
const status = logger.getRetryQueueStatus();
console.log(`${status.count} logs pending retry`);

// Attendre que tous les logs soient envoyés
const success = await logger.flush(30000); // timeout de 30 secondes
if (success) {
  console.log('All logs sent successfully');
}

// Vider la queue de retry si nécessaire
logger.clearRetryQueue();
```

## Niveaux de logs

Niveaux supportés (par ordre de gravité) :

- **`debug`** - Informations de développement et dépannage
- **`info`** - Flux général de l'application
- **`warning`** - Situations potentiellement problématiques
- **`error`** - Événements d'erreur qui pourraient permettre à l'application de continuer
- **`critical`** - Erreurs très graves qui pourraient causer l'arrêt de l'application

## Validation des données

Le SDK valide et nettoie automatiquement les données :

- **Message** : Requis, max 1024 caractères
- **Level** : Doit être un niveau valide, par défaut 'info'
- **Source** : Max 100 caractères
- **Context** : Objets seulement, max 5000 caractères sérialisés
- **User ID** : Doit être un nombre

## Exemples d'intégration

### Middleware Express.js

#### Version CommonJS

```javascript
const express = require('express');
const { createLogger } = require('@checklogs/node-sdk');

const app = express();
const logger = createLogger('your-api-key');

// Middleware de logging des requêtes
app.use((req, res, next) => {
  const requestLogger = logger.child({
    request_id: Math.random().toString(36),
    method: req.method,
    url: req.url
  });
  
  req.logger = requestLogger;
  next();
});

// Gestionnaire de route
app.get('/users/:id', async (req, res) => {
  try {
    req.logger.info('Fetching user', { user_id: req.params.id });
    
    // ... votre logique ici ...
    
    req.logger.info('User fetched successfully');
    res.json({ user: userData });
  } catch (error) {
    req.logger.error('Failed to fetch user', { 
      error: error.message,
      stack: error.stack 
    });
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

#### Version ES Modules

```javascript
import express from 'express';
import { createLogger } from '@checklogs/node-sdk';

const app = express();
const logger = createLogger('your-api-key');

// ... le reste du code est identique ...
```

### Monitoring de tâches CRON

```javascript
const { createLogger } = require('@checklogs/node-sdk');

const logger = createLogger('your-api-key', {
  source: 'cron-job',
  defaultContext: { job: 'daily-cleanup' }
});

async function dailyCleanup() {
  const endTimer = logger.time('cleanup', 'Starting daily cleanup');
  
  try {
    logger.info('Cleanup started');
    
    // ... logique de nettoyage ...
    
    logger.info('Cleanup completed successfully', { 
      records_cleaned: 1500,
      duration_ms: endTimer()
    });
  } catch (error) {
    logger.critical('Cleanup failed', { 
      error: error.message,
      duration_ms: endTimer()
    });
    throw error;
  }
}
```

### Monitoring de base de données

```javascript
import { createLogger } from '@checklogs/node-sdk';
import mysql from 'mysql2/promise';

const logger = createLogger('your-api-key', {
  source: 'database',
  defaultContext: { service: 'mysql' }
});

class Database {
  constructor() {
    this.connection = null;
    this.logger = logger.child({ module: 'database' });
  }
  
  async connect() {
    try {
      this.connection = await mysql.createConnection({
        host: 'localhost',
        user: 'root',
        password: 'password',
        database: 'myapp'
      });
      
      this.logger.info('Database connected successfully');
    } catch (error) {
      this.logger.critical('Database connection failed', {
        error: error.message,
        host: 'localhost'
      });
      throw error;
    }
  }
  
  async query(sql, params = []) {
    const endTimer = this.logger.time('query', 'Executing database query');
    
    try {
      const [results] = await this.connection.execute(sql, params);
      const duration = endTimer();
      
      // Logger les requêtes lentes
      if (duration > 1000) {
        this.logger.warning('Slow query detected', {
          sql: sql.substring(0, 100) + '...',
          duration_ms: duration,
          rows_affected: results.affectedRows || results.length
        });
      } else {
        this.logger.debug('Query executed', {
          duration_ms: duration,
          rows_affected: results.affectedRows || results.length
        });
      }
      
      return results;
    } catch (error) {
      endTimer();
      this.logger.error('Query failed', {
        sql: sql.substring(0, 100) + '...',
        error_code: error.code,
        error_message: error.message
      });
      throw error;
    }
  }
}
```

### Gestion globale des erreurs

```javascript
import { createLogger } from '@checklogs/node-sdk';

const logger = createLogger('your-api-key', {
  source: 'global-handler',
  defaultContext: { 
    pid: process.pid,
    node_version: process.version
  }
});

// Gestion des exceptions non capturées
process.on('uncaughtException', async (error) => {
  await logger.critical('Uncaught Exception', {
    error_name: error.name,
    error_message: error.message,
    stack: error.stack
  });
  
  // Attendre que le log soit envoyé avant de quitter
  await logger.flush(5000);
  process.exit(1);
});

// Gestion des promesses rejetées
process.on('unhandledRejection', async (reason, promise) => {
  await logger.error('Unhandled Promise Rejection', {
    reason: reason instanceof Error ? reason.message : String(reason),
    stack: reason instanceof Error ? reason.stack : undefined
  });
});

// Gestion propre de l'arrêt
process.on('SIGTERM', async () => {
  logger.info('Received SIGTERM, shutting down gracefully');
  await logger.flush(10000);
  process.exit(0);
});
```

## Support TypeScript

Définitions TypeScript complètes incluses :

```typescript
import { CheckLogsLogger, LogLevel, LogData } from '@checklogs/node-sdk';

const logger = new CheckLogsLogger('api-key');

const logData: LogData = {
  message: 'User action',
  level: 'info' as LogLevel,
  context: { userId: 123 }
};

await logger.log(logData);

// Types d'erreur
import { ApiError, NetworkError, ValidationError } from '@checklogs/node-sdk';

try {
  await logger.info('Test');
} catch (error) {
  if (error instanceof ApiError) {
    console.log('Status:', error.statusCode);
    console.log('Is auth error:', error.isAuthError());
  }
}
```

### Interfaces TypeScript

```typescript
interface LogData {
  message: string;
  level?: LogLevel;
  context?: Record<string, any>;
  source?: string;
  user_id?: number;
}

interface ClientOptions {
  timeout?: number;
  validatePayload?: boolean;
  baseUrl?: string;
}

interface LoggerOptions extends ClientOptions {
  source?: string;
  user_id?: number;
  defaultContext?: Record<string, any>;
  silent?: boolean;
  consoleOutput?: boolean;
  enabledLevels?: LogLevel[];
  includeTimestamp?: boolean;
  includeHostname?: boolean;
  includeProcessInfo?: boolean;
}

type LogLevel = 'debug' | 'info' | 'warning' | 'error' | 'critical';
```

## Bonnes pratiques

### Configuration des niveaux

```javascript
// En développement
const logger = new CheckLogsLogger('api-key', {
  enabledLevels: ['debug', 'info', 'warning', 'error', 'critical'],
  consoleOutput: true
});

// En production
const logger = new CheckLogsLogger('api-key', {
  enabledLevels: ['info', 'warning', 'error', 'critical'],
  consoleOutput: false
});
```

### Contexte structuré

```javascript
// ✅ Bon exemple
const userLogger = logger.child({
  module: 'user-service',
  version: '1.2.0'
});

await userLogger.info('User registration completed', {
  user_id: 123,
  email: 'user@example.com',
  registration_method: 'email',
  ip_address: req.ip
});

// ❌ Mauvais exemple
await logger.info('user ok');
```

### Gestion des performances

```javascript
// Utilisez des enfants pour éviter la répétition
const requestLogger = logger.child({
  request_id: generateRequestId(),
  user_id: req.user?.id
});

// Async/await sans bloquer
requestLogger.info('Request started'); // N'attendez pas
await processRequest();
requestLogger.info('Request completed'); // N'attendez pas

// Flush seulement quand nécessaire
await logger.flush(); // Avant l'arrêt de l'app
```

### Monitoring des métriques

```javascript
const performanceLogger = logger.child({ module: 'performance' });

// Mesurer les opérations importantes
const endTimer = performanceLogger.time('api-call', 'External API call');

try {
  const response = await fetch('https://api.external.com/data');
  const duration = endTimer();
  
  performanceLogger.info('API call successful', {
    status: response.status,
    duration_ms: duration
  });
} catch (error) {
  const duration = endTimer();
  
  performanceLogger.error('API call failed', {
    error: error.message,
    duration_ms: duration
  });
}
```

## Dépannage

### Problèmes courants

#### Erreurs d'authentification

```javascript
try {
  await logger.info('Test');
} catch (error) {
  if (error instanceof ApiError && error.isAuthError()) {
    console.error('Invalid API key. Check your configuration.');
  }
}
```

#### Timeouts

```javascript
const logger = new CheckLogsLogger('api-key', {
  timeout: 10000 // 10 secondes au lieu de 5
});
```

#### Rate limiting

```javascript
try {
  await logger.info('Test');
} catch (error) {
  if (error instanceof ApiError && error.isRateLimitError()) {
    console.warn('Rate limit exceeded, retrying later...');
    // Le SDK retente automatiquement
  }
}
```

### Debug

Activez le debug pour voir les requêtes :

```javascript
const logger = new CheckLogsLogger('api-key', {
  debug: true // Active les logs de debug du SDK
});
```

## Migration

### Depuis l'API REST directe

```javascript
// Avant (API REST directe)
await fetch('https://checklogs.dev/api/logs', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer api-key',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    message: 'User logged in',
    level: 'info'
  })
});

// Après (SDK)
const logger = createLogger('api-key');
await logger.info('User logged in');
```

### Depuis winston

```javascript
// Avant (winston)
const winston = require('winston');
const logger = winston.createLogger({
  transports: [new winston.transports.Console()]
});

logger.info('Test message');

// Après (checklogs + winston en backup)
const { CheckLogsLogger } = require('@checklogs/node-sdk');
const winston = require('winston');

const checklogsLogger = new CheckLogsLogger('api-key');
const winstonLogger = winston.createLogger({
  transports: [new winston.transports.Console()]
});

class HybridLogger {
  async info(message, context = {}) {
    // Envoyer vers checklogs et winston
    await Promise.allSettled([
      checklogsLogger.info(message, context),
      Promise.resolve(winstonLogger.info(message, context))
    ]);
  }
  
  // Répéter pour les autres niveaux...
}

const logger = new HybridLogger();
await logger.info('Test message');
```

## Changelog

### Version 1.0.2 (20/08/2025)
- Ajout du support TypeScript complet
- Nouvelles méthodes d'enregistrement