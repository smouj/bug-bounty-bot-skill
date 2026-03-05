name: Bug Bounty Hunter
version: 1.0.0
description: Sistema automatizado de descubrimiento de vulnerabilidades de seguridad y gestión de recompensas para programas de bug bounty
author: Security Ops Team
tags:
  - bug bounty
  - security
  - hunting
  - vulnerability
  - reconnaissance
dependencies:
  - python >= 3.9
  - nmap >= 7.0
  - nuclei >= 2.9.0
  - sqlmap >= 1.7
  - subfinder >= 2.6
  - httpx >= 1.4
  - waymore >= 2.0
  - dalfox >= 2.0
  - jsluice
  - grep.app
  - amass
  - massdns
---

## Propósito

La skill Bug Bounty Hunter automatiza el flujo de trabajo completo de descubrir, verificar y reportar vulnerabilidades de seguridad para obtener recompensas de bug bounty. Se integra con plataformas HackerOne, Bugcrowd y OpenBugBounty para maximizar la eficiencia y minimizar el esfuerzo manual.

Casos de uso real:
1. **Reconocimiento de objetivo**: Enumera subdominios, endpoints, archivos JS y servicios expuestos para un alcance dado
2. **Detección de vulnerabilidades**: Ejecuta escaneos automatizados para XSS, inyección SQL, SSRF, IDOR, configuraciones incorrectas y divulgación de información
3. **Generación de proof-of-concept**: Crea cadenas de explotación reproducibles con capturas de solicitudes HTTP y capturas de pantalla
4. **Integración con plataforma**: Envía hallazgos directamente a plataformas de bug bounty con reports formateados correctamente
5. **Detección de duplicados**: Verifica informes existentes antes del envío usando APIs de plataformas y bases de datos públicas
6. **Seguimiento de recompensas**: Monitorea estado de pago, comunica con equipos de seguridad y rastrea ganancias entre programas
7. **Caza continua**: Programa escaneos periódicos para nuevos activos y vulnerabilidades emergentes
8. **Validación de alcance**: Asegura que todas las pruebas permanezcan dentro de límites autorizados usando validación `--check-scope`

## Alcance

Comandos:
- `enumerate <target>`: Descubre subdominios, endpoints, archivos JS y URLs históricas
- `scan <targets_file>`: Ejecuta escaneos de vulnerabilidades automatizados con herramientas y plantillas configurables
- `test <target> <vuln_type>`: Verificación manual con payloads personalizados y generación de PoC
- `report <findings_file>`: Genera reporte listo para envío específico de plataforma
- `claim <report_file> [--platform <name>]`: Envía reporte vía API e inicia reclamación de recompensa
- `track [--report-id <id>]`: Monitorea estado, monto de recompensa y pago de reportes
- `verify <report-id>`: Re-prueba vulnerabilidades corregidas para cerrar el ciclo
- `config [key] [value]`: Gestiona tokens API, límites de tasa y configuraciones de herramientas
- `scope validate <target>`: Verifica si el objetivo está dentro del alcance autorizado
- `duplicates check <vulnerability>`: Busca en bases de datos públicas informes existentes

## Proceso de Trabajo Detallado

### 1. Enumeración de Objetivo & Validación de Alcance
```bash
$ bug-bounty-hunter enumerate --target example.com --output assets.json --include-subdomains --js
[+] Iniciando enumeración para example.com
[+] Ejecutando subfinder... 156 subdominios descubiertos
[+] Sondeando con httpx... 89 hosts activos
[+] Extrayendo endpoints JS... 34 archivos encontrados
[+] Recolectando URLs históricas con waymore... 2,450 endpoints
[+] Activos guardados en assets.json
```
- Usa `subfinder` para descubrimiento pasivo de subdominios
- `httpx` sondea servicios activos y tecnologías
- `waymore` recolecta URLs históricas de Wayback Machine y Common Crawl
- `jsfinder` extrae endpoints y secretos de archivos JavaScript
- `amass` para enumeración DNS más profunda (configurable)

