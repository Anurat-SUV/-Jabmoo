# จับหมู (Jabmoo) — HANDOFF / CONTEXT FILE #8
> วางไฟล์นี้ใน Project knowledge (แทนไฟล์ handoff เก่า) · **ชื่อไฟล์ handoff ต้องมีเลขเวอร์ชันเสมอ `jabmoo-handoff_8-v0821.NN.md` (ชื่อซ้ำ = A เห็นแต่ไฟล์แรก)** · Claude อ่านก่อนเสมอ ตอบไทย สั้น ตรง ไม่ชม · แนบ commit message ทุกครั้งที่ส่งไฟล์
> อัปเดตล่าสุด: **prod v0821.78 / dev v0821.78-dev** (2026-09-19) — **prod = dev ทุกบรรทัด ยกเว้น 6 บรรทัด dev-only** · A push เองผ่าน GitHub Desktop
> ✅ ยืนยันแล้ว: A ได้ไฟล์ .72 ครบ (แชท .71/.72 ตันแต่ไฟล์ออกทัน)

## 1. โปรเจกต์คืออะไร
เกมไพ่จับหมู (Gong Zhu แบบไทย) ของ A — single-file HTML (React 18 + Babel CDN) เล่นกับ bot 3 ตัว (Monte-Carlo AI) หรือออนไลน์ มีโปรไฟล์/XP/HoF/coin ผ่าน Supabase
ศัพท์: กอง(trick), ช้วน(Slam), บิ๊กช้วน(SuperSlam 788), หลัก(toH — ขั้นเหรียญ)

## 2. ไฟล์และตำแหน่ง
| ไฟล์ | เวอร์ชัน | หมายเหตุ |
|---|---|---|
| jabmoo.html | **v0821.78** | prod — COIN_ON=true, ~1.52MB |
| jabmoo-dev.html | **v0821.78-dev** | ต่าง prod **6 บรรทัด**: manifest-dev/icon-dev/ชื่อแอป DEV + `__JM_DEV=true` + VER + COIN_ON gate |
| jabmoo-admin.html | - | PIN=1828 · ลบผู้เล่นผ่าน SQL Editor เท่านั้น |
- Repo: https://github.com/Anurat-SUV/-Jabmoo (main=deploy Pages) | Live: https://anurat-suv.github.io/-Jabmoo/jabmoo.html
- **A push ผ่าน GitHub Desktop** (clone ที่ `D:\Work\VsProject\jabmoo-git\-Jabmoo`) — ขั้นตอน: ก๊อปไฟล์ทับ → Commit to main → Push origin
- **กฎเหล็ก: แก้ทั้งสองไฟล์ IN PLACE เสมอ ห้าม regen dev จาก prod** — python str.replace + `assert count==1` ทุก anchor
- export ไป /mnt/user-data/outputs/ เสมอ | **เปิดแชทใหม่ upload ทั้ง 2 ไฟล์ล่าสุด**
- **ขนาดไฟล์ ~1.52MB** — **ยังไม่ตัดสินใจ** single-file vs แยก `/img/*.webp`
- ไฟล์ asset ใน repo (root เดียวกับ html): icon-512/192/180/32.png, icon-maskable-192/512.png, icon-dev-512/192/180.png, manifest.json, manifest-dev.json
- **bump เลขเวอร์ชันทุกครั้งที่ส่งไฟล์** แม้แก้เล็ก
- **bump handoff ทุกครั้งที่ส่งไฟล์ด้วย** (บทเรียนใหม่: แชทตันแล้วเสีย context)

## 3. Supabase
- URL: https://kivamkangzuyjwfouusr.supabase.co | Key (publishable): sb_publishable_e4N4h_EupJMRg9rC3R8CGw_kgXZH7bI | RLS: select/insert/update เปิด, DELETE ปิด
- `players`: code(PK4), name, xp, games, jd_won/seen, qs_dodged/seen, slams, superslams, quits, wins, rank_sum, rk_games, pig_id, coins(cache)
- `coin_tx` (ledger append-only = ความจริง) + `coin_match` (audit; abandoned: seat "?", sum≠0 by design)
- `game_config` (key text PK, val jsonb) — remote override บอท row key='bots' `{"chars":{...},"lineup":{...}}` โหลดครั้งเดียว fail-open

