# 🎯 GALLERY SHOOTER

Jogo de tiro ao alvo (shooting gallery) em **arquivo único** — Phaser 3 + Web Audio. Sem build, sem dependências locais. Abre o `index.html` no navegador e joga.

## ▶️ Como rodar

```bash
# qualquer servidor estático serve; ou só abra o arquivo direto
python3 -m http.server 8000
# acesse http://localhost:8000
```

Requer conexão com internet apenas para carregar o Phaser 3 via CDN:
`https://cdn.jsdelivr.net/npm/phaser@3/dist/phaser.min.js`

## 🎮 Controles

| Ação | Tecla / Mouse |
|------|---------------|
| Mirar | Mouse (move) |
| Atirar | Botão esquerdo |
| Recarregar | `R` |
| Pausar / retomar | `ESC` ou `P` |

## 🕹️ Objetivo

Sobreviva a **10 rodadas** (`MAX_ROUNDS`). Cada rodada cresce em dificuldade: mais inimigos, spawn mais rápido, mais inimigos simultâneos. Alcance a maior pontuação possível — combo multiplica os pontos.

### Inimigos (`ENEMY_DEFS`)

| Tipo | HP | Visibilidade | Dano | Pontos | Ícone |
|------|----|--------------|------|--------|-------|
| Soldier | 1 | 3000–4000ms | 10 | 100 | S |
| Sniper | 1 | 1500–2000ms | 25 | 250 | ! |
| Tank | 2 | 4000ms | 15 | 400 | T |
| Suicide | 1 | 2000ms | 40 | 300 | >> |
| Ghost | 1 | 4500–6000ms | 20 | 500 | ~ |
| **Innocent** | 1 | 3000–4000ms | 0 | 0 | CIV |

Atirar em civil (`innocent`) zera o combo e penaliza. Cada rodada inclui pelo menos 1 civil; a quantidade cresce com a rodada.

### Itens (`ITEM_DEFS`) — coletados ao clicar, somem em 5000ms

| Item | Efeito | Ícone |
|------|--------|-------|
| med | Cura HP | 💊 |
| ammo | Pente cheio (30) | 🔫 |
| shield | Bloqueia 2 acertos | 🛡️ |
| slow | Câmera lenta 5s | ⏱️ |
| nade | Limpa todos inimigos na tela | 💥 |

### Combos

Aciertos consecutivos sem errar aumentam o multiplicador: ×1 → ×2 (3 hits) → ×3 (5 hits) → ×4 (8 hits). Erar (miss) ou levar dano reseta.

## 🧱 Estrutura do código

Arquivo único `index.html`. Tudo inline. Cenas (Phaser Scenes):

```
BootScene     → gera texturas proceduralmente (sem assets externos), inicia
MenuScene     → menu inicial
GameScene     → loop principal: spawn, tiro, dano, rodadas, itens
UIScene       → HUD (HP, munição, score, combo, rodada, escudo)
PauseScene    → overlay de pausa
GameOverScene → tela final / vitória
```

### Constantes de balanceamento (topo do script)

```js
const BASE_W = 1280, BASE_H = 720;   // resolução base (escalada ao viewport)
const MAX_ROUNDS = 10;               // rodadas para vitória
const CLIP_SIZE = 30;                // tiros por pente
const RELOAD_MS = 1000;              // tempo de recarga
```

### Áudio

Engine sintético com **Web Audio API** (sem arquivos de som): `Audio.shot()`, `hit()`, `kill()`, `hurt()`, `innocent()`, `item()`, `reload()`, além de drone ambiente e efeito de câmera lenta. Inicia no primeiro clique do menu (política de autoplay do navegador).

## 🛠️ Tech

- **Phaser 3** (CDN)
- **Web Audio API** — SFX sintéticos
- **Canvas** — sem sprites externos; texturas geradas em código
- Sem build step, sem npm, sem backend. PT-BR.