### 2. Escaneo de Vulnerabilidades
```bash
$ bug-bounty-hunter scan --targets assets.json --tools nuclei,sqlmap,dalfox --severity high,critical --rate-limit 5
[+] Cargando plantillas nuclei (2,400+ plantillas)
[+] Probando 89 objetivos...
[+] Nuclei: 12 XSS potenciales, 3 SSRF, 1 SQLi encontrados
[+] SQLmap: Confirmado 1 SQLi booleano en /product.php?id=
[+] Dalfox: 5 XSS reflejados en parámetros de búsqueda
[+] Hallazgos guardados en findings_20250115.json
```
- `nuclei` con selección de plantillas personalizada (`-templates cves/,xss/,ssrf/`)
- `sqlmap` con `--batch` para explotación automatizada
- `dalfox` para XSS con `--found-callback` para confirmación de payload
- `massdns` para comprobaciones de toma de subdominio
- Límite de tasa configurable por herramienta para evitar interrupción del servicio

### 3. Verificación Manual & Generación de PoC
```bash
$ bug-bounty-hunter test --target "https://example.com/search?q=" --vulnerability xss --payload '<svg onload=alert(document.domain)>' --screenshot
[+] Probando XSS en parámetro de búsqueda
[+] Payload entregado: <svg onload=alert(document.domain)>
[+] Alerta activada con dominio: example.com
[+] Captura de pantalla guardada: xss_proof_20250115_143022.png
[+] Solicitud HTTP registrada: xss_request_20250115_143022.har
[+] Confirmado: XSS reflejado en endpoint de búsqueda
```
- Usa navegador headless `dalfox` para XSS basado en DOM
- `grep.app` para buscar fugas de datos sensibles en código
- `jsluice` para extracción de claves API de archivos JS
- Captura HAR automática para reproducción de red

### 4. Detección de Duplicados
```bash
$ bug-bounty-hunter duplicates check --vulnerability "XSS in search parameter" --target example.com
[+] Buscando reportes públicos de HackerOne...
[+] No se encontraron duplicados en HackerOne
[+] Buscando divulgaciones de Bugcrowd...
[+] 1 reporte similar encontrado en Bugcrowd (endpoint diferente)
[+] Verificable como único: Sí
```
- Consulta API de plataforma con autenticación con alcance
- Busca bases de datos de divulgación pública (H1, Bugcrowd, OpenBugBounty)
- Compara endpoint, tipo de vulnerabilidad e impacto

### 5. Generación de Reporte
```bash
$ bug-bounty-hunter report --findings findings_20250115.json --platform hackerone --program example-com
[+] Generando reporte con formato HackerOne...
[+] Evaluación de impacto: Medio (XSS conduce a secuestro de sesión)
[+] Puntuación CVSS 3.1: 6.1
[+] Pasos para reproducir:
    1. Navegue a https://example.com/search
    2. Busque: <svg onload=alert(1)>
    3. Aparece cuadro de alerta con dominio
[+] Reporte guardado: h1_report_20250115.json
[+] Incluye: 3 capturas de pantalla, archivo HAR, comando curl
```
- Plantillas específicas de plataforma (H1, Bugcrowd, OBB)
- Puntuación CVSS automática basada en tipo de vulnerabilidad
- Incluye comando `curl` para reproducción
- Adjunta evidencia (capturas de pantalla, registros de red)

### 6. Reclamación de Recompensa
```bash
$ bug-bounty-hunter claim --report h1_report_20250115.json --platform hackerone --program "example-com"
[+] Autenticando con HackerOne...
[+] Enviando reporte a programa: example-com
[+] Reporte enviado exitosamente
[+] ID de reporte: https://hackerone.com/reports/1234567
[+] ID de seguimiento: BH-2025-001
[+] Recompensa estimada: $500-$1,500 (Severidad media)
[+] Notificación enviada a canal Slack #bug-bounties
```
- Usa API de plataforma con credenciales almacenadas
- Maneja autenticación de dos factores via tokens de sesión almacenados
- Configura webhooks para actualizaciones de estado

### 7. Seguimiento & Gestión
```bash
$ bug-bounty-hunter track --last-30-days
[+] Reportes recientes:
ID               | Objetivo         | Vulnerabilidad   | Estado    | Recompensa
-----------------|------------------|------------------|-----------|----------
BH-2025-001      | example.com      | XSS              | Triaged   | $750
BH-2025-002      | test.com         | SQLi             | Resuelto  | $1,200
BH-2025-003      | demo.org         | IDOR             | Duplicado| $0

Total ganado YTD: $12,450
```
- Muestra todos los reportes con estado y montos de recompensa
- Exporta a CSV para fines fiscales
- Envía resúmenes semanales por correo