## 4. ระบบ COIN
- GRANT 10,000 | หมด(<500) → เติมถึง 2,000/วัน เที่ยงคืนไทย | ต๋ง 15% ฝั่งได้→HOUSE | ledger-first | บัญชี BOT+HOUSE
- หลัก(toH): |n|≤50→0, ~ทุก100ขยับ1 | จ่าย=(4×หลักตัวเอง−รวมโต๊ะ)×mult | โต๊ะ 7 ระดับ ×100..×10K (bot ≤×500, min=50×mult)
- **A ตัดสินใจ: ไม่เอาเพดาน/sink สู้ด้วยบอทเก่งขึ้น** · จำโต๊ะล่าสุด localStorage `jm_lastTbl`

## 5. BOT — ตัวละคร + อาวุธ
`BOT_CHARS` knobs: dealsX, blur, hunt, campaign+huntP, antiLast, blowGuard, bayes, slamDef, hlak, **spGamble** — override ทาง game_config
| ตัว | dealsX | blur | อาวุธ | โต๊ะ |
|---|---|---|---|---|
| หมูเด้ง | 0.5 | 35% | — | ×100 |
| หมูทอด | 0.75 | 15% | — | ×100/200 |
| หมูป่า | 1.0 | 5% | hunt, blowGuard | ×200/500 |
| หมูเทพ | 1.2 | 0 | hunt, blowGuard | ×500 + SEAT TAKEOVER |
| เทพจับหมู | 1.5 | 0 | + bayes Q♠ (คนจริง) + campaign gated + SLAM DEFENSE + hlak(OFF) | ×500 boss |
- lineup ×100=[เด้ง,ทอด,เด้ง] ×200=[ทอด,ป่า,ทอด] ×500=[ป่า,เทพ,เทพจับหมู] · **ชื่อซ้ำเติมเลขอัตโนมัติ → "หมูเด้ง 1/2"**
- mcBudget ×100=600ms/110 ×200=1100/300 ×500+=1800/600 · min turn 0.68s
- **SPADE HIGH GAMBLE (.64)**: ตาม ♠ ตอน Q♠ ยัง live + มีคนตามหลัง + A♠/K♠ จะกินกอง → สุ่ม `spGamble` (default **0.5**) ตัด A/K ออกจากตัวเลือก · ปิดได้ `{"chars":{"thep":{"spGamble":0}}}` · **ยังไม่ bench**
- blowGuard / bayes Q♠ / SPADE CAMPAIGN + qsPosterior gate / SLAM DEFENSE (slamRisk + prune + penalty −150) / hlak (OFF) — ไม่แตะมาหลายรอบแล้ว
- heuristic เดิมครบ (Threat/EARLY HEART/788 containment/PIG EXIT/PROFIT GUARD/J♦ BAN/jdBoss/COVER GUARD/WOUNDED LEAD BAN/J♦ FOLLOW/J♦ COVER 50/50)

