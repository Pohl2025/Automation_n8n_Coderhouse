# Checkpoint 4 — Sincronización del Cerebro Agéntico con Ecosistemas de Negocio

Flujo de n8n que conecta un agente de IA con la casilla de soporte (Gmail) y un CRM,
automatizando la lectura de correos entrantes, la gestión de contactos y la redacción
de respuestas con aprobación humana.

## Arquitectura del flujo

Gmail Trigger → IF anti auto-reply → AI Agent → Set de limpieza →
Búsqueda en CRM → IF ¿existe? → (Update / Create) → Create Draft (Gmail)

## Nodos que cumplen la rúbrica

1. **Filtro_AutoReply (IF):** descarta correos automáticos (asuntos con "Auto-reply",
   "Out of office", "Undeliverable" o remitentes "no-reply@"). La salida positiva no se
   conecta a nada, cortando el bucle infinito de auto-respuestas.
2. **Buscar_Contacto (Look up):** busca el contacto por email antes de crearlo,
   previniendo el Error 409 (duplicados). Si existe, se actualiza; si no, se crea.
3. **Crear_Borrador (Create Draft):** la respuesta de la IA se guarda como borrador en
   Gmail, NO se envía. Barrera Human-in-the-loop: un humano revisa y aprueba antes del envío.
4. **Limpiar_Payload (Set):** normaliza el payload y extrae el email del remitente en
   formato limpio, evitando el Error 400 por datos mal formados.

## Nota sobre el CRM

Por restricciones de acceso a HubSpot, el CRM se simuló con **Airtable**, un conector
nativo de n8n con autenticación por token y permisos acotados a contactos (read/write),
respetando el principio de mínimo privilegio. La lógica Look up → Create/Update es
idéntica a la que se aplicaría sobre un CRM comercial.

## Mínimo privilegio (scopes)

- **Gmail:** lectura de correos y creación de borradores. Sin permiso de envío, lo que
  refuerza por diseño el guardrail Human-in-the-loop.
- **Airtable:** acceso restringido a la tabla de contactos.