### 8. Verificación de Corrección
```bash
$ bug-bounty-hunter verify --report-id BH-2025-001
[+] Re-probando vulnerabilidad del reporte BH-2025-001
[+] Objetivo: https://example.com/search
[+] Payload: <svg onload=alert(1)>
[+] Estado: Corregido (no se activó alerta)
[+] Vulnerabilidad ya no es reproducible
[+] Actualizando estado de reporte a "Resuelto"
[+] Notificando equipo de HackerOne
```
- Vuelve a ejecutar PoC original contra objetivo
- Verifica implementación de parche
- Actualiza estado de plataforma automáticamente

## Reglas de Oro

1. **Autorización Primero**: Solo prueba objetivos listados en programas de bug bounty autorizados. Usa `bug-bounty-hunter scope validate <target>` antes de escanear.
2. **Límite de Tasa**: Nunca excedas 10 solicitudes/segundo a menos que esté explícitamente permitido. Predeterminado: `--rate-limit 5`. Usa `--throttle 2000` para retrasos de 2s.
3. **Sin Exfiltración de Datos**: No descargues, modifiques o elimines datos. Solo proof-of-concept. Usa modo `--read-only` cuando esté disponible.
4. **Cumplimiento de Alcance de Plataforma**: Cada plataforma tiene reglas. Ejecuta `bug-bounty-hunter config get platform-rules` antes de escanear.
5. **Sin DDoS/Agotamiento de Recursos**: Evita inundar endpoints con payloads. Limita `--concurrent 5` por defecto.
6. **Solo Evidencia**: Captura capturas de pantalla, solicitudes HTTP, comandos curl. NO ejecutes payloads destructivos (rm, drop table, etc.).
7. **Divulgación Responsable**: Nunca publiques antes del parche del proveedor. Usa canales de coordinación de plataforma.
8. **Reporte Preciso**: Reporta duplicados honestamente. No reclames recompensas por problemas conocidos.
9. **Privacidad de Datos**: Elimina PII recolectada después del envío del reporte. Usa flag `--purge-data` después de reclamar.
10. **Cumplimiento Legal**: Algunos países restringen investigación de seguridad. Verifica leyes locales antes de proceder.

**Recuerda**: Los programas de bug bounty son asociaciones con equipos de seguridad, no pruebas de penetración adversarial. Construye confianza, no interrupción.

## Ejemplos

**Ejemplo 1: Reconocimiento en nuevo objetivo**
```bash
$ bug-bounty-hunter enumerate --target "*.uber.com" --output uber_assets.json --threads 20 --exclude-wildcard
[+] Alcance: *.uber.com (autorizado vía HackerOne)
[+] Subfinder: 1,247 subdominios
[+] Hosts activos: 734
[+] Archivos JS: 156 que contienen claves API
[+] Exportado: uber_assets.json (54 MB)
```

**Ejemplo 2: Escaneo de inyección SQL dirigido**
```bash
$ bug-bounty-hunter scan --targets uber_assets.json --tool sqlmap --level 5 --risk 3 --dbs --threads 3 --output sqlmap_results/
[+] Probando 734 hosts activos
[+] Descubrimiento de parámetros completo
[+] Enumeración de bases de datos en 3 objetivos:
    - https://api.uber.com/coupons (MySQL)
    - https://riders.uber.com/profile (PostgreSQL)
    - https://partners.uber.com/earnings (SQL Server)
[+] SQLi confirmado: api.uber.com/coupons?id=1
[+] Extraído: 4 bases de datos, 12 tablas, 450+ filas
[+] Guardado: sqlmap_results/api.uber.com/
```

**Ejemplo 3: Verificación XSS con PoC**
```bash
$ bug-bounty-hunter test --target "https://partners.uber.com/earnings?date=" --vulnerability xss --payload "<img src=x onerror=alert(document.domain)>" --screenshot --har
[+] Prueba XSS en partners.uber.com/earnings
[+] Payload ejecutado: alert(document.domain) = partners.uber.com
[+] Cookies de sesión accesibles vía document.cookie
[+] Impacto: Secuestro de sesión posible
[+] Evidencia: screenshot_20250115_162233.png, request_20250115_162233.har
[+] Reproducible: Sí (Chrome 120, Firefox 121)
[+] CVSS:6.1 (Medio)
```

