<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>For Rai — A Little Book</title>
  <style>
    :root{--bg:#f4efe9;--paper:#fff;--accent:#d45d8a}
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
    body{display:flex;align-items:center;justify-content:center;background:linear-gradient(180deg,#fefefe, #f4efe9);padding:24px}
    .wrap{max-width:900px;width:100%;display:grid;grid-template-columns:1fr 300px;gap:24px;align-items:start}
    .book-container{perspective:2000px}
    .book{width:560px;height:360px;position:relative;margin:auto}
    .page{position:absolute;top:0;left:0;width:100%;height:100%;background:var(--paper);border-radius:8px;box-shadow:0 8px 30px rgba(10,10,10,.12);overflow:hidden;transform-origin:left;transition:transform .8s cubic-bezier(.2,.9,.3,1);display:flex;flex-direction:column}
    .page.right{transform-origin:right}
    .page.hidden{visibility:hidden}
    .page .content{padding:28px;white-space:pre-wrap;overflow:auto;font-size:18px;color:#222}
    .spine{position:absolute;left:50%;top:0;bottom:0;width:6px;background:linear-gradient(180deg,#e6dcd7,#d7c9c3);transform:translateX(-50%);border-radius:0 2px 2px 0}
    .controls{display:flex;gap:12px;align-items:center}
    button{background:var(--accent);color:#fff;border:0;padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600}
    button.secondary{background:transparent;color:var(--accent);border:2px solid rgba(0,0,0,.06)}
    .sidebar{padding:16px}
    .letter-list{max-height:420px;overflow:auto;border-radius:8px;background:rgba(255,255,255,.6);padding:8px}
    .letter-item{padding:8px;border-radius:6px;cursor:pointer}
    .letter-item:hover{background:rgba(0,0,0,.03)}
    h1{margin:0 0 10px 0;font-size:20px}
    .play-overlay{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;background:linear-gradient(180deg, rgba(0,0,0,.05), rgba(0,0,0,.03));pointer-events:none}
    .tap-to-play{pointer-events:auto;background:rgba(255,255,255,.95);padding:16px 22px;border-radius:999px;box-shadow:0 6px 18px rgba(10,10,10,.12);cursor:pointer}
    @media(max-width:920px){.wrap{grid-template-columns:1fr;gap:12px}.book{width:100%;height:420px}}
  </style>
</head>
<body>
  <div class="wrap">
    <div>
      <div class="book-container">
        <div class="book" id="book">
          <div class="spine"></div>
          <div class="page left" id="page-left" style="transform:rotateY(0deg)">
            <div class="content" id="left-content"> </div>
          </div>
          <div class="page right" id="page-right" style="transform:rotateY(0deg)">
            <div class="content" id="right-content"></div>
          </div>
          <div class="play-overlay" id="overlay">
            <div class="tap-to-play" id="startBtn">Tap to open & play music</div>
          </div>
        </div>
        <div style="display:flex;justify-content:center;margin-top:12px;gap:12px;" class="controls">
          <button id="prevBtn">◀ Previous</button>
          <button id="nextBtn">Next ▶</button>
          <button class="secondary" id="toggleMusic">Pause Music</button>
        </div>
      </div>
    </div>
    <aside class="sidebar">
      <h1>Your letters</h1>
      <div class="letter-list" id="letterList"></div>
      <p style="margin-top:12px;color:#444;font-size:14px">Tip: click a letter to jump directly to it.</p>
    </aside>
  </div>

  <!-- Background Music -->
  <audio id="bgm" loop preload="auto">
    <source src="https://docs.google.com/uc?export=download&id=1V97LL6bAIu3Vn8YEJThGB3LNXNU_Cmkd" type="audio/mpeg">
  </audio>

  <script>
    const letters = [
`500 Things I Love About Rai`,
`1. she's kind. tangina, sobrang bait...`,
`2. she's sweet. kahit hindi halata...`,
`3. she's beautiful. putangina, men...`,
`4. she's funny. gago, boi...`,
`5. she's everything, nasa kaniya na lahat...`,
`you thought i was gonna write 500 reasons?...`,
`Rai`,
`All About Rai\n\ndear rai, you really are the happiest accident...`,
`Why I Love Rai\n\ndear rai, honestly, i love you because you’re you...`,
`How Did We Meet?\n\ndear rai, do you still remember nung nag meet tayo sa fg?...`,
`Our Unforgettable Memories\n\ndear rai, our memories together? sobrang priceless...`,
`Rai’s Mood Swings\n\ndear rai, your mood swings?...`,
`When Rai is Angry\n\ndear rai, when you’re angry, nang l-like zone ka...`,
`When Rai is Sad\n\ndear rai, when you’re sad, it’s the worst feeling...`,
`When Rai is Happy\n\ndear rai, kapag masaya ka, i swear, it’s the best...`,
`Rai’s Personal Life\n\ndear rai, i know your academics drain you...`,
`Rai’s Insecurities\n\ndear rai, i wish you could see yourself...`,
`Rai’s Avoidant Attachment\n\ndear rai, i know you have that avoidant side...`,
`Why Do I Love Her So Much?\n\ndear rai, why do i love you so much?...`,
`My Prayer to God for Rai\n\ndear rai, every night before i sleep...`,
`so ayon, natapos mo na finally ‘tong book...`,
`i don’t know kung hanggang saan tayo dadalhin ng tadhana...`,
`and if dumating yung days na you’ll feel lost...`,
`please.. don’t be in love with someone else, rai :(`
    ];

    const leftContent = document.getElementById('left-content');
    const rightContent = document.getElementById('right-content');
    const prevBtn = document.getElementById('prevBtn');
    const nextBtn = document.getElementById('nextBtn');
    const letterListEl = document.getElementById('letterList');
    const bgm = document.getElementById('bgm');
    const overlay = document.getElementById('overlay');
    const startBtn = document.getElementById('startBtn');
    const toggleMusic = document.getElementById('toggleMusic');

    let index = 0;

    function render() {
      leftContent.textContent = letters[index - 1] || '';
      rightContent.textContent = letters[index] || '';
      letterListEl.innerHTML = '';
      letters.forEach((l, i) => {
        const div = document.createElement('div');
        div.className = 'letter-item';
        div.textContent = (i+1) + '. ' + l.split('\n')[0].slice(0,40);
        div.onclick = () => { index = i; flipTo(i); };
        if(i===index) div.style.fontWeight = '700';
        letterListEl.appendChild(div);
      });
    }

    function flipTo(i) {
      const left = document.getElementById('page-left');
      const right = document.getElementById('page-right');
      if(i>index){
        right.style.transform = 'rotateY(-180deg)';
        setTimeout(()=>{
          index = i;
          right.style.transform = 'rotateY(0deg)';
          render();
        }, 420);
      } else if(i<index){
        left.style.transform = 'rotateY(180deg)';
        setTimeout(()=>{
          index = i;
          left.style.transform = 'rotateY(0deg)';
          render();
        }, 420);
      } else render();
    }

    prevBtn.onclick = () => { if(index>0) flipTo(index-1); };
    nextBtn.onclick = () => { if(index<letters.length-1) flipTo(index+1); };

    startBtn.onclick = async () => {
      try{ await bgm.play(); }catch(e){ }
      overlay.style.display = 'none';
    };

    toggleMusic.onclick = () => {
      if(bgm.paused){ bgm.play(); toggleMusic.textContent = 'Pause Music'; }
      else { bgm.pause(); toggleMusic.textContent = 'Play Music'; }
    };

    render();
  </script>
</body>
</html>