## 6. UI — สถานะปัจจุบัน (.74 ทั้ง prod+dev)
**Landing**: hero ภาพเดียว `JM_HERO2` · เมนู 6 การ์ด · ชิปโป๊กเกอร์ 7 โต๊ะ · แถวตั้งค่า 3 การ์ดทอง · การ์ด VS BOT/ONLINE · root มี paddingTop safe-area
**Header**: `JM_ICON_PIG` = **ไอคอนแอปจริงของ A** (icon-180.png → 72px webp base64 ~5KB, .73) · ใช้ 2 จุด: header portrait (24px) + header landscape (22px) · .72 เคยเป็น SVG หมูใส่แว่น — ถอนออกแล้ว
**In-game portrait**:
- `TABLE_BG` = ภาพผ้าโต๊ะของ A (2:3, 720×1080 webp 56KB) เต็มพื้นที่ · ♠ นูนกลางผ้า
- 4 ที่นั่งบนขอบไม้: ชื่อ → ป้ายคะแนน `เกมนี้ | รวม` → ชิป Lv/💰 → avatar วงทอง (เรืองแสงเมื่อถึงตา) → หลังไพ่ซ้อน → กล่องไพ่แต้มแถวของตัวเอง บรรทัดเดียวไม่ตัดบรรทัด
- ไพ่ที่ลง: ชิดขอบโต๊ะของแต่ละคน คำนวณจากความสูงบล็อกที่นั่งที่ **วัดจาก DOM จริง** (topSeatRef/botSeatRef)
- ไพ่ในมือ: **12-13 ใบ → เลข+ดอกมุมซ้ายบน · ≤11 ใบ → กลางใบ** (แก้เคสจอ 360px Samsung S24)
- ปุ่มเล่น เขียว+ขอบทอง · ปุ่ม 😊 อิโมจิ (12 ตัว) + ปุ่ม 💬 แชตกลม (online เท่านั้น) มุมล่างขวาผ้า
- **แผงแชตบนผ้า (.76, portrait)**: แทน drawer ล่าง (โค้ดเก่าปิดด้วย `false&&`) · อยู่ขวาผ้า top=`max(ใต้บล็อกที่นั่งขวา(rightSeatRef), ใต้ไพ่ที่ลงขวา)` bottom=54 (เหนือปุ่ม 💬/😊) left=`max(50%+tW2/2+8, 100%−196px)` ไม่ทับไพ่ตัวเอง · online เท่านั้น · จาง (opacity .55) จนแตะ/โฟกัส input → เต็ม (chatOpen) · blur → จางกลับ · ซ่อนตอน roundSummary · landscape ยัง drawer เดิม · **TYPING MODE (.78)**: `chatOpen` = โหมดพิมพ์ → root portrait `scale(0.6)` มุมซ้ายบน + คอลัมน์แชตขวา 40% สูง = `visualViewport.height` (พ้นคีย์บอร์ด) input `autoFocus` · แผงบนผ้าซ่อนตอนพิมพ์ (ช่องล่างของแผงเป็น div แตะเข้าโหมด ไม่ใช่ input จริง) · ออกด้วย ✕ หรือปุ่ม 💬 · vv resize/scroll → `scrollTo(0,0)` กันหน้าเด้ง · **`CHAT_SOLO` (.77) = `!!window.__JM_DEV` บรรทัดเดียวกันทั้ง 2 ไฟล์ → dev โชว์แผง+ปุ่ม 💬 ใน vs BOT ด้วย (ทดสอบคนเดียว) prod ยัง online เท่านั้น**
- **ปุ่ม ⚡ จบเร็ว**: โผล่มุมซ้ายบนเมื่อไพ่แต้มครบ 16 ใบถูกเก็บแล้ว → auto-play + บอทข้าม MC (120ms) + โชว์กอง 0.35s · **(.75) effect auto-play นับไพ่แต้มเองทุก render: seen<16 → รีเซ็ต fastFinish** (บั๊ก: กด ⚡ เกม 8 แล้ว host เริ่ม rematch → ที่นั่ง A auto-play ตั้งแต่ตา 1)
- **นับถอยหลังกลางโต๊ะ (.71 ใหม่)**: `TURN_SHOW=10` — เลขใหญ่โปร่งแสงกลางผ้า ขนาดเท่าไพ่ที่ลง นับ 10→0, **แดง+เต้นเมื่อ ≤3 วิ** · turn timer จริง = **25 วิ** (`setInterval(...,1000)` 1 tick = 1 วิ)
- **เสียงบี๊บ (.71)**: `playCountBeep` เขียนใหม่ — 2 oscillator detune, gain ~3 เท่าของเดิม (A บอกเดิมเบาเกิน)
- **จำนวนเกม "N/M" (.72)**: โหมดกำหนดจำนวนเกม แสดง "5/8" แทนเลขเดี่ยว
- สรุปคะแนนต่อเกม: กล่องแยกตามที่นั่ง ธีมเขียว-ทอง 👑 คนได้สูงสุด · เกมสุดท้าย hold 5 วิก่อนเข้าหน้าจบเกม
**In-game landscape (.72 — รื้อใหม่ทั้งหมด, TODO ค้างเก่าเคลียร์แล้ว)**:
- `LandscapeView()` เขียนใหม่ ใช้ `TABLE_BG` ตัวเดียวกัน **หมุน 90°** เป็นผ้าโต๊ะ (เลิกธีมน้ำเงินเก่า)
- 3 ที่นั่งคู่แข่งวางบนราว ใช้ component ชุดเดียวกับ portrait (ชื่อ → ป้ายคะแนน → avatar วงทอง → หลังไพ่ซ้อน → ไพ่แต้ม)
- ที่นั่งตัวเองย้ายเข้าไปอยู่ในแผงมือไพ่ ข้างไพ่ 13 ใบ + ปุ่มเล่นเขียว-ทอง
- port ครบจาก portrait: สรุปคะแนนต่อเกม, ปุ่ม ⚡ จบเร็ว, drawer แชต, ปุ่มอิโมจิ, modal ประวัติคะแนน
- **safe-area ซ้าย/ขวา (.74)**: ที่นั่งซ้าย/ขวา + ไพ่ที่ลงของทั้งสอง + header + แผงมือไพ่ + ปุ่ม ⚡/😊/💬 เลี่ยง Dynamic Island ด้วย `max(env(safe-area-inset-left/right),var(--sal,0px))` (ผ้าโต๊ะยัง full-bleed)
**หน้าจบเกม portrait**: ถ้วย+ริบบิ้น · ตาราง · ปุ่ม เล่นอีกรอบ (ซ้าย) + หน้าแรก/ออกจากห้อง (ขวา) แถวเดียว · แถบ coin · root `position:fixed` สูง `--vh`
**หน้าจบเกม landscape (.74 ใหม่)**: `GameOver` รับ prop `land={orient==="landscape"}` → แยก branch เต็มจอ 2 คอลัมน์ ไม่ scroll · ซ้าย = ถ้วย+ริบบิ้น"จบเกม!"+ผู้ชนะ+กล่อง coin · ขวา = ตารางอันดับ + แถวปุ่ม [เล่นอีกรอบ | หน้าแรก | 📊 | 📋] · ประวัติคะแนนเป็น overlay เต็มจอ · portrait branch ไม่แตะ
**กติกา/วิธีเล่น**: รูป poster ของ A เต็มหน้า — ข้อ 3 เป็น HTML สด
**Hall of Fame**: แท็บ ฝีมือ → XP → เศรษฐี, default=ฝีมือ, โพเดียมทุกแท็บ ตารางเริ่มอันดับ 4
**หมูประจำตัว**: id 3 = "เจ้าหญิงหมู" รูปถือไพ่ของ A ขอบชมพู · หน้า welcome ใช้ avatar นี้
**PWA icons**: ไอคอนใหม่ของ A คีย์พื้นดำออก + maskable + ชุด DEV ติดป้ายแดง