**Ejemplo 4: Enviar a Bugcrowd**
```bash
$ bug-bounty-hunter claim --report xss_report_uber.json --platform bugcrowd --program "uber"
[+] Autenticación Bugcrowd: OK
[+] Programa: Uber (ID: 5678)
[+] Envío válido: Sí (dentro de alcance)
[+] Reporte enviado: https://bugcrowd.com/uber/reports/9876543
[+] Recompensa estimada: $750-$2,000
[+] Notificación Slack enviada a #uber-bounties
```

**Ejemplo 5: Rastrear ganancias**
```bash
$ bug-bounty-hunter track --platform hackerone --resolved --csv
Report ID,Target,Vulnerabilidad,Estado,Recompensa,Pagado
BH-2024-045,api.uber.com,SQLi,Resuelto,$1,500,$1,500
BH-2024-089,riders.uber.com,IDOR,Duplicado,$0,$0
BH-2025-012,partners.uber.com,XSS,Resuelto,$750,$750
BH-2025-023,m.uber.com,CSRF,Triaged,$500,$0
Total: $2,250 pagados, $500 pendientes
```

## Comandos de Rollback

- `bug-bounty-hunter scan stop --all`: Abortar todos los escaneos en ejecución inmediatamente
- `bug-bounty-hunter purge --findings <date>`: Eliminar hallazgos anteriores a fecha especificada (cumplimiento GDPR)
- `bug-bounty-hunter claim withdraw <report-id>`: Retirar envío antes de aceptación (solo H1)
- `bug-bounty-hunter config reset --tokens`: Limpiar todos los tokens API y credenciales almacenados
- `bug-bounty-hunter track delete <report-id>`: Eliminar reporte de base de datos de seguimiento (usar con precaución)
- `bug-bounty-hunter enumerate cleanup <target>`: Eliminar todos los artefactos de enumeración para objetivo dado
- `bug-bounty-hunter test revert <test-id>`: Deshacer cambios de estado de pruebas manuales (si aplica)
- `bug-bounty-hunter logs clear --before <timestamp>`: Limpiar logs anteriores a timestamp para privacidad

## Dependencias

Herramientas requeridas (auto-detectadas en PATH):
```
nmap         - Escaneo de red y detección de servicios
nuclei       - Escáner de vulnerabilidades con plantillas YAML
sqlmap       - Inyección SQL automatizada y toma de bases de datos
subfinder    - Enumeración de subdominios
httpx        - Sondeo HTTP y detección de servicios
waymore      - Recolección de URLs históricas de Wayback & Common Crawl
dalfox       - Escaneo XSS y análisis de parámetros
amass        - Enumeración DNS avanzada (opcional pero recomendado)
massdns      - Fuerza bruta DNS y resolución
jsluice      - Análisis de JavaScript para extracción de endpoints
grep.app     - Búsqueda de datos sensibles en repos públicos (scraper de tokens API)
```

Instalación:
```bash
# Instalar herramientas Go
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@v2.9.0
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@v2.6.0
go install -v github.com/projectdiscovery/httpx/cmd/httpx@v1.4.0
go install -v github.com/owasp-amass/amass/v3/...@master

# Instalar herramientas Python
pip3 install sqlmap dalfox jsluice waymore

# Instalar skill bug-bounty-hunter
git clone https://github.com/bug-bounty-bot/skill ~/.bug-bounty-hunter
echo 'export PATH="$HOME/.bug-bounty-hunter/bin:$PATH"' >> ~/.bashrc
```

Configuración:
```bash
# Inicializar configuración
bug-bounty-hunter config init

# Configurar tokens de plataforma (leer de env o establecer manualmente)
bug-bounty-hunter config set hackerone.token $HACKERONE_TOKEN
bug-bounty-hunter config set bugcrowd.token $BUGCROWD_TOKEN
bug-bounty-hunter config set openbugbounty.token $OPENBUGBOUNTY_TOKEN

# Configurar ajustes específicos de herramientas
bug-bounty-hunter config set nuclei.templates ~/nuclei-templates/
bug-bounty-hunter config set sqlmap.threads 3
bug-bounty-hunter config set rate-limit 5
bug-bounty-hunter config set max-concurrent-scans 2

# Agregar objetivos autorizados (validación de alcance)
bug-bounty-hunter scope add --target "*.uber.com" --platform hackerone --program "uber"
```

## Verificación

