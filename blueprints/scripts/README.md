# Controllo Sequenziale Tapparelle

Blueprint per Home Assistant che apre o chiude più tapparelle/persiane una alla volta, aspettando che ognuna completi il movimento prima di avviare la successiva.

## Perché?

Avviare più motori contemporaneamente può stressare l'impianto elettrico. Questo blueprint risolve il problema controllando le tapparelle in sequenza, riducendo il picco di assorbimento.

## Come funziona

1. Invia il comando apri/chiudi alla prima tapparella
2. Aspetta che raggiunga la posizione finale (0% chiusa o 100% aperta)
3. Passa alla tapparella successiva
4. Ripete fino a completare tutte

Se una tapparella non risponde entro il timeout, lo script continua comunque per non bloccarsi.

## Requisiti

- Home Assistant
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

1. Crea un nuovo script dal blueprint
2. Seleziona le tapparelle nell'ordine in cui vuoi che si muovano
3. Imposta il timeout (default: 120 secondi per tapparella)
4. Salva lo script

## Utilizzo

Richiama lo script con il parametro `direction`:
```yaml
service: script.NOME_TUO_SCRIPT
data:
  direction: close  # oppure "open"
```

### Esempio pulsante dashboard
```yaml
type: button
name: Chiudi Tutte
icon: mdi:window-shutter
tap_action:
  action: call-service
  service: script.NOME_TUO_SCRIPT
  data:
    direction: close
```

## Limitazioni

- Funziona solo con tapparelle che riportano la posizione in tempo reale
- Se hai relè semplici on/off senza feedback di posizione, usa un delay fisso invece del `wait_template`