# จับหมู (Jabmoo) — HANDOFF / CONTEXT FILE #7
> วางไฟล์นี้ใน Project knowledge (แทน jabmoo-handoff_6.md) · Claude อ่านก่อนเสมอ ตอบไทย สั้น ตรง ไม่ชม · แนบ commit message ทุกครั้งที่ส่งไฟล์
> อัปเดตล่าสุด: **prod v0821.70 / dev v0821.70-dev** (2026-09-19) — **prod = dev ทุกบรรทัด ยกเว้น 6 บรรทัด dev-only** · A push เองผ่าน GitHub Desktop

## 1. โปรเจกต์คืออะไร
เกมไพ่จับหมู (Gong Zhu แบบไทย) ของ A — single-file HTML (React 18 + Babel CDN) เล่นกับ bot 3 ตัว (Monte-Carlo AI) หรือออนไลน์ มีโปรไฟล์/XP/HoF/coin ผ่าน Supabase
ศัพท์: กอง(trick), ช้วน(Slam), บิ๊กช้วน(SuperSlam 788), หลัก(toH — ขั้นเหรียญ)

## 2. ไฟล์และตำแหน่ง
| ไฟล์ | เวอร์ชัน | หมายเหตุ |
|---|---|---|
| jabmoo.html | **v0821.70** | prod — COIN_ON=true, ~1.51MB |
| jabmoo-dev.html | **v0821.70-dev** | ~1.51MB — ต่าง prod **6 บรรทัด**: manifest-dev/icon-dev/ชื่อแอป DEV + `__JM_DEV=true` + VER + COIN_ON gate |
| jabmoo-admin.html | - | PIN=1828 · ลบผู้เล่นผ่าน SQL Editor เท่านั้น |
- Repo: https://github.com/Anurat-SUV/-Jabmoo (main=deploy Pages) | Live: https://anurat-suv.github.io/-Jabmoo/jabmoo.html
- **A push ผ่าน GitHub Desktop** (clone ที่ `D:\Work\VsProject\jabmoo-git\-Jabmoo`) — GitHub web Upload files พังที่ขนาด 1.4MB+ · ขั้นตอน: ก๊อปไฟล์ทับ → Commit to main → Push origin
- **กฎเหล็ก: แก้ทั้งสองไฟล์ IN PLACE เสมอ ห้าม regen dev จาก prod** — python str.replace + `assert count==1` ทุก anchor
- export ไป /mnt/user-data/outputs/ เสมอ | **เปิดแชทใหม่ upload ทั้ง 2 ไฟล์ล่าสุด**
- **ขนาดไฟล์ 1.51MB** (โต 650KB→1.51MB จาก asset ชุดใหม่) — **ยังไม่ตัดสินใจ** single-file vs แยก `/img/*.webp` · GitHub Desktop push ได้ ไม่ติดลิมิตแล้ว
- ไฟล์ asset ใน repo (root เดียวกับ html): icon-512/192/180/32.png, icon-maskable-192/512.png, icon-dev-512/192/180.png, manifest.json, manifest-dev.json
- **bump เลขเวอร์ชันทุกครั้งที่ส่งไฟล์** แม้แก้เล็ก (เคยพลาด: แก้หลายรอบไม่ bump → A แยกไม่ออกว่าโหลดตัวไหน)

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
- lineup ×100=[เด้ง,ทอด,เด้ง] ×200=[ทอด,ป่า,ทอด] ×500=[ป่า,เทพ,เทพจับหมู] · **ชื่อซ้ำเติมเลขอัตโนมัติ (.67) → "หมูเด้ง 1/2"**
- mcBudget ×100=600ms/110 ×200=1100/300 ×500+=1800/600 · min turn 0.68s
- **SPADE HIGH GAMBLE (.64 ใหม่)**: ตาม ♠ ตอน Q♠ ยัง live + มีคนตามหลัง + A♠/K♠ จะกินกอง → สุ่ม `spGamble` (default **0.5**) ตัด A/K ออกจากตัวเลือก = บางครั้งปล่อย บางครั้งกิน (A สั่งให้เป็น random ไม่ใช่กฎตายตัว) · ปิดได้ทาง remote `{"chars":{"thep":{"spGamble":0}}}` · **ยังไม่ bench**
- blowGuard / bayes Q♠ / SPADE CAMPAIGN + qsPosterior gate / SLAM DEFENSE (slamRisk + prune + penalty −150) / hlak (OFF) — รายละเอียดเดิมใน handoff_6 ไม่แตะเลยรอบนี้
- heuristic เดิมครบ (Threat/EARLY HEART/788 containment/PIG EXIT/PROFIT GUARD/J♦ BAN/jdBoss/COVER GUARD/WOUNDED LEAD BAN/J♦ FOLLOW/J♦ COVER 50/50)

