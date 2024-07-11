# Tutorial Project

## Deploy su Heroku

1. **Accedi a Heroku:**
    ```sh
    heroku login
    ```

2. **Crea una nuova applicazione su Heroku:**
    ```sh
    heroku create your-app-name
    ```

3. **Aggiungi l'add-on PostgreSQL:**
    ```sh
    heroku addons:create heroku-postgresql:hobby-dev
    ```

4. **Configura le variabili d'ambiente per l'app su Heroku:**
    ```sh
    heroku config:set DB_HOST=your-db-host DB_USER=your-db-user DB_PASS=your-db-pass DB_NAME=your-db-name SECRET_KEY=your_secret_key
    ```

5. **Effettua il deploy dell'app:**
    ```sh
    git add .
    git commit -m "Add authentication and prepare for Heroku deploy"
    git push heroku main
    ```

6. **Migra il database su Heroku:**
    ```sh
    heroku run sequelize db:migrate
    ```

7. **Apri l'applicazione:**
    ```sh
    heroku open
    ```

## Endpoint di autenticazione

- **Registrazione:** `POST /register`
- **Login:** `POST /login`
- **Recupera utenti (protetto):** `GET /users`
