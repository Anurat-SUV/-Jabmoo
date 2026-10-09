# จับหมู (Jabmoo) — HANDOFF / CONTEXT FILE #8
> วางไฟล์นี้ใน Project knowledge (แทนไฟล์ handoff เก่า) · **ชื่อไฟล์ handoff ต้องมีเลขเวอร์ชันเสมอ `jabmoo-handoff_8-v0821.NN.md` (ชื่อซ้ำ = A เห็นแต่ไฟล์แรก)** · Claude อ่านก่อนเสมอ ตอบไทย สั้น ตรง ไม่ชม · แนบ commit message ทุกครั้งที่ส่งไฟล์
> อัปเดตล่าสุด: **prod v0821.95 / dev v0821.95-dev** (2026-10-09) — **prod = dev ทุกบรรทัด ยกเว้น 7 บรรทัด dev-only** (manifest-dev/icon-dev ×3, ชื่อแอป DEV, VER+`__JM_DEV`, ป้าย debug, COIN_ON gate) · A push เองผ่าน GitHub Desktop

## 1. โปรเจกต์คืออะไร
เกมไพ่จับหมู (Gong Zhu แบบไทย) ของ A — single-file HTML (React 18 + Babel CDN) เล่นกับ bot 3 ตัว (Monte-Carlo AI) หรือออนไลน์ มีโปรไฟล์/XP/HoF/coin ผ่าน Supabase
ศัพท์: กอง(trick), ช้วน(Slam), บิ๊กช้วน(SuperSlam 788), หลัก(toH — ขั้นเหรียญ)

## 2. ไฟล์และตำแหน่ง
| ไฟล์ | เวอร์ชัน | หมายเหตุ |
|---|---|---|
| jabmoo.html | **v0821.95** | prod — COIN_ON=true, ~1.54MB |
| jabmoo-dev.html | **v0821.95-dev** | ต่าง prod 7 บรรทัด dev-only |
| jabmoo-admin.html | - | PIN=1828 · ลบผู้เล่นผ่าน SQL Editor เท่านั้น |
- Repo: https://github.com/Anurat-SUV/-Jabmoo (main=deploy Pages) | Live: https://anurat-suv.github.io/-Jabmoo/jabmoo.html
- **A push ผ่าน GitHub Desktop** (clone ที่ `D:\Work\VsProject\jabmoo-git\-Jabmoo`) — ก๊อปไฟล์ทับ → Commit to main → Push origin
- **กฎเหล็ก: แก้ทั้งสองไฟล์ IN PLACE เสมอ ห้าม regen dev จาก prod** — python str.replace + `assert count==1` ทุก anchor · ตรวจ `diff prod dev | grep -c '^<'` = 7 ก่อนส่งทุกครั้ง
- export ไป /mnt/user-data/outputs/ เสมอ | **เปิดแชทใหม่ upload ทั้ง 2 ไฟล์ล่าสุด** (sandbox รีเซ็ตได้กลางแชท — ไฟล์ใน outputs/ รอด แต่ /home/claude หาย)
- **bump เลขเวอร์ชันทุกครั้งที่ส่งไฟล์** แม้แก้เล็ก · **bump handoff เมื่อ A สั่ง** (A ปฏิเสธ handoff อัตโนมัติ 2 ครั้งใน .87-.93 — ถามก่อน)
- cache บนมือถือ: ถ้าเวอร์ชันมุมจอไม่ขยับหลัง push → เปิด URL เติม `?v=NN` เช็กว่า deploy แล้ว · PWA iOS ปิดแอปเปิดใหม่ 1-2 รอบ · "ลบข้อมูลเว็บไซต์" ใน Safari จะล้าง localStorage ด้วย (กู้ได้ด้วยรหัส 4 ตัว)

## 3. Supabase
- URL: https://kivamkangzuyjwfouusr.supabase.co | Key (publishable): sb_publishable_e4N4h_EupJMRg9rC3R8CGw_kgXZH7bI | RLS: select/insert/update เปิด, DELETE ปิด
- `players`: code(PK4), name, xp, games, jd_won/seen, qs_dodged/seen, slams, superslams, quits, wins, rank_sum, rk_games, pig_id, coins(cache) — **(.87) profLeaderboard select เพิ่ม `quits`**
- `coin_tx` (ledger append-only = ความจริง) + `coin_match` (audit) · `game_config` (key='bots' → `{"chars":{...},"lineup":{...}}` โหลดครั้งเดียว fail-open)

