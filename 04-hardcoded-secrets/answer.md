# ข้อ 4 — พบ API Key / URL ของ 3rd-party ถูก Hardcode ไว้ใน Source Code

> **โจทย์:** จะอธิบายผลกระทบให้คนที่ทำฟังอย่างไร และจะให้เขาแก้วิธีเก็บ Key อย่างไรให้ถูกต้อง

---

## 1. วิธีคุย: ไม่ตำหนิ แต่ทำให้เห็นผลกระทบที่จับต้องได้

**บรรยากาศ**
- คุยส่วนตัว หรือคอมเมนต์ใน code review อย่างสุภาพ — *"เรื่องนี้เจอบ่อยมากในหลายบริษัท เดี๋ยวเรามาแก้ด้วยกัน"* (blameless)
- เป้าหมายคือให้เขา **เข้าใจเหตุผล** จนไม่ทำอีกเอง ไม่ใช่แค่ทำตามเพราะถูกสั่ง

**อุปมาให้เห็นภาพ**
> การเขียน API Key ไว้ในโค้ด เหมือน **เขียนรหัสตู้เซฟไว้บนประตูตู้เซฟ** แล้วถ่ายสำเนาแจกทุกคนที่เข้าถึง repo
> และ **สำเนาเก่าทุกฉบับถูกเก็บไว้ตลอดไป** ใน Git history แม้วันนี้จะลบออกจากไฟล์แล้วก็ตาม

**ทำให้เห็นจริง (5 นาที)**
- เปิด Git history ให้ดู: `git log -p -S "<ส่วนต้นของ key>"` → key ยังอยู่ใน commit เก่า แม้ลบออกจากไฟล์ล่าสุดแล้ว
- ไล่ดูด้วยกันว่า **ใครเห็นโค้ดนี้ได้บ้าง**: ทุกคนที่ clone repo, outsource, อดีตพนักงาน, CI log, เครื่อง laptop ที่อาจหาย, screenshot ที่แชร์ในแชต
- ถ้า repo หลุดเป็น public แม้ช่วงสั้น ๆ มี bot สแกนหา key บน GitHub ตลอดเวลา และนำไปใช้ได้ภายในไม่กี่นาที

**ถามให้เขาคิดเอง** แทนการบรรยายอย่างเดียว
- *"ถ้าพรุ่งนี้ vendor บังคับเปลี่ยน key เราต้องทำอะไรบ้างถึงจะใช้ key ใหม่ได้?"*
- *"ถ้าเราทดสอบบนเครื่อง dev ตอนนี้ ระบบไปเรียก endpoint ไหน — sandbox หรือ production จริง?"*
- *"ถ้ามีคนเอา key นี้ไปใช้ เราจะรู้ได้ยังไง และรู้ได้ไหมว่าเป็นใคร?"*

---

## 2. ผลกระทบที่อธิบายให้ฟัง

| ด้าน | ผลกระทบ | ตัวอย่างที่เกิดขึ้นได้จริง |
|---|---|---|
| **Security** | ใครมีโค้ด = มีสิทธิ์ใช้ key ในนามบริษัท | เรียก Payment / SMS / Maps / AI API ด้วยสิทธิ์ของเรา, อ่านหรือแก้ข้อมูลลูกค้าที่ key นั้นเข้าถึงได้ |
| **การเงิน** | ถูกใช้โควต้าหรือเครดิตจนบิลพุ่ง | key ของ Cloud / AI / SMS ถูกขโมยไปใช้ บิลหลักแสนในคืนเดียว |
| **กฎหมาย** | ถ้า key เข้าถึงข้อมูลส่วนบุคคลได้ = ข้อมูลรั่ว | PDPA: ต้องแจ้ง สคส. ภายใน 72 ชั่วโมง, ค่าปรับทางปกครองสูงสุด 5 ล้านบาท และอาจมีค่าเสียหายทางแพ่ง |
| **ชื่อเสียง** | ลูกค้าและคู่ค้าเสียความเชื่อมั่น | ข่าวข้อมูลรั่วอยู่ได้นานกว่าตัว incident |
| **Operation** | เปลี่ยน key หรือ URL = แก้โค้ด + build + deploy ใหม่ | key หมดอายุตอนตีสอง ต้อง hotfix ทั้งระบบ |
| **Environment** | dev / staging / production ใช้ key และ URL ชุดเดียวกัน | ทดสอบบนเครื่อง dev แล้วตัดเงินลูกค้าจริง หรือส่ง SMS หาลูกค้าจริง |
| **Traceability** | ทุกคนใช้ key ตัวเดียวกัน | เกิดเหตุแล้วไม่รู้ว่าใคร / ระบบไหนเป็นคนเรียก |
| **Audit / Compliance** | ตรวจไม่ผ่านมาตรฐาน | ISO 27001, PCI DSS, SOC 2 กำหนดให้ควบคุมการเข้าถึงและจัดการ secret อย่างเป็นระบบ |
| **Frontend / Mobile** | key ที่อยู่ใน JavaScript หรือในแอปถูกแกะออกได้เสมอ | เปิด DevTools หรือ decompile APK/IPA ก็เห็น key |

