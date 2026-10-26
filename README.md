# Gestionale Fiera — Next.js + MySQL

Progetto iniziale che porta il gestionale della fiera da una pagina HTML statica a un'app Next.js con API server-side e MySQL. Mantiene il flusso di cassa, le cinque fasce, conferma vendita, statistiche, produttività oraria, esportazione CSV e gestione operatori.

## Attenzione: non è ancora collegato al server del cliente

Il progetto contiene il codice necessario, ma **non è stato possibile verificare la connessione dal backend al MySQL del cliente né eseguire un deployment**. Il test TCP fatto dal PC non garantisce che un provider cloud possa raggiungere il database. Non considerarlo pronto per la fiera finché non sono stati completati i passaggi di installazione e test.

## Requisiti

- Node.js 20 LTS o versione compatibile con Next.js 14.
- Un servizio che esegua Next.js/Node.js (GitHub Pages da solo non è sufficiente).
- Accesso MySQL con permessi per creare le tabelle richieste.
- Connettività dal server Next.js all'host MySQL e regole firewall che consentano solo le sorgenti necessarie.

## 1. Preparare le credenziali

1. La password MySQL precedente è stata esposta nella conversazione: chiedere al cliente di cambiarla prima di qualsiasi pubblicazione.
2. Copiare `.env.example` in `.env.local`.
3. Inserire nel file locale i dati MySQL aggiornati e generare un segreto casuale di almeno 32 caratteri per `SESSION_SECRET`.
4. Non caricare mai `.env.local` su GitHub. È già escluso dal `.gitignore`.
5. In produzione impostare le stesse variabili nella sezione Environment Variables del provider. Abilitare `DB_SSL=true` solo se il server MySQL è configurato per TLS con certificato verificabile.

## 2. Verificare e creare le tabelle

Il file `database.sql` definisce `users`, `operators` e `sales`. Prima di eseguirlo:

- confermare con il cliente che il database è dedicato a questo gestionale;
- verificare i permessi e fare un backup se contiene dati;
- eseguirlo una sola volta in MySQL Workbench sul database `chocozero_stand_sales_db`.

Usa `CREATE TABLE IF NOT EXISTS`, ma ciò non sostituisce una verifica della struttura preesistente.

## 3. Avvio locale

Da terminale nella cartella del progetto:

```bash
npm install
npm run dev
```

Aprire `http://localhost:3000`. Il backend userà il database configurato in `.env.local`.

## 4. Creare l'account amministratore

Dopo aver creato le tabelle e impostato le variabili ambiente:

```bash
npm run db:init-admin
```

Il comando chiede username e password (almeno 12 caratteri) e salva un hash bcrypt. Non esiste un PIN amministratore predefinito. L'operatore accede con un account con ruolo `operator`; la creazione degli account operatore richiede un flusso di gestione utenti da aggiungere se si desiderano credenziali separate per ogni cassiere. Nella versione iniziale l'admin crea gli operatori di vendita (nomi selezionabili in cassa), non gli account di login degli operatori.

Per creare un account di login operatore, usare un processo amministrativo sicuro o un comando di provisioning che inserisca in `users` un hash bcrypt e `role='operator'`; non inserire password in chiaro in SQL.

## 5. Endpoint implementati

- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/auth/me`
- `GET /api/operators`
- `POST /api/operators` (admin)
- `PATCH /api/operators` (admin)
- `GET /api/sales` (admin, vendite di oggi; `?all=1` per tutte le date)
- `POST /api/sales` (utente autenticato)
- `GET /api/stats` (admin)

## Limiti da completare prima della produzione

- Verificare compilazione e test end-to-end sul provider scelto.
- Confermare che il provider consenta e il database autorizzi connessioni remote dal backend; non aprire MySQL indiscriminatamente a tutto Internet.
- Predisporre un account MySQL con privilegi minimi, non un account amministrativo.
- Aggiungere provisioning/gestione account di login per gli operatori se ciascuno deve avere username/password personali.
- Decidere se l'amministratore deve poter consultare tutte le date o filtrare per giorno; la prima UI mostra statistiche di oggi e le vendite di oggi.
- Le cifre delle fasce sono stime convenzionali (2,50; 7,50; 17,50; 37,50; 62,50 euro), non incassi reali esatti. In particolare 62,50 per “Oltre 50€” è una stima convenzionale.
- Prima di una fiera reale, testare accessi, sessioni, connessione, vendite simultanee, esportazione e backup.
