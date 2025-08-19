# API REST Documentation

L'API REST de checklogs.dev vous permet d'intégrer facilement le monitoring de logs dans vos applications. Cette documentation couvre tous les endpoints disponibles, les formats de données et les meilleures pratiques.

## URL de base

```
https://checklogs.dev/api/logs
```

## Authentification

Toutes les requêtes API nécessitent une authentification via une clé API. Vous pouvez obtenir votre clé API depuis votre tableau de bord après avoir créé une application.

### Header d'authentification

```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

## Endpoints disponibles

### Créer un log

Envoie un nouveau log à votre application.

**POST** `/api/logs`

#### Paramètres du body

| Paramètre | Type | Requis | Description |
|-----------|------|--------|-------------|
| `message` | string | ✅ | Le message du log (max 1024 caractères) |
| `level` | string | ❌ | Niveau du log (`info`, `warning`, `error`, `critical`, `debug`) |
| `context` | object | ❌ | Données contextuelles (max 5000 caractères sérialisées) |
| `source` | string | ❌ | Source du log (max 100 caractères) |
| `user_id` | number | ❌ | ID de l'utilisateur associé |

#### Exemple de requête

```javascript
const response = await fetch('https://checklogs.dev/api/logs', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_API_KEY',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    message: 'User logged in successfully',
    level: 'info',
    context: {
      user_id: 123,
      ip_address: '192.168.1.1',
      user_agent: 'Mozilla/5.0...'
    },
    source: 'auth-service',
    user_id: 123
  })
});

const result = await response.json();
```

#### Réponse de succès (201)

```json
{
  "success": true,
  "status_code": 201,
  "timestamp": "2024-01-15T10:30:00Z",
  "message": "Log created successfully",
  "data": {
    "log_id": 12345,
    "application": "Mon Application",
    "level": "info",
    "received_at": "2024-01-15T10:30:00Z"
  }
}
```

### Récupérer les logs

Récupère les logs d'une application avec pagination et filtres.

**GET** `/api/logs`

#### Paramètres de requête

| Paramètre | Type | Description |
|-----------|------|-------------|
| `limit` | number | Nombre de logs à récupérer (max 1000, défaut 100) |
| `offset` | number | Décalage pour la pagination (défaut 0) |
| `level` | string | Filtrer par niveau de log |
| `since` | string | Date de début au format ISO 8601 |
| `until` | string | Date de fin au format ISO 8601 |

#### Exemple de requête

```javascript
const response = await fetch('https://checklogs.dev/api/logs?limit=50&level=error&since=2024-01-01T00:00:00Z', {
  headers: {
    'Authorization': 'Bearer YOUR_API_KEY'
  }
});

const result = await response.json();
```

#### Réponse de succès (200)

```json
{
  "success": true,
  "status_code": 200,
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "logs": [
      {
        "id": 12345,
        "level": "error",
        "message": "Database connection failed",
        "context": {
          "error_code": "CONN_TIMEOUT",
          "database": "primary"
        },
        "source": "api-server",
        "ip_address": "192.168.1.100",
        "user_id": null,
        "timestamp": "2024-01-15T10:25:00Z"
      }
    ],
    "pagination": {
      "limit": 50,
      "offset": 0,
      "total": 150,
      "has_more": true
    },
    "application": {
      "id": 1,
      "name": "Mon Application"
    }
  }
}
```

### Statistiques

Récupère les statistiques de votre application.

**GET** `/api/logs?stats=1`

#### Réponse de succès (200)

```json
{
  "success": true,
  "status_code": 200,
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "total_logs": 1500,
    "logs_today": 45,
    "stats_by_level": [
      {
        "level": "info",
        "count": 1200
      },
      {
        "level": "error",
        "count": 150
      },
      {
        "level": "warning",
        "count": 100
      }
    ],
    "daily_stats": [
      {
        "date": "2024-01-15",
        "count": 45
      },
      {
        "date": "2024-01-14",
        "count": 67
      }
    ],
    "application": {
      "id": 1,
      "name": "Mon Application"
    }
  }
}
```

## Codes d'erreur

### Erreurs d'authentification

| Code | Message | Description |
|------|---------|-------------|
| `401` | `INVALID_API_KEY` | Clé API manquante ou invalide |

### Erreurs de validation

| Code | Message | Description |
|------|---------|-------------|
| `400` | `INVALID_JSON` | Le JSON envoyé est malformé |
| `400` | `MISSING_MESSAGE` | Le champ message est requis |

### Erreurs serveur

| Code | Message | Description |
|------|---------|-------------|
| `500` | `DATABASE_ERROR` | Erreur de base de données |
| `500` | `INTERNAL_ERROR` | Erreur interne du serveur |

## Niveaux de logs

Les niveaux de logs supportés par ordre de gravité :

- **`debug`** - Informations de débogage pour le développement
- **`info`** - Informations générales sur le flux de l'application
- **`warning`** - Situations potentiellement problématiques
- **`error`** - Événements d'erreur qui pourraient permettre à l'application de continuer
- **`critical`** - Erreurs très graves qui pourraient causer l'arrêt de l'application

> **Note :** Les logs de niveau `error` et `critical` créent automatiquement des alertes dans votre tableau de bord.

## Limites et quotas

### Taille des données

- **Message** : Maximum 1024 caractères
- **Source** : Maximum 100 caractères  
- **Context** : Maximum 5000 caractères une fois sérialisé en JSON
- **Requête totale** : Maximum 10MB

### Rate limiting

- **100 requêtes par minute** par clé API
- **1000 logs par heure** par application
- Les limites peuvent être augmentées sur demande

### Pagination

- **Limite maximum** : 1000 logs par requête
- **Limite par défaut** : 100 logs par requête

## Exemples d'intégration

### Express.js Middleware

```javascript
const express = require('express');
const app = express();