**API Key กับ URL ต่างกันอย่างไร**
- **API Key / Token / Password = secret** → ต้องเก็บแบบลับ และหมุนเวียน (rotate) ได้
- **URL = config** ไม่ใช่ความลับ แต่ hardcode แล้วสลับ environment ไม่ได้ และเปลี่ยน vendor endpoint ต้อง deploy ใหม่ → ต้องย้ายไป config ตาม environment
- ยกเว้น URL ที่ **มี token อยู่ในตัว** (เช่น webhook URL ที่มี secret ใน path / query string) → ถือเป็น secret

---

## 3. วิธีแก้ — ทำตามลำดับ

### Step 1 — Contain: ถือว่า key รั่วแล้ว (ทำภายในวันนั้น)

1. **Rotate / Revoke key ที่ provider ทันที** แล้วออก key ใหม่
   > ลบออกจากโค้ดอย่างเดียว **ไม่พอ** เพราะ key เดิมยังใช้งานได้ และยังอยู่ใน Git history
2. ตรวจ **usage log ของ provider ย้อนหลัง** หาการเรียกที่ผิดปกติ (ปริมาณ, IP, เวลา)
3. ถ้า key เข้าถึงข้อมูลส่วนบุคคลได้ → แจ้งทีม Security / DPO ประเมินตามขั้นตอน PDPA

### Step 2 — Fix: แยก config ออกจากโค้ด

หลัก **12-Factor App (Config)**: โค้ดชุดเดียว deploy ได้ทุก environment โดยเปลี่ยนแค่ config ที่ inject เข้ามาตอน runtime

| ประเภท | Local (เครื่อง developer) | Server (dev / staging / prod) |
|---|---|---|
| **Secret** (API key, token, password) | `.env` ที่อยู่ใน `.gitignore` หรือ .NET User Secrets | **Secret Manager**: AWS Secrets Manager / Azure Key Vault / GCP Secret Manager / HashiCorp Vault แล้ว inject เป็น env var ตอน runtime |
| **Config ที่ไม่ลับ** (URL, timeout) | ไฟล์ config ของ environment | `appsettings.{Environment}.json`, env var, ConfigMap |

**ตัวอย่าง .NET**

```csharp
// ❌ ก่อนแก้
var client = new PaymentClient("https://api.vendor.com/v1", "sk_live_51H8xxxxxxxx");
```

```csharp
// ✅ หลังแก้: อ่านจาก configuration — โค้ดไม่รู้ว่าค่ามาจากไหน
public sealed class PaymentOptions
{
    public string BaseUrl { get; init; } = "";
    public string ApiKey  { get; init; } = "";
}

builder.Services.Configure<PaymentOptions>(builder.Configuration.GetSection("Payment"));

// Production: ดึง secret จาก Azure Key Vault
// ชื่อ secret "Payment--ApiKey" ใน Key Vault จะถูก map เป็น Payment:ApiKey ให้อัตโนมัติ
if (builder.Environment.IsProduction())
{
    builder.Configuration.AddAzureKeyVault(
        new Uri(builder.Configuration["KeyVault:Uri"]!),
        new DefaultAzureCredential());   // ใช้ Managed Identity ไม่ต้องมี password ในโค้ด
}
```

```jsonc
// appsettings.Production.json — มีแค่ config ที่ไม่ลับ
{
  "Payment": { "BaseUrl": "https://api.vendor.com/v1" },
  "KeyVault": { "Uri": "https://myapp-prod-kv.vault.azure.net/" }
}
```

```bash
# เครื่อง developer: เก็บ key ทดสอบนอก repo
dotnet user-secrets set "Payment:ApiKey" "sk_test_xxxxxxxx"
```

**ตัวอย่าง Node.js / Next.js**

```ts
// ❌ ก่อนแก้
await fetch('https://api.sms-vendor.com/v1/send', {
  headers: { 'x-api-key': 'a1b2c3d4e5f6...' },
});

// ✅ หลังแก้
await fetch(`${process.env.SMS_API_URL}/send`, {
  headers: { 'x-api-key': process.env.SMS_API_KEY! },
});
```

