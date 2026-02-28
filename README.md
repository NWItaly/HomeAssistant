*Versione temporanea a cui aggiungerò tutti i passaggi per utilizzare tutte le risorse che metterò disponibili*
# Utility per Home Assistant
## Blueprint

### Automations
- NofityBatteryLow
- TurnOffDevice

### Scripts
- [Sequential cover control](blueprints/scripts/README.md)

## Custom templates
Una volta creati e salvati i file è necessario riavviare HA.
- **escape_md**: Per fare l'escape dei caratteri speciali per markdownV2. Un utilizzo pratico è l'invio di testi su Telegram.
    ``` yaml
    {% from 'escape_md.jinja' import escape_md_v2 %}
    **Esempio**: {{ escape_md_v2("Prova-con caratteri speciali. Ed altre amenità") }}
    ```
    Risultato: `**Esempio:** Prova\\-con caratteri speciali\\.`
