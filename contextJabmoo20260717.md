# จับหมู (Jab Moo) — Project Context
อัปเดตล่าสุด: 14/07/2026 · เก็บไฟล์นี้ไว้ใน repo คู่กับโค้ดเป็น backup/สรุปโปรเจกต์

## 1. ภาพรวม
เกมไพ่ไทย "จับหมู" (Hearts variant) เป็นเว็บแอปไฟล์เดียว + สมุดจดคะแนนแยกอีกไฟล์
เจ้าของ: A (Anurat) · ผู้เล่นประจำ: A, Kob, Ped, Ae (+ Gung/กุ้ง, คนที่ 5-6 หมุนเวียน)

## 2. ไฟล์ / Infrastructure
| อะไร | ที่อยู่ |
|---|---|
| Repo | https://github.com/Anurat-SUV/-Jabmoo (branch `main` = ตัว deploy) |
| เกม (live) | https://anurat-suv.github.io/-Jabmoo/jabmoo.html |
| สมุดคะแนน (live) | https://anurat-suv.github.io/-Jabmoo/scorepad.html |
| Local repo | `D:\Work\VsProject\jabmoo` (VS Code, Windows) |
| Supabase (online multiplayer) | https://kivamkangzuyjwfouusr.supabase.co — publishable key ฝังใน jabmoo.html |
| PWA | manifest.json + icon-192.png + icon-512.png (จำเป็นสำหรับเต็มจอ iPhone) |

Workflow: แก้ไฟล์ → `git add . && git commit -m "..." && git push` → รอ ~1 นาที
ถ้า push โดน rejected: `git pull --no-rebase` แล้ว push ใหม่ (ถ้า conflict: `git checkout --ours <ไฟล์>`)
เช็คว่าได้ตัวใหม่: ดูเลขเวอร์ชัน (window.__JM_VER) ใต้ปุ่มออนไลน์หน้าแรกของเกม

## 3. jabmoo.html (ตัวเกม)
Stack: ไฟล์เดียว, React 18 + Babel-standalone จาก CDN (ห้าม dynamic import — โหลดไลบรารีด้วย UMD script inject)

### กติกาคะแนน
- Q♠ = −100 (โดนหมู), J♦ = +100, 10♣ = ตัวคูณ ×2, ไม่ได้ไพ่คะแนนเลย = −50, มี slam
- "หลัก" toH(n): n≥51 → floor((n+50)/100); n≤−51 → ceil((n−50)/100); อื่น ๆ = 0
- "ได้/เสีย" calcSpread(hs): แต่ละคน = Σ(หลักตัวเอง − หลักคนอื่น)
- คะแนนสูงสุดต่อเกม +788 (มีความหมายพิเศษใน scorepad)

### ฟีเจอร์หลัก
- Portrait เป็น layout หลัก (landscape ซ่อนหลังปุ่ม 🔄)
- ธีมคาสิโน: เขียวเข้ม+ทอง, โต๊ะวงรี, avatar+วงแหวนทองตอนถึงตา, หลังไพ่แดง-ทอง, confetti จบเกม
- Online multiplayer: host-authority + Supabase Realtime broadcast, ห้องโค้ด 4 ตัว, bot เติมที่นั่งว่าง, chat ใน lobby + ระหว่างเล่น, emote ลอย, timer 25 วิ/ตา + auto-play
- เสียง (WebAudio) + haptic + PWA เต็มจอ (iPhone ต้อง Add to Home Screen; Android ใช้ปุ่ม ⛶)
- กล่อง error แดงบนจอ + ปุ่มเริ่มใหม่ (จับ crash ที่มองไม่เห็นบน iPhone)

### Bot
- Lv1/Lv2 = heuristic (นับไพ่จาก playLog, กัน slam, duck-high, ไม่งัด A♠/K♠ ตอน Q♠ ลอย, ไม่งัด A♣/K♣ ตอน 10♣ ลอย, lead ต่ำเก็บสูง)
- Lv3 = Monte Carlo (จำลองแจกไพ่ที่มองไม่เห็นตาม void + เล่นจบรอบหลายร้อยครั้ง ~700ms/ตา) — benchmark ดีกว่า Lv2 ~เท่าตัว
- Lv4 = Claude API bot (ซ่อน ไม่โชว์ใน UI)

### บั๊กสำคัญที่เคยเจอ (กันทำซ้ำ)
- Bot ค้าง: effect ตั้ง timer แล้ว cleanup ล้างทุก re-render โดยไม่ตั้งกลับ → จำ key เฉพาะเมื่อ "เล่นแล้วจริง"
- จอเขียวค้างจบ 8 เกม: hooks อยู่หลัง early return → "Rendered fewer hooks" → hooks ทุกตัวต้องอยู่บนสุดก่อน return ใด ๆ
- ไพ่หลุดใต้จอมือถือ: ใช้ 100dvh + safe-area-inset (บน iOS ต้องมี viewport-fit=cover)
- iPhone ไม่รองรับ Fullscreen API → PWA เท่านั้น

## 4. scorepad.html (สมุดคะแนน — ใช้บน iPad)
Vanilla JS ไฟล์เดียว, localStorage key `jm-scorepad-v2`, ธีมคาสิโนเข้าชุด

