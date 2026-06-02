# -.zshrc
 Elle te donne un petit workflow recon avec ports, services, urls et creds, en s’appuyant sur les sorties Nmap et sur des recherches de mots-clés utiles comme password, secret, token ou key

# ===== Nmap recon helpers =====

## ports() {
  local file="$1"
  local out="${2:-open_ports.txt}"

  if [[ -z "$file" ]]; then
    echo "usage: ports <nmap-output-file> [out-file]"
    return 1
  fi

  if [[ ! -f "$file" ]]; then
    echo "file not found: $file"
    return 2
  fi

  local ports csv count
  ports=$(
    awk '
    {
      if (match($0, /([0-9]+)\/(tcp|udp)[[:space:]]+open/, m)) print m[1]
      if (match($0, /Ports:[[:space:]]*(.*)/, p)) {
        line = p[1]
        gsub(/ /,"",line)
        n = split(line, parts, /,/)
        for (i=1;i<=n;i++) {
          if (match(parts[i], /([0-9]+)\/(tcp|udp)\/open/, q)) print q[1]
        }
      }
    }' "$file" | sort -n -u
  )

  if [[ -z "$ports" ]]; then
    echo "No open ports found in $file"
    return 3
  fi

  count=$(echo "$ports" | wc -l)
  csv=$(printf "%s\n" "$ports" | tr '\n' ',' | sed 's/,$//')

  echo "Found $count open ports"
  echo "$csv"
  printf "%s" "$csv" > "$out" && echo "Saved -> $out"
}

services() {
  local file="$1"
  local out="${2:-services.txt}"

  if [[ -z "$file" ]]; then
    echo "usage: services <nmap-output-file> [out-file]"
    return 1
  fi

  if [[ ! -f "$file" ]]; then
    echo "file not found: $file"
    return 2
  fi

  awk '
  {
    if (match($0, /([0-9]+)\/(tcp|udp)[[:space:]]+open[[:space:]]+([^[:space:]]+)/, m)) {
      ver = ""
      if (match($0, /open[[:space:]]+[^[:space:]]+[[:space:]]+(.*)$/, v)) ver = v[1]
      print m[1] "/" m[2] " " m[3] " " ver
    }
    if (match($0, /Ports:[[:space:]]*(.*)/, p)) {
      line = p[1]
      gsub(/ /,"",line)
      n = split(line, parts, /,/)
      for (i=1;i<=n;i++) {
        if (match(parts[i], /([0-9]+)\/(tcp|udp)\/open\/([^\/]+)\/([^\/]*)/, q)) {
          print q[1] "/" q[2] " " q[3] " " q[4]
        }
      }
    }
  }' "$file" | sort -u | tee "$out"
}

urls() {
  local file="$1"
  local out="${2:-urls.txt}"

  if [[ -z "$file" ]]; then
    echo "usage: urls <nmap-output-file> [out-file]"
    return 1
  fi

  if [[ ! -f "$file" ]]; then
    echo "file not found: $file"
    return 2
  fi

  awk '
  function print_url(proto, port, host) {
    if (proto == "https" || port == 443 || port == 8443) print "https://" host ":" port
    else print "http://" host ":" port
  }
  /^Nmap scan report for / {
    host = $NF
    gsub(/[()]/, "", host)
  }
  {
    if (match($0, /([0-9]+)\/(tcp|udp)[[:space:]]+open[[:space:]]+([^[:space:]]+)/, m)) {
      port = m[1]
      svc = tolower(m[3])
      if (svc ~ /http|https|ssl|www|web/ || port ~ /^(80|81|88|3000|5000|8000|8080|8181|8443)$/) {
        if (host != "") print_url(svc, port, host)
      }
    }
  }' "$file" | sort -u | tee "$out"
}

creds() {
  local file="$1"
  local out="${2:-creds_hits.txt}"

  if [[ -z "$file" ]]; then
    echo "usage: creds <file-or-folder> [out-file]"
    return 1
  fi

  if [[ ! -e "$file" ]]; then
    echo "file not found: $file"
    return 2
  fi

  if [[ -f "$file" ]]; then
    grep -REni --color=never \
      --exclude-dir=".git" \
      "(password|passwd|secret|token|apikey|api_key|bearer|credential|admin)" \
      "$file" | tee "$out"
  else
    grep -REni --color=never \
      --exclude-dir=".git" \
      "(password|passwd|secret|token|apikey|api_key|bearer|credential|admin)" \
      "$file" | tee "$out"
  fi
}

##

Ce que fait chaque fonction
ports extrait les ports ouverts et te sort un CSV prêt à réutiliser avec nmap -p.

services sort une vue lisible avec port/proto service version.

urls génère automatiquement les URLs web probables à partir des ports détectés.

creds cherche des indices de secrets ou de credentials dans un fichier ou un dossier.

Petite remarque utile
Nmap indique que le -sV sert à faire la détection de version, donc ta fonction services devient bien plus utile si tu pars d’un scan avec version detection. Et pour les recherches de creds, des motifs simples comme password, secret, token ou api_key sont une bonne base de départ.