## 4. ระบบ COIN
- GRANT 10,000 | หมด(<500) → เติมถึง 2,000/วัน เที่ยงคืนไทย | ต๋ง 15% ฝั่งได้→HOUSE | ledger-first | บัญชี BOT+HOUSE
- หลัก(toH): |n|≤50→0, ~ทุก100ขยับ1 | จ่าย=(4×หลักตัวเอง−รวมโต๊ะ)×mult | โต๊ะ 7 ระดับ ×100..×10K (ปกติ bot ≤×500, min=50×mult)
- **เงินบอทข้างที่นั่งเป็น cosmetic** (localStorage seed 10,000, floor 5,000 top-up) — settle จริงหักบัญชี `BOT` ใน ledger ติดลบได้ไม่จำกัด → จ่ายคนเต็มเสมอ · TODO: seed ตามโต๊ะ (50×mult) ถ้าอยากให้ดูสมจริงที่ ×2K
- **🎂 BIRTHDAY PROMO (.90-.93)**: `const PROMO={mults:[1000,2000],min:50000,until:Date.parse("2026-10-20T23:59:59+07:00")}` · `promoOn()` / `promoTbl(t)` / `tblBotOK(t)` / `tblMin(t)` = gate กลางที่ชิป landing, `start()`, `lastTblMult`, ห้องออนไลน์ ใช้ร่วมกัน · ช่วงโปร ×1K และ ×2K เล่นกับบอทได้ ขั้นต่ำ 50,000 ชิปติด 🎂 · แบนเนอร์เหลืองเหนือแถวชิป + บรรทัดในห้องออนไลน์ · **หมดเวลาแล้วกลับเป็น 👥 คนล้วนเอง ไม่ต้อง push** (ยืดโปร = แก้ `until` บรรทัดเดียว)