### โครง 1 คืน (session)
เล่นหลาย "รอบ" สลับจับหมู/สามกองได้ไม่ต้องเรียง รอบละ 4 หรือ 8 เกม (default 8) ~12 รอบ/คืน
ผู้เล่น 4 ที่นั่ง/รอบ จากทั้งหมด 5-6 คน — สะสมจับคู่ตาม "ชื่อ" (สะกดต้องเหมือนเดิมทุกรอบ)

### การกรอก
- แตะช่องคะแนน → keypad ลอย: เลือก **+/−** ก่อน → กดเลข → Enter
- จับหมู: ช่องว่างตอนบันทึก = **−50 อัตโนมัติ**
- สามกอง: จำนวนเต็มอิสระ แต่ **4 คนรวมต้อง = 0** (มีผลรวมสดเตือน)
- แตะชื่อ = แก้ชื่อ / ลากชื่อทับช่องอื่น = สลับที่นั่ง (คะแนน+K ตามไป)
- ปุ่ม K เหนือชื่อ: กด +1 ไม่จำกัด (K, 2K, 3K…) กดค้าง 600ms = ล้าง

### การแสดงผล
- ตารางรอบปัจจุบัน: ทุกเกม + แถวรวมสะสม "รวม→N" ตั้งแต่จบเกม 2 + ท้าย: รวมรอบ/หลัก/ได้เสีย (จับหมู)
- ไฮไลต์วงกลมทอง: จับหมู **+788 พอดี** / สามกอง **≥ 36** ต่อเกม
- สรุปทุกรอบ: จับหมูแสดง**เฉพาะได้/เสีย**ต่อรอบ + รวมสะสม; สามกองแสดงคะแนน; แถว K สะสม (คอลัมน์ตรงกันทุกตาราง)
- รอบที่ผ่านมา (รายเกม): ใหม่บน เก่าล่างสุด หัวรอบ "จับหมู รอบ 2 · วันเวลา" + แถว K

### สรุปจบ Session (กติกาล่าสุด)
กด 🏁 จบ Session → ส่วนลด 2 ชั้น (ปุ่ม % ลัด + กำหนดเอง):
1. d1 = % ลดยอด**สามกอง**สะสม (เก็บทศนิยม 1 ตำแหน่ง ไม่ปัด)
2. **Grand Total = จับหมู + สามกอง(หลัง d1)** — **K แยกต่างหาก ไม่รวมยอดใด ๆ** (แสดงเป็นแถวเฉย ๆ)
3. d2 = % ส่วนลดสุดท้าย → Grand หลังส่วนลด ต่อคน กติกาเศษ: **.5 พอดีคงไว้ นอกนั้นตัดทศนิยมทิ้ง** (19.7→19, −25.6→−25) ไม่มีการบังคับผลรวม 0
ลำดับแถว: รวมจับหมู → รวมสามกอง → ส่วนลดสามกอง → สามกองหลังลด → Grand Total → K(แยก) → ส่วนลดสุดท้าย(%) → Grand หลังส่วนลด

### Dashboard (🖼)
- Header "🐷 จับหมู..สร้างสรรค์ Date dd/mm/yyyy"
- มุมซ้ายบน: แท่นรางวัลครึ่งความสูง + **ตัวการ์ตูน SVG** (กุ้ง/Gung = หญิงเดรสชมพู, อื่นชาย) ท่าดีใจลดหลั่นที่ 1-3, ที่ 4-6 ยืนพื้นคอตกเศร้า, 👑 = เคย +788, ⭕×n = จำนวนเกมสามกอง ≥36
- บน: ตารางสรุป (ลำดับแถวข้างต้น) / ล่าง: ทุกรอบเป็นตารางเล็ก grid
- 📷 บันทึกเป็นรูป: html2canvas (CDN) → share sheet → download → บอกให้แคปจอ
- 💾 Export JSON: share → download → clipboard
- 🗑 Session ใหม่: ล้างข้อมูลทุกรอบ (confirm ก่อน)

## 5. วิธีเทสก่อนส่ง (สำคัญ)
- Babel transform เช็ค syntax ทั้งไฟล์เกม
- jsdom + React จริง: mount Game แล้ว fast-forward timers — เดินเกมจนจบ 8 เกม ดู error (เคยจับ bot ค้าง + hooks crash ได้)
- scorepad: jsdom เปิดจริง จำลองกดทุกฟีเจอร์
- bot: benchmark 1 ตัวใหม่ vs 3 ตัวเก่า หมุนที่นั่ง หลายร้อยรอบ

## 6. งานค้าง / แผนต่อ
- **CHANNEL_ERROR ออนไลน์**: Supabase free tier โดน pause อัตโนมัติเมื่อไม่ใช้ ~1 สัปดาห์ → เข้า supabase.com กด Restore/Resume (ยังไม่ได้ยืนยันว่าแก้แล้ว)
- **Voice chat** (WebRTC mesh 4 คน ใช้ Supabase เป็น signaling ฟรี): งานใหญ่ระดับกลาง เทสต้องใช้ 2 เครื่องจริง — milestone ถัดไป (เสียงก่อน วิดีโอทีหลัง)
- Bot จูนลึกต่อด้วย benchmark / adaptive จำสไตล์ผู้เล่น (localStorage)
