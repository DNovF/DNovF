<!doctype html>
<html lang="id">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Mini Chess — Dimas</title>

  <!-- Chessboard.js + chess.js via CDN -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/css/chessboard.min.css"/>
  <style>
    body { font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; display:flex; gap:24px; align-items:flex-start; padding:24px; }
    #board { width: 420px; }
    .controls { max-width:420px; }
    .btn { display:inline-block; padding:8px 12px; border-radius:8px; cursor:pointer; border:1px solid #444; background:#222; color:#fff; margin:4px 0; }
    .small { padding:6px 10px; font-size:0.9rem }
    .status { margin:8px 0; font-weight:600 }
    .moves { font-family: monospace; white-space:pre-wrap; max-height:220px; overflow:auto; background:#f6f6f6; padding:8px; border-radius:6px; }
    @media(max-width:720px){ body{flex-direction:column; align-items:center} }
  </style>
</head>
<body>
  <div id="board"></div>

  <div class="controls">
    <h3>Mini Chess</h3>
    <div class="status" id="status">Status: siap</div>
    <div>
      <button id="newBtn" class="btn small">New Game</button>
      <button id="flipBtn" class="btn small">Flip Board</button>
      <button id="hintBtn" class="btn small">Hint (legal move)</button>
    </div>

    <div style="margin-top:10px;">
      <label><input type="checkbox" id="autoPlayAI" /> Main vs Random AI (AI bergerak setelah Anda)</label>
    </div>

    <div style="margin-top:12px;">
      <strong>Moves:</strong>
      <div id="moveList" class="moves"></div>
    </div>

    <p style="margin-top:12px; font-size:0.9rem">Catatan: lawan AI menggunakan strategi sederhana (pilih random move). Untuk engine lebih canggih bisa integrasi Stockfish via WASM/worker.</p>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/1.0.0/chess.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/js/chessboard.min.js"></script>

  <script>
    // setup
    const boardEl = document.getElementById('board');
    const statusEl = document.getElementById('status');
    const moveListEl = document.getElementById('moveList');
    const autoPlayCheckbox = document.getElementById('autoPlayAI');
    const cfg = {
      draggable: true,
      position: 'start',
      onDrop: onDrop,
      onSnapEnd: onSnapEnd,
      onDragStart: onDragStart
    };

    const game = new Chess();
    const board = Chessboard(boardEl, cfg);

    function updateStatus(){
      if (game.in_checkmate()) {
        statusEl.textContent = 'Status: Checkmate — ' + (game.turn() === 'w' ? 'Black' : 'White') + ' menang';
      } else if (game.in_draw()) {
        statusEl.textContent = 'Status: Draw';
      } else {
        statusEl.textContent = `Status: giliran ${game.turn() === 'w' ? 'Putih' : 'Hitam'} ${game.in_check() ? '(in check)' : ''}`;
      }
      moveListEl.textContent = game.history({verbose:false}).join(' ');
    }

    function onDragStart(source, piece, position, orientation) {
      if (game.game_over()) return false;
      // jika bukan giliran pemain (putih selalu pemain manusia di left-to-right)
      // kita izinkan semua karena ini human vs human; jika ingin block saat AI turn, add logic.
    }

    function onDrop(source, target) {
      const move = game.move({from: source, to: target, promotion: 'q'});
      if (move === null) return 'snapback';
      updateStatus();

      // jika auto-AI diaktifkan dan game belum selesai, jalankan AI
      if (autoPlayCheckbox.checked && !game.game_over()) {
        window.setTimeout(() => {
          makeRandomAIMove();
          board.position(game.fen());
          updateStatus();
        }, 300);
      }
    }

    function onSnapEnd() {
      board.position(game.fen());
    }

    function makeRandomAIMove(){
      const moves = game.moves();
      if (moves.length === 0) return;
      // pick random
      const mv = moves[Math.floor(Math.random()*moves.length)];
      game.move(mv);
    }

    document.getElementById('newBtn').addEventListener('click', ()=>{
      game.reset();
      board.start();
      updateStatus();
    });

    document.getElementById('flipBtn').addEventListener('click', ()=>{
      board.flip();
    });

    document.getElementById('hintBtn').addEventListener('click', ()=>{
      const moves = game.moves({square: null, verbose: true});
      // show a legal move example: pick a random legal move and highlight via status
      if (moves.length === 0) { statusEl.textContent = 'Status: tidak ada move legal'; return; }
      const mv = moves[Math.floor(Math.random()*moves.length)];
      statusEl.textContent = `Contoh legal move: ${mv.from} → ${mv.to}`;
    });

    // init
    updateStatus();
  </script>
</body>
</html>
