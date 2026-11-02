# Akap Player

Player de música local, editor de tags MP3 e sincronizador de letras — tudo no navegador, sem servidor, sem upload, funcionando offline (após o primeiro carregamento).

---

## Versão

**v2.2.0** — unificação dos 3 aplicativos (Player + Editor de Tags + Sincronizador de Letras)

| Componente | Versão interna | Observação |
|---|---|---|
| `index.html` (Player) | 2.2.0 | Contém o parser ID3, filtros, prefetch de tags, navegação por iframes |
| `tools-tags.html` (Editor de tags) | 2.0.0 | Sem áudio; usa `browser-id3-writer.mjs` local |
| `tools-lyrics.html` (Sincronizador) | 2.0.0 | Áudio próprio; ±20s; espelha o player via `AKAP_SEEK` |
| `browser-id3-writer.mjs` | 6.4.0 | Lib de escrita de tags (bundled por jsDelivr) |

> A versão do produto é a do `index.html`. As ferramentas têm versões internas independentes para facilitar manutenção.

---

## Estrutura de arquivos

akap-player/
├── index.html                 ← player principal (UI + lógica + parser ID3)
├── tools-tags.html            ← editor de metadados MP3
├── tools-lyrics.html          ← sincronizador de letras
├── browser-id3-writer.mjs     ← escrita de tags (import local)
└── README.md                  ← este arquivo


Todos os arquivos devem ficar **na mesma pasta**.

---

## Como executar

### Opção 1 — Servidor local (recomendado)

O `import('./browser-id3-writer.mjs')` exige HTTP/HTTPS (não funciona em `file://`).