// Middleware de logging
app.use(async (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', async () => {
    const duration = Date.now() - start;
    const level = res.statusCode >= 400 ? 'error' : 'info';
    
    try {
      await fetch('https://checklogs.dev/api/logs', {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer YOUR_API_KEY',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          message: `${req.method} ${req.url} - ${res.statusCode}`,
          level: level,
          context: {
            method: req.method,
            url: req.url,
            status_code: res.statusCode,
            duration_ms: duration,
            user_agent: req.get('User-Agent'),
            ip: req.ip
          },
          source: 'express-server'
        })
      });
    } catch (error) {
      console.error('Failed to send log:', error);
    }
  });
  
  next();
});
```

### Gestion d'erreurs globale

```javascript
// Gestionnaire d'erreurs non capturées
process.on('uncaughtException', async (error) => {
  try {
    await fetch('https://checklogs.dev/api/logs', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer YOUR_API_KEY',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message: `Uncaught Exception: ${error.message}`,
        level: 'critical',
        context: {
          error_name: error.name,
          error_message: error.message,
          stack: error.stack,
          timestamp: new Date().toISOString()
        },
        source: 'global-handler'
      })
    });
  } catch (logError) {
    console.error('Failed to log critical error:', logError);
  }
  
  process.exit(1);
});
```

### Monitoring de base de données

```javascript
const mysql = require('mysql2/promise');

const connection = await mysql.createConnection({
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'myapp'
});

// Wrapper pour les requêtes avec logging
async function queryWithLogging(sql, params = []) {
  const start = Date.now();
  
  try {
    const [results] = await connection.execute(sql, params);
    const duration = Date.now() - start;
    
    // Log des requêtes lentes
    if (duration > 1000) {
      await fetch('https://checklogs.dev/api/logs', {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer YOUR_API_KEY',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          message: 'Slow database query detected',
          level: 'warning',
          context: {
            sql: sql,
            duration_ms: duration,
            rows_affected: results.affectedRows || results.length
          },
          source: 'database'
        })
      });
    }
    
    return results;
  } catch (error) {
    // Log des erreurs de base de données
    await fetch('https://checklogs.dev/api/logs', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer YOUR_API_KEY',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message: `Database query failed: ${error.message}`,
        level: 'error',
        context: {
          sql: sql,
          error_code: error.code,
          error_message: error.message
        },
        source: 'database'
      })
    });
    
    throw error;
  }
}
```

## Bonnes pratiques

### Structuration des logs

1. **Messages clairs** : Utilisez des messages descriptifs et consistants
2. **Contexte riche** : Ajoutez des informations utiles dans le contexte
3. **Sources identifiables** : Utilisez des noms de sources cohérents

```javascript
// ✅ Bon exemple
{
  message: "User authentication successful",
  level: "info",
  context: {
    user_id: 123,
    login_method: "email",
    ip_address: "192.168.1.1",
    session_id: "abc123"
  },
  source: "auth-service"
}

// ❌ Mauvais exemple
{
  message: "ok",
  level: "info"
}
```

### Gestion des erreurs

1. **Retry automatique** : Implémentez une logique de retry pour les erreurs réseau
2. **Fallback local** : Sauvegardez localement en cas d'échec
3. **Monitoring silencieux** : Ne cassez pas votre application si le logging échoue

```javascript
async function sendLogWithRetry(logData, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch('https://checklogs.dev/api/logs', {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer YOUR_API_KEY',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(logData)
      });
      
      if (response.ok) {
        return await response.json();
      }
      
      throw new Error(`HTTP ${response.status}`);
    } catch (error) {
      console.warn(`Log attempt ${attempt} failed:`, error.message);
      
      if (attempt === maxRetries) {
        // Sauvegarder localement ou dans une queue
        console.error('Failed to send log after all retries');
        return null;
      }
      
      // Attendre avant le prochain essai
      await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
    }
  }
}
```

### Performance

1. **Logs asynchrones** : N'attendez pas la réponse pour continuer
2. **Batching** : Groupez plusieurs logs si possible
3. **Niveaux appropriés** : Utilisez `debug` en développement seulement

## Support et aide

- **Documentation SDK** : Utilisez notre [SDK Node.js](/read/?article=node-sdk) pour une intégration plus simple
- **Support** : contact@checklogs.dev
