# DNS-MINTIC

Listas de dominios que los Proveedores de Servicios de Internet (ISP) de Colombia deben inhabilitar por DNS, según los fundamentos jurídicos y administrativos vigentes en el país.

Los ISP de Colombia trabajamos en conjunto para cumplir estas normativas de forma eficiente, transparente y coordinada. Este repositorio reúne las listas en un formato listo para cargar en resolvedores DNS.

## Listas

Todas están en la carpeta [`DNS_PUBLICO/`](DNS_PUBLICO/).

| Archivo | Entidad / origen | Motivo del bloqueo |
|---|---|---|
| [`Block_MSAI.txt`](DNS_PUBLICO/Block_MSAI.txt) | MinTIC | Material de abuso sexual infantil (Ley 679 de 2001) |
| [`Block_Coljuegos`](DNS_PUBLICO/Block_Coljuegos) | Coljuegos | Juegos de suerte y azar en línea no autorizados |
| [`Block_DNDA.txt`](DNS_PUBLICO/Block_DNDA.txt) | Dirección Nacional de Derecho de Autor | Infracción de derechos de autor |

## Formato

- Un dominio por línea, en minúsculas.
- Sin `http://`, `https://`, rutas, puertos ni espacios.
- Sin comodines (`*`); para bloquear un subdominio se escribe completo (`ejemplo.blogspot.com`).
- Sin líneas vacías ni duplicados.
- Codificación UTF-8 con fin de línea Unix (LF).

## Dominios que NUNCA deben estar en las listas

Las listas oficiales a veces traen dominios de infraestructura compartida. Bloquearlos por DNS no afecta solo al sitio ilegal: tumba servicios completos para todos los usuarios. Por ejemplo, `google.com` en la lista bloquea el buscador, Gmail, Drive y el inicio de sesión de Google.

Antes de publicar cualquier actualización, revisa que no aparezca ninguno de estos dominios **raíz** ni sus servidores de contenido compartido:

| Servicio | Dominios protegidos |
|---|---|
| Google | `google.com`, `google.com.co`, `gstatic.com`, `googleapis.com`, `googleusercontent.com`, `ggpht.com`, `gvt1.com`, `googlevideo.com` |
| YouTube | `youtube.com`, `ytimg.com`, `youtu.be` |
| Blogger | `blogspot.com`, `blogger.com`, `bp.blogspot.com` y sus subdominios numerados (`1.bp.blogspot.com`, etc.) |
| Microsoft | `microsoft.com`, `live.com`, `outlook.com`, `office.com`, `windows.net` |
| Meta | `facebook.com`, `fbcdn.net`, `instagram.com`, `whatsapp.com`, `whatsapp.net` |
| Apple | `apple.com`, `icloud.com`, `mzstatic.com` |
| CDN / nube | `cloudflare.com`, `amazonaws.com`, `cloudfront.net`, `akamaihd.net` |

Un **subdominio específico** de un usuario sí puede bloquearse sin daño colateral (por ejemplo, `nombredelblog.blogspot.com`), porque solo afecta a ese sitio.

## Validación antes de publicar

Ejecuta esto en la raíz del repositorio antes de hacer commit. Si imprime algo, hay dominios que deben revisarse o quitarse:

```bash
grep -nxiE '(www\.)?(google\.com(\.co)?|gstatic\.com|googleapis\.com|googleusercontent\.com|ggpht\.com|youtube\.com|ytimg\.com|blogspot\.com|blogger\.com|([0-9]+\.)?bp\.blogspot\.com|t[0-9]+\.gstatic\.com|microsoft\.com|live\.com|outlook\.com|facebook\.com|fbcdn\.net|whatsapp\.(com|net)|instagram\.com|apple\.com|icloud\.com|cloudflare\.com|amazonaws\.com|cloudfront\.net)' DNS_PUBLICO/*
```

Revisión de formato (líneas vacías, espacios, `http`, duplicados):

```bash
for f in DNS_PUBLICO/*; do
  echo "== $f"
  grep -nE '^\s*$|\s|https?://|/|\*' "$f"
  sort "$f" | uniq -d
done
```

## Uso en el resolvedor DNS

**Unbound**

```bash
awk '{print "local-zone: \""$1"\" always_nxdomain"}' DNS_PUBLICO/Block_MSAI.txt > /etc/unbound/block_msai.conf
```

**BIND (RPZ)**

```bash
awk '{print $1" CNAME ."; print "*."$1" CNAME ."}' DNS_PUBLICO/Block_MSAI.txt >> /etc/bind/db.rpz
```

**Pi-hole / AdGuard Home**: agrega la URL *raw* del archivo como lista de bloqueo:

```
https://raw.githubusercontent.com/pipetelecom/DNS-MINTIC/main/DNS_PUBLICO/Block_MSAI.txt
```

Después de actualizar, recarga el servicio y prueba que Google siga resolviendo:

```bash
dig +short google.com @127.0.0.1
```

## Cómo actualizar las listas

1. Descarga la lista oficial más reciente de la entidad correspondiente.
2. Normalízala al formato descrito arriba.
3. Ejecuta los comandos de **Validación antes de publicar**.
4. Quita los dominios de infraestructura compartida que aparezcan y deja constancia en el mensaje del commit.
5. Haz commit y push.

## Aviso

Este repositorio solo facilita el cumplimiento de órdenes de bloqueo emitidas por las autoridades colombianas. Cada ISP es responsable de verificar las listas contra las fuentes oficiales. Si encuentras un dominio legítimo bloqueado por error, abre un *issue* indicando el dominio y el archivo.
