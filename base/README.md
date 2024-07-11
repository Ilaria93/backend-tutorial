segui le istruzioni per creare e modificare i file già presenti in questo repository:
# Tutorial Project

Questo tutorial ti guiderà attraverso la creazione di un'applicazione Node.js con Express e un database PostgreSQL utilizzando Docker. 

## Prerequisiti

- Node.js
- Docker
- Git
- VS Code

## Passo 1: Configurare il progetto

1. Inizializza un nuovo progetto Node.js:
    ```sh
    npm init -y
    ```

2. Installa le dipendenze:
    ```sh
    npm install express sequelize pg pg-hstore
    ```

3. Crea la struttura delle cartelle:
    ```sh
    mkdir config models
    touch config/config.json models/index.js models/user.js index.js Dockerfile docker-compose.yml .gitignore
    ```

4. Configura i file necessari:
   - `config/config.json`:
     ```json
     {
       "development": {
         "username": "myuser",
         "password": "mypassword",
         "database": "mydatabase",
         "host": "db",
         "dialect": "postgres"
       }
     }
     ```

   - `models/index.js`:
     ```javascript
     const { Sequelize, DataTypes } = require('sequelize');
     const config = require('../config/config.json').development;

     const sequelize = new Sequelize(config.database, config.username, config.password, {
       host: config.host,
       dialect: config.dialect
     });

     const db = {};

     db.Sequelize = Sequelize;
     db.sequelize = sequelize;
     db.User = require('./user')(sequelize, DataTypes);

     module.exports = db;
     ```

   - `models/user.js`:
     ```javascript
     module.exports = (sequelize, DataTypes) => {
       const User = sequelize.define('User', {
         name: {
           type: DataTypes.STRING,
           allowNull: false
         },
         email: {
           type: DataTypes.STRING,
           allowNull: false,
           unique: true
         }
       });

       return User;
     };
     ```

   - `index.js`:
     ```javascript
     const express = require('express');
     const { sequelize, User } = require('./models');

     const app = express();
     app.use(express.json());

     const PORT = process.env.PORT || 3000;

     app.get('/users', async (req, res) => {
       try {
         const users = await User.findAll();
         res.json(users);
       } catch (err) {
         res.status(500).json({ error: err.message });
       }
     });

     app.post('/users', async (req, res) => {
       try {
         const newUser = await User.create(req.body);
         res.json(newUser);
       } catch (err) {
         res.status(500).json({ error: err.message });
       }
     });

     app.listen(PORT, async () => {
       console.log(`Server running on port ${PORT}`);
       await sequelize.sync({ force: true });
     });
     ```

   - `Dockerfile`:
     ```Dockerfile
     FROM node:14

     WORKDIR /usr/src/app

     COPY package*.json ./

     RUN npm install

     COPY . .

     CMD [ "node", "index.js" ]

     EXPOSE 3000
     ```

   - `docker-compose.yml`:
     ```yaml
     version: '3.8'

     services:
       db:
         image: postgres:latest
         environment:
           POSTGRES_DB: mydatabase
           POSTGRES_USER: myuser
           POSTGRES_PASSWORD: mypassword
         ports:
           - "5432:5432"
         volumes:
           - pgdata:/var/lib/postgresql/data

       app:
         build: .
         ports:
           - "3000:3000"
         depends_on:
           - db
         environment:
           - DB_HOST=db
           - DB_USER=myuser
           - DB_PASS=mypassword
           - DB_NAME=mydatabase

     volumes:
       pgdata:
     ```

   - `.gitignore`:
     ```gitignore
     node_modules
     pgdata
     ```

## Passo 2: Avviare l'applicazione

1. Costruisci e avvia i container Docker:
    ```sh
    docker-compose up
    ```

2. L'applicazione dovrebbe essere in esecuzione su `http://localhost:3000`.

## Passo 3: Aggiungere nuove funzionalità

Per aggiungere nuove funzionalità, come aggiornare o cancellare utenti, puoi modificare `index.js` nel seguente modo:

- Aggiungi i seguenti endpoint a `index.js`:

   ```javascript
   // Endpoint: Get a user by ID
   app.get('/users/:id', async (req, res) => {
     try {
       const user = await User.findByPk(req.params.id);
       if (user) {
         res.json(user);
       } else {
         res.status(404).json({ error: 'User not found' });
       }
     } catch (err) {
       res.status(500).json({ error: err.message });
     }
   });

   // Endpoint: Update a user by ID
   app.put('/users/:id', async (req, res) => {
     try {
       const [updated] = await User.update(req.body, {
         where: { id: req.params.id }
       });
       if (updated) {
         const updatedUser = await User.findByPk(req.params.id);
         res.status(200).json(updatedUser);
       } else {
         res.status(404).json({ error: 'User not found' });
       }
     } catch (err) {
       res.status(500).json({ error: err.message });
     }
   });

   // Endpoint: Delete a user by ID
   app.delete('/users/:id', async (req, res) => {
     try {
       const deleted = await User.destroy({
         where: { id: req.params.id }
       });
       if (deleted) {
         res.status(204).send();
       } else {
         res.status(404).json({ error: 'User not found' });
       }
     } catch (err) {
       res.status(500).json({ error: err.message });
     }
   });