## 7. VIEWPORT / iOS
- อาการ: iPhone PWA เหลือแถบตายก้นจอ 59px · header ซ้อน status bar
- เหตุ: iOS 26 home-screen app รายงาน `innerHeight = screen − status bar` (430×873/932) แต่ `env(safe-area-inset-top)=0`
- **`--sal` (.74)**: side inset สำหรับ landscape — probe `env(safe-area-inset-left/right)` ก่อน ถ้าได้ 0 และ viewport ไม่ถูกหดอยู่แล้ว (`screen ยาว − innerWidth < 24`) และเป็น iOS จอยาว (ratio ≥1.95) → fallback 59px (จอสั้น ≥393) / 47px
- แก้: `--vh` จาก `max(innerHeight, visualViewport.height)` วัดซ้ำที่ 0.1/0.4/1/2.5/5s + pageshow/visibilitychange/resize · `--sat` fallback = `screen.height − innerHeight` ใช้ผ่าน `max(env(safe-area-inset-top),var(--sat,0px))` · status bar = `black` — **ต้อง Add to Home Screen ใหม่ถึงมีผล**
- ป้าย DEV แสดงสด: `ih= vv= sc= pwa sat= vh=` (เครื่องมือ debug หลัก)

## 8. ออนไลน์
- transport auto-reconnect / host heartbeat 3s / client resend >2.5s / host หลุด 30s→settle / SEAT TAKEOVER 45s บอทเทพ🤖
- **AUTO-REJOIN (.75)**: client เห็น frame ที่ `sub[mySeat]=true` ขณะยังต่ออยู่ → ส่ง `rejoin` เอง (throttle 3s) · host รับ `play` จากที่นั่งที่ subbed → un-sub แล้วรับไพ่ (แทนที่จะเมิน) — กันเคสโหวต ready หาย/host กด "เริ่มเลย"
- **REJOIN (.66)**: `selfId` เก็บใน localStorage `jm_peer_id` · host เก็บ `startPayload` → เห็น hello จาก id ที่ตรงกับที่นั่ง → ส่ง start ใหม่ + `{type:"rejoin",seat}` ปลด subbed คืนที่นั่งกลางเกม
- **กลับเข้าห้องเดิม (.69)**: `jm_last_room` {room,name,t} → ปุ่ม "↩ กลับเข้าห้องเดิม XXXX" (หมดอายุ 6 ชม.)
- **REMATCH READY VOTE (.65)**: client กด "พร้อมเล่นอีกรอบ" = โหวต · host เห็น "พร้อม N/M" · host กดเริ่ม → ที่นั่งที่ไม่พร้อมเล่นด้วยบอท
- **แชตในเกม (.68/.70)**: บับเบิลข้างที่นั่งคนส่ง 4.5 วิ (อิโมจิ 2.2 วิ) ทั้ง portrait/landscape · ตัดที่ 90 ตัวอักษร สูงสุด 4 บับเบิล

