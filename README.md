<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>幻想迷宮ローグライク Web完全版</title>
    <style>
        body {
            background: #121216;
            color: #f0f0f0;
            font-family: "Helvetica Neue", Arial, "Hiragino Kaku Gothic ProN", Meiryo, monospace;
            margin: 0;
            padding: 15px;
            display: flex;
            flex-direction: column;
            align-items: center;
            user-select: none;
        }
        h1 {
            margin: 0 0 10px 0;
            font-size: 24px;
            color: #ffcc00;
            text-shadow: 0 0 8px rgba(255, 204, 0, 0.4);
        }
        /* タイトル画面 */
        #titleScreen {
            background: #1e1e26;
            border: 2px solid #555;
            border-radius: 10px;
            padding: 30px 40px;
            text-align: center;
            max-width: 680px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.8);
        }
        .title-guide {
            text-align: left;
            background: #14141a;
            padding: 15px 20px;
            border-radius: 8px;
            border: 1px solid #333;
            margin: 20px 0;
            font-size: 14px;
            line-height: 1.7;
        }
        /* ゲーム画面全体 */
        #gameScreen {
            display: none;
            width: 900px;
            max-width: 98vw;
        }
        /* 上部ステータスバー */
        .status-bar {
            background: #1e1e26;
            border: 2px solid #444;
            border-radius: 8px;
            padding: 10px 15px;
            margin-bottom: 10px;
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            font-size: 15px;
            font-weight: bold;
        }
        .stat-item { margin-right: 12px; }
        /* メインの2カラム（左：マップ、右：操作＆図鑑パネル） */
        .main-container {
            display: flex;
            gap: 12px;
            align-items: stretch;
        }
        canvas {
            background: #09090c;
            border: 2px solid #555;
            border-radius: 8px;
            box-shadow: 0 0 15px rgba(0,0,0,0.8);
            flex-shrink: 0;
        }
        /* 右側の操作方法＆アイテム図鑑パネル */
        .side-panel {
            flex: 1;
            background: #1e1e26;
            border: 2px solid #444;
            border-radius: 8px;
            padding: 12px 15px;
            font-size: 13px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }
        .panel-section {
            margin-bottom: 8px;
            border-bottom: 1px solid #333;
            padding-bottom: 8px;
        }
        .panel-section:last-child {
            border-bottom: none;
            margin-bottom: 0;
            padding-bottom: 0;
        }
        .panel-title {
            font-weight: bold;
            color: #4da6ff;
            margin-bottom: 5px;
            font-size: 14px;
        }
        .key-badge {
            display: inline-block;
            background: #333;
            border: 1px solid #666;
            border-radius: 4px;
            padding: 1px 6px;
            font-family: monospace;
            font-weight: bold;
            color: #fff;
            margin-right: 4px;
        }
        .legend-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 4px;
            line-height: 1.5;
        }
        /* 下部メッセージログ */
        .log-box {
            background: #14141a;
            border: 2px solid #444;
            border-radius: 8px;
            padding: 10px 15px;
            margin-top: 10px;
            height: 68px;
            font-family: monospace;
            font-size: 14px;
            line-height: 1.5;
            overflow-y: hidden;
        }
        .log-line { color: #aaa; }
        .log-line.latest { color: #fff; font-weight: bold; }
        /* ボタン類 */
        button {
            background: #2b4c7e;
            color: #fff;
            border: 1px solid #4da6ff;
            border-radius: 6px;
            padding: 8px 16px;
            font-size: 15px;
            font-weight: bold;
            cursor: pointer;
            margin: 4px;
            transition: 0.15s;
        }
        button:hover { background: #3b69ad; }
        button.btn-large { padding: 12px 28px; font-size: 18px; }
        button.btn-sub { background: #333; border-color: #666; font-size: 13px; padding: 5px 10px; }
        button.btn-sub:hover { background: #444; }
        /* 敵HPバー */
        .hp-bar-bg {
            display: inline-block;
            width: 90px;
            height: 10px;
            background: #333;
            border: 1px solid #666;
            vertical-align: middle;
            margin: 0 6px;
        }
        .hp-bar-fill {
            height: 100%;
            background: #ff4d4d;
        }
        /* ショップモーダル */
        #shopModal {
            display: none;
            position: fixed;
            top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            background: #22222b;
            border: 3px solid #ffcc00;
            border-radius: 10px;
            padding: 20px 30px;
            box-shadow: 0 0 30px rgba(0,0,0,0.9);
            z-index: 100;
            text-align: center;
        }
    </style>
</head>
<body>

    <h1>🏰 幻想迷宮ローグライク 🏰</h1>

    <!-- ① タイトル画面（スタート＆ロード＆遊び方ガイド） -->
    <div id="titleScreen">
        <h2 style="margin-top:0; color:#ff6666;">深淵の10階層と邪竜ドラゴン</h2>
        <p>ランダムに変化する迷宮を探索し、B10Fに潜む邪竜（🐉）を討伐しよう！</p>

        <div class="title-guide">
            <strong>【🎮 基本の操作方法】</strong><br>
            ・移動と攻撃：<span class="key-badge">W</span><span class="key-badge">A</span><span class="key-badge">S</span><span class="key-badge">D</span>キー または <span class="key-badge">↑</span><span class="key-badge">←</span><span class="key-badge">↓</span><span class="key-badge">→</span>矢印キー（敵に重なると攻撃）<br>
            ・遠距離＆魔法：<span class="key-badge">F</span>弓矢 ／ <span class="key-badge">Q</span>炎魔法 ／ <span class="key-badge">C</span>氷魔法 ／ <span class="key-badge">X</span>回復魔法<br>
            ・アイテム使用：<span class="key-badge">P</span>ポーション（HP回復） ／ <span class="key-badge">E</span>パン（お腹回復） ／ <span class="key-badge">R</span>足踏み休息<br><br>
            <strong>【🎒 登場アイテム＆施設】</strong><br>
            ・🎁 <strong>宝箱</strong>：武器・盾・最大HP/MP増加・薬・パン・矢・金貨のどれかを入手<br>
            ・⛲ <strong>癒しの泉</strong>：HPとMPが全回復し、満腹度も20%回復<br>
            ・⛺ <strong>ショップ</strong>：集めた金貨（G）でアイテム購入や武器・防具の強化が可能
        </div>

        <button class="btn-large" onclick="startNewGame()">▶ ゲームスタート（はじめから）</button><br>
        <button class="btn-large" style="background:#3a6b4c; border-color:#66cc88;" onclick="loadGame(true)">📂 続きから始める（ロード）</button>
    </div>

    <!-- ② ゲーム本編画面 -->
    <div id="gameScreen">
        <!-- 上部ステータス -->
        <div class="status-bar">
            <span class="stat-item">🪜 階層: B<span id="stFloor">1</span>F/10F</span>
            <span class="stat-item">⭐ Lv:<span id="stLv">1</span> (EXP:<span id="stExp">0</span>/30)</span>
            <span class="stat-item" style="color:#ff8080;">❤️ HP:<span id="stHp">90</span>/<span id="stMaxHp">90</span></span>
            <span class="stat-item" style="color:#80bfff;">🔮 MP:<span id="stMp">40</span>/<span id="stMaxMp">40</span></span>
            <span class="stat-item" style="color:#ffcc66;">🍖 満腹:<span id="stFood">100</span>%</span>
            <span class="stat-item">⚔️ 攻:<span id="stAtk">18</span> 🛡️ 防:<span id="stDef">5</span></span>
            <span class="stat-item" style="color:#ffe066;">💰 資金:<span id="stGold">30</span>G</span>
        </div>

        <!-- 中央：左にマップ、右に操作方法・アイテム説明パネル -->
        <div class="main-container">
            <canvas id="gameCanvas" width="510" height="352"></canvas>

            <div class="side-panel">
                <!-- 操作方法 -->
                <div class="panel-section">
                    <div class="panel-title">🎮 操作方法（キー または ボタンクリック）</div>
                    <div><span class="key-badge">W/A/S/D</span>または<span class="key-badge">矢印キー</span>：移動・体当たり攻撃</div>
                    <div style="margin-top:4px;">
                        <button class="btn-sub" onclick="handleCommand('p')">[P] 🧪薬 (<span id="btnPot">2</span>)</button>
                        <button class="btn-sub" onclick="handleCommand('e')">[E] 🥖パン (<span id="btnBread">2</span>)</button>
                        <button class="btn-sub" onclick="handleCommand('r')">[R] 💤休む</button>
                    </div>
                </div>

                <!-- 遠距離＆魔法スキル -->
                <div class="panel-section">
                    <div class="panel-title">🔥 弓矢・魔法スキル（画面上のボタンでも発動可）</div>
                    <div>
                        <button class="btn-sub" onclick="startAim('f')">[F] 🏹弓矢 (残:<span id="btnArr">8</span>)</button>
                        <button class="btn-sub" onclick="startAim('q')">[Q] 💥ファイア (MP12)</button>
                    </div>
                    <div style="margin-top:4px;">
                        <button class="btn-sub" onclick="handleCommand('c')">[C] ❄️ブリザード周囲凍結 (MP20)</button>
                        <button class="btn-sub" onclick="handleCommand('x')">[X] ✨ヒール回復 (MP15)</button>
                    </div>
                </div>

                <!-- 周囲の敵HPゲージ -->
                <div class="panel-section">
                    <div class="panel-title">👾 周囲の敵・ボス HPゲージ</div>
                    <div id="enemyHpList">周囲に敵はいません</div>
                </div>

                <!-- アイテム＆マップ記号図鑑 -->
                <div class="panel-section">
                    <div class="panel-title">📖 アイテム・マップ記号（アイコン）図鑑</div>
                    <div class="legend-grid">
                        <div>🧙 <b>あなた</b> (勇者)</div>
                        <div>👾 <b>モンスター</b> (通常の敵)</div>
                        <div>👹 <b>ゴーレム</b> (B5F中ボス)</div>
                        <div>🐉 <b>邪竜</b> (B10Fラスボス)</div>
                        <div>🎁 <b>宝箱</b> (装備・薬・パン・金貨)</div>
                        <div>⛲ <b>癒しの泉</b> (HP/MP全回復)</div>
                        <div>⛺ <b>ショップ</b> (買い物・鍛冶)</div>
                        <div>💥 <b>罠</b> (ダメージ・ワープ)</div>
                        <div>🚪 <b>階段</b> (次の階層へ進む)</div>
                        <div>🧱 <b>壁</b> (通れないブロック)</div>
                    </div>
                </div>

                <!-- セーブ・ロード・タイトルボタン -->
                <div class="panel-section" style="text-align:right;">
                    <button class="btn-sub" style="background:#2e5c3e; border-color:#5cd685;" onclick="saveGame()">💾 セーブ</button>
                    <button class="btn-sub" onclick="loadGame(false)">📂 ロード</button>
                    <button class="btn-sub" onclick="returnToTitle()">🏠 タイトルへ</button>
                </div>
            </div>
        </div>

        <!-- 下部メッセージログ（3行） -->
        <div class="log-box">
            <div class="log-line" id="log0"></div>
            <div class="log-line" id="log1"></div>
            <div class="log-line latest" id="log2">▶ 冒険が始まった！ W/A/S/Dキー または 矢印キーで移動しよう！</div>
        </div>
    </div>

    <!-- ③ ショップ（買い物）ウィンドウ -->
    <div id="shopModal">
        <h3 style="margin-top:0; color:#ffcc00;">⛺ 迷宮のよろず屋（所持金: <span id="shopGold">0</span> G）</h3>
        <p>店主「いらっしゃい！ 好きなものを買って装備を鍛えていきな！」</p>
        <div style="display:flex; flex-direction:column; gap:6px; margin:15px 0;">
            <button onclick="buyItem(1)">[1] 🥖 焼きたてパン（満腹+60%） - 25 G</button>
            <button onclick="buyItem(2)">[2] 🧪 ポーション（HP+70回復） - 35 G</button>
            <button onclick="buyItem(3)">[3] 🏹 木の矢×5本（遠距離攻撃） - 25 G</button>
            <button onclick="buyItem(4)">[4] 🔨 武器の鍛錬（攻撃力+4） - 60 G</button>
            <button onclick="buyItem(5)">[5] 🛡️ 防具の鍛錬（防御力+3） - 60 G</button>
        </div>
        <button style="background:#555; border-color:#888;" onclick="closeShop()">[0] 店を出る</button>
    </div>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const W = 17, H = 11, TS = 30;

    let p = {};
    let floor = 1;
    let map = [];
    let traps = [];
    let enemies = [];
    let logs = ["", "", "🏰 幻想迷宮へようこそ！"];
    let aimMode = null; // null, 'f'(矢), 'q'(炎魔法)
    let inShop = false;
    let fx = null; // エフェクト描画用 {x, y, icon}

    function addLog(msg) {
        logs.shift();
        logs.push(msg);
        document.getElementById("log0").innerText = "▸ " + logs[0];
        document.getElementById("log1").innerText = "▸ " + logs[1];
        document.getElementById("log2").innerText = "▶ " + logs[2];
    }

    function getEnemyAt(x, y) {
        return enemies.findIndex(e => e.alive && e.x === x && e.y === y);
    }

    function findEmptySpot() {
        while (true) {
            let x = Math.floor(Math.random() * (W - 2)) + 1;
            let y = Math.floor(Math.random() * (H - 2)) + 1;
            if (map[y][x] === '.' && traps[y][x] === 0 && (x !== p.x || y !== p.y) && getEnemyAt(x, y) === -1) {
                return { x, y };
            }
        }
    }

    // ランダムダンジョン生成（穴掘り法）
    function generateDungeon() {
        map = Array.from({ length: H }, () => Array(W).fill('#'));
        traps = Array.from({ length: H }, () => Array(W).fill(0));
        enemies = [];

        let cx = Math.floor(W / 2), cy = Math.floor(H / 2);
        map[cy][cx] = '.';
        p.x = cx; p.y = cy;

        let floors = 1;
        let target = Math.floor((H - 2) * (W - 2) * 0.52);
        while (floors < target) {
            let dir = Math.floor(Math.random() * 4);
            let dx = dir === 0 ? 1 : dir === 1 ? -1 : 0;
            let dy = dir === 2 ? 1 : dir === 3 ? -1 : 0;
            if (cx + dx >= 1 && cx + dx < W - 1 && cy + dy >= 1 && cy + dy < H - 1) {
                cx += dx; cy += dy;
                if (map[cy][cx] === '#') { map[cy][cx] = '.'; floors++; }
            }
        }

        // 宝箱($)3個、泉(+)1個、ショップ(S)1個、罠3個
        for (let i = 0; i < 3; i++) { let s = findEmptySpot(); map[s.y][s.x] = '$'; }
        let sp = findEmptySpot(); map[sp.y][sp.x] = '+';
        let sh = findEmptySpot(); map[sh.y][sh.x] = 'S';
        for (let i = 0; i < 3; i++) { let t = findEmptySpot(); traps[t.y][t.x] = Math.floor(Math.random() * 3) + 1; }

        // 階段とボス配置
        if (floor < 10) {
            let st = findEmptySpot(); map[st.y][st.x] = '>';
            if (floor === 5) {
                let b = findEmptySpot();
                enemies.push({ x: b.x, y: b.y, hp: 110, maxHp: 110, atk: 24, boss: 1, stun: 0, alive: true });
            }
        } else {
            let b = findEmptySpot();
            enemies.push({ x: b.x, y: b.y, hp: 220, maxHp: 220, atk: 38, boss: 2, stun: 0, alive: true });
        }

        // 通常モンスター配置
        let numE = Math.min(4, 2 + Math.floor(floor / 3));
        for (let i = 0; i < numE; i++) {
            let es = findEmptySpot();
            let ehp = 16 + floor * 5;
            enemies.push({ x: es.x, y: es.y, hp: ehp, maxHp: ehp, atk: 8 + floor * 2, boss: 0, stun: 0, alive: true });
        }
    }

    function startNewGame() {
        p = { x: 8, y: 5, hp: 90, maxHp: 90, mp: 40, maxMp: 40, food: 100, atk: 18, def: 5, lv: 1, exp: 0, gold: 30, potions: 2, breads: 2, arrows: 8 };
        floor = 1;
        aimMode = null;
        generateDungeon();
        document.getElementById("titleScreen").style.display = "none";
        document.getElementById("gameScreen").style.display = "block";
        addLog("🏰 冒険が始まった！ 右側のパネルで操作やアイテムを確認できます。");
        draw();
    }

    function returnToTitle() {
        inShop = false;
        document.getElementById("shopModal").style.display = "none";
        document.getElementById("gameScreen").style.display = "none";
        document.getElementById("titleScreen").style.display = "block";
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.font = "20px sans-serif";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";

        for (let y = 0; y < H; y++) {
            for (let x = 0; x < W; x++) {
                let px = x * TS + TS / 2;
                let py = y * TS + TS / 2 + 4;

                // 背景タイル
                if (map[y][x] === '#') {
                    ctx.fillStyle = "#2b2b36";
                    ctx.fillRect(x * TS + 1, y * TS + 1, TS - 2, TS - 2);
                } else {
                    ctx.fillStyle = "#14141c";
                    ctx.fillRect(x * TS + 1, y * TS + 1, TS - 2, TS - 2);
                    ctx.fillStyle = "#333";
                    ctx.fillRect(px - 1, py - 1, 2, 2);
                }

                // アイテム・罠・階段
                if (map[y][x] === '$') ctx.fillText("🎁", px, py);
                else if (map[y][x] === '+') ctx.fillText("⛲", px, py);
                else if (map[y][x] === 'S') ctx.fillText("⛺", px, py);
                else if (map[y][x] === '>') ctx.fillText("🚪", px, py);
                else if (traps[y][x] > 0) ctx.fillText("💥", px, py);
            }
        }

        // 敵の描画
        enemies.forEach(e => {
            if (!e.alive) return;
            let icon = e.boss === 2 ? "🐉" : e.boss === 1 ? "👹" : "👾";
            ctx.fillText(icon, e.x * TS + TS / 2, e.y * TS + TS / 2 + 4);
        });

        // プレイヤー描画
        ctx.fillText("🧙", p.x * TS + TS / 2, p.y * TS + TS / 2 + 4);

        // エフェクト描画
        if (fx) {
            ctx.fillText(fx.icon, fx.x * TS + TS / 2, fx.y * TS + TS / 2 + 4);
        }

        // UI数値更新
        document.getElementById("stFloor").innerText = floor;
        document.getElementById("stLv").innerText = p.lv;
        document.getElementById("stExp").innerText = p.exp;
        document.getElementById("stHp").innerText = Math.max(0, p.hp);
        document.getElementById("stMaxHp").innerText = p.maxHp;
        document.getElementById("stMp").innerText = p.mp;
        document.getElementById("stMaxMp").innerText = p.maxMp;
        document.getElementById("stFood").innerText = p.food;
        document.getElementById("stAtk").innerText = p.atk;
        document.getElementById("stDef").innerText = p.def;
        document.getElementById("stGold").innerText = p.gold;

        document.getElementById("btnPot").innerText = p.potions;
        document.getElementById("btnBread").innerText = p.breads;
        document.getElementById("btnArr").innerText = p.arrows;

        // 周囲の敵HPゲージ更新
        let hpHtml = "";
        enemies.forEach(e => {
            if (!e.alive) return;
            let dist = Math.abs(e.x - p.x) + Math.abs(e.y - p.y);
            if (dist <= 6 || e.boss > 0) {
                let name = e.boss === 2 ? "🐉邪竜" : e.boss === 1 ? "👹ゴーレム" : "👾敵";
                let pct = Math.max(0, Math.min(100, Math.floor((e.hp / e.maxHp) * 100)));
                let stunTxt = e.stun > 0 ? " <span style='color:#66ccff;'>[❄️凍結]</span>" : "";
                hpHtml += `<div style="margin-bottom:3px;">${name}(距:${dist}) <div class="hp-bar-bg"><div class="hp-bar-fill" style="width:${pct}%"></div></div> ${e.hp}/${e.maxHp}${stunTxt}</div>`;
            }
        });
        document.getElementById("enemyHpList").innerHTML = hpHtml || "周囲に敵はいません";
    }

    function showEffect(x, y, icon) {
        fx = { x, y, icon };
        draw();
        setTimeout(() => { fx = null; draw(); }, 180);
    }

    function damageEnemy(idx, dmg) {
        let e = enemies[idx];
        let name = e.boss === 2 ? "🐉邪竜ドラゴン" : e.boss === 1 ? "👹門番ゴーレム" : "👾モンスター";
        e.hp -= dmg;
        showEffect(e.x, e.y, "💥");
        addLog(`⚔️ ${name} に ${dmg} のダメージ！`);

        if (e.hp <= 0) {
            e.alive = false;
            let g = e.boss === 1 ? 150 : (15 + Math.floor(Math.random() * 15) + floor * 2);
            p.gold += g;
            addLog(`🎉 ${name} を倒した！（+${g} G）`);

            if (e.boss === 2) {
                draw();
                setTimeout(() => {
                    alert(`🏆 CONGRATULATIONS!! B10Fの邪竜を討伐し、大迷宮を完全制覇しました！\n最終Lv:${p.lv} / 所持金:${p.gold}G`);
                    returnToTitle();
                }, 200);
                return true;
            } else if (e.boss === 1) {
                p.potions++; p.breads++; p.arrows += 5; p.exp += 30;
                addLog("💎 中ボス撃破ボーナス！ 薬・パン・矢5本を入手！");
            } else {
                p.exp += 15;
            }

            while (p.exp >= 30) {
                p.lv++; p.exp -= 30;
                p.maxHp += 15; p.hp += 15; p.maxMp += 8; p.mp = p.maxMp;
                p.atk += 3; p.def += 2;
                addLog(`✨ レベルアップ！ Lv.${p.lv} になった！（HP+15/MP全快/攻+3/防+2）`);
            }
        }
        return false;
    }

    function startAim(type) {
        if (inShop || p.hp <= 0) return;
        if (type === 'f' && p.arrows <= 0) { addLog("💦 矢が1本もない！"); return; }
        if (type === 'q' && p.mp < 12) { addLog("💦 MPが足りない！（必要MP:12）"); return; }
        aimMode = type;
        addLog(type === 'f' ? "🏹 矢を撃つ方向キー（W/A/S/D または 矢印）を押してください" : "🔥 ファイアを放つ方向キー（W/A/S/D または 矢印）を押してください");
    }

    function handleCommand(key) {
        if (inShop || p.hp <= 0) return;

        // 方向指定モード（弓矢 or ファイアボール）
        if (aimMode) {
            let fdx = (key === 'd' || key === 'arrowright') ? 1 : (key === 'a' || key === 'arrowleft') ? -1 : 0;
            let fdy = (key === 's' || key === 'arrowdown') ? 1 : (key === 'w' || key === 'arrowup') ? -1 : 0;
            let mode = aimMode;
            aimMode = null;

            if (fdx === 0 && fdy === 0) {
                addLog("⚠️ 発射をキャンセルしました。");
                return;
            }

            if (mode === 'f') p.arrows--;
            if (mode === 'q') p.mp -= 12;

            let ax = p.x, ay = p.y, hit = false;
            let range = mode === 'f' ? 6 : 4;
            for (let s = 0; s < range; s++) {
                ax += fdx; ay += fdy;
                if (ax < 0 || ax >= W || ay < 0 || ay >= H || map[ay][ax] === '#') break;
                let eIdx = getEnemyAt(ax, ay);
                if (eIdx !== -1) {
                    let dmg = mode === 'f' ? p.atk : (30 + floor * 4);
                    if (damageEnemy(eIdx, dmg)) return;
                    hit = true;
                    break;
                }
            }
            if (!hit) addLog(mode === 'f' ? "💨 矢は誰にも当たらなかった。" : "💨 炎は闇へ消えた。");
            endTurn();
            return;
        }

        let dx = 0, dy = 0, turnUsed = false;
        if (key === 'w' || key === 'arrowup') { dy = -1; turnUsed = true; }
        else if (key === 's' || key === 'arrowdown') { dy = 1; turnUsed = true; }
        else if (key === 'a' || key === 'arrowleft') { dx = -1; turnUsed = true; }
        else if (key === 'd' || key === 'arrowright') { dx = 1; turnUsed = true; }
        else if (key === 'f') { startAim('f'); return; }
        else if (key === 'q') { startAim('q'); return; }
        else if (key === 'p') {
            if (p.potions > 0) {
                p.potions--; p.hp = Math.min(p.maxHp, p.hp + 70);
                showEffect(p.x, p.y, "💚");
                addLog(`🧪 ポーションを使ってHPが70回復した！（HP:${p.hp}）`);
                turnUsed = true;
            } else addLog("💦 ポーションを持っていない！");
        } else if (key === 'e') {
            if (p.breads > 0) {
                p.breads--; p.food = Math.min(100, p.food + 60);
                showEffect(p.x, p.y, "🥖");
                addLog(`🥖 パンを食べて満腹度が ${p.food}% に回復した！`);
                turnUsed = true;
            } else addLog("💦 パンを持っていない！");
        } else if (key === 'r') {
            if (p.food > 0 && p.hp < p.maxHp) {
                p.hp = Math.min(p.maxHp, p.hp + 3);
                addLog("💤 足踏みをして少し休んだ（HP+3）。");
            } else addLog("💤 足踏みをした。");
            turnUsed = true;
        } else if (key === 'x') {
            if (p.mp >= 15) {
                p.mp -= 15; p.hp = Math.min(p.maxHp, p.hp + 80);
                showEffect(p.x, p.y, "✨");
                addLog(`✨ ヒールを唱えた！ HPが80回復した！（HP:${p.hp}）`);
                turnUsed = true;
            } else addLog("💦 MPが足りない！（必要MP:15）");
        } else if (key === 'c') {
            if (p.mp >= 20) {
                p.mp -= 20; turnUsed = true;
                showEffect(p.x, p.y, "❄️");
                addLog("❄️ ブリザード発動！ 周囲の敵を凍らせた！");
                for (let i = 0; i < enemies.length; i++) {
                    if (!enemies[i].alive) continue;
                    if (Math.abs(enemies[i].x - p.x) + Math.abs(enemies[i].y - p.y) <= 4) {
                        enemies[i].stun = 2;
                        if (damageEnemy(i, 22 + floor * 3)) return;
                    }
                }
            } else addLog("💦 MPが足りない！（必要MP:20）");
        }

        // 移動と近接攻撃
        if (dx !== 0 || dy !== 0) {
            let nx = p.x + dx, ny = p.y + dy;
            if (nx < 0 || nx >= W || ny < 0 || ny >= H || map[ny][nx] === '#') {
                addLog("🧱 壁だ！ そちらには進めない。");
                draw();
                return;
            }
            let eIdx = getEnemyAt(nx, ny);
            if (eIdx !== -1) {
                let dmg = p.atk + Math.floor(Math.random() * 5) - 2;
                if (damageEnemy(eIdx, dmg)) return;
            } else {
                p.x = nx; p.y = ny;
                let tile = map[ny][nx];
                if (tile === '$') {
                    map[ny][nx] = '.';
                    showEffect(nx, ny, "✨");
                    let r = Math.floor(Math.random() * 7);
                    if (r === 0) { p.atk += 4; addLog("🎁 宝箱から『名刀』発見！ 攻撃力+4！"); }
                    else if (r === 1) { p.def += 3; addLog("🎁 宝箱から『鋼の盾』発見！ 防御力+3！"); }
                    else if (r === 2) { p.maxHp += 20; p.hp += 20; addLog("🎁 宝箱から『命の果実』発見！ 最大HP+20！"); }
                    else if (r === 3) { p.potions++; addLog("🎁 宝箱から『ポーション』を見つけた！"); }
                    else if (r === 4) { p.arrows += 5; addLog("🎁 宝箱から『木の矢×5本』を見つけた！"); }
                    else if (r === 5) { p.breads++; addLog("🎁 宝箱から『焼きたてパン』を見つけた！"); }
                    else { let g = 35 + Math.floor(Math.random() * 30); p.gold += g; addLog(`💰 宝箱から『金貨袋(${g}G)』を見つけた！`); }
                } else if (tile === '+') {
                    map[ny][nx] = '.';
                    p.hp = p.maxHp; p.mp = p.maxMp; p.food = Math.min(100, p.food + 20);
                    showEffect(nx, ny, "⛲");
                    addLog("⛲ 癒やしの泉を飲んだ！ HPとMPが全回復した！");
                } else if (tile === 'S') {
                    openShop();
                } else if (tile === '>') {
                    floor++;
                    p.mp = p.maxMp;
                    addLog(`🚪 階段を降りて B${floor}F へ進んだ！（MP全回復）`);
                    generateDungeon();
                    draw();
                    return;
                } else if (traps[ny][nx] > 0) {
                    let t = traps[ny][nx];
                    if (t === 1) {
                        let d = 6 + floor * 2; p.hp -= d;
                        addLog(`⚠️ トゲの罠を踏んだ！ ${d} のダメージ！`);
                    } else if (t === 2) {
                        let s = findEmptySpot(); p.x = s.x; p.y = s.y;
                        addLog("🌀 バネの罠を踏んで別の場所へワープさせられた！");
                    } else {
                        let d = 12 + floor * 3; p.hp -= d; traps[ny][nx] = 0;
                        addLog(`🪨 落石の罠が発動！ ${d} の大ダメージ！`);
                    }
                }
            }
        }

        if (turnUsed) endTurn();
        else draw();
    }

    function endTurn() {
        // 満腹度減少
        if (p.food > 0) {
            p.food--;
            if (p.food === 15) addLog("🍽️ お腹が減ってきた（残り15%）。[E]でパンを食べよう！");
        } else {
            p.hp -= 2;
            addLog("😵 空腹ダメージでHPが 2 減った！");
        }

        // 敵の行動
        enemies.forEach(e => {
            if (!e.alive || p.hp <= 0) return;
            if (e.stun > 0) { e.stun--; return; }
            let dist = Math.abs(e.x - p.x) + Math.abs(e.y - p.y);
            let name = e.boss === 2 ? "🐉邪竜" : e.boss === 1 ? "👹ゴーレム" : "👾敵";
            if (dist === 1) {
                let dmg = Math.max(1, e.atk - p.def + Math.floor(Math.random() * 4) - 1);
                p.hp -= dmg;
                showEffect(p.x, p.y, "💢");
                addLog(`${name} の攻撃！ 勇者は ${dmg} のダメージを受けた！`);
            } else if (dist <= 6) {
                let mx = p.x > e.x ? 1 : p.x < e.x ? -1 : 0;
                let my = p.y > e.y ? 1 : p.y < e.y ? -1 : 0;
                if (mx !== 0 && map[e.y][e.x + mx] !== '#' && getEnemyAt(e.x + mx, e.y) === -1 && (e.x + mx !== p.x || e.y !== p.y)) {
                    e.x += mx;
                } else if (my !== 0 && map[e.y + my][e.x] !== '#' && getEnemyAt(e.x, e.y + my) === -1 && (e.x !== p.x || e.y + my !== p.y)) {
                    e.y += my;
                }
            }
        });

        draw();

        if (p.hp <= 0) {
            setTimeout(() => {
                alert(`💀 勇者は B${floor}F で力尽きてしまった…… GAME OVER`);
                returnToTitle();
            }, 150);
        }
    }

    // ショップ処理
    function openShop() {
        inShop = true;
        document.getElementById("shopGold").innerText = p.gold;
        document.getElementById("shopModal").style.display = "block";
    }
    function closeShop() {
        inShop = false;
        document.getElementById("shopModal").style.display = "none";
        draw();
    }
    function buyItem(id) {
        if (id === 1 && p.gold >= 25) { p.gold -= 25; p.breads++; addLog("🥖 パンを購入した！"); }
        else if (id === 2 && p.gold >= 35) { p.gold -= 35; p.potions++; addLog("🧪 ポーションを購入した！"); }
        else if (id === 3 && p.gold >= 25) { p.gold -= 25; p.arrows += 5; addLog("🏹 木の矢×5本を購入した！"); }
        else if (id === 4 && p.gold >= 60) { p.gold -= 60; p.atk += 4; addLog(`🔨 武器を鍛えて 攻撃力:${p.atk} になった！`); }
        else if (id === 5 && p.gold >= 60) { p.gold -= 60; p.def += 3; addLog(`🛡️ 盾を鍛えて 防御力:${p.def} になった！`); }
        else { alert("お金（G）が足りません！"); }
        document.getElementById("shopGold").innerText = p.gold;
        draw();
    }

    // セーブ＆ロード
    function saveGame() {
        const data = { p, floor, map, traps, enemies };
        localStorage.setItem("rogue_web_save_v2", JSON.stringify(data));
        addLog("💾 現在の進行データをブラウザにセーブしました！");
        alert("セーブしました！");
    }
    function loadGame(fromTitle) {
        const raw = localStorage.getItem("rogue_web_save_v2");
        if (!raw) {
            alert("セーブデータが見つかりません。");
            return;
        }
        const data = JSON.parse(raw);
        p = data.p; floor = data.floor; map = data.map; traps = data.traps; enemies = data.enemies;
        if (fromTitle) {
            document.getElementById("titleScreen").style.display = "none";
            document.getElementById("gameScreen").style.display = "block";
        }
        addLog("📂 セーブデータをロードして再開しました！");
        draw();
    }

    // キーボード入力受付
    window.addEventListener("keydown", e => {
        if (inShop) {
            if (e.key >= '1' && e.key <= '5') buyItem( parseInt(e.key) );
            if (e.key === '0' || e.key === 'Escape') closeShop();
            return;
        }
        if (document.getElementById("gameScreen").style.display === "block") {
            let k = e.key.toLowerCase();
            if (["arrowup", "arrowdown", "arrowleft", "arrowright", " "].includes(k)) e.preventDefault();
            if (k === "escape") { returnToTitle(); return; }
            handleCommand(k);
        }
    });
</script>
</body>
</html>
