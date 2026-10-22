# OpenWrt router par AdBlock-Fast setup: kam memory me poore ghar ke liye ad blocking

*(English summary neeche hai / English summary below)*

Is guide se aap apne **OpenWrt router** par **AdBlock-Fast** lagaoge. Ye router ke DNS se hi ads, trackers aur malware ke domains block kar deta hai, to ghar ke **saare devices** (phone, laptop, TV) par ek saath kaam karta hai. Koi app nahi lagani padti.

Ye guide un logon ke liye bhi hai jo **AdGuard Home** nahi chalana chahte, kyunki wo router par zyada memory leta hai.

> **Last tested:** October 2026. JioRouter AX6000 (JIDU6401), OpenWrt 25.12.6, `adblock-fast` 1.2.4-r4, `luci-app-adblock-fast`. Doosre router ya version me menu ke naam thode alag ho sakte hain.
>
> Ye ek unofficial community guide hai. Apne risk par use karo, aur pehle config ka backup lo.

---

## TL;DR: 8 line me poora kaam

1. Router par **password** aur **internet** theek ho (Step 0).
2. `adblock-fast`, `luci-app-adblock-fast` aur 4 helper packages install karo (Step 1).
3. **Services → AdBlock-Fast** kholo (Step 2).
4. **Sirf 1-2 blocklist** chuno. Hagezi Pro kaafi hai (Step 3).
5. Basic settings jaisi hain waisi rakho, **Save & Apply** (Step 4).
6. **Enable** aur **Start** dabao (Step 5).
7. Router aur phone dono se **test** karo (Step 6).
8. **Memory** check karo aur **reboot test** karo (Step 7, 8).

---

## Ye kaise kaam karta hai

```
Aapka phone / laptop / TV
        |
        |  "ads.example.com kahan hai?" (DNS sawal)
        v
   Router ka DNS (dnsmasq + AdBlock-Fast ki blocklist)
        |
        +-- Domain blocklist me hai  ->  jawab: "nahi mila" (NXDOMAIN)  -> ad nahi aata
        |
        +-- Blocklist me nahi hai    ->  asli jawab (internet se)       -> site normal khulti hai
```

AdBlock-Fast alag se koi bada DNS server nahi chalata. Ye blocklist ko router ke **apne dnsmasq** (jo pehle se chal raha hota hai) me daal deta hai. Isi liye ye halka rehta hai.

---

## AdBlock-Fast ya AdGuard Home? (seedhi tulna)

| | **AdBlock-Fast** | **AdGuard Home** |
|---|---|---|
| Kaise chalta hai | Router ke dnsmasq me blocklist daalta hai | Alag program, apna DNS server |
| Memory | Kam. Is guide ke tester ne ~20-25 MB dekha (apna router par neeche Step 7 se naap lo) | Aam taur par zyada, kyunki wo alag program hai |
| Setup | Aasan, LuCI me hi | Alag install aur alag web panel |
| Har device ke alag rules | Nahi | Haan |
| Query statistics / dashboard | Basic | Bahut achha |
| Upstream DoH/DoT | Nahi (router ka apna DNS) | Haan |

**Seedha matlab:** agar aapko sirf "ads block ho jayein aur router halka rahe" chahiye, to AdBlock-Fast theek hai. Agar aapko har bachhe/device ke alag rules, bade statistics ya encrypted DNS chahiye, to AdGuard Home ke fayde hain. Dono ek saath mat chalana, wo ek doosre se takraate hain.

> Memory ke numbers aapke router, blocklist ki size aur uske RAM par depend karte hain. Upar wala "~20-25 MB" ek tester ka naap hai, guarantee nahi. Apna Step 7 me khud check karo.

---

## Zaroori shabdon ka matlab

| Shabd | Matlab |
|---|---|
| **DNS** | Internet ki "phonebook". Website ka naam ko uske address me badalta hai |
| **dnsmasq** | OpenWrt ka built-in chhota DNS/DHCP program |
| **Blocklist** | Un domains ki list jinhe block karna hai (ads, trackers, malware) |
| **NXDOMAIN** | DNS ka jawab "ye naam hai hi nahi". Block hone par yahi aata hai |
| **Allowlist** | Un domains ki list jinhe kabhi block nahi karna |
| **LuCI** | Router ka web panel (`http://192.168.1.1`) |
| **SSH** | Router me commands type karne ka tareeka |

---