## 6. UI — สถานะปัจจุบัน (.70 ทั้ง prod+dev)
**Landing**: hero ภาพเดียว `JM_HERO2` · เมนู 6 การ์ด · ชิปโป๊กเกอร์ 7 โต๊ะ · แถวตั้งค่า 3 การ์ดทอง · การ์ด VS BOT/ONLINE · root มี paddingTop safe-area (แก้ .53)
**In-game portrait (โต๊ะใหม่ .36-.63)**:
- `TABLE_BG` = ภาพผ้าโต๊ะของ A (2:3, 720×1080 webp 56KB, .41) เต็มพื้นที่ · ♠ นูนกลางผ้า
- 4 ที่นั่งบนขอบไม้: ชื่อ → **ป้ายคะแนน `เกมนี้ | รวม`** (.38, สลับ+ขยาย .40) → ชิป Lv/💰 (+20% .63) → avatar วงทอง (เรืองแสงเมื่อถึงตา) → หลังไพ่ซ้อน (หน้าตัด ~1.3 ใบ, โชว์ขอบ 6 ชั้น .40) → **กล่องไพ่แต้มแถวของตัวเอง บรรทัดเดียวไม่ตัดบรรทัด (.63)**
- **ถอดแถบคะแนน 4 ช่องด้านบนออกแล้ว** (.38) · ถอดบรรทัด "ถึงตาคุณ! N ใบ" (.39) · ถอดป้าย lead/ได้กอง (.37)
- ไพ่ที่ลง: ชิดขอบโต๊ะของแต่ละคน คำนวณจากความสูงบล็อกที่นั่งที่ **วัดจาก DOM จริง** (topSeatRef/botSeatRef) + จองที่ไพ่แต้ม 1 แถว → ไม่ขยับ ไม่ทับ
- ไพ่ในมือ: **12-13 ใบ → เลข+ดอกมุมซ้ายบน · ≤11 ใบ → กลางใบ** (.60, ขนาด +10% .61) — แก้เคสจอ 360px (Samsung S24) อ่านไม่ออก
- ปุ่มเล่น เขียว+ขอบทอง · ปุ่ม 😊 อิโมจิ (12 ตัว) + **ปุ่ม 💬 แชตกลมข้างกัน (.70, online เท่านั้น)** มุมล่างขวาผ้า
- **ปุ่ม ⚡ จบเร็ว (.62)**: โผล่มุมซ้ายบนโต๊ะเมื่อไพ่แต้มครบ 16 ใบถูกเก็บแล้วและยังไม่ถึงตา 13 → auto-play ที่นั่งตัวเอง + บอทข้าม MC (120ms) + โชว์กอง 0.35s
- **สรุปคะแนนต่อเกม**: กล่องแยกตามที่นั่ง ธีมเขียว-ทอง ทับกลางไพ่ตัวเอง 👑 คนได้สูงสุด (.49-.50)
- **เกมสุดท้าย**: โชว์สรุปคะแนน 5 วิ ก่อนเข้าหน้าจบเกม (.50)
**หน้าจบเกม**: ถ้วย+ริบบิ้น ("จบเกม!" จัดกลาง .51) · ตาราง · **ปุ่ม เล่นอีกรอบ (ซ้าย) + หน้าแรก/ออกจากห้อง (ขวา) แถวเดียว (.53)** · แถบ coin · Good Game! · root `position:fixed` สูง `--vh` ไม่ล้นจอ (.52)
**กติกา/วิธีเล่น**: รูป poster ของ A เต็มหน้า (.36) — วิธีเล่นตัดเป็น 2 ท่อน ข้อ 3 เป็น HTML สด (ข้อความจับหมูที่ถูกต้อง)
**Hall of Fame (.46)**: แท็บ ฝีมือ → XP → เศรษฐี, default=ฝีมือ, "สะสม"→"XP", **โพเดียมทุกแท็บ** ตารางเริ่มอันดับ 4
**หมูประจำตัว**: id 3 = "เจ้าหญิงหมู" รูปถือไพ่ของ A (.58) ขอบชมพู · หน้า welcome ใช้ avatar นี้ (.59)
**PWA icons (.57-)**: ไอคอนใหม่ของ A คีย์พื้นดำออก + maskable + ชุด DEV ติดป้ายแดง
- **landscape in-game ยังธีมน้ำเงินเก่า** (งานค้าง)