```bash
# .env.example — commit ไฟล์นี้ (ไม่มีค่าจริง) ให้ทีมรู้ว่าต้องตั้งตัวแปรอะไร
SMS_API_URL=
SMS_API_KEY=
```

```gitignore
# .gitignore — ไฟล์ที่มีค่าจริงห้าม commit
.env
.env.*
!.env.example
```

> ⚠️ **Next.js:** ตัวแปรที่ขึ้นต้นด้วย `NEXT_PUBLIC_` จะถูกฝังลงใน JavaScript ที่ส่งไปให้ browser — **ห้ามใส่ secret** ต้องเรียก 3rd-party จากฝั่ง server (Route Handler / API) เท่านั้น
>
> ⚠️ **Mobile App:** ห้ามฝัง secret ของ 3rd-party ไว้ในแอป ให้แอปเรียก backend ของเรา (BFF) แล้ว backend ถือ key และเรียก 3rd-party แทน

**บน Kubernetes / CI/CD**
- ใช้ External Secrets Operator (หรือ CSI Secrets Store) ดึงจาก Secret Manager มาเป็น Kubernetes Secret แล้ว inject เป็น env var
- Pipeline ใช้ GitHub Actions Secrets หรือ OIDC เชื่อมกับ Cloud โดยไม่ต้องเก็บ long-lived key

### Step 3 — Git history: rotate สำคัญกว่าการลบ

- เมื่อ rotate แล้ว key เก่าใช้ไม่ได้ → ค่าใน history ไม่เป็นอันตรายแล้ว
- ถ้านโยบายบริษัทต้องลบออกจาก history ด้วย: ใช้ `git filter-repo` หรือ BFG Repo-Cleaner แล้ว force push — ต้องประสานทั้งทีม (ทุกคน clone ใหม่) และ **ไม่ช่วยถ้ามีคน clone หรือ fork ไปแล้ว**
- สรุป: **ลบ history = ทางเลือก, rotate = บังคับ**

### Step 4 — Prevent: กันไม่ให้เกิดซ้ำ

| ระดับ | เครื่องมือ / วิธี |
|---|---|
| เครื่อง developer | pre-commit hook ด้วย **gitleaks** หรือ detect-secrets — สแกนก่อน commit ถ้าเจอ secret จะ commit ไม่ได้ |
| Repository | เปิด **GitHub Secret Scanning + Push Protection** — block การ push ที่มี secret |
| CI pipeline | รัน gitleaks ทุก Pull Request, เจอ secret = build fail |
| Code review | เพิ่มข้อ "มี secret / URL hardcode ไหม" ใน checklist ของ PR |
| ฝั่ง provider | key แยกต่อ environment, สิทธิ์น้อยที่สุดเท่าที่จำเป็น (least privilege), IP allowlist, ตั้ง quota / budget alert |
| กระบวนการ | rotation schedule (เช่น ทุก 90 วัน) — ทำได้ง่ายเพราะไม่ต้อง deploy ใหม่แล้ว |
| ทีม | เขียน guideline สั้น ๆ เรื่องการจัดการ secret + แชร์ 15 นาทีใน team meeting |

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: vX.Y.Z   # ใช้ tag ล่าสุดของ gitleaks
    hooks:
      - id: gitleaks
```

---

## 4. Checklist ที่ให้ developer ทำ

- [ ] Rotate / revoke key เดิมที่ provider และออก key ใหม่ แยกต่อ environment
- [ ] ตรวจ usage log ย้อนหลัง รายงานถ้าพบสิ่งผิดปกติ
- [ ] ย้าย key ไป Secret Manager / User Secrets และย้าย URL ไป config ตาม environment
- [ ] `.env` อยู่ใน `.gitignore` และมี `.env.example`
- [ ] ค้นทั้ง repo ว่ามี secret ตัวอื่นอีกไหม (`gitleaks git .` สแกนทั้ง history)
- [ ] ติดตั้ง pre-commit hook และเปิด secret scanning ใน CI
- [ ] Deploy ทดสอบทุก environment ด้วย config ใหม่ และยืนยันว่า dev เรียก sandbox ไม่ใช่ production

---

## สรุปสั้น

1. **ถือว่ารั่วแล้ว → Rotate ทันที** (ลบจากโค้ดอย่างเดียวไม่พอ)
2. **Secret ไป Secret Manager, URL ไป config ตาม environment** — โค้ดชุดเดียว deploy ได้ทุกที่
3. **กันซ้ำด้วยเครื่องมืออัตโนมัติ** (gitleaks + push protection + CI) ไม่พึ่งความจำของคน
4. **คุยแบบไม่ตำหนิ** ให้เห็นผลกระทบจริง เพื่อให้เขาเข้าใจและทำถูกเองในครั้งต่อไป