## Kya chahiye (checklist)

- [ ] Router par **OpenWrt 25.12** chal raha ho
- [ ] Router ko **internet** mil raha ho
- [ ] Router par **password set** ho
- [ ] **AdGuard Home** ya koi aur ad blocker router par na chal raha ho (ek hi chalao)
- [ ] 20 minute ka shaanti ka time

---

## SSH kaise kholte hain (jahan guide me "SSH karo" likha ho)

**Windows:**
1. **Windows key** dabao, `PowerShell` type karo, **Enter**.
2. Ye likho aur Enter dabao:
   ```
   ssh root@192.168.1.1
   ```
3. Pehli baar "Are you sure..." poochega: `yes` likho.
4. **Router ka password** type karo. Type karte waqt **kuch dikhta nahi**, ye normal hai. Enter dabao.
5. `root@OpenWrt:~#` dikhe to aap router ke andar ho.

Commands ko copy karke SSH window me **right-click** karke paste karo, Enter dabao.

> **"REMOTE HOST IDENTIFICATION HAS CHANGED"** ka error aaye (firmware flash ke baad aisa hota hai) to apne PC ke PowerShell me ye chalao aur dobara SSH karo:
> ```
> ssh-keygen -R 192.168.1.1
> ```

---

## Step 0: Pehle router ka internet check karo

AdBlock-Fast ko packages aur blocklists download karne ke liye internet chahiye.

SSH me ye chalao:
```
ping -c 3 8.8.8.8
```

**Kya dikhna chahiye:** `3 packets received, 0% packet loss`. Reply na aaye to router ka internet hi band hai, pehle usse theek karo.

> Router abhi reboot ya WAN reconnect hua ho to 1-2 minute ruko, kyunki us time downloads "Operation not permitted" jaisa error de sakte hain.

---

## Step 1: Packages install karo

Router par SSH me ye chalao:

```
apk update
apk add adblock-fast luci-app-adblock-fast gawk grep sed coreutils-sort
/etc/init.d/rpcd reload
```

Phir browser me **Ctrl + F5** dabao (page refresh).

**Kya dikhna chahiye:** LuCI ke upar **Services** menu me **AdBlock-Fast** dikhta hai.

**Ye 4 extra packages (`gawk`, `grep`, `sed`, `coreutils-sort`) kyun?** Router me ye teeno ke chhote versions pehle se hote hain, par AdBlock-Fast ko bade versions chahiye. Na ho to wo panel me warning deta hai: **"Missing recommended package: 'gawk, grep, sed, coreutils-sort'"**, aur bade blocklists ke saath sahi kaam nahi karta.

**Check karo ki install ho gaye:**
```
apk info | grep -E '^(gawk|grep|sed|coreutils-sort)'
```
Chaaron ke naam dikhne chahiye.

> **Custom firmware bana rahe ho** (Firmware Selector se) to in sab ko package list me end me jod do, aur baad me alag se install nahi karna padega:
> ```
> adblock-fast luci-app-adblock-fast gawk grep sed coreutils-sort
> ```
> (Build me error aaye to wahi error batayega ki kaunsa naam hatana hai.)

---

## Step 2: AdBlock-Fast ka panel kholo

1. LuCI me **Services → AdBlock-Fast** kholo.
2. Upar **Status** section dekho.

**Abhi ye dikhna normal hai:**
- **Service Status:** `Failed to start` (kyunki koi blocklist chuni nahi hai)
- **Service Errors:** `No blocked list URLs nor blocked-domains enabled`

Ye error Step 3 aur 5 ke baad chale jayenge. Agar **Service Warnings** me `Missing recommended package: 'gawk, grep, sed, coreutils-sort'` dikhe, to Step 1 dobara karo.

<!-- ![Step 2: Status before setup](screenshots/ab-step-02-status.png) -->

---

## Step 3: Blocklist chuno (sabse zaroori faisla)

Isi page par neeche **Basic Configuration** ke baad **"Allowed and Blocked Lists URLs"** ki ek table dikhegi, har list ke saamne **Enable** ka checkbox hai.

### Meri suggestion (kam memory, kam galtiyan)
1. **Hagezi - Pro** ko **Enable** karo. Ek yehi kaafi hai.
2. **Optional:** **CERT Polska - Dangerous Websites** bhi enable karo. Ye phishing aur malware par focus karti hai, aur Hagezi ke saath kam overlap karti hai.