## 5. BOT — ตัวละคร + อาวุธ
`BOT_CHARS` knobs: dealsX, blur, hunt, campaign+huntP, antiLast, blowGuard, bayes, slamDef, hlak, spGamble, pigLead, **tcP (.95)**, **slamKeep (.95, OFF)** — override ทาง game_config
| ตัว | dealsX | blur | อาวุธ | tcP [rush,wait,suspect] | โต๊ะ |
|---|---|---|---|---|---|
| หมูเด้ง | 0.5 | 35% | — | .8/.15/.05 | ×100 |
| หมูทอด | 0.75 | 15% | — | .8/.15/.05 | ×100/200 |
| หมูป่า | 1.0 | 5% | hunt, blowGuard | .5/.3/.2 | ×200/500 |
| หมูเทพ | 1.2 | 0 | hunt, blowGuard | .35/.35/.3 | ×500 + SEAT TAKEOVER |
| เทพจับหมู | 1.5 | 0 | + bayes Q♠ + campaign gated + SLAM DEFENSE + hlak(OFF) | .25/.35/.4 | ×500 boss |
- lineup ×100=[เด้ง,ทอด,เด้ง] ×200=[ทอด,ป่า,ทอด] ×500=[ป่า,เทพ,เทพจับหมู] (โต๊ะโปร ×1K/×2K ใช้ lineup ×500 + budget ×500) · mcBudget ×100=600ms/110 ×200=1100/300 ×500+=1800/600 · min turn 0.68s
- **SPADE HIGH GAMBLE (.64)** `spGamble` default 0.5 — **bench แล้ว (2026-09-21) +4.7±5.7 n=150 = กลาง** กฎทำงาน ~1 ครั้ง/2 เกม
- **PIG TOP GUARD + VOID SEED (.86)** bench +1.4±5.0 กลาง · `pigLead:0` ปิด
- **10♣ PATIENCE (.95, A 2026-10-09)** — ที่มา: วัด self-play 150 เกม 10♣ ออกกองดอกจิกที่ 1-2 **61%** (30%+31%) เพราะ MC ทิ้งทันทีที่ "ปลอดภัย" (มีใบสูงกว่า 10 ในกอง) ไม่รอเป้า → A กับ Kob จับทาง: ถือ Q♠ + โพดำ 3-4 ใบ นำดอกจิกต่ำล่อ 10♣ ออกก่อน / ไม่ลงเหนือ 10 ใน 2 กองแรก · แก้ = **นิสัยสุ่มต่อเกมต่อบอท** (hash: ใบแรกที่ลงในเกม + totals + ที่นั่ง → sticky ทั้งเกม) 3 โหมด: `rush` = เดิม · `wait` = ทิ้งเมื่อคนกินกองมีแผล (pile รอบ ≤ −50 หรือ totals ≤ −150) หรือเสี่ยงติดมือ (ดอกจิกคุ้ม ≤2 / ดอกจิกออก ≥7 / pile ตัวเอง ≤ −100) · `suspect` = wait + จำคนที่นำดอกจิก <10 ในกองดอกจิกแรก (คนล่อ = น่าจะถือ Q♠) เก็บ 10♣ ไว้ให้เขาตราบที่เขายังตามดอกจิกได้ · เป็น MC prune ก่อน rollout · **bench paired n=150 (deals 30/80ms): wait −0.7±5.5 · suspect +4.7±6.6 · mix จริง −2.0±6.4 = กลาง ไม่ถดถอย** · histogram หลังเปิด (น้ำหนักจริง): กองดอกจิกที่ 1-2 จาก 61% → **45%** (22%+23%), ทิ้งนอกดอก 34% — เหลือความเสี่ยงแบบ "มีใบสูงกว่าในกองแล้วโยน" 33% (จาก 40%)
- **SLAM KEEP package (.95) — ใส่โค้ดแล้วแต่ OFF** ที่มา: A เห็นบอทกันช้วนด้วยการไม่ให้โพแดง (ถูก) แต่ทิ้ง A♣ เร็วบนกองสะอาด → K♣ ของคนกลายเป็นใบคุมดอกอัตโนมัติ · 3 ส่วน: (ก) MC prune `opts.slamKeep`: ขณะ slamOn ตามดอกนอกโพแดงบนกองสะอาด ถ้าผู้ต้องสงสัยยังตามดอกนั้นได้ → ตัด A/K ออก (มีใบต่ำ) · กองที่มีโพแดง หรือกองใบพิเศษที่ผู้ต้องสงสัยกำลังจะกินตอน 788 ยัง live → บังคับกินด้วยใบชนะต่ำสุด (ข) heuristic `SLAM_KEEP_H`: เลิกทิ้ง A/K ตำแหน่งสุดท้ายขณะ slamThreat ถ้าผู้ต้องสงสัยยังตามได้ (ค) `SLAM_TELL_EARLY` ใน slamRisk: คนเดียวกินทุกกอง ≥3 กอง + โพแดงยังไม่ออกเลย → r=0.55 · **ผล bench: self-play n=100 −7.3±7.5 · bench มือช้วน (scripted slammer ถือ ♥≥7 รวม Q/K/A + A/K ≥2, ฝ่ายกัน thepjab) อัตราช้วนสำเร็จ 29% ทั้งเปิด/ปิด** · diag: A♣/K♣ ถูกทิ้งตั้งแต่ตา 1-4 ก่อน risk read จะขึ้น (ส่วนใหญ่ "norisk clean hadLower") แม้เพิ่ม tell แล้วก็ทันแค่บางเคส · **เปิดทดลองจริง: `{"chars":{"thepjab":{"slamKeep":true}}}` (ส่วน MC) / const 2 ตัวต้องแก้ไฟล์** · ช้วนธรรมดา vs บิ๊กช้วน: หลักเดียวกัน แต่บิ๊กช้วนให้ใบคุมเล็งกองใบพิเศษก่อน (ใบเดียวจบ 788)
- heuristic เดิมครบ (Threat/EARLY HEART/788 containment/PIG EXIT/PROFIT GUARD/J♦ BAN/jdBoss/COVER GUARD/WOUNDED LEAD BAN/J♦ FOLLOW/J♦ COVER 50/50) · blowGuard / bayes / CAMPAIGN / SLAM DEFENSE (.33) / hlak(OFF) ไม่แตะ
- **สิ่งที่บอท "ไม่มี" (อธิบาย A 2026-09-21)**: ไม่มีขั้นอ่านมือทั้งใบ/ตั้งเป้าคะแนน/วางแผนข้ามกอง — MC มอง 1 กองลึก ที่เหลือ heuristic เล่นแทน; แผนที่คุ้ม 5-10 แต้มต่อกองมองไม่เห็น (noise ±10-15) ต้องใส่เป็น prune · A จะส่ง log เกมที่เล่นดีมาให้แปลงเป็นกฎทีหลัง (ยังไม่ส่ง)
- **กฎที่ A บอกแล้วยังไม่ทำ**: ถือดอกจิก ≥5 ใบ มี 2-3 ตัวต่ำคุ้ม → เก็บ 10♣ รอคนลบหนัก (ใกล้เคียง wait แต่ยังไม่ถ่วงน้ำหนัก "ใบคุ้ม")

