# Changelog — Akap Player

Todas as mudanças notáveis deste projeto são documentadas aqui.
Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/).
Versionamento baseado em [SemVer](https://semver.org/lang/pt-BR/).

---

## [2.2.0] — 2025-01-XX

### Adicionado
- **Prefetch de tags em background** — fila com 4 workers que lê ID3 de todas as faixas logo após carregar a biblioteca. Filtros de gênero/álbum/artista ficam preenchidos automaticamente, sem precisar clicar em cada faixa.
- **Normalização de gênero** em `metaOf()` — `(17)Rock` vira `Rock`, `Pop;Rock` vira `Pop`.
- **Tratamento de flags por frame** no parser ID3 (v2.3/v2.4): compressão, encriptação, agrupamento e unsync.
- **Suporte a `TDRC`** (ID3v2.4) como fallback de ano, além de `TYER` (v2.3).
- **Limite de tag aumentado** de 32 MB para 64 MB.

### Corrigido
- **Filtro de gênero vazio** — `TCON` não era lido por causa de `len` negativo em v2.3. Cálculo agora usa `>>> 0`.
- **Filtro de álbum vazio** — `TALB` pelo mesmo motivo.
- **Filtro de artista puxando nome da música** — `label()` usava o primeiro bloco do filename mesmo quando era numérico (`"01 - Música.mp3"` → artista = `"01"`). Agora descarta blocos puramente numéricos.
- **`rebuildFilterOptions()` chamado antes do `readID3` resolver** — os filtros eram reconstruídos com cache vazio. Agora só reconstrói após o parse.
- **`decodeTagText()` não parava no primeiro `\0`** em UTF-16 — podia incluir bytes de lixo após o terminador.

### Alterado
- `applyTags()` agora chama `rebuildFilterOptions()` + `buildQueue()` somente depois de o `readID3` resolver.
- `setTracks()` e `loadAndroidTracks()` disparam `prefetchAllTags()` após montar a fila.

---

## [2.1.0] — 2025-01-XX

### Adicionado
- **Dois iframes persistentes** (`frameTags` e `frameLyrics`) em vez de um recriado a cada troca.
- **Detecção de origem** das mensagens por `event.source` (comparação com `contentWindow`).
- **Botões ±10s** no player (e atalhos `,` e `.`).
- **Botões ±20s / ±5s** no sincronizador de letras.
- **Correção da busca web** no sincronizador — usa `currentFileName` preenchido tanto pelo input quanto por `AKAP_LOAD_TRACK`.

### Corrigido
- **Navegação travava** ao alternar Player → Letras → Tags → Player. Causa: um único iframe era destruído/reutilizado e as flags `iframeReady` ficavam stale. Agora cada editor tem seu próprio iframe e estado.
- **Estouro lateral** no editor de letras em celulares estreitos. Causa: `.wrap` com `94vw` + falta de `overflow-x:hidden` + `mark-panel` fixo sem `left/right`. Corrigido com `calc(100% - 20px)` e `left/right: 8px`.
- **Busca web** dizia "Não foi possível obter o nome da música" quando a faixa vinha do player (o `<input type="file">` ficava vazio). Agora `currentFileName` é preenchido nos dois caminhos.

### Alterado
- `openPage()` agora alterna `display` entre os dois iframes em vez de reescrever `srcdoc`.
- `sendTrackToEditor()` recebe a página como argumento e escolhe o iframe correto.

---

## [2.0.0] — 2025-01-XX

### Adicionado
- **Separação em 3 arquivos** (`index.html`, `tools-tags.html`, `tools-lyrics.html`) + `browser-id3-writer.mjs` local.
- **Contrato `postMessage` documentado** (ver README).
- **`README.md`** com instruções de execução, estrutura e compatibilidade.
- **`CHANGELOG.md`** (este arquivo).

### Removido
- **Dependência de CDN `esm.sh`** — `browser-id3-writer` agora é importado localmente.
- **`base64` inline** dos editores no `index.html` — cada ferramenta é um arquivo HTML próprio.
- **Service Worker via `Blob URL`** — era código morto (blob URLs não podem registrar SW).

### Alterado
- `tools-tags.html` — sem áudio; apenas metadados.
- `tools-lyrics.html` — áudio próprio; espelha o player via `AKAP_SEEK`.
- `index.html` — parser ID3 próprio no player (título/artista/álbum/capa/letras).
- `index.html` — `loadedmetadata` usa `{ once: true }` para não vazar listeners.

### Corrigido
- **`pagehide` revogava `objectURL` mesmo em BFCache** — o áudio parava de tocar ao voltar. Agora só revoga se `!e.persisted`.
- **Tecla `Space` conflitava com botões focados** — o handler global não incluía `button` na checagem de editável.
- **`prevTrack` ignorava shuffle** — agora respeita o modo.
- **`rebuildFilterOptions` era chamado em `applyTags`** — podia resetar filtros silenciosamente. Movido para `setTracks` e `prefetch`.

---

## [1.6.0] — 2024-XX-XX

### Adicionado
- **Filtros por gênero, álbum e artista** na biblioteca.
- **Fila de reprodução respeitando filtros** — `next`/`prev` pulam apenas as faixas visíveis.
- **`buildQueue()`** e `playQueue`.
- **Parser ID3 no player** para título/artista/álbum/capa/letras.
- **Painel de letras sincronizadas** no player (`toggleLyricsBtn`).
- **Visualizador** com Canvas + Web Audio API.
- **Media Session** para controles na tela de bloqueio.
- **Persistência** de pasta (IndexedDB) e estado (localStorage).

### Corrigido
- **`label()` retornava `"01"` como artista** em arquivos `"01 - Música.mp3"`.
- **`buildQueue` incluía faixa atual** mesmo quando ela não batia com os filtros (unshift manual).

---

## [1.0.0] — 2024-XX-XX

### Adicionado
- **Player completo** com play/pause, seek, volume, shuffle, repeat.
- **Editor de tags MP3** (base64 inline) com `jsmediatags` + `browser-id3-writer` via CDN.
- **Sincronizador de letras** (base64 inline) com marcação por `Enter`, exportação TXT/LRC.
- **Suporte Android** via `AndroidBridge` (pickFolder, listAudioFiles, stream local).
- **Seleção de pasta** com `showDirectoryPicker` + fallback `<input webkitdirectory>`.
- **Restauração automática** da última pasta autorizada.