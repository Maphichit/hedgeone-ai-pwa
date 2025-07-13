// HedgeOne AI - Full PWA System with Dashboard, Withdraw, Countdown, Firebase // Framework: Next.js + Tailwind CSS + Firebase

'use client';

import { useEffect, useState } from 'react'; import { auth, firestore } from '@/lib/firebase'; import { onAuthStateChanged, GoogleAuthProvider, signInWithPopup, } from 'firebase/auth'; import { doc, getDoc } from 'firebase/firestore';

export default function Home() { const [user, setUser] = useState(null); const [profit, setProfit] = useState(0); const [wallet, setWallet] = useState(0); const [timeLeft, setTimeLeft] = useState(''); const [installPrompt, setInstallPrompt] = useState(null);

useEffect(() => { const unsubscribe = onAuthStateChanged(auth, async (currentUser) => { setUser(currentUser); if (currentUser) { const userRef = doc(firestore, 'users', currentUser.uid); const snap = await getDoc(userRef); if (snap.exists()) { const d = snap.data(); setProfit(d.profitToday || 0); setWallet(d.wallet || 0); const expiry = new Date(d.trialEndsAt); const now = new Date(); const hoursLeft = Math.max(0, Math.floor((expiry - now) / 3600000)); setTimeLeft(${hoursLeft} ชั่วโมง); } } }); window.addEventListener('beforeinstallprompt', (e) => { e.preventDefault(); setInstallPrompt(e); }); return () => unsubscribe(); }, []);

const installApp = () => { if (installPrompt) { installPrompt.prompt(); installPrompt.userChoice.then(() => setInstallPrompt(null)); } };

const signIn = () => { signInWithPopup(auth, new GoogleAuthProvider()); };

const withdraw = () => { if (wallet < 500) { alert('❌ ถอนขั้นต่ำ 500 บาท'); return; } alert('📤 กำลังดำเนินการถอนเงิน...'); };

return ( <main className="p-6 max-w-xl mx-auto font-sans"> <div className="bg-white p-4 rounded-xl shadow-md space-y-4"> <h1 className="text-2xl font-bold text-indigo-600">🚀 HedgeOne AI Dashboard</h1> <p className="text-sm text-gray-600">ระบบเทรดทองคำอัจฉริยะ พร้อมถอนได้ทุกวัน</p>

{user ? (
      <div>
        <p className="text-green-600">👋 สวัสดี {user.displayName}</p>
        <p className="text-sm text-gray-600">กระเป๋า: ฿{wallet.toFixed(2)}</p>
      </div>
    ) : (
      <button onClick={signIn} className="bg-black text-white py-2 px-4 rounded">
        เข้าสู่ระบบด้วย Google
      </button>
    )}

    {installPrompt && (
      <button onClick={installApp} className="w-full border py-2 rounded">
        📲 ติดตั้งแอป HedgeOne
      </button>
    )}

    <div className="bg-gray-50 p-3 rounded">
      <p className="text-sm font-semibold">📈 กำไรวันนี้</p>
      <p className="text-lg text-green-600">+{profit}%</p>
    </div>

    <div className="bg-yellow-50 p-3 rounded">
      <p className="text-sm font-semibold">🕒 ทดลองใช้งานฟรี</p>
      <p className="text-sm text-red-500">เหลือเวลา: {timeLeft}</p>
    </div>

    <button onClick={withdraw} className="w-full bg-blue-600 text-white py-2 rounded">
      📤 ถอนเงิน
    </button>
  </div>
</main>

); }