## 6. UI — สถานะปัจจุบัน
**Landing**: hero `JM_HERO2` · ชิป 🪙 ขวาบน **+20% (.89)** · เมนู 6 การ์ด · **แบนเนอร์โปร 🎂 (.91)** · ชิป 7 โต๊ะ (promo ติด 🎂) · แถวตั้งค่า 3 การ์ด · การ์ด VS BOT/ONLINE
**Hall of Fame (.87/.94)**: 3 แท็บ ฝีมือ/XP/เศรษฐี — **ลิสต์แนวตั้งธรรมดา ไม่มีโพเดียม** อันดับ 1-3 เน้น (ป้ายเลขสีทอง/เงิน/ทองแดง ขอบสี avatar ใหญ่ 👑 หน้าชื่อที่ 1) · แท็บฝีมือโชว์ 🏃 จำนวนหนีเกม · **(.94) ตัดกล่อง "คุณ" ด้านล่างออกทั้ง 3 แท็บ · กล่อง "ความสำเร็จระดับตำนาน" ย้ายไปท้ายหน้ากติกา (ใต้ภาพ poster)**
**⚡ จบเร็ว (.62/.75/.91)**: โผล่เฉพาะไพ่แต้มครบ 16 ใบ · **(.88-.91) เคยเพิ่มโหมด cantWinAny (ไพ่ตัวเองชนะกองไม่ได้แล้ว) ทั้งแบบปุ่ม ◀/▶ และแบบ ⚡ → A ถอนทั้งหมดใน .91: "ควรหัดนับไพ่เอง รู้ก่อนความตื่นเต้นหาย"** — อย่าเสนออีก
**In-game portrait/landscape, แชต typing mode, countdown, หน้าจบเกม, copy log, viewport/iOS (--vh/--sat/--sal)**: ไม่เปลี่ยนจาก .86 — ดูรายละเอียดใน handoff .86 ถ้าต้องแตะ (สรุป: TABLE_BG ผ้าโต๊ะ A, 4 ที่นั่งบนราว, ไพ่ชิดขอบวัดจาก DOM, ไพ่ 12-13 ใบ corner index, แผงแชตบนผ้า+typing mode ย่อผ้า 60% root สูง typingHRef, countdown 10→0 บี๊บ, GameOver แนวนอน 2 คอลัมน์, 📋 copy 2 เกมล่าสุด)
**XP ต่อเกม** (ตอบ A 2026-10-01): ฐาน 20 · อันดับ 40/25/12/5 · J♦ +15 (+15 ถ้าเก็บนอกดอก) · คะแนน ≥0 +10 · ไม่โดน Q♠ +5 · ช้วน +60 / 788 +150 (+30 heartOff) · จบแมตช์ ≥8 เกม +30, 4-7 เกม +15 · Lv: 150/400/800/1.4K/3K/5K/7.5K/10.5K/14K/18.5K/24K/31K/40K/50K

## 7. ออนไลน์ / AUDIO
ไม่เปลี่ยนจาก .86 (transport reconnect, AUTO-REJOIN .84, REMATCH READY VOTE, แชตบับเบิล, jmAC เดียว) — ดู handoff .86

## 8. TODO
1. ทดสอบจริง .71-.95 บนมือถือ: countdown/บี๊บ · landscape · --sal · หน้าจบเกมแนวนอน · typing mode iOS · HoF ใหม่ · โปรโต๊ะ ×1K/×2K
2. ตัดสินใจ single-file vs `/img/` (~1.54MB)
3. ทดสอบ 2 เครื่อง: rematch หลัง ⚡ (.75) + rejoin (.84 ต้อง push prod ให้ host) + ready vote + แชต
4. **หลัง 20 ต.ค.**: โปรหมดเอง — ถ้าอยากต่อ แก้ `PROMO.until`
5. บอท: ดูผลจริง 10♣ PATIENCE จาก log ของ A/Kob · ลอง `slamKeep:true` ทาง game_config แล้วดู log ช้วน · hlak 0.3 · กฎ "ดอกจิกคุ้ม ≥5 ใบเก็บ 10♣" · seed เงินบอทตามโต๊ะ
6. (พักไว้) บอทเรียนรู้จาก play_log · Bayesian ขยาย · รูปหมูตัวที่ 5 · ลงโทษ host หนี · Elo · สามกอง (project แยก)

