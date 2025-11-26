# Thermostat-Custom-Button-Card-
Custom Button Card for Home Assistant/Thermostat
type: custom:button-card
entity: climate.hallway
show_icon: false
show_name: false
layout: horizontal
tap_action:
  action: none
hold_action:
  action: none
double_tap_action:
  action: none
styles:
  card:
    - padding: 18px
    - border-radius: 18px
    - height: 140px
    - color: white
    - border: 1px solid rgba(255,255,255,0.10)
    - box-shadow: 0 6px 18px rgba(0,0,0,0.35), inset 0 0 0 1px rgba(255,255,255,0.06)
    - background: |
        [[[
          const mode = (entity.state || 'off').toLowerCase();
          const action = (entity.attributes.hvac_action || '').toLowerCase();
          let key;
          if (mode === 'off') key = 'off';
          else if (action === 'heating') key = 'heating';
          else if (action === 'cooling') key = 'cooling';
          else if (action === 'drying') key = 'drying';
          else if (action === 'fan') key = 'fan';
          else if (action === 'idle' || mode === 'idle') key = 'idle';
          else key = 'idle';
          const g = {
            cooling: 'linear-gradient(135deg, #0e6fdc, #5aa8ff)',
            heating: 'linear-gradient(135deg, #ff5a3c, #ff9966)',
            drying:  'linear-gradient(135deg, #6a7dff, #8dc6ff)',
            fan:     'linear-gradient(135deg, #43cea2, #185a9d)',
            idle:    'linear-gradient(135deg, #b8c1cc, #8a93a5)',
            off:     'linear-gradient(135deg, #4b5563, #1f2937)'
          };
          return g[key] || g.idle;
        ]]]
    - backdrop-filter: blur(4px)
    - overflow: hidden
  grid:
    - grid-template-columns: 44px 1.3fr 1fr
    - grid-template-areas: "\"arrows left right\""
    - align-items: center
  custom_fields:
    arrows:
      - grid-area: arrows
      - align-self: center
      - justify-self: center
    left:
      - grid-area: left
      - align-self: center
    right:
      - grid-area: right
      - justify-self: end
      - text-align: right