## 7. VIEWPORT / iOS (แก้ยาว .43-.47)
- อาการ: iPhone PWA เหลือแถบตายก้นจอ 59px · header ซ้อน status bar
- เหตุ: iOS 26 home-screen app รายงาน `innerHeight = screen − status bar` (430×873/932) แต่ `env(safe-area-inset-top)=0`
- แก้ปัจจุบัน: `--vh` จาก `max(innerHeight, visualViewport.height)` วัดซ้ำที่ 0.1/0.4/1/2.5/5s + pageshow/visibilitychange/resize · `--sat` fallback = `screen.height − innerHeight` (เมื่อ standalone + portrait + env=0 + gap 40-100) ใช้ผ่าน `max(env(safe-area-inset-top),var(--sat,0px))` ทุกจุด · status bar = `black` (ทึบ) — **ต้อง Add to Home Screen ใหม่ถึงมีผล**
- ป้าย DEV แสดงสด: `ih= vv= sc= pwa sat= vh=` (เครื่องมือ debug หลัก ขอภาพจาก A ได้)

## 8. ออนไลน์
- transport auto-reconnect / host heartbeat 3s / client resend >2.5s / host หลุด 30s→settle / SEAT TAKEOVER 45s บอทเทพ🤖
- **REJOIN (.66 ใหม่)**: `selfId` เก็บใน localStorage `jm_peer_id` (เดิมสุ่มใหม่ทุกโหลด) · host เก็บ `startPayload` → เห็น hello จาก id ที่ตรงกับที่นั่ง → ส่ง start ใหม่ + `{type:"rejoin",seat}` ปลด subbed คืนที่นั่งกลางเกม + ข้อความในแชท
- **กลับเข้าห้องเดิม (.69)**: เก็บ `jm_last_room` {room,name,t} → ปุ่ม "↩ กลับเข้าห้องเดิม XXXX" บนหน้าเข้าออนไลน์ (หมดอายุ 6 ชม.)
- **REMATCH READY VOTE (.65)**: จบแมตช์แล้ว client กด "พร้อมเล่นอีกรอบ" = โหวต (ไม่เริ่มเอง) · host เห็น "พร้อม N/M" · host กดเริ่ม → ที่นั่งที่ไม่พร้อม/ออกไปแล้วเล่นด้วยบอท · ปุ่มขวาของ client = "ออกจากห้อง" ส่ง `{type:"leave"}`
- **แชตในเกม (.68)**: ข้อความเด้งเป็นบับเบิลข้างที่นั่งคนส่ง 4.5 วิ (อิโมจิ 2.2 วิ) ใช้ overlay เดียวกัน ทั้ง portrait/landscape · ตัดที่ 90 ตัวอักษร สูงสุด 4 บับเบิล

## 9. AUDIO (.66)
- เดิมสร้าง `AudioContext` ใหม่ทุกเสียง (11 จุด) → iOS จำกัดจำนวน context ต่อหน้า สลับแอปไปมาหลายรอบแล้วเสียงเงียบจนกว่าจะรีโหลด (A เจอบ่อยมาก)
- แก้: `jmAC()` = context เดียวใช้ร่วม + `resume()` อัตโนมัติเมื่อกลับเข้าแอป (visibilitychange)