```bash
# Python 3
cd akap-player
python -m http.server 8000

# Node.js
npx serve akap-player

# PHP
php -S localhost:8000 -t akap-player

Abra http://localhost:8000/.

Opção 2 — Android (WebView)

O app já roda em WebView com servidor local (127.0.0.1). Basta garantir que:

· O AndroidBridge esteja exposto com os métodos pickFolder, listAudioFiles, getStreamPort, getPersistedFolder, tryRestoreFolder.
· Os 4 arquivos estejam nos assets do app, servidos por um servidor local (ou WebViewAssetLoader).

Opção 3 — Hospedagem estática

Qualquer servidor estático (GitHub Pages, Netlify, Vercel, Cloudflare Pages) funciona. Não há backend.

---

Funcionalidades

Player (index.html)

· Biblioteca local — escolha uma pasta (showDirectoryPicker) ou vários arquivos.
· Persistência — a última pasta, faixa, posição e volume são lembrados (localStorage + IndexedDB).
· Leitura de tags ID3 — título, artista, álbum, gênero, capa e letras (USLT/SYLT), com suporte a v2.2, v2.3 e v2.4.
· Pré-carregamento de tags em background — os filtros de gênero/álbum/artista ficam preenchidos sem precisar clicar em cada faixa.
· Filtros combinados — gênero + álbum + artista + busca textual.
· Fila respeitando filtros — next/prev pulam apenas as faixas visíveis.
· Controles — play/pause, anterior/próxima, ±10s, volume, embaralhar, repetir.
· Atalhos — Espaço, ←/→ (±5s), ,/. (±10s), N/P (faixa), M/R (modos).
· Visualizador — barras reativas ao áudio (Web Audio API).
· Media Session — controles na tela de bloqueio / notificação do Android.
· Letras sincronizadas — exibe a linha atual no painel de letras do player.
· Navegação por iframes persistentes — os editores mantêm o estado ao trocar de aba.

Editor de tags (tools-tags.html)

· Sem áudio — apenas metadados; o player continua tocando normalmente.
· Leitura local — parser ID3 próprio (sem CDN, sem servidor).
· Escrita local — browser-id3-writer via import('./browser-id3-writer.mjs').
· Campos — título, artista, álbum, ano, gênero, número da faixa, artista do álbum, comentário, letra, letra LRC (SYLT) e capa (APIC).
· 100% offline — após o primeiro carregamento, nenhuma requisição externa.
· Download — gera um novo MP3 (*-editado.mp3), sem alterar o original.
· Feedback ao player — envia AKAP_SAVED com o arquivo editado; o player recarrega mantendo a posição.

Sincronizador de letras (tools-lyrics.html)

· Áudio próprio — assume o áudio do player ao abrir (o player pausa).
· ±20s / ±5s — quatro botões de pulo rápido.
· Marcação por Enter — marca o tempo atual e avança para a próxima linha.
· Desfazer — Backspace reverte a última marcação.
· Edição manual de tempo — campo mm:ss.cc em cada linha.
· Salvamento automático — localStorage preserva a letra e os tempos.
· Exportação — TXT, LRC e cópia para a área de transferência.
· Busca web — botão que pesquisa o nome da faixa no Google (usa o nome real da faixa, mesmo quando recebida do player).
· Sincronização com o player — ao voltar, o player retoma no ponto que o editor deixou.

---

Comunicação entre o player e os editores

O player e os editores se comunicam por postMessage. Contrato:

De → Para Tipo Payload Quando
Editor → Player AKAP_READY — Ao carregar o iframe
Editor → Player AKAP_TIME { time, playing } Em cada timeupdate
Editor → Player AKAP_PLAYER_CONTROL { action: 'pause'\|'play'\|'seek', time? } Controle remoto (opcional)
Editor → Player AKAP_SAVED { file } Após editar e baixar o MP3
Player → Editor AKAP_LOAD_TRACK { file, time } Ao abrir ou trocar de faixa
Player → Editor AKAP_SEEK { time } Ao sincronizar o tempo
Player → Editor AKAP_GET_TIME — Antes de sair do editor de letras

A origem é identificada por event.source (comparação com frameTags.contentWindow / frameLyrics.contentWindow), o que permite dois iframes simultâneos sem ambiguidade.

---

Armazenamento

Chave Onde O que guarda
orbita-player-state-v1 localStorage Faixa atual, posição, volume, shuffle, repeat, nome da pasta
directoryHandle IndexedDB (orbita-player-db) Handle da pasta escolhida (Chromium)
marcaletra_state_v1 localStorage (iframe) Letra e tempos do sincronizador

Nada é enviado para servidores. Tudo fica no dispositivo.

---

Compatibilidade

Recurso Navegadores
showDirectoryPicker (pasta) Chrome/Edge 86+, Opera 72+
Fallback <input webkitdirectory> Chrome, Edge, Firefox, Safari
MediaSession Chrome/Edge Android, Safari iOS
import() de .mjs Todos os navegadores modernos
Web Audio API Todos os navegadores modernos

Firefox não suporta showDirectoryPicker, mas o fallback via <input webkitdirectory> funciona.

Limitações conhecidas

· Tags no Android — o parser não consegue ler o File diretamente; as tags só aparecem após o primeiro fetch do stream (o prefetch em background pula os arquivos Android e deixa o cache para quando o usuário abrir o editor).
· Arquivos muito grandes — o parser lê até 64 MB de tag ID3. Capas gigantes acima disso são ignoradas.
· Gênero composto — TCON com múltiplos valores (Pop;Rock) é normalizado para o primeiro valor.
· file:// — não funciona para o editor de tags (o import() do .mjs é bloqueado). Use HTTP local.

---

Histórico de versões

v2.2.0 — correção dos filtros e prefetch

· Parser ID3 reescrito com suporte a flags por frame e >>> 0 (evita len negativo em v2.3).
· label() não usa mais número como artista ("01 - Música" → artista desconhecido, título correto).
· metaOf() normaliza gênero ((17)Rock → Rock).
· Prefetch de tags em background — 4 workers, atualização incremental dos filtros.
· applyTags só reconstrói filtros após o readID3 resolver.

v2.1.0 — navegação estável

· Dois iframes persistentes (frameTags, frameLyrics) em vez de um recriado.
· Detecção de origem por event.source.
· Correção do travamento ao alternar Player → Letras → Tags → Player.

v2.0.0 — unificação

· Separação em 3 arquivos (index, tools-tags, tools-lyrics) + browser-id3-writer.mjs local.
· Remoção do CDN esm.sh.
· Contrato postMessage documentado.
· Player com ±10s; sincronizador com ±20s.

v1.x — app único

· Player + editores embutidos em base64 no mesmo HTML.
· Dependência de CDN para escrita de tags.
· Navegação recriava o iframe a cada troca de aba (bug corrigido na v2.1).

---

Solução de problemas

Sintoma Causa provável Correção
"Não foi possível obter o nome da música" na busca Faixa sem nome (raro) Recarregue a faixa no player
Filtros vazios MP3s sem tags ID3 Use o editor de tags para adicioná-las
Editor de tags não baixa file:// Sirva via HTTP (python -m http.server)
Navegação trava Cache antigo Hard refresh (Ctrl+Shift+R)
Áudio do player e do editor tocam juntos Estado inconsistente Volte ao Player, pause, reabra o editor
Download de MP3 falha import() do .mjs falhou Verifique se browser-id3-writer.mjs está na mesma pasta

Licença

Projeto pessoal. Uso livre para fins não comerciais. As bibliotecas de terceiros mantêm suas licenças originais (browser-id3-writer — MIT).


---

## Verificação de versão

Fiz a conta com base no que foi entregue:

| Arquivo | Versão declarada no código | O que contém |
|---|---|---|
| `index.html` | `v2.2 · Akap Player` (constante `APP_VERSION`) | Player completo, parser ID3 corrigido, prefetch, 2 iframes |
| `tools-tags.html` | `2.0.0` (implícito no título) | Editor sem áudio, import local |
| `tools-lyrics.html` | `2.0.0` (implícito no título) | Editor com áudio, ±20s, busca corrigida |
| `browser-id3-writer.mjs` | `6.4.0` (comentário no topo) | Lib de escrita |

**Versão do produto: `v2.2.0`** (major 2, minor 2, patch 0).

### Mapa mental das versões

v1.0 ─── app único (base64, CDN)
│
├─ v1.6 ── filtros adicionados, versão original do usuário
│
v2.0 ─── unificação em 3 arquivos + .mjs local
│
├─ v2.1 ── correção da navegação (2 iframes)
│
└─ v2.2 ── correção dos filtros + prefetch + parser ID3 robusto  ← você está aqui