## 9. บทเรียน bench + เครื่องมือ
- Paired self-play se ~5-7 ที่ n=150 deals30/80ms (~11 นาที/รัน, sandbox 2 core → รันขนานเกิน 2 ช้า) · เห็นเฉพาะ effect >±12 · **bench วัดการโดนคนจับทางไม่ได้** — กฎกัน exploit อาจออกกลาง/ลบเล็กน้อยใน self-play แต่ยังควรทำถ้า A ยืนยันจากโต๊ะจริง (10♣ PATIENCE ผ่านเพราะกลาง; SLAM KEEP ปิดไว้เพราะไม่เห็นผลแม้ใน bench ช้วน)
- เครื่องมือ (สร้างใหม่ทุกแชทจากคำอธิบาย ~5 นาที): `extract.js` (babel transform script → engine_raw.js) → `load.js` (node vm + stub window/document/React/localStorage, ตัด createRoot, export `{gReduce,botPlay,botPlayMC,botChar,mkRound,slamRisk,slammerPolicy,...}`) · **hook ใน vm ต้อง set ผ่านฟังก์ชันที่ export จากใน vm** (`setSK:(v)=>{globalThis.__SK=v}`) — `globalThis` ของ node กับของ vm คนละตัว (บั๊กที่ทำให้ bench sk รอบแรกเทียบ on กับ on) · `bench.js <variant> N DEALS MS` paired aa · `slambench.js` scripted slammer (seat 0 slammerPolicy บน deal ที่ ♥≥7) วัด slam% · `tc.js heur|mc N` histogram ว่า 10♣ ออกกองดอกจิกที่เท่าไหร่ · `repro_gN.js` สร้าง S จาก log ผ่าน gReduce แล้วเรียก botPlayMC + `dest_*.js` วิ่ง rollout เองนับว่าไพ่สำคัญไปจบที่ใคร (ใช้ตอบ "ทำไมลง X")
- **playwright**: copy react/react-dom/babel จาก node_modules ไป `shot/` แทน CDN, serve `cd shot && setsid nohup python3 -m http.server 8766 &` (ต้องเช็ก `curl -s -o /dev/null -w "%{http_code}"` ก่อน — server ตายเมื่อ sandbox รีเซ็ต) · patch สำเนา shot/: `needName?<WelcomeModal` → `false?`, stub `profLeaderboard`/`coinRichList`/`coinBalance` คืนข้อมูลปลอม, `useState(60000)` แทน coins, expose `window.__sRef/__ap/__rd` ไว้ยัด state กลางเกม + เล่นแทนคน · **ห้ามใช้ `pkill -f` กับ pattern ที่ match เชลล์ตัวเอง** (ฆ่า shell ของ Claude เอง exit 144)
- Setup: `npm i @babel/core @babel/preset-react @babel/standalone@7.23.5 react@18 react-dom@18` · pip playwright/PIL มีแล้ว · คำสั่งเดียวจำกัด 2 นาที (background ด้วย setsid nohup + poll)

## 10. วิธีทำงาน
A ส่ง log/ภาพ/ไอเดีย → reproduce → แก้ → regression (เล่นจริงผ่าน playwright) → bench ถ้าเป็นพฤติกรรมบอท → bump → export ทั้ง 2 ไฟล์ + commit message → A push
- **วิเคราะห์ log ไพ่ (A สั่ง 2026-09-21)**: จากข้อมูลที่ผู้เล่นคนนั้นเห็นได้จริง ณ ตอนนั้นเท่านั้น · ห้ามตอบ "ผลเท่ากัน" · reproduce ด้วย engine แล้วอธิบายด้วยตัวเลข (ตัวอย่างที่ A พอใจ: G1 T10 J♦ = เลือกคนรับ J♦, G4 T2 K♠ = MC −66 vs −89 เพราะ 7♠ ก็หนีไม่พ้น, G8 T6 นำ J♦ = ตายอยู่แล้ว 4% เท่านั้นที่จะได้ แต่ MC มองไม่เห็นว่าคนจะทิ้ง 10♣ ทับ)
- A ชอบ preview ภาพก่อนตัดสินใจ UI — ทำ screenshot แล้ว **ส่งไฟล์ภาพ** ด้วย
- A ไม่อยากให้บอท "ทำทุกครั้ง" — ชอบสุ่ม/นิสัยให้เดาทางไม่ถูก (spGamble, tcP เป็นต้นแบบ)
- **แชทยาว: ใกล้ตันสร้าง handoff ใหม่ทันที** (แต่ A สั่งเองเมื่อไหร่ค่อยทำ)

## 11. ประวัติ .87-.95
**.87** HoF ลิสต์แนวตั้ง + 🏃 หนีเกม | **.88** NO-WIN AUTO ◀/▶ (ถอน .90) | **.89** ชิป coin landing +20% | **.90** ⚡ cantWinAny (ถอน .91) + 🎂 promo ×2K | **.91** ถอน cantWinAny + แบนเนอร์โปร | **.92** promo min 50,000 | **.93** promo ×1K+×2K | **.94** HoF ตัดกล่อง "คุณ" + legend achievements → หน้ากติกา | **.95** 10♣ PATIENCE (ON, bench กลาง) + SLAM KEEP (OFF, remote knob)