### Ye mat chunna (shuru me)
| List | Kyun nahi |
|---|---|
| **OISD Big, StevenBlack, 1Hosts, AdGuardTeam CNAME Trackers** | Hagezi Pro se kaafi overlap karti hain. Zyada list se memory badhti hai aur blocking me khaas fayda nahi |
| **Kboghdady - YouTube Ads DNS** | YouTube ads zyadatar DNS se block nahi hote, aur kabhi-kabhi videos atak sakte hain |
| **AdGuardTeam - CNAME Clickthroughs** | Email/newsletter ke click-tracking links toot sakte hain (jaise shopping/bank ke mail ke buttons) |

> **Zyada list = zyada memory + zyada galat block.** Pehle ek-do list se shuru karo. Baad me zaroorat lage to ek-ek jodo.

Hagezi Pro + CERT Polska lagane par is tester ke router par lagbhag **3 lakh (309086)** domains block hue.

<!-- ![Step 3: Lists table](screenshots/ab-step-03-lists.png) -->

---

## Step 4: Basic settings

Usi page par upar **Basic Configuration** tab kholo. Ye jaisa hai waisa rakho:

| Setting | Kya rakhna hai | Kyun |
|---|---|---|
| **DNS Service** | `dnsmasq servers file` | Router me jo pehle se chal raha hai uska hi use karta hai, aam taur par sabse kam memory leta hai |
| **Use ad-blocking on the dnsmasq instance(s)** | `Ad-blocking on all instances` | Saare network par block hoga |

> Is router par ye options **"not supported on this system"** dikhte hain: `dnsmasq.ipset`, `dnsmasq.nftset`, `smartdns.domainset` aur `unbound.adb_list`. Unhe chhedo mat. **`dnsmasq servers file` hi sahi hai.**

Neeche **Save & Apply** dabao.

<!-- ![Step 4: Basic configuration](screenshots/ab-step-04-basic.png) -->

---

## Step 5: Enable aur Start

1. Page ke upar **Service Control** me:
   - Pehle **Enable** dabao (taaki router restart hone par bhi chalta rahe).
   - Phir **Start** dabao.
2. Kuch second ruko. Blocklist download hogi.
3. Page refresh karo (**Ctrl + F5**).

**Kya dikhna chahiye:** Status me `Failed to start` ki jagah AdBlock-Fast ka version aur blocked domains ki sankhya dikhti hai. Service Errors me ab `No blocked list URLs...` nahi hota.

**SSH se bhi check kar sakte ho:**
```
/etc/init.d/adblock-fast status
```
Kuch aisa dikhna chahiye:
```
adblock-fast 1.2.4-r4 is blocking 309086 domains (with dnsmasq.servers)
```
(Sankhya aapki lists ke hisaab se alag hogi.)

> **Service Control me aur buttons ka matlab:** **Redownload** lists dobara download karta hai. **Pause (1m)** 1 minute ke liye block band karta hai (jab koi site toot rahi ho aur check karna ho). **Stop** band karta hai. **Disable** boot par chalna band karta hai.

<!-- ![Step 5: Service control and status](screenshots/ab-step-05-start.png) -->

---

## Step 6: Test karo

### 6A. Router se (SSH me)
```
nslookup ads.doubleclick.net 127.0.0.1
nslookup pagead2.googlesyndication.com 127.0.0.1
```
**Kya dikhna chahiye:** `server can't find ...: NXDOMAIN`. Matlab block ho raha hai.

> **Ek zaroori baat:** har ad-wala domain Hagezi Pro me block nahi hota. Jaise `doubleclick.net`, `googleadservices.com` aur `adservice.google.com` aise tester ke router par **normal IP** deke aaye (block nahi hue), jabki `ads.doubleclick.net` aur `pagead2.googlesyndication.com` block hue. Ye khaabi nahi hai. Kuch list Google ke bade domains jaan-bujhkar nahi rokti, taaki Google ki services na toote. **Isliye ek hi domain se test mat karo.**

### 6B. Laptop (Windows) se
Pehle purana DNS cache saaf karo, phir test karo (PowerShell me):
```
ipconfig /flushdns
nslookup ads.doubleclick.net
```
Jawab me `Server` me router ka naam/address dikhna chahiye, aur `NXDOMAIN` aana chahiye.

### 6C. Asli test: browser me
Koi ads wali website (news ya recipe site) kholo aur dekho ki ads kam aa rahe hain ya nahi.

