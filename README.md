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
alias: Antwort auf Ping mit Pong 2
triggers:
  - event_type: meshcore_message
    event_data:
      channel: "#pingpong"
    trigger: event
conditions:
  - condition: template
    value_template: "{{ trigger.event.data.message | lower | trim == 'ping' }}"
actions:
  - delay:
      hours: 0
      minutes: 0
      seconds: 0
      milliseconds: 200
  - data:
      channel_idx: 1
      message: >-
        {% set sender = trigger.event.data.sender or
        trigger.event.data.sender_name or trigger.event.data.src_node_name or
        trigger.event.data.from or trigger.event.data.from_name or 'unbekannt'
        %} {% set rx = (trigger.event.data.rx_log_data or []) %} {% if rx %} {%
        set ns = namespace(parts=[]) %} {% for entry in rx %} {% set hash_size =
        entry.path_hash_size | default(2) | int %} {% set chars = hash_size * 2
        %} {% set path_str = entry.path | string %} {% set ns2 =
        namespace(pairs=[]) %} {% for i in range(0, path_str | length, chars)
        %}{% set chunk = path_str[i:i+chars] %}{% if chunk | length == chars
        %}{% set ns2.pairs = ns2.pairs + [chunk] %}{% endif %}{% endfor %} {% if
        ns2.pairs %}{% set ns.parts = ns.parts + ['Hops ' + entry.path_len |
        string + '=' + ns2.pairs | join(',')] %}{% else %}{% set ns.parts =
        ns.parts + ['Hops ' + entry.path_len | string + '=direkt'] %}{% endif %}
        {% endfor %} @[{{ sender }}] -51588 -Pfade: {{ ns.parts | join('; ') }}
        {% else %} @[{{ sender }}] -51588 -Pfad: direkt oder unbekannt {% endif
        %}
    action: meshcore.send_channel_message
  - target:
      entity_id: input_text.meshcore_letzter_pfad
    data:
      value: >-
        {% set rx = (trigger.event.data.rx_log_data or []) %} {% if rx %} {% set
        ns = namespace(parts=[]) %} {% for entry in rx %} {% set hash_size =
        entry.path_hash_size | default(2) | int %} {% set chars = hash_size * 2
        %} {% set path_str = entry.path | string %} {% set ns2 =
        namespace(pairs=[]) %} {% for i in range(0, path_str | length, chars)
        %}{% set chunk = path_str[i:i+chars] %}{% if chunk | length == chars
        %}{% set ns2.pairs = ns2.pairs + [chunk] %}{% endif %}{% endfor %} {% if
        ns2.pairs %}{% set ns.parts = ns.parts + ['Hop ' + entry.path_len |
        string + '=' + ns2.pairs | join(',')] %}{% else %}{% set ns.parts =
        ns.parts + ['Hop ' + entry.path_len | string + '=direkt'] %}{% endif %}
        {% endfor %} {{ ns.parts | join('; ') }} {% else %}direkt{% endif %}
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