## 9. AUDIO
- `jmAC()` = AudioContext เดียวใช้ร่วมทั้งหน้า + `resume()` อัตโนมัติเมื่อกลับเข้าแอป (visibilitychange) — แก้ iOS เสียงเงียบหลังสลับแอป
- `playCountBeep` (.71) ดังขึ้น 3 เท่า 2 oscillator detune

## 10. TODO
1. **ทดสอบจริง .71-.74**: นับถอยหลัง + เสียงบี๊บบนมือถือจริง · landscape ใหม่ทุกขนาด · **ยืนยัน --sal ว่าไพ่แต้มซ้ายพ้น Dynamic Island จริง (ถ้ายังโดน = เพิ่มค่า fallback)** · หน้าจบเกมแนวนอน
2. ตัดสินใจ single-file vs `/img/` แยก (~1.52MB)
3. ทดสอบจริง 2 เครื่อง: **rematch หลัง 8 เกมโดยกด ⚡ ในเกมสุดท้าย (.75)** + online rejoin (.66) + rematch ready (.65) + แชตบับเบิล (.68)
4. วัดผล live ×500: spGamble (.64) ไม่เคย bench · SQL Kob per_match + bot_blowups<−600 · hlak ลอง `{"chars":{"thepjab":{"hlak":0.3}}}`
5. Bayesian ขยาย · botChar ลง coin_match seats meta · รูปหมูตัวที่ 5 · ลงโทษ host หนี · per-player min online / Elo / Edge Function / สามกอง (project แยก)