**Browser/phone me ye cheezein band rakho, nahi to ads block nahi honge:**
- **Secure DNS / DNS-over-HTTPS** (Chrome: Settings → Privacy and security → Security → *Use secure DNS* → off)
- Phone me **Private DNS** (Android) ya koi VPN/Private Relay (iPhone)

Ye sab browser/phone ko router ke DNS ko bypass karne dete hain.

---

## Step 7: Memory naap lo (sachchai khud dekho)

Ye check karta hai ki AdBlock-Fast aapke router par asal me kitni memory leta hai. SSH me, ek-ek karke chalao:

```
/etc/init.d/adblock-fast stop
free -m
```
(`available` ya `free` wali value note kar lo.)

```
/etc/init.d/adblock-fast start
free -m
```
Dono ke beech ka farak lagbhag AdBlock-Fast ki memory hai. Ye sirf andaza hai, kyunki router par aur cheezein bhi chal rahi hoti hain.

Isse bhi dekh sakte ho:
```
top -bn1 | head -12
```
Isme `dnsmasq` ki memory (VSZ/RSS jaisi columns) dekh sakte ho.

> Jitni zyada blocklist, utni zyada memory. Agar memory zyada lage, to ek list kam karo (jaise CERT Polska hata do) aur **Save & Apply** karke **Redownload** karo.

---

## Step 8: Reboot test aur lists ka auto-update

### Reboot test
1. **System → Reboot → Perform reboot**.
2. 2-3 minute ruko.
3. **Services → AdBlock-Fast** me Status dekho, aur Step 6A dobara karo.

**Kya dikhna chahiye:** AdBlock-Fast bina kuch kiye chalu hai, aur `NXDOMAIN` aa raha hai.

> Restart par AdBlock-Fast aksar purani downloaded list ko cache se dobara use karta hai ("Found cache file, reusing it"), isliye har reboot par poori download nahi hoti.

### Lists ka auto-update
AdBlock-Fast ke panel me **List Updates Schedule** tab hai. Wahan lists ko hafte me ek baar update karne ka option rakh sakte ho, taaki blocklist purani na ho. Zyada baar update karne ki zaroorat nahi.

---

## Koi site toot jaye to (allowlist)

Kabhi-kabhi blocklist ek sahi site ya app ko bhi block kar deti hai (jaise login ya payment page).

1. **Services → AdBlock-Fast** me **Pause (1m)** dabao aur dekho ki site khulti hai. Khulti hai to blocklist ki galti hai.
2. Us domain ko **allowlist** me daalo. Panel me neeche **Allowed and Blocked Domains** jaisa section dikhega (naam aapke version me thoda alag ho sakta hai). Wahan domain `Allow` karke likho (jaise `example.com`).
3. **Save & Apply**.

Domain pata nahi hai to **DNS Query Log** tab (agar on ho) me dekh sakte ho ki kaunsa domain block hua. Ye log aapki browsing ka record hota hai, isliye zaroorat na ho to usse on mat rakho.

---

## Dikkat aaye to (troubleshooting)

| Dikkat | Kya karo |
|---|---|
| Status me **Failed to start** | Koi blocklist enable karo (Step 3), phir Start dabao |
| **Missing recommended package: 'gawk, grep, sed, coreutils-sort'** | Step 1 ka `apk add ...` dobara karo |
| `apk update` ya download me **Operation not permitted** | 1-2 minute ruko (WAN abhi connect ho raha hai) aur dobara try karo. `ping -c 3 8.8.8.8` se internet check karo |
| Block hi nahi ho raha | Step 6 me kai domain try karo. Browser me DoH/Private DNS band hai? Laptop par `ipconfig /flushdns` chalao |
| `doubleclick.net` block nahi hua | Normal hai (Step 6A ka note). Dusre domains se test karo |
| Koi site/app kaam nahi kar rahi | Pause karke check karo, phir allowlist me daalo |
| YouTube/Instagram ke ads abhi bhi aate hain | DNS-level blocking un ads ko aksar nahi rok pata (ads wahi servers se aate hain jahan se video aata hai). Ye is tareeke ki limit hai |
| Router ki memory kam pad rahi hai | Ek list kam karo, ya Hagezi Pro ki jagah halki list chuno. Step 7 se naapo |
| Reboot ke baad band | **Service Control → Enable** dabao |
| AdGuard Home bhi chal raha hai | Ek hi chalao. AdGuard Home band karo ya AdBlock-Fast |