## 10. TODO
1. **landscape in-game → ธีมเขียว-ทอง** + ขยายโต๊ะ, บีบแถบผู้เล่นบน (ค้างมานาน)
2. ตัดสินใจ single-file vs `/img/` แยก (1.51MB)
3. ทดสอบจริง: online rejoin (.66) + rematch ready (.65) + แชตบับเบิล (.68) ต้องใช้ 2 เครื่อง
4. วัดผล live ×500: spGamble (.64) ไม่เคย bench · SQL Kob per_match + bot_blowups<−600 · hlak ลอง `{"chars":{"thepjab":{"hlak":0.3}}}`
5. Bayesian ขยาย · botChar ลง coin_match seats meta · รูปหมูตัวที่ 5 (ตอนนี้ avatar อิงที่นั่ง ไม่ใช่ตัวละคร) · ลงโทษ host หนี · per-player min online / Elo / Edge Function / สามกอง (project แยก)

## 11. บทเรียน bench + เครื่องมือ
- Paired self-play se~13-16 ที่ n~150 → เห็นเฉพาะ effect >±25 · อาวุธจาก human-read ห้ามใส่กับบอท (gate isBot) · hard rule จาก trace เป็น prune ไม่ใช่ penalty
- เครื่องมือ (sandbox รีเซ็ตทุกแชท สร้างใหม่จากคำอธิบาย): extract.py / bench.js (`--mode aa|camp|hlak|slamdef|slamrate`) / trace.js / mount.js
- **playwright chromium**: copy react/react-dom/babel จาก node_modules ไป `shot/` แทน CDN, serve `setsid nohup python3 -m http.server 8765 &`, screenshot ทุก viewport · **shot3.py** วนเล่นเกมจริงหลาย viewport แล้วจับภาพ = ตัวจับ error หลักก่อนส่งไฟล์ทุกครั้ง
- ทดสอบ UI ที่มีเฉพาะ online: ก๊อปไฟล์ใน shot/ แล้ว replace `{net&&` → `{true&&` ชั่วคราวเพื่อถ่ายภาพ
- Setup: `npm i @babel/core @babel/preset-react @babel/standalone@7.23.5 jsdom react@18 react-dom@18` · pip playwright/PIL/numpy มีแล้ว

## 12. วิธีทำงาน
A ส่ง log/ภาพ/ไอเดีย → reproduce → แก้ → regression (shot3.py ทุกครั้ง) → bench ถ้าเป็นพฤติกรรมบอท → **bump vMMDD.N ทุกครั้ง** → export ทั้ง 2 ไฟล์ + commit message → A push
- รูปจาก A: ขอ transparent PNG (ตรวจ alpha จริง — ChatGPT ฝังพื้นดำ/หมากรุกปลอม), ย่อ webp q75-82 ฝัง base64
- A ชอบเห็น preview ภาพก่อนตัดสินใจ UI — ทำ screenshot เทียบให้เสมอ
- เมื่อวิเคราะห์ log ไพ่: ไล่ไพ่ที่เหลือของทุกคนก่อนตอบ ("ทำไมบอทลงใบนี้") หลายครั้งคำตอบคือ **ผลเท่ากันทั้งสองทาง**

## 13. ประวัติ .35-.70
.35-.41 โต๊ะใหม่ (TABLE_BG, 4 ที่นั่ง, ไพ่ชิดขอบ, poster กติกา/วิธีเล่น, ไอคอน PWA) | .42 วัดบล็อกที่นั่งจาก DOM | .43-.47 iOS viewport (--vh/--sat/status bar ทึบ) | .46 HoF โพเดียมทุกแท็บ+ฝีมือก่อน | .48-.51 สรุปคะแนนธีมใหม่→กล่องตามที่นั่ง, อิโมจิ 12 ตัว, hold 5 วิเกมสุดท้าย | .52-.53 GameOver ไม่ล้นจอ + ปุ่มแถวเดียว + landing safe-area | .54 corner index + color-scheme dark (Android) | .55-.56 (dev only) หน้าไพ่ดอกเยอะ → **ถอนออก .60** | .57-.59 หมูเจ้าหญิง avatar id3 + welcome | .60-.61 กฎ corner index ตามจำนวนใบ + ไพ่แต้มแถวเดียว | .62 ⚡ จบเร็ว | .63 ไพ่แต้มแถวตัวเอง + ชิป +20% | .64 SPADE HIGH GAMBLE | .65 rematch ready vote | .66 online rejoin + AudioContext เดียว | .67 ชื่อบอทซ้ำเติมเลข | .68 แชตบับเบิลในเกม | .69 ปุ่มกลับเข้าห้องเดิม | **.70 ปุ่มแชตกลมบนโต๊ะ**