## 11. บทเรียน bench + เครื่องมือ
- Paired self-play se~13-16 ที่ n~150 → เห็นเฉพาะ effect >±25 · อาวุธจาก human-read ห้ามใส่กับบอท (gate isBot) · hard rule จาก trace เป็น prune ไม่ใช่ penalty
- เครื่องมือ (sandbox รีเซ็ตทุกแชท สร้างใหม่จากคำอธิบาย): extract.py / bench.js (`--mode aa|camp|hlak|slamdef|slamrate`) / trace.js / mount.js
- **playwright chromium**: copy react/react-dom/babel จาก node_modules ไป `shot/` แทน CDN, serve `setsid nohup python3 -m http.server 8765 &`, screenshot ทุก viewport · **shot3.py** วนเล่นเกมจริงหลาย viewport = ตัวจับ error หลักก่อนส่งไฟล์ทุกครั้ง
- **patch ที่ต้องทำในสำเนา shot/ เพื่อถ่ายภาพ** (สะสมจาก .71/.72):
  - bypass welcome modal → patch `needName` / `hasName`
  - ปิด `COIN_ON` + ปิด name check ใน `onStart`
  - online-only UI → replace `{net&&` → `{true&&`
  - อยากเห็น countdown → บังคับ `timerOff=false` + ลด `TURN_SECS` เหลือ 12-14
- Setup: `npm i @babel/core @babel/preset-react @babel/standalone@7.23.5 jsdom react@18 react-dom@18` · pip playwright/PIL/numpy มีแล้ว

## 12. วิธีทำงาน
A ส่ง log/ภาพ/ไอเดีย → reproduce → แก้ → regression (shot3.py ทุกครั้ง) → bench ถ้าเป็นพฤติกรรมบอท → **bump vMMDD.N ทุกครั้ง** → export ทั้ง 2 ไฟล์ + commit message → A push
- รูปจาก A: ขอ transparent PNG (ตรวจ alpha จริง — ChatGPT ฝังพื้นดำ/หมากรุกปลอม), ย่อ webp q75-82 ฝัง base64
- A ชอบเห็น preview ภาพก่อนตัดสินใจ UI — ทำ screenshot เทียบให้เสมอ
- เมื่อวิเคราะห์ log ไพ่: ไล่ไพ่ที่เหลือของทุกคนก่อนตอบ — หลายครั้งคำตอบคือ **ผลเท่ากันทั้งสองทาง**
- **แชทยาว: พอรู้สึกว่าใกล้ตัน ให้สร้าง handoff ใหม่ทันที อย่ารอจบงาน**

## 13. ประวัติ .35-.72
.35-.41 โต๊ะใหม่ (TABLE_BG, 4 ที่นั่ง, ไพ่ชิดขอบ, poster, ไอคอน PWA) | .42 วัดบล็อกที่นั่งจาก DOM | .43-.47 iOS viewport | .46 HoF โพเดียม | .48-.51 สรุปคะแนนกล่องตามที่นั่ง + อิโมจิ 12 + hold 5 วิ | .52-.53 GameOver ไม่ล้นจอ + landing safe-area | .54 corner index + color-scheme dark | .55-.56 (dev only ถอนออก .60) | .57-.59 หมูเจ้าหญิง avatar id3 | .60-.61 corner index ตามจำนวนใบ | .62 ⚡ จบเร็ว | .63 ไพ่แต้มแถวตัวเอง | .64 SPADE HIGH GAMBLE | .65 rematch ready vote | .66 online rejoin + AudioContext เดียว | .67 ชื่อบอทซ้ำเติมเลข | .68 แชตบับเบิล | .69 ปุ่มกลับห้องเดิม | .70 ปุ่มแชตกลม | **.71 countdown กลางโต๊ะ + เสียงบี๊บดังขึ้น** | **.72 ไอคอนหมูแว่น + เกม N/M + landscape รื้อใหม่ธีมเขียว-ทอง** | **.73 header ในเกมใช้ไอคอนแอปจริง** | **.74 landscape safe-area ซ้าย/ขวา (--sal) + หน้าจบเกมแนวนอน** | **.75 fastFinish ไม่ค้างข้าม rematch + client auto-rejoin เมื่อถูก sub** | **.76 แผงแชตใสบนผ้า portrait (แทน drawer)** | **.77 CHAT_SOLO: dev เห็นแชตใน vs BOT** | **.78 typing mode: โต๊ะย่อ 60% + คอลัมน์แชตสูงเท่า visualViewport**
