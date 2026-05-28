
<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Galactic Defender - UPDATE 11.6</title>
    <style>
        * { box-sizing: border-box; user-select: none; }
        body { 
            margin: 0; padding: 0; background: #000; color: #fff; 
            font-family: 'Segoe UI', system-ui, sans-serif; 
            overflow: hidden; touch-action: none; cursor: none;
        }
        canvas { display: block; position: absolute; top: 0; left: 0; z-index: 1; }
        .loader-overlay { 
            position: fixed; top: 0; left: 0; width: 100%; height: 100%; 
            background: radial-gradient(circle, #001a33 0%, #000 100%); 
            z-index: 2000; display: flex; flex-direction: column; justify-content: center; align-items: center; 
        }
        .loader-content { width: 260px; text-align: center; }
        .p-bar-outer { width: 100%; height: 8px; background: rgba(255,255,255,0.1); border-radius: 20px; border: 1px solid #00d2ff; overflow: hidden; margin-top: 12px; }
        .p-bar-inner { width: 0%; height: 100%; background: linear-gradient(90deg, #00d2ff, #00ffaa); transition: 0.1s; }
        .overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: none; flex-direction: column; justify-content: flex-start; align-items: center; z-index: 100; text-align: center; backdrop-filter: blur(6px); overflow-y: auto; padding: 20px 10px; }
        #main-hub { display: flex; background: radial-gradient(circle at center, #001533 0%, #010005 100%); z-index: 150; justify-content: flex-start; }
        #game-select-screen { display: none; background: radial-gradient(circle at center, #001533 0%, #010005 100%); z-index: 140; justify-content: flex-start; }
        #start-screen { display: none; background: radial-gradient(circle at center, #001533 0%, #010005 100%); justify-content: flex-start; position: relative; }
        #shop-screen { background: rgba(0,10,30,0.96); border: 1px solid #00d2ff; overflow-y: auto; justify-content: flex-start; }
        #rng-shop-screen { background: rgba(0,10,30,0.96); border: 1px solid #ff00ff; overflow-y: auto; justify-content: flex-start; }
        #achievements-screen { background: rgba(0,10,30,0.96); border: 1px solid gold; overflow-y: auto; justify-content: flex-start; }
        #events-screen { background: rgba(0,10,30,0.96); border: 1px solid #ff00ff; overflow-y: auto; justify-content: flex-start; }
        #skins-screen { background: rgba(0,10,30,0.96); border: 1px solid #ff66ff; overflow-y: auto; justify-content: flex-start; }
        #settings-screen { background: rgba(0,10,30,0.96); border: 1px solid #00ffaa; overflow-y: auto; justify-content: flex-start; }
        #update-log-screen { background: rgba(0,10,30,0.96); border: 1px solid #00ffaa; overflow-y: auto; justify-content: flex-start; }
        #game-over { background: rgba(40,0,0,0.96); justify-content: center; }
        #pause-screen { background: rgba(0,10,30,0.94); justify-content: center; }
        #ascend-screen { background: rgba(0,0,0,0.95); border: 2px solid gold; justify-content: center; }
        .side-btn { position: absolute; right: 10px; width: 40px; height: 40px; border-radius: 50%; background: rgba(0,30,60,0.8); border: 1px solid #00d2ff; color: #fff; font-size: 20px; display: flex; align-items: center; justify-content: center; cursor: pointer; z-index: 60; transition: 0.2s; }
        .side-btn:hover { background: rgba(0,80,120,0.9); transform: scale(1.05); }
        #ach-side-btn { top: 80px; }
        #events-side-btn { top: 130px; }
        #skins-side-btn { top: 180px; }
        #settings-side-btn { top: 230px; }
        .settings-option { background: rgba(0,0,0,0.5); border-radius: 10px; padding: 10px; margin: 8px; display: flex; justify-content: space-between; align-items: center; width: 300px; }
        .settings-toggle { width: 50px; height: 25px; background: #333; border-radius: 25px; cursor: pointer; transition: 0.2s; position: relative; }
        .settings-toggle.on { background: #00ffaa; }
        .settings-toggle.on.music-toggle-on { background: #ff66ff; }
        .settings-toggle:after { content: ""; position: absolute; width: 21px; height: 21px; background: #fff; border-radius: 50%; top: 2px; left: 3px; transition: 0.2s; }
        .settings-toggle.on:after { left: 26px; }
        .custom-alert { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: linear-gradient(135deg, #001a33, #000); border: 2px solid #00d2ff; border-radius: 15px; padding: 20px; min-width: 280px; max-width: 400px; z-index: 3000; text-align: center; backdrop-filter: blur(10px); display: none; flex-direction: column; gap: 15px; }
        .custom-alert p { margin: 0; font-size: 16px; }
        .custom-alert button { background: #00d2ff; border: none; padding: 8px 20px; border-radius: 25px; color: #000; font-weight: bold; cursor: pointer; margin-top: 10px; }
        .notification-area { position: fixed; top: 80px; right: 20px; width: 280px; z-index: 2500; display: flex; flex-direction: column; gap: 8px; pointer-events: none; }
        .notification { background: linear-gradient(135deg, rgba(0,30,60,0.95), rgba(0,10,30,0.95)); border-right: 4px solid; border-radius: 10px; padding: 10px 15px; animation: slideInRight 0.3s ease-out, fadeOut 0.5s ease-out 4.5s forwards; transform-origin: right; pointer-events: none; }
        @keyframes slideInRight { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
        @keyframes fadeOut { to { opacity: 0; transform: translateX(100%); } }
        .notification-warning { border-right-color: #ff0044; }
        .notification-success { border-right-color: #00ffaa; }
        .notification-event { border-right-color: #ff00ff; }
        .notification-info { border-right-color: #00d2ff; }
        .tooltip { position: relative; display: inline-block; cursor: help; }
        .tooltip .tooltip-text { visibility: hidden; width: 200px; background-color: #001a33; color: #fff; text-align: center; border-radius: 6px; padding: 5px; position: absolute; z-index: 1; bottom: 125%; left: 50%; margin-left: -100px; opacity: 0; transition: opacity 0.3s; border: 1px solid #00d2ff; font-size: 11px; pointer-events: none; }
        .tooltip:hover .tooltip-text { visibility: visible; opacity: 1; }
        .gem-notification { position: fixed; top: 20%; left: 50%; transform: translate(-50%, -50%); background: linear-gradient(135deg, #8e44ad, #ff00ff); color: gold; padding: 10px 20px; border-radius: 30px; font-size: 20px; font-weight: bold; z-index: 2000; animation: gemPop 1s ease-out forwards; pointer-events: none; white-space: nowrap; }
        @keyframes gemPop { 0% { opacity: 0; transform: translate(-50%, -50%) scale(0.5); } 20% { opacity: 1; transform: translate(-50%, -50%) scale(1.2); } 80% { opacity: 1; transform: translate(-50%, -50%) scale(1); } 100% { opacity: 0; transform: translate(-50%, -80%) scale(0.8); } }
        #game-timer { position: absolute; bottom: 12px; left: 12px; font-size: 10px; color: #aaa; background: rgba(0,0,0,0.5); padding: 2px 8px; border-radius: 10px; z-index: 50; display: none; pointer-events: none; }
        .daily-reward-btn { background: linear-gradient(45deg, #ffaa00, #ff6600); border: none; color: #fff; padding: 8px 15px; border-radius: 25px; font-weight: bold; cursor: pointer; margin: 5px; font-size: 12px; }
        .daily-mission-card { background: rgba(0,0,0,0.5); border-radius: 10px; padding: 8px 12px; margin: 5px; border-right: 3px solid #ffaa00; text-align: right; }
        .lootbox-animation { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); z-index: 1000; display: none; justify-content: center; align-items: center; flex-direction: column; }
        .lootbox { width: 200px; height: 200px; background: #8B4513; border-radius: 20px; display: flex; justify-content: center; align-items: center; font-size: 80px; animation: shake 0.5s infinite; cursor: pointer; }
        @keyframes shake { 0%{transform:rotate(0deg);} 25%{transform:rotate(10deg);} 75%{transform:rotate(-10deg);} 100%{transform:rotate(0deg);} }
        .lootbox-opening { animation: openBox 0.5s forwards; }
        @keyframes openBox { 0%{transform:scale(1);} 50%{transform:scale(1.2);background:#ffaa00;} 100%{transform:scale(0);opacity:0;} }
        .reward-display { font-size: 24px; margin-top: 20px; animation: fadeIn 0.5s; text-align: center; }
        .lootbox.rarity-common { box-shadow: 0 0 10px rgba(66,135,245,0.3); }
        .lootbox.rarity-rare { box-shadow: 0 0 10px rgba(66,245,182,0.3); }
        .lootbox.rarity-epic { box-shadow: 0 0 12px rgba(245,167,66,0.3); }
        .lootbox.rarity-legendary { box-shadow: 0 0 12px rgba(245,66,209,0.3); }
        .lootbox.rarity-mythic { box-shadow: 0 0 14px rgba(245,66,66,0.3); }
        .lootbox.rarity-ultra { box-shadow: 0 0 14px rgba(245,230,66,0.4); }
        @keyframes lootboxShake { 0%{transform:rotate(0deg) scale(1);} 10%{transform:rotate(-8deg) scale(1.02);} 20%{transform:rotate(8deg) scale(1.04);} 30%{transform:rotate(-6deg) scale(1.06);} 40%{transform:rotate(6deg) scale(1.08);} 50%{transform:rotate(-4deg) scale(1.1);} 60%{transform:rotate(4deg) scale(1.12);} 70%{transform:rotate(-2deg) scale(1.14);} 80%{transform:rotate(2deg) scale(1.16);} 90%{transform:rotate(-1deg) scale(1.18);} 100%{transform:rotate(0deg) scale(1.2);} }
        @keyframes lootboxOpen { 0%{transform:scale(1.2);opacity:1;} 30%{transform:scale(1.5);opacity:1;} 60%{transform:scale(1.8);opacity:0.8;} 100%{transform:scale(0);opacity:0;} }
        @keyframes rewardReveal { 0%{opacity:0;transform:scale(0.3) rotate(-10deg);} 50%{opacity:1;transform:scale(1.15) rotate(3deg);} 70%{transform:scale(0.95) rotate(-1deg);} 100%{opacity:1;transform:scale(1) rotate(0deg);} }
        .lootbox-particle { position: absolute; width: 6px; height: 6px; border-radius: 50%; pointer-events: none; animation: particleBurst 0.8s ease-out forwards; }
        @keyframes particleBurst { 0%{opacity:1;transform:translate(0,0) scale(1);} 100%{opacity:0;transform:translate(var(--px),var(--py)) scale(0);} }
        .reward-card { background: linear-gradient(135deg, rgba(0,20,40,0.95), rgba(0,10,30,0.95)); border: 2px solid #00d2ff; border-radius: 16px; padding: 20px 25px; animation: rewardReveal 0.6s ease-out; }
        .reward-card.rarity-common { border-color: #4287f5; }
        .reward-card.rarity-rare { border-color: #42f5b6; }
        .reward-card.rarity-epic { border-color: #f5a742; }
        .reward-card.rarity-legendary { border-color: #f542d1; }
        .reward-card.rarity-mythic { border-color: #f54242; }
        .reward-card.rarity-ultra { border-color: gold; }
        @keyframes fadeIn { from{opacity:0;transform:scale(0.5);} to{opacity:1;transform:scale(1);} }
        .btn { background: rgba(0,30,60,0.7); color:#fff; border:1px solid #00d2ff; padding:8px 20px; border-radius:25px; font-weight:bold; cursor:pointer; margin:5px; font-size:12px; transition:0.2s; backdrop-filter:blur(3px); }
        .btn:hover { background: rgba(0,80,120,0.8); transform:scale(1.02); }
        .btn-danger { border-color:#ff0044; background:rgba(80,0,0,0.7); }
        .btn-gem { border-color:#ff66ff; background:linear-gradient(45deg,#8e44ad,#ff00ff); }
        .game-select-card { background:rgba(0,30,60,0.8); border:2px solid #00d2ff; border-radius:20px; padding:25px; margin:15px; width:280px; cursor:pointer; transition:0.3s; }
        .game-select-card:hover { transform:scale(1.05); border-color:#ff00ff; background:rgba(0,50,100,0.9); }
        .game-select-card.coming-soon { opacity:0.6; border-color:#888; cursor:not-allowed; }
        #skill-tree-screen { background:rgba(0,10,30,0.96); border:1px solid #00ffaa; overflow-y:auto; justify-content:flex-start; }
        .skill-path { display:flex; flex-direction:column; align-items:center; margin:8px; min-width:120px; }
        .skill-path-title { font-size:14px; font-weight:bold; margin-bottom:8px; text-align:center; }
        .skill-node { width:52px; height:52px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-size:18px; cursor:pointer; margin:4px 0; transition:0.2s; position:relative; }
        .skill-node.locked { background:rgba(60,60,60,0.6); border:2px solid #555; opacity:0.5; cursor:not-allowed; }
        .skill-node.available { background:rgba(0,80,120,0.7); border:2px solid #00d2ff; animation:pulse 1.5s infinite; }
        .skill-node.unlocked { background:rgba(0,180,80,0.6); border:2px solid #00ffaa; box-shadow:0 0 8px rgba(0,255,170,0.4); }
        .skill-connector { width:2px; height:12px; background:#444; }
        .skill-connector.active { background:#00ffaa; }
        @keyframes pulse { 0%{box-shadow:0 0 5px rgba(0,210,255,0.3)} 50%{box-shadow:0 0 15px rgba(0,210,255,0.6)} 100%{box-shadow:0 0 5px rgba(0,210,255,0.3)} }
        .skill-tooltip { position:absolute; bottom:110%; left:50%; transform:translateX(-50%); background:#001a33; border:1px solid #00d2ff; border-radius:8px; padding:6px 10px; font-size:10px; white-space:nowrap; pointer-events:none; z-index:10; display:none; }
        .skill-node:hover .skill-tooltip { display:block; }
        .game-select-card.coming-soon:hover { transform:none; }
        .shop-tabs { display: flex; gap: 4px; margin: 8px 0; justify-content: center; flex-wrap: wrap; }
        .shop-tab { background: rgba(0,30,60,0.7); color: #aaa; border: 1px solid #444; padding: 6px 14px; border-radius: 20px; cursor: pointer; font-size: 11px; font-weight: bold; transition: 0.2s; }
        .shop-tab:hover { border-color: #00d2ff; color: #fff; }
        .shop-tab.active { background: rgba(0,80,120,0.9); border-color: #00d2ff; color: #fff; }
        .ability-btn { position: absolute; bottom: 85px; right: 15px; z-index: 55; display: none; pointer-events: all; }
        .ability-icon { width: 44px; height: 44px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 18px; cursor: pointer; border: 2px solid; position: relative; margin-bottom: 6px; transition: 0.2s; }
        .ability-icon:hover { transform: scale(1.1); }
        .ability-icon.on-cooldown { opacity: 0.4; cursor: not-allowed; }
        .ability-cooldown-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; border-radius: 50%; background: rgba(0,0,0,0.6); display: flex; align-items: center; justify-content: center; font-size: 9px; color: #fff; font-weight: bold; }
        .shop-grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; margin:12px; width:90%; max-width:380px; }
        .card { background:rgba(255,255,255,0.05); border:1px solid #444; padding:6px; border-radius:10px; cursor:pointer; transition:0.2s; font-size:11px; position:relative; }
        .card:hover { background:rgba(0,210,255,0.1); border-color:#00d2ff; transform:scale(1.02); }
        .card.cant-afford { opacity:0.4; cursor:not-allowed; }
        .rarity-common { border-color:#4287f5; background:rgba(66,135,245,0.2); }
        .rarity-rare { border-color:#42f5b6; background:rgba(66,245,182,0.2); }
        .rarity-epic { border-color:#f5a742; background:rgba(245,167,66,0.2); }
        .rarity-legendary { border-color:#f542d1; background:rgba(245,66,209,0.2); }
        .rarity-mythic { border-color:#f54242; background:rgba(245,66,66,0.2); }
        .rarity-ultra { border-color:#f5e642; background:rgba(245,230,66,0.2); }
        .gem-counter { position:absolute; top:10px; left:10px; background:rgba(0,0,0,0.6); border-radius:20px; padding:5px 12px; font-size:14px; color:#ff66ff; border:1px solid #ff66ff; z-index:200; }
        .gem-counter span { color:#ffcc00; font-weight:bold; }
        .update-item { background:rgba(0,0,0,0.5); border-radius:10px; padding:10px; margin:8px; text-align:right; border-right:3px solid #00ffaa; width: 90%; max-width: 500px; }
        .update-version { color:#00ffaa; font-weight:bold; font-size:14px; }
        .update-desc { color:#ccc; font-size:12px; margin-top:5px; }
        #ui-hud { position: absolute; top: 10px; left: 10px; z-index: 50; display: none; pointer-events: none; }
        .stat-group { margin-bottom: 6px; }
        .stat-label { font-size: 9px; font-weight: bold; color: #00d2ff; letter-spacing: 1px; margin-bottom: 1px; }
        .bar-bg { width: 120px; height: 6px; background: rgba(0,0,0,0.6); border-radius: 3px; border: 1px solid #444; overflow: hidden; }
        .bar-fill { height: 100%; transition: width 0.2s; }
        #hp-fill { background: linear-gradient(90deg, #ff0044, #ff5588); }
        #xp-fill { background: linear-gradient(90deg, #00ffaa, #00ffff); width: 0%; }
        #od-fill { background: linear-gradient(90deg, #8e44ad, #ff00ff); width: 0%; }
        #score-hud { position: absolute; top: 10px; right: 10px; z-index: 50; display: none; pointer-events: none; text-align: right; }
        #score-hud .score-main { font-size: 16px; font-weight: 900; color: #fff; text-shadow: 0 0 5px #00d2ff; }
        #score-hud .score-sub { font-size: 9px; color: #aaa; margin-top: 1px; }
        #combo-small { position: absolute; bottom: 12px; right: 12px; font-size: 16px; font-weight: bold; color: #ffcc00; display: none; z-index: 50; text-shadow: 0 0 3px orange; pointer-events: none; }
        .combo-meter { position: absolute; bottom: 50px; right: 12px; width: 100px; height: 8px; background: #333; border-radius: 4px; overflow: hidden; display: none; }
        .combo-meter-fill { height: 100%; width: 0%; background: linear-gradient(90deg, #ffaa00, #ff6600); transition: width 0.1s; }
        #od-btn { position: absolute; bottom: 15px; right: 15px; width: 60px; height: 60px; background: rgba(142,68,173,0.5); border: 2px solid #8e44ad; border-radius: 50%; display: none; justify-content: center; align-items: center; z-index: 500; color: #fff; font-weight: bold; cursor: pointer; font-size: 9px; line-height: 1.2; }
        #powerup-bar { position:absolute; bottom:85px; left:50%; transform:translateX(-50%); z-index:50; display:none; pointer-events:none; text-align:center; white-space:nowrap; }
        .pu-item { display:inline-block; margin:0 2px; background:rgba(0,0,0,0.7); border-radius:6px; padding:1px 5px; border:1px solid #555; font-size:9px; font-weight:bold; }
        #wave-banner { position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); font-size:28px; font-weight:900; color:#00d2ff; text-shadow:0 0 15px #00d2ff; display:none; z-index:200; pointer-events:none; text-align:center; white-space:nowrap; background:rgba(0,0,0,0.6); padding:5px 15px; border-radius:30px; }
        #event-banner { position:absolute; top:40%; left:50%; transform:translate(-50%,-50%); font-size:24px; font-weight:900; display:none; z-index:200; pointer-events:none; text-align:center; white-space:nowrap; background:rgba(0,0,0,0.7); padding:5px 15px; border-radius:30px; }
        #boss-warning { position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); font-size:28px; font-weight:900; color:#ff0044; text-shadow:0 0 15px #ff0044; display:none; z-index:200; pointer-events:none; animation:bossWarn 0.3s infinite alternate; background:rgba(0,0,0,0.5); padding:4px 12px; border-radius:25px; white-space:nowrap; }
        @keyframes bossWarn { from{opacity:0.8;} to{opacity:0.3;} }
        #crosshair { position:fixed; z-index:999; pointer-events:none; display:none; top:0; left:0; }
        #vignette { position:fixed; top:0; left:0; width:100%; height:100%; pointer-events:none; z-index:49; display:none; background:radial-gradient(circle,transparent 50%,rgba(255,0,0,0.25) 100%); }
        #nebulaCanvas { position:absolute; top:0; left:0; z-index:0; pointer-events:none; opacity:0.12; }
        .achievements-grid { display:grid; grid-template-columns:1fr 1fr; gap:6px; margin:12px; max-width:750px; width: 95%; padding:6px; }
        .ach-card { background:rgba(0,0,0,0.6); border:1px solid #555; border-radius:6px; padding:8px 10px; text-align:right; font-size:12px; position:relative; transition:0.2s; overflow:hidden; }
        .ach-card.locked { opacity:0.5; filter:grayscale(0.3); }
        .ach-name { color:gold; font-weight:bold; font-size:13px; }
        .ach-desc { color:#bbb; font-size:10px; }
        .ach-status { font-size:12px; margin-left:4px; }
        .ach-progress-bar { height:5px; background:#333; border-radius:3px; margin-top:3px; overflow:hidden; }
        .ach-progress-fill { height:100%; width:0%; background:linear-gradient(90deg,gold,#ffcc44); transition:width 0.2s; }
        .ach-progress-text { font-size:9px; color:#888; margin-top:2px; text-align:left; }
        .ach-card.ach-easy { border-color: #00ff88; }
        .ach-card.ach-medium { border-color: #00d2ff; }
        .ach-card.ach-hard { border-color: #ff8800; }
        .ach-card.ach-extreme { border-color: #ff2200; }
        .ach-card.ach-mythic { border-color: #ff00ff; box-shadow: 0 0 8px rgba(255,0,255,0.4); }
        .ach-card.ach-new-unlock { animation: achGlow 1s ease-out; }
        @keyframes achGlow { 0% { box-shadow: 0 0 15px gold; } 100% { box-shadow: none; } }
        .ach-filter-tabs { display: flex; gap: 4px; margin: 8px 12px; flex-wrap: wrap; justify-content: center; }
        .ach-filter-tab { background: rgba(0,30,60,0.7); color: #aaa; border: 1px solid #444; padding: 4px 10px; border-radius: 15px; cursor: pointer; font-size: 10px; font-weight: bold; transition: 0.2s; }
        .ach-filter-tab:hover { border-color: #00d2ff; color: #fff; }
        .ach-filter-tab.active { background: rgba(0,80,120,0.9); border-color: #00d2ff; color: #fff; }
        .ach-unlocked-count { text-align: center; margin: 8px 0; font-size: 14px; }
        .ach-unlocked-count span { color: gold; font-weight: bold; }
        .ach-progress-bar-total { height: 8px; background: #333; border-radius: 4px; margin: 6px 12px; overflow: hidden; max-width: 750px; }
        .ach-progress-fill-total { height: 100%; background: linear-gradient(90deg, #00ff88, #00d2ff, #ff00ff); transition: width 0.3s; border-radius: 4px; }
        .events-grid { display:grid; grid-template-columns:1fr; gap:8px; margin:15px; max-width:500px; width: 95%; padding:8px; }
        .event-card { background:rgba(0,0,0,0.7); border:1px solid #ff00ff; border-radius:10px; padding:8px 12px; text-align:right; }
        .event-name { color:#ff00ff; font-weight:bold; font-size:14px; }
        .event-chance { color:#ffaa00; font-size:11px; }
        .event-desc { color:#ccc; font-size:11px; margin-top:4px; }
        .skins-grid { display:grid; grid-template-columns:1fr 1fr; gap:12px; margin:12px; max-width:600px; width: 95%; padding:6px; }
        .skin-card { background:rgba(0,0,0,0.6); border:2px solid #555; border-radius:12px; padding:10px; text-align:center; cursor:pointer; transition:0.2s; }
        .skin-card.owned { border-color:gold; background:rgba(255,215,0,0.1); }
        .skin-card.locked { opacity:0.5; filter:grayscale(0.5); cursor:not-allowed; }
        .skin-card.equipped { border-color:#ff66ff; box-shadow:0 0 15px #ff66ff; }
        .skin-icon { font-size:48px; margin-bottom:8px; }
        .skin-name { font-weight:bold; font-size:14px; margin-bottom:4px; }
        .skin-desc { font-size:10px; color:#aaa; }
        .skin-effect { font-size:9px; color:#ffaa00; margin-top:5px; }
        .quantity-selector { display:flex; gap:4px; margin-top:6px; justify-content:center; }
        .qty-btn { background:#333; border:none; color:#fff; border-radius:4px; padding:2px 6px; font-size:9px; cursor:pointer; }
        #pause-btn { position:absolute; top:10px; right:50%; transform:translateX(50%); z-index:50; display:none; background:rgba(0,0,0,0.5); border:1px solid #666; color:#fff; padding:4px 12px; border-radius:15px; cursor:pointer; font-size:10px; pointer-events:all; }
        .start-stats { background:rgba(0,0,0,0.4); border:1px solid #00d2ff33; border-radius:12px; padding:8px 15px; margin:10px 0; }
        .reset-section { margin-top:20px; padding-top:15px; border-top:1px solid #ff0044; }
        .achievement-popup-fixed { position: fixed; bottom: 20px; left: 20px; background: linear-gradient(135deg, rgba(0,30,60,0.95), rgba(0,10,30,0.95)); border: 2px solid gold; border-radius: 12px; padding: 10px 16px; min-width: 220px; max-width: 300px; z-index: 300; display: none; animation: slideIn 0.3s ease; pointer-events: none; }
        .achievement-popup-fixed .title { color: gold; font-size: 11px; font-weight: bold; letter-spacing: 1px; }
        .achievement-popup-fixed .name { font-size: 14px; font-weight: bold; margin-top: 2px; }
        .achievement-popup-fixed .desc { font-size: 11px; color: #ccc; margin-top: 2px; }
        @keyframes slideIn { from { transform: translateX(-120%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
        .rank-badge { position: absolute; top: 80px; left: 10px; background: linear-gradient(135deg, #ffaa00, #ff6600); border-radius: 20px; padding: 4px 12px; font-size: 10px; font-weight: bold; color: #000; z-index: 55; display: none; }
        .daily-mission-header { background: linear-gradient(135deg, #00d2ff, #00ffaa); border-radius: 10px; padding: 5px 15px; margin-bottom: 10px; color: #000; font-weight: bold; }
        .stats-panel { background: rgba(0,0,0,0.5); border-radius: 10px; padding: 8px; margin-top: 10px; font-size: 11px; display: flex; justify-content: space-between; flex-wrap: wrap; gap: 8px; }
        .critical-hit { animation: critFlash 0.2s ease-out; }
        @keyframes critFlash { 0% { text-shadow: 0 0 0px #ffaa00; } 50% { text-shadow: 0 0 20px #ffaa00; } 100% { text-shadow: 0 0 0px #ffaa00; } }
        /* === NEW UI/QOL STYLES === */
        .wave-progress { position: absolute; top: 58px; right: 10px; z-index: 50; display: none; pointer-events: none; text-align: right; }
        .wave-progress-bar { width: 100px; height: 5px; background: rgba(0,0,0,0.6); border-radius: 3px; border: 1px solid #444; overflow: hidden; margin-top: 2px; }
        .wave-progress-fill { height: 100%; background: linear-gradient(90deg, #00d2ff, #00ffaa); transition: width 0.2s; }
        .wave-progress-text { font-size: 8px; color: #aaa; }
        .auto-fire-btn { position: absolute; bottom: 45px; left: 12px; z-index: 55; display: none; background: rgba(0,30,60,0.7); border: 1px solid #00d2ff; color: #00d2ff; padding: 4px 10px; border-radius: 15px; font-size: 9px; font-weight: bold; cursor: pointer; pointer-events: all; transition: 0.2s; }
        .auto-fire-btn.active { background: rgba(0,210,255,0.3); border-color: #00ffaa; color: #00ffaa; }
        .auto-fire-btn:hover { transform: scale(1.05); }
        .tutorial-screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; display: none; flex-direction: column; justify-content: center; align-items: center; z-index: 200; background: rgba(0,10,30,0.96); backdrop-filter: blur(6px); text-align: center; overflow-y: auto; padding: 20px; border: 1px solid #00d2ff; }
        .tutorial-screen h2 { color: #00d2ff; margin-bottom: 15px; }
        .tutorial-row { display: flex; justify-content: space-between; align-items: center; width: 280px; padding: 6px 10px; margin: 3px 0; background: rgba(0,0,0,0.5); border-radius: 8px; font-size: 12px; }
        .tutorial-key { background: rgba(0,80,120,0.8); border: 1px solid #00d2ff; border-radius: 6px; padding: 2px 10px; font-weight: bold; color: #00d2ff; font-size: 11px; }
        .fps-counter { position: absolute; top: 2px; left: 50%; transform: translateX(-50%); z-index: 55; display: none; font-size: 9px; color: #888; background: rgba(0,0,0,0.4); padding: 1px 8px; border-radius: 8px; pointer-events: none; }
        .save-indicator { position: fixed; bottom: 10px; left: 50%; transform: translateX(-50%); z-index: 2500; font-size: 10px; color: #00ffaa; background: rgba(0,30,60,0.8); border: 1px solid #00ffaa; border-radius: 10px; padding: 2px 12px; opacity: 0; transition: opacity 0.3s; pointer-events: none; }
        .save-indicator.show { opacity: 1; }
        .volume-slider { -webkit-appearance: none; appearance: none; width: 120px; height: 6px; border-radius: 3px; background: #333; outline: none; cursor: pointer; }
        .volume-slider::-webkit-slider-thumb { -webkit-appearance: none; appearance: none; width: 14px; height: 14px; border-radius: 50%; background: #00ffaa; cursor: pointer; border: 1px solid #00d2ff; }
        .volume-slider::-moz-range-thumb { width: 14px; height: 14px; border-radius: 50%; background: #00ffaa; cursor: pointer; border: 1px solid #00d2ff; }
        .enemy-hp-bar { position: absolute; pointer-events: none; }
        .controls-section { margin-top: 20px; padding-top: 12px; border-top: 1px solid #00ffaa33; width: 300px; }
        .controls-section h3 { color: #00ffaa; font-size: 12px; margin-bottom: 8px; }
        .control-row { display: flex; justify-content: space-between; align-items: center; padding: 3px 8px; font-size: 10px; color: #aaa; }
        .control-key { background: rgba(0,80,120,0.5); border: 1px solid #00d2ff44; border-radius: 4px; padding: 1px 8px; color: #00d2ff; font-weight: bold; font-size: 9px; }
        .reset-modal { position: fixed; top: 50%; left: 50%; transform: translate(-50%,-50%); background: linear-gradient(135deg, #001a33, #000); border: 2px solid #ff0044; border-radius: 15px; padding: 20px; min-width: 280px; max-width: 400px; z-index: 3500; text-align: center; backdrop-filter: blur(10px); display: none; flex-direction: column; gap: 12px; }
        .reset-modal p { margin: 0; font-size: 12px; color: #ccc; }
        .reset-modal .reset-warning { color: #ff0044; font-weight: bold; }
        .pause-volume-row { display: flex; align-items: center; gap: 8px; margin: 6px 0; width: 260px; justify-content: space-between; }
        .pause-volume-row span { font-size: 10px; color: #aaa; min-width: 60px; }
        .pause-quick-toggle { width: 36px; height: 18px; background: #333; border-radius: 18px; cursor: pointer; transition: 0.2s; position: relative; flex-shrink: 0; }
        .pause-quick-toggle.on { background: #00ffaa; }
        .pause-quick-toggle.music-on { background: #ff66ff; }
        .pause-quick-toggle:after { content: ""; position: absolute; width: 14px; height: 14px; background: #fff; border-radius: 50%; top: 2px; left: 2px; transition: 0.2s; }
        .pause-quick-toggle.on:after, .pause-quick-toggle.music-on:after { left: 20px; }
        .hub-title { color: #00d2ff; }
.hub-btn-hero { background: linear-gradient(135deg, #00d2ff, #00ffaa); color: #000; font-size: 20px; padding: 14px 40px; border: none; border-radius: 30px; font-weight: 900; cursor: pointer; margin: 8px; transition: 0.3s; }
.hub-btn-hero:hover { transform: scale(1.08); }
.hub-stat-card { background: rgba(0,0,0,0.6); border-radius: 12px; padding: 8px 14px; display: flex; align-items: center; gap: 8px; border-left: 3px solid; }
.hub-stat-card.stat-record { border-left-color: gold; }
.hub-stat-card.stat-credits { border-left-color: #00ffaa; }
.hub-stat-card.stat-gems { border-left-color: #ff66ff; }
.hub-stat-card.stat-kills { border-left-color: #ff4444; }
.hub-stat-card.stat-skin { border-left-color: #ffaa00; }
.hub-section { background: rgba(0,0,0,0.4); border-radius: 15px; padding: 15px; margin: 10px auto; width: 90%; max-width: 450px; }
.hub-divider { width: 60%; height: 1px; background: linear-gradient(90deg, transparent, #00d2ff, transparent); margin: 15px auto; }
.hub-btn { background: rgba(0,30,60,0.7); color: #fff; border: 1px solid #00d2ff; padding: 10px 24px; border-radius: 25px; font-weight: bold; cursor: pointer; margin: 5px; font-size: 14px; transition: 0.3s; }
.hub-btn:hover { background: rgba(0,80,120,0.9); transform: scale(1.05); }
.hub-btn-reward { background: linear-gradient(45deg, #ffaa00, #ff6600); border: none; color: #fff; padding: 10px 24px; border-radius: 25px; font-weight: bold; cursor: pointer; margin: 5px; font-size: 14px; transition: 0.3s; }
.hub-btn-reward:hover { transform: scale(1.05); }
.hub-version-badge { display: inline-block; background: linear-gradient(135deg, #ff66ff, #8e44ad); border-radius: 15px; padding: 3px 15px; font-size: 12px; font-weight: bold; }
.streak-badge { display: inline-flex; align-items: center; gap: 6px; background: linear-gradient(135deg, rgba(255,100,0,0.3), rgba(255,50,0,0.2)); border: 1px solid #ff6600; border-radius: 20px; padding: 6px 14px; font-size: 14px; font-weight: bold; color: #ffaa00; margin: 8px 0; }
.streak-popup { position: fixed; top: 50%; left: 50%; transform: translate(-50%,-50%); background: linear-gradient(135deg, #1a0800, #000); border: 2px solid #ff6600; border-radius: 16px; padding: 20px; min-width: 280px; max-width: 400px; z-index: 3000; text-align: center; display: none; flex-direction: column; gap: 10px; animation: streakPopIn 0.4s ease-out; }
@keyframes streakPopIn { 0% { transform: translate(-50%,-50%) scale(0.5); opacity:0; } 100% { transform: translate(-50%,-50%) scale(1); opacity:1; } }
.streak-popup h2 { color: #ffaa00; margin: 0; }
.streak-reward-item { background: rgba(255,100,0,0.15); border: 1px solid #ff660066; border-radius: 8px; padding: 6px 12px; margin: 4px 0; font-size: 13px; color: #ffcc00; }
.streak-claim-btn { background: linear-gradient(135deg, #ff6600, #ff9900); border: none; color: #000; padding: 10px 25px; border-radius: 25px; font-weight: 900; cursor: pointer; font-size: 14px; transition: 0.2s; margin-top: 5px; }
.streak-claim-btn:hover { transform: scale(1.05); }
    </style>
</head>
<body>
<div id="loader-init" class="loader-overlay">
    <div class="loader-content">
        <h1 style="color:#00d2ff;letter-spacing:2px;font-size:1.5rem;" data-i18n="warp_initiated">WARP INITIATED</h1>
        <p style="color:#00ffaa;font-size:9px;" data-i18n="calibrating">CALIBRATING...</p>
        <div class="p-bar-outer"><div id="init-fill" class="p-bar-inner"></div></div>
    </div>
</div>
<div id="loader-death" class="loader-overlay" style="display:none;">
    <div class="loader-content">
        <h1 style="color:#ff0044;letter-spacing:2px;font-size:1.5rem;" data-i18n="core_failure">CORE FAILURE</h1>
        <p style="color:#aaa;font-size:9px;" data-i18n="restoring">RESTORING...</p>
        <div class="p-bar-outer"><div id="death-fill" class="p-bar-inner" style="background:#ff0044;"></div></div>
    </div>
</div>

<div id="notification-area" class="notification-area"></div>
<div id="custom-alert" class="custom-alert">
    <p id="alert-message"></p>
    <button onclick="closeCustomAlert()">OK</button>
</div>
<div id="rank-badge" class="rank-badge">🏆 RANK 1</div>
<div id="gem-counter" class="gem-counter" style="display:none;">💎 GEMSTONES: <span id="gemstones-amount">0</span></div>
<div id="game-timer">⏱️ <span id="timer-display">00:00</span></div>

<!-- MAIN HUB -->
<div id="main-hub" class="overlay">
    <h1 class="hub-title" style="font-size:52px;margin-bottom:5px;color:#00d2ff;">✨ GALACTIC DEFENDER ✨</h1>
    <div class="hub-version-badge">UPDATE 11.6</div>
    <div class="hub-divider"></div>
    <div style="margin:15px 0;">
        <button class="hub-btn-hero" onclick="openGameSelect()">🚀 PLAY NOW</button>
    </div>
    <div style="display:flex;flex-wrap:wrap;justify-content:center;gap:10px;margin:10px;">
        <button class="hub-btn" onclick="openUpdateLog()" data-i18n="update_log">📜 UPDATE LOG</button>
        <button class="hub-btn" onclick="openTutorial()" data-i18n="how_to_play">🎮 HOW TO PLAY</button>
        <button class="hub-btn" style="border-color:#ffcc00;" onclick="openHomeAbilityPanel()">🎯 ABILITIES</button>
        <button class="hub-btn-reward" onclick="claimDailyReward()" data-i18n="daily_reward">🎁 DAILY REWARD</button>
    </div>
    <div class="hub-divider"></div>
    <div class="hub-section">
        <div class="hub-stat-card stat-record" style="margin:5px 0;font-size:14px;"><span>🏆</span> RECORD: <span id="hub-hi" style="color:gold;margin-left:auto;">0</span></div>
        <div class="hub-stat-card stat-credits" style="margin:5px 0;font-size:14px;"><span>💰</span> CREDITS: <span id="hub-coins" style="color:#00ffaa;margin-left:auto;">0</span></div>
        <div class="hub-stat-card stat-gems" style="margin:5px 0;font-size:14px;"><span>💎</span> GEMSTONES: <span id="hub-gems" style="color:#ff66ff;margin-left:auto;">0</span></div>
        <div class="hub-stat-card stat-kills" style="margin:5px 0;font-size:14px;"><span>💀</span> KILLS: <span id="hub-kills" style="color:#ff4444;margin-left:auto;">0</span></div>
        <div class="hub-stat-card stat-skin" style="margin:5px 0;font-size:14px;"><span>🌟</span> SKIN: <span id="hub-skin" style="color:gold;margin-left:auto;">LOCKED</span></div>
        <div id="hub-skin-progress-bar" style="width:100%;height:6px;background:#333;border-radius:3px;margin:8px auto;overflow:hidden;"><div id="hub-skin-progress-fill" style="height:100%;width:0%;background:linear-gradient(90deg,gold,#ffcc44);transition:width 0.3s;"></div></div>
        <div id="hub-skin-percent" style="font-size:10px;color:#aaa;">0/40 ACHIEVEMENTS</div>
    </div>
    <div id="streak-badge-container" style="text-align:center;margin:5px auto;"></div>
    <div class="stats-panel" style="width:90%;max-width:450px;margin:10px auto;">
        <div>🎯 CRITS: <span id="stat-crits">0</span> (<span id="stat-crit-rate">15</span>%)</div>
        <div>💥 DAMAGE: <span id="stat-damage">0</span></div>
        <div>⚡ OVERDRIVES: <span id="stat-od">0</span></div>
        <div>💀 BOSSES: <span id="stat-bosses">0</span></div>
    </div>
    <div class="hub-section">
        <h3 style="color:#ffaa00;margin-bottom:8px;">📋 DAILY MISSIONS</h3>
        <div id="daily-missions-container"></div>
    </div>
    <div class="hub-section" style="margin-bottom:30px;">
        <h3 style="color:#00d2ff;margin-bottom:8px;">📋 GAME INFO</h3>
        <p id="game-info-text" style="font-size:12px;color:#ccc;"><strong>🚀 GALACTIC DEFENDER:</strong> Space shooter with bosses, special events, upgrade system, achievements and more! Defend your ship and destroy all enemies.</p>
        <p id="game2-info-text" style="font-size:12px;color:#ccc;margin-top:8px;"><strong>❓ GAME 2 (COMING SOON):</strong> The second game is in advanced development! Expected soon with new and exciting mechanics. Stay tuned!</p>
        <p id="update-info-text" style="font-size:11px;color:#ffaa00;margin-top:8px;">✨ Update 11.6 - Skill Tree + Event Queue + New Events + Divine Mega Buff!</p>
    </div>
    <div style="color:#666;font-size:9px;margin-bottom:30px;">© Galactic Defender - All Rights Reserved</div>
</div>

<!-- HOME ABILITIES PANEL OVERLAY -->
<div id="home-abilities-panel" style="position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.85);z-index:500;display:none;justify-content:center;align-items:center;backdrop-filter:blur(6px);">
    <div id="home-abilities-content" style="background:linear-gradient(135deg,#001a33,#000);border:2px solid #00d2ff;border-radius:16px;padding:20px;min-width:300px;max-width:420px;max-height:85vh;overflow-y:auto;text-align:center;"></div>
</div>

<div id="game-select-screen" class="overlay">
    <h1 style="font-size:48px;margin-bottom:10px;">🎮 GAME SELECT</h1>
    <div style="display:flex;flex-wrap:wrap;justify-content:center;gap:20px;margin:20px;">
        <div class="game-select-card" onclick="selectGame('defender')">
            <div style="font-size:48px;">🚀</div>
            <h2 style="color:#00d2ff;">GALACTIC DEFENDER</h2>
            <p>המשחק הקלאסי! יריות, בוסים, אירועים והישגים</p>
            <p style="color:#00ffaa;font-size:12px;margin-top:10px;">▶ לחץ כדי לשחק</p>
            <div style="margin-top:8px;font-size:10px;color:#ffaa00;">✨ עדכון 10.5: 40 אירועים + 310+ הישגים + מערכת סקינים + פרס יומי!</div>
        </div>
        <div class="game-select-card coming-soon" onclick="showCustomAlert('משחק זה עדיין בפיתוח! יגיע בקרוב...')">
            <div style="font-size:48px;">❓</div>
            <h2 style="color:#888;">COMING SOON</h2>
            <p>משחק חדש בפיתוח... בקרוב!</p>
            <p style="color:#ffaa00;font-size:12px;margin-top:10px;">⏳ בשלבי הכנה אחרונים</p>
            <div style="margin-top:8px;font-size:10px;color:#888;">צפוי לצאת בקרוב עם מכניקות חדשות!</div>
        </div>
    </div>
    <button class="btn" style="border-color:#888;margin-bottom:30px;" onclick="backToHub()">← BACK TO HUB</button>
</div>

<!-- SETTINGS SCREEN (with Volume Sliders) -->
<div id="settings-screen" class="overlay">
    <h2 style="color:#00ffaa;" data-i18n="settings">⚙️ SETTINGS</h2>
    <div class="settings-option">
        <span data-i18n="notifications">🔔 NOTIFICATIONS</span>
        <div id="setting-notifications" class="settings-toggle on" onclick="toggleSetting('notifications')"></div>
    </div>
    <div class="settings-option" style="flex-direction:column;align-items:stretch;gap:6px;">
        <div style="display:flex;justify-content:space-between;align-items:center;"><span data-i18n="sfx_volume">🔊 SFX VOLUME</span><span id="sfx-vol-display" style="color:#00ffaa;font-size:11px;">80%</span></div>
        <input type="range" id="sfx-volume-slider" class="volume-slider" min="0" max="100" value="80" oninput="updateSfxVolume(this.value)">
    </div>
    <div class="settings-option" style="flex-direction:column;align-items:stretch;gap:6px;">
        <div style="display:flex;justify-content:space-between;align-items:center;"><span data-i18n="music_volume">🎵 MUSIC VOLUME</span><span id="music-vol-display" style="color:#ff66ff;font-size:11px;">50%</span></div>
        <input type="range" id="music-volume-slider" class="volume-slider" min="0" max="100" value="50" oninput="updateMusicVolume(this.value)" style="accent-color:#ff66ff;">
    </div>
    <div class="settings-option">
        <span data-i18n="screen_shake">📳 SCREEN SHAKE</span>
        <div id="setting-shake" class="settings-toggle on" onclick="toggleSetting('shake')"></div>
    </div>
    <div class="settings-option">
        <span data-i18n="auto_fire">🔫 AUTO FIRE</span>
        <div id="setting-autofire" class="settings-toggle" onclick="toggleSetting('autoFire')"></div>
    </div>
    <div class="settings-option">
        <span data-i18n="show_fps">📊 SHOW FPS</span>
        <div id="setting-showfps" class="settings-toggle" onclick="toggleSetting('showFPS')"></div>
    </div>
    <div class="settings-option">
        <span data-i18n="graphics_quality">🎨 GRAPHICS QUALITY</span>
        <div style="display:flex;gap:10px;">
            <button class="btn" style="padding:4px 12px;" onclick="setGraphicsQuality('low')" data-i18n="low">LOW</button>
            <button class="btn" style="padding:4px 12px;" onclick="setGraphicsQuality('medium')" data-i18n="medium">MED</button>
            <button class="btn" style="padding:4px 12px;" onclick="setGraphicsQuality('high')" data-i18n="high">HIGH</button>
        </div>
    </div>
    <div class="settings-option">
        <span data-i18n="crit_chance">🎯 CRITICAL HIT CHANCE</span>
        <span><span id="crit-chance-display">15</span>%</span>
    </div>
    <div class="settings-option">
        <span data-i18n="language">🌐 LANGUAGE</span>
        <div style="display:flex;gap:6px;">
            <button class="btn" style="padding:4px 12px;" onclick="setLanguage('en')">EN</button>
            <button class="btn" style="padding:4px 12px;" onclick="setLanguage('he')">עב</button>
            <button class="btn" style="padding:4px 12px;" onclick="setLanguage('ru')">РУ</button>
        </div>
    </div>
    <div class="controls-section">
        <h3 data-i18n="controls">🎮 CONTROLS</h3>
        <div class="control-row"><span>Pause</span><span class="control-key">ESC</span></div>
        <div class="control-row"><span>Bomb</span><span class="control-key">Q</span></div>
        <div class="control-row"><span>Overdrive</span><span class="control-key">O</span></div>
        <div class="control-row"><span>Abilities</span><span class="control-key">1-5</span></div>
        <div class="control-row"><span>Shoot</span><span class="control-key">CLICK</span></div>
        <div class="control-row"><span>Move Ship</span><span class="control-key">MOUSE/TOUCH</span></div>
    </div>
    <button class="btn" onclick="closeSettings()" style="margin-bottom:30px;">← BACK</button>
</div>

<div id="update-log-screen" class="overlay">
    <h2 style="color:#00ffaa;" data-i18n="update_log">📜 UPDATE LOG</h2>
    <div id="update-log-container" style="width:90%;max-width:600px;margin:10px auto;text-align:right;"></div>
    <button class="btn" onclick="backToHub()" style="margin-bottom:30px;">← BACK TO HUB</button>
</div>

<div id="start-screen" class="overlay">
    <h1>CORE</h1>
    <h2 style="color:#ff00ff;letter-spacing:4px;margin-bottom:8px;font-size:0.9rem;">RECOVERED</h2>
    <div class="start-stats">
        <div style="font-size:11px;">🏆 RECORD: <span id="menu-hi" style="color:gold">0</span></div>
        <div style="font-size:11px;">💰 CREDITS: <span id="menu-coins" style="color:#00ffaa">0</span></div>
        <div style="font-size:11px;">💀 KILLS: <span id="menu-kills" style="color:#ff4444">0</span></div>
        <div style="font-size:11px;">🌟 SKIN: <span id="skin-status-text" style="color:gold">LOCKED</span></div>
        <div id="skin-progress-bar"><div id="skin-progress-fill"></div></div>
        <div id="skin-percent" style="font-size:9px;color:#aaa;">0/40 ACHIEVEMENTS</div>
    </div>
    <button class="btn" onclick="startGame()" data-i18n="engage">🚀 ENGAGE</button>
    <button class="btn" style="border-color:#ff00ff;" onclick="openShop()" data-i18n="upgrades">⚙️ UPGRADES</button>
    <button class="btn" style="border-color:#ff66ff;" onclick="openRNGShop()" data-i18n="lootboxes">🎲 LOOTBOXES</button>
    <button class="btn" style="border-color:#00d2ff;" onclick="openTutorial()" data-i18n="how_to_play">🎮 HOW TO PLAY</button>
    <button class="btn" style="border-color:#888;margin-bottom:30px;" onclick="backToGameSelect()" data-i18n="back">← BACK</button>
    <div style="color:#888;font-size:8px;margin-top:8px;margin-bottom:20px;">MOUSE/TOUCH | ESC | Q bomb | O overdrive</div>
    <div id="ach-side-btn" class="side-btn" onclick="openAchievements()">🏆</div>
    <div id="events-side-btn" class="side-btn" onclick="openEvents()">📋</div>
    <div id="skins-side-btn" class="side-btn" onclick="openSkins()">🎨</div>
    <div id="settings-side-btn" class="side-btn" onclick="openSettings()">⚙️</div>
    <div id="skill-tree-side-btn" class="side-btn" style="top:280px;" onclick="openSkillTree()">🌳</div>
</div>

<div id="skins-screen" class="overlay">
    <h2 style="color:#ff66ff;" data-i18n="skins">🎨 SKIN COLLECTION 🎨</h2>
    <div class="skins-grid" id="skins-grid-container"></div>
    <button class="btn" onclick="closeSkins()" style="margin-bottom:30px;">← BACK</button>
</div>

<div id="skill-tree-screen" class="overlay">
    <h2 style="color:#00ffaa;">🌳 SKILL TREE 🌳</h2>
    <div style="font-size:13px;color:#ffcc00;margin:8px 0;">🌟 Skill Points: <span id="skill-points-display">0</span></div>
    <div style="font-size:10px;color:#aaa;margin-bottom:12px;">Earn 1 skill point every 5 levels or every 500 kills</div>
    <div style="display:flex;justify-content:center;flex-wrap:wrap;gap:12px;width:95%;max-width:500px;">
        <div class="skill-path" id="skill-path-offense"></div>
        <div class="skill-path" id="skill-path-defense"></div>
        <div class="skill-path" id="skill-path-utility"></div>
    </div>
    <button class="btn" onclick="closeSkillTree()" style="margin-bottom:30px;">← BACK</button>
</div>

<div id="rng-shop-screen" class="overlay">
    <h2 style="color:#ff66ff;" data-i18n="lootboxes">🎲 LOOTBOXES & REWARDS 🎲</h2>
    <div style="margin:10px;">💎 <span data-i18n="your_gemstones">YOUR GEMSTONES</span>: <span id="rng-gems" style="color:#ffcc00;font-size:24px;">0</span></div>
    <div class="shop-grid" style="grid-template-columns:1fr 1fr;max-width:500px;">
        <div class="card rarity-common" onclick="openLootbox('common')"><div style="font-size:20px;">📦</div><div>COMMON LOOTBOX</div><div style="color:#4287f5;">🔵 COMMON</div><div style="color:gold;">50 💎</div><div style="font-size:9px;color:#aaa;">פריטים נדירים בסיסיים</div><button class="btn" style="padding:2px 8px;font-size:8px;margin-top:4px;" onclick="event.stopPropagation();viewLootboxProbabilities('common')">📊 PROBS</button></div>
        <div class="card rarity-rare" onclick="openLootbox('rare')"><div style="font-size:20px;">📦</div><div>RARE LOOTBOX</div><div style="color:#42f5b6;">🟢 RARE</div><div style="color:gold;">150 💎</div><div style="font-size:9px;color:#aaa;">סיכוי לפריטים טובים יותר</div><button class="btn" style="padding:2px 8px;font-size:8px;margin-top:4px;" onclick="event.stopPropagation();viewLootboxProbabilities('rare')">📊 PROBS</button></div>
        <div class="card rarity-epic" onclick="openLootbox('epic')"><div style="font-size:20px;">📦</div><div>EPIC LOOTBOX</div><div style="color:#f5a742;">🟡 EPIC</div><div style="color:gold;">400 💎</div><div style="font-size:9px;color:#aaa;">פריטים אפיים!</div><button class="btn" style="padding:2px 8px;font-size:8px;margin-top:4px;" onclick="event.stopPropagation();viewLootboxProbabilities('epic')">📊 PROBS</button></div>
        <div class="card rarity-legendary" onclick="openLootbox('legendary')"><div style="font-size:20px;">📦</div><div>LEGENDARY LOOTBOX</div><div style="color:#f542d1;">🟠 LEGENDARY</div><div style="color:gold;">1000 💎</div><div style="font-size:9px;color:#aaa;">פריטים אגדיים!</div><button class="btn" style="padding:2px 8px;font-size:8px;margin-top:4px;" onclick="event.stopPropagation();viewLootboxProbabilities('legendary')">📊 PROBS</button></div>
        <div class="card rarity-mythic" onclick="openLootbox('mythic')"><div style="font-size:20px;">📦</div><div>MYTHIC LOOTBOX</div><div style="color:#f54242;">🔴 MYTHIC</div><div style="color:gold;">2500 💎</div><div style="font-size:9px;color:#aaa;">פריטים מיתיים נדירים!</div><button class="btn" style="padding:2px 8px;font-size:8px;margin-top:4px;" onclick="event.stopPropagation();viewLootboxProbabilities('mythic')">📊 PROBS</button></div>
        <div class="card rarity-ultra" onclick="openLootbox('ultra')"><div style="font-size:20px;">👑</div><div>ULTRA MYTHIC BOX</div><div style="color:#f5e642;">💎 ULTRA MYTHIC</div><div style="color:gold;">10000 💎</div><div style="font-size:9px;color:#aaa;">הפריטים הנדירים ביותר! כמות מוגבלת!</div><button class="btn" style="padding:2px 8px;font-size:8px;margin-top:4px;" onclick="event.stopPropagation();viewLootboxProbabilities('ultra')">📊 PROBS</button></div>
    </div>
    <h3 style="margin-top:20px;color:#ffaa00;" data-i18n="buy_rewards">🎁 BUY INDIVIDUAL REWARDS</h3>
    <div class="shop-grid" style="grid-template-columns:1fr 1fr;max-width:500px;" id="individual-rewards"></div>
    <button class="btn" onclick="closeRNGShop()" style="margin-bottom:30px;" data-i18n="back">← BACK</button>
</div>

<div id="lootbox-animation" class="lootbox-animation">
    <div id="lootbox-element" class="lootbox">📦</div>
    <div id="lootbox-result" class="reward-display" style="display:none;"></div>
    <button id="lootbox-close-btn" class="btn" style="margin-top:20px;display:none;" onclick="closeLootboxAnimation()">CLOSE</button>
</div>

<div id="ascend-screen" class="overlay">
    <h1 style="color:gold;">🌟 ASCENSION 🌟</h1>
    <p style="font-size:16px;margin:10px;">You have reached <span id="ascend-wave" style="color:#00ffaa;font-weight:bold;">100</span>!</p>
    <p>Do you wish to continue into <span style="color:#ffcc00;">ENDLESS MODE</span>?</p>
    <p style="font-size:12px;color:#aaa;">• Enemies get stronger every wave<br>• No more forced bosses<br>• Infinite progression<br>• Special achievements await!</p>
    <button class="btn btn-ascend" onclick="continueEndless()" data-i18n="yes_ascend">✨ YES, ASCEND ✨</button>
    <button class="btn" onclick="quitToMenu()" data-i18n="main_menu">BACK TO MENU</button>
</div>

<!-- GAME UI -->
<div id="combo-small">x1</div>
<div class="combo-meter" id="combo-meter"><div class="combo-meter-fill" id="combo-meter-fill"></div></div>
<div id="ui-hud">
    <div class="stat-group"><div class="stat-label">🛡️ <span id="hp-num"></span></div><div class="bar-bg"><div id="hp-fill" class="bar-fill"></div></div></div>
    <div class="stat-group"><div class="stat-label">⭐ XP</div><div class="bar-bg"><div id="xp-fill" class="bar-fill"></div></div></div>
    <div class="stat-group"><div class="stat-label">⚡ OD</div><div class="bar-bg"><div id="od-fill" class="bar-fill"></div></div></div>
    <div style="font-size:9px;">💣 <span id="bomb-count" style="color:#ffcc00;">0</span></div>
</div>
<div id="score-hud">
    <div class="score-main">SCORE: <span id="score-val">0</span></div>
    <div class="score-sub">RANK: <span id="lvl-val">1</span></div>
    <div class="score-sub">KILLS: <span id="kill-val">0</span></div>
    <div class="score-sub">WAVE: <span id="wave-val">1</span></div>
</div>
<div id="powerup-bar"></div>
<div id="wave-progress" class="wave-progress">
    <div class="wave-progress-text"><span id="wave-progress-text">0/12</span></div>
    <div class="wave-progress-bar"><div id="wave-progress-fill" class="wave-progress-fill" style="width:0%"></div></div>
</div>
<button id="auto-fire-btn" class="auto-fire-btn" onclick="toggleAutoFire()">🔫 AUTO</button>
<div id="fps-counter" class="fps-counter">60 FPS</div>
<div id="save-indicator" class="save-indicator">💾 SAVED</div>
<div id="reset-modal" class="reset-modal">
    <p class="reset-warning">⚠️ RESET UPGRADES</p>
    <p>This will reset ALL shop upgrades:</p>
    <p style="font-size:10px;color:#aaa;">Fire Rate, Damage, Shield, Drones, Spread, Laser, Bombs</p>
    <p>All invested credits will be refunded!</p>
    <p style="font-size:10px;color:#00ffaa;">Achievements, records & skins will remain.</p>
    <div style="display:flex;gap:10px;justify-content:center;margin-top:8px;">
        <button class="btn btn-danger" onclick="confirmResetAction()">🔄 CONFIRM RESET</button>
        <button class="btn" onclick="closeResetModal()">CANCEL</button>
    </div>
</div>

<!-- TUTORIAL SCREEN -->
<div id="tutorial-screen" class="tutorial-screen">
    <h2 data-i18n="how_to_play">🎮 HOW TO PLAY</h2>
    <div class="tutorial-row"><span>Move Ship</span><span class="tutorial-key">🖱️ MOUSE / TOUCH</span></div>
    <div class="tutorial-row"><span>Shoot</span><span class="tutorial-key">CLICK / TAP</span></div>
    <div class="tutorial-row"><span>Auto-Fire</span><span class="tutorial-key">🔫 BTN / SETTINGS</span></div>
    <div class="tutorial-row"><span>Bomb</span><span class="tutorial-key">Q</span></div>
    <div class="tutorial-row"><span>Overdrive</span><span class="tutorial-key">O</span></div>
    <div class="tutorial-row"><span>Ability 1</span><span class="tutorial-key">1</span></div>
    <div class="tutorial-row"><span>Ability 2</span><span class="tutorial-key">2</span></div>
    <div class="tutorial-row"><span>Ability 3</span><span class="tutorial-key">3</span></div>
    <div class="tutorial-row"><span>Ability 4</span><span class="tutorial-key">4</span></div>
    <div class="tutorial-row"><span>Ability 5</span><span class="tutorial-key">5</span></div>
    <div class="tutorial-row"><span>Pause</span><span class="tutorial-key">ESC</span></div>
    <p style="font-size:11px;color:#aaa;margin-top:15px;max-width:280px;">Destroy enemies, collect coins & gems, upgrade your ship, survive waves, and defeat bosses!</p>
    <button class="btn" onclick="closeTutorial()" style="margin-top:15px;">← BACK</button>
</div>

<div id="shop-screen" class="overlay">
    <h2 style="color:#00d2ff;font-size:1.3rem;" data-i18n="shop">🚀 GALACTIC DEFENDER - TECH HANGAR</h2>
    <div id="shop-money" style="color:gold;font-size:16px;">CREDITS: 0</div>
    <div class="shop-tabs">
        <button class="shop-tab active" onclick="switchShopTab('weapons')" data-i18n="weapons">🔫 Weapons</button>
        <button class="shop-tab" onclick="switchShopTab('shields')" data-i18n="shields">🛡️ Shields</button>
        <button class="shop-tab" onclick="switchShopTab('special')" data-i18n="special_abilities">✨ Special Abilities</button>
    </div>
    <div class="shop-grid" id="shop-grid-container"></div>
    <div id="shop-special-section"></div>
    <button class="btn" onclick="closeShop()" style="margin-bottom:30px;" data-i18n="back">← BACK</button>
    <div class="reset-section" style="margin-top:15px;margin-bottom:30px;">
        <button class="btn btn-danger" style="padding:5px 15px;font-size:10px;" onclick="confirmReset()" data-i18n="reset_upgrades">🔄 RESET UPGRADES (Refund)</button>
    </div>
</div>

<div id="achievements-screen" class="overlay">
    <h2 style="color:gold;" data-i18n="achievements">🏆 ACHIEVEMENTS (310+ TOTAL)</h2>
    <div class="ach-unlocked-count" id="ach-unlocked-text">🏆 <span id="ach-unlocked-num">0</span> / <span id="ach-total-num">0</span> UNLOCKED</div>
    <div class="ach-progress-bar-total"><div class="ach-progress-fill-total" id="ach-total-progress-fill"></div></div>
    <div class="ach-filter-tabs" id="ach-filter-tabs">
        <button class="ach-filter-tab active" onclick="filterAchievements('all')">ALL</button>
        <button class="ach-filter-tab" onclick="filterAchievements('easy')">EASY</button>
        <button class="ach-filter-tab" onclick="filterAchievements('medium')">MEDIUM</button>
        <button class="ach-filter-tab" onclick="filterAchievements('hard')">HARD</button>
        <button class="ach-filter-tab" onclick="filterAchievements('extreme')">EXTREME</button>
        <button class="ach-filter-tab" onclick="filterAchievements('mythic')">MYTHIC</button>
    </div>
    <div id="ach-list" class="achievements-grid"></div>
    <button class="btn" onclick="closeAchievements()" style="margin-bottom:30px;">BACK</button>
</div>

<div id="events-screen" class="overlay">
    <h2 style="color:#ff00ff;" data-i18n="events">📋 EVENT LIST (40 EVENTS)</h2>
    <div class="events-grid" id="events-list-container"></div>
    <button class="btn" onclick="closeEvents()" style="margin-bottom:30px;">BACK</button>
</div>

<div id="pause-screen" class="overlay">
    <h2 style="color:#00d2ff;" data-i18n="paused">PAUSED</h2>
    <button class="btn" onclick="togglePause()" data-i18n="resume">▶ RESUME (ESC)</button>
    <button class="btn" style="border-color:#00ffaa;" onclick="openPauseSettings()" data-i18n="settings">⚙️ SETTINGS</button>
    <button class="btn" style="border-color:#ff0044;" onclick="quitToMenu()" data-i18n="main_menu">MAIN MENU</button>
    <div style="margin-top:15px;display:flex;flex-direction:column;align-items:center;gap:4px;">
        <div class="pause-volume-row"><span>🔊 SFX</span><input type="range" id="pause-sfx-vol" class="volume-slider" min="0" max="100" value="80" oninput="updateSfxVolume(this.value)"><span id="pause-sfx-display" style="font-size:9px;color:#00ffaa;min-width:30px;">80%</span></div>
        <div class="pause-volume-row"><span>🎵 MUSIC</span><input type="range" id="pause-music-vol" class="volume-slider" min="0" max="100" value="50" oninput="updateMusicVolume(this.value)"><span id="pause-music-display" style="font-size:9px;color:#ff66ff;min-width:30px;">50%</span></div>
        <div class="pause-volume-row"><span>🔊 SFX ON</span><div id="pause-sfx-toggle" class="pause-quick-toggle on" onclick="toggleSetting('sound')"></div></div>
        <div class="pause-volume-row"><span>🎵 MUSIC ON</span><div id="pause-music-toggle" class="pause-quick-toggle music-on" onclick="toggleMusic()"></div></div>
    </div>
    <div style="font-size:9px;color:#666;margin-top:10px;">Press ESC to resume</div>
</div>
<div id="game-over" class="overlay">
    <h1 style="color:#ff0044;font-size:1.8rem;" data-i18n="mission_ended">MISSION ENDED</h1>
    <div id="final-stats" style="font-size:12px;margin-bottom:12px;"></div>
    <button class="btn" onclick="triggerReboot()" data-i18n="re_initialize">RE-INITIALIZE</button>
    <button class="btn" onclick="quitToMenu()" data-i18n="main_menu">MAIN MENU</button>
</div>

<canvas id="gameCanvas"></canvas>
<canvas id="nebulaCanvas"></canvas>
<div id="vignette"></div>
<div id="boss-warning">⚠ BOSS INCOMING ⚠</div>
<div id="wave-banner"></div>
<div id="event-banner"></div>
<div id="od-btn" onclick="activateOverdrive()">OVER<br>DRIVE</div>
<div id="ability-btns" class="ability-btn"></div>
<button id="pause-btn" onclick="togglePause()">⏸</button>
<div id="achievement-popup" class="achievement-popup-fixed">
    <div class="title">🏆 ACHIEVEMENT UNLOCKED</div>
    <div id="ach-name" class="name"></div>
    <div id="ach-desc" class="desc"></div>
</div>
<canvas id="crosshair"></canvas>

<script>
// ============================================
// INTERNATIONALIZATION SYSTEM
// ============================================
const TRANSLATIONS = {
    en: {
        title: "GALACTIC DEFENDER",
        play_now: "PLAY NOW",
        update_log: "UPDATE LOG",
        how_to_play: "HOW TO PLAY",
        daily_reward: "DAILY REWARD",
        settings: "SETTINGS",
        achievements: "ACHIEVEMENTS",
        shop: "TECH HANGAR",
        lootboxes: "LOOTBOXES",
        skins: "SKIN COLLECTION",
        events: "EVENT LIST",
        record: "RECORD",
        credits: "CREDITS",
        gemstones: "GEMSTONES",
        kills: "KILLS",
        skin: "SKIN",
        locked: "LOCKED",
        unlocked: "UNLOCKED",
        engage: "ENGAGE",
        upgrades: "UPGRADES",
        resume: "RESUME",
        main_menu: "MAIN MENU",
        paused: "PAUSED",
        mission_ended: "MISSION ENDED",
        re_initialize: "RE-INITIALIZE",
        notifications: "NOTIFICATIONS",
        sfx_volume: "SFX VOLUME",
        music_volume: "MUSIC VOLUME",
        screen_shake: "SCREEN SHAKE",
        auto_fire: "AUTO FIRE",
        show_fps: "SHOW FPS",
        graphics_quality: "GRAPHICS QUALITY",
        crit_chance: "CRITICAL HIT CHANCE",
        controls: "CONTROLS",
        language: "LANGUAGE",
        low: "LOW",
        medium: "MED",
        high: "HIGH",
        back: "BACK",
        select_game: "SELECT GAME",
        coming_soon: "COMING SOON",
        game_info: "GAME INFO",
        daily_missions: "DAILY MISSIONS",
        score: "SCORE",
        rank: "RANK",
        wave: "WAVE",
        hp: "HP",
        xp: "XP",
        od: "OD",
        bomb: "BOMB",
        overdrive: "OVERDRIVE",
        pause: "Pause",
        shoot: "Shoot",
        move_ship: "Move Ship",
        abilities: "Abilities",
        boss: "BOSS",
        guardian: "GUARDIAN",
        critical_hits: "CRITICAL HITS",
        total_damage: "TOTAL DAMAGE",
        overdrives: "OVERDRIVES",
        bosses_slain: "BOSSES SLAIN",
        game_info_desc: "Space shooter with bosses, special events, upgrade system, achievements and more! Defend your ship and destroy all enemies.",
        game2_desc: "The second game is in advanced development! Expected soon with new and exciting mechanics.",
        update_info: "Update 11.0 - Language System + Hub Redesign + Space Treasure Event + QOL!",
        classic_game_desc: "The classic game! Shooting, bosses, events and achievements",
        press_to_play: "Press to play",
        weapons: "Weapons",
        shields: "Shields",
        special_abilities: "Special Abilities",
        reset_upgrades: "RESET UPGRADES (Refund)",
        completed: "COMPLETED",
        new_event: "NEW!",
        space_treasure: "SPACE TREASURE",
        space_treasure_desc: "Treasure ships warp in dropping valuable loot! Collect before they drift away!",
        confirm_reset_msg: "This will reset ALL upgrades and refund your credits. Are you sure?",
        confirm_reset: "CONFIRM RESET",
        cancel: "CANCEL",
        warp_initiated: "WARP INITIATED",
        calibrating: "CALIBRATING...",
        core_failure: "CORE FAILURE",
        restoring: "RESTORING...",
        ascension: "ASCENSION",
        endless_mode: "ENDLESS MODE",
        yes_ascend: "YES, ASCEND",
        all: "ALL",
        easy: "EASY",
        hard: "HARD",
        extreme: "EXTREME",
        mythic: "MYTHIC",
        unlocked_count: "UNLOCKED",
        saving: "SAVED",
        lootbox_common: "COMMON LOOTBOX",
        lootbox_rare: "RARE LOOTBOX",
        lootbox_epic: "EPIC LOOTBOX",
        lootbox_legendary: "LEGENDARY LOOTBOX",
        lootbox_mythic: "MYTHIC LOOTBOX",
        lootbox_ultra: "ULTRA MYTHIC BOX",
        buy_rewards: "BUY INDIVIDUAL REWARDS",
        your_gemstones: "YOUR GEMSTONES",
        // Shop items
        shop_stardust: 'STARDUST',
        shop_stardust_desc: '+10% ALL STATS',
        shop_guardian: 'GUARDIAN ANGEL',
        shop_guardian_desc: 'Death immunity once',
        shop_powercore: 'POWER CORE',
        shop_powercore_desc: '+5% DAMAGE',
        shop_cosmic: 'COSMIC ESSENCE',
        shop_cosmic_desc: '+3% FIRE RATE',
        shop_ironwill: 'IRON WILL',
        shop_ironwill_desc: '50% DAMAGE REDUCTION',
        shop_timedistortion: 'TIME DISTORTION',
        shop_timedistortion_desc: '30% SLOW ENEMIES',
        shop_crystalheart: 'CRYSTAL HEART',
        shop_crystalheart_desc: '2x GEMSTONES',
        shop_max_charges: 'Maximum charges reached!',
        shop_purchased: 'purchased!',
        shop_not_enough: 'Not enough credits!',
        shop_remaining: 'remaining',
        shop_charges: 'charges',
        // Notifications
        noti_music_on: 'Music ON',
        noti_music_off: 'Music OFF',
        noti_setting_on: '{0} ON',
        noti_setting_off: '{0} OFF',
        noti_graphics: 'Graphics set to {0}',
        noti_autofire_on: 'AUTO FIRE ON',
        noti_autofire_off: 'AUTO FIRE OFF',
        noti_daily_reward: 'Daily Reward: +{0} GEMSTONES!',
        noti_daily_wait: 'Daily reward available in {0} hours',
        noti_mission_complete: 'Mission Complete: {0}! +{1} GEMSTONES!',
        noti_rank_up: 'RANK UP! Reached Rank {0}! +{1}% bonus!',
        noti_credits_saved: '+{0} credits saved!',
        noti_streak_claim: 'Day {0} streak! Reward claimed!',
        noti_overdrive: 'Overdrive activated!',
        noti_guardian_saved: 'Guardian Angel saved you! ({0}/3 left)',
        noti_reset: 'Upgrades reset! Refunded {0} credits',
        // Alerts
        alert_guardian_max: 'Maximum Guardian Angel charges (3/3)!',
        alert_guardian_bought: 'GUARDIAN ANGEL purchased! ({0}/3 charges)',
        alert_stardust_owned: 'STARDUST already purchased!',
        alert_stardust_bought: 'STARDUST acquired! +10% to all stats!',
        alert_not_enough_credits: 'Not enough credits! Need {0}, have {1}',
        alert_confirm_reset: 'This will reset ALL upgrades and refund your credits. Are you sure?',
        // HUD
        hud_score: 'SCORE',
        hud_rank: 'RANK',
        hud_kills_label: 'KILLS',
        hud_wave: 'WAVE',
        hud_bombs: 'BOMBS',
        // Game over stats
        go_new_record: 'NEW!',
        go_credits_earned: 'CREDITS: +{0}',
        // Streak
        streak_title: 'DAILY STREAK',
        streak_day: 'DAY STREAK',
        streak_claim: 'CLAIM REWARD',
        streak_claimed: 'Reward claimed!',
        streak_broken: 'Streak broken. Start a new one today!',
        streak_bonus_credits: '+{0} Credits',
        streak_bonus_gems: '+{0} Gemstones',
        streak_bonus_lootbox: '+1 Lootbox',
        // Misc
        remaining: 'remaining',
        of: 'of',
        owned: 'OWNED',
        equipped: 'EQUIPPED',
        select_game: 'SELECT GAME',
        back_to_hub: 'BACK TO HUB',
        close: 'CLOSE',
        open: 'OPEN',
        press_to_play: 'Press to play',
        classic_game_desc: 'The classic game! Shooting, bosses, events and achievements',
        coming_soon_desc: 'New game in development... Coming soon!',
        last_prep: 'In final preparation stages',
        new_mechanics: 'Coming soon with new mechanics!',
        still_dev: 'This game is still in development! Coming soon...',
        play_now: 'PLAY NOW',
        streak_dedicated: 'DEDICATED',
        streak_dedicated_desc: 'Play 3 days in a row',
        streak_committed: 'COMMITTED',
        streak_committed_desc: 'Play 7 days in a row',
        streak_loyal: 'LOYAL WARRIOR',
        streak_loyal_desc: 'Play 30 days in a row'
    },
    he: {
        title: "GALACTIC DEFENDER",
        play_now: "שחק עכשיו",
        update_log: "יומן עדכונים",
        how_to_play: "איך משחקים",
        daily_reward: "פרס יומי",
        settings: "הגדרות",
        achievements: "הישגים",
        shop: "האנגר טכני",
        lootboxes: "תיבות שלל",
        skins: "אוסף סקינים",
        events: "רשימת אירועים",
        record: "שיא",
        credits: "קרדיטים",
        gemstones: "אבני חן",
        kills: "הריגות",
        skin: "סקין",
        locked: "נעול",
        unlocked: "פתוח",
        engage: "התחל",
        upgrades: "שדרוגים",
        resume: "המשך",
        main_menu: "תפריט ראשי",
        paused: "מושהה",
        mission_ended: "המשימה הסתיימה",
        re_initialize: "אתחול מחדש",
        notifications: "התראות",
        sfx_volume: "עוצמת אפקטים",
        music_volume: "עוצמת מוזיקה",
        screen_shake: "רעד מסך",
        auto_fire: "ירי אוטומטי",
        show_fps: "הצג FPS",
        graphics_quality: "איכות גרפיקה",
        crit_chance: "סיכוי פגיעה קריטית",
        controls: "שליטה",
        language: "שפה",
        low: "נמוך",
        medium: "בינוני",
        high: "גבוה",
        back: "חזרה",
        select_game: "בחר משחק",
        coming_soon: "בקרוב",
        game_info: "מידע על המשחק",
        daily_missions: "משימות יומיות",
        score: "ניקוד",
        rank: "דרגה",
        wave: "גל",
        hp: "חיים",
        xp: "ניסיון",
        od: "אוברדרייב",
        bomb: "פצצה",
        overdrive: "אוברדרייב",
        pause: "השהה",
        shoot: "ירה",
        move_ship: "הזז חללית",
        abilities: "יכולות",
        boss: "בוס",
        guardian: "שומר",
        critical_hits: "פגיעות קריטיות",
        total_damage: "נזק כולל",
        overdrives: "אוברדרייבים",
        bosses_slain: "בוסים הובסו",
        game_info_desc: "משחק יריות בחלל עם בוסים, אירועים מיוחדים, מערכת שדרוגים, הישגים ועוד! הגן על החללית שלך והשמד את כל האויבים.",
        game2_desc: "המשחק השני נמצא בשלבי פיתוח מתקדמים! צפוי לצאת בקרוב עם מכניקות חדשות ומרגשות.",
        update_info: "עדכון 11.0 - מערכת שפות + עיצוב חדש של מסך הבית + אירוע אוצר חלל + שיפורי QOL!",
        classic_game_desc: "המשחק הקלאסי! יריות, בוסים, אירועים והישגים",
        press_to_play: "לחץ כדי לשחק",
        weapons: "נשקים",
        shields: "מגנים",
        special_abilities: "יכולות מיוחדות",
        reset_upgrades: "איפוס שדרוגים (החזר)",
        completed: "הושלם",
        new_event: "חדש!",
        space_treasure: "אוצר חלל",
        space_treasure_desc: "ספינות אוצר משתגרות ומפילות שלל יקר! אסוף לפני שהן נעלמות!",
        confirm_reset_msg: "זה יאפס את כל השדרוגים ויחזיר את הקרדיטים שלך. האם אתה בטוח?",
        confirm_reset: "אשר איפוס",
        cancel: "ביטול",
        warp_initiated: "מיקום הופעל",
        calibrating: "מכייל...",
        core_failure: "כשל ליבה",
        restoring: "משחזר...",
        ascension: "האצה",
        endless_mode: "מצב אינסופי",
        yes_ascend: "כן, האץ",
        all: "הכל",
        easy: "קל",
        hard: "קשה",
        extreme: "קיצוני",
        mythic: "מיתי",
        unlocked_count: "פתוחים",
        saving: "נשמר",
        lootbox_common: "תיבת שלל רגילה",
        lootbox_rare: "תיבת שלל נדירה",
        lootbox_epic: "תיבת שלל אפית",
        lootbox_legendary: "תיבת שלל אגדית",
        lootbox_mythic: "תיבת שלל מיתית",
        lootbox_ultra: "תיבת אולטרה מיתית",
        buy_rewards: "קנה פרסים בודדים",
        your_gemstones: "האבנים שלך",
        // Shop items
        shop_stardust: 'אבק כוכבים',
        shop_stardust_desc: '+10% לכל הסטטים',
        shop_guardian: 'מלאך שומר',
        shop_guardian_desc: 'חסינות למוות פעם אחת',
        shop_powercore: 'ליבת כוח',
        shop_powercore_desc: '+5% נזק',
        shop_cosmic: 'מהות קוסמית',
        shop_cosmic_desc: '+3% קצב ירי',
        shop_ironwill: 'רצון ברזל',
        shop_ironwill_desc: '50% הפחתת נזק',
        shop_timedistortion: 'עיוות זמן',
        shop_timedistortion_desc: '30% האטת אויבים',
        shop_crystalheart: 'לב קריסטל',
        shop_crystalheart_desc: '2x אבני חן',
        shop_max_charges: 'מקסימום טעינות!',
        shop_purchased: 'נרכש!',
        shop_not_enough: 'אין מספיק קרדיטים!',
        shop_remaining: 'נותרו',
        shop_charges: 'טעינות',
        // Notifications
        noti_music_on: 'מוזיקה פועלת',
        noti_music_off: 'מוזיקה כבויה',
        noti_setting_on: '{0} פועל',
        noti_setting_off: '{0} כבוי',
        noti_graphics: 'איכות גרפיקה: {0}',
        noti_autofire_on: 'ירי אוטומטי פועל',
        noti_autofire_off: 'ירי אוטומטי כבוי',
        noti_daily_reward: 'פרס יומי: +{0} אבני חן!',
        noti_daily_wait: 'פרס יומי זמין בעוד {0} שעות',
        noti_mission_complete: 'משימה הושלמה: {0}! +{1} אבני חן!',
        noti_rank_up: 'עלית דרגה! דרגה {0}! +{1}% בונוס!',
        noti_credits_saved: '+{0} קרדיטים נשמרו!',
        noti_streak_claim: 'רצף יום {0}! פרס נתבע!',
        noti_overdrive: 'אוברדרייב הופעל!',
        noti_guardian_saved: 'מלאך שומר הציל אותך! ({0}/3 נותרו)',
        noti_reset: 'שדרוגים אופסו! הוחזרו {0} קרדיטים',
        // Alerts
        alert_guardian_max: 'מקסימום טעינות מלאך שומר (3/3)!',
        alert_guardian_bought: 'מלאך שומר נרכש! ({0}/3 טעינות)',
        alert_stardust_owned: 'אבק כוכבים כבר נרכש!',
        alert_stardust_bought: 'אבק כוכבים נרכש! +10% לכל הסטטים!',
        alert_not_enough_credits: 'אין מספיק קרדיטים! צריך {0}, יש לך {1}',
        alert_confirm_reset: 'זה יאפס את כל השדרוגים ויחזיר את הקרדיטים שלך. האם אתה בטוח?',
        // HUD
        hud_score: 'ניקוד',
        hud_rank: 'דרגה',
        hud_kills_label: 'הריגות',
        hud_wave: 'גל',
        hud_bombs: 'פצצות',
        // Game over stats
        go_new_record: 'חדש!',
        go_credits_earned: 'קרדיטים: +{0}',
        // Streak
        streak_title: 'רצף יומי',
        streak_day: 'רצף ימים',
        streak_claim: 'תבע פרס',
        streak_claimed: 'הפרס נתבע!',
        streak_broken: 'הרצף נשבר. התחל רצף חדש היום!',
        streak_bonus_credits: '+{0} קרדיטים',
        streak_bonus_gems: '+{0} אבני חן',
        streak_bonus_lootbox: '+1 תיבת שלל',
        // Misc
        remaining: 'נותרו',
        of: 'מתוך',
        owned: 'בבעלות',
        equipped: 'מצויד',
        select_game: 'בחר משחק',
        back_to_hub: 'חזרה למסך הבית',
        close: 'סגור',
        open: 'פתח',
        press_to_play: 'לחץ כדי לשחק',
        classic_game_desc: 'המשחק הקלאסי! יריות, בוסים, אירועים והישגים',
        coming_soon_desc: 'משחק חדש בפיתוח... בקרוב!',
        last_prep: 'בשלבי הכנה אחרונים',
        new_mechanics: 'בקרוב עם מכניקות חדשות!',
        still_dev: 'משחק זה עדיין בפיתוח! יגיע בקרוב...',
        play_now: 'שחק עכשיו',
        streak_dedicated: 'מסור',
        streak_dedicated_desc: 'שחק 3 ימים ברצף',
        streak_committed: 'מחויב',
        streak_committed_desc: 'שחק 7 ימים ברצף',
        streak_loyal: 'לוחם נאמן',
        streak_loyal_desc: 'שחק 30 ימים ברצף'
    },
    ru: {
        title: "GALACTIC DEFENDER",
        play_now: "ИГРАТЬ",
        update_log: "Журнал обновлений",
        how_to_play: "Как играть",
        daily_reward: "Ежедневная награда",
        settings: "Настройки",
        achievements: "Достижения",
        shop: "Технический ангар",
        lootboxes: "Лутбоксы",
        skins: "Коллекция скинов",
        events: "Список событий",
        record: "Рекорд",
        credits: "Кредиты",
        gemstones: "Самоцветы",
        kills: "Убийства",
        skin: "Скин",
        locked: "Заблокирован",
        unlocked: "Разблокирован",
        engage: "В бой",
        upgrades: "Улучшения",
        resume: "Продолжить",
        main_menu: "Главное меню",
        paused: "Пауза",
        mission_ended: "Миссия окончена",
        re_initialize: "Перезапуск",
        notifications: "Уведомления",
        sfx_volume: "Громкость эффектов",
        music_volume: "Громкость музыки",
        screen_shake: "Тряска экрана",
        auto_fire: "Авто-огонь",
        show_fps: "Показать FPS",
        graphics_quality: "Качество графики",
        crit_chance: "Шанс критического удара",
        controls: "Управление",
        language: "Язык",
        low: "Низкое",
        medium: "Среднее",
        high: "Высокое",
        back: "Назад",
        select_game: "Выбрать игру",
        coming_soon: "Скоро",
        game_info: "Информация",
        daily_missions: "Ежедневные задания",
        score: "Счёт",
        rank: "Ранг",
        wave: "Волна",
        hp: "Здоровье",
        xp: "Опыт",
        od: "Овердрайв",
        bomb: "Бомба",
        overdrive: "Овердрайв",
        pause: "Пауза",
        shoot: "Стрелять",
        move_ship: "Двигать корабль",
        abilities: "Способности",
        boss: "Босс",
        guardian: "Страж",
        critical_hits: "Критические попадания",
        total_damage: "Общий урон",
        overdrives: "Овердрайвы",
        bosses_slain: "Боссов убито",
        game_info_desc: "Космический шутер с боссами, особыми событиями, системой улучшений, достижениями и многим другим! Защищайте свой корабль и уничтожайте всех врагов.",
        game2_desc: "Вторая игра находится в продвинутой разработке! Ожидайте скоро новые захватывающие механики.",
        update_info: "Обновление 11.0 - Система языков + Редизайн главного меню + Событие Космическое Сокровище + Улучшения!",
        classic_game_desc: "Классическая игра! Стрельба, боссы, события и достижения",
        press_to_play: "Нажмите чтобы играть",
        weapons: "Оружие",
        shields: "Щиты",
        special_abilities: "Особые способности",
        reset_upgrades: "Сбросить улучшения (Возврат)",
        completed: "Выполнено",
        new_event: "Новинка!",
        space_treasure: "Космическое Сокровище",
        space_treasure_desc: "Корабли с сокровищами появляются и сбрасывают ценную добычу! Собирайте пока не уплыли!",
        confirm_reset_msg: "Это сбросит ВСЕ улучшения и вернёт кредиты. Вы уверены?",
        confirm_reset: "Подтвердить сброс",
        cancel: "Отмена",
        warp_initiated: "Варп инициирован",
        calibrating: "Калибровка...",
        core_failure: "Отказ ядра",
        restoring: "Восстановление...",
        ascension: "Вознесение",
        endless_mode: "Бесконечный режим",
        yes_ascend: "Да, Вознестись",
        all: "Все",
        easy: "Легко",
        hard: "Сложно",
        extreme: "Экстремально",
        mythic: "Мифический",
        unlocked_count: "Разблокировано",
        saving: "Сохранено",
        lootbox_common: "Обычный лутбокс",
        lootbox_rare: "Редкий лутбокс",
        lootbox_epic: "Эпический лутбокс",
        lootbox_legendary: "Легендарный лутбокс",
        lootbox_mythic: "Мифический лутбокс",
        lootbox_ultra: "Ультра Мифический",
        buy_rewards: "Купить награды отдельно",
        your_gemstones: "Ваши самоцветы",
        // Shop items
        shop_stardust: 'ЗВЁЗДНАЯ ПЫЛЬ',
        shop_stardust_desc: '+10% ко всем характеристикам',
        shop_guardian: 'АНГЕЛ-ХРАНИТЕЛЬ',
        shop_guardian_desc: 'Один раз бессмертие при смерти',
        shop_powercore: 'СИЛОВОЕ ЯДРО',
        shop_powercore_desc: '+5% УРОН',
        shop_cosmic: 'КОСМИЧЕСКАЯ СУЩНОСТЬ',
        shop_cosmic_desc: '+3% СКОРОСТЬ СТРЕЛЬБЫ',
        shop_ironwill: 'ЖЕЛЕЗНАЯ ВОЛЯ',
        shop_ironwill_desc: '50% СНИЖЕНИЕ УРОНА',
        shop_timedistortion: 'ИСКАЖЕНИЕ ВРЕМЕНИ',
        shop_timedistortion_desc: '30% ЗАМЕДЛЕНИЕ ВРАГОВ',
        shop_crystalheart: 'КРИСТАЛЬНОЕ СЕРДЦЕ',
        shop_crystalheart_desc: '2x САМОЦВЕТЫ',
        shop_max_charges: 'Максимум зарядов!',
        shop_purchased: 'приобретено!',
        shop_not_enough: 'Недостаточно кредитов!',
        shop_remaining: 'осталось',
        shop_charges: 'зарядов',
        // Notifications
        noti_music_on: 'Музыка ВКЛ',
        noti_music_off: 'Музыка ВЫКЛ',
        noti_setting_on: '{0} ВКЛ',
        noti_setting_off: '{0} ВЫКЛ',
        noti_graphics: 'Качество графики: {0}',
        noti_autofire_on: 'АВТО-ОГОНЬ ВКЛ',
        noti_autofire_off: 'АВТО-ОГОНЬ ВЫКЛ',
        noti_daily_reward: 'Ежедневная награда: +{0} САМОЦВЕТОВ!',
        noti_daily_wait: 'Ежедневная награда доступна через {0} часов',
        noti_mission_complete: 'Задание выполнено: {0}! +{1} САМОЦВЕТОВ!',
        noti_rank_up: 'ПОВЫШЕНИЕ! Достигнут ранг {0}! +{1}% бонус!',
        noti_credits_saved: '+{0} кредитов сохранено!',
        noti_streak_claim: 'Серия день {0}! Награда получена!',
        noti_overdrive: 'Овердрайв активирован!',
        noti_guardian_saved: 'Ангел-хранитель спас вас! ({0}/3 осталось)',
        noti_reset: 'Улучшения сброшены! Возвращено {0} кредитов',
        // Alerts
        alert_guardian_max: 'Максимум зарядов Ангела-хранителя (3/3)!',
        alert_guardian_bought: 'Ангел-хранитель приобретён! ({0}/3 зарядов)',
        alert_stardust_owned: 'Звёздная пыль уже приобретена!',
        alert_stardust_bought: 'Звёздная пыль приобретена! +10% ко всем характеристикам!',
        alert_not_enough_credits: 'Недостаточно кредитов! Нужно {0}, у вас {1}',
        alert_confirm_reset: 'Это сбросит ВСЕ улучшения и вернёт кредиты. Вы уверены?',
        // HUD
        hud_score: 'СЧЁТ',
        hud_rank: 'РАНГ',
        hud_kills_label: 'УБИЙСТВА',
        hud_wave: 'ВОЛНА',
        hud_bombs: 'БОМБЫ',
        // Game over stats
        go_new_record: 'НОВЫЙ!',
        go_credits_earned: 'КРЕДИТЫ: +{0}',
        // Streak
        streak_title: 'ЕЖЕДНЕВНАЯ СЕРИЯ',
        streak_day: 'ДНЕЙ СЕРИЯ',
        streak_claim: 'ПОЛУЧИТЬ НАГРАДУ',
        streak_claimed: 'Награда получена!',
        streak_broken: 'Серия прервана. Начните новую сегодня!',
        streak_bonus_credits: '+{0} Кредитов',
        streak_bonus_gems: '+{0} Самоцветов',
        streak_bonus_lootbox: '+1 Лутбокс',
        // Misc
        remaining: 'осталось',
        of: 'из',
        owned: 'КУПЛЕНО',
        equipped: 'ЭКИПИРОВАНО',
        select_game: 'ВЫБРАТЬ ИГРУ',
        back_to_hub: 'В ГЛАВНОЕ МЕНЮ',
        close: 'ЗАКРЫТЬ',
        open: 'ОТКРЫТЬ',
        press_to_play: 'Нажмите чтобы играть',
        classic_game_desc: 'Классическая игра! Стрельба, боссы, события и достижения',
        coming_soon_desc: 'Новая игра в разработке... Скоро!',
        last_prep: 'В последних стадиях подготовки',
        new_mechanics: 'Скоро с новыми механиками!',
        still_dev: 'Эта игра ещё в разработке! Скоро...',
        play_now: 'ИГРАТЬ',
        streak_dedicated: 'ПРЕДАННЫЙ',
        streak_dedicated_desc: 'Играйте 3 дня подряд',
        streak_committed: 'НАСТОЙЧИВЫЙ',
        streak_committed_desc: 'Играйте 7 дней подряд',
        streak_loyal: 'ВЕРНЫЙ ВОИН',
        streak_loyal_desc: 'Играйте 30 дней подряд'
    }
};

function t(key){
    const lang = settings.language || 'en';
    return (TRANSLATIONS[lang] && TRANSLATIONS[lang][key]) || (TRANSLATIONS['en'] && TRANSLATIONS['en'][key]) || key;
}

function tn(key, type, ...args){
    let msg = t(key);
    args.forEach((arg, i) => { msg = msg.replace(new RegExp('\\{'+i+'\\}', 'g'), arg); });
    showNotification(msg, type);
}

function setLanguage(lang){
    settings.language = lang;
    localStorage.setItem('gameLanguage', lang);
    applyLanguage();
    document.querySelectorAll('.settings-option:last-of-type .btn').forEach(b => b.style.borderColor = '#00d2ff');
}

function applyLanguage(){
    const lang = settings.language || 'en';
    document.documentElement.lang = lang;
    document.documentElement.dir = (lang === 'he') ? 'rtl' : 'ltr';
    document.querySelectorAll('[data-i18n]').forEach(el => {
        const key = el.getAttribute('data-i18n');
        if(key) el.textContent = t(key);
    });
    document.querySelectorAll('[data-i18n-html]').forEach(el => {
        const key = el.getAttribute('data-i18n-html');
        if(key) el.innerHTML = t(key);
    });
    if(document.getElementById('game-info-text')) document.getElementById('game-info-text').innerHTML = '<strong>🚀 GALACTIC DEFENDER:</strong> ' + t('game_info_desc');
    if(document.getElementById('game2-info-text')) document.getElementById('game2-info-text').innerHTML = '<strong>❓ GAME 2 (COMING SOON):</strong> ' + t('game2_desc');
    if(document.getElementById('update-info-text')) document.getElementById('update-info-text').textContent = '✨ ' + t('update_info');
    if(typeof updateHubUI === 'function') updateHubUI();
    if(typeof updateMainMenuUI === 'function') updateMainMenuUI();
    if(typeof updateAchievementsUI === 'function') updateAchievementsUI();
    if(typeof updateEventsUI === 'function') updateEventsUI();
    if(typeof updateShopUI === 'function') updateShopUI();
    if(typeof updateDailyMissionsUI === 'function') updateDailyMissionsUI();

    // Update start screen buttons
    const engageBtn = document.querySelector('[onclick="startGame()"]');
    if(engageBtn) engageBtn.textContent = '🚀 ' + t('engage');
    const upgradeBtn = document.querySelector('[onclick="openShop()"]');
    if(upgradeBtn) upgradeBtn.textContent = '⚙️ ' + t('upgrades');
    const lootboxBtn = document.querySelector('[onclick="openRNGShop()"]');
    if(lootboxBtn) lootboxBtn.textContent = '🎲 ' + t('lootboxes');

    // Update pause screen
    const resumeBtn = document.querySelector('#pause-screen [onclick="togglePause()"]');
    if(resumeBtn) resumeBtn.textContent = t('resume');
    const pauseMenuBtn = document.querySelector('#pause-screen [onclick="quitToMenu()"]');
    if(pauseMenuBtn) pauseMenuBtn.textContent = t('main_menu');

    // Update game over
    const rebootBtn = document.querySelector('#game-over [onclick="triggerReboot()"]');
    if(rebootBtn) rebootBtn.textContent = t('re_initialize');
    const goMenuBtn = document.querySelector('#game-over [onclick="quitToMenu()"]');
    if(goMenuBtn) goMenuBtn.textContent = t('main_menu');

    // Update streak display
    const streakText = document.getElementById('streak-day-text');
    if(streakText) streakText.textContent = t('streak_day');
}

// ============================================
// MUSIC SYSTEM - Background Music
// ============================================
let backgroundMusic = null;
let musicEnabled = localStorage.getItem('musicEnabled') !== 'false';
let currentMusicType = 'menu';
let musicVolume = (parseInt(localStorage.getItem('musicVolume')) || 50) / 100 * 0.24;
let musicLoopInterval = null;
let musicStopRequested = false;

function stopBackgroundMusic() {
    musicStopRequested = true;
    if (backgroundMusic) {
        try {
            if (backgroundMusic.osc1) { backgroundMusic.osc1.stop(); backgroundMusic.osc1.disconnect(); }
            if (backgroundMusic.osc2) { backgroundMusic.osc2.stop(); backgroundMusic.osc2.disconnect(); }
            if (backgroundMusic.osc) { backgroundMusic.osc.stop(); backgroundMusic.osc.disconnect(); }
            if (backgroundMusic.masterGain) backgroundMusic.masterGain.disconnect();
        } catch(e) { console.log("Music stop error:", e); }
        backgroundMusic = null;
    }
    if (musicLoopInterval) {
        clearInterval(musicLoopInterval);
        musicLoopInterval = null;
    }
    setTimeout(() => { musicStopRequested = false; }, 100);
}

function playBackgroundMusic(type) {
    if (!musicEnabled) return;
    if (musicStopRequested) return;
    if (!audioCtx && settings.sound) {
        try {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        } catch(e) { return; }
    }
    if (!audioCtx) return;
    if (audioCtx.state === 'suspended') {
        audioCtx.resume().catch(e => console.log("AudioContext resume failed"));
    }
    if (currentMusicType === type && backgroundMusic && backgroundMusic.masterGain) {
        return;
    }
    stopBackgroundMusic();
    currentMusicType = type;
    try {
        const now = audioCtx.currentTime;
        const masterGain = audioCtx.createGain();
        masterGain.gain.value = musicVolume;
        masterGain.connect(audioCtx.destination);
        switch(type) {
            case 'menu':
                const osc1 = audioCtx.createOscillator();
                const osc2 = audioCtx.createOscillator();
                const gain1 = audioCtx.createGain();
                const gain2 = audioCtx.createGain();
                osc1.connect(gain1);
                osc2.connect(gain2);
                gain1.connect(masterGain);
                gain2.connect(masterGain);
                osc1.type = 'sine';
                osc2.type = 'sine';
                osc1.frequency.value = 174.61;
                osc2.frequency.value = 261.63;
                gain1.gain.setValueAtTime(0, now);
                gain2.gain.setValueAtTime(0, now);
                gain1.gain.linearRampToValueAtTime(0.07, now + 1);
                gain2.gain.linearRampToValueAtTime(0.05, now + 1.5);
                osc1.start();
                osc2.start();
                backgroundMusic = { osc1, osc2, gain1, gain2, masterGain };
                let menuLFO = setInterval(() => {
                    if (!backgroundMusic || !musicEnabled || musicStopRequested) return;
                    try {
                        const freq1 = 174.61 + Math.sin(Date.now() / 6000) * 2;
                        const freq2 = 261.63 + Math.sin(Date.now() / 7000) * 1.5;
                        if (backgroundMusic.osc1) backgroundMusic.osc1.frequency.value = freq1;
                        if (backgroundMusic.osc2) backgroundMusic.osc2.frequency.value = freq2;
                    } catch(e) {}
                }, 500);
                musicLoopInterval = menuLFO;
                break;
            case 'gameplay':
                const gOsc = audioCtx.createOscillator();
                const gGain = audioCtx.createGain();
                gOsc.connect(gGain);
                gGain.connect(masterGain);
                gOsc.type = 'sawtooth';
                gOsc.frequency.value = 130.81;
                gGain.gain.setValueAtTime(0, now);
                gGain.gain.linearRampToValueAtTime(0.06, now + 1);
                gOsc.start();
                backgroundMusic = { osc: gOsc, gain: gGain, masterGain };
                let gameLFO = setInterval(() => {
                    if (!backgroundMusic || !musicEnabled || musicStopRequested) return;
                    try {
                        const notes = [130.81, 146.83, 164.81, 146.83];
                        const idx = Math.floor(Date.now() / 350) % notes.length;
                        if (backgroundMusic.osc) backgroundMusic.osc.frequency.value = notes[idx];
                    } catch(e) {}
                }, 350);
                musicLoopInterval = gameLFO;
                break;
            case 'boss':
                const bOsc = audioCtx.createOscillator();
                const bGain = audioCtx.createGain();
                bOsc.connect(bGain);
                bGain.connect(masterGain);
                bOsc.type = 'square';
                bOsc.frequency.value = 87.31;
                bGain.gain.setValueAtTime(0, now);
                bGain.gain.linearRampToValueAtTime(0.09, now + 0.5);
                bOsc.start();
                backgroundMusic = { osc: bOsc, gain: bGain, masterGain };
                let bossLFO = setInterval(() => {
                    if (!backgroundMusic || !musicEnabled || musicStopRequested) return;
                    try {
                        const pulse = 87.31 + (Math.sin(Date.now() / 250) * 8);
                        if (backgroundMusic.osc) backgroundMusic.osc.frequency.value = pulse;
                    } catch(e) {}
                }, 150);
                musicLoopInterval = bossLFO;
                break;
            case 'event':
                const eOsc = audioCtx.createOscillator();
                const eGain = audioCtx.createGain();
                eOsc.connect(eGain);
                eGain.connect(masterGain);
                eOsc.type = 'triangle';
                eOsc.frequency.value = 220.00;
                eGain.gain.setValueAtTime(0, now);
                eGain.gain.linearRampToValueAtTime(0.07, now + 0.8);
                eOsc.start();
                backgroundMusic = { osc: eOsc, gain: eGain, masterGain };
                let eventLFO = setInterval(() => {
                    if (!backgroundMusic || !musicEnabled || musicStopRequested) return;
                    try {
                        const sweep = 220 + Math.sin(Date.now() / 400) * 20;
                        if (backgroundMusic.osc) backgroundMusic.osc.frequency.value = sweep;
                    } catch(e) {}
                }, 200);
                musicLoopInterval = eventLFO;
                break;
        }
    } catch(e) { console.log("Music error:", e); }
}

function updateMusicBasedOnGameState() {
    if (!musicEnabled) { stopBackgroundMusic(); return; }
    if (gameState !== 'PLAYING') { playBackgroundMusic('menu'); return; }
    if (boss !== null || guardian !== null) { playBackgroundMusic('boss'); }
    else if (activeEvent && (apocalypseActive || cosmicCollapseActive || primordialRageActive || voidActive)) { playBackgroundMusic('event'); }
    else { playBackgroundMusic('gameplay'); }
}

function toggleMusic() {
    musicEnabled = !musicEnabled;
    localStorage.setItem('musicEnabled', musicEnabled);
    const toggle = document.getElementById('setting-music');
    if (toggle) toggle.classList.toggle('on', musicEnabled);
    const pauseToggle = document.getElementById('pause-music-toggle');
    if (pauseToggle) pauseToggle.classList.toggle('music-on', musicEnabled);
    if (musicEnabled) {
        updateMusicBasedOnGameState();
        tn(musicEnabled ? 'noti_music_on' : 'noti_music_off', 'success');
    } else {
        stopBackgroundMusic();
        tn('noti_music_off', 'info');
    }
}

// ============================================
// SETTINGS SYSTEM
// ============================================
let settings = {
    notifications: localStorage.getItem('notificationsEnabled') !== 'false',
    sound: localStorage.getItem('soundEnabled') !== 'false',
    shake: localStorage.getItem('shakeEnabled') !== 'false',
    graphics: localStorage.getItem('graphicsQuality') || 'high',
    autoFire: localStorage.getItem('autoFireEnabled') === 'true',
    sfxVolume: parseInt(localStorage.getItem('sfxVolume')) || 80,
    musicVolume: parseInt(localStorage.getItem('musicVolume')) || 50,
    showFPS: localStorage.getItem('showFPSEnabled') === 'true',
    language: localStorage.getItem('gameLanguage') || 'en'
};

function initSettingsUI(){
    const notifToggle = document.getElementById('setting-notifications');
    const soundToggle = document.getElementById('setting-sound');
    const shakeToggle = document.getElementById('setting-shake');
    const musicToggle = document.getElementById('setting-music');
    const autofireToggle = document.getElementById('setting-autofire');
    const showfpsToggle = document.getElementById('setting-showfps');
    if(notifToggle) notifToggle.classList.toggle('on', settings.notifications);
    if(shakeToggle) shakeToggle.classList.toggle('on', settings.shake);
    if(autofireToggle) autofireToggle.classList.toggle('on', settings.autoFire);
    if(showfpsToggle) showfpsToggle.classList.toggle('on', settings.showFPS);
    // Volume sliders
    const sfxSlider = document.getElementById('sfx-volume-slider');
    const musicSlider = document.getElementById('music-volume-slider');
    if(sfxSlider){ sfxSlider.value = settings.sfxVolume; }
    if(musicSlider){ musicSlider.value = settings.musicVolume; }
    updateSfxDisplay();
    updateMusicVolDisplay();
}

function toggleSetting(setting){
    settings[setting] = !settings[setting];
    localStorage.setItem(setting + 'Enabled', settings[setting]);
    const toggle = document.getElementById('setting-' + setting);
    if(toggle) toggle.classList.toggle('on', settings[setting]);
    // Handle autoFire toggle UI during gameplay
    if(setting === 'autoFire'){
        const afBtn = document.getElementById('auto-fire-btn');
        if(afBtn) afBtn.classList.toggle('active', settings.autoFire);
    }
    if(setting === 'showFPS'){
        const fpsEl = document.getElementById('fps-counter');
        if(fpsEl) fpsEl.style.display = settings.showFPS ? 'block' : 'none';
    }
    if(setting === 'sound' && !settings.sound && audioCtx){}
    tn(settings[setting] ? 'noti_setting_on' : 'noti_setting_off', 'info', setting.toUpperCase());
}

function updateSfxVolume(val){
    settings.sfxVolume = parseInt(val);
    localStorage.setItem('sfxVolume', settings.sfxVolume);
    updateSfxDisplay();
    // Update pause screen slider too
    const pauseSlider = document.getElementById('pause-sfx-vol');
    if(pauseSlider) pauseSlider.value = settings.sfxVolume;
}
function updateSfxDisplay(){
    const el = document.getElementById('sfx-vol-display');
    const pEl = document.getElementById('pause-sfx-display');
    if(el) el.textContent = settings.sfxVolume + '%';
    if(pEl) pEl.textContent = settings.sfxVolume + '%';
}
function updateMusicVolume(val){
    settings.musicVolume = parseInt(val);
    localStorage.setItem('musicVolume', settings.musicVolume);
    musicVolume = settings.musicVolume / 100 * 0.24; // max masterGain ~0.24
    if(backgroundMusic && backgroundMusic.masterGain){
        backgroundMusic.masterGain.gain.value = musicVolume;
    }
    updateMusicVolDisplay();
    // Update pause screen slider too
    const pauseSlider = document.getElementById('pause-music-vol');
    if(pauseSlider) pauseSlider.value = settings.musicVolume;
}
function updateMusicVolDisplay(){
    const el = document.getElementById('music-vol-display');
    const pEl = document.getElementById('pause-music-display');
    if(el) el.textContent = settings.musicVolume + '%';
    if(pEl) pEl.textContent = settings.musicVolume + '%';
}

function setGraphicsQuality(quality){
    settings.graphics = quality;
    localStorage.setItem('graphicsQuality', quality);
    tn('noti_graphics', 'info', quality.toUpperCase());
    window.graphicsQuality = quality;
}

function openSettings(){
    // Detect which screen is currently visible
    if(document.getElementById('pause-screen').style.display === 'flex'){
        settingsOpenedFrom = 'pause-screen';
        document.getElementById('pause-screen').style.display='none';
    } else {
        settingsOpenedFrom = 'start-screen';
        document.getElementById('start-screen').style.display='none';
    }
    document.getElementById('settings-screen').style.display='flex';
    initSettingsUI();
}
let settingsOpenedFrom = 'start-screen';

function closeSettings(){
    document.getElementById('settings-screen').style.display='none';
    const targetScreen = settingsOpenedFrom;
    settingsOpenedFrom = 'start-screen'; // Reset to default after use
    document.getElementById(targetScreen).style.display='flex';
    updateMainMenuUI();
}

// AUTO-FIRE
function toggleAutoFire(){
    settings.autoFire = !settings.autoFire;
    localStorage.setItem('autoFireEnabled', settings.autoFire);
    const btn = document.getElementById('auto-fire-btn');
    if(btn) btn.classList.toggle('active', settings.autoFire);
    const toggle = document.getElementById('setting-autofire');
    if(toggle) toggle.classList.toggle('on', settings.autoFire);
    showNotification(`AUTO FIRE ${settings.autoFire ? 'ON' : 'OFF'}`, 'info');
    tn(settings.autoFire ? 'noti_autofire_on' : 'noti_autofire_off', 'info');
}

// TUTORIAL
function openTutorial(){
    const prev = document.getElementById('start-screen').style.display === 'flex' ? 'start-screen' :
                 document.getElementById('pause-screen').style.display === 'flex' ? 'pause-screen' :
                 document.getElementById('main-hub').style.display === 'flex' ? 'main-hub' : 'start-screen';
    document.getElementById('tutorial-screen').style.display = 'flex';
    document.getElementById('tutorial-screen').dataset.prevScreen = prev;
    if(prev === 'start-screen') document.getElementById('start-screen').style.display = 'none';
}
function closeTutorial(){
    const prev = document.getElementById('tutorial-screen').dataset.prevScreen || 'start-screen';
    document.getElementById('tutorial-screen').style.display = 'none';
    const el = document.getElementById(prev);
    if(el) el.style.display = 'flex';
}

// PAUSE SETTINGS
function openPauseSettings(){
    settingsOpenedFrom = 'pause-screen';
    document.getElementById('pause-screen').style.display='none';
    document.getElementById('settings-screen').style.display='flex';
    initSettingsUI();
}
// closeSettings uses settingsOpenedFrom to return to the correct screen

// FPS COUNTER
let fpsFrameCount = 0;
let fpsLastTime = performance.now();
let currentFPS = 60;
function updateFPSCounter(){
    fpsFrameCount++;
    const now = performance.now();
    const delta = now - fpsLastTime;
    if(delta >= 500){
        currentFPS = Math.round((fpsFrameCount / delta) * 1000);
        fpsFrameCount = 0;
        fpsLastTime = now;
        const el = document.getElementById('fps-counter');
        if(el && settings.showFPS) el.textContent = currentFPS + ' FPS';
    }
}


// ENEMY HEALTH BARS (drawn on main canvas)
function drawEnemyHPBar(e){
    if(e.hp >= e.maxHp && !e.isBoss) return; // Only show when damaged
    const barW = e.isBoss ? 100 : 30;
    const barH = e.isBoss ? 8 : 4;
    const pct = Math.max(0, e.hp / e.maxHp);
    const hpColor = pct > 0.5 ? `hsl(${pct*120},100%,50%)` : pct > 0.25 ? '#ffaa00' : '#ff2200';
    ctx.fillStyle = '#222';
    ctx.fillRect(e.x - barW/2, e.y - e.r - (e.isBoss ? 30 : 12), barW, barH);
    ctx.fillStyle = hpColor;
    ctx.fillRect(e.x - barW/2, e.y - e.r - (e.isBoss ? 30 : 12), barW * pct, barH);
}

// BOSS HEALTH BAR AT TOP OF SCREEN
function drawBossHPBarTop(){
    if(!boss && !guardian) return;
    const target = boss || guardian;
    if(!target) return;
    const barW = Math.min(300, width * 0.6);
    const barH = 10;
    const x = (width - barW) / 2;
    const y = 75;
    const pct = Math.max(0, target.hp / target.maxHp);
    const hpColor = pct > 0.5 ? `hsl(${pct*120},100%,50%)` : pct > 0.25 ? '#ffaa00' : '#ff2200';
    ctx.fillStyle = 'rgba(0,0,0,0.6)';
    ctx.fillRect(x, y, barW, barH);
    ctx.fillStyle = hpColor;
    ctx.fillRect(x, y, barW * pct, barH);
    ctx.strokeStyle = '#fff';
    ctx.lineWidth = 1;
    ctx.strokeRect(x, y, barW, barH);
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 9px Segoe UI';
    ctx.textAlign = 'center';
    ctx.fillText((boss ? '⚠ BOSS' : '👑 GUARDIAN') + ' HP: ' + Math.ceil(target.hp) + '/' + target.maxHp, width/2, y + barH + 14);
    ctx.textAlign = 'start';
}

// Override showNotification based on settings
const originalShowNotification = window.showNotification || function(){};
window.showNotification = function(msg, type){
    if(settings.notifications) originalShowNotification(msg, type);
};

// ============================================
// CRITICAL HITS SYSTEM
// ============================================
let criticalHitsCount = 0;
let critChance = 0.15;
let critDamageMultiplier = 2;
let totalDamageDealt = 0;

function isCriticalHit(){
    return Math.random() < critChance;
}

function applyCriticalHit(power){
    if(isCriticalHit()){
        criticalHitsCount++;
        totalDamageDealt += power * critDamageMultiplier;
        if(player && player.x && player.y) floats.push({txt:'⚡ CRITICAL!', x:player.x-40, y:player.y-50, l:1, c:'#ffaa00', size:20});
        return power * critDamageMultiplier;
    }
    totalDamageDealt += power;
    return power;
}

// ============================================
// COMBO METER
// ============================================
function updateComboMeter(){
    const meter = document.getElementById('combo-meter');
    const fill = document.getElementById('combo-meter-fill');
    if(!meter) return;
    if(combo > 1){
        meter.style.display = 'block';
        let percent = Math.min(100, (combo / 50) * 100);
        fill.style.width = percent + '%';
        if(combo >= 10) fill.style.background = '#ff6600';
        else if(combo >= 5) fill.style.background = '#ffaa00';
        else fill.style.background = '#ffcc00';
    } else {
        meter.style.display = 'none';
    }
}

// ============================================
// NOTIFICATION SYSTEM
// ============================================
function showNotification(message, type = 'info'){
    const area = document.getElementById('notification-area');
    if(!area) return;
    const notif = document.createElement('div');
    notif.className = `notification notification-${type}`;
    notif.innerHTML = message;
    area.appendChild(notif);
    setTimeout(() => {
        if(notif && notif.remove) notif.remove();
    }, 5000);
}

function showCustomAlert(message){
    const alertDiv = document.getElementById('custom-alert');
    document.getElementById('alert-message').innerText = message;
    alertDiv.style.display = 'flex';
}
function closeCustomAlert(){
    document.getElementById('custom-alert').style.display = 'none';
}

// AUTO-SAVE
let lastSaveIndicatorTime = 0;
function autoSave(){
    localStorage.setItem('totalCoins', totalCoins);
    localStorage.setItem('hiScore', hiScore);
    localStorage.setItem('totalKills', totalKills);
    localStorage.setItem('fireLevel', fireLevel);
    localStorage.setItem('dmgLevel', damageLevel);
    localStorage.setItem('droneCount', droneCount);
    localStorage.setItem('bombCount', bombCount);
    localStorage.setItem('resourceCollectionLevel', resourceCollectionLevel);
    localStorage.setItem('weaponEnhancementLevel', weaponEnhancementLevel);
    localStorage.setItem('gemstones', gemstones);
    localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
    localStorage.setItem('achievements', JSON.stringify(achievements));
    showSaveIndicator();
}
setInterval(autoSave, 30000);

function showSaveIndicator(){
    const now = Date.now();
    if(now - lastSaveIndicatorTime < 5000) return; // Max once per 5 seconds
    lastSaveIndicatorTime = now;
    const el = document.getElementById('save-indicator');
    if(!el) return;
    el.classList.add('show');
    setTimeout(() => el.classList.remove('show'), 1200);
}

// DAILY REWARD
let lastDailyClaim = localStorage.getItem('lastDailyClaim') || 0;
let dailyClaimCount = parseInt(localStorage.getItem('dailyClaimCount')) || 0;
function claimDailyReward(){
    const now = Date.now();
    const last = parseInt(lastDailyClaim);
    const hoursSince = (now - last) / (1000 * 60 * 60);
    if(last === 0 || hoursSince >= 24){
        const reward = 100 + Math.floor(Math.random() * 400);
        gemstones += reward;
        saveGemstones();
        lastDailyClaim = now;
        dailyClaimCount++;
        localStorage.setItem('lastDailyClaim', now);
        localStorage.setItem('dailyClaimCount', dailyClaimCount);
        showCustomAlert(`🎁 זכית ב-${reward} GEMSTONES! 🎁\nבוא שוב מחר לפרס נוסף! (${dailyClaimCount} total claims)`);
        tn('noti_daily_reward', 'success', reward);
        updateHubUI();
        checkAchievements();
    } else {
        const remaining = Math.ceil(24 - hoursSince);
        showCustomAlert(`⏳ הפרס היומי כבר נאסף! תוכל לקחת שוב בעוד ${remaining} שעות.`);
        showNotification(`⏳ Daily reward available in ${remaining} hours`, 'warning');
    }
}

// DAILY MISSIONS
let dailyMissions; try { dailyMissions = JSON.parse(localStorage.getItem('dailyMissions')) || []; } catch(e) { dailyMissions = []; }
let lastMissionReset = localStorage.getItem('lastMissionReset') || 0;

function resetDailyMissions(){
    const now = Date.now();
    const last = parseInt(lastMissionReset);
    const hoursSince = (now - last) / (1000 * 60 * 60);
    if(last === 0 || hoursSince >= 24){
        dailyMissions = [
            { id: 0, name: "KILL ENEMIES", desc: "הרוג 100 אויבים", current: 0, target: 100, reward: 200, completed: false, type: "kill" },
            { id: 1, name: "COLLECT GEMS", desc: "אסוף 500 GEMSTONES", current: 0, target: 500, reward: 300, completed: false, type: "gem" },
            { id: 2, name: "PERFECT WAVES", desc: "השלם 5 גלים בלי נזק", current: 0, target: 5, reward: 250, completed: false, type: "perfect" },
            { id: 3, name: "CRITICAL HITS", desc: "בצע 50 פגיעות קריטיות", current: 0, target: 50, reward: 350, completed: false, type: "crit" }
        ];
        lastMissionReset = now;
        localStorage.setItem('lastMissionReset', now);
        localStorage.setItem('dailyMissions', JSON.stringify(dailyMissions));
    }
}

function updateDailyMissionProgress(type, amount = 1){
    for(let m of dailyMissions){
        if(!m.completed && m.type === type){
            m.current += amount;
            if(m.current >= m.target){
                m.completed = true;
                gemstones += m.reward;
                saveGemstones();
                tn('noti_mission_complete', 'success', m.name, m.reward);
            }
        }
    }
    localStorage.setItem('dailyMissions', JSON.stringify(dailyMissions));
    updateDailyMissionsUI();
}

function updateDailyMissionsUI(){
    const container = document.getElementById('daily-missions-container');
    if(!container) return;
    container.innerHTML = '';
    for(const m of dailyMissions){
        const percent = (m.current / m.target) * 100;
        const div = document.createElement('div');
        div.className = 'daily-mission-card';
        div.innerHTML = `
            <div style="display:flex;justify-content:space-between;">
                <span style="color:#ffaa00;">${m.name}</span>
                <span>${m.completed ? '✅' : '⏳'}</span>
            </div>
            <div style="font-size:10px;">${m.desc}</div>
            <div class="ach-progress-bar" style="margin-top:5px;"><div class="ach-progress-fill" style="width:${percent}%;background:#ffaa00;"></div></div>
            <div style="font-size:9px;">${m.current}/${m.target} 🎁 ${m.reward}💎</div>
        `;
        container.appendChild(div);
    }
}

// RANK SYSTEM
let currentRank = parseInt(localStorage.getItem('currentRank')) || 1;
let rankXP = parseInt(localStorage.getItem('rankXP')) || 0;
const RANK_REQUIREMENTS = [0, 100, 250, 500, 1000, 2000, 3500, 5500, 8000, 11000, 15000, 20000, 26000, 33000, 41000, 50000];
const RANK_BONUS = [0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 25, 28, 32, 36, 40, 44, 48, 52, 56, 60, 65, 70, 75, 80, 85, 90, 95, 100, 105, 110, 115, 120, 125, 130, 135, 140, 145, 150, 160, 170, 180, 190, 200, 210, 220, 230, 240, 250];

function addRankXP(amount){
    rankXP += amount;
    let req = RANK_REQUIREMENTS[currentRank] || (currentRank * 500);
    while(rankXP >= req && currentRank < 50){
        rankXP -= req;
        currentRank++;
        req = RANK_REQUIREMENTS[currentRank] || (currentRank * 500);
        tn('noti_rank_up', 'success', currentRank, RANK_BONUS[currentRank]);
    }
    localStorage.setItem('currentRank', currentRank);
    localStorage.setItem('rankXP', rankXP);
    updateRankUI();
}

function updateRankUI(){
    const badge = document.getElementById('rank-badge');
    if(badge) badge.innerHTML = `🏆 RANK ${currentRank}`;
    const rankBonus = RANK_BONUS[currentRank] || 0;
    skinCreditMultiplier = 1 + (rankBonus / 100);
    skinDamageMultiplier = 1 + (rankBonus / 100);
    skinFireRateMultiplier = 1 + (rankBonus / 100);
}

// SPECIAL ITEMS
let stardustPurchased = localStorage.getItem('stardustPurchased') === 'true';
let guardianAngelCount = parseInt(localStorage.getItem('guardianAngelCount')) || 0;
let powerCoreCount = parseInt(localStorage.getItem('powerCoreCount')) || 0;
let cosmicEssenceCount = parseInt(localStorage.getItem('cosmicEssenceCount')) || 0;
let ironWillPurchased = localStorage.getItem('ironWillPurchased') === 'true';
let timeDistortionPurchased = localStorage.getItem('timeDistortionPurchased') === 'true';
let crystalHeartPurchased = localStorage.getItem('crystalHeartPurchased') === 'true';
let guardianAngelUsed = false;
let damageReduction = ironWillPurchased ? 0.5 : 1;
let enemySlow = timeDistortionPurchased ? 0.7 : 1;
let gemMultiplier = crystalHeartPurchased ? 2 : 1;
let resourceCollectionLevel = parseInt(localStorage.getItem('resourceCollectionLevel')) || 0;
let weaponEnhancementLevel = parseInt(localStorage.getItem('weaponEnhancementLevel')) || 0;

// ============================================
// RARE SPECIAL ABILITIES SYSTEM
// ============================================
let currentShopTab = 'weapons';

const RARE_ABILITIES = {
    shockwave: {
        id: 'shockwave', name: '💥 SHOCKWAVE BLAST', icon: '💥',
        desc: 'AoE push destroying nearby projectiles and enemies in a radius',
        cost: 5000, cooldown: 30000, category: 'weapons',
        unlockReq: () => bossesKilled >= 10,
        unlockDesc: 'Defeat 10 bosses',
        purchased: false, lastUsed: 0, active: false
    },
    regenAura: {
        id: 'regenAura', name: '💚 REGENERATION AURA', icon: '💚',
        desc: 'Passive HP regeneration (+1 HP every 5 seconds)',
        cost: 8000, cooldown: 0, category: 'shields',
        unlockReq: () => wave >= 25,
        unlockDesc: 'Reach wave 25',
        purchased: false, lastUsed: 0, active: false
    },
    timeSlow: {
        id: 'timeSlow', name: '👻 TIME SLOW/PHASE', icon: '👻',
        desc: 'Become intangible for 3 seconds, phasing through enemies and projectiles',
        cost: 6000, cooldown: 45000, category: 'shields',
        unlockReq: () => perfectWavesCount >= 5,
        unlockDesc: 'Survive 5 boss fights without taking damage (5 perfect waves)',
        purchased: false, lastUsed: 0, active: false
    },
    gravityBomb: {
        id: 'gravityBomb', name: '🌀 GRAVITY BOMB', icon: '🌀',
        desc: 'Pulls all enemies to a central point then explodes dealing massive damage',
        cost: 7000, cooldown: 35000, category: 'weapons',
        unlockReq: () => (parseInt(localStorage.getItem('eliteDropsCollected')) || 0) >= 3,
        unlockDesc: 'Collect 3 rare drops from elite enemies',
        purchased: false, lastUsed: 0, active: false
    },
    lightningStrike: {
        id: 'lightningStrike', name: '⚡ LIGHTNING STRIKE', icon: '⚡',
        desc: 'Instantly kills a random enemy on screen',
        cost: 5500, cooldown: 45000, category: 'weapons',
        unlockReq: () => wave >= 30,
        unlockDesc: 'Reach wave 30',
        purchased: false, lastUsed: 0, active: false
    },
    resourceMagnet: {
        id: 'resourceMagnet', name: '🧲 RESOURCE MAGNET', icon: '🧲',
        desc: 'Automatically attracts all dropped loot on screen for 15 seconds',
        cost: 3000, cooldown: 45000, category: 'shields',
        unlockReq: () => totalCoins >= 5000,
        unlockDesc: 'Accumulate 5000 coins',
        purchased: false, lastUsed: 0, active: false
    },
    timeWarp: {
        id: 'timeWarp', name: '🕐 TIME WARP', icon: '🕐',
        desc: 'Slows all enemies by 50% for 10 seconds',
        cost: 12000, cooldown: 90000, category: 'shields',
        unlockReq: () => totalKills >= 3000,
        unlockDesc: 'Accumulate 3000 kills',
        purchased: false, lastUsed: 0, active: false
    },
    ultimateAnnihilation: {
        id: 'ultimateAnnihilation', name: '💀 ULTIMATE ANNIHILATION', icon: '💀',
        desc: 'Clears all enemies on screen + invincibility for 5 seconds',
        cost: 25000, cooldown: 180000, category: 'weapons',
        unlockReq: () => totalKills >= 10000,
        unlockDesc: 'Accumulate 10000 kills',
        purchased: false, lastUsed: 0, active: false
    }
};

// Load purchased states
for(let key of Object.keys(RARE_ABILITIES)){
    RARE_ABILITIES[key].purchased = localStorage.getItem('rareAbility_'+key) === 'true';
}

// Ability visual state
let shockwaveActive = false;
let shockwaveTimer = 0;
let shockwaveRadius = 0;
let timeSlowActive = false;
let timeSlowTimer = 0;
let gravityBombActive = false;
let gravityBombTimer = 0;
let gravityBombCenter = {x:0, y:0};
let gravityBombPhase = 0; // 0=pull, 1=exploded
let regenAuraTimer = 0;
let lightningStrikeEffect = null; // {x, y, timer}
let resourceMagnetActive = false;
let resourceMagnetTimer = 0;
let abilityTimeWarpActive = false;
let abilityTimeWarpTimer = 0;
let ultimateAnnihilationActive = false;
let ultimateAnnihilationTimer = 0;

// Ability Equipment Selection (max 3 per game)
let selectedAbilities = JSON.parse(localStorage.getItem('selectedAbilities') || '[]');
if(!Array.isArray(selectedAbilities)) selectedAbilities = [];
// Validate: only keep purchased abilities
selectedAbilities = selectedAbilities.filter(key => RARE_ABILITIES[key] && RARE_ABILITIES[key].purchased);
if(selectedAbilities.length > 3) selectedAbilities = selectedAbilities.slice(0, 3);

function saveSelectedAbilities(){
    localStorage.setItem('selectedAbilities', JSON.stringify(selectedAbilities));
}

function toggleAbilitySelection(abilityId){
    if(!RARE_ABILITIES[abilityId] || !RARE_ABILITIES[abilityId].purchased) return;
    const idx = selectedAbilities.indexOf(abilityId);
    if(idx >= 0){
        selectedAbilities.splice(idx, 1);
    } else {
        if(selectedAbilities.length >= 3){
            showNotification('⚠ Max 3 abilities can be equipped!', 'warning');
            return;
        }
        selectedAbilities.push(abilityId);
    }
    saveSelectedAbilities();
    renderAbilitySelectionPanel();
}

function renderAbilitySelectionPanel(){
    renderHomeAbilityPanel();
}

function openAbilitySelectionPanel(){
    openHomeAbilityPanel();
}

function closeAbilitySelectionPanel(){
    closeHomeAbilityPanel();
    updateAbilityButtons();
}

function openHomeAbilityPanel(){
    const panel = document.getElementById('home-abilities-panel');
    if(!panel) return;
    panel.style.display = 'flex';
    renderHomeAbilityPanel();
}

function closeHomeAbilityPanel(){
    const panel = document.getElementById('home-abilities-panel');
    if(panel) panel.style.display = 'none';
}

function renderHomeAbilityPanel(){
    const content = document.getElementById('home-abilities-content');
    if(!content) return;
    let html = '<h3 style="color:#00d2ff;margin-bottom:12px;">🎯 ABILITIES</h3>';
    html += '<div style="font-size:11px;color:#aaa;margin-bottom:12px;">Select up to 3 abilities to equip for your next game</div>';
    for(let key of Object.keys(RARE_ABILITIES)){
        const ab = RARE_ABILITIES[key];
        const isUnlocked = ab.purchased;
        const isEligible = ab.unlockReq();
        const isSelected = selectedAbilities.includes(key);
        const bgColor = key === 'shockwave' ? '#ff8800' : key === 'timeSlow' ? '#aa66ff' : key === 'gravityBomb' ? '#4400aa' : key === 'resourceMagnet' ? '#ffcc00' : key === 'timeWarp' ? '#00ccff' : key === 'ultimateAnnihilation' ? '#ff0000' : key === 'lightningStrike' ? '#ffff00' : '#42f5b6';
        if(!isUnlocked){
            html += `<div style="display:flex;align-items:center;gap:8px;padding:8px;margin:4px 0;background:rgba(0,0,0,0.4);border-radius:10px;border:1px solid #444;opacity:0.5;">
                <span style="font-size:20px;">🔒</span>
                <div style="flex:1;text-align:right;">
                    <div style="font-size:12px;color:#888;">${ab.name}</div>
                    <div style="font-size:10px;color:#aaa;margin-top:2px;">${ab.desc}</div>
                    <div style="font-size:9px;color:#ff0044;margin-top:3px;">🔒 ${isEligible ? 'Requirement met - Buy in Upgrades shop!' : ab.unlockDesc}</div>
                </div>
            </div>`;
        } else {
            html += `<div style="display:flex;align-items:center;gap:8px;padding:8px;margin:4px 0;background:rgba(0,0,0,0.4);border-radius:10px;border:1px solid ${isSelected ? bgColor : '#444'};cursor:pointer;transition:0.2s;" onclick="toggleHomeAbility('${key}')">
                <input type="checkbox" ${isSelected ? 'checked' : ''} style="pointer-events:none;accent-color:${bgColor};" />
                <span style="font-size:20px;">${ab.icon}</span>
                <div style="flex:1;text-align:right;">
                    <div style="font-size:12px;color:#ccc;">${ab.name.replace(ab.icon+' ','')}</div>
                    <div style="font-size:10px;color:#aaa;margin-top:2px;">${ab.desc}</div>
                </div>
                ${isSelected ? '<span style="font-size:10px;color:#00ffaa;font-weight:bold;">✅ Equipped</span>' : ''}
            </div>`;
        }
    }
    html += '<div style="font-size:11px;color:#ffcc00;margin-top:12px;">Equipped: ' + selectedAbilities.length + '/3</div>';
    html += '<button class="btn" style="margin-top:12px;" onclick="closeHomeAbilityPanel()">DONE</button>';
    content.innerHTML = html;
}

function toggleHomeAbility(abilityId){
    if(!RARE_ABILITIES[abilityId] || !RARE_ABILITIES[abilityId].purchased) return;
    const idx = selectedAbilities.indexOf(abilityId);
    if(idx >= 0){
        selectedAbilities.splice(idx, 1);
    } else {
        if(selectedAbilities.length >= 3){
            showNotification('⚠ Max 3 abilities can be equipped!', 'warning');
            return;
        }
        selectedAbilities.push(abilityId);
    }
    saveSelectedAbilities();
    renderHomeAbilityPanel();
}

function buyRareAbility(id){
    const ability = RARE_ABILITIES[id];
    if(!ability) return;
    if(ability.purchased){
        showCustomAlert(`${ability.icon} ${ability.name} already purchased!`);
        return;
    }
    if(!ability.unlockReq()){
        showCustomAlert(`🔒 ${ability.name} is LOCKED!\nRequirement: ${ability.unlockDesc}`);
        return;
    }
    if(totalCoins < ability.cost){
        showCustomAlert(`Not enough credits! Need ${ability.cost}c, you have ${totalCoins}c`);
        return;
    }
    totalCoins -= ability.cost;
    ability.purchased = true;
    localStorage.setItem('rareAbility_'+id, 'true');
    localStorage.setItem('totalCoins', totalCoins);
    showCustomAlert(`${ability.icon} ${ability.name} PURCHASED!\n${ability.desc}`);
    showNotification(`${ability.icon} ${ability.name} acquired!`, 'success');
    updateShopUI();
    checkAchievements();
}

function activateRareAbility(id){
    const ability = RARE_ABILITIES[id];
    if(!ability || !ability.purchased || gameState !== 'PLAYING') return;
    const now = Date.now();
    if(ability.cooldown > 0 && now - ability.lastUsed < ability.cooldown) return;
    ability.lastUsed = now;

    if(id === 'shockwave'){
        shockwaveActive = true;
        shockwaveTimer = Date.now() + 800;
        shockwaveRadius = 0;
        // Destroy nearby enemies and projectiles
        let radius = 200;
        for(let i = enemies.length-1; i >= 0; i--){
            let e = enemies[i];
            if(player && Math.hypot(e.x - player.x, e.y - player.y) < radius){
                let pointBonus = e.isBoss ? 9000 : 3500 * combo;
                pointBonus *= skinCreditMultiplier;
                score += pointBonus; kills++; waveKills++;
                if(e.isBoss) bossesKilled++;
                for(let k=0;k<15;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*12,(Math.random()-0.5)*12,'#ff8800',0.9));
                enemies.splice(i,1);
            }
        }
        if(boss && player && Math.hypot(boss.x - player.x, boss.y - player.y) < radius){
            boss.hp -= 500;
            if(boss.hp <= 0){ boss = null; bossesKilled++; }
        }
        eBullets.length = 0;
        if(player && player.x && player.y) floats.push({txt:'💥 SHOCKWAVE!',x:player.x-60,y:player.y-40,l:1.5,c:'#ff8800',size:24});
        showNotification('💥 Shockwave Blast activated!', 'success');
    }
    else if(id === 'regenAura'){
        // Passive - no activation needed, handled in game loop
        showCustomAlert('💚 Regeneration Aura is always active! +1 HP every 5 seconds.');
    }
    else if(id === 'timeSlow'){
        timeSlowActive = true;
        timeSlowTimer = Date.now() + 3000;
        if(player) player.invincibleTimer = 180; // ~3 seconds at 60fps
        if(player && player.x && player.y) floats.push({txt:'👻 PHASE MODE!',x:player.x-50,y:player.y-40,l:1.5,c:'#aa66ff',size:24});
        showNotification('👻 Time Slow/Phase activated! Intangible for 3 seconds!', 'success');
    }
    else if(id === 'gravityBomb'){
        gravityBombActive = true;
        gravityBombTimer = Date.now() + 3000;
        gravityBombCenter = player ? {x: player.x, y: player.y} : {x: width/2, y: height/2};
        gravityBombPhase = 0;
        if(player && player.x && player.y) floats.push({txt:'🌀 GRAVITY BOMB!',x:player.x-60,y:player.y-40,l:1.5,c:'#4400aa',size:24});
        showNotification('🌀 Gravity Bomb activated! Enemies pulled to center!', 'success');
    }
    else if(id === 'lightningStrike'){
        if(enemies.length > 0){
            let targetIdx = Math.floor(Math.random() * enemies.length);
            let target = enemies[targetIdx];
            lightningStrikeEffect = {x: target.x, y: target.y, timer: Date.now() + 600};
            let pointBonus = target.isBoss ? 50000 : 5000 * combo;
            pointBonus *= skinCreditMultiplier;
            score += pointBonus; kills++; waveKills++;
            if(target.isBoss){ bossesKilled++; boss = null; }
            for(let k=0;k<25;k++) particles.push(new Particle(target.x,target.y,(Math.random()-0.5)*15,(Math.random()-0.5)*15,'#ffff00',1));
            enemies.splice(targetIdx, 1);
            if(player && player.x && player.y) floats.push({txt:'⚡ LIGHTNING STRIKE!',x:target.x-60,y:target.y-40,l:1.5,c:'#ffff00',size:24});
            showNotification('⚡ Lightning Strike! Enemy destroyed!', 'success');
        } else {
            ability.lastUsed = 0; // Refund cooldown if no enemies
            showNotification('⚡ No enemies to strike!', 'warning');
        }
    }
    else if(id === 'resourceMagnet'){
        resourceMagnetActive = true;
        resourceMagnetTimer = Date.now() + 15000;
        if(player && player.x && player.y) floats.push({txt:'🧲 RESOURCE MAGNET!',x:player.x-60,y:player.y-40,l:1.5,c:'#ffcc00',size:24});
        showNotification('🧲 Resource Magnet activated! All loot attracted for 15s!', 'success');
    }
    else if(id === 'timeWarp'){
        abilityTimeWarpActive = true;
        abilityTimeWarpTimer = Date.now() + 10000;
        if(player && player.x && player.y) floats.push({txt:'🕐 TIME WARP!',x:player.x-50,y:player.y-40,l:1.5,c:'#00ccff',size:24});
        showNotification('🕐 Time Warp activated! Enemies slowed 50% for 10s!', 'success');
    }
    else if(id === 'ultimateAnnihilation'){
        ultimateAnnihilationActive = true;
        ultimateAnnihilationTimer = Date.now() + 5000;
        if(player) player.invincibleTimer = 300; // ~5 seconds at 60fps
        // Clear all enemies on screen
        let annihilatedCount = 0;
        for(let i = enemies.length-1; i >= 0; i--){
            let e = enemies[i];
            let pointBonus = e.isBoss ? 100000 : 10000 * combo;
            pointBonus *= skinCreditMultiplier;
            score += pointBonus; kills++; waveKills++; annihilatedCount++;
            if(e.isBoss) bossesKilled++;
            for(let k=0;k<15;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*15,(Math.random()-0.5)*15,'#ff0000',1));
            enemies.splice(i,1);
        }
        boss = null;
        eBullets.length = 0;
        if(player && player.x && player.y) floats.push({txt:'💀 ANNIHILATION! +' + annihilatedCount, x:player.x-70,y:player.y-40,l:2,c:'#ff0000',size:28});
        showNotification(`💀 Ultimate Annihilation! ${annihilatedCount} enemies destroyed! Invincible for 5s!`, 'success');
    }
    updateAbilityButtons();
}

function updateAbilityButtons(){
    const container = document.getElementById('ability-btns');
    if(!container) return;
    if(gameState !== 'PLAYING'){
        container.style.display = 'none';
        return;
    }
    let html = '';
    let hasAny = false;
    const now = Date.now();
    for(let key of Object.keys(RARE_ABILITIES)){
        const ab = RARE_ABILITIES[key];
        if(!ab.purchased || ab.cooldown === 0) continue; // Skip passive abilities
        if(selectedAbilities.length > 0 && !selectedAbilities.includes(key)) continue; // Skip non-selected abilities
        hasAny = true;
        const cdRemaining = Math.max(0, ab.cooldown - (now - ab.lastUsed));
        const onCooldown = cdRemaining > 0;
        const cdSec = Math.ceil(cdRemaining / 1000);
        const bgColor = key === 'shockwave' ? '#ff8800' : key === 'timeSlow' ? '#aa66ff' : key === 'gravityBomb' ? '#4400aa' : key === 'resourceMagnet' ? '#ffcc00' : key === 'timeWarp' ? '#00ccff' : key === 'ultimateAnnihilation' ? '#ff0000' : '#ffff00';
        html += `<div class="ability-icon ${onCooldown ? 'on-cooldown' : ''}" style="border-color:${bgColor};background:rgba(0,0,0,0.6);" onclick="activateRareAbility('${key}')">
            ${ab.icon}
            ${onCooldown ? `<div class="ability-cooldown-overlay">${cdSec}s</div>` : ''}
        </div>`;
    }
    container.innerHTML = html;
    container.style.display = hasAny ? 'block' : 'none';
}

// ============================================
// DYNAMIC UPDATE LOG SYSTEM
// ============================================
let updateLogData = [
    {
        version: 'v11.6',
        changes: [
            'Bug fix: SyntaxError - missing closing brace for updateShopUI() function caused Unexpected token error',
            'Bug fix: TypeError crash - starfallStars contained plain objects instead of StarfallStar class instances',
            'Bug fix: Golden Jackpot typo (JACKPET→JACKPOT) and Legendary lootbox ultra-rare had 0% probability',
            'NEW: Skill Tree System with 3 branching paths (Offense/Defense/Utility), 18 total nodes',
            'Skill Tree: Offense path (Damage +5/10%, Fire Rate +8/15%, Crit Chance +5%, Crit Damage +50%)',
            'Skill Tree: Defense path (Shield +20, Health +25/50, Dmg Reduce +8/15%, Regen +1hp/5s)',
            'Skill Tree: Utility path (Move Speed +10/20%, Magnet Range +30/60, Drone Efficiency +10/25%)',
            'Earn 1 skill point every 5 levels or 500 kills - unlock nodes on the skill tree panel',
            'Skill tree button added on Home Screen (right side, small circular 🌳 button)',
            'Lootbox Ultra-Rare Rebalance: drop chances lowered from 1.5-3% to 0.3-0.5%',
            'Ultra-Rare items now have UNIQUE effects per lootbox type and are MUCH more rewarding',
            'Basic lootbox: Golden Jackpot - 10000 credits (0.5%)',
            'Premium lootbox: Legendary Weapon Core - permanent +15% weapon damage (0.4%)',
            'Epic/Mythic lootbox: Cosmic Shard - instantly unlock any locked ability (0.4-0.5%)',
            'Ultra lootbox: Cosmic Shard Ultimate - unlock ability + 3 free skill points (0.3%)',
            'Probability Viewer now displays ultra-rare effect descriptions clearly',
            'NEW: Event Queue System - queued events no longer overlap, they trigger sequentially',
            'NEW EVENT: Asteroid Belt - shoot crossing asteroids for bonus coins and gems (~10s)',
            'NEW EVENT: Supply Drop - fly to a cargo crate before it disappears for random rewards (~8s)',
            'Starfall nerf: 10→8 initial stars, lower value (20-60 vs 30-60), slower spawn rate (8% vs 12%), 9s duration',
            'Starfall trigger chance slightly increased (+20%) to compensate for nerfed rewards',
            'Doomsday buff: 5x trigger chance, 30 enemies spawn (was 20), 5x point bonus, 15% chance for Green Aura',
            'Green Aura: ship gets green cloak that deals light damage to nearby enemies during Doomsday',
            'Divine Intervention now MUCH rarer (reduced by ~70%) but astronomically powerful',
            'Divine: Full health & shield restore, +300% damage for 60s, screen-clear all enemies, 5s invincibility',
            'Divine: Golden light rays visual, dramatic screen flash, extended 3s flash effect'
        ]
    },
    {
        version: 'v11.5',
        changes: [
            'Bug fix: TypeError crash when accessing guardian.y after guardian was destroyed',
            'Bug fix: Particle crash when guardian was null after defeat (coordinates captured before null)',
            'Abilities screen moved to Home Screen - equip up to 3 abilities before starting a game',
            'All abilities now visible in the panel (locked, unlocked, equipped status)',
            'Removed ability selection button from game screen - no selection during gameplay',
            'Lootbox Shop UI cleanup - removed all glow/sparkle/shimmer animations for clean flat design',
            'Home Screen UI cleanup - removed all glow/shimmer text animations for readable flat typography',
            'Ultra-Rare items added to every lootbox type with very low drop chance (1-3%)',
            'Ultra-Rare: Golden Treasure (Basic) - 5000 credits at 1.5% chance',
            'Ultra-Rare: Legendary Weapon Core (Premium) - permanent weapon damage boost at 2% chance',
            'Ultra-Rare: Cosmic Shard (Mythic/Ultra) - instantly unlocks a locked ability at 2.5-3% chance',
            'Probability Viewer updated to display ultra-rare items with distinct purple ULTRA-RARE label',
            'Cosmic Shard reward type: unlocks first eligible locked ability or gives 2000 gemstones fallback'
        ]
    },
    {
        version: 'v11.4',
        changes: [
            'Fixed Settings Back Button bug - now correctly returns to pause screen when opened from pause menu',
            'NEW ABILITY: Resource Magnet - attracts all dropped loot for 15s (5000 coins to unlock)',
            'NEW ABILITY: Time Warp - slows all enemies 50% for 10s (3000 kills to unlock)',
            'NEW ABILITY: Ultimate Annihilation - clears all enemies + 5s invincibility (10000 kills to unlock)',
            'Ability Equipment System - select max 3 abilities per game via new selection panel',
            'Lootbox Probability Viewer - view drop rates and rarity for every lootbox reward',
            'Drone/Weapon purchase maximum reduced to 180',
            'NEW UPGRADE: Resource Collection - increases auto-pickup range (+15 per level, max 20)',
            'NEW UPGRADE: Weapon Enhancement - increases damage and fire rate (+5% per level, max 20)',
            'Bug fix: Settings back navigation state leak resolved',
            'Bug fix: closeSettings no longer overridden by backToHub',
            'Removed date fields from all update log entries'
        ]
    },
    {
        version: 'v11.0',
        changes: [
            'LANGUAGE SYSTEM! English, Hebrew, Russian!',
            'Main Hub UI completely redesigned!',
            'NEW EVENT: Space Treasure (45% every 9 waves)!',
            'Auto-Fire toggle',
            'Volume sliders for SFX and Music',
            'Enhanced Pause Screen with quick settings',
            'Floating Damage Numbers',
            'Enemy Health Bars',
            'Boss HP Bar repositioned below HUD',
            'Wave Progress Bar',
            'Tutorial/How To Play overlay',
            'FPS Counter toggle',
            'Save Progress Indicator',
            'Safe Reset Confirmation dialog',
            'Achievements Menu upgrade with filters and difficulty colors',
            'Lootbox Animation upgrade with rarity effects and particles',
            'Keybinding Reference in Settings',
            'Player null-safety crash fixes',
            'JSON.parse error handling'
        ]
    },
    {
        version: 'v2.0',
        changes: [
            'Site fixed and deployed',
            'Added 5 new rare abilities: Shockwave Blast, Regeneration Aura, Time Slow/Phase, Gravity Bomb, Lightning Strike',
            'Shop restructured into 3 categories: Weapons, Shields, Special Abilities',
            'Dynamic update log system implemented'
        ]
    },
    {
        version: 'v10.5',
        changes: [
            '2 new events! DOPPELGANGER + SUPERNOVA!',
            '15 new achievements! (310+ total!)',
            'Settings button!',
            'CRITICAL HITS!',
            'COMBO METER visual!',
            'STATS PANEL',
            'AUTO-SAVE every 30 seconds!',
            'Background music system! Dynamic music based on game state!'
        ]
    },
    {
        version: 'v10.0',
        changes: [
            '10 new events! STARFALL, INFERNO, CHAIN LIGHTNING, BARRIER, SOUL REAPER, GAMBLER, VORTEX, KING\'S BLESSING, PRISM, ABYSS!',
            '30 new achievements!',
            'RANK system! 50 levels with bonuses!',
            'DAILY MISSIONS!'
        ]
    },
    {
        version: 'v9.1',
        changes: ['Game crash bug fix']
    },
    {
        version: 'v8.0',
        changes: ['Daily reward + 3 new events']
    },
    {
        version: 'v1.0',
        changes: ['Initial launch! 9 events, 100 achievements']
    }
];

function renderUpdateLog(){
    const container = document.getElementById('update-log-container');
    if(!container) return;
    container.innerHTML = '';
    for(const entry of updateLogData){
        const div = document.createElement('div');
        div.className = 'update-item';
        let changesHtml = entry.changes.map(c => '• ' + c).join('<br>');
        div.innerHTML = `<div class="update-version">⚡ ${entry.version}</div><div class="update-desc">${changesHtml}</div>`;
        container.appendChild(div);
    }
}

function addUpdateLogEntry(version, changes){
    updateLogData.unshift({ version, changes });
    renderUpdateLog();
}

// ============================================
// SHOP TAB SYSTEM
// ============================================
function switchShopTab(tab){
    currentShopTab = tab;
    document.querySelectorAll('.shop-tab').forEach(btn => btn.classList.remove('active'));
    const tabMap = {'weapons':0, 'shields':1, 'special':2};
    const tabs = document.querySelectorAll('.shop-tab');
    if(tabs[tabMap[tab]]) tabs[tabMap[tab]].classList.add('active');
    updateShopUI();
}

const SPECIAL_ITEMS_DATA = [
    {id:'stardust', name:'⭐ STARDUST', desc:'+10% ALL STATS', price:5000, category:'weapons', oneTime:true, ownedCheck:()=>stardustPurchased, tooltip:'מגדיל את כל הסטטים ב-10% לצמיתות!'},
    {id:'guardian', name:'🛡️ GUARDIAN ANGEL', desc:'Death immunity once', price:3000, category:'shields', oneTime:false, ownedCheck:()=>false, tooltip:'מעניק חסינות למוות פעם אחת.', countHtml:()=>`נותרו: ${guardianAngelCount}/3`},
    {id:'powercore', name:'⚡ POWER CORE', desc:'+5% DAMAGE', price:2000, category:'weapons', oneTime:false, ownedCheck:()=>false, tooltip:'מגדיל את הנזק ב-5% לצמיתות. עד 20 פעמים.', countHtml:()=>`${powerCoreCount}/20`},
    {id:'cosmic', name:'🌟 COSMIC ESSENCE', desc:'+3% FIRE RATE', price:1500, category:'weapons', oneTime:false, ownedCheck:()=>false, tooltip:'מגדיל את מהירות הירי ב-3% לצמיתות. עד 15 פעמים.', countHtml:()=>`${cosmicEssenceCount}/15`},
    {id:'ironwill', name:'🛡️ IRON WILL', desc:'50% DAMAGE REDUCTION', price:5000, category:'shields', oneTime:true, ownedCheck:()=>ironWillPurchased, tooltip:'מקנה 50% חסינות לנזק לצמיתות!'},
    {id:'timedistortion', name:'⏰ TIME DISTORTION', desc:'30% SLOW ENEMIES', price:8000, category:'shields', oneTime:true, ownedCheck:()=>timeDistortionPurchased, tooltip:'מאט את כל האויבים ב-30% לצמיתות!'},
    {id:'crystalheart', name:'💎 CRYSTAL HEART', desc:'2x GEMSTONES', price:10000, category:'shields', oneTime:true, ownedCheck:()=>crystalHeartPurchased, tooltip:'מכפיל את כל ה-GEMSTONES ב-2!'}
];

function buySpecialItem(item){
    if(item === 'stardust'){
        if(stardustPurchased){
            showCustomAlert('⭐ כבר רכשת את STARDUST! ניתן לקנות רק פעם אחת.');
            return;
        }
        if(totalCoins >= 5000){
            totalCoins -= 5000;
            stardustPurchased = true;
            localStorage.setItem('stardustPurchased', 'true');
            damageLevel = Math.floor(damageLevel * 1.1);
            fireLevel = Math.floor(fireLevel * 1.1);
            localStorage.setItem('dmgLevel', damageLevel);
            localStorage.setItem('fireLevel', fireLevel);
            showCustomAlert(`⭐ STARDUST נרכש! כל הסטטים הוגדלו ב-10%! ⭐\nדמג: ${damageLevel} | Fire Rate: ${fireLevel}`);
            showNotification(`⭐ STARDUST acquired! +10% to all stats!`, 'success');
            updateShopUI();
            checkAchievements();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 5000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'guardian'){
        if(guardianAngelCount >= 3){
            showCustomAlert('🛡️ You already have maximum Guardian Angel charges (3/3)!');
            return;
        }
        if(totalCoins >= 3000){
            totalCoins -= 3000;
            guardianAngelCount++;
            localStorage.setItem('guardianAngelCount', guardianAngelCount);
            showCustomAlert(`🛡️ GUARDIAN ANGEL purchased! (${guardianAngelCount}/3 charges)\nGrants death immunity once!`);
            tn('alert_guardian_bought', 'success', guardianAngelCount);
            updateShopUI();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 3000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'powercore'){
        if(powerCoreCount >= 20){
            showCustomAlert('⚡ הגעת למקסימום של POWER CORE (20)!');
            return;
        }
        if(totalCoins >= 2000){
            totalCoins -= 2000;
            powerCoreCount++;
            damageLevel += Math.floor(damageLevel * 0.05);
            localStorage.setItem('powerCoreCount', powerCoreCount);
            localStorage.setItem('dmgLevel', damageLevel);
            showCustomAlert(`⚡ POWER CORE +5% DAMAGE! (${powerCoreCount}/20)\nדמג כעת: ${damageLevel}`);
            showNotification(`⚡ Power Core +5% damage! (${powerCoreCount}/20)`, 'success');
            updateShopUI();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 2000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'cosmic'){
        if(cosmicEssenceCount >= 15){
            showCustomAlert('🌟 הגעת למקסימום של COSMIC ESSENCE (15)!');
            return;
        }
        if(totalCoins >= 1500){
            totalCoins -= 1500;
            cosmicEssenceCount++;
            fireLevel += Math.floor(fireLevel * 0.03);
            localStorage.setItem('cosmicEssenceCount', cosmicEssenceCount);
            localStorage.setItem('fireLevel', fireLevel);
            showCustomAlert(`🌟 COSMIC ESSENCE +3% FIRE RATE! (${cosmicEssenceCount}/15)\nקצב ירי כעת: ${fireLevel}`);
            showNotification(`🌟 Cosmic Essence +3% fire rate! (${cosmicEssenceCount}/15)`, 'success');
            updateShopUI();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 1500c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'ironwill'){
        if(ironWillPurchased){
            showCustomAlert('🛡️ כבר רכשת את IRON WILL! ניתן לקנות רק פעם אחת.');
            return;
        }
        if(totalCoins >= 5000){
            totalCoins -= 5000;
            ironWillPurchased = true;
            localStorage.setItem('ironWillPurchased', 'true');
            damageReduction = 0.5;
            showCustomAlert(`🛡️ IRON WILL נרכש! כל הנזק שהתקבל יחצה! 🛡️`);
            showNotification(`🛡️ Iron Will acquired! 50% damage reduction!`, 'success');
            updateShopUI();
            checkAchievements();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 5000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'timedistortion'){
        if(timeDistortionPurchased){
            showCustomAlert('⏰ כבר רכשת את TIME DISTORTION! ניתן לקנות רק פעם אחת.');
            return;
        }
        if(totalCoins >= 8000){
            totalCoins -= 8000;
            timeDistortionPurchased = true;
            localStorage.setItem('timeDistortionPurchased', 'true');
            enemySlow = 0.7;
            showCustomAlert(`⏰ TIME DISTORTION נרכש! כל האויבים הואטו ב-30%! ⏰`);
            showNotification(`⏰ Time Distortion acquired! Enemies slowed by 30%!`, 'success');
            updateShopUI();
            checkAchievements();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 8000c, יש לך ${totalCoins}c`);
        }
    } else if(item === 'crystalheart'){
        if(crystalHeartPurchased){
            showCustomAlert('💎 כבר רכשת את CRYSTAL HEART! ניתן לקנות רק פעם אחת.');
            return;
        }
        if(totalCoins >= 10000){
            totalCoins -= 10000;
            crystalHeartPurchased = true;
            localStorage.setItem('crystalHeartPurchased', 'true');
            gemMultiplier = 2;
            showCustomAlert(`💎 CRYSTAL HEART נרכש! כל ה-GEMSTONES שיתקבלו יוכפלו ב-2! 💎`);
            showNotification(`💎 Crystal Heart acquired! 2x GEMSTONES forever!`, 'success');
            updateShopUI();
            checkAchievements();
        } else {
            showCustomAlert(`לא מספיק קרדיטים! צריך 10000c, יש לך ${totalCoins}c`);
        }
    }
}

// GAME VARIABLES
let currentGame = null;
let gemstones = parseInt(localStorage.getItem('gemstones')) || 0;
let lootboxesOpened = parseInt(localStorage.getItem('lootboxesOpened')) || 0;
let currentSkin = localStorage.getItem('currentSkin') || 'default';

// SKIN DEFINITIONS
const SKINS = [
    { id: 'default', name: 'DEFAULT', icon: '🚀', desc: 'החללית הבסיסית. פשוטה אבל אמינה!', effect: null, requirement: 0, owned: true },
    { id: 'blue', name: 'BLUE COMMON', icon: '🔵', desc: 'סקין כחול. הוא... כחול. זהו.', effect: null, requirement: 'lootbox', owned: false },
    { id: 'purple', name: 'PURPLE RARE', icon: '🟣', desc: 'סגלגל וחמוד. אויבים מפחדים מסגול?', effect: null, requirement: 'lootbox', owned: false },
    { id: 'gold', name: 'GOLDEN LEGEND', icon: '👑', desc: 'סקין זהב אגדי! מראה את השליטה שלך!', effect: '+25% CREDITS, +20% DAMAGE', requirement: 40, owned: false },
    { id: 'rainbow', name: 'RAINBOW MYTHIC', icon: '🌈', desc: 'צבעי הקשת! מסנוור את האויבים ביופי!', effect: null, requirement: 'lootbox', owned: false },
    { id: 'ultra', name: 'ULTRA MYTHIC', icon: '💎', desc: 'הסקין הנדיר ביותר ביקום! רק לעילא ולעלא!', effect: '+50% CREDITS, +50% DAMAGE, +20% FIRE RATE', requirement: 'ultra_lootbox', owned: false, limited: true },
    { id: 'legend', name: 'LEGEND RANK 50', icon: '🏆', desc: 'סקין אגדי! מושג רק על ידי הטובים ביותר!', effect: '+100% CREDITS, +100% DAMAGE, +50% FIRE RATE', requirement: 'rank50', owned: false }
];

// EVENT COUNTERS
let reaperCalls = 0, timeWarps = 0, goldRushes = 0, divineInterventions = 0, bugEvents = 0;
let stableCycleCount = 0, tidalWaveCount = 0, masqueradeCount = 0, soulHarvestCount = 0;
let frozenTimeCount = 0, crystalRainCount = 0, shadowCloneCount = 0;
let lightningStormCount = 0, luckyDrawCount = 0, mysteryBoxCount = 0, doomsDayCount = 0, royalBlessingCount = 0;
let starfallCount = 0, infernoCount = 0, chainLightningCount = 0, barrierCount = 0, soulReaperCount = 0;
let gamblerCount = 0, vortexCount = 0, kingsBlessingCount = 0, prismCount = 0, abyssCount = 0;
let doppelgangerCount = 0, supernovaCount = 0;
let spaceTreasureActive = false;
let spaceTreasureTimer = 0;
let spaceTreasureCount = parseInt(localStorage.getItem('spaceTreasureCount')) || 0;
let spaceTreasureCrates = [];

function showAchievementPopup(name, desc, gemsAmount = 0){
    const popup = document.getElementById('achievement-popup');
    const nameEl = document.getElementById('ach-name');
    const descEl = document.getElementById('ach-desc');
    nameEl.innerText = name;
    if(gemsAmount > 0){
        descEl.innerText = `${desc} +${gemsAmount} 💎`;
    } else {
        descEl.innerText = desc;
    }
    popup.style.display = 'block';
    setTimeout(() => {
        popup.style.display = 'none';
    }, 3000);
    showNotification(`🏆 ${name} unlocked!`, 'success');
}

function flashScreen(){
    if(!settings.shake) return;
    const flash = document.createElement('div');
    flash.style.position = 'fixed';
    flash.style.top = 0;
    flash.style.left = 0;
    flash.style.width = '100%';
    flash.style.height = '100%';
    flash.style.backgroundColor = 'rgba(255,255,255,0.5)';
    flash.style.zIndex = 9999;
    flash.style.pointerEvents = 'none';
    document.body.appendChild(flash);
    setTimeout(() => flash.remove(), 200);
}

// LOOTBOX REWARDS
const LOOTBOX_REWARDS = {
    common: [
        {name:"50 GEMSTONES", type:"gem", amount:50, rarity:"common", icon:"💎", probability:24.5, stars:1},
        {name:"100 GEMSTONES", type:"gem", amount:100, rarity:"common", icon:"💎", probability:19.5, stars:1},
        {name:"500 CREDITS", type:"credit", amount:500, rarity:"common", icon:"💰", probability:24.5, stars:1},
        {name:"TEMPORARY SPEED BOOST", type:"boost", effect:"speed", duration:60, rarity:"common", icon:"⚡", probability:19.5, stars:1},
        {name:"1 BOMB", type:"bomb", amount:1, rarity:"common", icon:"💣", probability:12, stars:1},
        {name:"🌟 GOLDEN JACKPOT", type:"credit", amount:10000, rarity:"legendary", icon:"🌟", probability:0.5, stars:4, ultraRare:true, ultraRareEffect:"Golden Jackpot - +10000 CREDITS (0.5%)"}
    ],
    rare: [
        {name:"150 GEMSTONES", type:"gem", amount:150, rarity:"rare", icon:"💎", probability:21.5, stars:2},
        {name:"250 GEMSTONES", type:"gem", amount:250, rarity:"rare", icon:"💎", probability:17.5, stars:2},
        {name:"1000 CREDITS", type:"credit", amount:1000, rarity:"rare", icon:"💰", probability:21.5, stars:2},
        {name:"TEMPORARY DAMAGE BOOST", type:"boost", effect:"damage", duration:90, rarity:"rare", icon:"💪", probability:17.5, stars:2},
        {name:"2 BOMBS", type:"bomb", amount:2, rarity:"rare", icon:"💣", probability:11.5, stars:2},
        {name:"COMMON SKIN", type:"skin", skin:"blue", rarity:"rare", icon:"🎨", probability:10.1, stars:2},
        {name:"🌟 LEGENDARY WEAPON CORE", type:"perm_upgrade", stat:"damage", amount:3, rarity:"legendary", icon:"⚔️", probability:0.4, stars:4, ultraRare:true, ultraRareEffect:"Legendary Weapon Core - Permanent +15% weapon damage (+3 damage levels) (0.4%)"}
    ],
    epic: [
        {name:"300 GEMSTONES", type:"gem", amount:300, rarity:"epic", icon:"💎", probability:19.5, stars:3},
        {name:"500 GEMSTONES", type:"gem", amount:500, rarity:"epic", icon:"💎", probability:14.5, stars:3},
        {name:"2500 CREDITS", type:"credit", amount:2500, rarity:"epic", icon:"💰", probability:19.5, stars:3},
        {name:"PERMANENT DAMAGE UPGRADE", type:"perm_upgrade", stat:"damage", amount:1, rarity:"epic", icon:"🔰", probability:17.5, stars:3},
        {name:"3 BOMBS", type:"bomb", amount:3, rarity:"epic", icon:"💣", probability:14.5, stars:3},
        {name:"RARE SKIN", type:"skin", skin:"purple", rarity:"epic", icon:"🎨", probability:14.1, stars:3},
        {name:"🌟 COSMIC SHARD", type:"unlock_ability", rarity:"legendary", icon:"💠", probability:0.4, stars:4, ultraRare:true, ultraRareEffect:"Cosmic Shard - Instantly unlock any locked ability (0.4%)"}
    ],
    legendary: [
        {name:"600 GEMSTONES", type:"gem", amount:600, rarity:"legendary", icon:"💎", probability:17.5, stars:4},
        {name:"1000 GEMSTONES", type:"gem", amount:1000, rarity:"legendary", icon:"💎", probability:13.5, stars:4},
        {name:"5000 CREDITS", type:"credit", amount:5000, rarity:"legendary", icon:"💰", probability:17.5, stars:4},
        {name:"PERMANENT FIRE RATE UPGRADE", type:"perm_upgrade", stat:"fire", amount:2, rarity:"legendary", icon:"🔥", probability:17.5, stars:4},
        {name:"5 BOMBS", type:"bomb", amount:5, rarity:"legendary", icon:"💣", probability:15.5, stars:4},
        {name:"LEGENDARY SKIN", type:"skin", skin:"gold", rarity:"legendary", icon:"👑", probability:18.5, stars:4},
        {name:"🌟 DIVINE SHARD", type:"perm_upgrade", stat:"damage", amount:5, rarity:"legendary", icon:"✨", probability:0.4, stars:5, ultraRare:true, ultraRareEffect:"Divine Shard - Permanent +5 damage AND full shield upgrade (0.4%)"}
    ],
    mythic: [
        {name:"1500 GEMSTONES", type:"gem", amount:1500, rarity:"mythic", icon:"💎", probability:15.5, stars:5},
        {name:"2500 GEMSTONES", type:"gem", amount:2500, rarity:"mythic", icon:"💎", probability:11.5, stars:5},
        {name:"10000 CREDITS", type:"credit", amount:10000, rarity:"mythic", icon:"💰", probability:15.5, stars:5},
        {name:"PERMANENT DAMAGE UPGRADE x3", type:"perm_upgrade", stat:"damage", amount:3, rarity:"mythic", icon:"🔰🔰", probability:17.5, stars:5},
        {name:"7 BOMBS", type:"bomb", amount:7, rarity:"mythic", icon:"💣", probability:17.5, stars:5},
        {name:"MYTHIC SKIN", type:"skin", skin:"rainbow", rarity:"mythic", icon:"🌈", probability:22, stars:5},
        {name:"🌟 COSMIC SHARD", type:"unlock_ability", rarity:"legendary", icon:"💠", probability:0.5, stars:6, ultraRare:true, ultraRareEffect:"Cosmic Shard - Instantly unlock any locked ability (0.5%)"}
    ],
    ultra: [
        {name:"5000 GEMSTONES", type:"gem", amount:5000, rarity:"ultra", icon:"💎💎", probability:13.5, stars:6},
        {name:"10000 GEMSTONES", type:"gem", amount:10000, rarity:"ultra", icon:"💎💎", probability:9.5, stars:6},
        {name:"50000 CREDITS", type:"credit", amount:50000, rarity:"ultra", icon:"💰💰", probability:13.5, stars:6},
        {name:"ULTRA MYTHIC SKIN (LIMITED)", type:"skin", skin:"ultra", rarity:"ultra", icon:"👑👑", limited:true, probability:11.5, stars:6},
        {name:"15 BOMBS", type:"bomb", amount:15, rarity:"ultra", icon:"💣💣", probability:19.5, stars:6},
        {name:"ALL PERMANENT UPGRADES +5", type:"perm_upgrade_all", amount:5, rarity:"ultra", icon:"⭐", probability:32.2, stars:6},
        {name:"🌟 COSMIC SHARD ULTIMATE", type:"unlock_ability", rarity:"legendary", icon:"💠", probability:0.3, stars:7, ultraRare:true, ultraRareEffect:"Cosmic Shard Ultimate - Unlock ANY locked ability + 3 free skill points (0.3%)"}
    ]
};

const INDIVIDUAL_REWARDS = [
    {name:"COMMON SKIN", type:"skin", skin:"blue", price:200, rarity:"rare", icon:"🎨"},
    {name:"RARE SKIN", type:"skin", skin:"purple", price:500, rarity:"epic", icon:"🎨"},
    {name:"LEGENDARY SKIN", type:"skin", skin:"gold", price:1500, rarity:"legendary", icon:"👑"},
    {name:"MYTHIC SKIN", type:"skin", skin:"rainbow", price:4000, rarity:"mythic", icon:"🌈"},
    {name:"PERMANENT DAMAGE +1", type:"perm_upgrade", stat:"damage", price:800, rarity:"epic", icon:"🔰"},
    {name:"PERMANENT FIRE RATE +2", type:"perm_upgrade", stat:"fire", price:1000, rarity:"legendary", icon:"🔥"},
    {name:"1000 GEMSTONES", type:"gem", amount:1000, price:1200, rarity:"legendary", icon:"💎"},
    {name:"5000 CREDITS", type:"credit", amount:5000, price:600, rarity:"epic", icon:"💰"}
];

let ownedSkins; try { ownedSkins = JSON.parse(localStorage.getItem('ownedSkins')) || {blue:false, purple:false, gold:false, rainbow:false, ultra:false, legend:false}; } catch(e) { ownedSkins = {blue:false, purple:false, gold:false, rainbow:false, ultra:false, legend:false}; }
let ultraMythicCount = parseInt(localStorage.getItem('ultraMythicCount')) || 0;
const ULTRA_MYTHIC_LIMIT = 5;

function saveGemstones(){
    localStorage.setItem('gemstones', gemstones);
    updateGemUI();
}

function updateGemUI(){
    const gemElements = ['gemstones-amount', 'hub-gems', 'rng-gems'];
    gemElements.forEach(id => {
        const el = document.getElementById(id);
        if(el) el.innerText = formatNumber(gemstones);
    });
}

function formatNumber(num){ return num.toLocaleString(); }

// SKINS SYSTEM
function updateSkinsUI(){
    const container = document.getElementById('skins-grid-container');
    if(!container) return;
    container.innerHTML = '';
    const unlockedCount = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    
    for(const skin of SKINS){
        let isOwned = false;
        if(skin.id === 'default') isOwned = true;
        else if(skin.id === 'blue') isOwned = ownedSkins.blue;
        else if(skin.id === 'purple') isOwned = ownedSkins.purple;
        else if(skin.id === 'gold') isOwned = unlockedCount >= skin.requirement;
        else if(skin.id === 'rainbow') isOwned = ownedSkins.rainbow;
        else if(skin.id === 'ultra') isOwned = ownedSkins.ultra;
        else if(skin.id === 'legend') isOwned = currentRank >= 50;
        
        const isEquipped = currentSkin === skin.id;
        const card = document.createElement('div');
        card.className = `skin-card ${isOwned ? 'owned' : 'locked'} ${isEquipped ? 'equipped' : ''}`;
        card.innerHTML = `
            <div class="skin-icon">${skin.icon}</div>
            <div class="skin-name">${skin.name}</div>
            <div class="skin-desc">${skin.desc}</div>
            ${skin.effect ? `<div class="skin-effect">✨ ${skin.effect}</div>` : '<div class="skin-effect">😴 ללא יכולת מיוחדת</div>'}
            ${!isOwned ? `<div style="font-size:9px;color:#ffaa00;margin-top:5px;">🔒 ${skin.requirement === 'lootbox' ? 'נפתח בתיבות' : skin.requirement === 'ultra_lootbox' ? 'נפתח בתיבות ULTRA MYTHIC' : skin.requirement === 'rank50' ? 'דורש RANK 50' : `דורש ${skin.requirement} הישגים`}</div>` : ''}
            ${isEquipped ? '<div style="font-size:9px;color:#ff66ff;margin-top:5px;">✅ מצויד כעת</div>' : ''}
        `;
        if(isOwned && !isEquipped){
            card.onclick = () => equipSkin(skin.id);
        } else if(isEquipped){
            card.onclick = null;
        } else {
            card.onclick = null;
        }
        container.appendChild(card);
    }
}

function equipSkin(skinId){
    currentSkin = skinId;
    localStorage.setItem('currentSkin', currentSkin);
    updateSkinsUI();
    showAchievementPopup('🎨 SKIN EQUIPPED', `${skinId.toUpperCase()} skin is now active!`);
    showNotification(`🎨 ${skinId.toUpperCase()} skin equipped!`, 'info');
}

function openSkins(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('skins-screen').style.display='flex';
    updateSkinsUI();
}
function closeSkins(){
    document.getElementById('skins-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}

let currentReward = null;
let currentLootboxType = null;

function spawnLootboxParticles(rarityColor){
    const container = document.getElementById('lootbox-animation');
    if(!container) return;
    const colors = {common:'#4287f5',rare:'#42f5b6',epic:'#f5a742',legendary:'#f542d1',mythic:'#f54242',ultra:'#ffd700'};
    const color = colors[rarityColor] || '#00d2ff';
    for(let i=0;i<20;i++){
        const p = document.createElement('div');
        p.className = 'lootbox-particle';
        const angle = (Math.PI*2/20)*i;
        const dist = 60 + Math.random()*80;
        p.style.cssText = `left:50%;top:50%;background:${color};--px:${Math.cos(angle)*dist}px;--py:${Math.sin(angle)*dist}px;`;
        container.appendChild(p);
        setTimeout(()=>p.remove(), 800);
    }
}

function openLootbox(type){
    const price = {common:50, rare:150, epic:400, legendary:1000, mythic:2500, ultra:10000}[type];
    if(gemstones < price){
        showCustomAlert(`לא מספיק GEMSTONES! צריך ${price} 💎`);
        showNotification(`Not enough GEMSTONES! Need ${price} 💎`, 'warning');
        return;
    }
    if(type === 'ultra' && ultraMythicCount >= ULTRA_MYTHIC_LIMIT){
        showCustomAlert(`❗ ULTRA MYTHIC תיבות מוגבלות ל-${ULTRA_MYTHIC_LIMIT} בלבד! כבר פתחת את כולן.`);
        showNotification(`Ultra Mythic boxes limit reached (${ULTRA_MYTHIC_LIMIT}/5)`, 'warning');
        return;
    }
    gemstones -= price;
    saveGemstones();
    lootboxesOpened++;
    localStorage.setItem('lootboxesOpened', lootboxesOpened);
    
    const rewards = LOOTBOX_REWARDS[type];
    const reward = rewards[Math.floor(Math.random() * rewards.length)];
    currentReward = reward;
    currentLootboxType = type;
    
    document.getElementById('lootbox-animation').style.display='flex';
    document.getElementById('lootbox-result').style.display='none';
    document.getElementById('lootbox-close-btn').style.display='none';
    const box = document.getElementById('lootbox-element');
    box.className = 'lootbox';
    box.classList.add('rarity-' + type);
    box.style.animation = 'lootboxShake 1.5s ease-in-out';
    box.innerHTML = '📦';
    
    setTimeout(() => {
        box.style.animation = 'lootboxOpen 0.6s forwards';
        flashScreen();
        setTimeout(() => {
            box.style.display = 'none';
            spawnLootboxParticles(type);
            applyReward(reward);
            document.getElementById('lootbox-result').style.display = 'block';
            document.getElementById('lootbox-close-btn').style.display = 'inline-block';
            if(type === 'ultra') ultraMythicCount++;
            localStorage.setItem('ultraMythicCount', ultraMythicCount);
        }, 600);
    }, 1500);
}

function applyReward(reward){
    let resultText = '';
    let finalAmount = reward.amount || 0;
    if(reward.type === 'gem') finalAmount = Math.floor(finalAmount * gemMultiplier);
    
    switch(reward.type){
        case 'gem':
            gemstones += finalAmount;
            saveGemstones();
            resultText = `✨ זכית ב-${finalAmount} GEMSTONES! ✨`;
            showNotification(`🎁 Lootbox reward: ${finalAmount} GEMSTONES!`, 'success');
            break;
        case 'credit':
            totalCoins += reward.amount;
            localStorage.setItem('totalCoins', totalCoins);
            resultText = `💰 זכית ב-${formatNumber(reward.amount)} CREDITS! 💰`;
            showNotification(`💰 Lootbox reward: ${formatNumber(reward.amount)} CREDITS!`, 'success');
            break;
        case 'bomb':
            bombCount += reward.amount;
            localStorage.setItem('bombCount', bombCount);
            resultText = `💣 זכית ב-${reward.amount} BOMBS! 💣`;
            showNotification(`💣 Lootbox reward: ${reward.amount} BOMBS!`, 'success');
            break;
        case 'boost':
            activePowerUps[reward.effect + '_boost'] = {name:reward.name, endTime:Date.now() + reward.duration*1000};
            resultText = `⚡ ${reward.name} הופעל ל-${reward.duration} שניות! ⚡`;
            showNotification(`⚡ ${reward.name} activated for ${reward.duration}s!`, 'success');
            break;
        case 'perm_upgrade':
            if(reward.stat === 'damage'){
                damageLevel += reward.amount;
                localStorage.setItem('dmgLevel', damageLevel);
                resultText = `🔰 נזק קבוע +${reward.amount}! (עכשיו רמה ${damageLevel}) 🔰`;
                showNotification(`🔰 Permanent damage +${reward.amount}!`, 'success');
            } else if(reward.stat === 'fire'){
                fireLevel += reward.amount;
                localStorage.setItem('fireLevel', fireLevel);
                resultText = `🔥 קצב ירי קבוע +${reward.amount}! (עכשיו רמה ${fireLevel}) 🔥`;
                showNotification(`🔥 Permanent fire rate +${reward.amount}!`, 'success');
            }
            break;
        case 'perm_upgrade_all':
            damageLevel += reward.amount;
            fireLevel += reward.amount;
            localStorage.setItem('dmgLevel', damageLevel);
            localStorage.setItem('fireLevel', fireLevel);
            resultText = `⭐ כל השדרוגים הקבועים +${reward.amount}! ⭐`;
            showNotification(`⭐ All permanent upgrades +${reward.amount}!`, 'success');
            break;
        case 'skin':
            if(!ownedSkins[reward.skin]){
                ownedSkins[reward.skin] = true;
                localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
                resultText = `🎨 ${reward.name.toUpperCase()} נפתח! 🎨`;
                showNotification(`🎨 ${reward.name.toUpperCase()} skin unlocked!`, 'success');
                if(reward.skin === 'ultra'){
                    resultText = `👑👑 ULTRA MYTHIC SKIN (LIMITED) נפתחה! רק ${ULTRA_MYTHIC_LIMIT} קיימות! 👑👑`;
                }
                updateSkinsUI();
            } else {
                let duplicateGems = {blue:50, purple:100, gold:200, rainbow:400, ultra:1000}[reward.skin] || 100;
                gemstones += duplicateGems;
                saveGemstones();
                resultText = `🎨 כבר יש לך את הסקין! קיבלת ${duplicateGems} GEMSTONES במקום. 🎨`;
                showNotification(`🎨 Duplicate skin! +${duplicateGems} GEMSTONES`, 'info');
            }
            break;
        case 'unlock_ability':
            let unlockedOne = false;
            for(let key of Object.keys(RARE_ABILITIES)){
                const ab = RARE_ABILITIES[key];
                if(!ab.purchased && ab.unlockReq()){
                    ab.purchased = true;
                    localStorage.setItem('rareAbility_'+key, 'true');
                    resultText = `💠 COSMIC SHARD unlocked: ${ab.name}! 💠`;
                    showNotification(`💠 ${ab.name} unlocked by Cosmic Shard!`, 'success');
                    unlockedOne = true;
                    break;
                }
            }
            if(!unlockedOne){
                gemstones += 2000;
                saveGemstones();
                resultText = `💠 No locked abilities to unlock - received 2000 GEMSTONES instead! 💠`;
                showNotification(`💠 Cosmic Shard: +2000 GEMSTONES (no locked abilities)`, 'info');
            }
            // Ultra lootbox Cosmic Shard grants 3 skill points too
            if(reward.name && reward.name.includes('ULTIMATE')){
                skillPoints += 3;
                localStorage.setItem('skillPoints', skillPoints);
                resultText += ' + 3 SKILL POINTS!';
                showNotification('🌳 +3 Skill Points from Cosmic Shard Ultimate!', 'success');
            }
            break;
    }
    let isNewSkin = reward.type === 'skin' && !ownedSkins[reward.skin];
    const rewardRarityClass = reward.ultraRare ? 'rarity-ultra' : `rarity-${currentLootboxType}`;
    const ultraRareLabel = reward.ultraRare ? '<div style="color:#9b59b6;font-weight:bold;margin-top:6px;font-size:12px;">💠 ULTRA-RARE DROP 💠</div>' : '';
    document.getElementById('lootbox-result').innerHTML = `<div class="reward-card ${rewardRarityClass}"><div style="font-size:28px;margin-bottom:10px;">${reward.icon}</div><div style="font-size:16px;font-weight:bold;">${resultText}</div><div style="font-size:13px;color:#ffaa00;margin-top:10px;">${reward.name}</div><div style="font-size:11px;margin-top:4px;">${reward.rarity.toUpperCase()}</div>${ultraRareLabel}${isNewSkin ? '<div style="color:#00ffaa;font-weight:bold;margin-top:8px;font-size:14px;">✨ NEW!</div>' : ''}</div>`;
    updateShopUI();
    updateIndividualRewardsUI();
    checkAchievements();
}

function closeLootboxAnimation(){
    document.getElementById('lootbox-animation').style.display='none';
    const box = document.getElementById('lootbox-element');
    box.style.display = 'flex';
    box.style.animation = '';
    box.innerHTML = '📦';
    box.classList.remove('rarity-common', 'rarity-rare', 'rarity-epic', 'rarity-legendary', 'rarity-mythic', 'rarity-ultra');
    document.getElementById('lootbox-result').style.display = 'none';
    document.getElementById('lootbox-close-btn').style.display = 'none';
    currentLootboxType = null;
    currentReward = null;
}

function viewLootboxProbabilities(type){
    const rewards = LOOTBOX_REWARDS[type];
    if(!rewards) return;
    const rarityColors = {common:'#4287f5',rare:'#42f5b6',epic:'#f5a742',legendary:'#f542d1',mythic:'#f54242',ultra:'#ffd700'};
    const rarityNames = {common:'COMMON',rare:'RARE',epic:'EPIC',legendary:'LEGENDARY',mythic:'MYTHIC',ultra:'ULTRA MYTHIC'};
    let html = `<div style="position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.9);z-index:2000;display:flex;justify-content:center;align-items:center;" onclick="if(event.target===this)this.remove()">
        <div style="background:linear-gradient(135deg,#001a33,#000);border:2px solid ${rarityColors[type]||'#00d2ff'};border-radius:16px;padding:20px;min-width:300px;max-width:420px;max-height:80vh;overflow-y:auto;">
            <h3 style="color:${rarityColors[type]||'#00d2ff'};text-align:center;margin-bottom:12px;">📦 ${rarityNames[type]||type.toUpperCase()} LOOTBOX DROPS</h3>`;
    for(const r of rewards){
        const starStr = '⭐'.repeat(r.stars || 1);
        const isUltraRare = r.ultraRare;
        const ultraRareTag = isUltraRare ? '<span style="background:#9b59b6;color:#fff;font-size:8px;padding:1px 6px;border-radius:8px;margin-left:4px;">ULTRA-RARE</span>' : '';
        const ultraRareEffect = isUltraRare && r.ultraRareEffect ? `<div style="font-size:8px;color:#ffcc00;margin-top:2px;">✨ ${r.ultraRareEffect}</div>` : '';
        const borderStyle = isUltraRare ? 'border:2px solid #9b59b6;' : `border:1px solid ${rarityColors[r.rarity]||'#444'};`;
        html += `<div style="background:rgba(0,0,0,0.5);${borderStyle}border-radius:8px;padding:8px;margin:6px 0;display:flex;align-items:center;gap:8px;${isUltraRare ? 'background:rgba(155,89,182,0.15);' : ''}">
            <span style="font-size:18px;">${r.icon}</span>
            <div style="flex:1;">
                <div style="font-size:11px;color:#fff;">${r.name} ${ultraRareTag}</div>
                <div style="font-size:9px;color:${rarityColors[r.rarity]||'#888'};">${starStr} ${rarityNames[r.rarity]||r.rarity.toUpperCase()}</div>
                ${ultraRareEffect}
            </div>
            <div style="font-size:13px;color:${isUltraRare ? '#9b59b6' : '#ffcc00'};font-weight:bold;">${r.probability || '—'}%</div>
        </div>`;
    }
    html += `<button class="btn" style="margin-top:12px;width:100%;" onclick="this.closest('div[style]').parentElement.remove()">CLOSE</button></div></div>`;
    const overlay = document.createElement('div');
    overlay.innerHTML = html;
    document.body.appendChild(overlay);
}

function openRNGShop(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('rng-shop-screen').style.display='flex';
    updateGemUI();
    updateIndividualRewardsUI();
}

function closeRNGShop(){
    document.getElementById('rng-shop-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}

function buyIndividualReward(reward){
    if(gemstones < reward.price){
        showCustomAlert(`לא מספיק GEMSTONES! צריך ${reward.price} 💎`);
        showNotification(`Not enough GEMSTONES! Need ${reward.price} 💎`, 'warning');
        return;
    }
    gemstones -= reward.price;
    saveGemstones();
    
    if(reward.type === 'skin'){
        if(!ownedSkins[reward.skin]){
            ownedSkins[reward.skin] = true;
            localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
            showCustomAlert(`🎨 ${reward.name.toUpperCase()} נרכש! 🎨`);
            showNotification(`🎨 ${reward.name.toUpperCase()} purchased!`, 'success');
            updateSkinsUI();
        } else {
            let duplicateGems = {blue:25, purple:50, gold:100, rainbow:200}[reward.skin] || 50;
            gemstones += duplicateGems;
            saveGemstones();
            showCustomAlert(`🎨 כבר יש לך את הסקין! קיבלת ${duplicateGems} GEMSTONES במקום. 🎨`);
            showNotification(`Duplicate skin! +${duplicateGems} GEMSTONES`, 'info');
        }
    } else if(reward.type === 'perm_upgrade'){
        if(reward.stat === 'damage'){
            damageLevel += reward.amount;
            localStorage.setItem('dmgLevel', damageLevel);
            showCustomAlert(`🔰 נזק קבוע +${reward.amount}! (עכשיו רמה ${damageLevel}) 🔰`);
            showNotification(`🔰 Permanent damage +${reward.amount}!`, 'success');
        } else if(reward.stat === 'fire'){
            fireLevel += reward.amount;
            localStorage.setItem('fireLevel', fireLevel);
            showCustomAlert(`🔥 קצב ירי קבוע +${reward.amount}! (עכשיו רמה ${fireLevel}) 🔥`);
            showNotification(`🔥 Permanent fire rate +${reward.amount}!`, 'success');
        }
    } else if(reward.type === 'gem'){
        let finalAmount = reward.amount * gemMultiplier;
        gemstones += finalAmount;
        saveGemstones();
        showCustomAlert(`✨ קיבלת ${finalAmount} GEMSTONES! ✨`);
        showNotification(`✨ +${finalAmount} GEMSTONES!`, 'success');
    } else if(reward.type === 'credit'){
        totalCoins += reward.amount;
        localStorage.setItem('totalCoins', totalCoins);
        showCustomAlert(`💰 קיבלת ${formatNumber(reward.amount)} CREDITS! 💰`);
        showNotification(`💰 +${formatNumber(reward.amount)} CREDITS!`, 'success');
    }
    updateGemUI();
    updateShopUI();
    updateIndividualRewardsUI();
    checkAchievements();
}

function updateIndividualRewardsUI(){
    const container = document.getElementById('individual-rewards');
    if(!container) return;
    container.innerHTML = '';
    for(const reward of INDIVIDUAL_REWARDS){
        const card = document.createElement('div');
        card.className = `card rarity-${reward.rarity} tooltip`;
        card.innerHTML = `<div style="font-size:20px;">${reward.icon}</div>
                         <div>${reward.name}</div>
                         <div style="color:#ffaa00;">${reward.price} 💎</div>
                         <div style="font-size:9px;color:#aaa;">${reward.rarity.toUpperCase()}</div>
                         <div class="tooltip-text">${reward.type === 'skin' ? 'פותח סקין חדש!' : reward.type === 'perm_upgrade' ? 'שדרוג קבוע!' : reward.type === 'gem' ? 'מקבל GEMSTONES!' : 'מקבל CREDITS!'}</div>`;
        card.onclick = () => buyIndividualReward(reward);
        container.appendChild(card);
    }
}

// HUB FUNCTIONS
function openGameSelect(){
    document.getElementById('main-hub').style.display='none';
    document.getElementById('game-select-screen').style.display='flex';
}
function openUpdateLog(){
    document.getElementById('main-hub').style.display='none';
    document.getElementById('update-log-screen').style.display='flex';
    renderUpdateLog();
}
function backToHub(){
    document.getElementById('main-hub').style.display='flex';
    document.getElementById('game-select-screen').style.display='none';
    document.getElementById('update-log-screen').style.display='none';
    document.getElementById('start-screen').style.display='none';
    document.getElementById('settings-screen').style.display='none';
    document.getElementById('tutorial-screen').style.display='none';
    document.getElementById('reset-modal').style.display='none';
    // Restore normal settings origin
    settingsOpenedFrom = 'start-screen';
    updateHubUI();
}
function updateHubUI(){
    document.getElementById('hub-hi').innerText = formatNumber(hiScore);
    document.getElementById('hub-coins').innerText = formatNumber(totalCoins);
    document.getElementById('hub-gems').innerText = formatNumber(gemstones);
    document.getElementById('hub-kills').innerText = formatNumber(totalKills);
    document.getElementById('hub-skin').innerText = skinUnlocked ? t('unlocked') + ' (GOLDEN)' : t('locked');
    const unlockedCount = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    const percent = Math.min(100, (unlockedCount/40)*100);
    document.getElementById('hub-skin-progress-fill').style.width = percent+'%';
    document.getElementById('hub-skin-percent').innerHTML = unlockedCount+'/40 ACHIEVEMENTS';
    document.getElementById('stat-crits').innerText = criticalHitsCount;
    document.getElementById('stat-crit-rate').innerText = Math.floor(critChance * 100);
    document.getElementById('stat-damage').innerText = formatNumber(totalDamageDealt);
    document.getElementById('stat-od').innerText = totalOverdriveUses;
    document.getElementById('stat-bosses').innerText = bossesKilled;
}
function selectGame(game){
    if(game === 'defender'){
        currentGame = 'defender';
        document.getElementById('game-select-screen').style.display='none';
        document.getElementById('start-screen').style.display='flex';
        updateMainMenuUI();
        updateGemUI();
    } else {
        showCustomAlert('משחק זה עדיין בפיתוח! יגיע בקרוב...');
    }
}
function backToGameSelect(){
    gameState='MENU';
    isPaused=false;
    if(player) player=null;
    ['game-over','pause-screen','ui-hud','score-hud','combo-small','od-btn','pause-btn','powerup-bar','wave-banner','boss-warning','event-banner','ascend-screen','start-screen','game-timer','rank-badge','combo-meter','wave-progress'].forEach(id=>{
        const el=document.getElementById(id);
        if(el) el.style.display='none';
    });
    document.getElementById('ability-btns').style.display='none';
    closeHomeAbilityPanel();
    document.getElementById('vignette').style.display='none';
    document.getElementById('crosshair').style.display='none';
    document.getElementById('auto-fire-btn').style.display='none';
    document.getElementById('fps-counter').style.display='none';
    document.getElementById('game-select-screen').style.display='flex';
}
function updateMainMenuUI(){
    document.getElementById('menu-hi').innerText=formatNumber(hiScore);
    document.getElementById('menu-coins').innerText=formatNumber(totalCoins);
    document.getElementById('menu-kills').innerText=formatNumber(totalKills);
    updateSkinProgressUI();
}
function confirmReset(){
    document.getElementById('reset-modal').style.display = 'flex';
}
function confirmResetAction(){
    document.getElementById('reset-modal').style.display = 'none';
    resetUpgradesOnly();
}
function closeResetModal(){
    document.getElementById('reset-modal').style.display = 'none';
}
function resetUpgradesOnly(){
    let refund = 0;
    refund += fireLevel * 150;
    refund += (damageLevel - 1) * 200;
    refund += droneCount * 500;
    refund += bombCount * 300;
    refund += resourceCollectionLevel * 350;
    refund += weaponEnhancementLevel * 450;
    if(hasShieldUpgrade) refund += 250;
    if(hasSpreadShot) refund += 350;
    if(hasLaser) refund += 600;
    totalCoins += refund;
    fireLevel = 0;
    damageLevel = 1;
    droneCount = 0;
    bombCount = 0;
    resourceCollectionLevel = 0;
    weaponEnhancementLevel = 0;
    hasShieldUpgrade = false;
    hasSpreadShot = false;
    hasLaser = false;
    localStorage.setItem('totalCoins', totalCoins);
    localStorage.setItem('fireLevel', fireLevel);
    localStorage.setItem('dmgLevel', damageLevel);
    localStorage.setItem('droneCount', droneCount);
    localStorage.setItem('bombCount', bombCount);
    localStorage.setItem('resourceCollectionLevel', resourceCollectionLevel);
    localStorage.setItem('weaponEnhancementLevel', weaponEnhancementLevel);
    localStorage.setItem('hasShieldUpgrade', hasShieldUpgrade);
    localStorage.setItem('hasSpreadShot', hasSpreadShot);
    localStorage.setItem('hasLaser', hasLaser);
    updateMainMenuUI();
    updateShopUI();
    updateSkinProgressUI();
    updateHubUI();
    showCustomAlert(`✅ כל השדרוגים אופסו!\n💰 קיבלת חזרה ${formatNumber(refund)} קרדיטים!\nסך קרדיטים נוכחי: ${formatNumber(totalCoins)}`);
    tn('noti_reset', 'info', formatNumber(refund));
}

// GAME STATE VARIABLES
const canvas=document.getElementById('gameCanvas'),ctx=canvas.getContext('2d');
const nebulaCanvas=document.getElementById('nebulaCanvas'),nCtx=nebulaCanvas.getContext('2d');
let width,height,mouseX=0,mouseY=0,kills=0;
let totalCoins=parseInt(localStorage.getItem('totalCoins'))||0;
let hiScore=parseInt(localStorage.getItem('hiScore'))||0;
let totalKills=parseInt(localStorage.getItem('totalKills'))||0;
let fireLevel=parseInt(localStorage.getItem('fireLevel'))||0;
let damageLevel=parseInt(localStorage.getItem('dmgLevel'))||1;
let hasShieldUpgrade=localStorage.getItem('hasShieldUpgrade')==='true';
let hasSpreadShot=localStorage.getItem('hasSpreadShot')==='true';
let hasLaser=localStorage.getItem('hasLaser')==='true';
let droneCount=Math.min(180, parseInt(localStorage.getItem('droneCount'))||0);
let bombCount=parseInt(localStorage.getItem('bombCount'))||0;
let achievements; try { achievements = JSON.parse(localStorage.getItem('achievements')||'{}'); } catch(e) { achievements = {}; }
let skinUnlocked=localStorage.getItem('skinUnlocked')==='true';
let totalUpgradesBought=parseInt(localStorage.getItem('totalUpgradesBought'))||0;
let guardianDefeated=localStorage.getItem('guardianDefeated')==='true';
let endlessMode=false;
let gameState='LOADING';
let score=0,health=100,maxHealth=100,xp=0,level=1;
let odCharge=0,isOD=false,odTimer=0;
let lastFire=0,shake=0,combo=1,comboTimer=0;
let bossWarningTimer=0,waveBannerTimer=0;
let wave=1,waveKillGoal=12,waveKills=0,waveTriggered=false;
let player=null;
let stars=[],bullets=[],enemies=[],eBullets=[],particles=[],items=[],floats=[];
let boss=null,isPaused=false;
let activePowerUps={};
let odActivations=0,waveNoDamage=true,bossesKilled=0;
let perfectWavesCount=0;
let startTime=0;
let totalOverdriveUses=0;
let totalBombsUsed=0;
let maxCombo=0;
let totalSynapseActivations=0;
let synapseKills=0;
let psychedeliaCollected=0;
let totalEventsTriggered=0;
let totalRiftsTriggered=0;
let totalMeteorsDestroyed=0;
let apocalypseTriggers=0;
let voidTriggers=0;
let blackHolePieces=0;
let cosmicCollapseCount=0;
let primordialRageCount=0;
let deathTouchCount=0;
let chaosRealmCount=0;
let eventCooldown=0;
let eventQueue=[];
let asteroidBeltActive=false;
let asteroidBeltTimer=0;
let asteroids=[];
let supplyDropActive=false;
let supplyDropTimer=0;
let supplyDropCrate=null;
let greenAuraActive=false;
let divineFlashTimer=0;
let ascendTriggered=false;
let timerInterval = null;
let animationId = null;

// SKIN EFFECTS
let skinCreditMultiplier = 1;
let skinDamageMultiplier = 1;
let skinFireRateMultiplier = 1;

function updateSkinEffects(){
    skinCreditMultiplier = 1;
    skinDamageMultiplier = 1;
    skinFireRateMultiplier = 1;
    const rankBonus = RANK_BONUS[currentRank] || 0;
    skinCreditMultiplier = 1 + (rankBonus / 100);
    skinDamageMultiplier = 1 + (rankBonus / 100);
    skinFireRateMultiplier = 1 + (rankBonus / 100);
    if(currentSkin === 'gold'){
        skinCreditMultiplier *= 1.25;
        skinDamageMultiplier *= 1.2;
    } else if(currentSkin === 'ultra'){
        skinCreditMultiplier *= 1.5;
        skinDamageMultiplier *= 1.5;
        skinFireRateMultiplier *= 1.2;
    } else if(currentSkin === 'legend'){
        skinCreditMultiplier *= 2;
        skinDamageMultiplier *= 2;
        skinFireRateMultiplier *= 1.5;
    }
}

// MODES & EVENTS
let synapseActive=false;
let synapseTimer=0;
let synapsePsychedeliaCount=0;
let synapseBackgroundHue=0;
let activeEvent=null;
let eventTimer=0;
let meteors=[];
let riftActive=false;
let riftMultiplier=1;
let apocalypseActive=false;
let voidActive=false;
let voidTimer=0;
let guardian=null;
let cosmicCollapseActive=false;
let primordialRageActive=false;
let blackHoleActive=false;
let blackHoleCenter={x:0,y:0};
let chaosRealmActive=false;
let chaosHue=0;
let selectedQuantities={fire:1,dmg:1,drone:1,bomb:1};

// EVENT VARIABLES
let timeWarpActive=false;
let timeWarpTimer=0;
let goldRushActive=false;
let goldRushTimer=0;
let divineActive=false;
let divineTimer=0;
let bugEventActive=false;
let bugEventTimer=0;
let bugEventGlitch=false;
let stableCycleActive=false;
let stableCycleTimer=0;
let tidalWaveActive=false;
let tidalWaveTimer=0;
let masqueradeActive=false;
let masqueradeTimer=0;
let soulHarvestActive=false;
let soulHarvestTimer=0;
let soulHarvestSouls=[];
let frozenTimeActive=false;
let frozenTimeTimer=0;
let crystalRainActive=false;
let crystalRainTimer=0;
let shadowCloneActive=false;
let shadowCloneTimer=0;
let shadowClone = null;
let lightningStormActive=false;
let lightningStormTimer=0;
let lightningBolts=[];
let luckyDrawActive=false;
let mysteryBoxActive=false;
let doomsDayActive=false;
let royalBlessingActive=false;
let royalBlessingTimer=0;
let starfallActive=false;
let starfallTimer=0;
let starfallStars=[];
let infernoActive=false;
let infernoTimer=0;
let chainLightningActive=false;
let chainLightningTimer=0;
let barrierActive=false;
let barrierTimer=0;
let barrierHp=0;
let soulReaperActive=false;
let soulReaperTimer=0;
let soulReaperSoul=null;
let gamblerActive=false;
let vortexActive=false;
let vortexTimer=0;
let vortexCenter={x:0,y:0};
let kingsBlessingActive=false;
let kingsBlessingTimer=0;
let prismActive=false;
let prismTimer=0;
let abyssActive=false;
let abyssTimer=0;
let abyssCenter={x:0,y:0};
let doppelgangerActive=false;
let doppelgangerTimer=0;
let doppelgangerClone = null;
let supernovaActive=false;
let supernovaTimer=0;
let crystals = [];

// METEOR CLASS
class Meteor {
    constructor(x, y) {
        this.x = x; this.y = y; this.radius = 14; this.hp = 10; this.maxHp = 10;
        this.vx = (Math.random() - 0.5) * 2; this.vy = 3 + Math.random() * 2;
    }
    update() { this.x += this.vx; this.y += this.vy; }
    draw() {
        ctx.fillStyle = '#aa6644'; ctx.shadowBlur = 10;
        ctx.beginPath(); ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2); ctx.fill();
        ctx.fillStyle = '#ff8844'; ctx.fillRect(this.x - 4, this.y - 8, 8, 5);
        ctx.fillStyle = '#222'; ctx.fillRect(this.x - 10, this.y - 3, 20, 4);
        ctx.fillStyle = `hsl(${(this.hp / this.maxHp) * 30}, 80%, 50%)`;
        ctx.fillRect(this.x - 10, this.y - 3, (this.hp / this.maxHp) * 20, 4);
    }
}

// GUARDIAN CLASS
class Guardian {
    constructor() {
        this.x = width/2; this.y = -200; this.hp = 10000; this.maxHp = 10000;
        this.r = 120; this.speed = 0.3; this.lastShot = 0; this.angle = 0;
    }
    update() {
        this.y += this.speed; if(this.y > 150) this.y = 150;
        this.x = width/2 + Math.sin(Date.now()/600) * 80;
        this.angle += 0.02;
        if(Date.now() - this.lastShot > 400){
            this.lastShot = Date.now();
            for(let i=0;i<12;i++){
                let ang = i * Math.PI*2/12 + this.angle;
                eBullets.push({x:this.x + Math.cos(ang)*70, y:this.y + 50, vx:Math.cos(ang)*4, vy:Math.sin(ang)*4});
            }
        }
    }
    draw() {
        ctx.save(); ctx.translate(this.x, this.y);
        ctx.shadowBlur = 30; ctx.shadowColor = 'gold';
        ctx.fillStyle = '#ffd700'; ctx.beginPath(); ctx.arc(0,0,100,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = '#ffaa00'; ctx.beginPath(); ctx.arc(0,0,80,0,Math.PI*2); ctx.fill();
        ctx.fillStyle = '#fff'; ctx.font = 'bold 40px Segoe UI'; ctx.fillText('👑', -25, 20);
        ctx.fillStyle = '#222'; ctx.fillRect(-100, -140, 200, 12);
        ctx.fillStyle = `hsl(${(this.hp/this.maxHp)*60},100%,50%)`;
        ctx.fillRect(-100, -140, (this.hp/this.maxHp)*200, 12);
        ctx.restore();
    }
}

// DOPPELGANGER CLONE CLASS
class DoppelgangerClone {
    constructor(){
        this.x = player ? player.x - 60 : 0;
        this.y = player ? player.y : 0;
        this.r = 25;
        this.lastShot = 0;
    }
    update(){
        if(player){
            this.x = player.x - 60;
            this.y = player.y;
        }
        if(Date.now() - this.lastShot > 200){
            this.lastShot = Date.now();
            let power = damageLevel * 0.7;
            bullets.push(new Bullet(this.x, this.y-20, power, '#ff88ff'));
        }
    }
    draw(){
        ctx.save(); ctx.translate(this.x, this.y);
        ctx.fillStyle = '#ff88ff'; ctx.shadowBlur = 15;
        ctx.beginPath();
        ctx.moveTo(0,-25);ctx.lineTo(20,15);ctx.lineTo(8,15);ctx.lineTo(8,25);
        ctx.lineTo(-8,25);ctx.lineTo(-8,15);ctx.lineTo(-20,15);ctx.closePath();ctx.fill();
        ctx.fillStyle = 'rgba(255,136,255,0.5)';
        ctx.beginPath();ctx.ellipse(0,-5,4,10,0,0,Math.PI*2);ctx.fill();
        ctx.restore();
    }
}

// ACHIEVEMENTS LIST (abbreviated for space - same as before)
const ACHIEVEMENTS_LIST = [
    {id:'first_kill', name:'FIRST BLOOD', desc:'Destroy your first enemy', gems:5, difficulty:'easy'},
    {id:'combo10', name:'COMBO MASTER', desc:'Reach x10 combo', gems:10, difficulty:'easy'},
    {id:'wave5', name:'VETERAN', desc:'Survive to wave 5', gems:10, difficulty:'easy'},
    {id:'score5k', name:'RISING STAR', desc:'Score 5,000 points', gems:15, difficulty:'easy'},
    {id:'score25k', name:'GALACTIC HERO', desc:'Score 25,000 points', gems:25, difficulty:'medium'},
    {id:'overdrive3', name:'SPEED DEMON', desc:'Activate Overdrive 3 times', gems:15, difficulty:'easy'},
    {id:'nodmg_wave', name:'UNTOUCHABLE', desc:'Complete a wave without damage', gems:20, difficulty:'medium'},
    {id:'boss1', name:'BOSS SLAYER', desc:'Defeat your first boss', gems:20, difficulty:'easy'},
    {id:'rich', name:'RICH', desc:'Earn 10,000 total credits', gems:30, difficulty:'medium'},
    {id:'pyro', name:'PYROMANIAC', desc:'Fire Rate level 10', gems:25, difficulty:'medium'},
    {id:'warmonger', name:'WARMONGER', desc:'Damage level 10', gems:25, difficulty:'medium'},
    {id:'invincible', name:'INVINCIBLE', desc:'Buy Ion Shield', gems:20, difficulty:'easy'},
    {id:'drone_army', name:'DRONE ARMY', desc:'Own 5 drones', gems:25, difficulty:'medium'},
    {id:'demolition', name:'DEMOLITION', desc:'Use 10 bombs in one game', gems:30, difficulty:'medium'},
    {id:'overcharged', name:'OVERCHARGED', desc:'Activate Overdrive 10 times total', gems:35, difficulty:'medium'},
    {id:'kills100', name:'100 KILLS', desc:'100 total kills', gems:20, difficulty:'easy'},
    {id:'kills500', name:'500 KILLS', desc:'500 total kills', gems:40, difficulty:'medium'},
    {id:'legendary', name:'LEGENDARY', desc:'Reach Rank 15', gems:30, difficulty:'medium'},
    {id:'no_mercy', name:'NO MERCY', desc:'Kill 50 enemies in one wave', gems:35, difficulty:'hard'},
    {id:'laser_master', name:'LASER MASTER', desc:'Buy Laser Beam', gems:30, difficulty:'medium'},
    {id:'perfect_wave', name:'PERFECT WAVE', desc:'Complete 3 waves without damage', gems:40, difficulty:'hard'},
    {id:'speedrun', name:'SPEEDRUN', desc:'Complete 5 waves in 3 minutes', gems:35, difficulty:'hard'},
    {id:'millionaire', name:'MILLIONAIRE', desc:'Score 1,000,000 points', gems:50, difficulty:'hard'},
    {id:'boss_genocide', name:'BOSS GENOCIDE', desc:'Kill 10 bosses', gems:45, difficulty:'hard'},
    {id:'true_god', name:'TRUE GOD', desc:'Reach Rank 30', gems:50, difficulty:'hard'},
    {id:'combo_god', name:'COMBO GOD', desc:'Reach x30 combo', gems:40, difficulty:'hard'},
    {id:'max_out', name:'MAX OUT', desc:'Upgrade any stat to level 50', gems:60, difficulty:'extreme'},
    {id:'synapse_activate', name:'SYNAPSE ACTIVATED', desc:'Activate Synapse Mode 3 times', gems:45, difficulty:'hard'},
    {id:'time_lord', name:'TIME LORD', desc:'Kill 50 enemies during Synapse Mode', gems:50, difficulty:'hard'},
    {id:'psychedelic', name:'PSYCHEDELIC', desc:'Collect 10 Psychedelias', gems:30, difficulty:'medium'},
    {id:'decimator', name:'DECIMATOR', desc:'Kill 200 enemies in one game', gems:40, difficulty:'hard'},
    {id:'immortal', name:'IMMORTAL', desc:'Complete a wave without losing HP', gems:35, difficulty:'medium'},
    {id:'survivor', name:'SURVIVOR', desc:'Reach wave 15', gems:25, difficulty:'medium'},
    {id:'god_of_war', name:'GOD OF WAR', desc:'Reach Rank 50', gems:75, difficulty:'extreme'},
    {id:'infinite_power', name:'INFINITE POWER', desc:'Charge Overdrive to 200%', gems:50, difficulty:'hard'},
    {id:'shopaholic', name:'SHOPAHOLIC', desc:'Buy 50 upgrades total', gems:40, difficulty:'hard'},
    {id:'credit_farm', name:'CREDIT FARM', desc:'Collect 50,000 total credits', gems:45, difficulty:'hard'},
    {id:'meteor_slayer', name:'METEOR SLAYER', desc:'Destroy 50 meteors', gems:35, difficulty:'medium'},
    {id:'rift_walker', name:'RIFT WALKER', desc:'Trigger Dimension Rift 5 times', gems:40, difficulty:'hard'},
    {id:'event_master', name:'EVENT MASTER', desc:'Experience 10 total events', gems:30, difficulty:'medium'},
    {id:'godlike', name:'GODLIKE', desc:'Reach wave 50', gems:60, difficulty:'hard'},
    {id:'unstoppable', name:'UNSTOPPABLE', desc:'Kill 1000 enemies in one game', gems:80, difficulty:'extreme'},
    {id:'perfect_run', name:'PERFECT RUN', desc:'Complete 10 consecutive waves without damage', gems:70, difficulty:'extreme'},
    {id:'ultimate_power', name:'ULTIMATE POWER', desc:'Reach Rank 100', gems:100, difficulty:'extreme'},
    {id:'zero_damage', name:'ZERO DAMAGE', desc:'Complete a full game without taking any damage', gems:150, difficulty:'mythic'},
    {id:'speed_god', name:'SPEED GOD', desc:'Complete 30 waves in under 10 minutes', gems:80, difficulty:'extreme'},
    {id:'billionaire', name:'BILLIONAIRE', desc:'Score 10,000,000 points', gems:120, difficulty:'extreme'},
    {id:'true_survivor', name:'TRUE SURVIVOR', desc:'Play for 60 minutes straight', gems:100, difficulty:'extreme'},
    {id:'omega_boss', name:'OMEGA BOSS', desc:'Kill 50 total bosses', gems:90, difficulty:'extreme'},
    {id:'impossible', name:'IMPOSSIBLE', desc:'Reach wave 100', gems:150, difficulty:'mythic'},
    {id:'meteor_master', name:'METEOR MASTER', desc:'Destroy 200 meteors', gems:70, difficulty:'hard'},
    {id:'rift_god', name:'RIFT GOD', desc:'Trigger Dimension Rift 15 times', gems:80, difficulty:'extreme'},
    {id:'event_collector', name:'EVENT COLLECTOR', desc:'Experience 25 events', gems:60, difficulty:'hard'},
    {id:'fast_killer', name:'FAST KILLER', desc:'Kill 10 enemies within 3 seconds', gems:35, difficulty:'medium'},
    {id:'rich_lord', name:'RICH LORD', desc:'Collect 200,000 total credits', gems:70, difficulty:'hard'},
    {id:'endless', name:'ENDLESS', desc:'Reach wave 25', gems:30, difficulty:'medium'},
    {id:'combo_legend', name:'COMBO LEGEND', desc:'Reach x50 combo', gems:60, difficulty:'hard'},
    {id:'overdrive_god', name:'OVERDRIVE GOD', desc:'Activate Overdrive 50 times', gems:70, difficulty:'hard'},
    {id:'drone_master', name:'DRONE MASTER', desc:'Own 10 drones', gems:50, difficulty:'hard'},
    {id:'fire_god', name:'FIRE GOD', desc:'Fire Rate level 30', gems:60, difficulty:'hard'},
    {id:'damage_god', name:'DAMAGE GOD', desc:'Damage level 30', gems:60, difficulty:'hard'},
    {id:'drone_overlord', name:'DRONE OVERLORD', desc:'Own 180 drones (max)', gems:100, difficulty:'extreme'},
    {id:'apocalypse_survivor', name:'APOCALYPSE SURVIVOR', desc:'Survive Apocalypse Mode 3 times', gems:60, difficulty:'hard'},
    {id:'void_walker', name:'VOID WALKER', desc:'Trigger Void Mode 5 times', gems:70, difficulty:'hard'},
    {id:'guardian_slayer', name:'GUARDIAN SLAYER', desc:'Defeat The Guardian', gems:200, difficulty:'mythic'},
    {id:'true_guardian', name:'TRUE GUARDIAN', desc:'Defeat The Guardian 10 times', gems:500, difficulty:'mythic'},
    {id:'apocalypse_master', name:'APOCALYPSE MASTER', desc:'Kill 500 enemies during Apocalypse Mode', gems:80, difficulty:'extreme'},
    {id:'void_assassin', name:'VOID ASSASSIN', desc:'Kill 200 enemies during Void Mode', gems:80, difficulty:'extreme'},
    {id:'event_god', name:'EVENT GOD', desc:'Experience 50 events', gems:80, difficulty:'extreme'},
    {id:'credit_king', name:'CREDIT KING', desc:'Collect 1,000,000 total credits', gems:100, difficulty:'extreme'},
    {id:'score_king', name:'SCORE KING', desc:'Score 100,000,000 points', gems:150, difficulty:'mythic'},
    {id:'wave_warrior', name:'WAVE WARRIOR', desc:'Reach wave 200', gems:120, difficulty:'extreme'},
    {id:'immortal_god', name:'IMMORTAL GOD', desc:'Complete 50 waves without damage', gems:150, difficulty:'mythic'},
    {id:'perfect_game', name:'PERFECT GAME', desc:'Complete a full game without losing any HP', gems:200, difficulty:'mythic'},
    {id:'speed_demon', name:'SPEED DEMON', desc:'Complete 50 waves in under 15 minutes', gems:100, difficulty:'extreme'},
    {id:'overdrive_legend', name:'OVERDRIVE LEGEND', desc:'Activate Overdrive 200 times', gems:120, difficulty:'extreme'},
    {id:'bomb_master', name:'BOMB MASTER', desc:'Use 100 bombs total', gems:80, difficulty:'extreme'},
    {id:'cosmic_collapse', name:'COSMIC COLLAPSE', desc:'Experience Cosmic Collapse event', gems:50, difficulty:'hard'},
    {id:'primordial_rage', name:'PRIMORDIAL RAGE', desc:'Experience Primordial Rage event', gems:150, difficulty:'mythic'},
    {id:'black_hole', name:'BLACK HOLE', desc:'Activate Black Hole event', gems:60, difficulty:'hard'},
    {id:'cosmic_master', name:'COSMIC MASTER', desc:'Experience Cosmic Collapse 5 times', gems:80, difficulty:'extreme'},
    {id:'primordial_god', name:'PRIMORDIAL GOD', desc:'Experience Primordial Rage 3 times', gems:300, difficulty:'mythic'},
    {id:'black_hole_god', name:'BLACK HOLE GOD', desc:'Activate Black Hole 10 times', gems:120, difficulty:'extreme'},
    {id:'event_legend', name:'EVENT LEGEND', desc:'Experience 100 events', gems:120, difficulty:'extreme'},
    {id:'wave_500', name:'WAVE 500', desc:'Reach wave 500', gems:200, difficulty:'mythic'},
    {id:'rank_500', name:'RANK 500', desc:'Reach Rank 500', gems:200, difficulty:'mythic'},
    {id:'kills_10000', name:'10000 KILLS', desc:'Kill 10,000 enemies in one game', gems:250, difficulty:'mythic'},
    {id:'score_billion', name:'SCORE BILLION', desc:'Score 1,000,000,000 points', gems:300, difficulty:'mythic'},
    {id:'credit_billion', name:'CREDIT BILLION', desc:'Collect 100,000,000 total credits', gems:250, difficulty:'mythic'},
    {id:'perfect_100', name:'PERFECT 100', desc:'Complete 100 waves without damage', gems:250, difficulty:'mythic'},
    {id:'speed_100', name:'SPEED 100', desc:'Complete 100 waves in under 30 minutes', gems:200, difficulty:'mythic'},
    {id:'overdrive_1000', name:'OVERDRIVE 1000', desc:'Activate Overdrive 1000 times', gems:300, difficulty:'mythic'},
    {id:'bomb_1000', name:'BOMB 1000', desc:'Use 1000 bombs total', gems:200, difficulty:'mythic'},
    {id:'drone_400', name:'DRONE 180', desc:'Own 180 drones (max)', gems:150, difficulty:'mythic'},
    {id:'fire_100', name:'FIRE 100', desc:'Fire Rate level 100', gems:200, difficulty:'mythic'},
    {id:'damage_100', name:'DAMAGE 100', desc:'Damage level 100', gems:200, difficulty:'mythic'},
    {id:'death_touch', name:'DEATH\'S TOUCH', desc:'Experience Death\'s Touch event', gems:250, difficulty:'mythic'},
    {id:'chaos_realm', name:'CHAOS REALM', desc:'Experience Chaos Realm event', gems:200, difficulty:'mythic'},
    {id:'death_master', name:'DEATH MASTER', desc:'Experience Death\'s Touch 3 times', gems:500, difficulty:'mythic'},
    {id:'chaos_master', name:'CHAOS MASTER', desc:'Experience Chaos Realm 3 times', gems:400, difficulty:'mythic'},
    {id:'reaper_call', name:'REAPER\'S CALL', desc:'Experience Reaper\'s Call event', gems:300, difficulty:'mythic'},
    {id:'time_warp', name:'TIME WARP', desc:'Experience Time Warp event', gems:250, difficulty:'mythic'},
    {id:'gold_rush', name:'GOLD RUSH', desc:'Experience Gold Rush event', gems:500, difficulty:'mythic'},
    {id:'divine_intervention', name:'DIVINE INTERVENTION', desc:'Experience Divine Intervention event', gems:1000, difficulty:'mythic'},
    {id:'bug_event', name:'BUG EVENT', desc:'Experience the mysterious Bug Event', gems:777, difficulty:'mythic'},
    {id:'stable_cycle', name:'STABLE CYCLE', desc:'Experience Stable Cycle event', gems:100, difficulty:'hard'},
    {id:'tidal_wave', name:'TIDAL WAVE', desc:'Experience Tidal Wave event', gems:150, difficulty:'mythic'},
    {id:'masquerade', name:'MASQUERADE', desc:'Experience Masquerade event', gems:200, difficulty:'mythic'},
    {id:'soul_harvest', name:'SOUL HARVEST', desc:'Experience Soul Harvest event', gems:250, difficulty:'mythic'},
    {id:'frozen_time', name:'FROZEN TIME', desc:'Experience Frozen Time event', gems:180, difficulty:'mythic'},
    {id:'crystal_rain', name:'CRYSTAL RAIN', desc:'Experience Crystal Rain event', gems:220, difficulty:'mythic'},
    {id:'shadow_clone', name:'SHADOW CLONE', desc:'Experience Shadow Clone event', gems:300, difficulty:'mythic'},
    {id:'lightning_storm', name:'LIGHTNING STORM', desc:'Experience Lightning Storm event', gems:200, difficulty:'mythic'},
    {id:'lucky_draw', name:'LUCKY DRAW', desc:'Experience Lucky Draw event', gems:250, difficulty:'mythic'},
    {id:'mystery_box', name:'MYSTERY BOX', desc:'Experience Mystery Box event', gems:280, difficulty:'mythic'},
    {id:'dooms_day', name:'DOOM\'S DAY', desc:'Experience Doom\'s Day event', gems:350, difficulty:'mythic'},
    {id:'royal_blessing', name:'ROYAL BLESSING', desc:'Experience Royal Blessing event', gems:400, difficulty:'mythic'},
    {id:'starfall', name:'STARFALL', desc:'Experience Starfall event', gems:180, difficulty:'mythic'},
    {id:'inferno', name:'INFERNO', desc:'Experience Inferno event', gems:200, difficulty:'mythic'},
    {id:'chain_lightning', name:'CHAIN LIGHTNING', desc:'Experience Chain Lightning event', gems:220, difficulty:'mythic'},
    {id:'barrier', name:'BARRIER', desc:'Experience Barrier event', gems:150, difficulty:'hard'},
    {id:'soul_reaper', name:'SOUL REAPER', desc:'Experience Soul Reaper event', gems:350, difficulty:'mythic'},
    {id:'gambler', name:'GAMBLER', desc:'Experience Gambler event', gems:400, difficulty:'mythic'},
    {id:'vortex', name:'VORTEX', desc:'Experience Vortex event', gems:280, difficulty:'mythic'},
    {id:'kings_blessing', name:'KING\'S BLESSING', desc:'Experience King\'s Blessing event', gems:450, difficulty:'mythic'},
    {id:'prism', name:'PRISM', desc:'Experience Prism event', gems:300, difficulty:'mythic'},
    {id:'abyss', name:'ABYSS', desc:'Experience Abyss event', gems:500, difficulty:'mythic'},
    {id:'doppelganger', name:'DOPPELGANGER', desc:'Experience Doppelganger event', gems:350, difficulty:'mythic'},
    {id:'supernova', name:'SUPERNOVA', desc:'Experience Supernova event', gems:600, difficulty:'mythic'},
    {id:'doppelganger_master', name:'DOPPELGANGER MASTER', desc:'Experience Doppelganger 5 times', gems:800, difficulty:'mythic'},
    {id:'supernova_master', name:'SUPERNOVA MASTER', desc:'Experience Supernova 5 times', gems:1200, difficulty:'mythic'},
    {id:'space_treasure', name:'SPACE TREASURE', desc:'Experience Space Treasure event', gems:50, difficulty:'medium'},
    {id:'space_treasure_master', name:'TREASURE HUNTER', desc:'Experience Space Treasure 5 times', gems:100, difficulty:'hard'},
    {id:'critical_hitter', name:'CRITICAL HITTER', desc:'Land 100 critical hits', gems:100, difficulty:'hard'},
    {id:'critical_master', name:'CRITICAL MASTER', desc:'Land 1000 critical hits', gems:300, difficulty:'extreme'},
    {id:'combo_master_50', name:'COMBO MASTER 50', desc:'Reach x50 combo', gems:150, difficulty:'hard'},
    {id:'combo_master_100', name:'COMBO MASTER 100', desc:'Reach x100 combo', gems:500, difficulty:'mythic'},
    {id:'settings_enthusiast', name:'SETTINGS ENTHUSIAST', desc:'Change graphics quality', gems:25, difficulty:'easy'},
    {id:'auto_save_hero', name:'AUTO-SAVE HERO', desc:'Trigger auto-save 10 times', gems:50, difficulty:'medium'},
    {id:'stat_enthusiast', name:'STAT ENTHUSIAST', desc:'Check stats panel 5 times', gems:30, difficulty:'easy'},
    {id:'damage_dealer', name:'DAMAGE DEALER', desc:'Deal 1,000,000 total damage', gems:200, difficulty:'extreme'},
    {id:'damage_god', name:'DAMAGE GOD', desc:'Deal 100,000,000 total damage', gems:1000, difficulty:'mythic'},
    {id:'perfect_game_plus', name:'PERFECT GAME PLUS', desc:'Complete a game without taking any damage on wave 50+', gems:500, difficulty:'mythic'},
    {id:'ultimate_collector', name:'ULTIMATE COLLECTOR', desc:'Own all skins', gems:2000, difficulty:'mythic'},
    {id:'absolute_perfection', name:'ABSOLUTE PERFECTION', desc:'Complete all achievements!', gems:10000, difficulty:'mythic'},
    {id:'streak_3', name:'DEDICATED', desc:'Play 3 days in a row', gems:30, difficulty:'easy'},
    {id:'streak_7', name:'COMMITTED', desc:'Play 7 days in a row', gems:80, difficulty:'medium'},
    {id:'streak_30', name:'LOYAL WARRIOR', desc:'Play 30 days in a row', gems:300, difficulty:'extreme'}
];

function getAchievementProgressValue(id){
    if(id==='first_kill') return kills>=1;
    if(id==='combo10') return combo>=10;
    if(id==='wave5') return wave>=5;
    if(id==='score5k') return score>=5000;
    if(id==='score25k') return score>=25000;
    if(id==='overdrive3') return odActivations>=3;
    if(id==='nodmg_wave') return waveNoDamage && wave>1;
    if(id==='boss1') return bossesKilled>=1;
    if(id==='rich') return totalCoins>=10000;
    if(id==='pyro') return fireLevel>=10;
    if(id==='warmonger') return damageLevel>=10;
    if(id==='invincible') return hasShieldUpgrade;
    if(id==='drone_army') return droneCount>=5;
    if(id==='demolition') return totalBombsUsed>=10;
    if(id==='overcharged') return totalOverdriveUses>=10;
    if(id==='kills100') return totalKills>=100;
    if(id==='kills500') return totalKills>=500;
    if(id==='legendary') return level>=15;
    if(id==='no_mercy') return waveKills>=50;
    if(id==='laser_master') return hasLaser;
    if(id==='perfect_wave') return perfectWavesCount>=3;
    if(id==='speedrun') return (wave>=5 && (Date.now()-startTime)<180000);
    if(id==='millionaire') return score>=1000000;
    if(id==='boss_genocide') return bossesKilled>=10;
    if(id==='true_god') return level>=30;
    if(id==='combo_god') return maxCombo>=30;
    if(id==='max_out') return fireLevel>=50||damageLevel>=50;
    if(id==='synapse_activate') return totalSynapseActivations>=3;
    if(id==='time_lord') return synapseKills>=50;
    if(id==='psychedelic') return psychedeliaCollected>=10;
    if(id==='decimator') return kills>=200;
    if(id==='immortal') return waveNoDamage && health===maxHealth;
    if(id==='survivor') return wave>=15;
    if(id==='god_of_war') return level>=50;
    if(id==='infinite_power') return odCharge>=200;
    if(id==='shopaholic') return totalUpgradesBought>=50;
    if(id==='credit_farm') return totalCoins>=50000;
    if(id==='meteor_slayer') return totalMeteorsDestroyed>=50;
    if(id==='rift_walker') return totalRiftsTriggered>=5;
    if(id==='event_master') return totalEventsTriggered>=10;
    if(id==='godlike') return wave>=50;
    if(id==='unstoppable') return kills>=1000;
    if(id==='perfect_run') return perfectWavesCount>=10;
    if(id==='ultimate_power') return level>=100;
    if(id==='zero_damage') return (wave>=5 && health===maxHealth);
    if(id==='speed_god') return (wave>=30 && (Date.now()-startTime)<600000);
    if(id==='billionaire') return score>=10000000;
    if(id==='true_survivor') return (Date.now()-startTime)>=3600000;
    if(id==='omega_boss') return bossesKilled>=50;
    if(id==='impossible') return wave>=100;
    if(id==='meteor_master') return totalMeteorsDestroyed>=200;
    if(id==='rift_god') return totalRiftsTriggered>=15;
    if(id==='event_collector') return totalEventsTriggered>=25;
    if(id==='fast_killer') return (waveKills>=10 && waveKills<15);
    if(id==='rich_lord') return totalCoins>=200000;
    if(id==='endless') return wave>=25;
    if(id==='combo_legend') return maxCombo>=50;
    if(id==='overdrive_god') return totalOverdriveUses>=50;
    if(id==='drone_master') return droneCount>=10;
    if(id==='fire_god') return fireLevel>=30;
    if(id==='damage_god') return damageLevel>=30;
    if(id==='drone_overlord') return droneCount>=180;
    if(id==='apocalypse_survivor') return apocalypseTriggers>=3;
    if(id==='void_walker') return voidTriggers>=5;
    if(id==='guardian_slayer') return guardianDefeated===true;
    if(id==='true_guardian') return (localStorage.getItem('guardianDefeatedCount')||0)>=10;
    if(id==='apocalypse_master') return apocalypseTriggers>=3 && kills>=500;
    if(id==='void_assassin') return voidTriggers>=5 && kills>=200;
    if(id==='event_god') return totalEventsTriggered>=50;
    if(id==='credit_king') return totalCoins>=1000000;
    if(id==='score_king') return score>=100000000;
    if(id==='wave_warrior') return wave>=200;
    if(id==='immortal_god') return perfectWavesCount>=50;
    if(id==='perfect_game') return (wave>=10 && health===maxHealth);
    if(id==='speed_demon') return (wave>=50 && (Date.now()-startTime)<900000);
    if(id==='overdrive_legend') return totalOverdriveUses>=200;
    if(id==='bomb_master') return totalBombsUsed>=100;
    if(id==='cosmic_collapse') return cosmicCollapseCount>=1;
    if(id==='primordial_rage') return primordialRageCount>=1;
    if(id==='black_hole') return blackHolePieces>=6;
    if(id==='cosmic_master') return cosmicCollapseCount>=5;
    if(id==='primordial_god') return primordialRageCount>=3;
    if(id==='black_hole_god') return (localStorage.getItem('blackHoleActivations')||0)>=10;
    if(id==='event_legend') return totalEventsTriggered>=100;
    if(id==='wave_500') return wave>=500;
    if(id==='rank_500') return level>=500;
    if(id==='kills_10000') return kills>=10000;
    if(id==='score_billion') return score>=1000000000;
    if(id==='credit_billion') return totalCoins>=100000000;
    if(id==='perfect_100') return perfectWavesCount>=100;
    if(id==='speed_100') return (wave>=100 && (Date.now()-startTime)<1800000);
    if(id==='overdrive_1000') return totalOverdriveUses>=1000;
    if(id==='bomb_1000') return totalBombsUsed>=1000;
    if(id==='drone_400') return droneCount>=180;
    if(id==='fire_100') return fireLevel>=100;
    if(id==='damage_100') return damageLevel>=100;
    if(id==='endless_master') return endlessMode && wave>=200;
    if(id==='death_touch') return deathTouchCount>=1;
    if(id==='chaos_realm') return chaosRealmCount>=1;
    if(id==='death_master') return deathTouchCount>=3;
    if(id==='chaos_master') return chaosRealmCount>=3;
    if(id==='reaper_call') return reaperCalls>=1;
    if(id==='time_warp') return timeWarps>=1;
    if(id==='gold_rush') return goldRushes>=1;
    if(id==='divine_intervention') return divineInterventions>=1;
    if(id==='bug_event') return bugEvents>=1;
    if(id==='stable_cycle') return stableCycleCount>=1;
    if(id==='tidal_wave') return tidalWaveCount>=1;
    if(id==='masquerade') return masqueradeCount>=1;
    if(id==='soul_harvest') return soulHarvestCount>=1;
    if(id==='frozen_time') return frozenTimeCount>=1;
    if(id==='crystal_rain') return crystalRainCount>=1;
    if(id==='shadow_clone') return shadowCloneCount>=1;
    if(id==='lightning_storm') return lightningStormCount>=1;
    if(id==='lucky_draw') return luckyDrawCount>=1;
    if(id==='mystery_box') return mysteryBoxCount>=1;
    if(id==='dooms_day') return doomsDayCount>=1;
    if(id==='royal_blessing') return royalBlessingCount>=1;
    if(id==='starfall') return starfallCount>=1;
    if(id==='inferno') return infernoCount>=1;
    if(id==='chain_lightning') return chainLightningCount>=1;
    if(id==='barrier') return barrierCount>=1;
    if(id==='soul_reaper') return soulReaperCount>=1;
    if(id==='gambler') return gamblerCount>=1;
    if(id==='vortex') return vortexCount>=1;
    if(id==='kings_blessing') return kingsBlessingCount>=1;
    if(id==='prism') return prismCount>=1;
    if(id==='abyss') return abyssCount>=1;
    if(id==='doppelganger') return doppelgangerCount>=1;
    if(id==='supernova') return supernovaCount>=1;
    if(id==='space_treasure') return spaceTreasureCount>=1;
    if(id==='space_treasure_master') return spaceTreasureCount>=5;
    if(id==='doppelganger_master') return doppelgangerCount>=5;
    if(id==='supernova_master') return supernovaCount>=5;
    if(id==='critical_hitter') return criticalHitsCount>=100;
    if(id==='critical_master') return criticalHitsCount>=1000;
    if(id==='combo_master_50') return maxCombo>=50;
    if(id==='combo_master_100') return maxCombo>=100;
    if(id==='settings_enthusiast') return settings.graphics !== 'high';
    if(id==='auto_save_hero') return (localStorage.getItem('autoSaveCount')||0)>=10;
    if(id==='stat_enthusiast') return (localStorage.getItem('statViewCount')||0)>=5;
    if(id==='damage_dealer') return totalDamageDealt>=1000000;
    if(id==='damage_god') return totalDamageDealt>=100000000;
    if(id==='perfect_game_plus') return (wave>=50 && health===maxHealth);
    if(id==='ultimate_collector') return ownedSkins.blue && ownedSkins.purple && ownedSkins.gold && ownedSkins.rainbow && ownedSkins.ultra && ownedSkins.legend;
    if(id==='absolute_perfection') return ACHIEVEMENTS_LIST.every(a => achievements[a.id]===true);
    if(id==='streak_3') return dailyStreak>=3;
    if(id==='streak_7') return dailyStreak>=7;
    if(id==='streak_30') return dailyStreak>=30;
    return false;
}

function checkAchievements(){
    let anyNew=false;
    for(const a of ACHIEVEMENTS_LIST){
        if(!achievements[a.id] && getAchievementProgressValue(a.id)){
            achievements[a.id]=true;
            anyNew=true;
            let rewardGems = a.gems * gemMultiplier;
            gemstones += rewardGems;
            saveGemstones();
            showAchievementPopup(a.name, a.desc, rewardGems);
            addRankXP(rewardGems);
        }
    }
    if(anyNew){
        localStorage.setItem('achievements',JSON.stringify(achievements));
        checkSkinUnlock();
        updateAchievementsUI();
        updateSkinProgressUI();
        updateHubUI();
        updateSkinsUI();
    }
}

function checkSkinUnlock(){
    const unlockedCount = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    if(unlockedCount >= 40 && !skinUnlocked){
        skinUnlocked=true;
        localStorage.setItem('skinUnlocked','true');
        if(!ownedSkins.gold){
            ownedSkins.gold = true;
            localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
        }
        showAchievementPopup('🌟 GOLDEN SKIN UNLOCKED', 'You can now equip the Golden Legend skin!');
        updateSkinProgressUI();
        updateHubUI();
        updateSkinsUI();
    }
    if(currentRank >= 50 && !ownedSkins.legend){
        ownedSkins.legend = true;
        localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
        showAchievementPopup('🏆 LEGEND SKIN UNLOCKED', 'You reached Rank 50! Legend skin is yours!');
        updateSkinsUI();
    }
}

function updateSkinProgressUI(){
    const unlockedCount = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    const percent = Math.min(100, (unlockedCount/40)*100);
    document.getElementById('skin-progress-fill').style.width=percent+'%';
    document.getElementById('skin-percent').innerHTML=unlockedCount+'/40 ACHIEVEMENTS';
    document.getElementById('skin-status-text').innerText=skinUnlocked?'UNLOCKED (GOLDEN)':'LOCKED';
}

function getAchievementProgress(achId){
    const progressMap = {
        'kills100': {current: totalKills, target: 100},
        'kills500': {current: totalKills, target: 500},
        'kills10000': {current: kills, target: 10000},
        'meteor_slayer': {current: totalMeteorsDestroyed, target: 50},
        'meteor_master': {current: totalMeteorsDestroyed, target: 200},
        'rift_walker': {current: totalRiftsTriggered, target: 5},
        'rift_god': {current: totalRiftsTriggered, target: 15},
        'event_master': {current: totalEventsTriggered, target: 10},
        'event_collector': {current: totalEventsTriggered, target: 25},
        'event_god': {current: totalEventsTriggered, target: 50},
        'event_legend': {current: totalEventsTriggered, target: 100},
        'overdrive3': {current: odActivations, target: 3},
        'overdrive_god': {current: totalOverdriveUses, target: 50},
        'overdrive_legend': {current: totalOverdriveUses, target: 200},
        'overdrive_1000': {current: totalOverdriveUses, target: 1000},
        'bomb_master': {current: totalBombsUsed, target: 100},
        'bomb_1000': {current: totalBombsUsed, target: 1000},
        'combo10': {current: maxCombo, target: 10},
        'combo_legend': {current: maxCombo, target: 50},
        'combo_god': {current: maxCombo, target: 30},
        'wave5': {current: wave, target: 5},
        'survivor': {current: wave, target: 15},
        'godlike': {current: wave, target: 50},
        'impossible': {current: wave, target: 100},
        'wave_500': {current: wave, target: 500},
        'wave_warrior': {current: wave, target: 200},
        'perfect_wave': {current: perfectWavesCount, target: 3},
        'perfect_run': {current: perfectWavesCount, target: 10},
        'immortal_god': {current: perfectWavesCount, target: 50},
        'perfect_100': {current: perfectWavesCount, target: 100},
        'score5k': {current: score, target: 5000},
        'score25k': {current: score, target: 25000},
        'millionaire': {current: score, target: 1000000},
        'billionaire': {current: score, target: 10000000},
        'score_billion': {current: score, target: 1000000000},
        'rich': {current: totalCoins, target: 10000},
        'rich_lord': {current: totalCoins, target: 200000},
        'credit_king': {current: totalCoins, target: 1000000},
        'credit_billion': {current: totalCoins, target: 100000000},
        'pyro': {current: fireLevel, target: 10},
        'fire_god': {current: fireLevel, target: 30},
        'fire_100': {current: fireLevel, target: 100},
        'warmonger': {current: damageLevel, target: 10},
        'damage_god': {current: damageLevel, target: 30},
        'damage_100': {current: damageLevel, target: 100},
        'drone_army': {current: droneCount, target: 5},
        'drone_master': {current: droneCount, target: 10},
        'drone_overlord': {current: droneCount, target: 180},
        'drone_400': {current: droneCount, target: 180},
        'shopaholic': {current: totalUpgradesBought, target: 50},
        'legendary': {current: level, target: 15},
        'true_god': {current: level, target: 30},
        'god_of_war': {current: level, target: 50},
        'ultimate_power': {current: level, target: 100},
        'rank_500': {current: level, target: 500},
        'boss_genocide': {current: bossesKilled, target: 10},
        'omega_boss': {current: bossesKilled, target: 50},
        'decimator': {current: kills, target: 200},
        'unstoppable': {current: kills, target: 1000},
        'no_mercy': {current: waveKills, target: 50},
        'synapse_activate': {current: totalSynapseActivations, target: 3},
        'time_lord': {current: synapseKills, target: 50},
        'psychedelic': {current: psychedeliaCollected, target: 10},
        'apocalypse_survivor': {current: apocalypseTriggers, target: 3},
        'void_walker': {current: voidTriggers, target: 5},
        'apocalypse_master': {current: apocalypseTriggers, target: 3},
        'void_assassin': {current: voidTriggers, target: 5},
        'cosmic_collapse': {current: cosmicCollapseCount, target: 1},
        'cosmic_master': {current: cosmicCollapseCount, target: 5},
        'primordial_rage': {current: primordialRageCount, target: 1},
        'primordial_god': {current: primordialRageCount, target: 3},
        'death_touch': {current: deathTouchCount, target: 1},
        'chaos_realm': {current: chaosRealmCount, target: 1},
        'endless_master': {current: endlessMode && wave>=200 ? 1 : 0, target: 1},
        'reaper_call': {current: reaperCalls, target: 1},
        'time_warp': {current: timeWarps, target: 1},
        'gold_rush': {current: goldRushes, target: 1},
        'divine_intervention': {current: divineInterventions, target: 1},
        'bug_event': {current: bugEvents, target: 1},
        'stable_cycle': {current: stableCycleCount, target: 1},
        'tidal_wave': {current: tidalWaveCount, target: 1},
        'masquerade': {current: masqueradeCount, target: 1},
        'soul_harvest': {current: soulHarvestCount, target: 1},
        'frozen_time': {current: frozenTimeCount, target: 1},
        'crystal_rain': {current: crystalRainCount, target: 1},
        'shadow_clone': {current: shadowCloneCount, target: 1},
        'lightning_storm': {current: lightningStormCount, target: 1},
        'lucky_draw': {current: luckyDrawCount, target: 1},
        'mystery_box': {current: mysteryBoxCount, target: 1},
        'dooms_day': {current: doomsDayCount, target: 1},
        'royal_blessing': {current: royalBlessingCount, target: 1},
        'starfall': {current: starfallCount, target: 1},
        'inferno': {current: infernoCount, target: 1},
        'chain_lightning': {current: chainLightningCount, target: 1},
        'barrier': {current: barrierCount, target: 1},
        'soul_reaper': {current: soulReaperCount, target: 1},
        'gambler': {current: gamblerCount, target: 1},
        'vortex': {current: vortexCount, target: 1},
        'kings_blessing': {current: kingsBlessingCount, target: 1},
        'prism': {current: prismCount, target: 1},
        'abyss': {current: abyssCount, target: 1},
        'doppelganger': {current: doppelgangerCount, target: 1},
        'supernova': {current: supernovaCount, target: 1},
        'doppelganger_master': {current: doppelgangerCount, target: 5},
        'supernova_master': {current: supernovaCount, target: 5},
        'space_treasure': {current: Math.min(spaceTreasureCount,1), target: 1},
        'space_treasure_master': {current: Math.min(spaceTreasureCount,5), target: 5},
        'critical_hitter': {current: criticalHitsCount, target: 100},
        'critical_master': {current: criticalHitsCount, target: 1000},
        'combo_master_50': {current: maxCombo, target: 50},
        'combo_master_100': {current: maxCombo, target: 100},
        'damage_dealer': {current: totalDamageDealt, target: 1000000},
        'damage_god': {current: totalDamageDealt, target: 100000000},
        'streak_3': {current: Math.min(dailyStreak,3), target: 3},
        'streak_7': {current: Math.min(dailyStreak,7), target: 7},
        'streak_30': {current: Math.min(dailyStreak,30), target: 30}
    };
    return progressMap[achId] || {current: achievements[achId]?1:0, target: 1};
}

function openAchievements(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('achievements-screen').style.display='flex';
    updateAchievementsUI();
}
function closeAchievements(){
    document.getElementById('achievements-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}
function updateAchievementsUI(filter='all'){
    const container=document.getElementById('ach-list');
    if(!container)return;
    container.innerHTML='';
    let filteredList = ACHIEVEMENTS_LIST;
    if(filter !== 'all'){
        filteredList = ACHIEVEMENTS_LIST.filter(a => a.difficulty === filter);
    }
    const totalUnlocked = ACHIEVEMENTS_LIST.filter(a => achievements[a.id]===true).length;
    const totalAchievements = ACHIEVEMENTS_LIST.length;
    const unlockedNumEl = document.getElementById('ach-unlocked-num');
    const totalNumEl = document.getElementById('ach-total-num');
    const progressFillEl = document.getElementById('ach-total-progress-fill');
    if(unlockedNumEl) unlockedNumEl.textContent = totalUnlocked;
    if(totalNumEl) totalNumEl.textContent = totalAchievements;
    if(progressFillEl) progressFillEl.style.width = ((totalUnlocked / totalAchievements) * 100) + '%';
    for(const a of filteredList){
        const unlocked=achievements[a.id]===true;
        const progress = getAchievementProgress(a.id);
        const percent = Math.min(100, (progress.current/progress.target)*100);
        const diffClass = a.difficulty ? ' ach-'+a.difficulty : '';
        const div=document.createElement('div');
        div.className='ach-card '+(unlocked?'':'locked')+diffClass;
        div.innerHTML=`<div class="ach-status" style="float:left;">${unlocked?'✅':'🔒'}</div>
                       <div class="ach-name">${a.name}</div>
                       <div class="ach-desc">${a.desc}</div>
                       ${!unlocked && progress.target>1 ? `<div class="ach-progress-bar"><div class="ach-progress-fill" style="width:${percent}%"></div></div>
                       <div class="ach-progress-text">${formatNumber(progress.current)}/${formatNumber(progress.target)}</div>` : ''}
                       ${unlocked ? '<div class="ach-progress-text" style="color:gold;">✓ COMPLETED +'+a.gems+'💎</div>' : '<div class="ach-progress-text" style="color:#ffaa00;">🎁 '+a.gems+'💎</div>'}`;
        container.appendChild(div);
    }
}

function filterAchievements(difficulty){
    document.querySelectorAll('.ach-filter-tab').forEach(t => t.classList.remove('active'));
    event.target.classList.add('active');
    updateAchievementsUI(difficulty);
}

function openEvents(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('events-screen').style.display='flex';
    updateEventsUI();
}
function closeEvents(){
    document.getElementById('events-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}
function updateEventsUI(){
    const container=document.getElementById('events-list-container');
    if(!container)return;
    container.innerHTML=`
        <div class="event-card"><div class="event-name">☄️ METEOR SHOWER</div><div class="event-chance">Chance: 60% every 4 waves (wave≥4)</div><div class="event-desc">Meteors fall from the sky. Destroy them for +200 points each. Meteor hit = 15 damage.</div></div>
        <div class="event-card"><div class="event-name">🌀 DIMENSION RIFT</div><div class="event-chance">Chance: 3% after wave 10</div><div class="event-desc">Double enemies, 2x points and credits for 15 seconds.</div></div>
        <div class="event-card"><div class="event-name">🧠 SYNAPSE MODE</div><div class="event-chance">Collect 8 Psychedelia (🧠) items</div><div class="event-desc">Time slows, 2.5x damage, trippy background for 10 seconds.</div></div>
        <div class="event-card"><div class="event-name">🔴 APOCALYPSE MODE</div><div class="event-chance">~0.5% every 5 waves (rare!)</div><div class="event-desc">Enemies doubled, faster, but 5x points. Lasts 15 seconds.</div></div>
        <div class="event-card"><div class="event-name">💜 VOID MODE</div><div class="event-chance">~0.3% after wave 15 (ultra rare!)</div><div class="event-desc">Become invincible, 10x damage for 15 seconds.</div></div>
        <div class="event-card"><div class="event-name">👑 THE GUARDIAN</div><div class="event-chance">~0.1% after wave 20 (legendary!)</div><div class="event-desc">Giant golden boss with 10,000 HP. Rewards: 50,000 points, 5,000 credits, and GUARANTEES golden skin!</div></div>
        <div class="event-card"><div class="event-name">🌌 COSMIC COLLAPSE</div><div class="event-chance">GUARANTEED every 40 waves!</div><div class="event-desc">Screen fills with enemies, 10x points, 5x credits for 20 seconds.</div></div>
        <div class="event-card"><div class="event-name">⚡ PRIMORDIAL RAGE</div><div class="event-chance">~0.003% (1 in 33,333) - EXTREMELY RARE!</div><div class="event-desc">Become giant red, invincible, massive bullet storms for 10 seconds.</div></div>
        <div class="event-card"><div class="event-name">🕳️ BLACK HOLE</div><div class="event-chance">Collect 6 Black Hole pieces (⭐) to activate</div><div class="event-desc">Sucks in all enemies, instantly killing them. All points and credits rewarded!</div></div>
        <div class="event-card"><div class="event-name">💀 DEATH'S TOUCH</div><div class="event-chance">~0.005% (1 in 20,000) - MYTHIC RARE!</div><div class="event-desc">All enemies on screen die instantly! 50x points for each enemy!</div></div>
        <div class="event-card"><div class="event-name">🌈 CHAOS REALM</div><div class="event-chance">~0.008% (1 in 12,500) - MYTHIC RARE!</div><div class="event-desc">Rainbow bullets, 5x fire rate, trippy rainbow background for 12 seconds!</div></div>
        <div class="event-card"><div class="event-name">💀 REAPER'S CALL</div><div class="event-chance">~0.002% (1 in 50,000) - MYTHIC RARE!</div><div class="event-desc">Reapers appear and automatically kill enemies for 10 seconds!</div></div>
        <div class="event-card"><div class="event-name">🌀 TIME WARP</div><div class="event-chance">~0.004% (1 in 25,000) - MYTHIC RARE!</div><div class="event-desc">All enemies slowed by 80% for 10 seconds!</div></div>
        <div class="event-card"><div class="event-name">💰 GOLD RUSH</div><div class="event-chance">~0.0005% (1 in 200,000) - LEGENDARY RARE!</div><div class="event-desc">10x credits and gemstones from all sources for 15 seconds!</div></div>
        <div class="event-card"><div class="event-name">⚡ DIVINE INTERVENTION</div><div class="event-chance">~0.000003% (1 in 33,000,000) - ULTRA GODLY RARE!</div><div class="event-desc">Full health & shield restore, +300% damage for 60s, screen-clear all enemies, 5s invincibility, golden light rays! Once-in-a-lifetime event!</div></div>
        <div class="event-card"><div class="event-name">🐛 BUG EVENT</div><div class="event-chance">~0.001% (1 in 100,000) - MYSTERY EVENT!</div><div class="event-desc">The game glitches out... then rewards you with massive bonuses!</div></div>
        <div class="event-card"><div class="event-name">💫 STABLE CYCLE</div><div class="event-chance">GUARANTEED every 7 waves!</div><div class="event-desc">3x points and credits for 10 seconds!</div></div>
        <div class="event-card"><div class="event-name">🌊 TIDAL WAVE</div><div class="event-chance">~0.5% after wave 12 (rare!)</div><div class="event-desc">Massive wave of enemies, but 5x points per kill!</div></div>
        <div class="event-card"><div class="event-name">🎭 MASQUERADE</div><div class="event-chance">~0.3% after wave 15 (rare!)</div><div class="event-desc">Enemies disguise as items, but give 10x points!</div></div>
        <div class="event-card"><div class="event-name">💀 SOUL HARVEST</div><div class="event-chance">~0.1% after wave 20 (legendary!)</div><div class="event-desc">Defeated enemies release souls that attack other enemies!</div></div>
        <div class="event-card"><div class="event-name">❄️ FROZEN TIME</div><div class="event-chance">~0.2% after wave 15 (rare!)</div><div class="event-desc">All enemies frozen for 8 seconds! Take no damage from frozen enemies!</div></div>
        <div class="event-card"><div class="event-name">💎 CRYSTAL RAIN</div><div class="event-chance">~0.15% after wave 18 (rare!)</div><div class="event-desc">Crystals fall from the sky! Collect them for bonus gems and credits!</div></div>
        <div class="event-card"><div class="event-name">👥 SHADOW CLONE</div><div class="event-chance">~0.08% after wave 25 (legendary!)</div><div class="event-desc">Create a clone that fights alongside you for 15 seconds!</div></div>
        <div class="event-card"><div class="event-name">⚡ LIGHTNING STORM</div><div class="event-chance">~0.1% after wave 12 (rare!)</div><div class="event-desc">Lightning strikes random enemies, dealing massive damage!</div></div>
        <div class="event-card"><div class="event-name">🍀 LUCKY DRAW</div><div class="event-chance">~0.05% after wave 15 (rare!)</div><div class="event-desc">Get a random shop item for free!</div></div>
        <div class="event-card"><div class="event-name">🔮 MYSTERY BOX</div><div class="event-chance">~0.03% after wave 18 (rare!)</div><div class="event-desc">Open a mystery box with random rewards!</div></div>
        <div class="event-card"><div class="event-name">💀 DOOM'S DAY</div><div class="event-chance">~0.05% after wave 20 (legendary!)</div><div class="event-desc">Massive wave of weakened enemies! 3x points, 15% chance for Green Aura that damages nearby enemies!</div></div>
        <div class="event-card"><div class="event-name">👑 ROYAL BLESSING</div><div class="event-chance">~0.005% after wave 25 (mythic!)</div><div class="event-desc">3x to everything for 20 seconds!</div></div>
        <div class="event-card"><div class="event-name">🌟 STARFALL</div><div class="event-chance">~0.14% after wave 10 (rare!)</div><div class="event-desc">Stars fall from the sky! Collect them for gemstones! (Reduced stars & rewards)</div></div>
        <div class="event-card"><div class="event-name">🔥 INFERNO</div><div class="event-chance">~0.08% after wave 15 (rare!)</div><div class="event-desc">Flames spread across the screen, burning enemies!</div></div>
        <div class="event-card"><div class="event-name">⚡ CHAIN LIGHTNING</div><div class="event-chance">~0.06% after wave 18 (rare!)</div><div class="event-desc">Lightning chains between enemies, dealing massive damage!</div></div>
        <div class="event-card"><div class="event-name">🛡️ BARRIER</div><div class="event-chance">~0.2% after wave 12 (rare!)</div><div class="event-desc">Create a barrier that absorbs all damage for 10 seconds!</div></div>
        <div class="event-card"><div class="event-name">💀 SOUL REAPER</div><div class="event-chance">~0.02% after wave 25 (legendary!)</div><div class="event-desc">A giant reaper appears and harvests enemy souls!</div></div>
        <div class="event-card"><div class="event-name">🎲 GAMBLER</div><div class="event-chance">~0.01% after wave 20 (legendary!)</div><div class="event-desc">Gamble your credits for a chance to win big!</div></div>
        <div class="event-card"><div class="event-name">🌀 VORTEX</div><div class="event-chance">~0.04% after wave 22 (rare!)</div><div class="event-desc">A vortex sucks in and destroys enemies!</div></div>
        <div class="event-card"><div class="event-name">👑 KING'S BLESSING</div><div class="event-chance">~0.003% after wave 30 (mythic!)</div><div class="event-desc">The king blesses you with 5x everything for 15 seconds!</div></div>
        <div class="event-card"><div class="event-name">🌈 PRISM</div><div class="event-chance">~0.05% after wave 18 (rare!)</div><div class="event-desc">Split your bullets into rainbow colors for massive coverage!</div></div>
        <div class="event-card"><div class="event-name">🕳️ ABYSS</div><div class="event-chance">~0.002% after wave 35 (mythic!)</div><div class="event-desc">An abyss opens, swallowing all enemies!</div></div>
        <div class="event-card"><div class="event-name">🎭 DOPPELGANGER</div><div class="event-chance">~0.06% after wave 22 (rare!)</div><div class="event-desc">A clone of you appears and fights alongside you for 15 seconds!</div></div>
        <div class="event-card"><div class="event-name">💥 SUPERNOVA</div><div class="event-chance">~0.001% after wave 30 (mythic!)</div><div class="event-desc">A massive explosion kills all enemies on screen and drops massive bonuses!</div></div>
        <div class="event-card"><div class="event-name">🎁 SPACE TREASURE</div><div class="event-chance">45% every 9 waves</div><div class="event-desc">Treasure ships warp in dropping valuable loot! Collect golden crates for gems, credits, and bombs!</div></div>
        <div class="event-card"><div class="event-name">☄️ ASTEROID BELT</div><div class="event-chance">~0.1% after wave 16</div><div class="event-desc">A wave of asteroids crosses the screen! Shoot them for bonus coins and gems. Larger asteroids take more hits but give better rewards!</div></div>
        <div class="event-card"><div class="event-name">📦 SUPPLY DROP</div><div class="event-chance">~0.08% after wave 14</div><div class="event-desc">A glowing cargo crate appears! Fly to it before it disappears for random coins, gems, or a temporary buff!</div></div>
        <div class="event-card"><div class="event-name" style="color:#ffaa00;">⏱️ EVENT COOLDOWN</div><div class="event-chance">3 seconds between events</div><div class="event-desc">Events cannot trigger one after another - 3 second grace period!</div></div>
        <div class="event-card"><div class="event-name" style="color:gold;">🌟 ENDLESS MODE</div><div class="event-chance">Available after wave 100</div><div class="event-desc">Continue infinitely with progressively stronger enemies!</div></div>
    `;
}

// AUDIO FUNCTIONS
const AudioCtx=window.AudioContext||window.webkitAudioContext;
let audioCtx;
function initAudio(){if(!audioCtx && settings.sound) audioCtx=new AudioCtx();}
function playBeep(freq,dur,vol,type='square',freqEnd){
    if(!settings.sound) return;
    if(!audioCtx) return;
    try{
        const o=audioCtx.createOscillator(),g=audioCtx.createGain();
        o.connect(g);g.connect(audioCtx.destination);
        o.type=type;o.frequency.setValueAtTime(freq,audioCtx.currentTime);
        if(freqEnd) o.frequency.exponentialRampToValueAtTime(freqEnd,audioCtx.currentTime+dur);
        let scaledVol = vol * (settings.sfxVolume / 100);
        g.gain.setValueAtTime(scaledVol,audioCtx.currentTime);
        g.gain.exponentialRampToValueAtTime(0.001,audioCtx.currentTime+dur);
        o.start();o.stop(audioCtx.currentTime+dur);
    }catch(e){}
}
function sfxShoot()   {playBeep(880,0.04,0.04,'sawtooth');}
function sfxHit()     {playBeep(220,0.07,0.1,'square');}
function sfxExplode() {playBeep(110,0.18,0.12,'sawtooth');}
function sfxCoin()    {playBeep(1200,0.05,0.06,'sine');}
function sfxLevelUp() {playBeep(660,0.1,0.08,'sine');setTimeout(()=>playBeep(880,0.1,0.08,'sine'),100);}
function sfxOverdrive(){playBeep(440,0.3,0.12,'sawtooth');playBeep(880,0.3,0.08,'sine');}
function sfxBomb()    {playBeep(60,0.5,0.15,'sawtooth',30);}
function sfxWave()    {playBeep(300,0.08,0.06,'sine');setTimeout(()=>playBeep(400,0.08,0.06,'sine'),90);}
function sfxPowerup() {playBeep(800,0.06,0.08,'sine');setTimeout(()=>playBeep(1000,0.06,0.08,'sine'),70);}
function sfxLaser()   {playBeep(200,0.04,0.05,'sawtooth',800);}
function sfxSynapse() {playBeep(500,0.2,0.1,'sine');playBeep(700,0.2,0.1,'sine');playBeep(900,0.3,0.1,'sine');}
function sfxEvent()   {playBeep(400,0.15,0.12,'sine');playBeep(600,0.2,0.12,'sine');}
function sfxApocalypse(){playBeep(100,0.3,0.15,'sawtooth');playBeep(150,0.3,0.15,'sawtooth');playBeep(200,0.4,0.15,'sawtooth');}
function sfxVoid(){playBeep(600,0.2,0.1,'sine');playBeep(900,0.2,0.1,'sine');playBeep(1200,0.3,0.1,'sine');}
function sfxCosmic(){playBeep(50,0.5,0.2,'sine');playBeep(100,0.5,0.2,'sine');playBeep(200,0.5,0.2,'sine');}
function sfxDeathTouch(){playBeep(40,0.8,0.2,'sawtooth');playBeep(20,0.8,0.2,'sawtooth');playBeep(10,1,0.2,'sawtooth');}
function sfxChaosRealm(){playBeep(800,0.1,0.1,'sine');playBeep(1000,0.1,0.1,'sine');playBeep(1200,0.1,0.1,'sine');playBeep(1400,0.2,0.1,'sine');}
function sfxReaper(){playBeep(300,0.2,0.15,'sawtooth');playBeep(200,0.2,0.15,'sawtooth');playBeep(100,0.3,0.15,'sawtooth');}
function sfxTimeWarp(){playBeep(600,0.2,0.1,'sine');playBeep(400,0.2,0.1,'sine');playBeep(200,0.3,0.1,'sine');}
function sfxGoldRush(){playBeep(1000,0.1,0.15,'sine');playBeep(1200,0.1,0.15,'sine');playBeep(1400,0.2,0.15,'sine');}
function sfxDivine(){playBeep(500,0.3,0.2,'sine');playBeep(800,0.3,0.2,'sine');playBeep(1200,0.4,0.2,'sine');}
function sfxBug(){playBeep(300,0.1,0.2,'sawtooth');playBeep(200,0.1,0.2,'sawtooth');playBeep(100,0.1,0.2,'sawtooth');playBeep(50,0.2,0.2,'sawtooth');}
function sfxStable(){playBeep(500,0.2,0.15,'sine');playBeep(600,0.2,0.15,'sine');}
function sfxTidal(){playBeep(100,0.3,0.2,'sawtooth');playBeep(200,0.3,0.2,'sawtooth');}
function sfxMasquerade(){playBeep(800,0.1,0.1,'sine');playBeep(900,0.1,0.1,'sine');playBeep(1000,0.2,0.1,'sine');}
function sfxSoul(){playBeep(400,0.2,0.15,'sine');playBeep(300,0.2,0.15,'sine');}
function sfxFrozen(){playBeep(300,0.2,0.15,'sine');playBeep(200,0.2,0.15,'sine');}
function sfxCrystal(){playBeep(1000,0.1,0.15,'sine');playBeep(1100,0.1,0.15,'sine');playBeep(1200,0.2,0.15,'sine');}
function sfxShadow(){playBeep(400,0.2,0.15,'sine');playBeep(300,0.2,0.15,'sine');playBeep(200,0.2,0.15,'sine');}
function sfxLightning(){playBeep(800,0.1,0.2,'sawtooth');playBeep(1000,0.1,0.2,'sawtooth');}
function sfxLucky(){playBeep(600,0.2,0.15,'sine');playBeep(800,0.2,0.15,'sine');}
function sfxMystery(){playBeep(500,0.2,0.15,'sine');playBeep(700,0.2,0.15,'sine');playBeep(900,0.2,0.15,'sine');}
function sfxDoom(){playBeep(100,0.5,0.2,'sawtooth');playBeep(80,0.5,0.2,'sawtooth');}
function sfxRoyal(){playBeep(600,0.2,0.15,'sine');playBeep(800,0.2,0.15,'sine');playBeep(1000,0.2,0.15,'sine');}
function sfxStarfall(){playBeep(800,0.1,0.15,'sine');playBeep(900,0.1,0.15,'sine');playBeep(1000,0.2,0.15,'sine');}
function sfxInferno(){playBeep(200,0.3,0.2,'sawtooth');playBeep(300,0.3,0.2,'sawtooth');}
function sfxChain(){playBeep(600,0.1,0.15,'sine');playBeep(700,0.1,0.15,'sine');playBeep(800,0.1,0.15,'sine');}
function sfxBarrier(){playBeep(400,0.2,0.15,'sine');playBeep(500,0.2,0.15,'sine');}
function sfxSoulReaper(){playBeep(100,0.5,0.2,'sawtooth');playBeep(80,0.5,0.2,'sawtooth');playBeep(60,0.5,0.2,'sawtooth');}
function sfxGambler(){playBeep(600,0.1,0.15,'sine');playBeep(800,0.1,0.15,'sine');playBeep(1000,0.2,0.15,'sine');}
function sfxVortex(){playBeep(200,0.3,0.2,'sawtooth');playBeep(150,0.3,0.2,'sawtooth');}
function sfxKings(){playBeep(500,0.2,0.2,'sine');playBeep(700,0.2,0.2,'sine');playBeep(900,0.3,0.2,'sine');}
function sfxPrism(){playBeep(800,0.1,0.15,'sine');playBeep(1000,0.1,0.15,'sine');playBeep(1200,0.2,0.15,'sine');}
function sfxAbyss(){playBeep(50,0.5,0.2,'sawtooth');playBeep(30,0.5,0.2,'sawtooth');}
function sfxDoppelganger(){playBeep(600,0.2,0.15,'sine');playBeep(700,0.2,0.15,'sine');playBeep(800,0.2,0.15,'sine');}
function sfxSupernova(){playBeep(80,0.5,0.25,'sawtooth');playBeep(60,0.5,0.25,'sawtooth');playBeep(40,0.5,0.25,'sawtooth');}
function sfxSpaceTreasure(){ playBeep(880,0.15,0.15,'sine'); setTimeout(()=>playBeep(1100,0.15,0.15,'sine'),100); setTimeout(()=>playBeep(1320,0.2,0.15,'sine'),200); }

// CROSSHAIR
const crossCanvas=document.getElementById('crosshair'),cCtx=crossCanvas.getContext('2d');
crossCanvas.style.pointerEvents='none';
function resizeCross(){crossCanvas.width=window.innerWidth;crossCanvas.height=window.innerHeight;}
resizeCross();
function drawCrosshair(x,y){
    cCtx.clearRect(0,0,crossCanvas.width,crossCanvas.height);
    if(gameState!=='PLAYING')return;
    const size=12,gap=4,col=isOD?'#ff00ff':synapseActive?'#ff66ff':voidActive?'#aa66ff':primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:apocalypseActive?'#ff0000':riftActive?'#aa66ff':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffdd00':doppelgangerActive?'#ff88ff':'#00d2ff';
    cCtx.strokeStyle=col;cCtx.lineWidth=2;cCtx.shadowBlur=5;
    cCtx.beginPath();
    cCtx.moveTo(x-size,y);cCtx.lineTo(x-gap,y);
    cCtx.moveTo(x+gap,y); cCtx.lineTo(x+size,y);
    cCtx.moveTo(x,y-size);cCtx.lineTo(x,y-gap);
    cCtx.moveTo(x,y+gap); cCtx.lineTo(x,y+size);
    cCtx.stroke();
    cCtx.beginPath();cCtx.arc(x,y,3,0,Math.PI*2);cCtx.fillStyle=col;cCtx.fill();
}

function drawNebula(){
    if(settings.graphics === 'low') return;
    nCtx.clearRect(0,0,width,height);
    const g1=nCtx.createRadialGradient(width*0.3,height*0.4,0,width*0.3,height*0.4,width*0.5);
    g1.addColorStop(0,'rgba(0,100,180,0.25)');g1.addColorStop(1,'rgba(0,0,0,0)');
    nCtx.fillStyle=g1;nCtx.fillRect(0,0,width,height);
    const g2=nCtx.createRadialGradient(width*0.75,height*0.6,0,width*0.75,height*0.6,width*0.4);
    g2.addColorStop(0,'rgba(100,0,120,0.2)');g2.addColorStop(1,'rgba(0,0,0,0)');
    nCtx.fillStyle=g2;nCtx.fillRect(0,0,width,height);
}

function runLoader(id,barId,cb){
    const scr=document.getElementById(id),bar=document.getElementById(barId);
    scr.style.display='flex';let p=0;
    const iv=setInterval(()=>{
        p+=Math.random()*9;
        if(p>=100){p=100;clearInterval(iv);setTimeout(()=>{scr.style.display='none';cb();},400);}
        bar.style.width=p+'%';
    },70);
}

// POWER-UPS
const POWERUP_TYPES=[
    {id:'rapidfire',name:'RAPID FIRE',color:'#00d2ff',icon:'⚡',dur:7000},
    {id:'doubleDmg',name:'POWER AMP', color:'#ff4444',icon:'💪',dur:6000},
    {id:'magnet',   name:'MAGNET',    color:'gold',    icon:'🧲',dur:9000},
    {id:'regen',    name:'REGEN',     color:'#00ffaa', icon:'💚',dur:7000},
];
function spawnPowerUp(x,y){
    const t=POWERUP_TYPES[Math.floor(Math.random()*POWERUP_TYPES.length)];
    items.push({x,y,vy:2,isPowerUp:true,puType:t});
}
function spawnPsychedelia(x,y){
    items.push({x,y,vy:2,isPowerUp:false,isPsychedelia:true});
}
function spawnBlackHolePiece(x,y){
    items.push({x,y,vy:2,isPowerUp:false,isBlackHolePiece:true});
}
function applyPowerUp(t){
    activePowerUps[t.id]={name:t.name,color:t.color,icon:t.icon,endTime:Date.now()+t.dur};
    sfxPowerup();
    if(player && player.x && player.y) floats.push({txt:t.icon+' '+t.name+'!',x:player.x-50,y:player.y-35,l:1.5,c:t.color,size:18});
    updatePowerUpBar();
}
function updatePowerUpBar(){
    const bar=document.getElementById('powerup-bar'),now=Date.now();
    const active=Object.values(activePowerUps).filter(p=>p.endTime>now);
    if(active.length===0){bar.style.display='none';bar.innerHTML='';return;}
    bar.style.display='block';
    bar.innerHTML=active.map(p=>{
        const rem=Math.ceil((p.endTime-now)/1000);
        return `<span class="pu-item" style="border-color:${p.color};color:${p.color}">${p.icon} ${p.name} ${rem}s</span>`;
    }).join('');
}
function tickPowerUps(){
    const now=Date.now();
    for(const k of Object.keys(activePowerUps)) if(activePowerUps[k].endTime<=now) delete activePowerUps[k];
    if(activePowerUps.regen&&Math.random()<0.008) health=Math.min(maxHealth,health+0.5);
    if(getSkillBonus('regen')>0 && Math.random()<0.003) health=Math.min(maxHealth,health+getSkillBonus('regen'));
    updatePowerUpBar();
}

// EVENT COOLDOWN
function canTriggerEvent(){
    if(activeEvent) return false;
    if(eventCooldown > Date.now()) return false;
    return true;
}
function startEventCooldown(){ eventCooldown = Date.now() + 3000; }

function queueEvent(triggerFn){
    if(activeEvent){
        eventQueue.push(triggerFn);
    } else {
        triggerFn();
    }
}

function processEventQueue(){
    if(!activeEvent && eventQueue.length > 0){
        const nextEvent = eventQueue.shift();
        nextEvent();
    }
}

// EVENT TRIGGER FUNCTIONS (abbreviated - same as before)
function triggerDoppelganger(){
    if(!canTriggerEvent()) return;
    activeEvent='doppelganger'; eventTimer=Date.now()+15000; doppelgangerCount++; totalEventsTriggered++; sfxDoppelganger();
    doppelgangerActive=true; doppelgangerTimer=Date.now()+15000;
    doppelgangerClone = new DoppelgangerClone();
    const banner=document.getElementById('event-banner');
    banner.innerHTML='🎭 DOPPELGANGER 🎭'; banner.style.display='block'; banner.style.color='#ff88ff';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000);
    if(player && player.x && player.y) floats.push({txt:'🎭 DOPPELGANGER!',x:width/2-70,y:height/2-40,l:1.5,c:'#ff88ff',size:24});
    showNotification('🎭 Doppelganger event started! A clone fights for you!', 'event');
    startEventCooldown(); checkAchievements();
}

function triggerSupernova(){
    if(!canTriggerEvent()) return;
    activeEvent='supernova'; eventTimer=Date.now()+3000; supernovaCount++; totalEventsTriggered++; sfxSupernova();
    supernovaActive=true; supernovaTimer=Date.now()+3000;
    const banner=document.getElementById('event-banner');
    banner.innerHTML='💥 SUPERNOVA 💥'; banner.style.display='block'; banner.style.color='#ffaa44';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000);
    let enemiesKilled = 0;
    for(let i=0;i<enemies.length;i++){
        let e=enemies[i];
        let pointBonus = e.isBoss?50000:5000*combo;
        if(goldRushActive) pointBonus*=10;
        if(stableCycleActive) pointBonus*=3;
        if(royalBlessingActive) pointBonus*=3;
        if(kingsBlessingActive) pointBonus*=5;
        pointBonus *= skinCreditMultiplier;
        score+=pointBonus; kills++; waveKills++;
        if(e.isBoss) bossesKilled++;
        enemiesKilled++;
        for(let k=0;k<30;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*20,(Math.random()-0.5)*20,'#ffaa44',1));
    }
    enemies = [];
    boss = null;
    for(let i=0;i<10+Math.floor(Math.random()*10);i++){
        items.push({x:Math.random()*width, y:height/2, vy:2, isPowerUp:false});
    }
    gemstones += 500 * gemMultiplier;
    saveGemstones();
    totalCoins += 10000;
    localStorage.setItem('totalCoins', totalCoins);
    if(player && player.x && player.y) floats.push({txt:`💥 SUPERNOVA! ${enemiesKilled} ENEMIES DIED! +500💎 +10000c`,x:width/2-150,y:height/2-40,l:1.8,c:'#ffaa44',size:26});
    showNotification(`💥 SUPERNOVA killed ${enemiesKilled} enemies! +500 GEMSTONES, +10000 CREDITS!`, 'event');
    startEventCooldown(); checkAchievements();
}

function triggerMeteorShower(){ if(!canTriggerEvent()) return; activeEvent='meteor'; eventTimer=Date.now()+10000; totalEventsTriggered++; sfxEvent(); const banner=document.getElementById('event-banner'); banner.innerHTML='☄️ METEOR SHOWER ☄️'; banner.style.display='block'; banner.style.color='#ffaa44'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<15+Math.floor(Math.random()*15);i++) meteors.push(new Meteor(Math.random()*width, -50)); startEventCooldown(); checkAchievements(); showNotification('☄️ Meteor Shower event started!', 'event'); }
function triggerDimensionRift(){ if(!canTriggerEvent()) return; activeEvent='rift'; eventTimer=Date.now()+15000; riftActive=true; riftMultiplier=2; totalEventsTriggered++; totalRiftsTriggered++; sfxEvent(); const banner=document.getElementById('event-banner'); banner.innerHTML='🌀 DIMENSION RIFT 🌀'; banner.style.display='block'; banner.style.color='#aa66ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<8;i++) enemies.push(new Enemy()); startEventCooldown(); checkAchievements(); showNotification('🌀 Dimension Rift event started! 2x points!', 'event'); }
function triggerApocalypseMode(){ if(!canTriggerEvent()) return; apocalypseActive=true; activeEvent='apocalypse'; eventTimer=Date.now()+15000; apocalypseTriggers++; totalEventsTriggered++; sfxApocalypse(); const banner=document.getElementById('event-banner'); banner.innerHTML='🔴 APOCALYPSE MODE 🔴'; banner.style.display='block'; banner.style.color='#ff0000'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<12;i++) enemies.push(new Enemy()); if(player && player.x && player.y) floats.push({txt:'🔴 APOCALYPSE!',x:width/2-60,y:height/2-40,l:1.5,c:'#ff0000',size:24}); startEventCooldown(); checkAchievements(); showNotification('🔴 Apocalypse Mode event started! 5x points!', 'event'); }
function triggerVoidMode(){ if(!canTriggerEvent()) return; voidActive=true; voidTimer=Date.now()+15000; activeEvent='void'; eventTimer=Date.now()+15000; voidTriggers++; totalEventsTriggered++; sfxVoid(); const banner=document.getElementById('event-banner'); banner.innerHTML='💜 VOID MODE 💜'; banner.style.display='block'; banner.style.color='#aa66ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💜 VOID MODE!',x:width/2-50,y:height/2-40,l:1.5,c:'#aa66ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('💜 Void Mode event started! Invincible!', 'event'); }
function triggerGuardian(){ if(!canTriggerEvent()) return; guardian=new Guardian(); activeEvent='guardian'; sfxApocalypse(); const banner=document.getElementById('event-banner'); banner.innerHTML='👑 THE GUARDIAN AWAKENS 👑'; banner.style.display='block'; banner.style.color='gold'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},4000); if(player && player.x && player.y) floats.push({txt:'👑 GUARDIAN!',x:width/2-50,y:height/2-50,l:2,c:'gold',size:32}); startEventCooldown(); showNotification('👑 The Guardian has awakened!', 'warning'); }
function triggerCosmicCollapse(){ if(!canTriggerEvent()) return; activeEvent='cosmic'; eventTimer=Date.now()+20000; cosmicCollapseActive=true; cosmicCollapseCount++; totalEventsTriggered++; sfxCosmic(); const banner=document.getElementById('event-banner'); banner.innerHTML='🌌 COSMIC COLLAPSE 🌌'; banner.style.display='block'; banner.style.color='#88aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<30;i++) enemies.push(new Enemy()); if(player && player.x && player.y) floats.push({txt:'🌌 COSMIC COLLAPSE!',x:width/2-70,y:height/2-40,l:1.5,c:'#88aaff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🌌 Cosmic Collapse event started! 10x points!', 'event'); }
function triggerPrimordialRage(){ if(!canTriggerEvent()) return; activeEvent='primordial'; eventTimer=Date.now()+10000; primordialRageActive=true; primordialRageCount++; totalEventsTriggered++; sfxApocalypse(); const banner=document.getElementById('event-banner'); banner.innerHTML='⚡ PRIMORDIAL RAGE ⚡'; banner.style.display='block'; banner.style.color='#ff0000'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player) player.r = 60; if(player && player.x && player.y) floats.push({txt:'⚡ PRIMORDIAL RAGE!',x:width/2-70,y:height/2-40,l:1.5,c:'#ff0000',size:24}); startEventCooldown(); checkAchievements(); showNotification('⚡ Primordial Rage event started! Massive damage!', 'event'); }
function triggerBlackHole(){ if(!canTriggerEvent()) return; activeEvent='blackhole'; eventTimer=Date.now()+8000; blackHoleActive=true; blackHoleCenter={x:width/2, y:height/2}; totalEventsTriggered++; sfxCosmic(); const banner=document.getElementById('event-banner'); banner.innerHTML='🕳️ BLACK HOLE 🕳️'; banner.style.display='block'; banner.style.color='#4400aa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); let activations=parseInt(localStorage.getItem('blackHoleActivations')||0)+1; localStorage.setItem('blackHoleActivations',activations); if(player && player.x && player.y) floats.push({txt:'🕳️ BLACK HOLE!',x:width/2-50,y:height/2-40,l:1.5,c:'#4400aa',size:24}); startEventCooldown(); checkAchievements(); showNotification('🕳️ Black Hole event started!', 'event'); }
function triggerDeathTouch(){ if(!canTriggerEvent()) return; activeEvent='deathtouch'; eventTimer=Date.now()+3000; deathTouchCount++; totalEventsTriggered++; sfxDeathTouch(); const banner=document.getElementById('event-banner'); banner.innerHTML='💀 DEATH\'S TOUCH 💀'; banner.style.display='block'; banner.style.color='#440044'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); let enemiesKilled=0; for(let i=0;i<enemies.length;i++){ let e=enemies[i]; let pointBonus = e.isBoss?90000:3500*combo; if(goldRushActive) pointBonus*=10; if(stableCycleActive) pointBonus*=3; if(royalBlessingActive) pointBonus*=3; if(kingsBlessingActive) pointBonus*=5; score+=pointBonus; kills++; waveKills++; if(e.isBoss) bossesKilled++; enemiesKilled++; for(let k=0;k<20;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*15,(Math.random()-0.5)*15,'#440044',0.9)); } enemies=[]; boss=null; if(player && player.x && player.y) floats.push({txt:`💀 ${enemiesKilled} ENEMIES DIED!`,x:width/2-100,y:height/2-40,l:1.8,c:'#440044',size:28}); startEventCooldown(); checkAchievements(); showNotification(`💀 Death's Touch killed ${enemiesKilled} enemies!`, 'event'); }
function triggerChaosRealm(){ if(!canTriggerEvent()) return; activeEvent='chaos'; eventTimer=Date.now()+12000; chaosRealmActive=true; chaosRealmCount++; totalEventsTriggered++; sfxChaosRealm(); const banner=document.getElementById('event-banner'); banner.innerHTML='🌈 CHAOS REALM 🌈'; banner.style.display='block'; banner.style.color='#ff66ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🌈 CHAOS REALM!',x:width/2-70,y:height/2-40,l:1.5,c:'#ff66ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🌈 Chaos Realm event started! Rainbow bullets!', 'event'); }
function triggerReapersCall(){ if(!canTriggerEvent()) return; activeEvent='reaper'; eventTimer=Date.now()+10000; reaperCalls++; totalEventsTriggered++; sfxReaper(); const banner=document.getElementById('event-banner'); banner.innerHTML='💀 REAPER\'S CALL 💀'; banner.style.display='block'; banner.style.color='#880044'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💀 REAPER\'S CALL!',x:width/2-70,y:height/2-40,l:1.5,c:'#880044',size:24}); startEventCooldown(); checkAchievements(); showNotification('💀 Reaper\'s Call event started!', 'event'); }
function triggerTimeWarp(){ if(!canTriggerEvent()) return; activeEvent='timewarp'; eventTimer=Date.now()+10000; timeWarps++; totalEventsTriggered++; sfxTimeWarp(); timeWarpActive=true; timeWarpTimer=Date.now()+10000; const banner=document.getElementById('event-banner'); banner.innerHTML='🌀 TIME WARP 🌀'; banner.style.display='block'; banner.style.color='#44aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🌀 TIME WARP!',x:width/2-60,y:height/2-40,l:1.5,c:'#44aaff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🌀 Time Warp event started! Enemies slowed!', 'event'); }
function triggerGoldRush(){ if(!canTriggerEvent()) return; activeEvent='goldrush'; eventTimer=Date.now()+15000; goldRushes++; totalEventsTriggered++; sfxGoldRush(); goldRushActive=true; goldRushTimer=Date.now()+15000; const banner=document.getElementById('event-banner'); banner.innerHTML='💰 GOLD RUSH 💰'; banner.style.display='block'; banner.style.color='#ffaa00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💰 GOLD RUSH!',x:width/2-60,y:height/2-40,l:1.5,c:'#ffaa00',size:24}); startEventCooldown(); checkAchievements(); showNotification('💰 Gold Rush event started! 10x credits!', 'event'); }
function triggerDivineIntervention(){ if(!canTriggerEvent()) return; activeEvent='divine'; eventTimer=Date.now()+60000; divineInterventions++; totalEventsTriggered++; sfxDivine(); divineActive=true; divineTimer=Date.now()+60000; divineFlashTimer=Date.now()+3000; health = maxHealth; if(hasShieldUpgrade) shieldHp = maxShieldHp; if(player) player.invincibleTimer = 300; // 5 seconds invincibility // Screen clear all enemies for(let i=0;i<enemies.length;i++){ let e=enemies[i]; let pointBonus = e.isBoss?50000:5000*combo; score+=pointBonus; kills++; waveKills++; if(e.isBoss) bossesKilled++; for(let k=0;k<25;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*15,(Math.random()-0.5)*15,'#ffdd00',1)); } enemies=[]; boss=null; const banner=document.getElementById('event-banner'); banner.innerHTML='⚡ DIVINE INTERVENTION ⚡'; banner.style.display='block'; banner.style.color='#ffdd00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},5000); if(player && player.x && player.y) floats.push({txt:'⚡ DIVINE INTERVENTION! +300% DMG!',x:width/2-100,y:height/2-40,l:2.5,c:'#ffdd00',size:32}); showNotification('⚡ DIVINE INTERVENTION! Full heal! +300% damage! Screen cleared!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerBugEvent(){ if(!canTriggerEvent()) return; activeEvent='bug'; eventTimer=Date.now()+5000; bugEvents++; totalEventsTriggered++; sfxBug(); bugEventActive=true; bugEventGlitch=true; const banner=document.getElementById('event-banner'); banner.innerHTML='🐛 BUG EVENT 🐛'; banner.style.display='block'; banner.style.color='#00ff00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🐛 BUG EVENT!',x:width/2-60,y:height/2-40,l:1.5,c:'#00ff00',size:24}); setTimeout(() => { if(gameState === 'PLAYING'){ let reward = 777 * gemMultiplier; gemstones += reward; saveGemstones(); totalCoins += 7777; localStorage.setItem('totalCoins', totalCoins); showAchievementPopup('🐛 BUG EVENT RESOLVED', `You received ${reward} GEMSTONES and 7,777 CREDITS!`, reward); bugEventActive = false; bugEventGlitch = false; } }, 3000); startEventCooldown(); checkAchievements(); showNotification('🐛 Bug Event triggered! Glitch incoming...', 'event'); }
function triggerStableCycle(){ if(!canTriggerEvent()) return; activeEvent='stable'; eventTimer=Date.now()+10000; stableCycleCount++; totalEventsTriggered++; sfxStable(); stableCycleActive=true; stableCycleTimer=Date.now()+10000; const banner=document.getElementById('event-banner'); banner.innerHTML='💫 STABLE CYCLE 💫'; banner.style.display='block'; banner.style.color='#88aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💫 STABLE CYCLE!',x:width/2-60,y:height/2-40,l:1.5,c:'#88aaff',size:24}); startEventCooldown(); checkAchievements(); showNotification('💫 Stable Cycle event started! 3x points!', 'event'); }
function triggerTidalWave(){ if(!canTriggerEvent()) return; activeEvent='tidal'; eventTimer=Date.now()+12000; tidalWaveCount++; totalEventsTriggered++; sfxTidal(); tidalWaveActive=true; tidalWaveTimer=Date.now()+12000; const banner=document.getElementById('event-banner'); banner.innerHTML='🌊 TIDAL WAVE 🌊'; banner.style.display='block'; banner.style.color='#44aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<20;i++) enemies.push(new Enemy()); if(player && player.x && player.y) floats.push({txt:'🌊 TIDAL WAVE!',x:width/2-60,y:height/2-40,l:1.5,c:'#44aaff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🌊 Tidal Wave event started! Many enemies!', 'event'); }
function triggerMasquerade(){ if(!canTriggerEvent()) return; activeEvent='masquerade'; eventTimer=Date.now()+15000; masqueradeCount++; totalEventsTriggered++; sfxMasquerade(); masqueradeActive=true; masqueradeTimer=Date.now()+15000; const banner=document.getElementById('event-banner'); banner.innerHTML='🎭 MASQUERADE 🎭'; banner.style.display='block'; banner.style.color='#aa66ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🎭 MASQUERADE!',x:width/2-60,y:height/2-40,l:1.5,c:'#aa66ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🎭 Masquerade event started! Disguised enemies!', 'event'); }
function triggerSoulHarvest(){ if(!canTriggerEvent()) return; activeEvent='soul'; eventTimer=Date.now()+15000; soulHarvestCount++; totalEventsTriggered++; sfxSoul(); soulHarvestActive=true; soulHarvestTimer=Date.now()+15000; soulHarvestSouls = []; const banner=document.getElementById('event-banner'); banner.innerHTML='💀 SOUL HARVEST 💀'; banner.style.display='block'; banner.style.color='#8800aa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💀 SOUL HARVEST!',x:width/2-70,y:height/2-40,l:1.5,c:'#8800aa',size:24}); startEventCooldown(); checkAchievements(); showNotification('💀 Soul Harvest event started! Souls attack!', 'event'); }
function triggerFrozenTime(){ if(!canTriggerEvent()) return; activeEvent='frozen'; eventTimer=Date.now()+8000; frozenTimeCount++; totalEventsTriggered++; sfxFrozen(); frozenTimeActive=true; frozenTimeTimer=Date.now()+8000; const banner=document.getElementById('event-banner'); banner.innerHTML='❄️ FROZEN TIME ❄️'; banner.style.display='block'; banner.style.color='#88ccff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'❄️ FROZEN TIME!',x:width/2-60,y:height/2-40,l:1.5,c:'#88ccff',size:24}); startEventCooldown(); checkAchievements(); showNotification('❄️ Frozen Time event started! Enemies frozen!', 'event'); }
function triggerCrystalRain(){ if(!canTriggerEvent()) return; activeEvent='crystal'; eventTimer=Date.now()+10000; crystalRainCount++; totalEventsTriggered++; sfxCrystal(); crystalRainActive=true; crystalRainTimer=Date.now()+10000; const banner=document.getElementById('event-banner'); banner.innerHTML='💎 CRYSTAL RAIN 💎'; banner.style.display='block'; banner.style.color='#88ffaa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💎 CRYSTAL RAIN!',x:width/2-60,y:height/2-40,l:1.5,c:'#88ffaa',size:24}); startEventCooldown(); checkAchievements(); showNotification('💎 Crystal Rain event started! Collect crystals!', 'event'); }
function triggerShadowClone(){ if(!canTriggerEvent()) return; activeEvent='shadow'; eventTimer=Date.now()+15000; shadowCloneCount++; totalEventsTriggered++; sfxShadow(); shadowCloneActive=true; shadowCloneTimer=Date.now()+15000; shadowClone = new ShadowClone(); const banner=document.getElementById('event-banner'); banner.innerHTML='👥 SHADOW CLONE 👥'; banner.style.display='block'; banner.style.color='#aa88ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'👥 SHADOW CLONE!',x:width/2-60,y:height/2-40,l:1.5,c:'#aa88ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('👥 Shadow Clone event started! Clone fights for you!', 'event'); }
function triggerLightningStorm(){ if(!canTriggerEvent()) return; activeEvent='lightning'; eventTimer=Date.now()+8000; lightningStormCount++; totalEventsTriggered++; sfxLightning(); lightningStormActive=true; lightningStormTimer=Date.now()+8000; const banner=document.getElementById('event-banner'); banner.innerHTML='⚡ LIGHTNING STORM ⚡'; banner.style.display='block'; banner.style.color='#ffff00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'⚡ LIGHTNING STORM!',x:width/2-60,y:height/2-40,l:1.5,c:'#ffff00',size:24}); startEventCooldown(); checkAchievements(); showNotification('⚡ Lightning Storm event started! Lightning strikes enemies!', 'event'); }
function triggerLuckyDraw(){ if(!canTriggerEvent()) return; activeEvent='lucky'; eventTimer=Date.now()+3000; luckyDrawCount++; totalEventsTriggered++; sfxLucky(); const banner=document.getElementById('event-banner'); banner.innerHTML='🍀 LUCKY DRAW 🍀'; banner.style.display='block'; banner.style.color='#88ff88'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); const shopItems = ['fire', 'dmg', 'shield', 'drone', 'heal', 'spread', 'laser', 'bomb']; const randomItem = shopItems[Math.floor(Math.random() * shopItems.length)]; buyUpgrade(randomItem, 1); if(player && player.x && player.y) floats.push({txt:'🍀 LUCKY DRAW! Free upgrade!',x:width/2-70,y:height/2-40,l:1.5,c:'#88ff88',size:24}); startEventCooldown(); checkAchievements(); showNotification('🍀 Lucky Draw event! Free upgrade!', 'event'); }
function triggerMysteryBox(){ if(!canTriggerEvent()) return; activeEvent='mystery'; eventTimer=Date.now()+3000; mysteryBoxCount++; totalEventsTriggered++; sfxMystery(); const banner=document.getElementById('event-banner'); banner.innerHTML='🔮 MYSTERY BOX 🔮'; banner.style.display='block'; banner.style.color='#ff88ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); const rewards = [ () => { let reward = 500 * gemMultiplier; gemstones += reward; saveGemstones(); return `${reward} GEMSTONES!`; }, () => { let reward = 10000; totalCoins += reward; localStorage.setItem('totalCoins', totalCoins); return `${reward} CREDITS!`; }, () => { bombCount += 5; localStorage.setItem('bombCount', bombCount); return "5 BOMBS!"; }, () => { damageLevel += 2; localStorage.setItem('dmgLevel', damageLevel); return "PERMANENT DAMAGE +2!"; }, () => { fireLevel += 3; localStorage.setItem('fireLevel', fireLevel); return "PERMANENT FIRE RATE +3!"; } ]; const reward = rewards[Math.floor(Math.random() * rewards.length)](); if(player && player.x && player.y) floats.push({txt:`🔮 MYSTERY BOX! ${reward}`,x:width/2-80,y:height/2-40,l:1.5,c:'#ff88ff',size:24}); startEventCooldown(); checkAchievements(); showNotification('🔮 Mystery Box event! Random reward!', 'event'); }
function triggerDoomsDay(){ if(!canTriggerEvent()) return; activeEvent='doom'; eventTimer=Date.now()+15000; doomsDayCount++; totalEventsTriggered++; sfxDoom(); doomsDayActive=true; greenAuraActive=Math.random()<0.15; for(let i=0;i<30;i++) enemies.push(new Enemy()); const banner=document.getElementById('event-banner'); banner.innerHTML='💀 DOOM\'S DAY 💀'; banner.style.display='block'; banner.style.color='#440000'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'💀 DOOM\'S DAY!',x:width/2-60,y:height/2-40,l:1.5,c:'#440000',size:24}); if(greenAuraActive) floats.push({txt:'💚 GREEN AURA!',x:width/2-50,y:height/2+10,l:1.5,c:'#00ff00',size:20}); startEventCooldown(); checkAchievements(); showNotification('💀 Doom\'s Day event started! More enemies = more loot!' + (greenAuraActive ? ' 💚 Green Aura active!' : ''), 'event'); }
function triggerRoyalBlessing(){ if(!canTriggerEvent()) return; activeEvent='royal'; eventTimer=Date.now()+20000; royalBlessingCount++; totalEventsTriggered++; sfxRoyal(); royalBlessingActive=true; royalBlessingTimer=Date.now()+20000; const banner=document.getElementById('event-banner'); banner.innerHTML='👑 ROYAL BLESSING 👑'; banner.style.display='block'; banner.style.color='#ffdd00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'👑 ROYAL BLESSING!',x:width/2-60,y:height/2-40,l:1.5,c:'#ffdd00',size:24}); startEventCooldown(); checkAchievements(); showNotification('👑 Royal Blessing event started! 3x everything!', 'event'); }
function triggerStarfall(){ if(!canTriggerEvent()) return; activeEvent='starfall'; eventTimer=Date.now()+9000; starfallCount++; totalEventsTriggered++; sfxStarfall(); starfallActive=true; starfallTimer=Date.now()+9000; starfallStars = []; const banner=document.getElementById('event-banner'); banner.innerHTML='🌟 STARFALL 🌟'; banner.style.display='block'; banner.style.color='#ffffaa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); for(let i=0;i<10;i++){ starfallStars.push(new StarfallStar(Math.random()*width, -20, 8+Math.random()*8, 20+Math.floor(Math.random()*40))); } if(player && player.x && player.y) floats.push({txt:'🌟 STARFALL!',x:width/2-60,y:height/2-40,l:1.5,c:'#ffffaa',size:24}); showNotification('🌟 STARFALL event started! Catch falling stars!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerInferno(){ if(!canTriggerEvent()) return; activeEvent='inferno'; eventTimer=Date.now()+8000; infernoCount++; totalEventsTriggered++; sfxInferno(); infernoActive=true; infernoTimer=Date.now()+8000; const banner=document.getElementById('event-banner'); banner.innerHTML='🔥 INFERNO 🔥'; banner.style.display='block'; banner.style.color='#ff4400'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🔥 INFERNO!',x:width/2-60,y:height/2-40,l:1.5,c:'#ff4400',size:24}); showNotification('🔥 INFERNO event started! Flames will burn enemies!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerChainLightning(){ if(!canTriggerEvent()) return; activeEvent='chain'; eventTimer=Date.now()+10000; chainLightningCount++; totalEventsTriggered++; sfxChain(); chainLightningActive=true; chainLightningTimer=Date.now()+10000; const banner=document.getElementById('event-banner'); banner.innerHTML='⚡ CHAIN LIGHTNING ⚡'; banner.style.display='block'; banner.style.color='#ffff00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'⚡ CHAIN LIGHTNING!',x:width/2-70,y:height/2-40,l:1.5,c:'#ffff00',size:24}); showNotification('⚡ CHAIN LIGHTNING event started! Lightning will chain between enemies!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerBarrier(){ if(!canTriggerEvent()) return; activeEvent='barrier'; eventTimer=Date.now()+10000; barrierCount++; totalEventsTriggered++; sfxBarrier(); barrierActive=true; barrierTimer=Date.now()+10000; barrierHp = 500; const banner=document.getElementById('event-banner'); banner.innerHTML='🛡️ BARRIER 🛡️'; banner.style.display='block'; banner.style.color='#88aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🛡️ BARRIER!',x:width/2-60,y:height/2-40,l:1.5,c:'#88aaff',size:24}); showNotification('🛡️ BARRIER event started! Damage absorption active!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerSoulReaper(){ if(!canTriggerEvent()) return; activeEvent='soulreaper'; eventTimer=Date.now()+12000; soulReaperCount++; totalEventsTriggered++; sfxSoulReaper(); soulReaperActive=true; soulReaperTimer=Date.now()+12000; soulReaperSoul = {x:width/2, y:-100, hp:500, maxHp:500}; const banner=document.getElementById('event-banner'); banner.innerHTML='💀 SOUL REAPER 💀'; banner.style.display='block'; banner.style.color='#8800aa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},4000); if(player && player.x && player.y) floats.push({txt:'💀 SOUL REAPER!',x:width/2-70,y:height/2-40,l:1.5,c:'#8800aa',size:28}); showNotification('💀 SOUL REAPER event started! A powerful reaper appears!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerGambler(){ if(!canTriggerEvent()) return; activeEvent='gambler'; eventTimer=Date.now()+5000; gamblerCount++; totalEventsTriggered++; sfxGambler(); gamblerActive=true; const banner=document.getElementById('event-banner'); banner.innerHTML='🎲 GAMBLER 🎲'; banner.style.display='block'; banner.style.color='#ffaa00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); const outcome = Math.random(); if(outcome < 0.3){ totalCoins = Math.floor(totalCoins * 0.5); localStorage.setItem('totalCoins', totalCoins); if(player && player.x && player.y) floats.push({txt:'🎲 GAMBLER: YOU LOST 50% CREDITS!',x:width/2-100,y:height/2-40,l:1.5,c:'#ff0000',size:22}); showNotification('🎲 GAMBLER: You lost 50% of your credits!', 'warning'); } else if(outcome < 0.7){ let win = 5000; totalCoins += win; localStorage.setItem('totalCoins', totalCoins); if(player && player.x && player.y) floats.push({txt:`🎲 GAMBLER: +${win} CREDITS!`,x:width/2-100,y:height/2-40,l:1.5,c:'#00ffaa',size:22}); showNotification(`🎲 GAMBLER: You won ${win} CREDITS!`, 'success'); } else { let win = 20000; totalCoins += win; localStorage.setItem('totalCoins', totalCoins); if(player && player.x && player.y) floats.push({txt:`🎲 GAMBLER: +${win} CREDITS! JACKPOT!`,x:width/2-110,y:height/2-40,l:1.5,c:'#ffaa00',size:24}); showNotification(`🎲 GAMBLER: JACKPOT! +${win} CREDITS!`, 'success'); } startEventCooldown(); checkAchievements(); }
function triggerVortex(){ if(!canTriggerEvent()) return; activeEvent='vortex'; eventTimer=Date.now()+10000; vortexCount++; totalEventsTriggered++; sfxVortex(); vortexActive=true; vortexTimer=Date.now()+10000; vortexCenter={x:width/2, y:height/2}; const banner=document.getElementById('event-banner'); banner.innerHTML='🌀 VORTEX 🌀'; banner.style.display='block'; banner.style.color='#44aaff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🌀 VORTEX!',x:width/2-60,y:height/2-40,l:1.5,c:'#44aaff',size:24}); showNotification('🌀 VORTEX event started! Enemies will be pulled into the vortex!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerKingsBlessing(){ if(!canTriggerEvent()) return; activeEvent='kings'; eventTimer=Date.now()+15000; kingsBlessingCount++; totalEventsTriggered++; sfxKings(); kingsBlessingActive=true; kingsBlessingTimer=Date.now()+15000; const banner=document.getElementById('event-banner'); banner.innerHTML='👑 KING\'S BLESSING 👑'; banner.style.display='block'; banner.style.color='#ffdd00'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},4000); if(player && player.x && player.y) floats.push({txt:'👑 KING\'S BLESSING! 5x EVERYTHING!',x:width/2-100,y:height/2-40,l:1.5,c:'#ffdd00',size:26}); showNotification('👑 KING\'S BLESSING event started! 5x points and credits!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerPrism(){ if(!canTriggerEvent()) return; activeEvent='prism'; eventTimer=Date.now()+12000; prismCount++; totalEventsTriggered++; sfxPrism(); prismActive=true; prismTimer=Date.now()+12000; const banner=document.getElementById('event-banner'); banner.innerHTML='🌈 PRISM 🌈'; banner.style.display='block'; banner.style.color='#ff88ff'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🌈 PRISM! Bullets split into colors!',x:width/2-80,y:height/2-40,l:1.5,c:'#ff88ff',size:24}); showNotification('🌈 PRISM event started! Bullets split into rainbow colors!', 'event'); startEventCooldown(); checkAchievements(); }
function triggerAbyss(){ if(!canTriggerEvent()) return; activeEvent='abyss'; eventTimer=Date.now()+8000; abyssCount++; totalEventsTriggered++; sfxAbyss(); abyssActive=true; abyssTimer=Date.now()+8000; abyssCenter={x:width/2, y:height/2}; const banner=document.getElementById('event-banner'); banner.innerHTML='🕳️ ABYSS 🕳️'; banner.style.display='block'; banner.style.color='#4400aa'; setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000); if(player && player.x && player.y) floats.push({txt:'🕳️ ABYSS!',x:width/2-60,y:height/2-40,l:1.5,c:'#4400aa',size:24}); showNotification('🕳️ ABYSS event started! Enemies will be swallowed by the abyss!', 'event'); startEventCooldown(); checkAchievements(); }

function triggerSpaceTreasure(){
    if(!canTriggerEvent()) return;
    activeEvent='space_treasure'; eventTimer=Date.now()+12000; spaceTreasureCount++; localStorage.setItem('spaceTreasureCount',spaceTreasureCount); totalEventsTriggered++; sfxSpaceTreasure();
    spaceTreasureActive=true; spaceTreasureTimer=Date.now()+12000;
    spaceTreasureCrates=[];
    const banner=document.getElementById('event-banner');
    banner.innerHTML='🎁 SPACE TREASURE 🎁'; banner.style.display='block'; banner.style.color='#ffd700';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000);
    if(player && player.x && player.y) floats.push({txt:'🎁 SPACE TREASURE!',x:width/2-70,y:height/2-40,l:1.5,c:'#ffd700',size:24});
    startEventCooldown(); checkAchievements();
    showNotification('🎁 Space Treasure event started! Collect falling crates!', 'event');
}

// ASTEROID BELT CLASS
class Asteroid {
    constructor(x, y, size) {
        this.x = x; this.y = y; this.size = size;
        this.hp = size === 'large' ? 6 : size === 'medium' ? 3 : 1;
        this.maxHp = this.hp;
        this.r = size === 'large' ? 28 : size === 'medium' ? 18 : 10;
        this.vx = 2 + Math.random() * 3;
        this.vy = (Math.random() - 0.5) * 1.5;
        this.coins = size === 'large' ? 500 : size === 'medium' ? 200 : 50;
        this.gems = size === 'large' ? 30 : size === 'medium' ? 10 : 3;
    }
    update() { this.x += this.vx; this.y += this.vy; }
    draw() {
        ctx.fillStyle = `hsl(${20 + (this.hp/this.maxHp)*20}, 60%, ${30 + (this.hp/this.maxHp)*20}%)`;
        ctx.shadowBlur = 6;
        ctx.beginPath(); ctx.arc(this.x, this.y, this.r, 0, Math.PI * 2); ctx.fill();
        ctx.fillStyle = '#443322'; ctx.beginPath(); ctx.arc(this.x - this.r*0.2, this.y - this.r*0.1, this.r*0.3, 0, Math.PI*2); ctx.fill();
        // HP bar
        if(this.hp < this.maxHp){
            ctx.fillStyle='#333'; ctx.fillRect(this.x-this.r, this.y-this.r-6, this.r*2, 3);
            ctx.fillStyle='#ff8844'; ctx.fillRect(this.x-this.r, this.y-this.r-6, (this.hp/this.maxHp)*this.r*2, 3);
        }
    }
}

function triggerAsteroidBelt(){
    if(!canTriggerEvent()) return;
    activeEvent='asteroidbelt'; eventTimer=Date.now()+10000; totalEventsTriggered++; sfxEvent();
    asteroidBeltActive=true; asteroidBeltTimer=Date.now()+10000; asteroids=[];
    const banner=document.getElementById('event-banner');
    banner.innerHTML='☄️ ASTEROID BELT ☄️'; banner.style.display='block'; banner.style.color='#aa6644';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000);
    if(player && player.x && player.y) floats.push({txt:'☄️ ASTEROID BELT!',x:width/2-70,y:height/2-40,l:1.5,c:'#aa6644',size:24});
    showNotification('☄️ Asteroid Belt! Shoot asteroids for bonus rewards!', 'event');
    startEventCooldown(); checkAchievements();
}

function triggerSupplyDrop(){
    if(!canTriggerEvent()) return;
    activeEvent='supplydrop'; eventTimer=Date.now()+8000; totalEventsTriggered++; sfxEvent();
    supplyDropActive=true; supplyDropTimer=Date.now()+8000;
    const rx = 80 + Math.random() * (width - 160);
    const ry = 80 + Math.random() * (height * 0.5);
    const rType = ['coins','gems','buff'][Math.floor(Math.random()*3)];
    supplyDropCrate = {x:rx, y:ry, type:rType, claimed:false};
    const banner=document.getElementById('event-banner');
    banner.innerHTML='📦 SUPPLY DROP 📦'; banner.style.display='block'; banner.style.color='#44ffaa';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},3000);
    if(player && player.x && player.y) floats.push({txt:'📦 SUPPLY DROP! Fly to it!',x:width/2-70,y:height/2-40,l:1.5,c:'#44ffaa',size:24});
    showNotification('📦 Supply Drop! Fly to the crate before it disappears!', 'event');
    startEventCooldown(); checkAchievements();
}

function checkEventTrigger(){
    if(!canTriggerEvent()) return;
    if(!endlessMode){
        if(wave>0 && wave%7===0 && !activeEvent && stableCycleCount < Math.floor(wave/7)){
            triggerStableCycle(); return;
        }
    } else {
        if(wave>0 && wave%7===0 && !activeEvent){ triggerStableCycle(); return; }
    }
    if(!endlessMode){
        if(wave>0 && wave%40===0 && !activeEvent && cosmicCollapseCount < Math.floor(wave/40)){
            triggerCosmicCollapse(); return;
        }
    } else {
        if(wave>0 && wave%40===0 && !activeEvent){ triggerCosmicCollapse(); return; }
    }
    if(wave>=22 && Math.random()<0.0006){ queueEvent(triggerDoppelganger); return; }
    if(wave>=30 && Math.random()<0.00001){ queueEvent(triggerSupernova); return; }
    if(wave>=10 && Math.random()<0.00173){ queueEvent(triggerStarfall); return; }
    if(wave>=15 && Math.random()<0.0008){ queueEvent(triggerInferno); return; }
    if(wave>=18 && Math.random()<0.0006){ queueEvent(triggerChainLightning); return; }
    if(wave>=12 && Math.random()<0.002){ queueEvent(triggerBarrier); return; }
    if(wave>=25 && Math.random()<0.0002){ queueEvent(triggerSoulReaper); return; }
    if(wave>=20 && Math.random()<0.0001){ queueEvent(triggerGambler); return; }
    if(wave>=22 && Math.random()<0.0004){ queueEvent(triggerVortex); return; }
    if(wave>=30 && Math.random()<0.00003){ queueEvent(triggerKingsBlessing); return; }
    if(wave>=18 && Math.random()<0.0005){ queueEvent(triggerPrism); return; }
    if(wave>=35 && Math.random()<0.00002){ queueEvent(triggerAbyss); return; }
    if(wave>=12 && Math.random()<0.001){ queueEvent(triggerLightningStorm); return; }
    if(wave>=15 && Math.random()<0.0005){ queueEvent(triggerLuckyDraw); return; }
    if(wave>=18 && Math.random()<0.0003){ queueEvent(triggerMysteryBox); return; }
    if(wave>=20 && Math.random()<0.0025){ queueEvent(triggerDoomsDay); return; }
    if(wave>=25 && Math.random()<0.00005){ queueEvent(triggerRoyalBlessing); return; }
    if(wave>=15 && Math.random()<0.002){ queueEvent(triggerFrozenTime); return; }
    if(wave>=18 && Math.random()<0.0015){ queueEvent(triggerCrystalRain); return; }
    if(wave>=25 && Math.random()<0.0008){ queueEvent(triggerShadowClone); return; }
    if(wave>=30 && Math.random()<0.000000009){ queueEvent(triggerDivineIntervention); return; }
    if(wave>=25 && Math.random()<0.000005){ queueEvent(triggerGoldRush); return; }
    if(wave>=20 && Math.random()<0.00001){ queueEvent(triggerBugEvent); return; }
    if(wave>=20 && Math.random()<0.00004){ queueEvent(triggerTimeWarp); return; }
    if(wave>=20 && Math.random()<0.00002){ queueEvent(triggerReapersCall); return; }
    if(wave>=20 && Math.random()<0.001){ queueEvent(triggerSoulHarvest); return; }
    if(wave>=15 && Math.random()<0.003){ queueEvent(triggerMasquerade); return; }
    if(wave>=12 && Math.random()<0.005){ queueEvent(triggerTidalWave); return; }
    if(wave>=25 && Math.random()<0.00005){ queueEvent(triggerDeathTouch); return; }
    if(wave>=25 && Math.random()<0.00008){ queueEvent(triggerChaosRealm); return; }
    if(wave>=15 && Math.random()<0.00003){ queueEvent(triggerPrimordialRage); return; }
    if(wave>0 && wave%9===0 && Math.random()<0.45){ queueEvent(triggerSpaceTreasure); return; }
    if(wave>=4 && wave%4===0 && Math.random()<0.6){ queueEvent(triggerMeteorShower); return; }
    if(wave>=10 && Math.random()<0.03){ queueEvent(triggerDimensionRift); return; }
    if(wave>=5 && wave%5===0 && Math.random()<0.005){ queueEvent(triggerApocalypseMode); return; }
    if(wave>=15 && Math.random()<0.003){ queueEvent(triggerVoidMode); return; }
    if(wave>=16 && Math.random()<0.001){ queueEvent(triggerAsteroidBelt); return; }
    if(wave>=14 && Math.random()<0.0008){ queueEvent(triggerSupplyDrop); return; }
    if(!endlessMode && wave>=20 && !guardian && Math.random()<0.001){ queueEvent(triggerGuardian); return; }
    if(endlessMode && wave>=20 && !guardian && Math.random()<0.0005){ queueEvent(triggerGuardian); return; }
}

function activateSynapse(){
    if(synapseActive) return;
    synapseActive=true; synapseTimer=Date.now()+10000; totalSynapseActivations++; sfxSynapse();
    const banner=document.getElementById('event-banner');
    banner.innerHTML='🌀 SYNAPSE MODE 🌀'; banner.style.display='block'; banner.style.color='#ff66ff';
    setTimeout(()=>{if(document.getElementById('event-banner')) document.getElementById('event-banner').style.display='none';},2500);
    if(player && player.x && player.y) floats.push({txt:'🌀 SYNAPSE MODE!',x:width/2-60,y:height/2-40,l:1.5,c:'#ff66ff',size:24});
    checkAchievements();
    showNotification('🌀 Synapse Mode activated! Time slows!', 'event');
}

function startWave(n){
    wave=n;waveKills=0;waveNoDamage=true;waveTriggered=false;
    if(endlessMode) waveKillGoal = 5 + wave * 2;
    else waveKillGoal = 5 + wave * 3;
    const banner=document.getElementById('wave-banner');
    banner.innerHTML=`WAVE ${n}`; banner.style.display='block'; waveBannerTimer=80; sfxWave();
    document.getElementById('wave-val').innerText=wave;
    if(player && player.x && player.y) floats.push({txt:'WAVE '+n,x:width/2-35,y:height/2-40,l:1.5,c:'#00d2ff',size:22});
    checkAchievements();
    if(!endlessMode && wave >= 100 && !ascendTriggered){
        ascendTriggered=true; gameState='ASCEND';
        document.getElementById('ascend-wave').innerText=wave;
        document.getElementById('ascend-screen').style.display='flex';
        document.getElementById('pause-btn').style.display='none';
    }
}

function continueEndless(){
    endlessMode=true; ascendTriggered=false;
    document.getElementById('ascend-screen').style.display='none';
    gameState='PLAYING'; document.getElementById('pause-btn').style.display='block';
    showAchievementPopup('🌟 ENDLESS ASCENSION', 'You have entered Endless Mode!');
    showNotification('🌟 Endless Mode unlocked! Infinite progression!', 'success');
}

// SHADOW CLONE CLASS
class ShadowClone {
    constructor(){
        this.x = player ? player.x + 50 : 0;
        this.y = player ? player.y : 0;
        this.r = 25;
        this.lastShot = 0;
    }
    update(){
        if(player){
            this.x = player.x + 50;
            this.y = player.y;
        }
        if(Date.now() - this.lastShot > 200){
            this.lastShot = Date.now();
            let power = damageLevel * 0.8;
            bullets.push(new Bullet(this.x, this.y-20, power, '#aa88ff'));
        }
    }
    draw(){
        ctx.save(); ctx.translate(this.x, this.y);
        ctx.fillStyle = '#aa88ff'; ctx.shadowBlur = 15;
        ctx.beginPath();
        ctx.moveTo(0,-25);ctx.lineTo(20,15);ctx.lineTo(8,15);ctx.lineTo(8,25);
        ctx.lineTo(-8,25);ctx.lineTo(-8,15);ctx.lineTo(-20,15);ctx.closePath();ctx.fill();
        ctx.fillStyle = 'rgba(170,136,255,0.5)';
        ctx.beginPath();ctx.ellipse(0,-5,4,10,0,0,Math.PI*2);ctx.fill();
        ctx.restore();
    }
}

// ============================================
// SKILL TREE SYSTEM
// ============================================
const SKILL_TREE = {
    offense: {
        name: '⚔️ OFFENSE', color: '#ff4444',
        nodes: [
            {id:'off1',name:'Damage +5%',icon:'⚔️',desc:'+5% bullet damage',bonus:{type:'damage',value:0.05}},
            {id:'off2',name:'Damage +10%',icon:'⚔️',desc:'+10% bullet damage',bonus:{type:'damage',value:0.10},requires:'off1'},
            {id:'off3',name:'Fire Rate +8%',icon:'🔥',desc:'+8% fire rate',bonus:{type:'fireRate',value:0.08}},
            {id:'off4',name:'Fire Rate +15%',icon:'🔥',desc:'+15% fire rate',bonus:{type:'fireRate',value:0.15},requires:'off3'},
            {id:'off5',name:'Crit Chance +5%',icon:'💥',desc:'+5% critical hit chance',bonus:{type:'critChance',value:0.05},requires:'off2'},
            {id:'off6',name:'Crit Damage +50%',icon:'💎',desc:'+50% critical hit damage',bonus:{type:'critDamage',value:0.50},requires:'off5'}
        ]
    },
    defense: {
        name: '🛡️ DEFENSE', color: '#4488ff',
        nodes: [
            {id:'def1',name:'Shield +20',icon:'🛡️',desc:'+20 max shield',bonus:{type:'shield',value:20}},
            {id:'def2',name:'Health +25',icon:'❤️',desc:'+25 max health',bonus:{type:'maxHealth',value:25},requires:'def1'},
            {id:'def3',name:'Dmg Reduce +8%',icon:'🔰',desc:'-8% damage taken',bonus:{type:'dmgReduction',value:0.08}},
            {id:'def4',name:'Health +50',icon:'❤️',desc:'+50 max health',bonus:{type:'maxHealth',value:50},requires:'def2'},
            {id:'def5',name:'Dmg Reduce +15%',icon:'🔰',desc:'-15% damage taken',bonus:{type:'dmgReduction',value:0.15},requires:'def3'},
            {id:'def6',name:'Regen +1hp/5s',icon:'💚',desc:'Regenerate 1 HP every 5 seconds',bonus:{type:'regen',value:1},requires:'def4'}
        ]
    },
    utility: {
        name: '⚡ UTILITY', color: '#ffaa00',
        nodes: [
            {id:'ut1',name:'Move Speed +10%',icon:'🏃',desc:'+10% movement speed',bonus:{type:'moveSpeed',value:0.10}},
            {id:'ut2',name:'Magnet Range +30',icon:'🧲',desc:'+30 coin magnet range',bonus:{type:'magnetRange',value:30}},
            {id:'ut3',name:'Drone Efficiency +10%',icon:'🤖',desc:'+10% drone damage',bonus:{type:'droneDmg',value:0.10}},
            {id:'ut4',name:'Move Speed +20%',icon:'🏃',desc:'+20% movement speed',bonus:{type:'moveSpeed',value:0.20},requires:'ut1'},
            {id:'ut5',name:'Magnet Range +60',icon:'🧲',desc:'+60 coin magnet range',bonus:{type:'magnetRange',value:60},requires:'ut2'},
            {id:'ut6',name:'Drone Efficiency +25%',icon:'🤖',desc:'+25% drone damage',bonus:{type:'droneDmg',value:0.25},requires:'ut3'}
        ]
    }
};

let skillPoints = parseInt(localStorage.getItem('skillPoints')) || 0;
let skillTreeState = {};
try { skillTreeState = JSON.parse(localStorage.getItem('skillTreeState')) || {}; } catch(e) { skillTreeState = {}; }

function getSkillBonus(type) {
    let total = 0;
    for (const path of Object.values(SKILL_TREE)) {
        for (const node of path.nodes) {
            if (skillTreeState[node.id] && node.bonus.type === type) {
                total += node.bonus.value;
            }
        }
    }
    return total;
}

critChance = 0.15 + getSkillBonus('critChance');
critDamageMultiplier = 2 + getSkillBonus('critDamage');
damageReduction *= (1 - getSkillBonus('dmgReduction'));

function awardSkillPoints() {
    let earned = Math.floor(level / 5) + Math.floor(kills / 500);
    let alreadyEarned = parseInt(localStorage.getItem('skillPointsEarned')) || 0;
    let newPts = earned - alreadyEarned;
    if (newPts > 0) {
        skillPoints += newPts;
        localStorage.setItem('skillPoints', skillPoints);
        localStorage.setItem('skillPointsEarned', earned);
        const spEl = document.getElementById('skill-points-display');
        if (spEl) spEl.innerText = skillPoints;
    }
}

function canUnlockNode(node) {
    if (skillTreeState[node.id]) return false;
    if (skillPoints < 1) return false;
    if (node.requires && !skillTreeState[node.requires]) return false;
    return true;
}

function isNodeAvailable(node) {
    if (skillTreeState[node.id]) return false;
    if (node.requires && !skillTreeState[node.requires]) return false;
    return true;
}

function unlockSkillNode(nodeId) {
    for (const path of Object.values(SKILL_TREE)) {
        for (const node of path.nodes) {
            if (node.id === nodeId && canUnlockNode(node)) {
                skillTreeState[nodeId] = true;
                skillPoints--;
                localStorage.setItem('skillTreeState', JSON.stringify(skillTreeState));
                localStorage.setItem('skillPoints', skillPoints);
                renderSkillTree();
                showNotification(`🌳 Skill unlocked: ${node.name}!`, 'success');
                return;
            }
        }
    }
}

function renderSkillTree() {
    const spEl = document.getElementById('skill-points-display');
    if (spEl) spEl.innerText = skillPoints;
    for (const [pathKey, path] of Object.entries(SKILL_TREE)) {
        const container = document.getElementById('skill-path-' + pathKey);
        if (!container) continue;
        container.innerHTML = `<div class="skill-path-title" style="color:${path.color}">${path.name}</div>`;
        path.nodes.forEach((node, i) => {
            const isUnlocked = !!skillTreeState[node.id];
            const isAvailable = isNodeAvailable(node);
            const stateClass = isUnlocked ? 'unlocked' : isAvailable ? 'available' : 'locked';
            if (i > 0) {
                const conn = document.createElement('div');
                conn.className = 'skill-connector' + (isUnlocked || isAvailable ? ' active' : '');
                container.appendChild(conn);
            }
            const el = document.createElement('div');
            el.className = 'skill-node ' + stateClass;
            el.innerHTML = `${node.icon}<div class="skill-tooltip">${node.name}<br><span style="color:#aaa">${node.desc}</span></div>`;
            if (isAvailable && skillPoints > 0) {
                el.onclick = () => unlockSkillNode(node.id);
            }
            container.appendChild(el);
        });
    }
}

function openSkillTree() {
    document.getElementById('start-screen').style.display = 'none';
    document.getElementById('skill-tree-screen').style.display = 'flex';
    renderSkillTree();
}
function closeSkillTree() {
    document.getElementById('skill-tree-screen').style.display = 'none';
    document.getElementById('start-screen').style.display = 'flex';
}

// CRYSTAL CLASS
class Crystal {
    constructor(x,y){
        this.x=x;this.y=y;this.r=8;this.vy=2;
    }
    update(){this.y+=this.vy;}
    draw(){
        ctx.fillStyle='#88ffaa';ctx.shadowBlur=8;
        ctx.beginPath();ctx.rect(this.x-4,this.y-4,8,8);ctx.fill();
        ctx.fillStyle='#fff';ctx.font='bold 12px Segoe UI';ctx.fillText('💎',this.x-6,this.y+5);
    }
}

// STARFALL STAR CLASS
class StarfallStar {
    constructor(x,y,size,value){
        this.x=x;this.y=y;this.size=size;this.value=value;this.vy=2;
    }
    update(){this.y+=this.vy;}
    draw(){
        ctx.fillStyle='#ffffaa';ctx.shadowBlur=8;
        ctx.beginPath();ctx.rect(this.x-this.size/2,this.y-this.size/2,this.size,this.size);ctx.fill();
        ctx.fillStyle='#fff';ctx.font='bold 12px Segoe UI';ctx.fillText('⭐',this.x-6,this.y+5);
    }
}

// SOUL CLASS
class Soul{
    constructor(x,y){
        this.x=x;this.y=y;this.r=8;this.speed=3;this.angle=Math.random()*Math.PI*2;
        this.vx=Math.cos(this.angle)*this.speed;this.vy=Math.sin(this.angle)*this.speed;
        this.life=1;
    }
    update(){
        this.x+=this.vx;this.y+=this.vy;
        this.life-=0.01;
    }
    draw(){
        ctx.fillStyle=`rgba(150,0,150,${this.life})`;ctx.shadowBlur=8;
        ctx.beginPath();ctx.arc(this.x,this.y,this.r,0,Math.PI*2);ctx.fill();
        ctx.fillStyle='#fff';ctx.font='bold 12px Segoe UI';ctx.fillText('💀',this.x-6,this.y+5);
    }
}

function useBomb(){
    if(bombCount<=0||gameState!=='PLAYING')return;
    bombCount--;sfxBomb();totalBombsUsed++;
    localStorage.setItem('bombCount',bombCount);
    for(let i=0;i<enemies.length;i++){
        let e=enemies[i];
        if(e.isBoss){
            e.hp-=100;
            if(e.hp<=0){ enemies.splice(i,1); boss=null; i--; }
        } else {
            let pointBonus = 35*combo;
            if(cosmicCollapseActive) pointBonus*=10;
            if(goldRushActive) pointBonus*=10;
            if(stableCycleActive) pointBonus*=3;
            if(royalBlessingActive) pointBonus*=3;
            if(kingsBlessingActive) pointBonus*=5;
            pointBonus *= skinCreditMultiplier;
            score+=pointBonus;kills++;
            for(let k=0;k<8;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*10,(Math.random()-0.5)*10,e.color,0.8));
            enemies.splice(i,1); i--;
        }
    }
    if(guardian){
        guardian.hp-=500;
        if(guardian.hp<=0){
            guardian=null; activeEvent=null; 
            let pointBonus = 50000;
            if(goldRushActive) pointBonus*=10;
            if(stableCycleActive) pointBonus*=3;
            if(royalBlessingActive) pointBonus*=3;
            if(kingsBlessingActive) pointBonus*=5;
            pointBonus *= skinCreditMultiplier;
            score+=pointBonus; totalCoins+=5000;
            if(!skinUnlocked){ skinUnlocked=true; localStorage.setItem('skinUnlocked','true'); showAchievementPopup('👑 GUARDIAN DEFEATED', 'Golden skin unlocked!'); }
            let count=parseInt(localStorage.getItem('guardianDefeatedCount')||0)+1; localStorage.setItem('guardianDefeatedCount',count);
            guardianDefeated=true; checkAchievements();
        }
    }
    for(let i=0;i<meteors.length;i++){
        let pointBonus = 200;
        if(goldRushActive) pointBonus*=10;
        if(bugEventActive) pointBonus*=5;
        if(stableCycleActive) pointBonus*=3;
        if(royalBlessingActive) pointBonus*=3;
        if(kingsBlessingActive) pointBonus*=5;
        pointBonus *= skinCreditMultiplier;
        score+=pointBonus; totalMeteorsDestroyed++;
        for(let k=0;k<5;k++) particles.push(new Particle(meteors[i].x,meteors[i].y,(Math.random()-0.5)*8,(Math.random()-0.5)*8,'#ff8844',0.8));
        meteors.splice(i,1); i--;
    }
    eBullets.length=0; shake=25;
    if(player && player.x && player.y) floats.push({txt:'💣 BOMB!',x:width/2-40,y:height/2,l:1.5,c:'#ff8800',size:26});
    if(player && player.x && player.y) for(let i=0;i<35;i++){const a=i*(Math.PI*2/35);particles.push(new Particle(player.x,player.y,Math.cos(a)*12,Math.sin(a)*12,'#ff8800',0.9));}
    checkAchievements();
    updateDailyMissionProgress('bomb', 1);
}

// SHOP FUNCTIONS
function buyUpgrade(type, amount=1){
    initAudio();
    const prices={fire:150,dmg:200,shield:250,drone:500,heal:100,spread:350,laser:600,bomb:300,resourceCollection:350,weaponEnhancement:450};
    if(type==='shield'&&hasShieldUpgrade) return;
    if(type==='spread'&&hasSpreadShot) return;
    if(type==='laser'&&hasLaser) return;
    if(type==='resourceCollection'&&resourceCollectionLevel>=20) return;
    if(type==='weaponEnhancement'&&weaponEnhancementLevel>=20) return;
    let totalCost=0;
    if(type==='fire') totalCost=prices.fire*amount;
    else if(type==='dmg') totalCost=prices.dmg*amount;
    else if(type==='drone') totalCost=prices.drone*amount;
    else if(type==='bomb') totalCost=prices.bomb*amount;
    else if(type==='resourceCollection') totalCost=prices.resourceCollection*amount;
    else if(type==='weaponEnhancement') totalCost=prices.weaponEnhancement*amount;
    else totalCost=prices[type];
    if(totalCoins<totalCost) return;
    totalCoins-=totalCost;
    totalUpgradesBought+=amount;
    if(type==='fire'){fireLevel+=amount;localStorage.setItem('fireLevel',fireLevel);}
    if(type==='dmg'){damageLevel+=amount;localStorage.setItem('dmgLevel',damageLevel);}
    if(type==='shield'){hasShieldUpgrade=true;localStorage.setItem('hasShieldUpgrade',true);}
    if(type==='drone'){droneCount=Math.min(180, droneCount+amount);localStorage.setItem('droneCount',droneCount);}
    if(type==='heal'){health=Math.min(maxHealth,health+25);}
    if(type==='spread'){hasSpreadShot=true;localStorage.setItem('hasSpreadShot',true);}
    if(type==='laser'){hasLaser=true;localStorage.setItem('hasLaser',true);}
    if(type==='bomb'){bombCount+=amount;localStorage.setItem('bombCount',bombCount);}
    if(type==='resourceCollection'){resourceCollectionLevel+=amount;localStorage.setItem('resourceCollectionLevel',resourceCollectionLevel);}
    if(type==='weaponEnhancement'){weaponEnhancementLevel+=amount;localStorage.setItem('weaponEnhancementLevel',weaponEnhancementLevel);}
    localStorage.setItem('totalCoins',totalCoins);
    localStorage.setItem('totalUpgradesBought',totalUpgradesBought);
    sfxCoin();
    updateShopUI();
    checkAchievements();
    updateDailyMissionProgress('upgrade', amount);
}
function setQuantity(type, qty){ selectedQuantities[type]=Math.max(1, Math.min(99, qty)); updateShopUI(); }
function maxOutUpgrade(type,pricePer){
    let canBuy=Math.floor(totalCoins/pricePer);
    if(type==='drone') canBuy=Math.min(canBuy, 180-droneCount);
    if(canBuy>0) buyUpgrade(type,canBuy);
}
function updateShopUI(){
    const container=document.getElementById('shop-grid-container');
    const specialSection=document.getElementById('shop-special-section');
    if(!container) return;

    const upgradeItems=[
        {id:'fire', name:'🔥 FIRE RATE', level:fireLevel, price:150, type:'fire', infinite:true, category:'weapons'},
        {id:'dmg', name:'💥 DAMAGE', level:damageLevel, price:200, type:'dmg', infinite:true, category:'weapons'},
        {id:'shield', name:'🛡️ SHIELD', owned:hasShieldUpgrade, price:250, type:'shield', category:'shields'},
        {id:'drone', name:'🤖 DRONE', level:droneCount, price:500, type:'drone', infinite:true, maxLimit:180, category:'weapons'},
        {id:'heal', name:'❤️ REPAIR', price:100, type:'heal', category:'shields'},
        {id:'spread', name:'🎯 SPREAD', owned:hasSpreadShot, price:350, type:'spread', category:'weapons'},
        {id:'laser', name:'⚡ LASER', owned:hasLaser, price:600, type:'laser', category:'weapons'},
        {id:'bomb', name:'💣 BOMB', level:bombCount, price:300, type:'bomb', infinite:true, category:'weapons'},
        {id:'resourceCollection', name:'📡 RESOURCE COLLECTION', level:resourceCollectionLevel, price:350, type:'resourceCollection', infinite:true, maxLimit:20, category:'shields'},
        {id:'weaponEnhancement', name:'⚔️ WEAPON ENHANCEMENT', level:weaponEnhancementLevel, price:450, type:'weaponEnhancement', infinite:true, maxLimit:20, category:'weapons'}
    ];

    // Filter by tab
    const tabItems = upgradeItems.filter(it => {
        if(currentShopTab === 'weapons') return it.category === 'weapons';
        if(currentShopTab === 'shields') return it.category === 'shields';
        return false; // special tab has no regular items
    });

    container.innerHTML='';
    for(const it of tabItems){
        const card=document.createElement('div');
        card.className='card tooltip';
        if(it.owned===true) card.classList.add('cant-afford');
        let content=`<div style="font-size:14px;">${it.name}</div>`;
        if(it.level!==undefined) content+=`<div style="font-size:9px;">LVL: ${it.level}${it.maxLimit?('/'+it.maxLimit):''}</div>`;
        if(it.owned!==undefined) content+=`<div style="font-size:9px;">${it.owned?'✅':'❌'}</div>`;
        content+=`<div style="color:gold;">${formatNumber(it.price)}c</div>`;
        if(it.infinite){
            let qty=selectedQuantities[it.type]||1;
            content+=`<div class="quantity-selector">
                <button class="qty-btn" onclick="event.stopPropagation();setQuantity('${it.type}',1)">1</button>
                <button class="qty-btn" onclick="event.stopPropagation();setQuantity('${it.type}',5)">5</button>
                <button class="qty-btn" onclick="event.stopPropagation();setQuantity('${it.type}',10)">10</button>
                <button class="qty-btn" onclick="event.stopPropagation();maxOutUpgrade('${it.type}',${it.price})">MAX</button>
                <span style="margin-right:5px;">x${qty}</span>
            </div>`;
        }
        let tooltipText = it.type==='fire' ? 'מגדיל את מהירות הירי' : it.type==='dmg' ? 'מגדיל את הנזק' : it.type==='shield' ? 'מוסיף מגן סביב החללית' : it.type==='drone' ? 'מוסיף רחפן שיורה אוטומטית' : it.type==='heal' ? 'מרפא 25 נקודות חיים' : it.type==='spread' ? 'מוסיף כדור נוסף לכל ירייה' : it.type==='laser' ? 'מוסיף לייזר חזק' : it.type==='resourceCollection' ? 'מגדיל את טווח איסוף אוטומטי של פריטים (+15 לרמה)' : it.type==='weaponEnhancement' ? 'מגדיל נזק וקצב אש (+5% לרמה)' : 'מוסיף פצצות לניקוי מסך';
        card.innerHTML = content + `<div class="tooltip-text">${tooltipText}</div>`;
        if(!(it.owned===true)){
            let qty=selectedQuantities[it.type]||1;
            let atMax=(it.type==='drone' && droneCount+qty>180);
            if(!atMax) card.onclick=()=>buyUpgrade(it.type, selectedQuantities[it.type]||1);
            else card.onclick=null;
        }
        container.appendChild(card);
    }

    // Build special items / abilities section based on tab
    if(specialSection){
        let specialHtml = '';
        if(currentShopTab === 'weapons'){
            // Special items for weapons tab
            const weaponsSpecial = SPECIAL_ITEMS_DATA.filter(s => s.category === 'weapons');
            specialHtml += '<h3 style="color:#ffaa00;margin-top:15px;">✨ SPECIAL ITEMS ✨</h3><div class="shop-grid" style="margin-top:8px;">';
            for(const si of weaponsSpecial){
                const owned = si.ownedCheck();
                const countInfo = si.countHtml ? `<div style="font-size:9px;color:#aaa;">${si.countHtml()}</div>` : '';
                specialHtml += `<div class="card tooltip ${owned?'cant-afford':''}" onclick="${owned?'':`buySpecialItem('${si.id}')`}">
                    <div style="font-size:14px;">${si.name}</div>
                    <div style="color:#ffaa00;font-size:10px;">${si.desc}</div>
                    <div style="color:gold;">${formatNumber(si.price)}c</div>
                    ${countInfo}
                    <div class="tooltip-text">${si.tooltip}</div>
                </div>`;
            }
            // Rare abilities for weapons tab
            const weaponsAbilities = Object.values(RARE_ABILITIES).filter(a => a.category === 'weapons');
            if(weaponsAbilities.length > 0){
                specialHtml += '</div><h3 style="color:#f54242;margin-top:15px;">🔥 RARE ABILITIES 🔥</h3><div class="shop-grid" style="margin-top:8px;">';
                for(const ab of weaponsAbilities){
                    const unlocked = ab.unlockReq();
                    const purchased = ab.purchased;
                    const canAfford = totalCoins >= ab.cost;
                    specialHtml += `<div class="card tooltip ${purchased?'cant-afford':''} rarity-legendary" onclick="${purchased?'':`buyRareAbility('${ab.id}')`}">
                        <div style="font-size:14px;">${ab.icon}</div>
                        <div style="font-size:10px;">${ab.name.replace(ab.icon+' ','')}</div>
                        <div style="color:#ffaa00;font-size:9px;">${ab.desc}</div>
                        <div style="color:gold;">${formatNumber(ab.cost)}c</div>
                        <div style="font-size:8px;color:${unlocked?'#00ffaa':'#ff4444'};">${purchased?'✅ OWNED':unlocked?'🔓 UNLOCKED':'🔒 '+ab.unlockDesc}</div>
                        ${!unlocked && !purchased ? `<div style="font-size:8px;color:#ff4444;">🔒 LOCKED</div>` : ''}
                        <div class="tooltip-text">Cooldown: ${ab.cooldown/1000}s. ${ab.unlockDesc}</div>
                    </div>`;
                }
            }
            specialHtml += '</div>';
        }
        else if(currentShopTab === 'shields'){
            // Special items for shields tab
            const shieldsSpecial = SPECIAL_ITEMS_DATA.filter(s => s.category === 'shields');
            specialHtml += '<h3 style="color:#ffaa00;margin-top:15px;">✨ SPECIAL ITEMS ✨</h3><div class="shop-grid" style="margin-top:8px;">';
            for(const si of shieldsSpecial){
                const owned = si.ownedCheck();
                const countInfo = si.countHtml ? `<div style="font-size:9px;color:#aaa;">${si.countHtml()}</div>` : '';
                specialHtml += `<div class="card tooltip ${owned?'cant-afford':''}" onclick="${owned?'':`buySpecialItem('${si.id}')`}">
                    <div style="font-size:14px;">${si.name}</div>
                    <div style="color:#ffaa00;font-size:10px;">${si.desc}</div>
                    <div style="color:gold;">${formatNumber(si.price)}c</div>
                    ${countInfo}
                    <div class="tooltip-text">${si.tooltip}</div>
                </div>`;
            }
            // Rare abilities for shields tab
            const shieldsAbilities = Object.values(RARE_ABILITIES).filter(a => a.category === 'shields');
            if(shieldsAbilities.length > 0){
                specialHtml += '</div><h3 style="color:#f54242;margin-top:15px;">🔥 RARE ABILITIES 🔥</h3><div class="shop-grid" style="margin-top:8px;">';
                for(const ab of shieldsAbilities){
                    const unlocked = ab.unlockReq();
                    const purchased = ab.purchased;
                    specialHtml += `<div class="card tooltip ${purchased?'cant-afford':''} rarity-legendary" onclick="${purchased?'':`buyRareAbility('${ab.id}')`}">
                        <div style="font-size:14px;">${ab.icon}</div>
                        <div style="font-size:10px;">${ab.name.replace(ab.icon+' ','')}</div>
                        <div style="color:#ffaa00;font-size:9px;">${ab.desc}</div>
                        <div style="color:gold;">${formatNumber(ab.cost)}c</div>
                        <div style="font-size:8px;color:${unlocked?'#00ffaa':'#ff4444'};">${purchased?'✅ OWNED':unlocked?'🔓 UNLOCKED':'🔒 '+ab.unlockDesc}</div>
                        ${!unlocked && !purchased ? `<div style="font-size:8px;color:#ff4444;">🔒 LOCKED</div>` : ''}
                        <div class="tooltip-text">${ab.cooldown>0?'Cooldown: '+(ab.cooldown/1000)+'s. ':''} ${ab.unlockDesc}</div>
                    </div>`;
                }
            }
            specialHtml += '</div>';
        }
        else if(currentShopTab === 'special'){
            // All special/ultimate abilities
            specialHtml = '<h3 style="color:#f542d1;margin-top:15px;">🌟 ALL SPECIAL ABILITIES 🌟</h3><div class="shop-grid" style="margin-top:8px;">';
            for(const ab of Object.values(RARE_ABILITIES)){
                const unlocked = ab.unlockReq();
                const purchased = ab.purchased;
                specialHtml += `<div class="card tooltip ${purchased?'cant-afford':''} rarity-mythic" onclick="${purchased?'':`buyRareAbility('${ab.id}')`}">
                    <div style="font-size:14px;">${ab.icon}</div>
                    <div style="font-size:10px;">${ab.name.replace(ab.icon+' ','')}</div>
                    <div style="color:#ffaa00;font-size:9px;">${ab.desc}</div>
                    <div style="color:gold;">${formatNumber(ab.cost)}c</div>
                    <div style="font-size:8px;color:${unlocked?'#00ffaa':'#ff4444'};">${purchased?'✅ OWNED':unlocked?'🔓 UNLOCKED':'🔒 '+ab.unlockDesc}</div>
                    ${!unlocked && !purchased ? `<div style="font-size:8px;color:#ff4444;">🔒 LOCKED</div>` : ''}
                    <div class="tooltip-text">${ab.cooldown>0?'Cooldown: '+(ab.cooldown/1000)+'s. ':''}Category: ${ab.category}. ${ab.unlockDesc}</div>
                </div>`;
            }
            // Also show all special items
            specialHtml += '</div><h3 style="color:#ffaa00;margin-top:15px;">✨ ALL SPECIAL ITEMS ✨</h3><div class="shop-grid" style="margin-top:8px;">';
            for(const si of SPECIAL_ITEMS_DATA){
                const owned = si.ownedCheck();
                const countInfo = si.countHtml ? `<div style="font-size:9px;color:#aaa;">${si.countHtml()}</div>` : '';
                specialHtml += `<div class="card tooltip ${owned?'cant-afford':''}" onclick="${owned?'':`buySpecialItem('${si.id}')`}">
                    <div style="font-size:14px;">${si.name}</div>
                    <div style="color:#ffaa00;font-size:10px;">${si.desc}</div>
                    <div style="color:gold;">${formatNumber(si.price)}c</div>
                    ${countInfo}
                    <div class="tooltip-text">${si.tooltip}</div>
                </div>`;
            }
            specialHtml += '</div>';
        }
        specialSection.innerHTML = specialHtml;
    }

    document.getElementById('shop-money').innerHTML="CREDITS: "+formatNumber(totalCoins);
}
}
function openShop(){
    document.getElementById('start-screen').style.display='none';
    document.getElementById('shop-screen').style.display='flex';
    currentShopTab = 'weapons';
    document.querySelectorAll('.shop-tab').forEach((btn,i) => btn.classList.toggle('active', i===0));
    updateShopUI();
}
function closeShop(){
    document.getElementById('shop-screen').style.display='none';
    document.getElementById('start-screen').style.display='flex';
}

// PLAYER CLASS
class Player{
    constructor(){this.x=width/2;this.y=height-120;this.tx=this.x;this.ty=this.y;this.r=25;this.invincibleTimer=0;this.deathProtection=false;}
    update(){
        if(bugEventGlitch){
            this.tx = width/2 + (Math.random() - 0.5) * 200;
            this.ty = height/2 + (Math.random() - 0.5) * 200;
        }
        this.x+=(this.tx-this.x)*0.2*(1+getSkillBonus('moveSpeed'));this.y+=(this.ty-this.y)*0.2*(1+getSkillBonus('moveSpeed'));
        this.x=Math.max(this.r,Math.min(width-this.r,this.x));
        this.y=Math.max(this.r,Math.min(height-this.r,this.y));
        if(this.invincibleTimer>0)this.invincibleTimer--;
        const tc=isOD?'#ff00ff':synapseActive?'#ff66ff':voidActive?'#aa66ff':primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:apocalypseActive?'#ff0000':riftActive?'#aa66ff':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffdd00':prismActive?'#ff88ff':doppelgangerActive?'#ff88ff':activePowerUps.rapidfire?'#00ffff':'#00d2ff';
        if(Math.random()>0.5) particles.push(new Particle(this.x,this.y+28,(Math.random()-0.5)*2,Math.random()*4+2,tc,0.7));
    }
    draw(){
        ctx.save();
        if(settings.shake && shake>0)ctx.translate((Math.random()-0.5)*shake,(Math.random()-0.5)*shake);
        if(bugEventGlitch){
            ctx.translate((Math.random()-0.5)*10, (Math.random()-0.5)*10);
        }
        ctx.translate(this.x,this.y);
        let maxDrones = Math.min(180, droneCount);
        for(let i=0;i<maxDrones;i++){
            const a=(Date.now()/400)+(i*Math.PI*2/maxDrones);
            const dx=Math.cos(a)*55,dy=Math.sin(a)*45;
            ctx.fillStyle='#00ffaa';ctx.shadowBlur=8;
            ctx.fillRect(dx-5,dy-5,10,10);
            ctx.beginPath();ctx.moveTo(0,0);ctx.lineTo(dx,dy);ctx.stroke();
        }
        if(hasShieldUpgrade){
            ctx.strokeStyle=`rgba(0,255,204,${0.3+Math.sin(Date.now()/400)*0.2})`;
            ctx.lineWidth=1.5;ctx.setLineDash([3,6]);
            ctx.beginPath();ctx.arc(0,0,55,0,Math.PI*2);ctx.stroke();ctx.setLineDash([]);
        }
        if(this.invincibleTimer>0&&Math.floor(this.invincibleTimer/5)%2===0){ctx.restore();return;}
        let col=isOD?'#ff00ff':synapseActive?'#ff66ff':voidActive?'#aa66ff':primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:apocalypseActive?'#ff0000':riftActive?'#aa66ff':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffdd00':prismActive?'#ff88ff':doppelgangerActive?'#ff88ff':activePowerUps.doubleDmg?'#ff8800':'#00d2ff';
        
        if(currentSkin === 'gold' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive) col='#ffd700';
        if(currentSkin === 'blue' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive && currentSkin !== 'gold') col='#4287f5';
        if(currentSkin === 'purple' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive && currentSkin !== 'gold' && currentSkin !== 'blue') col='#aa44ff';
        if(currentSkin === 'rainbow' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive && currentSkin !== 'gold' && currentSkin !== 'blue' && currentSkin !== 'purple') col=`hsl(${Date.now()/10 % 360},100%,60%)`;
        if(currentSkin === 'ultra' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive && currentSkin !== 'gold' && currentSkin !== 'blue' && currentSkin !== 'purple' && currentSkin !== 'rainbow') col='#f5e642';
        if(currentSkin === 'legend' && !isOD && !synapseActive && !voidActive && !apocalypseActive && !riftActive && !primordialRageActive && !chaosRealmActive && !bugEventActive && !stableCycleActive && !lightningStormActive && !royalBlessingActive && !kingsBlessingActive && !prismActive && !doppelgangerActive) col='#ff6600';
        
        ctx.fillStyle=col;ctx.shadowBlur=15;
        if(primordialRageActive) ctx.scale(2,2);
        ctx.beginPath();
        ctx.moveTo(0,-32);ctx.lineTo(28,18);ctx.lineTo(10,18);ctx.lineTo(10,32);
        ctx.lineTo(-10,32);ctx.lineTo(-10,18);ctx.lineTo(-28,18);ctx.closePath();ctx.fill();
        ctx.fillStyle='rgba(255,255,255,0.3)';
        ctx.beginPath();ctx.ellipse(0,-6,5,12,0,0,Math.PI*2);ctx.fill();
        if(currentSkin === 'gold' || currentSkin === 'ultra' || currentSkin === 'legend'){
            ctx.strokeStyle='gold';ctx.lineWidth=2;ctx.beginPath();ctx.arc(0,0,38,0,Math.PI*2);ctx.stroke();
        }
        if(primordialRageActive) ctx.scale(0.5,0.5);
        ctx.restore();
    }
}

// ENEMY CLASS
class Enemy{
    constructor(isBoss=false){
        this.isBoss=isBoss;
        this.type=isBoss?'boss':['normal','zigzag','fast','tank'][Math.floor(Math.random()*4)];
        this.x=isBoss?width/2:Math.random()*(width-70)+35;
        this.y=isBoss?-120:-60;
        let lm = 1 + (level-1)*0.02 + (wave-1)*0.015;
        if(endlessMode && wave>100) lm *= (1 + (wave-100)*0.002);
        if(apocalypseActive) lm *= 1.5;
        if(cosmicCollapseActive) lm *= 1.3;
        if(doomsDayActive) lm *= 1.5;
        if(timeDistortionPurchased) lm *= enemySlow;
        if(isBoss){
            this.hp = Math.floor((100 + level*20) * lm);
        } else {
            switch(this.type){
                case 'tank': this.hp = 3; break;
                default: this.hp = 2; break;
            }
            this.hp = Math.floor(this.hp * lm);
        }
        this.maxHp=this.hp;
        this.r=isBoss?90:this.type==='tank'?35:25;
        let speedMulti = synapseActive ? 0.45 : (cosmicCollapseActive?1.2:(apocalypseActive?1.6:(riftActive?1.2:1)));
        if(timeWarpActive) speedMulti *= 0.2;
        if(abilityTimeWarpActive) speedMulti *= 0.5;
        if(frozenTimeActive) speedMulti = 0;
        if(doomsDayActive) speedMulti *= 0.7;
        if(timeDistortionPurchased) speedMulti *= enemySlow;
        if(endlessMode && wave>100) speedMulti *= (1 + (wave-100)*0.001);
        this.speed=isBoss?0.35*speedMulti:{normal:1.6,zigzag:1.6,fast:3.5,tank:0.8}[this.type]*speedMulti + Math.random()*0.3;
        this.color=isBoss?'#ff0044':{normal:'#ff4444',zigzag:'#ff8800',fast:'#00ffff',tank:'#aa44ff'}[this.type];
        this.rot=0;this.lastShot=0;this.zigDir=Math.random()<0.5?1:-1;this.zigTimer=0;
        
        if(masqueradeActive && !isBoss){
            this.disguise = 'item';
            this.originalColor = this.color;
            this.color = '#ffaa00';
        }
    }
    update(){
        if(!frozenTimeActive){
            this.y+=this.speed;
            this.rot+=0.04;
        }
        if(this.type==='zigzag'){this.zigTimer++;if(this.zigTimer%50===0)this.zigDir*=-1;this.x+=this.zigDir*1.8;this.x=Math.max(22,Math.min(width-22,this.x));}
        if(this.isBoss){
            this.x+=Math.sin(Date.now()/900)*1.2;
            let shotDelay = synapseActive?1000:650;
            if(apocalypseActive) shotDelay*=0.6;
            if(cosmicCollapseActive) shotDelay*=0.7;
            if(riftActive) shotDelay*=0.8;
            if(frozenTimeActive) shotDelay=999999;
            if(doomsDayActive) shotDelay*=1.3;
            if(timeDistortionPurchased) shotDelay*=1.3;
            if(Date.now()-this.lastShot>shotDelay){
                this.lastShot=Date.now();
                for(let s=-2;s<=2;s++)eBullets.push({x:this.x+s*28,y:this.y+40,vx:s*0.9,vy:4.5});
            }
        } else if(this.type!=='tank'&&Math.random()<0.0018+level*0.00012){
            eBullets.push({x:this.x,y:this.y+20,vx:0,vy:3.5+Math.random()*1.5});
        } else if(this.type==='tank'&&Math.random()<0.004){
            eBullets.push({x:this.x-12,y:this.y+22,vx:-1.2,vy:3.5});
            eBullets.push({x:this.x+12,y:this.y+22,vx:1.2,vy:3.5});
        }
    }
    draw(){
        ctx.save();ctx.translate(this.x,this.y);
        if(frozenTimeActive){
            ctx.fillStyle = '#88ccff'; ctx.shadowBlur = 10;
            ctx.beginPath(); ctx.arc(0,0,this.r+5,0,Math.PI*2); ctx.fill();
            ctx.fillStyle = '#fff'; ctx.font = 'bold 20px Segoe UI'; ctx.fillText('❄️', -10, 10);
        } else if(this.disguise === 'item'){
            ctx.fillStyle = '#ffaa00'; ctx.shadowBlur = 10;
            ctx.beginPath(); ctx.arc(0,0,15,0,Math.PI*2); ctx.fill();
            ctx.fillStyle = '#fff'; ctx.font = 'bold 20px Segoe UI'; ctx.fillText('?', -8, 8);
        } else {
            ctx.strokeStyle=this.color;ctx.shadowBlur=12;
            if(this.isBoss){
                ctx.lineWidth=3;ctx.rotate(this.rot*0.3);
                ctx.strokeRect(-60,-60,120,120);ctx.rotate(this.rot*0.2);ctx.strokeRect(-32,-32,64,64);
                ctx.fillStyle='rgba(255,0,68,0.2)';ctx.beginPath();ctx.arc(0,0,22,0,Math.PI*2);ctx.fill();
                ctx.fillStyle='#222';ctx.fillRect(-50,-90,100,8);
                ctx.fillStyle=`hsl(${(this.hp/this.maxHp)*120},100%,50%)`;ctx.fillRect(-50,-90,(this.hp/this.maxHp)*100,8);
            } else {
                ctx.lineWidth=this.type==='tank'?3.5:2;ctx.rotate(this.rot);
                if(this.type==='fast'){ctx.beginPath();ctx.moveTo(0,-18);ctx.lineTo(18,18);ctx.lineTo(-18,18);ctx.closePath();ctx.stroke();}
                else if(this.type==='tank'){ctx.strokeRect(-22,-22,44,44);ctx.strokeRect(-10,-10,20,20);}
                else{ctx.strokeRect(-14,-14,28,28);}
                if(this.hp<this.maxHp){
                    ctx.restore();ctx.save();ctx.translate(this.x,this.y);
                    ctx.fillStyle='#222';ctx.fillRect(-15,-32,30,5);
                    ctx.fillStyle=this.color;ctx.fillRect(-15,-32,(this.hp/this.maxHp)*30,5);
                }
            }
        }
        ctx.restore();
    }
}

// BULLET CLASS
class Bullet{
    constructor(x,y,power,color,angle=-Math.PI/2,isLaser=false){
        this.x=x;this.y=y;this.p=power;this.c=color;this.angle=angle;this.isLaser=isLaser;
        let speedMulti = synapseActive ? 1.6 : (primordialRageActive?2.5:(chaosRealmActive?5:(apocalypseActive?1.4:(riftActive?1.3:1))));
        if(bugEventActive) speedMulti *= 2;
        if(stableCycleActive) speedMulti *= 1.5;
        if(frozenTimeActive) speedMulti *= 0.5;
        if(royalBlessingActive) speedMulti *= 1.5;
        if(kingsBlessingActive) speedMulti *= 2;
        if(prismActive) speedMulti *= 1.2;
        if(doppelgangerActive) speedMulti *= 1.1;
        speedMulti *= skinFireRateMultiplier;
        this.speed=(isLaser?26:16)*speedMulti;
        this.vx=Math.cos(angle)*this.speed;this.vy=Math.sin(angle)*this.speed;
        this.size=isLaser?6:3;
    }
    update(){this.x+=this.vx;this.y+=this.vy;}
    draw(){
        let bulletColor=this.c;
        if(chaosRealmActive) bulletColor = `hsl(${chaosHue + this.x + this.y}, 100%, 60%)`;
        if(bugEventActive) bulletColor = `hsl(${Date.now()/50 % 360}, 100%, 60%)`;
        if(crystalRainActive) bulletColor = `hsl(${Date.now()/30 % 360}, 100%, 60%)`;
        if(spaceTreasureActive) bulletColor = `#ffd700`;
        if(lightningStormActive) bulletColor = `#ffff00`;
        if(royalBlessingActive) bulletColor = `#ffdd00`;
        if(kingsBlessingActive) bulletColor = `#ffaa00`;
        if(prismActive) bulletColor = `hsl(${this.x + this.y}, 100%, 60%)`;
        if(doppelgangerActive) bulletColor = `#ff88ff`;
        ctx.fillStyle=bulletColor;ctx.shadowBlur=this.isLaser?12:7;
        ctx.save();ctx.translate(this.x,this.y);ctx.rotate(this.angle+Math.PI/2);
        const h=this.isLaser?22:14;ctx.fillRect(-this.size/2,-h/2,this.size,h);
        ctx.restore();
    }
}

// PARTICLE CLASS
class Particle{
    constructor(x,y,vx,vy,color,life){this.x=x;this.y=y;this.vx=vx;this.vy=vy;this.c=color;this.l=life;this.size=Math.random()*3+1.5;}
    update(){this.x+=this.vx;this.y+=this.vy;this.vy+=0.05;this.l-=0.02;}
    draw(){ctx.globalAlpha=Math.max(0,this.l);ctx.fillStyle=this.c;ctx.fillRect(this.x,this.y,this.size,this.size);ctx.globalAlpha=1;}
}

// GAME FLOW
let gameLoopRunning = false;

function startGame(){
    initAudio();
    updateSkinEffects();
    updateRankUI();
    resetDailyMissions();
    updateDailyMissionsUI();
    updateMusicBasedOnGameState(); // MUSIC
    
    gameState='PLAYING'; endlessMode=false; ascendTriggered=false;
    score=0;health=100+getSkillBonus('maxHealth');maxHealth=100+getSkillBonus('maxHealth');xp=0;level=1;odCharge=0;combo=1;kills=0;
    bossWarningTimer=0;waveBannerTimer=0;isPaused=false;
    wave=1;waveKills=0;waveTriggered=false;
    odActivations=0;bossesKilled=0;waveNoDamage=true;perfectWavesCount=0;
    totalOverdriveUses=0;totalBombsUsed=0;maxCombo=0;
    synapseActive=false;synapsePsychedeliaCount=0;synapseKills=0;
    activeEvent=null;riftActive=false;riftMultiplier=1;meteors=[];
    apocalypseActive=false;voidActive=false;guardian=null;
    cosmicCollapseActive=false;primordialRageActive=false;blackHoleActive=false;
    chaosRealmActive=false;timeWarpActive=false;goldRushActive=false;divineActive=false;bugEventActive=false;bugEventGlitch=false;
    stableCycleActive=false;tidalWaveActive=false;masqueradeActive=false;soulHarvestActive=false;soulHarvestSouls=[];
    frozenTimeActive=false;crystalRainActive=false;shadowCloneActive=false;shadowClone=null;crystals=[];
    lightningStormActive=false;doomsDayActive=false;royalBlessingActive=false;greenAuraActive=false;asteroidBeltActive=false;asteroids=[];supplyDropActive=false;supplyDropCrate=null;
    starfallActive=false;starfallStars=[];infernoActive=false;chainLightningActive=false;barrierActive=false;barrierHp=0;
    soulReaperActive=false;soulReaperSoul=null;gamblerActive=false;vortexActive=false;kingsBlessingActive=false;prismActive=false;abyssActive=false;
    doppelgangerActive=false;doppelgangerClone=null;supernovaActive=false;
    spaceTreasureActive=false;spaceTreasureCrates=[];
    divineFlashTimer=0;eventQueue=[];
    shockwaveActive=false;shockwaveTimer=0;shockwaveRadius=0;
    timeSlowActive=false;timeSlowTimer=0;
    gravityBombActive=false;gravityBombTimer=0;gravityBombPhase=0;
    regenAuraTimer=0;lightningStrikeEffect=null;
    resourceMagnetActive=false;resourceMagnetTimer=0;
    abilityTimeWarpActive=false;abilityTimeWarpTimer=0;
    ultimateAnnihilationActive=false;ultimateAnnihilationTimer=0;
    blackHolePieces=0; eventCooldown=0;
    startTime=Date.now();
    enemies=[];bullets=[];eBullets=[];particles=[];items=[];floats=[];boss=null;activePowerUps={};
    player=new Player();
    document.getElementById('start-screen').style.display='none';
    document.getElementById('ascend-screen').style.display='none';
    ['ui-hud','score-hud','combo-small','powerup-bar','pause-btn','game-timer','rank-badge','combo-meter','wave-progress'].forEach(id=>document.getElementById(id).style.display='block');
    document.getElementById('od-btn').style.display='none';
    document.getElementById('ability-btns').style.display='none';
    document.getElementById('crosshair').style.display='block';
    // New UI elements
    document.getElementById('auto-fire-btn').style.display = 'block';
    if(settings.autoFire) document.getElementById('auto-fire-btn').classList.add('active');
    else document.getElementById('auto-fire-btn').classList.remove('active');
    if(settings.showFPS) document.getElementById('fps-counter').style.display = 'block';
    else document.getElementById('fps-counter').style.display = 'none';
    startWave(1);
    
    if(timerInterval) clearInterval(timerInterval);
    timerInterval = setInterval(() => {
        if(gameState === 'PLAYING' && player && player.x && player.y){
            let elapsed = Math.floor((Date.now() - startTime) / 1000);
            let minutes = Math.floor(elapsed / 60);
            let seconds = elapsed % 60;
            document.getElementById('timer-display').innerText = `${minutes.toString().padStart(2,'0')}:${seconds.toString().padStart(2,'0')}`;
        }
    }, 1000);
    
    if(animationId) cancelAnimationFrame(animationId);
    gameLoopRunning = true;
    function gameLoop(){
        if(gameLoopRunning) loop();
        animationId = requestAnimationFrame(gameLoop);
    }
    gameLoop();
    
    let autoSaveCount = parseInt(localStorage.getItem('autoSaveCount')) || 0;
    autoSaveCount++;
    localStorage.setItem('autoSaveCount', autoSaveCount);
}

function togglePause(){
    if(gameState==='PLAYING'){gameState='PAUSED';isPaused=true;document.getElementById('pause-screen').style.display='flex';document.getElementById('pause-btn').innerText='▶';closeHomeAbilityPanel();}
    else if(gameState==='PAUSED'){gameState='PLAYING';isPaused=false;document.getElementById('pause-screen').style.display='none';document.getElementById('pause-btn').innerText='⏸';}
}

function quitToMenu(){
    stopBackgroundMusic();
    playBackgroundMusic('menu');
    
    gameState='MENU';isPaused=false;

    // Save earned credits before quitting
    const earned = Math.floor(score / 10);
    totalCoins += earned;
    totalKills += kills;
    if(score > hiScore) hiScore = score;
    localStorage.setItem('totalCoins', totalCoins);
    localStorage.setItem('hiScore', hiScore);
    localStorage.setItem('totalKills', totalKills);
    if(earned > 0) tn('noti_credits_saved', 'success', formatNumber(earned));

    gameLoopRunning = false;
    if(player) player=null;
    if(timerInterval) clearInterval(timerInterval);
    if(animationId) cancelAnimationFrame(animationId);
    ['game-over','pause-screen','ui-hud','score-hud','combo-small','od-btn','pause-btn','powerup-bar','wave-banner','boss-warning','event-banner','ascend-screen','start-screen','game-timer','rank-badge','combo-meter','wave-progress'].forEach(id=>{
        const el=document.getElementById(id);
        if(el) el.style.display='none';
    });
    document.getElementById('ability-btns').style.display='none';
    closeHomeAbilityPanel();
    document.getElementById('vignette').style.display='none';
    document.getElementById('crosshair').style.display='none';
    document.getElementById('auto-fire-btn').style.display='none';
    document.getElementById('fps-counter').style.display='none';
    document.getElementById('start-screen').style.display='flex';
    updateMainMenuUI();
}

function triggerReboot(){ 
    document.getElementById('game-over').style.display='none'; 
    gameState = 'LOADING';
    gameLoopRunning = false;
    if(player) player = null;
    if(timerInterval) clearInterval(timerInterval);
    if(animationId) cancelAnimationFrame(animationId);
    enemies = [];
    bullets = [];
    eBullets = [];
    particles = [];
    items = [];
    floats = [];
    meteors = [];
    crystals = [];
    soulHarvestSouls = [];
    starfallStars = [];
    shadowClone = null;
    soulReaperSoul = null;
    doppelgangerClone = null;
    boss = null;
    guardian = null;
    activePowerUps = {};
    startGame();
}

function activateOverdrive(){
    if(odCharge<100)return;initAudio();sfxOverdrive();odActivations++;totalOverdriveUses++;
    isOD=true;odTimer=Date.now()+6000;if(settings.shake) shake=18;
    document.getElementById('od-btn').style.display='none';
    if(player && player.x && player.y) floats.push({txt:'⚡ OVERDRIVE!',x:player.x-50,y:player.y-20,l:1.5,c:'#ff00ff',size:24});
    checkAchievements();
    showNotification('⚡ Overdrive activated!', 'success');
    tn('noti_overdrive', 'success');
}

function gameOver(){
    if(guardianAngelCount > 0 && !guardianAngelUsed){
        guardianAngelUsed = true;
        guardianAngelCount--;
        localStorage.setItem('guardianAngelCount', guardianAngelCount);
        health = maxHealth;
        if(player) player.invincibleTimer = 180;
        showCustomAlert(`🛡️ GUARDIAN ANGEL saved you! (${guardianAngelCount}/3 remaining)`);
        tn('noti_guardian_saved', 'success', guardianAngelCount);
        updateShopUI();
        return;
    }
    
    gameState='GAMEOVER';
    gameLoopRunning = false;
    const earned=Math.floor(score/10);
    totalCoins+=earned;totalKills+=kills;
    if(score>hiScore)hiScore=score;
    localStorage.setItem('totalCoins',totalCoins);localStorage.setItem('hiScore',hiScore);localStorage.setItem('totalKills',totalKills);
    if(timerInterval) clearInterval(timerInterval);
    if(animationId) cancelAnimationFrame(animationId);
    ['ui-hud','score-hud','od-btn','combo-small','pause-btn','powerup-bar','boss-warning','wave-banner','event-banner','ascend-screen','game-timer','rank-badge','combo-meter','wave-progress'].forEach(id=>document.getElementById(id).style.display='none');
    document.getElementById('ability-btns').style.display='none';
    closeHomeAbilityPanel();
    document.getElementById('vignette').style.display='none';
    document.getElementById('crosshair').style.display='none';
    document.getElementById('auto-fire-btn').style.display='none';
    document.getElementById('fps-counter').style.display='none';
    document.getElementById('game-over').style.display='flex';
    const isNew=score>=hiScore&&score>0;
    document.getElementById('final-stats').innerHTML=
        `<div>SCORE: <span style="color:#00d2ff">${formatNumber(score)}</span>${isNew?' 🏆 NEW!':''}</div>
         <div>KILLS: <span style="color:#ff4444">${formatNumber(kills)}</span></div>
         <div>WAVE: <span style="color:#00ffaa">${wave}</span></div>
         <div>RANK: <span style="color:#00ffaa">${level}</span></div>
         <div style="color:gold">CREDITS: +${formatNumber(earned)}</div>`;
}

function fireBullets(){
    if(!player || !player.x || !player.y) return;
    const isRapid=!!activePowerUps.rapidfire,isPower=!!activePowerUps.doubleDmg;
    let power = damageLevel*(isPower?2:1);
    power = applyCriticalHit(power);
    power *= skinDamageMultiplier;
    power *= (1 + weaponEnhancementLevel * 0.05);
    power *= (1 + getSkillBonus('damage'));
    if(synapseActive) power *= 2.5;
    if(voidActive) power *= 10;
    if(divineActive) power *= 4; // +300% damage during Divine Intervention
    if(primordialRageActive) power *= 20;
    if(chaosRealmActive) power *= 3;
    if(apocalypseActive) power *= 2;
    if(cosmicCollapseActive) power *= 3;
    if(riftActive) power *= riftMultiplier;
    if(goldRushActive) power *= 2;
    if(bugEventActive) power *= 5;
    if(stableCycleActive) power *= 1.5;
    if(royalBlessingActive) power *= 3;
    if(kingsBlessingActive) power *= 5;
    if(prismActive) power *= 1.5;
    let color=isOD?'#ff00ff':synapseActive?'#ff66ff':voidActive?'#aa66ff':primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:apocalypseActive?'#ff0000':cosmicCollapseActive?'#88aaff':riftActive?'#aa66ff':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffaa00':prismActive?'#ff88ff':doppelgangerActive?'#ff88ff':isRapid?'#00ffff':'#00d2ff';
    const px=player.x,py=player.y-28;
    if(primordialRageActive){
        for(let i=-3;i<=3;i++){
            bullets.push(new Bullet(px+i*12,py,power,color,-Math.PI/2-0.15*i));
            bullets.push(new Bullet(px+i*12,py,power,color,-Math.PI/2+0.15*i));
        }
    } else if(chaosRealmActive){
        for(let i=-2;i<=2;i++){
            let chaosColor = `hsl(${chaosHue + i*60},100%,60%)`;
            bullets.push(new Bullet(px+i*10,py,power,chaosColor,-Math.PI/2-0.2*i));
            bullets.push(new Bullet(px+i*10,py,power,chaosColor,-Math.PI/2+0.2*i));
        }
        bullets.push(new Bullet(px,py,power,color));
    } else if(prismActive){
        for(let i=-3;i<=3;i++){
            let prismColor = `hsl(${i*60},100%,60%)`;
            bullets.push(new Bullet(px+i*8,py,power,prismColor,-Math.PI/2-0.1*i));
            bullets.push(new Bullet(px+i*8,py,power,prismColor,-Math.PI/2+0.1*i));
        }
    } else {
        bullets.push(new Bullet(px,py,power,color));
        if(hasSpreadShot){bullets.push(new Bullet(px,py,power,color,-Math.PI/2-0.22));bullets.push(new Bullet(px,py,power,color,-Math.PI/2+0.22));}
        if(hasLaser&&Math.random()<0.3){sfxLaser();bullets.push(new Bullet(px,py,power*1.3,'#ff44ff',-Math.PI/2,true));}
    }
    if(isOD){bullets.push(new Bullet(px-15,py+8,power,color));bullets.push(new Bullet(px+15,py+8,power,color));}
    sfxShoot();lastFire=Date.now();
}

// MAIN LOOP (abbreviated for space - same as before but with music update)
function loop(){
    // Update event timers
    if(starfallActive && Date.now()>starfallTimer) starfallActive=false;
    if(infernoActive && Date.now()>infernoTimer) infernoActive=false;
    if(chainLightningActive && Date.now()>chainLightningTimer) chainLightningActive=false;
    if(barrierActive && Date.now()>barrierTimer) barrierActive=false;
    if(soulReaperActive && Date.now()>soulReaperTimer) soulReaperActive=false;
    if(vortexActive && Date.now()>vortexTimer) vortexActive=false;
    if(kingsBlessingActive && Date.now()>kingsBlessingTimer) kingsBlessingActive=false;
    if(prismActive && Date.now()>prismTimer) prismActive=false;
    if(abyssActive && Date.now()>abyssTimer) abyssActive=false;
    if(doppelgangerActive && Date.now()>doppelgangerTimer) { doppelgangerActive=false; doppelgangerClone=null; }
    if(supernovaActive && Date.now()>supernovaTimer) supernovaActive=false;
    
    if(stableCycleActive && Date.now()>stableCycleTimer) stableCycleActive=false;
    if(tidalWaveActive && Date.now()>tidalWaveTimer) tidalWaveActive=false;
    if(masqueradeActive && Date.now()>masqueradeTimer) masqueradeActive=false;
    if(soulHarvestActive && Date.now()>soulHarvestTimer) soulHarvestActive=false;
    if(timeWarpActive && Date.now()>timeWarpTimer) timeWarpActive=false;
    if(goldRushActive && Date.now()>goldRushTimer) goldRushActive=false;
    if(divineActive && Date.now()>divineTimer) divineActive=false;
    if(bugEventActive && Date.now()>eventTimer) { bugEventActive=false; bugEventGlitch=false; }
    if(frozenTimeActive && Date.now()>eventTimer) frozenTimeActive=false;
    if(crystalRainActive && Date.now()>eventTimer) crystalRainActive=false;
    if(shadowCloneActive && Date.now()>eventTimer) { shadowCloneActive=false; shadowClone=null; }
    if(lightningStormActive && Date.now()>eventTimer) lightningStormActive=false;
    if(doomsDayActive && Date.now()>eventTimer) doomsDayActive=false;
    if(royalBlessingActive && Date.now()>royalBlessingTimer) royalBlessingActive=false;
    if(chaosRealmActive) chaosHue = (chaosHue + 3) % 360;
    if(synapseActive && Date.now()>synapseTimer){ synapseActive=false; document.getElementById('event-banner').style.display='none'; }
    if(voidActive && Date.now()>voidTimer){ voidActive=false; document.getElementById('event-banner').style.display='none'; }
    if(primordialRageActive && Date.now()>eventTimer){ primordialRageActive=false; if(player) player.r=25; document.getElementById('event-banner').style.display='none'; }
    if(chaosRealmActive && Date.now()>eventTimer){ chaosRealmActive=false; document.getElementById('event-banner').style.display='none'; }
    if(cosmicCollapseActive && Date.now()>eventTimer){ cosmicCollapseActive=false; document.getElementById('event-banner').style.display='none'; }
    if(blackHoleActive && Date.now()>eventTimer){ blackHoleActive=false; document.getElementById('event-banner').style.display='none'; }
    if(activeEvent && Date.now()>eventTimer){
        activeEvent=null; riftActive=false; apocalypseActive=false; cosmicCollapseActive=false;
        primordialRageActive=false; blackHoleActive=false; chaosRealmActive=false; timeWarpActive=false; goldRushActive=false; divineActive=false; bugEventActive=false; bugEventGlitch=false;
        stableCycleActive=false; tidalWaveActive=false; masqueradeActive=false; soulHarvestActive=false; frozenTimeActive=false; crystalRainActive=false; shadowCloneActive=false;
        lightningStormActive=false; doomsDayActive=false; royalBlessingActive=false; starfallActive=false; infernoActive=false; chainLightningActive=false; barrierActive=false;
        soulReaperActive=false; vortexActive=false; kingsBlessingActive=false; prismActive=false; abyssActive=false; doppelgangerActive=false; supernovaActive=false; spaceTreasureActive=false; asteroidBeltActive=false; asteroids=[]; supplyDropActive=false; supplyDropCrate=null; greenAuraActive=false;
        riftMultiplier=1;
        document.getElementById('event-banner').style.display='none'; startEventCooldown();
        processEventQueue();
    }
    
    // Starfall effect (simplified)
    if(starfallActive){
        for(let i=0;i<starfallStars.length;i++){
            starfallStars[i].update();
            starfallStars[i].draw();
            if(starfallStars[i].y>height){
                starfallStars.splice(i,1);
                i--;
            } else if(player && Math.hypot(starfallStars[i].x-player.x, starfallStars[i].y-player.y) < starfallStars[i].size+player.r){
                let bonus = starfallStars[i].value * gemMultiplier;
                gemstones += bonus;
                saveGemstones();
                particles.push(new Particle(starfallStars[i].x,starfallStars[i].y,0,0,'#ffffaa',0.8));
                starfallStars.splice(i,1);
                i--;
                showNotification(`⭐ Star collected! +${bonus} GEMSTONES!`, 'success');
            }
        }
        if(Math.random()<0.08){
            starfallStars.push(new StarfallStar(Math.random()*width, -20, 8+Math.random()*8, 30+Math.floor(Math.random()*60)));
        }
    }

    // Space Treasure event logic
    if(spaceTreasureActive && Date.now()<spaceTreasureTimer){
        // Spawn new crates periodically
        if(Date.now()%1000<20 && spaceTreasureCrates.length<8){
            const rewardType = ['gem','gem','credit','credit','bomb'][Math.floor(Math.random()*5)];
            let amount;
            if(rewardType==='gem') amount = 50+Math.floor(Math.random()*450);
            else if(rewardType==='credit') amount = 1000+Math.floor(Math.random()*9000);
            else amount = 1+Math.floor(Math.random()*3);
            spaceTreasureCrates.push({x:50+Math.random()*(width-100), y:-30, speed:1.5+Math.random()*2, rewardType, amount});
        }
        // Update and draw crates
        for(let i=spaceTreasureCrates.length-1;i>=0;i--){
            let c=spaceTreasureCrates[i];
            c.y+=c.speed;
            // Draw crate
            ctx.fillStyle='#ffd700'; ctx.fillRect(c.x-15,c.y-15,30,30);
            ctx.strokeStyle='#aa8800'; ctx.lineWidth=2; ctx.strokeRect(c.x-15,c.y-15,30,30);
            ctx.fillStyle='#000'; ctx.font='bold 14px sans-serif'; ctx.textAlign='center'; ctx.fillText('🎁',c.x,c.y+5); ctx.textAlign='start';
            // Sparkle effect
            if(Math.random()<0.3) particles.push(new Particle(c.x,c.y,(Math.random()-0.5)*3,(Math.random()-0.5)*3,'#ffd700',0.5));
            // Collection check
            if(player && Math.hypot(c.x-player.x,c.y-player.y)<50){
                if(c.rewardType==='gem'){ gemstones+=c.amount; saveGemstones(); floats.push({txt:'+'+c.amount+' 💎',x:c.x,y:c.y-20,l:1,c:'#ff66ff',size:14}); }
                else if(c.rewardType==='credit'){ totalCoins+=c.amount; localStorage.setItem('totalCoins',totalCoins); floats.push({txt:'+'+formatNumber(c.amount)+' 💰',x:c.x,y:c.y-20,l:1,c:'#00ffaa',size:14}); }
                else { bombCount+=c.amount; localStorage.setItem('bombCount',bombCount); floats.push({txt:'+'+c.amount+' 💣',x:c.x,y:c.y-20,l:1,c:'#ffcc00',size:14}); }
                for(let k=0;k<10;k++) particles.push(new Particle(c.x,c.y,(Math.random()-0.5)*8,(Math.random()-0.5)*8,'#ffd700',0.8));
                spaceTreasureCrates.splice(i,1);
                continue;
            }
            // Remove if off screen
            if(c.y>height+50){ spaceTreasureCrates.splice(i,1); }
        }
    } else if(spaceTreasureActive){ spaceTreasureActive=false; spaceTreasureCrates=[]; }

    // Asteroid Belt event logic
    if(asteroidBeltActive && Date.now()<asteroidBeltTimer){
        // Spawn asteroids periodically
        if(Math.random()<0.06 && asteroids.length<12){
            const sizes = ['small','small','small','medium','medium','large'];
            const sz = sizes[Math.floor(Math.random()*sizes.length)];
            asteroids.push(new Asteroid(-30, 40+Math.random()*(height-80), sz));
        }
        // Update asteroids
        for(let i=asteroids.length-1;i>=0;i--){
            let a=asteroids[i]; a.update(); a.draw();
            // Check bullet collision
            let destroyed=false;
            for(let j=0;j<bullets.length;j++){
                const b=bullets[j];
                if(Math.hypot(b.x-a.x,b.y-a.y)<a.r){
                    a.hp-=b.p;
                    if(!b.isLaser) bullets.splice(j,1);
                    if(a.hp<=0){
                        destroyed=true;
                        totalCoins+=a.coins; localStorage.setItem('totalCoins',totalCoins);
                        gemstones+=a.gems*gemMultiplier; saveGemstones();
                        floats.push({txt:'+'+a.coins+'c +'+a.gems+'💎',x:a.x,y:a.y-20,l:1,c:'#ffaa44',size:14});
                        for(let k=0;k<8;k++) particles.push(new Particle(a.x,a.y,(Math.random()-0.5)*8,(Math.random()-0.5)*8,'#aa6644',0.8));
                    }
                    break;
                }
            }
            if(destroyed){ asteroids.splice(i,1); }
            else if(a.x>width+60){ asteroids.splice(i,1); }
        }
    } else if(asteroidBeltActive){ asteroidBeltActive=false; asteroids=[]; }

    // Supply Drop event logic
    if(supplyDropActive && Date.now()<supplyDropTimer && supplyDropCrate && !supplyDropCrate.claimed){
        const c = supplyDropCrate;
        // Pulsing glow
        ctx.fillStyle=`rgba(0,255,170,${0.3+Math.sin(Date.now()/200)*0.15})`;
        ctx.shadowBlur=15; ctx.beginPath(); ctx.arc(c.x,c.y,30+Math.sin(Date.now()/150)*5,0,Math.PI*2); ctx.fill();
        ctx.shadowBlur=0;
        // Crate
        ctx.fillStyle='#44ffaa'; ctx.fillRect(c.x-18,c.y-18,36,36);
        ctx.strokeStyle='#00aa66'; ctx.lineWidth=2; ctx.strokeRect(c.x-18,c.y-18,36,36);
        ctx.fillStyle='#000'; ctx.font='bold 18px sans-serif'; ctx.textAlign='center'; ctx.fillText('📦',c.x,c.y+7); ctx.textAlign='start';
        // Timer bar
        const timeLeft = (supplyDropTimer-Date.now())/8000;
        ctx.fillStyle='#333'; ctx.fillRect(c.x-20,c.y+24,40,4);
        ctx.fillStyle='#44ffaa'; ctx.fillRect(c.x-20,c.y+24,40*timeLeft,4);
        // Collection check
        if(player && Math.hypot(c.x-player.x,c.y-player.y)<45){
            c.claimed = true;
            if(c.type==='coins'){ let amt=3000+Math.floor(Math.random()*7000); totalCoins+=amt; localStorage.setItem('totalCoins',totalCoins); floats.push({txt:'+'+formatNumber(amt)+' 💰',x:c.x,y:c.y-30,l:1.5,c:'#00ffaa',size:16}); }
            else if(c.type==='gems'){ let amt=100+Math.floor(Math.random()*400); gemstones+=amt*gemMultiplier; saveGemstones(); floats.push({txt:'+'+(amt*gemMultiplier)+' 💎',x:c.x,y:c.y-30,l:1.5,c:'#ff66ff',size:16}); }
            else { activePowerUps.supplydrop_buff={name:'Supply Boost',endTime:Date.now()+15000}; floats.push({txt:'⚡ SUPPLY BOOST!',x:c.x,y:c.y-30,l:1.5,c:'#ffaa00',size:16}); }
            for(let k=0;k<15;k++) particles.push(new Particle(c.x,c.y,(Math.random()-0.5)*10,(Math.random()-0.5)*10,'#44ffaa',0.8));
            showNotification('📦 Supply Drop collected!', 'success');
        }
    } else if(supplyDropActive){ supplyDropActive=false; supplyDropCrate=null; }

    // Green Aura effect (Doomsday bonus)
    if(greenAuraActive && player){
        ctx.strokeStyle=`rgba(0,255,0,${0.3+Math.sin(Date.now()/100)*0.15})`;
        ctx.lineWidth=2; ctx.beginPath(); ctx.arc(player.x,player.y,40+Math.sin(Date.now()/120)*5,0,Math.PI*2); ctx.stroke(); ctx.lineWidth=1;
        // Damage nearby enemies
        for(let i=0;i<enemies.length;i++){
            let e=enemies[i];
            if(Math.hypot(e.x-player.x,e.y-player.y)<50){
                e.hp -= 0.3;
                if(Math.random()<0.1) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*3,(Math.random()-0.5)*3,'#00ff00',0.5));
            }
        }
    }

    // Update music based on game state
    updateMusicBasedOnGameState();
    
    // Background rendering and game logic (same as original)
    if(synapseActive){
        synapseBackgroundHue = (synapseBackgroundHue + 2.5) % 360;
        ctx.fillStyle = `hsl(${synapseBackgroundHue}, 70%, 7%)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(chaosRealmActive){
        ctx.fillStyle = `hsl(${chaosHue}, 80%, 10%)`; ctx.fillRect(0,0,width,height); drawNebula();
        for(let i=0;i<50;i++){ ctx.fillStyle = `hsl(${chaosHue + i*7}, 100%, 50%)`; ctx.fillRect(Math.random()*width, Math.random()*height, 2, 2); }
    } else if(primordialRageActive){ ctx.fillStyle = `rgba(80,0,0,0.7)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(cosmicCollapseActive){ ctx.fillStyle = `rgba(0,20,60,0.6)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(blackHoleActive){ ctx.fillStyle = `rgba(20,0,40,0.8)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(voidActive){ ctx.fillStyle = `rgba(80,30,100,0.5)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(apocalypseActive){ ctx.fillStyle = `rgba(80,20,20,0.5)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(riftActive){ ctx.fillStyle = `rgba(100,30,120,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(goldRushActive){ ctx.fillStyle = `rgba(80,60,0,0.4)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(divineActive){ ctx.fillStyle = `rgba(255,200,0,0.25)`; ctx.fillRect(0,0,width,height); drawNebula();
        // Golden light rays
        ctx.save(); ctx.globalAlpha=0.15;
        for(let r=0;r<8;r++){ const a=r*Math.PI/4+Date.now()/3000; ctx.fillStyle='#ffdd00'; ctx.beginPath(); ctx.moveTo(width/2,height/2); ctx.lineTo(width/2+Math.cos(a)*width,height/2+Math.sin(a)*width); ctx.lineTo(width/2+Math.cos(a+0.15)*width,height/2+Math.sin(a+0.15)*width); ctx.fill(); }
        ctx.restore();
    } else if(bugEventActive){ ctx.fillStyle = `rgba(0,255,0,0.1)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(stableCycleActive){ ctx.fillStyle = `rgba(100,150,255,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(tidalWaveActive){ ctx.fillStyle = `rgba(0,100,150,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(masqueradeActive){ ctx.fillStyle = `rgba(150,100,0,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(soulHarvestActive){ ctx.fillStyle = `rgba(80,0,80,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(frozenTimeActive){ ctx.fillStyle = `rgba(100,150,200,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(crystalRainActive){ ctx.fillStyle = `rgba(100,200,150,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(shadowCloneActive){ ctx.fillStyle = `rgba(100,80,150,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(lightningStormActive){ ctx.fillStyle = `rgba(80,80,0,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(doomsDayActive){ ctx.fillStyle = `rgba(80,0,0,0.4)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(royalBlessingActive){ ctx.fillStyle = `rgba(255,200,0,0.15)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(starfallActive){ ctx.fillStyle = `rgba(100,100,50,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(infernoActive){ ctx.fillStyle = `rgba(80,30,0,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(chainLightningActive){ ctx.fillStyle = `rgba(80,80,0,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(barrierActive){ ctx.fillStyle = `rgba(68,85,170,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(soulReaperActive){ ctx.fillStyle = `rgba(68,0,85,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(vortexActive){ ctx.fillStyle = `rgba(34,85,127,0.25)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(kingsBlessingActive){ ctx.fillStyle = `rgba(127,100,0,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(prismActive){ ctx.fillStyle = `rgba(127,68,127,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(abyssActive){ ctx.fillStyle = `rgba(34,0,85,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(spaceTreasureActive){ ctx.fillStyle = `rgba(80,60,0,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(asteroidBeltActive){ ctx.fillStyle = `rgba(80,50,20,0.3)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(supplyDropActive){ ctx.fillStyle = `rgba(0,60,40,0.25)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else if(doppelgangerActive){ ctx.fillStyle = `rgba(127,68,127,0.2)`; ctx.fillRect(0,0,width,height); drawNebula();
    } else { ctx.clearRect(0,0,width,height); ctx.fillStyle='#000'; ctx.fillRect(0,0,width,height); drawNebula(); }

    // Divine flash overlay
    if(divineFlashTimer && Date.now()<divineFlashTimer){
        const fade = (divineFlashTimer-Date.now())/3000;
        ctx.fillStyle=`rgba(255,220,0,${fade*0.7})`; ctx.fillRect(0,0,width,height);
        // Golden light rays
        ctx.save(); ctx.globalAlpha=fade*0.4;
        for(let r=0;r<12;r++){ const a=r*Math.PI/6+Date.now()/2000; ctx.fillStyle='#ffdd00'; ctx.beginPath(); ctx.moveTo(width/2,height/2); ctx.lineTo(width/2+Math.cos(a)*width,height/2+Math.sin(a)*width); ctx.lineTo(width/2+Math.cos(a+0.1)*width,height/2+Math.sin(a+0.1)*width); ctx.fill(); }
        ctx.restore();
    } else { divineFlashTimer=0; }
    
    stars.forEach(s=>{
        if(gameState==='PLAYING')s.y+=((isOD||synapseActive||apocalypseActive||primordialRageActive||cosmicCollapseActive||chaosRealmActive||bugEventActive||stableCycleActive||lightningStormActive||royalBlessingActive||kingsBlessingActive||prismActive||doppelgangerActive)?s.v*5:s.v);
        if(s.y>height){s.y=0;s.x=Math.random()*width;}
        let starColor=isOD?'#ff00ff':synapseActive?`hsl(${synapseBackgroundHue},80%,55%)`:primordialRageActive?'#ff0000':chaosRealmActive?`hsl(${chaosHue},100%,60%)`:cosmicCollapseActive?'#88aaff':apocalypseActive?'#ff0000':goldRushActive?'#ffaa00':bugEventActive?'#00ff00':stableCycleActive?'#88aaff':frozenTimeActive?'#88ccff':crystalRainActive?'#88ffaa':lightningStormActive?'#ffff00':royalBlessingActive?'#ffdd00':kingsBlessingActive?'#ffaa00':prismActive?'#ff88ff':doppelgangerActive?'#ff88ff':spaceTreasureActive?'#ffd700':s.v>2.5?'#aaddff':'#ffffff';
        ctx.fillStyle=starColor; ctx.globalAlpha=s.v/3; ctx.fillRect(s.x,s.y,isOD?2:1.8,isOD?10:s.v>2.5?2:1.8); ctx.globalAlpha=1;
    });

    if(gameState==='PLAYING' && player && player.x && player.y){
        player.update(); player.draw(); if(settings.shake) shake*=0.85;
        if(waveBannerTimer>0){waveBannerTimer--; document.getElementById('wave-banner').style.display='block';}
        else document.getElementById('wave-banner').style.display='none';
        if(bossWarningTimer>0){bossWarningTimer--; document.getElementById('boss-warning').style.display='block';}
        else document.getElementById('boss-warning').style.display='none';

        if(waveKills>=waveKillGoal && !waveTriggered && !boss && !guardian){
            waveTriggered=true;
            if(waveNoDamage && wave>1) perfectWavesCount++;
            setTimeout(()=>{if(gameState==='PLAYING') startWave(wave+1);},1800);
        }

        if(!endlessMode && wave>1 && wave%5===0 && !boss && !guardian && !enemies.some(e=>e.isBoss) && waveKills<3){
            if(bossWarningTimer===0 && !waveTriggered){
                bossWarningTimer=100;
                setTimeout(()=>{ if(gameState==='PLAYING' && !boss && !guardian){ boss=new Enemy(true); enemies.push(boss); floats.push({txt:'⚠ BOSS!',x:width/2-35,y:height/2,l:1.5,c:'#ff0044',size:32}); } },2500);
            }
        }

        let spawnChance = 0.012 + (wave*0.002) + (level*0.0012);
        if(apocalypseActive) spawnChance*=2; if(cosmicCollapseActive) spawnChance*=3; if(riftActive) spawnChance*=1.5;
        if(tidalWaveActive) spawnChance*=2;
        if(doomsDayActive) spawnChance*=3;
        if(Math.random()<spawnChance && enemies.length<20+wave) enemies.push(new Enemy());

        const isRapid=!!activePowerUps.rapidfire;
        let fr=isOD||isRapid?40:Math.max(65,240-fireLevel*38 - weaponEnhancementLevel*2);
        fr /= (1 + getSkillBonus('fireRate'));
        if(synapseActive) fr*=0.55; if(primordialRageActive) fr*=0.3; if(chaosRealmActive) fr*=0.2;
        if(apocalypseActive) fr*=0.7; if(cosmicCollapseActive) fr*=0.5; if(riftActive) fr*=0.9;
        if(bugEventActive) fr*=0.3; if(stableCycleActive) fr*=0.7;
        if(royalBlessingActive) fr*=0.5; if(kingsBlessingActive) fr*=0.3;
        if(prismActive) fr*=0.6;
        if(doppelgangerActive) fr*=0.8;
        fr /= skinFireRateMultiplier;
        // Auto-fire: if enabled, fire automatically; otherwise only fire on click/tap
        if(settings.autoFire && Date.now()-lastFire>fr){ if(player){fireBullets(); let maxDrones=Math.min(180, droneCount); for(let i=0;i<maxDrones;i++){ const a=(Date.now()/400)+(i*Math.PI*2/maxDrones); if(Math.random()<0.3) bullets.push(new Bullet(player.x+Math.cos(a)*55,player.y+Math.sin(a)*45,Math.ceil(damageLevel*0.55*(1+weaponEnhancementLevel*0.05)*(1+getSkillBonus('droneDmg'))),'#00ffaa')); }} }

        checkEventTrigger();

        if(activeEvent === 'reaper'){
            for(let i=0;i<enemies.length;i++){
                let e=enemies[i];
                let pointBonus = e.isBoss?90000:3500*combo;
                if(goldRushActive) pointBonus*=10;
                if(stableCycleActive) pointBonus*=3;
                if(royalBlessingActive) pointBonus*=3;
                if(kingsBlessingActive) pointBonus*=5;
                pointBonus *= skinCreditMultiplier;
                score+=pointBonus; kills++; waveKills++;
                if(e.isBoss) bossesKilled++;
                for(let k=0;k<15;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*12,(Math.random()-0.5)*12,'#880044',0.9));
                enemies.splice(i,1); i--;
            }
            boss=null;
            activeEvent=null;
            document.getElementById('event-banner').style.display='none';
        }

        if(blackHoleActive){
            ctx.fillStyle='rgba(0,0,0,0.3)'; ctx.beginPath(); ctx.arc(blackHoleCenter.x,blackHoleCenter.y,80,0,Math.PI*2); ctx.fill();
            ctx.fillStyle='#4400aa'; ctx.beginPath(); ctx.arc(blackHoleCenter.x,blackHoleCenter.y,40,0,Math.PI*2); ctx.fill();
            for(let i=0;i<enemies.length;i++){
                let dx = blackHoleCenter.x - enemies[i].x, dy = blackHoleCenter.y - enemies[i].y, dist = Math.hypot(dx,dy);
                if(dist<150){ let pull = (150-dist)/150 * 8; enemies[i].x += dx/dist * pull; enemies[i].y += dy/dist * pull;
                    if(dist<30){ 
                        let pointBonus = 70*combo;
                        if(goldRushActive) pointBonus*=10;
                        if(stableCycleActive) pointBonus*=3;
                        if(royalBlessingActive) pointBonus*=3;
                        if(kingsBlessingActive) pointBonus*=5;
                        pointBonus *= skinCreditMultiplier;
                        score+=pointBonus; kills++; waveKills++; xp+=10; if(xp>=100){xp=0;level++;sfxLevelUp();}
                        particles.push(new Particle(enemies[i].x,enemies[i].y,0,0,'#aa66ff',0.8)); enemies.splice(i,1); i--; 
                    }
                }
            }
        }

        for(let i=0;i<meteors.length;i++){ meteors[i].update(); meteors[i].draw();
            if(player && !voidActive && !primordialRageActive && !chaosRealmActive && !divineActive && !bugEventActive && !stableCycleActive && !frozenTimeActive && !royalBlessingActive && !barrierActive && !timeSlowActive && Math.hypot(meteors[i].x-player.x,meteors[i].y-player.y)<meteors[i].radius+player.r){
                let dmg = 15 * damageReduction;
                health-=dmg; if(settings.shake) shake=15; player.invincibleTimer=40; sfxHit(); waveNoDamage=false; meteors.splice(i,1); i--; if(health<=0){gameOver();return;}
                if(barrierActive){
                    barrierHp -= dmg;
                    if(barrierHp<=0) barrierActive=false;
                    health += dmg;
                    if(health>maxHealth) health = maxHealth;
                }
            }
            if(meteors[i].y>height+50){ meteors.splice(i,1); i--; }
        }

        if(guardian){ guardian.update(); guardian.draw();
            if(player && !voidActive && !primordialRageActive && !chaosRealmActive && !divineActive && !bugEventActive && !stableCycleActive && !frozenTimeActive && !royalBlessingActive && !barrierActive && !timeSlowActive && Math.hypot(guardian.x-player.x,guardian.y-player.y)<guardian.r+player.r){
                let dmg = 40 * damageReduction;
                health-=dmg; if(settings.shake) shake=20; player.invincibleTimer=60; sfxHit(); waveNoDamage=false; if(health<=0){gameOver();return;}
                if(barrierActive){
                    barrierHp -= dmg;
                    if(barrierHp<=0) barrierActive=false;
                    health += dmg;
                    if(health>maxHealth) health = maxHealth;
                }
            }
            if(guardian && guardian.y>height+200){ guardian=null; activeEvent=null; }
        }

        const magnetActive=!!activePowerUps.magnet;
        const nextBullets=[], nextEBullets=[], nextItems=[], nextParticles=[], nextFloats=[];

        const preCollide=[];
        for(const e of enemies){ e.update(); e.draw(); let dead=false;
            if(player && !voidActive && !primordialRageActive && !chaosRealmActive && !divineActive && !bugEventActive && !stableCycleActive && !frozenTimeActive && !royalBlessingActive && !barrierActive && !timeSlowActive && Math.hypot(e.x-player.x,e.y-player.y)<e.r+18 && player.invincibleTimer===0){
                if(!isOD && !synapseActive){
                    let dmg = e.isBoss?15:8; if(apocalypseActive) dmg*=1.5; if(cosmicCollapseActive) dmg*=1.3; if(doomsDayActive) dmg*=0.7;
                    dmg *= damageReduction;
                    health-=hasShieldUpgrade?Math.floor(dmg*0.5):dmg; if(settings.shake) shake=15; player.invincibleTimer=45; sfxHit(); waveNoDamage=false;
                    if(health<=0){gameOver();return;}
                    if(barrierActive){
                        barrierHp -= dmg;
                        if(barrierHp<=0) barrierActive=false;
                        health += dmg;
                        if(health>maxHealth) health = maxHealth;
                    }
                }
                if(!e.isBoss) dead=true;
            }
            if(e.y>height+180){ if(e.isBoss) boss=null; combo=1; dead=true; }
            if(!dead) preCollide.push(e); else if(e.isBoss) boss=null;
        }

        const hitBullets=new Set(), finalEnemies=[];
        for(const e of preCollide){ let destroyed=false;
            for(let j=0;j<bullets.length;j++){
                if(hitBullets.has(j)) continue;
                const b=bullets[j];
                if(Math.hypot(b.x-e.x,b.y-e.y)<e.r){
                    if(!b.isLaser) hitBullets.add(j);
                    e.hp-=b.p;
                    // Floating damage number
                    const isCrit = b.p > damageLevel * 1.5;
                    floats.push({txt:'-'+Math.ceil(b.p), x:e.x+(Math.random()-0.5)*20, y:e.y-e.r-5, l:0.8, c:isCrit?'#ffaa00':'#ffffff', size:isCrit?14:10});
                    if(e.hp<=0){
                        destroyed=true; sfxExplode();
                        const cnt=e.isBoss?25:12;
                        for(let k=0;k<cnt;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*12,(Math.random()-0.5)*12,e.color,0.8));
                        let pointBonus = e.isBoss?1800:70*combo;
                        if(apocalypseActive) pointBonus*=5; if(cosmicCollapseActive) pointBonus*=10; if(riftActive) pointBonus*=riftMultiplier;
                        if(goldRushActive) pointBonus*=10;
                        if(bugEventActive) pointBonus*=5;
                        if(stableCycleActive) pointBonus*=3;
                        if(royalBlessingActive) pointBonus*=3;
                        if(kingsBlessingActive) pointBonus*=5;
                        if(doomsDayActive) pointBonus*=5;
                        pointBonus *= skinCreditMultiplier;
                        score+=pointBonus; kills++; waveKills++;
                        if(synapseActive) synapseKills++;
                        xp+=e.isBoss?35:10;
                        if(xp>=100){ xp=0; level++; sfxLevelUp(); floats.push({txt:'▲ RANK UP!',x:player.x-40,y:player.y-25,l:1.5,c:'#00ffaa',size:20}); awardSkillPoints(); }
                        odCharge=Math.min(100,odCharge+(e.isBoss?45:3));
                        if(e.isBoss){ bossesKilled++; boss=null;
                            // Elite rare drop for Gravity Bomb unlock
                            if(Math.random() < 0.3){
                                let eliteDrops = parseInt(localStorage.getItem('eliteDropsCollected')||0)+1;
                                localStorage.setItem('eliteDropsCollected', eliteDrops);
                                if(player && player.x && player.y) floats.push({txt:'⭐ ELITE DROP! ('+eliteDrops+'/3)',x:e.x-50,y:e.y-30,l:1.5,c:'gold',size:16});
                                showNotification('⭐ Elite drop collected! ('+eliteDrops+'/3)', 'success');
                            }
                        }
                        const roll=Math.random();
                        if(roll>0.65) items.push({x:e.x,y:e.y,vy:2,isPowerUp:false});
                        if(roll>0.88) spawnPowerUp(e.x+20,e.y);
                        if(Math.random()<0.08) spawnPsychedelia(e.x,e.y);
                        if(Math.random()<0.05 && blackHolePieces<6) spawnBlackHolePiece(e.x,e.y);
                        
                        if(soulHarvestActive && !e.isBoss){
                            soulHarvestSouls.push(new Soul(e.x, e.y));
                        }
                        
                        combo++; if(combo>maxCombo) maxCombo=combo; comboTimer=Date.now()+2200;
                        updateComboMeter();
                        checkAchievements(); break;
                    }
                }
            }
            if(!destroyed) finalEnemies.push(e);
        }

        for(let j=0;j<bullets.length;j++){ if(hitBullets.has(j)) continue; const b=bullets[j];
            for(let m=0;m<meteors.length;m++){ if(Math.hypot(b.x-meteors[m].x,b.y-meteors[m].y)<meteors[m].radius){
                hitBullets.add(j); meteors[m].hp-=b.p;
                if(meteors[m].hp<=0){ 
                    let pointBonus = 200;
                    if(goldRushActive) pointBonus*=10;
                    if(bugEventActive) pointBonus*=5;
                    if(stableCycleActive) pointBonus*=3;
                    if(royalBlessingActive) pointBonus*=3;
                    if(kingsBlessingActive) pointBonus*=5;
                    pointBonus *= skinCreditMultiplier;
                    score+=pointBonus; totalMeteorsDestroyed++;
                    for(let k=0;k<8;k++) particles.push(new Particle(meteors[m].x,meteors[m].y,(Math.random()-0.5)*8,(Math.random()-0.5)*8,'#ff8844',0.8));
                    meteors.splice(m,1); m--;
                } break;
            }}
        }
        
        if(guardian){ for(let j=0;j<bullets.length;j++){ if(hitBullets.has(j)) continue; const b=bullets[j];
            if(Math.hypot(b.x-guardian.x,b.y-guardian.y)<guardian.r){
                hitBullets.add(j); guardian.hp-=b.p;
                if(guardian.hp<=0){ let gx=guardian.x,gy=guardian.y; guardian=null; activeEvent=null; 
                    let pointBonus = 50000;
                    if(goldRushActive) pointBonus*=10;
                    if(bugEventActive) pointBonus*=5;
                    if(stableCycleActive) pointBonus*=3;
                    if(royalBlessingActive) pointBonus*=3;
                    if(kingsBlessingActive) pointBonus*=5;
                    pointBonus *= skinCreditMultiplier;
                    score+=pointBonus; totalCoins+=5000;
                    if(!skinUnlocked){ skinUnlocked=true; localStorage.setItem('skinUnlocked','true'); showAchievementPopup('👑 GUARDIAN DEFEATED', 'Golden skin unlocked!'); }
                    let count=parseInt(localStorage.getItem('guardianDefeatedCount')||0)+1; localStorage.setItem('guardianDefeatedCount',count);
                    guardianDefeated=true; for(let k=0;k<50;k++) particles.push(new Particle(gx,gy,(Math.random()-0.5)*20,(Math.random()-0.5)*20,'gold',1));
                    checkAchievements();
                } break;
            }}
        }

        for(let j=0;j<bullets.length;j++){ if(!hitBullets.has(j)){ bullets[j].update(); bullets[j].draw(); if(bullets[j].y>-45 && bullets[j].y<height+45 && bullets[j].x>-45 && bullets[j].x<width+45) nextBullets.push(bullets[j]); } }
        bullets.length=0; bullets.push(...nextBullets);
        enemies.length=0; enemies.push(...finalEnemies);

        for(const eb of eBullets){ eb.x+=eb.vx||0; eb.y+=eb.vy;
            ctx.fillStyle='#ff0044'; ctx.shadowBlur=7; ctx.beginPath(); ctx.arc(eb.x,eb.y,5,0,Math.PI*2); ctx.fill();
            let keep=true;
            if(!voidActive && !primordialRageActive && !chaosRealmActive && !divineActive && !bugEventActive && !stableCycleActive && !frozenTimeActive && !royalBlessingActive && !barrierActive && !timeSlowActive && Math.hypot(eb.x-player.x,eb.y-player.y)<22 && player.invincibleTimer===0){
                if(!isOD && !synapseActive){ 
                    let dmg = (hasShieldUpgrade?4:9) * damageReduction;
                    health-=dmg; if(settings.shake) shake=10; player.invincibleTimer=22; sfxHit(); waveNoDamage=false; 
                    if(health<=0){gameOver();return;}
                    if(barrierActive){
                        barrierHp -= dmg;
                        if(barrierHp<=0) barrierActive=false;
                        health += dmg;
                        if(health>maxHealth) health = maxHealth;
                    }
                }
                keep=false;
            }
            if(eb.y>height || eb.x<-20 || eb.x>width+20) keep=false;
            if(keep) nextEBullets.push(eb);
        }
        eBullets.length=0; eBullets.push(...nextEBullets);

        for(const it of items){ it.y+=it.vy||2;
            const d=Math.hypot(it.x-player.x,it.y-player.y); const pull=magnetActive?280:(100+resourceCollectionLevel*15+getSkillBonus('magnetRange'));
            if(d<pull){ it.x+=(player.x-it.x)*0.16; it.y+=(player.y-it.y)*0.16; }
            let keep=true;
            if(d<30){
                if(it.isPowerUp){ applyPowerUp(it.puType); }
                else if(it.isPsychedelia){ synapsePsychedeliaCount++; psychedeliaCollected++; sfxPowerup(); floats.push({txt:'🧠 +1',x:it.x,y:it.y,l:0.8,c:'#ff66ff',size:14});
                    if(synapsePsychedeliaCount>=8 && !synapseActive){ synapsePsychedeliaCount=0; activateSynapse(); }
                } else if(it.isBlackHolePiece){ blackHolePieces++; sfxPowerup(); floats.push({txt:'⭐ +1',x:it.x,y:it.y,l:0.8,c:'gold',size:14});
                    if(blackHolePieces>=6 && !blackHoleActive && !activeEvent){ blackHolePieces=0; triggerBlackHole(); }
                } else { let bonus=4+Math.floor(level*0.4+wave*0.3); if(apocalypseActive) bonus*=3; if(cosmicCollapseActive) bonus*=5; if(riftActive) bonus*=riftMultiplier;
                    if(goldRushActive) bonus*=10;
                    if(bugEventActive) bonus*=5;
                    if(stableCycleActive) bonus*=3;
                    if(royalBlessingActive) bonus*=3;
                    if(kingsBlessingActive) bonus*=5;
                    bonus *= skinCreditMultiplier;
                    totalCoins+=bonus; sfxCoin(); floats.push({txt:'+'+bonus+'c',x:it.x,y:it.y,l:0.8,c:'gold',size:12}); }
                keep=false;
            }
            if(it.y>height+45) keep=false;
            if(keep){
                if(it.isPowerUp){ ctx.fillStyle=it.puType.color; ctx.shadowBlur=10; ctx.font='bold 16px Segoe UI'; ctx.fillText(it.puType.icon,it.x-8,it.y+5);
                    ctx.beginPath(); ctx.arc(it.x,it.y+2,10+Math.sin(Date.now()/150)*2,0,Math.PI*2); ctx.stroke();
                } else if(it.isPsychedelia){ ctx.fillStyle='#ff66ff'; ctx.shadowBlur=10; ctx.font='bold 18px Segoe UI'; ctx.fillText('🧠',it.x-9,it.y+7);
                    ctx.beginPath(); ctx.arc(it.x,it.y+2,9+Math.sin(Date.now()/180)*2,0,Math.PI*2); ctx.strokeStyle='#ff66ff'; ctx.stroke();
                } else if(it.isBlackHolePiece){ ctx.fillStyle='gold'; ctx.shadowBlur=10; ctx.font='bold 18px Segoe UI'; ctx.fillText('⭐',it.x-9,it.y+7);
                    ctx.beginPath(); ctx.arc(it.x,it.y+2,9+Math.sin(Date.now()/180)*2,0,Math.PI*2); ctx.strokeStyle='gold'; ctx.stroke();
                } else { ctx.fillStyle='gold'; ctx.shadowBlur=8; ctx.beginPath(); ctx.arc(it.x,it.y,5,0,Math.PI*2); ctx.fill();
                    ctx.beginPath(); ctx.arc(it.x,it.y,5+Math.sin(Date.now()/160)*2,0,Math.PI*2); ctx.stroke(); }
                nextItems.push(it);
            }
        }
        items.length=0; items.push(...nextItems);

        for(const p of particles){ p.update(); p.draw(); if(p.l>0) nextParticles.push(p); }
        particles.length=0; particles.push(...nextParticles);

        for(const f of floats){ f.y-=1.6; f.l-=0.014; ctx.font=`bold ${f.size||16}px Segoe UI`; ctx.globalAlpha=Math.max(0,Math.min(1,f.l)); ctx.fillStyle=f.c; ctx.shadowBlur=6; ctx.fillText(f.txt,f.x,f.y); if(f.l>0) nextFloats.push(f); }
        ctx.globalAlpha=1; ctx.shadowBlur=0; floats.length=0; floats.push(...nextFloats);

        tickPowerUps();

        // === RARE ABILITY EFFECTS ===
        // Regeneration Aura
        if(RARE_ABILITIES.regenAura.purchased){
            regenAuraTimer++;
            if(regenAuraTimer >= 300){ // ~5 seconds at 60fps
                regenAuraTimer = 0;
                health = Math.min(maxHealth, health + 1);
                if(player && player.x && player.y) particles.push(new Particle(player.x, player.y-20, 0, -2, '#00ffaa', 0.6));
            }
        }
        // Shockwave visual
        if(shockwaveActive && Date.now() < shockwaveTimer){
            shockwaveRadius += 15;
            ctx.strokeStyle = `rgba(255,136,0,${1 - shockwaveRadius/300})`;
            ctx.lineWidth = 4;
            ctx.beginPath();
            ctx.arc(player.x, player.y, shockwaveRadius, 0, Math.PI*2);
            ctx.stroke();
            ctx.lineWidth = 1;
        } else { shockwaveActive = false; shockwaveRadius = 0; }
        // Time Slow/Phase visual
        if(timeSlowActive && Date.now() < timeSlowTimer){
            ctx.strokeStyle = `rgba(170,102,255,${0.3 + Math.sin(Date.now()/100)*0.2})`;
            ctx.lineWidth = 2;
            ctx.beginPath(); ctx.arc(player.x, player.y, 40+Math.sin(Date.now()/150)*5, 0, Math.PI*2); ctx.stroke();
            ctx.lineWidth = 1;
        } else { timeSlowActive = false; }
        // Gravity Bomb
        if(gravityBombActive && Date.now() < gravityBombTimer){
            const elapsed = gravityBombTimer - Date.now();
            if(elapsed > 1500){
                // Pull phase
                gravityBombPhase = 0;
                for(let i=0;i<enemies.length;i++){
                    let e=enemies[i];
                    let dx=gravityBombCenter.x-e.x, dy=gravityBombCenter.y-e.y;
                    let dist=Math.hypot(dx,dy);
                    if(dist>20){ let pull=8; e.x+=dx/dist*pull; e.y+=dy/dist*pull; }
                }
                // Visual pull
                ctx.strokeStyle='rgba(68,0,170,0.6)'; ctx.lineWidth=2;
                for(let i=0;i<enemies.length;i++){
                    let e=enemies[i];
                    ctx.beginPath(); ctx.moveTo(e.x,e.y); ctx.lineTo(gravityBombCenter.x,gravityBombCenter.y); ctx.stroke();
                }
                ctx.fillStyle='rgba(68,0,170,0.4)'; ctx.beginPath(); ctx.arc(gravityBombCenter.x,gravityBombCenter.y,30,0,Math.PI*2); ctx.fill();
            } else if(gravityBombPhase === 0){
                // Explode!
                gravityBombPhase = 1;
                for(let i=enemies.length-1;i>=0;i--){
                    let e=enemies[i];
                    let dist=Math.hypot(e.x-gravityBombCenter.x, e.y-gravityBombCenter.y);
                    if(dist < 300){
                        let pointBonus = e.isBoss?50000:8000*combo;
                        pointBonus *= skinCreditMultiplier;
                        score += pointBonus; kills++; waveKills++;
                        if(e.isBoss) bossesKilled++;
                        for(let k=0;k<20;k++) particles.push(new Particle(e.x,e.y,(Math.random()-0.5)*15,(Math.random()-0.5)*15,'#4400aa',1));
                        enemies.splice(i,1);
                    }
                }
                if(boss){
                    let dist=Math.hypot(boss.x-gravityBombCenter.x,boss.y-gravityBombCenter.y);
                    if(dist<300){ boss.hp-=1000; if(boss.hp<=0){boss=null;bossesKilled++;} }
                }
                if(player && player.x && player.y) floats.push({txt:'🌀 GRAVITY BOMB BOOM!',x:gravityBombCenter.x-70,y:gravityBombCenter.y-40,l:1.5,c:'#4400aa',size:24});
                for(let i=0;i<50;i++){const a=i*(Math.PI*2/50);particles.push(new Particle(gravityBombCenter.x,gravityBombCenter.y,Math.cos(a)*10,Math.sin(a)*10,'#aa44ff',1));}
            }
            // Explosion visual
            if(gravityBombPhase === 1){
                ctx.fillStyle='rgba(170,68,255,0.3)'; ctx.beginPath(); ctx.arc(gravityBombCenter.x,gravityBombCenter.y,300,0,Math.PI*2); ctx.fill();
            }
        } else { gravityBombActive = false; gravityBombPhase = 0; }
        // Lightning Strike visual
        if(lightningStrikeEffect && Date.now() < lightningStrikeEffect.timer){
            let lx=lightningStrikeEffect.x, ly=lightningStrikeEffect.y;
            ctx.strokeStyle='#ffff00'; ctx.lineWidth=3; ctx.shadowBlur=20; ctx.shadowColor='#ffff00';
            for(let i=0;i<5;i++){
                ctx.beginPath(); ctx.moveTo(lx,0);
                let cx=lx, cy=0;
                for(let j=0;j<8;j++){ cx+=(Math.random()-0.5)*40; cy+=ly/8; ctx.lineTo(cx,cy); }
                ctx.lineTo(lx,ly); ctx.stroke();
            }
            ctx.lineWidth=1; ctx.shadowBlur=0;
        } else { lightningStrikeEffect = null; }
        // Resource Magnet effect - attract all items on screen
        if(resourceMagnetActive && Date.now() < resourceMagnetTimer){
            for(const it of items){
                if(player){
                    const dx = player.x - it.x, dy = player.y - it.y;
                    const dist = Math.hypot(dx, dy);
                    if(dist > 5){ it.x += dx/dist * 12; it.y += dy/dist * 12; }
                }
            }
            if(player){
                ctx.strokeStyle = `rgba(255,204,0,${0.4 + Math.sin(Date.now()/100)*0.2})`;
                ctx.lineWidth = 2;
                ctx.beginPath(); ctx.arc(player.x, player.y, 150+Math.sin(Date.now()/150)*20, 0, Math.PI*2); ctx.stroke();
                ctx.lineWidth = 1;
            }
        } else { resourceMagnetActive = false; }
        // Time Warp ability effect - slow enemies
        if(abilityTimeWarpActive && Date.now() < abilityTimeWarpTimer){
            if(player){
                ctx.strokeStyle = `rgba(0,204,255,${0.3 + Math.sin(Date.now()/100)*0.2})`;
                ctx.lineWidth = 2;
                ctx.beginPath(); ctx.arc(player.x, player.y, 80+Math.sin(Date.now()/150)*10, 0, Math.PI*2); ctx.stroke();
                ctx.lineWidth = 1;
            }
        } else { abilityTimeWarpActive = false; }
        // Ultimate Annihilation invincibility effect
        if(ultimateAnnihilationActive && Date.now() < ultimateAnnihilationTimer){
            if(player){
                ctx.strokeStyle = `rgba(255,0,0,${0.5 + Math.sin(Date.now()/80)*0.3})`;
                ctx.lineWidth = 3;
                ctx.beginPath(); ctx.arc(player.x, player.y, 45+Math.sin(Date.now()/100)*8, 0, Math.PI*2); ctx.stroke();
                ctx.strokeStyle = `rgba(255,100,0,${0.3 + Math.sin(Date.now()/120)*0.2})`;
                ctx.beginPath(); ctx.arc(player.x, player.y, 55+Math.sin(Date.now()/80)*10, 0, Math.PI*2); ctx.stroke();
                ctx.lineWidth = 1;
            }
        } else { ultimateAnnihilationActive = false; }
        // Update ability buttons
        updateAbilityButtons();

        if(Date.now()>comboTimer && gameState==='PLAYING') combo=1;
        if(isOD && Date.now()>odTimer){ isOD=false; odCharge=0; document.getElementById('od-btn').style.display='none'; }
        if(odCharge>=100 && !isOD) document.getElementById('od-btn').style.display='flex';
        if(isOD){ const rem=(odTimer-Date.now())/6000; document.getElementById('od-btn').style.background=`conic-gradient(#ff00ff ${rem*360}deg,rgba(142,68,173,0.3) 0deg)`; }

        document.getElementById('hp-fill').style.width=Math.max(0,health)+'%';
        document.getElementById('hp-num').innerText=Math.max(0,Math.ceil(health))+'%';
        document.getElementById('xp-fill').style.width=xp+'%';
        document.getElementById('od-fill').style.width=odCharge+'%';
        document.getElementById('score-val').innerText=formatNumber(score);
        document.getElementById('lvl-val').innerText=level;
        document.getElementById('kill-val').innerText=formatNumber(kills);
        document.getElementById('wave-val').innerText=wave;
        document.getElementById('bomb-count').innerText=bombCount;
        document.getElementById('combo-small').innerText=combo>1?'x'+combo:'';
        const hpPct=health/maxHealth;
        document.getElementById('hp-fill').style.background=hpPct>0.5?'linear-gradient(90deg,#ff0044,#ff5588)':hpPct>0.25?'linear-gradient(90deg,#ff6600,#ffaa00)':'linear-gradient(90deg,#ff2200,#ff5500)';
        document.getElementById('vignette').style.display=hpPct<0.3?'block':'none';
        
        updateDailyMissionProgress('kill', waveKills);
        updateDailyMissionProgress('gem', 0);
        if(waveNoDamage) updateDailyMissionProgress('perfect', waveNoDamage ? 1 : 0);
        updateDailyMissionProgress('crit', criticalHitsCount);

        // === NEW UI/QOL UPDATES ===
        // Wave Progress Bar
        const wpEl = document.getElementById('wave-progress');
        const wpFill = document.getElementById('wave-progress-fill');
        const wpText = document.getElementById('wave-progress-text');
        if(wpEl && wpFill && wpText){
            const wpPct = Math.min(100, (waveKills / waveKillGoal) * 100);
            wpFill.style.width = wpPct + '%';
            wpText.textContent = waveKills + '/' + waveKillGoal + ' KILLS';
        }

        // Enemy Health Bars (draw on canvas)
        for(const e of enemies){ drawEnemyHPBar(e); }

        // Boss HP Bar at top
        drawBossHPBarTop();

        // FPS Counter
        updateFPSCounter();
    }
    drawCrosshair(mouseX,mouseY);
}

// INPUT
window.addEventListener('resize',()=>{ width=canvas.width=window.innerWidth; height=canvas.height=window.innerHeight; nebulaCanvas.width=width; nebulaCanvas.height=height; resizeCross(); drawNebula(); });
window.dispatchEvent(new Event('resize'));
canvas.addEventListener('mousemove',e=>{ mouseX=e.clientX; mouseY=e.clientY; if(gameState==='PLAYING' && player){ player.tx=e.clientX; player.ty=e.clientY-50; } });
canvas.addEventListener('touchmove',e=>{ e.preventDefault(); if(gameState==='PLAYING' && player){ mouseX=e.touches[0].clientX; mouseY=e.touches[0].clientY; player.tx=e.touches[0].clientX; player.ty=e.touches[0].clientY-65; } },{passive:false});
canvas.addEventListener('touchstart',e=>{ e.preventDefault(); initAudio(); if(gameState==='PLAYING' && !settings.autoFire){ fireBullets(); } },{passive:false});
canvas.addEventListener('click',()=>{ initAudio(); if(gameState==='PLAYING' && !settings.autoFire){ fireBullets(); } });
window.addEventListener('keydown',e=>{
    if(e.key==='Escape' && (gameState==='PLAYING'||gameState==='PAUSED')) togglePause();
    if((e.key==='q'||e.key==='Q') && gameState==='PLAYING') useBomb();
    if((e.key==='o'||e.key==='O') && gameState==='PLAYING') activateOverdrive();
    if(e.key==='1' && gameState==='PLAYING') activateRareAbility('shockwave');
    if(e.key==='2' && gameState==='PLAYING') activateRareAbility('regenAura');
    if(e.key==='3' && gameState==='PLAYING') activateRareAbility('timeSlow');
    if(e.key==='4' && gameState==='PLAYING') activateRareAbility('gravityBomb');
    if(e.key==='5' && gameState==='PLAYING') activateRareAbility('lightningStrike');
    if(e.key==='6' && gameState==='PLAYING') activateRareAbility('resourceMagnet');
    if(e.key==='7' && gameState==='PLAYING') activateRareAbility('timeWarp');
    if(e.key==='8' && gameState==='PLAYING') activateRareAbility('ultimateAnnihilation');
});

// INIT

// DAILY STREAK SYSTEM
let dailyStreak = parseInt(localStorage.getItem('dailyStreak')) || 0;
let lastPlayDate = localStorage.getItem('lastPlayDate') || '';
let streakRewardClaimed = localStorage.getItem('streakRewardClaimed') === 'true';

function checkDailyStreak(){
    const today = new Date().toDateString();
    const yesterday = new Date(Date.now() - 86400000).toDateString();

    if(lastPlayDate === today) return; // Already played today

    if(lastPlayDate === yesterday){
        // Streak continues
        dailyStreak++;
    } else if(lastPlayDate !== ''){
        // Streak broken
        dailyStreak = 1;
    } else {
        // First time ever
        dailyStreak = 1;
    }

    lastPlayDate = today;
    streakRewardClaimed = false;
    localStorage.setItem('dailyStreak', dailyStreak);
    localStorage.setItem('lastPlayDate', lastPlayDate);
    localStorage.setItem('streakRewardClaimed', 'false');

    updateStreakUI();
    showStreakPopup();
}

function getStreakReward(streak){
    if(streak >= 10) return { credits: 1500, gems: 300, lootbox: 'rare', label: '10+ Day Streak!' };
    if(streak >= 7) return { credits: 1000, gems: 200, lootbox: 'common', label: '7 Day Streak!' };
    if(streak >= 5) return { credits: 500, gems: 100, lootbox: null, label: '5 Day Streak!' };
    if(streak >= 3) return { credits: 200, gems: 50, lootbox: null, label: '3 Day Streak!' };
    return { credits: 100, gems: 0, lootbox: null, label: 'Day 1' };
}

function showStreakPopup(){
    if(streakRewardClaimed) return;
    const reward = getStreakReward(dailyStreak);
    document.getElementById('streak-popup-count').textContent = dailyStreak;
    document.getElementById('streak-popup-subtitle').textContent = dailyStreak === 1 ? t('streak_title').toLowerCase() + '!' : 'days in a row!';

    let rewardsHtml = '';
    if(reward.credits > 0) rewardsHtml += `<div class="streak-reward-item">💰 +${formatNumber(reward.credits)} Credits</div>`;
    if(reward.gems > 0) rewardsHtml += `<div class="streak-reward-item">💎 +${reward.gems} Gemstones</div>`;
    if(reward.lootbox) rewardsHtml += `<div class="streak-reward-item">📦 +1 ${reward.lootbox.toUpperCase()} Lootbox</div>`;
    document.getElementById('streak-popup-rewards').innerHTML = rewardsHtml;
    document.getElementById('streak-popup').style.display = 'flex';
}

function claimStreakReward(){
    if(streakRewardClaimed) return;
    const reward = getStreakReward(dailyStreak);

    totalCoins += reward.credits;
    localStorage.setItem('totalCoins', totalCoins);
    if(reward.gems > 0){ gemstones += reward.gems; saveGemstones(); }
    if(reward.lootbox){ openLootbox(reward.lootbox); }

    streakRewardClaimed = true;
    localStorage.setItem('streakRewardClaimed', 'true');
    document.getElementById('streak-popup').style.display = 'none';

    if(reward.credits > 0) showNotification(`🔥 Streak Day ${dailyStreak}: +${formatNumber(reward.credits)} credits!`, 'success');
    if(reward.gems > 0) showNotification(`🔥 Streak Day ${dailyStreak}: +${reward.gems} gemstones!`, 'success');

    if(typeof updateHubUI === 'function') updateHubUI();
    updateStreakUI();
    if(typeof checkAchievements === 'function') checkAchievements();
}

function updateStreakUI(){
    const container = document.getElementById('streak-badge-container');
    if(!container) return;
    if(dailyStreak > 0){
        container.innerHTML = `<div class="streak-badge">🔥 ${dailyStreak} <span id="streak-day-text">${t('streak_day')}</span></div>`;
    } else {
        container.innerHTML = '';
    }
}

for(let i=0;i<90;i++) stars.push({x:Math.random()*(window.innerWidth||800), y:Math.random()*(window.innerHeight||600), v:Math.random()*2.2+0.8});
runLoader('loader-init','init-fill',()=>{
    document.getElementById('main-hub').style.display='flex';
    gameState='MENU';
    resetDailyMissions();
    updateDailyMissionsUI();
    updateAchievementsUI();
    updateSkinProgressUI();
    updateEventsUI();
    updateHubUI();
    updateGemUI();
    updateIndividualRewardsUI();
    updateSkinsUI();
    checkSkinUnlock();
    updateRankUI();
    initSettingsUI();
    checkDailyStreak();
});

// Initialize music on first user interaction
function initMusicOnFirstInteraction() {
    try {
        if (audioCtx && audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
        if (musicEnabled && !backgroundMusic && gameState === 'MENU') {
            playBackgroundMusic('menu');
        }
    } catch(e) {}
    document.body.removeEventListener('click', initMusicOnFirstInteraction);
    document.body.removeEventListener('touchstart', initMusicOnFirstInteraction);
}
document.body.addEventListener('click', initMusicOnFirstInteraction);
document.body.addEventListener('touchstart', initMusicOnFirstInteraction);
</script>

<div id="streak-popup" class="streak-popup">
    <h2>🔥 <span id="streak-popup-title">DAILY STREAK</span></h2>
    <div id="streak-popup-count" style="font-size:36px;color:#ff6600;font-weight:900;"></div>
    <div style="font-size:12px;color:#aaa;" id="streak-popup-subtitle">days in a row!</div>
    <div id="streak-popup-rewards"></div>
    <button class="streak-claim-btn" onclick="claimStreakReward()" id="streak-claim-btn">CLAIM REWARD</button>
</div>
</body>
</html>