custom_fields:
  arrows:
    card:
      type: vertical-stack
      cards:
        - type: custom:button-card
          entity: climate.hallway
          show_name: false
          show_icon: true
          icon: mdi:chevron-up
          tap_action:
            action: call-service
            service: climate.set_temperature
            service_data: |
              [[[
                const mode = (entity.state || '').toLowerCase();
                const r = (v)=> isNaN(v) ? null : Math.round(parseFloat(v));
                const min = r(entity.attributes.min_temp ?? 45);
                const max = r(entity.attributes.max_temp ?? 95);
                const clamp = (v)=> Math.max(min, Math.min(max, v));
                const t  = r(entity.attributes.temperature);
                const hi = r(entity.attributes.target_temp_high);
                const lo = r(entity.attributes.target_temp_low);
                const data = { entity_id: entity.entity_id };
                if (mode === 'heat')       data.temperature     = clamp((t ?? hi ?? 70) + 1);
                else if (mode === 'cool')  data.temperature     = clamp((t ?? lo ?? 72) + 1);
                else if (mode === 'heat_cool' || mode === 'auto') {
                  if (lo !== null) data.target_temp_low  = clamp(lo + 1);
                  if (hi !== null) data.target_temp_high = clamp(hi + 1);
                  if (lo === null && hi === null) data.temperature = clamp((t ?? 71) + 1);
                } else                      data.temperature     = clamp((t ?? 71) + 1);
                return data;
              ]]]
          styles:
            card:
              - width: 36px
              - height: 36px
              - border-radius: 10px
              - background: rgba(255,255,255,0.14)
              - border: 1px solid rgba(255,255,255,0.18)
              - box-shadow: inset 0 0 0 1px rgba(255,255,255,0.06)
              - padding: 0
            icon:
              - width: 22px
              - color: white
        - type: custom:button-card
          entity: climate.hallway
          show_name: false
          show_icon: true
          icon: mdi:chevron-down
          tap_action:
            action: call-service
            service: climate.set_temperature
            service_data: |
              [[[
                const mode = (entity.state || '').toLowerCase();
                const r = (v)=> isNaN(v) ? null : Math.round(parseFloat(v));
                const min = r(entity.attributes.min_temp ?? 45);
                const max = r(entity.attributes.max_temp ?? 95);
                const clamp = (v)=> Math.max(min, Math.min(max, v));
                const t  = r(entity.attributes.temperature);
                const hi = r(entity.attributes.target_temp_high);
                const lo = r(entity.attributes.target_temp_low);
                const data = { entity_id: entity.entity_id };
                if (mode === 'heat')       data.temperature     = clamp((t ?? hi ?? 70) - 1);
                else if (mode === 'cool')  data.temperature     = clamp((t ?? lo ?? 72) - 1);
                else if (mode === 'heat_cool' || mode === 'auto') {
                  if (lo !== null) data.target_temp_low  = clamp(lo - 1);
                  if (hi !== null) data.target_temp_high = clamp(hi - 1);
                  if (lo === null && hi === null) data.temperature = clamp((t ?? 71) - 1);
                } else                      data.temperature     = clamp((t ?? 71) - 1);
                return data;
              ]]]
          styles:
            card:
              - width: 36px
              - height: 36px
              - border-radius: 10px
              - background: rgba(255,255,255,0.14)
              - border: 1px solid rgba(255,255,255,0.18)
              - box-shadow: inset 0 0 0 1px rgba(255,255,255,0.06)
              - padding: 0
            icon:
              - width: 22px
              - color: white
  left: |
    [[[
      const r = (v) => (v === null || v === undefined || v === 'unknown' || v === 'unavailable')
        ? null : Math.round(parseFloat(v));

      const cur   = r(entity.attributes.current_temperature);
      const mode  = (entity.state || 'off').toLowerCase();
      const act   = (entity.attributes.hvac_action || '').toLowerCase();
      const hi    = r(entity.attributes.target_temp_high);
      const lo    = r(entity.attributes.target_temp_low);
      const single= r(entity.attributes.temperature);
      const set   = r(single ?? lo ?? hi);

      const cap = (s)=> s ? s.charAt(0).toUpperCase()+s.slice(1) : '';

      let word;
      if (['heating','cooling','idle','drying','fan'].includes(act)) {
        word = cap(act);
      } else if (mode === 'heat') {
        word = 'Heating';
      } else if (mode === 'cool') {
        word = 'Cooling';
      } else if (mode === 'idle') {
        word = 'Idle';
      } else if (mode === 'off') {
        word = 'Off';
      } else {
        word = cap(mode);
      }

      let target = null;
      if (word === 'Heating') target = hi ?? single;
      else if (word === 'Cooling') target = lo ?? single;

      const label = (word === 'Heating' || word === 'Cooling')
        ? `${word}${target!==null?` to ${target}°`:''}`
        : word;

      const setLine = (set !== null)
        ? `<div style="font-size:14px;opacity:.9;">Set <strong>${set}°</strong></div>`
        : '';

      const icon = (w)=> {
        if (w==='Heating') return `
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none">
            <path d="M12 2C9 6 16 7 12 12s-3 7 0 8c5 0 8-3 8-8 0-4-3-7-8-10Z"
              fill="currentColor" opacity=".95"/>
          </svg>`;
        if (w==='Cooling') return `
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none">
            <path d="M12 2v20M4.2 6.2l15.6 11.6M4.2 17.8L19.8 6.2" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>`;
        if (w==='Idle') return `
          <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor" opacity=".95">
            <path d="M12,2A10,10 0 1,0 22,12A10,10 0 0,0 12,2M10,9H12V15H10V9M14,9H16V15H14V9Z" />
          </svg>`;
        return `
          <svg width="18" height="18" viewBox="0 0 24 24" fill="none">
            <circle cx="12" cy="12" r="8" stroke="currentColor" stroke-width="2" opacity=".9"/>
          </svg>`;
      };

      return `
        <div style="display:flex;flex-direction:column;gap:8px;min-width:0;">
          <div style="font-size:56px;line-height:1;font-weight:700;white-space:nowrap;text-shadow:0 2px 6px rgba(0,0,0,.35)">${cur!==null?cur:'--'}°</div>
          ${setLine}
          <div style="display:flex;align-items:center;gap:8px;font-size:16px;font-weight:600;text-shadow:0 1px 4px rgba(0,0,0,.3)">
            <span style="display:inline-flex;align-items:center;justify-content:center;color:#fff">${icon(word)}</span>
            <span style="min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;">${label}</span>
          </div>
        </div>
      `;
    ]]]
  right:
    card:
      type: vertical-stack
      cards:
        - type: custom:button-card
          show_name: false
          show_icon: false
          styles:
            card:
              - background: none
              - box-shadow: none
              - border: none
              - padding: 0
            grid:
              - grid-template-areas: "\"panel\""
            custom_fields:
              panel:
                - grid-area: panel
                - padding: 0
          custom_fields:
            panel: |
              [[[
                const r = (v) => (v === null || v === undefined || v === 'unknown' || v === 'unavailable')
                  ? null : Math.round(parseFloat(v));

                const hallT = r(states['sensor.hallway_temperature']?.state);
                const hallH = r(states['sensor.hallway_humidity']?.state);
                const baseT = r(states['sensor.basement_temperature']?.state);

                const row = (svg, label, val, unit='') => `
                  <div style="display:flex;align-items:center;gap:6px;">
                    <span style="display:inline-flex;width:16px;height:16px;color:rgba(255,255,255,.95)">${svg}</span>
                    <span style="font-size:12px;opacity:.9">${label}</span>
                    <span style="margin-left:6px;font-weight:600">${val!==null?val:'--'}${val!==null?unit:''}</span>
                  </div>`;

                const thermo   = `<svg viewBox="0 0 24 24"><path d="M10 3a2 2 0 1 1 4 0v8.1a4 4 0 1 1-4 0V3z" fill="currentColor"/></svg>`;
                const droplet  = `<svg viewBox="0 0 24 24"><path d="M12 3s6 7 6 11a6 6 0 1 1-12 0c0-4 6-11 6-11Z" fill="currentColor"/></svg>`;
                const home     = `<svg viewBox="0 0 24 24"><path d="M3 12L12 4l9 8v8H3v-8Z" fill="currentColor"/></svg>`;

                return `
                  <div style="display:flex;flex-direction:column;gap:4px;padding:6px 8px;border-radius:12px;background:rgba(0,0,0,.18);box-shadow:inset 0 0 0 1px rgba(255,255,255,.06);">
                    ${row(thermo, 'Hallway', hallT, '°')}
                    ${row(droplet, 'Humidity', hallH, '%')}
                    ${row(home, 'Basement', baseT, '°')}
                  </div>
                `;
              ]]]
        - type: grid
          columns: 3
          square: false
          cards:
            - type: custom:button-card
              entity: climate.hallway
              show_name: false
              show_icon: true
              icon: mdi:fire
              tap_action:
                action: call-service
                service: climate.set_hvac_mode
                service_data:
                  entity_id: climate.hallway
                  hvac_mode: heat
              styles:
                card:
                  - width: 26px
                  - height: 24px
                  - margin: 4px auto 0 auto
                  - border-radius: 8px
                  - border: 1px solid rgba(255,255,255,.25)
                  - background: |
                      [[[
                        const is = (entity.state||'').toLowerCase()==='heat';
                        return is ? 'rgba(255,255,255,.25)' : 'rgba(0,0,0,.18)';
                      ]]]
                  - box-shadow: inset 0 0 0 1px rgba(255,255,255,.06)
                icon:
                  - width: 16px
                  - color: white
            - type: custom:button-card
              entity: climate.hallway
              show_name: false
              show_icon: true
              icon: mdi:snowflake
              tap_action:
                action: call-service
                service: climate.set_hvac_mode
                service_data:
                  entity_id: climate.hallway
                  hvac_mode: cool
              styles:
                card:
                  - width: 26px
                  - height: 24px
                  - margin: 4px auto 0 auto
                  - border-radius: 8px
                  - border: 1px solid rgba(255,255,255,.25)
                  - background: |
                      [[[
                        const is = (entity.state||'').toLowerCase()==='cool';
                        return is ? 'rgba(255,255,255,.25)' : 'rgba(0,0,0,.18)';
                      ]]]
                  - box-shadow: inset 0 0 0 1px rgba(255,255,255,.06)
                icon:
                  - width: 16px
                  - color: white
            - type: custom:button-card
              entity: climate.hallway
              show_name: false
              show_icon: true
              icon: mdi:power
              tap_action:
                action: call-service
                service: climate.set_hvac_mode
                service_data:
                  entity_id: climate.hallway
                  hvac_mode: "off"
              styles:
                card:
                  - width: 26px
                  - height: 24px
                  - margin: 4px auto 0 auto
                  - border-radius: 8px
                  - border: 1px solid rgba(255,255,255,.25)
                  - background: |
                      [[[
                        const is = (entity.state||'').toLowerCase()==='off';
                        return is ? 'rgba(255,255,255,.25)' : 'rgba(0,0,0,.18)';
                      ]]]
                  - box-shadow: inset 0 0 0 1px rgba(255,255,255,.06)
                icon:
                  - width: 16px
                  - color: white
extra_styles: |
  @media (max-width: 500px) {
    ha-card { height: auto !important; padding: 12px !important; }
  }
  @media (min-width: 900px) {
    ha-card { height: 150px !important; padding: 22px !important; }
  }
