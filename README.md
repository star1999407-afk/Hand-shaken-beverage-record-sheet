<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>手搖成癮診斷書</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/lucide-static/0.321.0/lucide.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;700;900&display=swap');
        body { font-family: 'Noto Sans TC', sans-serif; background-color: #FDFCF8; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .animate-fadeIn { animation: fadeIn 0.4s ease-out forwards; }
    </style>
</head>
<body>
    <div id="root"></div>
    <script type="text/babel">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, deleteDoc, doc } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-firestore.js";

        // 注意：在 GitHub 上使用時，請確保下方的 firebaseConfig 是正確的
        const firebaseConfig = JSON.parse(window.__firebase_config || '{}');
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = window.__app_id || 'drink-diary-github';

        const { useState, useEffect } = React;

        function App() {
            const [user, setUser] = useState(null);
            const [activeTab, setActiveTab] = useState('prescribe');
            const [historyData, setHistoryData] = useState([]);
            const [loading, setLoading] = useState(true);
            const [shopName, setShopName] = useState('');
            const [drinkName, setDrinkName] = useState('');

            useEffect(() => {
                signInAnonymously(auth);
                return onAuthStateChanged(auth, setUser);
            }, []);

            useEffect(() => {
                if (!user) return;
                const q = collection(db, 'artifacts', appId, 'users', user.uid, 'records');
                return onSnapshot(q, (snapshot) => {
                    const data = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
                    setHistoryData(data.sort((a, b) => new Date(b.date) - new Date(a.date)));
                    setLoading(false);
                });
            }, [user]);

            const handleAdd = async () => {
                if (!shopName || !drinkName || !user) return;
                await addDoc(collection(db, 'artifacts', appId, 'users', user.uid, 'records'), {
                    shop: shopName, drink: drinkName, cost: 0, 
                    date: new Date().toISOString().split('T')[0], createdAt: new Date().toISOString()
                });
                setShopName(''); setDrinkName('');
                alert("已同步至 GitHub 雲端病例庫");
            };

            if (loading) return <div className="flex h-screen items-center justify-center font-bold">同步中...</div>;

            return (
                <div className="max-w-md mx-auto p-5 pb-32 animate-fadeIn">
                    <header className="text-center py-8">
                        <h1 className="text-2xl font-black text-[#3E2723] border-2 border-black inline-block px-4 py-2 rounded-xl">GitHub 診斷書</h1>
                    </header>
                    {activeTab === 'prescribe' ? (
                        <div className="bg-white p-6 rounded-3xl shadow-lg space-y-4">
                            <input className="w-full p-3 border rounded-xl" placeholder="診療所" value={shopName} onChange={e=>setShopName(e.target.value)} />
                            <input className="w-full p-3 border rounded-xl" placeholder="藥劑品項" value={drinkName} onChange={e=>setDrinkName(e.target.value)} />
                            <button onClick={handleAdd} className="w-full bg-[#3E2723] text-white py-4 rounded-xl font-bold">確認存檔</button>
                        </div>
                    ) : (
                        <div className="space-y-3">
                            {historyData.map(item => (
                                <div key={item.id} className="bg-white p-4 rounded-2xl border flex justify-between">
                                    <div><div className="font-bold">{item.shop}</div><div className="text-sm">{item.drink}</div></div>
                                    <div className="text-xs text-stone-400">{item.date}</div>
                                </div>
                            ))}
                        </div>
                    )}
                    <nav className="fixed bottom-8 left-1/2 -translate-x-1/2 bg-[#3E2723] text-white flex p-2 rounded-full space-x-4">
                        <button onClick={()=>setActiveTab('prescribe')} className="px-6 py-2">開處方</button>
                        <button onClick={()=>setActiveTab('history')} className="px-6 py-2">病例庫</button>
                    </nav>
                </div>
            );
        }
        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>