Verificar instalación:
```bash
$ bug-bounty-hunter --version
Bug Bounty Hunter 1.0.0

$ bug-bounty-hunter tools check
nmap:          v7.94 (✓)
nuclei:        v2.9.0 (✓)
sqlmap:        v1.7.4 (✓)
subfinder:     v2.6.0 (✓)
httpx:         v1.4.0 (✓)
waymore:       v2.0 (✓)
dalfox:        v2.0 (✓)
jsluice:       v0.1.6 (✓)
All dependencies installed successfully.
```

Verificar conectividad API:
```bash
$ bug-bounty-hunter platforms test
Testing Hackerone...  Authenticated (user: security_researcher)
Testing Bugcrowd...   Authenticated (user: @archomboldt)
Testing OpenBugBounty...   Not configured
All configured platforms reachable.
```

Probar en objetivo seguro:
```bash
# Ejecutar contra testphp.vulnweb.com (vulnerable intencionalmente)
$ bug-bounty-hunter enumerate --target testphp.vulnweb.com --output test_assets.json
$ bug-bounty-hunter scan --targets test_assets.json --tool nuclei --severity all
[+] Found XSS in /search.php?test= parameter
[+] Found SQLi in /listproducts.php?cat= parameter
[+] Found LFI in /include.php?file= parameter
# Esperado: 3+ vulnerabilidades en sitio de prueba
```

## Solución de Problemas

**Problema: "Command not found: bug-bounty-hunter"**
Solución: Asegurar que `~/.bug-bounty-hunter/bin` esté en PATH. Ejecutar `source ~/.bashrc` o logout/login.

**Problema: "nuclei: template not found"**
Solución: Actualizar plantillas: `nuclei -update-templates`. Establecer ruta personalizada: `bug-bounty-hunter config set nuclei.templates ~/custom-templates/`.

**Problema: "Rate limited by target"**
Solución: Reducir concurrencia: `bug-bounty-hunter config set rate-limit 2`. Agregar retrasos: `bug-bounty-hunter config set throttle 5000`. Verificar si IP bloqueada.

**Problema: "Authentication failed on platform API"**
Solución: Verificar permisos de token: debe tener `report:create` y `target:read`. Regenerar token en dashboard de plataforma. Asegurar token no expirado.

**Problema: "SQLmap hangs or times out"**
Solución: Reducir `--level` y `--risk`. Usar `--time-sec 10`. Agregar `--tamper=space2comment`. Verificar bloqueo por WAF.

**Problema: "False positives from nuclei"**
Solución: Usar plantillas específicas: `-templates cves/,xss/`. Agregar `-exclude-templates Dos/,PathTraversal/`. Verificar manualmente antes de envío.

**Problema: "Duplicate report rejected"**
Solución: Verificar reglas de alcance de plataforma; algunas prohíben ciertas herramientas. Buscar bases de datos públicas exhaustivamente antes de envío. Algunos programas tienen alcance privado—usar UI de plataforma para verificar.

**Problema: "Cannot claim bounty (program not found)"**
Solución: Verificar ID/programa spelling. Algunos programas requieren invitación. Verificar si programa abierto a todos los investigadores o solo por invitación.

**Problema: "No subdomains discovered"**
Solución: Intentar `amass` en lugar de `subfinder`. Agregar `--passive-only false` para fuerza bruta DNS. Verificar spelling de objetivo y alcance.

**Problema: "JavaScript files not parsed"**
Solución: Instalar `jsluice`: `pip3 install jsluice`. Verificar que archivos sean JavaScript (no CSS/HTML minificados). Usar flag `--extract-secrets`.

**Problema: "Memory errors during scan"**
Solución: Reducir `--threads`. Dividir lista de objetivos en lotes. Usar archivo swap: `sudo swapon -a`. Monitorear con `top`.

## Seguridad & Ética

- Siempre obtén autorización explícita antes de escanear
- Nunca pruebes objetivos fuera de alcance o programas no participantes
- No explotes vulnerabilidades más allá de proof-of-concept (sin exfiltración, modificación o eliminación de datos)
- Reporta hallazgos responsablemente solo a través de canales oficiales
- Respeta el anonimato del investigador si la plataforma lo permite
- Coordina con equipos de seguridad de vendor vía plataforma
- Mantén hallazgos confidenciales hasta resolución
- Cumple con reglas de programa de bug bounty y requisitos legales

**Recuerda**: Los programas de bug bounty son asociaciones con equipos de seguridad, no pruebas de penetración adversarial. Construye confianza, no interrupción.