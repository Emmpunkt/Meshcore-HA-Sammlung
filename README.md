# Ping-Pong

Diese Automation "lauscht" auf dem Kanal #pingpong und beantwortet ein "Ping"
mit deiner Postleitzahl und dem Namen und Pfad des Absenders  
Wenn mehrere Pfade erkannt werden, werden auch diese gesendet.
Die Idee hinter dieser Automatisierung:  
Über den Kanal #pingpong kann man einfach das Netzwerk und die Erreichbarkeit testen, ohne auf andere Leute angewiesen zu sein.  
Das macht natürlich nur Sinn, wenn diese Automatisierung an mehreren Stellen im Netz vorhanden ist.   
Man kann also auch Nachts um 3 testen ob man erreichbar ist. 😉

---

## ⚠️ Wichtiger Hinweis
> **Achtung:**   
> Bitte bedenke das du mit Automationen ein Netzwerk lahmlegen kannst.  
> Dies wird zur Blockierung deines Nodes und/oder Repeaters führen.  
> Automationen immer nur einsetzen wenn sie dem Netz nicht schaden!
---
## 💡 Voraussetzungen!
> Der Kanal #pingpong muss auf dem Node, der an Home Assistant angeschlossen ist, schon vorhanden sein.  
> Falls nicht muss du ihn vorher am Gerät einstellen.    
> Das Meshcore-HA Plugin muss natürlich installiert und funktiontüchtig sein.  [Meshcore-HA](https://github.com/meshcore-dev/meshcore-ha.git)

---
## 📂 Automation Ping-Pong

```yaml
alias: Antwort auf Ping mit Pong
triggers:
  - event_type: meshcore_message
    event_data:
      channel: "#pingpong"
    trigger: event
conditions:
  - condition: template
    value_template: "{{ trigger.event.data.message | lower == 'ping' }}"
actions:
  - delay:
      hours: 0
      minutes: 0
      seconds: 10
      milliseconds: 0
  - data:
      channel_idx: 1
      message: |
        {% set sender =
            trigger.event.data.sender
            or trigger.event.data.sender_name
            or trigger.event.data.src_node_name
            or trigger.event.data.from
            or trigger.event.data.from_name
            or 'unbekannt'
        %}
        {% set rx = (trigger.event.data.rx_log_data or []) %}
        @[{{ sender }}] -51588
        {% if rx %}-Pfade: {% for entry in rx -%}
          {% set path_pairs = entry.path | list | batch(2) | map('join') | list %}
          {% if path_pairs %}Hops {{ entry.path_len }}={{ path_pairs | join(',') }}{% else %}Hops {{ entry.path_len }}=direkt{% endif %}{% if not loop.last %}; {% endif %}
        {%- endfor %}{% else %}- Pfad: direkt{% endif %}
    action: meshcore.send_channel_message
  - target:
      entity_id: input_text.meshcore_letzter_pfad
    data:
      value: |
        {% set rx = (trigger.event.data.rx_log_data or []) %} {% if rx %}
          {% for entry in rx -%}
            {% set path_pairs = entry.path | list | batch(2) | map('join') | list %}
            {% if path_pairs %}Hop {{ entry.path_len }}={{ path_pairs | join(',') }}{% else %}Hop {{ entry.path_len }}=direkt{% endif %}{% if not loop.last %}; {% endif %}
          {%- endfor %}
        {% else %}
          direkt
        {% endif %}
    action: input_text.set_value

```
---
## ℹ️ Automation erstellen
> Erstelle in Home Assistant eine neue Automation und stelle über das 3-Punkte Menu die Darstellung auf "In YAML bearbeiten" um.
> Kopiere die oben angezeigte YAML in deine Automation.  
> Ersetze meine Postleitzahl mit deiner und ändere ggf. den Alias (Name der Automation).  
> Evtl. musst du den channel_idx anpassen.  
> Wenn er falsch ist, landet deine Antwort auf einem anderen Kanal😆

---

