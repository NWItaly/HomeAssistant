# Controllo Sequenziale Tapparelle

Blueprint per Home Assistant che apre o chiude più tapparelle/persiane una alla volta, aspettando che ognuna completi il movimento prima di avviare la successiva.

## Perché?

Avviare più motori contemporaneamente può stressare l'impianto elettrico. Questo blueprint risolve il problema controllando le tapparelle in sequenza, riducendo il picco di assorbimento.

## Come funziona

1. Calcola la direzione (se impostata su Auto)
2. Invia il comando alla prima tapparella
3. Aspetta che raggiunga la posizione target
4. Attende il delay configurato (se > 0)
5. Passa alla tapparella successiva
6. Ripete fino a completare tutte

Se una tapparella non risponde entro il timeout, lo script continua comunque per non bloccarsi.
Se una tapparella non è raggiungibile (stato `unavailable`/`unknown` o `current_position` non disponibile), viene saltata.

## Requisiti

- Home Assistant 2024.8.0 o superiore
- Tapparelle con attributo `current_position` (deve riportare posizione 0-100%)
- La maggior parte dei dispositivi Zigbee/Z-Wave lo supporta. Testa prima con i tuoi dispositivi.

## Installazione

### Via HACS (consigliato)
1. Aggiungi questo repository come repository personalizzato in HACS
2. Scarica il blueprint
3. Vai in Impostazioni → Automazioni e Scene → Blueprint
4. Clicca "Importa Blueprint" e seleziona il file scaricato

### Manuale

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/NWItaly/HomeAssistant/blob/main/blueprints/script/sequential_cover_control.yaml)

Oppure copia il file YAML in `config/blueprints/script/` e riavvia Home Assistant.

## Configurazione

| Parametro | Default | Descrizione |
|---|---|---|
| **Covers** | — | Tapparelle da controllare, nell'ordine in cui si muoveranno |
| **Direction** | Auto | `Auto` decide in base alla posizione media, `Open` apre, `Close` chiude |
| **Max open position** | 100% | Posizione massima di apertura. Usa `set_cover_position` invece di `open_cover` se diverso da 100 |
| **Max close position** | 0% | Posizione massima di chiusura. Usa `set_cover_position` invece di `close_cover` se diverso da 0 |
| **Delay between covers** | 0s | Pausa tra una tapparella e la successiva. Utile per distribuire ulteriormente il carico |
| **Use motor_run_status sensor** | false | Se abilitato, attende il sensore `sensor.NOME_TAPPARELLA_motor_run_status` per rilevare inizio e fine movimento. Richiede che il dispositivo esponga esattamente questo sensore. In caso contrario fa fallback su stato e `current_position` |
| **Enable debug logging** | false | Scrive messaggi di avanzamento nel Logbook e nel Log di sistema per ogni tapparella e ogni fase |
| **Timeout per cover** | 120s | Tempo massimo di attesa per ogni tapparella prima di passare alla successiva |

### Direzione automatica (Auto)

Quando la direzione è `Auto`, lo script calcola la media delle posizioni di tutte le tapparelle raggiungibili:
- media < 50% → apre
- media ≥ 50% → chiude

### Posizioni parziali

Impostando **Max open position** o **Max close position** a valori non predefiniti, lo script usa `cover.set_cover_position` al posto dei comandi nativi `open_cover` / `close_cover`. Questo permette ad esempio una chiusura parziale al 30% senza raggiungere la chiusura completa.

> **Nota:** quando si usa `set_cover_position`, lo script applica automaticamente un delay aggiuntivo di 1 secondo dopo il comando per dare tempo alla tapparella di registrare il movimento prima di iniziare l'attesa.

## Utilizzo

Lo script non richiede parametri runtime: tutto è configurato al momento della creazione dal blueprint. Puoi creare script separati per scenari diversi (es. "Chiudi tutto", "Apri parziale soggiorno").

### Esempio pulsante dashboard
```yaml
type: button
name: Chiudi Tutte
icon: mdi:window-shutter
tap_action:
  action: call-service
  service: script.NOME_TUO_SCRIPT
```

### Esempio automazione
```yaml
action:
  - action: script.NOME_TUO_SCRIPT
```

## Limitazioni

- Funziona solo con tapparelle che riportano la posizione in tempo reale
- Se hai relè semplici on/off senza feedback di posizione, usa un delay fisso invece del `wait_template`
- Le tapparelle non raggiungibili al momento dell'esecuzione (stato `unavailable`/`unknown` o senza `current_position`) vengono saltate senza interrompere lo script