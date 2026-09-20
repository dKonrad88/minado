# Minado 11 — campo minado em 11 fases

Jogo **single-file** (`index.html`) da suíte pessoal. PWA instalável, **100% offline**
(sem Supabase, sem login) — o progresso fica no `localStorage` do aparelho.

- **Live:** https://dkonrad88.github.io/minado/ (GitHub Pages, publica no push da `main`)
- **Repo:** `dKonrad88/minado` — trabalhar sempre no `index.html`
- **Na Biblioteca:** entrada no array `APPS` de `dKonrad88/apps` (`index.html`)
- **Idioma:** PT-BR. Confirmar decisões de produto com o Diego, mas **commit + push é autônomo**.
- É jogo **só pra diversão**, como o caça-níquel e o River Raid — não é pra vender.

## O jogo

11 fases em dificuldade crescente (5×5 com 3 minas → 11×14 com 38), cada uma vale até
3 ⭐ pelo tempo. Desbloqueio sequencial, 33 ⭐ no total.

Regras que fogem do campo minado clássico:

- **A primeira jogada nunca explode:** as minas só são sorteadas depois do 1º toque,
  excluindo a casa tocada e as 8 vizinhas (`gerar(seguro)`), então sempre abre uma clareira.
- **Dicas** (1 a 3 por fase) revelam uma casa segura; usar dica limita a 2 ⭐.
- **Três estados de marca** (`cel[i].fl`): `0` nada · `1` 🚩 certeza · `2` 🟡 dúvida.
  Só a `1` desconta do contador, protege do toque e conta no chord. A `2` é anotação.
- **Chord:** tocar num número que já tem todas as bandeiras abre a vizinhança.
- **Derrota em 3 tempos:** mina pisada acende → minas surgem em ondas → pausa pra olhar
  → estouro geral (clarão + tremor). Um toque pula. No fim, "👀 Ver o campo" remonta
  o tabuleiro inteiro com todos os números.

## Estrutura

- `index.html` — o jogo inteiro (HTML/CSS/JS, sem build, sem CDN).
- `manifest.json` / `sw.js` — PWA. O SW é **network-first** na navegação (pega a versão
  nova quando online, abre offline pelo cache). Subir `CACHE='minado-vN'` força limpeza.
- `.nojekyll` — obrigatório na suíte (o build Jekyll já travou publicação por dias no `viagem`).
- Ícones gerados com PIL (grade 3×3 + bandeira vermelha).

## Ajustes dentro do jogo

Rodapé do mapa → **⚙️ Ajustes**: som/vibração, **🔄 Atualizar o jogo** e zerar progresso.
O Atualizar desregistra o service worker, apaga os caches e recarrega com `?v=<timestamp>` —
é o caminho pro Diego pegar no celular o que foi mudado aqui sem esperar cache.
Subir `VERSAO` no topo do `<script>` a cada mudança publicada.

## Como testar (workflow da suíte)

Não precisa de login → dá pra testar direto no browser.

1. Editar `index.html`.
2. Copiar pra `<scratchpad>/minado/` e servir com um `server.py` mínimo (o TCC bloqueia
   ler `~/Documents` pelo preview). Abrir no Browser pane, `resize_window` mobile.
3. **Teste de lógica via `javascript_tool`** (o que pegou bugs de verdade): rodar N partidas
   por fase conferindo nº de minas, contagem dos vizinhos, 1ª jogada segura e detecção de
   vitória; e o ciclo das marcas. Ver o console — foi assim que apareceu um `$('#btSom')`
   morto que derrubava o registro do service worker.
4. **commit + push na `main`** e **conferir o site publicado com `curl`/`gh api`**, não só o push.