**Router par ye commands chala ke dekho (SSH me):**
```
/etc/init.d/adblock-fast status
apk info | grep -E '^(adblock-fast|gawk|grep|sed|coreutils-sort)'
free -m
```

---

## Surakshit rehne ke liye aur privacy

- Packages **official OpenWrt feed** se aate hain (`apk add`), signed. Koi third-party package install karne ki zaroorat nahi.
- **Sirf jaani-maani blocklists** chuno. Anjaan source se list daalne se koi apna domain bhi block kara sakta hai, ya aapke sahi sites ko roak sakta hai.
- Ad blocking sirf **DNS** par hoti hai. Ye HTTPS ke andar ka content nahi dekhti, aur aapke browsing ka content nahi padhti.
- **DNS Query Log** aapki browsing ka record hota hai. Zaroorat ho tabhi on rakho, aur screenshots share karne se pehle usme se domains hata do.
- **Router ka password set rakho.** Bina password ke LAN par koi bhi router (aur uski settings) badal sakta hai.
- Router ka config backup (`.tar.gz`) **private** rakho. Isme WiFi password hote hain. Public jagah (GitHub, Telegram) par upload mat karo.
- Guide ya screenshots share karne se pehle apna **MAC, WiFi password, serial number** hata do.

---

## Wapas hatana ho to

1. **Services → AdBlock-Fast → Service Control** me **Stop** aur **Disable** dabao.
2. Pura hatana ho to SSH me:
   ```
   apk del luci-app-adblock-fast adblock-fast
   ```
3. Router ke DNS ko normal karne ke liye reboot kar do.

(`gawk`, `grep`, `sed`, `coreutils-sort` rakh sakte ho, wo harmless hain.)

---

## Aksar poochhe jane wale sawal

**Kya ye har device par kaam karta hai?** Jo device router ke DNS ka use karta hai (zyadatar ghar ke devices) us par haan. Jo device apna alag DNS ya VPN use karta hai wo bypass kar sakta hai.

**Kya isse internet slow hoga?** DNS ka jawab dnsmasq me hi ban jata hai, to aam taur par fark nahi padta. Par jyada badi list aur kam RAM wale router par thodi der lag sakti hai.

**Kya ye YouTube ads rokta hai?** Aksar nahi. DNS se YouTube ads rokna mushkil hai.

**AdBlock-Fast aur Tailscale ek saath chalte hain?** Maine sirf ek hi setup par dono chalaye hain (is guide ke tester ke), aur dono chal rahe the. Doosre setup me alag dikkat aa sakti hai.

**Kitni lists lagayein?** Shuru me ek (Hagezi Pro). Zaroorat lage to ek aur.

---

## English summary

**What this is:** a step-by-step guide to set up **AdBlock-Fast** on an OpenWrt router (tested on a JioRouter AX6000, OpenWrt 25.12.6, `adblock-fast` 1.2.4-r4) for network-wide DNS ad blocking with a small memory footprint, as an alternative to AdGuard Home.

**Steps:** make sure the router has internet and a password, install `adblock-fast luci-app-adblock-fast gawk grep sed coreutils-sort` with `apk`, open **Services, AdBlock-Fast**, enable **one or two** blocklists (Hagezi Pro, optionally CERT Polska), keep **DNS Service = dnsmasq servers file** and **ad-blocking on all instances**, Save and Apply, press **Enable** and **Start**, then test with `nslookup` on several ad domains (not just one), measure memory with `free -m`, and do a reboot test.

**Tips:** more lists means more memory and more false positives, so start small. Turn off DNS-over-HTTPS and Private DNS in browsers and phones or they will bypass the router. DNS blocking cannot stop most YouTube ads. Use the allowlist for sites that break.

**Compared with AdGuard Home:** AdBlock-Fast feeds the blocklist into the router's existing dnsmasq, so it is usually lighter. AdGuard Home gives per-client rules, richer statistics and encrypted upstream DNS. Do not run both at once. The ~20-25 MB figure is one user's measurement, so measure on your own router.

---

## Credits

OpenWrt aur `adblock-fast` ke developers, aur Hagezi, CERT Polska jaisi blocklist banane walon ko dhanyavaad. Ye guide ek real JioRouter AX6000 par kiye gaye setup se bani hai.
